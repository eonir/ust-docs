# Vehicle

The Vehicle module adds drivable vehicles for several players: a driver and passengers, raycast wheel physics, fuel,
health, crashes, running over enemies, repairs, a trunk, a HUD and saving. Every kind of vehicle is one **Vehicle
Definition** asset: a sports car, a family car and a truck come ready to drive, and a new one is a copy of the closest
preset with a few values changed.

The module is optional: add `UST_DISABLE_VEHICLE` to the Scripting Define Symbols to remove it, see
[Modules](../../architecture/modules.md#disabling-an-optional-module). Boats are planned for a later version.

This module is described on several pages:

- [Vehicle Definition](definition.md)
- [Vehicle prefab](prefab.md)
- [Damage, repair and fuel](damage-fuel.md)

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

In the [Setup Window](../../getting-started/setup-window.md), **Vehicle > Add** does steps 1, 4 and 5. By hand:

1. *Tools > UST > Setup Player Prefab*, or *Tools > UST > Advanced > Vehicle > Add Vehicles To Selected Player* on the
   player prefab (or the fix in Validate Scene).
2. *Tools > UST > Advanced > Vehicle > Create Default Vehicles*, or right click your own model > **UST > Create Vehicle
   From Model**.
3. Place vehicles in the scene. Vehicles placed in the scene need no registration in the Network Prefabs list.
4. *Tools > UST > Advanced > UI > Add Vehicle HUD* (also added by *Create Game UI*).
5. *Tools > UST > Advanced > Vehicle > Add Vehicles Saver To Scene* if the scene is saved.
6. Give players a repair tool and fuel: see [Repair](damage-fuel.md#repair) and [Fuel](damage-fuel.md#fuel).

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
and the controls. The keys of the controls come from the input actions: they follow the device in use and the keys
the player chose in *Options > Keys*. It reads `VehicleOccupant.Local`, so a HUD of your own can do the same.
*Create Game UI* adds it, or *Tools > UST > Advanced > UI > Add Vehicle HUD*.

| Field | Default | Meaning |
|---|---|---|
| Panel | | Shown while seated, hidden otherwise |
| Name Label | | Name of the vehicle |
| Speed Label, Speed Format | , {0} km/h | Speed. {0}: the speed. |
| Fuel Fill, Fuel Label | | Filled image and litres. Hidden for a vehicle without fuel. |
| Health Fill | | Filled image of the health of the vehicle |
| Hint Label | | The controls |
| Driver Hint | [{0}] Get out   [{1}] Handbrake | {0}: the key to get out, {1}: the handbrake key |
| Passenger Hint | [{0}] Get out | {0}: the key to get out |
| Get Out Action, Handbrake Action | Player/Interact, Player/Jump | Actions whose keys the hints show |

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
- A driving animation: the seated parameter is there for one, see [Use your own character](../../guides/import-character.md).
