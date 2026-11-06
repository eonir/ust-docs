# Vehicle

The Vehicle module adds drivable vehicles for several players: a driver and passengers, raycast wheel physics, fuel,
health, crashes, running over enemies, repairs, a trunk, a HUD and saving. Every kind of vehicle is one **Vehicle
Definition** asset: a sports car, a family car and a truck come ready to drive, and a new one is a copy of the closest
preset with a few values changed.

The module is optional: add `UST_DISABLE_VEHICLE` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module). Boats are planned for a later version.

## Quick test

1. Open a playable scene, for example the demo scene, and save it.
2. Run *Tools > UST > Advanced > Vehicle > Create Vehicle Test Scene*.
3. Press Play, walk to a vehicle and press **Interact**.

The tool copies the open scene to `Modules/Vehicle/Demo/Vehicle Test.unity` and leaves the original as it was. In
the copy it:

- builds a flat test track 700 m away from the map, with ramps, a jump, speed bumps, a hill, a side slope and a
  slalom;
- turns off the spawn points of the map and adds one on the track;
- creates the three default vehicles if they are missing and parks them at the spawn;
- adds the **Vehicle Occupant** to the player prefab, the vehicle HUD, the vehicles saver when the scene has a save
  manager, and a **Vehicle Test Kit** that gives every player a Wrench and 3 Jerrycans;
- adds the scene to the Build Settings, as Netcode scene management needs.

## Setup in your own scene

1. *Tools > UST > Advanced > Vehicle > Add Vehicles To Selected Player* on the player prefab (or the fix in Validate Scene).
2. *Tools > UST > Advanced > Vehicle > Create Default Vehicles*, or *Create Vehicle From Selected Model* for your own model.
3. Place vehicles in the scene. Vehicles placed in the scene need no registration in the Network Prefabs list.
4. *Tools > UST > Advanced > UI > Add Vehicle HUD* (also added by *Create Game UI*).
5. *Tools > UST > Advanced > Vehicle > Add Vehicles Saver To Scene* if the scene is saved.
6. Give players a repair tool and fuel: see [Repair](#repair) and [Fuel](#fuel).

To spawn vehicles at runtime (bought in a shop, crafted, built), register their prefabs with *Validate Network
Prefabs* and list them in the **Vehicles Saver**, so a load can spawn them again.

## Controls

| Input | In a vehicle |
|---|---|
| Interact | Get in (aiming at the vehicle), get out |
| Move | Throttle and steering. Back brakes while rolling forward, then reverses once stopped |
| Jump | Handbrake: brakes the rear wheels, and slides them when the **Handbrake Grip** of the definition is under 1 |

The controls reuse the actions of the `PlayerInputReader`, so rebinding them rebinds the vehicles too. Getting in
picks the driver seat when it is free, the first free passenger seat otherwise. The camera follows the vehicle at
the **Camera Distance** of the definition and goes back to the player on the way out.

## Vehicle Definition

*Create > UST > Vehicle > Vehicle Definition*. Several prefabs can share one.

| Field | Default | Meaning |
|---|---|---|
| Display Name, Icon, Description | | For your UI and the prompts |
| **Engine** | | |
| Engine Power | 90 kW | How fast the vehicle gets to speed. 75 for a family car, 250 for a sports car |
| Max Speed, Max Reverse Speed | 140, 25 km/h | |
| Brake Strength | 1 g | Braking deceleration |
| Coast Deceleration | 1.2 m/s² | Slowing down with no pedal pressed |
| Drive Type | Front Wheel | Front, Rear or All Wheel |
| **Steering** | | |
| Max Steer Angle | 32° | At low speed |
| Steer Angle At Max Speed | 10° | Lower is more stable at high speed |
| Steer Speed | 160°/s | How fast the wheels turn |
| **Grip** | | |
| Grip | 1 | Tire friction, as a multiple of the weight on the tire. 0.7 heavy truck, 1.3 racing tires |
| Side Stiffness | 14 | How fast a tire kills its sideways slide. Higher feels planted, lower feels loose |
| Handbrake Grip | 1 | Side grip of the rear wheels with the handbrake. 1: the handbrake only brakes. Lower lets the rear slide: 0.7 for a sports car, 0.4 for an arcade drift |
| Anti Roll | 0.4 | Leans less in turns. 0: off |
| **Body** | | |
| Mass | 1300 kg | |
| Center Of Mass | (0, 0.3, 0.1) | From the pivot, on the ground under vehicles made by the tools. Lower is harder to roll over |
| Auto Flip Delay | 3 s | Puts the vehicle back on its wheels after this long upside down and nearly stopped. 0: never |
| **Wheels and suspension** | | |
| Wheel Radius | 0.36 m | |
| Suspension Length | 0.3 m | Travel |
| Suspension Stiffness | 1 | 1 rests at half travel. Higher sits higher and feels firmer |
| Suspension Damping | 0.45 | Fraction of critical damping. 0.3 bounces a little, 1 never bounces |
| Ground Layers | Everything | Layers the wheels drive on |
| **Fuel** | | |
| Fuel Capacity | 50 L | 0: no fuel needed |
| Start Fuel | 1 | Fraction of the tank for a new vehicle |
| Idle Consumption, Full Consumption | 0.1, 1.5 L/min | Engine running and standing still, at top speed |
| **Health** | | |
| Max Health | 500 | |
| Resistances | | By damage type, see [Damage types](combat.md#damage-types) |
| Default Hit Damage | 5 | Damage from a melee hit by an item without damage of its own |
| Damage From Players | off | Weapons and tools of players damage it. Repairs always work |
| Damage From Enemies | on | Enemies damage it, for example when it blocks their way |
| **Crashes** | | |
| Crash Safe Speed | 18 km/h | Impact speed below which a crash does nothing |
| Crash Damage Per Kmh | 3 | Damage to the vehicle per km/h above the safe speed |
| Ramming Damage Per Kmh | 1.5 | Damage to an enemy or a player run over, per km/h. 0: none |
| Ramming Min Speed | 12 km/h | |
| Ramming Damage Type | | Empty: untyped |
| **Camera** | | |
| Camera Distance | 7 m | Third person camera distance while in the vehicle |

The settings are scaled by the mass, so a stiffness of 1 or a side stiffness of 14 feel the same on a kart and on a
truck: change the mass without retuning the rest.

### The three presets

| | Sport Car | Family Car | Truck |
|---|---|---|---|
| Power, top speed | 260 kW, 220 km/h | 95 kW, 160 km/h | 220 kW, 110 km/h |
| Drive | Rear | Front | All |
| Mass | 1250 kg | 1400 kg | 4500 kg |
| Grip, side stiffness | 1.25, 18 | 1, 14 | 0.85, 10 |
| Suspension | 0.2 m, firm | 0.3 m | 0.4 m |
| Seats, trunk slots | 2, 8 | 4, 16 | 2, 40 |
| Fuel tank | 60 L | 50 L | 120 L |
| Health | 400 | 500 | 900 |

### Tuning tips

| Feels | Change |
|---|---|
| Slow to pick up speed | Engine Power up |
| Spins the wheels at launch, slides out of turns | Grip up, or Engine Power down on a rear drive |
| Floaty, drifts in every turn | Side Stiffness up |
| Too twitchy at speed | Steer Angle At Max Speed down |
| Rolls over in turns | Center Of Mass Y down, Anti Roll up |
| Bounces after a bump | Suspension Damping up |
| Body hits the ground | Suspension Stiffness up, or Suspension Length up |
| Stops dead when releasing the pedal | Coast Deceleration down |
| Handbrake spins the car | Handbrake Grip up, 1 to only brake |

## Vehicle prefab

| Component | On | Role |
|---|---|---|
| `NetworkObject` | Root | **Dont Destroy With Owner** on: the vehicle stays when its driver leaves the game |
| `Rigidbody` | Root | Mass and center of mass come from the definition |
| `NetworkTransform` | Root | **Authority Mode: Owner**, scale not synced |
| `NetworkRigidbody` | Root | Keeps the rigidbody kinematic where the vehicle is not simulated |
| `VehicleMotor` | Root | The wheels and their physics |
| `Vehicle` | Root | Seats, fuel, health, damage, repairs, save |
| `VehicleSeat` | Children | One per seat, the driver seat checked |
| `InventoryContainer` + `StorageContainer` | *Trunk* child with a trigger | The trunk |
| `VehicleFuelCap` | *Fuel Cap* child with a trigger | Refuelling |
| Colliders | Body | At least one solid collider. The wheels have none |

### Vehicle

| Field | Default | Meaning |
|---|---|---|
| Definition | | The Vehicle Definition |
| Camera Target | the vehicle | What the camera follows while inside |
| Seated Parameter | Seated | Animator bool set on a seated character. Skipped when the controller has no such bool |
| Enter Range | 2.5 m | From the closest point of the vehicle |
| Rammed Layers | player and enemy layers | The vehicle body goes through these layers instead of stopping dead, and runs over what is on them. The tools fill it with the layers of the player prefab and of the enemy prefabs, never Default. Nothing: players and enemies are solid obstacles |
| Drive, Enter, Full, Wrecked prompts | | `{0}` is the display name |

### Vehicle Motor: wheels

Each wheel has a **Mount**, a **Visual** and an **Axle**. The mount is the top of the suspension: the wheel center sits
**Suspension Length** below it at full extension, and its X position tells left from right for the anti-roll. Front
wheels steer; the **Drive Type** of the definition picks the driven ones. The visual is placed, steered and spun on
every peer, from what that peer sees, so it may be any child of the vehicle. Three wheels or more.

Selected in the scene, the motor draws each suspension (orange front, blue rear), each wheel at full extension and the
center of mass in magenta.

### Vehicle Seat

| Field | Meaning |
|---|---|
| Driver | This seat drives. One per vehicle |
| Exit Point | Where the player gets out. Empty: on the side of the seat. When it is blocked, the other side, the front, the back and the roof are tried |
| Hide Occupant | Hides the seated character, for a closed cabin. On for every seat of the default vehicles, which have no sitting animation |
| Seated Parameter | Animator bool for this seat only, for example a different pose for the driver. Empty: the one of the vehicle |

The seat transform is where the feet of the character go, facing forward: the offset between the feet and the pivot
of the character is read from its `CharacterController`, so any pivot works. The exits are measured at the feet too.
With a sitting animation, put the seat where the feet are in that pose. The gizmo shows the seat (orange driver, blue passengers) and its exit point.

### Vehicle Occupant (player)

| Field | Meaning |
|---|---|
| Input Reader | Move, Jump and Interact are read from it |
| Animator | For the seated parameter |
| Disable While Seated | Turned off on the owner while seated. The setup menu lists the item user, the hotbar, the builder, the ranged weapon, its aim zoom and the consumer. The controller, the interactor and the camera zoom are handled already |
| Exit Delay | 0.4 s: the press that got in never gets out at once |

While seated, on every peer, the character is held on its seat, its `CharacterController` and its foot IK are off and
it ignores the colliders of the vehicle. A component turned off by something else meanwhile, such as death, is left as
it is on the way out, and a player who dies in a vehicle gets back control at respawn.

## Damage, crashes and wrecks

- **Crashes**: an impact faster than the **Crash Safe Speed** damages the vehicle. Detected where the physics run,
  applied by the server.
- **Running over**: hitting anything damageable (an enemy, a player) at more than the **Ramming Min Speed** deals
  **Ramming Damage Per Kmh** times the speed, at most once every half second per target. On the **Rammed Layers** the
  vehicle drives through its victims; on other layers they stop it like a wall.

!!! warning "Rammed Layers and the ground"
    Never put the layer of the ground in **Rammed Layers**: the vehicle would fall through it. With a player or enemies
    on the Default layer, move them to a layer of their own first, or leave the field empty. The damage comes from the driver, with the source **Player**: kills
  are credited to them, and player victims follow the PvP option of their `PlayerDamageReceiver`.
- **Weapons and enemies**: through `IItemHitReceiver` and `IDamageable`, reduced by the resistances of the definition
  and the `IDamageModifier` components of the vehicle. The vehicle implements `IBreakable`, so enemies only swing at
  it when it can take their damage.
- **Wrecked** at 0 health: everyone gets out, it cannot be entered or driven, and its prompt says so. Any repair makes
  it drivable again.

## Repair

Add a **Repair Tool** trait (*Vehicle/Repair Tool*) to an item that also has a **Melee Swing** trait, then swing it at
the vehicle.

| Field | Default | Meaning |
|---|---|---|
| Repair Amount | 50 | Health per hit |
| Repair Percent | 0 | Extra health per hit, in percent of the max health |
| Cost | | Items taken on each hit, for example scrap metal. Empty: free |
| Durability Loss Per Hit | 1 | When the item has durability |

The player gets a message after each hit: the health reached, "needs no repair", or the missing cost. The Wrench of
the test scene repairs 50 per hit for free.

## Fuel

Add a **Fuel** trait (*Vehicle/Fuel*) to an item: a jerrycan, a fuel can.

| Field | Default | Meaning |
|---|---|---|
| Litres | 10 | Fuel given by one item |
| Empty Item | | Given back once used, for example an empty jerrycan. Empty: used up |

Interact with the **Fuel Cap** of the vehicle (left rear on the default vehicles): one fuel item is taken from the
hotbar or the inventory. The server burns fuel while someone drives, from **Idle Consumption** standing still to
**Full Consumption** at top speed. Out of fuel, the engine stops pushing; the vehicle still rolls, steers and brakes.
A definition with a **Fuel Capacity** of 0 never needs fuel, and its fuel cap shows no prompt.

## Trunk

The *Trunk* child is a regular storage, shared by every player, opened with **Interact** by aiming at the back of the
vehicle. Its **Access Range** is 4 m on the default vehicles. A trunk in a ward follows the ward, like any storage.

## Multiplayer

- The driver owns the vehicle while driving: the physics run on their machine, so the driving has no lag for them.
  The other players see it through the `NetworkTransform`.
- Nobody driving: the server owns it and runs its physics. It parks with its brakes on, and still reacts to being
  pushed or hit.
- Getting out of the driver seat gives the vehicle back to the server: the driver sends its exact position and velocity
  with the request, so the vehicle keeps rolling from where the driver saw it, without a jump back. A new driver
  taking over a rolling vehicle keeps its speed too.
- The server keeps the seats, the fuel and the health. Two players asking for the same seat at once: the first one
  gets it, the second one gets the next free seat or nothing.
- Every peer holds each seated player on their seat itself, so passengers never lag behind the vehicle.
- A player who leaves the game frees their seat; a driver who leaves gives the vehicle back to the server, still
  rolling. A player who joins sees everyone already seated.
- Getting out, a remote player only becomes solid again once their owner has moved them out of the vehicle, so a
  passenger getting out never shoves the vehicle.
- A vehicle in someone's ward can only be entered by the players the ward allows.

## Saving

The **Vehicles Saver** saves every vehicle: position, rotation, health, fuel and the state of its `IObjectSaveable`
components.

| Vehicle | Saved by | On load |
|---|---|---|
| Placed in the scene | Its scene id. The trunk is saved by the `WorldItemsSaver`, as every storage placed in the scene | Moved back where it was |
| Spawned at runtime | The name of its prefab, which must be in the **Prefabs** list of the saver | Spawned again, with its trunk |

A runtime vehicle whose prefab is not listed is skipped with a warning. Nobody is seated after a load: a player saved
while seated is saved standing next to the vehicle, through `ISavedPositionOverride` (UST.Core), which
`PlayerSaveIdentity` asks before saving the position. Each vehicle holds one storage: two `StorageContainer` on one
vehicle would share the same save key.

## HUD

`VehicleHUDView` shows the vehicle of the local player while seated: name, speed, a fuel bar and litres, a health bar,
and the controls (*[E] Get out [Space] Handbrake* for the driver). Every text and format is in the inspector. It reads
`VehicleOccupant.Local`, so a HUD of your own can do the same.

## Scripting

| Member | Use |
|---|---|
| `VehicleOccupant.Local` | The occupant of the local player |
| `VehicleOccupant.LocalSeatChanged` | Raised when the local player gets in or out |
| `VehicleOccupant.Vehicle`, `SeatIndex`, `IsDriving` | Where the player sits |
| `Vehicle.OccupantsChanged`, `HealthChanged`, `FuelChanged`, `WreckedChanged` | Every peer, for sounds and effects: engine on, smoke, fire |
| `Vehicle.ServerWrecked` | Server, for quests or loot |
| `Vehicle.SpeedKmh`, `Velocity` | Every peer |
| `Vehicle.ServerRepair(amount)`, `ServerRefuel(litres)` | Server only |
| `Vehicle.ServerEject(occupant)`, `ServerEjectAll()` | Server only |
| `Vehicle.RequestEnter(seat)`, `RequestExit()` | Local player, for your own seat UI |
| `VehicleMotor.Input` | Set by the occupant each frame; set it yourself for an AI driver on the server |

## Validate Scene

The Vehicle checks list, for each vehicle of the scene and of the saver: a missing NetworkObject, Dont Destroy With
Owner off (fix), a NetworkTransform not on Owner authority (fix), a missing NetworkRigidbody, no definition, fewer
than 3 wheels, no seat or no driver seat, no solid collider. They also flag a player prefab without Vehicle Occupant
(fix), a missing HUD (fix) and a saved scene without Vehicles Saver (fix).

## Performance

- One raycast per wheel per physics step on the peer that simulates the vehicle, and one per wheel per frame on every
  peer for the wheel models.
- Fuel and seats change rarely; the steering angle is sent only when it changes by more than half a degree.
- The `NetworkTransform` of the default vehicles sends position and rotation in full precision, for smooth driving
  at speed: about the cost of one player per moving vehicle.

## Not in this version

- Boats and water (planned), aircraft, motorcycles.
- Changing seats without getting out, weapons from the seats, horn and engine sounds (the events are there to hook
  them).
- A driving animation: the seated parameter is there for one, see [Use your own character](../guides/import-character.md).
