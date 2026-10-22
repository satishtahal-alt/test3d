# Draw3D Benchmark: The Forgotten Watchtower

A small, complete wilderness adventure made with Draw3D's own authoring systems, to
measure what the engine can make today and to expose what it cannot. Built 2026-10-18
(amendment 7ch). The level is `editor/genres/watchtower.json`, offered as a genre in
**World > New world**; it is built by `editor/host/examples/watchtower.rs`, which says to
the host only what a person says to it from the editor (the same commands the editor
sends), so every part of it is a maker, a design, a thing or a text that can be clicked,
inspected and changed like anything else.

**Second pass, the same day (amendment 7cj).** The user played the first version with a
controller and judged it not good enough for a commercial demo: an unclothed mannequin; an
axe that had to be put in hand from the inventory; old side-steps on the shoulder
buttons and no button to swing the axe; a pause menu a controller could not work; wolves
far too fast; a tower made of house pieces; no life besides the wolves; no style; health
drawn as rows of # signs. Each was a fair finding, and the first validation had not
looked for any of them: it counted beats done (quest taken, wolves felled, beacon lit)
and never judged what a player sees or presses. This pass mends every one (part D, G22 to
G33) and part E now has a **polish bar**, judged by looking at each screenshot, not only
by counting.

Status words used below: **verified** means a scripted play-through or a test showed it
working (part E says which); **built, not verified** means it exists and was not
exercised; **not used by hand** means nobody has played it with a keyboard or a pad yet.

---

## A. Overview

### Concept and intended experience

A wooded valley at late afternoon. A trail winds north from a trailhead to a tower on a
hill whose beacon has been dark for twenty winters. The ranger at the trailhead asks you
to light it. You follow the trail through a painted conifer forest, find lamp oil, an axe
and a backpack at the woodcutters' abandoned camp, get past the wolves that hold the
clearing under the hill, climb the tower, and kindle the beacon. The game then ends on an
ending screen. About five to ten minutes on a first play.

### Locations and sequence

| # | Location | Where (x, z; north is -z) | What happens there |
|---|---|---|---|
| 1 | **Trailhead** | (0, 46) | Where a game starts, looking north up the valley at the tower on its hill. The ranger (talk: E) gives the quest. A signpost, a waymark lantern (checkpoint), a log and a boulder. |
| 2 | **The trail** | (0, 44) to (0, -72) | A dirt path painted on the land, winding north between eight woods of spruce, pine and birch; ferns, foxgloves, bluebells and blueberries under the trees and grass along the edges; fallen logs, mossy boulders, outcrops, toadstools, a cairn. |
| 3 | **The old camp** | (-22, 2), by a side path west at (3, 5) | A cold fire ringed with logs, a tarp over a bedroll, crates, a dead tree, and the camp chest (E: lamp oil, a tin of food, medicine). The woodcutters' axe and a backpack lie by the fire (E: taken; I: worn and held; B: draw). |
| 4 | **The wolves** | clearing (1, -50); up the hill (1, -62) | Past a second waymark lantern (checkpoint) at (7, -25): two wolves in the clearing, and a wolf and the grey alpha up the hill. Each keeps 11 m of ground, calls the others within 12 m, hunts you inside its ground and lets you go outside it. |
| 5 | **The watchtower** | hilltop (0, -80), 6.5 m above the valley | A stone tower of two storeys on four cells, entered by an archway on the south; stairs inside to a lookout open to the sky behind a crenellated parapet. |
| 6 | **The beacon** | on the lookout | E: kindle (uses the lamp oil). It burns, the quest's last step is done, the ranger's reward is given, and the game ends. |
| 7 | **Ending** | anywhere | The ending screen: the start's ending words, how long it took; keep exploring, play again from the start, or back to editing (in an exported game: leave). |

### Visual direction and key design decisions

- **Style:** `evergreen` (styles.txt): a painted conifer forest, blue shade and gold
  light; the game starts at 16:30, so the light is warm and long.
- **One focal point:** the tower on its raised hill, on the axis of the trail from the
  trailhead; the woods frame the trail, which bends so the view opens and closes.
- **Trail readable:** the path is painted dirt (nothing grows on it), with grass and
  flowers only along its edges; trees within 3.2 m of it are taken away by hand (each
  kept away by name, listed under what was changed in World outline); the clearings
  (trailhead, camp, wolves, hilltop) are painted bare.
- **Distinct places:** the trailhead (signpost, lantern, ranger), the camp (tarp, fire
  ring, logs, crates, dead tree), the clearing (boulders, standing stones, a dead snag),
  the hilltop (grey stone tower, cairn, outcrop).
- **Wolves met a pair at a time:** the library's wolves keep 22 m and call from 24 m; the
  valley's are kinds of this world like them that keep 11 m and call from 12, so the
  lanterns (where you come back when you fall) and the tower's archway are outside their
  ground. This was a finding of the first play-through (part D, G6).
- **No survival meters:** the level carries its own `stats` text (as F10 writes a text
  in a world) with no hunger, thirst or tiredness, only bleeding and rest.
- **A stone tower from a house:** a building on cells has storeys and stairs; each of
  its wall pieces is given a design of its own (the library's new tower wall, doorway,
  corner and parapet), which is how a person changes one thing's design.

### Controls (a controller first; the keyboard as well)

A controller is the way this level is meant to be played (the user's rule, 2026-10-18):
with one plugged in, every prompt and hint shows its buttons from the start.

| Controller | Keyboard and mouse | What it does here |
|---|---|---|
| Left stick | W A S D | Walk (the way the eye looks) |
| Right stick | Mouse | Look about |
| RB (held) | Shift | Run |
| A | Space | Jump |
| X or RT | Left click | Swing what is in hand (the axe hews; with nothing, the fist) |
| B | F | Dash; with the stick aside or back, dodge out of a bite |
| Y | E | Talk, search, take, kindle (whatever the prompt at the bottom says) |
| LB | Middle click, Y | Lock on to the wolf before you, and off again |
| LT | C | Crouch |
| Right stick pressed | B | Put the axe away (a swing draws it again) |
| D-pad up / down | Wheel | The eye nearer, farther |
| D-pad right / left | Q / R | The next thing kept into the hand / use it |
| Back | I | Inventory |
| Start | Esc | The menu: go on, inventory, journal, controls, save, load, leave |
| In the menus: D-pad or left stick, A, B | Arrows, Enter, Esc | Move, choose, go back |
| | Tab | (In the editor) back to editing |

A person's own buttons (the Character Studio's Buttons tab, kept in
`%APPDATA%\Draw3D\controls.txt`) are laid over these; part E says what the user's own file
changes.

### Completion condition

The Player start says `ends quest watchtower_beacon done`. The quest's last step is done
when the world's fact `beacon_lit` holds, which the beacon puts when it is kindled. The
ending screen then holds the game; the world's fact `ended` is put so a saved game knows.

---

## B. Systems used

How the level is made, system by system, and how the systems meet. Every name in
`code` is a word a person writes or a setting a person changes; nothing in the level is a
script of its own.

| System | What it contributes | Where it lives | How it connects |
|---|---|---|---|
| **Terrain** (land maker, land tools) | Level valley floor (`flatten`), wooded rises east and west (`raise`), the tower's hill 6.5 m high with a level top, the trail's slope smoothed (`smooth`) | The `island` maker; Terrain (F8) | The ground every maker stands things on; `bare` ground is what woods and painted groups keep off |
| **Ground paint** | The trail and the camp's side path as dirt, clearings as worn ground; `details` on the trail, so nothing grows on it | Terrain > paint (dirt) | Woods and brushes do not grow on dirt or ground made bare |
| **Generator presets** (Tree Generator, plants) | Designs grown from numbers: `valley-spruce`, `valley-birch`, `dead-tree`, `lady-ferns`, `ostrich-ferns`, `bracken`, `tussock`, `meadow-grass`, `foxgloves`, `bluebells`, `blueberries` | Plants > Tree Generator (and the families' presets) | Each is a design of the world, named in woods' `of` and in groups |
| **Woods** (makers) | Eight woods along the trail, each given its own mix (`of`) of the grown trees and the library's `pine-tall` and `spruce-layered`, varied in colour (`vary`) | World outline (Ctrl+M) | A tree that stood within 3.2 m of the trail was deleted by hand: kept away by name, listed under what was changed |
| **Groups and the brush** | `forest-floor` (ferns, tussock, foxgloves, bluebells) painted in the woods; `trail-edge` (grass, flowers, blueberries) along the trail | Objects in this world (group), Paint | Painted things are each a thing of a "painted" maker, erasable by brush |
| **Library designs** | Logs, boulders, outcrops, toadstools, cairn, standing stones, dead snag, signpost, crates, campfire, chest, table, the axe and backpack to find, the ranger, wolves, the beacon, lanterns, the tower's stone pieces | Objects > Asset Library (Ctrl+L) | Taken into the world (`library_take`) and put down; nine designs added to the library this pass (part D) |
| **Structure on cells** (L3, 7bv) | The tower: a building on a 2×2 layout of 4 m cells, two storeys (`storeys 2`), no roof (`roof none`), boards on its floors, a stairwell (`stair c0_0`) with a flight to the lookout, an opening in its south side (a gate) | Build > A house… on a Grid layout (Ctrl+G); its settings in World outline | Each wall piece given a design of its own (World outline: one thing's "its design"): tower wall, doorway, corner, parapet |
| **Kinds of this world** (`like`) | `camp_chest` (like the library chest: `stock lamp_oil 1`, food, medicine, `no perhaps`), `valley_wolf` and `valley_alpha` (like the library's, `defends 11`, `calls 12`, `sees 12 150`; the wolves bite every 1.6 s) | Object text and meshes (F6), or the Inspector's rows | A kind says only what differs; the rest is the library design's |
| **The player** | The scout (Quaternius's model, `walks default`): walk, run, jump, dash, dodge, strike, search, reach, stoop; the axe held in the right hand and drawn (B), the backpack worn | characters.txt, gear.txt; the inventory (I) | Gear is found as items (`found-axe`, `found-backpack`), kept, then worn or held through `items.gd` |
| **Creatures and fights** (`watch.gd`, `fight.gd`) | Wolves that see (12 m), keep their ground, call each other, hunt and bite; the walker's health (5), blows that count two with a weapon drawn, a dash that steps out of a bite; falling sends you back to the last lantern passed | The Inspector's "How it sees, hears and goes" and its enemy part | `wolves_down`, `alphas_down` are world facts; a lantern says `checkpoint` |
| **Items, goods, inventory** | Lamp oil, food and medicine as goods; the axe and backpack as items; the inventory's rows and buttons; the HUD's goods and items | goods.txt (new good `lamp_oil`), gear.txt, Characters > Game display | Goods are counted facts; the quest and the act go by them |
| **Acts** (`acts.txt`) | `search` at the chest (takes everything), `pick_up`/take for the items, `talk` with the ranger, and the new `kindle` (hands one lamp oil to the thing, puts `kindled`, heard 10 m off) | F10 > acts | What a thing `offers` at a `point` |
| **When this happens, do this** (7cg) | The beacon says `on act kindle do put beacon_lit`: the act's moment puts the world's fact | The Inspector's "What it can do" | A world fact is what lights, quests and the ending go by |
| **Lights and sounds** | The beacon `lights 1 0.6 0.25 40 0 1.7 0 glow 0.45` `when beacon_lit`, `sounds kindled` once, `hums fire 18` while lit; the lanterns light their surroundings | sounds.txt (new `fire`, `kindled`); Window > Sounds | `when` takes any condition of `cond.gd` |
| **Talk and quest** (`talks.txt`, `quests.txt`) | The ranger's talk `ranger_beacon` (answers, a branch for the story, the quest taken by "I will light it"); the quest `watchtower_beacon`: two steps (`done has 1 lamp_oil`, `done fact beacon_lit`), each marked in the world (`at the camp chest`, `at the beacon`), hints, a reward of 5 coins | F10 > talks, quests; the Journal (J) | The HUD shows the step and its hint; the journal lists it |
| **The ending** (new this pass) | The Player start's `ends` (a condition) and `ending` (words): the ending screen | Inspector at the start flag, or World outline's Player start | Any condition: a quest done, a fact, goods had |
| **A world's own text** | The level's `stats` text: no hunger, thirst or tiredness | F10 in the world (here written by the builder) | Opened, a world keeps its own texts unless the project's file of that name is saved while it is open (part D, G9) |
| **Style, light, time** | `evergreen`; the game begins at 16:30 (the Player start's `hour`, new) | Sky (Environment), Window > Styles | Trees, grass, rocks and bark follow the style's palette |
| **Genre and export** | Listed in World > New world as "The Forgotten Watchtower"; exportable as any world (World > Export the game…) | genres.txt, genres.ps1 | Exported (`export.ps1 -World genres\watchtower.json -Name Watchtower`) and started at the trailhead; not played through as a game (part E) |

---

## C. Recreation guide: how to rebuild the level

How a person makes this level again in the editor, from an empty world to a game that
ends. It follows the order the builder (`watchtower.rs`) does it in, which is the order
the work depends on. It does not list every object placed: the positions are in the
builder and in the table of part A. Menu and window names are the editor's own (see
`Draw3D_UI_Inventory.md`); where a step was only done by the builder's commands and not
by clicking, it says so.

To look at the finished level while following this: **World > New world**, "Genre: The
Forgotten Watchtower" opens a copy of it with no file.

### Stage 1. The land: a valley, its rises and the tower's hill

- **Goal:** level ground for the valley floor, low wooded rises east and west, and a hill
  6.5 m high in the north with a flat top about 18 m across.
- **Where:** **World > New world** (Ctrl+N), "an island" (land and nothing else). Then
  **Terrain** on the toolbar > *Terrain tools* (F8).
- **Setup:** `flatten` over the whole valley (about 100 by 170 m) at one height, strength
  1, soft 8. `raise` in strokes along x = ±40 (three passes of about a metre each). For
  the hill, `raise` about (0, -80) with a brush about 17 m, gently (overlapping dabs add
  up), until it is about 6.5 m high; `flatten` its top (radius 10) to the height it
  reached, never higher (a top raised by flatten stands as a step at its rim); then grade
  the way up: `flatten` with a 3 m brush along the trail from (0, -50) to (0, -73), each
  dab a little higher than the last, and `smooth` over it. Walk it: nowhere should it be
  steeper than about 25 degrees (part D, G17).
- **Expected result:** a long green valley with gentle sides and a round hill closing it
  in the north, its top level, a way up it the walker can walk.
- **Dependencies:** none.

### Stage 2. The trail and the clearings

- **Goal:** a winding dirt path from the trailhead to the hilltop, a side path west to
  the camp, and worn clearings where things happen.
- **Where:** **Paint** > *Ground: grass, sand, rock or dirt under the brush* (F8, paint).
- **Setup:** paint `dirt`, brush about 1.7 m, soft 0.9, with "what grows goes by it" on
  (so nothing grows on the path), along the trail's bends (part A's table, row 2), and
  1.3 m along the side path. At the trailhead, the camp and the clearing paint dirt at
  half strength with a wide soft edge, and use Terrain's *clear* (`bare`) over them.
- **Expected result:** a brown path that bends north and climbs the hill, and three worn
  open places, with grass everywhere else.
- **Dependencies:** stage 1.

### Stage 3. The look: style and hour

- **Goal:** one painted look for everything: a conifer forest in warm late light.
- **Where:** **Sky** on the toolbar (Environment) > style `evergreen`; **Window > Styles**
  to read what it says. The hour a game begins at is the Player start's (stage 9).
- **Setup:** style `evergreen`. Nothing else: trees, grass, rock and bark take their
  colours from the style.
- **Expected result:** blue-green grass and foliage, warm light.
- **Dependencies:** none (it can be given at any time; it changes what woods are made of
  only where a wood says no designs of its own).

### Stage 4. The woods, the forest floor and what lies about

- **Goal:** woods that frame the trail and never stand on it, a forest floor under them,
  flowers along the path, and details that make places of it.
- **Where:** **Plants** > *Tree Generator* (presets spruce, birch, bare tree: each
  "Save as an object" under a name of its own); the plant presets the same way (lady
  fern, ostrich fern, bracken, tussock, meadow grass, foxglove, bluebell, blueberry);
  **Objects** > *Objects in this world* to make two groups; **Plants** > *Add a forest*
  eight times; **World outline** (Ctrl+M) for each forest's settings; **Paint** > *Paint
  with the group*; **Objects** > *Asset Library* (Ctrl+L) for the rest.
- **Setup:**
  - Designs: `valley-spruce`, `valley-birch`, `dead-tree` (the Tree Generator), and the
    eight plants. (The Generator's pine preset was tried and left out: its flat crown
    reads as a palm from a distance; the library's Tall pine is used instead, G20.)
  - Groups: `forest-floor` = lady-ferns×4 (0.8 to 1.3), ostrich-ferns×2, bracken×2,
    tussock×2, foxgloves, bluebells×2, varied "foliage hue 0.03 light 0.15";
    `trail-edge` = meadow-grass×4, tussock×2, foxgloves, bluebells, blueberries, varied
    with blooms among bloom1 to bloom3.
  - Eight forests either side of the trail (part A's places), reach 11 to 16 m, 24 to 34
    trees each; in World outline each one's designs (`of`), e.g. "valley-spruce*3,
    pine-tall*2, spruce-layered*2, valley-birch", and "how near the road" (`keep_off`)
    0; each varied "foliage hue 0.03 light 0.14".
  - Any tree that came to stand within about 3 m of the trail: picked and Delete (its
    forest keeps it away by name; World outline lists it under what was changed by hand).
  - Paint `forest-floor` in the woods (brush 2.6 m, spacing 1.9) and `trail-edge` along
    the trail's edges (1.2 m, spacing 1.3).
  - From the Asset Library: Fallen log (new, Woodland), Mossy boulder, Outcrop, Toadstools,
    Cairn, Standing stone, Dead snag. Put a few beside the trail, never on it.
- **Expected result:** a dense, varied conifer wood either side of a clear path; ferns and
  flowers under the trees; logs and rocks at the bends.
- **Dependencies:** stages 1 and 2 (painted ground is what keeps the woods off the
  path); stage 3 for the colours.

### Stage 5. The trailhead and the camp

- **Goal:** a start that shows the destination, and an abandoned camp with supplies.
- **Where:** **Objects** > *Asset Library*; **Object text and meshes** (F6) to write a
  kind of this world; **Characters** > "Where a game starts, in the Inspector" (the
  green flag).
- **Setup:**
  - Trailhead: the Player start at (0, 46) facing north (the flag, moved and turned in
    the Inspector); the Old ranger (People), a Signpost, a Waymark lantern (new,
    Woodland: it says `checkpoint`), a log and a boulder.
  - Camp: Campfire, two Fallen logs round it, a Tarp shelter (new), a Crates stack, a
    Dead snag; Axe and Backpack from Gear to find, by the fire.
  - The camp chest, a kind of this world written in F6:
    `kind camp_chest like chest-search` / `called the camp chest` / `stock lamp_oil 1` /
    `stock food 1` / `stock medicine 1` / `no perhaps`. Put it under the tarp.
- **Expected result:** from the flag, the trail runs north to the hill with the tower on
  it; a ranger with a "!" over them; a camp in a clearing west of the trail.
- **Dependencies:** stages 2 and 4. The lamp oil good (stage 8) must exist for
  `stock lamp_oil` to be understood.

### Stage 6. The watchtower

- **Goal:** a stone tower on the hilltop that reads as a watchtower from the trailhead,
  can be gone into, and climbed to a battlemented top.
- **Where:** **Objects** > *Asset Library* (Ctrl+L), the Woodland shelf, "Stone
  watchtower"; the Inspector for the beacon's raise.
- **Setup:**
  - The library's **Stone watchtower** (`watchtower-keep`) put down at the middle of
    the hilltop, turned so its archway (its +z side) faces the trail (turn 0 here).
  - Inside, in the room: the library's Crates and Chest (search).
  - The **Beacon brazier** on the top floor: put down at the tower's middle, 1.5 m back,
    and raised 8.02 m (Inspector: raise). Nothing else up there offers an act (what is
    nearer is offered first).
- **Expected result:** a square keep of pale dressed stone, 6.8 m across and 9.7 m to the
  top of its merlons: corner stones, two darker bands, arrow slits, an archway with a
  keystone, a corbelled parapet; a stone stair climbing round the outside of the walls
  from the south-east corner, a landing at each corner and a breast-high wall along it,
  forty steps to a gap in the battlements; the top floor whole, the dark beacon there.
- **Dependencies:** stage 1 (the hilltop level). The design is one thing: its walls,
  stair and top are parts of it; its solid boxes and hulls are what is stood on and
  walked into (each flight one slanted slab under its seen steps). It is written by
  `editor/library/watchtower-keep.js` from a handful of numbers (how wide, how thick, how
  high, the stair's width, a step's rise): run it with node after a number is changed;
  by hand it is the Object text window (F6). Why not the Build tools, and why the stair
  is outside: part D, G31.

### Stage 7. The wolves

- **Goal:** an encounter that is readable and fair: two pairs of wolves met one at a
  time, which do not follow you to a checkpoint or into the tower.
- **Where:** **Object text and meshes** (F6), or the Inspector's "How it sees, hears and
  goes" on one wolf put down; **Objects** > *Asset Library* (Animals that hunt).
- **Setup:**
  - `kind valley_wolf like wolf` with `defends 11`, `calls 12`, `sees 12 150`, `attacks bite 1.4 1 1.6` (a bite every 1.6 s: G7);
    `kind valley_alpha like wolf-alpha` (new: The grey alpha) with the same three.
  - Two valley wolves in the clearing (about (5, -46) and (-3, -48)); a valley wolf and
    the alpha up the hill (about (5, -61) and (0, -62)).
  - A Waymark lantern at (7, -25), outside the wolves' ground: falling, you are back here.
  - The library's deer (five, east of the trail and west of the clearing), rabbits
    (seven, along the trail) and a fox by the camp: neutral, they graze and run from you
    and from the wolves. Four dead trees of this world that sing (`kind singing_snag like
    dead-snag`, `hums birds 22`) along the trail.
- **Expected result:** walking up the trail, the first pair sees you and comes; beaten
  (or left behind beyond their ground), the second pair waits further up the hill.
- **Dependencies:** stage 5 (the lantern design); stage 2 (where the trail is).

### Stage 8. What is done, and what it leads to

- **Goal:** lamp oil to find, a beacon kindled with it, a quest that leads there, and a
  ranger who asks for it.
- **Where:** F10 (Moves, poses and acts: the texts) for goods, acts, sounds, quests and
  talks; the Inspector's "What it can do" for the beacon's line; **Window > Sounds** to
  hear the sounds.
- **Setup:**
  - goods.txt: `good lamp_oil` (called flask of lamp oil, flasks of lamp oil; looks box).
  - acts.txt: `act kindle` (says kindle, for walker, unless kindled, stands 0.7, does
    reach, touches hand.R, at 0.6, heard 10, hands lamp_oil 1, puts kindled).
  - sounds.txt: `sound fire` (a loop), `sound kindled`.
  - The beacon brazier's own lines (it comes with them; to make one of your own, write
    them in F6 or the Inspector): `stock lamp_oil 0 1`, `offers kindle front`,
    `point front 0 1.0 -0.85 from 0 -1`, `lights 1 0.6 0.25 40 0 1.7 0 glow 0.45`,
    `when beacon_lit`, `sounds kindled`, `hums fire 18`, and in "What it can do",
    "When this happens…": `on act kindle do put beacon_lit`.
  - quests.txt: `quest watchtower_beacon` from ranger; step "Find lamp oil…" `done has 1
    lamp_oil`, `at the camp chest`, a hint; step "Climb the watchtower and kindle its
    beacon" `done fact beacon_lit`, `at the beacon`, a hint; thanks, `reward 5 coin`,
    after.
  - talks.txt: `talk ranger_beacon` of ranger: a start point that says what the quest
    stage is; answers "Can I help?" (if the quest is new, to ask), "What happened to it?"
    (to story), "Goodbye."; at ask `{asks watchtower_beacon}` and "I will light it."
    (`takes watchtower_beacon`).
- **Expected result:** E beside the ranger opens the talk; taking the quest shows its
  step top right and a mark over the camp chest; the chest gives the oil and the step
  moves on to the beacon; E at the beacon kindles it, it burns, the quest is done.
- **Dependencies:** stages 5 and 6 (the things the quest names must be there and called
  so: "the camp chest", "the beacon").

### Stage 9. The game around it: start, ending, display

- **Goal:** a game that begins in the afternoon at the trailhead and ends when the beacon
  burns, without survival meters it does not use.
- **Where:** the green flag in the Inspector (Characters > "Where a game starts, in the
  Inspector"), or the Player start in World outline: "a game begins at", "the game ends
  when", "the ending says"; F10 in the world for its own `stats`; **Characters > Game
  display** for the HUD.
- **Setup:**
  - Player start: a game begins at 16.5; the game ends when `quest watchtower_beacon
    done`; the ending says "The beacon on the Forgotten Watchtower burns again…".
  - The world's own stats text (F10 > stats, written while this world is open): `notice
    8` and the statuses bleeding and rested, no `stat` lines. (Do not save the project's
    stats.txt while this world is open: see part D, G9.)
  - The display: as the engine has it (the quest step top right, goods under it, items
    bottom left).
- **Expected result:** Play (Tab) begins at the trailhead at half past four; there are no
  hunger or thirst bars; the ending screen appears the moment the beacon is lit.
- **Dependencies:** stage 8.

### Stage 10. Playtest and refine

- **Goal:** the whole sequence played, and what is wrong found.
- **Where:** Play (Tab); **Playtest** (P) for the world's facts and switches (tick
  `beacon_lit` to see the ending without climbing); **Journal** (J); the Inspector on
  any thing (Tab back to editing, click it: HOW IT WORKS).
- **Setup:** play it through; then `editor\run.ps1 -Demo watchtower` shows the scripted
  play-through (with `-Check` it leaves pictures in `editor\godot\demo`).
- **Expected result:** the sequence of part A, in five to ten minutes.
- **Dependencies:** everything above.

---

## D. Capability-gap record

Each gap is classed as the brief asks: **(1)** existing capability not yet configured
correctly; **(2)** authoring or usability limitation; **(3)** missing reusable engine
capability; **(4)** minor defect or polish; **(5)** large or unrelated, for later. Kind
says whether it is missing functionality, a bug, workflow friction or an aesthetic
matter.

| # | Intended result | Before this pass | Workaround | Done in this pass | Reusable | Left |
|---|---|---|---|---|---|---|
| **G1** (3, missing) | A clear ending when the beacon is lit | A quest's `thanks` was told for 9 s and the game went on; nothing could say "the game is over" | none | The Player start's `ends` (any cond.gd condition) and `ending` (words); an ending screen holding the game (keep exploring, play again from the start, back to editing / leave); fact `ended` put | Yes: any world, any condition (a quest done, a fact, goods had) | The ending's look is the game screens' own (no style of its own, no credits roll); not shown in an exported game by any check (part E) |
| **G2** (3, missing, small) | The game to begin at half past four, in warm light | A game always began at 9 (the Environment window's hour is a look while editing) | none | The start's `hour` | Yes | — |
| **G3** (4, polish) | The quest step readable | A step with its hint ran off the left of the screen | — | The step wraps within about a third of the screen | Yes, every quest | — |
| **G4** (4, bug; construction postponed) | A watchtower of three storeys | A building's stairs go up one cell further along each storey; in a tower two cells across the second flight was placed on the east wall, half outside (`stair#c1_0_up1` at x 3.5) | The tower is two storeys with a lookout on top | Recorded only: modular construction is postponed by the user | — | Flights that turn back (a switchback) when the row ends, or a stair that winds in one cell |
| **G5** (1 and 2: configured; friction) | A grey stone tower, not a house | A building's walls are a house's facades (half-timbering, windows, flower boxes), from the style | Each of the 20 wall pieces given a design of its own (World outline: one thing's "its design"; the host's `one_design`) from four new library designs | The designs: Stone tower wall, Stone doorway, Stone parapet, Stone tower corner | The designs, yes; the way, yes (it is the editor's own) | No "give every wall of this structure this design": a person clicks 20 pieces (2) |
| **G6** (1, configured; level design) | Wolves that are a fair, readable encounter | The library's wolves keep 22 m of ground and call from 24 m: in the first play-through the lantern (the checkpoint) and the tower's door were inside their ground, and a fallen walker came back into the pack again and again | Kinds of this world like them: `defends 11`, `calls 12`, `sees 12 150` (and a slower bite, G7); the lantern moved out of their reach; two pairs apart | As said | The way (a kind `like` another saying what differs) | Fairness judged by a script, not a person (G14) |
| **G7** (4, balance; engine-wide rule) | An alpha that can be fought | The alpha bit every 0.95 s, and a walker is spared for 0.8 s after a bite: a bitten walker was bitten again before they could step out | — | The library's grey alpha bites every 1.6 s (and from a little farther); the valley wolves (kinds of this world) every 1.6 s too, after the fifth play-through, where two wolves biting every 1.1 s downed the walker seven times | The design, yes | Any creature whose `attacks … EVERY` is under about a second can chain a walker down; nothing warns of it (the Inspector could) |
| **G8** (1, configured) | No survival meters in a game without survival | Hunger, thirst and tiredness bars showed in every world | The world's own `stats` text (as F10 writes one in a world): statuses only | As said | The way, yes | G9 |
| **G9** (2, limitation) | The level's texts its own | A world keeps its own copy of a text, but the project's file of that name, saved while the world is open, replaces it; quests, talks, acts and goods are the project's, so the level's quest sits in quests.txt beside the village's | none | Recorded | — | A clear rule (and a sign in F10) for which text a world is using, and texts that belong to a world (5) |
| **G10** (5, large; parked by the user) | A player character that suits a woodland adventure | The only humanoid models are mannequins (the scout is Quaternius's UAL model, faceless and yellow) | The axe in hand and the backpack worn for a silhouette | Recorded | — | The Character Maker's tier 2 (faces, hair, clothes) waits on the user's choice of assets |
| **G11** (2/4, aesthetic) | The beacon seen burning from the valley | What lights by a condition is lit only within three of the view's 32 m cells of the eye (not from the trailhead, 126 m off), and its glow is a ball 0.26 m across | none | Recorded | — | A flame or glow sized by the design (`lights … glow R`), and far lights kept lit |
| **G12** (4, bug, not explained) | Library designs read | A design with `colour course 6f6b63` made the whole Rural collection unreadable: taking `fallen-log` (an earlier design) failed "there is no kind 'fallen-log'" | The colour renamed `band`; with that one change it was read | Recorded with what was seen | — | Why `course`; and an error that names the design at fault |
| **G13** (content) | Woodland props, a beacon, a tower | The library had no fallen log, tent, beacon, lantern on a post, or stone tower pieces; the checkpoint lamp is green and modern | — | Woodland shelf: 8 designs; The old ranger; The grey alpha | Yes, every one | A tent with a sloped roof (a part cannot be turned: the tarp is flat) |
| **G14** (validation) | The encounter fair for a person | — | A scripted fighter that steps out of a bite as it is gathered and strikes otherwise | Part E | — | Played by hand |
| **G15** (2, friction) | One genre rebuilt | `genres.ps1 -Over` rebuilt all six (and backed each up) | Restored from git, backups removed | `-Only NAME` | Yes | — |
| **G16** (1, as designed) | Kindling puts a fact the world goes by | An act's `puts` is a fact about the thing (`kindled`), not the world's | `on act kindle do put beacon_lit` (7cg) carries it to the world | — | The way, yes | — |
| **G17** (1 and 2: configured; friction) | A hill the trail climbs to the tower's door | Overlapping raise dabs add up (the hill came to 15 m, not 6.5), and a top flattened above the ground about it stands as a step at its rim: in the fourth play-through the walker stopped 6 m short of the archway, against it | The top levelled at the height the hill came to; the way up graded dab by dab with `flatten`, each a little higher; the builder walks the trail a metre at a time and fails if it is steeper than 0.5 (rise over run): 0.46 now | As said | The check is the builder's; a person sees the slope only by walking it | No slope shown on the land, and no "ramp between two heights" tool (7q's specialised terrain tools) |
| **G18** (4, bug in an existing capability; fixed) | The tower's upper storey given stone parapets | A piece of a maker given a design of its own (`each`) was stood on the ground beneath it: the upper storey's eight parapets came down onto the ground floor, inside the walls and across the archway. The seventh play-through stopped at the archway against "BoxShape3D (3.5, 1.1, 0.34) at y 12.63", the parapet's collider at the foot of the tower; the tower looked one storey high | — | Host `land.rs` `by_hand`: a place its maker stood more than 0.25 m above the ground keeps that height when given another design. Protocol test p72 | Yes: every storey of every building, and anything a maker stands above the ground | — |
| **G19** (4, bug: nothing says so) | Stairs the walker can climb | A building's flight rises a storey (2.8 m) over 0.7 of a cell: in cells of 3.5 m that is 49 degrees, steeper than the walker walks up, and they stood against it. Nothing said so (the shelter's 4 m cells give 45 degrees) | Cells of 4 m, and the four stone designs made 4 m long | Recorded | — | A flight that is never steeper than the walker climbs (longer, or turning), or a warning in the structure's settings |
| **G20** (aesthetic, and two small fixes) | A beacon seen burning; a readable prompt; a conifer wood | The glow over a light was a ball 0.26 m across, whatever lit; once kindled, the beacon's prompt still said "Kindle the beacon? You have no flasks of lamp oil." (the why-not prompt ignored an act's `unless`); the Generator's pine preset has a flat crown that reads as a palm from afar | — | `lights … glow R` (things.gd; the beacon's is 0.45 m); the why-not prompt skips an act its `while`/`unless` rules out (acts.gd `_wants`); the library's Tall pine in place of the pine preset | Yes, the first two for every light and act | No flame or smoke a design can show while lit; lit only near the eye (G11) |
| **G21** (4, bug in export; fixed) | The level as a game on its own, beginning at the trailhead | An exported game began at (0, 0) whatever the world said: the export's writer and the game's boot both read the world's start from whichever host answer came last after opening it, which by then was another; and a game made of the world never used its Player start's eye or stamina | — | `main.gd`: the start read from the open command's own answer, and the world's Player start preferred when it has one, in the writer (`--write --from`) and the game's boot; `play` uses the Player start in a game too | Yes: every exported world | Not checked: a side-on genre's eye in an exported game |
| **G22** (4, polish; fixed) | Health and breath a player reads at a glance | Drawn as text, "Health #####" and "Breath ##########", over the stats bars; a hurt wolf had "##--" over it | — | Bars: health (green to red) and breath (gold, grey winded) top left, words "3 / 5"; a hurt one has a red bar over it facing the eye (`fight.gd`); the stats bars stand under them wherever they end (`stats.gd`) | Yes: every fight and every game with stamina | — |
| **G23** (4, polish; fixed) | No stealth meter in a game of fights | "unseen  seen 100%" stood at the bottom whenever anything that sees was about, wolves too | — | The meter is shown only where going unseen is part of the game: someone who catches by sight or `spots`, or an eye on a post (`watch.gd`) | Yes | — |
| **G24** (3, missing; built) | A controller to work every screen | The menu, the ending and the controls page relied on Godot's own focus moves, with almost no frame round what was chosen, no left stick, and the hints said the keyboard's keys until a pad button was pressed | — | The game's screens worked by the D-pad or the left stick, A and B whatever Godot's own actions are (`gameui.gd` `pad`); a gold frame and gold words round what is chosen (`screens_theme`); "A choose  B back" under the menu; with a controller plugged in the hints are its buttons from the start (`main.gd`); the controls page lists only what the device in use has, without Godot's button numbers | Yes: every game | Not tried with a real controller here (none is plugged into this machine): the presses were sent as a controller sends them |
| **G25** (3, missing; built) | A logical controller layout, and a button that swings what is in the hand | Blows were asked by the dash button and only with a foe before you; X ran, the bumpers side-stepped, RT dashed: nothing obviously swung the axe | — | A new button, **attack** (X, RT, left click); the walker's fist blow (`strike`) and every sword cut asked by it; a new move **hew** for an axe in hand (`when wields axe`, drawn as it is wound up, the blade striking); the layout of an action game (A jump, B dodge, X strike, Y act, LB lock on, RB run, LT crouch, R3 put away); the side-steps on Z and X only (`controls.gd`, `moves.txt`, `player.gd`, `aim.gd`) | Yes: every game | A person's own buttons still lie over this (part E) |
| **G26** (4, workflow; fixed) | Gear picked up is ready to use | The axe went into the pack, to be taken in hand from the inventory | — | Something held goes into empty hands as it is picked up, something worn onto a place nothing is worn on; the inventory is only for changing them (`items.gd` `first_use`, `things.gd`); told with the button that uses it | Yes | — |
| **G27** (1, configuration; tuned) | Wolves a player can run from | Wolves ran at 8.2 m/s against the walker's 8.5 run and 4.4 walk, and gathered for a bite in 0.25 s: unescapable, and they seemed to fly | — | The library's wolf: walk 1.6, run 5.6 m/s, slower to turn; a bite gathered for 0.4 s, long enough to see (`moves.txt`) | Yes: every wolf and hound bite | The pace of a fight is still for a hand test |
| **G28** (3, missing; built) | A dressed hero | The walker was Quaternius's bare base model | Gear made of rough shapes (hat, belt, pack) | Quaternius's CC0 ranger outfit (hood, pauldron, bracers, belt, boots) worn by the head of his base character, baked into one file (`outfitmake.gd`, `characters/ranger.glb`) on UAL 2's bones, so every clip plays on it; `character ranger` (characters.txt), made the walker by this world's own characters text | Yes: any world can say the ranger walks; outfitmake.gd makes others of the pack | One outfit; the female ranger and the peasants are in the download, not yet made |
| **G29** (4, bug; fixed) | A model's own textures shown | A model whose meshes have colours in their corners was shown in those colours, its textures thrown away: the ranger was white | — | Only a mesh with no picture of its own is shown in its corners' colours (`animated.gd` `_paint`) | Yes: every model of any pack | — |
| **G30** (3, missing; built) | An axe that looks like one | The axe was a rod and a box | — | KayKit's CC0 Fantasy Weapons Bits (axes, swords, bows, shields), and `model PATH turn x y z` so a file made upright lies as gear lies (`gearbook.gd`); the axe is KayKit's axe_C | Yes: any gear | The other weapons are in the folder, not yet given to gear.txt |
| **G31** (3, missing; built as a design) | A tower that is a tower | The tower was a two-storey house with stone walls given to each piece (the Build tools make houses; a narrow, tall keep with a stair round its walls is not among them, G4 and G19) | — | A design of the library, **Stone watchtower** (`watchtower-keep`): walls, a forty-step stair climbing round their outside, a top floor and battlements, all parts of one design with solid boxes and slanted slabs, written by `library/watchtower-keep.js` from a few numbers. The stair was first put round the inside: in a 6 m room the third-person eye had nowhere to be, and the walker fell off its open edge into the well | Yes: a design anyone puts down | A tower *maker* (choose height, width, sides, stair) is not built: the design is made again by the script; round towers need a round part shape |
| **G32** (3, missing; built, not shown working) | The walker climbs steps | The walker could climb ramps but not a step 0.2 m high (the tower's first stair, of real steps, stopped it at the first): those who walk the world's ways step up to 0.3 m, the person's walker never did | Each flight of the tower walked as one slanted slab under its seen steps (`solid hull`): how games make stairs | `player.gd` `_step_up`: on foot and in no move, blocked at the feet with room above, up 0.3 m at most. Its first form never ran (it asked for `motor.doing` to be empty, which it never is); the second's check in the demo put a box the walker went straight through (a test wrongly made), so it is **not shown working** and the demo no longer claims it | Would be: every kerb and doorstep | A check that puts a step where the walker collides with it |
| **G33** (aesthetic) | A style, not something put together | The library's evergreen style at its afternoon | — | A style of its own, **golden_valley** (styles.txt): deeper greens, warm low sun, gold haze, a rose sunset while the tower is climbed and a blue dusk when the beacon is lit; deer, rabbits, a fox and birdsong for life | Yes: any world | Ambient sound is still the few recipes of sounds.txt |

---

## E. Validation record

### The second pass (7cj): the polish bar

The first pass was called complete on a count of beats (quest taken, wolves felled, beacon
lit), and the user found by playing it that what a player sees and presses had not been
looked at. This pass is judged against a bar written down first, each point checked **by
looking at the screenshots** of the scripted play-through (`editor\godot\demo\NN-watchtower-*.png`)
as well as by its numbers. What could not be judged that way is said.

| # | The bar | Judged by | Met? |
|---|---|---|---|
| 1 | The hero is dressed, textured, and what he carries is seen | screens 01, 04, 07, 15: the ranger's green hood, leather, bracers, boots, the pack on his back, KayKit's axe in his hand and in the swing | Yes |
| 2 | Nothing on the screen is raw: no # bars, no meter that does not belong, no missing letters | every screen: health and breath are bars; no stealth meter in this level (still shown in the stealth genre: `-Demo stealth` screen 03); no box glyphs seen; an emptied chest says "The camp chest is empty." | Yes. The editor's own bar (only while playing in the editor, not in a game made of the level) sits over the top-left corner |
| 3 | A controller works everything, and is what the hints name | the menu opened with Start, moved by the D-pad and the left stick, the gold frame on what is chosen (screen 08), A opened Controls (screen 09, only the controller's buttons by name), B back twice; hints "Y  search the crates", "Back inventory  Start menu" | Yes, with presses sent as a controller sends them (through Godot's input, the screens' focus, the game's). **No real controller is on this machine** |
| 4 | Gear is ready when taken; a button swings it | "In hand: an axe" on taking it, nothing opened; the attack button's move was `hew` (screen 07) | Yes |
| 5 | Wolves can be run from; a bite is seen coming; the fallen lie down | wolf run 5.6 m/s against the walker's 8.5; a bite gathered 0.4 s; each fallen wolf's figure points 0.1 up (on its side) | Yes by numbers; whether the fight is fair is for a hand |
| 6 | The tower reads as a tower and is climbed with the eye in the open | screens 01, 13 (a square stone keep from the trailhead and at its door), 14 (on the top floor through the gap) | Yes; climbed 0, 2, 4, 6, 8 m, one flight at a time |
| 7 | There is life, and a look of its own | deer, rabbits and a fox in screens 01, 04, 10, 12; the light warm and low at the trailhead (01), the sky rose at the top (15), dusk at the beacon (17) | Yes |
| 8 | The fire looks like fire | screen 17: flames rising from the brazier, a flickering light | Yes |

**Not met, or not shown:** the walker's step-up onto low things (built, not shown working:
part D, G32); the grey alpha was felled by the script in the last run (the scripted fighter
fought down three of the four with the axe); the hill's graded way up shows the land's
steep colours in orange and blue blotches (screen 13: an aesthetic matter of the land's
colouring on slopes, left); the inside of the tower's room was not looked at; the length
and difficulty of the whole are for a hand.

### The second pass: what was run

| Check | Result |
|---|---|
| `editor\run.ps1 -Demo watchtower -Check` (the last run) | 11 of 11 beats, no script error (the last of eight runs while this was made; each failure before it became a mending above): the walker wears `characters/ranger.glb`; the game begins at 16:30; the quest taken; the camp chest gave lamp oil, food and medicine; the axe went into the hand on being taken ("In hand: an axe"), the backpack onto the back, no inventory opened; the attack button's move was `hew`; the menu by controller presses: open on Start at "Go on", two D-pad downs to "Journal", the left stick down to "Controls", A opened its page, B back to the menu; the fights: three wolves fought down with the axe (the walker downed none of the times, health 5 to 2), the grey alpha left standing and felled by the script, which says so; every fallen one lying (up 0.1); the stair climbed one flight at a time, 0, 2, 4, 6 and 8 m; at the beacon "Y  kindle the beacon"; kindled (oil 1 to 0, `beacon_lit` by the wire, the quest done, lit); the ending shown and `ended` put; played again: the quest, `beacon_lit` and the oil as made, at the trailhead, no ending |
| `Godot ... --headless --script res://chartest.gd` (the moves, buttons and gear checks) | 192 checks, 0 failed. Before: 16 failed, all but one because they pressed dash for blows now asked by attack (updated), and one because it rewrote a block of characters.txt by text that file no longer had since 7cb (mended); the axe's model was not made (a model with no bones refused: mended) |
| Regressions (`-Demo NAME -Check`), each with no script error | `-Demo fights` 11 of 11, `finds` 6 of 6, `usable` 8 of 8, `inventory` 8 of 8, `stealth` 10 of 10 (its seen-meter still shown, "HUNTED  seen 70%"), `talk` 8 of 8; none with a script error. Before this pass's last changes the fights demo's boss and bandit-captain blows pressed dash: changed to the attack button (they were run before that change, and passed) |
| Host tests (`cargo test --release -- --test-threads=1`) | 66 and 72 passed (no host code changed in this pass; the library's new design is read by them) |
| The level exported (`editor\export.ps1 -World genres\watchtower.json -Name Watchtower`) and started (`Watchtower.exe -- --check`) | built (build\Watchtower.zip, 64 MB; the KayKit axe's picture now goes with its glTF file, and the two packs the ranger is made from do not); started: 1583 things, the walker on the ground at the trailhead (0, 46) at 16:30, every text of the world written as it has them (characters among them). Not played through as a game on its own |

### The second pass: what the user's own buttons did

The user's own buttons (`%APPDATA%\Draw3D\controls.txt`, profile "First_Profile") set X to dash, LB to step aside, RT to run, D-pad down to put away and the left click to run, over the new layout. At the user's asking (2026-10-18) all five were taken out of the file and the profile; the layout of part A applies.
