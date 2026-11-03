# UST Documentation

UST is a multiplayer survival template for Unity 6. It gives you the systems every survival game needs, already
networked with Netcode for GameObjects, so you can focus on your own game.

## What is included

| Module | What it does |
|---|---|
| **Player** | Third person controller, crouch, jump, stamina, interaction |
| **Vitals** | Health, hunger, thirst, stamina or any stat you create, with decay, regeneration and gradual effects |
| **Life** | Death, respawn, spawn points |
| **Inventory** | Slot inventory, hotbar, optional weight limit, items on the ground, loot bag on death, starting kits |
| **Save** | World and player saves in JSON, autosave, pluggable player identity (Steam, Unity Authentication...) |
| **Combat** | Melee swings and ranged weapons validated by the server, ammo, aiming, optional PvP |
| **Effects** | Buffs and debuffs, effect zones, consumables (food, drinks, medicine) |
| **Crafting** | Recipes on items, crafting stations and hand crafting, queued crafts with timers |
| **Weather** | Day and night cycle with its own day and night lengths, colors and sky |
| **Temperature** | Felt temperature from zones, fires, roofs and clothes, with cold and heat stages |
| **Equipment** | Flexible slots from a layout, armor and resistances, effects and models while worn |
| **AI** | Melee enemies that chase, attack and break walls, spawned around players |
| **Building** | Build tools, snapping pieces with a live preview, build menu, built chests and workbenches, saved |
| **Gathering** | Tools with tiers, resource nodes, area spawners that respawn objects at random places |

## Design principles

- **Data driven.** Items, vitals, effects, categories and kits are ScriptableObjects. Most features need no code.
- **Modular.** Every module lives in its own assembly and can be removed or disabled.
- **Multiplayer first.** Every system works in host, client and dedicated server modes, and the server validates
  everything that matters for cheating (items, damage, access).

!!! tip "New here?"
    Start with the [Requirements](getting-started/requirements.md), then follow the
    [Quick Start](getting-started/quick-start.md) to play the demo with two players.
