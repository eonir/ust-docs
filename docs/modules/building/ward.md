# Ward

Part of [Building](index.md).

## Ward

Without a ward, anyone can take down any built piece and open any built chest, as in Valheim. A **ward** is a piece
that protects the area around it. Add `BuildWard` to a build piece; the demo **Ward** is a post with a glowing orb.

While the ward is on, inside its **Radius**, only its owner and the players they let in can:

- build, and take pieces down,
- open storages and move items in or out of any world container,
- use crafting stations, and take their output.

Loot bags stay open to everyone. Player inventories are never affected.

| Who | Interact with the ward |
|---|---|
| Owner, the player who built it | Turns it on or off |
| Another player, ward off | Joins the ward, or leaves it |
| Another player, ward on | "This area is protected" message |

So the owner turns the ward off, friends interact with it to join, and the owner turns it on again. A ward cannot be
built where its area would overlap the active ward of another player, and the preview says *Too close to another
ward*. Building inside a ward you are not part of shows *Protected by a ward*.

| Field | Default | Meaning |
|---|---|---|
| Radius | 15 | Size of the protected sphere |
| Active Visual | | Optional object shown while the ward is on |
| Turn On Prompt, Turn Off Prompt | Turn on ward, Turn off ward | Interaction text for the owner, or anyone while the ward is off |
| Join Prompt, Leave Prompt | Join ward, Leave ward | Interaction text for a player who is not, or is, part of the ward |
| Protected Prompt | Ward: turn it off to join | Shown to other players while the ward is on |

The owner and the joined players are remembered by their save id (`IPlayerDataSource.PlayerKey`), so the ward keeps
them across sessions. Without the Save module, the session client id is used and is lost on restart.

Your own systems can respect wards: `ProtectionZones.IsAllowed(clientId, position)` in UST.Core, and implement
`IProtectionZone` for other kinds of protected areas (a safe zone around a spawn, a claim flag...).
