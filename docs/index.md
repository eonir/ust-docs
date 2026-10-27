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
| **Combat** | Melee swings validated by the server, weapon damage, optional PvP |
| **Gathering** | Tools with tiers, resource nodes, area spawners that respawn objects at random places |

## Design principles

- **Data driven.** Items, vitals, categories and kits are ScriptableObjects. Most features need no code.
- **Modular.** Every module lives in its own assembly and can be removed or disabled.
- **Multiplayer first.** Every system works in host, client and dedicated server modes, and the server validates
  everything that matters for cheating (items, damage, access).

!!! tip "New here?"
    Start with the [Requirements](getting-started/requirements.md), then follow the
    [Quick Start](getting-started/quick-start.md) to play the demo with two players.
