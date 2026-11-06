# AI

The AI module adds simple melee enemies, run by the server: they wander around their home, notice the players they
see or hear, chase and hit them, break walls in their way, search around the last place they saw a player who hides,
and go home when they give up. A spawn director
keeps a few enemies around each player, so the size of the map never changes how many exist.

The module is optional: add `UST_DISABLE_AI` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module). It needs the **AI Navigation** package.

## Setup

1. Bake a NavMesh in your scene: a `NavMeshSurface` on the ground, then **Bake**. *Tools > UST > Advanced > Demo > Create Demo Island* builds a test map with its NavMesh, see [Demo island](#demo-island).
2. Right click a character model (an FBX with a humanoid Animator, not a full prefab with scripts) in the Project
   window or the Hierarchy > **UST > Create Enemy From Model**. It makes a prefab in `Modules/AI/Prefabs` with a `NetworkObject`, a
   `NetworkTransform`, a capsule, a `NavMeshAgent`, `EnemyHealth` and `EnemyAI`. Without a controller on the model, it
   uses the controller of the player, so the same animations work at once.
3. Run *Tools > UST > Validation > Validate Network Prefabs*.
4. Add the spawn director: [Setup Window](../getting-started/setup-window.md) > **AI > Add**, or *Tools > UST > Advanced > AI > Add Enemy Spawner To Scene*:
   a director with every enemy prefab of the project.

!!! tip "No enemy model yet?"
    Use your player model with another material for tests. For a free model safe for the Asset Store, prefer CC0 packs
    (Quaternius, Kenney) over Mixamo characters.

## Enemy definition

Every stat of a kind of enemy lives in an **Enemy Definition** asset (*Create > UST > AI > Enemy Definition*). Several
prefabs can share one: a zombie with three models is three prefabs and one definition, balanced in one place. The
prefab only holds the model, the animator and the components. *Create Enemy From Selected Model* makes the definition
with the prefab; for an older prefab, right click it > **UST > Create Enemy Definition**.

An enemy without a definition still works with the default values, and logs a warning.

| Field | Default | Meaning |
|---|---|---|
| Display Name, Icon, Description | | For your UI: bestiary, target frame |
| **Health** | | |
| Max Health | 50 | |
| Default Hit Damage | 5 | Damage taken from a melee hit by an item without damage of its own |
| Resistances | | By damage type, see [Damage types](combat/index.md#damage-types) |
| Loot | | Items dropped on death, see [Loot](#loot) |
| Despawn Delay | 5 | Seconds the body stays |
| **Senses** | | |
| Sight Range, Sight Angle | 18, 140 | A player in this cone and in line of sight is noticed |
| Hearing Range | 4 | A player this close is noticed, even behind |
| Lose Range | 30 | Distance reset: the chase stops past this distance. In every mode, a chased target stays visible up to this distance or the Sight Range |
| Eye Height | 1.6 | |
| Sight Blockers | Everything | Layers that block the view |
| **Movement** | | |
| Walk Speed, Run Speed | 1.5, 4 | Wandering and returning, chasing |
| Wander Radius, Wander Pause | 10, 2 to 6 s | Around home |
| Leash Range | 45 | Distance reset: the enemy goes home when its target is past this distance from home |
| **Chase** | | |
| Chase Reset | Both | What makes the enemy give up: **Distance** (Lose Range, Leash Range), **Timer** (Lose Sight Time) or **Both**, whichever comes first |
| Lose Sight Time | 10 | Timer reset: seconds without seeing or hearing the target |
| Search Points | 3 | Points visited around the last place the target was seen. 0: only that place |
| Search Radius | 6 | Radius of the search around that place |
| Search Pause | 1 to 2.5 s | Time spent looking around at each point |
| Heal On Return | on | Back to full health after giving up, so players cannot wear an enemy down by pulling it again and again. With Ignore Players On Return: at once, when it gives up. Without: once home. |
| Ignore Players On Return | on | On its way home, the enemy notices no player, except one who hits it |
| **Attack** | | |
| Attack Range, Attack Damage, Attack Cooldown | 1.8, 15, 1.6 | |
| Attack Damage Type | | Resisted by armor and built pieces. Empty: untyped. |
| Attack Delay | 0.5 | Seconds between the start of the animation and the hit |
| Attack Obstacles | on | Breaks walls and other obstacles in the way of its target |
| Stuck Time | 1.5 | Seconds without progress before it looks for something to break |
| Obstacle Give Up Time | 6 | Seconds of attacks on the same obstacle before giving up, only for obstacles that do not implement `IBreakable` |
| **Performance** | | |
| Think Interval | 0.2 | Seconds between two decisions |

### Loot

Each **Loot** entry is rolled on its own when the enemy dies, on the server: **Chance** (0 to 1) decides whether it
drops, then a count is picked between **Min** and **Max**, both included. The items appear as world items around the
body, split into full stacks, at full durability. An item needs a **World Prefab** with a `WorldItem` and its Item
Database, or it is skipped.

## Enemy AI

`EnemyAI` runs the behaviour from the definition: wander, notice, chase, attack, break obstacles, search, go home.

The states, readable through `EnemyAI.Current`:

| State | What the enemy does |
|---|---|
| Idle, Wander | Walks to random points around its home, with pauses. Notices players in its view cone and line of sight, or within hearing range. |
| Chase, Attack | Runs to its target and hits it. It keeps the target while it hears it or has it in line of sight (no view cone: it is looking for it). |
| Search | The target hid: the enemy runs to the last place it saw it, looks around, then walks to **Search Points** random points around that place and looks around at each one. Seeing or hearing any player again restarts the chase. |
| Return | Gave up: walks home. With **Ignore Players On Return**, it notices no player unless one hits it, and **Heal On Return** heals it at once; without, it can notice players again and heals once home. |
| Dead | |

An enemy never tracks a player through walls: hiding works. Only the way to the last seen place may be forced open,
so a player who runs into a closed base still gets its walls broken, but random search points never make the enemy
break anything. A hit from a player the enemy cannot see sends it searching around that player. `EnemyAI.LastSeenPosition`
gives the last seen place, and the search radius is drawn as a gizmo while the enemy is selected. An
obstacle is only chosen when a swing can reach it and when it can be damaged: built pieces tell it through
`IBreakable`, so an enemy never swings forever at a scene piece, a warded piece or a wall the Building Rules protect.
An obstacle that moves out of reach, or that stops accepting damage, is dropped and ignored for a while. On the
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
`PieceHealth`, with the source **Enemy** (see the Enemy Damage option of the [Building rules](building/rules.md#building-rules)).
When the path to its target is blocked, by a closed base for example, the enemy hits the damageable thing in front of
it. Being hit by a player makes the enemy chase that player.

## Chase, search and reset

### How a chase goes

1. **Notice.** An idle or wandering enemy notices a player in its view cone (**Sight Range**, **Sight Angle**) and in
   line of sight (**Sight Blockers**), or any player within **Hearing Range**, even behind it or behind a wall. Being
   hit by a player also starts a chase.
2. **Chase.** Every **Think Interval**, the enemy checks that it still hears its target or has it in line of sight.
   There is no view cone during a chase: the enemy is looking for its target. A chased target stays visible up to the
   **Sight Range** or the **Lose Range**, whichever is larger. Each time the target is perceived, the enemy remembers
   where it is (`EnemyAI.LastSeenPosition`) and when.
3. **Search.** When the target is no longer perceived, the enemy runs to the last seen place and looks around for
   **Search Pause**. Then it walks to **Search Points** random points within **Search Radius** of that place, on the
   NavMesh, and looks around at each one. Any player seen (in the view cone) or heard during the search restarts the
   chase on that player.
4. **Reset.** The enemy gives up and goes home when the **Chase Reset** says so, or when every search point is visited.

### Chase Reset

| Mode | Gives up when |
|---|---|
| Distance | The target is past **Lose Range** from the enemy, or past **Leash Range** from home. Search points are also kept inside the leash. |
| Timer | The enemy has not seen or heard its target for **Lose Sight Time**. Distance never ends the chase: an enemy that keeps its target in view follows it anywhere. |
| Both (default) | Whichever comes first. |

In every mode the search also ends once every search point is visited, so **Search Points** and **Search Pause** set
how long an enemy keeps looking when the timer is long or off.

### Going home

| Ignore Players On Return | Heal On Return | Result |
|---|---|---|
| on (default) | on (default) | Notices nobody on the way home, except a player who hits it, and is back to full health at once, when it gives up. Pulling it again always finds it full: no kiting. |
| on | off | Notices nobody on the way home, keeps its damage. |
| off | on | Can notice players again on the way home, heals once it gets home. A player who pulls it again before it gets there finds it hurt: kiting is possible. |
| off | off | Can notice players again, keeps its damage. |

### Hiding and bases

- An enemy never tracks a player through walls: breaking line of sight and getting out of hearing range works.
- On its way to the last seen place, the enemy breaks what blocks it when **Attack Obstacles** is on, exactly as
  during a chase. A player who runs into a closed base still gets its walls broken.
- Random search points never make it break anything: they are only places to look.
- A point it cannot reach is dropped after a time based on its distance and the speed, so the search never hangs.

### Several players

- When noticing, the enemy takes the closest player it sees or hears.
- During a chase it stays on its target. A player who hits it becomes its target, even out of sight: the enemy then
  searches around that player.
- During a search it looks for any player, not only the lost one.
- If its target dies, it looks for another player and goes home if there is none. If its target disconnects, it
  finishes its search, then goes home.
- The timer and the last seen place belong to each enemy and follow its current target.

Everything runs on the server. Clients receive the position and the Y rotation through the `NetworkTransform`, and the
health through the `NetworkVariable` of `EnemyHealth`: the look-around and the heal on reset need nothing else.

### Example settings

| Enemy | Chase Reset | Lose Sight Time | Search Points | Search Radius | Ignore Players On Return |
|---|---|---|---|---|---|
| Slow zombie that forgets fast | Both | 6 | 1 | 4 | on |
| Hunter that tracks you down | Timer | 25 | 5 | 12 | off |
| Guard bound to its post | Distance (short Leash Range) | | 2 | 5 | on |
| Mindless creature | Distance | | 0 | | on |

### Scripting

| Member | Use |
|---|---|
| `EnemyAI.Current` | Current state, for UI or sounds: an alert icon on Search, a growl on Chase |
| `EnemyAI.Target` | Current target, or null |
| `EnemyAI.LastSeenPosition` | Server only. Where the target was last seen or heard |
| `EnemyAI.ServerAggro(Transform)` | Server only. Starts a chase on this target, for a noise, a trap or a horn |
| `EnemyHealth.ServerRestoreHealth()` | Server only. Back to full health |

`State` is only known on the server: to show it on clients, send it yourself, with a `NetworkVariable` in a component
of yours.

### Cost

While chasing or searching, an enemy casts one sight ray per decision (every **Think Interval**), and a search runs
the same check as wandering for every player. With the default 0.2 s, that is 5 rays per second per alert enemy:
nothing to speak of next to the NavMeshAgent.

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
| Despawn On Time Skip | on | With Night Only: the living enemies of the director are removed when the night is skipped by [sleeping](sleep.md) |
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

*Tools > UST > Advanced > Demo > Create Demo Island* builds a small test map in the open scene, centered on the origin: a 300 m
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
