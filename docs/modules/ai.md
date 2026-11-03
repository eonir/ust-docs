# AI

The AI module adds simple melee enemies, run by the server: they wander around their home, notice the players they
see or hear, chase and hit them, break walls in their way, and go home when the players get away. A spawn director
keeps a few enemies around each player, so the size of the map never changes how many exist.

The module is optional: add `UST_DISABLE_AI` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module). It needs the **AI Navigation** package.

## Setup

1. Bake a NavMesh in your scene: a `NavMeshSurface` on the ground, then **Bake**. *UST Tools > Demo > Create Demo
   Island* builds a test map with its NavMesh, see [Demo island](#demo-island).
2. Select a character model (an FBX with a humanoid Animator, not a full prefab with scripts) and run *UST Tools > AI >
   Create Enemy From Selected Model*. It makes a prefab in `Modules/AI/Prefabs` with a `NetworkObject`, a
   `NetworkTransform`, a capsule, a `NavMeshAgent`, `EnemyHealth` and `EnemyAI`. Without a controller on the model, it
   uses the controller of the player, so the same animations work at once.
3. Run *UST Tools > Validation > Validate Network Prefabs*.
4. Run *UST Tools > AI > Add Enemy Spawner To Scene*: a director with every enemy prefab of the project.

!!! tip "No enemy model yet?"
    Use your player model with another material for tests. For a free model safe for the Asset Store, prefer CC0 packs
    (Quaternius, Kenney) over Mixamo characters.

## Enemy definition

Every stat of a kind of enemy lives in an **Enemy Definition** asset (*Create > UST > AI > Enemy Definition*). Several
prefabs can share one: a zombie with three models is three prefabs and one definition, balanced in one place. The
prefab only holds the model, the animator and the components. *Create Enemy From Selected Model* makes the definition
with the prefab; for an older prefab, select it and run *UST Tools > AI > Create Definition For Selected Enemy*.

An enemy without a definition still works with the default values, and logs a warning.

| Field | Default | Meaning |
|---|---|---|
| Display Name, Icon, Description | | For your UI: bestiary, target frame |
| **Health** | | |
| Max Health | 50 | |
| Default Hit Damage | 5 | Damage taken from a melee hit by an item without damage of its own |
| Resistances | | By damage type, see [Damage types](combat.md#damage-types) |
| Loot | | Items dropped on death: item, min, max, chance |
| Despawn Delay | 5 | Seconds the body stays |
| **Senses** | | |
| Sight Range, Sight Angle | 18, 140 | A player in this cone and in line of sight is noticed |
| Hearing Range | 4 | A player this close is noticed, even behind |
| Lose Range | 30 | The chase stops past this distance |
| Eye Height | 1.6 | |
| Sight Blockers | Everything | Layers that block the view |
| **Movement** | | |
| Walk Speed, Run Speed | 1.5, 4 | Wandering and returning, chasing |
| Wander Radius, Wander Pause | 10, 2 to 6 s | Around home |
| Leash Range | 45 | The enemy goes home past this distance from home |
| **Attack** | | |
| Attack Range, Attack Damage, Attack Cooldown | 1.8, 15, 1.6 | |
| Attack Damage Type | | Resisted by armor and built pieces. Empty: untyped. |
| Attack Delay | 0.5 | Seconds between the start of the animation and the hit |
| Attack Obstacles | on | Breaks walls and other obstacles in the way of its target |
| Stuck Time | 1.5 | Seconds without progress before it looks for something to break |
| **Performance** | | |
| Think Interval | 0.2 | Seconds between two decisions |

## Enemy AI

`EnemyAI` runs the behaviour from the definition: wander, notice, chase, attack, break obstacles, go home. On the
prefab it only keeps the **Animator** and the names of its parameters, the same as the player controller, so an enemy
made from the player model reuses its Animator Controller.

| Field | Default | Set to |
|---|---|---|
| Animator | | The Animator of the model |
| Speed Parameter | Velocity | Float: current speed, smoothed, in m/s |
| Forward Parameter | MoveY | Float: speed divided by the run speed of the definition, 0 to 1 |
| Grounded Parameter | Grounded | Bool: true from the spawn, so the locomotion plays instead of a fall |
| Attack Trigger | Attack | Trigger: each attack |
| Dead Parameter | Dead | Bool: true on death |

Leave a name empty to skip that parameter, for a controller that does not have it. Each peer computes the speed
parameters from the movement it sees, and the server sends attacks and deaths by RPC: the enemy needs no
`NetworkAnimator`, which keeps the network traffic per enemy low.

Hits go through `IDamageable` in UST.Core: players take them through `PlayerDamageReceiver`, built pieces through
`PieceHealth`, with the source **Enemy** (see the Enemy Damage option of the [Building rules](building.md#building-rules)).
When the path to its target is blocked, by a closed base for example, the enemy hits the damageable thing in front of
it. Being hit by a player makes the enemy chase that player.

## Enemy health

`EnemyHealth` holds the **Definition** of the enemy. Damage taken is multiplied by the resistances of the definition
and by the `IDamageModifier` components of the enemy. Projectiles deal the damage of the weapon, melee hits the damage
of the **Melee Weapon** trait (any trait implementing `IItemDamage`). `EnemyHealth.ServerKilled` tells who made the
kill, for quests or experience.

## Spawn director

`EnemySpawnDirector` keeps enemies alive around the players: they appear in a ring between **Min Distance** and **Max
Distance** of a player, out of sight, and despawn when every player is further than **Despawn Distance**.

| Field | Default | Meaning |
|---|---|---|
| Enemies | | Prefabs and their weight |
| Zone | | Optional collider: enemies only spawn inside. One director per zone: forest, swamp... |
| Max Per Player, Max Total | 5, 25 | |
| Interval | 4 | Seconds between two checks. Each check spawns at most one enemy per player and despawns the far ones: lower fills the area faster. |
| Min Distance, Max Distance | 30, 60 | |
| Despawn Distance | 90 | |
| Out Of Sight | on | No spawn in front of a player |
| Avoid Bases | on | No spawn inside wards and station build ranges. Bases are safe from spawns, not from attacks. |
| Night Count Multiplier | 1.5 | Multiplies Max Per Player and Max Total in full night, with a [day and night cycle](weather.md) |
| Night Only | off | Spawns only at night, for creatures of the dark. They stay at dawn until far from players. |
| Nav Mesh Search | 4 | A random spawn point moves to the closest walkable NavMesh point within this distance. None that close: another point is tried. |

## Performance

- The AI and the NavMeshAgent run on the server only. Clients receive the position and play the animations from the
  movement they see, so no speed is sent.
- Decisions happen every **Think Interval**, not every frame.
- The director keeps only the enemies near players.
- The animator of the enemy prefab is set to **Cull Update Transforms**: it stops when off screen.
- Built pieces do not carve the NavMesh: enemies walk up to walls and break them.
- The `NetworkTransform` of an enemy made by the tool only sends the position and the Y rotation, in half precision,
  and only when they change by more than 2 cm or 2°. For an enemy made by hand, set the same.
- The director itself costs nothing to speak of: one check every **Interval**. What costs is the number of enemies
  alive, so **Max Per Player** and **Max Total** are the settings to lower on weak hardware.

### Scaling up

The defaults keep about 5 enemies around each player. Raising the caps is possible; here is what grows, from the most
to the least likely to limit you:

1. **Network**: each moving enemy sends its position and Y rotation. With the optimized `NetworkTransform`, expect
   around 50 to 60 KB/s per client for 100 moving enemies. A host sends that to every client: fine on LAN, heavy on a
   home connection with a low upload.
2. **Rendering**: every visible enemy is a skinned mesh with an Animator. Low poly models scale far better. Off
   screen, culling makes them almost free.
3. **AI logic**: decisions run at 5 Hz, spread over frames, and NavMeshAgents are native: around a millisecond for
   100 enemies.
4. **`Update` calls**: one per enemy, light. The overhead of separate `Update` methods only shows with thousands of
   objects, so a central manager would not help at these numbers.

To measure on your target hardware:

1. Set **Max Per Player** and **Max Total** to the count you want, and **Interval** to 0.5 to fill quickly.
2. Open the Profiler (CPU and Rendering) and the Network Profiler of Netcode for GameObjects, with one client
   connected.
3. Read the milliseconds per frame and the bytes sent.

If it does not hold, the fixes are on the network and the rendering: send far enemies less often, give models LODs,
lower the caps. An AI level of detail (far enemies thinking once a second) is planned for a later version.

## Demo island

*UST Tools > Demo > Create Demo Island* builds a small test map in the open scene, centered on the origin: a 300 m
island with beaches, grass and rocky slopes, a sea, 140 demo trees, and a baked NavMesh with the sea made not
walkable. An object named *Floor* is turned off. The assets go to `Assets/UST/Demo/Island`. Rebake the
`NavMeshSurface` on *Demo Island* after moving things.

It is a test map: for a real map, see the tips below.

- Use terrain trees for the forest you see, and resource nodes only where players gather.
- Share a few materials, give rocks LODs, and bake occlusion culling.
- Keep the map small and dense rather than large and empty.

## Not in this version

- Ranged enemies, groups, fleeing.
- Saving enemies: they are spawned again around players.
