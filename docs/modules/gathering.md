# Gathering

The Gathering module covers everything players collect: resources hit with a tool (trees, rocks, ore), and objects
that appear again over time (sticks, stones, berries, whole trees). It can be disabled with the
`UST_DISABLE_GATHERING` define.

## Tools

### Tool types

`ToolType` (*Create > UST > Gathering > Tool Type*) is a kind of tool: Axe, Pickaxe, Sickle, Shovel...

### Tool trait

**Add Trait > Gathering > Tool** on the item. The item also needs a **Melee Swing** trait to be swung.

| Field | Default | Meaning |
|---|---|---|
| Tool Type | | Kind of tool |
| Tier | 1 | Strength of the tool. Only the comparison with the node minimum tier matters, so you can number freely and insert tiers later (1 stone, 2 bronze, 3 iron, or 10, 20, 30). |
| Gather Multiplier | 1 | Multiplies the resources of each hit |
| Node Damage | 1 | Health removed from the node on each hit |
| Durability Loss Per Hit | 1 | Wear per valid hit, if the item has durability. A tool at 0 breaks and disappears. |

A better tool is usually faster (more node damage) and more productive (higher multiplier).

## Resource nodes

`ResourceNode` goes on the root of a network prefab: a tree, a rock, an ore vein, a bush.

### Prefab

- root: `NetworkObject`, `ResourceNode`, a collider,
- child: the model.

Add it to the **Network Prefabs** list.

### Requirements

| Field | Default | Meaning |
|---|---|---|
| Required Tool | | Tool type needed. Empty: anything works, bare hands included (bushes). |
| Min Tier | 1 | Minimum tool tier |
| Below Tier | Blocked | What happens with a weaker tool |
| Reduced Yield Multiplier | 0.25 | Yield with a weaker tool, in Reduced Yield mode |

| Below Tier | Result | Feels like |
|---|---|---|
| **Blocked** | Nothing is gathered, the player gets "Tool too weak" | Valheim: you need bronze for oak |
| **Reduced Yield** | Gathering works with a reduced yield | Rust: any tool works, a better one is faster |

### Health

| Field | Default | Meaning |
|---|---|---|
| Max Health | 10 | Number of node damage points before depletion |
| Bare Hand Damage | 1 | Damage of a hit without a tool, when no tool type is required |

### Drops

| Field | Meaning |
|---|---|
| Drops Per Hit | Given on every hit |
| Depletion Drops | Given once, on the last hit (the big log when the tree falls) |
| Delivery | **To Inventory**: straight into the inventory, what does not fit falls on the ground. **To Ground**: everything falls on the ground. |

Each drop has an item, a min and max quantity, and a chance (0 to 1). The quantity is multiplied by the tool **Gather
Multiplier**, and by the **Reduced Yield Multiplier** below the tier.

### Depletion

| Field | Default | Meaning |
|---|---|---|
| When Depleted | Despawn | **Despawn** or **Regrow**, see below |
| Regrow Delay | 300 | Seconds before a Regrow node is full again |
| Show Growth | on | Regrow: the node is shown growing back instead of hidden |
| Growth Target | | Transform scaled while growing. A child holding the model is best. Empty: the whole object. |
| Growth Start Scale | 0.2 | Scale at the start of the growth |

**Despawn.** The node disappears.

- Created by an `ObjectSpawner`: the spawner creates a new one after its respawn delay, at a new random position.
- Placed in the scene by hand: it is gone for good, and the save remembers it. Use it for unique resources.

**Regrow.** The node stays where it is. It loses its colliders, so it cannot be hit, and either hides or grows back
from **Growth Start Scale** to full size. It can be gathered again once fully grown.

Only the end time of the growth is synchronized: every peer computes the scale itself, so nothing is sent while the
node grows, and a player joining sees the right stage at once.

!!! tip "Which mode?"
    **Despawn with a spawner** for resources that move around: wood, stones, the trees of a forest. **Regrow placed by
    hand** for fixed resources: a berry bush near the base, an ore vein always at the same place.

!!! warning "Spawner and Regrow"
    Avoid Regrow on nodes created by a spawner. They never disappear, so the spawner never replaces them, and the
    spawner only saves the position of its objects. A warning is logged when a spawner holds a Regrow node.

### Feedback

`ResourceNode.LocalHitResult(node, result)` is raised on the client of the player who hit:

| Result | Meaning |
|---|---|
| Gathered | Resources were given |
| Depleted | Last hit, the node is depleted |
| WrongTool | The tool type does not match |
| TierTooLow | The tool tier is too low, in Blocked mode |

`GatherFeedback`, placed once in the scene, turns WrongTool and TierTooLow into on screen messages:

| Field | Default |
|---|---|
| Wrong Tool Text | `Requires: {0}` |
| Tier Too Low Text | `Tool too weak: {0} tier {1} required` |

`{0}` is replaced by the name of the required tool type and `{1}` by the minimum tier. Both can be moved or removed,
which also suits translations. The messages are shown by a [Screen Message View](ui.md#screen-messages).

### Saving

`ResourceNodesSaver`, in the scene, saves the nodes placed in the scene: their health, whether they are depleted, and
the time left before they regrow. On load, a node missing from the save was depleted in Despawn mode and is removed
again. Nodes created by a spawner are saved by their spawner instead.

## Object spawners

`ObjectSpawner` keeps up to **Max Alive** network objects in an area. When one disappears (picked up, depleted), a
new one appears after a random delay at a random valid position. It works with any network prefab: world items to
pick up, resource nodes, or your own objects.

### Content

| Field | Default | Meaning |
|---|---|---|
| Spawner Id | auto | Unique id used by saves. Generated automatically, a duplicated spawner gets a new one. |
| Prefabs | | Network prefabs with a weight each: Stick 70, Stone 30 |
| Max Alive | 10 | Objects present at the same time. The area is filled when the server starts. |
| Respawn Delay | 60 to 120 | Random delay, in seconds, between a disappearance and the replacement |
| Retry Delay | 5 | Delay before trying again when no valid position was found |

For world items, the content of each object comes from **Placed Item** and **Placed Count** on the `WorldItem` of
the prefab.

### Area

| Shape | Placement |
|---|---|
| **Box** | Random point in the box, snapped to the ground by a raycast from the top of the box |
| **Sphere** | Random point in the sphere, snapped to the ground the same way |
| **Points** | One of the transforms of the **Points** list, snapped to the ground when there is ground right below |

The area is drawn in green when the spawner is selected. Use **Points** for complex places, such as caves or bridges,
where a raycast from above could land on the wrong floor.

### Placement rules

| Field | Default | Meaning |
|---|---|---|
| Ground Layers | Everything | Layers objects can appear on |
| Max Slope | 35 | Steepest ground accepted, in degrees |
| Align To Ground | off | Tilts the object to follow the ground |
| Random Yaw | on | Random rotation around Y |
| Min Spacing | 2 | Minimum distance between two objects of the spawner |
| Min Player Distance | 15 | Objects never appear closer to a player, so nobody sees them pop. Ignored when the world is first filled. |
| Placement Attempts | 10 | Random positions tried before giving up until the next retry |
| Initial Fill Delay | 0.5 | Delay before the first fill, which leaves time for the save to be restored |

!!! tip
    Keep the area large enough for **Max Alive** objects at **Min Spacing**. Otherwise the spawner keeps retrying
    without finding room. For testing, set **Min Player Distance** to 0 and a short respawn delay.

### Saving

Each spawner saves its own objects (position, rotation and, for world items, their content) and the time left
before each pending respawn. Its objects are marked with `SpawnerManagedObject`, so `WorldItemsSaver` and
`ResourceNodesSaver` skip them and nothing is loaded twice.
