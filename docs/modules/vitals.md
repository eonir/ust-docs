# Vitals

A vital is any stat that goes up and down: health, hunger, thirst, stamina, temperature, sanity, oxygen... UST makes
no difference between them: each vital is a `VitalDefinition` asset, and a character gets the vitals listed in its
profile. Adding a new one needs no code.

## Quick setup

1. *Tools > UST > Create Default Vitals* and pick a folder. It creates Health, Hunger, Thirst, Stamina and a profile.
2. On the player root, add `PlayerVitals` and assign the profile.
3. Add the optional components you need:

| Component | Role |
|---|---|
| `VitalStaminaSource` | Lets `PlayerController` spend the stamina vital to sprint and jump |
| `SprintDecayModifier` | Makes some vitals decay faster while running |
| `PlayerLife` | Death, respawn, spawn points |
| `VitalsDebugHUD` | Debug bars, kill key and +/- buttons |

## Vital definition

Create one with *Create > UST > Vitals > Vital Definition*.

### General

| Field | Meaning |
|---|---|
| Display Name | Name shown in the UI |
| Icon | Icon shown in the UI |
| Color | Color of the vital bar |
| Authority | Who calculates the vital, see below |
| Max Value | Upper limit. Values are always clamped between 0 and max. |
| Start Value | Value on spawn and after a respawn |

### Authority

| Authority | Calculated by | Use for |
|---|---|---|
| **Server** | The server | Gameplay stats: health, hunger, thirst. Clients cannot cheat them. |
| **Owner** | The owning client | Stats that must react instantly to input: stamina. |

A server vital checked by a client with 100 ms of latency would let the player sprint at zero stamina and then be
corrected, which feels terrible. That is why stamina is calculated by its owner and synchronized to the others.

### Decay

| Field | Meaning |
|---|---|
| Decay Amount | Amount lost on each tick |
| Decay Interval | Seconds between two ticks. 0 disables decay. |

Decay works in ticks on purpose: each change is a network message, so one message every few seconds is cheap while a
change every frame is not.

!!! tip "Smooth decay"
    For a smoother bar, use shorter intervals with smaller amounts: 0.2 every 1.4 s rather than 1 every 7 s.

### Regeneration

| Field | Meaning |
|---|---|
| Regen Amount | Amount gained on each tick |
| Regen Interval | Seconds between two ticks. 0 disables regeneration. |
| Regen Delay | Seconds without any loss before regeneration starts again |
| Regen Requirements | Other vitals that must be at or above a ratio (0 to 1) to regenerate |

Every loss caused by damage, consumption or a depletion effect restarts the **Regen Delay**. Decay does not.

Example: health regenerates only while hunger and thirst are above 30 %:

| Vital | Min Normalized |
|---|---|
| Hunger | 0.3 |
| Thirst | 0.3 |

### Depletion effects

Effects applied while the vital is **at zero**.

| Field | Meaning |
|---|---|
| Target | The vital to drain |
| Drain Amount | Amount removed from the target on each tick |
| Interval | Seconds between two ticks |

Example: no water drains 1 health every 2 seconds. With 100 health, the player dies of thirst in about 3 minutes 20,
twice as fast if hunger is empty too.

!!! note
    Each drain restarts the regen delay of the target. Keep the **Regen Delay** of health longer than the depletion
    interval, or regeneration would squeeze in between two drains and compensate them.

### Default values created by the tool

| Vital | Authority | Decay | Regen | At zero |
|---|---|---|---|---|
| Health | Server | none | 1 every 2 s, after 5 s, if hunger and thirst are at 30 % or more | Death (with `PlayerLife`) |
| Hunger | Server | 1 every 10 s | none | -1 health every 2 s |
| Thirst | Server | 1 every 7 s | none | -1 health every 2 s |
| Stamina | Owner | none | 2 every 0.1 s, after 1 s | nothing |

## Vitals profile

`VitalsProfile` lists the vitals of a character. Uncheck **Enabled** on an entry to turn a vital off without removing
it, for example to make a game without hunger. Different characters can use different profiles.

## Runtime values

`PlayerVitals` holds the current values. Vital definitions are shared assets and never hold any value, so changing a
definition in Play Mode changes the rules for everyone but not anybody's current value.

!!! warning "Play Mode changes stay"
    ScriptableObjects are assets: a value changed in Play Mode is still there after you stop. Take care not to keep
    a test value, such as thirst decaying 50 per second.

## Scripting

### Reading

```csharp
float health = vitals.GetValue(healthVital);
float ratio = vitals.GetNormalized(healthVital);    // 0 to 1
bool hasThirst = vitals.Has(thirstVital);           // false if disabled in the profile

foreach (VitalDefinition vital in vitals.Definitions) { }
```

Values are synchronized, so reading works on every peer.

### Changing

```csharp
vitals.Modify(healthVital, -25f);                   // instant damage
vitals.Modify(hungerVital, 30f);                    // food
bool spent = vitals.TryConsume(staminaVital, 20f);  // spend only if available

vitals.ApplyOverTime(healthVital, 40f, 10f, 1f);    // heal 40 over 10 s, one tick per second
vitals.ApplyOverTime(healthVital, -15f, 5f, 0.5f);  // bleed 15 over 5 s
vitals.ClearOverTime(healthVital);                  // bandage: stop every gradual effect
```

Every method can be called from any peer. When the caller does not calculate the vital, the request is forwarded to
the peer that does: the server for server vitals, the owner for owner vitals. Only the owner of a character can send
requests for it, so a client cannot heal or kill somebody else.

### Decay multipliers

```csharp
vitals.SetDecayMultiplier(thirstVital, this, 2f);   // thirst decays twice as fast
vitals.RemoveDecayMultiplier(thirstVital, this);
```

- Several sources can register a multiplier on the same vital, they multiply each other (x2 and x1.5 give x3).
- The source is any Unity object, usually the component that applies the effect.
- Call it on the peer that calculates the vital: the server for server vitals, the owner for owner vitals. The
  source cannot travel over the network, so a call on the wrong peer is ignored.
- The multiplier scales the amount of each tick, not its frequency.

### Events

```csharp
vitals.Changed += (vital, previous, current) => { };
vitals.Depleted += vital => { };    // reached zero
vitals.Restored += vital => { };    // left zero
```

Events are raised on every peer.

!!! note "Late joiners"
    A client joining a running game receives the current values but no `Changed` event for them. A UI should read
    the values when it initializes.

## Stamina

`VitalStaminaSource` connects `PlayerController` to a stamina vital:

| Field | Default | Meaning |
|---|---|---|
| Vitals | | The `PlayerVitals` of the character |
| Stamina | | The vital used as stamina |
| Flush Interval | 0.1 | Consumption is batched and sent every 0.1 s instead of every frame |

Without this component, the controller has unlimited stamina.

## Faster decay while running

`SprintDecayModifier` applies decay multipliers while the character sprints. Add one entry per vital, such as
Thirst x2 and Hunger x1.5. The owner shares its running state over the network, then every peer applies the
multiplier, which only matters on the peer that calculates each vital.

## Death and respawn

`PlayerLife` kills the character when its health vital is empty. The server decides the death.

| Field | Default | Meaning |
|---|---|---|
| Vitals | | The `PlayerVitals` of the character |
| Health | | The vital whose depletion kills |
| Respawn Delay | 5 | Seconds before the respawn |
| Use Spawn Points On Join | true | New players arrive on a random spawn point |
| Spawn Height Offset | 0.1 | Height added to every spawn position, so the capsule never starts inside the ground |
| Disable While Dead | | Components disabled for the owner while dead: controller, input, item user, hotbar input... |
| Animator / Dead Parameter | Dead | Bool set on the animator if the parameter exists |
| On Died / On Respawned | | UnityEvents raised on every peer, for a death screen or sounds without code |

**On death:** vitals are frozen (no decay, no regeneration, no gradual effect), the components of **Disable While
Dead** are turned off for the owner, and the `Died` event is raised. `PlayerDeathDrop` uses it to drop the inventory.

**On respawn:** every vital goes back to its start value, gradual effects are cleared, the character is moved to a
random `PlayerSpawnPoint` (or to where it first appeared if there is none), and `Respawned` is raised.

`Kill()` and `Respawn()` are public and server only, for a "Respawn now" button or a death zone.
`RespawnRemaining` gives the seconds left before the respawn on every peer, as shown by the
[death screen](ui.md#death-screen).

### Spawn points

Add `PlayerSpawnPoint` to empty GameObjects in the scene. They show as cyan spheres with an arrow for the facing
direction. A random enabled one is picked on join and on respawn.

## Debug HUD

`VitalsDebugHUD` draws one bar per vital at the top left for the local player.

- **K** kills the local player: assign the health vital to **Kill Vital**.
- **-10 / +10** buttons appear next to each bar when the cursor is free, for example with the inventory open.

It is drawn with IMGUI and is not meant to ship.
