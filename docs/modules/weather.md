# Weather

The Weather module holds the day and night cycle and the weather. The sun and the moon cross the sky, and the light,
the ambient, the fog and the sky change through the day. On top, the weather of the world and of each weather zone
changes over time: clear, rain, storm, fog, snow, or anything you make. Day and night have their own length. Other modules read the clock through
`WorldClock` in UST.Core: nights are colder with the [Temperature](temperature.md) module, and enemies come out in
greater numbers with the [spawn director](ai.md#spawn-director).

The module is optional: add `UST_DISABLE_WEATHER` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module).

## Setup

Run *UST Tools > Advanced > Weather > Add Day Night Cycle To Scene*. It adds a **Day Night Cycle** object with a `NetworkObject`
and `DayNightCycle`, and uses the directional light of the scene as the sun. Optionally, add a second directional
light for the moon and assign it.

## Day Night Cycle

**Time**

| Field | Default | Meaning |
|---|---|---|
| Start Hour | 8 | Hour of a new world, from 0 to 24 |
| Sunrise, Sunset | 6, 20 | Hours the sun rises and sets |
| Day Minutes | 20 | Real minutes from sunrise to sunset |
| Night Minutes | 8 | Real minutes from sunset to sunrise: short nights, or long scary ones |
| Twilight Hours | 1.5 | Length of dawn and dusk, when the darkness fades in and out |
| Paused | off | Stops the clock, for a fixed time of day |
| Time Speed | 1 | Multiplies the speed of time on top of Day and Night Minutes: 10 to watch a day go by in tests, 0.5 for slower days. Set on the server, synchronized. |

**Sun and moon**

| Field | Default | Meaning |
|---|---|---|
| Sun | the scene sun | Directional light turned and tinted along the day |
| Moon | | Optional directional light for the night |
| Sun Azimuth | 30 | Direction of the path of the sun: where it rises |
| Max Sun Height | 65 | Height of the sun at noon, in degrees |

**Colors**: every gradient and curve goes from midnight on the left to midnight on the right, noon in the middle.

| Field | Meaning |
|---|---|
| Sun Color | Color of the sun along the day: blue night, orange dawn and dusk, white noon |
| Sun Intensity | Intensity curve. Its keys follow the default sunrise and sunset: move them if you change those hours. |
| Moon Color, Moon Intensity | Applied with the darkness |
| Ambient Color | Ambient light color, for the **Color** and **Gradient** environment lighting modes |
| Ambient Intensity | In full day and full night, for every environment lighting mode |
| Drive Fog, Fog Color | Changes the fog color along the day |
| Drive Skybox, Skybox Exposure | Changes the exposure of the skybox, on a copy of the material: the asset stays untouched. Play Mode only. |
| Drive Reflections, Reflection Intensity | Lowers the environment reflections at night (1 day, 0.15 night). Without it, metallic and glossy surfaces keep reflecting the day sky and glow at night. |

**Network**

| Field | Default | Meaning |
|---|---|---|
| Sync Interval | 2 | Seconds between two time updates from the server. Every peer moves the time itself in between, so the light stays smooth. |

The time is saved with the world.

## Testing an hour

The **Time Of Day** slider, at the top of the Day Night Cycle inspector, shows the light of any hour:

- in Edit Mode, it previews the lighting of the scene. The lighting before the first drag is remembered, and
  **Reset Preview** puts it back exactly (sun rotation, color and intensity, ambient, fog and reflections). The sun
  changes can also be undone with Ctrl+Z. Reset before saving the scene;
- in Play Mode, the world jumps to that hour for everyone, on the server. The inspector also shows the day number and
  the darkness.

!!! tip "Shiny surfaces at night"
    A metallic or glossy material that glows at night reflects the sky captured in daylight. **Drive Reflections**
    lowers the environment reflections with the darkness. Reflection probes baked in daylight do the same: bake them
    at a neutral hour, or give them a lower intensity.

## Weather

### Setup

1. Run *UST Tools > Advanced > Weather > Add Weather To Scene*. It adds a **Weather Director** with a `NetworkObject`, and
   creates five weather states in `Modules/Weather/Data`: Clear, Rain, Storm and Fog for the world, Snow for zones,
   with simple rain and snow particles.
2. For zones with their own weather, run *right click in the Hierarchy > UST > Weather Zone*: a 100 m trigger box with Snow,
   added to the director. Move and resize it, and change its weathers.
3. For effects on players out in the weather (Wet in the rain), select the player prefab and run *UST Tools > Advanced > Weather > Add Weather Exposure To Selected Player*.

The default particles are basic, made to be replaced: assign your own prefab to **Particles** of each state.

### Weather state

A **Weather State** (*Create > UST > Weather > Weather State*) is everything one weather changes. A new weather is a
new asset: a sand storm, ash rain, a blood moon fog.

| Field | Default | Meaning |
|---|---|---|
| Display Name, Icon | | For your UI |
| Duration Minutes | 3 to 8 | Real minutes the weather lasts, picked at random between min and max |
| Transition Seconds | 20 | Seconds to blend in from the previous weather |
| Particles | | Optional prefab around the camera: rain, snow, dust. Its emission follows the blend. |
| Particles Height | 12 | Height of the particles above the camera |
| Fog, Fog Density | off, 0.02 | Turns the fog on with this density. Off: the fog of the scene is kept. |
| Fog Color, Fog Color Amount | grey, 0.6 | How much this color replaces the fog color of the scene or of the day and night cycle |
| Sun Multiplier, Ambient Multiplier | 1, 1 | 0.4 for a dark storm |
| Sound, Volume, Indoor Volume | | Optional loop around the camera, quieter under a roof |
| Temperature Offset | 0 | Degrees added where this weather is, read by the [Temperature](temperature.md) module: -10 for snow |
| Wind | 0.2 | From 0 to 1, read with `WorldWeather.Wind(position)` for trees, flags, sails |
| Exposed Effect | | Optional effect on players out in this weather, not under a roof. Needs `WeatherExposure` on the player. |

### Weather director

| Field | Default | Meaning |
|---|---|---|
| Weathers | Clear 4, Rain 2, Storm 0.5, Fog 1 | Weathers of the world outside every zone, with their weight: 2 is picked twice as often as 1 |
| Zones | | Weather zones of the scene. Each one picks from its own table, at its own pace. |
| Zone Blend Seconds | 4 | Seconds to blend when the player walks from one zone into another. A change of weather in place uses the Transition Seconds of the new weather instead. |
| Roof Layers | Everything | What counts as a roof for the indoor volume |

When a weather ends, the next is picked from the table, and may be the same again. The server picks, every peer
shows the weather where its player stands. Weathers are saved with the world, with the time left.

### Weather zone

`WeatherZone` on an object with a collider, which can be a trigger, and listed on the director.

| Field | Default | Meaning |
|---|---|---|
| Weathers | | Weathers of the zone and their weight. One entry: always that weather, like a dome that is always clear. |
| Priority | 0 | Where zones overlap, the highest priority wins |

Snow on the mountain, sand storms in the desert, rain all the time in the swamp: one zone each.

A zone covers exactly its collider: a player is in the zone when inside the box, in the world weather outside it. The
weather follows the player within **Zone Blend Seconds**, and the particles already in the air finish falling for
their lifetime (9 seconds for the default snow), so a few flakes still fall just after leaving a snow zone. Shape the
collider to the area: a tall box covers a mountain top, a low wide box a valley.

### Weather exposure

`WeatherExposure`, on the player, gives the **Exposed Effect** of the weather where the player stands while they are
out in it, and ends it under a roof. The effect comes from the Effects module: a *Wet* effect can speed up the cold,
slow the player or drain stamina.

| Field | Default | Meaning |
|---|---|---|
| Roof Layers | Everything | Set to the Building layer, so trees do not count as roofs |
| Roof Height | 8 | |
| Check Interval | 1 | Seconds between two checks on the server |

### Testing a weather

In game, the [character stats panel](ui.md#character-stats-panel) shows the weather where the player stands, with
`WeatherExposure` on the player. In Play Mode, two inspectors show the current weather and let you change it at
runtime:

| Inspector | Shows | Buttons |
|---|---|---|
| Weather Director | The weather shown on this screen, then the world and every zone with its weather and the time left before it changes | One per weather of each table, to force it |
| Weather Exposure, on the player | Whether the player is under a roof, and the region they stand in with its weather and time left | One per weather of that region |

A forced weather lasts its normal duration, then the region picks again. From code, `ServerSetWeather` only accepts
states that are in one of the director's weather tables, because clients only know those: other states are refused
with a warning. Buttons only appear on the host or server:
a client sees the weather but cannot change it. The current weather is greyed out.

### With the day and night cycle

The director runs after the cycle every frame, and multiplies its light: a storm at noon is darker than a clear noon,
a storm at night is darker than a clear night. Without a cycle, the director starts from the light of the scene and
puts it back when it is removed. If the cycle does not drive the fog color (**Drive Fog** off), the director blends
from the fog color of the scene, captured when the director starts: set it by hand in the Lighting settings. The same
applies to the sun: if the cycle moves another light than the scene sun, the director uses the scene sun intensity
captured at start.

## Other modules and the clock

| Module | What changes | Setting |
|---|---|---|
| Temperature | Outdoor temperature drops at night, see [Day and night](temperature.md#day-and-night) | **Night Offset** of `AmbientTemperature`, -8 by default |
| Temperature | Each weather adds its **Temperature Offset** where it is: rain -3, snow -10 | On the Weather State |
| AI | More enemies at night, or only at night | **Night Count Multiplier** (1.5) and **Night Only** of `EnemySpawnDirector` |
| Sleep | Players skip the night in a bed, and the world catches up with the skipped time | See [Sleep](sleep.md) |

Your own code reads the same clock:

```csharp
IWorldClock clock = WorldClock.Current;   // null without a day and night cycle
if (clock != null && clock.IsNight) { /* light the torches */ }
float darkness = WorldClock.Darkness;     // 0 day, 1 night, smooth at twilight

DayNightCycle cycle = FindAnyObjectByType<DayNightCycle>();
cycle.Sunrise += day => Debug.Log("Day " + day);
cycle.ServerSkipToMorning();              // sleeping: systems catch up, see Sleep
cycle.ServerSetHour(22f);                 // a debug command
cycle.TimeSpeed = 60f;                    // server: one minute of play is an hour of game
```
