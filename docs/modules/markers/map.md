# Map and minimap

Part of [Markers](index.md).

The map shows the world from above with the pins and pings of every player, the other players and the places of the
world. The minimap shows the same around the player, in a corner of the screen.

## Setup

The [Setup Window](../../getting-started/setup-window.md) adds it with the Markers module: **Map and minimap > Add**.
By hand, run *Tools > UST > Advanced > Map > Add Map To Scene*. It:

1. adds a **Map Area** fitted to the terrains of the scene, or to every renderer when there is none;
2. bakes its picture into `<Scene>_Map.png`, next to the scene. Save the scene first;
3. adds the map window and the minimap to the **UST UI** canvas, and assigns the Map input.

*Create Game UI* adds the window and the minimap too, without the area.

## Using the map

| Input | Keyboard / mouse | Gamepad | Does |
|---|---|---|---|
| Map | M (the M key on every keyboard layout) | d-pad right | Opens and closes the map. Escape or B closes it. |
| Left click | left button | A (gamepad cursor) | Places a pin, or removes yours when clicked |
| Right click | right button | X (gamepad cursor) | Pings the point for everyone, like the Ping input |
| Zoom | wheel, toward the mouse | **+** and **-** buttons | Between Min Metres and Max Metres |
| Move | drag | drag with A held | The map stops at the edges of the area |
| Center | **Center** button | **Center** button | Back on the player |

The map does not open over another window, nor while the player types in the chat. It opens centered on the player.

**Pins** are the placed markers of the [Marker Service](index.md#marker-service): they follow the same limit per player
(**Max Markers Per Player**, the oldest goes first), take the color of the player, show in the world too, and are
saved with the world. A click closer than **Pick Radius** to a pin acts on it: yours is removed, the pin of another
player is left alone. A pin lands on the ground under the clicked point (**Ground Layers** of the area); with nothing
under it, at the height of the area.

**Icons** on the map: pings (blinking), pins with the name of their owner, the other players with their name, the
places of the world (`WorldMarker`), and an arrow for the local player, turned to where the camera looks. Each kind
follows the Display options of the markers (*Pings*, *Placed markers*, *Markers on far players*, *Places of the
world*).

## Minimap

The minimap sits in the top right corner, north up, centered on the player. It hides while a window or a menu is
open, and when the scene has no map picture. Markers out of its view stay on its edge, other players excepted.

Players turn it off and set its zoom in *Options > Display*: **Minimap** (on) and **Minimap zoom** (150 m, from 50
to 500).

| Minimap field | Default | Meaning |
|---|---|---|
| Root | | Object shown and hidden: the frame and the view |
| View | | The `MapView` drawing the minimap |
| Metres | 150 | Metres shown across the minimap, unless the player set another zoom in the options |
| Hide With UI | on | Hidden while a window or a menu is open |

## Map Area

One per scene: the part of the world the map shows, and its picture.

| Field | Default | Meaning |
|---|---|---|
| Map Texture | | Picture of the area seen from above. Baked, or painted by hand. |
| Size | 500 x 500 | Width (X) and depth (Z) in metres, centered on the object |
| Outside Color | dark grey | Shown around the picture when the view goes past the area |
| Ground Layers | Default | Layers a pin placed on the map lands on |
| Search Height | 500 | Height above the area the ground is searched from, and the height of the bake camera |
| Bake Resolution | 2048 | Pixels of the longest side of the baked picture, 256 to 8192 |
| Bake Layers | all but UI | Layers drawn in the baked picture |
| Bake Background | dark grey | Color where nothing is drawn |

### Baking the picture

*Tools > UST > Advanced > Map > Bake Map Texture* renders the area from straight above, orthographic, with the
lighting of the scene and without fog, and saves `<Scene>_Map.png` next to the scene. Bake it again after changing the
scene. Pieces built during the game are not on the picture.

- The top of the map is the +Z direction of the world.
- Bake at daytime lighting: a scene lit for the night gives a dark map.
- *Fit Map Area To Scene* sizes the area again to the terrains, or to every renderer.
- For a hand-drawn map, assign your own texture to **Map Texture**, with the area covering the same part of the world.

## Map Window

| Field | Default | Meaning |
|---|---|---|
| Root, View | | Object shown and hidden, and the `MapView` of the window |
| Zoom In Button, Zoom Out Button, Center Button, Close Button | | Buttons of the window |
| Hint Text | | Line at the bottom with the controls |
| Hint, Gamepad Hint | | The controls with a mouse, and with the gamepad cursor. The line follows the device in use. |
| Start Metres | 400 | Metres shown across the map when it opens |
| Min Metres | 60 | Closest zoom |
| Max Metres | 0 | Farthest zoom. 0: the whole area, plus 10 % |
| Zoom Step | 1.25 | Zoom change per wheel notch or button |
| Pick Radius | 18 | Clicks closer than this to a pin, in canvas units, act on the pin |
| Toggle Action, Toggle Key | Player/Map, M | Opens and closes the map. The key is used when no action is assigned. |
| Close Action, Close Key | UI/Cancel, Escape | Closes the map |

## Map View

Draws a part of the area in a rectangle of the UI, with the markers and the player arrow. The map window and the
minimap each have one. Use your own for a map on a table, in a quest window, or a minimap of another shape: set its
`Center` (world X and Z) and `VisibleMetres` (metres across its width) from code.

| Field | Default | Meaning |
|---|---|---|
| Map Image | | `RawImage` showing the picture. Only the part inside the area is drawn. |
| Icon Layer, Icon Template | | Parent of the icons, stretched over the view, and the icon copied for each marker: an Image, with an optional text child for the name |
| Player Arrow | | Arrow of the local player |
| No Map Label | | Shown when the scene has no map area or no picture |
| Default Icon | | Icon of the markers without their own. Empty: a dot. |
| Show Pings, Show Placed, Show Teammates, Show Scene | on | Kinds of markers drawn, on top of the Display options |
| Show Labels | on | Names under the icons. Off on the minimap. |
| Keep On Edge | off | Markers out of view stay on the edge, players excepted. On for the minimap. |
| Edge Margin | 10 | Margin of the edge, in canvas units |
| Ping Size, Placed Size, Teammate Size, Scene Size | 30, 24, 18, 24 | Size of each kind of icon |

The view needs a `RectMask2D` so the picture does not draw outside it, and an Image as its background: its color is
the **Outside Color** of the area. Clicks, drags and the wheel over the view raise `Clicked`, `Dragged` and
`Scrolled`, which the map window uses.

## Scripting

```csharp
MapWindow.Current.Open();
MapWindow.Current.CenterOnPlayer();
MarkerService.Current.RequestToggleMarker(point);   // a pin there, or remove yours

myView.Center = new Vector2(x, z);                  // your own MapView
myView.VisibleMetres = 300f;
if (myView.TryScreenToWorld(screenPoint, null, out Vector3 ground)) { }
```

## Not in this version

- Fog of war: the whole map is visible from the start.
- Choosing an icon for a pin.
- A minimap turning with the player.
