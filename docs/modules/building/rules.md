# Rules, health and support

Part of [Building](index.md).

## Building rules

`BuildingRules`, one per scene, sets the rules of the session. Without it the defaults apply.

| Field | Default | Meaning |
|---|---|---|
| Pieces Take Damage | on | Built pieces have health and can be destroyed |
| Player Damage | on | Players can damage built pieces, for raids |
| Enemy Damage | on | Enemies and any non player attacker can damage built pieces |
| Ward Blocks Player Damage | off | Players cannot damage pieces inside a ward they are not part of |
| Default Hit Damage | 5 | Damage of a hit by an item without a Structure Damage trait, when the hit carries no damage |
| Projectile Damage Multiplier | 0.25 | Multiplies projectile damage on pieces: weapon damage is made for characters |
| Destroyed Refund Ratio | 0.5 | Part of the cost dropped on the ground when a piece is destroyed or collapses |
| Structural Support | on | Pieces with a Piece Support need support, and collapse without it. Off: everything holds. |
| Support Check Radius | 30 | Distance around a built or removed piece where support is computed again |

Typical setups: PvE with **Player Damage** off; PvP raids with everything on; a peaceful builder with **Pieces Take
Damage** off.

## Piece health

Add `PieceHealth` to a build piece to make it destructible. The demo pieces have one.

| Field | Default | Meaning |
|---|---|---|
| Max Health | 200 | |
| Damage Multiplier | 1 | Multiplies all damage taken: 0.5 for stone, 2 for thatch |
| Resistances | | By damage type, on top of the multiplier: stone takes little from Slash, a lot from Blunt. See [Damage types](../combat/index.md#damage-types). |

### How much damage a hit does

Every hit on a piece goes through three steps:

```
base damage
  1. the item has the Building > Structure Damage trait  ->  damage of the trait
  2. else, a projectile (bullet, bolt)                    ->  damage of the weapon
  3. else, a melee hit                                    ->  Default Hit Damage (Building Rules)

x Projectile Damage Multiplier (Building Rules)   projectiles only
x Damage Multiplier (Piece Health)                 the material of the piece
x Resistance to the damage type (Piece Health)     the type of the weapon or the attack
```

| Setting | Where | Scope |
|---|---|---|
| **Structure Damage** trait | On the item, *Add Trait > Building > Structure Damage* | That item only: a sledgehammer at 60, a sword at 5 |
| **Default Hit Damage** | Building Rules | Every melee item without the trait |
| **Projectile Damage Multiplier** | Building Rules | Every projectile, with or without the trait |
| **Damage Multiplier** | Piece Health | That piece: 0.5 for stone, 2 for thatch |

Example, with the defaults: a pistol dealing 25 to characters deals 25 x 0.25 = 6.25 to a wooden wall, and
3.1 to a stone wall at 0.5. A sledgehammer with Structure Damage 60 deals 60 to the wooden wall.

!!! tip "Give your tools the trait"
    The trait is how you make raiding tools: axes and hammers that break walls, weapons that barely scratch them.
    Without it, every melee item deals the same Default Hit Damage to pieces.

Damage also comes from anything else through `IDamageable.ServerTakeDamage(amount, source, attackerClientId)` in
UST.Core: enemies, traps, fire. The amount is used as the base damage, then multiplied by the Damage Multiplier of the
piece. An enemy hitting with `ItemHit` counts as an enemy, since its attacker is not a player. Melee and projectile hits
need the **Hit Layers** of the weapon to see the pieces.

At 0 health the piece is destroyed: its removal handlers run, its containers spill, and **Destroyed Refund Ratio** of
its cost drops on the ground. Health is saved with the piece, and the build HUD shows the health of the aimed piece.
Pieces placed in the scene never take damage. With a build tool in the hand, **R** repairs the aimed piece, for free,
like in Valheim. `ServerRepair(amount)` gives health back from your own code.

## Structural support

Add `PieceSupport` to a build piece to make it need support, like in Valheim. The demo pieces have one.

- A piece touching the ground has **full support** (100 %). The ground is anything solid that is not a build piece and
  does not move: terrain, rocks, scene buildings. Pieces placed in the scene by you also count as ground.
- Any other piece gets the best support of the pieces it touches, **minus a loss**: small when held from below
  (**Vertical Loss**), larger when held from the side (**Horizontal Loss**), and in between for diagonals.
- Under **Collapse Below**, the piece breaks: it drops **Destroyed Refund Ratio** of its cost and the content of its
  containers, like a destroyed piece.

| Field | Default | Meaning |
|---|---|---|
| Horizontal Loss | 0.2 | Support lost when held from the side. 0.2: four floors out from a pillar. |
| Vertical Loss | 0.125 | Support lost when held from below. 0.125: about seven levels high. |
| Collapse Below | 0.1 | The piece breaks under this support |

The loss is the one of the piece being held, so its material sets how far it reaches: a stone floor at 0.1 spans
twice as far as wood. Pieces without `PieceSupport` (furniture, decorations) never collapse and hold nothing.

In game, with the build tool:

- the preview takes the color of its support on the **Support Colors** gradient of `PlayerBuilder`: red just above
  the collapse limit, then orange, yellow, green, and blue on the ground. It turns to **Invalid Color** with *Not
  enough support*;
- the HUD shows the support of the preview and of the aimed piece.
- `SupportTint`, on the player, tints the built pieces with the same colors, so the weak point of a base shows before
  a pillar comes down.

| Field | Default | Meaning |
|---|---|---|
| Mode | Nearby | **None**, **Aimed Piece** only, or every piece within **Radius** |
| Radius | 30 | |
| Strength | 0.6 | How much of the support color replaces the color of the piece |
| Refresh Interval | 0.25 | Seconds between two updates of the tint |

The tint uses a property block on `_BaseColor` (URP) and `_Color` (Built-in), and is removed as soon as the build tool
leaves the hand. A shader without these properties is not tinted.

Removing or destroying a piece computes the support around it again on the next frame. Pieces left without support
collapse, which computes their neighbours again: an overhang falls apart piece by piece, over a few frames. Support is
computed by the server only, and computed again for every piece after loading a save.

## Stations

Each piece decides: a piece with a **Required Station** (field of `BuildPiece`) can only be built within the **Build
Range** of a station of that type (20 by default, on `CraftingStation`), and a piece without one can be built
anywhere. Wood pieces can need a Workbench and stone pieces a Forge, for example. The preview says *Needs a Workbench nearby*. Station types are the Crafting station
types, so a workbench both crafts and allows building. A built workbench works too: build it first, it should need no
station itself.

Your own stations can count: implement `IStation` and register it in `Stations`, in UST.Core.

## Range display

`BuildRangeDisplay`, on the player, shows while a build tool is in the hand:

- the area of every active ward nearby, blue when you are part of it, red otherwise;
- the build range of the stations the selected piece needs, in yellow. Nothing for a piece without a Required
  Station;
- the future area of a ward you are about to place, to see at once if it overlaps another one.

| Field | Default | Meaning |
|---|---|---|
| Mode | Ring | **None**, **Ring** on the ground, **Dome**, or **Ring And Dome** |
| Max Distance | 80 | Wards and stations further away are not shown |
| Own Ward Color, Foreign Ward Color, Station Color | blue, red, yellow | |
| Ring Material | M_RangeRing | Tinted by the colors |
| Ring Width | 0.08 | |
| Segments | 96 | Points of the ring |
| Ground Layers | Everything | The ring follows these layers: terrain, floors, foundations |
| Dome Material | M_RangeDome | Uses the **UST/Range Dome** shader: faint inside, brighter on the edge |
| Dome Alpha | 0.35 | |

The ring costs almost nothing and follows the ground. The dome is a large transparent surface: it covers the whole
screen when you stand inside, so it costs more on low end hardware and hides a bit of the view. The UST/Range Dome
shader works in the Built-in pipeline and URP; in HDRP, give the dome your own transparent material.
