# Lobby

The Lobby module is the main menu of a multiplayer game: choose a name, host a game, find one in the list, join one
with its code, or connect by IP address. While joining, a loading screen waits until the player is in the world. In
game, **Escape** opens the session menu: the code to share, the players, Leave and Quit.

Online sessions use **Unity Gaming Services**: Lobby to list and find sessions, Relay to connect players without
opening ports, Authentication to sign in anonymously. **Direct IP** needs none of them, for the local network and tests.

The module is optional: add `UST_DISABLE_LOBBY` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module). It is also left out when the **Multiplayer
Services** package (`com.unity.services.multiplayer`) is not installed.

## Setup

1. Run *Tools > UST > Advanced > Lobby > Add Lobby To Scene*. It adds:
    - the **UST Lobby UI** canvas, with `LobbyMenu` and `SessionConnector`;
    - the **Network Session** object (`NetworkObject` and `PlayerNames`), shared with the [Chat](chat.md) and the
      [Voice](voice.md);
    - an EventSystem if the scene has none.

    It also turns off the debug `NetworkSessionMenu`, which would draw over the lobby.
2. For online sessions, link the project to Unity Cloud: *Edit > Project Settings > Services*, pick or create a
   project. Lobby and Relay are free up to a generous use, and the bill, if any, goes to the account of the game.
   Without it, online buttons show a message and Direct IP still works.

## Playing

| Panel | What it does |
|---|---|
| Main | Name of the player (kept between sessions), then Host, Find, Join with a code, Direct IP, Quit |
| Host a game | Session name, max players, optional password (8 to 64 characters), world (the [save slot](save.md)), private |
| Find a game | Public sessions of the same game version with room left, newest first. Join, with the password if the session has one. |
| Join with a code | The 6-character code the host shares |
| Direct IP | Host on a port, or join an address and port. Players outside the local network need the port open on the router of the host. |
| Loading | Shown until the player spawns. Cancel goes back. |
| Session (in game) | Session name, code and Copy, the players, Resume, Leave, Quit |

A **private** session is not listed: players join it with its code only. When the host leaves, the session closes and
the others go back to the main menu with a message. Errors (wrong password, session full, no network) appear under
the window.

## Session Connector

| Field | Default | Meaning |
|---|---|---|
| Default Max Players | 8 | Shown in the Host panel |
| Relay Region | | Region such as `europe-west4`. Empty: the closest one. |
| Match Game Version | on | Only list sessions with the same `Application.version`, so players never join an incompatible build |
| Listed Sessions | 30 | Most sessions shown by a search |
| Default Address, Default Port | 127.0.0.1, 7777 | Shown in the Direct IP panel |
| Listen Address | 0.0.0.0 | Address the direct host listens on. 0.0.0.0 accepts the local network. |

## Lobby Menu

Every panel, field and button is a reference: restyle, move or replace them freely. **Menu Key** (Escape) opens the
session menu in game when no other window is open. The texts of the loading and error messages are fields too.

## Player names

`PlayerNames` (UST.Net) shares the name of each player with everyone. The lobby sets the local name before joining;
without the lobby, the name is `Player` and 4 digits, picked once per machine.

```csharp
PlayerNames.LocalName = "Eonir";                 // kept in player.json, sent at once if connected
string name = PlayerNames.NameOf(clientId);      // "Player 3" while the name has not arrived
PlayerNames.Changed += () => RefreshMyList();
```

Names are cleaned on the server: 20 characters at most, no rich text.

## Testing on one machine

Each copy of the game needs its own Unity Gaming Services profile, or they sign in as the same player and cannot join
each other.

- **Multiplayer Play Mode**: each virtual player gets its own profile automatically.
- **Builds**: start each one with `-ustProfile name`, for example `Game.exe -ustProfile p2`.

With the [Save module](save.md), give each build its own player id too: `-ustPlayerId name`.

## Scripting

Use `SessionConnector` from your own menu:

```csharp
SessionConnector connector = SessionConnector.Current;
try
{
    await connector.HostAsync("My world", 4, isPrivate: false, password: null, saveSlot: "World1");
    string code = connector.Session.Code;                 // share it
    await connector.JoinByCodeAsync("ABC123");
    IList<ISessionInfo> sessions = await connector.FindSessionsAsync();
    await connector.JoinByIdAsync(sessions[0].Id);
    connector.HostDirect(7777, "World1");
    connector.JoinDirect("192.168.1.20", 7777);
    await connector.LeaveAsync();
}
catch (SessionFailure failure)
{
    ShowError(failure.Message);                            // already written for the player
}
```

`SessionChanged` is raised when the session is created, joined, left, or its players change.

## Not in this version

- Steam lobbies and Steam networking. `SessionConnector` and `IPlayerIdentityProvider` are the places to plug them.
- Friends list and invites.
- Host migration: when the host leaves, the session ends.
