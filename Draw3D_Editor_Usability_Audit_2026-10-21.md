# Draw3D — Editor usability and cognitive complexity audit (2026-10-21)

An audit, not a change. Nothing in the editor was modified to write it. It answers the
brief of 2026-10-21 ("the editor exposes too much implementation terminology, structural
complexity and unnecessary configuration").

It builds on two earlier documents and does not repeat them:
- `Draw3D_UI_Inventory.md` lists every control. Its sections 11 ("one task spread over
  several places") and 12 ("names that are suspect") are the start of this audit.
- `Draw3D_UI_Overhaul_Proposal.md` (2026-10-15) covers window sizing and panel layout.

This audit is about **words, concepts and decisions**: what the editor asks a person to
understand before they can do an ordinary thing.

---

## 0. How this was done, and what it can and cannot claim

**Evidence used:**
- **Code.** Every label, tooltip and status message quoted here was read in the GDScript
  that produces it. File and line are given. The main files were:
  - `rulesview.gd` (Behaviours and rules)
  - `inspect.gd` (Inspector)
  - `desk.gd` (bar, toolbar, context row, menus)
  - `buildbar.gd` (construction bar)
  - `terrain.gd`, `abilities.txt`, `behaviours.txt`, `uses.txt`, `acts.txt`
  - `studio.gd`, `styleview.gd`, `main.gd` (window titles)
- **Screenshots of the running editor.** These were taken by the scripted demos in
  `editor/godot/demo/` (for example `01-behave-made.png`, `02-behave-given.png`,
  `07-abilities-follows-inspector.png`, `01-wired-inspector.png`,
  `04-studio-movement.png`, `12-sorts-palette-window.png`). They show what the code
  draws. They were made by a script, not by a person.
- **Your report.** The ranger sentence, quoted in the brief, is the one piece of
  evidence here that a person was confused. It is marked **Observed (user)**.

**What was not done:** I did not drive the live editor by hand for this audit. That
keeps to the standing rule that automated runs never take your mouse. Nothing here
claims a person found a screen confusing, except where you said so.

**Labels used on each finding:**
- **Observed (code)**: the wording or flow is exactly what the code produces.
- **Observed (screenshot)**: also visible in a demo screenshot.
- **Observed (user)**: you reported it.
- **Inferred risk**: my design reasoning about a likely problem. It has not been seen.

**A note on authorship.** I wrote almost all of this interface across many passes. I
have treated it as a set of hypotheses to test, not a design to defend. Several root
causes below are habits of mine that I carried from pass to pass.

---

## 1. The root causes

The findings in section 2 mostly come from five causes. Fixing a cause fixes many
findings at once, so the plan in section 7 is organised around the causes.

### R1. A private writing voice used for every label

Almost every label, tooltip and status line in the editor is written in one unusual
register. It avoids ordinary software words and writes a description of the data in
place of a name. Examples, all **Observed (code)**:

| Where | What it says | What a person would expect |
|---|---|---|
| Behaviours window, top (`rulesview.gd:368`) | "Ranger (a ranger): behaves as 'run_away', its own (its design, ranger, says 'deer_ways')." | "Ranger: **Runs away**. (Other rangers: Graze and flee.)" |
| Same (`rulesview.gd:359`) | "Nothing is picked. Pick a thing in the view (a left click), and it is shown here: what it behaves as, and the behaviour chosen given to it." | "Select a character to give it a behaviour." |
| Same, after giving (`rulesview.gd:1459`) | "Ranger behaves as 'run_away', its own. Ctrl+Z takes it back." | "Ranger now runs away. (Undo: Ctrl+Z)" |
| Context row (screenshot `07-abilities-follows-inspector.png`) | "Works on: this one thing, not its design. Picked: one cot_own_cot — put down by itself." | "Selected: Cot" |
| Inspector (`inspect.gd:1810`) | "'cot', put down by hand, of the design 'cot_own_cot'." | "Cot" (and the design under Advanced) |
| Inspector (`inspect.gd:1873`) | "Nothing: it stands there and is in the way, and does nothing in play." | "No gameplay. [Make it usable…]" |
| Inspector, conditions (`inspect.gd:1878`) | "No condition: it is always as it is." | (nothing shown) |
| Abilities (`abilities.txt`) | "says runs from you", "needs creature" | "Runs away from the player" |

**Why it matters:**
- The sentences are accurate, but they **describe the engine's model**: own line against
  design line, put down by hand against made by a maker.
- They **do not name the outcome**. A person scanning for "runs away" or "door" or
  "selected" does not find those words.
- Words such as "its own", "put down by hand", "goes by", "says" and "behaves as" are
  used as technical terms, but nowhere are they defined as terms.

The ranger sentence the user reported is this voice at its worst. It carries four
concepts at once:
- an instance name
- a design name
- a behaviour identifier
- a precedence rule (own over design)

A person needs none of these to see that the ranger runs away.

### R2. Internal identifiers and file names shown as if they were words

Shown to the person by default, all **Observed (code/screenshot)**:
- **Identifiers with underscores:** `run_away`, `deer_ways`, `door_lock`, `cut_thrust`,
  `dodge_counter`, `cot_own_cot`, `toolbox-search`, `my_my_lookout`.
- **File names:**
  - `behaviours.txt`, `moves.txt`, `acts.txt`, `abilities.txt`, `gear.txt`,
    `styles.txt`, `stats.txt`, `sequences.txt`
  - These appear in tooltips, the Inspector glosses (`inspect.gd:3195-3248`) and window
    sections ("the set 'creature' of moves.txt").
- **Function keys as part of labels:** "Its design's text (F6)", "Its acts' text (F10)",
  "Edit its text (advanced, F6)".
- **Engine notation:**
  - "the one thing 'cot' makes (kind#it)"
  - "kind live"
  - Raw lines in the Inspector: `follows 2.5 you`, `someone`, `body 0.4 0.8`,
    `weighs 40`, `offers search_scrap top`, `point top 0 0.4 -0.25 from 0 -1`,
    `on act search do give coin 2`
  - Screenshots: `07-abilities-follows-inspector.png`, `01-wired-inspector.png`.

The Inspector does add a plain gloss after each raw line ("— it holds 3 scrap to begin
with"). That is useful. But the raw line comes first, in the same type. The gloss is the
part a person needs, and it sits second.

### R3. One outcome, several systems, each with its own window and words

The engine grew features in tiers. Each tier got its own place in the editor. So one
intention can now be expressed in several places, which differ in subtle ways.

**"Make this run away"**, **Observed (code)**. There are three mechanisms:
1. **Ability `shies`** ("runs from you"). Inspector → What it can do → Give it…
   (`abilities.txt:91`).
2. **A design word `flees`**. Written in the design text (F6).
3. **Behaviours `run_away`, `skittish`, `deer_ways`**. Behaviours and rules (Ctrl+B,
   `behaviours.txt:83-137`).

They interact by a rule the window states in passing: "while one behaves so, its
design's own running away and straying (flees, shies, roams) wait, and the behaviour
decides" (`rulesview.gd:400`). So a person can give both, and one silently overrides
the other.

**"Make a door the player can open"**, **Observed (code)**. There are four mechanisms:
1. Acts `open` / `shut` / `try` (`acts.txt:160-187`).
2. The design word `opens`. This is listed by `abilities.txt:49` only as "elsewhere: a
   design's text (F6)", so it is not in the Give it… list.
3. A `key` line ("it is opened with a key").
4. The behaviour `door_lock` (`behaviours.txt:52`).

On top of these, drawn houses get doors of their own (`buildbar.gd:310`, `1809`).
"Make it usable" offers bed, seat, food, drink, medicine, container, chest, well,
weapon, creature, flyer and pet. **It does not offer a door** (`uses.txt`).

**"When X happens, do Y"**, **Observed (code)**. There are five mechanisms:
1. `on TRIGGER do ACTION` rows. Inspector, "When this happens, do this" (`wires.gd`).
2. `while WORD CONDITION`. Inspector field (7cf).
3. Behaviours with states and events (Ctrl+B).
4. Acts' `while` / `unless` (F10 text).
5. Conditions and facts, watched in Signals (Ctrl+K).

The Inspector has a section "What it goes by: conditions and facts" that reports on
them. Every one of these is legitimate. But a person cannot tell which to reach for, and
the editor never says "for this, use that".

### R4. The scope decision ("this one or every one of its design") asked first, everywhere

The engine's model has a design (a kind) and the things placed from it. A thing may
carry its own lines that override its design. The editor puts this distinction in front
of nearly every edit, **Observed (code)**:

- **Behaviours.** Two buttons side by side, "Give this one 'X'" and "Give every ranger
  'X'" (`rulesview.gd:374-375`). The status sentence then reports which one applies,
  "its own" or "as every ranger does".
- **Parameters.** Two columns: the default for every one, and "this one's"
  (`rulesview.gd:459-474`).
- **Abilities.** Giving an ability that only a design may say **silently makes a new
  design**, named `cot_own_cot`, "like the cot" (`abilities.txt:27-30`, screenshot
  `07-abilities-follows-inspector.png`). The person did not ask for a new design. They
  see its name from then on.
- **Abilities on a maker's thing.** "Only a thing put down by hand is given abilities
  here … Ctrl+D puts a copy of this down by hand" (`inspect.gd:1924`). Giving a tree in
  a forest an ability first requires understanding makers, hand placement and
  duplication.
- **Context row.** "Works on: this one thing, not its design."

Scope is a real and useful capability, and it must stay. The problem is that it is
asked **before** the task, on equal terms with it, every time. Most edits want "this
one", and "all of them" is the exception.

### R5. Everything shown at once; explanation in place of defaults

- **The Behaviours window opens with all of this showing at once** (`rulesview.gd:88-160`,
  screenshots `01-behave-made.png`, `02-behave-given.png`):
  - For
  - Behaviour, with New / Copy / Rename / Delete / Save to the library / Bring from the
    library / Bring in from words…
  - What it is for
  - Needs of whatever has it
  - Who behaves so
  - Parameters, with Add a parameter
  - States and steps, with Add a step / Add… / Put inside / Up / Down / Remove
  - Problems
  - Running

  A person who wants "this ranger runs away" needs one chooser and maybe two numbers.
  The library management and the step editor are for a different task: authoring a
  behaviour.
- **Tooltips of 30–60 words** carry the explanation that labels do not. Examples:
  "Duplicate (Ctrl+D)" at `desk.gd:369` and "Show it alone" at `desk.gd:382`. Tooltips
  are slow to reach, and the long ones are rarely read.
- **Menu entries are sentences.** Examples:
  - "Save the world  (to X.json; what that file has is kept first)"
  - "Autosave every 5 minutes while there are changes in no file  (to
    backups\autosave-NAME.json; offered after Draw3D stops without closing)"
  - Both at `desk.gd:582-585`.
- **The Inspector opens with a lot before HOW IT WORKS.** Position in metres, turn in
  degrees, "kind live", and "the one thing 'cot' makes (kind#it)" come first. Then
  HOW IT WORKS, in capitals, followed by every section unfolded (screenshot
  `07-abilities-follows-inspector.png`).

### R6. The same control in two or three places, under different names

All **Observed (screenshot)**: `01-behave-made.png`, `01-bar-build-open.png`.

- **Build** appears three times, and each opens something different:
  - The bar menu "Build v" opens every construction tool as a list (`desk.gd:289`).
  - The left toolbar "Build ›" opens roads, lots, streets and grid structures.
  - The right construction bar "Build ‹" opens house, tower, wall, fence and hedge.
- **Paint** appears twice: left toolbar and right bar. The left one alone holds three
  unlike things (ground, objects, where plants grow; `desk.gd:614-649`).
- **Erase** appears twice: left toolbar and right bar.
- **Colours / look of the world** go by four names:
  - Sky (toolbar)
  - Environment (window title)
  - Styles (window)
  - Look (construction bar)

  There are two palettes. The world style's palette is in Styles (`styleview.gd:16`).
  A house's palettes are in Look (`buildbar.gd:778`), and one of them is called
  "as the world's style".
- **"Paint where plants grow"**, **"clear"** and **"let grow"** are three names for one
  idea: where vegetation is allowed (inventory section 12 already flags this).

### What already works, and should be the model

Some parts of the editor already follow the standard proposed in section 5:

- **The construction bar** (`buildbar.gd`, the right side):
  - Pictures, one-word names: House, Tower, Wall, Fence, Hedge, Path, Water.
  - "Nothing is asked first: pick one and drag on the land" (`01-bar-build-open.png`).
  - Its tooltips start with the action: "A house: press on the land and drag."
  - You judged this direction "finally good" (7cv) and were "extremely pleased" (7cw).
    That is the strongest evidence in this audit of what works for you.
- **The Character Studio's Combos tab** (`04-studio-movement.png`):
  - Columns: Combo, Moves ("slash → thrust"), and "Ready to see".
  - A timeline with the phases coloured.
  - Its weakness is only the identifiers it shows (`cut_thrust`).
- **Undo** is named in every change message ("Ctrl+Z takes it back"). That feedback is
  good. Only the wording is long.
- **Terrain tools** (`terrain.gd:71-84`): each tool has one short paragraph that is
  shown while the tool is chosen. The paragraphs are long but task-shaped ("Raise:
  lifts the ground under the brush").

---

## 2. Prioritised findings

Ranked by user impact:
- how often the surface is met,
- multiplied by how much it blocks an ordinary task.

Each finding has the same parts: the screen, the wording, the user's goal, why it is a
problem, the proposal, what must keep working, and the files.

### F1. Behaviour of a selected character (highest impact)

**Screen:** Behaviours and rules window (Ctrl+B, or Inspector → Behaviour…).
**Evidence:** Observed (user), Observed (code, screenshot).

**Wording:**
- "Ranger (a ranger): behaves as 'run_away', its own."
- Buttons "Give this one 'run_away'" and "Give every ranger 'run_away'".
- "Needs of whatever has it: a body that goes about by itself (someone)."
- "It moves the body: while one behaves so, its design's own running away and straying
  (flees, shies, roams) wait".

**User's goal:** make this NPC run away from the player. Later, perhaps, tune the
distances.

**Why it is confusing:**
- R1, R2 and R4 together.
- The window's first job, choosing a behaviour for the selected character, is mixed
  with a second job, managing and authoring the behaviour library.
- "its own" is engine vocabulary for an instance override.
- "(a ranger)" repeats the design name with no explanation of why it matters.
- "someone" is a raw trait word.
- The shies / flees / roams warning reveals the three-system overlap (R3) without
  resolving it.

**Proposal (interaction redesign, presentation over the existing model):**

```
Ranger                                         [Edit steps…]
Behaviour   [ Runs away from the player   v ]
            Runs from someone until far enough away, then stops.
  From              [ Player   v ]
  Runs until        [ 25 ] m away
  Gives up after    [ 12 ] s
                                  ( ) Only this ranger   ( ) All rangers
```

Steps for the user:
1. Select the ranger.
2. In the Inspector, open the Behaviour drop-down.
3. Choose "Runs away from the player".

That is the whole task. The scope choice is a small radio group, and it defaults to
"Only this ranger".

The status line reads: "Ranger now runs away from the player. (Undo: Ctrl+Z)". Only
when other rangers differ does it add: "Other rangers: Graze and flee".

The full Behaviours window keeps every capability. It opens from **Edit steps…**, and
it opens on authoring. Its sections fold by default, except States and steps. Library
management (New, Copy, Rename, Delete, Save to library, Bring from library, Bring in
from words) moves into one "Behaviour ▾" menu.

**Must keep working:**
- `behaves NAME [with …]` lines, on a thing or a design.
- Per-instance parameters.
- One undo step for each change.
- `rules.gd` precedence.
- The `-Demo behave` and `-Demo rules1` demos.
- The `needs` / `lacks_of` check. It shows as a warning only when the character
  cannot behave so: "Needs to be a creature that moves on its own. [Make it one]".

**Files:**
- `rulesview.gd`: `_fill_for`, `_fill_about`, `_fill_params`, `_init`
- `inspect.gd:1896`
- `behaviours.txt`: a display name per behaviour, see F2

**Engineering:**
- Mostly presentation.
- One content addition: a `title` line per behaviour, for display names.
- Showing the `shies` ability in the same picker (F3) is a small mapping. It is not a
  rewrite.

### F2. Identifiers and file names shown by default (affects every screen)

**Evidence:** Observed (code, screenshot). See R2 for the list.

**User's goal:** read what something is called.

**Problem:**
- Underscored ids and `.txt` names read as programmer artefacts.
- `cot_own_cot` and `my_my_lookout` are generated names. The person never chose them.

**Proposal (cosmetic, plus one small engineering step):**
- Every list that shows a behaviour, move, combo, style, design or act shows a **display
  name**:
  - from a `title` / `called` line where the content has one (designs already have
    `called`; assets have `called Windmill`),
  - otherwise a humanised id: `run_away` → "Run away", `deer_ways` → "Deer ways".
  - The id stays visible under Advanced, and in the text editors.
- File names appear only in Advanced sections and in buttons whose job is to open that
  file ("Open styles.txt" can stay, under Advanced).
- Generated design names take the user's words: "Cot (follows you)", not `cot_own_cot`.
- Function keys move out of the label into a dimmed shortcut column, as menus already
  do for Ctrl+S.

**Must keep working:** ids in saved worlds and texts, which are unchanged.
**Files:** `rulesview.gd`, `inspect.gd` (`_gloss`, `works_what`), `studio.gd` lists,
`styleview.gd`, `designs.gd`, `abilities.gd`, `desk.gd` (`_picked_words`).

### F3. Several mechanisms for one outcome, with no guidance (R3)

**Evidence:** Observed (code).

**User's goals:**
- run away
- open a door
- when X do Y

**Problem:**
- The person must know which system to use, and that systems can override one
  another.
- The editor says so only in a dim line once the overlap already exists
  (`rulesview.gd:400`).

**Proposal (interaction redesign; small engineering):**
- **One "What it does" list per selected thing**, in the Inspector. It mixes, in one
  list and by outcome, the abilities (`abilities.txt`), the behaviours
  (`behaviours.txt`) and the use presets (`uses.txt`):
  - "Runs away from the player"
  - "Follows the player"
  - "Wanders nearby"
  - "Opens and closes (door)"
  - "Can be sat on"
  - "Graze, watch, then flee"
- The person picks an outcome. The editor writes whichever lines that outcome needs,
  exactly as it does today.
- When two picks conflict (a movement ability and a behaviour that moves the body), the
  list says so when the second is picked, and offers a choice: "Replace 'Wanders
  nearby' with 'Runs away'?" Today both are kept and one waits silently.
- The "When this happens" rows and the "while" field stay where they are. The
  behaviour editor gets a one-line pointer in each: "For several steps or states, use a
  behaviour: Edit steps…".

**Must keep working:** every line format; `abilities.gd`'s give and take-away; each
`uses.txt` preset; Signals.

**Files:** `inspect.gd` (`_fill_abilities`, `_fill_usable`), `abilities.gd`,
`rulesview.gd`, `abilities.txt`, `uses.txt`.

### F4. The Inspector reads as a dump of the data (R2, R5)

**Evidence:** Observed (code, screenshots `07-abilities-follows-inspector.png`,
`01-wired-inspector.png`).

**What it shows, top to bottom:**
1. kind
2. position
3. turn
4. "the one thing 'cot' makes (kind#it)"
5. HOW IT WORKS
6. "'cot', put down by hand, of the design 'cot_own_cot'."
7. "What it does in play", with raw lines
8. What it goes by
9. What it can do
10. Make it usable
11. Its shape
12. a row of buttons to other windows: F10, Behaviour…, F6, Signals…, World outline

**User's goal:** see what this is and change the thing they came to change.

**Proposal (interaction redesign):** a fixed order:

1. **Header.** Display name and a one-line summary built from the glosses that already
   exist: "Cot — follows the player at 2.5 m".
2. **What it does.** Each entry is a plain sentence with its key numbers inline, and an
   × to remove it. "Add…" opens the list from F3.
3. **Look.** Colour, texture, shape.
4. **Placement.** Position and turn, folded.
5. **Advanced**, folded and remembered per person. It holds:
   - the raw lines
   - design and kind
   - the maker
   - conditions and facts
   - the links to text editors and Signals

The standing rule "a click shows how it works" (7bz) is kept. Everything is still
reachable from the click, one fold away.

**Must keep working:**
- every editable field
- HOW IT WORKS content
- "Never shown while playing"
- `-Demo held`, `-Demo wired`, `-Demo abilities`

**Files:** `inspect.gd` (`_fill_works` and the sections it calls).

### F5. Duplicate entry points: Build ×3, Paint ×2, Erase ×2, four names for "look" (R6)

**Evidence:** Observed (screenshot, code).

**User's goal:** find the tool.

**Problem:**
- The same word leads to different lists.
- Different words lead to the same window.

**Proposal (interaction redesign, layout):**
- **The right construction bar becomes the one place for Build.** It is the precedent
  that works. Roads, streets, lots and grid structures join it as further groups.
- **The left toolbar keeps the general tools:**
  - Select, Move, Erase
  - Terrain
  - Objects
  - Plants
  - Paint, ground only, renamed **"Paint ground"**
- **Remove the bar's Build menu, or make it a menu that opens the right bar.**
- **One "World look" entry.** Sky and Environment become **"Time & weather"**. Styles
  becomes **"World style"**, with the style palette. A house's palette offers "Follow
  world style" first.

This needs your review: the standing rule says no regrouping until the inventory is
reviewed (`ui-inventory-before-renaming`). This audit is that review's input.

**Must keep working:** every tool; `desk.pick` by name for scripts and tours.
**Files:** `desk.gd` (`TOOLS`, `GROUPS`, `_edit_row`, `_build`), `buildbar.gd`.

### F6. Giving an ability silently creates a design; and maker-made things refuse abilities (R4)

**Evidence:** Observed (code `abilities.txt:27-30`, `inspect.gd:1924`; screenshot).

**User's goal:**
- make this cot follow me
- make this tree climbable

**Problem:**
- A new design name appears that the person did not choose (`cot_own_cot`).
- For a tree in a forest, the person is told to understand hand placement and use
  Ctrl+D first.

**Proposal:**
- **Interaction:**
  - Name the generated design in the person's words.
  - Say once, in the change message: "Made a copy of the Cot design for this one, so
    other cots don't change."
- **Small engineering:** for a maker's thing, offer "Take this tree out of the forest
  and give it…". Today that is two manual steps; this does them as one undo step
  (`host copy_one` exists).

**Files:** `abilities.gd`, `inspect.gd`.

### F7. Context row and status sentences (R1)

**Evidence:** Observed (code `desk.gd:543-559`, screenshots).

**Wording:**
- "Picked: one cot — it belongs to the arrangement of 40 things made by the maker
  'woods' (Forest); 12 of them are pines".
- "Nothing picked. Left click picks a thing. W A S D and right drag move the view,
  middle drag turns it, the wheel goes nearer and farther: nobody walks until
  Play (Tab)."

**Proposal (cosmetic):**
- "Selected: Pine (in Forest 'woods')".
- The navigation hint shows once, then lives under Keys.

**Files:** `desk.gd` (`_picked_words`, `says`).

### F8. Menus written as sentences (R5)

**Evidence:** Observed (code `desk.gd:576-589`, `_paint`, `_objects`).

**Proposal (cosmetic):**
- Short labels, in a standard menu form.
- A dimmed right-hand shortcut.
- Explanations go into the tooltip or a status line.

Examples:

| Now | Proposed |
|---|---|
| "Save the world (to X.json; what that file has is kept first)" | "Save" |
| "Autosave every 5 minutes while there are changes in no file (…)" | "Autosave ✓" |
| "Open an earlier save of this world… (3 kept)" | "Restore a backup…" |

### F9. The Character Studio's attack-then-dodge path

**Evidence:** Observed (screenshot `04-studio-movement.png`, code `studio.gd:130`).

The path works. The remaining problems are words:
- `cut_thrust`
- "Someone to strike 1.4 m, aside 0.0"
- "Ready to see"
- Phase labels "windup / cut / recover", which are fine
- The tab "Clips (list)"

**Proposal (cosmetic):**
- Display names for moves and combos.
- "Practice target: 1.4 m ahead".
- "Ready to see" becomes "Plays on this character" (✓ or ✗ with the reason).
- The combo's button shown inside the combo editor ("Played with: Attack, Attack"),
  so the person does not have to go to the Buttons tab.

### F10. Missing capabilities found while walking the tasks (engineering, not wording)

**Evidence:** Observed (code).

- **Snow cannot be painted on the ground.**
  - Terrain paint has rock, grass, sand, dirt, and "as generated" (`terrain.gd:52`).
  - Snow exists only as weather that lies on surfaces facing up (`lying.gd`,
    `weather.gd:42`). That is global and driven by weather.
- **A part of a design cannot spin.**
  - The windmill's sails are `part block` lines of one design (`library/rural.txt:525`).
  - `turns AXIS D` turns a whole loose thing (`kind.rs:667`, `things.gd:332`).
  - Joint motors exist (`loose.gd`).
  - I found no word that spins one part. `sway` bends parts in the wind; it does not
    spin them.
- **No door preset** in Make it usable (F3).

---

## 3. Confusing terms and proposed replacements

The user's own terms are marked ★. They were chosen by you on 2026-10-10/11: maker,
arrangement, thing, sort. They are proposals for your decision, not changes I would
make on my own.

| Term now | Where (verified) | Problem | Proposed | Notes |
|---|---|---|---|---|
| behaves as 'X' | Behaviours window, Inspector glosses | Describes the data line | "Behaviour: X" or the outcome ("Runs away") | |
| its own / as every X does | `rulesview.gd:368-370` | Instance-override jargon | "Only this one" / "All Xs" (radio) | Shown only when they differ |
| (a ranger) after a name | `rulesview.gd:371` | Design name repeated with no reason | Drop; show the design under Advanced | |
| design / kind / sort ★ | Everywhere | Three words for nearly one idea; "kind" is engine-only | "Type" for design/kind in the UI ("All rangers", "Ranger type") | Keep "design" if you prefer it; drop "kind" from the UI |
| put down by hand | Inspector, context row | Unclear opposite | "Placed by you" (opposite: "In forest 'woods'") | |
| maker ★ / arrangement ★ | Context row, World outline | Long sentences that use both | Name the maker by what it is: "Forest 'woods'", "Street 'high'" | You chose these; proposal only |
| picked / chosen | Everywhere | Two words for selection; "chosen" also means a list choice | "Selected" | |
| someone | Needs line, Inspector gloss | A raw trait word | "Moves on its own (a creature)" | |
| needs / lacks | Behaviours window | Fine idea, odd framing | "Needs: moves on its own. [Make it a creature]" | Warning only when missing |
| Parameters | Behaviours window | Programming word | "Settings" | |
| States and steps | Behaviours window | Accurate; heavy as the first view | "Steps (advanced)" | Kept as is inside Edit steps… |
| facts / conditions / goes by | Inspector, Signals | Engine logic words | "Switches" (a fact) and "Only when…" (a condition) | Signals → "Switches and triggers" |
| offers / acts | Inspector glosses, acts.txt | Engine words for interactions | "Can be used: search (E)" | |
| the walker | Everywhere | Internal name for the player | "The player" | Folk "walker" stays in texts |
| Works on: this one thing, not its design | Context row | Scope as a constant banner | Remove; scope shown only in the controls that have it | |
| HOW IT WORKS (capitals) | Inspector | Shouting; vague | "What it does" | |
| Its design's text (F6) / Its acts' text (F10) | Inspector buttons | File editors as primary actions | Under Advanced: "Edit as text…" | |
| Tree Generator / Plant Generator | Plants | Fine | Keep | |
| World outline (makers) | Window title | The bracket repeats a jargon word | "World outline" | |
| Objects in this world (designs) | Window title | Same | "Objects in this world" | |
| Object text and meshes (kinds) | Window title | Same | "Object text (advanced)" | |
| Sky / Environment / Styles / Look | Toolbar, windows, right bar | Four names for world appearance | "Time & weather", "World style", "Look" for a house's look only | |
| clear / let grow / Paint where… | Terrain, Paint menu | Three names for where plants may grow | "Plants allowed here / not here" (one brush with two modes) | Inventory 12 already flags this |
| Give this one / Give every X | Behaviours | Two equal buttons for one choice | One Apply, with an "Only this one / All Xs" radio | |
| Bring in from words… | Behaviours | Unclear | "Import from text…" | |
| Ready to see | Studio | Unclear | "Plays on this character" | |
| Someone to strike | Studio | Odd | "Practice target" | |
| Make my own from it | Styles | Fine, but makes "my_my_lookout" | "Customise (makes a copy)" with the name "Lookout (mine)" | |
| Ctrl+Z takes it back | Every change message | Long | "Undo: Ctrl+Z" | |

---

## 4. Before and after, task by task

For each task: today's steps (from the code), the decisions the person must make, and
the simplest workflow the existing engine could support. "Steps" counts clicks or
choices. It does not count reading.

### T1. Make an NPC run away from the player

**Today**, route A (Behaviours):
1. Click the NPC.
2. Inspector: scroll to the buttons under HOW IT WORKS. Click "Behaviour…".
3. The window opens on whatever behaviour was last shown.
4. Open the chooser. Pick `run_away`, or `skittish`, or `deer_ways`. Nothing says which
   one fits.
5. Read "Needs of whatever has it". If it is red, find out how to make the NPC a
   "someone".
6. Choose "Give this one" or "Give every ranger".
7. Read the sentence "Ranger (a ranger): behaves as 'run_away', its own."

**Today**, route B (Abilities): Inspector → What it can do → Give it… → "runs from you".
The effect is similar, through a different system. Route A then overrides route B
silently.

**Decisions the person must make:** which system; which of three behaviours; scope;
whether the NPC "can"; plus three numbers whose defaults are fine.

**After:**
1. Click the NPC.
2. Inspector → What it does → **Add… → "Runs away from the player"**.

Done: 2 steps, no required decisions. The defaults are used, and the numbers sit inline
to change. The scope radio defaults to "Only this one". If the NPC cannot move by
itself, the same click also makes it a creature, and says so.

### T2. A character performs an attack followed by a dodge

**Today:**
1. Characters → Character Studio.
2. Combos tab → New combo → name it → choose moves → order them → Save.
3. Buttons tab → assign the combo to a button.

**Decisions:** the name (an id), moves by id, the button in another tab.

**After:**
1. Studio → Combos → **New combo**.
2. Add "Slash", then add "Dodge". The preview plays at once.
3. **Played with:** [Attack] then [Attack] (inline).
4. Save.

The engine does not change. The button choice is shown inside the combo, and names are
display names.

### T3. Add a door that the player can open

**Today**, inferred from code and not tried by hand for this audit:
- A drawn house gets a door in its first side, or where clicked with Windows.
- A free-standing door object needs:
  - the `opens` word, through the design text (F6), which is not in Give it…;
  - for a lock, `key`, or the `door_lock` behaviour in Ctrl+B with its states.

**After:**
1. Place a door (Objects).
2. Inspector → What it does → Add… → **"Door: opens when used"**.
3. Options, inline:
   - Closes: after [3] s / never / when used again
   - Locked: no / needs [Key]

This is a new `door` preset over the existing lines (`opens`, `key`, `door_lock`
parameters). It is small engineering.

### T4. Build a house and move it without separating its parts

**Today:**
- Right bar → Build → House → drag.
- Then either drag its white cross, or right-drag any part.
- The left toolbar's **Move tool** on a house piece does not move it. It prints "A house
  goes as one: drag its white cross, or drag any part of it with the right button."
  (`desk.gd:495-496`).

**After:** Move on any part of a house moves the whole house. To move one piece, use
Alt+click (not offered by default).

The one change: the Move tool does what its message describes, in place of the message.

### T5. Paint a patch of snow onto the terrain

**Today:** not possible. Paint offers rock, grass, sand and dirt. Snow lies only from
weather, everywhere at once.

**After (engineering):** add "Snow" to Paint ground surfaces, rendered with the
lying-snow look already in `lying.gd`.

Workflow:
1. Paint ground.
2. Choose Snow.
3. Drag.

The decision for you: whether painted snow and weather snow should be one thing, with
weather adding to what is painted.

### T6. Make a windmill's blades rotate

**Today:** not possible on the library windmill, because the sails are parts of one
design. A workaround exists:
1. Model the sails as their own design (Objects → Modelling or New shape).
2. Place them on the tower.
3. Write `turns z 30` (Physics), or set up a hinge joint with a motor.

None of this is offered by name. `turns` is listed only as "elsewhere: the Inspector's
Physics".

**After:**
- **Small (presentation):** Inspector → What it does → Add… → **"Spins"**, with axis
  (shown as an arrow in the view) and speed (turns per minute). This writes `turns`.
- **Larger (engineering):** "Spin a part…". Click the sails in the view, and only that
  part turns. This needs a per-part spin in the host and in `things.gd`, for which I
  found no existing word.

### T7. Create a creature with different movement and behaviour

**Today:** the settings are spread over:
- the Inspector abilities (`creature`, `roams`, `hunts`…),
- the design text (`moves SET`, `body`, `weighs`),
- the Character Studio (moves),
- Behaviours (Ctrl+B).

Giving abilities silently makes a design such as `cot_own_cot`.

**After:** an Inspector **"Creature" card**:
- Moves like: Deer / Wolf / Bird / Your own…
- Speed, Size, Weight
- Behaviour picker (T1)
- "Edit moves…" opens the Studio

Creating a new type is said plainly, once: "Made a new type 'Swift deer' from Deer."

### T8. Change the look of a whole environment with a style or palette

**Today:**
1. Sky (toolbar) → Environment window → style row → Styles window.
2. Pick a style → "Give the world this style".
3. To change colours: "Make my own from it" (names it `my_lookout`) → palette swatches.
4. Separately, a house's palettes live in the right bar's Look.

**After:**
1. **World style** (one entry) → a gallery with a thumbnail per style.
2. Hover to preview, click to apply.
3. Optionally, **Customise colours…**: a copy is made automatically, named
   "Lookout (mine)".

Houses default to "Follow world style".

---

## 5. Which changes are which

| Change | Cosmetic (wording) | Interaction redesign | Engineering |
|---|---|---|---|
| Display names in place of ids; no file names by default (F2) | ✓ | | small: `title` lines, humanise |
| Status lines and context row in plain words (F7, R1) | ✓ | | |
| Menu entries shortened (F8) | ✓ | | |
| Studio wording (F9) | ✓ | button shown in the combo | |
| Behaviour card in the Inspector; window opens on authoring (F1) | | ✓ | |
| Scope as a default-off radio (R4) | | ✓ | |
| Inspector order and Advanced fold (F4) | | ✓ | |
| One "What it does" list across abilities, behaviours and uses (F3) | | ✓ | small: one catalogue view over three files; conflict prompt |
| One Build, one Paint ground, one Erase; World style naming (F5) | | ✓ (needs your review) | |
| Move tool moves a whole house (T4) | | ✓ | small |
| Generated design named in the person's words; take a tree out of a forest in one step (F6) | ✓ | | small |
| Door preset (T3) | | | small: `uses.txt` + `abilities.gd` |
| Spins ability (T6) | | | small: maps to `turns` |
| Spin one part (T6) | | | **larger**: host + `things.gd` |
| Paint snow (T5) | | | **medium**: host surface + shader |

Nothing in this table changes:
- the world file format,
- `behaviours.txt` / `abilities.txt` / `uses.txt` grammar (only additions),
- the host protocol,
- any working system's behaviour.

---

## 6. The Draw3D interface standard (proposed)

These rules apply to every screen, new or changed. The examples are from this audit.

1. **Name the outcome, not the representation.**
   - "Runs away from the player", not "behaves as 'run_away'".
   - "Door: opens when used", not "offers open".
2. **Labels are 1–4 words.**
   - Buttons are verbs: Apply, Add, Remove, Edit steps…
   - Headings are nouns: Behaviour, Look, Placement.
   - Sentences belong in status messages and tooltips, and only where needed.
3. **One word per concept, everywhere.** A glossary (section 3) is kept with
   `Draw3D_UI_Inventory.md`. Selection is always "Selected". The player is always "the
   player".
4. **No identifiers, file names or engine notation by default.**
   - Display names come from `called` / `title`, or a humanised id.
   - Ids, `.txt` files, function keys, raw lines and `kind#it` appear under
     **Advanced** only.
5. **The common case needs no decisions.**
   - Every action has defaults that produce a sensible result at once.
   - Numbers are shown inline after the fact, to tune, not asked first.
6. **Scope defaults to "this one".** "All of this type" is a secondary choice, shown in
   the control that applies, never as a banner.
7. **Progressive disclosure, in three levels:**
   - **Default:** the outcome and its 1–3 key settings.
   - **More:** every setting of that outcome.
   - **Advanced:** text, ids, conditions, internal structure.

   Folds remember their state per person. Nothing a click can reach today is removed.
8. **One place per job.**
   - A tool or window has one entry point with one name.
   - Other places may link to it ("Edit steps…"), but they never offer a different
     copy of it.
9. **One mechanism per outcome, as the person sees it.** Where the engine has several,
   one list presents them by outcome, and the editor handles conflicts by asking:
   "Replace X with Y?"
10. **Do what the control says.** A tool that cannot act on the selection either acts
    on the sensible thing (the whole house) or is disabled with a short reason. It
    never answers with instructions to go elsewhere.
11. **Feedback is short and consistent.**
    - Form: "Ranger now runs away. (Undo: Ctrl+Z)".
    - Errors say what is missing and offer the fix as a button: "Needs to move on its
      own. [Make it a creature]".
12. **Show before committing, where it is cheap.**
    - Hover-preview styles.
    - The Studio previews moves.
    - The view tints what a brush will touch (already done for tiles).
13. **Keep the house voice for content, not controls.** The descriptive voice can stay
    in `about` texts, library descriptions and the checklist. It does not belong on
    buttons, headings, field labels or status lines.

---

## 7. Staged implementation plan

Each stage is small, independent and testable. None restarts finished work or changes
architecture. Each stage ends with its acceptance check (section 8), a PROGRESS entry,
an inventory update and a checklist rewrite, as the standing rules require.

**Stage U0. Decisions only (you).** Nothing is built until you decide:
- **(a) Your terms.** Maker / arrangement / thing / sort: keep, or replace as section 3
  proposes. And "design" vs "type".
- **(b) Consolidation.** May the toolbars be merged as F5 proposes?
- **(c) Paint snow and weather snow.** One thing, or two?
- **(d) Per-part spin.** Is it worth its larger cost now?

**Stage U1. Words.** Cosmetic, highest impact for the least risk.
- Display names for behaviours, moves, combos, styles and designs. Ids and file names
  move behind Advanced.
- Rewrite every status/toast line, the context row, the Behaviours "For" line and the
  menu entries to standard rules 2, 3 and 11.
- Add a **vocabulary check** to the headless test set. It scans visible labels for:
  - `.txt`
  - underscores in shown names
  - "its own"
  - "kind#"
  - "behaves as"
  - "F6" / "F10" inside labels

**Stage U2. The Behaviour card and the Inspector order.** Interaction.
- The Inspector gains its header summary, "What it does", Look, Placement, and
  Advanced (F4).
- The behaviour picker with inline settings and a scope radio (F1).
- The Behaviours window opens on authoring, its library actions in one menu.

**Stage U3. One "What it does" list.** Interaction, with small engineering.
- Abilities, behaviours and use presets shown together, by outcome.
- A conflict prompt.
- Generated designs named in plain words.
- "Take out of the forest and give…" as one step (F3, F6).

**Stage U4. Task presets.** Small engineering.
- The Door preset (T3).
- The Spins ability (T6, whole thing).
- The Creature card (T7).
- The Move tool moving a whole house (T4).
- Combo buttons shown in the combo editor (T2).

**Stage U5. One place per job.** Interaction, only after U0(b).
- One Build (the right bar), Paint ground, one Erase.
- Time & weather, and World style.

**Stage U6. Capability gaps.** Engineering, only after U0(c, d).
- Paint snow (T5).
- Spin one part (T6).

U1 can start as soon as you agree, because it changes no behaviour. U2 and U3 are where
the ranger problem is actually solved.

---

## 8. Acceptance criteria

### Functional (every stage)
- The full cargo test suite passes (`--test-threads=1`). GDScript compiles headless.
- These demos still run to the end, with the same outcomes in their screenshots:
  - `-Demo behave`, `-Demo rules1`, `-Demo wired`, `-Demo held`
  - `-Demo abilities`, `-Demo usable`, `-Demo studio`, `-Demo styles`
  - the construction demos
- Every world in `editor/` loads and saves byte-identical texts when nothing is
  changed.
- Every `.txt` file under `editor/godot` and `editor/library` parses with no new
  problems.
- Each change is still one undo step, and is undone exactly.
- No capability reachable from a click today becomes unreachable. A script lists every
  control id before and after a stage. A removed id must be named in the stage's notes
  with where its function went.

### Usability (checked by you, by hand, per stage)
- **Task scripts.** Each of T1–T8 that the stage touches is done by you from a fresh
  world, with no help. For each one, record:
  - the steps taken,
  - any word you did not understand,
  - any place you hesitated about where to go.

  Targets:
  - T1 in at most 3 steps;
  - T3 and T6 (whole thing) in at most 4;
  - none needs a text editor, a file name or an id.
- **Say it back.** Before changing a setting, you can say what it will do from its label
  alone. Every label you could not is a defect.
- **The ranger sentence.** After U2, selecting the ranger shows wording you would accept
  without explanation. "Ranger: Runs away from the player" is the target.
- **Vocabulary check passes.** The U1 headless check finds no banned pattern outside
  Advanced sections and text editors.
- **No silent overrides.** Giving a second movement outcome to the same thing always
  shows the conflict prompt (a headless test plus your check).
- **Scope safety.** With the defaults, no change to one thing alters another. Checked
  headlessly by comparing every other thing's lines before and after.

---

## 9. What this audit did not cover

- Window sizing and panel layout. These are covered by `Draw3D_UI_Overhaul_Proposal.md`.
- Play-mode windows (Playtest, Physics, Journal, Who can do what), apart from their
  names. Of these, only the Behaviours window's Running section was read.
- Modelling, Sounds, Game display (HUD), Signals' internals, Tiles and Layout. Their
  window titles were read (`main.gd:973-1035`), but their contents were not walked
  task by task.
- Gamepad use of the editor.
- Any by-hand trial. The task walks are from code and demo screenshots. T3's door path
  is the least certain, and is marked as inferred.
