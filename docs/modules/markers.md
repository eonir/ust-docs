# Markers

The Markers module shows points of the world on screen, with an icon and the distance, kept on the edge of the screen
with an arrow when they are out of view:

- **Pings**: a player shows the aimed point to everyone for a few seconds.
- **Placed markers**: a player leaves a marker that stays for everyone and is saved with the world.
- **Far players**: the other players, beyond the range of their [nameplate](ui.md#player-nameplates).
- **Places of the world**: any object with a `WorldMarker`, such as a base, a trader or a quest.

Each kind can be turned off in the [options](settings.md), with the edge arrows and the distance. The module is
optional: add `UST_DISABLE_MARKERS` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module).

## Setup

Run *Tools > UST > Advanced > Markers > Add Markers To Scene*. It adds a `MarkerService` to the **Network Session** object and the
markers view to the **UST UI** canvas, and assigns the Ping input. *Create Game UI* adds both too.

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

Add it to any object (*right click in the Hierarchy > UST > Add World Marker*): every peer shows it.

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

Run *Tools > UST > Advanced > Map > Add Map To Scene*. It adds a **Map Area** fitted to the terrains of the scene (or to every
renderer when there is none), bakes its picture, and adds the map window and the minimap to the **UST UI** canvas.
*Create Game UI* adds the window and the minimap too.

| Input | Keyboard / mouse | Gamepad | Does |
|---|---|---|---|
| Map | M | d-pad right | Opens and closes the map. Escape or B closes it. |
| Left click | left button | A (gamepad cursor) | Places a pin, shared and saved, or removes yours when clicked |
| Right click | right button | X (gamepad cursor) | Pings the point for everyone |
| Wheel, drag | wheel, drag | +, - and Center buttons | Zooms toward the mouse, moves the map |

Pins are the placed markers of the `MarkerService`: the same limit per player, the same colors, shown in the world too.
A pin of another player is left alone when clicked. A pin lands on the ground under the clicked point (**Ground
Layers** of the area).

The minimap sits in the top right corner, north up, centered on the player. It hides while a window or a menu is open.
Players turn it off and set its zoom in the Display options (*Minimap*, *Minimap zoom*, 150 m). Markers out of its view
stay on its edge. Each kind of marker follows the same options as on screen.

### Map Area

| Field | Default | Meaning |
|---|---|---|
| Map Texture | | Picture of the area seen from above. Baked, or painted by hand. |
| Size | 500 x 500 | Width (X) and depth (Z) in metres, centered on the object |
| Ground Layers | Default | Layers a pin lands on |
| Search Height | 500 | Height the ground is searched from, and the height of the bake camera |
| Bake Resolution | 2048 | Pixels of the longest side of the baked picture |
| Bake Layers | all but UI | Layers drawn in the baked picture |

**Tools > UST > Advanced > Map > Bake Map Texture** renders the area from above with the lighting of the scene and saves
`<Scene>_Map.png` next to the scene. Bake it again after changing the scene; pieces built in game are not on the
picture. **Fit Map Area To Scene** sizes the area again. The top of the map is +Z.

`MapView` draws a part of the area in any rectangle of the UI, with the markers and the player arrow: set its `Center`
(world X and Z) and `VisibleMetres`. Use it for a map on a table, in a quest window, or a minimap of your own.

```csharp
MapWindow.Current.Open();
MarkerService.Current.RequestToggleMarker(point);   // pin, or remove yours there
```

## Not in this version

- Fog of war on the map.
- Choosing an icon for a placed marker.
