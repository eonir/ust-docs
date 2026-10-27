# FAQ

**The player falls through the ground when it spawns.**
Set the `CharacterController` Center Y to half its height, and keep **Spawn Height Offset** on `PlayerLife` above 0.

**Nothing happens when I press Play.**
The player is spawned by the NetworkManager: click **Host** in the session menu.

**The camera does not rotate.**
The `CinemachineCamera` needs a `CinemachineInputAxisController` with the Look action assigned.

**The item in the hand is huge.**
Your rig has a scale on its bones, usually x100 from a centimetre export. `PlayerHeldItem` compensates it, but
exporting the character with *Apply Scalings: FBX Units Scale* fixes it at the source.

**"Held visual ... must not contain a NetworkObject".**
You assigned the world prefab as the held visual. Assign the model prefab instead.

**I changed a vital or an item in Play Mode and the change stayed.**
ScriptableObjects are assets: Unity does not revert them when Play Mode stops.

**Two players in Multiplayer Play Mode get the same save.**
They should not: each virtual player has its own id. If you replaced the identity provider, make sure it returns a
different id per player.

**A new player does not get the starting kit.**
The kit is only given to players without a save. Delete the player file from the save folder to test it again.

**A client logs "NetworkPrefab could not be found".**
A prefab spawned at runtime is not in the NetworkManager prefab list. Run *Tools > UST > Validate Network Prefabs*,
then restart the Multiplayer Play Mode players.

**My swing does nothing.**
The item needs a **Melee Swing** trait, `PlayerItemUser` needs the Attack action, and the target needs an
`IItemHitReceiver` on its network object (`ResourceNode`, `PlayerDamageReceiver`...).

**I cannot hurt other players.**
PvP is off by default: enable **Allow Player Damage** on `PlayerDamageReceiver`.

**A tree gives nothing.**
Check the tool type and the tier: `ResourceNode.LocalHitResult` tells the player *WrongTool* or *TierTooLow*.

**A tree placed in the scene comes back after a reload.**
Add `ResourceNodesSaver` to the scene.

**A spawner logs a warning about a node that regrows in place.**
Nodes created by a spawner should use **Despawn**: the spawner replaces them elsewhere. Use **Regrow** for nodes placed
by hand.

**The item feed shows nothing.**
It lists items added by `PlayerInventory.ServerAdd`. Starting kits and loaded saves are silent on purpose.
