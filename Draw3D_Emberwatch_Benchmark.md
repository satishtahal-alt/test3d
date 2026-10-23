# Draw3D Benchmark: Emberwatch — The Last Beacon

A 30-minute vertical slice authored through Draw3D's own editor interface, to measure how far a
person can make a survival-defence game of this kind with it. Begun 2026-10-22.

Companion document: [Draw3D_Emberwatch_Authoring_Workflow.md](Draw3D_Emberwatch_Authoring_Workflow.md),
the cumulative record of the interface steps actually performed.

Status words: **verified** (a scripted run or test showed it), **built, not verified**,
**not used by hand** (nobody has played it with a keyboard or a pad).

---

## Stage 1. Baseline: what the engine has for Emberwatch

Read from the code on branch `m2-stable-terrain` at commit `7ef42e8` (G5), before any
Emberwatch work. "Reachable" names the interface a person uses. "Text" means a window that
takes typed lines: F10 (*What is written*), F6 (*Object text and meshes*), or the Behaviours
window's *Bring in from words*. Those are the editor's own interface for these systems, not
code.

### 1.1 What exists and is reachable

| Need | What Draw3D has | Reachable through |
|---|---|---|
| Valley land | Land maker; raise, lower, smooth, flatten, cliffs; ground paint (dirt, rock, sand) | World > New world; toolbar **Terrain** > Terrain tools (F8) |
| Forest, fields, reeds | Forests and meadows (makers), Tree Generator, plant families, painted groups | toolbar **Plants**; right bar **Brushes** |
| Hamlet buildings | Houses drawn as outlines with roofs, doors and windows; walls and palisades drawn by dragging; paths; dug ponds; platforms; ladders; stairs | right-hand construction bar (7cu–7df) |
| Props | 329 library designs: barricade boards, barrier of planks, torch, workbench, lamp posts, campfire, beacon brazier, chests to search, crates, rubble, ruined wall, wolves, alpha, hound, people | Objects > Asset Library (Ctrl+L), click to place; Inspector (F7) to move, turn, raise |
| Things of this world | A kind `like` a library design, saying what differs | F6, or the Inspector's rows for one thing |
| Time of day | World clock, a style's light by the hour, `dark` fact, lamps `when dark`; the Player start's `hour` | Weather window (toolbar); Inspector at the start flag |
| Weather | Weather is the world fact `weather`; weathers chain by themselves (`lasts HOURS then NAME`); rain, wind, gusts, falling, wet ground | Weather window; F10 > weather |
| Survivors' days | `goes HOUR at PLACE`, `goes HOUR do WORK`, `follows`, `runs`, `tends` | F6 / Inspector |
| Behaviours | States, events (`on hour`, `on fact`, `on hears`, `on sees`, `on told`), steps (`go`, `hurry`, `patrol`, `follow`, `flee`, `put`, `tell`, `say`, `sound`, `act`) | Window > Behaviours and rules (Ctrl+B): tree editor, or *Bring in from words* |
| A behaviour switched by a condition | `while WORD CONDITION` for follows, roams, hunts, preys, attacks, flees… | F6 / Inspector |
| When this happens, do this | `on TRIGGER do ACTION` (an act done, a fact, an hour) | Inspector "What it can do" |
| Wolves | See, hear, hunt, search where last noticed, call the pack (`calls`), keep ground (`defends`), prey on others (`preys`), flee badly hurt (`flees`); the alpha | creatures in the library; F6 / Inspector |
| Attacks on defences | `barricade N`: battered by those that hunt or search with it in the way; broken, it lies there; `board_up` puts it up again with scrap | library designs; act `board_up` |
| Combat | Walker health, attack button, dodge, lock-on, weapons drawn (×2), takedown from behind, creature health and `attacks` | built in; gear from the library |
| Survival | Stats (hunger, thirst, fatigue) and statuses written as text | F10 > stats |
| Items | Goods (scrap, wood, food, medicine, lamp oil, lantern, barrier), items with ids, inventory (I), searching (acts `search…`) | F10 > goods; library chests |
| Crafting | Recipes at a bench (`craft` act): torch, lockpick, crate, barrier of planks; a barrier is put down from the inventory in play | F10 > recipes; library workbench |
| Quests and talk | Quests with steps, hints, markers, rewards; talks with answers that take quests and put facts; journal (J) | F10 > quests, talks |
| Endings | The Player start's `ends` (a condition) and `ending` (words): one ending screen | Inspector at the start flag |
| Climbing | `climbable` + wall climb; ladders as ways; vault and wall run moves | F6; construction bar ladders |
| Cutaway | Rooms seen cut open, side-on eye | F6 `cutaway`, eyes |
| Save and load | Saved games (F5/F9) of facts by stable id; autosave; things placed in play, felled, made, kept | built in |
| Playtesting | Play (Tab), Playtest window (P) with facts and switches | bar |

### 1.2 Partial, or missing, for this game

Each of these is checked when the game needs it. What is built for it is general, and reachable
from the editor.

| # | Needed | Today | Plan |
|---|---|---|---|
| B1 | A 30-minute evening-to-dawn night | An hour of the world always lasts a minute (`clock.gd` `DAY_S`). 16:30 to 06:00 takes 13.5 minutes | A Player start setting, *a game hour lasts (minutes)*, in the Inspector and World outline |
| B2 | Several endings, chosen by what happened | One `ending` text for one `ends` condition | Endings chosen by condition, each with its own title and words, set at the Player start |
| B3 | Wolves that test defences on their way in | Barricades are battered only by those that hunt or search. Those sent by a behaviour (`go`, `hurry`) walk at them and stop | A behaviour's `go`/`hurry` batters a barricade in its way, if the one going `attacks` |
| B4 | Fire and the lit beacon drive wolves off | `flees` runs from creatures and people, never from a fire | To check: `while flees …` on a condition (`fact beacon_lit`), or a new `flees` target |
| B5 | Survivors harmed by wolves, and counted | `health N NAME` counts the fallen. Whether a scheduled villager can be preyed on and fought is unverified | To check by playing |
| B6 | The ending says what came of each choice | Ending words are fixed | To check: the facts named in the ending words |
| B7 | Authoring by the interface, scripted | Tours and demos drive single tools with input events, but set worlds up through host commands. Nothing authors a whole world through the interface | `author.gd`: a script that does only what a person does (clicks the bar, the toolbar and windows, types into fields, drags in the view) and records each step it performed |

### 1.3 How "through the interface" is done here

The author is a script, `editor/godot/author.gd`, run by `editor\run.ps1 -Author emberwatch`.
It may only do what a person at the editor does:

- press buttons on the bar, the toolbar, the construction bar and in windows, found by the words
  they show, by a left click sent through Godot's input (`Input.parse_input_event`), as the mouse
  sends it. The real pointer is never moved (the standing rule on the user's input);
- type into fields and text windows, key by key, as the keyboard sends;
- click and drag in the view at places on the land, as the mouse would;
- read what a person can see: window text, the context row, notices.

It never calls the host (`ed.run`), never writes a file, and never sets a variable of the
editor's. Each step it performs is written to `editor/godot/author-emberwatch.log` as the menu
path a person would follow. The workflow document is compiled from that log, so it records only
steps that were performed. Where the interface could not do something, the script stops and
says so. That is how the gaps in this document were found.

---

## Stage 2. The smallest complete route

Built and played: hamlet → gather and prepare → beacon → wolf attack → ending. The world is
`editor/emberwatch.json`, made from an empty island by `editor\run.ps1 -Author emberwatch`.
Every stage went through the interface, and each step is recorded in
`editor/godot/author-emberwatch.log`. It is played by `editor\run.ps1 -Demo emberwatch`, which
uses the walker's own controls: walking, E for every act, the inventory, F5/F9.

| Step of the route | How it was made (interface) | Verified by `-Demo emberwatch` |
|---|---|---|
| Arrive at 16:30 on the south road | Characters › Place the game start…; Inspector: begins at 16.5, an hour lasts 2.2 min | the clock reads 16:30, `hour_lasts` is 2.2, Wren is the dressed ranger |
| Mara gives The Last Beacon | `talks` and `quests` texts (F10); Mara a kind in Object text, put down | her talk opens with E; two answers take the quest; its step shows top right |
| Bram's Broken Gate | as above, plus the `starts_broken` behaviour (Behaviours › Import from text…) | the three gaps start broken; his talk takes the quest; it is done once the gaps are mended, and gives a kit |
| Gather supplies | crates and the woodpile as kinds with `stock` | scrap 3, wood 8, oil 1, food 2, cloth 1 searched out |
| Mend the gaps | the act `mend` (`acts` text); gap kinds with `barricade 10 gaps_broken` | the west breach and the north gate mended: their facts set, standing again |
| Craft and place a barricade kit | `recipes` text; Bram's bench | made at the bench, put down from the inventory across the mill gate |
| Save, then load | F5/F9 (built in) | the striker taken after the save is gone again after F9; mended gaps, quests and Wren's place kept |
| Find Pip at the mill | Pip a kind with `follows 3`, `while follows fact pip_found`; the `call_dog` act | called, Pip follows at 2.8 m |
| Climb the ridge | land raised, levelled and smoothed with the Terrain tools; a dirt trail painted | the ridge path walked to the beacon top (23.6 m up); `beacon_seen` set |
| Repair and light the beacon | the `old_beacon` kind: `offers repair_beacon`, `light_beacon`; `on act … do put …`; the fuel valve | valve opened; repaired (3 scrap and the striker); lit (2 oil): `beacon_lit` |
| The night begins | the `night_watch` behaviour on the bell: `on fact beacon_lit` → night (assault, wave 1, storm, an hour of 1.1 min) | assault, wave 1, storm and pace all set the moment the beacon burns |
| The wolves come | the `ashfang_raid` behaviour (run, gate); wolves `restless` | the first run comes down the storm road to the north gate and batters it: 10 blows, **no wolf through it while it stood**, and it gave way after 18 to 35 s (two runs) |
| Dawn, and an ending | the `endings` text; the start's `the game ends when` set to `fact dawn` | clock turned to six; the ending chosen by condition: **Dawnkeeper** |

**The unprepared night** (`editor\run.ps1 -Demo emberdark`) plays the same world without
mending anything or going to the beacon. At midnight the night_watch starts the assault anyway.
The wolves come in through the broken gaps and go for Wren when they see her, and dawn ends as
**Cold Valley**. Preparation is what tells the two endings apart: mended gates hold the raid
out, and the lit beacon gives Dawnkeeper.

## Stage 3. The connected systems

| System | State | Evidence |
|---|---|---|
| NPC schedules | Built: survivors `goes 7 at the square` and `goes 22 at the square`; Tomas and Pip by behaviours | the demos' running lists show each going and arriving; in the storm the survivors shelter in their houses, opening and shutting their doors (in the log) |
| Construction-based defences | Built and verified for the gates: mend (wood and scrap), board up, a crafted kit put down; a gate battered down counts `gaps_broken` | `-Demo emberwatch`: the mended north gate held for 10 blows |
| Weather | Built: overcast at 18:00, rain at 20:00, storm at 22:00 and at night, by the night_watch (`put weather …`) | the weather fact read `storm` at the beacon and at midnight; rain drawn |
| Time pressure | Built: an hour lasts 2.2 minutes (an evening of about 17 minutes), then 1.1 minutes from the assault | `hour_lasts` read 2.2, then 1.1 |
| Optional tasks | The Broken Gate verified end to end. The Boy's Dog: Pip found and following verified, Finn's hand-in not played. Hunter's Cache and Farmer's Choice written and placed, **not played** | quest stages in the demo |
| Persistent consequences | Verified: F5/F9 keep mended gaps, quests and where Wren stood; a fact set after the save is undone by loading | `-Demo emberwatch`, the "saved and loaded" beat |
| Endings by what happened | Two built and verified: Dawnkeeper (prepared, lit) and Cold Valley (unlit). Embers in the Dark and The Hamlet Fell are written, **not reached by a demo** | the two demos' dawn beats |
| Survivors harmed | **Not seen.** In the storm every survivor sheltered indoors, and wolves open no doors; the wolves only reached Wren | `-Demo emberdark`: `lost` stayed 0 |
| Fire drives wolves off | Written as `flees 7 you while fact beacon_lit`; **not checked** in play | none |

## Stage 4. Evaluation

### 4.1 Authoring effort

| Measure | Emberwatch |
|---|---|
| Interface steps performed (one world, from an empty island) | 654 in the last run, scripted at about 3 a second (200 s); the log lists each |
| Of those, in the view (clicks and drags on the land) | about 230 |
| Asset Library steps (find, bring in, place) | about 110 |
| Text typed into the editor's text windows | about 21,700 characters, about 760 lines: kinds 245, behaviours 130, talks 110, acts 75, quests 65, stats 50, endings 30, goods 25, recipes 15 |
| Stages | 8: world, land, hamlet, rules, kinds, places (with the gate turns), nature, start |

A person doing the same by hand would spend most of the time on the typed texts (kinds and
behaviours) and on placing about 60 things, one click each. Steering the camera takes most of
the rest, because each placement needs a view of the spot. Nothing needed a code editor: all
the game content is typed into the editor's own text windows and checked there when applied.

### 4.2 What was hard through the interface

Each of these came up while authoring, and each is in the workflow document where it happened:

- A house's door goes in its first side, which can't be chosen while dragging.
- There is no "raise the land to H metres". You stroke and read the height back.
- Nothing shows a slope as too steep to walk. The first beacon top had a rim the walker could not climb.
- Quests, talks, acts, goods, recipes and stats have no form editor. They are typed, and checked only when applied.
- A kind written `like wolf` is refused until the library's wolf has been brought into the world, and the message doesn't say so.
- **`Put one down` turns each thing differently.** The gates stood askew and the wolves walked round them. To turn one, you find `>  Placement` folded below a long Inspector. The Inspector names the thing by its kind ("Gap mill"), not by what it is called ("the mill gate"). From above, thin boards are too narrow to click.
- A selected path or paving shows its settings panel over the view, and the panel takes the clicks meant for the land. Deselect first.
- An editor started by a script never opens the world it is given (`-World`).
- Applying a text used to write the project's shared file too. This is fixed with a checkbox (9.6 in the workflow document).

### 4.3 Visual quality

From the demos' pictures (`editor/godot/demo/*ember*`), not judged by a person:

- **By day the valley reads well.** The arrival shows the path up to the palisaded hamlet, the houses, the windmill and the woods; the Golden valley style suits the evening.
- **The lit beacon is the strongest picture:** flame, glow, rain and the storm sky.
- **At night, away from lamps, nothing can be seen.** The first assault pictures were black at the north gate. The lamps in the square do light it in pools. The fix was authoring, not engine work: lamp posts at each gate, from the library. Between lights the storm night is still very dark (moonlight 0.025).
- The ruins' crates show as plain red boxes with a "?" marker. The ridge is rough and low-poly, and the mountains read as low hills.

### 4.4 Playability

Both routes play through with the walker's own controls. The main route takes about 8
minutes of play in the demo, with the clock moved on twice (to the assault, and to dawn).
Played unhurried, it would be the evening's 17 minutes (16:30 to midnight at 2.2 minutes an
hour), or less if the beacon is lit early, then the night at 1.1 minutes an hour (6.6 minutes
to dawn): about 25 to 30 minutes, the target. Untested by hand: the controller, how the night
feels, whether the beacon is found without the quest marker, and how fighting a pack feels.

### 4.5 Integration

These had to work together, and did: talks put facts → quests advance → acts need goods and
facts → behaviours hear facts (`on fact beacon_lit`) → the night_watch sets the weather, the
pace and the waves → the raiders' behaviours hear the wave → the fight and barricade systems →
the end condition → endings chosen by condition. Every joint is a world fact, so a saved game
keeps all of it.

### 4.6 Performance

The demos' frame counter showed 57 to 90 fps in play, with worst frames of 11 to 23 ms in
the hamlet and outliers of 60 to 150 ms when a world opens or the clock is turned. Not
measured: a whole night played through with all three waves.

### 4.7 Automated evidence

See the closing table in section 4.9.

### 4.8 Known limitations

- **Survivors are never harmed.** They shelter indoors in the storm, and wolves open no doors. The Hamlet Fell ending (`fact lost >= 3`) can't be reached as the world stands.
- Embers in the Dark, the Hunter's Cache, the Farmer's Choice and Finn's hand-in are written but not played by any demo.
- Waves 2 and 3 (the west breach and the mill) come 150 s and 300 s after the first. The demos move the clock past them, so they have never been watched.
- The kit's boards are put down where Wren faces, and nothing fits them to a gap. A wolf went past them at the mill.
- Lamps are lit only within 3 cells (about 100 m) of the camera (unchanged).
- The demos move the clock (to midnight and to dawn) instead of playing the night through.
- No one has played Emberwatch by hand.

### 4.9 Automated evidence (final runs, 2026-10-23)

| Check | Result |
|---|---|
| `run.ps1 -Author emberwatch -Check` | the whole world made through the interface from an empty island: 654 steps, 200 s, problems: none |
| `run.ps1 -Author emberwatch -Only gates` | the three gates selected from the side and turned in the Inspector; the turns are kept in the file as changes to each thing |
| `run.ps1 -Demo emberwatch -Check` (13 beats) | all expectations met; the north gate took 10 blows and let no wolf through while it stood; Dawnkeeper |
| `run.ps1 -Demo emberdark -Check` (4 beats) | all expectations met in each of the last two runs (wolves inside for about 250 wolf-seconds, Wren's health 5 → 0, Cold Valley). The raid itself came in each of the last five runs. Before the raiders were made wherever they stand, it failed to come in 3 runs of 6 |
| `cargo test --release -- --test-threads=1` (host) | 189 passed |
| Regression demos for what changed: fights, shelter, bell, regions, remembers, perceives, inspect | all pass (run before the last `folk.gd` change, which touches only `restless` things) |
| `-Demo guard` | fails on "goes back to its round", and fails the same way on the committed code, so it predates this work (Known gaps in PROGRESS.md) |
| Not run | the whole tour; `-Tour -Only ui` (the checklist's text changed; its `auto` line was kept short) |
