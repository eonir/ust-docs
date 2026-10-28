# UI

UST ships a complete prototype game UI in uGUI: hotbar, inventory window with drag and drop, opened containers,
vital bars, interaction prompt, item feed, screen messages and death screen. Every view only listens to the gameplay
systems, so you can restyle it, move it or replace it without touching gameplay code.

## Creating the UI

*Tools > UST > UI > Create Game UI* builds everything in a canvas named **UST UI**:

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

It also adds an `EventSystem` using the Input System (or converts an old one), and `CameraUILock` on the
Cinemachine camera.

The tool only creates what is missing: running it again never moves or duplicates what is already in the scene.

!!! note "Nothing shows in Edit Mode"
    Slots and bars are created at runtime, once the local player has spawned: there is nothing to show before. The
    inventory window and the death screen are disabled until they are needed.

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

### Sprites

Every texture of `Assets/UST/UI/Sprites` is imported as a UI sprite automatically. A `_b12` suffix in the file name
sets a 9-slice border of 12 pixels, so frames scale without stretching their corners. The default sprites are white
and get their color from the Image, so one sprite serves every color.

### Saving your layout as the default

*Tools > UST > UI > Save Layout As Default* stores the placement of every element of the **UST UI** canvas (anchors,
pivot, position, size, scale and active state) in `Assets/UST/UI/UILayoutDefault.asset`. A confirmation tells you
what will be replaced. From then on, **Create Game UI** places the elements it creates with your layout.

*Tools > UST > UI > Reset Default Layout* deletes the saved layout, after a confirmation: **Create Game UI** goes
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
- moving, jumping, sprinting, crouching and selecting the hotbar still work.

### Slot interactions

| Action | Result |
|---|---|
| Drag to another slot | Move, merge or swap |
| Right drag, or Shift + drag | Move half the stack |
| Drop outside any UI | Drop the items in the world |
| Shift + click | Send the stack to the other side: the opened container, otherwise between inventory and hotbar |

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
| Max Entries | 6 | Lines at once, the oldest goes first |
| Visible Time, Fade Time | 3, 0.5 | Duration of a line after its last gain |
| Punch Scale, Punch Time | 1.2, 0.15 | Effect on each gain |
| Format | `{0} +{1}` | `{0}` is the item name, `{1}` the total |

It listens to `PlayerInventory.LocalItemsReceived`: picking up, looting, gathering and crafting all show up.

## Screen messages

`ScreenMessageView` shows short messages such as "Tool too weak". Warnings are red and shake. The same message posted
again shakes again instead of restarting.

| Field | Default |
|---|---|
| Info Color, Warning Color | white, red |
| Visible Time, Fade Time | 1.5, 0.4 |
| Shake Amplitude, Frequency, Time | 12, 40, 0.35 |

Post messages from any system, locally:

```csharp
LocalMessages.Post("Inventory full", LocalMessageType.Warning);
LocalMessages.Post("Game saved");
```

## Death screen

`DeathScreenView` fades in a full screen panel when the local player dies, with the respawn countdown from
`PlayerLife.RespawnRemaining`, and fades out on respawn. **Countdown Format** is `Respawn in {0}`.

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

## Debug overlays

`VitalsDebugHUD`, `InventoryDebugHUD` and `NetworkSessionMenu` are drawn with IMGUI for testing and are not meant to
ship. `InventoryDebugHUD` also uses Tab: remove it from the player when you use the inventory window.
