# A. Game Development Lifecycle Map

Draw3D lifecycle validation programme. Written 2026-11-14.

The order in which a game is made in Draw3D, phase by phase, broken into workflows with their starting conditions, outcomes and dependencies. Each workflow is tracked in the register (document C) by the rules of the framework (document B).

## 1. How to read it

- **L1 to L12** are the twelve lifecycle phases, in your order.
- Your "Phase 1 investigation" covers **L1 and L2**. Your "Phase 2 investigation" covers **L3**.
- Each workflow has an id (L3.2), a start, an outcome, the UI path as the editor words it, and what it depends on.
- **How far each has been taken:**
  - L1 to L3 were read in the code and run through the UI by Claude on 2026-11-14. Their records are in documents D and E.
  - L4 to L12 are **listed only**: found by reading the menus, windows and existing notes. Nothing in them has been tested for this programme, and their UI paths are unconfirmed until each phase is investigated. They are here so the whole road is visible.

## 2. What inspecting the engine changed in the breakdown

Your twelve phases stand. Five things about the engine as it is shape how they connect.

1. **There is no project yet, only worlds and one shared library.** A world is one file. Characters, moves, items, abilities, effects, sounds and behaviours live in one library inside the engine's own folder, shared by every world. So L1 today is "the Begin window, the list of games, and the Project window", not "create a project folder". Document D covers this and what a project folder would change.
2. **Characters and moves exist before any world.** They are library assets, made and edited with no world open. A world only chooses which it uses. So L3 does not strictly need L2 to edit a character, only to play one.
3. **Importing is needed early.** A player model of your own is an import (L9.3). L9 is not a late phase to wait for; its import workflow is pulled forward whenever an earlier phase needs a file.
4. **Later phases can override earlier choices.** A class chosen at the title (L6.6, L10.2) decides the played character and moveset over what L3 set. A profile (L3.3) decides the moveset and buttons over the character's own. These crossings are marked as dependencies, because they are where "the wrong character appeared" comes from (document E).
5. **Test play is its own workflow.** Play, Edit here, Playtest and "Come back where I was" are used in every phase, and they keep state of their own between plays. They are listed once, as L3.5.

## 3. The lifecycle

### L1. Game project creation and management

| Id | Workflow | Starts from | UI path | Outcome | Depends on |
|---|---|---|---|---|---|
| L1.1 | Start Draw3D and choose where to begin | The editor started by hand | Begin window (shown at start; later World › Begin: new, open, worked on lately…) | A new world, a saved world, the Project, or the library's characters, chosen | – |
| L1.2 | Keep a world as a game | A saved world open | Begin › ＋ Keep … here as a game | The world is listed under GAMES on this computer | L2.4 |
| L1.3 | Begin a new game from an existing one | A game listed in Begin | Begin › GAMES › New game from it… | A copy opened and saved to a file of its own | L1.2 |
| L1.4 | See what the game is made of | Any world open | World › Project: what the game is made of… (Overview, then each category) | The player, the world, and every asset, each with a page | L2 |
| L1.5 | Set the game as a whole | A world open | Project › Project Settings | The game's flow, title, saves, what players may change | L2; reaches into L10, L12 |
| L1.6 | Find, move, back up or share a game's files | A saved world | World › Show in folder | The folder of the world's file opened | L2.4. **No project folder exists**: see document D |
| L1.7 | Check the game has everything it needs | A world open | Project › Overview (Missing files), Project › Assets & Objects › Files brought in | Missing files named | L9. Partial: see document D |

### L2. World creation and configuration

| Id | Workflow | Starts from | UI path | Outcome | Depends on |
|---|---|---|---|---|---|
| L2.1 | A new blank world | Begin, or any world | Begin › New world… (or World › New world…, Ctrl+N) › A plot of land (blank) / An empty island / Nothing at all › Make the new world… › Yes | A world with land (or none) and no file | L1.1 |
| L2.2 | A new generated world | The same | New world… › A village / A city / A dungeon / An overworld / A diorama; or World › Open-world generator… | A world built by a generator, editable after | L1.1 |
| L2.3 | A new world from a map file | A map exported as JSON | New world… › A village or city from a map; or World › New world from a map's layout… | A world built from the drawn map | L1.1 |
| L2.4 | A new world from a genre template | The same | New world… › Genre: … | A copy of a ready-made world, with no file | L1.1 |
| L2.5 | Save a world | A world with changes | World › Save (Ctrl+S); World › Save as… (Ctrl+Shift+S); World › Autosave every N minutes | The world's file written; what it had before kept as a backup | L2.1 |
| L2.6 | Close, open and go back to a world | A saved world | World › Open… (Ctrl+O); World › Recent: …; Begin › WORKED ON LATELY | The world as it was saved | L2.5 |
| L2.7 | Restore an earlier save | A world saved more than once | World › Restore a backup… | An earlier save opened with no file | L2.5 |
| L2.8 | Duplicate a world | A saved world | World › Save as… to another name; or L1.3 | A second, separate file | L2.5 |
| L2.9 | Set where and how a game starts in the world | A world with land | Create › Characters and the start (the Player start), then its settings in World outline / the Inspector | Where play begins, the camera, the hour, the title, the classes offered | L2.1. Listed; not yet tested |
| L2.10 | Set the world's look, hour and weather | A world | Window › Styles; Window › Time and weather | The style and sky the world opens with | L2.1. Belongs with L4; first touched here |
| L2.11 | Join several worlds | Two saved worlds | Project › Worlds (Catalogue, Connections map); Inspector › In a level | A way from one world into another | L2.5. Listed; not yet tested |
| L2.12 | Bring one world into another | Two worlds | World › Bring in another world as a section… | The second world's contents placed in the first | L2.5. Listed; not yet tested |

### L3. Player character, movesets and controls

| Id | Workflow | Starts from | UI path | Outcome | Depends on |
|---|---|---|---|---|---|
| L3.1 | Choose who the world plays | A world open | Project › Overview › The game plays as; or Character Studio › Play as this | The world's own choice of player character | L2.1 |
| L3.2 | Give the player a moveset | A played character | Project › Overview › Set up the player → › Moves | The set of moves the character has in play | L3.1. **Crossed by L3.3 and L6.6** |
| L3.3 | Lay out the buttons | A played character | Project › Controls & Input (The character played; Game buttons & layouts; Presses; Wheels; Test); Window › Profile & buttons | What each key and controller button does | L3.2 |
| L3.4 | Resolve the model, the rig and the animations | A character | Window › Character Studio (Make…, clips, bones); Create › Import; Animations from Mixamo… | A body that shows every move with a clip | L9.3 for a model of your own |
| L3.5 | Play and check | A world with a player | Play (Tab); Controls (H); Playtest (P); Project › Controls & Input › Test; Edit here (F12) | The player moves and each assigned action happens | L3.1 to L3.4 |
| L3.6 | Save, close, reopen: the setup holds | L3.5 done | L2.5, L2.6 | The same character, moves and buttons after reopening | L2.5, L2.6 |
| L3.7 | Make a new character or a new moveset | The library | Character Studio › ＋ New version; Animator › ＋ New move / ＋ New combo; Project › ＋ Create | A character or set of your own, without new code | L3.4. Listed; not yet tested |

### L4. Environment and world construction (listed only)

| Id | Workflow | UI path | Recorded limits |
|---|---|---|---|
| L4.1 | Shape and paint the ground | Window › Terrain (F8); Terrain bar (Shift+F8) | Land noise, mountain profile and slope colouring are fixed |
| L4.2 | Draw buildings, walls, paths, roads, ponds | Create › Build / Town / Paths / Water / Roads (the construction bar) | Window, door, fence and gate forms are fixed lists |
| L4.3 | A world laid in tiles or hexes | Window › Tile Designer | Edge noise and the held tile's shapes are fixed |
| L4.4 | Plants at scale | Create › Plants; Brushes; World outline › Paint where… | Default woods and meadow plants have no numbers |
| L4.5 | Look, time and weather | Window › Styles; Window › Time and weather | No sky picture; presets for sky, cloud and night cannot be made |
| L4.6 | Regions | Window › Regions (Ctrl+R) | Region numbers fixed |
| L4.7 | Interiors and furnishing | World › Furnish: buildings, yards and camps… | No separate interiors editor found |
| L4.8 | Caves and ways between worlds | Project › Worlds | Caves are built in code |

Depends on L2. Feeds L5 (where people stand), L7 (where quests point), L8 (where foes live).

### L5. Characters, NPCs and interactions (listed only)

| Id | Workflow | UI path | Recorded limits |
|---|---|---|---|
| L5.1 | See and place who is in the world | Create › Characters and the start › Creatures and characters in this world… | – |
| L5.2 | Populate an area | World › Populate: people, creatures and things… | – |
| L5.3 | Daily routines | Window › Routines (Ctrl+H) | – |
| L5.4 | Talks | Project › Gameplay › Talks | – |
| L5.5 | What a thing or person can do | Window › What things can do; Inspector (F7) › What it does | Some behaviour words are still typed |
| L5.6 | Shops and traders | Inspector abilities | A typed field is recorded; unconfirmed |
| L5.7 | Companions | Inspector › What it does | No window of its own found |
| L5.8 | Camps, workers and sides | Project › Gameplay › Camps / Workers / Sides | Some failures are silent |

Depends on L3.4 (characters and their clips), L4 (places).

### L6. Gameplay mechanics, combat and abilities (listed only)

| Id | Workflow | UI path | Recorded limits |
|---|---|---|---|
| L6.1 | Moves and combos | Window › Animator; Window › Combo graph | Wider than the screen at 1280×720 |
| L6.2 | Powers, shots and statuses | Window › Shots and powers; Project › Abilities & Effects | No beams, charging, bouncing shots |
| L6.3 | Impact feel | Window › Impact feel | Only the player's blows stop, shake or flash |
| L6.4 | Needs and stats | World › Needs…; Project › Overview | No "begins in a status" setting |
| L6.5 | Items, goods, recipes, loot | Project › Gameplay; World › Loot tables… | – |
| L6.6 | Classes and skills | Project › Gameplay › Builds and classes / Skills | **A class decides the played character and moves over L3** |
| L6.7 | Puzzles, signals, physics | Project › Gameplay › Puzzles; Window › Signals; Physics (F2) | – |
| L6.8 | Kinds of harm | Project › Gameplay › Kinds of harm | – |

Depends on L3 (the player's moveset is where combat starts).

### L7. Quests, objectives and progression (listed only)

| Id | Workflow | UI path | Recorded limits |
|---|---|---|---|
| L7.1 | Quests | Project › Gameplay › Quests | No visible countdown for timed quests |
| L7.2 | Endings and lore | Project › Gameplay › Endings / Lore | Notes disagree on whether a codex exists |
| L7.3 | Tutorial | Project › Gameplay › Tutorial | – |
| L7.4 | Cutscenes | Project › Gameplay › Cutscenes | – |
| L7.5 | Rewards and levels | A quest's Experience field; Builds and classes | No achievements |
| L7.6 | The journal | Played with J | Its look has no UI path |

Depends on L5 (who gives quests), L6 (what is rewarded).

### L8. AI, enemies and environmental behaviours (listed only)

| Id | Workflow | UI path | Recorded limits |
|---|---|---|---|
| L8.1 | Behaviours | Window › Behaviours (Ctrl+B) | Some words typed |
| L8.2 | Foes | Window › Foes (Ctrl+U) | Its demo is unsteady |
| L8.3 | Threats, raids, hordes | Project › Gameplay › Threats / Bases and raids | Several were never reached in play |
| L8.4 | Wildlife | Create › Wildlife (unconfirmed) | – |
| L8.5 | When this happens | Inspector › When this happens | – |

Depends on L5, L6.

### L9. Asset creation, importing and visual production (listed only)

| Id | Workflow | UI path | Recorded limits |
|---|---|---|---|
| L9.1 | Use the Asset Library | Window › Asset Library (Ctrl+L) | A design is copied into the world, not linked |
| L9.2 | Objects in this world | Window › Objects in this world | – |
| L9.3 | Import a model, a kit, a picture, a sound, animations | Create › Import; Project › ＋ Create | Files go into the engine's folder, not a game's |
| L9.4 | Tree and Plant Generator | Create › Plants | Petal and needle shapes fixed |
| L9.5 | Modelling and Edit shape | Window › Modelling; Edit › Edit shape | – |
| L9.6 | A character's look | Character Studio › Make… | – |
| L9.7 | Replace or restyle one generated thing | World outline › This one …; its design list | No world-wide "swap this design for that" found |

Pulled forward whenever an earlier phase needs a file. The ownership questions are in document D.

### L10. Game interface, menus and HUD (listed only)

| Id | Workflow | UI path | Recorded limits |
|---|---|---|---|
| L10.1 | The display while playing | Window › Game display | The inventory screen's look is code |
| L10.2 | Title, intro cards, pause menu | Project › Project Settings › The game's flow | Several screens "look as made" |
| L10.3 | What players may change | Project Settings › The player | No subtitles or colour-blind modes |
| L10.4 | Wheels and hotbar | Project › Controls & Input | – |
| L10.5 | Maps | Game display › minimap; Tile Designer › A map in hand | – |

Depends on L1.5. L10.2 crosses L3.1 (the class picker).

### L11. Audio, effects, animation and cinematic presentation (listed only)

| Id | Workflow | UI path | Recorded limits |
|---|---|---|---|
| L11.1 | Sounds | Window › Sounds | A full form is unconfirmed |
| L11.2 | Music | Regions › Its music only | No music editor found |
| L11.3 | Effects | Window › Effects | – |
| L11.4 | Clips and animation | Window › Animator; Animations from Mixamo… | No real Mixamo file was ever used |
| L11.5 | Cameras | Window › Play camera | – |
| L11.6 | Moods | Project › Styles & Materials › Moods | – |

### L12. Saving, loading, optimisation, packaging and exporting (listed only)

| Id | Workflow | UI path | Recorded limits |
|---|---|---|---|
| L12.1 | The player's saved games | Project Settings › The game's flow | Loot on the ground is not saved |
| L12.2 | Graphics and performance | Window › Graphics and performance | – |
| L12.3 | Watch performance in play | Debug and testing (F3) › Performance monitor | – |
| L12.4 | Measure the finished game | World › Measure the finished game… | – |
| L12.5 | Export the game | World › Export the game… | No exported game was ever played by hand; profiles are not copied into it |

Depends on everything. L12.5 depends on L1.6 and L1.7: what a game's files are.

## 4. The benchmark game (proposed, for your approval)

A small game that grows one phase at a time. Its only job is to make each workflow be used for real.

**Working name:** Benchmark. **Its world file:** one, in a folder of its own outside the engine's folders.

| After phase | The benchmark has |
|---|---|
| L1, L2 | One blank plot of land, saved, kept as a game, reopened, and duplicated once. |
| L3 | One player character chosen for it, with a small moveset (walk, run, jump, one attack) and buttons set, all holding after reopening. |
| L4 | A path, a few trees, one small house, a chosen look and hour. |
| L5 | One person who can be talked to. |
| L6 | One item to pick up and use; the attack does harm. |
| L7 | One quest from that person, with an ending. |
| L8 | One foe that fights back. |
| L9 | One imported model placed, and one generated thing replaced. |
| L10 | A title, a health display, a pause menu. |
| L11 | One sound, one effect, one short cutscene. |
| L12 | A saved game that loads, and an exported build that runs. |

Nothing goes into it that the editor's windows could not make. A part that cannot be made is a finding against the engine.

## 5. Places where two windows do one job

Found while listing. Each is a place a maker can be unsure which one decides. They are not faults by themselves; each is checked when its phase is investigated.

- **Buttons:** the Animator's move box, Profile & buttons, Project › a character › Buttons, Project › Controls & Input.
- **Combos:** the Animator, Project › Moves & Combat, the Combo graph.
- **What a character carries:** the Studio's Gear, the profile, the Animator.
- **New worlds:** New world, the Open-world generator, Begin.
- **Designs:** the Asset Library and Objects in this world.
- **Behaviours:** the Behaviours window, What things can do, Project › Behaviours.
- **Needs:** the Needs window and Project › Needs and stats.
- **Graphics the player gets:** Graphics › Players are offered, and Project › Their settings.

## 6. Sources

- The editor's menus (`editor/godot/desk.gd`), its windows (`editor/godot/main.gd`), the Project window's categories (`editor/godot/projectview.gd`).
- `editor/godot/authorable.txt` for what is still fixed in code.
- `PROGRESS.md` (Known gaps), `Draw3D_Still_To_Do_2026-11-08.md`, `Draw3D_Gameplay_Loop_Tracker.md`, `Draw3D_Living_World_Generator_2026-11-14.md`, `Draw3D_Settlement_Plan_2026-11-12.md` for recorded limits.
- L1 to L3: the UI test of 2026-11-14 (documents D and E).
