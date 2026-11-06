# Ranged weapons

Part of [Combat](index.md).

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
| Damage | Damage Type | Pierce for bullets and bolts. Empty: untyped. |
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
| Visuals | Muzzle Effect | Flash shown on the muzzle point at each shot, created once per weapon model, see [Muzzle flash](#muzzle-flash) |
| Visuals | Impact Effect | Spawned where the projectile hits |
| Visuals | Muzzle Name | Name of the muzzle child, when the model has no `HeldItemPoint` of type Muzzle |
| Visuals | Fire Trigger, Reload Trigger | Animator triggers. Empty or missing: none. |

### Muzzle flash

On the first shot of a held weapon, every peer creates its **Muzzle Effect** once, as a child of its muzzle point, at
the position and rotation of that point. Every later shot only turns it off and on again, which restarts its particle
systems: nothing is created or destroyed while firing. It stays on the weapon model, which the held item keeps
hidden while another item is in the hand (see [Item in the hand](../inventory/held-item.md#item-in-the-hand)).
Any prefab works: its particle systems play by themselves (**Play On Awake** on, **Looping** off).

To move the flash, move the muzzle point of the weapon model: the flash sits exactly on it.

The demo weapons use `Assets/UST/VFX/Muzzle.prefab`: a flash, sparks and a light that lives as long as the
flash, with the additive material `M_MuzzleFlash`. To change it, open the prefab and edit its particle systems:
**Start Size** of *Flash* for its size, **Start Color** for its color, the **Lights** module for the light. For a
weapon with its own flash, duplicate the prefab and assign the copy to the **Muzzle Effect** of that weapon, in the
Item Database.

The muzzle point is the `HeldItemPoint` of type Muzzle of the held model, or its child named **Muzzle Name**, with its
Z axis out of the barrel. Keep it an empty object: a flash placed there would play each time the weapon is taken in
hand. *Tools > UST > Advanced > Weapons > Build Muzzle Flash* rebuilds the default flash, after a confirmation.

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
| Inventory, Input Reader, Held Item | The `PlayerInventory`, `PlayerInputReader` and `PlayerHeldItem` of the player. The input reader gives the aim input |
| Fire Action | Usually the same Attack action as `PlayerItemUser` |
| Reload Action, Reload Key | Reload input. Without an action, the key is read directly. |
| Hit Layers | Layers projectiles can hit |
| Chest Height | Height of the chest above the pivot. Shots start from the chest when the weapon has no muzzle. |
| Origin Tolerance | Anti-cheat: max distance accepted between the shot origin and the chest |
| Log Refused Shots | Logs why the server refused a shot, for debugging |
| Animator, Network Animator | Animator of the character, and its `NetworkAnimator` for the fire and reload triggers |
| Aiming Parameter | Animator bool set while aiming with a ranged weapon. Default `Aiming`. Empty: none |
| Raise After Shot | Seconds the weapon stays in the aim pose after a hip shot (0.6). 0 fires from the hip pose |
| Hip Fire Delay | Seconds between a hip click and the shot, to turn and raise the weapon (0.15). No delay while aiming |
| Weapon Alignment | 0 to 1: turns the weapon in the hand so the muzzle points at the crosshair, while aiming (1) |
| Alignment Speed | How fast that correction follows (12) |

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
| `PlayerItemRig` (on the Animator object) | Turns the spine, chest and head towards the aim, up and down included, and keeps the left hand on the weapon. See [Inventory > Item rig](../inventory/held-item.md#item-rig). |
| Held Visual trait > Aim Yaw Offset, Aim Pitch Offset | Per item correction of the upper body aim, tuned in the Held Item Tuning window |
| `AimCameraZoom` (on the CinemachineCamera) | Narrows the field of view while aiming, and moves the camera to the aim distance of `CameraZoom`, see [Player > Camera zoom](../player/camera.md#camera-zoom) |

Aiming and sprinting exclude each other: the one started first wins.
