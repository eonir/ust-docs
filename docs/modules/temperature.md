# Temperature

The Temperature module gives the player a felt temperature, in degrees Celsius, from where they stand: the outdoor
temperature of the scene, cold caves and hot deserts, campfires and forges, roofs, and the clothes they wear. Past
thresholds, stages such as *Cold* or *Freezing* apply effects and damage, which resistances can counter.

It works without weather: a game with no weather and no day and night still has cold caves and warm houses. The
[day and night cycle](weather.md) only cools the nights through the outdoor temperature, see
[Outdoor temperature](#outdoor-temperature).

The module is optional: add `UST_DISABLE_TEMPERATURE` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module).

## Setup

1. Run *UST Tools > Advanced > Temperature > Add Temperature To Scene*: an `AmbientTemperature` with the outdoor temperature.
2. Select the player prefab and run *UST Tools > Advanced > Temperature > Add Temperature To Selected Player*. It adds
   `PlayerTemperature` with four stages (Cold, Freezing, Hot, Burning) and creates the **Cold** and **Heat** damage
   types in `Modules/Temperature/Data` if the project has none.
3. Place zones and heat sources, and give clothes insulation, see below.

The felt temperature and the current stage show in the [character stats panel](ui.md#character-stats-panel).

## How the felt temperature is made

```
outdoor temperature (AmbientTemperature + night offset + offsets of other systems)
  -> + Temperature Offset of the weather where the player stands (snow -10, rain -3)
  -> temperature zone: Add degrees, or Set a temperature (highest priority wins)
  -> + every heat source in range (full at the center, nothing at the radius)
  -> shelter: under a roof or in a Shelter zone, goes towards the Shelter Temperature
  -> insulation of the worn clothes: pulls it towards comfort, never past it
= target, that the felt temperature follows at Adapt Speed degrees per second
```

Example: outdoor 15, in a snow zone (Add -25) it is -10. Next to a campfire (+25 at the center, 6 m radius) at 3 m it
is 2.5. With furs giving 10 cold insulation it feels 10, the bottom of the comfort range: no stage.

## Outdoor temperature

`AmbientTemperature`, one per scene.

| Field | Default | Meaning |
|---|---|---|
| Base Temperature | 15 | Outdoor temperature of the scene |
| Shelter Temperature | 18 | Temperature under a roof, when Shelter Strength is 1 |
| Night Offset | -8 | Degrees added in full night, with a [day and night cycle](weather.md) in the scene |

Any system changes the whole scene with an offset, without knowing zones or players:

```csharp
AmbientTemperature.SetOffset(this, -8f);   // night
AmbientTemperature.RemoveOffset(this);     // day again
```

### Day and night

Temperature and the [day and night cycle](weather.md) are separate modules. The cycle never sets a temperature: it
only publishes its **darkness** through `WorldClock` (0 in full day, 1 in full night, smooth over twilight).
`AmbientTemperature` turns it into degrees with **Night Offset**:

```
outdoor temperature = Base Temperature + Night Offset x darkness + offsets of other systems
```

With Base Temperature 15 and Night Offset -8:

| Time | Darkness | Outdoor temperature |
|---|---|---|
| Noon | 0 | 15 |
| Sunset, middle of twilight | 0.5 | 11 |
| Midnight | 1 | 7 |

What happens to the rest at night:

| Element | At night |
|---|---|
| Zone in **Add** mode (snowfield, -25) | Follows the night: -10 at noon, -18 at midnight |
| Zone in **Set** mode (cave at -10, house at 20) | Same day and night: the outside does not reach inside |
| Heat source | Same heat, so a campfire matters most at night |
| Shelter | Goes towards the Shelter Temperature, day and night |

To set how cold nights are, change **Night Offset** on `AmbientTemperature`, not the Day Night Cycle. 0 keeps nights
as warm as days; -20 makes them deadly without a fire. Without a day and night cycle in the scene, the darkness stays
0 and nights change nothing. Without the Temperature module, the cycle runs the same.

## Temperature zone

`TemperatureZone` on an object with a collider, which can be a trigger.

| Field | Default | Meaning |
|---|---|---|
| Mode | Add | **Add**: degrees added to the outdoor temperature. **Set**: the temperature inside, whatever outside. **Shelter**: inside counts as under a roof. |
| Value | -10 | Degrees for Add, temperature for Set |
| Priority | 0 | The highest priority wins when Add and Set zones overlap. Shelter zones always count. |

## Heat source

`HeatSource`: a campfire, a forge, a torch, or a negative value for an ice block. Disable the component to turn it
off, for a fire that went out.

| Field | Default | Meaning |
|---|---|---|
| Heat | 25 | Degrees added at the center, fading to 0 at the radius |
| Radius | 6 | |

## Player temperature

| Field | Default | Meaning |
|---|---|---|
| Comfort Min, Comfort Max | 10, 25 | Insulation pulls the temperature towards this range, never past it |
| Adapt Speed | 1 | Degrees per second the felt temperature moves. Low values let a player cross a cold area quickly. |
| Shelter Strength | 0.8 | How far a roof brings the temperature towards the Shelter Temperature |
| Detect Roof | on | Anything solid above the player, within Roof Height, counts as a roof |
| Roof Layers | Everything | Set to the Building layer so only built roofs count, not trees |
| Roof Height | 8 | |
| Stages | Cold, Freezing, Hot, Burning | See below |
| Tick Interval | 0.5 | Seconds between two updates on the server |

### Stages

| Field | Meaning |
|---|---|
| Name, Color | Shown in the stats panel |
| Below | On: the stage applies under the threshold (cold). Off: over it (heat). |
| Threshold | Felt temperature where the stage starts |
| Effect | Optional effect kept active during the stage: slower, hungrier, colder vision... |
| Damage Per Second, Damage Type | Damage during the stage, of a type: Cold, Heat. The first tick comes one Tick Interval after the player spawns, and a late tick never catches up more than one extra interval, so a hitch or a late join cannot deal a burst of damage. |
| Countered At | The whole stage, effect and damage, is skipped when the player resists its damage type enough |

Defaults:

| Stage | Threshold | Damage | Countered at |
|---|---|---|---|
| Cold | under 5 | none | 0.5: clothes resisting half the cold keep it away |
| Freezing | under -5 | 1 per second, Cold | 0.25 |
| Hot | over 30 | none | 0.5 |
| Burning | over 40 | 1 per second, Heat | 0.25 |

When several stages are reached, the deepest wins: Freezing over Cold.

### Countering with resistances

Two ways to protect a player, that work together:

- **Insulation** (Cold Insulation, Heat Insulation of the [Equippable trait](equipment.md#equippable-trait)) changes
  the felt temperature: furs make the snow feel milder, so stages start later.
- **Resistances** to the Cold or Heat damage type (on the equipment, or natural on `PlayerDamageReceiver`) reduce the
  damage of the stages, and past **Countered At** skip the stage entirely, effect included. A cold-blooded race with
  Cold 0 never freezes.

Damage of the stages comes from the world: armor does not reduce it and clothes do not wear from it, only resistances
count.

## Not in this version

- A HUD icon of the temperature: the stats panel shows it.
- Weather: rain making players wet and colder. It will only set an offset and a stage.
