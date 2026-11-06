# Markers

The Markers module shows points of the world on screen, with an icon and the distance, kept on the edge of the screen
with an arrow when they are out of view:

- **Pings**: a player shows the aimed point to everyone for a few seconds.
- **Placed markers**: a player leaves a marker that stays for everyone and is saved with the world.
- **Far players**: the other players, beyond the range of their [nameplate](../ui/hud.md#player-nameplates).
- **Places of the world**: any object with a `WorldMarker`, such as a base, a trader or a quest.
- **Your loot bag**: where your items fell when you died, see [Marker of the bag](../inventory/world-items.md#marker-of-the-bag).

Each kind can be turned off in the [options](../settings.md), with the edge arrows and the distance. The module is
optional: add `UST_DISABLE_MARKERS` to the Scripting Define Symbols to remove it, see
[Modules](../../architecture/modules.md#disabling-an-optional-module).

## Setup

In the [Setup Window](../../getting-started/setup-window.md): **Markers > Add**, which also adds the map. By hand,
run *Tools > UST > Advanced > Markers > Add Markers To Scene*. It adds a `MarkerService` to the **Network Session**
object and the markers view to the **UST UI** canvas, assigns the Ping input, and adds a `LootBagMarker` to every
loot bag prefab of the project. *Create Game UI* adds the service and the view too.

## Pinging and placing

| Input | Mouse | Gamepad | Does |
|---|---|---|---|
| Press Ping | middle button | d-pad left | Pings the aimed point for everyone, with your name and color |
| Hold Ping | middle button | d-pad left | Places a marker there, or removes your marker if you aim close to it |

Pings and markers are ignored while a window is open or a build piece is previewed (the middle button removes pieces
then). Each player has its own color.

## Marker Service

| Field | Default | Meaning |
|---|---|---|
| Ping Action, Ping Key | Player/Ping, None | The input. The key is used when no action is assigned. |
| Hold Seconds | 0.5 | Holding longer places or removes a marker instead of pinging |
| Range | 300 | Farthest point that can be pinged, in metres |
| Ping Seconds | 6 | How long a ping stays |
| Ping Cooldown | 0.75 | Shortest time between two pings of a player, checked on the server |
| Max Markers Per Player | 10 | The oldest marker of the player goes when a new one is placed |
| Remove Radius | 3 | Holding closer than this to one of your markers removes it |
| Teammate Height, Teammate Color | 2.4, light blue | The marker of the other players |
| Player Colors | 8 colors | Colors of the pings and markers of each player |

Placed markers are saved with the world. After loading, they belong to nobody: any player can remove them.

## Markers View

| Field | Default | Meaning |
|---|---|---|
| Default Icon | a dot | Icon of the markers without their own |
| Max Distance | 500 | Markers further away are hidden, unless they have their own distance. The options change it. |
| Edge Margin | 48 | Margin from the edges of the screen for the markers out of view |
| Hide Distance Under | 3 | Closer markers show no distance |
| Teammate Min Distance | 30 | Closer players have no marker: their nameplate is enough |
| Distance Format | {0} m | |

Restyle the **Marker** child: it is copied for each marker. Its **Arrow** turns towards the marker out of view.

## World Marker

Add it to any object (right click it in the Hierarchy > **UST > Add World Marker**): every peer shows it.

| Field | Meaning |
|---|---|
| Label | Text under the icon. Empty: the name of the object. |
| Icon, Color | Look of the marker |
| Height | Height of the icon above the pivot |
| Show Offscreen | Kept on the edge of the screen when out of view |
| Max Distance | Hidden further than this. 0: the distance of the options. |

## Scripting

```csharp
// Your own marker, on this peer only: a quest target, an objective
Marker marker = MarkerRegistry.Add(new Marker
{
    kind = MarkerKind.Scene,
    target = boss.transform,
    height = 3f,
    label = "Boss",
    color = Color.red,
    expiresAt = Time.unscaledTime + 60f   // 0: stays until removed
});
MarkerRegistry.Remove(marker);

MarkerService.Current.RequestPing(point);                         // ping from code
MarkerService.Current.ServerPlace(clientId, point, "Camp");       // server: a shared marker
MarkerService.Pinged += (sender, point) => PlaySound(point);
worldMarker.SetLabel("Talk to the trader");
```

## Map and minimap

The map window, the minimap and the pins placed on the map are described in [Map and minimap](map.md).

## Not in this version

- Choosing an icon for a placed marker.
