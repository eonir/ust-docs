# Doc à compléter (prochaine update)

Options visibles dans l'inspector mais absentes de la doc. Hors doc publiée (pas dans docs/).

## Store / description
- Gathering : la ligne "Tools with tiers, resource nodes, area spawners that respawn objects at random places" doit dire que l'utilisateur choisit entre récolte bloquée par le tier (**Blocked**) et rendement réduit (**Reduced Yield**), et la doc doit lister les options utilisateur.

## Player
- `PlayerFootIK` : aucun réglage documenté (Foot Bone Height, Probe Above/Below, Reach Down, Max Hip Drop, Plant Height, Hip Response, Dead Zone, Max Foot Lift, Weight Response, Align To Slope). Mérite sa section dans player.md.
- `PlayerController` : champ **Player Camera** non documenté. Code mort : `lookSpeed` / `lookXLimit` commentés, à supprimer.

## Inventory / Combat
- `PlayerHeldItem` : **Hold Type Parameter** (paramètre Animator).
- `PlayerRangedWeapon` : **Aiming Parameter** (paramètre Animator).
- `PlayerItemRig` : **Aim Bones** (rempli par Setup Item Rig, à mentionner).
- `HeldVisualTrait` : **Aim Pose Rotation**, **Rest Pose Rotation** (réglés par Held Item Tuning, citer avec les positions).
- `InventoryDebugHUD` : **Drop Key**.

## Crafting
- `CraftingWindow` : textes réglables **Hand Title**, **Empty Queue Text**, **Output Full Text**.
- `CraftingRecipeRow` : **Unavailable Alpha**.

## UI
- `ItemFeedView` : **Default Icon Size**, **Default Font Size**.
- `VitalsBarsView` : **Bar Sprite**, **Default Font Size**.
- `ScreenMessageView` : **Shake Frequency**.

## Net / Debug
- `NetworkSessionMenu` : **Address**, **Listen Address**, **Width**.
- `VitalsDebugHUD` : **Button Step**.
