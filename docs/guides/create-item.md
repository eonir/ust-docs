# Create an item

1. *Create > UST > Inventory > Item Definition*. Set its name, icon, max stack and weight.
2. Create the **model prefab**: only the mesh, no `NetworkObject`.
3. Create the **world prefab**: an empty root with `NetworkObject`, `WorldItem` (with the database) and a collider,
   and the model prefab as a child.
4. Add the world prefab to the **Network Prefabs** list and assign it to **World Prefab** on the item.
5. To show the item in the hand: **Add Trait > Visual > Held Visual** and assign the model prefab.
6. Right click the `ItemDatabase` > **Collect All Items In Project**.

!!! tip "Testing"
    Open the inventory with Tab: the **Give** panel lists every item of the database.
