# Equipment

The Equipment module lets players wear items: helmets, armor, boots, capes, rings... Worn items protect with armor and
resistances, change the speed, keep effects active, and can show a model on the character. The slots come from a
layout asset, so a game with four ring slots or a backpack slot only changes data.

The module is optional: add `UST_DISABLE_EQUIPMENT` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module).

## Setup

In the [Setup Window](../getting-started/setup-window.md), **Equipment > Add** does steps 1 and 2. By hand:

1. *Tools > UST > Setup Player Prefab*, or *Tools > UST > Advanced > Equipment > Add Equipment To Selected Player* on
   the player prefab. It adds:
    - an **Equipment** child with an `InventoryContainer`, using the database of the inventory;
    - `PlayerEquipment` on the root, with a default layout (Head, Chest, Legs, Hands, Feet, Back, 2 Rings) created in
      `Modules/Equipment/Data`;
    - the equipment container to the **Other Containers** of `PlayerDeathDrop`, so worn items drop on death.
2. *Tools > UST > Create Game UI* adds the equipment panel on the left of the inventory. For a UI built before, run
   *Tools > UST > Advanced > UI > Add Equipment Panel*. The panel goes just before the inventory panel in its horizontal layout group,
   so on its left, with the look of the container panel.

| Equipment Panel field | Default | Meaning |
|---|---|---|
| Placeholder Color | faded white | Tint of the slot icon in empty slots |
| Show Slot Names | on | Writes Head, Ring... in each slot |
| Slot Name Size, Slot Name Color | 13, faded white | |
3. Give items the **Equipment > Equippable** trait, see below.

## Slots and layout

An **Equipment Slot** (*Create > UST > Equipment > Equipment Slot*) is a kind of slot: Head, Chest, Ring... Its
**Display Name** is written in each slot of that kind, and its **Icon** shows, faded, while the slot is empty.

The **Equipment Layout** (*Create > UST > Equipment > Equipment Layout*) lists the slots of a player in display
order, each with a **Count**: Ring x4 gives four ring slots. The equipment container and the panel are built from it at
runtime, so the panel always matches the layout, whatever its size. Assign it on `PlayerEquipment`.

!!! warning "Changing a released layout"
    Saves store worn items by slot index. Add new slots at the end of the layout: inserting or reordering slots makes
    old saves load items into slots of another kind, where they stay until taken off.

## Equippable trait

| Field | Default | Meaning |
|---|---|---|
| Slot | | Kind of slot the item goes in |
| Armor | 0 | Flat protection, see below. Armor of every worn item adds up. |
| Resistances | | By damage type, while worn, see [Damage types](combat/index.md#damage-types) |
| Wear Per Hit | 1 | Durability lost each time the wearer takes damage, for items with durability. Damage from the world (cold, gas) does not wear. |
| Cold Insulation, Heat Insulation | 0 | Degrees of cold or heat kept away while worn, see [Temperature](temperature.md#countering-with-resistances) |
| Speed Multiplier | 1 | 0.9 for heavy armor. Worn items multiply together. |
| Effects | | Effects active while worn, from the Effects module: a ring of regeneration, warm clothes |
| Model | | Optional prefab shown on the character |
| Bone | Head | Humanoid bone the model is attached to |
| Position Offset, Rotation Offset, Scale | | Placement of the model on the bone |

Equippable items never stack. The Item Database warns about an equippable without a slot.

### Armor

Armor works like in Valheim: a hit weaker than twice the total armor loses the armor value, a stronger hit is reduced
less and less.

```
damage < 2 x armor  ->  damage x damage / (4 x armor)
otherwise           ->  damage - armor
```

With 10 armor: a hit of 10 deals 2.5, a hit of 20 deals 10, a hit of 40 deals 30. Damage from the world (source
**World**: cold, heat, gas) ignores armor: only resistances protect from it. Armor protects a lot against weak
enemies and little against strong ones. Then come the resistances of every worn item, multiplied together.

The order for a player: natural resistances of `PlayerDamageReceiver`, then equipment (armor, then resistances), then
any other `IDamageModifier`.

### Cold, fire, poison...

Use damage types: a **Cold** damage type, and furs with Cold 0.5. The [Temperature](temperature.md) module deals Cold
and Heat damage from the felt temperature, and adds insulation to the clothes.

## Using the equipment

- Drag an item between the inventory and its slot. Each slot only accepts its kind: the server refuses the rest.
- Right click an equippable item in the inventory to wear it: in an empty slot of its kind, or in place of the worn one.
- Right click, or Ctrl + click, a worn item to put it back in the inventory.

The armor, the resistances and the speed of the worn items show in the [character stats panel](ui/windows.md#character-stats-panel).

Worn items are saved with the player, synchronized to everyone, and dropped in the loot bag on death. Their weight is
not counted while worn.

## Inspecting

Armor has no field on the player: it is the sum of the worn items, so it only exists in Play Mode. Select the player
in Play Mode: the `PlayerEquipment` inspector lists what each slot wears, the total armor, and the total resistances
by damage type, natural ones of `PlayerDamageReceiver` included. The [character stats panel](ui/windows.md#character-stats-panel)
shows the same in game.

## Scripting

```csharp
PlayerEquipment equipment = player.GetComponent<PlayerEquipment>();
float armor = equipment.Armor;
ItemDefinition helmet = equipment.GetItem(0);
equipment.Changed += RefreshCharacterSheet;
```

## Not in this version

- Skinned armor sharing the skeleton of the character, and hiding body parts under it.
- Set bonuses.
