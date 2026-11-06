# UI

UST ships a complete prototype game UI in uGUI: hotbar, inventory window with drag and drop, opened containers,
vital bars, interaction prompt, item feed, screen messages and death screen. Every view only listens to the gameplay
systems, so you can restyle it, move it or replace it without touching gameplay code.

This module is described on several pages:

- [HUD](hud.md)
- [Windows](windows.md)

## Creating the UI

*Tools > UST > Create Game UI* builds everything in a canvas named **UST UI**:

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

*Tools > UST > Advanced > UI > Save Layout As Default* stores the placement of every element of the **UST UI** canvas (anchors,
pivot, position, size, scale and active state) in `Assets/UST/UI/UILayoutDefault.asset`. A confirmation tells you
what will be replaced. From then on, **Create Game UI** places the elements it creates with your layout.

*Tools > UST > Advanced > UI > Reset Default Layout* deletes the saved layout, after a confirmation: **Create Game UI** goes
back to the original layout. The UI already in your scene is not changed: delete the canvas and create it again to
rebuild it with the original layout.

!!! note
    The layout finds elements by their path in the canvas. After renaming or moving an element to another parent,
    save the layout again. Styles are never part of the layout.

### Canvases

There is one root canvas. **Inventory UI** and **Vital Bars** have their own nested canvas: Unity rebuilds a whole
canvas when anything in it changes, so bars that move every frame or an inventory being dragged around only rebuild
themselves instead of the whole HUD.

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
[Vitals > Debug HUD](../vitals.md#debug-hud) and [Inventory > Debug HUD](../inventory/world-items.md#debug-hud).

`NetworkSessionMenu`, the Host / Client / Server menu at the top of the screen:

| Field | Default | Meaning |
|---|---|---|
| Address | 127.0.0.1 | Address a client connects to. Use the local IP of the host to join from another computer of the network |
| Port | 7777 | Port of the host or server |
| Listen Address | 0.0.0.0 | Address the host and the server listen on. 0.0.0.0 accepts connections from the local network; 127.0.0.1 only from this computer |
| Width | 260 | Width of the menu, in pixels |

Joining over the internet needs port forwarding on the host, or a relay: replace the menu by your own, with Unity
Relay or Steam networking for example.
