# Effects

The Effects module adds buffs and debuffs (well fed, poisoned, cold, broken leg...), zones that give an effect, and
consumable items (food, drinks, medicine). Effects are applied by the server and synchronized to every peer, which
applies the modifiers itself, so speed, max vitals and decay are the same everywhere.

The module is optional: add `UST_DISABLE_EFFECTS` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module).

## Setup

1. Create an **Effect Database**: *Create > UST > Effects > Effect Database*.
2. Create your effects: *Create > UST > Effects > Effect*.
3. Fill the database: right click its header > **Collect All Effects In Project**, or drag the effects in the list.
4. On the player prefab root, add `PlayerEffects` and assign the database.
5. For consumables, add `PlayerConsumer` on the same object as `PlayerInventory`, see [Consumables](#consumables).
6. To show the effects, add `EffectsBarView` to an empty object of the HUD.

!!! warning "One database for everyone"
    Effects travel over the network as their index in the database. Every peer must use the same database, and an
    effect missing from it cannot be applied: `ServerApply` logs a warning and returns false.

## Effect definition

| Field | Default | Meaning |
|---|---|---|
| Display Name, Icon, Description | | Shown in the effects bar |
| Positive | on | Shown as a good effect. Off for debuffs. |
| Duration | 30 | Seconds. **0** lasts until removed by code, for wounds or equipment. |
| Stacking | Refresh | What happens when the effect is applied again while active, see below |
| Max Stacks | 1 | Highest number of stacks with the **Stack** rule. **0** means no limit. |
| Clear On Death | on | Removed when the character dies. Effects held by a source (equipment, temperature, weather, zone) stay while the source holds them. |
| Saved | on | Kept in saves with its remaining time. Turn it off for effects given back by something else, such as equipment. |
| Show In UI | on | Shown in the effects bar |
| Modifiers | | What the effect does, added with **Add Modifier** |

Each effect has a stable id, its asset GUID, so saves survive renames.

### Stacking

| Rule | Applied again while active |
|---|---|
| Refresh | Restarts the duration |
| Extend | Adds the duration to what is left. Two bandages heal one after the other. |
| Stack | Adds a stack up to **Max Stacks** and restarts the duration. Modifiers scale with the stacks. |
| Ignore | Nothing changes |

!!! note "Stacks share one timer"
    With **Stack**, every stack ends at the same time: applying the effect again restarts the duration of all of
    them. To chain applications instead, use **Extend**.

### Modifiers

| Modifier (class) | What it does | With stacks |
|---|---|---|
| Vitals > Over Time (`VitalOverTimeModifier`) | Adds **Amount Per Tick** to a vital every **Interval** seconds. Positive heals or feeds, negative poisons or bleeds. | Amount × stacks |
| Vitals > Max (`VitalMaxModifier`) | Raises or lowers the max of a vital: **Flat**, then **Percent** (0.1 for +10 %) | Flat and percent × stacks |
| Vitals > Decay (`VitalDecayModifier`) | Multiplies the decay of a vital: 0.5 halves it, 2 doubles it, 0 stops it | Multiplier applied once per stack |
| Vitals > Regeneration (`VitalRegenModifier`) | Multiplies the regeneration of a vital: 0 stops it, for a wound | Multiplier applied once per stack |
| Movement > Speed (`MoveSpeedModifier`) | Multiplies the movement speed: 0.7 for -30 %, 1.2 for +20 % | Multiplier applied once per stack |

!!! warning "No stack limit with multipliers"
    Decay, Regeneration and Speed apply their multiplier once per stack. With **Max Stacks** at 0, a speed of 1.1
    reaches ×17 at 30 stacks. Keep a real limit on these effects.

!!! tip "Last tick"
    An **Over Time** tick that falls exactly on the end of the effect can be missed by a frame. For a duration of 5 s
    with a 1 s interval, set the duration slightly higher (5.1) if every tick must count.

## Player effects

`PlayerEffects` holds the effects active on a character. The server adds and removes them, every peer receives the
list and applies the modifiers.

### Applying from code

```csharp
// Server only.
effects.ServerApply(poisoned);                  // duration of the effect, following its stacking rule
effects.ServerApply(poisoned, 3);               // 3 stacks
effects.ServerApply(poisoned, 1, 10f);          // 10 s instead of the effect duration, 0 for permanent
effects.ServerRemove(poisoned);
effects.ServerClear();
```

### Effects maintained by a source

Use a stable source token for each independent contribution, such as a zone component or an equipment slot:

```csharp
effects.ServerApplyExternal(warmth, this);
effects.ServerRemoveExternal(warmth, this);
```

Both calls are server only. Registering the same source repeatedly is safe and does not add stacks. Removing one
source keeps the effect active while another source still provides it. Equipment uses a separate token for each
slot; weather, temperature and each zone use their component as the source. The same overloads are available through
`IEffectReceiver` in Core, using `EffectAsset`.

Several external sources provide one persistent effect, without adding stacks together. A normal application, such
as a potion, keeps its own stacks and expiration independently. While an external source is active, the combined
effect has no expiration; removing the last source leaves the normal application active for its remaining duration,
if any. If the potion expires first, the external contribution remains with one stack.

Only normal applications are saved; external contributions are reconstructed by equipment and the environment.
`ServerRemove` and `ServerClear` explicitly remove normal applications and external registrations.

The legacy overloads without a source remain available and use the effect asset as one shared source. Use explicit
source tokens when several objects provide the same effect. Custom implementations of `IEffectReceiver` must also
implement the two overloads taking `object source`.

### Reading

Valid on every peer:

| Member | Returns |
|---|---|
| `Has(effect)` | True if the effect is active |
| `GetStacks(effect)` | Stacks, 0 if not active |
| `GetRemaining(effect)` | Seconds left, -1 if permanent or not active |
| `GetActive(list)` | Every active effect with its stacks and remaining time |
| `IsDead` | True while the character is dead |

Events, raised on every peer: `Added`, `Removed`, `Changed` (stacks or duration).

### Death and saves

On death, effects with **Clear On Death** that were applied directly (food, potions, timed effects) are removed.
Effects held by a source, such as the warmth of worn equipment, a temperature stage or the weather, stay as long as
their source holds them, because those sources do not apply them again after a respawn.

Applying a timed effect over the same effect when it is permanent never makes it timed: with Refresh, Extend or
Stack, a permanent effect stays permanent. `PlayerEffects` is saved with the player: each effect with
**Saved** keeps its stacks and remaining time. Effects given by a zone are never saved.

## Effect zones

`EffectZone` gives an effect to the players inside the bounds of its collider: warmth near a fire, poison gas, a
healing spring. It runs on the server and sets the collider as a trigger.

| Field | Meaning |
|---|---|
| Effect | Effect given |
| Mode | **While Inside**: the effect lasts while the player stays inside and ends on leaving. **On Enter**: applied on entering with its own duration, and keeps running after leaving. |
| Check Interval | Seconds between two checks of the players inside |

Overlapping zones giving the same effect work together: leaving one of them keeps the effect while another still
covers the player. Dead players are ignored.

## Consumables

A consumable is an item with the **Effects > Consumable** trait. Using it changes vitals at once, starts effects,
then removes one item.

| Field | Default | Meaning |
|---|---|---|
| Instant | | Vitals changed once, right when the item is used. Negative to damage. |
| Effects | | Effects started when the item is used, each following its own stacking rule |
| Cooldown | 0.5 | Seconds before another consumable can be used |
| Consume Item | on | Removes one item after the use. Off for items that are never used up. |
| Animation | | Clip played on the upper body when the item is used, on every screen: a bandage applied, a medkit opened, food eaten. See below. |
| Animation Speed | 1 | Speed of the clip |
| Animation Trigger | | Animator trigger for a state of your own, used when no Animation is set. Empty for none. |

### Use animation

Each consumable can play its own clip: drop it in **Animation** on its trait. The legs keep walking, only the upper
body plays it.

1. The Animator Controller of the player needs the **Use Item** state, once: right click the controller in the Project
   window > **UST > Add Use Item State**. Controllers made by *Create Animator Controller* or *Add Upper Body Layer*
   have it. *Validate Scene* warns when consumables have an animation and the controller misses the state, with a fix
   button.
2. On each consumable, set **Animation**, and **Animation Speed** if needed.

The Use Item state sits on the UpperBody layer, is entered from any state by the `UseItem` trigger, and goes back to
the default state of the layer when the clip ends. Its clip, `UST_UseItem`, is a placeholder: at runtime
`PlayerUseAnimation` replaces it by the clip of the item through an Animator Override Controller, so one state serves
every item and the controller asset never changes. `UseSpeed` sets its speed.

The player who uses the item sees the animation at once; the others see it as soon as the server accepted the use.
`PlayerUseAnimation` is added to the player by `PlayerConsumer` when missing. Your own systems can play a use
animation the same way:

```csharp
GetComponent<PlayerUseAnimation>().Play(clip, 1f);
```

### Using a consumable

`PlayerConsumer` uses consumables in three ways:

| Input | Result |
|---|---|
| Hotbar key (1 to 0) | Uses the consumable in that slot. The held item stays in hand. |
| Right click in the inventory window | Uses the consumable, from the inventory or the hotbar |
| **Use Action** | Uses the selected hotbar item if it is a consumable. Usually the same action as the attack: the held item decides what the input does. |

The owner sends the request, the server checks the item and the cooldown, removes the item, then applies the vitals
and the effects. The item is removed first, so a failed removal never gives a free use.

| Field | Meaning |
|---|---|
| Inventory, Vitals, Effects | Found on the same object when empty |
| Use Action | Input that uses the selected hotbar item |
| Animator, Network Animator | Found in the children when empty |

Events: `UseStarted` on the owner, `Used` on the server. `ServerApply(consumable)` applies a consumable without any
item, for scripted rewards.

### Examples

| Item | Consumable trait | Effect |
|---|---|---|
| First aid kit | Instant: Health +10 | None |
| Bandage | Effects: Bandage | Bandage: Duration 5, Stacking **Extend**, Vitals > Over Time on Health, 2 per tick, interval 1 |
| Cooked meat | Instant: Hunger +30 | Well Fed: Duration 300, Vitals > Decay on Hunger, multiplier 0.5 |

With the bandage, using a second one while the first heals adds 5 s: the first finishes, then the second continues.

### Other ways to use an item

Hotbar keys and the inventory window ask every `IItemUseHandler` of the player before doing anything else. Implement
it on a component of the player to add your own uses (read a note, deploy a tent...):

```csharp
public class NoteReader : NetworkBehaviour, IItemUseHandler
{
    public bool TryUse(InventoryContainer container, int slot)
    {
        ItemDefinition item = container.GetDefinition(slot);
        if (item == null || !item.TryGetTrait(out NoteTrait note)) return false;

        // Show the note. Return true even if nothing happens, so the key does not select the slot instead.
        return true;
    }
}
```

## Writing your own modifier

Derive from `EffectModifier`, mark the class `[Serializable]` and add `[EffectModifierMenu("Category/Name")]` to place
it in the **Add Modifier** menu. The same instance is shared by every character with the effect: keep it pure data
and store per character state in the `EffectContext`.

| Method | Called | Use it for |
|---|---|---|
| `Apply(context)` | On every peer when the effect starts, and again when its stacks change | Registering multipliers with `context.Source` as the key, so a second call replaces the first |
| `Remove(context)` | On every peer when the effect ends | Removing what `Apply` registered |
| `ServerTick(context, deltaTime)` | Every frame on the server while the effect is active | Changes that must happen once, such as healing |

`EffectContext` gives the `Definition`, the `PlayerEffects`, the `Target` object, its `PlayerVitals`, its
`IMovementSpeedModifiers` and the current `Stacks`. `CountTicks(this, deltaTime, interval)` returns how many times a
modifier acting at a fixed rate should act this frame.

```csharp
[Serializable, EffectModifierMenu("Vitals/Drain Into")]
public class DrainModifier : EffectModifier
{
    public VitalDefinition from;
    public VitalDefinition to;
    public float amountPerSecond = 1f;

    public override void ServerTick(EffectContext context, float deltaTime)
    {
        if (context.Vitals == null || from == null || to == null) return;

        int ticks = context.CountTicks(this, deltaTime, 1f);
        if (ticks == 0) return;

        float amount = amountPerSecond * context.Stacks * ticks;
        context.Vitals.Modify(from, -amount);
        context.Vitals.Modify(to, amount);
    }
}
```

## Effects bar

`EffectsBarView` shows the effects of the local player as icons, with their stacks and remaining time. Put it on an
empty object of the HUD: the icons are created as children. Positive and negative effects get their own color.

| Field | Default |
|---|---|
| Icon Size | 48 |
| Spacing | 6 |
| Positive Color, Negative Color | green, red |
| Refresh Interval | 0.2 |

## Testing

In Play Mode, the `PlayerEffects` inspector lists the active effects with their stacks and time, with a **Remove**
button each, and an **Apply** field to give any effect. Effects are applied by the server: select the player in the
host editor.
