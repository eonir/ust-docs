# Settings

The Settings module adds the options menu: what the HUD shows, the controls, the audio and the graphics. Every option
applies at once and is kept on the machine in `settings.json`. The lobby opens it from the main menu and from the session
menu in game.

The module is optional: add `UST_DISABLE_SETTINGS` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module). Without it, the saved settings and their defaults
still apply where they are read (camera sensitivity, nameplates, chat, voice), there is just no menu.

## Setup

Run *UST Tools > Advanced > Settings > Add Options Menu To Scene*. It adds the **UST Options UI** canvas with the `OptionsMenu`
and the `SettingsApplier`. Rebuild the lobby (*Add Lobby To Scene* after deleting **UST Lobby UI**) to get its
**Options** buttons; they hide themselves when the scene has no options menu.

Without the lobby, set **Open Key** on the options menu, or call `OptionsMenu.Open()` from your own menu.

## Options

| Tab | Option | Default |
|---|---|---|
| Display | Names of the players, health of the players, names visible up to | on, on, 40 m |
| Display | Chat messages on screen ([Chat](chat.md)) | on |
| Display | Pings, placed markers, markers on far players, places of the world, markers on the edges, markers visible up to ([Markers](markers.md)) | on, 500 m |
| Display | Minimap, minimap zoom ([Markers](markers.md#map-and-minimap)) | on, 150 m |
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
asset (empty: the one of the Close Action). For a menu built before, run **UST Tools > Advanced > Settings > Add Keys Tab To Options Menu**.

```csharp
InputRebinding.ResetAll();
InputRebinding.Changed += RefreshMyHints;
```

## Not in this version

- Language.
