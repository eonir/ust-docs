# Vehicle prefab

Part of [Vehicle](index.md).

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
