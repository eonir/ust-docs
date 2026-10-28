# Combat

The Combat module lets players swing the item they hold. The same swing gathers resources and hurts characters: what
a hit does depends on the traits of the item and on what it hits.

## How a swing works

`PlayerItemUser`, on the player root, handles the swing.

1. **Owner.** The attack input is pressed. The owner checks that the held item has a **Melee Swing** trait, that the
   cooldown is over and that there is enough stamina. It spends the stamina, plays the animation trigger and waits
   for the **Impact Delay**.
2. **Owner.** At impact, it casts a sphere from the center of the screen, ignoring the player itself. If it hits a
   network object with an `IItemHitReceiver` within range, it sends the hit to the server.
3. **Server.** The server checks the item actually held, the cooldown (25 % tolerance for latency) and the distance
   (range plus a tolerance), then calls every `IItemHitReceiver` of the target.

Detecting on the owner keeps aiming responsive. Validating on the server prevents a modified client from hitting
across the map or faster than the item allows.

!!! note "Impact delay instead of animation events"
    The hit happens after a timer rather than an animation event. It works with any animation, placeholders
    included, and a missing event can never cancel the hit.

## Melee Swing trait

**Add Trait > Combat > Melee Swing** on the item.

| Field | Default | Meaning |
|---|---|---|
| Range | 2.5 | Reach, measured from the chest of the character |
| Radius | 0.3 | Thickness of the sphere cast. Bigger is more forgiving. |
| Cooldown | 0.8 | Seconds between two swings |
| Impact Delay | 0.3 | Seconds between the start of the swing and the hit. Match it to the animation. |
| Stamina Cost | 0 | Stamina spent per swing. 0 spends nothing. |
| Animation Trigger | Attack | Animator trigger, sent through the `NetworkAnimator` so everyone sees it. Empty: none. |

## Player Item User

| Field | Default | Meaning |
|---|---|---|
| Inventory | | The `PlayerInventory`, found automatically |
| Attack Action | | Input action of the attack, usually Player/Attack |
| Hit Layers | Everything | Layers the swing can hit |
| Chest Height | 1.2 | Height distances are measured from |
| Server Range Tolerance | 1 | Extra reach accepted by the server |
| Animator, Network Animator | | Found automatically in the children |

`Swung(item)` is raised on the owner when a swing starts, for sounds or camera effects.

Add `PlayerItemUser` to **Disable While Dead** on `PlayerLife`, so the dead do not swing.

## Damage to players

**Add Trait > Combat > Melee Weapon** on the item and set its **Damage**.

`PlayerDamageReceiver`, on the player root, removes that damage from the health vital.

| Field | Meaning |
|---|---|
| Vitals, Health | The vital that loses the damage |
| Allow Player Damage | PvP. Off by default: players cannot hurt each other. |

Dead players ignore hits. PvP can be switched at runtime on the server:

```csharp
damageReceiver.AllowPlayerDamage = true;
```

## Hits and UI

Attacks are ignored while a UI is open, so clicking a slot of the inventory never swings. Call `UIInputFocus.Push()`
when your UI opens and `UIInputFocus.Pop()` when it closes. See [UI](ui.md).

## Reacting to hits

Implement `IItemHitReceiver` on the root of any network object: breakable crates, doors, creatures.

```csharp
using Unity.Netcode;
using UST.Inventory;

public class Barrel : NetworkBehaviour, IItemHitReceiver
{
    public void ServerReceiveHit(in ItemHit hit)
    {
        // Server only, the hit is already validated.
        // hit.item is the item used, hit.attacker the network object of the attacker.
        NetworkObject.Despawn(true);
    }
}
```

`ItemHit` contains the attacker, its client id, the item, the hit point, the direction, the **damage** and the
**kind** of hit (`Melee` or `Projectile`). `hit.ByPlayer` tells whether the attacker is a player. A damage of 0 means
the receiver reads it from the item, as melee hits do.

Resource nodes only react to melee hits: shooting a tree does not gather it.

## Ranged weapons

Firearms, crossbows and bows share one system. Each shot follows a **ballistic trajectory computed by the server**:
the projectile moves in small steps, with a raycast per step, and gravity bends its path. Damage lands when the
simulated projectile reaches a target. The projectile players see is only a visual following the same path, so what
they see matches what hits.

### How a shot works

1. **Owner.** The attack input is pressed with a ranged weapon held. The owner finds the point under the crosshair and
   sends the shot, from the muzzle towards that point, with the spread of the weapon.
2. **Server.** The server checks the weapon held, the loaded ammo, the fire rate and that the shot starts near the
   character, then removes one round and simulates the trajectory.
3. **Everyone.** A visual projectile follows the same trajectory, with the muzzle and impact effects.

The owner shows the shot at once and lowers its ammo counter before the server answers. A shot the server refuses is
given back after a second.

### Ranged Weapon trait

**Add Trait > Combat > Ranged Weapon** on the item. Set **Max Stack** to 1 on the item.

| Section | Field | Meaning |
|---|---|---|
| Damage | Damage | Damage carried by each hit |
| Damage | Durability Per Shot | Wear per shot, for items with durability |
| Firing | Automatic | Keeps firing while the attack input is held |
| Firing | Fire Interval | Seconds between two shots |
| Firing | Require Aim | Only fires while aiming |
| Firing | Hip Spread, Aim Spread | Max random deviation from the crosshair in degrees, without and with aiming. 0 always hits the crosshair. |
| Firing | Arm Alignment | See [Aiming](#aiming) |
| Ammo | Magazine Size | Rounds per magazine. 1 for a crossbow. |
| Ammo | Ammo Item | Item spent when reloading. Empty: unlimited ammo. |
| Ammo | Reload Time | Seconds |
| Ammo | Auto Reload | Reloads when firing with an empty magazine |
| Ballistics | Muzzle Velocity | m/s. Also sets when damage lands: slower projectiles need to lead moving targets. |
| Ballistics | Gravity Scale | 0 flies straight, 1 is the physics gravity |
| Ballistics | Max Range | Distance after which the projectile is removed |
| Visuals | Projectile Prefab | Visual moved along the trajectory: a model or a trail, without collider or NetworkObject |
| Visuals | Stick Time | Seconds the projectile stays where it hit, such as a bolt in a wall |
| Visuals | Muzzle Effect, Impact Effect | Spawned at the muzzle and where the projectile hits |
| Visuals | Muzzle Name | Name of the muzzle child, when the model has no `HeldItemPoint` of type Muzzle |
| Visuals | Fire Trigger, Reload Trigger | Animator triggers. Empty or missing: none. |

!!! warning "Projectile prefab"
    Remove every collider from the projectile prefab: shots would hit their own visual.

### Ammo

Rounds are loaded **in the weapon**: the magazine is stored in the item stack, so it follows the weapon when it is
moved, dropped, picked up or saved. Firing empties the magazine. Reloading moves rounds from the inventory to the
magazine, so the ammo item only decreases when reloading.

Reload with **R** (or a **Reload Action**). Changing the held item cancels a reload.

`AmmoCounterView`, on a TextMeshPro text, shows the ammo of the held weapon, such as `12 / 48`, and hides itself
without a ranged weapon.

| Field | Meaning |
|---|---|
| Root | Object shown only with a ranged weapon, such as a background panel. Must not hold this component. Empty: the label only. |
| Format | `{0}` loaded, `{1}` magazine size, `{2}` carried |
| Unlimited Format | Used when the weapon has no ammo item |
| Reloading Text | Shown while reloading |

### Player Ranged Weapon

`PlayerRangedWeapon`, on the player root, fires the held weapon.

| Field | Meaning |
|---|---|
| Fire Action | Usually the same Attack action as `PlayerItemUser` |
| Reload Action, Reload Key | Reload input. Without an action, the key is read directly. |
| Hit Layers | Layers projectiles can hit |
| Chest Height | Height of the chest above the pivot. Shots start from the chest when the weapon has no muzzle. |
| Origin Tolerance | Anti-cheat: max distance accepted between the shot origin and the chest |
| Log Refused Shots | Logs why the server refused a shot, for debugging |

`Fired(item)` and `ReloadStarted(item)` are raised on the owner, for sounds, recoil or camera shake. `LoadedAmmo`,
`ReserveAmmo`, `IsReloading`, `IsAiming` and `AimPoint` can be read by your own UI or effects.

### Aiming

!!! note "Work in progress"
    Aiming is still being tuned. This section lists what each part does. Recommended values will follow.

| Part | Role |
|---|---|
| `PlayerController` > Face Aim Point | While aiming or attacking, the character turns towards the point under the crosshair |
| `PlayerRangedWeapon` > Raise After Shot, Hip Fire Delay | A hip shot raises the weapon in the aim pose, then fires |
| `PlayerRangedWeapon` > Weapon Alignment | Turns the weapon in the hand so the muzzle points at the crosshair |
| Ranged Weapon trait > Arm Alignment | Turns the right arm from the shoulder towards the crosshair. Keep 0 when the aim animation already points the weapon forward. |
| `PlayerAimIK` (on the Animator object) | Turns the chest and head towards the aim, up and down included. Needs **IK Pass** on the UpperBody layer. |
| `PlayerHandIK` | Keeps the left hand on the weapon, see [Inventory > Left hand](inventory.md#left-hand) |
| `AimCameraZoom` (on the CinemachineCamera) | Narrows the field of view while aiming |

Aiming and sprinting exclude each other: the one started first wins.

## Facing the attack

Melee swings and shots turn the character towards where the camera aims, for the duration of the attack. Scripts call
`IAimFacing.FaceAim(duration)`, implemented by `PlayerController`: your own abilities can use it too.

## Known limits

- Ranged hits are validated against the positions the server knows, which are slightly late for moving players.
  There is no lag compensation yet.

- The swing is aimed from the camera: a player can hit what the crosshair shows even if the weapon model does not
  visually pass through it. This is the usual choice for third person games.
- Weapons do not wear yet. Tools wear when they gather.
