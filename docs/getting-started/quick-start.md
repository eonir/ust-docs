# Quick Start

## Where to start

*Tools > UST > Setup Window* is the entry point, and opens by itself the first time, see [Setup Window](setup-window.md). It shows each module of the
project with what it has in the open scene and on the player prefab, and adds what is missing in one click. From there:

- **Create Full Survival Demo** turns the open scene into a playable demo of every module, see
  [Build the demo scene](../guides/demo-scene.md). Save the scene first, so the map can be baked.
- **Add Every Module To Scene** sets up your own scene without the demo content.
- **Setup Player Prefab** adds the components of every module to the player.
- **Validate Scene** lists what is still wrong, with a fix button.

## Play the demo alone

1. Open the demo scene.
2. Press **Play**.
3. Click **Host a game** in the main menu, then **Start**. Without the Lobby module, use **Host** in the session menu at the
   top of the screen.

## Play the demo with two players in the editor

1. Open *Window > Multiplayer > Multiplayer Play Mode* and enable **Player 2**.
2. Press **Play**. A second editor window opens for the virtual player.
3. Host in the main editor, then join from the Player 2 window: **Direct IP** with the default address, or **Join
   with a code** with the code the host shows.

## Default controls

| Action | Keyboard and mouse |
|---|---|
| Move | WASD (ZQSD on AZERTY, keys are physical) |
| Look | Mouse |
| Sprint | Shift |
| Jump | Space |
| Crouch | C |
| Interact / pick up | E |
| Attack / gather | Left mouse button |
| Aim (ranged weapons) | Right mouse button |
| Reload | R |
| Camera distance | Mouse wheel (rotates the piece while building) |
| Hotbar | 1 to 0. On a consumable (food, potion), the key uses it and the held item stays in hand. Armor is equipped with a right click in the inventory. |
| Inventory | Tab, Escape to close. The character stats show next to it; the **Stats** button hides them |
| Map | M, on every keyboard layout (d-pad right on a gamepad): left click places or removes a pin, right click pings |
| Ping | Middle mouse button, hold to place a marker |
| Chat | Enter |
| Push to talk | V |
| Session menu, options | Escape. Keys can be changed in *Options > Keys* |
| Build (hammer in hand) | Right click opens the build menu, see [Building > Controls](../modules/building/index.md#controls) |
| Move items | Drag and drop, right drag for half. Ctrl + click sends the stack to the other side, Shift + click picks up half. **Sort** and **Take all** buttons in the window. |
| Drop items | Drag out of the inventory window |
| Kill yourself (debug HUD, host with default health authority) | K |

!!! note
    Without the Lobby module, the session menu at the top of the screen is a debug tool drawn with IMGUI, meant for
    testing. The rest of the UI is uGUI, see [UI](../modules/ui/index.md). Gamepad controls are listed in
    [Player > Gamepad](../modules/player/input.md#gamepad).
