# Settings

The Settings module adds the options menu: what the HUD shows, the controls, the audio and the graphics. Every option
applies at once and is kept on the machine in `settings.json`. The lobby opens it from the main menu and from the session
menu in game.

The module is optional: add `UST_DISABLE_SETTINGS` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module). Without it, the saved settings and their defaults
still apply where they are read (camera sensitivity, nameplates, chat, voice), there is just no menu.

## Setup

The [Setup Window](../getting-started/setup-window.md) adds it: **Settings > Add**. By hand, run
*Tools > UST > Advanced > Settings > Add Options Menu To Scene*. It adds the **UST Options UI** canvas with the
`OptionsMenu` and the `SettingsApplier`, with the Keys tab. Rebuild the lobby (*Add Lobby To Scene* after deleting
**UST Lobby UI**) to get its **Options** buttons; they hide themselves when the scene has no options menu.

Without the lobby, set **Open Key** on the options menu, or call `OptionsMenu.Open()` from your own menu.

## Options

| Tab | Option | Default |
|---|---|---|
| Display | Names of the players, health of the players, names visible up to | on, on, 40 m |
| Display | Chat messages on screen ([Chat](chat.md)) | on |
| Display | Pings, placed markers, markers on far players, places of the world, markers on the edges, markers visible up to ([Markers](markers/index.md)) | on, 500 m |
| Display | Minimap, minimap zoom ([Map and minimap](markers/map.md#minimap)) | on, 150 m |
| Controls | Mouse sensitivity, gamepad sensitivity | 1x |
| Controls | Invert vertical look | off |
| Keys | One line per action of the Player map, with a keyboard slot and a gamepad slot | `UST_InputActions` |
| Audio | Master volume, music, effects, ambience | 100% |
| Audio | Voice chat (push to talk, open mic, muted), voices of the others, microphone ([Voice](voice.md)) | push to talk, 100% |
| Graphics | Quality, resolution, display mode, VSync, frame rate limit (greyed out with VSync) | the current ones |
| Graphics | Render scale, upscaler (Automatic, Bilinear, Nearest, FSR 1.0, STP), FSR sharpness (greyed out without FSR) | from the URP asset |
| Graphics | Field of view | the one of the camera |

**Reset this tab** goes back to the defaults. Escape, B on a gamepad or Close closes the menu. Options of a module
that is not in the project do not show.

### Graphics

Render scale, upscaler and sharpness are set on the URP asset of the current quality level. In the editor, the asset
gets its values back when Play Mode stops, so testing the options never changes the project. Below 100% render scale,
FSR 1.0 upscales the image with sharpening, STP gives the best image on capable hardware.

### Audio

The master volume is the volume of the `AudioListener`. The other volumes apply to the `AudioSource`s that carry an
`AudioCategorySource` (Effects, Music, Ambience, Voice): add it to your sources and set their volume as usual, it is the
volume at 100%. The weather sounds use the Ambience volume.

With an `AudioMixer`, assign it on the `SettingsApplier` with the names of its exposed parameters: the volumes then set
those parameters, in decibels, instead.

### Field of view

The aim zoom of ranged weapons (`AimCameraZoom`) zooms from the field of view of the options and comes back to it.

## Options Menu

`OptionsMenu` builds its lines from the options registered in `OptionsCatalog`, with one copy of a template line per
option. *Add Options Menu To Scene* builds it and fills every reference.

| Field | Default | Meaning |
|---|---|---|
| Root | | Object shown and hidden |
| Tab Bar, Tab Template | | Parent of the tab buttons, and the button copied for each tab |
| Content | | Parent of the option lines, in a scroll view |
| Toggle Template, Slider Template, Choice Template | | `OptionRow` copied for each kind of option |
| Binding Template | | `OptionRow` of the Keys tab, with a keyboard and a gamepad button. Empty: no Keys tab. |
| Input Actions | | Actions listed in the Keys tab. Empty: the asset of the Close Action. |
| Rebind Maps | Player | Action maps listed in the Keys tab |
| Rebind Excluded | Look | Actions not listed in the Keys tab |
| Waiting Text | Press a key... | Shown on a key slot while waiting for the new key |
| Changed Binding Color | orange | Color of the keys changed by the player |
| Reset Button, Close Button | | **Reset this tab** and **Close** |
| Tab Color, Active Tab Color | grey, orange | Color of the tabs |
| Open Key | None | Opens and closes the menu. None: only the lobby buttons and code open it. |
| Close Action, Close Key | UI/Cancel, Escape | Closes the menu |

`OptionRow`, on each template line: **Label**, **Group** (a `CanvasGroup` faded while the option cannot be changed),
and the control of its kind: **Toggle**, **Slider** with its **Value Label**, **Dropdown**, or for a key line
**Primary Button**, **Primary Label** (keyboard) and **Secondary Button**, **Secondary Label** (gamepad).

The menu blocks the gameplay input while open and saves `settings.json` when it closes.

## Settings Applier

`SettingsApplier` applies the Audio and Graphics options, and registers the options of the Controls, Audio and
Graphics tabs. Saved settings apply when the game starts.

| Field | Default | Meaning |
|---|---|---|
| Mixer | | Optional `AudioMixer`. With one, the volumes set its exposed parameters in decibels. |
| Master Parameter, Music Parameter, Effects Parameter, Ambience Parameter | MasterVolume, MusicVolume, EffectsVolume, AmbienceVolume | Names of the exposed parameters of the mixer |
| Frame Rates | 30, 60, 120, 144, 165, 240 | Choices of the frame rate limit, after *Unlimited* |
| Field Of View Range | 50 to 100 | Range of the field of view slider |
| Render Scale Range | 0.5 to 1 | Range of the render scale slider |

## Adding your own options

Register them from your module when the game starts. They appear in their tab, or in a new tab:

```csharp
[RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSceneLoad)]
private static void RegisterOptions()
{
    OptionsCatalog.Register(OptionDefinition.Toggle("Display", "display.compass", "Compass", true));
    OptionsCatalog.Register(OptionDefinition.Slider("Gameplay", "gameplay.difficulty", "Difficulty", 1f, 0.5f, 2f, value => value.ToString("0.0") + "x"));
    OptionsCatalog.Register(OptionDefinition.Choice("Gameplay", "gameplay.crosshair", "Crosshair", 0, () => new[] { "Dot", "Cross", "None" }));
}
```

Then read and follow the setting where it is used:

```csharp
bool compass = GameSettings.GetBool("display.compass", true);
GameSettings.Changed += key => { if (key == "display.compass") Refresh(); };
GameSettings.SetFloat("gameplay.difficulty", 1.5f);   // from code, the menu follows
```

## Where the settings are kept

Nothing goes to PlayerPrefs. Two readable JSON files sit in `Application.persistentDataPath`
(`%USERPROFILE%\AppData\LocalLow\<Company>\<Product>` on Windows):

| File | Holds |
|---|---|
| `settings.json` | Every option, by key. Written when the options menu closes, and when the game quits. |
| `player.json` | The name and the local save id of the player. In the editor, each Multiplayer Play Mode virtual player has its own `player_editor_XXXXXXXX.json`. |

Players can back them up, fix them by hand, or sync them with Steam Cloud (add both files to the Auto-Cloud paths).
Delete `settings.json` to put every option back to its default. `GameSettings.FilePath` gives its full path.
`LocalStore` is the class behind both files: `new LocalStore("myfile.json")` gives you your own.

`OptionDefinition.interactable` greys a line out, `order` sorts it in its tab, and `OptionsCatalog.Remove(key)`
removes an option of the template. The keys of the template are in `SettingKeys`.

## Changing the keys

The **Keys** tab lists the actions of the **Player** map of `UST_InputActions`, Move as its four directions, with a
keyboard slot and a gamepad slot. Click a slot, then press the new key, mouse button or gamepad button. Escape cancels;
the wait also ends after 8 seconds. Changed bindings show in orange. **Reset this tab** puts every key back.

Changes apply to the actions themselves: every component and every on-screen prompt follows at once. They are saved in
`settings.json`, under `controls.bindings`. Several actions may share a key on purpose (R reloads, and repairs in build
mode), so a key used elsewhere is not refused.

On the options menu: **Rebind Maps** (Player) and **Rebind Excluded** (Look) choose the lines, **Input Actions** the
asset (empty: the one of the Close Action). For a menu built before, run **Tools > UST > Advanced > Settings > Add Keys Tab To Options Menu**.

```csharp
InputRebinding.ResetAll();
InputRebinding.Changed += RefreshMyHints;
```

## Not in this version

- Language.
