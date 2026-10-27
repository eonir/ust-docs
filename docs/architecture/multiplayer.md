# Multiplayer

UST uses **Netcode for GameObjects** in a host / client topology. A dedicated server works the same way, without a
local player.

## Who decides what

| Data | Authority | Why |
|---|---|---|
| Player movement | Owner | Instant response, no input latency |
| Stamina | Owner | Must react instantly to sprint and jump |
| Health, hunger, thirst | Server | Gameplay critical, cannot be cheated |
| Inventory, items on the ground, loot bags | Server | Prevents item duplication |
| Death, respawn | Server | |
| Saves | Server | Players cannot edit their own save |

Clients never change server data directly: they send a request, and the server validates it (distance, ownership,
weight, access rights) before applying it.

!!! note "Owner authoritative movement"
    A modified client could move faster than intended. This is the usual trade off for cooperative games. For a
    competitive game you would need server authoritative movement with client prediction.

## Network prefabs

Every prefab with a `NetworkObject` on its root must be in the **Network Prefabs** list of the NetworkManager:

- the player,
- the world prefab of every item,
- the loot bag.

Visual only prefabs, such as the model shown in the hand, must **not** have a `NetworkObject` and must not be in the
list.

## Testing

Use *Multiplayer Play Mode* to run up to four players in the editor. Its network simulation lets you add latency and
packet loss: test with about 150 ms and 2 % loss before shipping.
