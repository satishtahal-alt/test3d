# Emberwatch — how it was authored in the editor

A cumulative record of the interface steps used to make *Emberwatch: The Last Beacon*. Each
workflow is written down the first time it was used; after that it is only named.

**How the steps were performed.** Every step below was done through the editor's own
interface by `editor/godot/author.gd`. It drives the real editor: clicks sent as the mouse
sends them, keys as the keyboard sends them, and drags in the view. It never calls the
engine host and never writes the world itself. The script of *what* it does is
`author_emberwatch.gd`. The words it types are in `author_emberwatch_words.gd`. The record
of *what it actually did* is `editor/godot/author-emberwatch.log`, rewritten on every run,
and this document follows that log. To watch it work, run `editor\run.ps1 -Author
emberwatch` without `-Check` (it uses its own window and never takes your mouse).

One step is not interface work: placing the camera. A person flies the view with W A S D,
a middle drag and the wheel. The script sets where the view looks outright, because the
view changes nothing in the world.

**Labels.** Each workflow is marked:

- **[existing]** — the editor already did this;
- **[exposed]** — the capability existed, and this work made it reachable or usable from
  the interface;
- **[new]** — new engine functionality, built for this game and offered to every world.

---

## 1. A world and its file

**1.1 Start a new world** [existing]
`World › New world…` → the dropdown at the top → `An empty island` → `hills` field (3) →
`size` field (8 patches = 512 m) → `Make the new world…` → `Yes: replace the world in the
editor`. World outline and Quick place open by themselves.

**1.2 Give it a file** [existing]
`World › Save as…` → in the dialog's `File:` box type `emberwatch.json` → Enter → if the file
is there already, `OK` to write over it. From then on `World › Save` writes it.

**1.3 Open it again** [existing]
`World › Open…` → if asked about changes in no file, `Leave without saving` → type the file
name → Enter.
*Friction:* pressing Enter the instant the name is typed sometimes did nothing. Give the
dialog a moment.
*Found:* an editor started by a script never opens the `-World` file it is given. The
world must be opened through this menu.

**1.4 Clear the screen** [existing]
Each window's `x` puts it away. `Window › …` brings it back.

## 2. The land

**2.1 The Terrain window** [existing]
`Toolbar › Terrain › Terrain tools…` → the window is taller than a 720-pixel screen: press
`Collapse all`, then open `Tool`, `Its numbers` and `Under the pointer`.

**2.2 Level ground** [existing]
`flatten` → `radius, m` (42) → `how much` (1) → in the view, a short drag beginning at the
height you want.

**2.3 Raise a ridge or mountains** [existing]
`raise` → `radius, m` → `m each dab` → drag along the ridge line. Repeat, and between
strokes point at the top and read **Under the pointer** ("the land is 20.4 m over the
water"). A 24 m brush at 0.6 m a dab adds about 2 m a stroke: the ridge took 7 strokes. The
mountains took 9 strokes each with a 46 m brush at 0.7.
*Friction:* there is no "raise to height H". You stroke and read the height back.
Mountains raised near the coast push the island outward: keep them inside the shore.

**2.4 Level a hilltop** [existing]
`flatten` → small radius → a drag begun on the top.

**2.5 Make a slope walkable** [existing]
`smooth` → `radius, m` (9) → `how much` (0.6) → drag along the path several times.
*Found by playing:* the levelled top left a rim too steep to walk up. Nothing in the editor
shows a slope as too steep.

**2.6 A cliff to climb** [existing]
`cliff` → `high, m` (3.5) → `back, m` (10) → `turned, °` (240) → one click on the land.
*Found by playing:* the first cliff stood across the ridge path and blocked it. It was moved to the west flank.

**2.7 Paint trails** [existing]
`paint` → the `made` dropdown → `dirt` → `radius, m` (2), `soft rim, m` (1) → drag along each
trail. The storm road used radius 3.

## 3. The hamlet

**3.1 Draw a house and name it** [existing; naming made useful — see 9.4]
`Construction bar (right) › Build › House` → drag corner to corner on the land → `Esc` (the
house stays selected) → `Window › World outline` → `name` field → type (`the hall`) →
Enter → the window's `x`. Done for the hall, smithy, healer's hut, lodge, cottage, store
and farmhouse.
*Friction:* the house's door goes "in the first side", and you can't choose that side
while dragging.

**3.2 A palisade with gaps** [existing]
`Construction bar › Build › Fence` → drag along an arc → `Esc`. Three arcs leave three gaps
(north, east, west).

**3.3 A gate where a path crosses a fence** [existing]
`Construction bar › Paths › Path` → drag from outside, through the fence, into the hamlet
→ `Esc`. The fence gets a gate, which a player opens with E.

**3.4 A square, named as a place** [existing; named place new — see 9.4]
`Construction bar › Paths › Paving` → drag a small loop → `Esc` → name it in World outline
(`the square`).

## 4. The game's rules, as text

**4.1 Texts kept in this world only** [exposed: new checkbox]
`Characters › Moves, poses and uses (as text)…` opens the **Texts** window (F10). **Untick
`also the project's file`** before applying anything that belongs to this game only.
Otherwise every new world would begin with it.

**4.2 Add to a text** [existing]
In the Texts window: choose the text in the list on the left (`goods`, `acts`, `recipes`,
`stats`, `quests`, `talks`, `endings`) → click in the text → Ctrl+End → type the new entries
→ `Apply (Ctrl+Enter)` → read the red lines under it (nothing was reported wrong). For
`stats`, the whole text was replaced (Ctrl+A, then type).
*Friction:* quests, talks, acts, goods, recipes and stats have no form editor. They are
typed, and checked only when applied.

**4.3 Endings chosen by what happened** [new]
The `endings` text (new): `ending NAME`, `called` (title), `when CONDITION`, `says`
(paragraphs; `{fact}` shows a world fact's number). The first ending whose `when` holds is
shown when the Player start's `the game ends when` condition comes true.

## 5. What things are

**5.1 Bring a library design into the world** [existing]
`Toolbar › Objects › Asset Library…` → the `Search` field: type a word (`wolf`) → click the
design in the picture grid → `Bring into this world`.
*Needed because:* a kind written `like wolf` is refused ("there is no such kind") until the
library's wolf is in the world.

**5.2 A behaviour from words** [existing]
`Window › Behaviours` (scroll the Window menu down to it) → the `Behaviour ▾` button →
`Import from text…` → type the behaviour → `Check and import` → read the plain-words
summary that appears ("While it is Patrol… When it hears anything: changes to
Investigate…").

**5.3 A kind of this world** [existing]
`Toolbar › Objects › Object text (advanced)…` → click in the text → Ctrl+A → type the kind
(`kind mara like villager` …) → `Apply`. The name comes from the text's first line. The red
lines under it say what was not understood.
*Found by playing:* raiders that wait in a far-off den were let go by the world's regions (anything more than a region from the player stops), so the raid never came. The kind line `restless` (new, 9.8) keeps them going. In the Inspector it is the row `far off too` under *What it sees and hears*.

## 6. Putting things in the world

**6.1 Put down a kind of this world** [existing; fixed — see 9.5]
First press `Deselect` on the bar if anything is selected. A selected path or paving shows its settings panel over the view, and that panel takes the clicks meant for the land. Then: Object text → click the kind in the list on the left → `Put one down` (the window closes) →
**check that the row under the bar says `Placing: NAME.`** → click on the land, once for
each → `Esc`.

**6.1a Turn one thing that is down** [existing]
Look at it from the side (from above, thin boards are too narrow to click) → click it → the
Inspector opens on it, headed with its kind's name (`Gap north`, not what it is called,
*the north gate*) → scroll down to `>  Placement` and press it → `turned (°)` → type the
facing → Enter. `[` and `]` turn it 15° a press as well.
*Found by playing:* `Put one down` turns each thing a little differently as it is put down
(120°, 240°…). The gates stood askew across their gaps, and the wolves walked round their ends
into the hamlet. Each gate had to be turned to lie along the palisade.
*Friction:* the turn is folded away under Placement, below the thing's whole description; and
the Inspector names a thing by its kind, so a person looking for "the mill gate" reads "Gap mill".

**6.2 Put down a library design** [existing]
Asset Library → search, choose → `Place one…` → fold the window with its `-` (it covers the
view) → click on the land for each → `Esc` → `+` to unfold.

## 7. The valley's places

**7.1 A forest** [existing]
`Toolbar › Plants › Add a forest: click a place, or drag out its size…` → drag from its
middle out to its edge → `Esc` → name it in World outline.

**7.2 A field of crops** [existing]
`Construction bar › Brushes › Crops` → drag rows → `Esc`. The same way: `Waterside` (reeds),
`Flowers`, `Trees`.

**7.3 A pond** [existing]
`Construction bar › Water › Pond` → click places round it → click beside the first →
Enter → `Esc`.

**7.4 The world's look** [existing]
`Toolbar › Style` → in the Styles list, `Golden valley` → `Give the world this style`.

## 8. The game around it

**8.1 Where a game starts** [existing]
`Characters › Place the game start…` (or `Move the game start…` once there is one) → click
on the land.

**8.2 The start's settings** [existing; one new field]
`Characters › Game start in the Inspector` → `facing, degrees` → `a game begins at` (16.5)
→ **`an hour of play lasts`** (2.2 minutes, new) → `the game ends when` (`fact dawn`) →
`the ending says` (used only if no ending in the `endings` text holds). The fields are below
the Inspector's fold: scroll down.

**8.3 Play, save, load** [existing]
`Play (Tab)`. While playing, F5 saves the game and F9 loads it.

## 9. What had to change in the editor, and why

| # | What could not be done | Where it is now | Label |
|---|---|---|---|
| 9.1 | A game hour always lasted one minute, so an evening-to-dawn night took 13 minutes whatever the game needed | Player start: `an hour of play lasts (minutes)`, in the Inspector and World outline. While playing it is the world fact `hour_lasts`, so a behaviour can change the pace (`put hour_lasts 1.1` at midnight), and a saved game keeps it | new (`clock.gd`, `gameui.gd`, `makers.gd`, host `land.rs`, `lib.rs`) |
| 9.2 | One ending only | The `endings` text, chosen by condition (4.3) | new (`endbook.gd`, `endings.txt`, `gameui.gd`) |
| 9.3 | Wolves sent somewhere by a behaviour (`hurry the north gate`) walked up to a barricade and stopped. Only hunting or searching creatures broke barricades down | A creature with a blow (`attacks`) on its way somewhere by `go` or `hurry` breaks down a barricade in its way, and its `go` waits while it does | new (`fight.gd`, `rules.gd`) |
| 9.4 | `goes 22 at the square` and `go the farmhouse` found no place: only structures built on grid cells were places. Houses and squares drawn on the land had none | A drawn structure is a place: the middle of what was drawn, and (where it goes round) its inside | exposed (host `lib.rs`) |
| 9.5 | `Put one down` could put down a *different* kind from the one chosen. The editor armed by position in a list that changed before the click | Kinds are armed by name | fixed (`main.gd` `armed_kind`) |
| 9.6 | Applying a text in the Texts window always overwrote the project's shared file too (Emberwatch's stats replaced every new world's stats) | The checkbox `also the project's file`; unticked, the text is this world's alone | exposed (`textview.gd`) |
| 9.7 | Nothing could author a whole world through the interface for checking | `editor\run.ps1 -Author NAME` runs `author.gd` with `author_NAME.gd` and logs each step | new (tooling) |
| 9.8 | Wolves waiting in a den far from the hamlet stopped: the regions (PW1) let them go, or left them standing beyond 75 m, until the player came near; and those in a far region at the start were sometimes never made. A raid sent by their behaviour never came, or came short | The kind line `restless` (Inspector, under *What it sees and hears*: `far off too`): never let go, never left standing, made wherever it is | new (host `kind.rs`, `things.gd`, `folk.gd`, `inspect.gd`) |
