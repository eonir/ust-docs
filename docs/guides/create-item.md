# Create an item

1. Open *UST Tools > Item Database* and click **+ New Item**. The item is created and added to the database. Set
   its name, icon, max stack and weight on the right.
2. Create the **model prefab**: only the mesh, no `NetworkObject`.
3. Create the **world prefab**: an empty root with `NetworkObject`, `WorldItem` (with the database) and a collider,
   and the model prefab as a child.
4. Add the world prefab to the **Network Prefabs** list and assign it to **World Prefab** on the item.
5. To show the item in the hand: **Add Trait > Visual > Held Visual** and assign the model prefab.
6. To make it craftable: **Add Trait > Crafting > Recipe**, then set the ingredients, the output count and the
   station, see [Crafting](../modules/crafting.md#recipe-trait).
7. Check the warnings at the top of the item in the window: missing icon, no world prefab, weapon without Held
   Visual...

To make a variant, select an item and click **Duplicate**: the copy gets its own id and is added to the database.

!!! tip "Testing"
    Open the inventory with Tab: the **Give** panel lists every item of the database.
