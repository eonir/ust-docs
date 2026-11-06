# UI

UST ships a complete prototype game UI in uGUI: hotbar, inventory window with drag and drop, opened containers,
vital bars, interaction prompt, item feed, screen messages and death screen. Every view only listens to the gameplay
systems, so you can restyle it, move it or replace it without touching gameplay code.

## Creating the UI

*UST Tools > Create Game UI* builds everything in a canvas named **UST UI**:

| Element | Default place | Component |
|---|---|---|
| Hotbar | Bottom center | `HotbarHUDView` |
| Inventory window | Center, container on its left | `InventoryWindow` |
| Vital bars | Bottom left | `VitalsBarsView` |
| Interaction prompt | Under the crosshair | `InteractionPromptView` |
| Screen messages | Below the prompt | `ScreenMessageView` |
| Item feed | Bottom right | `ItemFeedView` |
| Death screen | Full screen | `DeathScreenView` |
| Drag icon | Follows the cursor | `ItemDragIcon` |
| Gathering messages | On the canvas | `GatherFeedback`, if the Gathering module is present |
| Crafting panel | Right of the container panel | `CraftingWindow`, if the Crafting module is present |
| Build menu and build HUD | Center, and above the hotbar | `BuildMenuWindow`, `BuildHUDView`, if the Building module is present |

It also adds an `EventSystem` using the Input System (or converts an old one), and `CameraUILock` on the
Cinemachine camera.

The tool only creates what is missing: running it again never moves or duplicates what is already in the scene.

!!! note "Nothing shows in Edit Mode"
    Slots and bars are created at runtime, once the local player has spawned: there is nothing to show before. The
    inventory window and the death screen are disabled until they are needed.


## Character stats panel

A **Stats** button in the corner of the inventory panel opens the character stats, alone at the left edge of the
screen, so it never pushes the other panels. Move it freely: it is outside any layout group. *Create Game UI* adds it; for a UI built before, run *UST Tools > Advanced > UI > Add Stats Panel*.

The panel lists what the game has, by section:

| Section | Lines | From |
|---|---|---|
| Vitals | Every vital, current / max | `PlayerVitals` |
| Defense | Armor, then each damage type resisted or weak: *Cold 50 % resist*, *Fire 30 % weak*, *Poison Immune* | `PlayerEquipment`, and the resistances of every `IResistanceSource` multiplied by type (natural resistances of `PlayerDamageReceiver`, worn items) |
| Movement | Speed from equipment, when not 100 % | `PlayerEquipment` |
| Inventory | Weight, when the weight limit is used | `PlayerInventory` |
| Environment | Weather where the player stands, felt temperature and stage: *Snow*, *4 °C Cold* | `WeatherExposure`, `PlayerTemperature` |

| Field | Default | Meaning |
|---|---|---|
| Open By Default | on | Shows the panel when the inventory opens, until the player hides it with the **Stats** button |
| Refresh Interval | 0.5 | Seconds between two updates while open |
| Section Order | Vitals, Environment, Defense, Movement, Inventory | Display order of the sections. Sections not listed come after, by name. Removing an entry does not hide its section |
| Panel, Toggle Button, Body | | The panel shown and hidden, the **Stats** button of the inventory that toggles it, and the text that lists the stats. Filled by *Add Stats Panel* |
| Section Color, Resist Color, Weak Color | gold, green, red | Section titles, resistances above 0 % and weaknesses below 0 % |
| Value Column | 60 | Position of the values, in percent of the width |

Add your own lines from any component of the player: implement `IStatProvider` (UST.Core) and add `StatLine`s with a
section, a label, a value and an order. A level module adds a "Level" section, a skill module its skills.

The sections are plain names, given by the modules:

| Section | Filled by |
|---|---|
| Vitals | Health, hunger, thirst, stamina… |
| Environment | `WeatherExposure` (weather where the player stands), `PlayerTemperature` (felt temperature and stage) |
| Defense | Armor and resistances: `PlayerEquipment`, `PlayerDamageReceiver` |
| Movement | Speed: `PlayerEquipment` |
| Inventory | Weight: `PlayerInventory` |

The names are constants of `StatSections` (UST.Core). A section of your own just needs a new name; add it to
Section Order to place it.

```csharp
public class PlayerLevel : MonoBehaviour, IStatProvider
{
    public int level = 1;

    public void CollectStats(List<StatLine> lines)
    {
        lines.Add(new StatLine("Level", "Level", level.ToString()));
    }
}
```

## Where to change what

| What | Where |
|---|---|
| Position and size of an element | In the scene, on its RectTransform under **UST UI** |
| Texts, fonts, formats | In the scene, on the TextMeshPro and the view component |
| Look of a slot | `Assets/UST/UI/Prefabs/ItemSlot.prefab` |
| Look of a vital bar | `Assets/UST/UI/Prefabs/VitalBar.prefab` |
| Look of an item feed line | `Assets/UST/UI/Prefabs/ItemFeedEntry.prefab` |
| Sprites | `Assets/UST/UI/Sprites` |

Slots, bars and feed lines are created from their prefab at runtime, so changing them in the scene would have no
effect: change the prefab.

`ItemSlotView`, on the slot prefab:

| Field | Meaning |
|---|---|
| Icon, Count | Item icon, and the stack count (hidden for 1) |
| Durability Root, Durability Fill | Bar shown for items with durability, and its filled image |
| Durability Full, Durability Empty | Colors of the bar, blended by the durability left: green when new, red when about to break |
| Selection | Shown on the selected hotbar slot |

### Sprites

Every texture of `Assets/UST/UI/Sprites` is imported as a UI sprite automatically. A `_b12` suffix in the file name
sets a 9-slice border of 12 pixels, so frames scale without stretching their corners. The default sprites are white
and get their color from the Image, so one sprite serves every color.

### Saving your layout as the default

*UST Tools > Advanced > UI > Save Layout As Default* stores the placement of every element of the **UST UI** canvas (anchors,
pivot, position, size, scale and active state) in `Assets/UST/UI/UILayoutDefault.asset`. A confirmation tells you
what will be replaced. From then on, **Create Game UI** places the elements it creates with your layout.

*UST Tools > Advanced > UI > Reset Default Layout* deletes the saved layout, after a confirmation: **Create Game UI** goes
back to the original layout. The UI already in your scene is not changed: delete the canvas and create it again to
rebuild it with the original layout.

!!! note
    The layout finds elements by their path in the canvas. After renaming or moving an element to another parent,
    save the layout again. Styles are never part of the layout.

### Canvases

There is one root canvas. **Inventory UI** and **Vital Bars** have their own nested canvas: Unity rebuilds a whole
canvas when anything in it changes, so bars that move every frame or an inventory being dragged around only rebuild
themselves instead of the whole HUD.

## Hotbar

`HotbarHUDView` shows the local player hotbar with the selected slot highlighted. It is always visible. While the
inventory window is open, it takes part in drag and drop like the inventory. While the window is closed, it only
displays.

## Inventory window

`InventoryWindow` opens and closes with **Tab** (or the assigned **Toggle Action**) and closes with **Escape**.

| Field | Meaning |
|---|---|
| Window | Object shown while open |
| Inventory Grid | Grid of the player inventory |
| Hotbar Grid | Grid of the hotbar taking part in drag and drop: the gameplay hotbar |
| Container Panel, Container Grid, Container Title | Side panel of an opened container |
| Close Distance Margin | An opened container closes when the player walks away further than its access range plus this margin |
| Toggle Action, Toggle Key, Close Key | Input |

While the window is open:

- the cursor is free,
- the camera does not orbit (`CameraUILock`),
- attacking, aiming and interacting are ignored,
- moving, jumping, sprinting, crouching and the hotbar keys still work.

### Slot interactions

| Action | Result |
|---|---|
| Drag to another slot | Move, merge or swap |
| Right drag, or Shift + drag | Move half the stack |
| Drop outside any UI | Drop the items in the world |
| Shift + click | Send the stack to the other side: the opened container, otherwise between inventory and hotbar |
| Right click | Use the item, for example a consumable, from the inventory or the hotbar. While a split stack follows the cursor, a right click cancels it instead. |

Every action is a request validated by the server, see [Inventory](inventory.md#moving-items).

### Opened containers

Interacting with a loot bag opens it in the window, so players take items one by one. Any container can be opened
from your own code:

```csharp
container.OpenLocally();   // shows it in the inventory window of the local player
```

Without an inventory window in the scene, a loot bag falls back to taking everything at once.

### Grids

`ContainerGridView` creates one slot per slot of its container inside a `GridLayoutGroup`, so the panel follows the
size of the container.

| Field | Meaning |
|---|---|
| Slot Prefab | The slot prefab |
| Slot Parent | Parent of the slots, this object if empty |
| Columns | Fixed column count. 0 keeps the GridLayoutGroup setting. -1 picks it from the number of slots, up to Max Auto Columns. |
| Interactable | Accepts drag and drop and clicks |

`-1` suits loot bags, whose size changes every time.

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

## Gamepad cursor

`GamepadCursor`, on the **UST UI** canvas, shows a cursor moved by the left stick whenever a window is open
(`UIInputFocus`) and the player uses a gamepad. It drives a virtual mouse, so every window made for the mouse works
with the gamepad: A clicks and drags, X right-clicks, the right stick scrolls. The character does not move meanwhile.
*Create Game UI* and *UST Tools > Advanced > Input > Assign Input Actions* add it.

| Field | Default | Meaning |
|---|---|---|
| Cursor Sprite, Cursor Color, Cursor Size | dot, gold, 22 | Look of the cursor. A sprite is pointed by its top left corner. |
| Speed | 1100 | Pixels per second at full deflection, on a 1080p screen |
| Scroll Speed | 45 | Scrolling with the right stick |

Your own window only has to call `UIInputFocus.Push()` when it opens and `Pop()` when it closes.

## Player nameplates

The name and the health of the other players, above their head. *Create Game UI* adds them, or *UST Tools > Advanced > UI > Add Player Nameplates*. Names come from [Player names](lobby.md#player-names), health from the health vital of `PlayerLife`.

| Field | Default | Meaning |
|---|---|---|
| Max Distance | 40 | Players further than this, in metres, have no nameplate |
| Fade Share | 0.25 | The nameplate fades out over the last quarter of the distance |
| Height | 2.1 | Height above the pivot of the player, in metres |
| Show Health | on | The health bar under the name |
| Hide Dead Players | on | |
| Require Line Of Sight, Occluders | on, everything | Hides the nameplate of a player behind a wall or the ground. Your own character never hides the others. |
| Sight Interval | 0.25 | Seconds between two checks of the line of sight, per player |

Restyle the **Nameplate** child: it is copied for each player.

## Death screen

`DeathScreenView` fades in a full screen panel when the local player dies, with the respawn countdown from
`PlayerLife.RespawnRemaining`, and fades out on respawn. **Countdown Format** is `Respawn in {0}`.

!!! warning "HUD elements and clicks"
    Untick **Raycast Target** on every text and image of the HUD: crosshair, ammo counter, compass, boss bar. A HUD
    element that catches raycasts silently blocks the buttons of the windows under it, for example the Stats button
    while a weapon is held. `AmmoCounterView` turns them off by itself; elements you add are up to you.

## Your own UI

### Finding the local player

```csharp
if (LocalPlayerLocator.TryGet(out PlayerInventory inventory))
{
    inventory.LocalItemsReceived += OnItemsReceived;
}
```

The player is spawned by the network, so try again every frame until it succeeds.

### Blocking gameplay input

Call `UIInputFocus.Push()` when your UI opens and `UIInputFocus.Pop()` when it closes. While `UIInputFocus.IsUIOpen`
is true, attacks, aiming, interaction and the camera orbit are ignored.

The mouse wheel has its own owner. `MouseWheelFocus.Claim(this)` takes it, `MouseWheelFocus.Release(this)` gives it
back; while it is claimed, the camera zoom and the hotbar ignore the wheel. `PlayerBuilder` claims it while a piece is
previewed, to rotate it. Claims are per owner, so two systems never cancel each other; release in `OnDisable` too.

## Debug overlays

`VitalsDebugHUD`, `InventoryDebugHUD` and `NetworkSessionMenu` are drawn with IMGUI for testing and are not meant to
ship. `InventoryDebugHUD` also uses Tab: remove it from the player when you use the inventory window. See
[Vitals > Debug HUD](vitals.md#debug-hud) and [Inventory > Debug HUD](inventory.md#debug-hud).

`NetworkSessionMenu`, the Host / Client / Server menu at the top of the screen:

| Field | Default | Meaning |
|---|---|---|
| Address | 127.0.0.1 | Address a client connects to. Use the local IP of the host to join from another computer of the network |
| Port | 7777 | Port of the host or server |
| Listen Address | 0.0.0.0 | Address the host and the server listen on. 0.0.0.0 accepts connections from the local network; 127.0.0.1 only from this computer |
| Width | 260 | Width of the menu, in pixels |

Joining over the internet needs port forwarding on the host, or a relay: replace the menu by your own, with Unity
Relay or Steam networking for example.
