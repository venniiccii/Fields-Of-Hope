# Fields of Hope

An educational farming and food-security game set in Baguio City, Ilocos Norte and Davao City.
Built with **Unity 6** and **C#**. It runs in a web browser (WebGL), as a Windows app, and can be exported for Android.

## Requirements

- **Unity Hub** (Windows or macOS, Intel or Apple Silicon)
- **Unity 6000.3.15f1**: use this exact version so the project is not upgraded
- Modules, added in Unity Hub > Installs > 6000.3.15f1 > Add modules:
  - **Web Build Support**: for the browser version
  - **Windows Build Support** or **Mac Build Support**: for a desktop app
  - **Android Build Support**: for an installable APK
- Optional, for the art and map tools in `Tools/`: **Python 3** with Pillow (`pip install pillow`)

## Open and play

1. Unity Hub > **Add** > **Add project from disk** > choose this folder.
2. The first open takes 5 to 15 minutes while Unity builds its cache (the `Library/` folder).
3. Open `Assets/Scenes/SampleScene.unity` and press **Play**.

The game builds itself at runtime (`GameApp` starts automatically), so the scene needs no setup.

### On a Mac

The steps are the same. Install Unity Hub for macOS, then Unity 6000.3.15f1 with the modules you need.
The browser version also plays in Safari or Chrome on a Mac with nothing installed.

## Controls

| Action | Mouse / keyboard | Touch |
| --- | --- | --- |
| Use a plot, button or person | Click | Tap |
| Look around the farm | Drag, or WASD / arrow keys | Swipe |
| Zoom | Mouse wheel, `+` / `-` keys, or the + / - buttons | Pinch |
| Back to the farm | `H` or the Home button | Home button |
| Tools | `1` basket, `2` watering can, `4` axe, `5` shovel | Hotbar |
| Close a panel / drop a tool | `Esc` | X button |

In the Unity editor and in development builds, **Settings > Developer tools** can skip time (+10 min, +1 hour, +1 day),
trigger heavy rain, pests or drought, and grant coins and XP. This helps when testing timers and events.

## Saving

There is no server or online database. Each player's farm is saved on their own device as JSON files:

- `accounts.json`: the farm names on that device (name only, no password)
- `saves/<farm>.json`: one file per farm

They live in Unity's `Application.persistentDataPath/FieldsOfHope`. In the browser this is the browser's IndexedDB storage.
Saves are written atomically (temporary file, then swapped in), so a crash cannot corrupt them.

## Tests

113 automated tests (Unity Test Framework / NUnit), all passing:

| Suite | Count | Covers |
| --- | --- | --- |
| `Assets/Tests/EditMode` | 101 | Game rules: planting, watering, harvest, soil, shop, deliveries, quests, events, locations, travel, saving and loading, the village routine (night, campfire, rain) |
| `Assets/Tests/PlayMode` | 12 | The real game screens: tutorial, accounts, night, rain, campfire, animals, zoom, the moon on the clock |

Run them from **Fields of Hope > Tests** in the menu bar (reports go to `TestResults/`) or from **Window > General > Test Runner**.

## Builds

From **Fields of Hope > Build** in the menu bar:

- **WebGL / Windows Release**: the finished game, no watermark, no expiry.
- **WebGL / Windows Evaluation Preview**: adds an "Evaluation preview" watermark and stops working 14 days after the build date.

Builds go to `Builds/`. A WebGL build is a static website (`index.html` + `Build/`) that any static host can serve
(Vercel, Netlify, GitHub Pages, itch.io). The build also writes a `vercel.json` with the right headers for its Brotli files.

- **Android APK Release** / **Android APK Evaluation Preview**: an installable `.apk` for Android 7.1 and newer, landscape only,
  in `Builds/Android-Release/FieldsOfHope.apk`. Needs the **Android Build Support** module (with OpenJDK and Android SDK & NDK Tools).
  The first Android build re-imports the art and takes 10 to 20 minutes.
  The package name is `com.fieldsofhope.game`; change `AndroidPackage` in `Assets/Editor/BuildTools.cs` to your own before publishing.
  The APK is signed with Unity's debug key, which is fine for installing on phones directly. For Google Play, build an `.aab`
  signed with your own key (Player Settings > Publishing Settings).

The app icon is `Assets/Icons/app_icon.png`.

## Code layout

```
Assets/Scripts/Data/GameDefs.cs        crops, locations, levels, prices, shop items
Assets/Scripts/Server/                 game rules and saving (runs inside the game, no internet needed)
    GameServer.Core.cs                 clock, saving, transactions
    GameServer.Auth.cs                 name-only accounts, loading saves
    GameServer.Farm.cs                 plots: plant, water, harvest, clear land
    GameServer.Community.cs            requests, deliveries, shop, locations, travel
    GameServer.Time.cs                 daily reset, events (rain, pests, drought), offline catch-up
    SaveData.cs                        what is saved
Assets/Scripts/Client/                 everything on screen
    GameApp.cs, TitleScreens.cs        start-up, title and name screens
    FarmScreen.cs                      the farm view, HUD, hotbar, plots, zoom
    Panels.*.cs, Tutorial.cs           menus and the guided tutorial
    MapView.cs, MapLayout.cs           buildings, trees and props from each location's layout
    Villagers.cs, Walkers.cs, Farmhand.cs, Routine.cs, VillagePaths.cs
                                       the villagers, their daily routine, the farmhand
    Animals.cs                         animals that wander by day and sleep at night
    Ambience.cs, LivingPainting.cs     day and night, lights, campfire, weather, rippling water
Assets/Resources/                      art, maps (Map/layout_*.json), music, font, water shader
Assets/Editor/                         build menu, test runner menu, art import settings
Assets/WebGLTemplates/                 the web page around the browser version
Tools/                                 Python tools for maps and sprites (see below)
```

## Tools (Python)

| Script | What it does |
| --- | --- |
| `map_builder.py` | Builds each location's ground image and `Map/layout_*.json` from the pieces in `Tools/source_pieces`: `python map_builder.py Tools/source_pieces Assets/Resources <preview folder>` |
| `make_field_art.py` | Each location's field tiles, obstacles and field edges |
| `make_village_art.py` | The campfire and the moon icon |
| `make_emotes.py` | The speech-bubble emotes |
| `slice_walk.py`, `slice_work.py`, `slice_farmhand.py`, `slice_animal.py`, `slice_sheet.py` | Cut AI-generated sprite sheets into game animation strips |
| `make_masks.py`, `round_corners.py`, `visibility.py` | Water masks, UI corners, a check for villagers hidden behind objects |

## Art, music and fonts

- Pixel art. Characters, animals, buildings and their animations were generated with Higgsfield AI and cleaned up with the tools above.
- Music: "Town Theme RPG" by cynicmusic (OpenGameArt.org), CC0. See `Assets/Resources/Audio/MUSIC-LICENSE.txt`.
- Font: Fredoka, SIL Open Font License. See `Assets/Resources/Fonts/Fredoka-OFL.txt`.
