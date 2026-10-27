# Combat

## Swinging

`PlayerItemUser`, on the player root, swings the held item when the attack input is pressed, if the item has a
**Melee Swing** trait.

1. The owner checks the cooldown and the stamina, plays the animation trigger, and waits for the **Impact Delay**.
2. At impact, the owner casts a sphere from the camera and sends the target to the server.
3. The server checks the item actually held, the cooldown and the distance, then calls every `IItemHitReceiver` of
   the target.

| Melee Swing field | Meaning |
|---|---|
| Range | Reach, measured from the character chest |
| Radius | Thickness of the sphere cast |
| Cooldown | Seconds between two swings |
| Impact Delay | Seconds between the start of the swing and the hit. Match it to your animation. |
| Stamina Cost | 0 spends nothing |
| Animation Trigger | Sent through the `NetworkAnimator` so every player sees it |

!!! note
    The impact uses a timer instead of an animation event, so it works with any animation, placeholders included.

Attacks are ignored while a UI is open: call `UIInputFocus.Push()` when your UI opens and `UIInputFocus.Pop()` when it
closes.

## Damage

`PlayerDamageReceiver`, on the player root, removes the **Melee Weapon** damage from the health vital. Dead players are
ignored.

## PvP

**Allow Player Damage** is off by default: players cannot hurt each other. Turn it on in the inspector, or at runtime
on the server:

```csharp
damageReceiver.AllowPlayerDamage = true;
```

## Reacting to hits

Implement `IItemHitReceiver` on the root of any network object: breakables, doors, creatures.

```csharp
public class Barrel : NetworkBehaviour, IItemHitReceiver
{
    public void ServerReceiveHit(in ItemHit hit)
    {
        // Server only, the hit is already validated.
        NetworkObject.Despawn(true);
    }
}
```
