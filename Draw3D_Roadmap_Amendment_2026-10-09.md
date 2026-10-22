# Draw3D Master Roadmap — Amendment 2026-10-09 (working version)

This amends `Draw3D_Master_Roadmap_Final.md` without changing it. Where the two differ,
this file is the working intent; the master's sections, principles and development order
stay as written except as amended here. Independent model reviews
(`engine-architecture-review-*.md`, `3D_World_Building_Engine_Technical_Review_Pass_1.md`)
are not merged here. Source: `Draw3D_Audit_and_Next_Milestone_2026-10-09.md`.

## 1. What the nine-stage runtime completed (marking the master's checkboxes)

Checkpoint A (Persistent World): all seven items built. Checkpoint B (Living World): built
except "navigation responds to world changes" (partly: a gate; not general) and "behaviours"
(five words). Section 1: built. Section 2: player, NPC, creature, rig, poses, locomotion,
cameras, attachment points, assignable move sets: built; generated/imported creatures,
novel body plans, layered animation, audio: not. Section 3: nav, clock, five behaviour
words: built. Section 4: weather, falling things, wind: built; accumulation, audio: not.
Section 5: three world styles driving generation and air: built; regional, materials: not.
Section 6: identity, provenance, overrides, freeze/detach, generator dependencies,
versioning, validation marks: **core built, no authoring UX**; sculpting, painting, rivers,
roads, caves, asset browser, studios: not. Sections 7–9: prototype items, physics, two
cameras; nothing authorable. Section 10: Windows export built, unverified elsewhere.

Hands-on verification of the runtime is still owed (PROGRESS.md "Now").

## 2. New principle O — Content is data, engine code is the interpreter

> A new kind of thing, creature, prop, affordance set, terrain feature or rule is content
> the user (or a program) writes and the engine interprets. Engine code adds vocabulary;
> it does not add things. Everything built as an example is written in that vocabulary.

- ☐ **Kind definitions** are document assets (`kind.NAME`): geometry recipe (parts, later a
  mesh asset), collider, affordance and behaviour lines, rig reference, footprint, style
  colour names, `like OTHER` for variants. Realized from data; a changed kind re-realizes
  only what was made from it; undone, saved and exported with the document.
- ☐ Built-in kinds migrate to kind text over time; the Rust functions remain as the
  interpreter's primitives (block, post, limb, blob, face, lathe, loft…).
- ☐ The written texts (moves, poses, acts, eyes, weathers) move into the document so they
  share undo, history and the project file with everything else.
- ☐ Gameplay rules are records: *when EVENT, if CONDITIONS, do ACTIONS*, with facts as the
  shared state they read and write.
- ☐ Every editor panel edits the same text a program would write (principle N's "one
  representation"); the editor must be fully usable without an LLM.

## 3. Terrain and geometry editing, made concrete (resolves §6 and §2 ambiguities)

### 3.1 Terrain

- ☐ **Persistent sculpt layer**: a height-delta field kept by the land node (document
  content, not a parameter), exported inside `shape` so every generator that samples the
  ground follows it. Regenerating vegetation, streets or anything else never touches it.
- ☐ **Brushes**: raise, lower, smooth, flatten (to a height), with radius and strength;
  undoable strokes; patch-local rebuild (fix: a land change must not remake all patches).
- ☐ **Stamps**: cliff, ledge, slope, hill, valley (parameterised shapes applied to the layer).
- ☐ **Paint layer**: surface class per sample (grass, rock, sand, dirt…) read by the ground
  colouring and by generators (a meadow does not sow on rock).
- ☐ **Feature lines**: paths/roads, rivers, coasts, region boundaries as polylines the land
  and dependents read (the route polyline is the first of these).
- ☐ Regenerating the land's own parameters (seed, amplitude) keeps the sculpt and paint
  layers; the user may clear or bake them explicitly.

### 3.2 Object and creature geometry: three tiers

| Tier | What the user physically changes | Data model | Milestone |
|---|---|---|---|
| T1 semantic | Words and parameters: "a tree", "a crate", `like crate` with a colour | kind text, parameters | M1 |
| T2 component/shape | Parts: add, remove, move, resize, recolour; attachment/interaction points | kind text parts list, later with handles | M1 (text), M3 (handles) |
| T3 direct mesh | Vertices and faces moved, extruded, bevelled; revolve, sweep, loft | an editable mesh asset (indexed polygon mesh with history of ops), baked to `MSH1` on realize | M3 |

### 3.3 Preserving edits through regeneration: six mechanisms, not one

| Intent | Mechanism | Status |
|---|---|---|
| Change the source | generator parameter / kind text edit | core built; editor in M1 |
| Protect one local change | edit record (Pin, Set, Exclude, Attach, Tag; **new:** content override for shape) | records built; content override M3 |
| Shape the land by hand | sculpt/paint layers | M2 |
| Keep this exactly as it is | freeze (subtree kept, regen skips it) | core/host built; editor M2 |
| Make it mine entirely | detach (independent node), adopt to return | built; editor key only |
| Reuse what I made | save as kind / variant (`like`) | M1 |

## 4. The editor as an authoring application: workspaces and their minimum

| Workspace | Minimum | Milestone |
|---|---|---|
| Project | Open…, Save As…, named worlds, title bar | M1 |
| Inspector | picked thing: origin, owner, path, parameters (editable), traits, kind link; owner hierarchy list | M1 |
| Kinds | list, text editor, apply, new, duplicate, delete, problems, live result | M1 |
| Terrain | brush/stamp/paint tools, layer clear/bake | M2 |
| Generators | parameters, inputs, freeze/thaw/regenerate, "what will change" preview | M2 |
| Written texts (moves, poses, acts, eyes, weather) | in-editor text editing with problems, undoable | M2 |
| Mesh | T3 tools on a mesh asset | M3 |
| Rules | event/condition/action records | M4 |
| Assets | browser of kinds, meshes, sets, styles; previews later | M3–M4 |
| Play | as today (Tab, F5/F9), plus controller verified | ongoing |

## 5. Next-phase order (replaces the undecided "after stage 9")

1. **M1 Kinds as content** — see `Draw3D_Milestone_M1_Brief.md`. Built 2026-10-09; waiting on the user.
2. **M2 Terrain sculpt layer** — patch-local rebuild; sculpt and paint layers; brushes and
   two stamps (cliff, ledge); feature line editing for the route; freeze/thaw and generator
   parameters in the editor; written texts into the document. Proof: sculpt a cliff with
   ledges, regenerate the woods, the cliff stands and the parkour route is climbable.
   Built 2026-10-09 (see `Draw3D_Milestone_M2_Brief.md`); waiting on the user.
3. **M3 Editable mesh assets** — T3 tools; content-override record; visual part handles;
   built-in kinds migrated as needed; asset browser v0. Proof: a generated house's roof
   edited by hand survives a street regeneration. Built 2026-10-09/10 as far as: parts
   that bend, meshes as text, the override (7a), a mesh's corners and faces dragged in
   the view (7c). Owed: handles on a kind's part, built-in kinds, the browser.
   **UI foundation** (built 2026-10-10, at the user's asking, ahead of W) — see 7e: one
   workspace that every tool is found in, and that W is put into.
   **W The World-Building Workflow** — see 7d and 7h. A world begun from nothing in
   the editor and gone on with. **W1 built 2026-10-10** (makers as things of the
   world; see 7h and `PROGRESS.md`): it settled where a game starts as a thing in the
   world, and growth in one place (a wood with a reach, as many woods as wanted).
   **W2** (what is made once used again: kinds and meshes between worlds, the code's
   things as kinds, compositions a person writes and keeps, the asset browser) is
   recommended next.
   **M3b The tools of going over a shape** — see 7f: numbers for a move and for a
   thing's place, several chosen at once, bevel, join, a kind's part. After W1.
4. **M4 Rules, items, behaviours as data** — rule records; inventory/equipment with
   effects; perception/patrol/chase words; choice among offers. Grown, 2026-10-10, into
   four steps and an authoring window (7g): conditions and rules, sequences, routines
   and reactions, profiles and what is worn. Proof: the parkour island
   benchmark steps 5–8 authored without Rust.
5. Then the master's phases 6–7 (advanced movement, vehicles, genre gauntlet, external
   playtest) on the authorable foundation.

The user's hands-on pass over the six waiting demos, the controller, and the build on
another machine run alongside M1 and feed PROGRESS.md as before.

## 6. Parkour island benchmark: today vs intended, per step

| Step | Today | After M1 | After M2 | After M3–M4 |
|---|---|---|---|---|
| 1 Create project, generate/shape island | fixed playground, seed by script | Open/Save As | seed/amp in editor | — |
| 2 Sculpt route: cliffs, ledges, rooftops, ruins, harbour | none; roofs climbable by accident of being solid | ruin walls and obstacles as user kinds | sculpt layer, cliff/ledge stamps | mesh-edited ruins |
| 3 Styles | 3 world styles | — | — | regional styles later |
| 4 Move sets | text, live | — | in-editor, undoable | — |
| 5 Obstacles with affordances | 18 fixed kinds | **user kinds with `offers`** | — | shape by handles/mesh |
| 6 Hostiles: perceive, patrol, chase, attack, disengage | sheep shy; keeper walks a schedule | bodied user kinds with today's words | — | behaviour words (M4) |
| 7 Weapons, shields, armour affecting movement | hatchet `does chop` | kinds with `kept`/`held`/`does` | — | effects on sets (M4) |
| 8 Objectives, events, unlocks | one errand in Rust | — | — | rule records (M4) |
| 9 Play, inspect, adjust, replay, keyboard+controller | play yes; inspect one line; controller untried | inspector | — | — |
| 10 Export, verify elsewhere | export yes; elsewhere no | — | — | external run (owed now) |

## 7. Removed or not reintroduced

- No link between Draw3D's style system and Draw5 (explicitly removed by the user).
- Nothing from the model reviews is adopted as a requirement by this amendment.

## 7a. Two decisions taken while building M3 (2026-10-09, evening)

**Small things and the systems they belong to.** A probe is made only where it is the
first part of something the roadmap has, in the words that thing will use, and where
the rest will add to it and not replace it; otherwise it waits. For each, PROGRESS.md
says which system it belongs to, what it shows, and what it leaves.

**Secondary motion is the first tier of section 9's cloth and simulation**, not a
feature of a character: (1) a bone that follows, one weight on a spring (built: a
kind's `sway` line, `sway.gd`); (2) weights joined to each other and stopped by what
is solid; (3) a weight for each corner of a mesh (cloth, per-vertex data, constraints:
the master's five cloth items). All three are said as what hangs from where, how
stiff, how damped, how heavy, how much of the wind, are stepped at a fixed step after
the pose, and can be put out. (2) and (3) wait for section 9's milestone: they need
what is solid about a body said as shapes (it is one capsule now) and a mesh that
keeps its corners' neighbours to Godot's side (it is a triangle soup there).

**The content override for shape (section 3.3, "protect one local change") is the
`Set` record there already is**, not a new op: one field of a generated thing's brief
names a mesh asset, and that thing's realizer bakes the mesh where the field is said.
Identity, provenance, undo, the project file, re-finding after keys change and the
look-at marks are the record's already. "Edit the roof that was generated" is a
command over it: what the realizer made of the thing is written out as a mesh's lines
and the record set to that mesh. It needs one thing a mesh has not: a colour for a
face. The roof's generator is not rewritten. This is M3's third part, next.

**Built (2026-10-09, night), as decided, and what it settles for the rest.** A
generated thing's shape is one of three things, in this order of authority: what its
generator's code makes; a mesh the document has, named in its brief by a `Set` record
(the host's `shape` command); or, for a thing of a kind, that kind's text. The record
is by the thing's key, so it is the generator that goes on saying where the thing is,
how big, and what colour its kind of thing is here, and the mesh that says what it
looks like. Two things follow for later work and are not to be re-decided by it:
(1) **a change by hand to a shape is always lines in a mesh asset**, whoever writes
them: typed (now), dragged (M3's fourth part, handles), or made by a tool; so undo,
the file, and "what changed here" are the document's for all of them. (2) **A mesh's
colours are said in the style's words where they can be** (`its`, `wall`, `roof2`,
so many times as light), so what is made by hand goes on following the style and
what its generator says of it. What it does not settle, and section 3.3 still owes:
a shape that follows a size (a house made wider under a roof made by hand: kept as
written and said, not fitted); what is walked into following the shape; and the same
for what is not a part of a house (each generator says one line for it).

## 7c. What a tool in the view is (decided building M3's fourth part, 2026-10-10)

**A tool that changes a shape in the 3D view writes lines of that shape's text, and
keeps nothing else.** Built for a mesh's corners and faces (`handles.gd`, the host's
`handles`): a corner dragged is `move A ... by x y z`, a face dragged `move face F by
...`, a face dragged with Ctrl `push F D`; one drag is one line and one document edit.
The host says which mesh a thing's shape is and where that mesh is in the thing's own
frame, so the tool does not know whether it is on a roof, a prop or anything to come.
What follows for later work: a new mesh operation (bevel, a loop, a sweep) is a word
the text gains first and a gesture that writes it second; a handle on a kind's part
writes that part's numbers the same way; and no tool has a store of its own to keep
in step with the document. A large mesh is worked on by a **reach** (every corner
within so far of the one held, moved as one line), since a generated shape written
out has no larger pieces to take hold of.

## 7d. The World-Building Workflow: the direction, and a milestone (the user, 2026-10-10)

**The longer aim is that a person makes and edits worlds from scratch with the same
capabilities the island was made with**, in the editor, without scripts or source:
a blank project and land of a chosen size, outline and elevation; terrain painted and
surfaces chosen; trees, bushes, flowers, rocks, props, buildings and other content
generated or placed; paths drawn and edited and structures put where they are wanted;
a player character and other characters and creatures; a randomized starting
arrangement from parameters the person gives; the result edited, saved, reopened and
gone on with. **Each new authoring capability is to become a part of this**, and is
judged by whether it does.

It is a milestone of its own, **W**, taken after M3's fourth part; whether before M4
or after it is the user's to say (recommended: before, see `PROGRESS.md`, "Next").

- **First deliverable (W1), end to end:** "New world…" (size, outline, height of
  hills, seed) makes an empty island with a place for the walker; woods, a meadow and
  rocks are each put down as a generator node with a place and an extent, a line is
  drawn and a street set along it, the walker's beginning and a creature put down; "a
  beginning from these numbers" makes the same kinds of node by a recipe; then the
  land is shaped and painted, a prop placed, a roof taken hold of, the world saved,
  opened and gone on with. Proof: the playground island made again this way by hand.
- **The generator of a beginning is a recipe, as data, over the reusable
  generators** (land, woods, meadow, street, placed things, and what comes): it adds
  ordinary nodes through the same commands the editor's buttons use, so everything
  after it is editing as usual. `playground`, which is such a recipe written in Rust
  for one island, becomes the first one written as data. **No generator is to be only
  one island's.**
- **Depends on**, in order: a blank document and the land's parameters said at its
  making; generator nodes added, placed, given an extent and removed from the editor
  (*corrected 2026-10-10: this said "the host's `generator` command has no panel".
  There was no such command: `generator` only swaps versions of a test generator. W1
  added the commands, see 7h*); more than one feature line, and a line begun by hand
  (3.1); the recipe as data; an asset browser over kinds, meshes,
  generators and recipes (section 4's Assets); from M3 as wanted, the built-in things
  as kinds and handles on a kind's part; and the editor's panels not covering each
  other, with one list of what was changed by hand.
- **Not its:** rules and behaviours (M4), surfaces as materials with standing cover,
  worlds larger than one island.

## 7e. The workspace: where every tool is found (the user, 2026-10-10; built the same day)

**Asked for before more engine work:** the editor had become panels and function keys
added one at a time; it is to be one application, with the world in the middle, every
tool found by looking, and room for the World-Building Workflow (7d) to be put into.
Built as the **UI foundation** (`PROGRESS.md`, "UI foundation — one workspace, tools
that can be found"). What it fixes for what comes after:

- **Navigation is by what is worked on**, in one bar: World, Environment, Things,
  Characters, Window; Undo, Redo, Play. **A new capability is an entry under one of
  these headings and, if it has a panel, a window** (`desk.add`): it does not get a
  key of its own first and a way to be found later. An entry that cannot be used now
  is shown greyed with the reason; an entry is never put there for something not built.
- **A second row is the context**: what is picked and what can be done with it, or the
  tool in use and the way out. What can be done to a selection is added there (and to
  the inspector), not as a new panel.
- **Editing and playing are two states of the one bar.** What is of use only in one is
  put away in the other and comes back.
- **A panel is a window** (`win.gd`): moved, folded, made another size, put away,
  brought back, the same for all. Floating; docking, if it is ever wanted, is a change
  to the frame and to nothing a window holds. Where windows are is kept per machine,
  apart from any world.
- **A number is set one way** (`num.gd`): a step less, the number, a step more, and a
  slider where a range is worth dragging. What sets a number of the document does so
  as one edit a step; a slider is for numbers that are not edits.
- **The UI calls what the keys called.** No panel's logic moved into the workspace, and
  the host gained one thing: `playground` takes the land's `amp` and `patches` with
  its seed, so that a new world can be asked for with them.
- **A list of what to try by hand is in the editor** (`checklist.txt`), written anew
  each milestone; a development aid, kept out of worlds and built games.

**How it serves W (7d).** W1's steps each have their place already: "New world" is
World's first entry and has a window, which today asks three numbers and is where an
outline, an empty beginning and a recipe will be asked for; generator nodes added and
placed belong under Environment and Things with the inspector as their window; a line
drawn belongs to the context row as a tool with a way out, as the land's tools and a
mesh held are; the walker's beginning and creatures belong under Characters; an asset
browser is a window. **The World-Building Workflow stands as written in 7d**: this
gave it somewhere to go and built none of it but the entry.

## 7f. Shaping by hand: what is wanted, what there is, what is owed (the user, 2026-10-10)

The user's way of working is a world generated and then gone over by hand, not long
modelling. The plan is to have the few tools that going-over needs, and no more until
use asks.

| Wanted | Is there | Where, or what is owed |
|---|---|---|
| A corner or a face moved | yes | Shape by hand (pick, then the row under the bar; or a mesh in Kinds and meshes) |
| A face pushed out on new corners (extrude along its own way) | yes | Ctrl + drag on a face (`push`) |
| Kept to X, Y or Z | yes, 2026-10-10 | the strip's row, or the keys X, Y, Z; in the thing's own frame. Shift: the world's upright |
| Kept to a plane | yes, 2026-10-10 | the strip's row: XZ (level), XY, YZ |
| Seen while it is done | partly | the line or lines it is kept to are drawn; how far it has gone is said in the strip. Owed: the number beside the pointer, and a grid to snap to |
| A move typed as numbers | partly | every drag is a line of the mesh's text (`move 9 by 1.06 0 0`), which can be written by hand in F6. Owed: three fields in the strip for the corner or face chosen |
| A thing put down, moved and turned by numbers | partly | right drag, `[ ]`, arrows. Owed: position, turn and size as fields in the inspector, with the same control as every other number |
| Several corners or faces chosen | partly | the reach takes those within so far. Owed: Shift + click to add, a ring or a loop |
| Bevel / chamfer | no | owed: a word of the mesh's text (`bevel EDGE BY`), then a tool |
| Extrude along a line that is drawn | no | owed, with W1's drawn path (7d) |
| Shapes joined into one, or one cut from another | no | owed: a mesh that names others (`join A B`); cutting is later and may never be wanted |
| Corners and faces of a kind's part, and of the built-in kinds | no | M3's remainder (5.3) |

**Order.** These are M3's remainder, taken as **M3b: the tools of going-over**, after
W1 unless W1 shows one of them is in its way: (1) position, turn and size as numbers in
the inspector, and a move as numbers in the strip; (2) several chosen at once; (3) bevel;
(4) join; (5) handles on a kind's part. Each a word of the text first, so that what a
tool does is written, undone and kept like everything else (7c).

## 7g. Behaviour: sequences, routines, conditions, profiles (a capability to come; the user, 2026-10-10)

**What is wanted.** NPCs, animals and whatever else has a body are set up from parts
that are written once and used many times, not from code written for each: **an action
sequence** (chop wood, open a door, get into a boat, greet, pet, sit, smoke: going
somewhere, a move, a pose, something done to a thing, a time, an end); **a routine** of
them (a day, a job: the lumberjack goes to the woodpile, chops, rests, goes home);
**reactions with conditions**, laid over a routine without replacing it (greet this one
otherwise than a stranger, stop for a cat, shelter from rain, rest at noon; where the
condition is not met the routine goes on); **the world read** (hour, season, weather,
who is near, what is to hand, what has happened); **rules for everyone, overridden by
one** (winter: warm clothes; each has their own); **profiles** shared by many and
changed for one (a walk, a greeting, idle poses, habits); and **moves, poses and
what is worn that fit the body** (not every action on every rig). And it is **authored**:
made, tried, changed, kept, named, sorted, found, copied, put together and used again in
the editor. A definition, its use by one character, and that character's changes to it
are three things, so that the definition can be bettered without the changes being lost.

**What there is already** (all of it content, read by code that knows none of it):

| Wanted | What Draw3D has | Short of it by |
|---|---|---|
| An action between a doer, a thing and an act | `acts.txt`, `acts.gd`: come, do, ride, leave, hold; a hand brought to a place; what it does to the thing as a fact | one step each: no sequence of several, no waiting, no "until" |
| Moves and poses as data, per kind of body | `moves.txt` (sets by name, phases, what ends them), `poses.txt` (per rig), `sway` | nothing; this is what a sequence's steps name |
| Someone who goes about | `folk.gd`: `goes HOUR x z`, `roams`, `shies`, `follows`, `idles`, `greets`, `offers` | each is one fixed habit with numbers; they are not composed, ordered or conditional |
| Finding a way | `nav.gd` (ways from what is solid, gates open and shut) | nothing for this |
| The hour and the weather | `clock.gd`, `weather.gd`, the style's `growth` | no seasons; no way for content to ask "is it raining" |
| What has happened | `state.gd` facts; `things.gd` `when X opens Y` | one rule form; this is M4's rule records |
| Who can do what, seen in one place | the hub (F4) | it shows who can do what and opens the line that says so; nothing is arranged in it |
| A definition and its uses | a kind and what is put down of it; `like` (a kind that is another but for some lines) | the pattern is there; behaviour is not yet a thing that has it |
| What is worn | a kind's parts | no sets of parts chosen by a rule |

**What Godot has, and what is used of it.** `AnimationTree` and its state machines
blend clips; Draw3D's bodies are posed by `figure.gd` from written poses, not clips, so
they stay the motor's and the figure's. Imported clips, when there are any, come in
under a pose's name. Behaviour trees and state machines for Godot exist as add-ons
(LimboAI: trees and hierarchical states, with an editor; Beehave: trees as nodes). They
are worth reading for their vocabulary (sequence, selector, condition, decorator, a
blackboard) and their debugger, and not taken as they are: they are scenes and
resources in Godot's own editor, and Draw3D's rule (principle O, and 7c) is that
content is text in the world's document, with identity, undone and kept with the world,
and that the host owns state. A tree kept in a `.tres` would be outside all of that.
So: **the words are borrowed, the interpreter is Draw3D's, and it is small**, because
the steps it runs are what there is already (go to, an act, a move, wait, a fact put).
No scripting language: lines of words, as moves and acts are.

**Kept apart, as they are now and must stay:** choosing what to do (new: the routine
and its reactions); the steps of doing it (new: the sequence; today `acts.gd`'s five);
showing it (`figure.gd`, poses); getting there (`nav.gd`, the motor); doing it to
something (`things.gd`, facts); what is so in the world (`state.gd`, the clock, the
weather).

**The shape of the data** (to be settled when it is built; written so that it is one
plan and not several):

- `sequence NAME` — steps, each one of: `go PLACE`, `act ACT with WHAT`, `move MOVE`,
  `wait S`, `put FACT`, `until COND`; with `wants RIG` or what it needs to hand, so that
  it is offered only to who can do it.
- `routine NAME` — entries `from HOUR do SEQUENCE at PLACE`, in order; `else` what is
  done with nowhere to be.
- `reaction NAME` — `when COND [near WHO R] do SEQUENCE`, with how pressing it is and
  how long before it may come again; laid over whatever routine there is.
- `cond` — words over what there is: the hour, the weather, the season (when there is
  one), a fact, who is within R, what is carried. The same words M4's rules use: one
  vocabulary, not two.
- `profile NAME` — a routine, reactions, a set of moves, idle and greeting, what is
  worn when; `like` another profile but for some lines.
- A kind names a profile; one put down may say lines of its own over it (the override
  record of 7a, used for behaviour as it is for shape). World-wide rules are reactions
  and wearing-rules the world itself names, which a profile or one put down may refuse.

**Order, and what it waits on.** It is M4 grown, not a milestone beside it:

1. **M4a, conditions and rules** (M4 as written in 5.4: rule records, the words of
   perception). Needed first: everything below asks conditions.
2. **M4b, sequences**: `acts.gd`'s steps made a list a line of text gives; today's acts
   become sequences of one or two steps, read by the same code. Proof: "chop wood" and
   "open the door and go in" written, with no new code for either.
3. **M4c, routines and reactions on the folk**: `goes`, `roams`, `idles`, `greets`,
   `follows`, `shies` become a routine and reactions written in those words (they stay
   as short forms). Proof: the lumberjack's day; he greets the keeper otherwise than
   the walker; he shelters when it rains.
4. **M4d, profiles, and what is worn**: a profile shared by three, one of them changed;
   sets of parts chosen by a rule (needs seasons, or uses the weather until there are).
5. **Authoring**: a window for sequences, routines and profiles on the workspace
   (the hub grown from showing to making): listed, found, copied, tried on someone in
   the view. Each step above has its text editable in F10 from the day it is built, as
   moves and acts are; the window comes when there is enough to arrange.

**Waits on:** W1 (people are put into a world that was begun by hand; a routine's
places are places in it); M4a. **Does not wait on:** imported animation, seasons,
relationships between characters (facts about two, which M4a's records can already be).
**Nothing of it is built**; nothing was begun in this pass. **2026-10-14 (7ag):** a
day as places by name (`goes HOUR at PLACE`, a structure; `at home`), and doors
opened and shut on the way by whoever could do the act: the short form a routine
will be written over. **Later the same day (7ah):** the first tier of M4a (conditions,
`cond.gd`) and M4b (sequences, `sequences.txt`, `work.gd`; a day's line `goes HOUR do
NAME`), not yet with today's acts rewritten as sequences. **Later still (7ai):** the
first tier of M4c: reactions (`reactions.txt`: `of`, `when`, `while`, `urgent`, `again`,
the steps of a sequence) laid over work for everyone with a body, with `face`, `move`,
`go walker`, `go indoors`; and talks with answers (`talks.txt`). Profiles (M4d) are
still unbuilt.

## 7h. Makers: how a world is composed (the user, 2026-10-10; W1 built the same day)

**The direction** is the master's principle P: first a broadly usable basic editor,
then its systems grown into specialised engines; the thing made kept apart from what
arranges it; makers that work together through connections said outright. What W1
fixed for everything after it, not to be decided again by the engines that come:

- **A maker is a generator node of the document.** The model the nine stages and M1
  to M3 built is the model: identity by key under the node, edit records, freeze,
  detach, undo as versions, one project file. What was missing was that only
  `playground` could reach it. Every operation on a maker is now a command of the
  host (`makers`, `nodes`, `add`, `compose`, `connect`, `disconnect`, `copy`,
  `remove`, `rename`, `move_node`, `turn_node`, `param`), and the editor's window and
  the playground both go by those and by nothing else.
- **What a kind of maker is, is said in one table** (`MAKERS`, `shapes/land.rs`): its
  inputs with what each takes, what it gives, whether it fills an area, whether a
  world has one. **A new engine is a generator and a line of that table**: it is then
  listed, added, connected, checked, made twice and taken out with no change to the
  host's commands or the editor.
- **Connections are by named inputs, checked by what is given.** An input takes one
  source, or, where its generator says so (`many`), several, each under a name of its
  own (`route`, `route+1`). Evaluation order, cycles and what is made again follow
  from the connections alone, as before.
- **Where and how far are the node's.** A maker's place and turn are its node's; one
  that fills an area has `reach` about that place. So placing, moving and the marks
  on the world are the same for every maker, and a tool in the view ends in one
  command (7c holds for makers as for shapes).
- **What arranges is apart from what is arranged, in the code as it stands:** `street`
  and `lot` both ask the `house` generator for a house; a wood asks for trees by kind.
  The house engine, the tree and rock engines replace what is asked, not who asks.
- **A composition is steps of `add`**, a later step naming an earlier maker as
  `@name`. The playground is one (`recipe`). A person's own, kept, named and placed
  as a group, is W2: it needs nothing new underneath.
- **Taking out is whole, and so is bringing back:** connections to a removed maker
  are removed with it and said; undo restores maker, settings, records, connections.

**What it leaves for the engines**, each a small thing to add when its engine comes:
an area that is not round; makers saying what ground they take, for others to keep
off; kinds of line (road, path, fence, river); a land with an outline; a maker whose
children are themselves customised and kept through its own regeneration (the
records and overrides of 3.3 already do this for a house's roof: the house engine is
where it is next needed).

## 7i. Design apart from distribution: how the engines are to be built (the user, 2026-10-10; W1b built the first step)

**The direction** is the master's principle Q: *Design → Save → Group → Generate or
Paint → Customise Instances → Reuse*. This section says how it sits on the code there
is, what was decided building W1b, and the milestones. None of the engines is built.

**What there is to build on, read from the code** (so that nothing is made twice):

| Wanted | What exists | What is missing |
|---|---|---|
| A design | A **kind** (M1): document text with parts, sizes, colours, traits, `like` another kind; a part may be a **mesh** (M3), also document text. Put down by `Place`. | A wood, a meadow and a street do not take kinds: they ask the code for "tree, variant 3". Eighteen things exist only as code. A kind's parts are boxes, blobs and meshes: no petal, blade or branch. |
| A group | Nothing. A meadow's shares of grass, flowers, bushes and rocks are four numbers of the world's style. | The asset, and an input that takes it. |
| A maker | W1: a generator node, listed, connected, placed (`MAKERS`). | A maker's "what" as an input (a design or group), beside its "where". |
| An instance | Identity by key under its maker; records kept through regeneration (`Pin` moved, `Set` a field, `Exclude` taken away); `detach` makes one a thing of its own. W1b: chosen, shown, counted and put back in the editor. | A record that says "of that design"; an instance's own numbers shown as such; "taken away" kept by name and not only by place. |
| Painting | The land's layer (M2): per-place values painted with a brush, kept per patch as document text; "grows here / not here" is already painted. | A layer that says *what* and *how thick*, and a maker that reads it. |
| An arrangement kept | `compose` and `recipe` (W1): steps of `add` with `@name`. | Saved by a person, named, put down as a group (the old W2). |
| Between worlds | Nothing: kinds and meshes live in one world's document. | A file of assets a world brings in. |

**Decided building W1b**, for every engine after it:

- **One thing is chosen at a time, and it is a maker or one instance.** The view, the
  bar, Makers and the inspector show the one selection. An engine's own window (a
  flower designer) will add a third sort, *a design*, to the same selection; it is not
  to have a selection of its own.
- **What is done to one instance is a record of its maker**, of a sort the host names
  (`changes`: taken away, moved, turned, changed), counted in `nodes`, taken back by
  `restore`. "Of another design" and "its own numbers" are to be two more sorts of the
  same thing, not a second mechanism.
- **A maker reads the ground and a line through its frame, turn and all.** A maker
  written for a new engine uses `Ground` and `line_of` and gets this; it does not
  compute world places itself.
- **What a maker's input means to it is said in its entry** (`Maker::line`). The same
  is owed for a design or group input when there is one.

**Decided building W1c**, for every engine after it:

- **Three words, and no more hierarchy:** a *thing* has a *sort* (rock, boat, a kind's
  name: `land::sort_of`); it *belongs to* a *maker*, which has a name of its own; and
  the maker *is* something (Meadow). A group of designs (D3) is an asset a maker is
  given, not a fourth level of what is selected.
- **A change by hand is of a thing by its key, never by its place.** `settle` is where
  every maker of the land gets this; a maker written for an engine goes through it.
  What a key means when a maker lays its things out anew (a seed) is the maker's to
  say, and is said to the person; D2 is to decide whether a seed belongs in the key.
- **Another of one instance is a thing of its own** (`copy_one`). "One more of this
  maker's, here" is the core's `Add` record and is D2's.
- **What a thing rests on is asked of `Ground::rest`**: the ground, or the water for
  what floats. A design that floats says so (`floats`, for a kind). An engine does
  not compute a height from the water itself.
- **What is said of the one selected is in parts that fold**, the instance's first and
  apart from the maker's. An engine's controls are parts of the same window.

**Decided building D1**, for every engine after it (the user's answers of 2026-10-11
are the first three):

- **The words, by context.** A *maker* is what creates things and controls how they
  are generated. An *arrangement* (or group) is the collection it created. A *thing*
  is one object; its *sort* is what kind it is. "This rock belongs to the Meadow
  arrangement made by the maker 'meadow'." It is not a level of selection: one thing,
  or one maker, is selected, as W1b decided.
- **A group need not have a generator.** A group assembled by hand is as much a group
  as one a maker made (D3 is to build it so: the asset first, a maker one of its users).
- **Duplicating an instance and adding to a maker are two actions.** The first is a
  thing of its own (`copy_one`); the second changes the maker's contents (D2).
- **Removing a maker says what goes**: "the entire maker and its generated contents",
  and is confirmed. An engine's own destructive actions say theirs the same way.
- **A design is said by name** (`land::Design`: a name, which of its sort, its parts
  of the whole), and there is **one list** (`designs`). An engine adds designs to that
  list; it does not keep a list of its own.
- **A maker's "what" is its parameter `of`**: a list of designs with parts. Empty is
  the maker's usual. A new maker takes designs by declaring `of` and asking
  `Given::read` and `Given::stood`, and stands what it gets with `Stood::site`.
- **A place a maker finds is named for the place, not for what stands there**, so that
  what is done by hand to it survives a change of design.
- **What the code draws is not changed by a person**: a design to change is a kind
  made after it (`design_own`), and E1's and E2's designs are to be kinds with parts
  a person sets, listed the same way.

**The milestones** (the order is by dependency; each is reached through the same
selection, commands and windows, and none adds a world arrangement of its own):

| | Milestone | What it gives a creator | Stands on |
|---|---|---|---|
| **W1b** | Choosing; one instance by itself | built 2026-10-10 | W1 |
| **W1c** | An instance says what it is; kept by name; another of one; the water | built 2026-10-10: a thing has a sort apart from its maker; "taken away" is of the thing by its key; another of one instance is a thing of its own, put down by a click; what floats rests on the water; the Makers window in parts | W1b |
| **D1** | A design is an asset a maker is given | built 2026-10-11: one list of designs (the code's twenty things, the world's kinds); a wood, a meadow and a street given designs with parts (`of`), keeping their places and names; Place by the same list; one of the code's made a kind to change (`design_own`); a thing says its design | M1 kinds, W1 inputs |
| **D2** | One instance changed beyond its place | "This house is that design"; "this tree is taller"; and, as an action of its own and not a mode of Duplicate, "add another rock to this meadow"; kept, listed, put back. To decide: a meadow's places named for the place, as a wood's are | D1, W1b and W1c records |
| **D3** | Groups, and assets between worlds | The list a maker is given (`of`: designs with parts), named and kept as an asset several makers use; a group assembled by hand, which needs no maker to exist; saved to a file another world opens; the asset browser | D1 |
| **D4** | Painting | A brush with radius, density and spacing that puts down the chosen design or group and rubs it out; makers that fill an area stay beside it | D1 (D3 for groups), M2's layer |
| **E1a** | Flower and grass design | Petals (count, shape, size, colour, arrangement), centre, stem (height, thickness, bend), leaves; grass as blades in tufts (length, width, bend, spread, colour and its variation). Built-in designs that read as flowers and grass. | D1; a new part sort in a kind (`petals`, `blades`) |
| **E1b** | Tree, bush and rock design | Trunk (height, thickness, lean), branching (how many, how they spread), foliage (form, volume, colour); pine-like and oak-like as starting designs, not a closed list; rocks with a silhouette | E1a's way of making parts |
| **E1c** | Meadows and woods of designs | A meadow from a group with density, shares and spacing; one flower varied in height and age; woods by ground and height, with clearings and edges | D3, D4, E1a, E1b |
| **E2** | House design | Floors, roof form, windows, doors, wall material and colour; porch and garage as parts added on; a street arranges designs; one house of it changed by itself | D1, D2; the `house` generator a street and a `lot` already ask |
| **E3** | Decorations, gardens, arrangements | A fence, a bench, a path as designs; a garden as an arrangement of makers that draws on the plant and rock designs; put down beside a house, kept and used again | D3, E1, E2, W1 `compose` |
| **Later** | Wind, seasons, weather, giving way | Grass and leaves moving; what grows changing by season and weather; bending to what walks through | each as the rendering, animation, physics and weather for it mature |

**Variation, said once** (it is the thing most easily got wrong): a design has
numbers; a maker or a brush may be told a *range* for some of them (height 0.8 to
1.3, hue a little either way, age), and gives each instance a value in it from the
instance's own key, so that it is the same every time it is made. That is variation,
and the instance is still that design. A different petal shape is a different design.
An instance may also be given a value by hand (D2), which is kept as a record.

**Grass, said once:** blades with a shape, many to a tuft, tufts drawn many at once
(one mesh, many places), so that a painted lawn is affordable; movement in the shader
when wind comes. No simulation of a blade is to be built before there is wind to
drive it. A flat green ground with no blades stays what the land looks like far off.

**Not decided, for the user:** whether E1 (plants) or E2 (houses) comes first after
D1 and D2; whether the designer is a window of numbers with the thing shown beside
it (as kinds are now) or works on the thing in the view; and whether painting (D4)
is wanted before any design engine, with the designs there are.

## 7j. Where things grow, one thing by itself, and a style that changes shape (the user, 2026-10-11; the first part of each built the same day)

The user's consolidated brief after trying D1. What it asked that is a decision for
every engine after it is written here; what was built is in `PROGRESS.md`.

**Decided building it.**

- **A maker that fills an area says where in three ways, each its own**: how far it
  reaches (a number, a ring); the level it keeps to (`level`: ground within so much
  of the height its own place stands at); and where it was painted (an asset of the
  document by the maker's id, `area.ID`: squares 2 m a side in the maker's own frame).
  A new maker that fills an area declares `reach` and `level`, reads `Area`, and
  asks `Area::painted` before it lays places out itself. Painted, its count is how
  many there are over all that is painted, and a place is named for its square: so
  painting more moves nothing that stands.
- **A reach is measured on the maker's own level**, in the view, and a reach changed
  keeps the maker as thick as it was (its count goes with the ground it covers).
- **What a maker keeps of single things is the maker's own data**: two parameters,
  `each` (a place given a design or a size, by its key) and `added` (things put down
  by hand as more of it, each with a name). They are written, opened, undone and copied
  with the maker, applied to the sites its plan lays out (`by_hand`), listed with the
  records (`changes`) and taken back the same way (`restore`). The core's `Add`
  record is not used for this: it carries a place and a size and not what the thing
  is, and a thing added by hand is first of all a design.
- **A thing shaped by itself has a design of its own** (`one_own`): a copy of the
  design it had, which nothing else is of. A shape is never kept on an instance
  apart from a design; so "this one only" and "every one of this design" are told
  apart by which design is being shaped, and the editor says which, with how many
  things follow.
- **A style gives a maker designs the same way a person does**: its lines `trees` and
  `strewn` are what a wood and a meadow are made of when they were given none of
  their own (`Given::read`'s `usual`). A style names only designs every world has.
- **Selection colours are roles, not colours** (`tint.gd`): a maker, one thing, the
  rest of its arrangement, what is new, a handle, a sketch. A tool asks for the
  colour of what it marks. The choice is this machine's, never a world's.

**Kept for later, in the user's words where they gave them** (master roadmap,
principles Q and R):

- Reusable designs shared by makers and worlds; named groups of designs used apart
  from any one maker; designs or groups painted onto terrain with a brush; individual
  things that differ from their maker's output; compositions of several kinds of
  maker and reusable designs.
- "Streets and other structural makers should eventually support houses alongside
  additional objects such as trees, rocks, and decorations. Adding a design should not
  silently destroy unrelated output; replacement should be an explicit operation."
  Today a street given designs has no houses (D1, recorded as not done): this is the
  requirement that replaces that.
- A shared Variation System (principle R): of single designs, of groups and
  compositions, of population and structure; locks; alternatives previewed and
  accepted or rejected without destroying what there is; seeds, weights, constraints;
  fauna presets by the same means.
- Maze and obstacle-course generators: specialised, later, on shared infrastructure.
- Trees with real structure and foliage that can answer wind; houses, decorations,
  terrain-aware distribution, wind, seasons, weather, and the animation and
  interaction that go with them. The conifers of 2026-10-11 are a shape made by code
  with six outlines: they test whether a style can change what stands, and are not the
  tree engine (E1b), which is to give a tree parts a person sets.

**The milestones after it** (replacing D2's row in the table above, which is now
partly built):

| | Milestone | What it gives a creator | Stands on |
|---|---|---|---|
| **D2** | One instance changed beyond its place | built in part 2026-10-11: one thing given another design, a size, or a design of its own to shape; one more added to an arrangement by hand; all kept by name, listed, put back. **Owed:** one thing's other numbers (colour, height apart from size); a meadow's places named for the place; a house of a street given a size | D1 |
| **A1** | Where a maker makes its things | built in part 2026-10-11: reach on its own level, keep to a level, a painted area. **Owed:** an area by slope, height band, surface or distance from a line; an area shared by several makers; the painted area shown filled, not as an outline | M2's layer, W1 |
| **S1** | A style that changes what stands | built in part 2026-10-11: `trees` and `strewn`, five designs. **Owed:** a style for one region and not the whole world; a style's lists edited in the editor and not in a file; groups (D3) as what a style names | D1, stage 7 |

## 7k. What the editor reaches, and a world's files (the user, 2026-10-11; built the same day)

The user's brief before a window of hands-on time: expose what the engine already
does, make saving dependable, keep the long-term requirements in view. What was built
is in `PROGRESS.md` ("Capability exposure, and worlds kept safe"); what the engine can
do and whether the editor reaches it is `Draw3D_UI_Inventory.md` section 14, to be
kept true each pass.

**Decided building it.**

- **A world's file is never written over without what it had being kept** (a copy in
  `backups` beside it, by the time it was written; a save that cannot keep does not
  write). Saving does not ask. A world is written whole to a part file and put in
  place. An earlier save, a demo's world and a new world have no file.
- **Which world is open is a person's context, kept on their machine**: the editor
  starts on the last one, and says which on the bar. A script has no such memory.
- **A maker's parameters are shown under words, with what each does, where the maker
  is edited** (Makers). The words are the editor's (`SETTINGS` in `makers.gd`), the
  parameters the host's: a new maker, or a new parameter, is given its words in the
  same pass, or it is reachable only in the Inspector under the engine's name. When
  the engines (E1 to E3) give designs parameters, the same holds for them, and the
  words should then come from the content (the design says what its numbers are
  called), not from a table in the editor.
- **A command of the host that no control reaches is listed as such** (inventory 14 C)
  with why, and where it would belong: not left to be found again.

**The long-term requirements the user restated, and where each is written:**

| Requirement | Where |
|---|---|
| Reusable groups and arrangements, edited and reused across makers and environments | master, principle Q (Groups; Arrangements kept); milestones D3, E3 |
| Placement and painting regions of any shape, beyond a round reach | master, principle Q ("Where a maker makes its things"); A1's owed part (an outline drawn; by slope, height, surface, distance from a line; shared by several makers) |
| Fuller customisation of one instance, and clearer tracking of what was changed | master, principle Q ("One thing may differ"); D2's owed part; the list of single changes begun 2026-10-11 (one maker at a time; one list over a world is owed) |
| Styles that reach geometry, structure and composition | master, principle Q ("A style says what things are made of"); S1 |
| A style chosen for each category or object, mixed and matched | master, principle Q (**added 2026-10-11**: "A style is chosen for each sort of thing") |
| One shared, controlled variation system for designs, groups, compositions, populations | master, principle R |
| Shared colour variation that keeps authored palettes, in set proportions, with an instance's palette wholly its own | master, principle R (**added 2026-10-11**: "Colour varied as one system") |
| Streets and neighbourhoods that hold houses with trees, rocks and decorations | master, principle Q ("A maker that arranges structure holds more than one sort of thing"); today a street given designs has no houses |
| Maze and obstacle-course generators, and other specialised tools | amendment 7j ("Kept for later"); master, principle P ("Specialised engines") |

None of them was begun in this pass.

## 7l. A town to work in, and what building it decided (the user, 2026-10-11; built the same day)

The user asked for a small living town on the test island as their main place to try
the editor in. What was built is in `PROGRESS.md` ("The island town, and what lives
in it"). Five things were decided in building it, each the first tier of something
this amendment already names, and each the user's to change:

1. **A world is built by a script only as a person would build it.** The town is put
   together by the host's own commands, in the order a person would give them
   (`editor/host/examples/town.rs`), and what is left is a world file. There is no
   town generator, and nothing reads the script afterwards. This is the composition of
   7h said as a program, until a composition can be kept and named (E3).
2. **A building is a design before it is an engine's.** Until the house engine (E2), a
   house that differs in outline from the street's own is a kind: parts and meshes as
   text, given to a street as any design is. What that cannot say (storeys and roofs
   as numbers, an inside, a collider that follows the parts) is what E2 is for, and is
   listed in PROGRESS.md.
3. **What is alive and has no body is content too.** Four words a kind may say
   (`flutters`, `swims`, `hops`, `perches`), shown going about by one script
   (`wild.gd`) while the world runs. They are the ambient tier of principle N's
   creatures: no body, no way-finding, nothing saved. A bird with bones that lands
   where it chooses is the next tier, and waits for behaviour (7g).
4. **Standing water is a thing put down in a hollow.** `sheer` (seen through) and
   `water R H` (there is water here) are said by a kind; the hollow is the land's own
   layer. This is not the water system of the master's section 9 (level, flow, shore,
   swimming in it): it is one sheet, and it was chosen because the sea is one height
   everywhere and the town stands six metres over it.
5. **Tending is the first sequence.** `tends SORT R MOVE [S]` is go to one, face it,
   do a move for a while, go to another: one fixed sequence of the steps there are
   (a way, a facing, a move asked by name), with its three places filled in by
   content. It is the shape 7g's action sequences are to have, written as one word
   until sequences can be written; it changes nothing in the world, and when M4
   brings conditions and effects, `tends` becomes one sequence among those, not a
   word of its own.

Kept for later, and not begun: the game built to run by itself from the town. (The
obstacle course was built the same day, at the user's asking: 7m.)

## 7m. A training ground, children and a tractor (the user, 2026-10-11; built the same day)

Asked for after the town: the obstacle course and the wall to wall check, a basic
parkour set of moves for the character and a course in the town to show it, children
at play with bubbles and toys, and an old tractor under ivy with what lives about it.
What was built is in `PROGRESS.md` ("The training ground, the children and the old
tractor"). What it decided, each the user's to change:

1. **A course is content, and shows the numbers.** Its pieces are kinds, one block
   each, as high and as far apart as the walker's numbers make each move possible:
   nothing in a piece says what is done at it. This is principle N held to: a thing
   to climb is whatever is solid. The price is said in the kinds' own text: change a
   number in `feel.cfg` and a piece may no longer be what it was for.
2. **Wall to wall needed nothing.** A jump off a wall and a hold to a wall were
   written in stage "Walls and edges"; two walls 2.6 m apart make them a way up. It
   is recorded as a property of those two moves and their numbers, not as a move.
3. **Three moves were written, and each changes what the walker did before.** At a
   run, what is low is gone over without being asked (`vault`: a garden fence too);
   held to a wall, jump with the way asked into it steps up the wall first
   (`wall_run`), and only otherwise pushes off; crouched with no way asked, jump is a
   backflip and no longer a long jump the way it faces. Each is one entry of
   `moves.txt` and is taken out of the walker's set by deleting its name there. The
   three words they needed (`running`, `still`, `low`) are the motor's and the
   grasp's: the first tier of "what a body can sense of where it is", which a slide,
   a run along a wall and a swing would each add one more word to.
4. **A child is a size and a pace.** `scale` and a set of moves with its own
   numbers, and one word (`runs`). What a child does at a toy is `tends` as the
   woodcutter's is, so it has the woodcutter's limit: nothing is done to the toy.
5. **A bubble is a fifth way of being alive without a body** (`drifts`), beside
   7l's four, and nothing joins it to who blows it but where both were put. When
   behaviour (7g) has effects, a blow makes its bubbles.
6. **The tractor is a prop made of parts**, a hundred and thirty lines of one kind.
   It is the largest thing written as a kind so far, and shows the want of two
   things this amendment already plans: parts turned (a wheel is a limb laid on its
   side; nothing can be tilted but a limb), and one design kept inside another (its
   ivy and its nest are its own lines, not designs put on it): D3's groups.

## 7n. The library: designs kept apart from any world (the user, 2026-10-11; built the same day)

Asked for: a visual asset library to create, browse, customize, save and reuse
designs across worlds, a rural and a contrasting urban starter collection, and two
showcase worlds. What was built is in `PROGRESS.md` ("The library: designs kept apart
from any world, and two worlds built of them"). It is the **Save** and **Reuse** of
principle Q ("Design → Save → Group → Generate or Paint → Customize → Reuse") and the
"assets between worlds" half of D3. What it decided, each the user's to change:

1. **The library is the text a world keeps, in a folder.** No second form of a
   design exists: a kind and its meshes are written to a file as they stand in the
   document and read back the same. So a design is compared, mended and passed on by
   hand, and everything that reads a kind already reads a library design.
2. **A world copies; it never refers.** Bringing a design in writes its text into
   the document. A world then opens anywhere, with or without a library, and what
   Draw3D comes with can be redrawn later without any world changing under its
   maker. The price: an improved original does not reach worlds that have the old
   one, until they ask ("Replace this world's with the library's").
3. **Three things are kept apart and named wherever one is changed**: the library's
   design (a file), this world's design (every thing of it follows), one thing
   (D2). Originals are never written: a person's copy has another name.
4. **A copy shares nothing**: its meshes are written again under its own name. A
   kind LIKE another is the other way of making a second design, and follows it.
5. **Customizing is saying, not typing**: a colour, a size, proportions, each a
   line of the kind's text rewritten by the host (`kind_tune`, the kind line
   `stretch`). It is the first tier of D2's "variation of one design": the numbers
   are the design's, not yet one thing's or a maker's.
6. **A collection is a look as well as a list.** The rural one is written in the
   style's colours and round; the urban one in colours of its own and straight. That
   is as far as "style" goes for now: see below.
7. **A showcase is built by the host's own commands** (`showcase.rs`), as the town
   is: nothing in either world could not have been done from the editor.

**Not built, and where each belongs.** Groups (a café with its tables kept and
placed as one; the user's "a group shouldn't need a generator"): D3, unchanged.
Painting designs onto land: D4. Parts added and moved in the view: M3's handles on
a kind's part, still owed. Pieces that follow a line (a road, a kerb, a fence): a
structural maker given designs (principle Q). Things that move by themselves (a
swing, sails, a traffic light): behaviour, 7g.

**From here to a style that is a whole world's look.** The user's direction: "complete
world aesthetics rather than isolated alternate models", spanning geometry,
vegetation, architecture, materials, lighting and atmosphere. What there is: a style
says colours, a roof's pitch, the hours and the light, and what a wood and a meadow
are made of (7j); a library design follows it only where its parts are written in
its colours. The steps, in the order their dependencies give, none begun:

| Step | What it adds | Needs |
|---|---|---|
| S2. A style names designs for more than woods | "the street lamp of this style", "its bench", "its house": a style line that says which library design stands for a role, so a maker asks for a role and a restyled world changes its lamps and its fences, not only their colour | roles named by makers; the library (built) |
| S3. Materials | a part says what it is made of (matt, metal, glass, lit), and a style says how each is drawn; lamps and signs that light by the hour | a second material in the content shader; `lights` for kinds |
| S4. Proportion as style | a style's numbers reach designs: how round a crown is, how thick a trunk, how tall a storey, as `stretch` and `scale` do for one design now | variation of a design by numbers (principle R) |
| S5. Light and air per style | `city` and others say their own hours, haze and night | written already for four styles; only content is owed |
| S6. A collection carries its style | a library file says the style it is drawn for, and offers to give a new world that style when its designs are first brought in | S2 to S5 |

It is not claimed that any of these exists. `city` is colours only.

## 7o. Spatial layout, tiles, modular construction: three milestones (the user, 2026-10-11; L1 and L2 built 2026-10-12; L3 first tier 2026-10-14, 7ad; L3 brought to a stopping point the same day, 7af)

**The direction.** Beside freeform editing, sculpted land and makers, a second way of
building: **structured spatial layouts**, to carry tile-based worlds, grid or hex
gameplay, and modular construction that fits itself together. Three milestones, in
this order, and not at once:

| | Milestone | What it is |
|---|---|---|
| **L1** | Spatial layout | cells to build on, square or hex; guides; things snapped to cells. **Built** |
| **L2** | Tile sets and placement rules | sets of pieces made for a cell, and rules for which may go where and beside what. **First tier built**: terrain tiles (below) |
| **L3** | Intelligent modular construction | buildings, gardens and other structures that adapt: walls that join, corners that turn, roofs that span. **First tier built** (7ad): a structure on cells, walls, fences, hedges, gates and floors. **Brought to a stopping point** (7af): buildings with rooms, doors and roofs, ponds, steps and ramps, shared sides, a village |

**What a layout is (decided building L1).**

- **A layout is a way of saying where, not content.** It has a form, a cell size, a
  place, a turn, and so many cells. A thing "in a cell" is an ordinary thing of the
  world that stands there; the layout says which cell from where the thing stands.
  Nothing is stored in a layout, so removing one loses nothing, and everything the
  editor does to a thing it does to a thing in a cell.
- **One module answers every question of cells** (`editor/host/src/layout.rs`): cell
  of a place, middle, corners, neighbours, steps apart, sides, the heading that faces
  a neighbour. The editor asks the host and has no cell arithmetic of its own. **L2
  and L3 are to ask the same module and keep none either.**
- **A cell is `[column, row]` in both forms**; hexes also give axial `[q, r]`. **Cell
  size means middle to middle** in both forms. Hexes are pointy-top with odd rows
  half a cell along; flat-top is the layout turned 30 degrees.
- **The layouts are one text of the world's document**, a line each. A world may
  have several. Which one is snapped to is the person's, kept on their machine.
- **Changing a layout may carry what stands in it** (each thing to where its cell
  then is), as one edit; or not, and then only the guides change.

**What L2 (tile sets and placement rules) can build on, and what it still needs** (as written after L1; what became of each row is in the table after it).

| L2 needs | What L1 gives | What is owed |
|---|---|---|
| where a tile goes | `cell`, `centre`, `heading`; `place` with a `yaw` | nothing |
| what is in a cell, and beside it | `layout_cells`, `cell` (`holds`, `neighbours`), `steps` | **occupancy by what a tile is, not only that something is there**: a tile's set and name read from the thing (its design), so a rule can ask "what is north of here" |
| a tile | a design (a kind: text, library, pictures, customizing all exist) | **a design that says it is a tile**: the cell size it is made for, which of its sides join what (a word per side), whether it may be turned |
| a tile set | the library's collections and groups | a set as a named list of tiles made for one form and size, and a layout that says which set it takes |
| painting many | the area brush of makers (`marks.gd` "paint") | a brush that puts a tile in every cell it passes, and a fill |
| rules | nothing | **the rules themselves**: allowed neighbours by side; a tile chosen from what its neighbours are (an edge, a corner, an inside piece); what to do when no tile fits. To be data in the set, not code |
| more than one thing a cell | the "more than one to a cell" switch | **layers**: ground, what stands on it, what hangs over it, each one to a cell |
| edges and corners | `corners`, the guides' sides | **a side and a corner as places things stand** (a wall on an edge, a post on a corner), named like cells |

**L2 as built (2026-10-12): terrain tiles.** The user's brief for L2 drew the line
that decides it: a layout's **cell** is somewhere to stand ordinary things, several
to a cell, each edited by itself; a **tile** is "a reusable piece of a landscape …
terrain and its authored contents", put into neighbouring cells, selected and
manipulated as a unit, its contents still editable. What was decided building it:

- **A tile put down is a maker** (`tile`), standing at the middle of its cell. Its
  arrangement is authored (its text says each thing and where) instead of scattered.
  So it is selected as makers are, each thing on it is changed alone as any maker's
  things are, and nothing new was needed to undo it or save it. **L3 is to follow
  this**: a building is a maker whose arrangement is cells.
- **Which cell a tile is in is where it stands** (as L1 decided for everything): no
  tile is stored in a layout. What is beside it, and which of its sides faces which
  neighbour, is asked of `layout.rs` (`beside`, `side_facing`); `tiles.rs` and
  `tileops.rs` have no arithmetic of cells.
- **Tile sets are files beside the library** (`library\tiles`), and a world has its
  own copy of each tile it uses (`tile.NAME`), as it has of each design.
- **A side says a word; a set says which words may meet** (`joins A B`). The rules
  are data of the set. A tile is turned to join, or refused.
- **After every edit the host tells each tile what is beside it** (in that edit), so
  nothing that moves, turns, removes or undoes a tile needs to know about tiles.
- **A tile's ground is a skin on the land: colour that blends, not height.** It is drawn
  through the land's own triangles (since the pass after L2), so it lies on the land
  however the land is shaped.
- **Cells hold several things** by default now; resizing keeps each thing in its own
  cell and reports what has no cell any more, moving nothing unasked.

**What of the L2 table above is now built, and what is still owed.**

| L2 needed | State after L2 |
|---|---|
| where a tile goes | built (`tile_put` by cell; turned by the layout's turns) |
| what is in a cell, and beside it | built (`tiles`: each tile's cell, turn, and for each side the neighbour, what both say, whether they join; `cell` says the tile) |
| a design that says it is a tile | built otherwise: a tile is its own text (ground, sides, paths, things), not a kind |
| a tile set | built (a file; a set's cell size and rules) |
| painting many | built (a drag puts one in every cell passed; Fill the empty cells) |
| rules | **first tier only**: may two sides meet. **Owed:** a tile chosen from its neighbours (edge, corner, inside), weights, a region filled by rules, what to do when nothing fits beyond refusing |
| layers | **owed.** One tile to a cell; ordinary things on top |
| edges and corners as places | **owed** (L3 needs it: a wall on an edge) |

**Owed after L2, in the order use is likely to ask for it.**

1. **Ground with height.** A tile that is a knoll, a hollow, a terrace, a shore, with
   water as a ground. It needs the land to take tiles into account, so that the
   walker and what is put down by hand stand on a tile's ground. This is the large
   one, and the one "terrain tile" most promises. **What it is to be, what it rests on
   and what must be decided first is under "7o, continued" below** (the underlying
   terrain, a tile's elevation and a tile's surface kept apart).
2. **Blending beyond colour along sides**: across corners; things that thin or spill
   toward a neighbour (saplings into a meadow, stones onto snow); a transition tile
   chosen by the host.
3. **Authoring a tile in the editor**: a new one from nothing; its ground, its
   sides and its paths by controls; its text in a window.
4. **Rules that choose** (above), and layers.
5. **Several cells selected, moved and turned as a region** (the user, 2026-10-12:
   "record multi-cell selection and rotation of a region as a future spatial-layout
   capability. It is not a prerequisite for L2"). It belongs to the layout, not to
   tiles: a region of cells with whatever is in them (tiles and things), turned by
   the layout's own turns about a cell, moved by whole cells, with the same "nothing
   reassigned unasked" rule. Not begun.
6. **Tiles of more than one cell**, and tiles that follow the land in steps.

Not to be pulled into another pass unasked: biome generation, automatic terrain
blending beyond the above, buildings (L3), and anything a game plays on tiles.

**What L3 (intelligent modular construction) can build on, and what it still needs.**
It needs what L2 still owes of edges and corners as places, and then: pieces that are **generated to fit** (a wall
as long as its run of cells, an opening where a door is asked for), which is the
makers' work (principle P) given cells as their input in place of a line or an area;
**levels** (a layout above a layout); a structure as **one maker whose arrangement is
cells**, so that it is selected, moved, varied and regenerated as makers are, with
one cell changed by hand kept as a change by hand (the six ways of amendment 3); and
styles reaching its pieces (7n, S1 to S4). None of it exists. L3 is not to begin by
writing a building generator beside the layout: it is to be a maker that reads one.

**Kept for later, and not to be pulled into L2 without a reason:** a hexagon-shaped
or painted layout area; a layout that follows the land's height in steps (terraces);
handles to drag a layout in the view; grid-based movement or turns for a game played
on a layout (the layout is in the document, so a game can read it when one wants to);
generating land from tiles.

### 7o, continued. Tiles and the land: three things to keep apart (after the user's first use by hand, 2026-10-12)

**What use showed.** Woods and meadows put over a hex layout grow across its tiles;
the land's tools shape and paint the land *under* the tiles; where the land was raised
or lowered by much, a tile's ground came apart from it (the land came through it in
places and it stood off the land in others); and paint put on the land under a tile is
not seen. The third was a fault and is mended (below). The others are what a tile is
today, and say what it is to become.

**The direction (the user, 2026-10-12): three things, kept apart.**

| | What it is | Whose it is | Today |
|---|---|---|---|
| **The underlying terrain** | the land: its hills, and what was done to it by hand (raised, lowered, cliffs, paint, where things grow) | the land's; it stays where it is when a tile is moved | built (M2). One height for everything |
| **A tile's elevation** | how high a tile's ground stands, and its form: a tile or a group of tiles raised as a platform or a terrace; a landform a tile carries (a knoll, a hollow, a bank, a shore) | the tile's (or a group's); it goes with the tile when it is moved, turned or replaced | **not built.** A tile has no height: it lies on the land |
| **A tile's surface** | what its ground looks like: its sort (grass, forest, rock, sand, snow, water to come), its paths, paint put on *it* | the tile's; it goes with the tile | the sort and the paths are built; paint on a tile is not |

To come, on those three, in no fixed order and none of it begun:

1. **Raised groups and platforms**: several tiles lifted together, with sides where a
   neighbour is lower and none between two that are level.
2. **Landforms**: a tile whose ground has a form of its own (knoll, hollow, terrace),
   over whatever the land is under it.
3. **Water and shorelines**: water as a tile's ground at the tile's own level (a pond
   on a hill, not only the sea), and the shore where it meets a tile of land.
4. **Transitions between neighbours**: a ramp, a bank or a cliff where two tiles of
   different height meet; ground of one spilling onto the next; across corners as
   well as along sides.
5. **Decoration aimed at tiles or regions**: a wood or a meadow told to grow on these
   cells, or on tiles of this ground, or on a region of them, in place of a round
   reach or painted squares; and kept off what a tile already has standing on it.

**What was mended now, and why it is not the first of those.** A tile's ground is
still a skin on the land with no height of its own, but it is now drawn through the
very triangles the land is drawn with (a sample a metre, each cut in four, cut off at
the outline of the cell, a corner leaning where the land's leans under a cliff, lit as
the land is lit). So however the land under a tile is shaped, the ground lies on it.
Before, it was drawn through rings of its own about the tile's middle with the land's
exact height at each corner, which is not the height the land is *drawn* at between
its samples: measured on a tile over ground raised 4 m, the land came through at 13
of every 100 places looked at, by up to 15 cm, and the tile stood up to 32 cm off it
elsewhere. That is the land's tools working properly under tiles; it gives a tile no
elevation.

**How woods and meadows meet tiles today (looked at, not changed).**

- A wood or a meadow knows nothing of tiles or cells. It fills a round reach about its
  own place, or the squares (2 m) painted for it. Its things stand on the land, as a
  tile's own do; a tile's ground is 6 cm over that.
- **`count` is how many places are tried, not how many things are made.** Over a reach
  the outer third thins: about three in four are made (asked 200, made 150; asked 40,
  made 26; asked 12, made 8; asked 6, made 3; asked 3, made 2). Painted, each square
  takes its share by chance, so the number is about what was asked (asked 1, made 2;
  asked 3, made 4; asked 8, made 9). For a few things in exact places, put them down
  by hand or add them to the arrangement. Changing either rule would move every tree
  of every world there is; it stays.
- The land's "clear" brush and paint that stops growth, used under a tile, take a
  wood's or a meadow's things away there though the paint itself is hidden by the
  tile; a tile's own things are its text's and stay.
- A maker's things and a tile's things do not know of one another: a tree may stand in
  a tile's hut.

**Whether what is built leaves a way to all this (looked at in the code, 2026-10-12).**
It does; nothing found asks for a redesign. What the way rests on:

- **A tile is a maker**, so elevation and a surface are parameters and text of a
  tile, saved, undone and moved with it by what exists.
- **The host already tells every tile, after every edit, what is beside it**
  (`tileops::settle`, the parameter `beside`). Neighbours' heights can be told the
  same way, and that is what sides, ramps, banks and shores are made from.
- **A tile's ground is now made from a block of heights and an outline.** Given a
  height of its own, the same code draws it raised, and its rim (30 cm today) becomes
  its sides.
- **Nearly everything that asks "how high is the ground here" asks in one of two
  places**: `Ground` in `shapes/land.rs` (every maker: 26 uses of `height` and
  `rest`) and `heights_at` in the host (placing by hand, cells, the editor's tools).

**Risks that are real, to be decided before elevation is begun.**

1. **One height is assumed.** `Ground` and `heights_at` read the land alone. The day a
   tile has a height, things of *other* makers and things put down by hand on it must
   stand on the tile, and so must whoever walks there, or they sink into it. Both
   places must change together, in one pass. The way that fits what is built: the
   host keeps, as a text of the document made again after every edit (as `beside`
   is), **where each tile stands, its outline, its ground and its height**; `Ground`
   and `heights_at` read it beside the land's shape. No maker's connections change
   and nothing depends on itself (a tile reads the land; the land never reads a
   tile). The same text is what decoration aimed at tiles (5) needs. It is something
   worked out from where tiles stand and is never the record of it.
2. **The land's own layer is not the place for a tile's height.** It keeps a sample a
   metre and makes a face only of a step over 2 m. A platform 1 m high, a kerb, or a
   side that runs along a hex's edge cannot be said in it. Writing tiles' heights
   into the land's layer would be quick and would be a dead end, and it would leave
   the height behind when the tile moved. A tile's height is to be the tile's own
   ground.
3. **The water is one level for the whole world** (`WATER`, 0 m: the sea; six uses).
   What floats, what grows and what colour the bed is all go by it. Water on a tile
   at another height needs water to be a surface with a level of its own where it
   is; the sea stays the case with no tile.
4. **A tile knows the tiles along its sides, not across its corners.** For colour
   that is the faint seam already recorded. For height it would be a gap at the
   corner of a raised square. Hexes have no neighbours across corners; squares need
   the four more.
5. **Every tile is planned again at every stroke on the land, anywhere** (each reads
   every part of the land's layer). With 25 tiles a stroke took 8 ms in the host;
   with hundreds it will be felt. A tile is to read only the parts under it, as a
   patch of the land does.
6. **A group of tiles is nothing yet.** Raised groups, regions for decoration, and
   "several cells selected, moved and turned as one" (owed above) are the same
   missing thing: a set of cells that is something. It belongs to the layout.

**Not to be done to get there:** a second terrain system beside the land; heights
baked into the land under tiles; a generator for islands of tiles.

**What came of this**: the user's answers, the tearing that remained, and the first
tier of decoration aimed at tiles, are under 7p, next.

### 7p. The land under tiles, and woods on tiles (the user's second look at tiles, 2026-10-12)

**What the user said after the pass above.** The mended ground "is an improvement,
although some tearing remains". Four things asked for, and decided:

1. **Terrain editing must be seen over tiles**, and the two surfaces told apart:
   "Underlying land: terrain sculpting or painting affects the land beneath the tiles.
   Keep the brush indicator visible above the tile, and identify the active target.
   Tile surface: future tile-specific editing will affect the tile itself." And: "Do
   not silently redirect existing terrain operations onto tiles." **So paint under a
   tile stays the land's** (the question left open above is answered).
2. **The remaining tearing to be understood before anything is done to it**: no
   "cosmetic patch or a generic smoothing tool without understanding the cause".
3. **Next capability: tile-aware decoration and makers**: "select individual tiles or
   regions and distribute decorations over their surfaces, with controllable density,
   variation and exclusions", sparse and irregular, a small usable increment.
4. **Independent tile elevation stays the capability after that**, distinct from the
   land ("do not implement it by writing tile heights into the land layer"); water and
   shores to accommodate raised tiles later, no water system now.

**The tearing: what it was.** Four causes, found by making it happen in the editor
(a tour part that holds the brush with the mouse where tiles meet and takes pictures
while it is held) and by measuring what is drawn. None was in a world's data, and
none was what the last pass mended (that a tile's ground lies on the land's own
triangles: it does, to the millimetre, in every case measured here).

| What was seen | Its cause | What was done |
|---|---|---|
| **A slit in a steep hillside that the sea and far trees were seen through**, along a line where the ground was raised by more than 2 m in a metre. In the land itself and in any tile's ground on it. Smoothing closed it only by taking the steepness away: the user's "seam that smoothing does not repair" | The land's own mesh. A step of more than 2 m between samples is made a face by leaning the lower corner in under the higher (`Layer::lean`, M2). Between corners that lean differently a triangle can come to face *into* the hill; drawn from its front only, it was not drawn from outside | **The land and a tile's ground are drawn from both sides** (`world_view.gd`, `BOTH`). The land is one sheet with no hole; nothing of its mesh changed, and no world |
| **A dark strip along the side two tiles share**, on steep ground | Every tile's ground had a rim hanging 30 cm straight down on every side. Where the land comes out past what is above it, a rim between two tiles stood out of the neighbour's ground | **A rim only along a side with no tile beside it**; between two tiles each laps 2 cm onto the other, in the same triangles and the same colour (`turf`, `TURF_LAP`) |
| **Specks of the land's colour through a tile's ground** on a face | The ground was 6 cm over the land *straight up*: on a face that is no distance at all, and each showed through the other | **A tile's ground is drawn a little nearer the eye than it is** (1 cm near by, more with distance; not in the sun's shadow pass): it is in front of the land it lies on at every steepness (`nearer`, in the paint of `skin`) |
| **Land through the ground, or ground left standing off the land, while a brush was dragged** | A stroke makes the land's patch and every tile's ground over it again at each step; each arrived in its own frame and was swapped in as it came, and a tile's ground was not drawn at all until its new one arrived | **A tile's ground stands as it stood, like the land, and a cell's land and grounds all change in one frame**, when the last of them is here (`world_view.gd`, `STANDS`) |

Measured by the tour's part `seams`, at 625 places about the stroke, each frame
looked at: a tile's ground against the land under it *as both are drawn* is off by
0.000 m while a brush is held, after it is let go, after a spike of 12 m, smoothed,
restored, lowered 3 m under the water, smoothed at the shore, restored again, and
with every step undone the land's heights are what they were.

**What a tool works on is said, and is one place in the code.** The land's panel has a
line "Works on: the land. Under a terrain tile it works on the land beneath…". The
tool is drawn on a tile's ground too, **in stripes**, as something seen through the
tile; on the land itself it is drawn plain. With the pointer over a tile the panel
names the tile and says what the tool chosen does there (shape: the land under it, the
ground follows; paint: on the land under it, covered; clear and let grow: what woods
and meadows may grow, not the tile's own things; restore: the land, not the tile).
The host says which tile lies over a place (`ground` answers `tiles`). In the code the
target is `terrain.gd`'s `target` (only `land` today) and every command is made in
`asked`: a tool for a tile's own surface is another target there, with its own
command, and is never reached by a tool for the land.

**Makers on tiles: the first tier.** A wood or a meadow has a fourth way to say where
it makes its things (after: its reach, the level of its middle, where it was painted):

- **`on`**, what a person says: `layout NAME`, and of it `cells 1,2 3,2`, or
  `ground forest rock`, or `tile meadow`; with a layout and nothing else, every cell
  that holds a tile. The host's `grow_on` writes it (cells set, added, taken away;
  grounds; names; none).
- **`over`**, what that comes to, which **the host works out after every edit** and
  nobody else writes (`tileops::settle`): each cell's outline in the maker's own frame
  and what stands on the tile there. So a tile moved, replaced, changed to another
  ground or taken out, a layout moved or resized, the maker itself moved or turned,
  and the maker follows in the same edit, undone with it.
- **Density, variation, exclusions.** `count` is then **how many on each tile** (not
  places tried over an area): a place where nothing can stand is tried again
  elsewhere on the tile, so a few asked for are a few made. `uneven`, 0 to 1: how
  unlike one tile's share is to the next's (1: from none to twice). `clear`: how far
  to keep from what the tile already has standing, beyond that thing's own width.
  What is bare by hand, under water or by a road is kept off as ever, and one thing
  is still taken away, moved or given a design by itself.
- **Each thing is named by its cell**, so a tile added or taken away moves nothing on
  another.
- **In the editor**: Makers > Where it makes its things: **Choose tiles…** (click
  tiles in the view; again to take one away; Esc ends), **All tiles of…** (a layout,
  or its tiles of one ground), **Not on tiles**, and the two numbers.
- **Nothing that was there moved**: a maker told no tiles makes exactly what it made.
  The four parameters are written into a world only for makers made from now on; a
  world written before opens and is written again byte for byte.

*What it is the first tier of:* decoration aimed at tiles and regions (5, above).
*Not in it:* a region as something of its own (a named set of cells); choosing tiles
by dragging or by a box; density painted within a tile; a maker other than a wood or
a meadow; things standing on a tile's ground rather than on the land under it (6 cm
lower: the same as a tile's own things).

**Where each tile stands is now worked out in one place** (`tileops::standing`): its
cell, its outline in the world, its ground, what stands on it. The land's tools and
the makers on tiles both ask there. It is the record "7o, continued" said elevation
needs, begun: a tile's height, when it has one, is to be said there.

**Elevation: how height is asked today, looked at again (2026-10-12).**

| Who asks how high the ground is | Where | What it reads |
|---|---|---|
| Every maker, for where its things stand | `Ground::height` and `rest`, `shapes/land.rs`: 28 places | the land's shape |
| Putting down and moving by hand; a layout's cells | `heights_at`, the host | the land's shape |
| The land's tools, and the editor's readouts | the host's `ground` | the land's shape; and now which tile is over the place |
| A tile's ground | `turf_land` | the land's samples under its cell |
| Makers on tiles | `over` | each cell's outline; no height yet |
| Whoever walks | the land's mesh, as a solid (`solid.gd`) | a tile's ground is no solid: it lies on one |
| What floats, what grows near the shore, the sea's bed | `WATER`, 0 m, in the host: 6 places; the editor's sea is a sheet at 0 and the walker floats below 0 (`main.gd`, `motor.gd`) | one level for the world |

Nothing found makes elevation unsafe to add. What it will cost is what was listed
(the six risks stand), and three things are now nearer:

- A raised tile's ground must be **a solid**, or it is walked through: the same line
  the land's mesh has (`collider mesh`), for a ground with a height.
- **`over` can carry a cell's height**, so makers on tiles stand on the tile's ground
  the day it has one, with no other change to them. Every other maker still needs
  the one question of height that asks the land and `standing` together (risk 1).
- What is drawn is ready for it: a tile's ground has its own paint, stands and
  changes with the land in one frame, and a tool says which surface it is on.

Still to be decided before it is begun, as before: tiles raised one at a time or as
a group (risk 6); what a side is where a neighbour is lower (rim, bank, ramp or
face); water with a level of its own (risk 3).

**Not done in this pass.** No tile has a height, a form or water. Paint is not put on
a tile. The colours where four tiles of different ground meet still show a short
straight edge at the corner (risk 4: a tile knows its sides, not its corners). Things
on steeply shaped ground still stand 15 to 30 cm off the land as it is drawn. Every
tile is still made again at every step of a stroke (risk 5): about 70 owners a step on
the showcase, which is why a brush held under tiles lags. Something standing in front
of a brush's outline still hides it.

### 7p, continued. What still flickered, and the stripes (the user's third look, 2026-10-12)

**What the user said.** Seams "generally improved, but tile edges can still flicker or
become distorted while editing. Flattening previously raised land seems particularly
susceptible to temporary tearing." And the stripes of a tool on a tile "obscure too
much of the underlying surface". To be understood before anything was done, and not
hidden "with arbitrary offsets".

**How it was made to happen.** The tour's part `seams` was given a step that looks at
what is drawn *in every frame* of a stroke (the pass before looked a few times a
second, and saw nothing): a hump where four patches of the land meet, with tiles over
it, then wide brushes (flatten, raise, lower) dragged over it and back with the mouse.
At 81 places a tile's ground was compared with the land under it as both were drawn.
Before anything was changed: one frame or more off in about one stroke of two, by 0.2
to 1.1 m.

**It was two things, both in what the view draws while a mesh is awaited; none in a
world's data, the host, or depth.**

| What was seen | Its cause | What was done |
|---|---|---|
| A tile's ground jumping up or down for a frame at a step of a stroke, by as much as the land under the tile's middle had moved: most where raised land is flattened | The pass before made a tile's ground "stand as it stood" until its new mesh arrives. It was drawn with its old mesh **where the tile is now**: and a tile is kept at the height of the land under its middle, so every step of a stroke moves it. Old shape, new place | **What stands as it stood stands where it stood** (`world_view.gd`, `stood_at`): the place it was drawn at is kept with it. A ray asks the same (`pick`) |
| A tile's ground off the land along one side of the line between two patches, for a frame | Land and grounds changed together *in one cell of the view* (32 m). A tile can lie across the line between two cells, and a stroke over the line makes the land again on both sides: one cell changed a frame before the other | **Every cell a stroke touches changes in one frame**, when the last mesh of any of them is here (`unsure`, `gated`, `_all_here`); a frame of cells is looked through before the first of them is put down. When more than 16 cells wait at once (the land made again, not a stroke) each cell changes as its own arrives, as before, until none waits (`MANY`, `apart`) |

After: no frame off in any stroke of that step, in four runs of five strokes each
(about 200 changes of what is drawn in a run).

**One thing this broke, found by the tour and mended before it was kept.** Holding
back every cell until the last had arrived was at first done for the land made
again as well (a road's line changed makes all of it again): its last sixteen cells
were then drawn in one long frame, and `-Tour -Only land` failed in three runs of
seven at "a place of the road's line taken away" (the line still had it), where the
committed view passed four of four. Why a long frame undid that step was not found.
The holding back is now for strokes only; five runs of five then passed.

**The stripes.** They were half the brush, half a metre apart, as strong as the plain
tint. They are now **thin, a metre apart and fainter** (about a quarter of the brush
covered), with soft edges; the outline is as it was, and the tool on bare land is
plain as before. No control was added.

**Not done.** What stands *on* a tile (its trees, its hut) goes to its new height at
once while the ground under it waits a frame or two: a thing can be seen a step off
the ground for that long. A stroke under many tiles still lags (risk 5), and waiting
for every cell of a stroke makes it no quicker. Not tried by hand.

## 7q. Construction tools and generators: a family, not a brush and a maker (the user, 2026-10-12; a direction: nothing of it is built)

**The user's direction, in their words.** "Draw3D needs a family of specialized
construction tools and reusable generators, not just basic terrain sculpting and
general-purpose makers." Raise, lower, smooth, flatten and the cliff and ledge stamps
"are useful for rough world layout, but detailed level design needs more precise,
intention-driven tools."

**A. Terrain-shaping tools that say what is meant.** Examples the user gave, each a
tool to come:

- **Ridge**: a traversable ridge or shelf pulled out from a mountain's side; width,
  height, falloff, shape.
- **Ridge around a mountain**: a winding or spiralling way round a chosen mountain;
  width, number of turns, elevation, spacing; then refined by hand.
- **Ledge or platform on a wall**: a shelf standing out level from a cliff's face,
  apart from the ledge stamp, which is made from the ground up.
- **Terraces, plateaus, passes, cliffs, slopes that can be walked**, and other
  landforms a game is played on.

Three requirements go with them:

1. **What they make can be edited afterwards**, and works with the land, tiles,
   colliders and navigation.
2. **Not every construction is a change to the land.** "Sometimes the intended
   result is a separate platform, shelf or structure with empty space beneath it."
   The land is one height at each place and cannot overhang; a shelf with air under
   it is a thing standing in the world, with a solid of its own. Each tool is to say
   which it makes, as the land's tools now say what they work on (7p).
3. **Seen before it is kept.** A brush's outline, and in time the result itself,
   shown before the tool is committed. "A future usability requirement, not a
   request to build a sophisticated preview system now."

**B. Generators and builders**, over chosen tiles, a region or the whole map, for
both broad filling and close work:

- **Mountain ranges**: peak count, height, spacing, shape, density, variation.
- **Hills and scattered landforms**: an area chosen, sparse or dense.
- **Water bodies**: ponds, lakes, rivers and joined systems; shores that can be
  edited; the land eased to meet them.
- **Nature**: trees, forests, flowers, fields, plant communities.
- **Rich water**: reeds, cattails, lily pads, wildlife, "where those designs and
  behaviors exist".

"A generator should establish a useful, editable starting arrangement, not produce
an opaque, uneditable result": its numbers tuned, one thing of it picked and
refined, a useful variant saved, the result used again elsewhere. A water generator
makes the water and its shore, puts out the plants chosen, and may add wildlife:
"composable choices rather than mandatory bundles".

**C. The workflow.** "Design or choose reusable elements → select an area or tiles →
generate or construct → customize the result → save and reuse it → build gameplay
on top." **Construction, visual style, and simulation or interaction stay apart and
compose**: one mountain generator in several styles; a game using what was built for
traversal, collision and level design. All of it on the layout and tiles, the land,
the library of designs and the makers that exist, "instead of becoming isolated
systems".

**Added to this direction the same day (the user's fourth brief, 2026-10-12): still
nothing of it built.**

- The terrain-shaping tools named again: winding mountain ridges, ledges on a wall,
  terraces, plateaus, passes that can be walked.
- The generators named again, over chosen tiles, regions or larger areas: mountain
  ranges, hills, ponds, lakes, rivers, vegetation.
- **Vegetation and a change to the land.** When the land is shaped where things grow
  or were painted: clearing them about the change is a choice offered, not done
  unasked; a brush that takes away only what is chosen (trees and not flowers; this
  design and not that); and what is left is stood again sensibly on the land as it now
  is, at its new height and facing as it should. Today a maker's things are made
  again at the land's new height when the land changes, and what the *world* grows is
  cleared by Clear and Let grow; nothing clears what a maker made by a brush, and
  nothing asks.
- **Vegetation that knows its surface.** Upright, or leant to the slope, or fixed to a
  wall, as suits the thing, with the person able to say otherwise. Today everything
  stands upright at the land's height. This is the "which way does the surface face
  here" that goes with the one question of height (1 below), and the same answer a
  brush of things needs (7t).

**How this sits with what is written already.** It widens three things and replaces
none: principle Q and 7i (design apart from distribution: Design → Save → Group →
Generate or Paint → Customize → Reuse) gain "construct" beside "generate", and
landforms and water beside plants and houses; 7f (shaping by hand) was about meshes,
and this is its counterpart for the land; 7o's layouts, tiles and regions are what
"select an area or tiles" means. A generator here is a **maker** in the sense of 7h
(a thing of the world with parameters, whose things can be worked on one by one:
W1b, D2), and is not a new kind of system.

**What stands today, against it** (so that nothing here is taken for built):

| Wanted | Today |
|---|---|
| Tools for the land that say what is meant | Six brushes and two stamps (cliff, ledge). What a stroke or a stamp did is kept as the heights it left (`Layer`): a cliff put down cannot be opened again and given another rise, only undone or worked over |
| A structure with air under it | None made by a tool. A tile's ground lies on the land and is no solid; a library design can be a platform, placed by hand |
| Seen before it is kept | A tool's outline on the land and on tiles; a stroke is staged and shown while the button is held, and Esc lets it go. No picture of a stamp's result before the click |
| Generators over tiles or a region | Woods and meadows over a reach, a level, paint, or tiles (7j, 7p). No region that is a thing with a name. No generator of landforms or of water |
| Water bodies | One level for the world (0 m, in six places of the host and two of the editor); a pond is a sheet a design carries (7l). No lake, river or shore that is made |
| Results edited one by one, saved, reused | For makers' things: yes (D2, the library). For anything done to the land: no |
| Construction, style and simulation apart | Style changes colours and what a wood is made of (S1); colliders and traits are content. The land's colour by height is in the land's maker, not a style's |

**What it depends on, and the order that gives.** Each a milestone to be named and
asked about when it comes; none is begun, and this pass built none.

1. **One question of height** (risk 1 of "7o, continued"): the land, where tiles
   stand (`tileops::standing`) and any structure with a walkable top, asked in one
   place by placement, makers, the walker's solids and navigation. **Tile elevation's
   first tier** needs it and is still the next substantial pass; everything below
   needs it too, which is the reason it stays first.
2. **A region as a thing** (owed since 7p): a named set of cells or an outline that
   a maker, a generator or a tool is pointed at. "Select an area or tiles" for every
   generator; today only a wood's `on` knows cells.
3. **A landform as a thing, not only the heights it left.** A terrace, plateau,
   ramp, pass or ridge kept as what it is (where, how wide, how high, how it eases
   off), applied to the land in order and made again when a number of it changes;
   the cliff and ledge stamps are its first two. This is what makes a specialised
   tool editable afterwards, and what a mountain generator makes many of. It must
   live with strokes done by hand over it (the six mechanisms of 3.3). **The
   decision to take before it is built:** kept in the land's layer as an ordered
   list beside the hand's heights, or as makers that write into the land.
4. **Tools on that record**: terrace, plateau, walkable slope, pass; then **ridge
   from a mountain's side**; then **ridge around a mountain**, which is a landform
   along a drawn line (W1's lines, 7d) with a height that changes along it.
5. **Structures apart from the land**: the shelf on a wall, a platform, a bridge.
   A design with a solid, placed against a face by a tool that finds the face. It
   shares the solid and the height question with raised tiles (1), and is where
   L3's modular pieces meet the land.
6. **Generators of landforms** over a region: hills, then mountain ranges. A maker
   whose things are landforms (3), each picked and changed by itself.
7. **Water with a level of its own** (risk 3), then **water bodies** as makers: a
   pond or lake over a region with its shore eased into the land, a river along a
   line. Then shore plants as makers told a shore, the way a wood is told tiles
   (`on`, 7p); wildlife last, as makers of creatures where the kinds and their
   behaviour exist (7g, `wild.gd`), each a choice and none bundled.
8. **Plant communities** are 7i's E1c, D3 and D4, unchanged; "woods on tiles" (7p)
   is their first tier aimed at tiles.
9. **Saved and used again**: a generator's settings, and an arrangement it made,
   kept in the library beside designs and tile sets (D3, E3).
10. **Previews**: the outline now; a stamp or a landform staged and shown before
    the click, by the staging a stroke already uses, when 3 is built; nothing more
    until use asks.

**Rules for whatever is built of this.**

- A tool says whether it changes the land or puts down a thing, before it is used.
- A generator's result is things of the world a person can pick, change, take away
  and keep: never one sealed lump.
- Shape, look and behaviour are three parameters, not one: a generator makes shape
  and says what each part is (peak, shore, bed); a style colours and dresses it; a
  game reads its solids and heights.
- Every piece is the first tier of one of the ten above, or it waits (the user's
  rule on probes).

**Not to be done to get there:** an island or mountain generator that writes
heights nobody can open again; a second terrain beside the land; a water system
before water has a level of its own; a preview system built ahead of the tools it
would show.

## 7r. Tiles flickering as the eye moves: what it was (the user's fourth look, 2026-10-12)

> **Since (7v, the same day):** the colour notch near a corner and the hard rim, both recorded below as not mended, are mended; and how tiles came to share a cell is found, reproduced and stopped. What follows is as it was written.

**What the user said.** After commit `43c3fd5`: "tile edges still flicker when the camera
moves", on "some flat tile boundaries as well as shaped terrain", perhaps worse after
sculpting. Not to be assumed the same cause as before, and not to be hidden with
offsets.

**How it was made to happen.** A part of the tour, `-Tour -Only flick`, kept. On a copy
of the user's own tile world (`editor\my tile world.json`, with their leave; the file
itself is not opened or written), with everything but the land and the tiles' grounds
left out and the wind still, the eye is turned 0.0003 rad and drawn back 0.04 % each
frame for 24 frames, and every dot that is one colour, another for one frame, and the
first again is counted, in the middle of the picture. From 12 m and from 45 m, about a
side two tiles share, a corner four share, a side on shaped land, and a side of hexes.

**It was this, and it was not what the two passes before mended.** Nothing arrives
late here: the world is still and everything is drawn as it is.

| What was seen | Its cause | The evidence | What was done |
|---|---|---|---|
| Short lines of dots along the sides of tiles changing colour from frame to frame as the eye moved; most where three or four grounds meet | Since 7p each tile's ground **went 2 cm past its side onto its neighbour's** (`TURF_LAP`), to leave no crack. Two sheets in the same triangles of the land at the same height: which of the two is drawn at a dot is decided by the last digit of their depths, and that changes with every move of the eye. It was said nothing would be seen of it because the two are one colour at their side. **They are not**, near a corner (below), so the strip showed | The same eye, the same frames: 386, 523, 285 dots (side, corner, shaped land, from 12 m) with the lap; **16, 16, 6 with it taken away**, all of those on the line of the sea, none on a tile. With 4x smoothing off: 790, 1169, 688 before; 0, 1, 0 after | **No ground goes past its side.** Each is cut off at it through the same triangles of the land, so the two have the same corners there and are one sheet. And a corner is told the same height whichever tile asks (a whole millimetre of the world, not of the tile: two tiles at different heights were told heights up to a millimetre apart for one corner). `land.rs` `turf`, `turf_land` |
| Would there then be cracks, with the land seen through? | The reason the lap was put there | The land painted loud and looked for between grounds, 30 frames from above and from low, on level and on shaped land: **no dot of it**, with the lap and without | Nothing more. The tour's part keeps this check |
| A thin dark line round tiles with no neighbour, seen from far off, coming and going as the eye moved | A tile's ground is drawn a little nearer the eye the farther off it is (7p). Its **rim** (turned 30 cm down into the land where no tile is beside it) was lit as a wall facing outward; from far off, drawn nearer, it came out in front of the land | Seen in the pictures from 45 m, gone in the same pictures after | The rim is lit as the ground it hangs from, and wound so its front is what is seen from outside |

**What is still to be seen, and is not a flicker.**

1. **Near a corner where three or four different grounds meet, two tiles disagree about
   the colour along their side** for up to the width of the blend (2.4 m of an 8 m
   tile): a notch of the wrong colour, standing still. This is the "short edge at a
   corner" of 7o, and it is longer and plainer than was written there; it is what the
   lap made flicker. The cause: each tile blends toward each neighbour across a side
   separately, and a tile does not know the ground of the tile across a corner. **The
   mend, designed and not built:** one colour for a place, worked out the same by
   every tile that draws it: each cell about the place (the tile's own, those across
   its sides, and for squares those across its corners) given a weight by how far
   inside or outside that cell the place is, the same function whoever asks; the
   colour is the weighted mix; toward a cell with no tile it turns into the land's
   grass, wholly at the side. It needs `beside` to say the grounds across corners too
   (`tileops::settle_tiles`). It changes how every blend looks, so it is to be looked
   at, not slipped in.
2. **A tile at the rim of what was laid does not always turn wholly into the land at
   its side**: where the uneven line leans outward it stops short, and a hard edge is
   left (snow against grass in the user's world). The same mend covers it.

**Found in the user's world while looking: tiles standing on one another.** Of its 67
tiles, five cells of "the hex country" hold two or three tiles each, and two more
tiles stand outside the layout: eight tiles "told nothing" of their neighbours
(`beside` empty), which the host does by design for "a tile in no cell" and for the
second in a cell (`tileops::board`: "the first in the document's order, where two
stand in one cell"). They are drawn in the same place as the tile under them, with a
hard edge all round. The showcase the world began as has none: its hex layout was 5 by
5 cells of 8 m, and is now 4 by 7 of 7 m. So they came of the layout being changed
with tiles in it (`tileops::carry`), in some order of changes not yet reproduced.
**Not mended, and nothing of the user's world was changed.** Next step: reproduce it
in a host test (cells taken away, then the size changed, then cells added, in each
order), then decide with the user what a change of layout does with a tile whose cell
goes (today: left where it stands, which is how it comes to share a cell later), and
have the Tiles window say which tiles share a cell or stand in none.

**Not looked into.** One case of the tour's part with the eye inside a hill (a
picture of the underside of the land) shows a line of dots at a silhouette with
smoothing off, 2 dots with it on: not a view a person works in.

## 7s. Painting what is seen: the land, a tile's surface, other faces (the user, 2026-10-12; traced, and a design: not built)

> **Since (7v, the same day):** the first tier of this is built: "paints on" in the paint tool, starting on what is seen, and a whole tile painted a ground of its own. Paint by place within a tile, and other faces, are not.

**What the user said.** Woods and meadows can be put on raised tiles, but "painting
surface materials such as grass or sand on raised land does not appear to change the
visible tile surface." Suspected: the paint goes to the land under the tile. "When the
user chooses to paint a visible surface, the operation should affect the surface they
are targeting. They should not have to discover hidden layer distinctions."

**Traced: the suspicion is right, and it is by the design of 7o and 7p, not a fault
in it.**

| What | Where it is kept | What draws it | What paints it today |
|---|---|---|---|
| The land's surface (grass, rock, sand, dirt) | In the land's layer, a sample a metre: `Spot.surface` and how much of it (`land.rs`) | The land's own mesh | Land > paint (`Stroke`, `tool: "paint"`) |
| A tile's ground | One word in the tile's **definition** (`tile.NAME`, `ground`), the same for every tile of that name put down | `turf`: its colour is worked out from that word, the neighbours' words, its paths and the place. **It never reads the land's surface** | Nothing. It is changed only by putting another tile in the cell (Tiles, Ctrl+T) |
| A cliff's face, a steep slope | The land again: the same samples, coloured by how steep | The land's mesh | Land > paint, as any land |
| A wall, a roof | A design's own colours | The design's mesh | Designs and Library (a design's colour, a copy of it) |

So a stroke of paint over a tile changes the land, which the tile's ground covers. The
editor says so (the brush is drawn in stripes there, and the panel says "works on the
land under the tile"), which is the rule of 7p (never onto a tile silently), but it
leaves the person with no way to do what they meant.

**The smallest sound change: a paint tool says what it paints on, and a tile can be
painted.** Three parts, in order; none built.

1. **A target the person sees and chooses.** The paint tool gains "Paints: what I see
   / the land under tiles". With "what I see", a stroke over a tile paints the tile's
   surface and a stroke over bare land paints the land: the brush is drawn plain on
   both, and the panel names which it is over. "The land under tiles" is today's
   behaviour, with its stripes. This is a choice shown, not a redirection. **Which is
   the default is the user's to say**; "what I see" is what their words ask for.
   Raise, lower, smooth and flatten are not changed: a tile has no height of its own
   until tile elevation is built.
2. **A tile's surface, kept with the tile and apart from the land** (the three things
   of "7o, continued": underlying terrain, tile elevation, tile surface). First tier:
   **one ground for the whole tile**, as a setting of the tile put down (`ground`,
   empty: as its definition says). `turf` and `settle_tiles` read the tile's own
   ground where it has one. Painting "sand" over a tile makes that tile sand, with its
   blends to its neighbours made again; undone as one step. Coarse, and true to what a
   tile is today. Second tier: **paint by place on a tile**, a small grid of surfaces
   kept in the tile's own settings (it moves, turns, is duplicated and removed with
   its tile), read by `turf` as the land's samples are. Never in the land's layer.
3. **Other faces later, by the same control.** "Paints on" grows a word when there is
   something to paint: steep faces only (a mask by slope over the land's own paint: a
   cliff painted rock without its top), and a structure's wall or roof when shelves
   and modular pieces exist (7q 5, L3), which is a design's material and not the
   land's.

**Rules.** The tool always says what it is painting on before the button goes down.
One stroke paints one kind of surface. Nothing a tile is painted with is written into
the land.

## 7t. One way to paint and to create: what to paint with, where, and paint (the user, 2026-10-12; a direction: nothing of it is built)

**The user's principle.** "Choose what you want to paint with, choose where to paint
it, and paint."

Asked for with it (from the brief; the four notes it names, among them
`Draw3D_Unified_Creation_Painting_and_Surface_Workflow.md`, were not on this machine
or among the artifacts that could be listed, and are to be read against this section
when they can be):

- **What is painted with.** A *material* (grass, sand, rock: a surface's look). A
  *single design* (a rock, a tree, a flower, a shrub). A *group* (a meadow, a forest,
  some trees picked together, any reusable arrangement).
- **Variation.** The very same design or group each time, or varied: turn, size,
  spacing, colour, saved variants, whatever the design lets be set.
- **Where.** A freehand stroke; chosen tiles; an outline drawn; a region along a path;
  in time a part of the map.
- **What is there already.** Chosen plainly each time: add to it, replace it, or clear
  it.
- **How it stands on the surface.** Sensible by default (upright, or leant to the
  slope, as suits the thing) and the person can say otherwise.
- "A meadow must not be the only way to place a rock merely because rocks are part of
  a meadow." Several tiles picked and trees painted on them, or an area drawn and a
  forest put in it, "without switching to an unrelated subsystem."
- The chain: "Select or create a design → place or paint it → make variants or groups
  → generate larger arrangements → customize individual instances → reuse the result."
  The library is the hub of it, "not a separate dead end": from a tree in the world to
  editing that tree, saving a variant, making a group to paint with, or opening the
  tree's generator, and back to the forest to refine it.

**What stands today, against it** (so that nothing here is taken for built).

| Wanted | Today | What is missing |
|---|---|---|
| Paint a material | Land > paint, on the land only | A tile's surface (7s) |
| Paint with one design | A design is put down by a click, one at a time (Designs, Library). Ctrl+D puts another down by a click | A stroke that puts many down; spacing; variation |
| Paint with a group | A wood or a meadow is a maker: dragged out as a reach, its area painted with a brush (7j), told tiles (7p). What it is made of is a setting (`of`) | A group as a thing chosen to paint with; a group made from things picked; the same act as painting one design |
| Vary or repeat | A maker varies by its seed. One thing of it can be given a design, a size, a shape (7j). A design copied and tuned is a variant (7n) | Variation as settings of the brush: turn, size, spacing, colour, which variants |
| Where: stroke, tiles, outline, path, region | Stroke: a maker's painted area. Tiles: a maker's `on`. Outline: a reach (a circle). Path: lines for roads and streets only | An outline drawn; a region as a thing with a name (7q 2); the same choice for every brush |
| Add, replace, clear | Clear and Let grow say what the *world* grows where. A thing is taken away one at a time, or with its maker | Replace and clear for what was painted or put down, by a brush, chosen plainly |
| Stands as the surface asks | Things stand upright at the land's height (`Ground::rest`); what floats, floats | Leant to the slope; fixed to a wall; said by the design, overridden by the person |
| The library as a hub | Library and Designs open a design to change it; "Make it mine"; a thing says its sort and its maker | From a thing in the world straight to its design, its variants, a brush of it, its generator, and back |

**How it is to be built: on what is there, not beside it.**

- **A brush is three choices and no new system.** *With what* is a design, a group or
  a material from the one list of designs (D1) and the library (7n). *Where* is a
  stroke or a region (7q 2). *How* is add, replace or clear, and the variation. What a
  stroke of things leaves is **a maker**, as a wood is today (7h): its things are
  things of the world that are picked, changed and taken away one by one (W1b, D2),
  never a sealed lump. A stroke of a material leaves paint (7s). This is 7i's "Generate
  or Paint" (E1c, D3, D4) said as one act; it replaces nothing there.
- **A group is a design that is an arrangement** (7i D3): a meadow, or things picked
  and saved together. A meadow is then one group among others, and a rock of it is a
  design that can be painted alone.
- **A generator is where a group or a brush comes from when it is too large or too
  particular to place by hand** (7q B): it makes an editable arrangement, which is the
  same kind of thing a stroke leaves.
- **Surface-aware standing** is one question asked in one place (7q 1: one height; and
  with it, which way the surface faces there), with the design saying what suits it
  (upright, leant, on a wall) and the brush able to say otherwise.
- **Not to be done to get there:** a universal brush framework built before one
  workflow is proved from end to end; a second way to scatter things beside makers; a
  material system before a tile can be painted at all.

**The order proposed** (each the first tier of something above, or it waits).

1. **Paint a tile's surface** (7s 1 and 2, first tier). Small; ends the confusion that
   began this.
2. **Paint with one design**: a stroke on the land or on tiles that puts down many of
   one design, with spacing, turn and size varied or not, and add / clear. What it
   leaves is a maker. This proves the whole act from end to end with what exists.
3. **Paint with a group**, and **a group made from things picked** (D3).
4. **Where, said once for every brush**: chosen tiles (there already for woods), then
   an outline drawn, then a region with a name (7q 2).
5. **Replace**, and clearing by a brush of what was painted (the vegetation clearing
   of 7q, below).
6. **The hub**: from a thing to its design, variants, brush and generator, and back
   (7u).
7. Standing by the surface (with 7q 1), then generators (7q 6 to 8).

Tile elevation's first tier (7q 1) is still owed and is not displaced by this: 1 above
is small enough to go before it or with it, and 2 onward want the user's word on
which comes first.

## 7u. The editor's workflow beside Godot, Unity and Unreal: what to take, what to leave (looked up 2026-10-12)

**What was looked up, and what was not.** Read on the makers' own pages on
2026-10-12: Unreal's Foliage Mode, Landscape Paint Mode and Landscape Brushes; Unity's
Paint Texture, Paint Trees, the Terrain shortcut keys and the Terrain Tools package's
brush keys; Godot's GridMap editor. Marked *(from memory)* below: not read again in
this pass, and to be checked before anything is built on it.

| Subject | What the engines do | For Draw3D |
|---|---|---|
| Painting a surface | **Unreal**: a landscape's material declares *target layers*; one is chosen in a list and painted; weight-blended layers take from each other, others lie over (snow); Shift + stroke does the opposite. **Unity**: *terrain layers* in a list; the first added fills the terrain; brush size and opacity. **Godot**: no terrain of its own *(from memory)*; plugins follow the same list-and-brush pattern | **Take:** a short list of what can be painted with, one chosen, always shown. Shift + stroke as the opposite (take paint off). **Leave:** layers of weights the person must manage, layer assets, blend methods. Draw3D keeps one surface for a place |
| Painting things | **Unreal Foliage Mode**: a palette of foliage types; tools Select, Lasso, Paint, Reapply, Single, Fill, Erase, Remove, Move; brush size, density, erase density; filters for *what kind of surface is painted on* (landscape, static mesh, foliage…); Shift erases; per-type placement settings (align to normal and others). **Unity Paint Trees**: a list of trees, brush size, density, height and width random or fixed, random turn; Shift erases all, Ctrl only the tree chosen; Mass Place with "keep existing trees" | **Take:** paint, erase and single as modes of one tool; density and random-or-fixed size and turn on the brush; **erase all or only what is chosen**; **the filter for which surfaces are painted on, shown** (this is the "target surface" of 7s); "keep what is there" asked plainly when many are placed; Reapply (settings changed, applied again to what was painted) as a later nicety. **Leave:** a palette apart from the library: Draw3D's is the list of designs |
| Painted things afterwards | In both, painted foliage is instances of a special kind: selected with the foliage tool's own Select, not as ordinary things *(Unreal: read; Unity: from memory)* | **Leave this.** Draw3D's rule is better for its users: what is painted is things of the world, picked like any other (7h, W1b) |
| Tiles and cells | **Godot GridMap**: a palette at the bottom with a filter; tools Selection, Erase, Paint, Pick (click a placed one to choose it in the palette), Fill a selected area, Move, Duplicate; turn the cursor's piece | **Take:** *Pick* (an eyedropper: click a thing or a tile to paint with it) for every brush; Fill a selection. Draw3D has paint, drag, replace, move and turn already (L2) |
| Brush size | **Unity**: `[` and `]` size, `-` and `=` opacity; the Terrain Tools package: hold S and move the mouse for size, A for strength, D for turn, in the view. **Unreal**: size and falloff are settings in the panel; no key on the pages read | **Take:** `[` `]` for every brush (the land's and a maker's area have them already; one meaning everywhere), and later **hold a key and drag in the view to size the brush**, the ring growing where it lies, which keeps the place worked on in sight. **Leave:** opacity and falloff as separate numbers until something needs them |
| Asset browser | A browser of assets with thumbnails, search and filters; double click opens the asset's own editor; drag into the view places it; right click offers what can be done with it; variants as assets that follow a parent (Unity's prefab variants, Unreal's child blueprints and material instances) *(from memory)* | **Take:** actions on a design where it is shown (Place, Paint with, Edit, Make a variant, Open its generator, Show where it is used). Variants that say what they are a variant of. **Leave:** folders, import settings, a file tree: Draw3D's library is shelves and groups |
| Acting on what is picked | Right click menus in the view and the outliner; a details panel or inspector for whatever is picked | **Take:** the same few actions offered from the thing picked as from the library (below). Draw3D's row under the bar and Makers window are the inspector; no second one |
| Choosing a region | Unreal: volumes placed in the world for procedural foliage and PCG; landscape *selection* masks *(from memory)*. Unity: none of its own for terrain. GridMap: a box of cells | **Take:** a region as a thing of the world with a name, that a brush, a maker or a generator is pointed at (7q 2). **Leave:** node graphs for generation |
| Panels | All three: panels docked, tabbed, saved as layouts; a mode or tool changes what the side panel shows | **Leave** docking for now. Draw3D has windows that move, fold and are remembered (7e); what is owed is that the tool's panel always says the four things below |

**The workflow to connect** (the user's chain; today each arrow that is missing is a
walk through a different window):

| From | To | Today | Owed |
|---|---|---|---|
| A thing in the world | Its design, to edit | The row under the bar says its sort and maker; Designs lists it | "Edit this design" on the thing |
| A design | A variant of it | Library: a copy under a name, tuned (7n) | "Make a variant" on the thing and on the design; the variant says its parent |
| A design or things picked | A group to paint with | A wood's `of` (a setting) | "Make a group of these"; "Paint with this" |
| A design | Its generator | None is opened from a design | "Open its generator" where a design came of one (a tree: 7i E1) |
| A forest made | One tree of it, refined | Click it: its own design, size, shape; taken away alone (7j, W1b) | Stands |
| Anything made | Used again elsewhere | Designs and tile sets saved to the library; worlds | Arrangements and a generator's settings saved (D3, E3) |

**Requirements for the interface, recorded and not built** (the user, 2026-10-12).

- **The wheel over a panel works the panel** (scrolls it) and does not move the eye.
  Today the eye is moved by the wheel wherever it is not taken by a control
  (`main.gd`, `_unhandled_input`); which panels let it through is not yet traced.
- **What is picked, and its outline, are let go or made right when the tool changes.**
- **A number changed does not make its panel jump**, or lose what was being done.
- **A brush is sized in the view**, with the place it covers in sight (above).
- **Four things are always plain: the tool, what it paints with, what surface it
  paints on, and whether it adds, replaces or clears.**

No renaming or regrouping of the interface is to be done from this section before the
user has looked at the inventory (the rule of 2026-10-11); the interface is to be
done as one coordinated pass after the workflow of 7t has been proved by its first
two steps.

Sources read: [Unreal Foliage Mode](https://dev.epicgames.com/documentation/en-us/unreal-engine/foliage-mode-in-unreal-engine),
[Unreal Landscape Paint Mode](https://dev.epicgames.com/documentation/en-us/unreal-engine/landscape-paint-mode-in-unreal-engine),
[Unreal Landscape Brushes](https://dev.epicgames.com/documentation/en-us/unreal-engine/landscape-brushes-in-unreal-engine),
[Unity Paint Texture](https://docs.unity3d.com/Manual/terrain-PaintTexture.html),
[Unity Paint Trees](https://docs.unity3d.com/Manual/terrain-Trees.html),
[Unity terrain shortcut keys](https://docs.unity3d.com/Manual/terrain-UsingTerrains.html),
[Unity Terrain Tools brush keys](https://docs.unity3d.com/Packages/com.unity.terrain-tools@3.0/manual/brush-controls-shortcut-keys),
[Godot GridMap](https://docs.godotengine.org/en/stable/tutorials/3d/using_gridmaps.html).

## 7v. Paint on what is seen; a tile's own surface; corners and rims mended; tiles in no cell (built 2026-10-12, after the user's five decisions)

The user answered the three questions of 7r to 7t and set this pass:

1. **Paint goes on what is seen, by default**: a tile's surface where a tile lies, the
   land elsewhere. "Paint the underlying land" stays as a choice. The target is said in
   the interface, and no stroke is sent to another surface than the one shown.
2. **A tile's surface, first tier**: a whole tile painted a ground (grass, sand, rock…),
   kept apart from the land under it and from other tiles. No tile elevation, no paint
   by place within a tile.
3. **The corner notches and hard rims of 7r mended with it**, if that stays coherent
   and can be checked; the flicker of 7r looked at again afterwards.
4. **A tile that loses its cell is kept**, said to have none, and can be given a cell
   or removed; no accidental stacking; no world rewritten without a safe, tested way.
5. **Shown in the editor** on a copy of the user's tile world.

Tile elevation, painting with a design or a group, any redesign of the bar, and the
larger workflow of 7t stay for later passes.

### What is built

**What paint goes on (decision 1).** The paint tool has a row "paints on" with two
choices: *what I see: a tile, or the land* (how it starts) and *the land, under any
tile* (the tool as it was). The blue line of the panel says which is in force and
what it means; the line under the pointer says, on a tile, which tile a stroke would
paint and what it would become; and that tile is tinted whole in the view, with no
brush drawn on tiles (a tile is painted whole, whatever the width of the brush). In
the other mode everything is as before: stripes on the tile, "paint goes on the land
under it". After a stroke the panel says which tiles it painted. Every other tool of
the panel works on the land, as before, and says so. To the host it is one word on
the same command: `sculpt` with `on: "seen"` or `on: "land"`; with nothing said it
is the land, so every script and world made before is as it was.

With `seen`, one stroke does two things in one edit (one Ctrl+Z): each tile the
*middle* of the brush was put down on is painted; and the land is painted where the
brush lies **and no tile does**. Under every tile the land is left as it is: one
stroke never paints both what is seen and what is hidden under it.

**A tile's own surface (decision 2).** A tile put down has a parameter `surface`:
nothing (its ground is what its text says), or one of the grounds. It is kept with
that tile's node and nowhere else: not in the land's layer (7o's rule holds), not in
the tile's text (which every tile of that name shares). Painted the ground its text
already says, or "as generated", it is nothing again. What follows the ground follows
the painted ground: the tile's colour, what its neighbours blend toward, a wood told
to grow "on tiles of sand", and "Save as a tile of my own". The host also takes
`tile_paint` (a node and a ground), which reaches the two grounds the land has no
word for (forest, snow); no control asks it yet.

*Backward compatibility.* A world written before today has tiles with no `surface`.
They read as unpainted; the parameter is added to a tile the first time that tile is
painted and at no other time, so opening and saving an old world changes nothing of
it. (The first version of this refused to paint such a tile, "no parameter
'surface'": found by painting in a copy of a world saved earlier, not by the tests,
which make their tiles new.) A tile's `beside` now has, for a square, four more
words: what is across each corner. A tile told only its four sides (an old world
before its first edit) is drawn from those four, as before.

**Corners (decision 3; 7r's notch).** The colour of a tile's ground at a place was
worked out from the tile's own side neighbours, so near a corner two tiles disagreed
along their shared side. It is now a mix over *every* cell about the place, each
weighted by how far outside that cell the place is, with an uneven line that depends
on the place and the ground only. Whichever tile is asked, the answer is the same:
half and half on a side, a third or a quarter each at a corner. A hex already knew
all the cells about it (every corner has a tile beside it); a square is now told its
four diagonal cells. Checked by a test that builds five tiles of five grounds about
corners, in squares and in hexes, and compares the colours two, three and four tiles
give for every corner they share (within 0.002).

**Rims (decision 3; 7r's hard edge).** Toward a cell with no tile, a ground now turns
wholly into the land at the side itself (the uneven line no longer stops short of
it), and into the land *as the land is coloured there*, not into plain grass: the
tile is told how high the land is, how steep, and what was painted on it, and asks
the same function the land does (`land_colour`). Two things it does not know: a
road's dirt, and paint at less than half strength in a soft rim.

**The flicker of 7r, looked at again.** `-Tour -Only flick` after all of the above:
on the showcase 0 dots at every case but one (1 dot, far off); on a copy of the
user's world 0 from 12 m (2 on the hex side) and 1 to 15 from 45 m, on the
shoreline as before; no land seen between grounds. As it was after 7r.

**Tiles in no cell (decision 4).** *How a tile is itself and where it is:* a tile put
down is a node of the document; it is itself by that node, and where it is is where
the node stands. Nothing else records a cell: the cell was always worked out from
the place. The fault was in how: any tile whose middle fell in a cell counted as in
it. A layout made smaller left the tiles of the cells that went where they stood
(rightly, and said so), but they then *counted* as in whatever cell they lay across;
and a second change of the layout carried them to the middle of that cell, on top
of the tile that had it. That is how the user's "hex country" came to hold two and
three tiles in a cell, and the test of this pass reproduces it step for step (with
the old rule the second change carries 12 tiles where 9 are in cells).

The rule now: **a tile is in a cell when it stands in the middle of it, as big as it
and of its form** (to 5 cm and 1 mm), which is what putting down, moving and
carrying make. Otherwise it is in no cell: a piece of land by itself, with
everything it had (its things, what was done to them by hand, its paint). Such a
tile
- is **said**: the host says of each tile why it has no cell (`loose`: outside the
  layout, another size, another form, its layout gone, not in the middle, or in the
  same cell as a tile that has it); the Tiles window lists them under "TILES IN NO
  CELL", above the tile picked, and says it of the tile picked; a change of layout
  that leaves any behind says where to find them;
- is **not carried** by a layout it is not in, so it is never stood on another;
- is **not built on**: putting a tile down in a cell one lies across, or moving one
  there, is refused with its name ("Fill the empty cells" leaves that cell);
  `stack: true` does it deliberately, which nothing in the interface asks yet;
- is **given a cell** by "Give the one picked the nearest empty cell", or by
  dragging it onto an empty cell: it is stood in the middle, made as big as the
  cell, turned to the layout's nearest turn, and keeps everything. It changes places
  with no tile (that one would then have no cell);
- is **removed** only by the person ("Remove the tile", Ctrl+Z brings it back).

Nothing of an existing world is rewritten. The user's world has 8 such tiles of 67
(6 standing exactly in a cell another tile has, 2 outside the layout): on a copy,
the window lists them with the reason for each, one was given the nearest empty
cell with its things and its ground, and that was undone. They are left for the
user to deal with; until then two grounds in one place still flicker there.

### How it was checked

- **Host tests**: all pass (37 and 49). New: the colours about a corner (above);
  `p49`, the whole of decisions 1, 2 and 4 through the host's commands, written,
  opened again and undone.
- **By hand of the tour, with pictures looked at** (`-Tour -Only surface`, new and
  kept): from the panel with the pointer, on a copy of the showcase and on a copy
  of the user's tile world: a tile among tiles painted sand, rock, grass (that tile
  is that ground; no other tile changed; the land at seven places under it is
  painted nothing); a stroke off the tiles (the land is painted); at the rim with a
  brush that lies over a tile (the land outside, nothing under the tile, no tile
  changed); told to paint the land under tiles (the land there, the tile as it
  was); a tile over land raised 4.5 m at a shared corner; every step undone. The
  user's file has the same checksum before and after.
- **Regressions run**: `-Tour -Only flick` (both worlds), `seams`, `tiles`, `ui`: no
  problems. The whole tour and the other demos were not run.
- **A demo**: `editor\run.ps1 -Demo paintseen` (3 beats).
- **Not tried by a person.**

### What is not right yet, or not built

- A tile is painted whole. Paint by place within a tile is the second tier.
- The panel's list of surfaces is the land's four (rock, grass, sand, dirt). Forest
  and snow can be given a tile only by the host's `tile_paint`.
- "What grows goes by the surface painted" says nothing of a tile: a woodland
  painted sand keeps its trees. Whether paint should clear a tile's things is a
  question for the user.
- Restore, clear and let grow know only the land. A tile's paint is taken off by
  painting "as generated".
- The tint of the tile that would be painted is the brush's yellow, which is near
  the colour of sand; and it follows the pointer a few times a second, not every
  frame.
- A tile with no tile on any side fades into the land all round, and what is left
  of its own colour is a small patch in its middle.
- A tile in no cell is drawn where it stands with a hard edge. That marks it, but
  nothing in the view says "no cell" in words: the Tiles window does.
- Stacking on purpose (`stack`) has no control, and two tiles in one place still
  flicker against each other: if stacking is to be a feature, one must be drawn
  over the other by a rule.
- Tile elevation (7o), paint with a design or a group (7t), and the interface's
  requirements of 7u: not begun.

## 7w. Tile elevation, first tier: one question of height (built 2026-10-13)

The first of 7q's ten, and the second of the three things 7o keeps apart (the
underlying terrain, **a tile's elevation**, a tile's surface). Asked for by the user
as the next milestone after the convenience pass, "following the one question of
height approach".

**What a tile's height is.** A parameter of the tile put down, `raise`: how far its
ground stands above the land under it, in metres (0 to 40, whole centimetres; never
below the land). It is kept with that tile and nowhere else: not in the land's
layer, not in the tile's text. So it goes with the tile when it is moved, changes
places with it, is undone with the edit that set it, and is written and opened with
the world. A tile at 0 is made exactly as it was before this pass.

**What its ground is then.** The land's own surface under its cell, lifted by
`raise`: the same triangles, corners and lean as before (7o, continued). So two
tiles raised alike meet in the same corners at the same heights and are one ground,
on level land or on a hillside, and a group raised together is one raised surface.
It is **not** a level top: a platform that is flat whatever the land under it is a
landform a tile carries (2 of 7o's list), and is not built.

**What a side is.** Where the tile beside a side stands lower, a face straight down
to that tile's ground, in the colour of earth, lit as a wall; where it stands as
high or higher, nothing (the higher one's side is its own); where there is no tile,
down to the land and 30 cm into it, as a rim was. A raised tile is told how high the
tile beside each side stands (`steps`, written by `tileops::settle` after every
edit, as `beside` is). No ramp, bank or shore: a face only.

**One question of height.** `Shape::top`: the land's height, and over it the lift
of a raised tile there. The host writes where the raised tiles are as one text of
the document, `tiles.raised` (each an outline in the land's frame and a height),
after every edit and only when it changed; a world with no raised tile has no such
text. Who asks what:

| Who | Asks | Since this pass |
|---|---|---|
| Every maker, for where its things stand (`Ground::height`, `rest`) | `top` | a house, a tree of a wood, a rock put down by hand, a tile's own things: all stand on a raised tile's ground |
| A layout's cells and guides (`heights_at`) | `top` | a cell's middle and corners are where one stands |
| The land's tools; a tile's own ground (`turf_land`); the land's mesh | the land alone (`Shape::height`) | unchanged: the land never reads a tile |
| The editor's readouts (`ground`) | both | `heights` is the land, `tops` what is stood on, and the tile said there says its `raise` |
| Whoever walks | solids | a raised tile's ground is a solid, as the land is (`mesh`); one on the land is none, as before |
| A click that puts down or drags | what is drawn | meets a tile's ground where it is seen (`main.gd` `stood_on`) |

Nothing depends on itself: a tile reads the land and the text; the land reads
neither. Terrain height authority is the land's, untouched.

**Decisions taken to build it, for the user to confirm or change.** Two were still
open in 7p. (1) *One at a time or as a group:* one at a time, and several in one
edit by a command (`tile_raise` with `nodes`); the editor's "Those beside it as
high" is the only group there is. A group as a thing waits for regions (2 of 7q).
(2) *What a side is:* a face. (3) *Lifted land, not a level top* (above).

**In the editor.** Tiles (Ctrl+T), under the tile picked: "Above the land (m)", "On
the land", "Those beside it as high"; Page Up and Page Down. `-Demo raised`;
checklist part 9.

**Risks of "7o, continued", as they now stand.** 1 (one height assumed): answered
for makers, cells and placing; **not** for the sea (3) and not for what faces a
slope. 2 (not in the land's layer): held. 3 (one water level): untouched; a raised
tile of water is not possible. 4 (corners): a raised square has no gap at a corner,
since each side's face stands over that neighbour's own edge. 5 (every tile planned
again at every stroke): worse by this pass, since **every maker that stands things
on the ground now reads `tiles.raised`** and is planned again when any tile's
height or place changes; what a maker made that did not move is not made again.
6 (a group is nothing yet): stands.

**Not in this tier:** a level top; ramps, banks, steps or stairs between heights; a
tile below the land; water at a tile's level; a group or a region raised as one
thing; a height painted or dragged in the view; tiles' own colour running down a
side; the walker's navigation over a side (it is a wall to it).

**Validated 2026-10-13** (asked for by the user before 7x). Walked in play on a
square tile and a hex: what is solid is what is drawn and what the host says is
stood on, with no hole; the walker stands on a raised tile, crosses to one as high,
comes down off a lower side and is stopped by it; a wood told tiles stands its
trees on a raised tile's ground; a hex tile is raised as a square is; a change of
height is answered in 20 to 35 ms in the tile showcase (67 tiles, 71 makers). No
defect found. One thing seen that was not designed: a raised tile's edge is an edge
the walker's hands catch (`ledge_hang`), as a cliff's is. Risk 5 above is not
answered by this: it was measured in one world of that size only.

## 7x. Paint with one design, first tier (built 2026-10-13)

The second of 7t's order: "a stroke on the land or on tiles that puts down many of
one design, with spacing, turn and size varied or not, and add / clear. What it
leaves is a maker."

**Built on what was there, not beside it.** A maker already kept things given to
it by hand, by name, in its own frame (`added`, 7j: "add to an arrangement"), and
each such thing was already a thing of the world: picked, moved, turned, sized,
given another design, taken away, undone, written. A stroke is many of those at
once. So what a stroke leaves is a maker, "painted", that makes nothing by numbers
and has only what was given to it; there is no second way to scatter things, no
new kind of thing in the world, and no new way of keeping them.

**What a stroke is.** The command `stroke`: a design (as the one list of designs
says it), where the brush was put down, how wide, how far apart, turned every way
or all alike, sized between two numbers, and add or clear. Where things go is not
chance within the stroke: it is the places of one lattice of the world, each a
little off its own, that the brush passed over. So the same ground painted twice
finds the same places, and painting over what was painted adds only where there is
room. Each thing's turn, size and which of its sort come from its place too. They
are then written out, one by one, as what that thing is: nothing about a painted
thing is worked out again later, and each can be changed by hand.

**Where it stands.** On what is stood on there, by the one question of height
(7w): the land, or a raised tile's ground, and it follows either. The brush meets
what is seen (7v). Nothing is put down under water but what floats.

**Whose it is.** One maker for each design: a stroke of rocks is added to "painted
rock" if there is one, wherever it was painted. **Decided to build it, for the user
to confirm or change:** the other ways are a maker for each stroke (many small
makers, each moved and removed alone), or the person saying which. And **clear
takes only what was painted with the design chosen**, never what a wood made or
what was put down one by one: a brush that clears those is 7t's "replace and clear"
(5 of its order) and is not this.

**Against 7t's table.** *Paint with one design:* a stroke that puts many down,
spacing and variation: built. *Vary or repeat:* turn, size and which of its sort
are settings of the brush; colour and saved variants are not. *Where:* a stroke
only. *Add, replace, clear:* add and clear; no replace. *Stands as the surface
asks:* upright, at the height of what is stood on; not leant. *The library as a
hub:* the brush is reached from Designs; not from the Library, and not from a
thing picked.

**Not in this tier:** a group to paint with; chosen tiles, an outline or a region
as where; replace; leaning to a slope or fixing to a wall; keeping clear of roads,
houses and other makers' things; painting from the Library or from a thing picked;
the brush's settings kept with a design; things dropped as the brush is dragged
(they land when it is let go).

## 7y. After the hand test of the brush and tiles; paint with a group; replace (built 2026-10-13)

The user tried 7w and 7x by hand and set three phases: mend what blocked, finish
the brush (replace, a group), begin generators (7z).

**What blocked, and what it was**

| Seen | What it was | What was done |
|---|---|---|
| Paint leaves rims round tiles | grounds turned into each other 2.4 m to either side of a side (three tenths of a cell): a painted tile showed its ground in its middle only | a tenth of a cell (0.8 m; at most 1 m), the line still uneven by about 0.7 m; every tile about a place still works out the one colour |
| Paint leaves gaps beside tiles | paint "on what is seen" skipped every land sample under a tile, the one under its side too; land colour runs sample to sample, and a rim tile turns into the land's colour at its side | samples under a tile's edge with open land beside them are painted; deeper, and under a side two tiles share, never |
| ...and at land edges | the first sample under the water was left with the sea bed | that one is painted, where dry land is beside it |
| A tile refused, the action clipped or not there | the reason in a one-line notice wider than the window; "anyway" only in the host | notices wrap; a button under the bar and a box in Tiles; gone by itself when it would join |
| Replacing unclear | nothing said a click on a cell with a tile replaces it | the row under the bar says what a click would do there |
| Grid and tiles apart | two windows | each has a way to the other; Tiles chooses the grid snapped to |
| What is edited unclear | each tool said its own thing or nothing | "Works on:" first in the row under the bar, for every state that edits; the row never cut off silently |
| Lowering at the least height | an edit that changed nothing, silently | refused with the reason and where the land is lowered; no step of undo |

Not the brush for designs: what a stroke puts down was measured against the sides
of tiles and is even.

**A layer.** What a brush painted is the things of the makers ("painted") that
were painted with that design, or with that group. Add, clear and replace are all
"of this brush's layer". This is what makes clearing predictable: the rock brush
never takes the rocks of a mix, and the mix's brush never takes rocks painted
alone. (7x cleared by what a thing was; it is by what painted it now. A thing
given another design by hand is still of its layer and is cleared with it, but
is not replaced: it was made particular.)

**A group** is designs with shares, sizes and whether each turns, under a name,
kept with the world (`groups.rs`: one line each). It is what the brush is dipped
in, not what was painted: changing it changes the next stroke only. It is the
same shape of words a wood's "what it is made of" has, and should become what a
wood is given too (not built).

**Replace** takes the things of one design's layer under the brush that are still
of that design to another design, in place: position, turn and relative size
kept; identity as a thing of a maker kept (picked, moved, deleted by itself);
what was done to it by hand that belonged to the old design (a shape of its own)
is dropped with the old design. Refused: a design with bones. Left: what stands
under water when the new design does not float.

**Not built of 7t**: where (chosen tiles, an outline, a region) said once for
every brush; replace to a group or within a group's layer; a group made from
things picked; a group in the library; the hub from a thing to a brush of its
design; strokes that land as they are dragged.

## 7z. Asset generators, first tier: a tree grown from numbers (built 2026-10-13)

**What it is.** A generator of designs is not a maker. A maker stands things in
the world; a generator makes what a thing IS, once, and what it makes is a design
like any other. So nothing downstream learns of it: placing, painting, groups,
woods, tiles, the library and shaping by hand all take a grown design because it
is a kind with a mesh.

**The smallest whole of it** (`shapes/grown.rs`, the commands `generators` and
`generate`, the window Grow): numbers, a look at what they make, a name, a
design; and the numbers kept beside the design so that it is grown again and the
world follows.

**What the tree showed is needed** (and no more was built than this): a table of
families; for each, numbers with bounds, a step, a usual value, a sentence, and
words for one that chooses; one function from numbers to a mesh; what of it is
solid. The editor's window is made from that table and knows nothing of trees.

**The way on**

1. **Rocks and boulders**: one lump with roughness, flatness and how many faces;
   moss by a colour. The same window, a second entry in the table.
2. **Bushes, ferns, grass tufts**: the tree's pieces without a trunk.
3. **Variants of one design**: several seeds kept under one name ("oak, any of
   5"), so that a wood or a group of them is not five designs to manage. This
   wants a kind to have sorts, as the code's own things have.
4. **What a mesh cannot keep**: giving to the wind (a grown tree stands stiff);
   a colour that follows the world's style. Either a kind's part learns "grown
   from numbers" beside "mesh", or a mesh learns to keep them.
5. **Underwater** (a requirement to keep in sight, not begun): what is rooted on
   the sea bed, coral, weed that rises, creatures at their depths. A grown design
   may say where it belongs (dry land, the shallows, the bed, afloat); the brush's
   "not under water" becomes "where this design belongs".
6. Not this: fences along a line, houses, roads. Those are makers.

## 7aa. Tree generator 2, and the living environment (built 2026-10-13, after the user's decisions)

**The user's decisions (2026-10-13), which this section follows.** The narrower
tile blend stays until a problem is shown; tiles and terrain are left alone. A
brush clears and replaces only its own painted layer. Groups are kept with the
world. Generated trees are to give to the wind and take the style's colours.
The tree generator is improved first; then rocks and boulders; then bushes and
ferns. Out of this milestone: tile placement and blending, multi-tile selection,
terrain manipulation, advanced water, weather simulation, seasons, vegetation
physics, the modeling toolkit. Underwater ecosystems stay a later requirement.

**Three things kept apart.**

| | What it is | Where it lives |
|---|---|---|
| Form | how it grows: spreading, upright, weeping, palm | one of the numbers; it decides the code that runs |
| Preset | numbers by a name, to begin from (oak, birch, pine, spruce, cypress, willow, palm, bare tree) | the family's table; of what was grown from one, only its name is kept |
| Design | what was grown and kept under a name of the person's | a kind of the document, like any other |

**The recipe.** One line of the kind: `grown FAMILY version=N preset=NAME
made=WORD number=value …`. It is the kind's, so it goes with every copy of the
kind (a world written and opened, a copy under another name, the library, another
world), and no other part of the host had to learn of it. `made` is a word for
the mesh as it was grown: when the mesh's word is another, it was shaped by hand
since.

**What happens to what was done by hand.** The recipe is never removed by an
edit of the mesh. Growing such a design again is refused by the host unless told
(`by_hand: discard`); the editor asks, and offers three things: keep it as it is,
keep what is shown as a new design, grow it again. Edits are not carried onto the
new shape: a mesh's lines name corners by number, and another tree has other
corners. Kept through growing again: every line of the kind that is not the
generator's own.

**Names.** A preset is called by one capitalised word in Grow ("Pine"). A design
is called by its name, the same in Grow, Designs, the library and the bar
("pine-1", "pine-tall", "conifer"). A design made from a preset is offered the
preset's name with a number, so that it is never the preset's name itself, nor a
name the library or the code has. The trees that were not grown (the code's tree
and conifer; the library's ten) are listed as that, each with the preset nearest
it: `pine-tall` and `conifer` are different designs and stay so; neither is the
preset Pine or Spruce.

**The wind is one thing.** A weather says its wind, its gusts and how far it
leans and sways what gives wholly (`weather.txt`). The world says, over every
weather, how many times as strong (`air` > `strength`, 0 for none) and the
least there is in any weather (`least`). What moves by the wind reads the
product and nothing else: what the world is drawn with (lean, sway, gust,
flutter), what swings after a body, what falls, the water's ripples. How much of
it a thing takes is the thing's to say: corner by corner, `gives`.

**The way on.** Rocks and boulders: a second entry of the table of families
(numbers: size, how many lumps, roughness, flatness, moss; presets: boulder,
slab, cairn, outcrop), one function, and `solid` as its mesh. The window, the
recipe, the question and the library need nothing new. Then bushes and ferns (the
tree's foliage without its trunk; `gives` from the ground). Kept in sight, not
begun: plants rooted under water, which want `gives` read as a current.

## 7ab. The editor's workflow: where things are found, an eraser, plain names (built 2026-10-13, at the user's asking)

**What was asked.** After trying the tree generator and the living environment by
hand, the user asked for one coherent pass on the editor as a person building a
world meets it, not one more layer of menus: a persistent toolbar with pictures
for the main activities; related actions gathered; a row for what is picked that
keeps the frequent things in one place; plain names; a general way to erase many
painted objects whose group is gone; proportionate follow-ups on the tree
generator and on older trees that stand still; the weather controls the living
demo did not show; and a small tuning of the wind. Painting with groups, replace
and a brush clearing its own layer were confirmed by hand as working and are not
reopened.

**The shape decided.** Three places, each for one question.

| Place | The question | What is there |
|---|---|---|
| The bar (top) | What about the world as a whole, or what I picked? | **World** (the file), **Edit** (undo, redo, and what is done to what is picked), **View** (what is shown; this machine's settings), **Characters**, **Window**; Undo, Redo, Play, Checklist, Keys |
| The toolbar (left, while editing) | What do I want to do? | **Select**, **Move**, **Paint**, **Erase**, **Terrain**, **Objects**, **Plants**, **Build**, **Sky**. Three are tools taken up at once (Select, Move, Erase); five open a short list beside the button (the windows and the things to add of that kind); Sky opens the Environment window. The one in use is lit |
| The row under the bar | What have I got in hand, and what can I do with it? | The tool in hand as a word on a plate (SELECT, MOVE, PAINT, ERASE, TERRAIN, PLACE, BUILD, PLANT, EDIT SHAPE, BUSY); what is picked and what it works on; the frequent things as buttons (Move, Duplicate, Delete, Edit shape, Inspect, Deselect; Tree Generator for a tree made there); the rest under **More**. One line: what is said gives way to the buttons, and is whole in its tooltip |

Rules this pass set, to be kept by later ones:

- **Every capability is under the activity a person would look for it under**, once.
  A window may be reached from two lists only where it truly serves both (World
  outline under Objects and under Plants). Nothing was added that only repeats a
  menu.
- **A list of the toolbar is short**: the windows of that activity and the things
  to add. Settings with several values are rows of buttons in a window
  (Environment), not a long list to scroll. That was the fault found in the living
  demo: the weather and the wind were the last entries of a list longer than the
  window, reached only by scrolling it; while playing only "the next weather" was
  offered. Not a fault of wiring.
- **The tool in hand is said in three places that agree**: the toolbar's light, the
  word at the head of the row, and the window of the tool where it has one.
- **Names say what a person is doing.** The window titles are the plain name with
  the earlier word after it where the text inside still uses it: *World outline:
  forests, streets, the land (makers)*; *Objects in this world (designs)*; *Asset
  Library*; *Terrain: shape and paint the land*; *Tree Generator*; *Object text and
  meshes (kinds)*; *Quick place and nudge*; *Grid layout*; *Terrain tiles*; *Edit
  shape*; *Erase objects*; *Environment*. Inside a window the words *maker*,
  *design*, *arrangement*, *thing*, *sort* stay as the user fixed them on
  2026-10-11: they are exact, and renaming them through 4000 lines of sentences
  would be the wholesale rename the user asked not to have. No host command, file
  format or script name was changed.
- **What chooses by name goes on working.** `desk.pick(heading, begins)` finds an
  entry under whatever heading it is under now; the environment is still a list by
  that name for scripts and keys.

**Erasing: a missing capability, not only a hidden one.** Traced: the host's
`stroke` with `how: clear` could already clear "what any brush painted" (`all`),
but only with a design or a group named, the editor never sent `all`, and a group
that was removed could not be named at all. So after "Remove the group" nothing
cleared what it had painted but one click at a time, or removing the whole maker
in Makers (all of it, everywhere). What was added:

- Host `erase`: an eraser `radius` wide at `dabs`; `painted` (on unless said: what
  any brush painted, whatever it was painted with, the group there or not);
  `placed` (off unless said: things put down one at a time); `only` (of those, one
  design). One edit a stroke. A painted maker left with nothing goes. Never: a
  design, a group, a library's asset; what a wood, a meadow or a street made; a
  house; a tile; the land; what is of a frozen maker. Host `erasable`: how many of
  each there are, for the window.
- The editor: **Erase** on the toolbar takes the eraser up with its window, which
  says the scope in one sentence and in counts, and says where each thing it never
  takes is removed instead (three buttons that lead there). The brush's own
  "Clear it…" is as it was.

**Tree generator, traced and decided.**

- *The 20,000 corners.* Two limits, both the host's. `mesh.rs` `MOST = 20000`: the
  most corners any mesh of the document may come to. A mesh is kept as text, a
  line a corner and a line a face; it is read again whenever something of it is
  built, its corners are shown as dots when it is shaped by hand, and every thing
  of it is drawn with all of it. And `grown.rs`, 19000: a grown tree is refused a
  little under that, so that it can still be added to by hand. It is a safeguard of
  the mesh's form (text) and of what a wood of such trees costs to draw, not a
  number to raise: a presets' tree is 1500 to 6000 corners. What is done: the
  generator says how many corners a tree came to and the most; the window says
  HEAVY from seven tenths of it; and numbers that are too much are said as that,
  with the numbers that multiply (levels, forks, leaves on a branch, boughs) and
  what to lower, with nothing kept or changed.
- *Pictures.* Under each picture is what it is: **preset**, **saved tree**,
  **library tree**, or **reference only**. A reference (a tree that was not made in
  the generator) says that it cannot be loaded or reproduced, that its picture is
  put beside what is made, and that the nearest preset is a starting point and not
  a copy.
- *Primitive trees.* The simpler trees (the code's tree and conifer; the library's
  layered spruce, lollipop tree and the rest) are first-class objects as they are:
  placed, painted, grouped, given to a forest, given wind. They are not replaced
  by grown ones. **To come, not begun:** a second form family in the generator for
  stylised geometric foliage: layered crowns (cones or discs in tiers), umbrella
  forms, flat or plate-like foliage, each from a few numbers and with `gives`
  written for it. It is another function and table entry beside `tree`, not a
  rewrite.
- *Older trees and the wind.* Traced: how a thing gives to the wind is one line of
  its kind (`gives`), which the library's trees and plants have had since
  2026-10-13. A world that brought them in before keeps its own older copies
  without the line: `showcase-tiles.json` has eight such (275 things). Nothing is
  wrong with their meshes. Mended without regenerating anything: host `gives`
  lists the kinds of a world that say nothing of the wind though the library's of
  the same name does, and with `fix` gives each the library's line (one line of
  text; one edit; undone like any). The Environment window says how many there
  are and offers it. For a tree of a person's own that was never the library's:
  Objects > "moves in the wind", the same line with usual numbers. The code's own
  tree, conifer, bush, grass and flowers always gave.
- The protection of a tree shaped by hand is as it was (7aa).

**The wind, tuned.** Three things, none of them every number by the same amount.

- *Clear weather* sways a little more (5 cm lean, 10 cm to and fro at a tree's
  top, where it was 4 and 7).
- *What grows low* (grass, flowers, a bush) gave too little of a sway measured in
  metres at a tree's top for it to be seen: a blade's tip went 2 cm. It is now also
  laid over by a part of its own height, as a wave going over a meadow
  (`world_view.gd` `rustle`): 9 hundredths of its height in clear weather, never
  more than a fifth in any. Nothing with no wind.
- *A storm's beginning.* Found: the swaying was worked out in what the world is
  drawn with as the time so far divided by the weather's beat. While one weather
  turned into another the beat changed, and the time so far divided by it went
  through many rounds in a few seconds: everything that gives thrashed as a storm
  came on. The rounds are now counted on frame by frame by whoever knows the
  weather (`weather.gd`), so a beat that changes changes how fast it goes from
  there on. And the storm itself comes on in 16 s where it took 5, leans 0.75 m
  where it leant 0.9, and gusts less deeply and less often.
- The wind's strength, none at 0, and each weather's own numbers are as they were.

**Later, kept in sight** (none begun): clouds; splashes and wakes; water that
moves what floats by more than the swell; currents; richer weather (fog banks,
lightning, wet ground); seasons. The generator's stylised foliage family (above).
A wind line for kinds like another kind. Painting where: chosen tiles, an outline,
a region (7t). Docked panels and a properties side panel, if floating windows
prove too many.

**Next milestone, unchanged:** adaptive modular construction (L3 of 7o with 7q):
editable structures, walls, paths, gardens, ponds and related scenery, on the
layout and tiles there are and with the objects there are.

## 7ac. After the workflow pass: what a person met, fixed or traced (built 2026-10-14, at the user's asking)

**What was asked.** Further findings from using the editor, to be addressed in this
pass where feasible, grouped, and classified honestly: one tool in hand at a time;
the wheel over a panel not moving the view; Erase as the one way in to every kind of
removal; a grid layout placed by dragging it; panels that keep still, fold, and
scroll; Build's street, which added data and showed nothing; cliffs and ledges that
look wrong as the eye moves; horses and cows, which a person could not find. Shared
defects and broken workflows before looks; existing systems reused; nothing counted
fixed because it compiles or because the data is there.

**1. Fixed, and seen working in the editor** (`-Tour -Only findings`, real mouse
events, and pictures looked at):

| Finding | What it was | What was done |
|---|---|---|
| A new tool did not put the last one away | Each tool kept its own state, and some starting a tool put away only some of the others: a land tool chosen while a brush was in hand left the brush working, since the left button asks the brush first | One rule, one place: `main.take_up(which)`. Whatever takes the left button (a brush, the eraser, placing, moving: `marks`; a land tool; putting tiles down; a mesh's corners; putting one down; tiles said with the mouse; a layout being moved) calls it first and every other is put away. Select calls it with none. Esc ends whichever is in hand, as before |
| The wheel over a window moved the view | Godot passes the wheel on to what is under a list when the list cannot scroll further (and over a window with no list at all), and it reached the view as "nearer" | The wheel over any window, list, the bar or the toolbar is theirs (`main.over_panel`, by the control under the pointer). Only the wheel over the world moves the view |
| Erase was for painted objects only | The eraser took what brushes painted and things placed one by one; tiles, a whole layout, the whole world and what is selected were elsewhere or nowhere | **Erase** is one window in five parts: *What is erased* (painted, placed one by one, terrain tiles; of those, only one); *With the eraser* (the brush); *All at once* (the whole world, or one layout's cells: what would go is said first, with counts, and goes only on "Erase them"); *What is selected* (Delete the one thing; the whole maker, asked twice, said what goes); *Removed elsewhere*. Host `erase` takes `everywhere`, `layout`, `tiles`; `erasable` says tiles and takes a `layout`. Each is one edit. A group's things go whether or not the group is still there. Nothing leaves Objects, Tiles or the library |
| A layout could be placed by numbers only | "Move it to where the view looks" and typed numbers | A yellow ring with four arrows at the middle of the layout chosen in the Layout window: dragged, the layout's cells are drawn where they would be and it is put where the button is let go. Or "Move with the pointer", then a click. Esc leaves it. One edit, its things and tiles going with their cells as the window says; Ctrl+Z. The middle's x and z are numbers in the window too |
| Panels shifted as their words changed | Lines said again and again (what a tool does, what is under the pointer, how many there are) grew and shrank, and everything under them moved | `section.gd`: parts under headings that fold, with Expand all and Collapse all, kept open or folded for as long as the editor runs; and words given room for so many lines whether they have them or not (what does not fit is in the tooltip). Terrain (Tool, Its numbers, What it does, Under the pointer, More) and Erase are in parts; Erase scrolls within its window |
| Build > street showed no street | Two faults. (a) Drawing the line: the road is made on the land, which the host goes on with after it answers, so the answer had no line in it yet: the editor stopped there with a script error, and the street was never added. The line stood; no houses. (b) With a road that already had a street, a second street was added along it and stood exactly on the first one's lots: data, and nothing new to see | (a) What follows a line is added when the line is made (`marks._along`, watched until the host is done). (b) A line with one of these along it already is not used again: a new line is drawn for it, and it is said why. And every maker added says what it made: so many things, or "made nothing here" with why it may be |
| Horses and cows could not be found | They exist and are complete (`town.txt`: horse, grey horse, cow, brown cow, goat, with their moves in `moves.txt`), but only as the island town's own designs: no other world had them, and the library did not list them | A library collection, **Farm animals** (`editor/library/animals.txt`), the same texts with the library's words. Objects > "Animals: horses, cows, goats" and Characters > "Farm animals" open the Asset Library on it; "Place one…" puts one down, which roams and grazes where it is put; a meadow given the design makes a herd |

**2. Traced, the cause written down, and left: cliffs and ledges.** Looked at from
four sides in the editor (`findings` pictures). The land is one height a metre, and
its mesh one corner a metre. A cliff put down by hand is a step in those heights; to
make its face stand sheer, the corner at its foot is moved level until it is under
its top (`land.rs` `lean`). Three things follow, all of the way the land is
represented, none a fault of the stamp's orientation or numbers:

- *At a cliff's ends*, where the step runs out, a corner is moved along one way and
  not the other: thin upright slivers stand there, seen edge on from some sides and
  not at all from others. That is what changes as the eye goes round.
- *Its face is shaded as a slope*: the way a corner faces is worked out from heights
  half a metre either side, so the face's light is the light of the top and the
  foot run together, and it looks soft from in front and dark from behind.
- *Its colour runs from the top's grass to the foot's rock* across the face, for
  the same reason (a colour a corner).

Mending these properly needs a face that is a face: corners of its own along the
cliff, facing out, with its own colour, not corners shared with the ground above and
below. That is the first of the **basic modelling tools** (7f, 7q): cliffs, walls,
shelves as shapes of their own laid into the land. It is not a few lines in the
land's mesh, and patching `lean` would change every cliff in every saved world for a
part of the problem. Left as it is; carried to that milestone.

**3. Deferred, to a named milestone.**

- **Basic modelling tools** (7f, 7q): cliffs and ledges as faces of their own (above);
  then walls, shelves and terraces. After adaptive modular construction's first tier
  if that is what the user chooses; not pulled into it.
- **Asset generation (E1-E3)**: animals beyond the five now in the library. Dogs,
  rabbits and squirrels are still only the town's own (`town.txt`), and sheep and
  cats only the probe's (`cast.txt`): they are the next to bring into the library,
  with words of their own, when a person needs them. Pigs, chickens, ducks and deer
  do not exist anywhere: each is a kind to write (a body on the sheep's bones, as the
  horse is, and its set of moves). Recorded, not begun.
- Folding parts for the other long windows (the Tree Generator, Objects, the
  Inspector, the Library): the same `section.gd`, window by window, when each is next
  worked on. World outline already folds.

**Rules this pass set:**

- One tool in hand at a time, put away in one place (`take_up`). A new tool calls it.
- The wheel and the pointer over a window are the window's.
- A command that succeeds says what came of it: so many made, or that nothing was,
  and why it may be. A maker that makes nothing is not a silent success.
- A clearing of more than a stroke is said before it is done, with what and where,
  and is one step of undo.
- A design kept in one world only is not available: available is listed where a
  person looks (the library, under its activity on the toolbar).

**Next milestone, unchanged:** adaptive modular construction (L3 of 7o with 7q).

## 7ad. Adaptive modular construction, first tier: a structure on cells (built 2026-10-14)

L3 of 7o, with 7q, begun at the user's word ("Proceed to adaptive modular
construction"). The milestone as named at the end of 7ab: "editable structures,
walls, paths, gardens, ponds and related scenery, on the layout and tiles there are
and with the objects there are". This is its first tier, and the first tier of the
system 7o asked for: **a structure as one maker whose arrangement is cells**, a maker
that reads a layout, not a building generator beside it.

**What a structure is.** A maker of the world (`structure`, `land::build`), listed
under Buildings as "Wall, fence or hedge on cells". It is told cells of a layout
(`on`, the same words a wood is told tiles in: `layout NAME; cells 1,2 3,2`, or every
tile of a layout, or its tiles of a ground). Round the **rim** of those cells it
builds, as it is told (`as`):

- a **stone wall** (courses of stones, each a little proud on either face, and a
  coping), a stone **pier** at every joint;
- a **fence** (two rails, posts no more than 2.4 m apart, pales), a **post** at every
  joint;
- a **hedge**, with a **corner** only where the rim turns;
- or **nothing**, when it is only its floor.

**Gates** in the sides it is told (`gates`: `2,1 east; 3,3 south-west`): a gap the
width of a gate in the middle of the side, the side's pieces either side of it, piers
of its own outside the gap where the side is long enough, and a leaf of boards that
swings (`leaf`; off, the gate is a gap). A **floor** on every cell if asked
(`floor`: paving, gravel or boards): level at the height of the highest ground in the
cell, down to the lowest, so on a slope it is a step. Its **height** (`height`; 0 the
usual: a wall 1.2 m, a fence 1.1, a hedge 1.6) and a **seed** for which stone is
where.

**What makes it adaptive.** Nothing of it is stored but those words. After every
edit, whatever it was, the host works out what the cells come to in the structure's
own frame (`over`, as for a wood on tiles) and their rim (`rim`: each side by its
cell and side, its two ends, how far the rim turns at its first end, whether a gate
was said for it), asking `layout.rs` (`edge`, `rim`, `side_near`, new). So a cell
added or taken away, a layout moved, turned or made another size, a tile put in a
layout whose tiles it surrounds, and the structure is built again in the same edit:
walls join, corners turn, a side that comes to lie between two of its cells has no
wall, and its gate waits under its name until the side is on the rim again.

**Kept by hand.** Every piece is named by its cell and side (`piece#c1_2_south`,
`pier#…`, `gate#…`, `floor#c1_2`), so a piece taken away, moved or given another
design by hand is kept by the maker as any maker's things are (the six ways of
amendment 3), and stays so when cells are added elsewhere. It is in
`KEEPS_BY_HAND`.

**On the ground.** Each piece stands on its own cell's ground (`Ground::height`:
the land, or a raised tile over it; 7w), sampled a little inside the side: its foot at
the lowest of the ground along it, less a sink, its top so high over the highest. A
wall on a slope steps a side at a time; a pier stands as high as the higher piece it
joins. A side between a raised tile and a lower one stands on the cell it encloses.

**In play.** Every piece is a solid as it is drawn (a box; a floor its own hull); a
gate is opened and shut by its latch from either side, as a door is (`opens`,
`offers open`/`shut`).

**In the editor.** Build (toolbar): "A wall, fence or hedge round cells" (a click on a
cell puts one down round that cell, and from then on a click on a cell adds it, again
takes it away, Esc ends); "A path, yard or terrace" (the same, built as nothing with a
floor); "Open or close gates in the structure chosen" (a click near a side). World
outline, the structure chosen: "Its cells" (Choose cells…, Open or close gates…, Round
all tiles of…), built as, height, floor, gates swing, the gates said; its seed under
"How it is built". A structure stands where its cells are: it has no "Move the whole
maker"; it is moved by moving its layout, or by other cells.

**Decisions taken building it, for the user to confirm or change.**

1. **One maker for walls, fences, hedges and floors** (`as`, `floor`), not a maker
   each: they are the same arrangement of cells, and a yard is a wall and a floor on
   the same cells. A garden is a hedge round cells and a meadow told the same cells
   (two makers: planting stays the meadow's).
2. **The rim is what is built round**: the cells' outline, holes included (a hole in
   the cells is walled round on its inside too). Inner walls between cells (rooms)
   are not in this tier.
3. **Piers at every joint of a wall or a fence, a hedge's corners only where it
   turns.** Pieces run from corner to corner and end inside the pier, so nothing is
   seen to overlap.
4. **A gate is said by a side, and kept by its words** (`cell side`): a side not on
   the rim keeps its gate waiting rather than losing it.
5. **`as`, not `of`**: `of` is already the host's word for the designs a maker is
   given (D1), and setting it on a structure was refused as "not given designs".
6. **The structure follows its cells, not its node.** Its node's place is nothing to
   what it builds.

**A fault found and mended on the way** (not of this tier, met by its tour): choosing
a row of a toolbar list that gives the pointer a tool for "click where" (Build > "Put
down a house…", and now "A wall… round cells") put the thing down at once under the
list: the list is put away while its row is being handled, and the button's release
then went on to the world as the click that says where. The row now keeps its release
(`desk.gd`). The older tours chose those rows by script (`desk.pick`) and never saw it.

**What the L3 table of 7o now stands at.**

| L3 needed | After this tier |
|---|---|
| edges and corners as places | **built** for a structure: a side of a cell named by its cell and side (`Edge`, `side_name`), a corner as the first end of a side of the rim. Not yet for anything else (a lamp on a corner put down by hand) |
| pieces generated to fit | **built** for a run of sides: a piece a side, piers at joints, a gap where a gate is asked for. Not yet: a piece longer than a side, a piece chosen by its neighbours (an end piece, a corner piece of its own) |
| a structure as one maker whose arrangement is cells | **built** |
| one cell changed by hand kept as a change by hand | **built**, by piece names |
| levels (a layout above a layout) | **owed** |
| styles reaching its pieces | **owed**: the pieces take the style's stone, wood and greens; a style cannot say "dry stone" or "brick" |

**Owed after this tier, in the order use is likely to ask for it** (none begun):

1. **Walls inside**: sides between two of its cells walled too, by choice (rooms,
   pens, a maze), with doors as gates are. **Built** (7af).
2. **Roofs that span**: a roof over its cells (a building is walls of storey height,
   doors and windows in sides, and a roof); the house generator's roof given an
   outline of cells instead of a box. **Built** (7af), a roof over each rectangle.
3. **Steps and ramps** where a floor or a raised tile meets one lower or higher, and
   floors that follow a raised tile's ground. **Built** for floors (7af).
4. **Ponds**: water as a floor at a level of its own. This needs water with a level
   of its own (risk 3 of 7o; 7q's seventh): it is not to be made as a sheet laid on a
   cell before that. **A raised pond built** (7af): a kerb with water at one level in it,
   on the ground; a pond in the land still waits for 7q's water.
5. **Pieces as designs**: a wall piece, a pier, a gate that is a kind of the
   library's, chosen for a structure as a wood is given designs, so a person's own
   fence is built round cells.
6. **Pieces chosen by their neighbours**: an end where a run stops, a corner piece
   that is not a pier, a gateway arch.
7. **Paths along a line** (W1's lines) rather than cell by cell, and paths that join
   (a crossing piece).

**Risks, as they stand.**

- **Every structure is built again at every edit that changes its cells' rim**; one
  with hundreds of sides is a few hundred leaves. Measured nowhere beyond the tours'
  sizes.
- **The stone wall's look is code**, not a design or a style: it cannot be changed
  by a person except piece by piece.
- **A side shared with another structure's cells** gets two walls, one each.
- **A gate's leaf is hung for a person's height**, not the wall's: on a 3 m wall it is
  a gate in a gap.

**Rules this tier set.**

- What fits itself to cells asks `layout.rs` for every side and corner and keeps no
  arithmetic of cells of its own.
- Every piece of a structure is named by the cell and side it stands on.
- A structure's sort is a word (`as`), its look a seed; neither moves a piece.

**Tried by hand (the user, 2026-10-14).** It works adequately; the roadmap goes on. One
fault met: *made higher, the gate seemed to go and had to be made again.* The host kept
the gate whatever the height or the sort (checked since in `p58`). The fault was the
height's field: it showed 0 for "as usual" (a wall's 1.2 m), so its **+** set 0.1 m,
which the host keeps to 0.3 m: the wall fell to knee height and its gate's leaf with it,
hardly to be seen. Mended: the field shows the height the structure stands at (the host
says it, `high`, beside its sort's `usual`) and steps from there; **As usual** puts
it back when it has a height of its own. The tour's `build` part clicks **+** and finds
1.2 m made 1.3 m and the gate kept.

## 7ae. Editor findings held for when the editor's systems are next improved (the user, 2026-10-14)

Not a milestone, and not to delay the roadmap. Each is taken up when the part of the
editor it touches is next worked on, and said in PROGRESS.md when it is.

1. **Settings before placing or painting.** Choosing a maker's tool or a paint tool
   (a meadow, a wall, a fence, a hedge, a brush) shows its settings and numbers at once,
   before the first click or stroke, and what is put down is made by them; today most
   are reached only in World outline or the Inspector after something has been made.
   The same for every tool that places or paints. Where it is begun: the tool's own
   row (the bar under the toolbar, or the window that tool opens), holding the words a
   maker would be added with. Not every tool at once. **Begun** (7af): Build's house,
   fence, hedge, path and pond are each a structure made as it says from the first
   click (`begin_adding(gen, with)`); the rest wait.
2. **Numbers dragged.** A number's field changed by pressing on it and dragging left or
   right, as in Blender, at a pace that suits its step; written as now for an exact
   value; perhaps Shift for finer and Ctrl for coarser. `num.gd` is the one number
   control, so it is done there once. A drag that is an edit of the world is one step of
   undo, not one per value it passes.

## 7af. Modular construction brought to a stopping point: rooms, roofs, steps, ponds, a village (built 2026-10-14, at the user's asking)

The user, after trying 7ad by hand: bring L3 to a coherent stopping point in one batch,
on the cell-based structure there is (not beside it), with a village to show it, then
stop expanding construction and name the engine systems to go to next. Breadth and
integration over polish.

**What a structure on cells now is.** Still one maker (`structure`, `land::build`), its
arrangement cells; what it is built as (`as`) now one of six:

| `as` | Round its cells | Inside |
|---|---|---|
| wall, fence, hedge | as in 7ad | walls inside (pens, a maze) with gates in them |
| **building** | the house's own **facades** (a wall, windows by bays, a door where a gate is said), a **corner** post where it turns; its walls stand at its floor, a **footing** under them where the ground falls away | **walls inside** (the house's wall with a doorway) and **doors** in them; boards or any floor; a **roof** |
| **pond** | a low stone **kerb**, a corner where it turns, no gates | a dark **bed** and **water** at one level over the highest ground, seen through (`sheer`) and water for what swims or hops (`water R H`) |
| none | nothing | its floor only: a path, a street, a square, a terrace |

Besides that:

- **Walls inside** (`walls`, said as gates are: `1,1 east`): a side between two of its
  cells, walled; said from either cell it is the one wall. Host `build_wall`; a gate
  said in one is a door (a building) or a gate (the others). Taking the wall away takes
  its door.
- **Roofs** (`roof`: none, tiles, slate, moss, thatch, the style's roof colours in that
  order): the house's own roof over each **rectangle** of its cells (a run of a row and
  the same runs in the rows after it: an L is two roofs), its ridge the longer way, on
  the eaves (a building's are level all round, over the highest ground of its sides).
  For a building or a stone wall (a barn); hexes have one each.
- **Steps or a ramp** (`steps`: steps, ramp, none): wherever a floor stands more than a
  step (0.22 m) above what is beside it (the ground, or a lower floor of its own) and no
  wall stands between, down from it, as wide as the way (a gate's, a door's, or most of
  an open side). Walked up as a slope (the `ramp` solid).
- **Where structures meet.** A side between the cells of two structures on the same
  layout is built **once**, by the one that is more of a building (`rank`: building,
  wall, hedge, fence; the first made where alike); the other is told it is `shared` and
  builds nothing there. A gate either says in it is a gate of the one that builds it: a
  yard's gate in the wall it shares with a house is a door of the house. A floor alone
  and a pond take no part (each builds its own side).
- **Floors**: garden **soil** (beds) beside paving, gravel and boards.
- **In the editor**: Build lists a house, a fence (a pen), a hedge (a garden), a path or
  square, a pond, as well as the wall; each is a structure **made as it says from the
  first click** (`begin_adding(gen, with)`: the first of 7ae's held findings, for these
  tools); "Walls inside the structure chosen" and "Open or close gates and doors".
  World outline, Its cells: **Walls inside…** (clicks), **roof over its cells**, **where
  a floor stands high**, the walls inside said, the sides another builds said. The maker
  is now called **Structure on cells**.

**The village** (`editor\showcase-village.json`, `editor\showcase.ps1 -Which village`,
`host\examples\village.rs`; `run.ps1 -Demo village`): one square layout of 4 m cells, 16
by 13; a street and a square of paving with lanes off them; a hall of two rooms on a
terrace raised a metre, steps down from it on every open side; the baker's house of two
rooms with a door to the street and one to its walled yard behind (the side they share
built once, by the house); a smith's house, a cottage (an L, two roofs), a shop; a stone
barn roofed in thatch; a pen of two paddocks with cows, a goat and a horse; a hedged
garden of soil beds with a meadow of flowers told the same cells, beehives and a
scarecrow; a pond off the square; an orchard (a wood of apple trees told its cells); a
well, benches, lamps, a café table, furniture inside the houses, woods beyond. 12
structures, 976 things; written, opened and written again the same file. Built only of
commands a person gives from the editor.

**Decisions taken building it, for the user to confirm or change.**

1. **A building's sides are the house generator's facades** (and its roof the house's
   roof), not new pieces: a house on cells looks like the houses there are, has their
   finished-project look, and their doors open as theirs do. Its door is in the middle
   of a side; windows are wherever the facade puts them.
2. **Which of two structures builds a shared side** is by kind (`rank`), then by which
   was made first. A floor alone and a pond never build another's side.
3. **A building's walls stand at its floor**, with a footing below where the ground
   falls away (found by the village's terrace: walls stood at the lowest ground under
   them, and a doorway measured from there was lower than a head on the floor).
4. **Steps appear by themselves** wherever a floor is high and nothing walls it off;
   `steps: none` turns them off for one structure. A path across a slope gets many.
5. **A roof is laid over rectangles of cells**; an L or a U is several roofs meeting,
   not one roof with valleys.
6. **A pond is raised**: a kerb on the ground with water in it, not a hollow in the land
   (the land layer is not dug by a structure; 7q's water with a level of its own is
   still owed for ponds and rivers in the land).
7. Code version of `structure` 2: worlds made with 7ad's are made again on opening
   (same pieces, same names, new pieces where asked).

**Not done, and where construction stops.** Construction stops here (the user: move
across the engine). Owed when it is next taken up, in the order use is likely to ask:
storeys and stairs inside (a layout above a layout: 7o's levels); a roof that is one
roof over an L (valleys, hips); windows and doors placed by the person (a facade's own
handles); pieces as library designs and chosen by their neighbours (7ad's 5 and 6);
paths along a line; ponds and streams in the land (needs 7q's water). A shared side
between a building and a building gets one facade, whose windows look into the
neighbour. A roof's chimney is on every rectangle. Steps can cross into a neighbouring
structure's cell. A structure's name when added from Build is "structure N".

**What the engine has to go to next, recommended** (the user asked for this at the stop;
none of it begun). The village is the place to try each in, and each first tier is
meant to be seen working there.

1. **Runtime playtesting**, first: a play session that is a tool, not a toggle. Start
   from any place or a saved test start; restart in a second; what happened written
   down (who went where, what was done, facts set, what got stuck); a pass over every
   door and step of a structure (can the walker get in, can it get out); later, a
   playtest recorded and played back. It multiplies every system after it.
2. **Navigation**: paths for everyone else, not only the person walking. A way graph
   that knows structures (a door, a gate, steps, a shared side) and the land; NPCs and
   animals go round walls and through doors (opening them); a pen keeps its animals in.
   Builds on `keeper`'s ways and the structure's rim, which already knows every way in.
3. **NPC and animal behaviour** (7g, planned and not built): routines and reactions
   authored as parts; the baker goes from bed to bakery to square and home through the
   doors; animals graze within their pen and come to a gate; reactions to the walker,
   the hour, the weather. Uses navigation; the cast and wild.gd are its first bodies.
4. **Interactions, inventory and resources**: what can be done with what (acts, items
   found, kept, held, used: there is a first tier) carried to resources that are made
   and used up: bread from the bakery, apples from the orchard, eggs, wood; a container,
   a shop, a crafting step. Every thing in the village a place something can be done.
5. **Dialogue and quests**: speech authored as parts (who says what, when, under which
   facts), choices that set facts, a quest as facts and steps with its state saved
   (the facts and the saved game are there); the first one in the village (a goat out
   of its pen; bread to the hall).
6. **Movement and animation**: what the walker and everyone does looking right: steps
   and stairs climbed as steps, a door opened by a hand, sitting, carrying, the moves
   there are (vault, wall run) blended, animals' gaits; the cast's parts that bend, given
   their animations.
7. **Combat and abilities**, last: a genre's, built on the movement, behaviour,
   interactions and items before it (9 of the master roadmap).

A milestone that crosses them is suggested, rather than each alone: **"a living
village"**: playtesting (1) to see it, navigation (2) and routines (3) for its people
and animals, a resource and a trade (4), one quest (5), and the movement that makes it
look right (6), each at a first tier. **Begun the same day (7ag):** playtesting,
navigation through doors and gates, and days kept by places' names, in the village;
4 to 7 wait.

## 7ag. A living village, first tier: people and animals who find their way through it, and a play session that watches it (built 2026-10-14, at the user's asking)

**Why this milestone.** The user asked to move from construction to the next engine
systems, choosing from the repository's actual state and not building the whole list.
What there was: a village of structures with doors (7af) and nobody in it; navigation
(`nav.gd`, Godot's navigation server) in which a shut door or gate was simply a wall;
people (`folk.gd`) who could keep an hour (`goes HOUR x z`) but only at coordinates of
their own, and could not open a door; acts (`acts.gd`) that let anyone with a hand open
one, used only by the walker; and playing as a toggle with nothing to watch it by. So
the highest-value step was the one 7af put first and second, joined to what makes them
visible: **runtime playtesting** and **navigation through structures**, with the first
tier of **routines** (7g) that they serve, all in the village. Interactions, resources,
dialogue, quests and combat were left for later (below), so that this is one coherent
increment and not several thin ones. The user's longer-term construction direction
(`Draw3D_Modular_Construction_Feedback_and_Next_Systems.md`: a Tiny Glade-like tool-led
workflow) is postponed by their decision; what was built here reads structures by name
and doors by their traits, so it holds for structures made that way later.

**What is built**

- **Doors as ways, for those who open them** (`nav.gd`). Every door or gate that opens
  by hand (`opens`, and `offers open`) is a link across it, a step before it to a step
  behind it, on a navigation layer of its own (`DOORS`). `route(from, to, opens)` asks
  for the ground only, or the ground and the doors, and says which doors the way goes
  through. A door that stands open is a way for everyone, as before.
- **Who opens doors** is whoever could do the act `open` (`acts.gd` `could`: the move
  `reach` in their set, a hand on their body). Nothing new is said in content: people
  can, animals cannot.
- **People open, go through and shut** (`folk.gd` `_doors`). Come to a shut door on
  their way, they open it by the act, as the walker does; once through, they turn back
  and shut it, by the act `shut`. A door they found open they leave open. What is said
  of it is said when the door actually opens or shuts, not when it is asked.
- **A pen keeps its animals** while its gate is shut (they have no way out), and not
  when it is left open: nothing was written for pens; it follows from the two above.
- **Places by name** (host `nodes`: a structure's `place`; `goes HOUR at PLACE` in a
  kind, `at home` for where it was put). A structure is a place: the middle of its cell
  nearest the middle of them all (never on a wall inside), the ground's height there,
  and its cells' corners. The people of a world keep their day by the village's names.
- **Where each is, and what happened** (`folk.gd` `place_of`, `events`): the structure
  someone stands in (after a moment in it, so a doorstep is not two places); a log of
  doors opened and shut and by whom, who came into or left which place, who reached
  where they were to be and who could get no nearer, and how near.
- **The Playtest window** (`playtest.gd`; P while playing, or the bar's "Playtest"): the
  hour and how fast it goes (as it runs, ten times, an hour a second) and hours to jump
  to; everyone, what they are doing, where they are and where they are to be, and for
  the one chosen **Watch them** (the eye follows them), **Back to the walker**, **Go to
  them**; how many doors are ways and how many are shut; **Check the ways to every
  place** from where the walker stands (reached or how far short, how long, through how
  many doors, and which are shut to animals); what happened, newest last; **Start again**
  (from the world's start, or here: what happened undone, run again at once).
- **Someone made indoors stands on the floor**, not on the roof (`body.put` with `top`;
  `solid.under` from a height): every body used to be dropped onto the highest surface
  under it, which in a roofed house was the roof.
- **The library's Village people** (`editor\library\people.txt`): someone of the
  village, the baker, the smith, the shopkeeper, the gardener, a boy and a girl, each
  able to open doors; a world gives one a day with a kind of its own that is `like` it.
- **The village, lived in** (`editor\showcase-village.json`, rebuilt): "the square" is a
  structure of its own now (13 structures); four people with days of their own (the
  baker: home, the yard, home, the square, home, the square, the hall, home; the smith:
  the barn and back; the shopkeeper: from the cottage to the shop; a neighbour: the
  garden, the pen, the square, the hall), the gardener among the beds, two children and
  someone else roaming, the animals in the pen. The pot plant that stood on the shop's
  doorstep was moved: the Playtest's own check and the shopkeeper's day found it.
- **`-Demo villagers`** (6 beats) and **`-Tour -Only villagers`** (new).

**Decisions taken building it, for the user to confirm or change**

1. **Who opens doors is said by what a body can do** (the act `open`), not by a new
   word: every person can, no animal can. A person who should not (a child kept in) has
   no way to say so yet.
2. **People shut what they open, and leave open what they found open.** A gate left
   open by the walker stays open, and the animals may find it.
3. **A place is a structure, at its middle cell.** Not yet a room, a door, a bench or a
   point marked by hand; `goes HOUR x z` still says a point.
4. **A day is a kind's** (`kind the_smith like smith` with its `goes` lines), not one
   placed person's own lines: 7g's "one put down may say lines of its own" is not built.
5. **The log is not kept**: it is for watching a run, and begins again with each.

**How it was checked** (PROGRESS.md has the numbers): host unit and protocol tests (p59
asks for a structure's place); `-Tour -Only villagers` with no problems (the smith
through two doors to the barn and the shopkeeper through two to the shop, each door
opened and shut again; four animals with no way out of the shut pen and a way out when
it is open; the Playtest window opened by P, twelve rows, the ways to 12 of 13 places
found, a house reached only through its door, the walker put beside someone, an hour a
second; started again with everyone back where they were put); `-Demo villagers -Check`
6 of 6; the parts beside it (PROGRESS.md). Pictures looked at. **Not tried by hand.**

**Not done** (in the order use is likely to ask)

- Interactions and resources (7af's 4): bread from the bakery, apples, a shop that
  sells; the people do not use anything yet, they only go where their day says.
- 7g's conditions, sequences and reactions (M4a, M4b): a day is places by the hour and
  nothing else; no "when it rains", "greet this one", "sit on the bench".
- Dialogue and quests (7af's 5).
- Movement and animation (7af's 6): doors are opened with the walker's reach; nobody
  sits, sleeps, carries or climbs stairs; steps are walked as slopes.
- Navigation: people go round one another only by stepping aside when stuck; two at
  one door can bump; animals are not herded and do not come to a gate; the ways are
  made only about who is moving (a check asks for them and waits).
- Playtesting: no recording and playing back; no "start here" from a place chosen with
  the mouse (Start again here is from where the walker stands); the log is not saved.

**Recommended next milestone: "the village at work"** (interactions, resources and one
quest, on 7g's first two steps). **Built the same day: 7ah.** The baker bakes bread that is a thing that can be
taken, bought or given; apples picked in the orchard; the shop holds and sells; one
quest written as facts and steps (the goat out of its pen, or bread to the hall),
offered by someone who says something; and under it 7g's M4a (conditions: the hour,
the weather, a fact, who is near) and M4b (a sequence of steps: go, act, wait, put a
fact), so that "bake, carry to the square, sell" is written and not coded. It uses
everything here (places, doors, routines, the Playtest window to watch it) and is what
makes the village a game. Movement and animation (sitting, carrying, stairs) can come
in the same pass only where a sequence needs it; combat stays last.

## 7ah. The village at work: goods, a day's work as sequences, and one quest (built 2026-10-14, at the user's asking)

**Why this milestone.** 7ag recommended it and the user asked for it: the village had
people who went from place to place and nothing to do there. The user's brief asked for
one complete gameplay loop through the real editor and runtime: reusable NPC behaviour
(conditions and sequences, 7g's M4a and M4b), a small real resource loop, one quest
the walker can finish or fail, and Playtest showing all of it. No combat; no
construction pass (the direction in
`Draw3D_Modular_Construction_Feedback_and_Next_Systems.md` stays postponed; nothing here
depends on how a structure was made: things are found by what they are called and the
place they are in).

**What there was to build on.** Acts (`acts.txt`, `acts.gd`: come, do, a fact put; the
walker's E and the folk's doors use the same code); facts kept by the host and saved
with a game (`state.gd`); places by name and days by the hour (7ag); the Playtest window.
Carrying one thing (a parcel) and keeping things (a lamp) existed; counted goods,
conditions, sequences and quests did not.

**What is built**

- **Goods** (`goods.txt`, `goodsbook.gd`, `goods.gd`). What is counted and not one thing of
  its own: bread, apples, flour, coins. How many is a fact, `has.NAME`, about whatever
  holds it (a thing, one of the folk, the world for the walker), so a saved game keeps it
  and stopping the world undoes it. A thing says `stock NAME N [MOST]` and `shows NAME x y
  z` (what it holds is drawn on it, in rows); whoever has goods that are `carried` holds
  them before them and holds themselves as one who carries. Things are found by what they
  are called or their kind, in a place by name (`named`).
- **Acts that move goods** (`acts.txt` lines `makes`, `uses`, `gains`, `hands`; `for
  walker`, `for folk`; `talks`). An act is offered only while all its goods lines can be
  done, and they are done together at its moment. When the walker stands at something
  whose acts are refused only for want of goods, the prompt says why ("Buy a loaf (2
  coins) at: The market stall has no loaves"; "You have no coins"). Ten new acts: talk,
  bake, take_bread, take_flour, add_flour, stock_bread, buy_bread, pick_apple,
  stock_apples, sell_apple, leave_bread.
- **Conditions** (`cond.gd`, 7g's M4a, first tier): `hour A B`, `has N GOOD [at THING [in
  PLACE]]`, `room N GOOD at THING`, `there THING`, `reach PLACE`, `fact NAME`, `weather
  NAME`, `not ...`. Each answers why it does not hold, in words; one vocabulary for work
  and quests.
- **Sequences** (`sequences.txt`, `seqbook.gd`, `work.gd`, 7g's M4b, first tier): steps `do
  ACT at THING [in PLACE]`, `go PLACE`, `wait M`, `say WORDS`, `check COND`, `put/clear
  FACT`; `needs COND [else SEQUENCE]`; `rests M`. A day's line `goes HOUR do NAME` makes
  it someone's work from that hour (`folk.gd` steers; `work.gd` says where and what next;
  acts do it). What cannot be done is said, not waited on: no such thing, nothing there,
  no room, no way (as near as they got), refused three times, too long on the way. They
  stop, say why, rest, try again; when the hour moves on they leave off.
- **Quests** (`quests.txt`, `questbook.gd`, `quests.gd`): taken on by talking (E beside
  someone: the act `talk`); steps each done when a condition holds of the walker; hints
  said while a condition holds; what fails it; what is given and the reward. How far it
  has come is a fact (`quest.NAME`). Shown under what the walker has, and told when a
  step is done, the quest done, or failed. Someone with no quest says what they `speak`.
- **Playtest** grown: what each is at (the sequence, the step, or why they stopped) and
  what they carry; a goods panel (every thing that holds goods, things of one name in one
  place together; who carries what; what the walker has); the quests and where each
  stands; and in what happened every loaf made, taken and handed over, every sequence
  begun, finished, stopped and why, every step of a quest. Every line of what happened is
  also printed to the run's log (`told:`), for whoever reads a run after.
- **The library's Village work** (group of Rural and island): bread oven, flour sacks,
  market stall, shop counter, long table; the apple tree holds four apples to pick; the
  Village people can all be talked to and say something of their own.
- **The village at work** (`host\examples\village.rs`, showcase rebuilt, 990 things): the
  oven in the baker's back room, flour in the barn, the stall in the square, the counter in
  the shop, the long table in the hall; the baker's day `do bake_bread` (6:00 to 12:30,
  13:30 to 17:00), the gardener's `do pick_apples`, and the cook in the hall who asks
  "Bread for the hall's supper" (four coins given, two loaves bought and left on the long
  table, three coins paid; failed if supper at seven comes first).
- **Found and mended on the way**: a way that went up a lip at the hall terrace's edge
  that no body could step: the baker stood against it for ever on his way to the barn.
  Ways were made up lips of 0.4 m and bodies stepped up none at all. Now the folk step up
  onto a lip of 0.3 m (`body.gd` STEP_UP; made lower, 0.2, the ways shut the gardener in
  the garden, whose floor's edge is such a lip), and ways are made up no more than that
  (`nav.gd`). Someone
  standing on the spot before a stall no longer keeps others from it (only what is built
  is in the way of a place). The baker standing by his stall no longer takes the
  walker's E from the loaf they came to buy: of what is in reach, what the walker looks toward
  comes first, and a thing before someone passing. The folk choose what to work at by
  whether they could do it once there (`acts.not_now`), not by the walker's "offered
  now", which also asks for a clear line to the spot from where they stand.

**Decisions taken building it, for the user to confirm or change**

1. **Goods are counts, not things.** A loaf is not picked up one by one as a parcel is:
   there are so many on the stall and so many in hand, and it is drawn from the count.
   Things that are one of their own (a parcel, a lamp) stay as they were.
2. **Prices are in the acts** (`buy_bread` says `hands coin 2`), not in the stall: one
   act per thing sold. A stall's own price list would be a later word.
3. **Talking takes a quest on at once**: there is no "accept" or "not now" yet.
4. **The walker's goods are about the world** (facts about no thing), as the walker's
   other facts are.
5. **Nobody's work is kept across a save**: a sequence begins again from its start when
   the world runs again; what was done (goods, doors) is kept.
6. **The folk step up onto a lip of 0.3 m, and ways are made up no higher.** The walker
   does not step up (how the walker moves was left as the user last tried it).

**How it was checked** (PROGRESS.md has the numbers): host tests (p43 knows the new
group); `-Tour -Only work` in play, on a copy of the showcase: the baker bakes, takes the
bread out, carries it and lays it on an emptied stall; the walker takes the quest from the
cook, is told the stall is empty and why, buys two loaves when the baker has brought
them, is refused a third for want of coins, leaves them on the long table and is paid;
picks an apple and sells it; the gardener picks three and sets them out at the shop;
Playtest shows each at work, the goods and the quest; the hour moving on sends the baker
home; without flour he fetches it from the barn; with none there he says so; started
again, the quest taken and failed at seven. `-Tour -Only villagers` still passes. Pictures
looked at. **Not tried by hand.**

**Not done**

- Reactions (7g's M4c: greet this one, shelter from rain), profiles (M4d), and a window
  made for writing sequences and quests: today they are texts, kept by the host with the
  world like the moves and the acts, and edited in What is written (F10) or as the files
  beside the editor (`editor\godot\sequences.txt`, `quests.txt`, `goods.txt`).
- Dialogue beyond one line (choices, accept or decline), quest markers in the world, a
  journal of finished quests.
- Goods are not loose things that can be put down anywhere; apples are not taken off the
  tree's mesh; nothing grows back or is eaten.
- NPCs do not trade with each other for coins, and are not paid.
- A sequence is one list: no branches but `needs ... else`, no loops but repeating it.
- Work is not kept in a saved game (decision 5).

**Recommended next milestone** (the user chose dialogue and reactions: 7ai)**: "a living economy in one valley" or "dialogue and
reactions"**, to be chosen by the user: either 7g's M4c (reactions laid over work: the
baker greets the walker, everyone shelters from rain, someone comes to the walker who
waits at the stall) with dialogue that can be answered (accept, decline, ask), which
makes the village a place one talks to; or the economy deepened (folk paid for work,
buying from one another, things that grow back and are used up over days), which makes
it a place that runs by itself. The first is closer to what a game needs next.

## 7ai. The village answers: reactions, talks with answers, markers and a journal (built 2026-10-14, at the user's asking)

The user asked for 7ah's recommendation, "reactions and dialogue, and all related to the
village milestone", before the next milestone (general-purpose physics, 7aj). Built on
branch `m2-stable-terrain` over commit 37e788d. What 7ah listed as not done and what was
done of it here:

| 7ah's "not done" | Now |
|---|---|
| Reactions (7g M4c: greet this one, shelter from rain) | **Built, first tier**: `reactions.txt`, `reactbook.gd`, laid over work by `work.gd` |
| Dialogue beyond one line (choices, accept or decline) | **Built**: `talks.txt`, `talkbook.gd`, `talk.gd` |
| Quest markers in the world | **Built**: "!" over who has a quest to ask, "▼" over where the step is done (`at THING` under a step) |
| A journal | **Built**: J, a window (`journal.gd`): quests and their steps, and what was said |
| Profiles (M4d), a window for writing sequences, quests and talks | Not built: they are texts in What is written (F10), as before |
| Goods put down anywhere, apples off the mesh, growing back; folk paid or trading | Not built (the economy deepened, 7ah's other choice) |
| Work kept in a save; branches beyond `needs ... else` | Not built |

**Reactions** (`reactions.txt`). A reaction is a sequence (the same steps, read by the
same lines of `seqbook.gd`) with what begins it: `of WHO` (everyone, people (whoever opens
doors), or someone by what they are called or their kind), `when CONDITION` (all of them),
`while CONDITION` (as when, and kept to after its steps until it does not hold), `urgent
N` (work is 0) and `again M` (minutes). `work.gd` looks a few times a second, for
everyone with a body (not only those with work), at what of theirs holds; the most
pressing begins once the act they are in has ended, over a less pressing one. Their work
waits and is taken up again from the step it was on (a wait begins again; a `do` chooses
its thing again). One done does not come again until what began it has stopped holding
and its minutes have gone: a greeting is once each time someone comes. What cannot be done
ends it, said. New steps for anyone's sequences: `face WHO` (turned to them while they
stand), `move MOVE` (asked of their body by name and waited for), `go walker`, `go
indoors` (the nearest place whose structure has a roof; the host already said a
structure's roof, `folk.gd` now keeps it as `roofed`). New conditions (`cond.gd`): `near
WHO R` (the walker, one of the folk by name or kind, or a thing by what it is called),
`quest NAME new|taken|done|failed|ended`, `walker ...` (the rest asked of the walker), and
`weather` of several names. The short forms (`greets`, `idles`) stay as they were.

In the village: everyone with hands shelters from rain or a storm (`go indoors`, urgent 3,
while it holds); the baker greets the walker with what the stall has (two reactions, one
for a bare stall), and the smith otherwise (a wave, "your loaf is put by"); the
shopkeeper comes in from the square to the walker who waits at her counter with an apple.

**Talks** (`talks.txt`). `talk NAME`, `of WHO`, `when CONDITION`; points (`at POINT`), each
with what is said (`says WORDS`, or `says if CONDITION : WORDS`, the first that holds) and
answers (`answer WORDS`, then `if CONDITION`, `to POINT` or the end, `takes QUEST`,
`gives N GOOD`, `pays N GOOD`, `put FACT`, `clear FACT`). Conditions are asked of the
walker. {step QUEST} and {asks QUEST} in words are the quest's. The act `talk` holds the
first talk of theirs whose `when` holds (`talk.gd`); with none, `quests.gd` speaks as in
7ah. While it is held the walker and they stand turned to each other (`listening`: the
walker's keys ask nothing of the body, their work and reactions wait), the pointer is
free, and a panel at the bottom shows who, what they say and the answers. Answering: 1 to
9, Up/Down (W/S) and E or Enter, a click; a controller's D-pad and A (or Y); Esc or B
leaves; the line under the answers says the keys for the scheme in use (F3). An answer
whose `if` does not hold is not shown; one that cannot be done now (coins or apples not
had, a quest taken already or too late) is shown greyed with why, and choosing it says
so. What is said both ways is written in what happened and kept for the journal.

In the village: the cook (good morning, afternoon or evening by the hour; is there
anything I can do: the quest asked, yes or not now, where to buy bread, if I have no
coins; once taken, what is still to do), the baker (what the stall has, what he needs to
bake), the shopkeeper (apples; an apple given as a gift, once: `pays 1 apple`, `put
gave_apple`).

**Markers and the journal.** Quests (`quests.gd`) put a "!" (Label3D, seen through walls)
over whoever asks a quest that may be taken on now, and a "▼" over the thing the step the
walker is on is done at, said by a new step line `at THING [in PLACE]` (`quests.txt`).
The journal (J while playing, or Journal in the bar; a window like the others) lists each
quest taken on with its steps (✓ done, → the one now with its hint, · to come) and what
was said in talks. Nothing in either is kept: both are made from the quests' facts and
the talks.

**Decisions made here, the user's to overrule**

1. **7ah's decision 3 is replaced**: where a talk is written of someone, a quest is taken
   on by an answer, and may be declined; without a talk, talking takes it on at once as
   before.
2. **A reaction is of whom by its own `of` line**, not by a word in the kind (`reacts
   NAME`): one text says the whole of it, and it is editable in F10 without touching the
   kinds. A kind's own word, and profiles, wait for M4d.
3. **Work interrupted begins its step again**: a `wait` interrupted waits its whole time
   again; nothing is kept part done.
4. **A talk holds the walker still**; walking away does not end it (Esc or B does).
5. **Conditions of a talk are the walker's**, of a reaction the one it is of; `walker ...`
   asks the walker from a reaction.

**How it was checked** (PROGRESS.md has the numbers): `-Tour -Only talk` (new) on a copy
of the showcase, in play; `-Tour -Only work` with the cook's quest now taken by an answer;
the parts beside it. Pictures looked at. **Not tried by hand.**

**Not done**

- Profiles and what is worn (M4d); a window for writing talks, reactions, sequences and
  quests (they are words in F10).
- Reactions between the folk beyond greeting (no conversations between them, nothing
  remembered of the walker beyond facts a talk puts).
- A talk that walks with the walker, or that someone begins (a reaction cannot yet open a
  talk with the walker).
- Voices, portraits, and talk lines over heads in the world: words are in the panel and
  in notices.
- The economy deepened (7ah): unchanged.

**Next**: the user's next milestone, general-purpose physics: built the same day (7aj).

## 7aj. General-purpose physics, authorable: bodies, a lab, a Physics window, and a puzzle of falling blocks (built 2026-10-14, at the user's asking)

The user's brief, after the village: develop reusable, user-authorable physics, collision
and interaction foundations, proved by a playable 3D falling-blocks game and moving-ball
demonstrations built from Draw3D's own authoring; and a standing requirement, for this and
every demo and world after it (kept as a memory): **whatever a demo shows, the user can
create, configure, arrange, inspect, recreate, modify, save and reuse without touching
engine code; where that is not yet so, it is said to be an engine gap.**

**What there was** (inspected first): Godot 4.7 with Jolt as its physics; `loose KG` (a
kind's word: a rigid body, friction 0.7, standing as made until something moving comes
near), colliders from a kind's `solid` lines (box, post, ramp, hull, mesh), one static
body a cell for everything else (`solid.gd`), weighted plates (`pressed`), the walker
pushing what is loose and taking hold of it (an act), and where a loose thing lies kept
on a save (`rests`). Nothing said bounce, grip, damping, a start speed, a hinge, a group,
gravity or speed; nothing could be watched or tuned while it ran; nothing could be put
above the ground.

**Who does what** (unchanged in principle, said here for physics):

| Where | What |
|---|---|
| The host (persistent) | A kind's lines (what a body is made of and how it moves); a placed thing's settings (`raise`); the world's texts (`physics.txt`, `puzzles.txt`); facts (`rests`, `puzzle.NAME.best`). The document is never changed while the world runs: what is tuned then is written into the kind when it stops |
| The editor (authoring) | Library > Physics lab (designs to place); Object text (F6) for kinds; World outline / Inspector for `raise`; What is written (F10) for the texts; the Physics window (F2) while playing |
| Godot (per frame) | Jolt simulates; `loose.gd` makes each body from what its kind says and moves what turns or glides; `puzzle.gd` plays a puzzle |

**Built**

- **Physics words of a kind** (`kind.rs` TRAITS 50; `things.gd`, `loose.gd`): `bounce B`,
  `grip F`, `damp L A`, `pushed X Y Z` (let go at that speed), `fast` (continuous
  collision), `group N` and `misses N` (eight groups; missing is made both ways, since two
  bodies meet if either looks for the other), `pinned X Y Z` (a hinge to the world about
  its own x), and of what is not loose: `kicks S [X Y Z]` (what touches it is sent off),
  `turns AXIS D`, `glides X Y Z S` (moved each tick, carrying and pushing what it meets).
  A collider `solid ball R [x y z]`. Something loose that says any of these is let go
  when the world runs (not stood as made). Something not loose that says one has a body of
  its own, out of its cell's. Each loose body is now made about the middle of its shapes
  (a ball held three metres up turns about the ball, not about its foot), and its thing's
  own frame is `loose.frame(key)`.
- **The world's numbers** (`physics.txt`, `physbook.gd`, F10): gravity (and which way is
  down), speed (slow motion), ticks a second, how far down before something lost is put
  back.
- **`raise`**, a setting of anything put down (`land.rs` Placed; World outline's "raised
  above where it rests", and the Inspector): so many metres over the ground it would rest
  on, kept when it is moved. A ball on a ramp's top, a lamp on a shelf. Worlds written
  before are written the same (checked against the committed host: the village, the town
  and a copy of the user's tile world, byte for byte).
- **The act `kick`** (`acts.txt`, new act word `sends S [UP]`): a ball offers it.
- **The Physics window** (F2 while playing, `physview.gd`): gravity and speed set now and
  kept in physics.txt; the physics held (the walker goes on) and stepped a tick at a time;
  Start again; everything that moves near the walker, with its speed and weight; one
  chosen, ringed, the way it goes drawn; its weight, bounce, grip and damping set now, and
  written into its kind when the world stops; how high it comes up after each bounce,
  against the fall before.
- **Library > Physics lab** (`editor\library\physics.txt`, 22 designs): rubber, bouncy,
  iron and marble balls, a ball held up, a ball sent rolling, a red ball of group 1; a
  gentle and a steep ramp, low and bouncy walls, an ice slab, a red gate (group 2); a
  bumper, a kicker, a spinner, a gliding platform, a pendulum and one let go, a frame for
  pendulums; the falling-blocks frame and the block it is drawn with.
- **The physics lab** (`host\examples\physics.rs`, `editor\showcase.ps1 -Which physics`,
  `editor\showcase-physics.json`): built only of those, by what a person does (designs
  brought in, three kinds of the world's own written `like` the library's, things put
  down, turned and raised). A drop of three balls, two ramps, a mound of the land, a pen
  of balls with a spinner, a sloped table with bumpers and kickers, a row of pendulums, a
  gate, a gliding platform, the puzzle frame.
- **Puzzles of blocks on a grid** (`puzzles.txt`, `puzzlebook.gd`, `puzzle.gd`): a board
  (across, up, deep), cell size, the kind blocks are drawn as, how fast they fall and
  quicken, how long a resting piece waits, what is cleared (`layer`, `row`, or `match N`),
  scores, points for dropping, the next piece (random or a shuffled bag), whether what is
  cleared bursts off as bodies of the physics, colours for blocks of any colour, and the
  pieces block by block. Played at whatever says `plays NAME` (the act `play`). Pieces
  move as the eye looks (left is left on the screen from any side), turn about three axes
  (nudged a cell aside if that frees them), fall faster or drop; the eye goes round a
  quarter at a time; held, over, again, left. Its own eye and a panel of its keys for the
  scheme in use (F3), and on the card (H). The best of each is a fact. Three puzzles are
  written: `blocks3d` (5 by 12 by 5, levels cleared), `blocks2d` (10 by 20, rows), and
  `columns` (columns of three of any colour, three in a line cleared and what is over
  falls): one vocabulary, three games.
- **Controls** (`controls.gd`): the puzzle's own actions with keys and controller buttons
  (`puzzle_*`), a Physics key (F2); all on the card.

**Fully authorable, and how** (each checked by a tour part that does it as a person would)

| Capability | Create | Configure / inspect | Arrange | Save, reuse |
|---|---|---|---|---|
| A ball, any loose thing and what it is made of | Library > Physics lab, Place one | F6 (its kind's lines); Physics window live, written into the kind | the tools (move, turn), `raise` | world file; Library > Save |
| A static surface that bounces or grips, a wall in a group | same | F6 | same | same |
| A bumper, kicker, spinner, gliding platform | same | F6 (`kicks`, `turns`, `glides`) | same | same |
| A hinge (pendulum) | same | F6 (`pinned`) | same | same |
| Collision groups | same | F6 (`group`, `misses`) | | same |
| Gravity, slow motion, steps | physics.txt (F10), Physics window | Physics window | | physics.txt kept with the world |
| A start speed, a kick | F6 (`pushed`), acts.txt (`sends`) | | | |
| A puzzle's rules, board, pieces, scoring | puzzles.txt (F10) | the HUD (points, level, next) | a frame placed anywhere | puzzles.txt kept with the world |
| A puzzle's blocks | F6 (`block-cube`, or any kind the puzzle names) | | | |
| Where a puzzle is played | Library (Falling blocks) or any kind that `plays NAME` | | placed | |

**Limited, or developer-only: engine gaps**

- Physics is said per **kind**, not per thing put down: two balls of different bounce are
  two kinds (`like`, or Library > Customize). The Physics window tunes one body live, and
  writes into its kind.
- **Kinds are text** for physics (F6); no inspector of fields for bounce or grip while
  editing, and no picture of colliders in the view.
- **Joints**: a hinge to the world only (no body to body, no limits, no motor, no spring).
- **Forces**: a start speed, kicks, pushes by the walker; no steady force, thrust or wind
  on bodies; no impulse from scripts that a person writes beyond acts.
- **No raycasts, shapecasts or overlap triggers** a person can use, beyond `pressed`
  plates and kickers.
- **What turns or glides moves in a fixed pattern**, not by facts or events (a door does).
- **A saved game keeps where loose things lie, not how fast they go.**
- **Collision shapes** a person can write: box, post, ramp, hull, ball, and the mesh for
  what does not move; no capsule; no compound shape seen while editing.
- **The cradle is not Newton's**: Jolt passes the swing along the row, and the middle
  pendulums swing too (measured: the last moves 0.42 m, the middle ones up to 0.47 m).
- **The puzzle is a grid game** of one family (pieces that fall; clears of levels, rows or
  matches). Its pieces are logic, not bodies, until they burst; no hold piece, no authored
  nudge tables, no sounds; it is played at a frame, not a world of its own.
- **No gamepad button for the Physics window or the journal** (none is free); keys only.

**Decisions made here, the user's to overrule**

1. **Physics is said by the kind** (with `like` for variants), as everything else a kind
   is; per-thing overrides wait.
2. **A loose thing that says any physical word is let go when the world runs**; old loose
   things (a crate) still stand until something comes by.
3. **The document is not changed while the world runs** (the host's rule, kept): the
   Physics window's "Write into its kind" is done when the world stops; texts may be
   written while it runs, as before.
4. **Speed is Godot's time scale**: the walker and the hour go slower too.
5. **The puzzle is drawn from a kind** (`block-cube`, the library's if the world has none),
   tinted by each piece's colour.
6. **`raise` is a setting of everything put down**, not of kinds.

**How it was checked** (PROGRESS.md has the numbers): host tests, all (50 unit, 59
protocol; p43 knows the Physics lab); worlds written before written the same; new tour
parts `physics` (the lab played and measured, the Physics window, the kick), `puzzle` (all
three puzzles, keyboard and D-pad, clears, bursts, the eye, held, over, the rules and the
block's kind changed, left), `recreate` (an experiment built from nothing by the editor's
own ways, changed, saved, opened again, played again); `-Demo physics`. Pictures looked
at. **Not tried by hand.**

**Next**: per-thing physics settings in the Inspector, colliders drawn while editing, and
body-to-body joints, so that an experiment is arranged by hand rather than written; or the
puzzle family widened (hold, sounds, a world of its own). (Per-thing physics and colliders drawn were built next, with sensors: 7ak.)

## 7ak. Physics made authorable, second tier: one thing's own physics, colliders drawn, and sensors that things go by (built 2026-10-14, at the user's asking)

The user's brief, after 7aj: make physics genuinely authorable. First, **per-instance
settings**: one placed thing may override its design's physics through the Inspector,
the design keeping the defaults, and inheritance and overrides plain to see; saving,
reopening, copying and editing keep them. Second, **colliders visible while editing**:
a thing's actual collider drawn apart from what is drawn of it, with a practical way to
inspect and adjust the supported shapes, and no clutter in normal editing or play.
Third, **reusable physics composition**: what is needed to make different interactions
from reusable objects and properties rather than scripts; the next most valuable
capability, built if it fits. The standing rule of 7aj holds: what a demo shows, a person
can make, change, save and use again by the editor's ways.

**The decision that shapes it** (taken here; the user's to overrule, and surfaced
because getting it wrong would be rework): **one thing's own physics is its own lines,
in the kind's own words, kept as one parameter of the node that put it down** (`own`,
on `placed`, e.g. `bounce 0.95; loose 6; solid box 0 0.22 0 0.45 0.44 0.43`). The host
lays them over what its design says when the thing is made (`kind::own_over`), so Godot
receives one set of traits and one collider as before and nothing in the running world
knows the difference. Rejected: a parameter per physics word (fifteen parameters on every
thing put down, and a second vocabulary beside the kind's); overrides kept by Godot (the
host could not check them, and a save would not hold them); a copy of the design per
variant (what 7aj did, which multiplies designs). Consequences: a thing that says
something of itself is its own content (a different request to the realizer), so a
thousand balls each with its own bounce are a thousand contents, which is right for
experiments and wrong for a forest: **only what is put down by hand has its own physics;
what a maker makes has its design's** (a maker's `each` could carry the same lines later).

**Built**

- **`own`, one thing's lines over its design's** (`kind.rs` `OWN`, `own_said`,
  `own_over`, `physical`; `land.rs` Placed; `shapes.rs` `owned`): the words `loose`,
  `bounce`, `grip`, `damp`, `pushed`, `fast`, `group`, `misses`, `kicks`, `turns`,
  `glides`, `pinned`, `senses`, `when`, `solid`, and `no WORD` to take away what the
  design says. A word said once of a design is replaced; another `misses` is one more;
  its own `solid` lines are what it is to walk into in place of its design's (`solid none`
  nothing, `solid auto` as its design), sized with the thing as the design's are. Anything
  else, or a number that is not one, is refused with the reason (`param` and the plan
  both check; the words a kind says are read as a kind reads them, and the numbers each
  word takes are counted). Kept as written, `; ` between. Something the code makes (a
  rock) may say them too. Worlds written before are written the same (nothing is said of
  a thing that says nothing).
- **The host's `physics` command** (asked of a node, or of a thing by its id, also while
  the world runs): its design's physical lines and collider, its own lines, the two laid
  over each other as it is made, how big its shape is in its design's frame, and the
  shapes that would go round it (`fits`: a ball, a box, a post).
- **The Inspector's Physics part** (`inspect.gd`; first, for a thing put down): weight
  (0: fixed in place), bounce, grip, and damping for what is loose or kicks for what is
  fixed, each beside what its design says (grey), in blue where this one says its own,
  with **design's** to take it back; all its own lines as text (any of the fifteen words:
  `senses`, `when`, `turns` ...); its collider in words, whose it is, and a choice of
  **as its design / a ball, a box or a post round it / nothing to walk into**, a shape of
  its own then sized by its numbers (r; w, h, d; r, h); and **show colliders**. Each set
  is one edit, undone with Ctrl+Z.
- **Colliders drawn** (`colliders.gd`; View > Colliders, the Inspector's button, the
  Physics window's switch): each shape as lines (Godot's own drawing of the shape, its
  edges only), in a colour for what it is: orange still, green loose, yellow fixed with a
  body of its own (bounces, grips, kicks, turns, glides), blue a room that senses or a
  plate, cyan the one picked; faint where something stands in front, except what is still
  (most of the world, which would cover everything); within 26 m of the eye (what is
  still within 14), never the ground. While editing they are the shapes each thing's
  content says, where it stands; while playing they are the bodies of the physics, where
  they are now (a ball's ring rolls with it). Off unless asked for; nothing of it is saved.
- **The Physics window** (`physview.gd`): **Keep for this one only**, beside "Write into
  its kind": the numbers tuned while it runs kept as what this one thing says of itself,
  when the world stops (the host's rule); **Show colliders**; filled at once when opened.
- **Composition: sensors and what goes by them** (`kind.rs`, `things.gd`, `loose.gd`):
  `senses NAME W H D [group N] [counts]`: a room over where a thing stands; the world's
  NAME holds while something loose is in it (of group N, if said), or with `counts` goes
  up by one each time something comes in. And `when NAME` now holds for what kicks, turns
  or glides: it does so only while NAME holds (a door and a lamp went by it already). A
  fact is the world's, so a lamp, a spinner, a gate, a reaction or a quest's condition may
  go by the same one, and none knows of the other.
- **Library > Physics lab, Sensors and signals** (`editor\library\physics.txt`, now 24):
  a **goal frame** (`senses goal 1.7 0.8 1.1 counts`) and a **signal lamp** (`when goal`).
- **The physics lab** (`host\examples\physics.rs`): two stations more. *Own*: four rubber
  balls of one design held 2.5 m up, the second bouncier, the third heavy and dead, the
  fourth given a box round it (by the host's `fits`, as the Inspector does). *Goal*: a ball
  on a steep ramp rolls into a goal; the lamp is lit and the library's spinner, told of
  itself alone `when goal`, starts.
- **`-Demo own`** (seven beats) and tour part **`own`**; host test **p60** and a unit test.

**How the third item stands: what composition needs, and what was chosen**

Interactions in Draw3D are made of three things, all content: **what a thing is made of**
(its kind's lines, now also its own), **facts** (the world's named values, which acts,
plates, reactions, quests and conditions already read and write), and **what goes by a
fact** (`when`). What was missing for physics was the joint between them: a body could
not say anything happened to it, and a moving part could not wait for anything. Of what
could come next, the most valuable that fitted was that joint (a sensor that writes a
fact, motion that reads one), because every later piece plugs into it: a counter is a
fact, a scoreboard is a fact shown, a door already opens by one. What is still needed,
in the order of value:

1. **Joints between two bodies** (a hinge, a slider, a spring, a weld), said between two
   things put down: the cradle, a seesaw, a chain, a trailer. The largest gap.
2. **Fields**: a room that pushes what is in it (`blows X Y Z`), for wind, a conveyor, a
   fan, water's current, and gravity of a place.
3. **Events as facts**: a body says when it is hit hard (`hits NAME [S]`), not only when
   it is in a room.
4. **Speeds and strengths by facts**, not only on and off (a motor's speed from a count).
5. **Per-thing physics for what a maker makes** (`each`), and colliders handled in the
   view (dragged), and a capsule.

**Fully authorable now** (added to 7aj's table)

| Capability | Create | Configure / inspect | Arrange | Save, reuse |
|---|---|---|---|---|
| One thing's own weight, bounce, grip, damping, kicks | Inspector > Physics (or its own lines) | there, beside the design's; Physics window live, kept for one | the tools | world file; copied with Ctrl+D |
| One thing's own collider (ball, box, post, none) | Inspector > Physics, the shape chosen | its numbers; drawn (View > Colliders) | | same |
| Any of the fifteen words of one thing | its own lines (Inspector) | there; refused with the reason if wrong | | same |
| A sensor (a goal, a gate's eye, a counter) | Library > Sensors and signals; or `senses` (F6 or own lines) | its room drawn blue; the fact in Playtest | placed | same |
| Motion, a kick or a lamp that waits for a fact | `when NAME` (F6, or one thing's own lines) | | | same |
| Colliders seen | View > Colliders; Inspector; Physics window | colours by what each is | | not saved (a view) |

**Limited, or developer-only: engine gaps** (7aj's, less what is done)

- Own physics only for **things put down by hand**, not for what a maker makes.
- A collider of its own is chosen and sized by numbers; **not dragged in the view**; no
  capsule; a shape of several parts is written as lines (its own lines take several
  `solid`).
- **Joints between bodies, fields, events, speeds by facts**: not yet (above).
- The Inspector's Physics part shows **five numbers**; the other words are its own lines,
  as text.
- A thing's own lines are **not shown in World outline** (Makers), only in the Inspector.
- 7aj's others stand: speeds not kept in a save, the cradle not Newton's, the puzzle of
  one family, no gamepad button for Physics or the journal.

**Decisions made here, the user's to overrule**

1. **One thing's own physics is lines in the kind's words, one parameter of what put it
   down, laid over its design by the host** (above).
2. **Only things put down by hand** have their own; what a maker makes has its design's.
3. **`when` gates motion and kicks** as it gates doors and lamps: what waits stands where
   it was made (a spinner keeps its angle), and goes on from there.
4. **A sensor counts what comes in after it has settled** (a few ticks): what was in its
   room when the world began is not a goal.
5. **Colliders are off by default**, drawn within 26 m (still ones 14), and still ones are
   not drawn through what is in front.
6. The Inspector shows a thing's physics **before** its maker's parameters.

**How it was checked** (PROGRESS.md has the numbers): host tests, all; worlds written
before written the same; the lab rebuilt and written the same after opening; tour part
`own` (all of it through the Inspector and the Physics window, saved, opened and played
again); `-Demo own -Check`; parts beside it. Pictures looked at. **Not tried by hand.**

**Next physics milestone, recommended: bodies joined, and the world acting on them.**
Joints between two things put down (hinge, slider, spring, weld; picked as two things,
drawn, broken by force), fields (`blows`), hits as facts, speeds by facts. With those, the
cradle, a seesaw, a chain, a fan and a conveyor are made from the library without a
line of engine code.

## 7al. Mechanisms: things joined to one another, forces, what strikes, and speeds by a number (built 2026-10-14, at the user's asking)

The user's brief, after 7ak: reusable physical mechanisms and interactions. Joints
between placed bodies (hinges, sliders, springs, welds); authorable continuous forces
(fans, wind, conveyors); impact events and numeric signals that drive other things,
motor speed among them. Not disconnected features: an order chosen from the
architecture, a working composable subset, and a meaningful construction made with the
editor's own tools, not hardcoded for a demo, inspected, edited, saved, opened and played
again. The distinctions kept: a design's defaults, one thing's own, what a maker makes,
and what happens while the world runs.

**The order, and why.** Signals first, because the rest plug into them: 7ak gave a fact
written by a sensor and motion that waits for one (`when`); what was missing was a
number that says *how much* (`by`) and a body that says it was struck (`hits`). Forces
second, because they are one word on a thing that already has a body or a room (a fan is
a room like a sensor's; a belt is a fixed body that moves what stands on it). Joints
last, because they are the only piece that refers from one thing to another, and so the
only one with an architectural choice (below). All of it is the same kind of line as
7aj's and 7ak's: said by a design (F6) or by one thing over its design (Inspector >
Physics, its own lines), checked by the host, read by `things.gd`, made by `loose.gd`.

**The decision that shapes it** (the user's to overrule): **a joint to another thing is
said by one of the two, in its own lines, naming the other by its name**
(`joins hinge 0 1.2 0 x to upper link`). The host checks that the name is another thing
put down by hand (not itself, not a maker of many), says which thing that is when the
world runs (`joints`), and, when that thing is renamed, rewrites what named it in the same
edit (so Ctrl+Z undoes both). Copied, the copy is joined to the same thing; the thing it
named taken away, the joint is to nothing and says why. Rejected: **a connection in the
maker graph** (a placed node reading another's output: content would then depend on the
other's content and be made again whenever either moved, and a pair joined both ways
would be a cycle); **a joint as a node of its own** (a third sort of object to list,
pick and save, and nothing in the Inspector to make it from; possible later as "joint
things" if many-to-many joints are wanted); **ids kept by Godot** (not stable, not
editable, not in the file). A design may say a joint too, without `to`: then it is to the
world where the thing stands (as `pinned` always was), and one thing may say `to NAME`
over it. Only things put down by hand name another; what a maker makes has its design's
joint, to the world.

**Built**

- **Joints** (`kind.rs` `joint_said`, `joins_to`; `things.gd` `joint`; `loose.gd`
  `_join_all`, `_join`, `_forces`): `joins hinge X Y Z AXIS`, `joins slider AXIS`,
  `joins spring X Y Z K`, `joins weld`, then any of `limits LO HI` (degrees; metres),
  `motor S` (degrees a second; metres a second) and `to NAME`. Said of something loose. A
  hinge is Godot's `HingeJoint3D` (its motor, its limits); a slider and a weld are a
  `Generic6DOFJoint3D` (a slider free or limited along its axis, with a linear motor; a
  weld held on every axis); a spring is a pull applied each step from that place of the
  thing to where it was on what it is joined to (stiffness K, a little damping), so that
  it hangs, sinks and bobs. What it is joined to is a body (loose, or fixed with a body of
  its own: a glider carries it), or, if that is still, the world where it stands; one not
  yet a body (far off) is joined when it is. One joint a thing (a chain is joined link by
  link). Joined again where they lie after a saved game is loaded.
- **Forces**: `blows X Y Z S REACH [W]`, a room REACH long ahead of a thing along its own
  X Y Z, W wide and high, in which what is loose gains S m/s each second that way, whatever
  it weighs (a fan, an updraft; a still thing may blow); `carries X Y Z S`, a fixed body
  whose top carries what stands on it (a belt: Godot's constant surface velocity, so
  friction does the carrying).
- **Signals**: `hits NAME [S]`, each strike at S m/s or more (1.5) adds one to the world's
  NAME (judged by how fast it came, before it bounced; the same two meeting again within
  0.4 s are one strike); `by NAME [MOST]`, what has a speed (a motor, `turns`, `glides`,
  `blows`, `carries`) goes at that speed times the world's NAME, at most MOST (5), and
  stands while NAME does not hold. `when NAME` now also gates `blows`, `carries` and a
  joint's motor. A count from `senses … counts` is a number too.
- **The Inspector's Physics part** (`inspect.gd` `_fill_joint`, `set_joint`,
  `joint_line`): for what is loose, **joined: …** in words (its own in blue, or its
  design's), the sort (as its design / not joined / a hinge / a slider / a spring /
  welded), **to the world or to a thing near it** (the host's `physics` lists those put
  down within 12 m, nearest first, with how far), the axis, where it holds (x y z), a motor,
  a spring's stiffness. Each is one edit of its own lines. Under its own lines, **its
  design also says: …** for the words without a row (`blows`, `hits`, `when`…).
- **Colliders drawn** (`colliders.gd`): rooms that blow in magenta; joints in white, a
  cross where each holds, its axis, and a line to what it holds to (a spring's line
  stretches), while editing (the host says what each name is) and while playing (where
  the bodies are).
- **Library > Physics lab > Mechanisms** (`physics.txt`, 35 designs now): conveyor belt,
  parcel, chime tray (`hits ring 1.2`, and `carries` what lands in it off its side),
  fan, updraft pad, paddle wheel (a hinge with a motor), wheel stand, swinging link,
  light crate, sliding gate (a slider with limits and a motor), weight on a spring.
- **The lab's bell machine** (`host\examples\physics.rs`, `editor\showcase-physics.json`,
  east of the goal): three parcels on a belt drop into the chime; the paddle wheel says
  `by ring 4`, the gate, the fan and a lamp `when ring`; two crates, one `joins weld to
  front crate`; a chain of two links under a frame (the lower `joins hinge 0 1.2 0 x to
  upper link`, the upper `pushed 0 0 -3`); a weight on a spring; an updraft with a light
  ball over it. Every piece a library design; every connection a line of one thing,
  sent as a person sends it.
- **`-Demo mech`** (seven beats, the newest), tour part **`mech`** (built from nothing in
  a new world through Library, Makers and the Inspector), host test **p61** and a unit test.

**Found and mended on the way**: a fan pushed nothing (`apply_central_force` before the
step was lost here; the speed is now added each step directly, as the brief says it: m/s
each second); a hinge motor given a very large torque made the joint give and the wheel
fall from its pin (now as much as such a thing needs); a chime with a floor counted only
the first parcel, the next landing on it (the tray now carries what lands off its side,
two of the new words together); `hits 3` was taken as a name (a fact's name begins with a
letter); a joint written by the Inspector said `0.0` for `0`.

**Fully authorable now** (added to 7ak's table)

| Capability | Create | Configure / inspect | Arrange | Save, reuse |
|---|---|---|---|---|
| A hinge, slider, spring or weld to the world | a design's lines (F6), or Library > Mechanisms | Inspector > Physics > joined; drawn white | placed, raised, turned | world file; library |
| A joint to another thing put down | Inspector > joined > to 'NAME' (or its own lines) | there; drawn; renamed, it follows | placed | world file; copied with Ctrl+D |
| A motor, its limits | the joint's `motor`, `limits` (Inspector: motor; limits in its lines) | Inspector | | same |
| A fan, an updraft, wind in a room | `blows` (Library: fan, updraft) | its room drawn magenta | placed, turned | same |
| A belt | `carries` (Library: conveyor belt) | | placed, turned | same |
| A count of strikes | `hits NAME [S]` (Library: chime tray) | Playtest shows the fact | | same |
| A speed by a number | `by NAME [MOST]` on what moves | its own lines | | same |

**Limited, or developer-only: engine gaps**

- **One joint a thing**; a plank joined at both ends (a bridge) is not yet; no joint
  that breaks under force; motor strength and spring damping are the engine's, not said;
  hinge and slider limits are lines, not Inspector numbers.
- **The thing joined to is chosen from a list** (things put down within 12 m), **not
  clicked in the view**, and a joint's place and axis are numbers, not handles.
- **What a maker makes** cannot be joined to, nor name another (its design's joint is to
  the world).
- **`blows` moves what is loose, not the walker or the folk**, has no fall-off, and is
  not the world's wind (`weather.txt` `air`), which it should one day be fed by.
- **`carries` is of fixed things**; whether it carries the walker was not tried.
- **Numbers come only from counts** (`senses … counts`, `hits`): no thresholds
  (`when ring 3`), no timers, no resets but by a quest or a loaded game; the strength
  of a strike is not kept.
- **Nothing of a mechanism changes while the world runs** (the document is the host's,
  and refused while running, as in 7aj).
- 7ak's others stand: own physics only for things put down by hand; colliders sized by
  numbers; not in World outline.

**Decisions made here, the user's to overrule**

1. **A joint to another thing is one line of one of the two, naming the other**; a
   design's joint is to the world (above).
2. **One joint a thing** (`joins` is said once; a thing's own replaces its design's).
3. **Forces are speeds gained each second, whatever a thing weighs** (`blows`), as the
   wind and gravity are; a belt is a surface that moves, friction does the rest.
4. **`by NAME` is a factor, and nothing while NAME does not hold**; `when` stays whether
   at all; both may be said; what is driven by nothing brakes (a motor at 0 holds).
5. **`hits` counts strikes**, judged by the speed it came at, one count per pair per
   0.4 s; it does not keep how hard.
6. **A spring pulls toward where it was put down** (no rest length of its own), from that
   place on it to the same place on what it is joined to.

**How it was checked** (PROGRESS.md has the numbers): host tests, all (p61 new); worlds
written before written the same; the lab rebuilt; tour part `mech` (a machine built from
nothing in a new world through Library, Makers and the Inspector, joints chosen in the
Inspector, refused when wrong, renamed and followed, drawn, played and measured, saved,
opened in another world and played again: the same); `-Demo mech -Check`, also in the
built game given the lab as its world; parts beside it (`recreate`'s fall now timed in
physics steps, not by the clock; `kinds` intermittent before this, as it was). Pictures
looked at. **Not tried by hand.**

**Next, recommended: signals you can reason with, and mechanisms handled in the view.**
Thresholds and comparisons on numbers (`when ring 3`, `when ring < 3`), a timer and a
reset as words, the strength of a strike kept as a number; a Signals view listing each
fact with what writes it and what reads it (the wiring of a machine, inspected in one
place); and a joint's other thing clicked in the view, its place and axis dragged as
handles. Then several joints a thing (bridges, ropes), joints that break, the world's
wind through `blows`, and physics for what a maker makes.

## 7am. Signals you can reason with: conditions, timers, resets, a window for the wiring, and joints picked in the view (built 2026-10-14, at the user's asking)

The user's brief, after 7al: expressive, inspectable signals for reusable gameplay and
mechanisms. Extend the world's numbers and `when` into a small, composable way of
authoring: objects that respond to numeric conditions (a counter reaching a target)
without scripts for a demo; delayed actions and machines that reset and can be used
again; an editor view of which objects write and read signals, their conditions, and
enough to diagnose what is disconnected or wrong; and an assessment of choosing a
joint's other thing in the scene and editing its anchor and axis visually, built in the
same increment only if it fits. Simple, consistent semantics that extend what is there,
not a second logic system; a meaningful machine made with the editor's own tools,
inspected, edited, saved, opened and run again without logic made for the demo.

**What was there, and what was missing.** Facts are the world's named values (7ak):
sensors, plates, strikes (`hits`) and acts write them; doors, lamps, moving parts, fans,
belts and motors go by one (`when NAME`, whether it holds) or by its number (`by NAME`).
Missing: *how much* as a condition (a count reaching three), *when* (a moment after, a
while later), a way back (a count taken back, so the machine can run again), and any
view of the whole: a fact's writers and readers were only findable by reading every
thing's lines.

**The decision that shapes it** (the user's to overrule): **everything new is said in
the one place things already say what they go by: `when`, and two more words of the
same kind.** `when` takes a condition, `[not] NAME [OP] [N] [for S]`; `clears` and
`ticks` are things a thing does *while its `when` holds*, as kicks, turns, glides, fans,
belts and motors already were. One reader (`state.gd` `cond`, `holds`) answers every
`when` in the editor and the game, and the host checks the same grammar
(`kind::when_said`). Rejected: **a logic node** (a gate or a timer as a new sort of
object between things: a second system beside facts, with its own wires to list and
save); **expressions** (`when ring >= 3 and not open`: a language to parse, explain and
debug, for what two things in a row already do: a post that `clears` or `ticks` by one
condition writes a fact another goes by); **timers as facts the host keeps** (the host
does not step the world; Godot does, and it already decides hold and step).

**Built**

- **Conditions** (`kind.rs` `when_said`, `COMPARE`; `state.gd` `cond`, `holds`, `number`):
  `when NAME` as before (it holds at all); `when NAME N` (its number is at least N);
  `when NAME >= <= > < = N`; `when not …` (the other way); `… for S` (only once that has
  held S seconds of the world running). A fact that is not there is 0; one that holds
  and is not a number is 1. Every `when` is watched once asked: what stands open or
  lights by one hears when it changes (`state.held`), what moves asks it each step. The
  conditions of texts take the same (`cond.gd`: `fact ring 3`, `fact not open`).
- **Timers and resets** (`loose.gd` `_timers`): `clears NAME [NAME…]` takes those facts
  back while its `when` holds (a thing that says no `when` clears nothing, and the
  Signals window says so); `ticks NAME S` adds one to NAME every S seconds while its
  `when` holds (always, without one). The world's time for `for` and `ticks` is the
  physics' own (`state.now`, stepped by `loose.gd`): held still, nothing waits; slowed,
  it waits longer in the world's seconds. It begins at 0 each time the world runs.
- **The host checks them** (design lines and one thing's own): a condition said wrongly
  (`when ring 3 soon`, `when ring >`, `ticks t 0`, `clears 3`) is refused with the form,
  and a design's wrong line is said with its number. `clears` and `ticks` may be said of
  one thing (`OWN`), and taken away (`no clears`). Worlds written before read the same.
- **The host's `signals`** (`lib.rs`; read, also while the world runs): every thing put
  down by hand, and each design a maker makes, whose lines as made say a word of signals
  (senses, hits, pressed, takes, clears, ticks, when, by, opens, lights, kicks, turns,
  glides, blows, carries, joins): its lines, which are its own, how many a maker makes,
  where the first stands; and every joint by name. Read whole up to sixty thousand
  things, and said when not.
- **Window > Signals** (Ctrl+K; `signalview.gd`): each fact, folded open to what writes
  it (orange) and what goes by it (blue), each line and whether it is its own; the texts'
  `put`, `clear` and `fact` lines too; while the world runs, each fact's number and
  whether each condition holds (and how long one that waits has). **What is wrong**,
  first: a fact gone by that nothing writes, with a written name like it ("did you mean
  'batch'?"); a count asked for that what writes it never reaches (a sensor that only
  holds, asked for 3); something that goes by a fact and does nothing by it; `by` on what
  has no speed; `clears` with no `when`; a joint to nothing. **Draw the wiring**: lines in
  the view from each writer of the chosen fact to each reader. **Pick it**: the chosen
  row's thing picked and shown in the Inspector, where its lines are changed. It edits
  nothing itself: a person changes a line where it lives (Inspector, F6, a text).
- **The Inspector** (`inspect.gd`): a line **signals: writes …  ·  goes by …** for the
  thing picked, and **Signals…** to open the window on it.
- **Joints picked in the view** (`main.gd` `pick_for`, `inspect.gd` `join_to_picked`,
  `joint_at_picked`): beside "to …", **pick…**: the next left click on a thing put down
  chooses what it is joined to (itself, a maker's thing or nothing is said and changes
  nothing); beside "at", **click on it…**: the next click on this thing sets where its
  joint holds, in its own frame. Esc, or another tool, and nothing is asked.
- **Library > Physics lab > Sensors and signals** (38 designs): **swinging gate** (opens
  and shuts by `when`, default `when open`), **timer post** (`ticks waited 1` while `when
  ring`), **reset post** (`clears ring` `when ring 3 for 6`).
- **The batcher** (`host\examples\physics.rs`, rebuilt `editor\showcase-physics.json`, at
  20, -15): two belts feed six parcels to a chime tray; each piece a library design and
  each connection one line of one thing. The chime counts as `batch`, its own name apart
  from the bell machine's `ring` (`hits batch 2.5; carries -1 0 0 3`); the belts run
  `when batch < 3` (and slower than their design, `carries 0 0 -1 1.2`); three lamps
  `when batch 1`, `2`, `3`; the gate `when batch 3 for 1`; the timer post `when batch 3`
  (its design's `ticks waited 1`); the reset post `when waited 6; clears batch waited`.
  It runs in rounds: three parcels, the belts stop, the gate opens a second later, six
  seconds on the lamps go out, the gate shuts, the belts run, three more.
- **`-Demo signals`** (seven beats, the newest), tour part **`signals`** (the batcher built
  from nothing in a new world), host test **p62** and a unit test.

**Found and mended on the way**: a belt started again by a fact moved nothing that had
come to rest on it (a body asleep does not feel the surface under it; `loose.gd` now
wakes what is on a belt when it starts or goes faster); the batcher counted past three
(parcels already falling when the belts stopped; a bounce counted twice at the design's
1.2 m/s: its own `hits batch 2.5`, slower belts); in the tour's world the chime's
counted parcels piled up at its open side, and the next ones came to rest on the pile
and were not counted (its own faster `carries` throws them clear); the window's rows
were cut off by an empty column while editing; things called the same could not be told
apart (now by where they stand); words of the texts' prose were taken for facts (`put`
and `clear` are read only at a line's start).

**How these meet what was there** (asked in the brief)

| With | How |
|---|---|
| `when` | Extended, not replaced: `when NAME` means what it meant; the rest is new. It gates all it gated (doors, lamps, kicks, turns, glides, blows, carries, a motor) and now `clears` and `ticks` |
| `hits`, sensors, plates | Writers, unchanged. A count (`hits`, `senses … counts`, `takes`, `ticks`) can be compared; one that only holds (a plate, a sensor without `counts`) is 1 at most, and the window says when a condition asks for more |
| `by` | Unchanged; a count taken back by `clears` stops what goes by it (nothing while it does not hold) |
| One thing's own lines | `when`, `clears`, `ticks` are its own over its design's (`no clears`); shown in the Inspector's signals line and as "(own)" in the window |
| Joints | A motor still goes by `when`, now any condition; the other thing and the anchor are clicked in the view |
| Save and reopen | The lines are the world's (its file); the batcher was saved, opened in another world and run again the same. What a `for` or a `ticks` has waited is not kept in a saved game: loaded, it waits anew (the facts themselves are kept) |
| Undo | Each line set is one edit (Ctrl+Z undid a wrong `when` in the tour) |
| Copying | A copy says the same (a lamp copied went by `batch 3`, and was listed) |
| What a maker makes | Its design's lines, the same for all; listed once with how many ("posts: 30 post"); nothing of its own |
| The built game | The same scripts read the same lines: the batcher ran its two rounds in the exported game |

**Fully authorable now** (added to 7al's table)

| Capability | Create | Configure / inspect | Arrange | Save, reuse |
|---|---|---|---|---|
| A threshold or comparison | `when NAME N`, `when NAME < N`… (F6, or one thing's own lines) | Inspector's signals line; Signals window: holds or not, playing | | world file; library designs |
| Something done a while after | `when … for S` | the window says how long it has held | | same |
| A reset | `clears NAME…` (Library: reset post) | the window lists it as a writer | placed | same |
| A timer | `ticks NAME S` (Library: timer post) | its number in the window, playing | placed | same |
| A gate that opens and shuts by a fact | Library: swinging gate, its own `when` | | placed, turned | same |
| The wiring of a machine | Window > Signals | what writes, what reads, what is wrong; drawn in the view | | not saved (a view) |
| A joint's other thing and anchor | Inspector > joined > pick… / click on it… | the Inspector; drawn (Colliders) | | world file |

**Limited, or developer-only: engine gaps**

- **One condition a thing**: no `and`/`or`; two conditions are two things in a row (a
  post that clears or ticks by one, another that goes by what it writes).
- **What waits is not kept in a saved game** (`for` and `ticks` begin again when loaded).
- **Signals run only near the walker**, where things have bodies (as sensors always
  did): a machine far off neither counts nor waits.
- **A strike's strength is not kept** (the 7al brief's third item): `hits` counts.
- **A joint's axis is chosen as x, y or z of its own**, not dragged; no handles for the
  anchor (it is clicked or typed); limits are lines.
- **The Signals window does not edit**: a line is changed where it lives; it does not
  show what an act writes about one thing (`open` of a door) or a quest's own facts.
- **A slider's or hinge's motor does not go back when its `when` stops holding** (7al:
  it brakes): a gate that opens and shuts by a fact is the swinging gate, not the slider.
- **Counting what falls is physical**: a tray counts strikes, and where what it counted
  goes matters (the batcher's own lines throw it clear).
- 7al's others stand: one joint a thing, no joints that break, `blows` not the world's
  wind, own physics only for what is put down by hand.

**Decisions made here, the user's to overrule**

1. **A condition is said in `when`, one a thing**, and `clears` and `ticks` are done while
   it holds (above); no expressions, no logic objects.
2. **A fact not there is 0; one that holds and is not a number is 1**; `when NAME` alone
   still means "holds at all", so `when ring` and `when ring 1` differ only for a fact put
   to 0.
3. **`for S` and `ticks` count the world's running time** (physics steps: held, they
   wait; slowed, they wait longer), from 0 each time the world runs, not kept in a saved
   game.
4. **`clears` takes back while its `when` holds** (not once when it comes to hold);
   without a `when` it does nothing, and that is said as wrong.
5. **The Signals window reads and does not edit**, and reads from the world each time (no
   second copy of the wiring to keep in step); its "wrong" is advice, never a refusal.
6. **Joint editing in the view is two clicks** (what it is joined to; where it holds);
   the axis stays a choice of three. Dragged handles were assessed (below) and left.

**The assessment of joint editing** (item 4 of the brief). Picking the other thing in the
view and placing the anchor by a click fit cleanly: the view already answers "what is
under this point" (`pick_at`, the same ray as picking), the Inspector already writes the
`joins` line, and the host already checks the name; what was added is one "the next
click is for…" mode (`pick_for`, ended by Esc or another tool, as every tool is). Dragged
handles for the anchor and the axis would need a gizmo of their own (`handles.gd` drags a
mesh's corners, not a point and a direction of a thing), a way to show an axis that is
not x, y or z, and a change of `joins` to say one; that is a piece of its own, and was
left so as not to take from the signals.

**How it was checked** (PROGRESS.md has the numbers): host tests, all (p62 new, a unit
test new); the lab rebuilt and written the same after opening; tour part `signals` (the
batcher built in a new world through the Library, Makers and the Inspector; a wrong
condition refused; the Signals window read, put wrong two ways and right, undone,
copied, drawn, a thing picked from it; a joint's other thing and anchor clicked in the
view; two rounds played and measured; saved, opened in another world, played again: the
same); `-Demo signals -Check`, also in the built game given the lab as its world; parts
beside it; `kinds` compared with the code of 5b079b5 (PROGRESS.md). Pictures looked at.
**Not tried by hand.**

**After it, the same day (the user's decision):** a controller is heard only while the
window is in front. An Xbox One controller on the user's machine had sent Y ("act") to the
editor behind other windows, beginning acts in scripted runs (the `kinds` part's "petted
counted 2"). Behind, the actions' controller bindings answer only a script's pretend
controller (device 99), what was held is let go, and other controller events are dropped
before anything reads them (`controls.gd` `hear_pads`, `unheard`; `main.gd` `_input`).

**Next, recommended: several joints a thing, and what waits kept in a saved game.**
Joints as many as a thing needs (a plank held at both ends, a rope of links, a bridge),
joints that break under force, and a timer's progress and a `for`'s wait kept with the
facts when a game is saved; then the strength of a strike as a number, the world's wind
through `blows`, and physics (and signals of their own) for what a maker makes.

## 7an. Signals, continued: how hard a thing was struck, and a row that leads to its rule (built 2026-10-14, at the user's asking)

The user's brief, after 7am: carry on the open signals work from the committed state,
extending the world's numbers and `when`, not a second mechanism. Numeric thresholds,
useful timers and delays, reusable resets, **and impact-strength values**, joined to what
is there; a Signals view of writers, readers, conditions, live values and what is wrong,
**an entry of which leads to the object or rule in the editor**; a substantial small
mechanism made with the editor; verified through editing, play, save and open, undo,
copying, references, generated things, overrides and the exported game. The Signals view
kept simple (no node graph yet); draggable joint anchors and axes deferred.

**What was there already (7am), and so not built again:** conditions (`when [not] NAME
[OP] [N] [for S]`), `clears` and `ticks`, the Signals window with values while playing and
what is wrong, a thing's row picked from it, the batcher. Two things of the brief were not
there: the strength of a strike as a number, and a row of a text (a sequence, a talk, a
quest) leading anywhere.

**Decided (open to the user's word):**

1. **How hard is a fact, said by `feels NAME [S]`.** Each strike at S m/s or more (0.5 if
   not said) puts the world's NAME to the speed it came at, to a tenth: the strength of
   the last blow. It is a word of its own beside `hits` (which counts), not a third
   number of `hits`: a thing may count, feel, or both, under names of its own, and
   everything that goes by a number goes by it as it is (`when blow 6`, `by blow`), and
   `clears blow` takes it back. One room about the thing feels for both (`loose.gd`).
   Not built: the hardest strike kept (a thing can keep it by a condition: a lamp `when
   blow 9` and nothing that clears it before it is seen); what struck it; strength as
   force or energy rather than speed.
2. **A text's row leads to its line.** In the Signals window, a row that a text says (a
   sequence's `put` or `fact`, a talk's, a quest's) opens the texts' window (F10) on that
   text at that line, chosen and scrolled to; the button reads **Open the text** for such
   a row, **Pick it** for a thing's. The texts' window now scrolls to a line it is opened
   at once it is laid out (it had moved its caret and left the view at the top, for
   Who can do what too).
3. **The mechanism:** a strength tester, beside the batcher in the lab and built again by
   the tour in a new world: a strength pad (new library design, `feels blow 1`), three
   signal lamps of their own lines `when blow 3`, `6`, `9`, a reset post of its own
   lines `when blow for 4; clears blow`, and a ball. Nothing of it is made for the demo:
   every part is a library design and every connection a line in the Inspector.

**Not done, deferred as the brief says:** joint anchors and axes dragged in the view
(they are clicked and typed, 7am). A node graph of the wiring.

**Checked** (the details in `PROGRESS.md`, "Signals, continued"): host tests all (53 unit,
62 protocol, p62 grown); `-Tour -Only signals` (a strength tester built from the Library
in a new world, struck at 5 and 10 m/s: felt 4.2 and 9.2, lamps 3 and then all three, reset
3.9 s after; saved, opened, the same; a sequence's row opened at its line, undone);
`-Demo signals -Check` 8 of 8; physics, mech, own, loose, hub, ui; `kinds` 3 of 3; the
built game's check, and the lab in the built game (the same strikes). Not tried by hand.

**Next, recommended, unchanged:** several joints a thing, and what waits kept in a saved
game; then joint anchors and axes dragged in the view.

## 7ao. Plants grown from numbers, and colour varied as one system (built 2026-10-15, at the user's asking)

The user asked for the procedural nature tools expanded on the tree generator's own
way of working (grass, flowers and plants, ferns, crops and fields; then berry bushes,
fruit-bearing plants and a broader variety of growth forms), and for one shared colour
variation that serves them and other designs. Rocks and boulders and modular
construction were not to be resumed. Built on branch `m2-stable-terrain`.

### What is built

**Six families beside the tree** (`shapes/grown.rs`, grown in `shapes/grown/plants.rs`):
each is an entry of the one table with its numbers under headings, its presets, and
an arm of `grow`. Nothing else in the host or the editor learned what a plant is: a
grown plant is a kind with a mesh, placed, painted, grouped, given to a wood, kept in
the library, grown again under its name with every thing of it following, and refused
over a mesh shaped by hand, exactly as a tree is.

| Family | Forms (code that grows differently) | Presets |
|---|---|---|
| Grass | tuft, fountain (arching, plumes), reeds (stiff, cattails), seeding (stalks with loose heads); a design is one tuft or a patch of many, clumped or even | lawn, meadow grass, tussock, fountain grass, reeds |
| Flower | daisy (rays round a raised eye), tulip (cup), spike (bells up a stem), globe (ball of star florets), sunflower (great disk facing out, leaves up a thick stem), poppy (four cupped petals, dark eye); side shoots | daisy, tulip, foxglove, lupin, allium, sunflower, poppy, cosmos |
| Fern | spreading, shuttlecock, tree fern (on a trunk), bracken (stalk then a level blade); outline lance/triangle/oblong; fiddleheads | lady fern, ostrich fern, tree fern, bracken |
| Crop | a patch of a field in rows: wheat (ears that fill, nod and go gold), maize (stalk, arching leaves, tassel, cobs), cabbage (leaves about a heart), lavender (grey mound, purple spikes); `ripe` 0 to 1 is how far grown (shorter and green when young), gaps, unevenness, a lean the wind gave | wheat ripe, wheat young, maize, cabbages, lavender |
| Berry bush | rounded, upright, arching canes, low mat; twigs, leaves or clumps; berries borne as bunches, strings, blackberries, hips or singly; some unripe, some clusters still in flower | blueberry, redcurrant, bramble, wild rose, cranberry, plain shrub |
| Fruiting plant | strawberry (rosette of trefoils, runners, cone berries with calyx), tomato (stem up a stake, trusses), pumpkin (vine along the ground, lobed leaves, ribbed fruit), grape vine (trunk to wires between posts, arms, bunches), fruit shrub (short trunk, round crown, fruit at its face); a stake or a trellis | strawberry, tomato, pumpkin, grape vine, orange shrub, lemon shrub |

Colours are the style's by name as a tree's are: `leaf1..`, `green1..`, `bloom1..`, a
new list **`fruit1..`** (style word `fruits`: by default strawberry red, blueberry,
blackberry, orange, lemon, apple green, tomato red, grape purple), `bark`, `wood`; and
rrggbb only where a style has no word (straw, a daisy's gold eye, a cattail).

**Colour variation, one system** (`shapes/vary.rs`). A variation is said by role:
foliage, bloom, fruit, wood, walls, roof, stone, own (rrggbb colours), all:

    bloom 45% among bloom1 bloom2 bloom4 bloom5 from bloom3, foliage light 0.08

- *N%*: how many things vary in that role; the rest keep the design's palette.
- *hue H*, *light L*: round the wheel and lighter or darker, by up to that much.
- *toward C A*: toward a colour (an autumn few, riper wheat).
- *among C ...*: some have one of these of the style's colours in place of the role's
  main colour (*from*); the rest of the role stays (a flower's petals change, its leaves
  do not). Without *from*, it is the one the design's own variation names, else the
  role's first: so a group's "bloom among bloom4" turns the daisies' white and the
  tulips' red alike.
- *seed N*: another mix of the same.

Who says it, role by role, each over the one before: the **design** (its kind's
`vary` lines: the Generator writes the family's at first, from its numbers), the
**group** it was painted with (after `|` in the group's line), the **maker** (its
`vary`), and over all **one thing's own**, kept by the maker by the thing's key
(`palettes`). Each thing draws once by a seed of its maker's node and its key, so
saving, opening, a maker made again or the design grown again draw the same; draws
come in four steps a role, so a thousand things of a design are a few meshes. What a
thing drew (`palette` in its brief) is a look: the style's palette with its roles
turned, given to the realizer; so it reaches every kind (the library's houses and
roofs too, by `walls`/`roof`) and, through a maker's variation, the code's own grass,
flowers, bushes, trees and rocks.

**Kept when grown again.** A design's `vary` lines are the person's: growing again
keeps them, but for lines that are exactly what the generator wrote last time, which
are written again for the new numbers (a daisy grown pink varies around pink).

**The editor.** The Generator has a button for each family (title "Tree Generator" or
"Plant Generator: Flower"), the pictures of that family, and under its numbers the
shared control **colours vary** (`varybox.gd`: the words, a role with its share, hue
and light, and the style's colours to click), **show a mix** (five side by side as
painted), **Set it on 'NAME'**, **Another mix**. The same control is the group's in
Objects and the maker's in World outline (Colour variation, Another mix, As its
designs say). One thing picked: its colours in words, Another colour, Its design's own
colours, Keep these colours / Back to the mix, and its bloom or fruit chosen from
swatches. Host commands `colours` (what each says, what one thing drew, the style's
swatches) and `vary` (set on a design, a group, a maker, or one thing: a pick, "base",
"" back to the mix, or `again`). `generate` without a name returns the family's
variation and, with `mix`, that many meshes as painted things would draw.

**The showcase.** `editor\showcase-nature.json` (`editor\showcase.ps1 -Which nature`,
`host\examples\nature.rs`): 32 designs grown, 4 groups with variations, a wood (oaks
and birches whose leaves vary), a woodland floor, a wildflower meadow, a berry hedge,
fields of patches (wheat with riper patches, young wheat, maize, cabbages, lavender of
three colours), a kitchen garden (strawberries, tomatoes, pumpkins, vines, citrus), a
cottage border, sunflowers (one red, kept by hand), a lawn: 1702 things. Everything in
it is made by commands the editor sends, and can be picked, changed and grown again.

### Decisions made here, the user's to overrule

1. **A variation is a role's, over the style's palette by name**, not a colour per
   face: it keeps the authored palette and turns with the style.
2. **One thing's own colours are kept by its maker, by key** (as every change by hand
   is), and never redrawn by anything said above them.
3. **Draws come in steps** (four per role, and the `among` colours): fewer meshes,
   less variety than a free jitter.
4. **A group's variation lives in the group's line**, over each design's, under a
   maker's.
5. **A design's variation written by the generator follows its numbers until the
   person changes it**; then it is theirs.
6. **Fields are patches put down side by side** in a maker that makes nothing by
   itself; `ripe` is a design's number. Growth over time, harvest and a field that
   fills an outline are not built.
7. The Tree Generator keeps its name and entry; the same window is the Plant Generator
   by a family button and a new Plants entry (no rename before the inventory review).

### Limits and known gaps

- A role a design does not use still makes its draws differ: grass in a meadow group
  whose variation says `bloom` is a few meshes more than it needs. Not wrong; wasteful.
- Variation between parts of one thing is the generator's (a second colour, mixed; a
  light that differs clump to clump; a clump of two flower colours), not the shared
  system's. The shared system turns whole roles of a thing.
- The code's houses of a street are not varied (the house generator's leaves do not
  take a palette); kinds that are houses are.
- Heavy for painting by thousands: a blueberry bush about 10,000 triangles, lavender and
  wheat patches 8,000 to 10,000. A wood of many is fine; a hedge of hundreds is not
  measured.
- Plants give to the wind by `gives` as trees do; no wave goes over a field.
- Looked at and judged by me in pictures: a tulip reads as a closed cup and a bit flat,
  a bramble seen from above is sparse, a tree fern from afar is like a small palm.
- Not tried by hand.

### How it was checked

- Unit: `shapes::vary::tests` (3: read and written back, a share drawn about so often
  and the same every time, a pick turns its role and nothing else); `shapes::grown::tests`
  (2 new: every preset of every family grows, within bounds, the same twice and another
  with another seed, silhouettes told apart; berries the style's fruit, wheat gold as it
  ripens and lower when young).
- Protocol `p63`: a daisy design's variation; painted, about so many varied; written and
  opened the same; grown again pink, the generator's line written again and the same
  things varied; the person's line kept; a maker's over the design's; one thing's own
  kept, again, base, back, undone, written and opened; a group over two designs, each
  its own `from`; the code's flowers varied by a maker; each family made and placed.
- The whole host suite twice: 58 unit, 63 protocol.
- Tour `-Only plants` (new: every preset of six families pictured, the mix, the window's
  controls clicked, a design made and set, painted, one thing in World outline, written
  and opened): 3 of 3 runs passed after its waits were mended. `-Only nature` (new):
  passed. `-Demo nature -Check`: 5 of 5.
- Beside it: ui, grow, workflow, brushmix, makers, designs, showcase passed. `library`
  fails "a design clicked in the library was not brought into the world" with the
  committed scripts and host too (built from `git archive HEAD`): not of this pass.

## 7ap. Plants walked through, bamboo and stranger forms, and what grows drawn lighter from farther off (built 2026-10-15, after the user's first look at 7ao)

The user, after 7ao: "the plant generator shows trees instead. when character walks
through fields there is no collision and movement. add bamboo and some generic non
standard tree and flower types." With it, the working defaults of the same day: the
role-based colour variation stands; next, make painting vegetation efficient at large
scale, then spatial authoring such as a field filling an outline; growth over time
and harvest are a later gameplay layer. The rest of the user's defaults of that day
reconciled `PROGRESS.md`'s "Now" (see there).

### What was found and mended

- **The Plant Generator showed trees.** The editor runs `editor\host\target\release`,
  which had not been built since before 7ao; its host knew only the tree, and the
  Generator fell back to it without a word (7ao's checks ran on a second build,
  `target-w`). Mended twice over: `run.ps1` builds the host again first whenever a
  source of it is newer than the program (and says when it cannot, an editor holding
  it open); and the Generator, asked for a family its host does not grow, says the
  host is older than the editor instead of showing what it has.

### What was built

**Walked through** (one rule for every kind, not a plant's own):

- A kind's `solid` may say **`wade x y z w h d M`**: a box gone through, not walked
  into; whoever is in it goes M slower (0 to 0.9). The generator writes it for what
  grows low and thick, about what of it is at a body's height, more the taller and the
  thicker the family grows (a field, tall grass, ferns, a low thicket, bamboo); a lawn
  and a strawberry are stepped over; a trunk, a bush's stem, a bamboo cane stays a
  `post` or a `box` beside it. Godot keeps the wade boxes in an area of each cell's on
  a layer of their own (`solid.gd` `WADE`); the motor asks how thick what it touches is
  (`thick_at`, a ball of 25 cm, so the hand's width between a field's patches is not
  open ground) and its plain ways on the ground are that much slower (`motor.gd`
  `wading`). Every body: the walker, the village's people, animals.
- **What gives bends away from whoever goes through it**, in the one shader that
  already moves it in the wind (`world_view.gd` `treads`): the walker, the four others
  nearest, and the walker's last few places, rising again over three and a half
  seconds. Only while playing: editing, and the Generator's booth (drawn with the same
  material), stand as made.

**More to grow** (each in the one generator, as 7ao's are):

- **Bamboo**, a family of its own: canes ringed at their joints, bowing under fans of
  narrow leaves on branches from so far up; a clump, a grove or a row; canes the
  style's green, golden, black or pale. Presets: green, giant, golden, black, dwarf
  bamboo, a bamboo screen. Thick canes are solid (a box each), between them waded.
- **Trees, six presets on two new numbers**: the trunk's `swell` (a bottle) and
  foliage `tufts` (stiff straps at each tip), and blossom (the tree's leaf colours may
  be the style's blooms): baobab, dragon tree, Joshua tree, flat-topped acacia, cherry
  blossom, twisted tree. Old trees grow as they did (`swell` 0, the old foliage words).
- **Flowers, four new forms**: trumpet (a lily), drooping bells down an arching stem (a
  bluebell), rose (rings of cupped petals: a rose, a peony), thistle (a spiny bulb and
  a tuft of florets). Presets: lily, bluebell, rose, peony, thistle. Bluebells joined the
  showcase's woodland floor.

**Painted by the thousand: what grows drawn lighter from farther off.** Measured first
(`-Tour -Only acres`, a new world, a field of 795 wheat patches and a meadow of 2114
plants, about 3000 things): strokes cost the host 30 to 40 ms and are drawn within a
second, and things are already drawn as instances, a mesh a region; what cost was
triangles, 28 to 46 million a frame (shadows included). So:

- The host makes, of every mesh that gives to the wind and has 1500 triangles or more,
  **three lighter ways** to draw it (`Mesh::lighter`, written after the mesh in its
  bytes as `LODS`: a reader of the mesh alone reads it as it was). It finds the pieces
  the mesh is made of (blades, leaves, petals, berries, florets; a trunk, its limbs);
  each lighter way keeps every piece big enough to be seen at its distance and a share
  of the small ones, the same every time, and draws each small piece it keeps bigger by
  as much as it leaves out: a blade or a stalk wider across, a clump or a petal bigger
  every way; never outside the box about the whole.
- Godot gives them to the mesh as its levels of detail and chooses by distance, a
  region at a time.
- Measured on the same field: **28.5 to 3.5 million triangles** from above, **43.5 to
  5.8 million** at its edge, 45.8 to 6.0 million inside the meadow; the frame's
  drawing from above 3.8 to 1.7 ms (on this machine's RTX 4080, where it was already
  quick: a weaker card gains more). The field looks the same at its edge; seen from
  far above, a little paler than drawn whole.

### Decisions made here, the user's to overrule

1. **Going through what grows is a kind's solid line** (`wade`), read the same for
   every body, not a property of a plant family or of the walker.
2. **A field slows, it does not stop**: wheat about a third slower, maize about
   two-thirds; a lawn and strawberries not at all.
3. **Bending is only while playing**, and only near the walker and the four others
   nearest it.
4. **Lighter ways are only for what gives to the wind** (what grows): a house, a wall
   or a rock is drawn whole at any distance.
5. **The lighter ways are made by the host, by leaving pieces out and drawing the rest
   bigger**, not by Godot's own simplification (which collapses thin blades and leaves
   a field the colour of the ground beneath it).

### Limits and known gaps

- Seen from far above, a field drawn lighter is a little paler than drawn whole.
- Bamboo and the heavier bushes are 7,000 to 13,000 triangles near to; lighter ways
  help only from farther off.
- What the code makes by itself (a meadow maker's own flowers, the library's plants)
  has no `wade` line: only designs grown in the Generator are waded.
- A thing's wade box is a box about it: a crop patch's rows and the gaps between them
  are one.
- Bending follows the walker and four others; a herd in a field bends it only where
  those are.
- Not tried by hand. Not seen in motion by the user.

### How it was checked

- Unit: `grown` (every preset of every family grows, within bounds, alike twice,
  silhouettes told apart, now with bamboo, the new trees and flowers; what is walked
  into and through; lighter ways: fewer triangles each, corners in bounds, in the
  bytes after the mesh, the same every time, none for a daisy or a wall).
- Protocol: p55 and p63 widened (the presets and families; each family's design says
  `wade`, bamboo its canes, grown again with one `solid` set). Whole host suite: 59
  unit, 63 protocol.
- Tours: `-Only acres` (new: the measurement above, whole against lighter, and a
  failure if lighter is not a quarter fewer), `-Only nature` (the walker slower in the
  wheat, slower again in the maize, what gives told where it is), `-Only plants` (now
  pictures every tree preset too: 60 presets of 8 families). Beside them: grow, ui,
  showcase, designs, makers, brushmix, workflow, town, physics pass. `-Demo nature
  -Check`: 7 of 7 (two beats new).
- Looked at: every new preset in the Generator's booth; the bamboo garden and the
  stranger trees in the showcase; the walker in the wheat and the maize; the field
  whole and lighter, from above, at its edge and inside it.

## 7aq. A field that fills an outline (built 2026-10-15, the next in the user's order)

The user's working defaults of 2026-10-15 put spatial authoring next after painting at
scale: "filling a selected outline with a field". Built as a maker over what there is,
not a tool of its own: an outline is a line (the **Line** maker, its places clicked
and dragged as any line's are), and a **Field** reads it, closed from its last place
back to its first.

### What was built

- **The Field maker** (`land.rs` `Field`, group "Growing things"): given designs (its
  `of`, as a wood is: one crop, or several with shares), it lays patches of them in
  rows inside its outline. The rows run along the outline's first side (`turn` turns
  them); a patch every `along` metres and the rows `across` apart (0: the design's own
  size, so patches meet); nothing within `margin` of the edge, under water, or within
  `keep_off` of a road; `gaps` of them left out. It takes the colour variation (`vary`,
  one patch's own colours) and keeps what is done to one patch by hand (`each`,
  `added`), as woods and meadows do.
- **Each patch is named by its row and its place along it, counted from the outline's
  first place** (`patch#r3c7`): drag any other place of the outline and the patches
  still inside keep their names, and so what was done to them (taken away, given
  another design, moved, coloured).
- **In the editor**: Plants > **Add a field**: click its corners on the land, Enter.
  With no crop grown in the world, wheat is grown first (the Generator's preset, a
  design named "wheat"). The outline is a line named "field outline", not a road (the
  land is not levelled under it); it is chosen when made, so its corners can be
  dragged at once. World outline, the field: **How the field is laid out** (a patch
  every, rows apart, rows turned, kept in from the edge, left out, keeps off roads,
  seed), **What it is made of**, **Colour variation**.

### Decisions made here, the user's to overrule

1. **An outline is a line**, the one there is, closed back to its first place: no
   separate shape or region object yet (7q's "a region as a thing" is still owed).
2. **The rows follow the outline's first side**, turned by a number; the first two
   corners clicked say which way a field is ploughed.
3. **Patches are named from the outline's first place**: moving that one place renames
   every patch (what was done by hand to them is then on other patches, as with a
   wood's seed). Any other place keeps them.
4. **A field does not keep clear of houses or other things by itself**: only of water
   and roads. The outline is drawn round what is to be left.
5. **With no crop in the world, wheat is grown for it**, a design of the world the person
   can then change, keep or delete.

### Limits and known gaps

- From nearly straight above, a field of thin upright stalks shows mostly ground
  between them; seen at a slant it reads as a field.
- The line tool does not draw the closing side back to the first corner while the
  outline is being clicked.
- A patch is the design's footprint, a rectangle: a field's edge is stepped, not cut
  along a slanting side.
- Growth over time and harvest are not built (the user's order: a gameplay layer).
- Not tried by hand.

### How it was checked

- Protocol `p64`: an L-shaped outline filled in rows, none outside; one patch taken
  away and one given maize; a corner dragged out, the field larger and the patches that
  were inside the same ones, the one taken away still away, the maize still maize;
  rows turned and turned back to the same patches; written and opened the same. The
  whole host suite: 59 unit, 64 protocol.
- Tour `-Only field` (new): Plants > Add a field, four clicks on the land of a new
  island, Enter; wheat grown first; the field filled, none outside, its outline not a
  road; one taken away, a corner moved out, the same patches and more. Beside it:
  makers, ui, workflow, findings, build pass.
- `-Demo field -Check`: 4 of 4. Looked at: the field drawn, its outline changed, turned
  and mixed with young wheat, walked into.

## 7ar. Signals finished to a first version: several joints a thing, what waits kept in a saved game (built 2026-10-15, at the user's asking)

The user's brief, after trying 7am and 7an by hand: finish Signals to a solid first
version and move on; keep `feels` apart from `hits`; review 7am's six open decisions
against what was built and decided since, settling the plain ones by the simplest
consistent way; build the agreed next work (several joints a thing, and timers kept
when a game is saved and opened) if it is still outstanding; keep joint anchors and
axes dragged in the view deferred; mend real blockers, and record smaller things.

**Tried by hand by the user (2026-10-15), not by script:** the batcher (conveyors and
parcels): counting, the lamps, the gate, the delay and the reset, in that order; and the
strength tester's ball, whose rebound answers how hard it struck. Nothing was reported
wrong. (Everything else of 7am and 7an is checked by script only.)

**Found still outstanding, in the code:** `joins` was a word said once (`kind::ONCE`):
a second line replaced the first, `things.gd` kept one joint, `loose.gd` kept one joint
by the thing's key, the Inspector edited one. A saved game kept the facts but not how
long a `for` had waited nor how far a `ticks` was to its next: loaded, both began again
(7am's decision 3, said there).

**7am's six decisions, reviewed**

| 7am | Now |
|---|---|
| 1. A condition is said in `when`, one a thing; `clears` and `ticks` while it holds; no expressions, no logic objects | Stands (the user, 2026-10-15: conditions stay in `when`). No `and`/`or`: two things in a row, as before |
| 2. A fact not there is 0; one that holds and is not a number is 1 | Stands: the simplest consistent rule, and every reader already uses it |
| 3. `for S` and `ticks` count the world's running time, from 0 each time it runs, not kept in a saved game | **Changed**: kept in a saved game (below). Still from 0 when the world begins to run without a game loaded |
| 4. `clears` takes back while its `when` holds; with no `when`, nothing, said as wrong | Stands |
| 5. The Signals window reads and does not edit; its "wrong" is advice | Stands; its rows lead to the thing or the text where a line is changed (7an) |
| 6. Joint editing in the view is two clicks; the axis a choice of three | Stands. Dragged anchor and axis handles stay deferred, as agreed |

**Built**

- **Several joints a thing.** `joins` may be said more than once, a line each, of a
  design or of one thing (`kind.rs`: out of `ONCE`). One thing's own `joins` lines
  replace its design's all together (the first own one takes the design's away, each
  further one is added: `own_over`); `no joins` takes them all. The host's `joints`
  lists each line by name (it did already), and a rename follows every one of them.
  `things.gd` reads them as a list; `loose.gd` makes one joint of the physics for each,
  keyed "thing/N", and a thing's joints by name are looked up by the name each says;
  View > Colliders draws them all; the Signals window reads the one with a motor.
- **The Inspector, several joints**: a chooser **joint N of M** over the joint's rows,
  and **+ another joint** (a hinge about its middle, to the world, to be told where and
  to what, with the rows and clicks there were). Changing one makes all of a thing's
  joints its own (its design's copied in); **not joined** takes away the one shown;
  **as its design** puts back its design's.
- **What waits, kept in a saved game.** Saving puts one fact of the world's, `waits`:
  how long each condition watched has held (`state.waits`) and how far each timer is
  to its next tick, by its thing (`loose.saving`). Loaded, a condition that holds in the
  loaded game has held that long again (`state.waited`), and each timer goes on from
  where it was, also one that is far off and has no body yet (`loose._ticked`). The
  Signals window names `waits` as the engine's ("a saved game (what waits)").
- **Library > Mechanisms: bridge plank** (hinged at its west end; join each to the one
  before it, the last to the world at its east end as well) and **bridge post** (41
  designs in the physics lab).
- **The lab's bridge** (`physics.rs`, at -20, -18; `showcase-physics.json` rebuilt: 150
  things put down, 40 designs): five planks, each hinged to the one before it by name,
  the last also to the world, a crate dropped on its middle.
- **`-Demo signals`** (10 beats: "Held at both ends", "Kept in a saved game"); tour part
  **`bridge`** (new); unit test and p61 grown.

**Recorded for later, not mended (small, not blocking):** a joint's limits are still
typed in its lines; the joint chooser names joints by sort and what they are to, so two
alike are told apart only by their number; a timer far off when saved keeps its tick
only until the world stops; what waits is kept per condition text, so two things that
say the very same `when … for S` share one wait (as they always did while running);
the walker is still the only thing that wakes machines (signals run near it).

**Not done, deferred as agreed:** joint anchors and axes dragged in the view; a node
graph of the wiring; joints that break.

**Next, the user's direction (breadth before polish):** the next major engine capability:
sound (7as).

**Checked** (the details in `PROGRESS.md`, "Signals finished to a first version"): host
tests all (59 unit, 64 protocol, p61 grown); tour part `bridge` (new: the bridge built
in the Inspector in a new world, held, unjoined and hanging, undone, saved and opened;
the lab's own bridge a minute; what waits through a saved game: the tick 0.48 s and the
reset 1.48 s after loading, as saved); parts mech, signals, own, physics;
`-Demo signals -Check` 10 of 10; `export.ps1 -Check`. Tried by hand by the user: the
batcher and the strength tester only.

## 7as. Sound, first tier: sounds written as recipes, things that sound by their lines, a weather's sound, a Sounds window (built 2026-10-15, the next major capability after 7ar)

The user's direction with 7ar: once Signals is a solid first version, move on to the
next major engine capability; breadth of capability before polishing any one system.
**Why sound:** the engine had none at all (no audio anywhere in the Godot side; roadmap
§4's positional audio, ambient audio, audio regions and environmental sound reactions
all unstarted), and it touches what is already built (signals, physics, weather,
the library, the exported game) without remaking any of it.

**The decision that shapes it** (the user's to overrule): **a sound is written, not
recorded**: a recipe of tones and hiss in a world text, `sounds.txt`, made into what is
heard when first wanted. It is content like every other text (kept with the world,
changed in F10, made again at once, in the exported game), needs no audio files, tools
or licences, and every sound a demo plays can be read and changed. Rejected for this
tier: **recorded files** (wav/ogg beside a world: wanted later for voices and music,
and the next tier's first piece, not a reason to wait); **a sound system of its own**
(events, buses, an editor graph): things say what they sound in their lines, gated by
the one `when` of 7am, as everything else is.

**Built**

- **`sounds.txt`** (`soundbook.gd`): `sound NAME`, then parts: `sine|square|saw|triangle
  F [to F2]` (a tone, swept), `noise LO HI` (a hiss in a band), each `at T`, `for D`,
  `loud V`, `flat` (held, else dying away as a struck thing does), `swell S` (comes and
  goes); and of the whole `length S`, `loud V`, `loop`. Sixteen written: click, chime,
  bell, thump, thud, wood, bounce, creak, whoosh, pop; birds, stream, machine, wind,
  rain, storm (round and round). A line not understood is said, with its line.
- **Made and played** (`sound.gd`): a recipe made into a 22 050 Hz sound when first
  wanted (16 sounds in about half a second all told; the longest, storm, 0.13 s), kept
  until the text is read again. Heard once where it is (16 voices; the oldest taken
  when all are in use), round and round where it stands, or all about.
- **What things say** (host `kind.rs`: traits, said once, one thing may say each of
  itself, refused with its form; `things.gd` reads them):
  - `sounds NAME`: heard once where it stands each time its `when` comes to hold (a
    lamp's click as it lights, a gate's creak as it opens);
  - `hums NAME [R]`: round and round while its `when` holds (always, without one),
    heard R metres off at most (12), each from a place of its own in its round;
  - `knocks NAME [S]`: each time it strikes or is struck at S m/s or more (1), louder
    and a little higher the harder: a loose thing by its contacts (`loose.gd`
    `_knocked`), a thing that is not loose by the room that already tells strikes
    (`hits`, `feels`; `_struck`).
  Things are listened for within 60 m of the ear (the camera), four times a second.
- **A weather's sound** (`weather.txt`: `sound NAME [N]`): heard all about while it is
  that weather, coming in and going over three seconds. Breeze and overcast the wind,
  rain the rain, storm a storm, snow a little wind; clear none.
- **Only while the world runs.** Editing, the world is quiet; Window > Sounds plays one
  when asked.
- **Window > Sounds** (`soundview.gd`): every sound written (once or round, how long,
  how many say it); **Listen** (double-click too), **Stop**, **Open the text** (F10 at
  its `sound NAME` line), **Pick one that sounds it**; who says each (things, from the
  host's `signals`, and weathers); what is wrong (a line not understood; a sound said
  and not written; one wanted while playing and not found); **How loud** and **Quiet**,
  this machine's own (kept with the editor's settings, never in a world).
- **The Signals window** knows `sounds` and `hums` do something by their `when` ("is
  heard ('click') as it comes to hold"; a thing that does several things says each).
- **Library**: a new group **Sound** with the **sound post** (`hums birds 15`); designs
  given sounds: signal lamp `sounds click`, swinging gate `sounds creak`, chime tray
  `knocks chime 1.2`, rubber ball `knocks bounce 1.5`, light crate `knocks thud 1.5`,
  strength pad `knocks thump 1`, bridge plank `knocks wood 1.5`, conveyor `hums machine
  8` (42 designs in the physics lab). Worlds that brought them in before keep their
  copies as they were.
- **The lab** (`showcase-physics.json` rebuilt: 152 things, 41 designs): a birds post by
  the start, its own `when not dark`; a stream post by the goal, its own `hums stream
  10`; and all that sounds by its design.
- **`-Demo sound`** (7 beats), tour part **`sound`**, unit test, protocol **p65**. Tours
  are quiet (the master is muted for them; everything is still made and played).

**Decisions made here, the user's to overrule**

1. Sounds are written recipes (above); recorded files are the next tier, beside them.
2. Three words on things, all gated by the one `when`: `sounds` (once, as it comes to
   hold), `hums` (while it holds), `knocks` (at a strike). No sound events of their own.
3. The world is heard only while it runs; editing is quiet.
4. A weather's sound is the weather's to say (`weather.txt`); time of day is said by a
   thing's `when` (`when not dark`).
5. How loud is this machine's, not the world's: a world says what sounds, not how loud
   a person wants it.
6. Positional: a thing is heard where it stands, fainter with distance and not at all
   past its reach (Godot's own falling off); a weather everywhere.

**Limited, or not there: engine gaps**

- No recorded sound (files), music, or voices; no footsteps (the walker, people,
  animals are silent); acts and doors that open by an act do not sound (a door that
  opens by `when` can say `sounds`).
- No echo, no sound through walls or muffled behind them, no Doppler; 16 voices at once.
- What is far from the ear (over 60 m) is not listened for; a machine far off is silent
  (it also does not run: 7am).
- A loop is made with its seam smoothed, not hidden: a hiss loop may be heard to go round.
- A sound is made the first time it is wanted: an 8 s loop holds a frame up about 0.1 s.
- A thing with a body of its own that is fixed (a bumper) knocks only if it also counts
  or feels strikes, or is loose.
- Not heard by anyone yet: the checks are that each sound is made, played where and
  when it should be and as loud; whether they sound good is for a person to say.

**Next, recommended:** recorded sounds beside the written ones (a `file` part of a
sound, the library's own sounds), footsteps by what is underfoot (the walker first),
music and places that sound (a region, not a post); then §4's other side, effects:
particles as content (what falls, sparks, dust, splashes), joined to the same `when`.

**Checked** (the details in `PROGRESS.md`, "Sound, first tier"): host tests all (60 unit,
65 protocol, p65 new); the recipes made headless; tour part `sound` (new); `-Demo sound
-Check` 7 of 7; problems [] and no script error (the .err logs read too): signals, mech, physics, own, loose. **weather**: one problem, not new: its storm check wants the lean and sway as written (0.75, 0.36) and gets them times the air's strength 2.5 (1.875, 0.9) that weather.txt has said since f3d71b0 (Known gaps); `weather.gd` multiplies them so, and sound does not touch it. Not mended (whether 2.5 was meant is the user's).; the built game: passed (delivery, saved, left, continued, no host left running); the game's world carries sounds with the other texts. Nobody has
listened yet.

## 7b. How much is tested, and when (the user's rule, 2026-10-09)

Tested narrowly unless there is a reason: the build and the unit tests of what
changed; one end-to-end check of the way of working touched; the parts beside it the
change could reach; the whole tour only when reach, evidence or risk is worth it.
Looked into further, and tested more widely, on a sign of: lost or spoiled work of
the user's; writing and opening, identity, provenance or an override not kept; a
crash or a hang; something broken across systems; or the milestone's own proof not
holding. One thing out of place that is none of these gets a few useful checks, is
written down exactly as seen and as not explained, and waits for its part to be
worked on next. It does not by itself hold up a commit. What was not run is never
said to have passed. The words in full are at the head of `PROGRESS.md`.

## 7at. The Character Studio, first tier: characters with bones of their own, clips from glTF libraries, a samurai set (built 2026-10-15, at the user's asking)

The user's brief: build toward a reusable character-authoring workflow (character and
body → rig → animation and poses → movement and action sets → equipment → reactions →
effects and sound → behaviour), reusing Godot's native systems and proven free assets
where they fit; research first, then a small runnable end-to-end prototype; the motor
governs movement and animation follows it; every automated run leaves the person's mouse
and controller alone. The research record, with sources, licences and costs:
`Draw3D_Character_Studio_Research.md`.

**What it answers in principle N** (the provisional list in `PROGRESS.md`): "a set has a
look of its own on bones it shares" is answered for characters (a look per character);
"a move is about nothing; an act is" stays; a hit is now a move's (`strikes`), and a
reaction a move begun upon it (`upon struck`). The six questions: no system that makes
authored moves harder (the motor is untouched in what it decides; two words added);
nothing coded that should be content (characters, maps, looks, libraries are text; the
clips are files); movement, presentation, interaction and physics still apart (the
figure shows, the motor moves, a strike is a query and an impulse, an act is an act); a
move set joins by `moves SET` in a character; an intended interaction joins as before
(acts on a clip-shown figure: it is a figure); places are said (hands, treads, carry,
head from the map; a blade on what is held).

**Built** (details in PROGRESS.md, "Character Studio, first tier")

- Assets as glTF made by script (`charmake.gd`): Draw3D's poses baked to a clip
  library, a samurai library from new pose text, a mannequin on another skeleton, a
  katana.
- `clips.gd`: runtime glTF reading (Godot's `GLTFDocument`); retargeting by Godot's
  humanoid profile names, rest-relative in the body's frame, with `stance` and `faces`.
- `characters.txt` (`charbook.gd`): libraries (with provenance and licence), maps,
  characters, looks; kept by the host with the world, as every text.
- `animated.gd`: a figure shown by clips through Godot's AnimationTree used as a mixer;
  clip time from the motor (phase progress, distance walked); transitions told from
  speed and facing; feet let down; things held; the figure API kept.
- Moves: `strikes PLACE R`, `upon struck`; the samurai set; `strike.gd`.
- Characters > Character Studio (`studio.gd`): preview, Body and bones, Clips (search,
  tags, play, loop, speed, scrub, source and licence), Movement (set, look, moves timed,
  try a move), Holds, Strikes and reactions; walk as, copy, copy the look.
- Automated runs: no pointer captured, no real controller heard, no focus taken.

**Decisions made here, the user's to overrule**

1. **The motor stays the only authority on position.** Clips never move the body; root
   motion is not used. A move's clip is played as far as the move is through its phase.
   (Root motion could later *measure* a clip to fill a move's numbers.)
2. **AnimationTree as a mixer, not a state machine.** What happens is decided once, by
   the motor and the moves; the tree only blends what the figure is told to show.
3. **Retargeting is baked when a character is made** (per clip, once, cached), not run
   live by `RetargetModifier3D`: one skeleton, deterministic clips, exact checks.
4. **Godot's humanoid profile is the shared vocabulary** of bones; each set of bones has
   a map to it. Non-humanoid bodies (a sheep, a horse) need their own vocabulary: not
   in this tier.
5. **Clip names use `_`** for a phase (`slash_cut` for `slash/cut`), because Godot's glTF
   reader makes `.` a `_`.
6. **A hit is a move's** (`strikes`, a timing written in moves.txt); a clip's marks are
   how it looks, shown beside the move's timing, and do not decide anything yet.
7. **Studio changes are the world's** (document edits, Ctrl+Z, Ctrl+S), never the file
   the project starts new worlds from; the walker is a character by a `walks` line.
8. **Draw3D's own assets first.** The pipeline was proven with files Draw3D makes, which
   go through exactly the path a downloaded file would. Nothing was downloaded.

**Limited, or not there: engine gaps**

- No external clip library yet (Quaternius CC0 proposed below); the samurai's clips are
  keyed by hand in the pose language, not captured motion.
- No layering (upper body of one clip over another's legs), no IK on a clip-shown figure
  but the edge holds, no `over` poses (carrying) on it.
- The folk are not given characters by content; only the walker, and bodies a script
  makes.
- A blow does no damage, is the same from every side, cannot be guarded; marks neither
  sound nor put facts.
- The Studio plays clips; it does not edit their keys or a move's numbers.

**Proposed, larger than the usual pass: for the user's approval**

- **Bring in Quaternius's Universal Animation Library (CC0).** Download the free
  Standard pack (glTF) and, if wanted, UAL 2 (parkour, combat combos, farming): one
  library entry, one map (and `stance`/`faces` if needed), a look. Small in code; needs
  the user's yes to download (size to be stated before), and it changes how every
  character looks. Alternative: keep Draw3D's own keyed clips and improve them by hand.
- **Feet and hands by Godot's IK** (`TwoBoneIK3D`) on clip-shown figures: feet on slopes
  and steps, the second hand on the hilt. Moderate; touches every clip-shown figure.
- **Characters for the folk** by content (a kind or a design names a character): the
  host's kinds would say it (host code). Moderate.

**Next smallest slice** (recommended): with the user's yes, the Quaternius library
brought in and mapped, and a look made from it for the mannequin, compared side by side
with Draw3D's own in the Studio; then upper-body layering (the sword held in guard while
running).

**Then, with the user's yes (the same day): Quaternius's Universal Animation Library,
Standard, brought in.** CC0, 14.5 MB from OpenGameArt; its Godot glTF kept (45 clips and
a T-pose, a 53-bone Rigify rig made in a T facing +z, its own model). Mapped (`map ual`:
`faces +z`, upper arms turned down) and given a look (`look ual`), with two characters
(`quaternius`, `mannequin_ual`). Three small engine changes it showed were needed, the
user's to overrule: a look may show a `part FROM TO` of a clip (one clip for a move's
several phases); a map's `stance` turns a bone in the body's frame; a model that faces
+z is turned half round when worn. The proposal "bring in Quaternius" is so done for the
Standard version; UAL 2 would be one more library line. Next smallest slice now:
upper-body layering (the sword held in guard while running), then IK for feet and the
second hand.

## 7au. The Character Studio, authoring: a real body in the preview, a phase changed, a move turned and mirrored (built 2026-10-15, at the user's asking)

**The ask**: make the Studio a place where actions are made well, not only browsed:
fix what did not work, preview and change actions, raise the quality of special actions
(samurai first), look at the RPG pack and other sources, keep everything reusable.

**Decided here, the user's to overrule**:
- The preview runs the game's own body, motor and moves (asked as a player asks); it
  does not imitate them. What is seen there is what the game does.
- A move is changed in two places, each where it belongs: what the body does in
  moves.txt (time, distance, turn, strike), how it looks in the character's look
  (clip, part, mirror, fade, pose bends). A clip's file is never changed.
- A pose is changed by `bend` (a place turned further over the clip, in the body's
  frame), not by keyframes: cheap, readable, reusable; keyframing is proposed apart.
- `spins` turns the body over a phase; what follows goes that way round.
- A phase strikes each thing once (was: a move once).
- "Save as a new move" puts the copy in the character's set in place of the one it came
  from; the original stays for others.

**Proposed, for approval, each larger than a pass**:
1. **IK on clip-shown figures** (Godot's TwoBoneIK3D / SkeletonModifier3D): feet planted
   through turns and on slopes, the second hand kept on the hilt. About two passes.
2. **Layering** (upper body of one clip over another's legs: guard held while running,
   carrying while walking): an AnimationTree blend with a bone filter. About one pass.
3. **Keyed pose edits** (a bend that changes over the phase, a few keys): a small curve
   editor in the tab. One pass; a full keyframe editor would be several and is not
   proposed (Godot's own editor can edit an imported clip if ever needed).
4. **UAL 2 Standard** download and look (CC0, 10.3 MB): less than a pass, the user's yes.
5. **The RPG pack** once the user has downloaded it: about an hour.

**Next smallest slice**: UAL 2 brought in (its combos as phases of the samurai's moves,
mirror and bend to fit), then IK for the second hand on the hilt.

## 7av. The Character Studio made reliable; phases, combos and Universal Animation Library 2 (built 2026-10-15, at the user's asking)

**The ask**: the preview looped what should play once, some moves showed nothing
(thrust, backflip, long jump on some characters), and the phase controls often seemed to
do nothing; make previewing and editing reliable first, in plain words with sliders,
then reusable phases and combos on Universal Animation Library 2.

**Root causes found** (each by a headless audit of every character's every move,
`studioaudit.gd`, which asks a body exactly as the preview does):
- *Looping*: the preview's "Round and round" box was ticked by default and applied to
  moves tried, so every try began again at its end.
- *Moves that never began*: the Studio looked for the words of a move's `when` list
  (`running`, `crouch`) as strings, but each is a list (`["running"]`): no run-up was
  ever asked, no crouch held, so thrust, long jump, backflip and dodge were refused by the
  motor, on every character. A move begun only by a hard landing was never tried.
- *Someone to strike* stood in the run-up's path from the start: a thrust's run was
  stopped before it was fast enough to be a run.
- *A phase with no clip at all*: the samurai's `thrust/recover` (shown standing). A pose
  for it was written and the samurai library baked again.
- *Phase controls that did nothing*: a phase shown by the clip of its own name
  (`slash/cut` by `slash_cut`, no row in the look) had its part, mirror, fade and pose
  controls greyed out or writing nothing.
- *A character switched while a move went* tried the old move again on the new one.

**Decided here, the user's to overrule**:
- One way of asking a body to show something (`trying.gd`), used by the preview and by
  the headless audit and checks: what is tested is what is seen.
- A move is played once and then stands where it ended; Loop is ticked by itself only
  for what goes on (a looping clip; standing, walking, running, crouching, sneaking).
- A move that needs a wall, an edge or something low is not faked on the bare floor: its
  clips are shown by themselves ("clips only"), and it is said so.
- Each move says how ready it is to be seen on the chosen character (ready, a phase with
  no clip, no clips, a clip the look names that it has not, clips only here), and a clip
  that might do for a missing phase is offered, never put in silently.
- A change to how a phase shown by its own-named clip looks writes a look row for that
  clip first; `backwards` is a new look word (a part played from its end).
- A combo is written in moves.txt (`combo NAME`, `then MOVE [wait S]`) and says only
  the order: when each next move may begin stays the move before it's (`lets ... from`),
  so the same windows hold for a player's presses and for a combo.
- Universal Animation Library 2 (downloaded with the user's yes) is a library of its own
  on a map of its own (`ual2`: Unreal's mannequin's names); its cuts are cut into phases
  at the hand's fastest moments, measured by script; its `_RM` clips play in place.

**Proposed, for approval** (unchanged from 7au unless said):
1. IK on clip-shown figures (feet planted, the second hand on the hilt): about two passes.
2. Layering (upper body over other legs): about one pass.
3. Keyed pose edits (a bend that changes over the phase): one pass.
4. Combos asked in the game by a button sequence or by a folk's way (a combo is now data;
   the motor already gives a player the windows): a small pass.
5. A practice floor in the preview with a wall, an edge and a low box, so the moves that
   need them are tried as moves rather than as clips: about one pass.
6. The RPG pack once the user has downloaded it: about an hour.

**Next smallest slice**: the swordsman's three cuts seen by eye and tuned (bends, fades,
the katana's grip on UAL 2's hand), then IK for the second hand on the hilt.

## 7aw. The Character Studio's playback made explicit; its panels steadied; one brush agreed on; a UI proposal (built 2026-10-15, at the user's asking)

**The ask** (after the user tried 7av by hand): once a combo had been played, Play went
on playing that combo when a clip, a move (rise) or another combo was chosen; dash-and-cut
did not look right. Fix playback first, at its root; then make the Studio easier to
understand (selection, phases, editing, panels that jump); look into the Plant Generator
and Objects disagreeing about what is painted; propose a broader UI overhaul, not build it.

**Root cause of the stale playback**: the preview had no notion of what was *chosen to
play*, only of what was *last tried* (`last_try`). Play, at the end of a try, replayed
`last_try`; choosing a row only started something when that list's handler ran, and the
handlers skipped a row "already chosen" (the clip list compared with the clip last
shown, the ways list with what was tried). Rows stayed chosen in one list while
something from another played, so a click on them changed nothing, and Play replayed the
combo. Nothing was wrong with the clips, the runner or the motor.

**Fixed by**:
- One target: whatever is chosen in any list (a clip, a way, a move, a phase, a move's
  clips, a combo) becomes `target`.
- `select_target` stops whatever played (`_halt`: the runner let go, a move's clips put
  away, the clip held), shows only that row chosen in the lists, and plays it or shows
  its start.
- `play_target` is the one way anything begins. Play, Replay, a change to the texts, the
  opponent's settings and a character switch all go through it.
- A target this character cannot play is said ("cannot play: …"), and nothing is played
  in its place.
- The combo being built, changed, becomes the target as it now is.
- `trying.gd`, the runner and the motor are unchanged.

**dash-and-cut**: not the playback bug.
- Its own definition was at fault. The dash passed the opponent and the swordsman never
  turned back round: the second cut swung at empty air ahead. The comment said "on the
  way back round", but nothing turned.
- With the opponent where the Studio puts them (straight ahead), the dash ran into them
  and stopped after 1 m.
- Measured headlessly: the blade sweeps the swordsman's left as it passes.
- Fixes:
  - `dash_slash` has a `turn` phase (0.25 s, 180°) after the dash; the recovery lets
    cut_b in at once.
  - The look splits Sword_Dash_RM's part at 0.62 s.
  - A move covering more than 2.5 m has its opponent 0.7 m to its left: in the Studio
    when aside is left at 0, and in the audit.
- Now the dash strikes, the swordsman turns 180°, and cut_b strikes again (twice in
  all).

**Usability (bounded)**:
- **Status:** two fixed one-line status lines ("Play: what · state · once/looped · the
  phase now"; what the body did).
- **Layout:** the preview and the tabs split by a draggable line, and the preview smaller
  to begin with.
- **Timeline:** click a phase to change it; the phase being changed is outlined; hovering
  says its role.
- **Change a phase:**
  - a **See it at** scrubber for the phase's clip;
  - **Look / Timing / Movement / More** in Godot `FoldableContainer`s, each saying whose
    it changes;
  - the paragraphs shortened into tooltips.
- **Show it with the clip chosen:** keeps the look row last clicked apart from what
  plays.

**Paint, Objects and the Plant Generator**:
- There is one brush (`marks.gd` `strew`), but each panel kept its own "chosen".
- Choosing in either never changed the brush, and only the long context row named what
  it held. So, after painting crates, choosing a plant preset left crates on the brush.
- Fixed:
  - In Objects, a row clicked while painting moves the brush to it (or puts it down if
    it cannot be painted), and says so.
  - In the Plant Generator, its Paint button shows and toggles its own brush. Another
    plant shown puts that brush down. A brush taken up in Objects is named on a fixed
    line.
- Not changed: the pendulum (a pinned loose kind) is still offered for painting. That
  is proposal item 5.

**Decided here, the user's to overrule**:
- Choosing a clip only shows it (Play plays it).
- A move, a way or a combo clicked plays at once (as before).
- Choosing anything stops what played.
- A character switch keeps what is chosen but plays nothing until Play.
- In Objects the brush follows the row clicked; the Plant Generator puts its brush down
  rather than make another design unasked.

**Proposed** (`Draw3D_UI_Overhaul_Proposal.md`): one active target shown everywhere;
choose / configure / apply kept apart; a panel standard (height bounded by the screen,
one scroll, folded advanced parts, a fixed status line); one list pattern; Godot's own
controls.
- **Soon:** bounded windows, a status-line helper, the bar's chip naming what is in
  hand, folding, painting offered only where it means something.
- **Medium:** one asset browser, a docked contextual panel, one route per task.
- **Larger, a decision each:** a docked layout, a real animation timeline, the renaming
  pass.

**Not built**: anything of the proposal beyond the Studio and the two painting panels;
a timeline scrubber for a whole move (the phase scrubber and the clip scrubber are);
IK, layering and the rest of 7av's proposals (animation work is parked, at the user's
word).

## 7ax. The Character Studio's workflow: what can be changed first, one Edit for a move or a combo, Save / Save as a copy / Discard, Where under the preview (built 2026-10-15, at the user's asking)

The user tried 7aw by hand: three cuts played right; dash-and-cut never played (choosing
it left three cuts going); phases could be changed, but there was no plain way back to
the whole combo; the tabs began with what cannot be changed; there was no way to make a
move or a combo from nothing, nor to scrub a move. They asked for the order of the tabs
to follow the work, for a move or combo to be edited where it is listed and saved in
place or as a copy, for part of a combo and the whole of it to be played from where it is
edited, and asked about keyframes, Mixamo, and one library of moves for every character,
creature and game.

**Dash-and-cut, the cause.** A click on a combo's row made the Combos list anew while the
Tree was still telling of the click. Godot refuses new rows then (`create_item` returns
null), so the code stopped before Play, and the combo before went on. 7aw's test called
`open_combo` itself, not a click, so it passed. Now:
- every list acts on a click just after it (`studio.gd` `_on_row`, deferred);
- the test (`studioplay.gd`) clicks rows as a person does: events pushed into Godot's own
  window, the computer's pointer untouched, the click said to land only if the Tree then
  has that row chosen. Clicked back and forth between the two combos while each plays,
  the one clicked plays every time.

**The tabs, in the order of the work**: Moves, Combos, Edit, Ways of moving; then what is
there to look at and choose from: Clips, Strikes and reactions, Holds, Body and bones.
- **Moves**: its set (sets are shared: anyone given the set has them), any move of
  moves.txt added to the set or taken out of it, the moves listed. A click plays one;
  Edit this move (or a double-click) opens it; New move makes one.
- **Combos**: the list; a click plays; Edit this combo (or a double-click); New combo.
- **Edit**: one move, or one combo and the move of it chosen.
  - What is edited, said at the top, and whether it is changed and not saved.
  - Save, Discard changes, Save as a copy (under a name written beside, or one found),
    Done (back to the list it came from; asked first if there are changes: Save, Discard,
    or keep editing).
  - A combo: its moves in order, one chosen; Play the whole combo; Play this move alone;
    Earlier, Later, Remove it; how long after the one before lets it in each is asked;
    Add at the end.
  - The move's phases in a row, one chosen; Add a phase after it (named, or found);
    Remove this phase (a move keeps one).
  - Then the phase as 7aw had it: Look, Timing, Movement, More. "Choose a clip…" opens a
    list of the character's clips in a window of its own, found by name or word, each
    played as it is chosen.
  - A phase on the bar under the preview clicked opens it in Edit: a combo that was
    playing opens as the combo, at that move.
- **Ways of moving**: the look's rows (stand, walk, run…), each shown by a clip; "Choose
  a clip for it…" uses the same list.

**How saving works.** Every change is written at once, as before, so it is seen at once.
What each entry was before its first change is kept (`_before`, in `_write`).
- Save keeps the changes (with this world; Ctrl+S writes the world to its file).
- Discard writes back what was (a move made new and never saved is gone again).
- Save as a copy writes the changes under the new name and puts the one edited back as it
  was. A move's copy goes into the set beside it, with how it looks. A combo's copy is a
  combo of its own; the changes to its moves' phases are the moves' own, and are kept.
- Ctrl+Z still takes back each change.

**A move `like` another** (cut_b is like cut_a) has some phases from that one. A change
to such a phase used to be refused. Now that phase is written into the move whole, where
it was, and then changed; the move it is like is untouched (`_inherited_phase`).

**Where, under the preview**: a slider and ◀ ▶ (a thirtieth of a second) for whatever Play
plays:
- a clip: shown at that moment;
- a move's clips, or one phase's: the same;
- a move or a combo tried: played again from its beginning to that moment, in steps of
  1/60 s, as the game would, so the body is where it would be. Held there; Play goes on.
How long a move takes is known once it has been played to its end; until then it is
estimated from its phases.

**Checked, by script only** (nobody has used any of it by hand):
- `studioplay.gd`: 49 checks, all passing. They cover rows clicked in every list; the two
  combos clicked back and forth; the tab order; a combo opened in Edit, its second move
  changed, played whole, discarded; a phase from the move it is like; the order changed
  and saved as a copy; a move changed and saved as a copy; a new move, a phase added,
  discarded; a new combo saved under a name; Where on a move (halfway, earlier, Play
  going on) and on a clip.
- `chartest.gd` 70/70; the tours `studio` and `moves` and `-Demo studio -Check` (8/8) with
  no problem and no script error. The tour and demo now choose tabs by name.

**Answered, not built** (each the user's to decide):
- **Keyframes.** A move is phases over clips: a phase's clip, its part, backwards, mirror,
  a pose bent over the whole phase, its time, how far it goes and turns, what strikes.
  There are no keys inside a clip, and no keyed curves. Where and the phase slider look
  at any moment; they do not key one. Keyed bends within a phase, or a real key timeline,
  is the larger item 10 of the UI proposal and overlaps the parked animation work (IK,
  layering).
- **Mixamo.** Clip libraries are read as glTF (`clips.gd`, GLTFDocument). Mixamo gives
  FBX on its own bones (mixamorig:…). Godot 4.7 can read FBX while running (FBXDocument),
  so the work is: read .fbx as well as .glb, a `map mixamo`, and an Import clips… button
  that writes the `library` entry. That is a pass of its own. Mixamo's files may not be
  given away loose; they may ship inside a game (research note, read again before any
  game ships).
- **One library of moves.** It is one already: moves.txt holds every move and combo, and
  any set may have any of them. A character (`moves SET`), and every kind (cast.txt,
  town.txt: folk, worker, horse, dog…), says which set it has.
  - What is missing is choosing a kind's set from a window; until then it is a line of its
    text.
  - Folk and creatures do a move when their ways say so. Enemies that choose combos for
    themselves are the behaviour system's (7g), not built.
  - A move saved in the Studio is the world's (kept with it), not the project's file that
    new worlds and the next game start from. Whether Save should also write the
    project-wide moves.txt, or a separate "save to the library", is a decision.

**Not built**: the above; a kind's set chosen from a window; renaming a phase or a move;
the clip picker's preview in the Edit tab itself rather than the main preview.

## 7ay. Mixamo's clips taken in: FBX read, a map for Mixamo's bones, Import clips… in the Studio (built 2026-10-15, at the user's asking)

The user means to download animations from Mixamo later, and asked whether they could
drop them in a folder or needed an import feature. 7ax answered: a pass of its own. This
is that pass, and only that: animation work beyond it stays parked, at the user's word.

**FBX read.** `clips.gd` reads `.fbx` as well as `.glb`/`.gltf`, with Godot 4.7's own
FBX reader, `FBXDocument` (ufbx). It has the same shape as `GLTFDocument`, so the bones,
the clips, retargeting and the characters that play them work the same way. glTF is read
as before.
- Godot makes Mixamo's `mixamorig:Hips` into `mixamorig_Hips`, and its take `mixamo.com`
  into `mixamo_com`. Centimetres become metres. All three were seen in a file read by
  script.
- Each file Mixamo gives has one take, always named "mixamo.com". A file whose only clip
  has that name has the clip named after the file instead ("Waving Step.fbx" gives
  `Waving_Step`, a name a look's line can use).
- A library's `file` may now be a folder (ending in `/`): every glTF and FBX file in it,
  read as one library. The bones are the first file's. Mixamo gives a file per clip, so
  this is how a library of Mixamo's clips is written.
- `Clips.bone` also finds `mixamorig:X` as `mixamorig_X`, and under a numbered namespace
  (`mixamorig1_X`). Mixamo numbers the namespace when a scene had more than one character.
- An FBX clip is at least as long as its last key, whatever its take says.

**`map mixamo`** in characters.txt: 22 of Godot's humanoid names, as `ual` and `ual2`
have.
- Chest is Spine1, UpperChest is Spine2, the upper arm is Arm, the lower arm is ForeArm,
  the upper leg is UpLeg, the lower leg is Leg, the toes are ToeBase.
- `faces +z`, like Mixamo's characters.
- `stance LeftUpperArm 0 0 -86` (and +86 for the right arm), the same as Quaternius's.
  Both are made in a T with the arms level: Quaternius's measured exactly level by
  script; Mixamo's from what Mixamo is known to give. Not measured on a real Mixamo file
  (see below).
- Fingers and the bones at the ends are not mapped.

**Import clips…** (Studio, Clips tab, beside Find). Godot's own file window opens, inside
the editor's window, never the system's. It starts in Downloads (later, where files were
last taken from).
- Choose FBX or glTF files and press **Import the files chosen**, or press **Import the
  whole folder**.
- The files are copied into `editor\godot\characters\NAME\`. NAME comes from the folder
  they came from. Files taken loose from Downloads or the Desktop go to `mixamo` (all FBX)
  or `imported`. That folder gets a `.gitignore` of `*`, so nothing in it can be
  committed (Mixamo's files may not be given away loose), and a `.gdignore`.
- The library is written as a document edit (Ctrl+Z takes it back, Ctrl+S keeps it):
  `file characters/NAME/`; its `rig`, found from the bones (the map with the most of
  Godot's names found on them, here `mixamo`); `from` (Mixamo, downloaded with the user's
  own sign-in, where from, the date); `licence` (Mixamo's terms in short, pointing to the
  research note); `tags NAME mixamo`.
- The character chosen gets the library at the end of its `clips` line. Its own clips
  come first, so a name it already has still plays its own.
- The Clips list is read again, and the first clip imported is chosen.
- Importing again under the same name adds files to the same library. What was written of
  its clips is kept.
- A world whose characters.txt was kept before `map mixamo` existed gets the map written
  in from the project's file.
- `.gitignore` also lists `editor/godot/characters/mixamo*/`.

**Checked, by script only.** No file of Mixamo's is on this machine. Downloading one
needs the user's Adobe sign-in, and the files may not be committed. So `fbxfixture.gd`
writes Mixamo-like FBX files: ASCII FBX 7.4, bones named `mixamorig:…`, centimetres, Y
up, facing +z, a T with level arms, one take named `mixamo.com`. It makes two of them, a
wave of the left arm with a step and a twist ("Waving Step.fbx"), and a nod with the
right arm raised ("Nod.fbx"). One bone is turned where it stands, as Mixamo's are.
- `chartest.gd` 76/76, the glTF checks unchanged among them:
  - the folder read as one library (two clips named by their files, 23 bones, the hips
    1 m up);
  - the map's 22 names all found;
  - a numbered namespace found;
  - both clips carried to the mannequin and to a person, every limb kept within 1.1° and
    4.0° of the file's way;
  - one FBX file by itself named by its file.
- `studioplay.gd` 58/58. Import clips… with the folder:
  - both files copied in, kept out of git;
  - the library written with rig mixamo, from and licence, nothing in characters.txt not
    understood;
  - the swordsman plays it after its own;
  - the clips listed, and `Nod` chosen;
  - Waving_Step on the mannequin's bones: the left arm level at the start, raised 60° at
    the middle, as in the file;
  - Nod plays;
  - a second import under the same name adds to the same library;
  - a file that is not a clip is refused.
  - The folder it copied in is taken away again afterwards.
- `-Tour -Only studio`: no problem, no script error.
- `-Tour -Only ui` fails: the Studio window does not fit the tour's 720 px window (least
  691×760). Commit 3066149 (7ax), run from a scratch copy, fails the same with the same
  numbers, so this pass did not cause it (PROGRESS, Known gaps).

**What could not be checked**:
- Mixamo's own bones: their lengths, how each is turned where it stands, whether the arms
  are exactly level. When a library with rig mixamo is on the disk, `chartest.gd` measures
  its arms and fails if they are more than 6° off level. It printed that none was there.
- The binary form of FBX, which is what Mixamo gives. Only ASCII FBX was read; it is the
  same reader.
- A Mixamo file "With Skin" (its mesh). As a library only its bones and clips are used.
  As a character's `model` it should work (the same `instance`) but was not tried.
- Clips downloaded without **In Place** move the hips forward. Carried across, they would
  walk the body out of itself, so the checklist says to tick In Place.
- The FileDialog itself was not clicked through (its result is what `import_clips` is
  given). Nobody has used any of this by hand.

**Not built**: choosing a library's name in the window (it is the folder's); a library's
clips given tags by their names (idle, walk…); a Mixamo character as a model.

## 7az. The Character Studio made easier to work in, and how a move reaches a player's hands: Edit's head pinned, side by side, buttons by colour; what presses each move, said and changed (built 2026-10-15, at the user's asking)

The user found the Studio usable enough, but too much scrolling between a parameter and
Play, and asked three questions: how a move becomes a character's in a game (can the
swordsman's cuts be given to their own character?), how keys and buttons are mapped to
moves, and how a combo is played in the game. A focused pass, not another redesign.

**What plays stays in sight.**
- Edit is two parts. **Pinned:** what is edited and whether it is changed (one line each,
  whatever they say), Save / Discard changes / Save as a copy / Done, a combo's moves and
  Play the whole combo / Play this move alone / Earlier / Later / Remove it, the move's
  phases, and ◀ Phase / Phase ▶ / Play this phase / Play the whole move / Whole move,
  clips only. **Scrolled:** Asked with (new, below), adding and removing a phase, the
  combo's waits and Add, then Look, Timing, Movement and More.
- **Side by side** (top row): the preview and its controls on the left, the tabs on the
  right, the window made wider (1240 px where there is room), and back. Godot's own
  SplitContainer turned from vertical to horizontal. Nothing is duplicated.
- A detachable parameter window was not built. Side by side gives the same thing (the
  numbers beside the preview, both in sight) without a second window. A real second
  window would need either the desk to hold one tool in two windows, or Godot's Window
  node (an OS window) with the Studio's controls moved into it. Either is a bigger piece
  of work, for the user to ask for if Side by side is not enough.
- The tabs' rows that could change height while playing (the phase heading, the edit
  state) are one line each now. The two status lines under the preview already were.

**Buttons by colour.** The words of a few buttons only, in the workspace's own colours:
- green: what plays (Play, Replay, Play it, the Play buttons in Edit);
- blue: what opens something to edit (Edit this move, Edit this combo, New move, New
  combo, Choose a clip…);
- gold: what keeps a change (Save, Save as a copy, Add to the set, Add at the end);
- orange: what undoes or takes away (Discard changes, Take out, Remove it, Remove this
  phase, Reset all, Stop).

Everything else stays as it was. `CUES` / `CUE_OF` in `studio.gd`; set as each button is
made.

**How a move becomes a character's** (found in the code, and checked):
- **Moves are one library** (moves.txt). A **set** is a list of move names. A character
  (characters.txt `moves SET`), a kind of the world (`moves SET` in its traits) or the
  walker made by the host has **one set**. Any set may name any move. Moves are not tied
  to a rig, a model or an animation library: the motor does a move whatever body it has.
- **What it looks like is the character's look**, not the move. A look row names a clip
  for a move or a phase (`cut_a/cut Sword_Regular_A part …`). A character plays only the
  clips of its own libraries (`clips ual2 ual samurai basic`). A phase with no clip on
  that character is shown as the body stands: "no clip" in the Moves list.
- **What it strikes with** is a place: `strikes blade` means the place named blade on what
  it holds (`holds … as blade`). A character holding nothing still strikes, at a point
  0.8 m ahead of its chest (`strike.gd`). So a sword is only for the look and the reach.
- **What presses it** is its `asks BUTTON`: a name in the controls' table (`controls.gd`
  MAP). Of the moves in a set that one button asks, the first in the set's order that can
  begin does. The walker's set has `dash` (asked with dash, whenever it stands) before
  anything added to the set. So the swordsman's cuts, added to the walker's set, never
  begin from a press: the dash wins. Checked (`chartest.gd`).
- So giving the swordsman's three cuts to another character takes:
  1. Its set has them (Moves, Add to the set).
  2. They come before `dash` in it (Moves, Earlier in the set). The dash button then cuts
     on the ground and dashes in the air (checked), or each cut is asked with another
     button (Edit, Asked with).
  3. Its look shows them (Edit, Look, Choose a clip…, from a library it has; or the
     swordsman's look and clips lines copied in its text).
  4. Optionally a sword (its `holds` line, F10; no control in the Studio).
  Or simply: its set made `swordsman` (Moves, Its set) and its look `swordsman` (its
  text).

**New in the Studio for that:**
- Moves: an **Asked with** column. Orange when an earlier move with the same button can
  begin whenever this one can, or no key presses it; hover for why.
- Moves: **Earlier in the set**, **Later in the set**, **The game's buttons** (a window:
  each button a move can be asked with, what presses it on a keyboard and on a
  controller, and which of this character's moves it asks, in order).
- Edit: **Asked with** for the move edited: a list of the buttons, or "by name only".
  Written as the move's `asks` line under its name, for every character with the move;
  Save keeps it, Discard puts it back. A move written `like` another cannot be made "by
  name only" (it would keep the other's button).
- Edit, a combo: **In the game**: how a player presses it, step by step ("1. cut_a: press
  dash (F or click on the keyboard, B or RT on a controller), when on the ground. 2.
  cut_b: press dash again while 'cut_a' is in its follow or its recover…"). Whether this
  character can play it so, and what stops it if not: a move without a button, one not
  in the set, a link missing, a move that always wins first.

**How a combo is played in the game** (found in the code, and checked):
- A combo (`combo NAME` / `then MOVE [wait S]`) is used only by the Studio's Play and the
  checks, which ask each move **by name** as soon as the one before lets it in. Nothing
  in the game reads a combo.
- In the game a chain is the moves' own windows: a phase's `lets NEXT [from F]` lets NEXT
  begin if its button is newly pressed while that phase lasts (from F of the way through).
  A press is remembered for `input_buffer` (0.12 s, F1). A button held down is one press.
- So there is no "advances by itself" in the game. Every move of a chain is a press,
  pressed in the window of the one before. Checked with the swordsman:
  - dash pressed, then again in each cut's window: the three cuts;
  - once: the first cut alone;
  - held down: the first alone;
  - pressed 0.3 s before the window: forgotten;
  - pressed after the chain ended: the first cut again.
- What stops the next move:
  - its `when` (a cut wants the ground);
  - its rest (`rests`);
  - the window (`lets … from F`, the phase it is in);
  - nothing pressed in it.
  A move ends after its last phase; its recovery is that phase. Interruption is the same
  thing: only what the move going `lets` in can begin before it ends. Being struck interrupts anything, in
  a set that has a move `upon struck`.
- Found while checking: pressed the moment the first cut ends (within its 0.05 s rest),
  the **second** cut begins, from rest. cut_b is in the set and asked with dash, and
  nothing says it only follows another move. Left as it is (PROGRESS, Known gaps).

**Not built, for the user to say:**
- A button of its own for attacks. Every face button and trigger of a controller is taken
  (RT is a second dash), as is the left click. Something must give way.
- Changing which key or button is which in the editor: the table is code
  (`controls.gd` MAP). The smallest reusable step is a `controls.txt` read over that
  table, with a row per action, so the card (H), the Studio and the game all follow it.
  Whether it belongs to the project or to each world (as moves.txt does) is the user's
  call.
- A move that only follows another: a `when` word (e.g. `chained`) that holds only while
  a move going lets it in. It would mend the finding above.
- A combo that advances by itself in the game: an assisted mode (one press plays the
  chain). It would read `combo` entries at runtime: a decision about how the game plays,
  not a Studio change.

**Checks:**
- `chartest.gd`: 86 checks, none failed, and the note above. New: the swordsman's cuts
  pressed in and out of their windows (five); the cuts in the walker's set after dash and
  before it, on the ground and in the air (three).
- `studioplay.gd`: 73 checks, none failed. New:
  - the pinned and scrolled parts;
  - Play the whole move in the same place with the numbers scrolled to the bottom, then
    pressed there by an event pushed into Godot's window;
  - a number changed: the same move played, not another; the same for a phase;
  - the edit-state line keeps its height;
  - Side by side and back;
  - the colours;
  - Asked with for the swordsman;
  - the combo's In the game words;
  - a cut added to the mannequin's set: after dash, then first by Earlier;
  - cut_c made the greet button's and discarded;
  - a `like` move refused "by name only".
- `-Tour -Only studio`: no problem, no script error. `-Tour -Only moves`: no script
  error. `-Demo studio -Check`: 8 of 8.
- `-Tour -Only ui` fails as it has since 7ax: the Studio is taller than the tour's
  720 px window. The Studio's least size measured headless: 703×734 now, against
  691×760 before this pass.

**Not run:** the whole tour.

**Nobody has used any of it by hand.**

## 7ba. Gear, buttons of one's own, moves and combos on any button, and a phase's numbers all in sight (built 2026-10-16, at the user's asking)

The user asked for six things:

- a simple way to map the controller and keyboard, with a move or a combo on any button, wanted only crouched, in the air or running, or with another button held;
- a toggle so a combo plays on one press;
- movesets and mappings saved as profiles that any project can use;
- a phase's parameters without scrolling;
- a sword that can be holstered or sheathed;
- from that, the weapons and equipment section: melee and ranged, places to carry them for any character type, accessories, shields and gloves, a grappler, hover shoes, a jetpack.

A pasted brief asked for an audit first, straightforward steps in the current architecture, and larger pieces put forward for approval. The audit, the stages after this one and the decisions are in `Draw3D_Gear_Actions_Controls_Plan.md`.

**Built** (script-checked; nothing used by hand; no real controller):

- **Buttons** (`controls.gd`):
  - The built-in table is now a default. The person's own ways (`own`, keyboard and controller kept apart) and buttons of their own (`made`) are read over it from `controls.txt` in Draw3D's folder (`%APPDATA%\Draw3D`). That folder is the same for every project, and every change is written to it at once.
  - A scripted run reads a folder of its own (`scripted-home`, or `DRAW3D_HOME`).
  - `apply` puts all of it into Godot's InputMap; `clashes` says what else a way presses.
  - New button: `sheathe` (B).
- **The Buttons tab** (`buttonstab.gd`, also Characters > Buttons):
  - a Keyboard and mouse / Controller toggle;
  - Change… or Add another… captures the next key, click or controller button (Esc leaves it); Nothing; As Draw3D has it;
  - Make a button, Remove it;
  - per button: this character's moves and combos in the order tried, with Only when, held, Earlier and Later, Take it off, Put on this button, and Plays by itself;
  - Save as a profile, Use this profile.
- **Moves** (`moveset.gd`, `motor.gd`, `move.gd`):
  - when-words `holding BUTTON`, `drawn`, `sheathed`, `with ITEM_OR_SORT` and `following`. With `following` on cut_b and cut_c, a press in cut_a's rest no longer starts cut_b from rest. Asked by its name, a `following` move may still begin from rest;
  - phase words `draws` and `sheathes`;
  - a combo with `asks BUTTON`, `when` and `plays by itself` may be in a set and is begun by its button. The motor's `combo_on` follows it: the next move comes on its button in the window, or by itself when the window opens and its wait is over, or when a move with no window to the next ends.
- **Profiles:** your buttons, plus a set's line, its moves, its combos and what they let in or are `like`, in `profiles/NAME.txt` in Draw3D's folder. Used, the buttons are read and the entries written into the world's `moves.txt` (document edits).
- **Gear:**
  - `gear.txt` and `gearbook.gd` (read with the characters) hold 21 places and 34 things of 40 sorts, made of simple parts (`model` takes a glTF file), with grip, second grip, points, worn, stows, pair, reach, weight, size, colour, from and licence.
  - A character's `gear ITEM [at PLACE] … [stowed PLACE …]` line puts one on. `animated.gd` attaches each thing to its place's bone with a `BoneAttachment3D`, moves it between hand and stowed place as the motor's `drawn` changes, and tells the motor what is carried (for `with`).
  - The swordsman carries the katana as gear, with `draw_blade` and `sheathe_blade` on the sheathe button. The cuts and the thrust want `drawn`.
  - `jet` and `hover` are walker moves that begin only with a jetpack or hover shoes worn.
- **The Gear tab** (`geartab.gd`, replacing Holds, which it shows at its foot):
  - what is worn and carried, with Where and Put away at by place name;
  - Nudge (cm) and Turn (degrees), seen at once and written;
  - Reset, Draw / put away, Take it off;
  - every thing of `gear.txt` by sort, and Put it on;
  - for a two-handed thing, how far the other hand is from its grip.
- **The Edit tab:**
  - a phase's numbers in two columns: Look on the left, and Timing and movement (the move, everyone's) on the right;
  - the pose on one row, the explanations folded under Details;
  - + Phase and − Phase beside the phases.

  Side by side, the preview keeps to its least width and someone-to-strike moves to a row of its own. Several drop-down lists no longer widen to their longest entry.

**Not built** (staged in the plan file):

- the second hand on a two-handed thing (IK, A1);
- dragging gear in the view;
- moves that suit a thing's sort, and reach and weight read by moves;
- firing (A2);
- gear in the world's inventory and slots (A3);
- gear in profiles and a shared gear library (A4);
- a stance for sheathed (A5);
- input sequences, chords and button icons.

**Checked:**

- chartest: 119 of 119;
- studioplay: 93 of 93;
- `-Tour -Only studio`: clean, pictures 19–27;
- `-Tour -Only moves`: clean;
- `-Tour -Only ui`: fails as it has since 7ax (the Studio window taller than that tour's room). The Studio's least size by tab is unchanged for the tabs that were there; the new Buttons tab is 310 px tall and the Gear tab scrolls.

## 7bb. The other hand on a thing of two hands, by Godot's own IK (built 2026-10-16, decision A1, at the user's yes)

The smallest version of A1 (`Draw3D_Gear_Actions_Controls_Plan.md`):

- **What it does.** A thing written `hands two` with a `second` grip, held in a hand place, gets one `TwoBoneIK3D` on the figure's skeleton. It brings the other arm's upper arm, forearm and hand so the palm (the hand extended 7 cm) is at a node put on the thing at its second grip.
- **When it is on:** only while the thing is drawn. A gear line's `free_hand` leaves it off.
- **In the Studio:** the Gear tab's *Other hand on its grip (IK)* writes or clears `free_hand`. For such a thing, the tab says how far the hand still is from the grip.
- **Code:** `animated.gd` `_other_hand`, `AfterIK`, `other_hand`; `charbook.gd` `free_hand`; `geartab.gd`.

**Found:**

- **A pole is required.** `TwoBoneIK3D` does nothing without one, even when the bones and target are right. The pole is a node behind and below that side's shoulder, out to its side.
- **The result can't be read outside the skeleton's update.** Godot gives back the clip's pose outside its own update, so the result is read by a small modifier after the IK (`AfterIK`).
- **The clips on disk hold things in one hand.** UAL 2's sword stance holds the right hand out at its side, and the mannequin's standing clip lets it hang. From those poses the greatsword's second grip is 0.89 m from the left shoulder and the spear's about as far, against an arm of 0.60 m. The IK brings the hand as far as the arm reaches: 28.5 cm short, with 28.4 cm out of reach.

**What would close that gap:**

- a two-handed stance: the held hand brought before the body as well (a second IK on the main arm, to a hold place);
- or clips made for two hands.

Neither is built. It is for the user to say (A1b).

**Checked:**

- **chartest, 127 of 127:**
  - a grip within reach: the palm on it, within 1.5 cm, the elbow below the shoulder and out at its side;
  - out of reach: as far as the arm goes;
  - off while it is put away;
  - `free_hand`: no IK.
- **studioplay, 95 of 95:** the box shown and ticked, unticked writes `free_hand`.
- **Studio tour:** clean; pictures 28 and 29 (the swordsman with a greatsword). At that preview size the left arm cannot be told apart: not judged by eye.

## 7bc. A two-handed stance: things of two hands held before the body, both hands by IK (built 2026-10-16, A1b, at the user's yes)

7bb brought the other hand toward a two-handed thing's second grip. From the clips on disk the grip was out of reach, because they hold things in one hand at the side.

**The smallest two-handed stance:**

- **The item.** A thing in `gear.txt` may say `two_hands_at PLACE`. Drawn, it is then put at that place before the body, on the place's bone (chest or hips), rather than in the hand.
- **The arms.** Both arms are brought by Godot's `TwoBoneIK3D` (each with a pole): the holding hand to its grip, and the other hand to its second grip. The same `_arm_ik` serves both (`animated.gd`).
- **Put away, or `free_hand`:** as before. A gear line's `free_hand` holds it in the one hand, and the Studio's box is now *Held in both hands (IK)*.

**Four new hold places:**

- `guard`: the blade up and ahead, before the chest. Used by the greatsword.
- `side_hold`: a long shaft at the right side, its point ahead. Used by the spear and the halberd.
- `shouldered`: a gun at the shoulder. Used by the crossbow, the rifle and the shotgun.
- `tool_hold`: a tool's head down and ahead. Used by the shovel and the pickaxe.
- `staff_hold`: across the chest. Used by the staff.

The rifle's and shotgun's second grips were brought nearer, so the other arm reaches them.

**Checked:**

- **chartest, 134 of 134.** Every thing of two hands on the standing mannequin, and the greatsword in the swordsman's sword stance: both hands within 0.4 cm of their grips (the swordsman's other hand 1.0 cm), every elbow below its shoulder. Put away, both arms are let go. With `free_hand`, it is in the one hand with no IK.
- **studioplay, 95 of 95.**
- **Studio tour:** clean; pictures 28–31. The mannequin holds the spear and the rifle before it with both arms; the swordsman's arms are too dark to judge at that size.

Not used by hand.

**What it is not:**

- a stance clip;
- hands turned to the grip (only their place is set);
- the thing following a move: a cut with a greatsword moves the body, but the thing stays at its hold place, because the arms follow the thing and not the clip;
- a grip correction per rig.

For the user (A1c): whether a move's phases may move the hold (a hold place per phase), or whether clips made for two hands are wanted.

## 7bd. The Character Studio after the user's hand test of 7ba (built 2026-10-16)

The user tried 7ba by hand and reported what follows. Each item is fixed, or followed up where it could not be reproduced.

**Fixed:**

- **A change that was not seen until Save.** Goes and Turns did not show in a looping preview until the user saved, left Edit and came back. Found headless: a clip chosen in the Clips tab and used for a phase ("From Clips tab") stayed what Play played, so the body never moved. Now a change of how far it goes or turns, or what it strikes, plays the whole move being edited (or its combo) in place of a clip or a phase's clips alone. A change of timing or look does so in place of a clip of the Clips tab. Using a clip for a phase makes the move what plays.
- **What is edited.** A banner above the tabs, in the colour of editing, is in sight on every tab. It says EDITING MOVE (or COMBO) and the name, which can be typed over to rename it. It holds Rename, Save, Discard, Copy and Done, and says whether it is saved.
- **Renaming a move.** A move is renamed wherever it is named: its sets, the combos' `then`, other moves' `lets` and `lands`, `like`, and every look row for it and its phases. A button's name and the words of `when` are left alone, even when spelt the same. Discard takes the rename back.
- **The moves list.** Each move is folded until its arrow is clicked; the one edited is unfolded and tinted.
- **Buttons, a click taken by mistake.** While a key was waited for, any click became the binding. The user's `run` became the left click this way (the log of the session says so, and their `controls.txt` still says `keys run click`). Now a mouse button counts only when clicked on a box shown for it ("Click here with the mouse button wanted"); a click anywhere else cancels and does what it would have done. This follows Unity's interactive rebinding, which leaves the mouse out unless asked and cancels on Esc. Every mouse button used to be shown as "click"; right and middle clicks are now said as such.
- **Buttons named.** As in Unreal's Player Mappable Key Settings (a fixed name and a display name), one of Draw3D's buttons can be given a word it is shown by (`label run jog` in `controls.txt`); the game still asks for `run`. A button of one's own is renamed, and so are the moves and combos asked or held with it.
- **Buttons listed by the character.** In order: *Asked by NAME's moves*, *Your own buttons*, *Always (moving about, the world)*: run, crouch, act, sheathe, out, use. Then *Not used by NAME* (folded), The eye, At a puzzle, The game. Labels such as run and dash no longer stand at the top for a character that has no such move.
- **Profiles.** The profile in use is shown at the top of the Buttons tab and remembered in `controls.txt` (`profile NAME`). **Save** writes over it. **Save as new** wants a name; it used to fall back to the character's name, which is how "swordsman" came to be written beside the user's First_Profile. **Use** makes a profile the one in use.
- **Gear, one thing in hand.** Every held thing went into the right hand: the staff put on the swordsman was drawn into the same hand as the katana, short sword and dagger, and put away flat along the back where the front view cannot see it. A gear line may now say `spare`: carried, not drawn, at its stow place, its points not the striking ones. Putting on a held thing makes it the one in hand, the others spare, and draws it so it is seen. **The one in hand** makes the thing chosen the one drawn.
- **Gear, no list in a list.** What it wears is a list as tall as its rows. Every thing of gear.txt is shown in columns as tall as they are (double-click to put one on, ✓ where it has it already).
- **Room.** Above the tabs, the preview keeps to its least height and the tabs have the rest; the line between them can be dragged. The two status lines are one. The banner is one row. Where the screen is at least 1400 px wide, the Studio opens side by side.

**Followed up, not reproduced: the freeze on the Buttons tab.** In the session where the Studio stopped taking clicks (log kept in `editor/godot/logs/20261016-111706-run.log`), keys still worked and the editor closed with code 0. That points to clicks being taken or blocked, not to a hang. Headless, the user's steps (new move, a clip used, loop, Goes changed, the Buttons tab, back to Moves) ran at normal frame times. The shutdown errors the user pasted (`_free_rids`, `PagedAllocator`, shaders never freed) are what every run prints on closing, including the scripts' runs; they are not the freeze. What is done now:

- the click that became `run` can no longer happen;
- the run log now says each tab shown, when a button is waited for and when not, and when the "Changes not saved" dialog opens (it blocks every click until answered).

**Checked:**

- **studioplay, 114 of 114**, 19 of them new. Among them: the user's steps (Goes seen at once, the body 1.5 m+ forward); the moves folded; the banner on every tab; a new move renamed and the rename discarded; cut_c renamed, its combo, `lets` and look rows following, then discarded to the very lines; a click on a tab while waiting changing nothing; a right click on the box taken; heavy renamed with its combo; run shown as jog and back; Save over the profile in use with no second file; the staff in hand and drawn with the katana at its hip; the katana in hand again.
- **chartest, 134 of 134.**
- **`-Tour -Only studio`:** no problems.
- **`-Tour -Only ui`:** fails on the Studio's window as before. Run on the committed code (7f5b4b8) from a scratch copy, it fails the same at 723×783; now it is 723×760.

Not used by hand.

## 7be. The Studio's second hand test: drawn by the cut, a phase before, a weapon's own moves, lock-on, the yard, a cape (built 2026-10-16)

The user tried 7bd by hand and asked for what follows ("everything else, I think you can start working on"). All of it is optional per game: nothing is on unless a set, a move, a thing or a line says so.

**Built:**

- **Drawn by the cut.** With the sword put away, cut_a and cut_b played nothing, because a cut wants it drawn. A set may now say `draws_by MOVE`: a move (or a combo) asked that wants what it carries drawn, while it is put away, begins that move first, and the one asked as soon as it is over (`motor.gd` `_draw_first`, `_after_drawn`). This is the same for any weapon a set draws. It is set in the Studio's Moves tab (*Draws by itself, with*). The swordsman's set has it in the project's moves.txt; a world's own moves.txt gets it from the box.
- **A phase before.** Edit's phases: **+ Before** puts a new phase before the one chosen (a draw before the wind-up); **+ After** as before.
- **What plays follows the tab in front.** The strip under the preview showed the last move's phases while a way of moving played. Now it shows the clip of a clip or a way, and the move's phases for a move. Each list's tab remembers what was last chosen on it, and showing that tab again shows it again, chosen and held (`_last_of`, `_tab_shown`). The Edit tab shows what is edited. While an edit is open and something else plays, the banner is dimmed and the line under the preview says "not the move edited".
- **The moves list.** The row of the move edited says *✎ editing* beside its name (the grey the user saw was its tint).
- **Clips sorted** by name whatever the capitals (the list was case-sensitive: *Rise* after every lower-case name), by library, or by seconds; a column's title sorts by it. The Clips tab says where a clip's use is changed (Edit, a phase; Ways of moving) and that the clip's file is never changed: keyframes of one's own are not built (`Draw3D_Character_Maker_Plan.md`).
- **A weapon's own moves and combos.** A new when-word `wields W`: the thing in the hand (or the one to be drawn into it) is W, or of that sort. So a sword's cut and a spear's thrust are asked with the same button and the one for what is in the hand begins. In Edit, beside the button: *anything in hand / a blade / a spear / …* and the thing's names; for a combo, *Only with in hand* under its moves. Every list of its `when` is written so. The Moves list says "spear in hand". Saving a combo kept only its `then` lines (its button and `when` were lost): kept now.
- **A weapon changed in a combo.** A phase may say `takes W`: as it begins, that thing it carries (or the first of that sort) is drawn into the hand and the one there put away (`motor.wield`, shown by `animated.gd` `_spare`). In Edit, *In hand: as it is / draws it / puts it away / takes NAME*. Three cuts and a hammer's blow: the hammer carried as gear, the third move's first phase `takes war_hammer`.
- **Lock-on.** A new button `lock` (Y, or the middle click; a controller has none until given one), and a when-word `locked`. Pressed, it locks on to the one nearest before the eye within 15 m (`lockon.gd`: anyone with a body, `body.gd` puts them in the group `lockable`; a target on a post). An orange mark is shown over them, the body faces them while going about, and the eye comes round behind toward them. Pressed again it lets go, as does going 22 m away. In the Buttons tab, under Always: *lock*, and *Lock on only while the lock button is held* (`lock held` in `controls.txt`). In Edit, *locked on* beside a move's button and a combo's hand: a backflip only in a fight.
- **The training yard.** **Yard**, under the preview, opens the Studio's preview as a place to fight in. The character chosen is steered by the person with the game's own buttons, and the eye follows. Who stands there is `yard.txt` (`yardbook.gd`; kept with the world as the other texts are, F10): `dummy NAME [as CHARACTER] at X Z [faces DEG] [moves SET] [does MOVE every S]`, `board NAME at X Z`. As written: two posts thrown back when struck, one that jumps every 2.5 s, one that cuts every 3 s with a sword, a target. Struck and moved, a dummy walks back to its place. What is changed in the Studio meanwhile is what the character does at the next press. Esc or Yard again leaves; while it is open the keys are not the editor view's (`main.gd` `keys_taken`). A script leads it by `yard.led`/`lead`, as a demo must.
- **A cape that swings.** A thing of `gear.txt` may say `sways [swings S] [damp D] [within DEG] [wind W]`: it hangs from where it is worn and comes after the body on a spring (`sway.gd`, as a kind's sway bones), never into the body. When not said, it is stiff. The cape says it. In the Gear tab: *Cloth: swings after the body*.

**Planned, not built** (`Draw3D_Character_Maker_Plan.md`): making a character (tier 1, proportions, colours, outfits, from what there is; tier 2, face, eyes, hair and expressions, needs a choice of assets: the user's); enemy types and reactions by how a blow came; creatures not on two legs; real cloth; the resizable split for other crowded windows.

**Checked:**

- studioplay, 137 of 137 (23 new: drawn first and not; + Before; a spear only, locked on, and back; a combo for a blade kept with its moves; the tab in front; clips by name and by seconds; the yard opened, locked on to the post, walked up to and cut, let go, left; the cape swinging back; the staff taken into the hand). One older check, *Reset: no nudge written*, failed in one of the five runs after the lists were mended and passed in the other four (and in the one run of the committed code); see Known gaps.
- chartest, 135 of 135 (the swordsman's sheathe checks changed: put away, the dash button now draws and cuts; without `draws_by`, as before).
- `-Tour -Only studio`: no problems.
- `-Tour -Only ui`: the Studio's window, as before (723×760).
- `-Demo yard -Check`: 5 beats of 5, the post struck 3 times.

Not used by hand.

## 7bf. Making a character, tier 1: the body and the clothes, with what is there (built 2026-10-16)

Tier 1 of `Draw3D_Character_Maker_Plan.md`, asked for after 7be ("character creation as the next step"). Nothing new was downloaded: it works on the models already in the project. The Character Studio has a new tab, **Make** (`maketab.gd`), after Gear. Every choice in it is a line of the character in characters.txt, written as a document edit: Ctrl+Z takes it back, Ctrl+S keeps it with the world, and a game reads the same lines.

**Built:**

- **Start from.** A list of the models there are: every character's model, and every library file with a mesh skinned to its bones (the mannequin, Quaternius's two, Draw3D's basic and samurai figures). *Use this model* writes the character's `model` and `rig`; *Copy with this model* makes a new character first (Copy, as before).
- **Proportions by bone.** `shape PART S`: PART is `head`, `build` (the chest and the hips), `arms` (upper arms and forearms), `legs` (thighs and shins), a name of Godot's humanoid profile, or a bone of its own; several multiply. Sliders: Height (`size`, the whole), Build, Head size, Arm length, Leg length. Seen on the figure as a slider is dragged, written 0.35 s after it stops, as one edit. `animated.gd` makes each bone named S times as big after the clips have posed it (`_find_shape`, `_apply_shape`, `reshape`), and gives what hangs from it its own size back: a longer arm ends in an ordinary hand, a bigger head does not need a bigger neck. Longer legs raise the hips by what they gained and lengthen every clip's stride as much (`leg_scale`); the points and treads are on the bones and follow them. *As the model is* takes it all out.
- **Colours.** `colour SURFACE r g b`: a material by its name, a surface by its number, or `all`, tinted (`tint`); the gear and what it holds keep their own colours. The mannequin has one surface; Quaternius's models have two (M_Main, M_Joints).
- **Outfits.** `outfit NAME` begins an outfit; the gear lines after it are its own, and any other word ends it. Gear lines written before the first outfit are worn in every outfit. The first outfit is what the character starts in. A figure wears `worn_gear()`; `wear_outfit(NAME)` takes the gear off and puts that outfit's on without making the figure again (what a game will call to give another outfit by name). In the tab: the outfits; *Next outfit*, on the figure only, while what plays goes on (a move, a combo, the yard); *New outfit*, *Copy this one*, Rename, *Starts in this*, *Remove it*. The Gear tab puts on into the outfit worn, says which things are in every outfit, and writes the blocks back in order (`CharBook.entry_with_gear`).

**Not built:** faces, eyes, hair and expressions (tier 2: the models have no face to move; the choice of assets is the user's, in the plan); a word or a button for a game to change outfits (`wear_outfit` is there to be called); a bone longer without being thicker (Godot scales a bone whole).

**Checked:**

- chartest, 148 of 148 (11 new): the words read and written back; the head 1.3 times, the hand 1.0; on legs 1.2 the hips 0.14 m higher with the heels on the ground, the walk's stride 1.2 times; a tint; an outfit put on and the first again; cut_a with the feet above the ground; Quaternius's joints alone tinted.
- studioplay, 152 of 153 (17 new, all passing): a slider seen at once and written after; each part a line; cut_a made so; a tint and No tint; As the model is; two outfits filled from the Gear tab; Next outfit while cut_a plays (not begun again) and in the yard; renamed, first, removed; Copy with Quaternius's model, its joints tinted, its legs longer. The older *Reset: no nudge written* (Gear) failed in 3 of 4 runs, once with the committed code: not this pass's (PROGRESS Known gaps, with a likely cause).
- `-Tour -Only studio`: no problems.
- `-Tour -Only ui`: the Studio's window as before (least 723×760, its known failure; the new tab did not widen it).
- `-Demo make -Check`: 6 beats of 6, pictures looked at.

Not used by hand.

## 7bg. The Studio's third hand test (built 2026-10-16)

The user tried 7be and 7bf by hand and asked that the yard's controller and arrow keys leave the Studio's buttons alone, that the right stick turn the eye there, why the sword was not drawn by the cut, that profiles show what they hold, for clearer phases in Edit, for versions of a move, for Make beside the character, a rename, a reset for the body, and for the sword not to go into the forearm on another model ("don't make it too complicated").

**Why the sword was not drawn by the cut.** The user's world (showcase-physics.json) keeps its own moves.txt, written before `draws_by`: the box *Draws by itself* was off, but the move beside it showed draw_blade, so it looked set. Their two profiles also each keep an old copy of the swordsman's set, and *Use* wrote that copy over the world's moves. And the Studio's Save writes into the world, which is on disk only after Ctrl+S: the world's file was last written at 11:17 that day. Changed:

- **Draws it by itself first, with:** the move beside it greyed while the box is off.
- **Why a move did not begin** said under the preview and in the editor's message line: "cut_a wants what it carries drawn (it is put away): tick Moves > Draws it by itself first, or play the move that draws it before" (`motor.gd` `why_not`, every when-word in plain words).
- **Profiles:** one chosen in the list says how its buttons differ from yours, and how many of its moves differ from the world's, before it is used. *Use* takes its buttons only; *and its moves*, ticked, also writes its moves (it was always both).
- **Save world** on the character's row: how many changes are not in the world's file, and a button that writes it (Ctrl+S).

**The yard.** The pad, the sticks and the arrow keys no longer go from one of the Studio's buttons to the next (nothing keeps the focus when the yard opens; `studio.gd` `_input` keeps keys and the pad from the Studio's controls while it is open; the mouse clicks them as before). The right stick turns the eye round the character as in the game (`yard.gd` `LOOK`); the right mouse button still does.

**Edit.** The phases are chips numbered in the phases' own colour (red where it strikes), the one chosen outlined; the chosen phase's heading larger, in the same colour. Done rebuilds the moves list, so the row no longer says it is edited; while it is open the row says *open in Edit, saved* or *changes not saved*. **Versions:** each Save keeps a version of the move, its text and the clips its phases are shown by (the first Save keeps *as it was* too), in `versions.txt`, kept with the world. *Versions (N)* beside Done: one chosen is gone back to as a change (Save keeps it, Discard takes it back); *Forget a version*.

**Make, on the character's row.** *Make…* beside Copy shows the Make panel in place of the tabs (it is no longer a tab among Moves and Gear); pressed again, or another tab asked for, the tabs again. **Rename** beside Copy: a field, Enter renames the character (a yard dummy shown as it too). Each body slider has **↺** beside it (that line taken out); *Reset the whole body* (was *As the model is*).

**The sword in the forearm.** A place in a hand is in the hand bone's own frame, and bones are not turned alike from model to model: the mannequin's hand has its y up the arm, UAL2's down to the fingers, so the katana pointed back into the forearm on UAL2's model. A place in a hand (or on a forearm) is now taken as the mannequin's hand has it and turned for another model by how its bone lies at rest, along the arm and with the body's ahead (`gearbook.gd` `hand_turn`, `along_arm`); the mannequin's own is not turned. Measured: on UAL2's model the blade points within 1° of where it points on the mannequin, standing and in the middle of cut_a (before, straight back). For what is still off on some model: Gear tab, the thing chosen while drawn, *Turn round*, *Roll 90°*, *Tip 90°* beside the turn numbers.

**Checked:**

- studioplay, 171 of 171 (20 new: the yard's focus, the pad's Down and the arrows, the right stick; Draws it by itself greyed and why cut_a did not begin; the phases' chips; versions kept, gone back to, forgotten; the row while open and after Done; Make on the row; a copy renamed; a slider's ↺; Save world said; a profile's differences);
- chartest, 151 of 151 (1 new: the katana on UAL2's model within 1° of the mannequin's, standing and mid-cut);
- `-Tour -Only studio`: no problems, no script errors. `-Only ui`: only its known Studio-window problem (in_the_room, resizes), no script errors.

Not used by hand.

## 7bh. The Studio's fourth hand test (built 2026-10-16)

The user tried 7bg by hand: they never saw the box *Draws it by itself first* (unticked boxes as dark as the window); the yard's eye turned the wrong way left and right, perhaps up and down; the grappler put on took the katana's place, and Draw then drew the grappler; a fit found for a model by nudging should be there for the next project; and, for later, a weapon designer, clothes that look like things, fingers round the grip, faces and expressions.

**Every box seen.** A box not ticked was as dark as the window: the user never saw *Draws it by itself first* in the last test. Every unticked box and round choice in the editor is now drawn light (`section.gd` `boxes_seen`, into Godot's default theme at start; greyer while it cannot be used).

**The yard's right stick turned the eye the wrong way** on both axes: it had copied the mouse's drag (the scene follows the hand) where the game's eye turns the way the stick is pushed. It now turns as the game's (`yard.gd`). Buttons tab: **Right stick: mirror left/right** and **mirror up/down**, for the game and the yard alike, kept with the buttons (`controls.txt` `mirror across`, `mirror updown`; `Controls.look`).

**What is drawn and put away.** Every thing held in a hand was drawn and put away one at a time: the grappler put on became the one in hand, the katana went to the hip, and Draw and Sheathe then moved the grappler. Gear tab, a thing chosen: **Carried**: *Drawn and put away, one at a time* (as before), *Drawn and put away with the one in hand* (`together`: two blades, a sword and a shield), or *Always where it is (never drawn or put away)* (`stays`: the grappler on the left wrist; still carried, so a move may want it). *The one in hand* leaves those as they are. The Moves tab says beside *Draws it by itself first* what drawing and putting away act on ("draws and puts away: katana").

**How a thing sits on a model, kept for every world.** Gear tab: **Keep for this model** keeps the chosen thing's nudge and turn at its place on the character's model in `fits.txt` in Draw3D's own folder (beside `controls.txt`); put on a character of that model in any world, it sits so; a place changed takes that place's fit; **Model's fit** puts it back after nudging.

**Asked for later, not built** (`Draw3D_Character_Maker_Plan.md`, "Gear and clothes that look like things"): a weapon designer (blade length and breadth, guard, handle, two blades; a shield's shape and size), clothes that bend with the body (grown from the body's mesh, or a pack's modular clothes), fingers closed on a grip; faces and expressions remain Tier 2 there.

**Checked:**

- studioplay, 180 of 180 (9 new: a box not ticked drawn light; the stick pushed right looks right, and mirrored the other way, kept in controls.txt; the grappler on the left hand that stays, the katana in hand and put away at the hip with the grappler still on the hand; what Draw acts on said; the grappler carried with the katana; a fit kept and a copy given the katana again sitting as kept);
- chartest, 151 of 151;
- `-Tour -Only studio`: no problems, no script errors.

Not used by hand.

## 7bi. Items, equipment and the game's own screens (built 2026-10-16, roadmap section 7)

The user chose roadmap section 7 next (items, equipment and the game's own screens), after parking the characters. What was there: things that say `kept` (stage 6: a lantern, a hatchet; keys 1 to 9, R uses), goods as counts (7ah), and gear worn by characters in the Character Studio (7ba onward). Three systems that did not meet. This joins them; nothing of them is rewritten.

**Items, each its own.** An item is a thing put down in the world that says `kept`: every fact about it is about its id, so two axes found are two axes, each in a saved game where it is. New kind word `gear ITEM` (host `shapes/kind.rs`, Godot `things.gd`): the design is that thing of gear.txt. Facts: `where` is now "kept", "held", **"equipped"** or where it was **dropped**; the world's `wields` (which equipped thing is the one in hand) and `off.ITEM` (one of the character's own gear lines taken off). `items.gd` is the one place they are put and read.

**Gear found and equipped in the world.** Library > **Gear to find** (`editor/library/gear.txt`, 18 designs: katana, short sword, dagger, greatsword, spear, axe, war hammer, bow, grappler, round shield, hat, helmet, boots, gloves, cape, backpack, pouch, amulet): each lies on a scrap of cloth and is drawn as gear.txt makes it, laid flat, while editing and playing; the Library window pictures it so too (`design_made` now answers its `traits`). E picks it up (the act `take`); kept, keys 1 to 9 or the inventory put it on or take it in hand. On a walker that is a character of the Studio (`walks`, Studio > Walk as this), what is equipped is laid over its own gear lines (`animated.gd` `game_on`, `game_off`, `game_hand`, `regear`): drawn and put away as the Studio has it, at the places gear.txt says, with the fit kept for the model (`fits.txt`). A thing taken in hand puts the character's own away (spare). A shield goes on the left arm, drawn and put away with the one in hand; a grappler on the left arm, always (`items.gd` `OTHER_HAND`).

**The game's own screens** (`gameui.gd`, a layer apart from the editor's windows, the same in Play and in a game made of the world):
- **The display while playing**: the quest step and goods top right; what was found, numbered as keys 1 to 9 take it, the one in hand gold and what is worn green; what is in hand and whether it is drawn; "I inventory, Esc menu".
- **Inventory (I, or Back on a controller)**: On you, In the pack, a row for each thing; chosen, what it is, where it goes, found how-manieth; Put on, Take in hand, Take off, Use, Drop, Close; goods under it. The game is held while it is open.
- **Menu (Esc, or Start)**: the game held; Go on, Inventory, Journal, Controls, Save the game, Load the saved game, Back to editing (in a game: Leave the game).
- Inside them, Godot's own ui_ actions: arrows or the D-pad, Enter or A, Esc or B.

**What changed for the person:** Esc while playing is the menu, not back to editing (Tab still is; the bar says "Stop: back to editing (Tab)"). Start on a controller is the menu, no longer Play. Back on a controller is the inventory, no longer the controls card (H, and the menu's Controls, show it). The old bottom-left line of what is kept is shown by the new display instead (its words are still made, for scripts).

**The swordsman can do what anyone does with things**: its set gained `wave reach stoop` (UAL's PickUp_Table and Interact clips, `look swordsman`); walked as, it could not pick anything up or open a door before.

**Checked:**

- `-Tour -Only inventory` (new): two axes, a helmet and a round shield put down from the library, the walker as the swordsman; each found and kept with its own id; the axe in hand by key 1 with the katana spare, the helmet on the head bone, the shield on the left forearm and not the one in hand; the inventory holding the game, an axe taken off and back; Esc shutting it, Esc opening the menu, saved from it; the other axe dropped 0.7 m before the walker and drawn there; changed and loaded: as saved; stopped: no facts, nothing given to the figure. No problems, no script errors.
- `-Demo inventory -Check`: 8 beats of 8, four things kept, no script errors; its pictures looked at.
- `-Tour -Only studio`, `-Only findings`: no problems. studioplay 180 of 180, chartest 151 of 151; studioaudit of the swordsman: wave, reach, stoop ready, each with a clip.
- Host: `p43` (the library) passes with Gear to find in it (its designs are pictured by Godot, so the two-parts and height checks skip that collection); the whole suite single-threaded: 60 and 65 pass.
- `-Only items` and `-Only library` fail as the committed code does (run from `git archive HEAD`): the walker picks up the hatchet before the lantern, and the library's click on the basketball hoop lands on another shelf. Not this pass's; recorded under Known gaps.

Not used by hand; no real controller.

**Next in section 7, not built:** an item's own numbers and wear; slots and a pack with room; others who keep and equip things (folk, enemies) and trading items as goods are traded; containers (a chest that keeps things); equipment that changes what moves can be done beyond `when with`; a game's own menus written as content (titles, options) and remapping inside the game; a HUD laid out by the game. Section 9 (health, damage, defeat) would add health to the display.

## 7bj. The animator pass: what is laid over the clips (built 2026-10-16)

The user linked ActionForge (actionforge.app), asked what it does well and what Draw3D could take from it, and then said: "go all the way to b 7, whatever order lowers the cost" (`Draw3D_ActionForge_Analysis.md`, items A1–A3 and B4–B7). ActionForge's code is not free; its ideas were built of Godot's own parts. Its CC0 hand poses and heavy sword combo (Quaternius's rig, the same as UAL 2's) are in `characters/actionforge/clips/` as the library `actionforge`; its props in `.../props/`.

**The cause of the open hand on the sword:** no map named a finger bone, so no clip ever moved one. The maps `ual`, `ual2` and `mixamo` name the thirty finger bones now (Godot's profile names: LeftThumbMetacarpal … RightLittleDistal). UAL's and UAL 2's clips close and open the hands as they were made to. The mannequin has no finger bones and shows none of this.

**One place for everything laid over the clips** (`overlay.gd`, owned by `animated.gd`; after the clips play, before the body is drawn; the clip's file never changed). In order:
- **Bearing** (A2): `bearing NAME DEGREES` on a character: posture, chest_turn, head_up, head_turn, head_tilt, shoulders, arms_out, arms_ahead. The arms' only over the plain ways (a move says where its arms go).
- **Gait** (B5): `gait NAME VALUE`: stride (thighs and shins swung ×, from how the standing clip holds them), cadence (steps quicker ×), bounce (the hips' rise and fall ×), sway (the hips rolled, degrees), arms (swung ×), lean (into speed, degrees); over the plain ways only. Styles ready made in the Make tab: stroll, brisk, heavy, sneak, proud, tired.
- **A clip over a region** (B4): a look row's `over CLIP REGION` (arms, left_arm, right_arm, upper, head, spine, legs, left_leg, right_leg, left_fingers, right_fingers, fingers): that part plays the other clip round and round while the row is shown, faded in and out with it. And gear.txt's `arm CLIP [left|right]`: the arm that holds the thing plays the clip over the stand and the walk; which of the clip's arms holds it is said, and in the other hand the clip is played the other way about (measured: Idle_Torch_Loop holds its torch up in the left hand, Idle_Lantern_Loop its lantern in the right, Idle_Shield_Loop the left). New things: `torch` (`arm Idle_Torch_Loop left`) and `lantern` (`arm Idle_Lantern_Loop`); the round shield `arm Idle_Shield_Loop left`. A layer lays the joints' own turns: the hand ends where the clip's would on the body's own chest, not the clip's (a hunched stance holds a torch lower).
- **Hand poses** (A1): gear.txt's `hand POSE` (and a gear line's own `hand POSE`): the fingers of the hand that holds it as a clip of one moment has them (fist, open_palm, open_palm_natural) or as a `pose` of characters.txt bends them, while it is drawn; let go when it is put away; both hands on a thing held in two. Every held thing of gear.txt says `hand fist` now.
- **Follow-through** (B5): `follow PART AMOUNT [feel stiff|loose|heavy]` (head, chest, arms, hands): each a weight on a spring (`sway.gd`, as a cape swings), the bone turned toward where it has swung.
- **Feet on the ground** (B6): on the ground, each foot brought onto the ground found under it by a ray (Godot's TwoBoneIK3D, the knee to the front), the body let down to the lower; a foot lifted in the stride brought less. On by itself; `feet off` says not. The Studio's floor is flat: seen in the world.

**Poses of one's own and keys** (B7): `pose NAME` entries in characters.txt (lines `BONE x y z [local]`); a bend may be `local` (about the part's own axes) and keyed `at P` (0 to 1 through the row's part: between keys of one part it goes from one to the next). A finger may be bent as a whole ("RightIndex": its three bones).

**The Studio:**
- **Edit, Adjust pose** (B7): the fingers in the list; a click on a joint in the preview chooses it. Under Details, at the top: its own axes, key at the playhead, both sides; Copy pose, Paste, Paste flipped, Mirror L→R, R→L; Poses (Lay it, Keep as a pose); Over it (a region, Lay the clip chosen over it, Take off). In Details so a phase's numbers stay in sight.
- **Ways of moving**: Over it (a region, Lay the clip chosen over it, Take off) for the row chosen.
- **Clips** (A3): a button for each of the words most clips are found by; a clip played as the pointer goes over its row ("play as the pointer goes over", on); a Leads to column (a start to its loop, a loop to its end, a cut to its recovery, by their names).
- **Make**: Bearing (eight sliders), Gait (a style and six sliders), Follow-through (four sliders, how it feels), Feet on the ground; each seen at once, written a moment after as one edit; Reset how it carries itself.

**Checked:**
- chartest.gd 177 of 177 (26 new): the new words read and a look row written again as it was; keyed bends halfway; UAL 2's fingers moved by the clip (left index 14° from rest); `hand fist` (index tip 0.116 m from the palm, open_palm 0.197); the free hand as the clip has it; put away, the fist and the open palm the same again; a pose of one's own as a hand (70.0°); `over Idle_Torch_Loop left_arm` (its joints 0.00° off the clip's, the hand 0.12 m up over Sword_Idle's stance, the right not moved); the torch in the right hand (its joints 0.00° off the clip mirrored, 0.23 m up); bearing posture -20 (head 0.13 m ahead and lower), arms_out 30; gait stride 1.5 and cadence 1.3; follow head 0.9; on a 25° slope the heels on the ground (-0.002, -0.004 m), with `feet off` not (0.122, -0.057). The three map checks now count 52 bones.
- studioplay.gd 197 of 197 (17 new): the pose editor's rows written (own axes, both sides, a key), copy and paste flipped, mirror, a pose kept and laid, over on a phase and a way, a click on the head in the preview, Discard back to as it was; Leads to, the word buttons, a clip played as a pointer's move goes over its row; the Make tab's lines and Reset. Once in four runs the old check "its surfaces (M_Main, M_Joints): the joints tinted" failed and passed on the others (Known gaps).
- `-Tour -Only studio`: no problems, no script errors.
- `-Demo animator -Check` (new, 9 beats, no script errors), its pictures looked at. The first run showed the torch hanging at the hip with the arm up: Idle_Torch_Loop holds its torch in the left hand (hence `arm CLIP left` and the mirror), and the Studio tries a way again with what is carried put away (the demo draws it after, as Put it on does). A key of no turn was not kept; it is now (a key at 0 says "as the clip has it" there).

Not used by hand; feet on the ground not yet seen on the island's own slopes by a person.

**Not built (C, by the analysis; each needs the user's approval):** rigging an unrigged model in the engine; eight directions made from one walk; a shot camera per animation. Nor B8 (Unreal and Reallusion import maps). Feet are not turned to the slope (only brought to its height).

## 7bk. The Studio after ActionForge, and the fifth hand test (built 2026-10-16)

The user tried the animator pass by hand the same day and found: the walk given to the swordsman in Ways of moving gone at every start, though the world was saved; a tiptoe for a moment before a walk began; the Over it controls not understood; the hand not at the sheath when the blade is drawn or put back, and nowhere to change it; the moves' names (reach, vault, slash) not saying what they are; few clips for sheathing; and the Studio not looking any different for ActionForge. They said: "build that pass: the save fix, start/stop following the walk, hands to the sheath, plain wording, and the Studio redone after ActionForge. Use the Studio's edits written to the project file for the save." On the larger items: no rigging of unrigged models (importing will do); the shot camera approved; eight directions to be explained before it is built.

**Why the walk was gone:** the editor gives a world the project's files (characters.txt, moves.txt, gear.txt…) when it opens it (`main.gd settle_written`: every file is "saved" the first time it is looked at), so the world's own copy, with the Studio's edits in it, was written over at each start. The world file kept the edit (`walk Walk_Loop at 2.4` was in showcase-physics.json); the start threw it away. **Mended as the user chose:** a text written in the editor (the Studio, the Gear tab, F10) is written into its project file too (`main.gd keeps_texts`, `write_entry`/`write_text` → `keep_file`), never by a script (a tour, a demo, studioplay), nor in a demo's world once handed over. The swordsman's walk the user had set was copied into characters.txt by hand. Two other differences between the user's world and the project were left alone and reported (the samurai look's walk, the swordsman set without wave/reach/stoop).

**The tiptoe:** the motor's `start` row (0.3 s as it sets off) was the samurai's `samurai_walk`, inherited by the swordsman's look, and the walk chosen took over only after it. A walk given in the Studio now gives `start` the same clip (fade 0.2), and a stand gives `stop` its clip (fade 0.25), unless that row was chosen apart (`studio.gd use_clip`, `_goes_with`). The swordsman's look says `start Walk_Loop fade 0.2` and `stop Sword_Idle fade 0.25` now.

**The hand to the sheath:** a look row's `hand_to_stow FROM TO [nudge X Y Z]`: through the phase the arm that holds the thing in hand is brought by Godot's TwoBoneIK3D to where that thing is put away (a node kept at the stow place, as the thing is put there), FROM of the way as the phase begins and TO as it ends (eased), the spot moved X right, Y up, Z ahead of the body (metres). The swordsman's draw_blade and sheathe_blade say it (reach 0→1, then 1→0). In the Studio's Edit, beside In hand: "hand to where it's kept", the % at the phase's beginning and end, and the spot moved by centimetres; a yellow ball in the preview is the spot.

**Plain words:** a move `says` what it is (moves.txt, all 38 moves written), shown in Moves as "What it is" with its length. The tabs are titled Moves, Combos, Edit a move, Buttons, What it carries, Walk, run, stand, Clips (list), When struck, Bones (their nodes keep their names). Over it is "One part plays another clip: [part] Play the chosen clip on it / Stop it" with an example in its tip; the pose tools "turn about the part itself", "only at this moment", "both sides at once"; Details is "More posing tools, and what it all means".

**The Studio laid out after ActionForge:** side by side on a workspace at least 1500 wide, the clip library at the left (`cliplibrary.gd`): every clip as a picture of the body in it (three moments, the last bright), drawn from a second body never shown, a few each frame; found by a word or by name; the pointer over a picture plays it; "For the walk/run/stand row" and "For the phase edited" give it. The preview and its timeline in the middle, the tools at the right. The window made 310 wider for it.

**Gait mended on the way:** stride was made bigger about the standing clip's pose, so a walk under a wide sword stance was pushed further from its middle, not swung further (chartest's gait check failed once the swordsman walked by Walk_Loop). It is made bigger about the walk's own middle now (its mean turn over the clip, `overlay.gd _middle`).

**Clips for sheathing:** none found that are free for sold games (CC0 or CC-BY); Quaternius's and Kenney's sets have none. Mixamo has several (sword sheathe and draw); the user downloads them with their own account and Import clips… reads the FBX.

**Checked:** chartest 182 of 182 (5 new: hand_to_stow read and written; the hand at the spot at the reach's end, within 9 cm, not at its beginning; a nudge of 10 cm up; off without the word); studioplay 209 of 209 (12 new: the tabs' titles; reach says what it is; the library's pictures each its own and found by a word; a picture clicked; given to the walk with its start; a start chosen apart kept; the Edit row ticked for draw_blade, the spot moved and the ball shown, unticked; the texts as they were); `-Demo studiolayout -Check` (new, 5 beats). Not used by hand; a restart of the editor with a walk chosen not tried by a script (scripts never write the project's files).

## 7bl. The sixth hand test: the library folded, the blade as the sheath lies, eight ways (built 2026-10-17)

The user tried the Studio after ActionForge by hand (2026-10-17) and asked: "the clip library should be collapsible to the side to make more room for the view port"; "there doesn't appear to be a way to manipulate the orientation of the sword when drawing or sheathing. the sword is pointed away from character and then magically sheaths in the opposite direction. being able to change the orientation along the animation to move closer to the sheath position and matching the reverse for draw is something needed. similar to the way you can adjust the grip"; and "sort out and also work on the eight directions" (explained in 7bk, now approved). Before it, their own tuning of sheathe_blade in the Studio was committed as it was written (5280088).

**The library folded:** the « at the library's head folds it to a strip 30 wide at the left (» Clips opens it); the preview is made wider by what it gave (`studio.gd library_folded`). Beside the tools on a workspace narrower than 1500 it starts folded, where before it was not shown at all. Its pictures are drawn only while it is open.

**The blade as the sheath lies:** a look row's `turn_to_stow FROM TO [turn X Y Z]`: the thing in the hand turned from as the grip holds it toward as it lies where it is put away (its place, as it would be put there), FROM of the way as the phase begins and TO as it ends (eased), and its grip point brought there as much; `turn` turns it more about its own axes, the most at the middle of the phase (to clear the hip), none at either end (`animated.gd _turn_to_stow`). A phase that `sheathes` puts the blade away as it begins, so the reach before it ends with the blade exactly as the sheath has it, and nothing jumps. The swordsman's sheathe_blade/reach says `turn_to_stow 0 1` and draw_blade/out `turn_to_stow 1 0` (the user's own changes to those rows kept). In Edit, In hand: "blade as it lies there", and under it the % at the phase's beginning and end and "turn (°)" x, y, z.

**Eight ways:** going one way while facing another (locked on, the yard's lock, a script's `face`). The look may name clips: `walk_back`, `walk_left`, `walk_right`, `run_back`, `run_left`, `run_right` (a run aside with none of its own is the walk aside). The animation tree has the four of each gait, blended by how far aside it goes (`animated.gd _make_tree`, `_show_ways`). With none named: back is the walk played backwards; aside, the walk (or, beyond a quarter round, the walk backwards) with the legs turned from the hips to the way it goes and the spine turned back, half in Spine and half in Chest, so the chest stays on the target (`_turn_legs`); never more than the gait's `legs_turn` (90° unless the Make tab's "Legs turned" says else). Going back and forth past a quarter round is smoothed over 0.2 s, and only past 100° or 80°, so it does not flicker at the side. Setting off and stopping are shown only going the way it faces. Walk, run, stand lists the six rows ("the walk, its legs turned", "the walk, played backwards" until a clip is chosen) and has "All eight ways": it walks round every way, a second and a half each, facing ahead.

**Not done:** none of the bundled libraries has a sideways or backwards walk (looked for: Quaternius's two, ActionForge's, the samurai's); Mixamo's strafes, downloaded by the user, can be imported and chosen for the rows. Turning from walking back into walking ahead skids as the motor always has (130° and more). Two of studioplay's checks failed now and then on this pass and passed in other runs: the tint check (its surfaces, M_Joints: two runs of seven; the committed code passed it in its one run) and 7bk's yellow ball hidden once unticked (two of seven). Each read the Studio 2 or 3 frames after a text was written, while the body and the phase are made again a frame or more later; they now wait up to 30 and 10 frames. The ball is also hidden now while no body is shown (it was left as it was).

**Checked:** chartest 192 of 192 (10 new: turn_to_stow read and written; the katana as the sheath has it at the reach's end, within 3° and 3 cm, as held at its beginning; turned 90° at the middle by `turn`, none at the beginning; as held without the word; locked on and going left, its legs turned about 90°, the chest ahead, no setting off shown; going back, the walk backwards and the legs not turned; ahead and to its right, the legs 45°; with walk_left and walk_right named, those blended in all the way and the legs not turned); studioplay 218 of 218 (9 new: folded, a strip, the preview wider; opened again, as it was; Edit's blade box ticked for sheathe_blade's reach; a turn written; unticked; the rows of the eight ways; All eight ways goes every way round facing ahead with its legs turned, and leaves the row a clip is given to as it was; the texts as they were); `-Demo eightways -Check` (new, 4 beats). Not used by hand; lock-on in the game not tried by a script.

## 7bm. The shot camera (built 2026-10-17)

The user said (2026-10-17): "proceed with shot camera then begin work on the following", the following being the game-inspired style library (7bn). The shot camera was approved in 7bk (ActionForge's Stage: "a shot camera per animation: lens, frame shape, thirds and safe-area guides").

**Shot:** a button on the preview's row (Front, Side, Back, Above, Shot). Pressed, the preview is seen through a lens (mm, as on 35 mm film, 36 mm wide: `studio.gd shot_fov`) in a frame of a shape (16:9, 2.39:1, 1.85:1, 4:3, 1:1, 9:16), what is outside it darkened; the thirds and two safe areas (90%: what moves; 80%: words) drawn over it (Guides). Keep as its shot writes one line into the move, before its phases: `shot ROUND HIGH FAR [lens MM] [frame W/H] [follows]` (`moveset.gd`, documented in moves.txt); Forget it takes it out. Tried again while Shot is pressed, the move is seen from its shot, the camera standing where the move began unless `follows`. Cutscenes are to take a move's shot the same way (not built).

**Found by the new check and mended:** the preview's Above put the eye under the floor (2.96 m below; its height was the wrong sign), and a drag could raise the eye only 34°. Above is above now, and a drag goes from 20° under to 77° over.

**Checked:** studioplay 226/226 (8 new: Above; Shot's row and lens; 2.39:1 at 50 mm, the camera's height of view exactly the frame's; kept, written before the phases and read back; taken again on trying it, the camera where the move began; forgotten; let go; the moves' text as it was); `-Demo shot -Check` (3 beats). Not used by hand.

## 7bn. A library of styles after games, and a reference of their mechanics (built 2026-10-17)

The user asked (2026-10-17) for "an extensible library of visual styles and gameplay-mechanics references using the 30 games catalogued in the accompanying reference document": styles that are coherent combinations of visual rules, editable, combinable, saved as one's own; a separate mechanics catalog classifying each mechanic (existing, reusable, new, unverified); verified versus unverified details; a few editable presets; a test scene; a report. Eight priorities were named. **The reference document was not with the request and was not found on the machine:** the eight were researched by web search; the other 22 wait for it.

**Inspected first:** a style system already existed (`styles.txt`, `shapes/style.rs`, a world's style kept as assets by part, F2, Environment's style list): colours of what is built, what grows and the ground, roof pitch and storeys, what woods and meadows are made of, and the light by the hour. It was extended, not paralleled.

**Added to a style** (host `shapes/style.rs`, `air.gd`, `clock.gd`, `figure.gd`): a part `look`, the picture: `tone NAME N` (linear, reinhard, filmic, aces, agx; exposure), `grade` (colour, contrast, brightness), `cast C N` (a colour correction), `glow N [N]`, `shade N` (SSAO), `film N N N` (dark corners, grain, flicker: a sheet over the world, under the windows), `figures C N` (a colour pass over every person and creature); a part `about`: `says`, `after`, `tags`, `yours`; and `takes PART from NAME`, one part of an earlier style in place of this one's (combining part by part). The host's `air` answer carries the picture; its `style` answer what each style says of itself.

**Styles window** (Window > Styles; Sky > Styles…; `styleview.gd`): every style, found by a word (its name, says, after, tags); what it is and is after; Give the world this style (an edit, undone with Ctrl+Z), No style; Try on a sample (a copy of the island town with no file, the world asked about first if it has changes in no file); Open styles.txt; Make my own from it (written at the end of styles.txt, `like` it, `yours`); for a style that is yours, the picture's controls (tone, exposure, colour, contrast, brightness, cast and its colour, glow, shade, corners, grain, flicker, figures and its colour), each one line written and the world given it again; Take PART from STYLE. A style not yours is never changed.

**Eight styles after games** (`styles.txt`): siege (Into the Dead: Our Darkest Days), dreamfield (GREENFIELD), silent_reel (Silent Sadie), rust_belt (Steel Rats), neon_noir (Neon Blood), gothic_ward (Withering Rooms), factory_deep (Darwin's Paradox!), bright_blade (Cloven Blade: no game of that name was found; made from the brief's words alone). Each says what it is, what it is after and its tags; each sets build, wild, ground, growth, air and look. Nothing of the games is taken but the look in words.

**Catalogs:** `Draw3D_Style_Library.md` (each style's fields as the brief lists them, confidence, open questions, verified and unverified, sources, the implementation report: reused, added, deferred); `Draw3D_Mechanics_Reference.md` (each game's loop and mechanics classed E/R/N/U against Draw3D's systems, and the shared capabilities: six of the seven found games are side-on 2.5D and four need perception and stealth, neither of which Draw3D has: the two recommended to investigate first, each for the user to approve).

**Found and mended:** the film sheet drew plain white over the world in the editor (its material given to the sheet before its shader had code); built whole first, it darkens the corners as it should.

**Deferred:** materials beyond colour (roughness, wear, textures), emissive signs and windows, rain and wet surfaces, outlines, pixel figures, a side-on camera, interiors filled by style, a style's own props, the 22 games of the missing document.

**Checked:** host `style` tests 4 (2 new: the picture, what a style is, a part taken, wrong lines said; the project's styles.txt read whole with the eight); protocol p13, p41; `-Demo styles -Check` (4 beats: "industrial" finds 2; the town with no style and in the eight, every picture's mean colour apart from every other's by 0.045 at least; my_neon made from neon_noir, its glow turned up and its air taken from silent_reel, neon_noir not changed by a letter; styles.txt as it was after); `-Tour -Only ui` (the Styles window passes the windows' checks; only the Studio's known in_the_room/resizes fails; the Environment window's style row was made too wide by a button at first, and failed folds/resizes, so it has a row of its own); studioplay 226/226; chartest 192/192. Not used by hand.

## 7di. The guard: patrol, hear, investigate, alarm, back to the round (built 2026-10-22; G2's third example)

**Why.** The user's brief after 7dh and commit 9414904: build the guard that 7dh owed, as a behaviour made of the behaviour system's own building blocks on the events `watch.gd` already has. Do not build a separate guard AI; add only the general capabilities it lacks; keep the existing semantics (what happens comes first, states taken up again, a lower state resumed). It was not to be another audit or a UI pass.

**What was there, read from the code first.** 9414904 changed the behaviour window only (the abilities line, U7). The runner, reader, conditions and events were as 7dh left them. `watch.gd` already had:
- a guard's eyes and ears (`sees`, `hears`) and its noticing, the stealth meter;
- the event `spots`, raised when that noticing is enough;
- `patrols X Z ...`, gone round only when nothing else decided its legs;
- `hunts`: catching the walker, and running while it hunts;
- noises from the walker's steps, landings, things broken, shots and sounds, with where each was made.

The library's `guard` design says all of these. The condition `sees WHO [R]` (7dh) asks the same eyes.

**What was missing, and added, each general (any behaviour may use it):**
- **The event `hears [WHAT]`.** `watch.gd` raises it when a thing with ears hears a noise, at most once a second while the noise goes on. It gives what was heard in one word (steps, landing, alarm, shot, sound, something_break ...) and where it was made. A noise the thing made itself (a bell it rings) is not raised. Registered in `rulebook.gd`'s EVENTS (it needs `hears`). Wires (`on ... do`) do not have it: the host's ON_NEEDS was left as it was.
- **The place `noticed`**, for `go` and `hurry`: where it last heard a noise, saw the walker (kept up while it sees them), or was told something from (`tell`). `events.gd` now passes where an event happened on to `rules.gd` (`heard(..., at)`). It is not kept in a saved game.
- **The step `patrol` ("Go round its patrol").** To the next place of its design's `patrols`, done there. The round is the same one `watch.gd` keeps (`patrol_k`), so it is taken up where it was left. A place it cannot get to is passed by, and that is said. It needs `someone` and `patrols`; a behaviour's needs may now be several words, and "a round to patrol" is said in the window.
- **The step `look_about S` ("Look about").** It stands and turns this way and that for S seconds (`folk.gd` turns it each frame by `rule_look`). It takes the look.
- **The step `hurry WHERE` ("Hurry to").** As `go`, at a run (`folk.gd`'s `rule_run`). Found by the demo: a guard told of an alarm 34 m off walked there at 0.85 m/s and ran out of its time limit. It takes the legs.
- **The event `catches`, and no reset behind a behaviour's back** (the user's decision the same day: "the guard's behaviour should decide what the guard does; the stealth system should decide whether the player has been caught"). `watch.gd` still finds the catch, counts it (the world's fact `caught`), and puts the walker back where they were last safe. It raises `catches` for whoever caught them (EVENTS; it needs `hunts`). Its `calm` no longer puts a guard whose legs a behaviour has (`ruled`) back home, nor begins its round again, nor clears the move it was asked: such a guard only forgets what it noticed. Those with no such behaviour are sent home as before. A world reset on a catch, if one is wanted, is a rule of its own (an `on fact caught`), not this.
- **Said when a choice gives up.** When the choice of a `most_pressing` that was going is not done, that is now said with why ("'In order' not done: took longer than 2 s") as the next choice is tried. Before, a lower choice taking over hid why.

**The guard, `guard_ways`** (seed `behaviours.txt` and the library's). It has parameters `look 5`, `alarm alarm`, `calls 40`, `warns Intruder! To me!`, `gives_up 30`. `on spots` (its eyes' noticing, enough) makes it `become alarm` in every state; `on catches` makes it say `{caught_says}` ("Got you.") and go back to patrol, from where it stands.
- `patrol`: repeat forever, `patrol`. `on hears`, go to investigate; `on told {alarm}`, go to answer.
- `investigate`: says "What was that?". In most pressing first: within a time limit of `gives_up`, go to noticed; look about; say "Nothing there."; go back to patrol. If that cannot be done, it goes back to patrol anyway, which is the way out of a place it cannot reach. `on told {alarm}`, go to answer.
- `answer`: says "Coming!". As investigate, at a run (`hurry noticed`): the place it was told from.
- `alarm`: on entering it says `{warns}`, puts the world fact `{alarm}`, rings `bell`, and `tell {alarm} {calls}`; leaving it clears the fact. In most pressing first: only while it sees the walker, hurry to them; else, within the limit, hurry to where it last noticed them, look about, back to patrol; else back to patrol.

Hearing never confirms a threat by itself: only seeing does, through `spots`. A guard that goes to look and sees nobody returns to its round. The library guard says `hunts`, so the stealth game's catch still happens when one reaches the walker; what the guard does then is its behaviour's (`on catches`).

**What was kept.** Nothing in `rules.gd`'s states, `on` handlers, resuming or body parts changed, except the line said when a choice gives up. `go` is as it was. A behaviour that holds the legs still decides where the body goes, and `watch.gd`'s own phases decide nothing for it but its noticing, running while it hunts, and catching.

**How.** In `editor/godot`: `rulebook.gd`, `rules.gd`, `watch.gd` (`hears`, `catches`, `calm`), `events.gd`, `folk.gd`, `rulesview.gd` (the needs line; templates for the three steps), `behaviours.txt`, `demo.gd` (`guard`). Also `editor/library/behaviours/behaviours.txt`. The host is unchanged.

**Checked:** see PROGRESS (7di).

**Owed.**
- `noticed` is not saved. A guard loaded mid-look has nowhere to go and goes back to its round, which is checked; it does not go on looking.
- Wires have no `on hears`.
- The guard's spoken words are parameters only for the alarm.
- Hearing again while it looks about does not send it to the new place until it is back on its round.

## 7dh. Behaviours and rules: Draw3D's own behaviour system, first pass (built 2026-10-21; G2 of the gameplay roadmap)

**Why.** The user's brief "Build the Native Behaviour and Rules System" after the research (`Draw3D_Behaviour_Tree_Research_2026-10-21.md`): a Draw3D-owned, reusable, no-code way to make, combine, test, save and reuse behaviours across entities and genres. It is to borrow proven patterns (LimboAI and others) without importing them, keep Draw3D the source of truth, reuse the existing conditions, events, reactions, perception, moves and Studio, and build the first usable vertical slice with tests: a door and a deer first, a farming routine and a platform mechanism as checks of the design. Nothing was downloaded.

**B0, decided from the code (the research's section 8, answered as recommended, since the brief says so).**
1. **Where it lives.** A behaviour is an entry of the world's written text `behaviours`, with `behaviours.txt` as the file every new world begins with. It is kept in the document like `reactions` and `acts`: each change is one undo step, saved with the world, and the same in a game on its own. A design or one thing says `behaves NAME [with KEY VALUE, ...]`. The host checks that line (`kind.rs` `behaves_said`). The word is said once (ONCE), so one thing's own line replaces its design's; that is how one thing has its own numbers. There is no second home in Godot resources.
2. **What runs it.** `rules.gd`, in GDScript. It is event-driven (what happens to a thing is heard at once) and looks at each thing five times a second, staggered. It does not look at what is far off and asleep. The state is the fact `state` about the thing, saved with the game; after a load, that state is entered again. A running step is not saved.
3. **The words.** In text they are `most_pressing`, `in_order`, `at_once [any|all]`, `repeat [N | until C | forever]`, `time_limit S`, `cooldown S`. The window calls them "Most pressing first", "In order", "At once", "Repeat", "Time limit", "Cooldown". `when C` (only while a condition holds) is a seventh block. Waiting is done by steps: `wait S`, `wait_until C`, `wait_for EVENT`. Every step ends going, done, or not done with why.
4. **States.** States are optional. The first one written is where a behaviour begins.
   - A state written with `when C` comes over a less pressing one by itself (`priority N`, 1 if not said).
   - The state it came over waits and is taken up again where it was, or begun again if that state says `restarts`.
   - A `when` state is left when its condition no longer holds and its steps are over.
   - It goes back to what it came over, skipping any that no longer hold.
   - A `become` step clears the waiting stack.
   - Seven changes of state in a second hold the thing where it is for 2 s, and that is said.
5. **What happens comes first.** Steps under `on EVENT` run before the regular steps. While they go, the regular steps wait, letting go of the body, and are taken up again after.
6. **One body.** The steps that move a thing (`go`, `wander`, `flee`), turn it (`face`) or play a move (`move`) each take a part of its body: legs, look or pose.
   - Two parts of an `at_once` may not take the same part; that is said when the behaviour is read.
   - At run time, a step that finds its part taken is not done, and says so. The exception is a choice in `most_pressing` that comes over another: it takes that part from the lower choice.
   - A behaviour that takes the legs decides the body's running and straying. The design's own `flees`, `shies` and `roams` wait (`folk.gd`).
7. **Reuse.**
   - `use NAME [with ...]` runs another behaviour's steps. It is shared, not copied: each thing's behaviour is made again from the text whenever the text changes.
   - A behaviour's parameters are `param NAME VALUE` (with a `says` line), written `{NAME}` where they are used.
   - Copy is an explicit, separate behaviour. Rename also renames every `use`, own line and design line that names it, as one undo step.
   - Delete leaves whatever used the behaviour saying it is missing.
   - "Save to the library" writes the world's behaviours into `behaviours.txt`; "Bring from the library" adds the entries this world lacks.
8. **Brought in.** Text only, read by the same reader. It is refused, with each problem named, if any step, move or used behaviour is unknown. No code is loaded.
9. **Growing it.** `rulebook.gd` lists each building block (BLOCKS), step (ACTIONS) and event (EVENTS) once, with what it is called, its fields, what it needs of a thing and which body part it takes. A step's doing is `_start_WORD`, `_tick_WORD` and `_stop_WORD` in `rules.gd`. A new capability is one entry plus those functions, and the window offers it from then on.
10. **Animation.** A `move` step names a move and never copies it.
    - Its row has **Edit animation**, which opens the move in the Studio's Edit tab (`studio.edit_move`).
    - The Studio's banner then has **◀ Back to behaviour**, which comes back to the same behaviour and step.
    - The banner says how many behaviours use the move ("used by N behaviours").
    - This is the Studio exception the research asked for (decision 5).

**Built.**
- **Host** (`kind.rs`):
  - `behaves` in TRAITS, ONCE and OWN, with `behaves_said`;
  - three more condition words, also in Godot's `cond.gd`: `its FACT [OP N]` (a fact about the thing asking), `state NAME`, and `sees WHO [R]` (with the thing's eyes, as `watch.gd` has them);
  - `lib.rs`: the op `behaving` (who says `behaves`, own or design), sharing `said_by` with `signals`.
- **Godot:**
  - `rulebook.gd`: the reader, tree by indentation (`book.gd` now counts depth, and reads a text given to it with `read_text`). It holds the registry, the checks with lines, and the writing-back.
  - `rules.gd`: the runner.
  - `rulesview.gd`: the window **Behaviours and rules** (Ctrl+B; Window menu; the Inspector's **Behaviour…**; Playtest's button), with these parts:
    - For: what it behaves as now; give it to this one, to every one of its design, or to the several chosen in one step; take it away.
    - The behaviour: New, Copy, Rename, Delete, Save to the library, Bring from the library, Bring in from words.
    - Needs: what a thing must say, what the one picked lacks, who behaves so.
    - Parameters: for every one, or this one's own.
    - States and steps: a tree with problems shown red. Under it, the chosen step's fields: lists for moves, states, behaviours and events, and a condition built of parts (not, a sort from a list, what it is about; all of these or any; + another).
    - Add a step, Add… (a state, when something happens, on entering or leaving, what it does, after a while), Put inside a block, Up, Down, Remove.
    - Running: each one that behaves so, while playing.
  - `playtest.gd`: a Behaviours section (state, step going, last not done, each `when` state holding or not, and the cost in milliseconds).
  - `events.gd` passes every event on to `rules.gd`.
  - `folk.gd`: `rule_want`, `rule_flee` and `ruled`.
  - `things.gd` reads `behaves`.
  - `studio.gd`: the Back button and the "used by" note.
  - `acts.txt`: `try` (needs no move, so any walker may try a door).
  - `library/shelter.txt`: `door-ruled` (offers `unlock_key` and `try`).
  - `abilities.txt`: `behaves` given elsewhere.
- **Seed behaviours:**
  - `door_lock`: locked, it says so; with its key carried, it unlocks itself for good and opens; it shuts after `shuts` seconds.
  - `run_away`.
  - `deer_ways`: grazing; watching (when near walker 18); fleeing (when near walker 8; uses `run_away`, then waits until the walker is not within 18, for at most 20 s).
  - `skittish`: sees the walker within 10 m and runs away (uses `run_away`), else wanders.
  - `timed_gate`: a switch's world fact opens it; it shuts after `open_for` seconds.
- **Mended on the way.** Undo in a scripted run wrote the text's file (`main.gd` `step`, now only when `keeps_texts`). Every other text write already kept to that.

**Cross-genre check (B5, as asked: evaluated, not built out).**
- **Platform mechanism:** made of existing pieces as `timed_gate`.
  - A pressure plate (`pressed`) or a lever puts a world fact; `on fact` hears it.
  - `after S` shuts the gate; `open` swings what opens.
  - Not yet: re-timing while it is pressed again, and a moving platform driven by a state (a joint's motor goes by `when`, not by a behaviour).
- **Farming:** schedules (`goes HOUR do WORK`), sequences, goods and conditions exist for the folk.
  - A crop could hold its stages as states, using `after S` and its own facts.
  - What is missing: a crop that looks like its stage (nothing draws a thing by a fact but doors, lights and `when` gates); an act to harvest by; the folk doing a behaviour's step at a thing (`act ACT at THING` is a sequence step, not a behaviour step yet).
  - Each is a separate, defined task. None was built.

**What changed from the roadmap.**
- G2's words are as the roadmap had them (`state NAME [priority P] [when C]`, `enter`, `on`), with `leave` as steps and `after S` added. The tree tier of the research came in the same pass.
- The guard example of G2's acceptance (patrol, hears, investigate, alarm) is not done: `watch.gd`'s own phases still decide a guard. The door and the deer are done.
- The beside strip of the construction bar has no button yet; the Inspector's HOW IT WORKS has.
- With several chosen, parameters are not shown mixed; the behaviour is given to each, as its own.

**How.** `editor/host/src/shapes/kind.rs` (with the test `a_behaviour_is_named_with_its_parameters_and_one_thing_has_its_own`), `editor/host/src/lib.rs`, and in `editor/godot`: `rulebook.gd`, `rules.gd`, `rulesview.gd` (new), `book.gd`, `cond.gd`, `things.gd`, `folk.gd`, `events.gd`, `playtest.gd`, `inspect.gd`, `studio.gd`, `main.gd`, `desk.gd`, `demo.gd` (`behave`), `behaviours.txt` (new), `acts.txt`, `abilities.txt`; `editor/library/shelter.txt`.

**Checked:** see PROGRESS (7dh).

**Owed.**
- The guard (G2's third example) on `watch.gd`'s events: built as 7di.
- The construction bar's button.
- Mixed values with several chosen.
- A deeper condition builder: nested groups are typed in a part's field today.
- Remarks inside an entry are lost when the window writes it.
- Folk cannot open a door that behaves: `nav.gd` makes a way only through what offers `open`.
- The farming and platform gaps above.
- Re-timing `after` on an event.
- The LimboAI fallback stays untried: the measure passed its budget (see PROGRESS), so probe P1 is not needed.

## 7dg. The user's findings after 7df mended; G1 and X1 of the gameplay roadmap (built 2026-10-21)

**Why.** The user's message after 7df, with four pictures (a platform's edge of stone walls drawn round in red; three of Tiny Glade: a mound raised, and one with rock faces dragged along its edge; a house of the editor's with a window over its door): "platform doesn't look right ... manufactured construction coming out of it. It should be sort of a natural edges, things that look like earth ... when you kind of click on the mound and you kind of hold and drag left and right, you can add sort of the edges that look natural, like natural earth, kind of like a rocky type surface"; "cut the fence, it's still erases half of the fence ... It only keeps half"; "when you draw out a house, some of the windows get placed where the door is ... this was a problem with the original build ... in the demo town, one of the buildings had a window blocking the door"; "I don't see the frame rate number or option that I asked for in a corner of the screen ... when a game world is running"; "have a look at these things and continue the roadmap."

**What was found and done.**
- **The fence cut losing half.** Found: a drawn line turned with several chosen (7df's `move_group`) was turned by its route's own turn, which the editor's handles, cuts and joins do not read (`makers.gd` `line_world` used the route's place only). Cut so, the first half was written in the route's turned frame and the other half in the world's, unturned: it stood elsewhere, turned about the route's place. Now `move_group` turns a drawn line in its places (the route keeps no turn); `line_world` and `_line_to` read and write a turn where an older world has one; and a cut whose other half comes to nothing (or fails) is put back as it was, with why, instead of leaving the line half gone. The 7de/7df hand tests' other causes (a gap smaller than a pier) were mended in 7df.
- **A window over the door.** Found: a facade's windows by bays (the identity prototype's `FacadeGen`, which the acceptance suite keeps unmodified) never looked for its door, so a side with an odd number of bays had a window over the door in its middle (the old demo town had the same); windows put by hand (7dc) were not looked at either. The host's facade (`shapes.rs` `WalledFacade`) now lays its bays itself where it has a door, by the same names, leaving out any within 0.25 m of the door's 0.9 m; one put by hand over the door goes to the nearer side if it fits, else is not made. Its code version went up, so worlds opened are made again by it (7df's upgrade on open).
- **The frame rate.** It was there (7dc: View > Frame rate, Ctrl+Shift+F) but hidden unless asked for, and this machine's preference said hidden (written 11:13 that day, by a person or a run). Now shown unless put away (a new preference; the old one is not read), on a layer of its own over the game's HUD, and switched from the game's menu (Esc) too. A tour or a demo no longer writes the preference.
- **Platforms of earth and rock.** A platform's edge may be `rocks`: the land's own slope, steeper (1 m), faced with rock all round: crags a metre apart and wider, sunk into the land inside the rim, their tops at the lip of the grass, each made by the rock generator of 7cx as a broken boulder of the style's rock with moss on top (`build.rs` `crag`), so they are one with the rocks painted about. Build > Platform makes one so now (it made a wall of stone; stone is still a choice beside it, with the land's slope). Chosen, a platform's edge is pressed and dragged along (Tiny Glade's gesture): rock faces come where the pointer goes, or, pressed on rock, are taken off there (`rocky`: stretches in metres along its rim; all of it is the edge `rocks`); one step each, rebuilt as it is dragged. A platform shows its handles when chosen now (its corners and height), without the + marks, which would take the press.
- **G1, one condition language and declared events** (the roadmap's first batch, on its own recommendations: the user said to continue the roadmap without answering its section 10).
  - `cond.gd` takes `all: A; B` and `any: A, B` (an any: inside an all: parted by commas), with why each does not hold ("none of them holds: it is 22:00, not between 06:00 and 20:00; nor the weather is clear, not rain"), anywhere a condition is written: talks' `if`, quests, work, `while`.
  - A thing's `when` may say a condition in those words (`when all: fact open; any: hour 6 20, weather rain`, `when hour 18 6 for 5`): `state.gd` asks `cond.gd`, its `for` kept as before; a fact's name as before.
  - `events.gd`: one place every event is raised (acts, loose, watch and talk call it), the last forty kept; and events about the world watched for there, of every thing whose lines wait on one: `on fact NAME`, `on hour H`, `on timer S`, `on enters R`, `on leaves R`.
  - The host validates the new words (`kind.rs` `cond_said`, `when_is_cond`, ON_HAPPENS of ten) and says what each event needs said of a thing (`ON_NEEDS`, in `{"op":"words"}` as `needs`, with `conditions`): the capability manifest the G2 entry point will offer from. A thing's own lines parted by semicolons keep an all:'s semicolons in its line.
  - Why not yet: the Playtest window lists, under what was done, what waits nearest the walker and why it has not happened ("no one has done 'ring' at it", "it is 14:00; it waits for 18:00", "the walker is 12 m off, and it waits for them within 4 m").
  - The Inspector's rows offer the new events (abilities.txt), with a value to begin from (18, 5 s, 4 m, a fact).
- **X1, autosave.** Every 3 minutes while there are changes in no file (World > Autosave, on or off), the world is written aside to `backups\autosave-NAME.json` beside its file (a new world's beside this machine's settings); the host's `write` with `autosave` leaves what is unsaved unsaved. A run that did not end as it should is told at the next start and offered the autosave, opened as an earlier save is (Save asks where). Measured first: writing the showcase village aside takes 2 ms on this machine.

**What changed from the roadmap.** Nothing of its order. G1's "the Playtest window shows, for a selected entity, each rule" is, for now, every thing that waits, nearest the walker first, not one chosen. `enters`/`leaves` are of the walker only. The capability manifest is said by the host and not yet used to narrow the Inspector's list (G2's panel will).

**How.** Host: `shapes.rs` (`WalledFacade` bays round a door, own windows moved aside, code version 101), `shapes/land/build.rs` (`EDGES` with `rocks`, `rocky`, `rocky_said`, crags along a rocky slope, `crag`; tests `a_rock_face_is_made_as_big_as_asked`, the wear test reading `bounds` as middle and size), `shapes/kind.rs` (ON_HAPPENS, ON_NEEDS, `can_happen`, CONDITIONS with all:/any:, `cond_said`, `when_is_cond`, `own_said` keeping groups; test `one_condition_language_and_world_events_are_said_rightly`), `lib.rs` (`move_group` turns a line in its places; `rocky` and `rim_long` said; `words` with `needs` and `conditions`; `write` with `autosave` and `ms`), `tests/protocol.rs` (p92 the turned line keeps no turn; p93 no window over a door; p94 a platform faced with rock where said; p95 an autosave written aside). Godot: `makers.gd` (`line_world`, `_line_to` with a turn, `_turned`; a cut put back if its other half is nothing), `line.gd` (a platform's handles without +; pressed and dragged along its edge, `_along_rim`, `_with`, `_rock_values`), `marks.gd` (a platform's handles shown), `buildbar.gd` (Platform's edge rocks by default; the edges beside it), `fps.gd` (shown unless put away; not written by a script), `main.gd` (the frame rate on its own layer; `events`, `autosave`), `gameui.gd` (Frame rate in the game's menu), `cond.gd` (`WORDS`, `parts`, all:/any: in `check`, `why`, `said`), `state.gd` (`when` in cond words), `events.gd` and `autosave.gd` (new), `acts.gd`, `loose.gd`, `watch.gd`, `talk.gd` (raised through `events`), `playtest.gd` (what waits and why), `inspect.gd` (a value for a new event), `abilities.txt` (the five events), `desk.gd` (World > Autosave), `demo.gd` (`mended`, 5 beats; `rules1`, 6 beats).

**Checked:** see PROGRESS (7dg).

**Owed.** The press on an unchosen mound (Tiny Glade needs no choosing first); a rock face dragged on a Raise-painted platform tried by eye; rock faces that turn with the rim's own bends more closely; a structure turned by its own turn (makers.gd's Turn on a drawn structure's node) is not read by its handles either; G2 next (states as pages, the Behaviour and rules… entry point, the manifest narrowing what is offered); the unclean-exit offer of X1 seen by hand (a script is kept out of it).

## 7df. Construction made sound; several chosen at once; the gameplay roadmap (built 2026-10-21)

**Why.** The user's brief "Post-Construction Roadmap and Reusable Gameplay Systems": finish the construction milestone first — the defects of their hand test (cut walls and fences losing end posts; wear that showed nothing, or regular gaps; wear lost on low fences), regression coverage, the tours repaired or their fate said, and a selection of several with group move, turn and lift — then a code-grounded roadmap into reusable gameplay systems (events, conditions, states, interactions, AI, items, combat, quests), with targeted research, a common contextual entry point, a vertical slice, milestones, a first batch, a manual plan and a records plan. Nothing of the gameplay roadmap is built.

**What was found and done.**
- Wear: a piece's mesh is its brief without its place (`engine_core::build::PLACE`), so all pieces of a structure were worn alike. Worn pieces now get one of seven seeds by their place (`worn_apart`); counts come by chance under one (`some`); a dry-stone wall wears through its stones (its cracks had been inside them), a fence through its pales, rails and posts, a hedge thins; all scaled by height.
- Cuts: the least gap is a pier and a hand, by sort (`CUT_GAPS`); a cut keeps clear of the places either side.
- Opening: a world written by older code is upgraded as it is opened (the host's `open`): it had stood empty, since nothing holds made things in a file. The showcase village had been empty since 2026-10-17.
- Several chosen: `group.gd`, Shift and a click; the host's `move_group` moves, turns and lifts makers and things as one edit, a drawn structure by its line, a joined wall once. What stays each one's own is said, not written over.
- Demos check themselves (`expect`, `demo: problems`).

**The roadmap.** `Draw3D_Gameplay_Roadmap_2026-10-21.md`. Its decisions, for the user to confirm (its section 10): behaviour stays content in the host's document, evaluated in Godot; one condition language (`cond.gd` taking in `when`, with all/any); declared events; states as pages (RPG Maker's model: the highest eligible wins), with `work.gd`'s priority and resume; a capability manifest from the host's words; one entry point, **Behaviour and rules…**, from HOW IT WORKS and the strip beside a structure, applying to several by their shared rows only; no behaviour-tree library now (LimboAI and Beehave assessed: MIT, but behaviour as scene nodes would be a second home for it); Luau stays parked. Order: G1 conditions and events, X1 autosave beside it, G2 states and the entry point, G3 the bell at the gate, then G4 interactions, G5 perception and AI, G6 items, G7 combat, G8 quests.

**How.** Host: `shapes/land/build.rs` (`worn_apart`, `some`, `towards`, `leaning`, the dry-stone, fence, post and hedge wear, `sunk` given worn pieces, `weathered` scaled), `shapes.rs` (weathering not over dry stone or wooden posts), `lib.rs` (`open` upgrades; `move_group`). Godot: `group.gd` (new), `main.gd` (Shift+click, right drag, keys, the open's words), `desk.gd` (the status line), `makers.gd` (`CUT_GAPS`), `demo.gd` (`expect`, demos `fences`, `several`).

**Checked:** see PROGRESS (7df).

**Owed.** A look or a setting given to several at once (G2); a structure on cells as one of several; the village tour's door offer and its "Walls inside…" beat (G4); the items tour (G6); the library tour rewritten as a protocol test and a demo; the ui tour's Studio beat skipped while the Studio is parked; the style tour's clock beat.

## 7de. Cut, joined, taken away; the core capabilities audit (built 2026-10-21)

**Why.** The user's brief after 7dd: "First, review the current code, latest PROGRESS.md, Tiny Glade construction reference, and UI inventory. Identify which Tiny Glade-inspired construction requirements are genuinely still missing or incomplete. Complete only the remaining milestone-critical items ... Then conduct a code-grounded Core Capabilities Audit ... Evaluate a reusable contextual editing bar for selected objects and entities ... Do not begin implementing the next major systems until the audit is complete and reviewed." What the reference's construction tools have that the code did not, and what was done:
- A wall or fence split into sections, and snapped back (reference 1.3, 5.2; the user's notes 3, 7; the 2026-10-18 comparison's "still missing"). **Built: Cut beside it and a click on a +, or Shift and a click on a +; joined by an end dragged or drawn onto another's end, the one that was there kept.**
- Delete on what is in focus, the hammer removing a whole structure (5.4, 12.1). **Built: Delete takes a drawn thing whole with its line, one step; Shift+Delete one piece.**
- The disable-snapping key keeping a window from becoming a door (1.1, 2.2, 12.2). **Built.**
- Style Clone across buildings, walls, fences, stairs (4.5, 12.4). **Built: Copy and Give beside every drawn sort, to one of the same sort.**
- Roof orientation changed (the user's notes 1). **Built: Turn its roof, on a house of four corners that meets no other.**
- Left, with why and how large, in the audit's section 1: roofs curved by dragging; supports under a lifted house; a stairs route tool; a quick eraser; group selection and transforms; a quick toolbar; trapdoors and bunting.

**The audit.** `Draw3D_Core_Capabilities_Audit_2026-10-21.md`: every system's capabilities, what is authored in the interface, how the systems join, what tests, tours, demos and hands each rests on, verified apart from partial apart from missing foundations; the contextual bar evaluated across eight sorts of thing as declared facets in one order (phases: extract the strip's helpers, then placed objects and creatures, then facets declared, then several selected); the next priorities in two directions for the user to choose, and four questions.

**How.** In Godot: `makers.gd` (`CUT_LINES`, `line_world`, `remove_drawn`, `cut_line`, `join_lines`, joining after a drawing), `marks.gd` (a stroke drawn on from an end joined), `line.gd` (`cutting`, a + cut with Shift or after Cut, an end let go on another's end joined), `main.gd` (`take_away`: a drawn thing whole), `buildbar.gd` (Cut, Copy, Give, take away beside a structure; take away beside a house; Turn its roof; `copied_sort`, `_copy_structure_look`; Ctrl at the foot of a wall), `icon.gd` (`cut_line`, `roof_turn`), `demo.gd` (`cuts`, 7 beats). In the host: `build.rs` (`roof_turn`, the roof's `across` on one oblong with nothing hidden), `shapes.rs` and `real.rs` (`across` in both roof builders and the roof's solid, `rise_over`), `lib.rs` (`roof_turn` in a structure's look), `tests/protocol.rs` (p88).

**Checked:** see PROGRESS (7de).

**Owed.** A cut anywhere along a wall, not only at a +; joining an end onto another's middle (a T); cut, join and Copy and Give for structures on cells; a roof turned on an L or a joined house; the turned roof meeting a neighbour's (its planes); a tour part for drawn construction.

## 7dd. Leaves under trees, snow with depth, rooftops and streets (built 2026-10-21)

**Why.** The user's message after 7dc: "autumn leaves should not appear everywhere. mostly around the trees and blowing around when you create the real depth. urban set should have ramp additions on the roof tops and boards across buildings and other parkour type addons and decor and power poles. complete the rest of the work not done as well. and add a button for me to cycle through seasons or weather somewhere in the corner". Their words, and what was done:
- Leaves mostly around the trees, blowing around. **Built: fallen leaves drawn under the 64 trees nearest the eye (`trees` of the paint), the loose leaves under them and heaped toward the trunk; lifted along a strong wind and come down further on.**
- Real depth. **Built: snow's caps (`drifts.gd`): a cap on whatever faces up of what is drawn, as deep as `air deep` says, rims rounded, pressed where printed; what is under a roof, a porch or a tree sheltered from snow, rain and dust, seen by an eye looking straight down.**
- Ramps on roofs, boards across buildings, parkour pieces, decor, power poles. **Built: on flat town roofs a ramp, a rail, crates; planks from roof to roof (`across`); fire ladders up the backs; power lines drawn along streets (`powerline`: poles, wires, lamps, transformers).**
- The rest of the work not done. **Built: a door where it is clicked (`line_gates` `sN@M`); platforms painted with a brush and joined where they meet; walls stacked with battlements taken off (`stands_on`, `topped`, `wall_top`); the dock's boat boarded and rowed (the boat with a body); designs of the library still, gently or freely in the wind (their own `gives`); a bar's neon lighting only what is near it; the Library and Styles windows fitting a small screen. Still owed: platforms blended at a shared edge with the land between (joined, not blended); the Character Studio's window on a small screen.**
- A button in a corner for the seasons or the weather. **Built: `corner.gd`: the season and the weather, a click to the next, a right click back.**

**How.** In the host: `build.rs` (the `powerline` sort and its poles and wires; `street_piece`: pole, wires, plank; rooftop ramp, rail, crates; the fire ladder; planks from `across`; `gap_between`, `share_of`; `stands_on`, `topped`, `bare_top`, `wall_top` and battlements; doors at `sN@M`; the dock's boat a body; neon's dim light), `tileops.rs` (`across`, `stands_on`, `topped`), `lib.rs` (`across` said), `shapes.rs` (the new pieces' styles). In Godot: `drifts.gd` and `corner.gd` (new), `lying.gd` (trees, leaves under them, blown), `world_view.gd` (trees, shelter, the `laid` hook), `weather.gd`/`weatherbook.gd` (`deep`), `env.gd` (Snow: how deep), `buildbar.gd` (Power line, Raise, a wall's top, the power line's height, wind for designs, the door where clicked), `makers.gd` (`paint_area` for platforms), `marks.gd`, `things.gd` (a light's `strength`), `icon.gd`, `library.gd`, `styleview.gd`, `main.gd`, `tour.gd` (the world clear within the room).

**Checked:** see PROGRESS (7dd).

**Owed.** Prints pressed into the cap's corners only (a metre apart on the land); leaves without depth of their own but the loose ones; melting from the sun's side; wires to houses; lamps on poles lit at night; planks between buildings other than flat-roofed town ones; a stacked wall's height not followed when the lower is made higher after; the lower wall's piers through the upper; the Character Studio's window on a small screen.

## 7dc. Weather that lies, a town, platforms and windows by hand (built 2026-10-21)

**Why.** The user's message after 7db, with a picture (a path beside a fence's gate and beside a wall's arch, not through them), and "work autonomously to complete these". Their words, and what was done:
- Paths do not line up with gates and arches. **Mended: a path through a wall, a fence or a hedge says where along the side it crosses (`opened` `NAME@METRES`), the side is cut into pieces about that place, the way through stands where the path is.**
- Gates and arches with a context menu for type and colour, and a door or a gate in an arch. **Built: beside a wall, fence or hedge, and over a gate or an arch clicked: arches round, pointed, low or flat (`arch_form`), open or with a gate, a studded door or a grille hung in them (`arch_fill`), their stone's colour; gates of boards, pickets, a farm gate, iron, two leaves (`gate_form`), any colour.**
- A frame rate indicator in a corner, toggled. **Built: View > Frame rate, Ctrl+Shift+F, kept on this machine.**
- The remaining work, the feedback notes and the transcript. **Built: raised platforms (notes section 6), paving painted as an area and one with what it overlaps (section 7), windows put by hand, joined side by side, window boxes for three or more, washing lines between two spaced upstairs, the door put where the foot of a wall is clicked (sections 2, 4 and the pasted tips), a house lifted off the ground (floating foundations), smoke from chimneys (sections 4, 8), the sun turned round (section 8), own pictures carried into an exported game (7db's owed). Still owed: merlons taken off by stacking, the boat boarded and rowed, a wind setting for library designs, terrain platforms painted with a brush (drawn round here), joined platforms blended at a shared edge.**
- Rain and snow and falling leaves; what falls lying on roofs and the ground, with intensity and persistence. **Built: `lying.gd` and the paint's `snow_cover`, `wet`, `litter`, `dust`: snow white on what faces up, rain wet and in puddles, leaves on the ground; how fast it lies (`air settles`), how long it stays (`air keeps`), how much falls (`air falls N`); weathers drizzle, blizzard, hail, leaves.**
- Footprints by what lives, the character too; leaves moved about in autumn. **Built: the walker and those with bodies press prints into what lies, a picture of the ground about the walker (`prints`), gone over `air prints` seconds; loose leaves about the eye kicked up as they go through.**
- Anything falling from the sky, meatballs, frogs, boxes. **Built: what falls has a shape (streak, flake, leaf, petal, hail, ball, meatball, box, crate, frog, fish, or a design of the world), lies where it lands a while, and may leave something lying; Environment > What falls, and The thing picked.**
- The Draw5 notes on weather and seasons. **Read (`MAP_WEATHER.md`, `PARTICLES.md`): deposits that build and are remembered, prints that press into them, an effect being any falling thing; taken over in that spirit.**
- Dandelions in the ground mix. **Built: dandelions and dandelion clocks (Plant Generator presets), in Flowers' mix and in a new mix for Grass.**
- An urban set: commercial, apartments, townhouses, lofts, bars with neon, urban decor and clutter. **Built: Build > Town (shop, apartments, townhouse, loft, bar, `URBAN`), their fronts (`front`: a shop's glass, awning and sign; a bar's dark front, awning, neon sign and blade sign; garage doors), sash, factory and modern windows, plant, vents, aerials and water towers on flat roofs; a kind trait `neon`; 21 designs of neon signs and street clutter and three Decorate sets.**
- More stairs up steeps and across water. **Built: steps of wood with a rail, log steps, a ramp (`climb_form`); a boardwalk, a humped stone bridge, a wooden footbridge (`water_form`).**
- Imperfection settings, not cookie cutter. **Built: `wear` (as new, a little worn, worn, falling apart) on houses, walls, fences, hedges, paths, yards and gardens: cracks, streaks, bricks and stones gone, coping broken, pales gone and broken, cobbles and flags gone and paths' edges ragged, panes broken and boarded, gates' boards split, arches' stones sunk, roof tiles gone and mossy.**

**How.** In the host: `build.rs` (`drawn_rim` cut about crossings and `gap_at`; `gate`, `arch_rise`/`arch_height`; `urban_piece`: awning, signboard, neonsign, rooftop; `washline`; `weathered`; `own_windows`; `platform` sort, `platform_soft`; bridges and climb forms in `pathway`; stones on `polyfloor`), `land.rs` (`Raised` soft and raised as well as dug), `tileops.rs` (crossings measured; platforms raise the land), `real.rs` (window forms, broken panes, a roof's chimney and wear), `shapes.rs` (`WalledFacade` own windows, `Mesh::chimney`, `wear`, `smokes` trait), `kind.rs` (`neon`), `grown.rs` (dandelions). In Godot: `lying.gd` (new), `fps.gd` (new), `weather.gd`/`weatherbook.gd` (shapes, the air's falls, settles, keeps, prints, season, sun), `world_view.gd` (what lies, prints, neon), `env.gd` (the new rows), `buildbar.gd` (Town, Windows, Platform, Paving, gates, wear, fronts, climb and water forms, lift, world season), `makers.gd` (urban sorts and palettes, `paint_area`), `marks.gd`, `things.gd` (chimney smoke), `clock.gd` (sun turned), `icon.gd`; `editor/library/urban.txt`; `export.ps1`.

**Checked:** see PROGRESS (7dc).

**Owed.** Merlons taken off by stacking; the boat boarded and rowed; a wind setting for designs of the library; platforms painted with a brush, and blended where two meet; snow piling deeper (it is a colour, not a depth) and melting from the sun's side first; a print in fallen leaves is a gap in the drawn leaves, the loose ones only kicked by the walker and those with bodies; what falls drawn about the eye only; a door put exactly where the wall's foot is clicked (it is put in that wall's middle piece).

## 7db. Context and the wild edges (built 2026-10-21)

**Why.** The user's hand test of 7da, with two pictures (the strip over a house as wide as the window, its icons in one row; a porch wrapped round a house whose roofs cross at a corner), their list, two of their Tiny Glade notes on advanced tricks, and "work autonomously to complete these things and use your judgement for the smaller reversible things that need some direction". Their words, and what each came to:
- The context bar can be huge: groups and submenus, like the preset colours, that fold out when needed. **Built: folds (`_fold`), the house's row ten buttons, a patch's sets, colours, season and look kept folded.**
- Porch roof not lined up. **Mended: cut on the slant at corners, a hip or a valley.**
- Paths still a line, not painted; dirt, rock, wood, cobblestone; walls with such options. **Built: a path dragged at its width; cobbles, flagstones, stepping stones as well as paving, gravel, earth, boards, each laid stone by stone; six more wall pictures.**
- Paths through walls make arches, over ponds stepping stones, up hills stone stairs (their notes, sections 2 and 7). **Built.**
- Their feedback on how the land is shaped (section 6, raised platforms). **Not in this pass: owed.**
- Decorate paints the ground but not walls. **Built: along a wall it hangs lanterns, baskets, banners, shields, wreaths, flower boxes (section 2: a lantern on a wall is a wall lantern).**
- Decor without wood, style or colours. **Built: benches and lamps of several styles, Colours by wood and paint.**
- The garden with few flowers. **Built: its beds planted with any of fifteen.**
- Never assume a usual colour means no options. **Built: a Colours fold beside every patch, by role, any colour (`vary ... even`).**
- Rocks over plants poke through. **Built: plants keep clear of what rocks and decor lie on (`lying`).**
- Trees with few options, not linked to the generator. **Built: the Trees brush grows the Tree Generator's sorts, each opened in it.**
- Shadows choppy and shimmering. **Changed: atlas, splits, blending, filtering, bias; softness by blur, not by the sun's size.**
- Waves on a shore, breaking on rocks. **Built: foam by depth in the sea's and a pond's water.**
- No reeds. **Built: reeds and bulrushes standing in the water, in Waterside's mix (and the grown reeds there before).**
- Birds disappear but their noise stays. **Mended: perched outside the crown; heard where they are.**
- Stack buildings on a flat roof. **Built.** Ctrl to unsnap a wall's end: **there since 7cm.** Merlons taken off by stacking, floating foundations, wells of towers, window boxes or washing lines by windows' spacing: **not built (a well and a washing line are decor in the mix).**
- A post-apocalyptic set: buildings, trees, decor, ravens, wolves. **Built: the wastes (designs, sets, a style, a palette); wolves from the creatures in Life's mix.**
- Pumpkins should not sway; a wind setting for anything the wind moves. **Built: each grown plant moves as such a plant does; `wind` beside every patch of grown things.**
- Sunsets like The Witcher 3 and Ghost of Tsushima. **Built: the golden hour in any style, and a richer default day.**
- A shore brush for a deck or tied boats, later for travel. **Built: Water > Dock.**
- Lily pads, shells, kelp, seaweed in Waterside. **Built.**
- Seasonal variations of trees and foliage. **Built: per patch.**
- Any selection updated by hand; own wall textures, own decor. **Built: any colour everywhere; pictures and models brought in from beside what they are for.**
- Mix Life's items. **Built: sets put in and taken out of a mix; sorts as before.**

**How.** In the host: `build.rs` (porch `cuts`, `pathway` laid by floor with `climbs`, `wet`, `jetty`; `archway`; `hung`; `mooredboat`; `beds`; `base`; eaves guarded), `land.rs` (`Ground::sunk`, `standing_on`, `rest` on a dug pond's water, `Stood::may_stand`, `wades`, `LYING`, `Clear::blocks_but_water`), `grown.rs` (`WIND`, `calmed`), `vary.rs` (`even`), `tileops.rs` (a pond's inside `water`; a stacked house left out of cutting), `lib.rs` (the structure says `hung`, `beds`, `dock`, `base`). In the editor: `buildbar.gd` (folds, Colours, seasons, wind, hanging, imports, gardens, sorts and sets), `marks.gd` (paths stroked; stacking), `makers.gd` (`roof_under`, `drawing_strokes`), `clock.gd` and `main.gd` (the golden hour, shadows), `world_view.gd` (foam; own pictures' folder), `wild.gd` and `sound.gd` (birds), `styles.txt` (`wastes`, the summer day), `library/decor.txt` (thirty designs), `sounds.txt` (`ravens`). Demo: `-Demo context`.

**Owed.** Terrain shaped as raised platforms dragged up and down (their notes, section 6); paths painted as areas; windows and doors placed one by one, joined when side by side, and the window boxes and washing lines that come of it; merlons and floating foundations; the boat boarded and rowed; own pictures carried into an exported game; a wind setting for designs of the library.

## 7da. Its details beside it (built 2026-10-21)

**Why.** The user's hand test of 7cz, with a picture of a patch of Life beside its strip: "refer to the docs and proceed with these things ... try to complete the milestone of things and i can test them out after and we can tune and improve the tiny glade milestone after." Their words, and what each became:
- The critter noise is too loud; a setting for its volume in the context bar. **Built: crickets quieter, the three nearest of one sound heard, a patch of Life quieter or louder beside it (`loud`).**
- Some birds don't move. **Mended: with no tree near, a bird goes about on the ground.**
- Fish swim out of the water. **Mended: the water says its outline; a fish keeps inside it.**
- The pond still not a hole in the ground; it looks like a modern pool. **Built: dug into the land, no bank raised, stones and grass to the water (their notes, section 6: water painted into the ground, with rocks and plants round it).**
- Every item of the menu bar with context options, texture and colour where they apply; fences and hedges lack them, among others. **Built: a panel beside a wall, a fence, a hedge, a path, a yard, a garden and a pond.**
- Doors, windows and lights; balconies and wrap-around porches; stairs. **Built beside a house: windows, doors, lanterns (lit), porch and balconies, its stair.** The reminder of 7cz, windows, doors and stairs placed one by one by hand (their notes, sections 2 and 4: a window tool, a window low on a wall a door, windows that join), stays owed: it is a placement tool of its own.

Their Tiny Glade notes read again for this: section 2 (a lantern on a wall a wall light, on the ground a post lantern), 4 (windows of different shapes, sills, plants and coverings; doors), 5 (colours of each part), 6 (ponds painted into the ground, rocks and vegetation round them), 7 (fences of their sorts), 8 (a lantern lights at once). What of it is built here is in the house's and the structure's own words, the way their looks were (7cm, 7cn): nothing is a second way of doing what was there.

**How.**
- Host `build.rs`: LOOKS gains `window_form`, `shutters`, `sill`, `door_form`, `door_colour`, `lamps`, `lamp_colour`, `fence_form`, `hedge_form`, `flower_colour`, `floor_colour`, `water_colour` (given to every piece and through a facade to its windows, doors and walls); params `porch`, `balcony` (PORCHES), `stair_form` (STAIR_FORMS), `stair_wood`, `stair_rail`, `dig` (DIG, DIG_SLOPE). Sites `porch#SIDE`, `balcony#SIDE_bK` of archetype `porch` (an outside corner run past, an inside one stopped short), `porch#SIDE_steps`; inside stairs by `inside_stair` (`steps` with wood and rail, `ladder`, `winder`). A dug pond: its level a hand under its edge, no bed slab, no bank pieces or mounds. Fence and hedge sorts; a floor's colour; a pond's water says its outline (`water R D 0 X Z...`).
- `shapes.rs` `made_from`: tower, towerroof, parapet, polyroof, polyfloor, pathway, winder, porch, porchroof, ladder of `style.build`; bank and climber of build and wild. A porch's roof is `porch#SIDE_roof`, of archetype `porchroof`.
- `real.rs`: `window`, `door`, `doorway` take the brief (`arc_band`, `arc_points`, `flat`); `doorway_lights` gives a wall with a doorway `lights ... when dark` (`shapes::traits`).
- `land.rs`: `Shape::dug`, `dug_in`, `sunk` (in `top` and the ground's own mesh, not `height`, which the terrain tools read); `DUG`, `dug_asset`; a patch of ground is made from its part. `tileops::settle_dug`. `Meadow` param `loud`.
- Godot: `buildbar.gd` `structure_chosen`, `_fill_structure`, `_house_details`, `_fill_house_detail`, `_choices`, `_tints`, LOUDS; `wild.gd` `_pool_at`, `_wet`, `_peck`; `things.gd` `water_outline`; `sound.gd` `ALIKE`, `loud_of`; `icon.gd` pictures for each; `library/decor.txt` lamp-post.

**Checked:** Host: 77 unit tests and 87 protocol tests pass (new: p87 a pond dug and a house's porch, balcony and stair; a unit test of every window, door, fence, hedge and porch sort and the doorway's light; p02, p68 and p73 told that a doorway gives a light and a drawn pond is dug, p73's pond kept on the land with `dig 0` to keep testing the bank). `-Demo details -Check` 8/8: dug 0.79 m in its middle, deeper to 1.4 (1.29 measured), water 2e5a6a, no bank pieces; 8 birds, all 8 moved, 1297 fish samples none out of the water, the patch quieter at 0.75, birdsong 3 at once; windows arched, no shutters, plain sills, door glazed a24e38, lanterns both; 2 storeys, a porch round it (28 pieces with roofs), a balcony of 4 pieces along its front; a wound stair; pickets f2efe6, a hedge in flower f0a0c0; bricks, 1.45 m high, ferns; paving, 1.6 m wide; 2 lamp posts and 3 lights lit at 21:00. No script errors. Pictures looked at. Not used by hand.

## 7cz. Decorated and alive (built 2026-10-21)

**Why.** The user's hand test of 7cy: "everything so far recently is finally headed in the right direction", with findings and asks, and freedom for small additions that fit. Their words, and what each became:
- The roofs view off is buggy, does not come back on. **Mended: a house of one storey stuck the button at its first step.**
- The collision effect is still visible, and the player is too slowed by foliage. **Mended: plants lean off from their foot; held back by a third as much.**
- A brush for ambient life: crickets, butterflies, moths, fireflies, frogs, birds, fish for ponds. **Built: Paint > Life, its sets, new small life; over a pond, life in it.**
- A contextual decorator brush: vines or ivy on walls, paintings; on a floor, furniture inside (couches, chairs, beds, dressers), benches, crates and barrels outside; smart and customizable, palettes, sets or items swapped. **Built: Paint > Decorate (walls, a room's floor, the land), sets beside it; a painting and a rug in the home set. Items one at a time by hand, as before.**
- Every thing painted or built has a decor option. **Built for houses (climbers, furniture) and patches (sets); walls, fences and hedges by the brush only.**
- Flesh out the menu bar. **Built: Build.**
- A preset or arrangement liked kept from the little context bar and recalled; in the global library, saying which projects use it. **Built: looks kept, `looks.json` in Draw3D's own folder, Library > Kept looks.**
- The paint system for fields of crops, to be harvested in games one day. **Built: Paint > Crops. Harvest owed.**
- Tools that do more by where they are used: the flower brush on the ground flowers, on a wall blooms; ivy, moss, vines. **Built: CLIMB_OF, `paint_pressed`.**
- A system for windows, doors and stairs: not now, a reminder. **Owed, with what 7cy owed from their Tiny Glade notes (windows placed by a tool, doors near a wall's foot).**

Their Tiny Glade notes (`Draw3D_Tiny_Glade_Personal_Feedback.md`) read again for this: vegetation that responds to structures and paths, decoration by context, and life that makes a place feel inhabited are what this answers; nothing of it is a second way of doing what is there (climbers are a house's, as its dressing is; decor and life are patches, as flowers are).

**How.**
- Host `build.rs`: params `climb` (words of CLIMBS: ivy, moss, vines, blooms), `climbed` (0.6), `climb_on` (sides, each `SIDE[:WORDS]`), `climb_colour`; sites `climb#SIDE_cJ` of archetype `climber` (made in build.rs: leaves as faces against the face, moss cushions, woody stems, flower clusters, giving a little to the wind) along every outside side not shared, not cut, not a gate, and clear of every way through (`ways`). The structure report says them. `kind.rs` trait `glows`.
- `wild.gd`: what glows is drawn unlit. `world_view.gd` treads: a lean by height over the foot. `motor.gd` `WADES` 0.35. `main.gd` `_houses_open`.
- `buildbar.gd`: brushes crops, decor, life (`bring`: library designs brought in); SORTS entries of no family are library designs (`_sort_design`, `_of_library`); SETS; CLIMB_OF and `paint_pressed` / `climb_motion` / `climb_release` (asked by `marks.gd` on a press of a bar brush); `_fill_climb`, `_fill_furnish`, `_pond_lively`; `keep_look`, `give_kept`, `kept_looks` (`Controls.home()/looks.json`). `desk.gd` `_build`. `library.gd` Kept looks.
- `editor/library/decor.txt`; `sounds.txt` crickets.

**Checked:** host tests 76 + 86 (new: climbers stand out from the face as high as told, moss low, blooms of their own colour; p86 a house climbed side by side, none over its door, going with it when moved; p43 knows the new collection); `-Demo decor -Check` 9 of 9 (roofs 1, 2, 0 and back on; blooms on s2 and ivy on s1, 12 climbers, the land not painted; moss on every side, 0.8, #8a9a3a, 26 climbers; a home of 6 things, then a store; a yard of 12, then a farmyard; 14 of life, then dusk; a pond given the lively set, 35 things; a field of 34 patches, maize put in; a look kept and given, red walls and a slate roof; through flowers held back 4%), pictures looked at (the ivy made thicker after the first). Not used by hand.

## 7cy. Where a tower meets a house, and cattails' heads (built 2026-10-20)

**Why.** The user's hand test of 7cx ("really good pass again"). Their findings, and what each became:
- Two towers at a house stand with their round walls inside it. They want a choice: one open space where they connect, or as it is; and each shape remembered if they are taken apart. **Built: walled or open, beside it; nothing of a line is changed, so apart each is whole.**
- Cattails have no palette; the ends could be bulbous or fuzzy and frilly, by some rule, a look that could go to other things. The most basic form, without much time. **Built: `plump`, `fuzz`, `head_tint` for every grass head, beside Waterside.**
- A mix's sorts are presets; could each reach the plant generator's settings, by a small icon? Only if cheap. **Built: sliders per sort, opening the Plant Generator on the patch's design.**
- Paths cut through foliage, but there are no sorts of path yet, and the yard does not work properly. **Not now, at the user's word.**
- Their notes on playing Tiny Glade, to be kept and consulted for construction, terrain, vegetation and decoration. **Kept: `Draw3D_Tiny_Glade_Personal_Feedback.md`.** Read against what exists: much of sections 1, 3 and 5 is built (handles, a house's controls together, colours and materials beside it, Copy/Give as the bucket); owed and named there: roofs dragged to a curve, a material proportion slider, windows and doors placed by a tool with doors near a wall's foot, lanterns that know a wall from the ground, paths with sorts that become arches, steps on slopes and stepping stones in water, raised platforms painted, chimneys with smoke, time of day as a control.

**How.**
- Host `tileops::cut_by_houses`: each house says the others it meets (`meets`, by id). Of a higher and a lower one, where either says `open_into`, the higher one's cut where it is inside the lower carries `open` after its height. `build::cut_rim` reads it (`RimSide::cut_open`); built, each storey that begins under the lower's top is left out, a `wall` piece named `…_over` from that top to the storey's, and a pier between two open stretches begins over it.
- Grass knobs `plump` (0.5 as before), `fuzz` (0), `head_tint` (0); `grown/plants.rs` `grass` scales plumes, cattails and husks by `plump`, swells a plump cattail, and stands frills (`fuzz` × 26 blades) out of each head.
- `buildbar.gd`: `by_join_walled` / `by_join_open` beside a house that meets another (`open_into`, one change with `together_after`); `_fill_heads` beside Waterside; `by_<brush>_tune_<sort>` under the sorts, `ed.grow.open_design`.

**Checked:** host tests 75 + 85; `-Demo meets -Check` 5 of 5 (walled 14 pieces, open 10 and 4 over, apart 12 and meeting none, back open with Ctrl+Z; heads plump 0.75, fuzz 0.5, rust; the Generator open on brush-cattails with the heads' numbers); `-Demo mixes -Check` 6 of 6. Not used by hand.

## 7cx. Mixes, rocks and towers (built 2026-10-20)

**Why.** The user's hand test of 7cw: "extremely pleased with the direction": more intuitive, customised on the fly, fewer menus; keep this for every system. Their findings, and what each became:
- Bushes to work as the flowers do: sorts put in or taken out of a mix, not one sort for another. **Built, for flowers, bushes, waterside plants and rocks.**
- Rocks are basic and round: shapes, textures and colours wanted, randomised, with palettes for where they are (desert, beach, forest). **Built: a rock generator.**
- Something that paints the decorative things, such as cattails, that the engine can make but no brush has. **Built: a Waterside brush.**
- The Look options (copy and give) shown for a house but not for flowers. **Built: Look for a patch, and Copy and Give beside a house.**
- A tower dragged over a house does not attach; either merge them or drag one out of the other. **Built: a tower out of a house's corner, a tower let go at a house goes onto it, and it moves with it.**
- Walls: fine as they are.
- Later, at their word: balconies and wraparound porches pulled out of a house; a clutter system (hay bales by a barn, clicked and moved); a character or a sheep shown while building, with its controls over it, keeping where the game starts. **Not built.**

**The rock generator** (host `shapes/grown/rocks.rs`, family `rock`, twenty knobs):
- Each stone is an icosahedron split once, twice or three times (detail), its corners pressed onto planes it is broken along (facets: none from below), lumped by five long waves (rough), flat below, stretched to its height, width and depth, turned, leant, sunk.
- **Forms:** a boulder with smaller ones at its foot (pieces); a slab; an outcrop of blocks, the middle ones tallest; stones lying about; a standing stone, or a ring of them.
- **Colour:** `place` picks a colour, a colour for bands and a colour for moss (the style's rock, forest grey, beach pale, desert red, mountain blue-grey, volcanic black, chalk); `tint` is one of your own (rrggbb). Moss lies on faces that look up. Each face a little its own shade (speckle), darker toward its foot.
- **Material:** `texture` (none, rough, cracked, grainy, smooth) writes `texture FILE S tinted` into the kind. Tinted is new (world_view.gd): the picture's lightness against its mean, over the colour it is laid on, so a red rock stays red and its stone shows. Without it, a picture laid over a design is coloured by the design's hue but has its own lightness, which made rocks near black.
- **Edges:** sharp (each face flat: the mesh without `smooth`) or soft.
- Presets: boulder, mossy boulder, slab, stones, beach pebbles, outcrop, desert red rock, lava rock, standing stone. Two grass presets were added for the water's edge: cattails and rushes.

**Mixes and patches** (buildbar.gd):
- `SORTS` says what may be in each brush's mix: its preset, family, words, picture and colour. Beside a patch, one button opens the sorts under the strip; each is lit when it is in. Put in, a sort is grown for that patch the first time, two shares of it; a rock takes the patch's place, colour, picture, edges and moss from the rocks already in it.
- A **patch** is a maker of a brush: "painted rocks", "painted rocks 2"… Its designs have its suffix ("brush-boulder-2"), so changing one patch leaves the others. A new patch copies the chosen one's sorts and numbers, then paints. Clicking a patch makes it the one its brush paints. A patch's look (sorts, numbers, how thick) is copied and given to another of the same brush, one change.
- Rocks beside them: their colour by place (chips) or any colour, their material (pictures), less or more moss, smaller, bigger, other shapes (each seed moved on), sharp or soft. Bushes and waterside plants: lower, taller. Changes over many designs are one change (`regrow_each`).
- Look on the bar, for a patch: New patch, Copy, Give, with words. Copy and Give now also end the strip over a house.

**A tower joined to a house:**
- **Pulled out** (line.gd `corner_towers`): a small green tower stands out past each corner of a chosen house that turns out (not an L's inside corner). Pressed and dragged, a round tower is staged with its middle on the corner, as wide as twice the pointer's distance from it (2.4 to 9 m), square to the house's first side; a click alone makes one 3 m wide. Let go, it is kept, chosen, one step.
- **Let go at a house** (makers.gd `tower_to_house`): a tower drawn or moved with its middle near a house's corner (within 1.6 times its half-width and a metre) or side goes onto it, corners preferred. Ctrl as it is let go holds it apart.
- **Joined** (host tileops.rs `follow_towers`, after every edit with `follow_joins`): a tower whose middle is within its own width of a house's side is joined to it by its middle ("c HOUSE SIDE ALONG OUT X Z"). The house moved, turned or pushed, the tower goes where the same part of that side is, unless the tower was itself moved by the edit; then it is joined again where it stands, or not at all.
- What of each is built where they meet is as 7cr had it: the house's wall inside the tower is left out; the tower's wall inside the house only as high as the house.
- **Found on the way:** a drawing let go while the host was still building its last drag step was committed when that landed, after the editor had already asked for it, so it was not chosen or joined. `commit` now takes `wait: true` (the editor's let-go: marks.gd, line.gd), and then waits for it, up to 30 s. Without it, a commit is as it was (two protocol tests say so).

**Checked:** host unit tests 74 (new `a_rock_is_coloured_by_where_it_is_found_and_has_a_picture_over_it`; every preset of every family grown, a slab low, a standing stone tall, an outcrop wide, cattails over rushes) and protocol 84 (new p84: a tower at a house's corner joined, moved with the house, back with Ctrl+Z, left where it is put by hand). `-Demo mixes -Check` 6 of 6: forest rocks of 5 sorts after two were put in, a second patch at place 3 while the first stays 1, cracked rock and moss 0.4 and a new seed, waterside of 3 sorts, bushes with brambles and blueberries in and shrubs out, two towers joined and one moved 4.1 m with a 4 m move. `inplace` 5 of 5, `onehouse` 6 of 6 (one Ctrl+Z a stroke still). Pictures looked at: rocks broken and mossy, desert red beside grey, towers at two corners. **Not run:** by hand; speed with many rocks.

**Owed:**
- A house dragged out of a tower; a tower snapping while it is dragged, not only when let go; the house's roof cut where a tower stands.
- Balconies, wraparound porches; the clutter system; a character or a sheep shown in the world while building, its settings over it, its start kept.
- Your own palettes kept on the strip; doors and windows placed by hand; furniture placed and locked along walls; the house's dressing bushes still round.

## 7cw. Changed where it stands: roofs with no gap, roofs off, a house's parts and materials beside it, painted plants that are real and changed beside them (built 2026-10-20)

**Why.** The user's hand test of 7cv: "very, very finally good in terms of the direction". Plop something down and adjust it at once, with buttons in context, is how it should work. Their findings, and what each became:
- Two joined houses showed a strip of wall between their roofs (a screenshot). **Mended.**
- Show and hide roofs, to see the layout inside and that the merge is right. **Built.**
- Colours shown visually and changeable, less text; an icon on the roof for its material, likewise windows. **Built for walls, roof, trim and timbers and the two materials; windows' own sorts are not.**
- Grass and flowers painted the same thing; wanted: dense grass in the wind, its length and shade (green, yellow, blue, white) after painting. **Built.**
- Round bushes are blobs; trees and rocks wanted settings from what was painted, as a house has. **Built for bushes and trees; rocks only thicker and thinner.**
- A house attached to a tower, or dragged out of one, as one; furniture placed and locked along walls; doors placed; your own palettes. **Not built: requirements, below.**

**The roof gap.** `hiding_solids` cut each roof where it lay inside the other, the first a hand bigger and the later a few centimetres smaller (0.06 m). Where two ridges of one height cross, the first's ridge lay inside the later's solid, and the later's in the first's: both were cut and nothing was left. The later one is now made smaller by 0.16 m, more than the first is made bigger (0.12 m). Where they are within that of each other one is kept. The test `a_joined_house_has_no_gap_in_its_roof` lays out an oblong, an L, a T, a cross, a narrow wing into a wide one and the other way, and an offset join. With gable and hipped roofs, it looks down on every 20 cm of each outline for roof over it (`roof_hides` is now a function both use).

**Built:**
- **Roofs off** (main.gd `open_houses`):
  - On the strip over a house and in View.
  - One press hides every roof (`roof`, `polyroof`, `towerroof`). A second also hides what each house made over its first floor's boards (`world_view.cut_above`, by the maker): upper floors, their walls and windows. A third puts all back.
  - For looking only; Show everything also ends it.
- **A house's parts beside it** (buildbar.gd `_fill_beside`):
  - Walls, roof, trim and timbers are each a picture of a house with that part in its colour now.
  - Pressed, a row opens under the strip: 8 to 12 chips of colours for that part and a picker for any colour, set when it closes.
  - The walls' and roof's materials are buttons showing the picture itself. Pressed, the others open: plain, plaster, painted plaster, bricks, stone, rough stone, boards, planks; plain, tiles, mossy, shingles, slates, tin, thatch.
  - Each is one change through `look_set`.
- **Brushes that grow their own** (buildbar.gd `BRUSHES`, `_grow`):
  - The first time a brush is used, it grows designs of the plant generator (host `generate` with a name) and its maker paints them.
  - Grass is one design of the grass family: about 45 cm, 22 blades a tuft, 12 tufts a patch, no seed heads, a little straw, and it gives in the wind. The old grass was nine blades and a wildflower, which is why grass and flowers looked alike.
  - Flowers are daisies, poppies and bluebells among grass of their own. Bushes are a leafy shrub of the bush family.
  - Trees and rocks paint the built-in ones, as before.
- **Grass's own colour:** a knob of the grass family, `tint`, as rrggbb; 0 keeps the style's greens.
- **Beside what was painted** (`_fill_painted`): clicked, the strip shows by where it was clicked.
  - **Grass:** shorter and longer (12 cm to 1.4 m), ten colours.
  - **Flowers:** daisy, poppy, bluebell, tulip, lupin, foxglove, each in the mix or out (grown the first time it is wanted). Also the grass among them shorter or longer.
  - **Bushes:** shrub, wild rose, blueberry, bramble, low mat; lower and taller.
  - **Trees:** mixed, broadleaf or conifers.
  - **Every brush:** thinner or thicker.
  - A change grows the design again, and all of it that stands follows. Each is one Ctrl+Z.
- **A brush's making and its first stroke one Ctrl+Z again:** a big stroke is built in the background, and `together` sent at once was refused as busy, so grass took two steps back; it is now sent once the build is done (buildbar.gd `together_after`).
- **Demo** `inplace` (5 beats).

**What is owed, kept as requirements:**
- **A tower and a house as one building:** a house dragged out of a tower or let go against one, its walls and roof meeting the tower's.
- **Doors placed and moved by hand;** windows of their own sorts beside the house.
- **A furniture mode:** furniture inside a house dragged along its walls and locked in place, so that characters walk in and live there.
- **Your own palettes on the strip:** kept, locked, changed.
- **Settings of their own for rocks** (a rock generator).
- **Less text everywhere else in the editor:** the strip shows the way.
- **The house's own dressing bushes are still round blobs.**

## 7cv. One house, one outline: wings pulled out, houses joined, a palette beside it, painted plants (built 2026-10-20)

**Why.** The user's hand test of 7cu. The bar was fine, but the experience was underwhelming:
- two houses dragged together "don't look right, don't feel right";
- there was no way on the house to make it an L;
- palettes were in a menu, not on the thing;
- grass and foliage were drawn as lines, where Tiny Glade paints them with a brush;
- the look is poor.

The user asked for the basic functionality first, comparable in how quickly things go together, with the polish to come later. They noted that the construction direction had been tried three times.

**What was wrong underneath.** Each house was a box of its own, and two that overlapped were patched where they met: walls cut, roofs cut, doors moved (7cr, 7ct). Every way of overlapping found a case the patches missed. Also, an L-shaped outline was roofed over rectangles cut row by row (`square_blocks`), so a wing's ridge could run any way, and each piece had its own chimney and gable. That accounts for the grey slab and the cream triangle in the user's screenshot.

**Built:**
- **Houses that overlap become one house** (makers.gd `join_houses`).
  - It happens when a house is dragged out into another, or moved, turned, pushed or given a wing into one.
  - On letting go, the house is turned square to the one it went into (to the nearest quarter turn), and the two outlines are made one (`Geometry2D.merge_polygons`).
  - The outline is squared to the first house's frame (`squared_outline`). That first house keeps its name, look and storeys; the other goes.
  - The drawing or move and the joining are one Ctrl+Z (`together`). Ctrl held on letting go keeps them apart.
  - A round tower never joins.
- **A wing pulled out of a side** (line.gd `_wing_pulled`).
  - A green + stands out past each side's arrow, on sides 3 m or longer.
  - Held and dragged out, it pulls a wing from that side: half the side wide, at least 3 m.
  - Dragged along the side as well, the wing slides; within half a metre of an end it is flush with it, and the house is an L. In the middle it is a T.
  - It is built live while held. Corners that no longer turn are let go (`_plainer`).
- **Which way is out for a side** comes from the way the outline goes round (`_outward`), so the arrows on an L's inner sides point out of the house.
- **One roof as a roofer roofs it** (host `build::wing_blocks`, used for houses in place of `square_blocks`).
  - The outline is covered by the fewest whole oblongs, biggest first: one wing over each arm, the wings overlapping where they meet.
  - Each wing's ridge runs along its arm.
  - A wing that runs into a bigger one, and is no wider than that one is deep, stops at the bigger one's ridge, so its ridge dies in the other's slope.
  - The cutting of 7ct makes them one roof with valleys.
  - One chimney to a house, on its biggest wing (`no_chimney`).
- **The roof's look** (host `shapes::roof`, gabled):
  - tiles laid one by one, in rows about 0.4 m up the slope, each about half a metre along;
  - every other row set over by half a tile, each tile a little lighter or darker and lifted a hair more or less, with a dark underlay between;
  - a board along each eave, barge boards up the gables, and a thicker ridge.
  - What hides a roof now reaches 0.3 m below the eaves, so where roofs meet the boards are hidden too.
- **Beside the house, in the view** (buildbar.gd `beside`). While a house is chosen and nothing is held, a strip over its roof shows:
  - the seven palettes as swatches;
  - gable, hipped or flat roof;
  - a storey fewer, how many, a storey more.
  - Each is one change, with no menu between.
- **Paint on the bar.** Grass, Flowers, Bushes, Trees, Rocks and Rub out, each a brush dragged on the land.
  - Each brush paints into a maker of its own, made the first time it is used and growing nothing until painted: `painted grass` and the like, a meadow or a wood of the plants the engine has.
  - It uses the painting of 7v.
  - New host setting `dense` on meadows and woods: so many plants to each 2 m square painted, so a brush is as thick wherever it goes. Before, the count was shared out over all that was painted.
  - The making of a brush and its first stroke are one Ctrl+Z.
- New pictures in icon.gd: grass, flowers, bushes, trees, rocks, three roof forms, storey up and down.

**Checked:**
- Host unit tests 72 of 72. `wing_blocks`: an L is two wings, the second running from the first's ridge; a T's stem stops at the bar's ridge; an oblong is one. Two older tests were adjusted, because the boards hang below the eaves: one measures a roof's top, not its size; one keeps the eave-hiding margin.
- Protocol tests 83 of 83.
- `-Demo onehouse -Check` 6 of 6, its pictures looked at:
  - a house dragged out with the strip beside it and four wing handles;
  - a wing pulled from the longest side to its end, giving an L of 6 corners with 2 roof wings and a cross gable;
  - a second house dragged into it, giving one house of 8 corners;
  - a palette and a storey from beside it;
  - grass 864, flowers 487, trees 24 painted;
  - six Ctrl+Z: the strokes, the storey, the colours, then the second house's drawing with its joining, leaving the L of 6 corners and no brushes.
- `-Demo joined -Check` 7 of 7, with no script errors. Its "overlapped" beat (a house dragged into another) now ends in one house, not two sharing a wall.
- **Not used by hand.**

**Not done, or not right yet:**
- A house dragged over another shows them overlapping until it is let go: the joining is not shown live.
- Handles show once a house is clicked, not on hover.
- Two chimneys were still seen on an L in the pictures (the second may be a sort's own); the gable end is plain where the walls are timbered.
- Joining one house into a house of another sort keeps the first's look without asking.
- Ladders and stairs that become one another, things that wrap round what they are turned against, decor placed by hand: not built.
- The ground of a new world is still a dull yellow-brown; the look as a whole is the style step's.
- The joining replaces 7cr's shared walls for houses that overlap. Houses side by side, touching but not overlapping, still share a wall as in 7cr.

## 7cu. The construction bar down the right side, as a Tiny Glade player builds (built 2026-10-20)

**Why.** After 7ct, the user asked whether construction was going the Tiny Glade way, and whether I knew how a player builds there with the tools (not only how the pieces behave). They asked for a construction bar of its own on the right side, with groups and the tools under them as pictures, each tool understanding what is built. Their references: `Tiny_Glade_Tool_and_Functionality_Reference_Consolidated.md` and `Draw3D_Style_References_Neva_and_Firewatch.md`. They also said merging two dragged-out houses sometimes doesn't look right.

**How a Tiny Glade player builds** (from the reference, the wiki's Tools and Building menu pages, and community guides; I have not played it, and the guides disagree on details):
1. Pick a tool from a small bar of pictures. Its groups have a few tools each: Build has a square building, a round building, a wall and a fence; Decor has three windows, a lantern, a flag and a chimney. Nothing is set before drawing.
2. Gesture on the land: drag, and the building grows under the pointer; let go, and it is there.
3. Shape it afterwards with handles on the thing itself: arrows push the sides, height, the roof's peak for pitch, a ring to turn it. Right click is focus mode.
4. Context does the rest: overlapping buildings become one with one roof, a low window becomes a door, a path through a wall makes a gate, a path across water makes stepping stones, and stairs become a ladder, platform or bridge by their slope. Ctrl holds it off.
5. Style afterwards: Change appearance on the part pointed at, and Style Clone from one thing to another.
6. Erase is a brush too (ruins). Ctrl+Z undoes, so nothing is ever wrong.
7. The charm is largely procedural detail and motion (irregular stones, ivy, clutter, things springing into place, sound, soft light), not more settings.

**What Draw3D had against that:** the rules underneath (7ck to 7ct: drawing live, walls cut, joins, roofs one) follow Tiny Glade, but the way in did not. It was one worded list under Build on the left: about 25 rows mixing roads, lots and streets, grid cells, and drawn things. Each sort of house was a row of its own, so a sort was chosen before drawing. Handles only show once a house is chosen. A look is set from the Inspector or World outline.

**Built:**
- **The construction bar** (`buildbar.gd`, made by desk.gd `build_bar`), down the right side while editing. It has five groups, each a picture with a word: Build, Paths, Water, Look and Erase. Pressing a group opens its tools beside the bar as pictures; pressing it again puts them away:
  - **Build:** House, Tower, Wall, Fence, Hedge.
  - **Paths:** Path, Yard, Garden.
  - **Water:** Pond (with life), Kerbed (a stone kerb).
  - **Erase:** the eraser, as on the left.
- **A tool is taken up at once** and drawn with on the land, as its row of the Build list draws (`makers.begin_drawing`). It is lit while it is in hand; pressing it again puts it down. Nothing is asked first: a house is drawn plain, and a tower is a house of the sort "tower".
- **Look, afterwards, on the house chosen:**
  - its sort (house, cottage, castle keep, tower, shelter, ruin);
  - its colours, seven palettes drawn as little houses in their colours;
  - **Copy** and **Give**, a style clone: Copy keeps one house's sort, colours, pictures, roof and storeys, and Give puts them on another house. Each is one step, undone with Ctrl+Z.
  - With no house chosen, it says to click one first.
- Windows keep clear of the bar (desk.gd `room`).
- New pictures in icon.gd: tower, wall, fence, hedge, path, yard, garden, pond, kerb, look, cottage, castle, shelter, ruin, copy, paste, and swatch.
- **The Build list on the left is kept** unchanged, at the user's word.

**Checked:**
- `-Demo bar -Check` 5 of 5, its pictures looked at. Each button was pressed as a hand would press it (`desk.hands`):
  - all five groups are at the right edge, and Build's shelf opens to the bar's left with its five tools;
  - House lights and a dragged house is built, then goes unlit;
  - a second house is dragged into the first, and they are told of each other's roofs;
  - a wall is drawn from the house's side;
  - Look shows the sorts; cottage, then the palette "painted blue"; the look copied and given to the second house (cottage, 7393b4);
  - Ctrl+Z takes the given look back, and Build pressed twice puts the shelf away.
- No script errors.
- **Not used by hand.**

**Not done:**
- **Handles shown on hover.** In Tiny Glade, pointing at a building shows its handles; here a house still has to be clicked. Showing one house's handles while another stays chosen would conflict with the right-drag and with Look, so it is its own step.
- **Decor** (windows, doors, lanterns, flags and chimneys placed by hand) is not on the bar, because none of it can be placed by hand yet. A drawn house makes its own windows and door.
- **Stairs** drawn by nodes, and **stepping stones** where a path crosses water, are not built.
- Number keys and a quick ring of tools at the pointer (Tiny Glade's Space) are not built.
- What the bar draws has the same merging as 7ct: two houses dragged into each other show two gables running into one another. The user's "merging doesn't look right" has not been pinned down to a case.

## 7ct. Roofs that are one where houses meet; a door kept when its side is cut (built 2026-10-20)

**Why.** What 7cr left owed, continuing construction as the user's direction document orders: roofs of houses that touch or overlap crossed one another, and a lower roof could be seen inside a taller house's upper room; a house's door in a side that came to stand inside another house was lost with that side.

**Built:**
- **Roofs run into each other as one roof** (host `shapes::hiding_solids`, `Mesh::cut_inside`):
  - A roof is not built inside the house it runs into. That means inside the other's walls up to its eaves, or under the other's roof (gable, hipped or flat, as `roof` builds it).
  - Each triangle is cut exactly where it goes in, so two roofs meet in a valley, and a lower roof ends at the taller house's wall.
  - **Cut both ways**, so what is left is the outside of the two together.
  - **Where two roofs lie in one plane**, side by side and as high, one gives way. The one built first (as with walls: the higher, or the earlier) is taken a hand bigger, its ridge cap included; the other a little smaller, so the first runs just under the other and no crack or flicker shows.
- **The oblongs of one house** (an L, a T, a U) are cut by one another the same way, so an L-shaped house has one roof with a valley, not two crossing.
- **Which houses are near.** The host tells each drawn house of the houses near enough (`roof_near`, in `tileops::cut_by_houses` after every edit, undone with it): their line, height, storeys, roof, form, pitch, and which is built first. The house's own build works out their eaves from the ground as their build does, and hands each of its roofs those solids in that roof's own frame (`hide`). Ruins, round towers and curved houses are not told.
- **A door left out with its side** goes to the nearest piece of wall still standing outside, at least 1.6 m long (`build::cut_rim`). A gate in a garden wall left out where a house stands does the same.
- Seen in the Inspector's structure (host `nodes`): `roof_near`.

**Checked:**
- Host unit tests 72 of 72, two of them new:
  - `a_roof_is_not_built_inside_the_one_it_runs_into`: nothing of a cut roof inside the other; about half left where the other covers half; all of it inside a bigger one gone; one apart from it untouched.
  - `a_door_left_out_where_a_house_stands_goes_to_the_nearest_wall`.
- Protocol tests 83 of 83, p83 new: each house told of the other once, the higher built first; an L's two roofs built; moved apart, told of none; undone, told again.
- `-Demo joined -Check` 7 of 7, beat 5 "Roofs that are one" new, its pictures looked at. Close up, two houses side by side, and overlapping: the lower roof ends on the taller's slope. An L-shaped house: one roof, one valley. A wing drawn across the end of a house: its roof runs into the main slope, nothing standing through.
- `-Demo houses` 7 of 7, `sorts` 8 of 8, `walls` 6 of 6: no script errors, no host panics.
- **Not used by hand.**
- A hipped or flat roof running into another is worked out by the same planes, but not seen in a picture.

**Not done:**
- A roof over a house that is not square-cornered (`polyroof`) is cut by others, but does not cut theirs.
- Round towers, ruins and curved houses neither cut nor are cut.
- Chimneys and dormers are cut as the roof is: one inside another house is cut off where it goes in.
- Where a moved door goes is chosen by the host, not asked; the person's `line_gates` still names the old side.
- Floors overlapping are still lifted 2 cm, not cut (7cr).

## 7cs. The user's hand test of 7cr: a house's sort undone, a house dragged by any part of it (2026-10-20)

**Why.** The user tried 7cr by hand (`-Demo joined`, then by hand in the same world) and said:
- Good: houses tracked around, "the house is kind of merge and blend".
- "Sometimes moving them does not move."
- The garden wall across the house does not move with it; they could see one move before.
- A house made a cottage and then a house again no longer had a house's look.
- On the look as a whole (kept, not acted on, at their word): the original built look is underwhelming, "generic", windows and roof tiles with "very little charm, very little character". If the planned base-style work is where that is answered, later is fine. It belongs with step 3 of their direction document of 2026-10-20 (a stylized base style after Firewatch and Neva).

**Found, from the run log and the code:**
- **Moving that did not move.** The log shows a second click in the same place, which picks what is behind (a window of the house), then a right click. The right button drags what is picked, and on a drawn house that was one piece, not the house. The toolbar's Move did the same, moving one window. Only the white cross moved the house. This is the likeliest cause; it was not seen happening, so it is not certain it is the only one.
- **The wall that did not follow.** That wall is drawn across the house, not joined to it, so it is cut where the house stands and stays where it was drawn (7cr). A wall drawn out from a house's side is joined and does follow (7cp, `-Demo sorts` beat 6). This is as built, not a fault. Whether a wall drawn across a house should be carried with it is the user's to say.
- **The sort undone.** Choosing "a house" set only the word `house`: the cottage's thatch, white walls, flowers and steep roof stayed.

**Built:**
- Each sort of house now sets everything any sort sets. What it does not say is given as a house drawn new has it (makers.gd `HOUSE_PLAIN`, `house_sort_values`): tiles, the world's colours, one storey, flowers and bushes.
- **The right button on any part of a chosen drawn house** drags the whole house, as its white cross does (line.gd `grab_whole`, main.gd `dragging_whole`): one edit, undone with Ctrl+Z.
  - A right click on one of its pieces says how to move it.
  - The toolbar's Move on a piece of a house says so too, instead of moving one window off it.

**Checked:**
- `-Demo joined -Check` 6 of 6: two beats added.
  - Cottage, then house, by the sort buttons: thatch then tiles, cottage white then the world's colours.
  - Right-dragged by a wall, real mouse events: the cross moved 4.0 m; undone, 0.0.
  - The pictures looked at.
- `-Demo sorts` 8 of 8, `letgo` 4 of 4, `houses` 7 of 7: no script errors.
- **Not used by hand.**
- In the demo the sort buttons are pressed by their signal, not clicked.

**Seen, not mended:** every hand run since 2026-10-18 ends with Windows code -1073741819 (an access violation) as the editor's window is closed, after Godot's own warnings of leaked render resources. It comes at exit, after the work is saved or not. Recorded, not looked into.

## 7cr. A wall stops where a house stands; houses that touch share their walls (built 2026-10-20)

**Why.** The next construction slice after 7cq, chosen by the user from what 7cp owed: "a wall put across a house shortened to it, and houses that touch joining their walls and roofs".

**Built** (host `tileops::cut_by_houses`, after every edit, with the rest of `settle_drawn`, undone with it; host `land::build` `grown`, `spans_inside`, `cut_rim`):
- **A wall, a fence or a hedge across a house** stops at the house's outside face. What of it lies in the house is not built (`cut`: each piece of its rim, how far along it is left out). An end of its own (a pier) stands at either face; beyond the house it goes on. Moved or pushed, the house cuts it where it is now; moved off it, the wall is whole again.
- **Two houses side by side** have one wall between them, not two in one place. The house standing higher builds it; of two as high, the one first in the document does. Its corners meet the other's walls with a corner post.
- **Two houses that overlap** are one inside: the walls of each that stand inside the other are left out. A house standing higher keeps its storeys over the lower one's top, so its upper walls stand whole over the lower roof. The house that gives way lays its floors 2 cm over the other's (`overlaps`), so the two are never one face and do not flicker.
- **Roofs:** each house keeps its own roof; side by side they meet over the shared wall, overlapping they cross, as two gables do. They are not merged into one roof.
- **Said where it is chosen:** World outline and the Inspector, under "How it is built", a part "Where it meets houses" says an end joined to a house (7cp), the stretches left out, and the houses it overlaps (host `nodes` `structure`: `cut`, `overlaps`).
- A shelter or a ruin is cut whole (no storeys over).

**Checked:**
- Protocol p82 (new):
  - a wall across a house has nothing inside it and a pier at each face;
  - side by side, only the higher house's wall stands between them;
  - overlapping, neither's ground-storey wall stands inside the other, the higher's upper storey over the lower is kept, and along the shared sides only one builds;
  - the one giving way has `overlaps` 1, the other 0;
  - moved apart, both are whole (`cut` empty);
  - undone, cut again.
- `-Demo joined -Check` 4 of 4, its pictures looked at:
  - a wall across a house, from the side and from above with the roof off;
  - two houses side by side;
  - the first dragged by its cross into the second, from the side and from above;
  - dragged out, then undone;
  - the note in World outline for the house that overlaps.
- `-Demo houses`, `sorts`, `walls` and `letgo` as before, no script errors, no host panics.
- Host unit tests 70 of 70; protocol tests 82 of 82.
- **Not used by hand.**

**Not done, or only partly:**
- Roofs are not merged. Where a lower house's roof runs into a taller one, it may be seen inside the taller house's upper room.
- Floors overlapping are lifted, not cut: a 2 cm step where the two meet.
- A house's door in a side that comes to stand inside another is left out with that side.
- Walls drawn on cells (`on`) are not cut by drawn houses, nor drawn houses by them.
- Gardens and ponds are not cut by houses.

## 7cq. Every edit failing after a house was pushed narrow: found, reproduced and mended (2026-10-20)

**Why.** The user's first test of 7cp by hand (after `-Demo sorts`, in the world it saved and opened again): live rebuilding as handles go, roof shapes changed live, the round tower and the other sorts each answering their controls in their own way, a cottage dragged out from Build, and the walls' handles and added places were all found good. Then, after some adjustments, a house could no longer be moved; a click said that the transaction had not finished and had no result; choosing another house did not help; things seemed locked. Asked: reproduce it against the code, find whether it is a regression, a tool handing over badly, a transaction gone wrong or something not yet understood, mend it if the cause is clear, and test that moving and editing go on working after a structure's size, roof or what is joined to it is changed.

**What was found (from the user's own session, not guessed).** Their editor's log said, in order: "Not moved: the transaction ended without a result", then the same for a wall, a place, a hedge and a fence. The host's own error output (`editor/godot/host.log.err`, 17:20) said why:
1. A build had panicked: `min > max ... min = 2.0, max = 1.15`. The stair chooser (7co, kept in 7cp, `land::build`) held each flight's length between 2 m and the room along its wall less a door's (`clamp(2.0, run_along - e - 0.6)`). A wall shorter than about 2.8 m made the upper bound less than the lower, and Rust's `clamp` panics then. A side pushed in by its arrow (it may go to 1.5 m from the side across), a small house or a narrow one, of two storeys or more, is enough.
2. The panic happened while the engine held its memory of work done (`Cache::memo`, a lock), and left the lock poisoned. Every build after it asked for the lock and panicked in turn (`memo: PoisonError`), each ending without an answer: from that moment nothing in the world could be changed until the editor was started again. That was the "locked".
3. A house's handle let go after its drag had failed said nothing and left the failed drag shown (`line.gd` `end`), so what came after looked as if it would not move.
- Ruled out by the same reproduction: a handle let go over a window, over the toolbar or over a folded window's title (the release reaches the handle and the move is kept, one step); another house chosen after (its handles show, it moves); a house moved after a side pushed, its roof raised, its stair moved and a wall joined (each one step, the wall following).

**Mended:**
- The stair chooser skips a wall too short for a flight of 2 m (no panic, the stair goes where there is room, or there is none).
- The engine's memory of work, found poisoned, is forgotten and the builds go on (`core/src/eval.rs` `Cache::memo`): it is only saved work, so forgetting it costs time, never what is made.
- A build that panics now ends in an error that says why ("the build failed: …", `core/src/engine.rs`), not in "the transaction ended without a result".
- A house's, a tower's or a height's handle let go when its drag was not kept: the drag is let go and the person is told ("Not changed: …. It is as it was"), and the house's places and height stay as the host has them.

**Checked:**
- `-Demo letgo -Check` (4 beats, new), driven by mouse events through the editor's own input (never the real pointer):
  - a cross let go over a window and over the toolbar (kept, one step, not held after);
  - another house chosen and moved;
  - one house moved after its side, its roof, its stair and a wall joined (each one step; the wall followed);
  - a side of a two-storey house pushed in as far as it goes (1.5 m), then the house moved and a hedge drawn (all kept).
- **The same demo against the host as committed** (7cp, built apart): the push panicked (`min = 2.0, max = 1.95`), the move after it was not kept and the hedge failed with "the transaction ended without a result", the host's output `memo: PoisonError`. As the user saw.
- Protocol p81 (new): 24 houses of the six sorts, small, narrow and thin, of two and three storeys, built, each moved, the narrow one pushed in further and its stair said to a corner with no room, and a hedge drawn after. It fails with the old clamp and passes now.
- Core: a memory of work left poisoned by a panicking thread is forgotten and taken up again (unit test).
- Host unit tests 70 of 70; protocol tests 81 of 81; core tests all pass.
- **Not checked by hand.** Automated runs drive the editor's input, not a person's hand: whether it now feels right with a mouse is the user's to find.

**The user's findings kept as requirements of the construction system (not built in this pass):**
- *Doors and windows.* Some houses have windows and some seemed to have none; on at least one no door was found. As built: a house drawn from Build has its door in the side where the drag began (`line_gates s0`), which may face away from the eye, and its windows by bays (`window_every`), so a short wall can have none; a shelter has no walls and a ruin's doorway is a gap. Owed: openings and entrances placed so that every house has a door one can see and reach, consistently.
- *Openings as modules.* Windows and doors placed, moved, sized, restyled and combined by hand (two or three windows together; whether window-sized openings may make a wider doorway is a design question to weigh, not a rule that every window may become a door).
- *How windows look.* Some windows look unsightly in some places: for the styling of openings, after the current construction milestones.
- *Roofs.* The live roof controls are good; the flat end of the range is not reached by the knob (it stops at a rise of 0.2; a flat roof is its own `roof_form`). Owed: the whole range of profiles, flatter ones included, by the same handle.
- *Hedges, fences and walls* were not assessed by hand, because of this fault: not to be judged missing or broken until tried on their own.
- *Trees and what grows*: how far they can be edited is for the capability checks later.

## 7cp. Houses of their sorts, stairs of their own, what is joined following, and a style and palette to build in (built 2026-10-20)

**Why.** After 7co the user asked for variety: "stairs are not in the same place in every house and the house building itself can be a castle or cottage or shelter or various things", with a Tiny Glade summary, and then a direction document: construction first (the Tiny Glade reference, `Tiny_Glade_Tool_and_Functionality_Reference_Consolidated.md`), then a first strong stylized style after Firewatch and Neva (`Draw3D_Style_References_Neva_and_Firewatch.md`), an editable palette of the world's colours at a scope the creator can see, and explicit save scopes for whatever is made to be reused. With it the joining tier 7co left for approval was approved ("proceed with the next pass").

**What was found first (inspected, not assumed):**
- *Already usable:* walls, fences, hedges and houses dragged out and rebuilt live with handles (7cl–7cn), the towers' rings and points (7cm), floors and stairs that follow a house (7co), paths that open gates and doors and growth that keeps clear (7ck), a structure's own colours and pictures (7cm, 7cn), colour variation at four scopes, by design, group, maker and one thing (`vary.rs`, the Varybox), a creature's or design's part colours in the Inspector (7cc), styles with colour roles, hours of light, tone, grade, cast, glow, shade and film (7bn), a style kept in the world's file, and a world saved, opened and undone as one document.
- *A foundation with its workflow incomplete:* a style's colour roles could be changed only by editing `styles.txt` (the Styles window had the picture's controls, not its colours); the renderer's haze had no glow toward the sun, no layer lying low, a fixed aerial distance, and the shade was the sky's light only; every house was the same sort, and every stair chose the same corner in houses alike.
- *Gaps:* walls and paths drawn up to a house stayed behind when it moved; no round building, keep, shelter or ruin.

**Built:**
1. **Houses of six sorts** (host `land::build`, the parameter `house`, `HOUSES`), each built its own way and every part of it still a setting:
   - *a house*, as before;
   - *a cottage*, a house given a low storey, a steep thatched roof, white walls, dark wood and flowers;
   - *a castle keep*: a turret at each corner (the wall's towers, roofed as said) and battlements round its top (a new piece, `parapet`), three storeys, stone, a flat roof;
   - *a round tower*: the ellipse inside the oblong it is dragged out as (`house_line`), so its handles push its sides as a house's do; a cone of a roof; windows in every third side; its stair wound up inside its wall (below);
   - *a shelter*: a roof on wooden posts, open all round, one storey;
   - *a ruin*: its walls broken off a metre or so at a time at heights of their own, higher at the corners, some pieces gone, its doorway a gap; no roof and no floors over its first.
   - In World outline (and the Inspector) under **A house of its sort**, a button each gives it a look and storeys to start from as one step; under Build > Draw on the land each sort has its own row, dragged out as a house is.
2. **Stairs of their own.** A drawn house chooses for itself which corner its stair is in and which of that corner's two walls it runs along, among those as good as the best (never along the wall with its door when another will do), by a number of its own (its node, mixed): houses alike have them in different places (six alike: four places in p79). The second flight, of a third storey, is at another corner whose stairwell the first does not stand on, chosen the same way.
   - **Its handle:** an orange ball over each flight. Dragged toward a corner, the stair goes there, along whichever wall of it the pointer is nearer, built again as it goes, one step (`stair_corner`, its places counted from 1, 0 the house's own choice; `stair_way`). "Let it choose where its stairs go" gives the choice back.
   - **A round tower's stair** is wound round inside its wall (a new piece, `winder`, walked up as a slope wound round), each storey's flight going on round from the one under it, its floor cut back to the flight's inner edge all along it.
3. **What is drawn up to a house follows it** (host `tileops::follow_joins`, after every edit, with the rest of `settle_drawn`): the end of a wall, a fence, a hedge, a border or a path that is at a house's side, or near it (a path to its door, 1.3 m), is joined to that side, so far along it and so far out (`joined`). The house moved, turned or a side of it pushed, the end goes where the same part of the side is now, built again as the handle goes and undone with it. An end moved by hand stays where it is put and is joined again where it is. Saved with the world.
4. **The air between the eye and what is far** (a style's `look` lines, `air.gd`, `clock.gd`, using Godot's own fog): `haze` (how much it glows toward the sun, how high a layer of it lies and how thick it is under that), `distance` (how much far things lose their colour to it), `shadow` (the colour the shade is lit by and how much of it, in place of the sky's light), `soft` (how soft the sun's shadows are). Each has a control in Window > Styles for a style of one's own.
5. **Two styles to build in** (`styles.txt`), original, after the references' lessons and not their art:
   - **lookout**, after Firewatch's light: a conifer wilderness, teal woods and golden grass, red-brown bark, warm amber sun late in the day, cool blue shade, haze lying low and glowing toward the sun, ridges fading by layers; a world begun empty (World > New world, an empty island or nothing) begins in it;
   - **storybook**, after Neva's palette: fresh mint and gold greens, pale trunks, pink and lilac flowers, soft low-contrast light with a bloom, lilac shade, a watercolour haze, a little grain.
6. **The palette** (Window > Styles, host `palette`): every colour a style colours the world with, by what it is for (walls, trim, stone, wood, roofs, windows; bark, leaves, greens, flowers, fruit, rocks; grass, cliffs, sand, under water, roads, the sea), each a swatch. In a style of one's own each is changed with a click and seen at once on everything that takes it; a colour put into a list (+) or taken out (−). What a change reaches is said there:
   - **a role** of the style: every house, tree, flower or patch of ground that takes it;
   - **one thing's own** (a house's or a wall's colours in Makers > Its look, a creature's parts in the Inspector, colours varied per design, group, maker or one thing): that thing's, kept when the style's change.

**Save scopes, as they are (inspected; nothing claimed that does not save):**
| What | One thing | This world | Every world |
|---|---|---|---|
| A style, its palette and its air | (a thing's own colours, above) | kept in the world's file while it has the style (Ctrl+S), undone like any edit | `styles.txt`: a style of one's own made there (Make my own) is offered to every world |
| A house's sort, its stair, its look | its own settings, saved with the world | | not yet (a house's look kept as a design to reuse: a gap) |
| Moves, combos, gear, buttons (7ba–7bh) | a character's set | the world's kinds | buttons and profiles in `%APPDATA%/Draw3D` |
- Not yet: a style or palette saved to a library outside the project (`styles.txt` is the project's file), a palette apart from a style, a structure's look saved as a reusable design. The shared step those need is a library that is the person's own, outside the project, with the project's copy kept apart from it; recorded for the audit after construction.

**Checked:** host unit tests 70 of 70; protocol tests 80 of 80 (p79 new: six houses alike have their stairs in at least three places, none along the door's side; a stair said to a corner and wall goes there, the other wall elsewhere; a round tower has two wound stairs at three storeys; a keep four turrets and battlements; a shelter posts and no walls, one storey; a ruin broken walls of stone, no roof, no floors; back to a house as it was. p80 new: a wall's end and a path's end follow the house moved and a side pushed out, undone back, and an end moved by hand stays and is no longer joined); `-Demo sorts -Check` 8 of 8, no script errors: the six sorts built (a keep's 4 turrets and 12 battlements, a tower's 2 wound stairs, a shelter's 12 posts, a ruin's 28 broken pieces and no roof), four houses alike with their stairs in three places, a stair dragged to the corner across (its corner 3, the other wall, kept when saved and opened), a wall's end and a path's end moved with the house (3.1 m, -2.0 m, as the cross was dragged), lookout, storybook and none on the same place with woods and a meadow (each style's haze, distance and shade read back), a palette of one's own (40 swatches; the first roof made blue and the first leaves rust, seen on the roofs and the crowns), and the world saved and opened again with every house's things as they were. Looked at in its pictures, not used by hand.

**Not done, owed to the construction workflow** (the Tiny Glade reference's operations, by what matters most to the core loop):
- houses that touch joining their walls and roofs; a wall put across a house shortened to it;
- windows placed on walls by hand (a low one a door), grouped in twos and threes;
- paths as a brush, with arches through walls and stepping stones across water;
- stairs as a route of their own outside (steps, platforms, ladders, bridges, round a tower);
- a house's base raised on stilts or a foundation by its own handle; a roof turned or bulged by dragging;
- cutting a wall in two and joining two walls;
- style clone (one structure's look given to another) and a structure's look saved to reuse;
- group selection and moving several together.

## 7co. A drawn house's floors over its first, and its stairs (built 2026-10-19)

**Why.** After 7cn, the user said "a two story house still only has one floor inside", and added, as a standing direction: "modular doesn't just mean a house or a wall. systems should connect seamlessly. when one adjusts the others should adapt." A house drawn on the land (7bu, 7cn) was given a facade for each storey but no floor over the first and no stair: the floors and flights of 7bv were only for a house on cells. Two storeys looked right from outside and were hollow inside.

**Built:**
1. **A floor for each storey over the first** (host `land::build`, kind `upper`, `inside_f1`, `inside_f2`...):
   - boards the shape of the drawn line, at the top of the storey under;
   - each with a stairwell cut out of one corner (the outline with that corner notched).
2. **A stair up to each** (kind `stair`, `inside_up1`...):
   - a flight along a wall from the floor under, coming up through the stairwell;
   - in a corner that is square enough (to about 17°), turns in, and has a side long enough to run it along;
   - not along a side with the door in it while another corner will do, and never over the doorway;
   - the flights go up by turns at two corners, so each stands on floor and not over the stairwell under it.
3. **Furnishing on every storey** of a drawn house now (only the ground floor before). Nothing stands on a stair or in a stairwell.
4. **Each part adapts.** All of this is built from the line every time, so it follows when the house changes:
   - its storeys (one storey: none; three: two floors, two stairs);
   - their height, set by the blue point (the floors go up with it, and the stairs are steeper);
   - its sides, moved by their arrows or their dots (the floors as wide as the house).

**The standing direction, as it stands.** Already adapting after every edit (`tileops::settle_drawn`):
- a path opens a gate or a door it reaches, and it closes again when the path goes;
- planting keeps clear of what is built and of the ways through it.

Not yet:
- a wall drawn up to a house, or a path to its door, does not follow the house when it moves;
- two houses that touch do not join (their walls and roofs stay two);
- a wall does not shorten for a house put across it.

Those are the next tier and await the user's approval.

**Checked:** host unit tests 70 of 70; protocol tests 78 of 78 (p78 new: two storeys give one floor and one stair, its floor about a storey over the ground floor, its stair not along the side with the door; three give two of each at two corners; taller storeys lift the floors; a side pushed out widens them; one storey removes them); `-Demo houses -Check` 7 of 7 (a floor and a stair counted at two storeys, none after the undo; seen from above with the roof off). Not used by hand.

## 7cn. A drawn house's own look, dragged out, and its handles (built 2026-10-19)

**Why.** After 7cm the user said "proceed with the next step". In the comparison document the next tiers were buildings with handles, paths as a brush, then windows, and the user had said with 7cm that they will want to restyle anything they don't like, with handles. A house was the one drawn structure with no look of its own and no handles: clicked round, coloured by the world's style, its roof always gabled.

**Built:**
1. **A house's own look.** Host `land::build::LOOKS`, now 12 words, carried in every piece's brief:
   - New colours, rrggbb: `wall_colour` (its plaster), `trim_colour` (round windows and doors, the band under the eaves), `pane_colour` (its glass).
   - Already there, now used by a house too: `wood_colour` (doors, timber framing, floor boards), `roof_colour`, `stone_colour` (footing and plinth).
   - Pictures: `texture` (over its walls, not over windows or doors) and `roof_texture` (over its roof), by the `texture` line.
   - `land::build::looked` lays what a brief says over the world's look. `shapes::made` calls it for every piece, so walls, windows, doors and roofs are made in it.
   - The facade generator (`WalledFacade`) declares these words. It passes them to the wall, windows and door it makes through a new core call, `Ctx::give_children`, which adds values to the children planned so far.
2. **Its roof.**
   - `roof_form`: two slopes and gables (as before), four slopes (hipped, its ridge as much shorter as the roof is wide), or flat with a rim. Each has a solid to match.
   - `roof_rise`, the towers' steepness word, is now a house roof's too, carried as `pitch`.
   - A house not square-cornered keeps the roof that rises to a point.
   - A finished project's roof (`real`) keeps its own gable unless another form is said.
3. **Its windows** (`window_every`, metres; 0 is 2.2 m as before).
   - A drawn side is in pieces of up to 3 m, each piece a facade. Where windows are further apart than a piece is long, each piece gets one window in its middle or none (`piece_bay`), so they fall about that far apart along the whole side. Before this, a piece longer than its spacing had none at all.
4. **"Its look" for a house** (`makers.gd` `_build_house_look`, in World outline and the Inspector):
   - six palettes plus "as the world's style", each one step;
   - a colour picker for each part, with x;
   - the two pictures and how often they repeat;
   - how the roof is made and how steep;
   - how far apart the windows are.
5. **Dragged out** (`marks.gd` `box_places`): Build > Draw on the land > A house is pressed at one corner and dragged to the other.
   - It is built as it goes (staged) and is square to the view, to 15°.
   - It is at least 3 m each way, and goes round (its door in the first side, as before).
   - Let go, it is one step and is chosen. Clicks still put corners for a house of any shape.
6. **Handles on a drawn house** (`line.gd`, Tiny Glade's set):
   - a white cross in its middle moves it;
   - a white ring off a corner turns it, by 5°;
   - an orange arrow on each side pushes that side out or in (it stops 1.5 m short of the side across);
   - the blue point over a side sets each storey's height (all storeys together);
   - a blue point over its roof sets its steepness;
   - its corners' dots, "+" and Alt+click, as a wall's.
   - Each handle is built again every frame (staged), is one step on letting go, and Esc gives it up.

**Found and mended:**
- **A closed line came open.** Dragging the first dot of a drawn line that goes round (a wall round a yard, a house) moved its first place and left its last, the same place, behind, so the line came open. The two now move together. This dates from 7cl.
- **A mesh lost its colours.** A house part written out as a mesh (M3's "change the roof that was generated") was made without what its structure says of how it looks. It is now made with it, and the `shape` answer says the look it was told (`look`).

**Not built (Tiny Glade has it):**
- stilts, and a roof that turns or bulges when dragged;
- buildings that snap together and join roofs;
- windows placed and stacked by hand;
- cutting and joining walls;
- paths as a brush.

The wood colour also paints the timber framing, so doors cannot be coloured apart from it. A palette's pictures darken its colours, since the picture multiplies them, so roof tiles over red look brown.

**Checked:** host unit tests (1 new: a house's own colours, its three roofs, a steeper roof taller, pictures over walls and roof and not windows); protocol tests (p77 new: the look reaches its windows, door and walls through their facades; windows further apart are fewer but on every side; one step, one undo); `-Demo houses -Check` 7 of 7; `-Demo walls` 6 of 6 and `-Demo looks` 7 of 7 as before. Not used by hand.

## 7cm. A drawn wall's look, its towers' handles, a place put in or taken out, snapping (built 2026-10-18)

**Why.** The user tried nothing of 7cl by hand yet, but found it "closer to the goal" and asked for the next step, with this in mind: they will want to change the palette, or at least restyle the objects themselves (if they don't like how the towers' roofs look, and so on), with handles to customize all of it, and textures where they apply. Until now a drawn wall took every colour from the world's style, its towers always had the same cone, and nothing about how it looked could be changed for that one wall.

**Built:**
1. **A structure's own look** (host `land::build`, `LOOKS`; carried in every piece's brief, so each piece is made again when it changes). Each is "" or 0 for as it was:
   - colours rrggbb: `stone_colour`, `cap_colour` (the coping, the piers' caps, the band round a tower's top), `wood_colour` (fences, posts, gates), `green_colour` (a hedge), `roof_colour` (its towers' roofs);
   - `texture` and `texture_repeat`: a picture of the project's textures laid over its pieces (not over a gate, water, a floor, steps or a path), by the same `texture FILE S` the designs have had since 7cc; its colour tints it;
   - `roof_texture`: one over its towers' roofs;
   - `course`: how high a course of its stones is (bricks small, rough blocks large).
2. **Its towers, said** (`tower_roof`, `tower_wide`, `tower_rise`, `roof_rise`):
   - roofed with a cone, a tall spire, a dome, a flat lid, battlements of stone (no roof), or left open;
   - how wide (0: by how thick the wall is), how far over the wall they stand (1.3 m as before), how steep their roofs are.
   - A tower's roof is now a piece of its own (`towerroof`, "tower roof"), so it has a picture of its own and is chosen and taken away as one.
   - A tower has a floor over its top now (it had none: the cone covered it).
3. **"Its look" in World outline and in the Inspector** (under "How it is built", `makers.gd` `_build_look`):
   - a palette: as the world's style, grey fieldstone, warm sandstone, red brick, whitewashed, mossy ruin, dark slate; each sets its colours, pictures and stone size together, as one edit;
   - a colour picker for each of its parts, seen as it is chosen (`livecolour.gd`), and "x" for back to the style's;
   - the picture over it and over its towers' roofs, how often it repeats, the size of a course;
   - with towers: how they are roofed, how wide, how high over the wall, how steep.
4. **Handles on its towers in the view** (`line.gd`): a green ring round each tower, dragged out or in for how wide they all are; a green point over each, dragged up and down for how far over the wall they stand. Built again every frame as they go (staged edits); each drag one step of history; Esc gives it up. The host says where its towers stand (`towers_at`) and how wide they are by default (`tower_usual`).
5. **A place put in, one taken out** (`line.gd`): a white "+" lies on the wall half way between each two of its places. Pressed, a place is put there and held; dragged, the wall is built again as it goes; let go, the place and the drag are one step. Alt and a click on a place takes it out (two stay at least): one step.
6. **Snapping** (`line.gd` `snapped_to_walls`): the ends of a wall dragged out, and a place of a wall dragged, go onto another drawn wall near them: onto its end within 1.6 m, onto it along its length within 0.9 m. Ctrl held, they don't.

**Not built (Tiny Glade has it):** cutting a wall in two and joining two; buildings with handles; paths as a brush with live arches; windows placed on walls; small things linked to windows and doors. Not here either: a look for buildings (their own walls, roofs and windows keep the world's style), and a visible cue while a place is snapped.

**Checked:** host unit tests (1 new: a wall's own stone colour, a picture over it and not its gate, the four roof forms made, taller in the order spire, cone, dome, lid, battlements taller than a cone's tower, no faces that fight); protocol tests; `-Demo looks -Check`; `-Demo walls -Check`. Not used by hand.

## 7cl. Walls drawn by dragging, built live, with handles (built 2026-10-18)

**Why.** The user found 7ck, and 7bu before it, did not build the way Tiny Glade does. Both drew with clicks and Enter, showed nothing until the line was finished, and had no handles on what was drawn. The user pointed to a video transcript of Tiny Glade's tools, the developers' Rust/Bevy talk and Bevy itself. What was read, what was compared and what Draw3D lacked are in `Draw3D_Tiny_Glade_Comparison.md`. The developers' talk itself was not seen. The user chose to measure the round trip first, then build drag-to-draw walls with handles that rebuild live while dragged.

**Measured first.** A drag step is the host's existing staged edit: shown, not kept until a commit, and dropped without trace (the host has had this since its drags of houses). One step costs:
- the whole wall made again;
- the cells and content sent;
- the meshes built in Godot;
- then drawn.

Results:
- **Answer:** 3–7 ms.
- **Ask to drawn:** about 7–10 ms on average, worst 24 ms (`-Demo walls`, last beat).

So the live preview is the real wall built by the host every frame. There is no stand-in drawn in the editor.

**Built:**
1. **Drag to draw.**
   - Applies to Build > Draw on the land: a stone wall, a fence and a hedge.
   - Pressed on the land and dragged: the wall is made again as the pointer goes, once a frame. It is a staged `compose` of its line and itself.
   - Let go: kept as one step of history (`commit`); Esc gives it up (`drop`).
   - The stroke is kept as the few places it bends at. Douglas–Peucker keeps it within 0.35 m, and no two places are nearer than 1.5 m, so a hand slowing at a corner does not leave two.
   - Ended within 1.5 m of where it began, after going 6 m, it goes round.
   - The pointer reads the land only (`land_at`), never the wall being built under it.
   - A click without a drag still puts places one at a time, Enter ending them, as before.
2. **A curve through the places** (host, `build::curved`, the structure's `curve` 1). Lines dragged out are given it.
   - The wall passes through every place on a centripetal Catmull–Rom curve, in steps of at most 2 m.
   - A place where the line turns by 55° or more stays a corner, which a wall with towers gets a tower at. So does each end of a line that does not go round.
   - `tileops::drawn_line` gives the curve, so paths crossing it, gates and keep-clear (7ck) all see the curve that is built.
3. **Handles on a drawn wall, fence or hedge chosen** (by clicking any piece of it, or in World outline). They are a second instance of `line.gd` (`walls_line`), kept apart from the Inspector's.
   - **Its places:** orange dots on the land, each dragged with the left button. The wall is made again each frame as a staged `param polyline` on its line, and letting go commits it as one step.
   - **Its height:** a blue point over its middle, dragged up and down (on an upright plane facing the eye). It is a staged `param height` on the structure, one step on letting go.
   - Esc while holding either: the wall is as it was.
   - The host now says of a drawn structure its `curve`, its line's node (`route`), the line's places (`controls`, `route_at`, `route_yaw`) and the line as built (`built_line`).

**Found and mended on the way:** an open drawn line measured each place's turn against the side two before (`drawn_rim`). Its second place was measured against the line from its far end back to its start, so an open wall could get a tower near one end. Lines that go round were right. This dates from 7bu.

**Not built (Tiny Glade has it):**
- snapping to other walls, with Ctrl to turn it off;
- cutting a wall in two, and joining two;
- a "+" on the curve for a place;
- buildings dragged out to a size, with handles on each side, roof and stilts;
- paths as a brush with a width, with live arches, steps and stepping stones;
- windows placed and stacked (doors when low);
- small things linked to windows;
- the tool staying in hand after a stroke. Here, letting go chooses the new wall so its handles show at once.

These are the next tiers in the comparison document, each for the user's approval.

**Checked:** see PROGRESS (Done, "Walls drawn by dragging").

## 7ck. Places drawn that know one another: paths that open their way, growth that keeps clear, a pond with life (built 2026-10-18)

The user's brief of 2026-10-18, "Integrated, Context-Aware Modular Construction", resumed construction with Tiny Glade's interaction design as the reference: the person chooses a tool, draws, drags, paints or places, and Draw3D interprets the action in its spatial context and gives a coherent, editable, undoable result. Buildings, gardens, ponds and paths as one workflow, not separate features; audit first; the most valuable vertical slice, not a superficial pass over everything; no showcase logic; tests that use the normal tools.

**The audit** is in `Draw3D_Modular_Construction_Feedback_and_Next_Systems.md`, section 6, with what was there, the friction points in order, the decisions and what is owed. In short: drawn structures (7bu) each planned alone; growth and the brush kept off roads and water and nothing else, so a meadow grew through a drawn house and its doorway; no path along a line existed; a pond made with the tools had no life, and the frogs, fish and lilies of the island town's pond were put by its script (`examples/town.rs`), of designs only `town.txt` had; one drawing was two undo steps.

**What is built where, one text the host keeps** (`shapes::land::BUILT`, `built.clear`; `Kept`, `Clear` in `land.rs`; written by `tileops::settle_drawn` after every edit, as `tiles.raised` is): each drawn house's, pond's, garden's and paved yard's inside, every drawn wall's foot along its line, every path, and every way through (a gate or a door and 2.6 m of ground either side), each with whose it is, in the land's frame. Woods, meadows and fields keep clear of all of it; a structure's dressing, a garden's rows and a pond's bank life of what is not its own; nothing is planted in a way through, its own included; the brush (`stroke`) keeps clear as it paints and says how many places it left (`kept_clear`).

**Paths, and the ways they open** (`build.rs`, `tileops::settle_drawn`): drawn along a line that does not go round and built as nothing round it with a floor, a structure is a path: a strip `wide` across (1.4 m when not said) that lies on the ground in pieces mitred where it bends (`pathway`: four corners and their heights, a hull to walk on). Where a path crosses a drawn wall, fence, hedge or garden edging, the piece it crosses is opened (a gate; an edging, a gap); where it crosses a drawn house's wall, or ends within 1.2 m of its side, the piece is a door. What is opened, and by which paths, is written on the structure (`opened`, `opened_by`), apart from the gates a person asked for (`line_gates`), which are never touched. Move the path's line and the openings move; take the path away (or undo it) and they close; `opens 0` keeps a structure shut to paths. The `nodes` answer says `opened`, `opened_by`, `opens`, `path`, `border`, `wide`.

**Gardens beside what is built:** a garden drawn along a line and not round is a border (a strip of soil, `pathway` pieces of soil, planted in rows, no edging along its middle); a garden drawn round a house (or a pond, or a yard) plants round it and lays no soil under it (`Clear::round_another`). A garden's gateway has no leaf of zero height any more.

**A pond's edge and its life** (`build.rs`): `edge bank`, a low broad bank (`BANK`, 2.6 m across its foot) of the land's own grass a hand above the water, its inner slope running down under it, with mounds where it turns; still standing on the land (7af: never dug). `life` (designs in its water), `bank` (designs along the outside of its edge) and `lively` (0 to 1): in the water what swims (`swims R H`) is put so far under the top as it swims over where it is put, what floats (`floats`) on the top, anything else on the bed; swimmers not over a bank's slope. Each is a thing of the pond (`life#...`), kept by hand as any is. The designs: the library's new group "Pond and water's edge" in `rural.txt` (lily pad, water lily, reeds, bulrushes, frog, goldfish, duck, dragonfly, two butterflies), written from the town's and made to float or swim as the pond asks. Sets of them in `editor/godot/ponds.txt` (lively, quiet, fish); a set chosen brings its designs from the library.

**In the editor:** Build > Draw on the land has "A path", "A pond with life" (bank, set lively), "A pond with a stone kerb", "A garden: draw round it for beds, or along a wall for a border", "A paved yard". One drawing is one step of history (`makers.gd` `drawing_mark`, `drawing_done`; host `together`, 7ce), also when the host answers before it has finished (`marks.gd` `_merge_when_done`). A drawn structure is named for what it is (house, stone wall, path, border, garden, pond, yard) where it was "structure N". World outline shows, for a drawn structure, a path's or border's width, "paths drawn across it open a way (a gate / a door)" and what opened it, and for a pond its edge, a pond-life set, its two lists of designs and how lively it is (`_build_context`).

**Decisions taken to build it** (the user's to overrule; section 6 of the construction record says why): a path opens what it crosses by itself, with a switch per structure; a path's end near a house's side opens a door there; a path across a house is two doors; what a person put by hand or painted is not taken away by something built over it later; ponds stay raised; only drawn structures take part in this tier.

**Found and mended on the way:** a drawn structure's dressing never came to the first of its words ("rocks" of "rocks grass ferns": a drawn pond had grass and ferns only, since 7bu): each place took its word by the piece's name's length, and drawn pieces' names are all as long and hold two places at most; drawn, the words now go in turn along the whole line (structures on cells as before). And `-Tour -Only build` chose "built as" by its place in the list, which 7bu's garden had moved: its "only its floor" step picked a garden; it now picks by the word.

**Continued the same day, at the user's word ("put structures on cells into what is built where, so meadows keep out of village houses too"):** `settle_drawn` also reads every structure on cells, from what `settle_makers` has just told it (`over`: its cells in its own frame; `rim`: its sides): each cell of a house, a pond, a garden or a floor alone (a path, a yard, a square on cells) is inside it, every side it builds and every wall inside has its foot, every gate and door is a way through. Woods and meadows told tiles (`on`) keep clear of them as those over a reach do. Worlds built on cells (the village showcase, the shelter, the genre worlds) open with nothing grown in their houses, on their paving or in their doorways any more: what grew there before was the fault this mends, and nothing a person put by hand is touched. Checked: protocol p74 (a house of four cells with a door, a path of paving on four cells, a meadow over both: nothing in the house, the doorway or on the paving; 70 in the house with the change taken out; the house taken away, the meadow grows there again).

**Owed** (section 6): a path's end drawn to a door, paths that join; a pond dug into the land (a decision: 7af says never); steps where a path climbs; a garden painted with the brush; pieces of the person's own; a preview while drawing.

**Checked:** host unit tests 66 (two changed: a piece a path crosses is opened; a path's piece and a bank are made with nothing in one plane); protocol 74 of 74 with p74 (below), p73 new (a house, a wall round it, a meadow, a path through the wall to the house: the wall opened in s0_3 by "the path", the house's door in s0_1; nothing of the meadow in the house, on the wall's foot, on the path or in its ways; the brush kept clear; undone back past the path, gate and door gone, again; the path moved: s0_5 and s1_0; `opens 0`; a border of soil and plants; a garden round the house with no soil under it; a pond with a bank and life: fish under the lilies' top, frogs outside; written and opened again the same); p43 with the new group. `-Demo glade -Check`, 11 of 11, every step through the Build entries and the line tool as a person's clicks: see PROGRESS. Not used by hand.

## 7cj. The watchtower made fit to show: a controller first, a dressed ranger, a real tower, life and a style (built 2026-10-18)

The user played 7ch with a controller and judged it not good enough for a commercial demo, "not at all": an unclothed model with no gear; an axe that had to be put in hand from the inventory though the hands were empty; old side-steps still on the shoulder buttons and no button to swing the axe; a pause menu the controller could not work; wolves far too fast; a tower that looked like a house; nothing alive but the wolves; no style; health drawn as # signs. And that the testing had not been the kind a completion could rest on. All of it was so: 7ch's checks counted what was done and never judged what a player sees and presses. The user's standing rule from it: **a controller is the default way to play and test** (unless a game is declared one of pointing and clicking), its buttons laid out logically, every screen workable by it.

**Controls, a controller first** (`controls.gd`, `moves.txt`, `player.gd`, `aim.gd`, `studio.gd`): a new button, **attack** (X, RT, left click), which every blow asks (the fist's `strike`, now with or without a foe before it, and the sword cuts); A jump, B dash and dodge, Y act, LB lock on, RB run, LT crouch (and aim), right stick pressed puts away; the side-steps on Z and X only; use on the D-pad's left. With a controller plugged in, every prompt and hint shows its buttons from the start (`main.gd`). A person's own buttons (`%APPDATA%\Draw3D\controls.txt`) still lie over these.

**The game's screens by a controller** (`gameui.gd`): the D-pad or the left stick goes from choice to choice, A chooses, B goes back, whatever Godot's own ui actions are; a gold frame and gold words round what is chosen (`screens_theme`); "A choose  B back" under the menu; the controls page lists only the device in use, by names, not Godot's numbers.

**An axe in hand at once** (`items.gd` `first_use`, `things.gd`): gear picked up goes into empty hands, or onto a place nothing is worn on; **hew**, an axe's own blow (`when wields axe`, drawn as it is wound up), shown by UAL's sword attack (characters.txt, look scout).

**A dressed ranger** (`outfitmake.gd`, `characters/ranger.glb`, `character ranger`): Quaternius's CC0 ranger outfit and the head of his base character, made one file on UAL 2's bones; downloaded with the user's yes; a world makes him its walker by its own characters text (the watchtower does). **Found and mended:** a model whose meshes carry colours in their corners was shown in them and its textures thrown away (`animated.gd` `_paint`): the ranger came out white.

**A real axe** (`characters/kaykit_weapons`, KayKit Fantasy Weapons Bits, CC0, within the 2026-10-16 approval of weapons): `model PATH turn x y z` in gear.txt (`gearbook.gd`), so a file made upright lies as gear lies.

**A real tower** (library/rural.txt `watchtower-keep`, the Woodland shelf): a square keep of pale dressed stone, 6.8 m across and 9.7 m high, its stone stair climbing round the outside of its walls (a landing at each corner, a breast-high wall along it) to a gap in the battlements 8 m up: one design, written by `library/watchtower-keep.js` from a few numbers; each flight is walked as one slanted slab (`solid hull`) under its seen steps. The stair is outside because within a 6 m room a third-person eye has nowhere to be, and the first try (a stair round the inside) let the walker fall into the well. The house-built tower (7ch) is gone from the level; why the Build tools could not make this is the benchmark's G31.

**The walker climbs steps** (`player.gd` `_step_up`): it stepped up nothing; now, on foot and in no move, up to 0.3 m, as the world's other walkers do (first written with a wrong test of "in no move", which never held; found when it did not climb).

**The display** (`fight.gd`, `stats.gd`, `watch.gd`): health and breath as bars (a hurt foe's bar over it, facing the eye); the stealth meter only where going unseen is part of the game.

**Wolves** (`moves.txt`): run 5.6 m/s (the walker runs 8.5, walks 4.4), slower to turn, a bite gathered 0.4 s.

**Life and a style:** the library's deer, rabbits and a fox; dead trees that sing (`hums birds`); a style of the level's own, `golden_valley` (warm low sun, deep greens, a rose sunset as the tower is climbed, a blue dusk when the beacon is lit).

**Checked:** see the benchmark's part E (the polish bar, judged screenshot by screenshot) and PROGRESS.md. Not played by hand; no real controller on this machine (the presses are sent as one sends them).

## 7ci. A future idea: guided recreation of a finished level (recorded 2026-10-18; not built)

Recorded at the user's asking with the wilderness benchmark (7ch). Nothing of it is built, and it is not to widen the benchmark's scope.

**The idea.** Draw3D could one day load a reference level and walk a person through making it again themselves, in the real editor: it asks for a piece of terrain, says which tool to open and which settings matter, the person does it, says it is done, and the guide goes on to a building, the woods, the characters, their behaviours, the interactions, and the rest. Where the person finds that an option the guide names is missing, or that the words do not match the interface, they say so: the guide records that limitation and helps them carry on where it can.

**What it is for.** Finished benchmarks become practical, interactive tutorials that teach how a whole world is made with Draw3D's own tools, step by step, in the order the work is done. Each run of a guide is also a test of the interface against its own documentation: every "this is not here" a person reports is a finding with the step, the tool and the words that did not match.

**How it relates to what exists.** The benchmark's recreation guide (`Draw3D_Wilderness_Adventure_Benchmark.md`, part C) is already written as steps, each with its goal, where in the editor, the settings, what is seen when it is done and what it depends on: that is the content a guided mode would step through. The in-editor checklist (`checklist.txt`, `checklist.gd`) is the nearest thing that exists: a list of what to try, ticked by hand. A guide would be that list made into steps with a reference world beside it, and a place to say a step could not be done.

**Shape, if it is taken up** (for the user to decide, nothing assumed): a guide is a text, like the checklist's, of steps (goal, where, settings, expected, depends on); a window shows one step at a time with Done, Back, and "This is not here / not as it says"; the reference level can be opened beside the person's own world, or its parts looked at in the Inspector; what is reported is written to a file kept with the guide, for whoever keeps the interface. It should work as a structured workflow without an LLM; later, the assistant (7ce) could read the guide and the person's world and answer "what next?" or "why does mine not look like that?".

**Not decided:** whether a step is checked by the engine (a structure of storeys exists, a quest is in quests.txt) or only ticked by the person; whether the reference level opens beside the person's world or replaces it for a moment; whether reports go to a file or into the guide itself.

## 7ch. The wilderness benchmark: the Forgotten Watchtower (built 2026-10-18)

The user asked (2026-10-18) for a small, complete, polished third-person wilderness adventure made with Draw3D's existing authoring systems, as a benchmark of what the engine can make and to expose its gaps: a trailhead with the destination in view, a trail through a composed forest, an abandoned camp with supplies, creatures guarding the way, equipment found, a watchtower whose beacon is restored by the engine's own interaction and outcome systems, and a clear ending. The LLM work is a separate workstream and not a dependency. After the benchmark, the user's queued item is the assistant (Qwen) making a vase-shaped sign that added a library entry with no visual and placed nothing.

**The level** (`editor/genres/watchtower.json`, built by `editor/host/examples/watchtower.rs` with the host's own commands, as the other genres are; World > New world, "Genre: The Forgotten Watchtower"): a valley flattened and raised by the land tools; a dirt trail painted on it; eight woods of Generator-grown and library trees, each given its designs; a forest floor and a trail edge painted with two groups; logs, boulders, outcrops, toadstools, a cairn, standing stones; the trailhead (start, the ranger, a signpost, a waymark lantern); the woodcutters' camp (a cold fire, a tarp, crates, the camp chest of lamp oil, food and medicine, an axe and a backpack); two pairs of wolves; a stone tower of two storeys on four cells whose walls are each given a stone design of their own, with a lookout and the beacon on top. The whole of it, its systems, how to make it again and what it found: `Draw3D_Wilderness_Adventure_Benchmark.md`.

**Added, general:**
- **A game's hour and its ending** (host `land.rs` Start: `hour`, `ends`, `ending`; `nodes` rows; `makers.gd` `ending_rows`, `set_ending`; `inspect.gd` the start's part; `gameui.gd` `begin_run`, `_ask_ending`, the ending screen, `play_again`; `main.gd` `play`): the hour a game begins at, and a condition in `cond.gd`'s words that ends it. While it holds the game is held at an ending screen with the start's words and how long it was played: Keep exploring, Play again from the start, Back to editing (in a game: Leave). The world's fact `ended` is put. In the Inspector at the flag and in World outline.
- **The quest step wraps** on the display when it is longer than about a third of the screen (it ran off the left edge with a hint after it).
- **`genres.ps1 -Only NAME`**: one genre built again, no other (`-Over` rebuilt all six and their backups).
- **Found and mended: an exported game began where the world says** (`main.gd`): the export's writer and the game's boot read the world's start from whichever host answer came last after opening it, so every exported world began at (0, 0); and `play` now uses the Player start in a game too (its eye and stamina).
- **Found and mended: the why-not prompt** (`acts.gd` `_wants`) skips an act its `while`/`unless` rules out (a beacon kindled said it wanted lamp oil).
- **`lights … glow R`** (`things.gd`): how big the glow over a light is.
- **Found and mended: a piece given a design of its own kept its height** (host `land.rs` `by_hand`): it was stood on the ground beneath it, so a building's upper storey given its own designs came down onto the ground floor (the tower's parapets across its archway). Test p72.

**Added, content:** goods.txt `lamp_oil`; acts.txt `kindle` (hands a lamp oil to the thing, puts `kindled`, heard 10 m off); sounds.txt `fire` (a loop) and `kindled`; quests.txt `watchtower_beacon`; talks.txt `ranger_beacon`; the library's Woodland shelf (Fallen log, Tarp shelter, Beacon brazier, Waymark lantern, Stone tower wall, Stone parapet, Stone doorway, Stone tower corner), People's The old ranger, Animals that hunt's The grey alpha.

**Found, not built (the benchmark's gap record):** a building's second flight of stairs goes into its wall when the stairwell's row has no further cell (the tower is two storeys; construction is postponed by the user); the only humanoid models are mannequins (Character Maker tier 2 waits on the user's choice of assets); a world's own text is replaced by the project's file of that name if that file is saved while the world is open, and quests, talks and acts are the project's, not a world's; what lights by a condition (the beacon) is lit only within three of the view's 32 m cells of the eye, so not from the trailhead 126 m off, and its glow is a ball 0.26 m across; a colour named `course` made a library collection unreadable with the error naming another design (not explained); twenty wall pieces were given their design one at a time (no "all the walls of this structure"); overlapping raise dabs add up and a top flattened above its surroundings stands as a step at its rim (the walker stopped short of the tower; the way up is now graded and the builder fails on a slope over 0.5); a creature that bites more often than the walker is spared after a bite (0.8 s) chains them down (the alpha and the valley wolves now bite every 1.6 s); a building's flight rises a storey over 0.7 of a cell, so in cells under 4 m it is steeper than the walker climbs, and nothing says so (the tower's cells are 4 m); the scripted fighter, not a person, judged the wolves fair.

**Checked:** see PROGRESS.md, "The Forgotten Watchtower" under Done, and the benchmark's part E. Not used by hand.

## 7cg. When this happens, do this: the catalogue's third tier (built 2026-10-18)

Asked for after 7cf as the next of the catalogue's tiers (7cc, the plan's item 3): what already happens to a thing wired to something done.

- **`on TRIGGER [WHICH] do ACTION ...`** (host `kind.rs` TRAITS, OWN, `ON_HAPPENS`, `ON_DOES`, `on_said`; `things.gd`; `wires.gd`, new). A design, or one thing put down by hand, may say as many as it likes. What may happen is what already happens, told where it happens: `senses` (something loose comes into the room it senses, `loose.gd`), `hits` (a strike it counts, `loose.gd`), `spots` (it sees the walker and goes after them or raises its alarm, `watch.gd`), `act [ACT]` (an act done at it, at the act's moment, `acts.gd`; ACT an act's name or the first part of it, so `search` is `search_scrap` too), `answer [WORD]` (the walker answers it in a talk, `talk.gd`; the talk's name, the point's, or the answer's first word). What may be done: `put FACT [V]`, `give GOOD [N]` (to the walker), `say WORDS` (told the walker within 12 m), `sound NAME` (heard where it is), `move MOVE` (its own if it has a body, otherwise the walker's). Only while the world runs; each done is written in what happened ("The toolbox, on act search: gave you 2 coins").
- **The catalogue** (`abilities.txt`, `abilitybook.gd`): `happens NAME [needs WORD] [which WHAT] : WORDS` and `does NAME TAKES : WORDS` list what may happen, what it needs said for it to happen, and what may be done; a new group "Does when something happens" has the ability `wired` ("when this happens, does this"). The host's `words` answers `happens` and `does`, and the Inspector says when the engine has one the catalogue does not list.
- **In the Inspector** ("What it can do", in HOW IT WORKS): under "when this happens, does this", a row for each line, reading "when [act] [search] do [give] [coin 2] x": what happens and what is done chosen from lists, what it names and its words written; a doing chosen begins with words of its own. A row whose trigger the thing does not say what it needs for (no `hits`, `senses`, `sees`, `offers`, not someone) says so in orange. "When this happens… (one more)" adds a row. Beside a row of an ability that is itself something that happens (sees the walker), **then…** adds a row for it, as `while` is beside what may be held. Each change is one step of undo; what the host would not take is refused with why; the x of the ability takes every row.
- **Playtest** lists each line done this run, by whom and how often ("When this happened, done").
- **The assistant** is told of it (`words` abilities lists the happens and does lines; `give` with an `on` line), and a line whose trigger the design does not say what it needs for comes back as a problem ("'hits' will never happen: the design says no 'hits'"), so it gives that too before Apply.
- **Host test** p71: every line kept, two of one trigger both; eleven wrong lines refused with why; one thing's own lines; `words`.
- **Found:** the Inspector is put away while playing (it is for editing), so what a row says while playing is never seen: the count went to Playtest instead. 7cf's "(now: yes / no, why)" beside `while` is the same and is never seen either; not changed here.

**Not built:** the catalogue's tiers 4 and 5 (moves and acts in the catalogue; a Catalogue window); a trigger with a condition (`on hits while fact armed`), more than one doing in a row, or a doing for another thing than itself and the walker; `on` for what happens to the walker (a trigger of its own: picked up, fallen, come into a room); `on answer` and `on senses` were not run by any script (the host takes them; the code is wired as for the others); `move` was not run.

**Checked:** `-Demo wired -Check` 8 of 8, no script error: the toolbox given 'when this happens, does this' and its row changed in the Inspector to `on act search do give coin 2`; searched, 0 coins to 2, told, and Playtest listed it done once; two rows more (put searched_box, sound chime): searched again, the fact held, one chime, two coins more; given 'sees' and its 'then…', `on spots do say Ah!` was done when the walker came 2.5 m from it; struck by a script's body at 4 m/s through `loose.gd`'s count, the fact struck 1 and Clang said; 'give coin many' refused, the thing's lines unchanged; the assistant (a scripted model) was told 'hits' will never happen, mended it with 'hits struck 1.5', applied (2 changes), one Ctrl+Z back to the design as it was; the x took every row. Regressions: -Demo abilities 9/9 (the catalogue 54 abilities in 6 groups, none missing, no problems), held 6/6, shelter 11/11, no script error. Host tests 66 + 71 (p71 new), all passed, --test-threads=1. Not used by hand.

## 7cf. What it does only while something holds: the catalogue's second tier (built 2026-10-18)

The user said, after 7ce: "continue with the next logical steps in development". 7cc named the catalogue's tiers; 7cd built the first (whom it goes by); 7ce proposed the second as next. This is it.

- **`while WORD CONDITION`** (host `kind.rs` TRAITS, `HELD`; `things.gd`; `folk.gd` `_held`). A design may say that what it does as it goes about holds only while a condition does: `while follows fact tamed`, `while roams hour 6 20`, `while hunts not near walker 3`. The words it may hold are follows, roams, shies, tends, idles, greets, runs, hunts, preys, attacks, flees; the condition is in the words a day's work and quests already use (`cond.gd`: hour, has, room, there, reach, fact, weather, near, quest, walker, not). One line for each word; a later one for the same word replaces it.
- **Asked again while playing.** Each of the folk that says `while` has its own copy of what its design says; twice a second the condition is asked of it where it stands, and the word is in that copy while it holds and gone while it does not. So a pup stands where it was put until the fact tamed holds, then keeps by the walker; a wolf hunts only by night. What happens says when it changes ("The cot now follows: while fact tamed").
- **In the Inspector** ("What it can do", in HOW IT WORKS): beside each ability that may be held, a field **while**: a condition written there (Enter, or clicking away, sets it), empty for always. While playing it says whether it holds now, and why not. Written, it is one step of undo; a condition that is none is refused with why; an ability taken away takes what held it with it.
- **The assistant** is told of it (`words` topic abilities; `give` with a `while` line), and a condition it drafts is looked over as the folk will ask it, so a wrong one comes back as a problem to mend before Apply.
- **Host:** `kind_check` also answers the traits a design comes to (what it is like and its own, laid over each other); `words` answers `held`.
- **Found and mended:** the Kinds panel fell (an index out of range) when a mesh was chosen that its list did not have yet and a kind of the same name did (`kindview.gd` `_say`), in `-Tour -Only playas`'s roof part.

**Not built:** the catalogue's tiers 3 to 5 (when this happens, do that; moves and acts in the catalogue; a Catalogue window); `while` on the words of physics, signals and uses (only what the folk do as they go about); `while` of one thing put down by hand (a design's word; given from the Inspector, a design of its own is made for it, as for follows).

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7ce. Modelled and asked for: Godot's solid shapes made into designs, an assistant asked in words whose changes are one step of undo, and a scout who walks (built 2026-10-18)

The user said, after 7cd: "let's get the godot modelling tools in as well as llm support with undoable similar to how you did for the draw5 project. set the default character to one that has an appropriate moveset for the level moving forward. use animations that make sense for example a crouch or sneak that isn't basic. then proceed with the next logical steps for this engine".

- **Modelling: Godot's solid shapes** (`modelling.txt`, `modelbook.gd`, `modeller.gd`, `modelview.gd`; Objects > Modelling).
  - A model is a list of shapes, one a line, each added to what is before it, cut out of it, or kept only where it meets it: box, ball (or egg), post, cone, ring, `extrude` (an outline seen from the front pushed back), `spin` (a line of places turned round the upright: a vase, a column, a dome), `sweep` (a round tube along a path: a pipe, a rail, a root). Each may be turned and coloured; what is cut gives the hollow it leaves its own colour. `smooth`; `solid mesh | box | none`.
  - Godot's own CSG nodes build it (CSGBox3D, CSGSphere3D, CSGCylinder3D, CSGTorus3D, CSGPolygon3D in depth, spin and path modes, under a CSGCombiner3D). The window shows the solid turning as the rows say (dragged to turn, the wheel nearer), a row a shape with its operation, its numbers, ↑ and ✕; Add a shape; New model from an arch, a cave, a vase, a pipe, a gable, a box with a door.
  - **Make it** bakes Godot's solid (`bake_static_mesh`) into a mesh of the world's (`mesh NAME_model`: corners shared, faces anticlockwise from outside, a `colour` line for each shape's faces) and a design NAME that stands on it (`part mesh NAME_model`, `solid mesh`, its footprint). Both are one step of undo. So a model is drawn, saved, painted, covered, given abilities and carried into an exported game as any design is: Godot's CSG is only how it is shaped, and nothing of it runs in a game.
  - Five models came with it: an arch (a doorway cut through a block, its top cut round by a post laid on its side), a cave (a dome of rock, a hollow cut in it, an opening, and all below the ground cut away so its floor is the land), a vase, a pipe, a pointed doorway.
  - A design made of a model says so in the Inspector's HOW IT WORKS ("Its shape: the model … of modelling.txt"), with **Change it in Modelling…**.
- **The assistant** (`assistant.gd`, `assistnet.gd`, `assisttools.gd`; Objects > Assistant, Ctrl+J), as Draw5's AS-1 to AS-3 were made.
  - A person asks in words; the model looks at the world through tools that read it as it is (`look`, `find_designs` over this world's designs and the Asset Library's, `get_design`, `words`: what a design's lines, a model's shapes, the abilities, the structures and the makers mean), and drafts changes with tools that change nothing: `put_down` (one, or many scattered, seeded), `write_design`, `give` (one line added to a design: an ability), `model` (a model of Godot's solid shapes, made into a design), `build` (a structure drawn round places: a building of storeys, furnished, a wall, a fence, a pond, a camp), `add_maker` (a wood, a meadow), `remove`, `drop_change`.
  - Each draft change is checked before it is made, and answered with what it will be and what is wrong, so the model can mend it: a design's text by the host (`kind_check`, new: read and built without writing), a model's lines by modelbook, a design to put down that is there, a name the engine has.
  - Nothing reaches the world until **Apply**. Apply makes every change of the draft in order through the editor's own commands, then the host makes all the steps since it began one step of history (`together`, new: the versions between are let go). One Ctrl+Z, or **Undo this**, takes all of it back; Ctrl+Y makes all of it again. What was made is told to the model.
  - Which model answers is the person's: Claude through the Claude API with their own key (`claude-opus-5-5`, `claude-sonnet-5-5`, `claude-haiku-5-5`, `claude-fable-5-1`; effort; the system prompt cached), LM Studio or Ollama on this computer, or any OpenAI-compatible API. Native tool calls, or calls written in a fenced block for a model that has none; calls a model wrote in its answer instead of making them are taken as made (Draw5's lesson). A key is never written into a world; "Remember" keeps the settings and key in `user://assistant.cfg` on this machine; ANTHROPIC_API_KEY is read where it is set; a script never reads or writes either. Ideas: requests to begin from.
- **The scout walks** (`characters.txt` `character scout`, `look scout`; KayKit's Movement clips listed with measured strides).
  - Who walks unless a world says otherwise: Quaternius's model, made by a person, playing Quaternius's clips and KayKit's, with every move of the walker's set (vaults, climbs, ledges, wall and ceiling climbing, searching, lying down, lockpicking, dodges).
  - Crouched and still it sits low on its heels (UAL's Crouch_Idle). Crouched and going it creeps: KayKit's Sneaking, a slow sneak of two seconds a cycle, each step set down, its feet kept to the ground it covers (stride 0.9 m on KayKit's bones, measured by script from how far each foot goes). It backs away with KayKit's Walking_Backwards, goes aside with its strafes, dodges back and to each side with its dodges, and flinches when struck.
  - A world that says who walks (the Studio's "Walk as this") keeps its own.
- **The next step taken:** a design made of a model is changed from where it is clicked (the Inspector's HOW IT WORKS, "Change it in Modelling…"). Proposed next, not built: the catalogue's tier 2 (`while`: an ability that holds only while a fact holds), and the assistant given that catalogue.

**Not built:** a model's shapes dragged in the view (they are rows of numbers; the mesh made of it can be dragged as any mesh, by Edit shape, and is then no longer the model's); caves cut into the land itself (a cave is a design on the land, its floor the land); a model of more than about 19 000 corners; the assistant seeing the picture, or running Play; the assistant's conversation kept with the world; streaming answers; the assistant tried against the real Claude API or a local model (only a scripted model, here).

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7cd. Furnished and set out: doors gone through, buildings, rooms and caves furnished, the game's display set out, more pictures, things covered with other things, and whom a creature follows (built 2026-10-18)

The user said, after 7cc: "the char gets hung up on the door because it faces it the way it faces search items like a cabinet. how do i configure and setup the hud and in game display text and font and placement … continue with the next steps … are the houses and buildings empty or can it be configured to add procedural furniture too with useful things that store items and beds that can be used to rest … can this same concept apply to if shape a cave and designate it as a room … when selecting some numbers the window still jumps in some UI like the world outline … adding pictures is that a texture … can you download some more useful free ones and what if i wanted to cover an object or surface such as a cone or wall with grass or flowers or cardboard boxes". They also asked what LLM support would cost and what modelling tools Godot gives; both are answered at the end, not built.

- **Doors are gone through** (acts.txt `passes`; `actbook.gd`; `acts.gd` `toward`). Since 7cb the walker, walking toward what offers an act, turned to it and stopped in front of it, as before a cupboard. A door offers one too, so the walker stopped before a shut door and, once it was open, before the open one ('shut'), held there while still pushed that way. An act may now say `passes`: it is gone through or past, not stopped before. open, shut and open_unlocked say it. E does it as before. In the Inspector's How it is done, "stops you before it" switches it for every thing that offers the act, or for this one.
- **The World outline stays where it was scrolled to** (`makers.gd` `_detail_of`, `_hold_scroll`). A number changed there builds the maker again, and the window was drawn again from its top. Now, drawn again for the same maker, it keeps its place (held a frame later, once laid out). The Inspector does the same (`inspect.gd` `_hold_scroll`).
- **Buildings, rooms and caves furnished** (host `land::build`: `furnish`, `furnish_middle`, `furnished`; key `furnishing`; `furnish.txt`; library `furnishing.txt`; `makers.gd` `_furnish_rows`, `furnish_with`).
  - A structure may be told designs to put inside it: against each wall inside, its back to the wall and its front (a design's -z, where a cupboard is come to from) to the room; and in the middle of each cell. Every storey of a building is furnished, on its boards. None goes in a doorway or where the stairs go up. How many of the places are used is a number (0.7 at first), the places chosen by the structure's seed.
  - Each piece is a thing of the structure, kept by its side's name: moved, taken away or given another design by hand, it stays so when the structure is built again.
  - A structure drawn round a place with "nothing round it" is a room with no walls: a cave, a camp. It is furnished along the inside of its line, and in its middle.
  - The structure's settings (World outline, and the Inspector on any piece of it) have "furnished with": a set of `furnish.txt`, or nothing. The sets are a home (beds, a wardrobe, a chest, chairs, a high shelf, a bookshelf; a table in the middle), a shelter (cupboards, toolboxes, crates, beds, shelves), a store room, and a camp or cave (a bed, crates, a chest; a stove in the middle). A set's designs are brought from the library if the world has none of them. Under it, the designs against the walls and in the middle can be said by hand, and how many places are used.
  - The library has a new shelf, "Furniture to use": a bed to sleep in (`offers sleep`: lain on, tiredness goes, time passes), a wardrobe, a chest, a high shelf and stacked crates to search (what is in them is by chance, `perhaps`, drawn again each run; the shelf is reached up to, the chest knelt at, by the search heights of 7cc), a chair to sit on, a plain table.
- **The game's display, set out** (`hud.txt`, `hudbook.gd`, `hudlook.gd`, `hudview.gd`; Characters > Game display).
  - Every piece of what is on the screen while playing is named in hud.txt: the quest step, the goods, the keys 1 to 9, what is in hand, the keys' hint, what E does, the stats' bars, health, the rounds left, the banner, the meter of being seen, a talk. For each, where it stands (a corner of the screen and how far in from it, or just under another piece), how big its letters are or how big all of it is, its font, the colour of its letters, a line round them, what it stands on, or hidden. What is not said is as the engine has it. `all` says a font, colour or size for every piece.
  - The pieces are made by the scripts that show them. `hudlook.gd` finds them and dresses them after those scripts every frame, so what they set of themselves is set over. What each was before is kept and put back when the line that changed it is taken away.
  - Window > Game display has a row per piece: shown, its corner, x and y, its letters (or all of it), its colour, its font, its backing. Each change is one step of undo, and is seen at once while playing. "Show it here, while editing" shows every piece over the view with words of its own. "Add a font…" copies a .ttf, .otf, .woff or .woff2 into the fonts folder.
  - hud.txt is kept with the world, as acts.txt is, so a game made of it looks so. An exported game carries the fonts folder (`export.ps1`).
  - Fonts that came with it: Kenney's font pack (CC0, 12 fonts: Blocks, Future, High, Mini, Pixel, Rocket and their square and narrow kinds), credited.
- **More pictures** (`editor\godot\textures`): 28 more from ambientCG (CC0), shrunk to 512 px. Asphalt, bark, cardboard (two), carpet, red cloth, corrugated metal, diamond plate, short grass, gravel, dirt, dry earth, sand, ice, lava, leather, metal scales, moss, plaster (two), paper, cobbles, a rock face, rust, snow, terrazzo, wicker, wood siding. 42 in all, credited. A picture laid over a design is a texture (`texture FILE S`). Your own are added with Its shape's "Add a picture…" (PNG, JPG, WebP) or put in that folder.
- **Covered all over with another design** (kind.rs TRAITS, ONCE `covered`; host `shapes.rs` `Mesh::cover`, `kind_outputs_covered`; `land.rs` `kind_read` reads the covering design too; abilities.txt `covered`; library `covering.txt`).
  - A design that says `covered DESIGN EVERY [SIZE]` has copies of that design strewn over all its outside: one about every EVERY metres, each SIZE times as big (give or take a quarter), standing out from the surface where it is, turned at random about it. None goes under what faces straight down, its foot. It is made into its mesh, the same each time for the same place, and seen, not walked into. At most about 200 000 corners are added.
  - Grass on a cone, flowers on a bank, ivy on a wall, boxes heaped on a mound. The new library shelf "To cover things with" has grass-tuft, flower-small, ivy-leaf, pebble and cardboard-box (which has the cardboard picture). Any design will do. In the Inspector it is the ability "is covered all over with another design", whose design is brought from the library if need be. HOW IT WORKS says what `covered`, `texture` and `climbable` mean.
- **Whom it follows, and whom it runs from**: the catalogue's first next tier (`folk.gd` `leader_of`, `shied_from`; `things.gd`; abilities.txt). `follows R [WHOM]` keeps by the nearest of whom it names: the folk of a side, a design, or a word of what they are called ("follows 1.5 cot": a chick that follows the cot that follows you). With "you" among them, it keeps by the walker when none of them is about. Those that follow one go abreast of each other. `shies R [V] [WHOM]` runs from them, or from the walker as well with "you". The catalogue's rows have the field "whom".
- **Found and mended:** what a maker made says the design each thing is of (`things`: `design`).

**Answered, not built.**
- *An LLM in the editor.* Draw3D is driven by text and small commands: every design, act, move, structure setting and HUD piece is lines a person writes, and every edit is a command to the host with one step of undo. So a model that is told those words (the catalogue, kind.rs's words, the host's ops) can be asked "put a furnished house of two storeys by the pond" and answer with commands the editor runs and the person can take back. Building it is about a milestone: a chat box, the words given to the model, its commands checked and run as one step of undo, and what it did shown. Running it costs per request, a fraction of a cent to a few cents for a level's worth of commands with an API model (Claude, others). A local model costs nothing to run but is weaker at it. Nothing of it need go into a game made with Draw3D; it is the editor's.
- *Modelling tools from Godot.* Godot has CSG shapes that work in a running program: CSGPolygon3D extrudes an outline to a depth, spins it round an axis (a lathe: vases, columns, wheels) or sweeps it along a path (pipes, rails, a cornice). CSG combines shapes: union, subtract, intersect, so a doorway is cut, or a cave is hollowed out of a rock mass, with overhangs the land cannot have. Its SurfaceTool and MeshDataTool make and change meshes. There is no ready face-by-face extrude like Blender's: that is an editor add-on, not part of a running program. Draw3D's shapes are its own design parts; a next tier for them would be parts that say extrude, spin, sweep and cut (CSG), turned parts, and caves cut out of rock.

**Not built:** furniture on the land of a structure drawn along a line that does not go round; furniture that fits itself to a room's size or keeps out of another's way (each is centred on its side); a cave's walls and roof themselves (a CSG tier, above); a HUD piece of your own (a counter of a fact, a picture); the HUD laid out by dragging pieces on the screen; a covering of only some of its faces, or one seen only from near; covering on a thing with bones; the catalogue's tiers 2 to 5 (while, when-this-do-that, moves and acts in the catalogue, a Catalogue window).

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7cc. A catalogue of what anything can do, given to anything; colours and pictures applied at once; searched low, middle and high; walls and ceilings climbed (built 2026-10-17)

The user tried 7cb by hand and said: "how can I change the color of the thing … how do I put texture … I tried changing the color of a platform and it crashed … once you choose it, it doesn't take effect yet sometimes … you should have the ability to just kind of apply it … the search for the toolbox on the ground is the same search animation for the cupboard … search low and search medium … a high shelf … the third option … I gave it two uses. So why do I not see it … there should be a catalog of functions and abilities … that gets updated as new things are added … I want the bed to follow the character as a pet … sort that out or come up with a plan … if you turn the cot into a creature and then you turn it into a bed … he lies the wrong way … we should be able to go ahead and tweak it … then do the movement for climbing walls and ceilings … a flyer is what it should be."

- **The crash, found and mended** (`livecolour.gd`). The run that fell left Godot's crash report in `editor\godot\logs\20261017-215902-run.log.err`. A small script did the same thing outside the editor: a colour picker whose window is drawn again while its popup is closing takes Godot down, every time. Both the Inspector's colour in "Its shape" and the Library's colours did that: they wrote the colour as the picker closed, and the write drew the window again. The colour is now written a moment after it changes, while the picker is still open, and the window is not drawn again until it has closed.
- **What is chosen is applied** (`num.gd` `applies`). A colour is seen as it is chosen, not when the picker is shut. A number typed into a field is set when the field is left, as Enter sets it, everywhere (before, only some fields did). A line of words in the Inspector, Makers, layouts, variation, designs and the tree generator is set the same way. While typed and not yet set, it is shown amber.
- **How a colour is changed, and a picture laid over it.** A design's colours are in the Inspector's "Its shape", one square per part, and in the Library's "Customize this world's". Every design can now say `texture FILE [S]` (kind.rs TRAITS, ONCE, OWN): a picture of `editor\godot\textures` laid over every part, in the thing's own frame, repeating every S metres, each part's colour tinting it (`world_view.gd`, the paint shader's `pic`). "Its shape" has a "texture" choice, "repeats every", and "Add a picture…" (PNG, JPG, WebP: copied into the textures folder). 14 pictures came with it, from ambientCG, CC0, shrunk to 512 px: bricks, concrete, fabric, grass, ground, marble, metal, paving, planks, rock, roof tiles, tiles, wood, wood floor (credited). An exported game carries the folder.
- **Searched low, at the middle, high** (acts.txt `does_low MOVE [H]`, `does_high MOVE [H]`; `acts.gd` `_does`). One act now has three moves, chosen by how high the place is over the doer's feet: under 0.6 m `does_low`, over 1.4 m `does_high`, otherwise `does`. The searches kneel at a toolbox on the floor (`search_low`), rummage at a cupboard (`rummage`) and reach up to a high shelf: a new move `search_high`, with a pose and, for the Quaternius looks, KayKit's Magic_Raise clip, which reaches one arm high. In "How it is done" each act has three rows (low down, the middle, high up), each changed for every thing or for this one only.
- **Its uses, seen** (`inspect.gd` `uses_of`, `take_use`). "What it can do" begins "Made usable as:" with each use it was made, as a design made for it or as its own lines, each a button that takes that use away. Taking away a design made for a use gives back the design it was made from. Taking away its own lines brings back what making it usable had taken from its design. A use that is only a design of its own (a creature) no longer takes away the uses it has, so a bed made a creature is still a bed, whichever was made first.
- **Why the uses were not seen.** Three faults. The Inspector read a thing's lines from what acts remembered of it, kept from the first time it was seen. It read them from what is drawn of it, and someone with a body is drawn by the folk, so that says only its name and design. And it asked for its own lines only after showing what it does. Now it reads a thing put down by hand from the texts (its design's, every design it is like, its own lines) and asks for its own lines first. A moment after every edit it asks again what is picked (`refresh_soon`).
- **The catalogue of what anything can do** (`abilities.txt`, `abilitybook.gd`, `abilities.gd`; the Inspector's "What it can do"). Every ability the engine has, in a person's words, in five groups: goes about by itself (a creature, wanders, follows you like a pet, runs from you, runs, idles, greets, keeps hours, tends, can be ridden, talks); alive with no body (flutters, swims, hops, perches, drifts); sees, hears and fights (17, from sees to drops); the walker meets it (climbable, hides, harms, slippery, breaks, barricade, checkpoint, key, plays); is seen and heard (texture, sheer, water, floats, sounds, hums, knocks, shows). Each says the lines it gives with fields `{NAME DEFAULT}`, what it `needs` (to follow you, a body that goes about) and the `rig` it is given if it has none.
  - In the Inspector, each ability the picked thing has is a row: its numbers to change and an x to take it away. "Give it…" lists every other ability, under its group.
  - What one thing may say of itself is written as its own lines. What only a design may say (someone, follows, a rig) is written into a design of its own: the one it has, if nothing else is of it, otherwise a copy made for it (`cot_own_cot`, like the cot).
  - The host now says every word a design may say (`{"op": "words"}`). Every word is given by an ability or named under `elsewhere` with where it is set (Make it usable, Physics and Signals, the level part, a design's text). Any word that is neither is listed in the Inspector, so a word new in the engine is seen to be missing from the catalogue. It lists none today.
  - New uses: `pet` (a creature that follows you). `flier` is now `flyer`, in uses.txt, moves.txt (the set `flyer`), the acts, the poses and the demo.
- **Lain the right way on a bed that is a creature** (`acts.gd` `_frame`, `where`, `_out`). For anything with a body, the places of its rig were used and its own were not. The rig `whole` has a `foot` of its own, on the ground under it, and so does the bed's place. The walker was laid at the creature's feet on the ground, along the way the body faced, not along the bed. Now its own places come first, turned with its figure (the way it faces), and the side it is got on from is its own. "How it is done" has "which way they lie" for a rest that lies: as it is, turned round, or a quarter to either side (it turns the place's `from` in its own lines).
- **Walls and ceilings climbed** (`moves.txt` `wall_climb`, `ceiling_crawl`; `move.gd` `climbs V`; `grasp.gd` `overhead`; `motor.gd` `overhead`, the conditions `climbable` and `ceiling`, the spot `ceiling`, `upon ceiling`; `watch.gd` tells the motor what is climbable; `things.gd`; kind.rs TRAITS `climbable`).
  - A thing that says `climbable` (the ability "can be climbed", or a design's text) is climbed: jump at it, from the ground or in the air, and it is held. Pushed into it, the walker climbs up; pulled away, down; to a side, along it, 1.4 m a second.
  - At its top the edge is caught (ledge_hang) and jump climbs over. Where the wall meets a climbable ceiling, the walker goes on under it.
  - Under what is overhead that says `climbable` (a ceiling, the underside of a bridge), jumped up to, the walker hangs by the hands and goes along it as the stick asks, at the same height. Crouch lets go of either.
  - Poses `wall_climb` and `wall_climb/hold/go` (hand over hand), `ceiling_crawl` and `ceiling_crawl/hold/go`. A move's phase with `climbs` says the side "go" while it goes, so a pose can be written for going and for being still.
  - Nothing that does not say `climbable` is climbed: walls in the shelter, the yard and the course are as they were.
- **Found and mended:** a new shape could be given the name of a design that already existed if the host's answer was late, just as the world stopped running. The list of what can be put down is now asked as well.

**The plan for the rest of the catalogue** (proposed; nothing of it is built):
1. *Whom it goes by.* "Follows you" follows the walker. The next tier names whom: follows the cot, flees the dog, hunts whoever carries the key. `follows`, `shies` and `tends` would take a target, which is a name, a kind, a side or "you", as `preys` already does.
2. *When it does it.* Each ability would take a condition, "while", in the words `cond.gd` already reads: follows you `while fact tamed`, or roams `while hour 6 20`. One field per row, the same everywhere.
3. *What it does when.* Triggers that already exist (an act's moment, `senses`, `hits`, `spots`, a talk's answer) would be listed as "when this happens" beside "do this" (put a fact, give a good, say something, play a sound, start a move). The catalogue's rows would then wire one to the other. The Signals window shows the facts between them.
4. *Moves and acts in the catalogue.* Each move of moves.txt and each act of acts.txt in the same list, to give to a set or a thing by name, with the same "missing" check against the files.
5. *The catalogue as a page.* Window > Catalogue: every ability, condition, trigger, move and act, searched as the Library is, with what each does and which things in this world have it.

**Not built:** the five steps above; textures on a creature's body or on what is drawn as a model; a texture per part (it is the whole design's); a climbing clip for the Quaternius and KayKit looks (they have none: Draw3D's pose is shown only on Draw3D's figure, and a character with a look shows its fall pose while climbing); climbing what the walker cannot see the top of within the eye; a ceiling crawled along by anyone but the walker (the folk have no climbing moves).

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7cb. Anything made, and made to do anything: the Library searched, facing what is at hand, animations chosen, your own uses, a creature, shapes, models brought in, the game exported (built 2026-10-17)

The user tried 7ca by hand and said: "I should be able to search the asset library … prior to searching, there's got to be an indication that he's facing that particular object … searching a cupboard … uses the same ground animation … it should be something that I'm able to configure … If anything is hard-coded that prevents me from recreating this from scratch, then that needs to be fixed. When using the gamepad and I go to inventory, I cannot select some items … you are authorized to download the general usage things … sleeping, searching, wall climbing, ceiling crawling … flying … one that holds on to ropes or the legs of a chicken … I want to be able to define my own [uses] … turn the bed into a creature … basic shape tools … import 3D objects … export the project into a game file I can run, exe if possible … the least amount of files, the better."

- **The Library searched.** It had a field, squeezed beside two lists, that matched names only. Now it has a row of its own ("Search"), and every word typed must be found in a design's name, what it is called, its group, its shelf or what it says. Ctrl+F opens the Library at it (`library.gd` `found`).
- **Facing what is at hand** (`acts.gd` `toward`, `player.gd`, `notice.gd` `facing_mark`). The walker near something that stands and offers an act (a cupboard, a door, a cot; not an item lying about, which is walked over as before), asked toward it at a walk, turns to it and stops in front of it. A green mark over it says it is what E will do. Asked away from it, or with nothing there, nothing changes. Once stopped there, they stay while still asked that way: going on a little no longer carries them past.
- **Searched by rummaging, and each act's animation chosen.** New moves `rummage` (standing, both hands in it), `search_low` (knelt at it), `eat`, `lockpick` (moves.txt; in the walker's, the folk's, the worker's and the swordsman's sets). Poses for each (poses.txt) and clips for the Quaternius looks: Chest_Open, Fixing_Kneeling, Consume, KayKit's Lockpicking. The search acts rummage, the loot acts kneel, unlocking picks, eating and drinking at something eat (acts.txt). In the Inspector, "How it is done" under Make it usable lists each act a thing offers with its move (or the pose it rests in): changed for every thing that offers it (acts.txt), or for "this one" only, which gives the thing an act of its own (a copy, `search_food_cupboard-search`) and offers that instead (`inspect.gd` `set_done`).
- **Your own uses** (`uses.txt`, `usebook.gd`). The presets of Make it usable are no longer in the code: each is `use NAME` with `says`, `line …` (one thing's own lines, with numbers measured of the thing: {high}, {grip}, {top}, {end}, {front}, {seat}, {r}, {h}…), `design …` (lines of a design of its own, for what one thing cannot say: its rig, that it is someone, its moves) and `none`. "Save as a use…" writes the picked thing's use lines as a new use. New uses: `chest` (knelt at), `creature`, `flier`.
- **Anything a creature** (host `real.rs` `whole_rig`: the rig `whole`, one bone `body`, places middle, top, foot, under; `poses.whole.txt`; moves.txt sets `creature` and `flier`, moves `flap`, `glide`). Make it usable: a creature gives the thing a design of its own (`cot_use_creature`, like the cot: rig whole, someone, moves creature, roams 6, body R H) and makes it of that: played, it wanders, rocking as it goes. A flier is `bodied`, offers `hang_on` (acts.txt: `rides hang`, taken hold of where one stands); hung from it (pose `hang`, `hang/go`), what the one hanging asks is asked of it, buttons too (`acts.gd` `over`): jump flaps it up, five times before it lands, and it glides down. The one hanging is never held below the ground under it.
- **Downloaded animations** (with the user's yes): Kay Lousberg's KayKit Character Animations 1.1, free version, CC0 (`characters/kaykit/`, six of its glTF files; the map `kaykit`; the library `kaykit`, clips named FILE_CLIP). Its lying, sitting on a chair, crawling, lockpicking, tools, using and aiming carried to the Quaternius characters and the swordsman (their `clips` name it). Lying by its clip (`lie Simulation_Lie_Idle`), the walker is not laid down again by the act (`animated.gd` `shows_clip`), and lies head first the way it faces. No free CC0 or CC-BY clips of hanging, wall or ceiling climbing or flying were found (Quaternius's free libraries, KayKit, monogon's; the rest are paid or bar redistribution): Draw3D's own poses are written for them (`hang`, `hang/go`, `crawl`, `fly`), shown on any character that has no clip of the name. `clipshot.gd` takes pictures of a character holding any clips, to see them before a look names them.
- **Basic shapes** (`shapemaker.gd`; Objects > New shape: a box, a cylinder, a cone, a ball). A new design of one shape, put down by a click. The Inspector's "Its shape" lists a design's parts as rows (where each stands, how wide, high and deep, its colour, x to remove) and adds boxes, cylinders, cones and balls. Each change writes the design again: one step of undo, and every thing of it follows. Parts it cannot show (a limb through many places, a mesh, a part on a bone) are kept as written.
- **Models made elsewhere** (`models.gd`; Objects > Import a 3D model: glTF, FBX, OBJ). Read, its foot put at its middle on the ground, written as one self-contained glTF (`editor/godot/models/NAME.glb`), made a design of this world (`model FILE [S] [TURN] [UP]`, a new line of a design, kind.rs TRAITS; a box round it to pick and walk into, not drawn: `world_view.gd` `hide_content`) and saved to your Library shelf, group Imported. Pictured in the Library as it is shown.
- **The game exported** (World > Export the game…, `main.gd` `export_game`; `export.ps1 -World FILE -Name NAME`). The world saved, then put together in the background: `build\NAME\NAME.exe` with the scripts packed inside it (game-export.cfg `embed_pck`), `engine-host.exe`, `world.json`, `feel.cfg`, `characters\` (every model and clip, with their licences, in their folders), `models\`, `CREDITS.txt`; and `build\NAME.zip` to carry to another computer. Saved games go to `%APPDATA%\Godot\app_userdata\NAME`. Told when done, the folder shown.
- **The inventory by a controller** (`gameui.gd`). A was the likely action and did nothing on what is eaten, drunk or used ("To eat, drink and use"); now it uses it. X uses or puts on, Y drops, right goes to the first button that can be pressed, B shuts. Said under the buttons.
- **HOW IT WORKS** names `model`, `someone`, `bodied`, `moves`; an act's move.

**Not built:** movement for wall or ceiling climbing (the poses exist; the motor's wall grip and climb are what move a body up a wall), a flier that carries someone's weight or flies by itself, retargeting onto a rig the engine has no map for (a map is written in characters.txt), a part turned in "Its shape", a model's box kept to its size when `model … S` is changed, the game's files in fewer than its folders (the host and Godot read characters and models as files).

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7ca. Use anything: the start clicked, what lies about marked, stats and statuses, effects, presets, aim and fire (built 2026-10-17)

The user said, of a screenshot of the shelter: "what is this and why can i not click on it … i'm assuming it's the starting point but how did you set that and why can't i move it … couldn't see the dagger. items in the vicinity should go or have some indicator. if i want to lay down a cot and designate it as a rest object … can i pick a random object and configure it for use … can the character aim with a gun in both 2.5d or top down combat … are there injuries capabilities or statuses. can i use and configure things like hunger or thirst or any other that i define. can i configure items to have multiple affects." Asked what to build, they chose the quick fixes, then the milestone, and approved survival needs.

- **The start of a game, clicked.** The green flag (a pole, a pennant, a bar the way the walker faces; seen from below it looks like a tick) is drawn by `marks.gd` and has no body, so a click went through it. Now a click on it (`marks.gd` `flag_hit`, `main.gd` `pick_start`) chooses its maker and opens the Inspector, whose HOW IT WORKS says what it is and has its place and facing as numbers, "Here: where the view looks", "Face the way the view looks", the eye a game begins with and how long the walker may run. Its box over the flag is dragged as before. Characters > "Where a game starts, in the Inspector".
- **What lies about is marked** (`notice.gd`, `stats.txt` `notice R [items] [holds]`, 8 m): an item lying about (gold) and a thing that holds something (blue: a crate with food in it) get a mark that bobs over them, their name once the walker is within 5 m (not while E's prompt names it), and an item's own model is outlined.
- **Stats you define and statuses** (`stats.txt`, `statbook.gd`, `stats.gd`): `stat NAME` with called, range, starts, per_hour (the world's hour: a minute) or per_minute, shown (a bar, warmer toward its bad end), colour, and `over N STATUS` / `under N STATUS`. `status NAME` with called, says, lasts S, every S EFFECTS, slows F, no_run, stacks N, colour. Shipped: hunger, thirst, tiredness; hungry, starving, thirsty, parched, tired, bleeding, poisoned, well rested. Each stat is a world fact by its name and each status `status.NAME` (and `status_left.NAME`), so conditions, doors, talks and saved games go by them with nothing new. Health (`fight.gd` `change_walker`) and breath are the engine's own and can be changed by an effect.
- **Effects, one grammar everywhere:** `hunger -30, thirst +10, health +1, status bleeding 20, cures poisoned, fact fed 1`. Said by an item (`effects`, `used_as WORD`, `keeps`: used from the inventory, used up unless it keeps), by a good (goods.txt `effects`, `used_as`: food, water, medicine are listed under "To eat, drink and use"), by an act at its moment (`effects`), by an act rested in (`rest S EFFECTS`), and by a status (`every`).
- **Rested on** (acts.txt `rests POSE [lying]`, `rest S EFFECTS`, `hastens K`; `acts.gd` `_settle`, `_rest`): something with no body (a cot, a bench) is lain or sat on until E again; new acts `sleep` (lie, tiredness -3 a second, a little health, time 8x, then well rested), `sit`, `eat_here`, `drink_here`. Poses `sit`, `lie` (poses.txt); clips `sit` (Sitting_Idle_Loop) for the Quaternius look.
- **Make it usable** (Inspector, HOW IT WORKS, `make_usable`): a thing put down by hand made a bed, a seat, food, a drink, medicine, a container, a place to drink at, a place to eat at, a weapon, or nothing, as its own lines (one edit). Its design's lines of the same words are taken away first (`no offers` …). What using it does, its button's word and "not used up" are set there; "Stats and statuses (stats.txt)" opens where they are written. The host lets one thing say `offers`, `point`, `kept`, `stock`, `called`, `gear`, `effects`, `used_as`, `keeps` of itself (`kind.rs` OWN, TRAITS).
- **Aim and fire** (`aim.gd`, gear.txt `fires DAMAGE SPEED RANGE [rounds] [reload] [every] [spread] [pellets] [uses GOOD] [heard]`): the bow, crossbow, slingshot, pistol, rifle and shotgun fire. The right button or the left trigger aims (drawing it if put away): from behind at the view's middle; from above at a mark the mouse or right stick moves (the eye stays); side-on along the walker's line and up and down, meeting whoever is in the lanes' band. The left button or right trigger fires (not aiming: at whoever is nearest before them within 22 degrees). R or the right stick loads it again from the good it uses (bullets, shells, arrows, bolts). The shot is `fight.gd`'s (`walker_shot`), heard so far off; the aim pose `aim`, `aim/up`, `aim/down` (poses.txt; Pistol_Aim_* over the upper body for the Quaternius look, `animated.gd` `aiming`). The rounds are a fact about the item.
- **HOW IT WORKS** names all of it: an item's effects and button, an act's effects, rest and time, a gear line that fires.

**Not built:** injuries to one part of the body (a status stands in), a Stats window of rows (stats.txt is the place, F10), a reload or eat clip for figures without one, aim assist that leads a moving target, a thing made a weapon other than by its own gear line.

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7bz. A click shows how it works, and changes it there (built 2026-10-17)

The user said: "proceed. ensure what you're making is editable in the engine. it's still not intuitive. when i click on an object or entity i should be able to see its info, modify its physical attributes or behaviors and related conditions … if i click on a house i should have access to all relevant settings including switching the camera view under certain conditions."

What was found first, by a script that clicked each sort of thing the shelter's house is made of: a left click picked the thing and showed nothing. The Inspector came up only with F7. Even then, a house showed its generator's parameters by their engine names ("as, height, floor, leaf, seed, on, over, rim, gates …") with no meaning, and nothing of what it does in play, what it goes by, or the eye. The house's settings in plain words were in World outline (Makers), which a click never led to. The physics and enemy parts appeared only for a thing put down by hand. No setting anywhere changed the eye by where the walker was.

- **A click opens the Inspector** on what is clicked (`main.gd` `click_pick`, used by the click and by scripts alike), unless View > "A click on a thing opens the Inspector" is off. The ground does not open it.
- **HOW IT WORKS, at the top of the Inspector** (`inspect.gd` `_fill_works`), for whatever is clicked:
  - **What it is.** A part of a structure, a thing put down by hand and of which design, or one of what a maker makes.
  - **Its settings**, for a part of a structure: the structure's own rows, the same as in World outline (`makers.gd` `structure_part` builds them into the Inspector): built as, height, storeys and stairwell, floor, roof, steps, gates that swing, cells, gates and walls inside, what dresses its foot. Changing one there builds all of it again, as before.
  - **What it does in play.** Every line it says of itself, and the lines of a structure's parts that do something (its doors), each with what it means. An `offers` line names its act in words: what it is offered as, while what, and what it does ("offers open handle — E: 'open' unless 'open': puts 'open' side; with the move 'reach'"). Also what the structure itself does: seen through, the eye inside it, a place the folk may be sent to.
  - **What it goes by.** Every condition and fact: an act's `while` and `unless`, what it needs or uses up, `when`, `by`, the facts it puts, the eye's condition.
  - **Where each is changed.** Its acts' text (F10, opened at the act's line), its design's text (F6), Signals, and its maker in World outline. The physics, enemy, joint and finds parts follow below as before.
  - Kept to the screen: 470 px wide at most and two thirds of its height, scrolled beyond that. Lists cut their choices short and the words of a box wrap.
- **The eye inside a structure, and when** (new structure settings `eye` and `eye_when`; host `land::build`, and the structure's description now says `cutaway`, `eye`, `eye_when`). While the walker is in one of its cells, the world is seen through that eye of `eyes.txt`, if its condition holds. The condition uses the words a quest or a day's work goes by (`cond.gd`: `hour 20 6`, `fact alarm`, `not weather rain`, `has 1 lantern`, `near the guard 6`); a wrong one is refused with the reason. Going out, or once the condition no longer holds, the eye from before is taken again, unless the person chose another (V) while inside (`eyezones.gd`). Structures one inside another: the smaller. A structure drawn along a line has no cells to be inside of, and says so.
- **Seen through** (`cutaway`, which only a script could set) is a box in the structure's settings.
- **Clicked again in the same place** (the view not moved): what is behind what was picked. A cupboard in a house is reached through its wall; again, what is behind that; when nothing more is, the first again (`world_view.gd` `pick` takes ids to look past).

**Not built:** the eye changed by a thing put down by hand (a zone of its own) or by a world-wide rule; one thing's own lines for a structure's parts (a door's lines are the structure's and the acts'); the folk's behaviour shown in the same words (their talks, reactions and quests are in their own windows, which the Inspector does not yet name).

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7by. Keys, drops, finds by chance, and breath spent by dodges and blows (built 2026-10-17)

The user said to proceed after 7bx was committed. With the enemy section's rows that need no approval built, this takes the small items left in section 5, item 6 of `Draw3D_Genre_Readiness_2026-10-17.md`: keys, drops, and what is found drawn by chance. It also takes two rows' smaller "missing" items: "items dropped (gear, not only goods)" and "stamina spent by dodging and blows too". Ladders, a gate anywhere along a side and a curve through a drawn line are left for the next pass. Survival needs, live drawing and enemies drawn by animated characters still wait for the user's word.

- **Keys** (`key GOOD`, a line of a kind: kind.rs TRAITS, ONCE, OWN; an act's `needs key` or `needs GOOD N`, `actbook.gd`, `goods.gd`). `needs` asks that the doer has so many of a good, and keeps them: nothing is used up. `needs key` means the good the thing itself says opens it, so one act serves every lock and the lock says its key. The new act `unlock_key` ("unlock", heard 2 m off) is offered before `unlock` (the lockpick), so whoever has the key uses it, and whoever has neither is told "Unlock the strongbox? You have no strongbox key." Keys are goods (`door_key`, `strongbox_key` in goods.txt), not items: they are counted, never worn or drawn in the hand, and a goods line could already be found, carried by the folk and looted.
- **Taking everything** (`gains everything`). An act line that takes all the thing has, of every good. It is offered while it has anything. New acts: `loot` ("take what is carried by", while fallen), `search` (all a thing holds) and `search_unlocked` (all a thing holds once it is unlocked). `loot_food` and `loot_scrap` stay as they were.
- **Finds by chance** (`perhaps GOOD N P`, a line of a kind, several added up; kind.rs; `goods.gd` `count`). N more of the good are in it one time in 1/P. The draw is the world's fact `finds`, put as the world begins to run (`main.gd` `play`). Each run therefore finds otherwise, a saved game keeps its draw, and the same draw always finds the same, because each line's chance comes from the draw, the thing's id, the good and the line's number. A thing's own `perhaps` lines replace all of its design's (`own_over`), as `pattern` and `joins` do.
- **Drops** (`dropped_by NAME`, a line of a kind or of one thing). A thing put down anywhere, such as a katana, that says it is carried by one of the folk (by what they are called, their kind, or their id). When the world begins to run its `where` is "with": unseen, not kept, not to be taken (`fight.gd` `carried_drops`; `things.gd`). When the first of the folk of that name falls, it lies 0.8 m beside them and is told ("The bandit captain dropped a katana."). A dropped item is one thing of the world with its own id, so it is kept by a saved game as any item is. No thing is made while the world runs.
- **Breath spent by dodges and blows** (`player.gd`). Where the Player start says `stamina`, a dodge, roll or dash costs a second of breath and a blow half a second. Winded, a dodge or a dash is not begun. The dash button before a foe is a blow, and is still struck.
- **The fallen nearer** (`acts.gd` `_nearest`). What a person in reach offers counted 0.8 m farther than it is, so the baker by his stall does not take the walker's key from the loaf. That held of the fallen too, and a crate beside a fallen bandit was offered before its loot. The fallen now count as near as they are.
- **Why, of what is found by chance or opens with a key.** What is offered says why nothing can be done ("Search the crate? The crate has nothing.") of a thing that says `perhaps` or `key`, not only of one that says `stock`.
- **The Inspector** has a part of its own, What is in it, what opens it: found by chance (commas between lines), opens with, and dropped by. Each field shows its design's value in grey and this thing's own in cyan.
- **Library.** In Shelter: a crate of chance finds (`crate-finds`), a strongbox opened by its key (`strongbox`: six coins, now and then a dose of medicine), and a door locked by a key (`door-key`). In Creatures and enemies, Fighters: a bandit captain (`bandit-captain`: five health, the strongbox key, three coins, `loot`).
- **The wilds** have a lookout to the south-west: the captain, a katana that says as its own `dropped_by the bandit captain`, the strongbox and three crates (`examples/wilds.rs`; `editor\genres\wilds.json` built again).

**Not built:** keys as items of gear (a key is goods); a table of finds shared by many things (each thing says its own lines); things made while the world runs (a drop is put down beforehand); what the folk carry drawn on them.

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7bx. A boss of phases, and acts seen side-on reached at once (built 2026-10-17)

The user tried the shelter by hand and found it reasonable for now, except that an act on a door or a cupboard began with a walk in the wrong direction. They asked to continue with the next part. That is item 4 of section 5 of `Draw3D_Genre_Readiness_2026-10-17.md`, attack patterns and phases. It is the last row of the enemy section that needs no approval: survival needs, live drawing and enemies drawn by animated characters still wait for the user's word.

- **The walk before an act, mended** (`acts.gd` `_stand`). Seen side-on, the place to stand for an act was off the walker's line: 0.2 to 0.6 m out from the door or cupboard, toward the eye. The walker walked at the eye, was brought back to the line every step, and walked in place facing the eye for up to 2.5 s before reaching. A trace showed it: about 150 steps of "come", facing (0, 1), before every open. The place to stand is now put on the line the walker is held to, so only the way along the line is walked. The folk are held to no line and are unchanged.
- **Patterns** (`pattern F MOVE [MOVE...]`, a line of a kind: kind.rs TRAITS and OWN; `fight.gd`). Below F of its health, a boss's blow is these moves one after another. Each is begun as the one before ends, and it turns to whom it hunts between them. After the last comes the `attacks` wait. There is a line for each phase, and the lowest F it is below is used. One that says no `pattern` strikes with its `attacks` move, as before. A thing's own pattern lines replace all of its design's, as its own `joins` do (`kind.rs` `own_over`).
- **Phases.** Crossing into the next pattern, it roars: its `roar` move, heard 14 m off, 1.4 s in which it does nothing. Its fact `phase` is which pattern it is in (0 the first).
- **Spent** (`tires S`). After its pattern it stands for S s, its `pant`, neither turning nor striking. This is the moment to strike it.
- **Weak** (`weak behind N`, `weak tired N`). A blow of the walker's from behind it, or while it is spent, does N times the harm, and the walker is told so.
- **Moves** (`moves.txt`, set `warlord`): `lunge` (gathered 0.4 s, then a rush of 2.4 m with the right hand out), `slam` (both fists raised 0.7 s, brought down before it), `roar` and `pant`, each with poses (`poses.txt`).
- **The warlord** (library Creatures and enemies, Fighters). It has 16 health, three phases (swing swing; swing lunge; lunge slam swing), `tires 1.6`, `weak behind 2`, `weak tired 2`, keeps to 9 m, and carries 4 scrap. It stands in the wilds inside a ring of boulders to the south-east (`examples/wilds.rs`).
- **The Inspector's enemy part** has three more fields: patterns, spent for, and weak. Several lines of one word are written with commas between.
- **Found and mended:** "behind" was judged by how the one struck faced after the blow. A blow turns whoever it strikes to face the striker, so a blow from behind never counted as one, takedowns included (they passed for other reasons: not seeing, not yet alert). Each fighter's facing is now kept from the step before.

**Not built:** weak points as parts of its body; a different harm for each move of a pattern; attack patterns chosen by conditions other than health (distance, a fact).

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7bw. Barricades, locks, crafting, searching heard, and stamina (built 2026-10-17)

The user said to continue after 7bt–7bv. This builds the next of `Draw3D_Genre_Readiness_2026-10-17.md` that needs no approval: item 2 of its section 5, "things that can be struck and broken by the folk", and the smaller items of item 6 that the shelter and Into the Dead turn on. Survival needs, live drawing and enemies drawn by animated characters still wait for the user's word.

- **Barricades** (`barricade N [NAME]`, a line of a kind: kind.rs TRAITS, ONCE, OWN). A thing in the way that takes N blows. A blow is the walker's strike or dash, or one of the folk's, and each is the fact `battered` about the thing, heard 12 m off. One that hunts or searches stands and strikes it when it lies along the way to its quarry, or when it is kept out by it. It strikes with its own `attacks` move, or else the first of its set that strikes (claw, swing, bite, strike). One that only catches by coming up to the walker breaks it down too. With N taken, it breaks as `breaks` does (`watch.gd`), with one difference: it is drawn lying flat where it stood and walked over, and the ways go through again. The fact `broken` about it holds while it lies. The new act `board_up` ("put up", a piece of scrap handed to it, `clears broken`) puts it up again. Barricades live in `fight.gd`, beside the rest of the striking, rather than in a script of their own.
- **Locks.** The acts `unlock` (a lockpick handed over and used up, `puts unlocked`), `open_unlocked` and `search_locker`. A locked door (`door-locked`) and a locked locker (`locker-locked`) are content only. Those who open doors cannot go through a locked door, because it offers no `open`, which `nav.gd` asks for.
- **Crafting.** An act's goods lines are now checked one after another, against what the lines before them leave. One act can therefore hand scrap to a workbench, use it there, make a lockpick and take it (`make_lockpick`). Before, every line was checked against the goods as they stood before the act. The good `lockpick` is new.
- **Searching heard.** The act word `heard R` makes a noise at the act's moment (`watch.gd` hears it). Searching is heard 5 m off, nailing boards 10 m, picking a lock 3 m.
- **Stamina**, a Player start setting (`stamina`, seconds; 0, as before: no limit; Makers: "runs for, seconds"). At a run, breath runs down. With none left the walker is winded and walks until a third of it is back, which comes back at half the rate it goes. It is shown as Breath under Health. It is a setting of the world and not of every game, so the course, the town and the stealth level run as they did.

The shelter genre has boards over the doorway at its east end, a locked locker on the top floor, a workbench that makes lockpicks, the dead nearer the house, and `stamina 8`.

**Not built:** boards put up where none were (only those broken are put up again); a door the folk batter (a door is not a barricade); keys as items; what is found drawn from a table by chance.

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7bv. Houses of storeys, seen side-on (built 2026-10-17)

The user asked, after the enemy section and building by drawing, for "modular for the 2.5D style buildings like in this war of mine and into the dead our darkest days". A building on cells may now be of up to six storeys (`storeys`, World outline: Makers, its cells' part). Each storey has a facade of its own with its own windows (the prototype's facade makes one row; one is stacked per storey rather than changing it), walls inside go up through every storey with their doors, boards lie over every cell at each storey but the one a flight comes up through, and a flight of steps goes from each floor up into the next cell along (`stair` names the first cell). The roof stands on the top storey. Stacking the flights in one cell was tried in thought and given up: the upper flights would begin over the hole below them. Each flight therefore climbs into the next cell, and each floor has a hole only where the flight that reaches it arrives.

Seen side-on, a wall that is cut away (`cutaway`) is now simply not drawn: no ghost, so the cross-section reads as This War of Mine's does. Seen from behind, the faint ghost is kept as before. A building's corner posts are cut away with its walls.

For a shelter, the library has a new collection, Shelter (side-on): a cupboard, a toolbox and a first-aid box that hold food, scrap and medicine (new goods); the acts `search_food`, `search_scrap` and `search_medicine`; rubble that breaks; a cot, a stove and a workbench; and the dead who wake at night. The genre "A shelter seen side-on (2.5D)" (host `examples/shelter.rs`) is a house of three storeys, five cells along the line and three deep, built of these.

**Not built** (see `Draw3D_Genre_Readiness_2026-10-17.md`): ladders and holes; dark rooms not yet entered; searching that takes time and is heard; locked doors and keys; barricades the dead break; storeys in a drawn house (its facades stack, but it has no upper floors or stairs).

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7bu. Building drawn on the land, as Tiny Glade has it (built 2026-10-17)

The user asked for "modular building like tiny glade style with decor and surrounding things like ponds and gardens". A structure that is fed a line (the line tool's `route`) and told no cells follows the line instead. Its rim is the line, cut into pieces of no more than 3 m and named by side and piece (`s2_0`), with a pier where it bends. When the line ends beside where it began, it goes round, and what it goes round is its inside. One generator still makes every structure, so a drawn wall and a wall on cells are made of the same pieces. A second, freehand generator was considered and not built, because it would have duplicated all of `build.rs`.

New words of a structure: `towers` (a round tower with a cone roof at every turn of 40 degrees or more, and at the ends of an open line), `dress` (flowers, bushes, rocks, ferns or grass strewn along the outside of its foot), `line_gates` (the sides of a drawn line with a gate, set by a click near a side), and the sort `garden` (a low edging, soil inside, flowers, wild flowers and low bushes in offset rows). Inside a drawn line there is a floor the shape of the outline (`polyfloor`, cut into triangles one ear at a time, so an L is filled as it is), a pond's bed and water (`polywater`), or a garden's soil. A drawn house gets one of two roofs. If every side runs along the first or square to it (an oblong, L, T or U), it gets a house's roof over each oblong of the outline; otherwise a roof that rises from every side to a point (`polyroof`).

Build > Draw on the land has the seven: a stone wall with towers, a fence, a hedge, a house of any shape (its door in the first side), a pond, a garden, and a paved yard. In World outline the structure says it is drawn and how many sides its line has. It has buttons for the line's places, the gates and the towers, and a box for each kind of dressing. The host answers `build_gate` for a drawn structure by the side nearest the click (`tileops::line_gate`).

**Not built:** the structure made live as the line is drawn (it is made when the line is ended); a curve through the places; a gate anywhere along a side; ivy; a pond dug into the land; bridges.

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7bt. Fights, and creatures among themselves (built 2026-10-17)

The user asked to work on "the advanced enemy behaviors etc. … until the completion of the enemy section". This is the first of 7bp's three recommended steps, the second tier of composable behaviour, together with the catalogue's "Combat and creatures" rows. Everything goes through `watch.gd` and `folk.gd`, plus one new script, `fight.gd`. Each piece is a line of a kind (host `kind.rs` TRAITS, ONCE, OWN, so one thing may say it too in the Inspector):

- `health N [NAME]`: it falls after N blows (a drawn blade's count two), lies where it fell for the run and in a saved game (the fact `fallen`), and NAME counts the fallen. Struck by the walker before it has noticed them, it falls at once and is hardly heard (a takedown).
- `attacks MOVE R DAMAGE S`: hunting, within R it stands, turns and does the move. The moves `swing`, `claw` and `bite` are new, each wound up long enough to be seen. What the move strikes is hurt.
- `shoots R DAMAGE S SPEED`: a shot that flies, stopped by anything solid; it hurts whoever it meets first, but never one of its own side.
- `enrages F`: badly hurt, twice as quick, at a run.
- `side NAME`, `preys NAMES`, `flees R [NAMES]`, `defends R`, `calls R`.

One that hunts the walker without `attacks` still catches them by coming up to them, as in 7bo, so the stealth level plays as before, except that its hunters are now as quick as their sets say. Fallen, what one carries (`stock`) is taken from it with E: two acts, `loot_food` and `loot_scrap`, are offered `while fallen`. A bandit has a tin of food, the brute scrap. No new mechanism was needed, only content.

The walker has five health while anything about can hurt them. The dash button is a blow (`strike`, a new move of the walker's set, `when foe`: the motor's new condition, set by `fight.gd`) when someone to fight is just before them. Dashing, dodging or rolling steps out of a blow or a shot. With none left the walker is down, back at the last checkpoint and whole again. Allies' blows and shots never hurt one another. Someone who sees one of its own side lying comes to look and calls. Hunters near and in sight run straight at their quarry rather than by the ways. Hunters now run at full pace: before, they ran at half of their set's run speed, an oversight of 7bo.

The library's Creatures and enemies gain bandits, an archer, a brute, wolves, a fox, deer and rabbits; the guards, the dead, the stalker and the hound gain health and sides. The genre "Fights and creatures (the wilds)" (host `examples/wilds.rs`) is a camp, the dead that hunt it, a wolf pack and its deer, and a fox and its rabbits. Nothing in it is scripted.

**Not built** (`Draw3D_Genre_Readiness_2026-10-17.md`, section 1): blocking and parrying, attack patterns and phases beyond `enrages`, weak points, status effects, drops, flanking, enemies drawn by animated characters.

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7bs. Findings on the journey (built 2026-10-17)

The user found that an item dropped could not be picked up again, and that the eye went wrong going from world to world. The first was an item dropped in a world not its own: in no cell of it, so not offered; such an item is now found from what was carried in with it. The second: the eye kept its turn, tip and room of the world left. On coming into a world the walker now faces away from the way, and the eye is behind them as it is at first, turned a little aside to where it has room and the way is not between. Chosen over moving the walker farther from the way (a cave mouth would leave them far off) and over seeing through ways (the cutaway would have to know them).

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7br. Items and a saved game across worlds (built 2026-10-17)

The user said: "Make inventory items and a saved game go across worlds on a journey."

**Items** are things of their world, and things of two worlds may have one id (node ids are counted from the same start in every world). So an item has its own id only in its own world; elsewhere one made of its world and its id, with what it is carried along. The world it left says it is `elsewhere`. Chosen over moving the thing into the other world's document (editing a world while it runs, and its file) and over a list of items apart from the facts (which a saved game, a drop and the inventory would all have to learn).

**A saved game of a journey** is one file of every world's saved game and which world the walker is in; loaded, the host puts each world's facts back (opening one not gone into yet), and the editor goes into the walker's world as through a way.

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7bq. Connected worlds (built 2026-10-17)

The user chose, of 7bp's three recommended steps: "Build connected worlds: a thing that leads to another world, and a hex overworld whose tiles lead into local worlds."

**Words:** `leads WORLD [NAME]` and `enters R` (host `kind.rs` TRAITS, ONCE, OWN; `leads` a signal word, so the host lists every way of a world whole). A way is any thing; the library's "Ways between worlds" has four made for it.

**Worlds apart while running.** The host keeps the worlds of a journey as they were left (`Away`: engine, history, facts, running, unsaved count) and swaps them; the editor is sent what differs, as after any change. Stopping goes home first, so the world being edited is never written over and keeps its history; a journey is only while the world runs. Chosen over writing the edited world to a file and opening the other (which would lose its history and its unsaved mark) and over a saved game per world (which would not keep who stands where and what is loose).

**What goes with the walker:** goods and the hour. Not yet: things kept (they are things of their world), followers, a saved game of the whole journey, a built game's other worlds.

**The hex overworld** is an ordinary world of hex terrain tiles with markers on some hexes; a local world is an ordinary world with a way back. Nothing of it is a special generator: every part is authored with what exists (layouts, tiles, the library, the Inspector), which is the direction's point.

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7bp. Enemies set by hand, rooms per run, walls seen through, the frame rate held (built 2026-10-17)

The user said (2026-10-17): "accessing the inventory during the game froze my computer. how do i edit the properties of the enemies and where they roam and how far they see etc? anything you build I should be able to replicate. when i click on the enemy i don't see relevant parameters anywhere. you can proceed with rooms that rearrange and cutaway walls plus as directed below", with a direction: "Draw3D — Open-World Styles, Mechanics Library and Demo Priorities" (inspect first, catalogue existing, partial and missing, reuse, bounded steps, no commit without approval).

**The freeze.** The editor drew without limit (vsync off, no cap): about a thousand frames a second behind the inventory, enough to stall a laptop's graphics and the whole machine. `main.gd` `pace`: the screen's rate (60 to 144), 60 with a game screen or the world held, 20 out of focus; automated runs and measurements unchanged.

**Enemies by hand.** The Inspector's "How it sees, hears and goes" for any thing of a kind put down by hand; its round dragged on the world (`line.gd` generalised to one thing's own places); sight and hearing drawn on the ground. Every number is a line of the thing's own (`own`), the same words a kind's text has, so what the Inspector sets can be written in F6 and the other way.

**Rooms per run.** `room NAME VERSION`, read by `rooms.gd`: one version of each room each run, by a seed; versions stand stacked in place; shown one at a time while editing. Reordering rooms is not done: a version is chosen, nothing is moved. Things are left out through the view's `stirred` (as a door or a broken crate), so what is not there is not in the way.

**Cutaway.** `cutaway` (a word; a structure's parameter for its building). Seen-through things are not drawn but still collide; a ghost of each is.

**Catalogue.** `Draw3D_Open_World_Catalogue.md`. Recommended next, each a larger step for the user's approval: composable behaviour's second tier (prey, predators of a kind, territory, fighting back), connected worlds (`leads WORLD`, a hex overworld of tiles leading into local worlds), survival needs as meters.

**Checked:** as PROGRESS says under Done. Not used by hand.

## 7bo. Side-on 2.5D, perception and stealth, creatures, a level and genres (built 2026-10-17)

The user said (2026-10-17): "you are authorized to do the 2.5d camera as well as the stealth and enemies that see and hear the player as well as the mechanics in the reference and additional styles in an order that cost the least. you may download additional animations that work better with the other genres. You can create a few enemies / creatures as an intro to the creature studio/catalog/library. you may build a demo level or levels to showcase the mechanics with a toggle if a level can showcase multiple. demo can go into the project management area which is to be sorted out at some point as well to be used as a potential genre when starting a project".

**Order, cheapest first, each on what was there:** the side-on eye (an eye of `eyes.txt` with two more words); perception and enemies (kind words read where the others are, steered by `folk.gd` as everyone is); the cheap mechanics of the reference as words on things (hides, harms, slippery, breaks, checkpoint); the creatures as a library file; five styles; one level built by a host example; genres as a text the New world window reads.

**Side-on (2.5D)** (`eyes.gd`, `eye.gd`, `player.gd`): an eye may say `round DEG` (held looking that way) and `plane [lanes D]` (whoever it follows goes only across it, held to the line they were on, W and S a lane farther or nearer). `eyes.txt` has `side`. A script leading the walker is held to the line too. The Player start maker has a parameter `eye` (host `land.rs` `Start`; the `nodes` answer carries it; World outline, the start: "seen through the eye"), so a world begins side-on.

**Perception and enemies** (`watch.gd`, new; words in host `kind.rs` TRAITS/ONCE/OWN and `things.gd`): `sees R DEG [H]`, `hears R`, `hunts [MOVE]`, `patrols X Z ...`, `spots NAME`, `sweeps DEG S`, and `when` honoured (a watcher whose `when` does not hold sleeps). How well the walker is seen: crouched, still, in what is gone through, hidden, in the dark away from a lamp. What is heard: steps by speed (run 9 m, walk 3.5, crouched 1), a hard landing, a thing broken, an alarm, every sound `sound.gd` plays at a place. Noticing grows by distance and visibility; modes calm, wary, hunt, search; caught within 1.15 m, the walker back where last safe (start or checkpoint, on the side-on line and lane they had there), hunters back at their posts. Facts: `caught`, `hunted`, each `spots` NAME. Shown: a "?" or "!" over each, cones on the ground cut by walls, a line at the bottom (seen %, why), words at the top. Sounds `spotted`, `caught`, `break` (`sounds.txt`). Move sets `guard`, `shambler`, `hound`, `stalker` (`moves.txt`).

**Mechanics of the reference, as words on what is met:** `hides` (crouched in it, unseen), `harms [WORDS]` (hurt, back to the last checkpoint), `slippery [N]` (the motor's new `slip`: accel, brake and turn that much less), `breaks [NAME]` (struck by a striking move or dashed into: `where` "broken", not drawn, walked through, pieces, a noise, NAME counts), `checkpoint [WORDS]` (passed within 3.5 m). Security cameras, camouflage when still, hiding spots, hazards, sludge, destructibles, checkpoints: of the reference's N and R items these are now E.

**Creatures** (`editor/library/creatures.txt`, collection "Creatures and enemies", the first of a creature catalogue): guard, shambler (the dead), stalker (`when dark`), hound, camera on a post; hiding grass, crate to break, acid pool, sludge, checkpoint lamp; low wall for cover, a backdrop wall for side-on levels, low and high platforms. Each a kind, its lines readable and changeable (F6); one thing may say the stealth words of itself (Inspector).

**The level and genres:** `editor/genres/stealth-side.json`, built by `host/examples/stealth.rs` (`editor/genres.ps1`): six parts west to east (grass and a patrolling guard; a camera, cover and a watchman; crates; sludge and acid; the dead; the stalker by night, platforms, the way out), each guard a kind of the world with its round and `when not quiet`. `editor/godot/genres.txt` (`genres.gd`) lists the genres; World > New world offers each ("Genre: Side-on stealth (2.5D)"), opened as a copy with no file. Playtest has the level's switches: side-on, night, the cones, and each fact the watchers go by (a toggle for the several things one level shows).

**Styles** (`styles.txt`): moonlit_stealth, security_feed, toxic_works, bright_side, last_light, after genres rather than one game.

**Animations:** none downloaded. Universal Animation Library 1 and 2, already here (CC0), have crouch walking, a zombie walk, idle and scratch, hits, a death, a torch idle and pistol clips; the enemies are made of parts on the person's and the sheep's bones as the village's people are, and move by their sets. Showing an enemy as a clip-animated character (the zombie clips) is the next step for them, not done.

**Not done (larger, each for the user):** vehicles (the bike of Steel Rats, air vehicles), rooms re-rolled per run, character switching, guardians and encounter phases, magnetism and light beams, ink, cutaway walls for side-on interiors, enemies shown by clip-animated characters, a level of survival (barricade, needs) though its parts exist (ticks, work, items).

**Checked:** `-Demo stealth -Check`, 10 beats: genre listed and opened; side-on, held to the line (0.0 m off), a lane 2.5 m; in the grass crouched seen 0%, the guard's most 0.05; standing in his way hunted after 1.7 s, caught, back 31 m at the start; behind the low walls the camera never saw; in the open it hunted, `alarm` held, the watchman came searching; a crate broken by a dash, `crates_broken` 1, through; sludge slip 0.85; acid hurt, back at the checkpoint; the dead heard steps; the stalker asleep by day, awake at night; Playtest's switches eye/night/cones/quiet, quiet sleeping the guard, the eye from behind again. The first run found: the guard's round in sight of the start, the camera's sight ray beginning inside its own post (never saw), and a dash into a crate stopped before it was asked about; all mended. Host tests and the regression parts as PROGRESS says. Not used by hand.

## 8. Change log line for the master (to copy in when the user approves)

| 2026-10-09 | Amendment file | Principle O (content as data); terrain and geometry editing made concrete; six edit-preservation mechanisms; editor workspaces; next-phase order M1–M4. Master unchanged. | Reconciliation after the nine-stage runtime; verified audit. |
