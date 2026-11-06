# Damage, repair and fuel

Part of [Vehicle](index.md).

## Damage, crashes and wrecks

- **Crashes**: an impact faster than the **Crash Safe Speed** damages the vehicle. Detected where the physics run,
  applied by the server.
- **Running over**: hitting anything damageable (an enemy, a player) at more than the **Ramming Min Speed** deals
  **Ramming Damage Per Kmh** times the speed, at most once every half second per target. On the **Rammed Layers** the
  vehicle drives through its victims; on other layers they stop it like a wall.

!!! warning "Rammed Layers and the ground"
    Never put the layer of the ground in **Rammed Layers**: the vehicle would fall through it. With a player or enemies
    on the Default layer, move them to a layer of their own first, or leave the field empty. The damage comes from the driver, with the source **Player**: kills
  are credited to them, and player victims follow the PvP option of their `PlayerDamageReceiver`.
- **Weapons and enemies**: through `IItemHitReceiver` and `IDamageable`, reduced by the resistances of the definition
  and the `IDamageModifier` components of the vehicle. The vehicle implements `IBreakable`, so enemies only swing at
  it when it can take their damage.
- **Wrecked** at 0 health: everyone gets out, it cannot be entered or driven, and its prompt says so. Any repair makes
  it drivable again.

## Repair

Add a **Repair Tool** trait (*Vehicle/Repair Tool*) to an item that also has a **Melee Swing** trait, then swing it at
the vehicle.

| Field | Default | Meaning |
|---|---|---|
| Repair Amount | 50 | Health per hit |
| Repair Percent | 0 | Extra health per hit, in percent of the max health |
| Cost | | Items taken on each hit, for example scrap metal. Empty: free |
| Durability Loss Per Hit | 1 | When the item has durability |

The player gets a message after each hit: the health reached, "needs no repair", or the missing cost. The Wrench of
the test scene repairs 50 per hit for free.

## Fuel

Add a **Fuel** trait (*Vehicle/Fuel*) to an item: a jerrycan, a fuel can.

| Field | Default | Meaning |
|---|---|---|
| Litres | 10 | Fuel given by one item |
| Empty Item | | Given back once used, for example an empty jerrycan. Empty: used up |

Interact with the **Fuel Cap** of the vehicle (left rear on the default vehicles): one fuel item is taken from the
hotbar or the inventory. The server burns fuel while someone drives, from **Idle Consumption** standing still to
**Full Consumption** at top speed. Out of fuel, the engine stops pushing; the vehicle still rolls, steers and brakes.
A definition with a **Fuel Capacity** of 0 never needs fuel, and its fuel cap shows no prompt.

## Trunk

The *Trunk* child is a regular storage, shared by every player, opened with **Interact** by aiming at the back of the
vehicle. Its **Access Range** is 4 m on the default vehicles. A trunk in a ward follows the ward, like any storage.
