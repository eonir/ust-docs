# UI

UST ships small uGUI components that listen to the gameplay systems. The final look is yours: every view only reads
events, so you can restyle it or replace it completely.

!!! note "Debug overlays"
    `VitalsDebugHUD`, `InventoryDebugHUD` and `NetworkSessionMenu` are drawn with IMGUI for testing. They are not
    meant to ship and will be replaced by uGUI views.

## Finding the local player

UI views live in the scene, while the player is spawned by the network. `LocalPlayerLocator.TryGet<T>(out T)` returns
a component of the local player once the client is connected:

```csharp
if (LocalPlayerLocator.TryGet(out PlayerInventory inventory))
{
    inventory.LocalItemsReceived += OnItemsReceived;
}
```

## Item feed

`ItemFeedView` lists the items the local player just received, such as "Wood +6". Several gains of the same item
close together are merged into one line instead of stacking, with a small scale punch on every gain.

**Setup:** an empty RectTransform in a screen space Canvas, anchored where the list should appear (left or right),
with `ItemFeedView`. A Vertical Layout Group is added automatically.

| Field | Default | Meaning |
|---|---|---|
| Entry Prefab | | Optional custom line. Empty: an icon and a label are built automatically. |
| Max Entries | 6 | Lines shown at the same time. The oldest goes first. |
| Visible Time | 3 | Seconds a line stays fully visible after its last gain |
| Fade Time | 0.5 | Seconds of fade out |
| Punch Scale, Punch Time | 1.2, 0.15 | Scale effect on each gain |
| Default Icon Size, Default Font Size | 32, 24 | Size of the automatic line |
| Format | `{0} +{1}` | `{0}` is the item name, `{1}` the total received |

The feed listens to `PlayerInventory.LocalItemsReceived`: picking up, looting, gathering and crafting all show up.

### Custom line

Create a prefab with `ItemFeedEntry`, an `Image` for the icon, a `TMP_Text` for the label and a `CanvasGroup` for the
fade. Assign it to **Entry Prefab**.

## Screen messages

`ScreenMessageView` shows short messages for the local player, such as "Tool too weak". Warnings are red and shake.
The same message posted again shakes again instead of restarting, so spamming a button does not flicker.

**Setup:** a TextMeshPro text in the Canvas, near the crosshair for example, with `ScreenMessageView`. Leave the text
empty.

| Field | Default | Meaning |
|---|---|---|
| Label | | The text, found in the children if empty |
| Info Color, Warning Color | white, red | Color per message type |
| Visible Time, Fade Time | 1.5, 0.4 | Duration of a message |
| Shake Amplitude, Frequency, Time | 12, 40, 0.35 | Shake of warnings |

### Posting messages

Any system can post a message, local only:

```csharp
LocalMessages.Post("Inventory full", LocalMessageType.Warning);
LocalMessages.Post("Game saved");
```

`GatherFeedback` uses it for gathering errors.

## Blocking gameplay input

When a UI is open, gameplay actions such as attacking must be ignored. Call `UIInputFocus.Push()` when your UI opens
and `UIInputFocus.Pop()` when it closes. `UIInputFocus.IsUIOpen` is true while at least one UI is open.

## Crosshair

Any centered image works. Disable **Raycast Target** on it, so it never blocks clicks on other UI.
