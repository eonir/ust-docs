# Vitals

A vital is any stat that goes up and down: health, hunger, thirst, stamina, temperature, sanity...

## Setup

1. *Tools > UST > Create Default Vitals* creates Health, Hunger, Thirst, Stamina and a profile.
2. On the player root, add `PlayerVitals` and assign the profile.
3. Optional: `VitalStaminaSource` (stamina for sprint and jump), `SprintDecayModifier`, `PlayerLife`, `VitalsDebugHUD`.

## Vital Definition

| Field | Meaning |
|---|---|
| Authority | **Server** for gameplay stats, **Owner** for stats that react to input (stamina) |
| Max Value, Start Value | |
| Decay Amount, Decay Interval | Lose *amount* every *interval* seconds. 0 disables decay. |
| Regen Amount, Regen Interval | Gain *amount* every *interval* seconds |
| Regen Delay | Seconds without loss before regeneration starts |
| Regen Requirements | Regenerate only while other vitals are above a ratio |
| Depletion Effects | While empty, drain another vital (no water: lose health) |

!!! tip "Smooth decay"
    Decay multipliers scale the amount of each tick. Use short intervals with small amounts (0.2 every 1.4 s
    rather than 1 every 7 s) for a smoother result.

## Profile

`VitalsProfile` lists the vitals of a character. Uncheck **Enabled** to turn one off without deleting it.

## Scripting

```csharp
vitals.Modify(health, -25f);                    // instant damage
vitals.ApplyOverTime(health, 40f, 10f, 1f);     // heal 40 over 10 s
vitals.ApplyOverTime(health, -15f, 5f, 0.5f);   // bleed 15 over 5 s
vitals.ClearOverTime(health);                   // stop bleeding
bool ok = vitals.TryConsume(stamina, 20f);      // spend if available

vitals.Changed += (vital, previous, current) => { };
vitals.Depleted += vital => { };
```

Every method can be called from any peer: the request is forwarded to whoever calculates the vital. Only the owner
of a player can send requests for that player.

`SetDecayMultiplier(vital, source, multiplier)` must be called on the peer that calculates the vital: the server for
server vitals, the owner for owner vitals.

## Death and respawn

`PlayerLife` kills the character when its health vital is empty, freezes its vitals, disables the components listed
in **Disable While Dead** for the owner, then respawns it after **Respawn Delay** on a random `PlayerSpawnPoint`.
Use `onDied` and `onRespawned` to plug a death screen or sounds without code.
