# Combat

The Combat module lets players swing the item they hold. The same swing gathers resources and hurts characters: what
a hit does depends on the traits of the item and on what it hits.

## How a swing works

`PlayerItemUser`, on the player root, handles the swing.

1. **Owner.** The attack input is pressed. The owner checks that the held item has a **Melee Swing** trait, that the
   cooldown is over and that there is enough stamina. It spends the stamina, plays the animation trigger and waits
   for the **Impact Delay**.
2. **Owner.** At impact, it casts a sphere from the center of the screen, ignoring the player itself. If it hits a
   network object with an `IItemHitReceiver` within range, it sends the hit to the server.
3. **Server.** The server checks the item actually held, the cooldown (25 % tolerance for latency) and the distance
   (range plus a tolerance), then calls every `IItemHitReceiver` of the target.

Detecting on the owner keeps aiming responsive. Validating on the server prevents a modified client from hitting
across the map or faster than the item allows.

!!! note "Impact delay instead of animation events"
    The hit happens after a timer rather than an animation event. It works with any animation, placeholders
    included, and a missing event can never cancel the hit.

## Melee Swing trait

**Add Trait > Combat > Melee Swing** on the item.

| Field | Default | Meaning |
|---|---|---|
| Range | 2.5 | Reach, measured from the chest of the character |
| Radius | 0.3 | Thickness of the sphere cast. Bigger is more forgiving. |
| Cooldown | 0.8 | Seconds between two swings |
| Impact Delay | 0.3 | Seconds between the start of the swing and the hit. Match it to the animation. |
| Stamina Cost | 0 | Stamina spent per swing. 0 spends nothing. |
| Animation Trigger | Attack | Animator trigger, sent through the `NetworkAnimator` so everyone sees it. Empty: none. |

## Player Item User

| Field | Default | Meaning |
|---|---|---|
| Inventory | | The `PlayerInventory`, found automatically |
| Attack Action | | Input action of the attack, usually Player/Attack |
| Hit Layers | Everything | Layers the swing can hit |
| Chest Height | 1.2 | Height distances are measured from |
| Server Range Tolerance | 1 | Extra reach accepted by the server |
| Animator, Network Animator | | Found automatically in the children |

`Swung(item)` is raised on the owner when a swing starts, for sounds or camera effects.

Add `PlayerItemUser` to **Disable While Dead** on `PlayerLife`, so the dead do not swing.

## Damage to players

**Add Trait > Combat > Melee Weapon** on the item and set its **Damage**.

`PlayerDamageReceiver`, on the player root, removes that damage from the health vital.

| Field | Meaning |
|---|---|
| Vitals, Health | The vital that loses the damage |
| Allow Player Damage | PvP. Off by default: players cannot hurt each other. |

Dead players ignore hits. PvP can be switched at runtime on the server:

```csharp
damageReceiver.AllowPlayerDamage = true;
```

## Hits and UI

Attacks are ignored while a UI is open, so clicking a slot of the inventory never swings. Call `UIInputFocus.Push()`
when your UI opens and `UIInputFocus.Pop()` when it closes. See [UI](ui.md).

## Reacting to hits

Implement `IItemHitReceiver` on the root of any network object: breakable crates, doors, creatures.

```csharp
using Unity.Netcode;
using UST.Inventory;

public class Barrel : NetworkBehaviour, IItemHitReceiver
{
    public void ServerReceiveHit(in ItemHit hit)
    {
        // Server only, the hit is already validated.
        // hit.item is the item used, hit.attacker the network object of the attacker.
        NetworkObject.Despawn(true);
    }
}
```

`ItemHit` contains the attacker, its client id, the item, the hit point and the direction. `hit.ByPlayer` tells
whether the attacker is a player.

## Known limits

- The swing is aimed from the camera: a player can hit what the crosshair shows even if the weapon model does not
  visually pass through it. This is the usual choice for third person games.
- Weapons do not wear yet. Tools wear when they gather.
