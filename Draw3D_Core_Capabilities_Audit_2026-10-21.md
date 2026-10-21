# Draw3D — Core Capabilities Audit

Written 2026-10-21, after construction pass 7de. Every claim here comes from the code as it stands at that point, the automated tests, the tour, the demos' recorded readouts, or the user's own hand tests as PROGRESS.md records them. Where something was not checked, this says so.

## How to read the verdicts

- **Verified**: built, reachable from the editor's interface, and backed by automated tests. Most are also tried by the user's hand.
- **Partial**: built and usable, with a named piece missing, or with the only evidence being scripts nobody has used by hand.
- **Missing foundation**: something other systems need and that is not there. It cannot be added inside one system's pass.

There are four kinds of evidence, and each says something different:

| Evidence | What it proves | How much there is now |
|---|---|---|
| **Test** (Rust) | The systems underneath: documents, generation, undo, saves, meshes. | Core 63, host unit 79, host protocol 91; all pass (2026-10-21) |
| **Tour** (`run.ps1 -Tour -Only X`) | The editor, used by script, checking the world afterwards. | 71 parts. Several fail or are intermittent; see section 4 |
| **Demo with -Check** | That a capability can be walked through, with values written down. It proves less than a tour. | 107 demos, 713 beats |
| **Hand** | That a person can do it and it feels right. | Recorded per pass in PROGRESS; most passes since 7db have none |

## Headlines

1. **The foundation is sound.**
   - The document and undo model, the generation pipeline and the save format are well tested.
   - At engine level they are fast: 340,000 things, one window edited in 53 µs (`core/scale-results.txt`). A drawn wall is rebuilt while it is dragged in 7–24 ms (7cl).
2. **Construction is the most complete authoring system, and the only one shaped by repeated hand tests** (7cq–7da).
   - This pass closed what was still clearly missing from Tiny Glade's handling: cut and join, Delete, Ctrl to keep a window, style copying for every drawn structure, and a roof turn.
   - What remains is listed in section 2.4. Most of it is refinement, with one exception: group selection is a missing foundation.
3. **Breadth runs far ahead of hand evidence.**
   - Gameplay, behaviour, items, sound, weather and physics have dozens of features each.
   - None has had a full hand pass since the village and physics play of 2026-10-14. That play reported five faults, still open.
   - Since 7db, nothing has been tried by hand.
4. **The editor has three overlapping places to change what is selected:**
   - the context row under the bar;
   - the strip beside the chosen thing (`buildbar.gd`), for construction and painted patches only;
   - the Inspector window, for everything.

   The strip is the right model. It is hand-coded per sort and missing for placed objects, creatures, makers and tiles. Section 5 evaluates it.
5. **Missing foundations, ranked:**
   1. one selection model that holds several things;
   2. a declared set of contextual controls ("facets") instead of hand-written panels;
   3. behaviour that can be composed beyond single words: states, and/or conditions, sequences edited as structure rather than as text;
   4. authored text content (quests, talks, sequences) edited only as raw text;
   5. no autosave.
6. **Automated coverage of the newest work is thin where it matters most.**
   - Drawn construction (7ck–7de) has protocol tests and demo readouts, but **no tour part**. A regression in how the editor drives it would only be seen by a demo run or by hand.
   - The village tour fails even at the committed code (since 7ck), and the items tour fails at the committed code too.

---

## 1. What this pass (7de) completed

These were checked against the Tiny Glade reference and the user's own notes as genuinely missing. Each now works:

| Item | Tiny Glade reference | Built | Evidence |
|---|---|---|---|
| A wall, fence, hedge or power line **cut in two** | 1.3, 5.2; personal notes 3 and 7 | **Cut** beside it, then click a "+"; or Shift and a click on a "+". Leaves a gap of up to 1 m. A wall that goes round is opened instead. One undo step. | p88; `-Demo cuts` beat 1: two walls, ends at −0.5 and 0.5, one step |
| Two of the same sort **joined** | 1.3; personal notes 3 | An end dragged onto another's end, or a wall drawn on from another's end, makes one wall. The one that was there keeps its look. Ctrl held keeps them apart. | p88; demo beats 2 and 3: one wall from −14 to 14, then four places round a corner |
| **Delete** takes what is in focus | 5.4, 12.1 | A drawn house, wall, path or pond chosen: Delete takes all of it, with the line it was drawn on, as one step. Shift+Delete takes one piece, as before. There is a button beside it too. | p88; demo beat 7: gone, one step, back with Ctrl+Z |
| **Ctrl keeps a low window a window** | 1.1, 2.2, 12.2 | Build > Windows with Ctrl held puts a window at the foot of a wall instead of the door. | Demo beat 5: `s2 0 6.65`, the door left on `s0` |
| **Style Clone** for every drawn thing | 4.5, 12.4 | Copy and Give beside a wall, fence, hedge, path, yard, pond, platform or garden, as a house already had. A look is given only to one of the same sort. | Demo beat 4: red brick and battlements given; the other wall kept its own line |
| **A roof turned** | Personal notes 1 ("roof orientation") | Beside a house: Turn its roof, so the ridge runs across it. Only for a house of four corners that meets no other. | Unit test (ridge across, no taller than 1.3×, no faces that fight); p88; demo beat 6 |

### Looked at, and left on purpose

| Item | Why left | Size |
|---|---|---|
| A roof **curved** by a sideways drag (personal notes 1) | It changes every roof builder and the planes where roofs meet (`roof_hides`). It is cosmetic for the milestone. | Medium |
| **Stilts** grown under a lifted house (TG 1.1) | 7dc's "lift" floats with nothing under it. Supports need to know what is below. | Small to medium |
| A **stairs tool** drawn as its own route (TG 2.4: steep becomes a ladder, level a walkway, attached to walls and roofs) | Stairs exist as a path's choice (7dc) and inside houses. A route tool is its own system. | Large |
| **Quick eraser** (Ctrl while painting) | Ctrl is already "no snapping" while drawing. Overloading it for brushes needs a decision. | Small |
| **Group selection and group move, turn, lift** (TG 12.3) | A missing foundation: section 6. | Large |
| A **radial quick toolbar** (Space, TG 12.6) | Convenience, not a milestone item. | Small |
| A **trapdoor** on a flat roof; **flags** forming bunting | Town roofs already have a hatch; banners hang (7db). | Small each |

**With 7de, the construction milestone is complete** except for those, and except for what only hand use will find. Every item in the 2026-10-18 comparison's "still missing" list is now built: cut and join, stilts as "lift", roofs that join, paths as a brush with arches, windows placed on walls, and small things hung on walls. The remaining gap list is the table above.

---

## 2. Capabilities, system by system

Each entry covers:
- what is built, with the files;
- what a person can author in the interface;
- how it joins other systems;
- the evidence;
- the verdict.

### 2.1 The document, undo, saves (core, host)

**Built:**
- Rust core:
  - `doc`, `engine`, `plan`, `store`, `jobs` (building off the main thread);
  - `kept`, `state` (what happened, kept apart from the world).
- The host protocol (`lib.rs`), with:
  - staged edits (shown, not kept);
  - `together` (several edits made one undo step);
  - history marks;
  - backups beside a world before each save.

**Authorable:** World > New, Open, Save and backups. Undo and redo everywhere. One drawing, one step.

**Joins:** every editing system writes through it. A panic in a build is answered as an error rather than locking the session (7cq).

**Evidence:**
- core 63 tests;
- protocol p01–p12, p18, p27–p30, p33, p69;
- in use by the user since M1.

**Verdict: Verified.**

**Gaps:**
- There is no autosave; a crash between saves loses the session.
- Document fields a host does not know are dropped on save (Known gaps, since W1).
- The editor keeps one world open at a time.

### 2.2 Generation and performance

**Built:**
- Makers (generators) build content per cell, off the main thread, with a memo.
- Godot draws it in tiers by distance (`world_view.gd`).

**Evidence:**
- `core/scale-results.txt` (at engine level, with a prototype house):
  - 17,000 to 340,000 things;
  - first build 10–205 ms;
  - one edit 20–53 µs;
  - a village remade in about 1 ms.
- A live wall rebuild takes 7–24 ms (7cl). A house is planned in about 11 ms (7ck).
- The frame rate shows in the corner (7dc).

**Verdict: Verified** for the engine. **Partial** for what is drawn:
- Godot's own cost for a large, fully dressed world has not been measured. That covers drawn houses with tiles laid one by one, snow caps, plants of about 10,000 triangles near the eye (Known gaps, 7ao) and shadows.
- The scale test uses the prototype house, not the drawn-construction generator.

### 2.3 Terrain

**Built:**
- Brushes: sculpt, flatten, cliff and ledge (`terrain.gd`, `land.rs`).
- Layouts of square or hex cells; terrain tiles with their own ground, raised; tile sets.
- Platforms, clicked or painted (7dc, 7dd).
- Ponds dug into the land (7da).

**Authorable:**
- Terrain window (F8), Layout (Ctrl+G), Tiles (Ctrl+T).
- Build > Platform and Raise. Water > Pond.

**Evidence:**
- p19–p20, p45–p53, p74, p87;
- tours `land`, `layout`, `tiles`, `raised`, `raisedwalk`, `surface`, `seams`;
- hand: M2, L1, L2 and four tile looks (2026-10-12).

**Verdict: Verified.**

**Partial:**
- An outline or region as a thing of its own that any maker can fill (7q, second item) is not built.
- Tiny Glade's dab-then-drag for height and steepness: platforms take numbers beside them, not a drag on the dab.

### 2.4 Construction (drawn)

**Built:**
- Walls, fences, hedges and power lines are dragged and rebuilt live (`line.gd`, `marks.gd`, host `build.rs`, `tileops.rs`).
- Houses of eleven sorts, including the town's. Each has:
  - handles: move, turn, sides, height, roof, wings, towers;
  - storeys, stairs, porches, balconies, windows and doors by hand;
  - roofs that join, stacking on a flat roof, open or walled where they meet;
  - wear, climbers, things hung on walls, furnishing.
- Paths with arches, gates, steps and stepping stones. Yards, paving, gardens, ponds, docks and platforms.
- Rooftop pieces, planks across, fire ladders.
- 7de: cut, join, Delete, Ctrl window, style copying, roof turn.
- Context between drawn things (`settle_drawn`):
  - keep-clear;
  - openings where paths cross;
  - walls cut where a house stands;
  - joined pieces following a house.

**Authorable:** the right-side construction bar and the strip beside the chosen thing. The left Build list draws the same tools.

**Joins:**
- The walker walks through doors, which are navigation links (7ag).
- Snow lies on drawn roofs, sheltered under them (7dd).
- The world's style colours everything (7da).
- Paths open what they cross.
- Plants keep clear.

**Evidence:**
- protocol p58–p59, p68, p72–p88; unit tests of every piece as a whole mesh;
- demo readouts: `walls`, `looks`, `houses`, `joined`, `onehouse`, `inplace`, `mixes`, `meets`, `decor`, `details`, `context`, `weathers`, `streets`, `rooftops`, `cuts`;
- hand tests: 7cq, 7cs, 7cu–7da.

**Verdict: Verified**, with these **Partial** items:
- 7db–7de have not been tried by hand.
- There is **no tour part** for drawn construction.
- The left Build list draws structures on cells (L3, 7ad and 7af) through a second construction route. A path drawn on the land does not open a gate in a structure on cells (Known gap, 7ck).
- Handles need a click on the house first, not a hover (7cu, 7cv).
- Joining shows only on letting go, not live (7cv).
- The rest of the list is in section 1.

### 2.5 Plants, generators, painting

**Built:**
- Generators (`grown/*.rs`, the Tree and Plant Generators): trees in four forms; grass, flowers, ferns, crops, bushes, fruiting plants, bamboo, rock (nine families).
- Brushes with mixes, sets and sorts.
- Colour variation by role.
- Seasons and wind per patch.
- Fields that fill an outline.
- An eraser for everything.

**Authorable:**
- Paint group and the strip beside a patch.
- The Plant and Tree Generator windows, opened from a sort.
- The Library.

**Evidence:**
- p52–p57, p63, p64;
- tours `brush`, `brushmix`, `grow`, `growth`, `plants`, `nature`, `field`, `acres`;
- hand: 7x, 7cw–7cz.

**Verdict: Verified.**

**Partial:**
- Growth over time and harvest are a gameplay layer not built.
- Fields do not keep clear of what stands.
- Only designs grown in the Generator are waded through (7ap).

### 2.6 Style, palette, looks

**Built:**
- Styles (`style.rs`, `styles.txt`, Window > Styles), eight after games.
- Palettes; the colour, picture and material of each part.
- Kept looks in the person's own library (`looks.json`).
- Copy and Give: now for every drawn sort, and for patches.
- Your own pictures and models brought in.

**Evidence:**
- p13, p41, p76, p77;
- tours `style` (intermittent on time, Known gaps) and `world`;
- demo `styles`;
- hand: the user found the built look "generic, little charm" (7cs). That was kept for the style step, which has not been done.

**Verdict: Partial.** The mechanism is verified. The construction and style direction document's style, estimate and audit steps after construction are not started.

### 2.7 Environment

**Built:**
- Weather with one wind (`weather.gd`, `weather.txt`).
- Seasons; what lies: snow with depth and shelter, rain, leaves, ash. Prints.
- Anything falling.
- Sky, sunsets, sun direction, volumetric light.
- Water with foam.
- Sound from written recipes (`sounds.txt`).
- The season and weather corner.

**Authorable:** the Environment window, the corner, Sounds, and each weather's own lines in `weather.txt`.

**Evidence:**
- tours `weather`, `living`, `sound`, `style`;
- demo readouts: `seasons`, `weathers`, `living`, `sound`;
- hand: none since 7aa's trees and air. PROGRESS: "Nobody has listened yet" to sound.

**Verdict: Partial.** Rich, and seen only in stills and readouts.

**Gaps:**
- No recorded sound or music.
- Fog has no controls of its own, while Tiny Glade's fog has density, height and clinging.
- How it looks in motion has been seen by no one.

### 2.8 Characters and movement

**Built:**
- The motor (`motor.gd`) with step-up, climbing, vaults, wall runs and backflips.
- Moves as data (`moves.txt`), combos, eight-way movement.
- The Character Studio (`studio.gd`, 6,500 lines): clips, phases, the yard, gear, the Make tab.
- Mixamo and KayKit clips; IK for two hands.

**Authorable:** the Character Studio, F1 movement numbers, F10 texts.

**Evidence:**
- tours `motor`, `moves`, `climb`, `studio`, `hands`, `course`, `firstwalk`;
- hand: four Studio hand tests (7bd–7bh). The training ground's three behaviour changes still await the user's word (PROGRESS "Still the user's to say").

**Verdict: Partial.** The user parked characters (2026-10-16). Owed:
- fingers round the grip;
- faces;
- a real Mixamo file;
- a real controller in the Studio. The user has played with a controller elsewhere (7cj), but the Studio's pad work was pushed as events.

### 2.9 Items, inventory, the game's screens

**Built:**
- Items with ids; gear to find; equipping.
- `gameui.gd`: HUD, inventory, menu, journal.
- Stats and statuses (`stats.txt`), effects, usable presets.
- Aim and fire. The HUD laid out in Window > Game display.

**Evidence:**
- tours `items` (fails at the committed code: Known gaps), `inventory`;
- demos `items`, `inventory`, `usable`, `anything`;
- hand: 7ca and 7cb were tried by the user (PROGRESS: "after the user tried 7ca by hand").

**Verdict: Partial.**
- What is worn shows only on a Studio character (Known gaps, 7bi).
- The items tour is broken.

### 2.10 Behaviour and life (NPCs, creatures)

**Built:**
- Folk, the village at work: goods, sequences, quests, talks, reactions (`folk.gd`, `work.gd`, `quests.gd`, `talk.gd`).
- Wild life (`wild.gd`); perception and stealth (`watch.gd`); fights and bosses (`fight.gd`).
- Behaviour said as kind words: about 80 words that say something once, 11 held words, `while` conditions, and `on TRIGGER do ACTION` with 5 triggers × 5 actions.

**Authorable:**
- The Inspector: what it can do, watch rows, `while`, wires.
- The Playtest window.
- Texts (F10) for quests, talks, sequences and reactions.
- The assistant drafts lines.

**Evidence:**
- p42, p70, p71;
- tours `village` (**fails at the committed code since 7ck**), `villagers`, `work`, `talk`, `errand`, `keeper`, `flock` (intermittent);
- hand: the village and physics play of 2026-10-14. Five faults were reported and are still open: a held pose, stairs while carrying, crowds, the ball sent flying, falling-block shapes.

**Verdict: Partial, with a missing foundation:**
- A thing's behaviour is a list of words. No states, no and/or of conditions (Known gaps, 7am: "one condition a thing"), no priorities, no behaviour edited as a structure.
- The Luau scripting prototype (`prototypes/luau-boundary`) is not wired in.
- Quests, talks and sequences are edited as raw text only, with checks.

### 2.11 Physics, mechanisms, signals

**Built:**
- Each thing's own physics and colliders; joints (several per thing).
- `blows`, `carries`, `hits`, `feels`.
- Conditions, `clears` and `ticks`. The Signals window.

**Authorable:** the Inspector's physics and joints (joints picked in the view), Signals, Physics (F2).

**Evidence:**
- p60–p62;
- tours `physics`, `mech`, `signals`, `bridge`, `puzzle`, `recreate`, `loose`;
- hand: the batcher and the strength tester (7ar).

**Verdict: Verified** for the first version.

**Partial:**
- Dragged joint anchors and axes are deferred.
- Walking into a ball can send it flying (a user report).

### 2.12 Genres, worlds, the built game

**Built:**
- Genres made from New world: wilds, stealth, shelter, watchtower.
- Connected worlds and journeys.
- Side-on view with a cutaway.
- Export of the game.

**Evidence:**
- p66, p67;
- tours `standalone`, `saves`, `journey`;
- hand: the watchtower was played with a controller and rejected (7ch). It was reworked in 7cj; PROGRESS records no hand test of the rework.
- The built game exits with a Windows fault code after it has done everything (Known gaps).

**Verdict: Partial.** The pieces exist. No genre has been played to the user's polish bar ("judge polish by looking at what a player sees").

### 2.13 The editor shell

**Built:**
- One workspace: the bar, windows on a desk, a left toolbar, the right construction bar.
- About 29 windows.
- The Inspector says HOW IT WORKS for anything clicked (7bz).
- The checklist, the Library, the assistant.

**Evidence:**
- tour `ui` (fails only on the Studio window's size, since 7ay), `workflow`, `findings`, `editing`, `chosen`;
- hand: in use throughout.

**Verdict: Verified**, with these **gaps**:
- One thing is selected at a time.
- The workspace floats and does not dock.
- The context row, the strip beside a thing and the Inspector overlap (section 5).

---

## 3. How the systems join, and where they don't

**Joined well:**
- Construction knows terrain: it stands on platforms, ponds are dug, and the land is raised under what is built.
- Plants keep clear of what is built and lean off the walker.
- Weather lies on what is drawn, sheltered under roofs.
- The world's style colours drawn pieces.
- The walker goes through drawn doors; town roofs and planks are walked over.
- A click on anything opens the Inspector.
- One undo step per gesture runs across systems (`together`).

**Seams found in the code:**
1. **Two construction routes.** Structures on cells (L3, the left Build list's grid tools) and drawn structures (the bar) are both live:
   - a path drawn on the land does not open a gate in a structure on cells;
   - 7de's cut, join and Copy/Give apply to drawn ones only.
2. **Three edit surfaces** for what is selected, each hand-coded (section 5).
3. **What a character wears** is a Studio character's own. The host's figure has no bones for gear (7bi).
4. **Text content and the interface**:
   - quests, talks, sequences and reactions are written as text and checked;
   - the Signals window shows who writes and reads each fact;
   - nothing edits them as structure;
   - the Inspector's wires (`on ... do ...`) are the only part of behaviour edited in rows.
5. **Gameplay over built worlds.** A drawn town is walkable and its doors open, but no system gives its buildings a use: a shop that trades, a bar with folk in it. The village at work (7ah) uses its own placed buildings, not drawn ones.

---

## 4. Automated testing and testing by hand

**Failing or intermittent, all recorded in PROGRESS Known gaps:**

| Check | State | Since |
|---|---|---|
| `-Tour -Only village` | fails, also at the committed code | 7ck (2026-10-18) |
| `-Tour -Only items`, `-Only library` | fail at the committed code | 2026-10-16 |
| `-Tour -Only ui` | fails on the Studio window only | 7ay |
| `-Tour -Only style` | fails on time passing, on this machine | 2026-10-13 |
| `-Tour -Only kinds`, `flock` | intermittent; a controller's stray presses were one cause, closed in 7am | — |
| The built game | exits with a fault code after finishing | 7at |

**Not covered by a tour:**
- drawn construction (7ck–7de);
- weather that lies, snow caps, the corner;
- the town;
- docks and boats boarded.

The protocol tests cover the host's side of all of these; the demos cover the editor side, but only as readouts.

**Never tried by hand:**
- 7db–7de;
- sound ("nobody has listened");
- the stealth level and side-on view;
- fights beyond the shelter;
- items since 7cb;
- travel between worlds;
- the assistant;
- the reworked watchtower.

**Recommendation:**
1. Before the next major system, add one tour part, `drawn`, that drives the bar through draw, handles, cut, join, Copy/Give, Delete and undo. The `cuts`, `onehouse` and `joined` demos already contain the steps.
2. Mend or retire the village tour. It is the only automated check of the village's whole day.

---

## 5. A contextual editing bar for whatever is selected

### What exists

`buildbar.gd`'s `_lay_beside` puts a strip over the chosen thing, about 2,300 lines of the file's 3,300:
- three fillers: a house, another drawn structure, a painted patch;
- shared helpers: `_fold`, `_small`, `_choices`, `_tints`, `_part_button`, `_picture`, `_opens`, `_any_colour`;
- each change written through `makers.look_set`, so one change is one undo step.

The user has judged it the right direction (7cw, 7cz). They also found it can grow huge, so 7db folded it.

**Where it is missing:**
- a placed object (a design put down);
- a creature or a person;
- a maker that is not a patch (a wood, a meadow, a street, a field);
- a terrain tile;
- a light;
- the walker.

For these, the colour, texture, size, behaviour and usable presets are in the Inspector window (`inspect.gd`: `_fill_shape`, `_texture_row`, `_fill_abilities`, `_fill_usable`, `_fill_watch`, `_fill_wires`) and the World outline window.

**Why it is one-off now:** each sort's panel is a `match` branch that knows its own words. A new system means another branch. The same idea has different controls by sort:
- a house's colours are a fold with a palette; a wall's are part buttons;
- Copy and Give sit in the house's "keep" fold but at the end of a wall's row;
- the Delete button sits after the house's keep fold, and at the end of a wall's row.

### Proposal: one bar, made of declared facets

A **facet** is one common control, declared once with:
- `applies(selection)`: whether it is shown;
- `read(selection)`: what it is now, for its chip's picture or colour;
- `choices(selection)`: what it opens;
- `write(selection, value)`: one undo step.

The bar shows the facets that apply, **always in the same order**, at most about eight in the row. The rest are under **More…**, which opens the Inspector at that facet's section. Advanced settings stay where they are.

| # | Facet | House | Wall, fence, hedge | Path, pond, platform | Painted patch | Placed object | Creature or person | Maker (wood, field) | Tile |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Sort / form | ✔ (Look) | ✔ | ✔ floor, edge | ✔ mix | — | ✔ kind | ✔ designs | ✔ set |
| 2 | Palette | ✔ | ○ | ○ | ✔ colours | ○ | ○ | ○ | — |
| 3 | Colours by part | ✔ | ✔ | ✔ | ✔ by role | ✔ (Inspector, LiveColour) | ○ coat | ○ vary | ○ |
| 4 | Material / picture | ✔ | ✔ | ✔ | ✔ rocks | ✔ (Inspector) | — | — | ✔ surface |
| 5 | Size / height | handles | handles + steps | width, height | thicker / thinner | ✔ (Inspector) | ○ | reach | raise |
| 6 | Season / wind / wear | wear | wear | wear | ✔ | ✔ wind (7dd) | — | ✔ | — |
| 7 | Details (its own fold) | ✔ | gates, top | steps, life | heads, beds | — | — | — | — |
| 8 | Behaviour, quick | — | — | — | life loudness | ✔ usable presets (Inspector) | ✔ follows, roams (Inspector) | — | — |
| 9 | Copy / Give / Keep | ✔ | ✔ (7de) | ✔ (7de) | ✔ | ○ | ○ | ○ | ○ |
| 10 | Delete | ✔ (7de) | ✔ (7de) | ✔ (7de) | Erase | Delete key | Delete key | Remove (World outline) | Tiles |
| 11 | More… to the Inspector | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ |

✔ built now (where it lives, if not in the strip). ○ the write path exists, but there is no chip. — does not apply.

**How it would work across types:** the selection decides which facets apply, and each facet knows its write path:
- `look_set`, or `params`, for makers and drawn structures;
- the design's own lines (`design_own`, `set_texture`, `set_part`) for a placed object;
- kind and own lines (`set_own`, `set_watch`) for a creature;
- regrowing designs for a patch.

Nothing new is needed in the host to begin. Every write path above already exists and already gives one undo step.

**With several selected** (once that foundation exists), the bar shows only the facets every selected thing has, and writes all of them in one `together` step. That is Tiny Glade's group transform and Style Clone flood in one model.

### Phasing

Each phase can be reviewed by hand before the next.

1. **Extract** the strip's helpers and the three fillers into `selectionbar.gd`, with no change in behaviour. Check: the existing demos (`inplace`, `details`, `context`, `cuts`) give the same readouts.
2. **Placed objects and creatures** get the bar:
   - for objects: colour, picture, size, wind, usable preset, Copy / Give, Delete, More…;
   - for creatures: their own colour, what they do (follows, roams), loudness, Delete, More….
   Each uses the Inspector's existing writes.
3. **Facets declared** rather than branched, and the same order for every sort, including houses and walls. Copy, Give and Delete always last.
4. **Several selected:** Ctrl+click adds to the selection (Tiny Glade's Ctrl + right-click). The bar shows the shared facets; one handle moves, turns and lifts the group.

**Risks:**
- The strip covers what it edits: keep it clamped above the thing, as now, and fold it.
- Facets that read differently per type: a "colour" is a part's colour on a house, a role's on a patch, a design's own on an object. The chip has to say which.
- Phase 3 touches all of `buildbar.gd`'s strip: do it after phase 1's extraction, not in one pass.
- Multi-select changes what Delete, Move and the Inspector mean when several things are chosen.

**Recommendation:** do phases 1 and 2 as one pass, because they make the bar general without rewriting it. Do phase 3 with them only if 2 shows the branches multiplying. Phase 4 is the selection foundation in section 6, item 1.

---

## 6. Missing foundations, ranked

1. **A selection of several things.**
   - Needed by: group move, turn and lift (TG 12.3); style flood; batch edits; Erase of what is selected; the bar's shared facets.
   - Now: `makers.chosen` (one maker) and `ed.picked` (one thing).
   - Size: large. It touches main.gd's selection, marks, the Inspector and the bar.
2. **Declared contextual facets** (section 5, phase 3). Without them, every new system adds another hand-written panel.
3. **Behaviour composition.**
   - What is there: single words, one condition a thing, wires of 5 × 5.
   - What is missing: and/or conditions, states with priorities (a guard patrols, then hunts, then searches, then returns), and behaviour shown as rows rather than words.
   - This decides how far gameplay can go without writing code.
   - The Luau prototype is the code route, if one is wanted: it is a decision, not a default.
4. **Structured editing of authored content**: quests, talks, sequences and reactions as rows with pickers, keeping the text as their store, as Signals already shows facts.
5. **Autosave and crash recovery.** Backups are taken only on save.
6. **Godot-side performance at scale**: a measured budget for a dressed town (drawn houses, snow, plants, shadows), like `scale-results.txt` is for the engine.

---

## 7. Recommended next priorities

**Before either direction:**
1. The user's hand test of 7db–7de: construction, weather, the town and this pass's cuts. Everything since 7da is untried, and the user's tests have been what moved construction.
2. A `drawn` tour part, and the village tour mended. These are small, and they protect what is most used.

**Then, one of two directions.** This is the user's call.

**A. Deepen authoring and the environment:**
1. the general selection bar (section 5, phases 1–2);
2. several selected (foundation 1);
3. the style pass the construction direction document orders next (the user's "generic" finding);
4. regions as things (7q);
5. fog and recorded sound.

- *For:* it compounds the part of the engine the user has validated most, and every later game is built with it.
- *Against:* gameplay stays at its untried breadth longer.

**B. Gameplay mechanics and behaviours:**
1. behaviour composition (foundation 3), proven on one real loop, such as a guard that patrols, hunts, searches and returns, in a drawn town;
2. the five faults from the 2026-10-14 play;
3. structured quests and talks (foundation 4);
4. one genre (the watchtower, or a town night) played to the polish bar with a controller.

- *For:* the engine exists to make games that are sold, and the gameplay systems have the least hand evidence.
- *Against:* it builds on an editor where several things cannot be selected, and on behaviour authored as text.

**My recommendation:** A1 and A2 first, the general bar and several selected. Then B.
- They are the two foundations both directions need. Gameplay authoring (placing guards, giving them behaviour, copying a setup to ten doors) needs the same selection and the same contextual controls.
- They are where the user's own feedback has pointed four times: 7cw, 7cz, 7db, and today's request.
- B1, behaviour composition, should then be designed together with how the bar shows behaviour. That way it is edited in context from the start, not as text first.

## 8. Open questions for the user

1. A or B after the hand test, or the order recommended above?
2. Structures on cells (L3) alongside drawn structures: keep both, or retire the cell route for building and keep layouts for tiles? Every construction feature now has to be built twice or left out of one.
3. Ctrl as Tiny Glade has it: no snapping, a window not a door, and (unbuilt) a quick eraser while painting. Overload it for brushes, or give the eraser its own key?
4. Behaviour: rows only (states, conditions, priorities), or rows plus a scripting route (Luau)?
