# Build pieces

Part of [Building](index.md).

## Build piece

Add `BuildPiece` to the root of a network prefab.

| Field | Default | Meaning |
|---|---|---|
| Id | | Stable id used by saves, the GUID of the prefab. Set automatically. |
| Display Name, Icon, Description | | Shown in the build menu |
| Category | | Build Category asset: the tab of the build menu. Empty: an *Other* tab, shown last. |
| Cost | | Items taken from the inventory and the hotbar |
| Refund Ratio | 1 | Part of the cost given back when the piece is removed |
| Removable | on | Off for pieces players can never take down |
| Required Station | | Station type needed nearby, e.g. a Workbench. Empty: anywhere. See [Stations](rules.md#stations). |
| Snap Prefix | Snap | Children whose name starts with it are snap points |
| Needs Support | on | Must land on something or snap to a piece. Off: can float, for lights or signs. The server checks it too: a piece that touches nothing is refused. |
| Placement Offset | 0 | Added when the piece is not snapped, in its own space. A negative Y sinks a foundation into the ground. |
| Snap Priority | 0 | Snap points of pieces with a higher priority win near the aim. The demo pillar has 1, so a pillar joins the pillar under a floor rather than the floor. |

**Snap points** are empty children at the corners of the piece. Select a piece to see them as yellow spheres. The demo
pieces show the pattern: a flat piece has its 8 corners, a wall the 4 corners of its middle plane so it stands on the
edges of floors, and a pillar the centers of its two ends so it stands on their corners.

Two snap modes, switched in game:

- **Corners**: the snap points themselves.
- **Middle**: the middles of the edges between them, found automatically (two snap points that differ along a single
  axis). A wall centered on the middle of a foundation edge, a half wall on a pillar...

A snap point of a built piece is used when it is within **Snap Radius** of the aimed point, or of the aim ray itself:
aim past the edge of a floor, in the air, to place the next floor beside it, even upstairs on pillars. Every pair of
snap points gives a position. The one whose center is closest to where you aim, pushed out of the aimed face, and that
nothing blocks, wins. Snap points of the piece under the crosshair count first, however far from where you aim: aim at
the middle of the top of a wall to stack the next wall on it. Then come those of pieces with a higher **Snap
Priority**. Hold Shift to place freely.

Pieces may clip into each other by a quarter of their size on each axis, so a pillar can pass through the corner of a
floor. Anything else solid, and a real overlap with another piece, blocks the placement.

The space a piece takes is read from its colliders (box, sphere, capsule, mesh), or its meshes without colliders.
Anything solid inside that box blocks the piece, except the terrain and triggers. Pieces may touch their neighbours.

### Layers

Building does not depend on a layer: pieces are found by their `BuildPiece` component, and the preview has no
collider. You can put pieces on their own layer, for example for the camera Deoccluder. *Tools > UST > Advanced > Building > Put Pieces On Building Layer* creates a **Building** layer when missing and moves every piece of every Build Piece Database to it,
children included. *Create Demo Pieces* does the same for the demo pieces. Then check these masks:

| Mask | Why |
|---|---|
| **Placement Layers** of `PlayerBuilder` | Must include the layer, or nothing can be built on or snapped to the pieces |
| Interact mask of `PlayerInteractor` | Must include it to use built wards, chests and workbenches |
| **Hit Layers** of `PlayerItemUser` and of weapons | Only if pieces should be hit |
| Cinemachine Deoccluder | In **Collide Against**, the camera is pushed by walls. In **Transparent Layers**, it goes through. |

!!! warning "Pieces made of several models"
    The layer must be set on **every object of the piece** that has a collider, not only on the root. A piece built
    from several models, or with its collider on a child like the demo pieces (*Model*), keeps the old layer on those
    children otherwise: the Deoccluder or the placement mask then ignore part of the piece. When changing the layer of
    the root, answer **Yes, change children**.

## Build categories

*Create > UST > Building > Build Category*: one asset per tab of the build menu.

| Field | Meaning |
|---|---|
| Display Name | Name of the tab. Empty: the asset name. |
| Icon | For your own build menu |
| Order | Tabs are sorted by it, then by name |

Assets rather than text: a typo cannot hide a piece, and renaming a category changes nothing else. *Create Demo
Pieces* creates **Structure** and **Utility**. The Item Database warns about a build tool whose categories match no
piece of its database, or with an empty entry.

## Build Piece Database

*Create > UST > Building > Build Piece Database*. Lists the pieces of a tool, and travels over the network as indices:
every peer must use the same one. Right click its header > **Collect All Pieces In Project** to fill it.

## Chests, workbenches and other functional pieces

Any prefab becomes buildable: add `BuildPiece` to a storage or a crafting station prefab and list it in the database.

- A built **chest** (`StorageContainer`) saves its content with the piece.
- A built **workbench** (`CraftingStation`) saves its queue and its output with the piece, and gives the ingredients
  of its queue back when it is removed.

Your own components save with the piece by implementing `IObjectSaveable`, and react to removal with
`IRemovalHandler`, both in UST.Core.
