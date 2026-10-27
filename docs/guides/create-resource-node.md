# Create a resource node

Example: a tree that needs an axe.

1. *Create > UST > Gathering > Tool Type*, name it Axe.
2. On your axe item: **Melee Swing** and **Tool** (type Axe, tier 1) traits.
3. Create a prefab: root with `NetworkObject`, `ResourceNode` and a collider, the tree model as a child.
4. On `ResourceNode`: required tool Axe, minimum tier 1, health 5, drop per hit 1 to 2 Wood, depletion drop 5 Wood,
   depleted behaviour **Despawn**.
5. Create an `ObjectSpawner` in the scene, add the tree prefab and set **Max Alive**.
6. Run *Tools > UST > Validate Network Prefabs*.

For a stronger tree, set its minimum tier to 2: a tier 1 axe is blocked, or gathers less with **Reduced Yield**.

## Feedback

- Add `GatherFeedback` to the scene and a [Screen Message View](../modules/ui.md#screen-messages) to the Canvas:
  hitting with the wrong tool shows "Requires: Axe".
- Add an [Item Feed View](../modules/ui.md#item-feed): every hit shows "Wood +2", merged while you keep chopping.

## A tree that grows back in place

For a tree placed by hand that should always come back at the same place:

1. Set **When Depleted** to **Regrow** and a **Regrow Delay**.
2. Keep **Show Growth** on and assign the child holding the model to **Growth Target**.
3. Add `ResourceNodesSaver` to the scene so the regrow time survives a reload.

Do not put this tree in a spawner.
