# Voice

The Voice module adds proximity voice chat with **Vivox**: players hear each other louder when close, and not at all
beyond the audible distance. Push to talk by default, or open mic. A list on screen shows who is speaking.

Vivox is part of Unity Gaming Services. It is free up to 5,000 players connected at once, and the bill, if any, goes to
the account of the game. It works in online sessions and in direct IP, as long as the project is linked to Unity Cloud.

The module is optional: add `UST_DISABLE_VOICE` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module). It is also left out when the **Vivox** package
(`com.unity.services.vivox`) is not installed.

## Setup

1. Install **Vivox** from the Package Manager (Unity Registry).
2. Link the project to Unity Cloud (*Edit > Project Settings > Services*), then open the
   [Unity Dashboard](https://cloud.unity.com), select the project, and turn on **Vivox Voice and Text Chat**.
3. In the [Setup Window](../getting-started/setup-window.md): **Voice > Add**. By hand, run *Tools > UST > Advanced > Voice > Add Voice To Scene*. It adds a `VoiceService` to the **Network Session** object and the
   list of speakers at the top left of the **UST UI** canvas. *Create Game UI* adds both too.

Each session gets its own voice channel, picked by the server. Players join it as soon as their character spawns.

## Voice Service

| Field | Default | Meaning |
|---|---|---|
| Audible Distance | 32 | Players further than this, in metres, are not heard |
| Conversational Distance | 2 | Players closer than this are heard at full volume |
| Fade Intensity, Fade Model | 1, Inverse By Distance | How the voice fades between the two distances |
| Positional | on | Off: everyone hears everyone at full volume, like a party call |
| Position Interval | 0.2 | Seconds between two position updates |
| Transmission | Push To Talk | Push To Talk, Open Mic (Vivox only sends when it hears a voice) or Muted |
| Push To Talk Key | V | Ignored while typing in the chat |

The voice comes from the head of the character and is heard from the camera.

## Voice Speakers View

Lists the players speaking, and shows *Mic on* while the local microphone sends.

| Field | Default | Meaning |
|---|---|---|
| Speakers Label | | Text listing the players speaking, one per line |
| Transmitting Indicator | | Object shown while the local microphone sends |
| Line Format | a green dot, then {0} | How a line looks. {0}: the name of the player. |

## Scripting

```csharp
VoiceService voice = VoiceService.Current;
voice.Transmission = VoiceTransmission.OpenMic;   // from your settings menu
voice.SetMuted(clientId, true);                    // mute a player, for the local player only
voice.SetOutputVolume(-10);                        // -50 to 50, 0 normal
voice.SetInputVolume(5);
bool talking = voice.IsSpeaking(clientId);
VoiceService.SpeakingChanged += (clientId, speaking) => ShowIcon(clientId, speaking);
VoiceService.StatusChanged += message => Debug.Log(message);
```

## Testing

On one machine, each copy of the game needs its own profile: Multiplayer Play Mode handles it, builds take
`-ustProfile name`, see [Testing on one machine](lobby.md#testing-on-one-machine). Two copies on the same machine also
share the microphone and the speakers: test with two machines or a headset.

## Not in this version

- Text chat over Vivox: the [Chat](chat.md) module covers it without a service.
- Choice of the microphone in game. `VivoxService.Instance.AvailableInputDevices` lists them for your settings menu.
- Radio or team channels.
