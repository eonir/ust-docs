# Gathering

## Tools

1. *Create > UST > Gathering > Tool Type*: Axe, Pickaxe, Sickle...
2. On the item: **Add Trait > Gathering > Tool**.

| Tool field | Meaning |
|---|---|
| Tool Type | Kind of tool |
| Tier | A number. Only the comparison with the node minimum tier matters, so you can insert tiers later. |
| Gather Multiplier | Multiplies the resources of each hit |
| Node Damage | Health removed from the node on each hit |
| Durability Loss Per Hit | Wear. A tool at 0 durability breaks and disappears. |

The item also needs **Melee Swing** to be swung.

## Resource nodes

`ResourceNode` goes on the root of a network prefab: a tree, a rock, a bush.

| Section | Fields |
|---|---|
| Requirements | Required tool (empty: anything, bare hands included), minimum tier, behaviour below the tier |
| Health | Max health, bare hand damage |
| Drops | Drops per hit and depletion drops, each with a min, a max and a chance. Delivered to the inventory (the rest falls on the ground) or on the ground. |
| Depletion | **Despawn** (an object spawner brings it back elsewhere) or **Regrow** in place after a delay |

**Below the minimum tier:**

- **Blocked**: nothing is gathered (Valheim style).
- **Reduced Yield**: gathering works with a reduced multiplier (Rust style).

`ResourceNode.LocalHitResult` is raised on the client of the player who hit, with *Gathered*, *Depleted*, *WrongTool*
or *TierTooLow*. Use it to show "Tool too weak".

## Object spawners

`ObjectSpawner` keeps up to **Max Alive** network objects in an area: sticks and stones to pick up, trees, rocks.
When one is picked up or depleted, another one appears after a random **Respawn Delay** at a random position.

| Section | Fields |
|---|---|
| Prefabs | Network prefabs with a weight each |
| Area | **Box** or **Sphere** (random position snapped to the ground) or **Points** (one of your transforms) |
| Placement | Ground layers, max slope, align to ground, random yaw, min spacing, **min player distance** |

Objects never appear closer than **Min Player Distance** to a player, so nobody sees them pop. Each spawner saves its
own objects and pending respawns.

!!! warning "Current limits"
    A node placed directly in a scene in **Despawn** mode comes back after a reload, and regrow timers are not saved.
    Use a spawner, or **Regrow**, for nodes placed in a scene.
