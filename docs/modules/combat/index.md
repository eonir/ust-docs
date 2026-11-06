# Combat

The Combat module lets players swing the item they hold. The same swing gathers resources and hurts characters: what
a hit does depends on the traits of the item and on what it hits.

This module is described on several pages:

- [Melee](melee.md)
- [Ranged weapons](ranged.md)

## Damage to players

**Add Trait > Combat > Melee Weapon** on the item and set its **Damage** and **Damage Type**.

`PlayerDamageReceiver`, on the player root, removes that damage from the health vital, after resistances.

| Field | Meaning |
|---|---|
| Vitals, Health | The vital that loses the damage |
| Allow Player Damage | PvP. Off by default: players cannot hurt each other. |
| Resistances | Natural resistances of the character by damage type, see [Damage types](#damage-types) |

It takes hits from held items and damage from enemies and traps (`IDamageable`), with the same PvP rule.

Dead players ignore hits. PvP can be switched at runtime on the server:

```csharp
damageReceiver.AllowPlayerDamage = true;
```

## Damage types

A **Damage Type** (*Create > UST > Combat > Damage Type*) names a kind of damage: Slash, Blunt, Pierce, Fire... Create
the ones your game needs: none is built in, so a game without types simply leaves them empty.

Who deals a type:

| Source | Field |
|---|---|
| Melee weapons | **Damage Type** of the Melee Weapon trait |
| Ranged weapons | **Damage Type** of the Ranged Weapon trait |
| Enemies | **Attack Damage Type** of the [Enemy Definition](../ai.md#enemy-definition) |
| Your code | the `type` argument of `IDamageable.ServerTakeDamage` |

### Which damage a hit deals

**Melee hits** (swing of the held item):

| Receiver | Damage, in this order | Type |
|---|---|---|
| Enemy | Melee Weapon trait, else **Default Hit Damage** of its definition | Melee Weapon trait |
| Player | Melee Weapon trait, else nothing | Melee Weapon trait |
| Built piece | **Structure Damage** trait, else **Default Hit Damage** of the Building Rules. The Melee Weapon damage is ignored, so a sword made to kill does not break walls like a sledgehammer. | Melee Weapon trait |

**Ranged hits** (bullets, bolts, arrows): every projectile carries the **Damage** of its Ranged Weapon trait.

| Receiver | Damage | Type |
|---|---|---|
| Enemy | Ranged Weapon damage | Ranged Weapon trait |
| Player | Ranged Weapon damage, if **Allow Player Damage** is on for a player shooter | Ranged Weapon trait |
| Built piece | **Structure Damage** trait if the weapon has one, else the Ranged Weapon damage. Then **x Projectile Damage Multiplier** of the Building Rules (0.25 by default), in both cases. | Ranged Weapon trait |

A shotgun deals its damage once per pellet that hits. Every projectile hits once: it stops at the first collider.

**Last, for every hit**: the resistances of the receiver to the type, then its `IDamageModifier` components, such as
armor.

Examples:

- an axe with Tool and Melee Weapon (20, Slash) hits an enemy for 20, a skeleton with Slash 0.5 for 10;
- a pickaxe with only the Tool trait hits an enemy for its Default Hit Damage;
- a rifle at 30 Pierce hits a skeleton with Pierce 0.3 for 9, and a wooden wall for 30 x 0.25 = 7.5;
- a crossbow at 40 Pierce with a Structure Damage of 60 hits that wall for 60 x 0.25 = 15.

Who resists it, with a list of **Resistances**: a damage type and a multiplier. 1 takes the damage, 0.5 halves it, 0
is immune, 2 doubles it. A type not in the list, or an untyped damage, is taken in full.

| Receiver | Resistances |
|---|---|
| Players | `PlayerDamageReceiver`, plus every `IDamageModifier` on the player, such as the [equipment](../equipment.md#armor) |
| Enemies | Their [Enemy Definition](../ai.md#enemy-definition), plus their `IDamageModifier` components |
| Built pieces | [Piece Health](../building/rules.md#piece-health), on top of its Damage Multiplier |

Examples: a skeleton with Pierce 0.3 and Blunt 2 shrugs off arrows and breaks under a club. A stone wall with Slash
0.2 laughs at swords and needs a sledgehammer.

`IDamageModifier` (UST.Core) lets any component change the damage its object takes: armor, a shield buff, a god mode
cheat. `ModifyDamage(amount, type, source)` returns the new amount; every receiver of UST applies all of them.

## Hits and UI

Attacks are ignored while a UI is open, so clicking a slot of the inventory never swings. Call `UIInputFocus.Push()`
when your UI opens and `UIInputFocus.Pop()` when it closes. See [UI](../ui/index.md).

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

`ItemHit` contains the attacker, its client id, the item, the hit point, the direction, the **damage** and the
**kind** of hit (`Melee` or `Projectile`). `hit.ByPlayer` tells whether the attacker is a player. A damage of 0 means
the receiver reads it from the item, as melee hits do.

Resource nodes only react to melee hits: shooting a tree does not gather it.

## Facing the attack

Melee swings and shots turn the character towards where the camera aims, for the duration of the attack. Scripts call
`IAimFacing.FaceAim(duration)`, implemented by `PlayerController`: your own abilities can use it too.

## Known limits

- Ranged hits are validated against the positions the server knows, which are slightly late for moving players.
  There is no lag compensation yet.

- The swing is aimed from the camera: a player can hit what the crosshair shows even if the weapon model does not
  visually pass through it. This is the usual choice for third person games.
- Weapons do not wear yet. Tools wear when they gather.
