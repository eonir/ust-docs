# Multiplayer

UST uses **Netcode for GameObjects** in a host / client topology. A dedicated server works the same way, without a
local player.

## Who decides what

| Data | Authority | Why |
|---|---|---|
| Player movement | Owner | Instant response, no input latency |
| Stamina | Owner | Must react instantly to sprint and jump |
| Health, hunger, thirst | Server | Gameplay values calculated by the server |
| Inventory, items on the ground, loot bags | Server | Prevents item duplication |
| Hits (gathering, damage) | Detected by the owner, validated by the server | Responsive aiming with server-side hit checks |
| Death, respawn | Server | |
| Saves | Server | The server stores world and player data |

Clients send gameplay requests, such as moving an item or using a consumable; the receiving system checks the
applicable ownership, range, weight or access rules. Server-authoritative vitals can only be changed by server code;
clients cannot submit arbitrary health, hunger or thirst deltas.

UST primarily targets cooperative games. These checks support consistent shared gameplay; they do not constitute
a complete anti-cheat system. Review the validation of each action and platform authentication before adapting the
template for untrusted competitive play.

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

!!! tip "Validate Network Prefabs"
    *Tools > UST > Validation > Validate Network Prefabs* follows every reference of the open scenes, the player prefab and the
    items, lists the network prefabs missing from the NetworkManager and adds them in one click. Run it before testing
    with a client. [Validate Scene](../getting-started/validate-scene.md) runs the same check with all the others.

## Testing

Use *Multiplayer Play Mode* to run up to four players in the editor. Its network simulation lets you add latency and
packet loss: test with about 150 ms and 2 % loss before shipping.
