# Doc à compléter (prochaine update)

Options visibles dans l'inspector mais absentes de la doc. Hors doc publiée (pas dans docs/).

## Store / description
- Gathering : la ligne "Tools with tiers, resource nodes, area spawners that respawn objects at random places" doit dire que l'utilisateur choisit entre récolte bloquée par le tier (**Blocked**) et rendement réduit (**Reduced Yield**), et la doc doit lister les options utilisateur.

## Code (pas de la doc)
- `PlayerController` : `lookSpeed` / `lookXLimit` commentés (code mort), à supprimer.

## Fait (2026-11-03)
Toutes les options d'inspector listées ici sont documentées : Foot IK, Player Camera, Input Reader et ses membres, états du controller, Held Visual poses, PlayerHeldItem, PlayerItemRig, LootBag, InventoryDebugHUD, PlayerRangedWeapon, AimCameraZoom, EnemyAI paramètres Animator, BuildWard prompts, Build menu / HUD / piece button, Crafting window / rows, Stats panel refs, vital bars et item feed par défaut, screen messages, ItemSlotView, NetworkSessionMenu, VitalsDebugHUD, ObjectSpawner zone, nouveaux inspectors Weather.
Restent volontairement hors doc : champs internes de sauvegarde (WorldItemsSaver, ResourceNodesSaver, PlayerVitals...), UILayoutPreset (données de l'outil).

## Captures

- Validate Scene : ajouter une capture de la fenêtre dans `docs/getting-started/validate-scene.md`, section *The window* (créer `docs/images/`).
