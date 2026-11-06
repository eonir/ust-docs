# Sleep

The Sleep module lets players skip the night in a bed. When enough players are in bed, every screen fades to black,
the clock jumps to sunrise and the world catches up with the skipped time: vitals go down (slower than awake), crafts
finish, timed effects wear off, resources respawn, the weather moves on and the creatures of the night leave.

It needs the [day and night cycle](weather.md). The module is optional: add `UST_DISABLE_SLEEP` to the Scripting
Define Symbols to remove it, see [Modules](../architecture/modules.md#disabling-an-optional-module). It is also
removed with `UST_DISABLE_WEATHER`.

## Setup

1. Add the day and night cycle if the scene has none: *UST Tools > Advanced > Weather > Add Day Night Cycle To Scene*.
2. Run *UST Tools > Advanced > Sleep > Add Sleep To Scene*. It adds a `SleepDirector` to the Day Night Cycle object.
3. Run *right click in the Hierarchy > UST > Bed* for a test bed, or add a `Bed` to your own bed model (see [Your own bed](#your-own-bed)).

Nothing goes on the player prefab: the first bed a player uses adds what it needs at runtime.

## Sleeping

- Look at a bed and press **Interact** to lie down. The prompt says *Sleep*, *Get up* or *Occupied*.
- **Interact**, **Jump** or a move gets up.
- By default, players can only go to bed at night.
- Every player in bed counts. When they reach the required share of the living players (all of them by default) for
  **Fall Asleep Seconds**, the night is skipped. Dead players and players still joining do not count.
- Everyone sees *Sleeping: 1/2* when the count changes.
- After the skip, the sleepers get up where they stood, or at the **Get Up Point** of the bed.

A player saved while in bed is saved standing next to it, not inside it.

## Sleep Director

| Field | Default | Meaning |
|---|---|---|
| Required Share | 1 | Share of the living players that must be in bed. 1: everyone, 0.5: half of them, rounded up. |
| Allow During Day | off | Players may go to bed during the day too. The skip still goes to the next sunrise, so a whole day and night pass. |
| Fall Asleep Seconds | 2 | Time the players must stay in bed before the skip starts |
| Fade Seconds | 1.5 | Length of the fade to black and of the fade back |
| Black Seconds | 1 | Time the screen stays black. The clock jumps in the middle. |
| Sleeping Parameter | Sleeping | Animator bool set on the sleeping character, on every peer. Missing in the controller: the character keeps its idle pose. |
| Disable While Sleeping | PlayerItemUser, PlayerBuilder, PlayerRangedWeapon | Type names of the player components turned off on the owner while in bed. Movement, interaction and camera zoom are handled already. Add your own. |
| Not Tired Message, Occupied Message, Count Message | | Texts shown to the players. Count Message takes `{0}` for the players in bed and `{1}` for the players needed. |

If someone gets up or leaves during the fade to black, the skip is cancelled and the screen fades back.

## Bed

| Field | Default | Meaning |
|---|---|---|
| Sleep Point | the bed, 0.5 m above | Where the feet of the character go in bed, facing the blue axis. Measured from its CharacterController, whatever the pivot of the model. |
| Get Up Point | | Where the feet of the character go when it gets up: put it on the ground. Empty: where it stood before lying down. |
| Use Range | 3.5 | Farthest a player can be from the bed to lie down, checked on the server |
| Sleep Prompt, Get Up Prompt, Occupied Prompt | | Interaction texts |

The bed needs a `NetworkObject`. One player per bed.

### Your own bed

1. Add a `NetworkObject` and a `Bed` to the root of the model. It needs a collider so that the interaction ray finds it.
2. Add an empty child on the mattress, where the feet of the character go, assign it as **Sleep Point**, and turn it
   so its blue axis points along the bed.
3. Optionally, add an empty child on the ground next to the bed as **Get Up Point**.

To let players build it, make it a [build piece](building.md): a `BuildPiece` and a `Bed` on the same prefab work
together.

### Lying animation

Without an animation, the character lies in its idle pose. To show it asleep, add a `Sleeping` bool to the player
Animator Controller and a transition to a lying animation while it is true. Place the **Sleep Point** where the feet of
the character are in that animation.

## Respawn in bed

Lying down in a bed makes it the respawn point of the player (*Respawn point set.*), until it lies in another one. The
player then respawns at its **Get Up Point**, or beside the bed. When the bed is destroyed, the spawn points of the
scene are used again.

To keep the respawn bed in the save, the player prefab needs a `Sleeper`: select it and run *UST Tools > Advanced > Sleep > Add Bed Respawn To Selected Player*. Without it, beds still set the respawn point, for the session only.

| Sleep Director field | Default | Meaning |
|---|---|---|
| Bed Sets Respawn | on | Off: beds are only for sleeping |
| Respawn Set Message | Respawn point set. | Shown to the player. Empty: no message. |

```csharp
Sleeper sleeper = player.GetComponent<Sleeper>();
sleeper.ServerSetRespawnBed(bed);
sleeper.ServerClearRespawnBed();
```

## What happens during the skip

The skipped time is the real time the night would have lasted: with 8 **Night Minutes**, sleeping at sunset skips
8 minutes. Every system registered in `TimeSkip` advances by it, at once.

| System | During the skip |
|---|---|
| Vitals | Decay at **Sleep Decay Multiplier** (0.5: half as fast as awake), regeneration at **Sleep Regen Multiplier** (1). Sleeping never takes a vital under **Sleep Floor** (15 %), and empty vitals drain nothing: nobody dies in their sleep. Gradual effects, such as the healing of a food, keep ticking. See [Vitals](vitals.md#sleep). |
| Effects | Timed effects run down and end if the night covers them. Their modifiers do not tick: a poison wears off instead of dealing its damage. |
| Crafting | Queues work through the skipped time: crafts finish as if it had passed. A long queue is spread over a few frames. A craft waiting for room in a full output stops using the time. |
| Gathering | Respawn delays of object spawners run down: what was due at night is back in the morning. |
| Weather | Weathers run out, and the ones that ended are picked again. |
| AI | **Night Only** spawn directors remove their living enemies, see [Spawn director](ai.md#spawn-director). |

Temperature, stamina and the rest simply carry on in the morning.

## Scripting

Skip the night from code, for a debug command or another way of resting:

```csharp
DayNightCycle cycle = FindAnyObjectByType<DayNightCycle>();
cycle.ServerSkipToMorning();          // the world catches up
cycle.ServerSkipToMorning(false);     // only the clock moves
float seconds = cycle.RealSecondsBetween(cycle.Hour, 6f);
```

Make your own system catch up with the skipped time by implementing `ITimeSkipListener` (UST.Core) and registering
it on the server:

```csharp
public class Composter : NetworkBehaviour, ITimeSkipListener
{
    private float progress;

    public override void OnNetworkSpawn()
    {
        if (IsServer) TimeSkip.Register(this);
    }

    public override void OnNetworkDespawn() => TimeSkip.Unregister(this);

    public void OnTimeSkipped(float seconds) => progress += seconds;
}
```

`seconds` is real time: advance your timers by it as if it had passed. A system that does not implement the interface
stays as it was during the skip, so remember it for every new system that runs on time.

| Member | Meaning |
|---|---|
| `TimeSkip.Skipped` | Server event raised after every listener, with the skipped seconds |
| `SleepDirector.Current` | The director of the scene |
| `SleepDirector.Fading` | Static event raised on every peer when the screen fades to black (true) or back (false) |
| `SleepDirector.IsSkipping` | True on every peer from the fade to black until the fade back |
| `Bed.Occupant`, `Bed.IsFree` | Client id of the player in the bed, synchronized |
| `Sleeper.IsSleeping` | On a player that has used a bed: true while it is in one |

## Not in this version

- No check for enemies near the bed.
