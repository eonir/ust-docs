# Chat

The Chat module adds a text chat to every session, online or direct IP. It runs on Netcode for GameObjects only: no
service, no account, nothing to pay. The server checks every message (length, rate) before sending it to everyone, and
announces the players who join and leave.

The module is optional: add `UST_DISABLE_CHAT` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module).

## Setup

Run *UST Tools > Advanced > Chat > Add Chat To Scene*. It adds a `ChatService` to the **Network Session** object (created with
its `NetworkObject` and `PlayerNames` if missing), and the chat box at the bottom left of the **UST UI** canvas.
*Create Game UI* adds both too.

## Using it

- **Enter** opens the chat, **Enter** sends, **Escape** cancels. Clicking elsewhere closes it.
- While typing, the character does not move and no shortcut fires: number keys, inventory, building, push to talk...
- Closed, each message stays **Visible Seconds** on screen, then fades. Opening the chat shows the last ones again.

Names come from [Player names](lobby.md#player-names): the name chosen in the lobby.

## Chat Service

| Field | Default | Meaning |
|---|---|---|
| Max Length | 160 | Longest message, in characters |
| Max Messages Per Window, Rate Window | 5, 5 | A player sending more than 5 messages in 5 seconds sees *You are sending messages too fast* and the extra messages are dropped |
| History Size | 50 | Messages kept on each peer |
| Announce Players | on | *X joined the game*, *X left the game* |
| Joined Format, Left Format, Too Fast Message | | Texts. `{0}` is the name. |

Messages are cleaned on the server: control characters and the tags that would break the display are removed, and the
text is shown without rich text, so nobody can write in red or as huge text.

## Chat View

| Field | Default | Meaning |
|---|---|---|
| Open Key | Enter | Numpad Enter works too |
| Command Key | None | Opens the chat with `/` typed, for commands of your own |
| Max Lines | 12 | Lines on screen |
| Visible Seconds, Fade Seconds | 8, 1.5 | How long a message stays when the chat is closed |
| Name Color, Local Name Color, System Color | | Colors of the names and of the messages of the game |

## Scripting

```csharp
ChatService.Current.Send("Hello");                        // from the local player
ChatService.Current.ServerPostSystem("A storm is coming"); // server: a message of the game
ChatService.Received += message =>
{
    if (message.IsSystem) return;
    Debug.Log(message.senderName + ": " + message.text);
};
```

To change how a line looks (timestamps, team colors, icons), override `Format` in a class deriving from `ChatView`.

Your own text fields should use the same focus as the chat, so the game ignores the keys typed in them:

```csharp
UIInputFocus.PushTyping();   // when the field gets the focus
UIInputFocus.PopTyping();    // when it loses it
```

## Not in this version

- Private messages, channels and commands. `Command Key` and `ChatService.Received` are the starting points.
- Profanity filter.
