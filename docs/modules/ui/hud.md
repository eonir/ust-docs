# HUD

Part of [UI](index.md).

## Hotbar

`HotbarHUDView` shows the local player hotbar with the selected slot highlighted. It is always visible. While the
inventory window is open, it takes part in drag and drop like the inventory. While the window is closed, it only
displays.

## Vital bars

`VitalsBarsView` creates one bar per vital of the local player, in profile order. Each bar uses the **Icon** and the
**Color** of its vital definition.

| Field | Default | Meaning |
|---|---|---|
| Bar Prefab | VitalBar | Look of a bar |
| Bar Sprite, Default Bar Size, Default Font Size | , 240 × 22, 14 | Used only without a Bar Prefab, to build a plain bar by code |
| Show Values | on | "74 / 100" on the bar |
| Smoothing | 8 | How fast bars follow their value. 0 snaps. |
| Hidden | | Vitals not shown |

Values are read every frame, so late joiners and loaded saves show correctly at once.

`VitalBarView`, on the bar prefab: **Icon**, **Fill** (a filled image) and **Value** (the "74 / 100" text). Restyle
the **VitalBar** prefab to change every bar.

## Interaction prompt

`InteractionPromptView` shows what the player aims at, such as "[E] Pick up Wood x3".

| Field | Default | Meaning |
|---|---|---|
| Interact Action | | Optional. The key shown follows the binding, rebinding and gamepads included. |
| Fallback Key | E | Key shown without an action |
| Format | `[{0}] {1}` | `{0}` is the key, `{1}` the prompt |

The prompt text comes from the target: `WorldItem` uses **Prompt Format** ("Pick up {0}") and **Stack Prompt Format**
("Pick up {0} x{1}"), `LootBag` its **Prompt**. It is hidden while a UI is open or the player is dead.

## Item feed

`ItemFeedView` lists the items the local player just received, such as "Wood +6". Gains of the same item close
together merge into one line, with a small scale punch on each gain.

| Field | Default | Meaning |
|---|---|---|
| Entry Prefab | ItemFeedEntry | Look of a line |
| Default Icon Size, Default Font Size | 32, 24 | Used only without an Entry Prefab, to build a plain line by code |
| Max Entries | 6 | Lines at once, the oldest goes first |
| Visible Time, Fade Time | 3, 0.5 | Duration of a line after its last gain |
| Punch Scale, Punch Time | 1.2, 0.15 | Effect on each gain |
| Format | `{0} +{1}` | `{0}` is the item name, `{1}` the total |

It listens to `PlayerInventory.LocalItemsReceived`: picking up, looting, gathering and crafting all show up.

## Screen messages

`ScreenMessageView` shows short messages such as "Tool too weak". Warnings are red and shake. The same message posted
again shakes again instead of restarting.

| Field | Default | Meaning |
|---|---|---|
| Label, Canvas Group | | The text and the group faded in and out |
| Info Color, Warning Color | white, red | Text color of each message type |
| Visible Time, Fade Time | 1.5, 0.4 | Seconds fully visible, then fading |
| Shake Amplitude | 12 | Horizontal shake of a warning, in pixels |
| Shake Frequency | 40 | Speed of the shake |
| Shake Time | 0.35 | Duration of the shake |

Post messages from any system, locally:

```csharp
LocalMessages.Post("Inventory full", LocalMessageType.Warning);
LocalMessages.Post("Game saved");
```

## Player nameplates

`PlayerNameplatesView` shows the name and the health of the other players, above their head. *Create Game UI* adds
it, or *Tools > UST > Advanced > UI > Add Player Nameplates*. Names come from [Player names](../lobby.md#player-names),
health from the health vital of `PlayerLife`. Players turn the names and the health off, and set the distance, in
*Options > Display*.

| Field | Default | Meaning |
|---|---|---|
| Max Distance | 40 | Players further than this, in metres, have no nameplate |
| Fade Share | 0.25 | The nameplate fades out over the last quarter of the distance |
| Height | 2.1 | Height above the pivot of the player, in metres |
| Show Health | on | The health bar under the name |
| Hide Dead Players | on | |
| Require Line Of Sight, Occluders | on, everything | Hides the nameplate of a player behind a wall or the ground. Your own character never hides the others. |
| Sight Interval | 0.25 | Seconds between two checks of the line of sight, per player |
| Template Name, Template Health, Template Health Root | | The name text, the filled health image, and the object holding the health bar, in the **Nameplate** child copied for each player |

Restyle the **Nameplate** child: it is copied for each player. The options of the players override **Max Distance**
and **Show Health**.

## Death screen

`DeathScreenView` fades in a full screen panel when the local player dies, with the respawn countdown from
`PlayerLife.RespawnRemaining`, and fades out on respawn. **Countdown Format** is `Respawn in {0}`.

!!! warning "HUD elements and clicks"
    Untick **Raycast Target** on every text and image of the HUD: crosshair, ammo counter, compass, boss bar. A HUD
    element that catches raycasts silently blocks the buttons of the windows under it, for example the Stats button
    while a weapon is held. `AmmoCounterView` turns them off by itself; elements you add are up to you.
