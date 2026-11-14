# D. Phase 1 Investigation Report: project and world foundation

Draw3D lifecycle validation programme. Investigated 2026-11-14. Covers lifecycle phases L1 (project) and L2 (world).

Nothing in the engine was changed. This report says what exists, what was tested and how, what was found, and what would have to change for project folders. Decisions are yours.

## 1. The short answer

- **Draw3D has no project. It has worlds, and one library.** A world is a single file. Everything else a game is made of (characters, moves, items, abilities, effects, sounds, behaviours, imported models, pictures, sounds, profiles, saved games) lives in the engine's own folders and is shared by every world on this computer.
- **The basic world workflow works through the UI.** New world, Save as, Save with a backup, keep as a game, close, open, and begin a second game as a copy all completed in the UI test with no step stuck (53 steps, 0 problems).
- **A game cannot be moved, shared or backed up as a unit.** Its world file travels; its imported files, its library entries' originals, its profiles and its saved games do not.
- **Opening a world saved before 2026-11-08 quietly changes what it is made of.** Between 45 and 58 of its characters and moves become "this world's own versions", and the library's default player character is added to it. This is the source of two of the "wrong character" reports (document E).

## 2. How this was investigated

| Method | What | Evidence |
|---|---|---|
| Code reading | The start-up path, new/open/save, the texts and library layer, the Worlds page, imports, export. | File names beside each claim below. |
| File reading | All 40 world files in `editor/`, `editor/genres` and `editor/dioramas` read for what they hold and who they would play. | Section 6. |
| UI test | `editor\run.ps1 -Author lifecycle`: 53 steps, real clicks and keys only, on a sandbox copy of the project's files. 0 steps stuck, 0 script errors. | `evidence/2026-11-14/ui-test-log.txt`, 31 pictures, `seen.json`. |
| Not done | No hand test. No export was run. Imports and designs were read in code only, not tested. | Marked "read in code" below. |

**Limits of the UI test** (framework, section 6): it ran on copies in a sandbox folder; the editor was put on "nothing open" by the script rather than by a real start; "Come back where I was", the title card and the class picker are switched off in scripted runs.

**An accident to report.** My first attempt used a port that another Claude session was already using for a demo (`holdg`, port 47870). My run could not start its host and did nothing, and I stopped it after about two minutes, but it overwrote the first part of that session's `run-47870.log`. That demo went on running; its early log lines (beats 1 to 6) are lost. If that session's result matters, it should be run again. I also restored seven tracked pictures in `editor/godot/author/` that the author harness clears at the start of every run.

## 3. What exists today

### 3.1 Is there a project?

No. The pieces that look like one:

| What the UI calls it | What it is |
|---|---|
| **Begin** (shown at start) | A window offering: New world…, Open a saved world…, Project…, Characters…, Animate and moves…, a GAMES list, and WORKED ON LATELY. |
| **GAMES** | A list of world files you marked "Keep here as a game". It is a list of file paths kept on this computer (`user://editor.cfg`). It is not a folder and holds nothing. |
| **Project window** | A browser over the world that is open plus the shared library. Its title is the world's name. With no world open it shows the library. |
| **"Every world of the project"** (Project › Worlds) | Every `.json` in three fixed folders: `editor/`, `editor/genres`, `editor/dioramas`, plus the world being edited. (`worldspage.gd` `world_dirs`) |
| **"The project's files"** | The text files in `editor/godot/` (`characters.txt`, `moves.txt`, and 85 more). They are the engine's own source folder. |

So today the engine installation is the one and only project, and every game made with it shares it.

### 3.2 Where everything is stored

| What | Where | Whose |
|---|---|---|
| A world: land, makers, placed things, the kinds of object it uses, its style | One `.json` file, anywhere you save it | The world's |
| The world's copy of every project text (60 texts in a new world) | Inside the world's file | The world's, but most of it "follows the library" (3.4) |
| Who the world plays (`assigned`) | Inside the world's file | The world's |
| Earlier saves | A `backups` folder beside the world's file, the newest 12 | The world's |
| Characters, looks, clip libraries, moves, sets, combos, items, abilities, shots, effects, sounds, impact feel, behaviours: **the library** | `editor/godot/*.txt` | Every world's |
| Character **profiles** (set of moves, what is carried, abilities, the controller layout) | `editor/godot/profiles.txt` | Every world's. Never copied into a world |
| Designs of the Asset Library | `editor/library/*.txt`; your own in `editor/library/mine/` | Every world's. Copied into a world when used |
| Imported models, kits, character bodies, animations, sounds | `editor/godot/models`, `kits`, `characters`, `sounds` | Every world's |
| Imported pictures and textures | `%APPDATA%\Draw3D\textures` | This computer's |
| Your own buttons, older button profiles, how a thing sits on a model | `%APPDATA%\Draw3D\controls.txt`, `profiles\`, `fits.txt` | This computer's |
| Saved games | `editor/saves/SLOT.json`, one folder for all worlds | Every world's |
| Recent worlds, the GAMES list | `user://editor.cfg`, as full paths | This computer's |
| Where you last stopped playing | `user://kept_place.json` | This computer's |
| Deleted library entries; old versions of changed ones | `editor/godot/trash/`, `editor/godot/mends/` | Every world's |

"`user://`" is `%APPDATA%\Godot\app_userdata\Editor-Adapter`.

### 3.3 Ways to create a world

All are reached from Begin › New world… or World › New world… (Ctrl+N), except where said.

| Way | What you get | Tested |
|---|---|---|
| A plot of land (blank) | A plot as wide as you say, sea around it | UI test: works |
| Nothing at all | Only the sea | UI test: works |
| An empty island; An island, wild; Two islands; Two lands; A town by the sea | Land by seed, hills and size | Listed; not tested |
| A diorama; A village; A city; A dungeon; An overworld | A generated world with its own settings, and for village and city a plan seen first | Listed; not tested |
| A village or city from a map | Built from a map file exported as JSON | Listed; not tested |
| Genre: … (6 of them) | A copy of a ready-made world, with no file | Listed; not tested |
| The demo island (everything) | The island the demos use | Listed; not tested |
| World › Open-world generator… | A large generated world in layers | Listed; not tested |
| Begin › GAMES › New game from it… | A copy of a game you kept, saved to a new file at once | UI test: works |
| World › Save as… | The open world under another name | UI test: works |

A new world has no file until you save it. It is asked twice before it replaces the world that is open.

### 3.4 How a world relates to the library

This is the part that explains most surprises, so it is set out step by step. (`assets.gd`, `main.gd` `settle_library`, `write_asset`)

1. A world's file carries a full copy of each library text, so an exported game is complete.
2. Each entry in that copy is one of two things:
   - **Following the library**: when the world opens, the entry is replaced by the library's current version.
   - **This world's own version**: marked with a note line; the library's changes never reach it.
3. Anything the library has that the copy lacks is **added** to the copy when the world opens.
4. A copy saved before 2026-11-08 has no marks. The first time it is opened, every entry that differs from today's library is marked as the world's own, so that "nothing it plays changes".
5. Changing a library asset while a world is open asks, if other worlds use it: **Update shared asset**, **Create separate version**, or **Cancel**. If no other world uses it, it is written to the library with no question.
6. Other texts (quests, talks, weather, crops…) are the world's own. A switch, "World texts: also for new worlds / this world only", decides whether a change is also written into the engine's starting file.

Designs from the Asset Library work differently: they are **copied** into the world when placed and never follow. Imported files are **linked** by a path relative to the engine's folder and never copied into a world.

## 4. Workflow records

Each record follows the framework's thirteen parts. "UI test" means the run of 2026-11-14.

### L1.1 Start Draw3D and choose where to begin

| Part | |
|---|---|
| 2. Outcome | The maker sees every way to begin and picks one. |
| 3. Start | The editor started by hand, with "Show this when Draw3D starts" ticked (the default). |
| 4. UI steps | The **Begin** window is shown. Later: World › Begin: new, open, worked on lately… |
| 5. Expected | Nothing is open; the buttons New world…, Open a saved world…, Project…, Characters…, Animate and moves…; GAMES; WORKED ON LATELY. |
| 6. What exists | `startview.gd`. Shown only when started by hand with no world named. Unticked, the last world opens straight away. |
| 7. Test | UI test P1.1. Picture `01-p1-1-begin.png`. The window was opened by the script's set-up, so the real start-up path is **not covered**. |
| 8. Function | The window showed all five buttons and both lists. No world open, 0 unsaved changes. |
| 9. Usability | Clear. "Project…" with nothing open shows the library, which the button does not say. |
| 10. Missing | No "New game" or "New project": a game is begun as a world. No way to choose a folder for a game. |
| 11. Acceptance (proposed) | (a) Started by hand, Begin is shown and nothing is open. (b) Each button opens what it names. (c) Nothing is marked unsaved before anything is done. |
| 12. Hand test | Document F, test 1. |
| 13. Regression | `author_lifecycle` P1.1. Rerun when `startview.gd` or `main.gd` start-up changes. |

**Status: Claude tested**, with the start-up path left to the hand test.

### L2.1 A new blank world

| Part | |
|---|---|
| 2. Outcome | A world with a plot of land and nothing else. |
| 3. Start | Begin, or any world. |
| 4. UI steps | Begin › **New world…** › the list shows "A plot of land (blank)" › **Make the new world…** › **Yes: replace the world in the editor**. |
| 5. Expected | Land in the view, no file, the Terrain window open with Extend land chosen. |
| 6. What exists | `newworld.gd`, `main.gd` `new_world`. Fourteen beginnings plus six genres. |
| 7. Test | UI test P1.2 (a plot) and P1.6 (Nothing at all). Pictures `02` to `04`. |
| 8. Function | Made one maker, "island (land)". No file. 60 texts. Terrain opened. The message said what to do next. |
| 9. Usability | The list of beginnings has 20 entries, so it opens as a search box, which works. The world already has 1 unsaved change before anything is done (F-L2-02). |
| 10. Missing | The new world has **no Player start** and nothing says so here (F-L2-01). It already has a player character nobody chose (document E, F-L3-01). |
| 11. Acceptance (proposed) | (a) Two presses after the choice make the world. (b) It has land and no file. (c) The window says what the world lacks to be played as a game. |
| 12. Hand test | Document F, test 2. |
| 13. Regression | `author_lifecycle` P1.2, P1.6. |

**Status: Claude tested.**

### L2.5 Save a world

| Part | |
|---|---|
| 2. Outcome | The world written to a file; what the file had before is kept. |
| 4. UI steps | World › **Save as…** › type a name › Enter. After that World › **Save** or Ctrl+S. |
| 5. Expected | The file exists, the title shows its name, unsaved changes are 0, a backup appears on the second save. |
| 6. What exists | `main.gd` `write_world`, `keep_before_writing`. The dialog is Draw3D's own. The newest 12 backups are kept. |
| 7. Test | UI test P1.3, P1.4. Picture `05`. |
| 8. Function | File written (799 KB for a blank plot). Unsaved 0. Listed under WORKED ON LATELY. Second save kept one backup beside it. The file held 1 maker, 0 kinds, 60 texts. |
| 9. Usability | **The dialog opens in the engine's own folder** (`E:/AI apps/Draw3d/editor`), among about twenty of the engine's sample worlds (F-L1-02). A blank world is 799 KB because it carries the whole library. |
| 10. Missing | No suggested folder for your games. No name suggested. |
| 11. Acceptance (proposed) | (a) The first save asks where. (b) Later saves write there and keep a backup. (c) The dialog starts somewhere meant for your games. |
| 12. Hand test | Document F, test 3. |
| 13. Regression | `author_lifecycle` P1.3, P1.4. The older tour part `keeping`. |

**Status: Claude tested.**

### L1.2 Keep a world as a game, and L1.3 begin a new game from it

| Part | |
|---|---|
| 2. Outcome | The world is one press away in Begin, and a second game can be begun as a copy without touching the first. |
| 4. UI steps | World › Begin… › **＋ Keep [name] here as a game**. Then Begin › GAMES › **New game from it…** › type a name › Enter. |
| 5. Expected | The game is listed. The copy is saved to its own file. The first file is unchanged. |
| 6. What exists | `main.gd` `keep_as_game`, `new_game_from`. The list is paths in `user://editor.cfg`. |
| 7. Test | UI test P1.5, P1.7. Pictures `16`, `17`, `22`. |
| 8. Function | Listed. The copy was written; the first file's checksum was unchanged; the copy held the same player and moves. |
| 9. Usability | The copy is not itself listed as a game. The save dialog again opens in the engine's folder. |
| 10. Missing | "Game" here means "a listed world". Nothing groups a game's worlds or files (F-L1-01). A moved or renamed file drops off the list with no message (read in code). |
| 11. Acceptance (proposed) | (a) Keep lists it. (b) New game from it never writes the original. (c) A listed game that has moved is said to be missing, not silently dropped. |
| 12. Hand test | Document F, test 5. |
| 13. Regression | `author_lifecycle` P1.5, P1.7. |

**Status: Claude tested.**

### L2.6 Close, open and go back to a world

| Part | |
|---|---|
| 2. Outcome | The world as it was saved. |
| 4. UI steps | World › **Open…** › type or choose the file › Enter. Also World › Recent: …, and Begin › WORKED ON LATELY. |
| 5. Expected | Asked first if the open world has unsaved changes. The same makers, player and settings after opening. |
| 6. What exists | `main.gd` `open_world`, `guard`. |
| 7. Test | UI test P1.6. Pictures `18`, `19`. The Open dialog was used; the Recent row and the Begin row were **not pressed**. |
| 8. Function | Asked (Save, then go on / Leave without saving / Stay here). Opened with 0 unsaved changes. The file was untouched. Same maker, same player choice, same moves line. |
| 9. Usability | After opening, the view sits 150 m out, so a 24 m plot is a dot in the sea (F-L2-03, picture `19`). The untouched empty world asked about "1 change" before letting go (F-L2-02). Times under WORKED ON LATELY are five hours off: 19:32 shown for 14:32 (F-L2-04). |
| 10. Missing | The view you saved from is not kept. |
| 11. Acceptance (proposed) | (a) Opening never loses unsaved work without asking. (b) What was saved is what opens. (c) The view opens on the world, not far from it. |
| 12. Hand test | Document F, test 4. |
| 13. Regression | `author_lifecycle` P1.6. |

**Status: Claude tested** (Open dialog only).

### L1.4 See what the game is made of (Project › Worlds)

| Part | |
|---|---|
| 2. Outcome | Every world of the game listed, with the ways between them. |
| 4. UI steps | World › Project: what the game is made of… › **Worlds**. |
| 5. Expected | Both benchmark worlds listed. |
| 7. Test | UI test P1.8. Picture `23`. |
| 8. Function | **One card was shown**: the world being edited. The other benchmark world, in the same folder, was not listed (F-L1-03). |
| 6. What exists | The list reads three engine folders only. |
| 11. Acceptance (proposed) | A game's worlds are listed wherever the game is saved. |
| 12. Hand test | Document F, test 6. |

**Status: Claude tested (fails)** for a game saved outside the engine's folders.

### Not yet tested in L1 and L2

L1.5 (Project Settings), L1.6 (files), L1.7 (dependency check), L2.2 to L2.4 (generated, map, genre), L2.7 (restore a backup), L2.9 (the Player start), L2.10 to L2.12. Status: **Discovered** for L1.6, L1.7; **listed** for the rest.

## 5. Your asset questions, one by one

Answers marked **(tested)** come from the UI test. The rest are **read in code** and not yet tested.

**Where are generated assets stored?**
In the world's file. A generated thing is not stored as an object: the world stores the **maker** (the forest, the meadow, the street) with its settings and seed, and the things are made again from it each time. Changes you make by hand to one thing are stored on its maker, against that thing's name.

**How are generated objects identified and referenced?**
By the maker's id plus the thing's place in it, such as "rock number 7 of this meadow". There is no id that outlives the maker. Changing the maker's seed makes hand changes "land on other things" (`makers.gd`).

**How are external assets referenced?**
By a path relative to the engine's folder (`model models/NAME.glb`, `kits/NAME/FILE`, `characters/NAME/`), or by bare name for pictures and sounds. Never by a path inside the game's own folder, because there is none.

**Embedded, copied or linked?**

| Asset | On import | In the world |
|---|---|---|
| A model | Copied into `editor/godot/models/` and re-written as `.glb` | Linked by relative path |
| A kit (folder of models) | Copied into `editor/godot/kits/NAME/` | Linked |
| A character body, animations | Copied into `editor/godot/characters/NAME/` | Linked, through the library's entry |
| A picture or texture | Copied into `%APPDATA%\Draw3D\textures` | Linked by name |
| A sound | Copied into `editor/godot/sounds/` | Linked by name |
| A design from the Asset Library | Stays in the library | **Copied** into the world when placed |
| A character, move, item, ability | Stays in the library | **Copied, then follows** the library unless made the world's own |

Nothing is embedded in the world file.

**What happens when an asset is moved, renamed or deleted?**

- **A model file missing:** one message, "The model … could not be read: there is no file …", and the thing is drawn without it.
- **A picture missing:** silent. The thing is drawn in plain colours.
- **A sound missing:** silent.
- **A library design changed:** the world keeps its copy; the Library says the world's differs, with a button to replace it.
- **A library design deleted:** no world changes; the file is moved to `mine/earlier`.
- **A library design renamed:** there is no rename.
- **A library character or move deleted:** it disappears from every world that only followed it, the next time each opens. A world's own version stays.
- **A removed import:** moved to `%APPDATA%\Draw3D\put_aside`, where it is still found by name.

**Can assets be shared across worlds?**
Yes, and today that is the only mode. Everything is shared by every world of every game on this computer. What does not exist is the opposite: an asset that belongs to one game and not to another.

**Can a project be moved without breaking references?**

- *The world file and the worlds it leads to, moved together to another folder on this computer:* travel between them still works, and models still load because they are in the engine's folder. Lost: the Worlds page listing, the GAMES and recent entries, "Come back where I was", the old backups folder.
- *To another computer:* also lost are every imported model, kit, character body, animation and sound, your pictures, `profiles.txt`, your own library designs, and saved games. The world would open with missing models and with different moves and buttons.

**How could dependencies be checked before running or exporting?**
What exists: Project › Overview shows "Missing files" for models, pictures, sounds, fonts and the title picture named by the world. It does **not** cover character bodies or animation libraries. Export copies what it finds and builds anyway when something is missing; it prints a line and goes on. Nothing is checked before Play.

One more, read in code and important: **the exported game does not include `profiles.txt`**. A profile decides a character's moveset, what it carries and its buttons while you test in the editor (document E). So an exported game would likely play with different moves and buttons from the ones you tested. This must be confirmed by a real export in L12.

## 6. What the forty world files hold

Read from disk on 2026-11-14.

| | Count |
|---|---|
| World files in the three engine folders | 40 |
| With a "who it plays" choice of their own (`assigned`) | 1 (`editor/world.json`) |
| With no copy of the characters at all | 14 |
| With a copy of the characters saved before 2026-11-08 (never settled on disk) | 26 |
| Whose copy says the ranger is played | 18 |
| Whose copy says the skyfarer is played | 3 |
| Whose copy says the castaway is played (a character the library no longer has) | 3 (the three islands) |
| Whose copy says two characters are played at once | 1 (`testworld.json`: ranger and skyfarer) |
| Whose copy names no played character | 1 (`showcase-physics.json`) |

Also found: **two different files named `hold_the_light.json`**, one in `editor/` (12 November) and one in `editor/godot/demo/` (13 November). The second is the one at the top of your recent list. `waylaid.json` and `survival_valley.json` are also in `editor/godot/demo/`, the folder demos write to.

## 7. Replacing a generated object: the rock

Read in code; not yet tested. The UI names are as the code has them.

**Identify it.** Click it. World outline shows a section "This one ROCK" under its maker.

**(a) Edit that one rock.**
- "its size (×)", "Another colour", "Move this one…", "Duplicate this one…", "Take away this one".
- **"Shape this one only…"** gives it a private copy of its design to reshape.
- "Put this one back as its maker makes it" undoes all of it.
- Right-click › "Make it my own (a copy, this one only)…".
- Edit › "Take out of its layout" cuts it loose from its maker.

**(b) Replace that one rock.**
- The **"its design"** list on "This one ROCK" gives it another design.
- The list holds only designs already in this world, and built-in ones. A Library design must be placed in the world once first.
- An imported model becomes a world design on import, so it appears in the list.

**(c) Replace every rock of a maker.**
- The maker's section **"What it is made of"**: "Add a design…", a share for each, "take out", "Back to the usual".
- From the Library: "Give to ▾".
- For things painted by brush: the **"Replace it…"** brush. It does not touch things made by a wood or meadow.

**What is missing.**
- No "swap design X for design Y across the whole world".
- No category-level replacement ("every rock, whoever made it").
- No reusable variants of one asset with a name ("mossy rock", "dry rock") chosen per world.
- Hand changes are tied to the maker's seed; change the seed and they move to other things.
- "Shape this one only" does not work for things with bones.

## 8. Findings

Severity and cause as defined in the framework.

| Id | Severity | Finding | Cause |
|---|---|---|---|
| F-L1-01 | Question | There is no project. The engine's folder is the only one, and every game shares its library, imports, profiles and saves. | Confirmed (code, files) |
| F-L1-02 | Major | The Save dialog for a new world opens in the engine's own folder, among its sample worlds. | Confirmed (UI test) |
| F-L1-03 | Major | Project › Worlds lists only worlds in three engine folders. A second world of the same game, saved beside the first elsewhere, is not listed. | Confirmed (UI test) |
| F-L1-04 | Major | The GAMES and recent lists are full paths on this computer. A game whose folder is moved or renamed drops off both with no message. | Likely (code) |
| F-L1-05 | Major | Imported files go into the engine's folders, shared by all games, and are not carried with a world. | Likely (code) |
| F-L1-06 | Major | Missing pictures and sounds are silent; the missing-files list does not cover character bodies or animations; export builds anyway. | Likely (code) |
| F-L1-07 | Major | The exported game does not carry `profiles.txt`, so its moves and buttons would differ from the editor's. | Likely (code); confirm in L12 |
| F-L1-08 | Major | Two files named `hold_the_light.json` in two folders; games the demos build are saved in the demos' own folder. | Confirmed (files) |
| F-L2-01 | Major | A new world has no Player start. The game starts "where the view last looked", which is saved with the world and moved at every play: three saves, three different starts. | Confirmed (UI test) |
| F-L2-02 | Minor | A new world nobody has touched has 1 unsaved change and asks before it is left. | Confirmed (UI test) |
| F-L2-03 | Minor | After opening, the view is 150 m out; a small world is a dot. | Confirmed (picture 19) |
| F-L2-04 | Minor | Times under WORKED ON LATELY are shown five hours off local time. | Confirmed (UI test) |
| F-L2-05 | Major | Opening a world saved before 2026-11-08 silently turns 45 to 58 of its characters and moves into "this world's own versions", with no message and 0 unsaved changes. They stop following the library. | Confirmed (UI test, four worlds) |
| F-L2-06 | Minor | A long message covers the weather buttons at the bottom right. | Confirmed (picture 19) |
| F-TEST-01 | Major | The existing author harness cannot choose from any list of more than twelve items since the search lists of 2026-11-06. The older author scripts (Emberwatch, Islandbound, the dioramas) would now stop at those steps. | Confirmed (first run) |

## 9. What project folders would need

Not a design. A list of what would have to change, for the review.

**What a project folder would hold:** the game's worlds; its own library (characters, moves, items, abilities, effects, sounds, behaviours); its profiles; its imported files; its own designs; its saved games; its settings; a list of its worlds.

**What would have to change in the engine:**

1. **A project location the editor knows.** Today `written_dir`, the library folder, the saves folder and the Worlds page's folders are all fixed to the engine's folders. Each becomes "the open project's folder".
2. **Begin gains "New game" and "Open game".** A world is created inside a game.
3. **Imports copy into the project**, and references become relative to the project, with the engine's own assets as a second place to look.
4. **Profiles move into the project** (or into the character), and export carries them.
5. **The engine's library becomes a starting set.** A new project begins with a copy, or with links to the built-in set plus its own additions. This is the largest decision: copy means games drift apart from engine fixes; link means the engine's changes reach every game.
6. **A dependency check** that lists every file and library entry a game uses and says which are missing, run before Play and before Export.
7. **A migration** for the forty existing worlds, deciding per world which library entries and imports go with it.

**Decisions needed from you before any of it:**

- Does a new game start with a **copy** of the engine's characters and moves, or **link** to them?
- Are profiles a property of the **game**, of the **character**, or both?
- Should the engine's sample worlds and your games be kept in **separate places** now, before project folders exist? This can be done without the larger change.
- Which existing worlds are **your games** and must be migrated, and which are samples?

## 10. What is recommended next

1. Review these findings and the proposed acceptance criteria.
2. Do the hand tests for L1 and L2 (document F, tests 1 to 6).
3. Decide the four questions in section 9. The project-folder work should not start until L3's findings (document E) are read with them, because profiles and the library's default player are part of the same decision.
