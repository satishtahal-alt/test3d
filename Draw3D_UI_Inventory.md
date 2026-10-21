# Draw3D — inventory of the editor's interface

Written 2026-10-11 and kept up since (last: 2026-10-14, mechanisms: 8.5 and 13za), from the code as it stands on branch `m2-stable-terrain` after the
consolidated pass of that day, and **brought up to date the same day after the
capability-exposure pass** (sections 13b and 14 say what that pass added and what the
engine can do that the editor still does not reach). It is a reference to review,
rename and regroup from. **On 2026-10-13, at the user's asking, the bar was regrouped
and the window titles and menu names were made plain** (section 13r says every one).
Sections 1 to 7 give the names on the screen now; sections 8.1 to 8.12 describe the
windows by their earlier titles, which 13r maps to the new ones.

**How to read it.** Each entry gives the exact name, where it is, what it really does
(read from the script that does it, named in brackets), what it changes and how far,
what it needs first, and what it overlaps or hides. *Scope* is the important column:
**view** (changes nothing in the world), **one thing**, **one maker and everything it
made**, **a design and every thing of it**, **the world**, **this machine** (a
preference, never saved with a world), or **the game being played**.

**How sure it is.** Everything here was read from the scripts in `editor/godot`
(`desk.gd`, `main.gd`, `makers.gd`, `designs.gd`, `marks.gd`, `handles.gd`,
`terrain.gd`, `palette.gd`, `inspect.gd`, `kindview.gd`, `textview.gd`, `hubview.gd`,
`feel.gd`, `newworld.gd`, `checklist.gd`, `tint.gd`, `controls.gd`). Where a control's
behaviour was taken from its label, its tooltip or its script's own description
rather than traced through the code that runs, the entry says **(not traced)**. What
scripted checks exercise is noted in `PROGRESS.md`, not here. Nothing here was tried
by hand.

**The words the editor uses** (as the user fixed them, 2026-10-11): a **thing** is one
object; its **sort** is what kind it is (a rock); a **design** is what a thing is,
apart from where it stands (rock no. 2, or a kind of your own); a **maker** is what
creates things and controls their making (a wood); its **arrangement** is the things
it made. The code and some labels also say **kind** (a design written as text in this
world), **mesh** (a shape of corners and faces a kind is made of), **node** and
**generator** (a maker, in the Inspector), **design/kind/sort** somewhat
interchangeably: see section 12.

---

## 1. The layout

*Rewritten 2026-10-13 for the workflow pass (amendment 7ab, section 13r below): the
bar's headings, the toolbar and the row under the bar are new.*

- **The bar** across the top (`desk.gd`). Two rows while editing: the menus and
  buttons; under them the *context row*, which says the tool in hand, what is
  picked and what it works on, and has buttons for it. While playing, a different
  first row and no second.
- **The toolbar** down the left side, while editing (`desk.gd`, pictures drawn by
  `icon.gd`): nine buttons, a picture and a word each, for what a person building
  a world does. The one in use is lit. It is not shown while playing.
- **Windows**: every panel is a floating window. Drag its title bar to move, its
  corner to resize, **-** folds it, **x** puts it away. The **Window** menu lists
  them all. Where each is, its size and whether it is folded are kept on this
  machine (`user://layout.cfg`). They do not dock; they keep off the toolbar.
- **The view**: the world, behind everything. The left button picks, or does what
  the tool in hand does.
- **Notices**: a box at the bottom middle says what just happened; it fades.
- **Going to Play** puts the editing windows and the toolbar away and brings them
  back afterwards.

---

## 2. The bar while editing

| Name | What it does | Scope | Notes |
|---|---|---|---|
| **World: FILE \* v** | menu, section 3. The heading says which world is open and shows **\*** while it has changes in no file | | In a window narrower than about 950 px the name is left out |
| **Edit v** | menu, section 4: undo, redo, and what is done to what is picked | | New 2026-10-13 |
| **View v** | menu, section 5: what is shown, and this machine's settings | | New 2026-10-13 |
| **Characters v** | menu, section 6 | | |
| **Window v** | menu, section 7 | | |
| **Undo** / **Redo** | Back, forward one step (Ctrl+Z, Ctrl+Y) | the world | Also in Edit, and in Quick place |
| **Play (Tab)** / **Play as NAME (Tab)** | Starts walking in the world from the green flag, or from where the view looks | the game being played | |
| **Checklist** | Opens the list of what to try by hand | this machine | |
| **Keys** | Opens the window of every key and mouse action | view | |

### The toolbar (left side)

| Button | Kind | What it does | Scope |
|---|---|---|---|
| **Select** | tool | Puts away whatever tool was in hand (a brush, the eraser, a land tool, something about to be placed, a shape's corners): the left button picks again. What is picked stays picked | view |
| **Move** | tool | With one thing picked: the next click says where it goes (the same as the row's Move). With a forest, meadow or street selected: the next click moves the whole of it. With neither: says to pick something first | one thing, or one maker and everything it made |
| **Paint ›** | list | Section 2a | |
| **Erase** | tool + window | Opens *Erase objects* (8.13) and takes the eraser up; pressed again, puts it away | the world, under the eraser |
| **Terrain ›** | list | Section 2a | |
| **Objects ›** | list | Section 2a | |
| **Plants ›** | list | Section 2a (the lists are kept under the heading *Vegetation*) | |
| **Build ›** | list | Section 2a (*Structures*) | |
| **Sky** | window | Opens *Environment* (8.14): time of day, weather, wind, style | |

Which is lit (`desk.gd` `tool_now`): Erase while the eraser is in hand; Paint while
a brush is (objects, a maker's area, or the land's paint); Move while one thing or a
maker is being moved; Terrain while a land tool other than paint is chosen or tiles
are being put down; Plants, Build or Objects while a forest, a road or house, or an
object is being placed; Select otherwise. A button whose list is open, and Sky
while its window is, are lit too.

### 2a. The toolbar's lists

**Paint**

| Entry | What it does | Scope |
|---|---|---|
| *Paint the ground* | heading | |
| **Ground: grass, sand, rock or dirt under the brush** (F8) | Opens Terrain with its paint tool chosen | the land's layer, or a tile's own surface (8.4) |
| *Paint objects: many at once, under the brush* | heading | |
| **Paint with 'X' (the object chosen in Objects)** | Only when an object is chosen there: opens Objects on it with its brush in hand | adds things, a maker "painted X" |
| **Paint with the group 'NAME' (its objects)** | One for each group of this world: opens Objects on the group with its brush in hand | adds things, a maker "painted NAME" |
| **Choose an object or a group to paint with… (Objects)** | Opens Objects, and says what to press there | view |
| *Paint where plants grow* | heading | |
| **Where one forest or meadow grows: select it in World outline, then 'Paint where…'** (Ctrl+M) | Opens World outline | view |
| **Where nothing grows, for every forest and meadow (Terrain: clear, let grow)** | Opens Terrain with "clear" chosen | the land's layer |
| *To take away what was painted: Erase, on the toolbar* | heading | |

**Terrain**

| Entry | What it does |
|---|---|
| **Terrain tools: raise, lower, smooth, flatten, cliffs; paint the ground** (F8) | Opens Terrain (8.4) |
| **Add land (an island)…** | Adds an island. Greyed if the world has one |
| **Grid layout: square or hex cells to build on, and snap to** (Ctrl+G) | Opens Grid layout (8.2c) |
| **Terrain tiles: ready-made pieces of land for a grid's cells** (Ctrl+T) | Opens Terrain tiles (8.2d) |

**Objects**

| Entry | What it does |
|---|---|
| *Picked: …* | heading: what is picked, in words |
| **Asset Library: objects with pictures, kept for every world** (Ctrl+L) | Opens the library (8.2b) |
| **Objects in this world: place one, paint many, make a group for a brush** | Opens Objects (8.2) |
| **Quick place: common props by a click, and nudging what is picked** | Opens Quick place and nudge (8.3) |
| **Animals: horses, cows, goats (Asset Library)…** | Opens the Asset Library on its Farm animals (since 2026-10-14) |
| **World outline: every forest, meadow, street and painted group; change or remove one** (Ctrl+M) | Opens World outline (8.1) |
| **Inspector: the numbers of what is picked, and of what made it** (F7) | Opens the Inspector (8.5) |
| **Object text and meshes: how an object is written** (F6), under *Advanced* | Opens 8.6 |

**Plants** (Vegetation)

| Entry | What it does | Scope |
|---|---|---|
| **Tree Generator: make a tree from a few numbers, and keep it as an object** | Opens the Tree Generator (8.2e) | |
| **Add a forest: click a place, or drag out its size…** | A click adds a wood reaching 40 m; press and drag sets the reach | the world |
| **Add a meadow: grass, flowers, bushes, rocks…** | The same, reaching 30 m | the world |
| **How many trees, which, how near the road…** | Opens the Inspector on the maker named "woods" | view |
| **How much grass, flowers, bushes, rocks…** | The same for "meadow" | view |
| **Change a forest or a meadow that is there: select it in World outline** (Ctrl+M) | Opens World outline | view |
| *To paint trees or flowers by hand: Paint, on the toolbar* | heading | |

**Build** (Structures)

| Entry | What it does | Scope |
|---|---|---|
| **Draw a road: click its places, Enter ends it…** | Click places on the land; Enter ends it | the world |
| **Put down a house…** | A click puts one house down by itself | the world |
| **Add a street of houses along a road…** | Draws a line if none is selected in World outline, then adds houses along it. A road that has a street along it already is not used again: a new line is drawn (since 2026-10-14). What it made is said | the world |
| *Build on a grid's cells* | heading (new 2026-10-14, 7ad) | |
| **A wall, fence or hedge round cells: click a cell of a layout, then more…** | A click on a cell of a layout puts a structure down round that cell; from then on a click on a cell adds it, again takes it away; Esc ends | the world |
| **A house: walls with windows, a door, a roof; click its cells…** | The same, a structure made as a building with a roof of tiles and boards on its floor from the first click (since 2026-10-14, 7af) | the world |
| **A fence round cells: a pen, a paddock…** | The same, made as a fence | the world |
| **A hedge round cells: a garden…** | The same, made as a hedge with garden soil on its cells | the world |
| **A path, yard or square: paving on cells, steps where it is high…** | The same, made as nothing round its cells and paving on them (until 7af this row opened World outline) | the world |
| **A pond: a kerb round cells, water in them…** | The same, made as a pond | the world |
| **Walls inside the structure chosen: rooms, pens…** | A click near a side between two of its cells puts a wall there, or takes it away; Esc ends. Off until a structure is chosen | one structure |
| **Open or close gates and doors in the structure chosen…** | A click near a side of its rim, or a wall inside, makes a gate (a door in a building) there, or takes it away; Esc ends. Off until a structure is chosen | one structure |
| **Grid layout: the cells to build on** | Opens the Layout window (also under Terrain) | view |
| *Barns, sheds, fences, walls, bridges: Objects > Asset Library* | heading | |
| *Draw on the land* | heading, and nine entries (2026-10-17, 7bu; 2026-10-18, 7ck, 7cl): see 13zy, 13zz14, 13zz15 | |

### The context row (second row)

First a word on a light plate: the tool in hand (**SELECT**, **MOVE**, **PAINT**,
**ERASE**, **TERRAIN**, **PLACE**, **BUILD**, **PLANT**, **EDIT SHAPE**, **BUSY**).
Then one of: what the engine is working on; what is being shaped; the maker tool
in use; the land tool in use; what is being placed; what is picked; the maker
selected; or "Nothing picked…". It is one line: what is said gives way to the
buttons and ends in "…", and is whole when the pointer rests on it.

| Button | Appears when | What it does | Scope |
|---|---|---|---|
| **Stop it (Esc)** | the engine is working | Stops the work; nothing of it is kept | the world |
| **Its text (F6)** | a mesh is being shaped | Opens Object text and meshes on that mesh | view |
| **Done (Esc)** | a mesh is being shaped | Puts the corners away; what was dragged stays | view |
| **No tool (Esc)** | a land tool is chosen | The left button picks again | view |
| **End the line (Enter)** | a line is being drawn | The places clicked become a line (a road) | the world |
| **Cancel (Esc)** | any maker tool is in use (placing, painting, erasing, adding, duplicating, moving) | Ends the tool | view |
| **Just pick (Esc)** | a thing is armed to be placed | The left button picks again | view |
| **Move** | one thing is picked | A click says where it is to stand | one thing |
| **Duplicate (Ctrl+D)** | one thing is picked | A frame follows the pointer; a click makes another, standing by itself | adds one thing |
| **Delete** | one thing is picked | Takes away that one thing; its maker keeps it away and can put it back (the Delete key) | one thing |
| **Edit shape** | one thing is picked | Opens the Inspector and takes hold of the thing's corners | a design and every thing of it, or one thing: the strip says which |
| **Tree Generator** | the thing picked is a tree made there, and that window is shut | Opens the Tree Generator with its numbers | view |
| **World outline (Ctrl+M)** | a maker is selected, World outline is shut | Opens it on that maker | view |
| **Inspect (F7)** | something picked or a maker selected, and the Inspector is shut | Opens the Inspector | view |
| **Deselect** | anything is picked or selected | Nothing is chosen | view |
| **More v** | any of the following applies | A list of them | |
| … *Edit its text (advanced, F6)* | the thing picked is of a kind of yours | Opens its text | view |
| … *Play as this* | it is of a kind with a person's bones | The walker becomes that kind, and play starts beside it | the world |
| … *Show its maker in World outline* | one thing is picked, World outline is shut | Opens it on its maker; the thing stays picked | view |
| … *Select its maker* | one thing of several is picked | The maker becomes what is selected instead | view |
| … *Look at it* / *Look at it (Home)* | a maker is selected / one thing is picked | The view goes to it | view |
| … *Show it alone (/)* | one thing is picked | Only it is drawn, with the land | view |
| **Show everything (/ or Esc)** | some of the world is not drawn | Draws all of it again | view |
| **Layout (Ctrl+G)**, **Stop snapping** | a layout is snapped to | As before | |
| **Stop putting down (Esc)**, **Put it down anyway**, **Tiles (Ctrl+T)** | tiles are being put down or a whole tile is picked | As before | |

---

## 3. World menu

| Entry | Key | What it does | Scope |
|---|---|---|---|
| *The world open: FILE (in FOLDER)* | | A heading | |
| **New world…** | Ctrl+N | Opens the New world window (8.11) | |
| **Open a world…** | Ctrl+O | A file dialog; asks first if there are unsaved changes | the world |
| **Open again: FILE (in FOLDER)** | | Up to four of the worlds opened or saved here before | the world |
| **Open an earlier save of this world… (N kept)** | | The folder `backups` beside the world's file; the one chosen opens with no file | the world |
| **Save the world (to FILE; what that file has is kept first)** | Ctrl+S | Keeps the file as it is in `backups`, then writes | the world's file |
| **Save the world as…** | Ctrl+Shift+S | Asks for a file; that becomes the world's file | the world's file |
| **Show the folder the world's file is in** | | Opens it in the machine's file manager | view |
| *N changes not in any file* | | A heading | |
| *(F5 and F9, while playing, save and load a GAME…)* | | A heading | |

Undo, Redo and "What needs a look" moved to **Edit**; World outline (Makers) to
**Objects** on the toolbar; Engine status to **View**. Which world the editor starts
on, and F5/F9, are as before.

---

## 4. Edit menu (new 2026-10-13)

| Entry | Key | What it does | Scope |
|---|---|---|---|
| **Undo** / **Redo** | Ctrl+Z / Ctrl+Y | As on the bar. Greyed, with the reason, when there is none | the world |
| *Picked: …* | | A heading | |
| **Duplicate the one picked…** | Ctrl+D | As the row's | adds one thing |
| **Move the one picked…** | | As the row's | one thing |
| **Edit the shape of what is picked** | | As the row's "Edit shape" | a design, or one thing |
| **Delete the one thing picked (its maker stays)** | Delete | As the row's | one thing |
| **Deselect** | | Nothing is chosen | view |
| *Advanced* | | A heading | |
| **Detach what is picked from what made it (see what it says)** | X | Cuts the thing loose from its maker | one thing |
| **What needs a look: changes by hand that lost their place** | I | Lists them | view |

What was the **Environment** menu is gone from the bar: adding land is under
Terrain, adding woods and meadows under Plants, a road under Build, and style,
hour, weather and wind are the Environment window (8.14). The same entries are
still a list by the name "Environment" for scripts and keys (`desk.pick`); a
person has no heading for it.

---

## 5. View menu (new 2026-10-13)

| Entry | Key | What it does | Scope |
|---|---|---|---|
| **Look at what is picked** | Home | The view goes to it | view |
| **Show what is picked alone** | / | Only it is drawn, with the land | view |
| **Hide what grows…** / **Hide other things…** | | Leaves a sort out of what is drawn, or brings it back | view |
| **Show everything** | / or Esc | Draws all of it again | view |
| **Colliders: what each thing is to walk into and meet, drawn as lines** | | On or off (new 2026-10-14, 7ak): orange still, green loose, yellow fixed with a body of its own, blue a room that senses or a plate, magenta a room that blows, white a joint (where it holds, its axis, a line to what it holds to; 7al), cyan the one picked; within 26 m (still ones 14); while playing, the bodies where they are | view, not saved |
| *Settings of this machine* | | A heading | |
| **Middle drag turns the view about the place under the pointer** | P | On or off | this machine, until the editor closes |
| **Colours: …** (four) | | What everything selected is marked in. Kept for the next start | this machine |
| **Engine status** | | A window of numbers | view |

What was the **Things** menu is gone from the bar: its windows are under Objects
(and Terrain, Plants) on the toolbar, a house and a street under Build, and what
is done to what is picked is in Edit.

---

## 6. Characters menu

As before (2026-10-11): the walker's kind, *Kinds: someone to walk as*, **Character Studio: characters with bones and clips of their own…** (new 2026-10-15, 7at: section 8.22), **Farm animals: horses, cows,
goats (Asset Library)…** (since 2026-10-14), *Walk as the one Draw3D makes again*, *Who can do what, with what* (F4), *Moves, poses and
acts* (F10), *Movement numbers* (F1), where a game starts, *Play from where the
view looks*.

---

## 7. Window menu

| Entry | Key | What it does | Scope |
|---|---|---|---|
| one entry for each window (section 8), "(open)" before those shown | their keys | Opens it, or puts it away | this machine |
| **Close all panels** | | Puts every window away | this machine |
| **Put every window back where it starts** | | Forgets where windows were put | this machine |

The orbit choice, what is shown and the selection colours moved to **View**.

---

## 8. The windows

### 8.1 Makers: what this world is built of (Ctrl+M) — `makers.gd`

The most loaded window. Top to bottom:

| Part | Control | What it does | Scope | Notes |
|---|---|---|---|---|
| top | a list of kinds of maker, under headings (Land and lines; Buildings; Growing things; Animals and boats; A game's start and its puzzles) | Chooses what **Add…** adds | view | |
| top | **Add…** | Adds a maker of that kind: at once, or by a click on the land, or after a line is drawn | the world | |
| top | the description under it | What the maker is, what it reads, how it is added | | |
| list | every maker in the world, by name, with what it made and what was changed by hand | Click selects it | view (selection) | Things put down one at a time are makers too ("One thing put down"), so a world with many boats has many rows |

With a maker selected (or one thing picked), under the list:

**The sentence in colour.** "CHOSEN: THE MAKER 'woods' (Woods)…" or "PICKED IN THE
VIEW: ONE ROCK (yellow frame)… It belongs to the arrangement of N things made by the
maker…". Its colour is the colour that thing is marked in.

**"This one SORT"** — only with one thing picked; everything here is of that one thing.

| Control | What it does | Scope | Notes |
|---|---|---|---|
| **Take away this one (Delete)** | Removes that thing; its maker keeps it away | one thing | |
| **Duplicate this one… (Ctrl+D)** | Another, by a click, standing by itself | adds one thing, of no arrangement | |
| **Move this one…** | A click says where | one thing | |
| **Put this one back as its maker makes it** | Takes back everything done to it by hand | one thing | Only shown if something was |
| **its design** (list) | Makes **this one thing** another design, where it stands (new) | one thing | "as its maker makes it" appears once it was changed |
| the **no. N** list beside it | Which of that design's sorts | one thing | |
| **its size (×)** | How big it is beside its design: 1 as designed (new) | one thing | Not for a house, nor for what has bones |
| *Its design: X — …* | Says whose the design is and how far shaping it would go (new) | | |
| **Shape this one only…** | Gives this thing a design of its own (a copy) and takes hold of its corners (new) | one thing | Makes a new design named `X-own` each time it has to copy: these collect under "Yours" in Designs |
| **Shape its design 'X' (all N of it)…** | Takes hold of the design's corners on this thing; every thing of the design follows (new) | **a design and every thing of it** | Only for a design of yours that more than one thing is of |
| **Add another SORT to this arrangement…** | Each click puts one more down as one of the maker's own things; Esc ends (new) | adds things **to the arrangement** | Deliberately not Duplicate |
| *This SORT belongs to the … arrangement made by the maker …* | Says the arrangement, the maker, the design | | |
| **Select its maker ('NAME')** | Selects the maker in place of the thing | view | |

Then a line: "ITS MAKER, 'NAME', AND ITS ARRANGEMENT. Everything below is of the
entire maker…", and the maker's parts, each of which folds:

**"Settings and placement"** (for one thing put down: "Where it stands, and on what")

| Control | What it does | Scope |
|---|---|---|
| **name** | Renames the maker (Enter sets it) | one maker |
| **stands at** east / south | Moves the whole maker, in metres | one maker and everything it made |
| **turned (°)** | Turns the whole maker | one maker and everything it made |
| **reach (m)** | How far it reaches; 0 is the whole land. Since 2026-10-11 it keeps its thickness: the count changes with it, and says so | one maker and everything it made |
| **rests on** (one thing put down) | As its sort does / the ground / the water | one thing |
| **Ask what a change would do first (Inspector)** | Opens the Inspector on the maker: the same numbers under the engine's names, each with **?** | view |
| **Look at it** | The view goes to it | view |
| **Move the whole maker…** / **Move it…** | A click says where its middle is; press and drag also sets the reach | one maker and everything it made |

**"Its cells: …"** — new 2026-10-14 (7ad); a structure only (**Structure on cells**,
called "Wall, fence or hedge on cells" until 7af). It has no **Move the whole maker…**: it stands where its cells are, and is
moved by moving its layout or choosing other cells (said under the sentence).

| Control | What it does | Scope | Notes |
|---|---|---|---|
| the sentence | What it is built as, round how many cells, how many sides, joints and gates; for a building its walls inside and its roof; for a pond its water; or that it stands on no cells yet | | |
| **Choose cells…** / **Done choosing cells (Esc)** | Clicks on cells of a layout add them, or take them away | one maker | A cell needs no tile. The same as Build's click after adding |
| **Open or close gates…** / **Done with gates (Esc)** | A click near a side of its rim, or a wall inside: a gate (a door in a building), or wall again | one maker | A side between two of its cells with no wall inside is refused, said why |
| **Walls inside…** / **Done with walls inside (Esc)** | A click near a side between two of its cells: a wall, or none again (a room, a pen) | one maker | New 2026-10-14 (7af). A wall taken away takes its door |
| **Round all tiles of…** | Every tile of a layout, now and later | one maker | Like a wood's "All tiles of…" |
| **built as** | a stone wall; a fence; a hedge; a building (walls, windows, doors); a pond (a kerb, water in it); nothing (only its floor) | one maker | The host's word `as`. Building and pond since 7af |
| **height (m)** | Over the ground along each side: the height it stands at, stepped from there | one maker | As usual: wall 1.2, fence 1.1, hedge 1.6. Until 2026-10-14 it showed 0 for the usual, and **+** made the wall 0.3 m |
| **As usual** | Back to the usual height for what it is built as | one maker | Only when it has a height of its own |
| **floor on its cells** | no floor; paving; gravel; boards; garden soil (beds) | one maker | Level on each cell, walked on. Soil since 7af |
| **roof over its cells** | no roof; tiles; slate; moss-green; thatch | one maker | New in 7af. A roof over each rectangle of its cells; for a building or a stone wall |
| **where a floor stands high** | steps; a ramp; neither | one maker | New in 7af. Down from a floor more than a step over what is beside it, where nothing walls it off |
| **gates swing (off: an open gap)** | A leaf that opens in play, or a gap | one maker | |
| **Gates and doors said: …** | Each gate or door by its side and cell; one whose side is neither on the rim nor walled inside waits | | |
| **Walls inside: …** | Each wall inside by its side and cell | | New in 7af |
| **N sides it shares with another structure are built by that one** | Said where its cells meet another structure's that builds the side between (the more of a building does) | | New in 7af |

Under **How it is built**: its seed (which stone where).

**"Where it makes its things: …"** — new 2026-10-11; a wood or a meadow only. Since
2026-10-12 it has a fourth way, **on terrain tiles** (13i): Choose tiles…, All tiles
of…, Not on tiles, uneven, clear.

| Control | What it does | Scope | Notes |
|---|---|---|---|
| the sentence | Says which of three ways holds: everywhere in its reach; on its own level; only where painted | | |
| **how many** | How many it tries to make (some find no ground and are left out) | one maker | The same number as "count" in the Inspector: two places |
| **keep to the level of its middle, within (m)** (box and number) | Makes nothing on ground higher or lower than that from where its middle stands | one maker | The answer to "trees off my plateau" that needs no painting |
| **Paint where…** | A brush on the land: it makes its things where painted and nowhere else | one maker | Once painted, the reach is ignored. Said, but surprising |
| **Rub out…** | The same brush, removing | one maker | Only when something is painted |
| **Nothing painted: fill its reach again** | Clears the paint | one maker | |
| **brush (m wide)** | The brush's width; **[** and **]** while painting | this machine, until the editor closes | |

**"What it is made of: …"** — a wood, a meadow, a street.

| Control | What it does | Scope | Notes |
|---|---|---|---|
| the rows | Each design given: which sort, how many **parts**, **take out** | one maker and everything it made | "Parts" are shares, not percentages |
| **Add a design…** (list) | Adds a design to what it is made of | one maker | |
| **All designs…** | Opens Designs | view | |
| **Back to the usual** | Gives it no designs: it makes what it always did (or what the style says) | one maker | In the *evergreen* style "the usual" is conifers: the label does not say so |

A **street given a design has no houses** (it is one or the other): a known gap, and a
surprise.

**"How the wood is laid out (3 settings)"** — new in the exposure pass; the heading is
the maker's own: *The land's shape*, *How the street is built*, *How the street is
furnished*, *How the wood / meadow is laid out*, *This house: its size and roof*, *The
flock…*, *The mooring…*, *The errand…*, *Where they lie…*, *Where the pen stands…*.
These are the maker's parameters, which until now were only in the Inspector under
the engine's names with no explanation.

| Control | What it does | Scope | Notes |
|---|---|---|---|
| each setting: a name in words, a number (- field +), and a line under it saying what it does | Sets that parameter (the host's `param`): Enter or a step; one step of undo; the maker makes its things again | one maker and everything it made | Land: layout of the hills (seed), height of the hills, size, level ground beside a road. Street: a house every (m), back from the line, plots left empty, seed. Furniture: the same two, how many sorts, seed. Woods: how many of the six tree shapes, keeps off roads by, seed. Meadow: seed. A house alone: width, depth, wall height, roof colour. Flock: how many, how far along the line, to one side. Mooring: water at least this deep, to one side. Errand, things to find, pen: where along the line. The engine's name for each is in its tooltip |
| **Lay it out another way** | The maker's seed becomes the next number | one maker and everything it made | Was "another seed", in the Inspector only. For a wood, meadow or flock the notice says that changes by hand, kept by name, land on other things |
| **Freeze: keep what it made exactly as it is** / **Thaw: make its things again, as things are now** | The host's `freeze` / `thaw` | one maker | Was in the Inspector only. Frozen, the settings are greyed and the heading says FROZEN |

Not in this part, on purpose: *how many* and *reach* of a wood or meadow (they are in
"Where it makes its things" and "Settings and placement"), what it is made of, and the
raw text parameters `each` and `added`, which stay in the Inspector.

**"Its arrangement: the N things it made"**

| Control | What it does | Scope |
|---|---|---|
| the sentence | How many of each sort | |
| **Put back the N taken away** / moved / turned / otherwise changed | Takes those changes by hand back, all of that kind | one maker and the things concerned |
| **Give the N their maker's design back** / **Make the N resized their own size again** / **Take out the N added by hand** (new) | The same for what the maker keeps of single things | one maker and the things concerned |
| **Each change, one by one** (new in the exposure pass): a row for every single change by hand, "one tree: resized", "one rock: moved — waiting" | **Show**: the view goes to it, and if it stands it is picked. **Put back**: that one change is taken back (the host's `changes`, and `restore` with one record) | one thing | Up to 30 rows. "Waiting" means its maker makes nothing under that thing's name now; the change is kept for when it does. This is the list of what was changed by hand that was missing |
| **Add one thing by hand…** (list of designs, new) | The next clicks each put one down as the maker's own | adds things to the arrangement |

**"What it reads"** — which land, which line. Lists to connect another. **Read by:** who
reads this one. *Scope: one maker.* The words "reads" and "read by" are the engine's;
a user would say "follows this road", "stands on this land".

**"Duplicate or remove the entire maker"**

| Control | What it does | Scope |
|---|---|---|
| **Duplicate the entire maker…** | A ring follows the pointer; a click puts another maker down with all it makes (a line is copied 8 m off at once) | adds a maker |
| **Remove the entire maker and its generated contents…** | Asks "ARE YOU SURE?", then removes the maker and everything it made | one maker and everything it made |

**In the view, with a maker selected:** its ring (or the outline of what was painted, or
with no reach a square about all it made), a stroke over each thing it made, a **box**
in its middle (drag: moves the whole maker) and a **ball** on its ring (drag: sets the
reach, measured on the maker's own level).

**Usability notes.** One window does five jobs: adding makers, listing them, editing
one thing, editing a maker, removing a maker. Its name is the engine's word. A user
looking for "change this tree" or "make the forest smaller" has to know that both are
under World > Makers.

### 8.2 Designs: what things are — `designs.gd`

| Control | What it does | Scope | Notes |
|---|---|---|---|
| the list, under **Yours** and **Built in** | Every design: how many sorts, how many stand, which makers were given it | view | No pictures: names only |
| **which of its sort** | any, or no. N | view | |
| **Place one…** | The next click on the land puts one down by itself | adds one thing | Same as the Place window |
| **Give it to 'MAKER'** | The maker selected in Makers makes its things of this too | one maker and everything it made | Greyed unless a wood, meadow or street is selected in *another window* |
| **Make it mine** (with a name) | Writes a **copy** of a built-in design as a design of this world | adds a design; **changes nothing that stands** | The name suggested ownership of something in the world. It now says it made a copy nothing is of |
| **Shape it in the view…** (yours; new) | Takes hold of its corners on a thing of it; with none standing, the next click puts one down first | **a design and every thing of it** | |
| **Change its text (Kinds and meshes)** (yours) | Opens its text | a design and every thing of it | Was "Change it" |

### 8.2b Library: designs for any world (Ctrl+L) — `library.gd` (new, 2026-10-11)

Designs kept in files apart from any world (`editor\library`), each with a picture.
It does not replace Designs (8.2): that window lists what THIS world can place,
with the code's own things; this one lists what is kept for every world.

| Control | What it does | Scope | Notes |
|---|---|---|---|
| the first box | Which shelf: Everything, each collection (Rural and island, Urban neighbourhood), My library (once something is saved), In this world | view | |
| the second box | A group of that shelf | view | |
| **find by name** | Lists only what has those letters in its name | view | |
| the grid | A picture and a name for each design; white: this world has it; grey: only in the library; blue: saved by the person | view | Pictures are taken when the window opens, a few a frame |
| the picture and three lines under it | What it is called and its name; its line, collection, group and footprint; and where it stands: not in this world, the same as the library's, or this world's own by now | view | The third line is the one that says which of two designs is meant |
| **Place one…** | The next click puts one down, a thing by itself. A design this world has not is brought in first | adds a design (if need be) and one thing | Two things in one press; the window says so |
| **Bring into this world** | Writes the design and what it needs into this world; puts nothing down | adds a design; **changes nothing that stands** | Only shown when the world has not got it |
| **Give it to 'MAKER'** | The maker selected in Makers makes its things of it too | one maker and everything it made | As in Designs: greyed unless a wood, meadow or street is selected in *another window* |
| a name, and **Make a copy of my own** | A design of that name, the same and sharing nothing | adds a design; changes nothing that stands | Of the library's if the world has not got it, else of this world's as it stands |
| **Replace this world's '…' with the library's** | This world's design is made the library's again | **a design and every thing of it** | Only shown where the two differ |
| **CUSTOMIZE THIS WORLD'S '…'**: a swatch for each colour | Every part of that colour is the colour chosen, when the picker is closed | **a design and every thing of it** | A colour of the style's stops following the style. The heading and the note above say the scope before anything is pressed |
| … **whole size ×**, **wider ×**, **taller ×**, **deeper ×** | The design's size, and its proportions | a design and every thing of it | Not shown for what has bones |
| … **Shape it in the view…**, **Change its text** | As in Designs (8.2) | a design and every thing of it | |
| **SAVE THIS WORLD'S '…' TO MY LIBRARY**: name, what it is called, group, **Save to my library** | Writes the design as this world has it to `library\mine` | the library; **the world is not changed** | A name Draw3D's collections have is refused; one's own is written over, the one before kept in `earlier` |
| **Remove from my library…** | Asks, then moves its file to `earlier` | the library; no world | Only for what the person saved |

### 8.2c Layout: a grid to build on (Ctrl+G) — `layoutview.gd`, `grid.gd` (new, 2026-10-12)

Things > "Layout: a square or hex grid to build on, and snap to". A layout is cells
of the world, square or six-sided, kept in the world's file. It is a guide: what is
put down in it is ordinary things. The host works out every cell (`layout.rs`).

| Control | What it does | Scope | Notes |
|---|---|---|---|
| **New square grid** / **New hex grid** | A layout where the view looks, 12 by 12 cells of 2 m, named "grid" (or "grid 2"…), and snapped to at once | the world (one edit) | |
| the list | This world's layouts: name, form, cell size, cells, how many things stand in it. The one snapped to begins "SNAPPED TO:" and is green. A click chooses which the controls below are of | view | Choosing does not snap to it: two different things, one click apart |
| **Snap to this layout** / **Stop snapping: place freely** | From then on what is put down or moved inside it goes to the middle of a cell, and [ ] turn by a quarter (squares) or a sixth (hexes). Changes nothing in the world | this machine, kept by the world's file | Also "Stop snapping" on the row under the bar |
| **Name** | Enter renames it | the world | |
| **Cells** | Squares (four neighbours) or hexes (six). Columns and rows are kept | the world | With the box below ticked, things go to where their cells then are |
| **Cell size (m)** | From the middle of a cell to the middle of the one beside it, 0.25 to 50. The same meaning in both forms | the world | Set with Enter or - +, not by clicking elsewhere |
| **Across (cells)** / **Deep (cells)** | Columns and rows, 1 to 60. The layout stays centred on its place | the world | A thing or a tile in a cell that is no longer there is left where it stands, and a message names it |
| **Turned (degrees)** | Turns the whole layout about its middle | the world | Hexes turned 30 lie flat-topped |
| *It covers … m by … m about X, Z; N cells, N things…, and N terrain tiles* | A readout | | |
| **Things and tiles in its cells go with their cells** | On (the default): a change of size, place, turn or form moves each thing standing by itself in a cell, and each terrain tile, to where that cell then is, in the same edit. Each stays in its own cell: in a smaller cell a thing stands nearer the middle in proportion. A tile is made as big as the cell. Off: only the guides change | how the changes above act | Only things put down by themselves and tiles; a wood's trees never move. Changed 2026-10-12 (L2): before, a thing kept its distance in metres and could land in the next cell |
| **Middle x, z (m)** | Where its middle stands, two numbers | the world (one edit each) | Since 2026-10-14 |
| **Move with the pointer** | The layout follows the pointer, its cells drawn where they would be; a click puts its middle there. Esc leaves it | the world (one edit) | Since 2026-10-14 |
| **To the view's middle** | Puts the layout's middle where the view looks (to half a metre) | the world | Was "Move it to where the view looks" |
| **Look at it** | The view goes to it | view | |
| **Remove this layout (what stands in it stays)** | Takes the layout out. Nothing else changes | the world | Ctrl+Z brings it back |
| **Show the guides** | The lines of every layout, on or off | this machine (not kept between sessions) | |
| **More than one thing to a cell** | On (the default since L2): the first thing put in a cell goes to its middle, and what is put in after it stands where it is put. Off: a cell with something in it takes nothing more | this machine (not kept between sessions) | Was off by default in L1 |
| *Under the pointer, in 'NAME': column C, row R (axial Q, R): empty / a barrel / the tile 'Meadow', and on it: a barrel* | A readout of the cell under the pointer | | Columns and rows are counted from 1 here, from 0 in the host |

**In the view** (`grid.gd`): white lines for the layout snapped to, grey for the
others, lying on the land and hidden by what stands on it; none while playing. The
cell under the pointer is outlined in the colour of "what is about to be put down"
(section 4, Selection colours), with a cross at its middle; red when a click or a
drop there would be refused. **The ring** (since 2026-10-14): while the Layout window
is open, the layout chosen in it has a yellow ring with four arrows at its middle,
seen through what stands there. A left drag on it moves the layout, its cells drawn
green where they would be; where the button is let go it is put (one edit, things and
tiles carried as the box says). Esc while dragging leaves it.

**What snapping changes, and nothing else:** a left click with a thing armed; keys
1 to 9; a right drag of what is picked; Shift + right click; "Move this one…";
"Duplicate this one…"; the pad's away / nearer / left / right (one cell a press); [
and ] (a quarter or a sixth). Each only inside the area of the layout snapped to.
"Add another … to this arrangement" (Makers) is not snapped. **In a cell that has a
terrain tile nothing is snapped** (8.2d): things stand where they are put and turn by
15 degrees.

**Unclear or overlapping.** "Snap to this layout" is per layout while "Show the
guides" and "More than one thing to a cell" are for all of them, in one window with
no line between but a separator. The window's title says "grid", its controls say
"layout", the buttons say "grid": one thing under two words. The list's click
chooses and does not snap.

### 8.2d Tiles: pieces of land for a layout's cells (Ctrl+T) — `tileview.gd` (new, 2026-10-12)

Things > "Tiles: pieces of land, each with what is on it, for a layout's cells". A
tile is a piece of terrain made once (its ground and the things on it) and put down
in cells of a layout. The sets are files kept with the library
(`editor\library\tiles`); a tile put down is a maker of the world (`tile`), and the
host works out its neighbours (`tiles.rs`, `tileops.rs`, `layout.rs`).

| Control | What it does | Scope | Notes |
|---|---|---|---|
| the set (a list) | Which set of tiles is shown | view | One set comes with Draw3D ("Terrain"); a person's own tiles are listed in the set they were made from, in blue |
| *Tiles go into 'NAME' (squares, 8 m)* | A readout: the layout snapped to, which is where tiles are put. "No layout yet…" or "Snap to a layout (Ctrl+G)…" otherwise | | Tiles and ordinary snapping use the same "layout snapped to" |
| the pictures | A picture of every tile of the set, in the form (square or hex) of the layout tiles go into. A click chooses one | view | Taken when the window is first opened, a few a second. White: put down in this world; grey: not yet; blue: yours |
| *NAME: what it is. Ground…; every side says…; N things…* | What the tile chosen is, what its sides say, whether this world has it, and why it does not go into this layout if it does not | | |
| **Put it down…** / **Stop putting down (Esc)** | From then on a left click on a cell puts the tile there and a drag puts one in every cell it passes; in a cell that has a tile it takes that one's place. A piece that does not join what is beside it is turned by the host to join; where no turn joins, the cell is marked red and the click refused | the world (one edit a tile) | Needs a layout snapped to (with one layout in the world it is snapped to for you). Stays on with the window shut; the row under the bar says so |
| **[ turn** / **turn ]** | Turns the tile about to be put down: a quarter in squares, a sixth in hexes | view | The same keys in the view while putting down. The turn is kept for the next tile, and follows the host's when the host turned one to join |
| **Fill the empty cells** | The tile chosen in every cell of the layout that has no tile; cells where it would not join are left empty and counted | the world (one edit) | |
| **New square grid for tiles** / **New hex grid for tiles** | A layout of 8 by 8 cells as big as the set's tiles, where the view looks, snapped to | the world (one edit) | Shown only while no layout is snapped to. The same layouts as in 8.2c |
| **THE TILE PICKED** | | | The part below is of the tile whose ground was clicked in the view |
| *'NAME' (the tile T), column C, row R of 'L', turned K; N things on it. Beside it: …* | A readout; and "DOES NOT JOIN: …" naming each side that does not | | |
| **[ Turn it** / **Turn it ]** | Turns the tile picked in its cell, with everything on it | one maker and everything it made | [ and ] in the view do the same |
| **Replace with the one chosen** | Takes the tile picked out and puts the tile chosen in its cell, turned the same way if that joins | the world (one edit) | What was done by hand to things on the old tile goes with the old tile |
| **Put down more of it** | Chooses the tile this one is, turned as it is, and begins putting down | view | Ctrl+D on a picked tile does the same |
| **Remove the tile (Delete)** | Takes the tile out of the world with everything on it | one maker and everything it made | Delete in the view does the same. No confirmation; Ctrl+Z brings it back |
| a name, **Save as a tile of my own** | The tile picked, as it stands now (with what was moved, taken away, given another design or added on it), written as a new tile of the same set to `library\tiles\mine\NAME.txt` | this machine's tiles (a file); the world is not changed | A name Draw3D's own tiles have is refused; one of yours is not written over from here |

**In the view** (`main.gd`, `grid.gd`): a left click on a tile's **ground** picks the
whole tile (its maker is selected; a frame stands about its cell and everything on it
is marked); a left click on a **thing on it** picks that thing alone, as on any
maker's thing. With a whole tile picked: **[ ]** turn it in its cell; a **right drag**
or **Shift + right click** moves it to another cell (onto a tile, the two change
places); the **pad** moves it a cell a press; **Delete** takes it out; **Ctrl+D**
puts down more of it. A thing put down or moved **on a tile** stands where it is put
and turns by 15 degrees: a tile is ground, not a slot.

**Unclear or overlapping.**

- **A tile is a maker**, so with a tile picked the Makers window shows it ("Terrain
  tile") and what was done by hand to its things; it has no settings there. The
  **Inspector** shows its raw settings (`tile`, `size`, `form`, `beside`, `layout`):
  `beside` and `layout` are kept by the host and are not for changing. Makers' own
  "Remove the whole maker" also removes a tile **(not traced)**.
- **"Take away this one (Delete)"** on the row under the bar removes the **whole
  tile** when a tile's ground is picked, and one thing when a thing is: the same
  button, by what is picked. The row's words say which.
- **Three things are now called by "cell"**: a layout's cell, the thing(s) standing
  in it, and the tile in it. The Layout window's readout says all three.
- **"Put it down…" in Tiles and "Place one…" in the Library** are two tools with the
  left button; choosing either ends the other.
- A tile's ground, what its sides say and its paths are not changed from the window:
  only by its text (`tile_def`, reached by no control).

### 8.3 Place and move — `palette.gd`

| Control | What it does | Scope | Notes |
|---|---|---|---|
| the top button ("put down: rock (click to choose)") | Opens a grid of everything placeable | view | Its own label is its state: easy to miss that it is a button |
| a thing in the grid | Arms it: each left click on the land puts one down | adds things | Keys 1–9 put the first nine down under the pointer without arming |
| **just pick** | Disarms | view | |
| **new kind… F6** | Opens Kinds and meshes | | |
| **all designs…** | Opens Designs | | |
| *picked: …* | What is picked | | |
| **turn left / turn right** | 15° ( [ and ] ) | one thing | |
| **away / nearer / left / right** | Moves it half a metre, as the view shows | one thing | |
| **take away**, **undo**, **redo** | As elsewhere | | Third copy of undo and redo |

Overlaps Designs almost wholly (Designs also chooses which sort). Two windows to put
a thing down.

### 8.4 The land, by hand (F8) — `terrain.gd`

Reached by **Environment > Shape and paint the land, and where things may grow**.

| Group | Tool (button) | What it does | Scope |
|---|---|---|---|
| shape | **raise**, **lower**, **smooth**, **flatten** | Brushes dragged on the land. Flatten levels to the height where the drag began | the land's own layer, where dragged |
| surface | **paint** | Changes the surface (rock, grass, sand, dirt, or "as generated"), not the shape. Not under water | the land's layer |
| what grows | **clear**, **let grow** | Says nothing grows under the brush, or that it does again. Plants nothing | the land's layer; every wood and meadow obeys it |
| stamp | **cliff**, **ledge** | A click raises a sheer block, or a level shelf | the land's layer |
| by hand | **restore** | Takes away what was done by hand (shape, paint, or both) | the land's layer |

| Control | What it does |
|---|---|
| the numbers, each a slider, a step either side and a field: **radius, m**; **m each dab**; **how much**; **high, m**; **back, m**; **above, m**; **turned, °**; **soft rim, m** | Only those the tool uses are shown. Keys: **[ ]** radius, **- =** strength or softness, **, .** turn a stamp |
| **made** (list) | What paint makes the surface |
| **what grows goes by the surface painted** (box) | On: rock, sand and dirt stop growth. Off: painted for looks alone |
| **takes away** (list) | What restore removes |
| **restore all of the land** | Removes everything done to the land by hand, everywhere (undoable) |
| **middle drag turns the view about the pointer (P)** | The orbit choice, again |
| the lines under the tools | What the tool is for; what a stroke did to what grows; what is kept under the pointer |

**In parts** (since 2026-10-14, `section.gd`): **Tool** (the tools, and the line
saying the one in hand), **Its numbers**, **What it does** (what it works on, what it
is for, what the last stroke did), **Under the pointer**, **More** (the keys, restore
all of the land, how the view turns; folded at first). Each heading folds its part;
**Expand all** and **Collapse all** at the top. The lines that change have room kept
for them, so the parts under them stay where they are; what does not fit is in the
tooltip.

**Usability notes.** "clear" and "let grow" control where *every* wood and meadow may
grow; Makers' new "Paint where…" controls where *one* maker grows. Both are brushes,
in different windows, and nothing links them. "restore all of the land" is a large
action in a small button. The window's name gives no hint of "terrain" or "sculpt".

**Under a terrain tile** (8.2d) these tools work on the land beneath it, and say so
(since 2026-10-12): the blue line "Works on: the land…", the tool drawn in stripes on
the tile's ground, and under the numbers which tile the pointer is over and what the
tool chosen does there. No control chooses a tile's own surface: there is no such
tool yet (`terrain.gd`, `target`).

### 8.5 Inspector (F7) — `inspect.gd`

| Control | What it does | Scope | Notes |
|---|---|---|---|
| the list at the top | Every maker ("node") in the world; choosing one shows its numbers | view | The same list as Makers, by another name |
| what is picked, in words | Its sort, its maker, its path under the maker, what was changed by hand | | Engine words: "generator", "path", "live" |
| **east (m)**, **south (m)**, **turned (°)** | Where the picked thing stands | one thing | |
| the maker's **parameters**, each a number or a text | Changes it and makes the maker's things again | one maker and everything it made | Includes raw text parameters: `of`, and since 2026-10-11 `each`, `added`, `level`. Editing `each` or `added` by hand here is possible and easy to get wrong |
| **?** beside a parameter | Says what the change would do before it is done | view | Valuable and hard to notice |
| **freeze** / **thaw** | Keeps what the maker made exactly as it is | one maker | |
| **another seed** | Lays the maker's things out anew | one maker and everything it made | **(not traced)** |
| a line's places (for a road) | Drag one on the world; add or remove a place | one line | |
| **edit this kind (F6)** | Opens the thing's kind | | |
| **edit its shape: write it out as a mesh** / **open its mesh 'X' (F6)** | For a roof, wall, window, door: makes its shape a mesh of yours | one thing | |
| **shape it by hand: take hold of its corners** | The shaping strip | a design and every thing of it, or one thing | |
| **the generated shape back** | Undoes that | one thing | |
| *Physics of this one* (new 2026-10-14, 7ak) | For a thing put down by hand, first, above the maker's parameters | | |
| **weight, kg (0: fixed)**, **bounce**, **grip**, and **damping of its going** (what is loose) or **kicks what touches it** (what is fixed) | Each beside what its design says (grey); in blue where this one says its own; set, it is this one's own | one thing | Its own lines are the parameter `own` of what put it down |
| **design's** | Takes this one's own number away: as its design again | one thing | Blank where there is nothing to take away |
| **its own lines** (text) | All it says of itself, in a kind's words, `;` between: any of loose, bounce, grip, damp, pushed, fast, group, misses, kicks, turns, glides, pinned, senses, when, solid, joins, blows, carries, hits, by, clears, ticks, or `no WORD` | one thing | Anything else is refused with the reason; a joint `to` a name that is not a thing put down by hand, too; `when` takes a condition (7am): `NAME`, `NAME N`, `NAME < N`, `not NAME`, `… for S` |
| **signals: …** (words; new 2026-10-14, 7am) | Which facts this one writes (hits, senses, ticks, clears…) and what it goes by (its `when`, `by`), as it is made | | Shown when it says any |
| **Signals…** (new 7am) | Opens the Signals window (8.20) on the first fact this one names, its row chosen | | |
| **its design also says: …** (new 2026-10-14, 7al) | The design's words that have no row of their own (blows, hits, when, by…), in grey | | Shown when there are any |
| **joined: …** (words; new 7al) | What it is joined to and how, and whether its design's or its own (blue) | | For what is loose |
| **joint N of M: …** (new 2026-10-15, 7ar) | Which of its joints the rows below show and change (a thing may be joined several ways, a `joins` line each); greyed with one or none | | Another thing picked shows its first |
| **+ another joint** (new 7ar) | Joins it one more way: a hinge about its middle, to the world, shown to be told where and to what. Its joints become its own, all of them (its design's copied in) | one thing | Greyed when it is not joined at all (the sort list joins it first) |
| **joint: as its design / not joined / a hinge / a slider / a spring / welded** | A joint of its own of that sort (where it was, or about its middle); of several, **not joined** takes away the one shown, and **as its design** puts back its design's in place of all its own | one thing | `joins` in its own lines |
| **to the world / to 'NAME' (d m)** | What it is joined to: the world where it stands, or a thing put down by hand within 12 m, nearest first | one thing | Renamed in Makers, the joint follows |
| **pick…** (new 2026-10-14, 7am) | The next left click in the view chooses what it is joined to: a thing put down by hand, by its name | one thing | Itself, a maker's thing or nothing: said, and as it was. Esc: not now |
| **click on it…** (new 7am) | The next left click on this one sets where its joint holds: that place, in its own frame | one thing | Hinges and springs |
| **about x / y / z** | The axis of its own a hinge turns about or a slider slides along | one thing | Hinges and sliders |
| **at x y z**, **motor**, **stiffness** | Where on it the joint holds; a motor's speed (°/s, m/s; 0 none); a spring's N/m | one thing | Limits are written in its own lines |
| **collider: …** (words) | What it is to walk into, and whether its design's or its own | | |
| **as its design / a ball round it / a box round it / a post round it / nothing to walk into** | A collider of its own, made to go round what is drawn of it | one thing | |
| **r**, or **w h d**, or **r h** | The size of its own collider | one thing | Shown when it has one |
| **show colliders** / **hide colliders** | View > Colliders | view | |

Overlaps Makers for selecting a maker, moving a thing and changing a count.

### 8.6 Kinds and meshes (F6) — `kindview.gd`

A list on the left (kinds, then meshes in blue), the text of the one chosen on the
right, what it could not understand under it.

| Control | What it does | Scope |
|---|---|---|
| **New** | A kind to start from | adds a design |
| **Duplicate** | A kind "like" the chosen one (it stays linked to it) | adds a design |
| **Delete** | Removes the kind, unless something is of it | a design |
| **Put one down** | The next click puts one down | adds one thing |
| **Walk as this** | The walker becomes this kind | the world |
| **Add the examples** | Writes Wren and two cats into this world | adds designs |
| **New mesh** | A block to start from | adds a mesh |
| **A kind of this mesh** | A kind whose one part is the mesh | adds a design |
| **Shape it in the view** | Takes hold of the mesh on a thing made of it | a design and every thing of it |
| **Apply (Ctrl+Enter)** | Writes the text into the world | a design and every thing of it |

**Usability notes.** This is a text editor for designs. "Kind" and "design" are the
same thing under two names in two windows. "Duplicate" here links the copy to the
original ("like"), unlike every other duplicate in the editor. Which things a mesh is
the shape of is said only in a status line.

### 8.7 What is written: moves, poses, acts (F10) — `textview.gd`

A list of texts (moves, poses, acts, eyes, weather), the text, **Apply (Ctrl+Enter)**,
**As the world has it** (drops what was typed). *Scope: the world; applying also
writes the file of that name beside the scripts.* **(not traced)**

### 8.8 Who can do what, with what (F4) — `hubview.gd`

Read-only: everyone with a body, every kind of thing, every act; what is said of the
one chosen; underlined words lead to where it is written. **open FILE** opens the text
file in the machine's own editor; **write it here (F10)** opens 8.7 at that entry.
*Scope: view.* **(not traced)**

### 8.9 Movement numbers (F1) — `feel.gd`

A readout of what the body is doing, then a slider, steps and a field for each number
the walker moves by. **all back to how they began**. *Scope: this machine's
`feel.cfg`, read by every world.* Available editing and playing.

### 8.10 Shape by hand — `handles.gd`

Opens itself when a shape is taken hold of (from Makers, Designs, the Inspector, Kinds
and meshes, or the context row).

| Control | What it does | Notes |
|---|---|---|
| first line (new) | **SHAPING ONE THING ONLY…**, or **SHAPING THE DESIGN 'X': EVERY thing of it changes, all N…**, or **SHAPING THIS ONE THING** (a generated roof) | The scope of every drag |
| second line | The mesh, its corners and faces, its size | |
| in the view | The shape's edges in the colour of what is picked; every corner a dot (up to 1,500); the one under the pointer marked | |
| left drag on a dot | Moves that corner (Shift: up and down only) | |
| left drag on a face | Moves it the way it looks (Ctrl: pushed out on new corners) | |
| **reach** (slider) | Corners within that distance go with the one dragged (**[ ]**) | |
| **a drag goes** free / X / Y / Z / XZ / XY / YZ | Keeps a drag to a line or a plane (keys X, Y, Z) | Y is up |
| **its text (F6)** | The mesh's lines | |
| **done (Esc)** | Puts the corners away | |

Each drag is one line of text in the mesh and one step of undo.

### 8.11 New world (Ctrl+N) — `newworld.gd`

**What it begins as** (The playground island / An empty island / Nothing at all),
**seed**, **hills (m)**, **size (patches)**, **Make the new world…** which asks once
more ("Yes: replace the world in the editor" / "No, keep this world"). *Scope: the
world in the editor; no file is touched.*

### 8.12 Checklist, Engine status, Keys and mouse

- **Checklist**: boxes to tick, kept on this machine; **untick all**.
- **Engine status**: numbers. Developer information.
- **Keys and mouse**: the whole list of keys.

---

### 8.13 Erase: objects, tiles, what is selected — `erase.gd` (2026-10-13; one place for every removal since 2026-10-14)

Opened by **Erase** on the toolbar, which also takes the eraser up. The eraser is a
brush (`marks.gd`, the tool "strew" with `wiping`): a drag is one stroke, one
command (`erase`), one step of undo. The window is in five parts that fold, with
Expand all and Collapse all, and scrolls within itself.

| Part | Control | What it does | Scope |
|---|---|---|---|
| What is erased | **objects painted with a brush (N in this world)** | On (as it starts): what any brush painted, with one object or a group, whether or not the group is still there | the world |
| | **objects placed one by one (N)** | On: things put down by a click too. Off, as it starts | the world |
| | **terrain tiles (N)** | On: tiles, whole, with what stands on them. Off, as it starts | the world |
| | **which** | *anything ticked (N)*, or *only NAME (N)*: one object or one tile | |
| | the orange line, the small line | "Erases: …. Everything else stays."; how many of each stand | |
| With the eraser | **Take up the eraser** / **Erasing: drag over the world** | Takes it up or puts it away (Esc too) | view |
| | **eraser (m wide)** | Its width; **[** and **]** while in hand | this machine |
| All at once | **in** | *the whole world*, or *the cells of the layout 'NAME'* | |
| | **Clear…** | Says what would be erased there, with counts; nothing goes yet | |
| | **Erase them** | Erases what was said: one edit, Ctrl+Z | the world, or the layout's cells |
| | **Cancel** | Nothing is erased | |
| What is selected | the line | What is selected, and what each button would take | |
| | **Delete the selected thing (Delete)** | The one thing picked; its maker makes the rest | one thing |
| | **Remove the whole maker…** / **Yes: remove 'NAME' and all it made** | The maker chosen and all it made, asked twice | a maker |
| Removed elsewhere (folded) | **A forest's or a street's own things…**, **One brush's own layer only…**, **Paint on the ground…** | Lead to World outline; Objects > Clear it…; Terrain > restore | view |

Never taken by the brush or all at once: a design, a group, a tile of Tiles, anything
of the library; what a wood, a meadow or a street made; a house; the land; what is of
a frozen maker.

### 8.14 Environment: time, weather, wind, style — `env.gd` (new, 2026-10-13)

Opened by **Sky** on the toolbar; while playing, by **Environment** on the bar (the
pointer looks about while playing: F1 frees it for the bar).

| Part | Control | What it does | Scope |
|---|---|---|---|
| Time of day | **Sunrise**, **Noon**, **Sunset**, **Night**, **+1 hour (T)** | Editing: shows the world at that hour (`clock.show_at`). Playing: makes it that hour (`clock.turn_to`) | view while editing; the game while playing |
| Weather | one button for each weather written in `weather.txt` | `weather.become` | view while editing; the game's fact while playing |
| Wind | **None**, **Half**, **As written**, **Twice**, **Three times**, and a number | `weather.set_strength`: every weather's wind so many times as strong | the world (saved with it, undone with Ctrl+Z) |
| Style (editing) | a list of the styles, **Open styles.txt** | `ed.restyle` | the world |
| Older trees (editing, only when there are any) | the orange line, **Let them move in the wind** | The host's `gives` with `fix`: each kind of this world that says nothing of the wind, though the library's of the same name does, is given that one line | a design and every thing of it; one step of undo |

The one that holds is lit in each row. The same entries are still a list for
scripts (`desk.pick("Environment", …)`).

### 8.15 Playtest: the world as it runs — `playtest.gd` (new 2026-10-14, amendment 7ag)

For playing only (P, or the bar's **Playtest**); put away when playing stops. Nothing in
it is kept, and nothing is changed but the hour and how fast it goes.

| Part | Control | What it does | Scope | Notes |
|---|---|---|---|---|
| top | the hour, and the place the walker is in | | | |
| top | **As it runs** / **Ten times** / **An hour a second** | How fast the day goes (`clock.day_s`) | this run | Lit: the one that holds |
| top | **06:00** **09:00** **Noon** **15:00** **18:00** **22:00** | The game's hour made that | this run | As Environment's hours |
| Everyone, and where they are | the list | Each with a body: what they are doing, the place they are in, where they are to be now; at work, which step of which sequence or why they stopped; what they carry | | Sorted by name. A row's whole text is its tooltip (7ah) |
| | **Watch them** | The eye follows the one chosen; the walker stays | the eye | |
| | **Back to the walker** | The eye is the walker's again | the eye | Also when playing stops |
| | **Go to them** | The walker is put beside the one chosen | the walker | |
| | (a line) | How many doors and gates are ways for those who open them, and how many are shut | | |
| The ways from here | **Check the ways to every place** | From where the walker stands to each place by name: reached (how far, through how many doors, and whether it is shut to animals) or how far short | | Waits for the ways to be made (up to 20 s) |
| Goods, and quests (7ah) | the goods panel | What every thing that holds goods holds now (things of one name in one place together: "an apple tree (10 of them): 39 apples"), who carries what, and what the walker has | | Read only |
| | the quests line | Each quest and where it stands: not taken on (from whom), the step it is on and its hint, done, failed | | Read only |
| What happened | the log | Doors opened and shut and by whom; who came into or left which place; who reached where they were to be, or how near they got; goods made, taken, given, bought, sold; work begun, finished, stopped and why, left off; quests taken on, their steps, done or failed | | Newest last; begins again with each run. Also printed to the run's log as `told:` lines |
| foot | **Start again** / **Start again here** | Stops and plays again at once, from the world's start or from where the walker stands; the window stays open | the run | What happened is undone, as stopping always does |

Since 7ai: a row also says a reaction someone is in; what happened has talks (both ways) and reactions begun, finished, stopped and left off; the log shows the last 200 lines.

### 8.16 Journal — `journal.gd` (new 2026-10-14, amendment 7ai)

For playing only (J, or the bar's **Journal**). Read only; nothing in it is kept.

| Part | What it shows | Notes |
|---|---|---|
| Quests | Each quest taken on: what it is called, who asked it and where it stands; its steps, ✓ done, → the one now with its hint, · to come | None taken: says who to look for (a !) |
| What was said | Every line of every talk since the world began to run, both ways, with the hour | The last 40 lines |

### 8.17 The talk panel — `talk.gd` (new 2026-10-14, amendment 7ai)

Shown at the bottom while a talk is held (E beside someone a talk is written of: `talks.txt`).

| Part | Control | What it does | Notes |
|---|---|---|---|
| top | who | Who is talking | |
| | what they say | The words of the point the talk is at | |
| answers | **1. ...** to **9. ...** | Answers the walker may give, in the order written; the chosen one marked ▶ | Greyed, with why, when it cannot be given (and choosing it says so); not shown when its `if` does not hold |
| keys | 1 to 9; Up/Down or W/S, and E, Enter or Space; a click | Answer | |
| | D-pad up/down, A or Y | Answer, with a controller | |
| | Esc, or B | Leave the talk | |
| foot | a line | The keys for the scheme in use (F3 changes it) | |

### 8.18 Physics: what moves, watched and tuned — `physview.gd` (new 2026-10-14, amendment 7aj)

For playing only (F2, or the bar's **Physics**). What is set on a body is undone when
playing stops; only **Keep in the world** and **Write into its kind** change the world.

| Part | Control | What it does | Scope | Notes |
|---|---|---|---|---|
| The world (physics.txt) | a line | Gravity, how fast, steps a second, held or not | | |
| | **Gravity, m/s²** (- number +) | Gravity now; what lies still is woken | this run | |
| | **How fast it runs: ¼ ½ 1 2** | Godot's time scale | this run | The walker and the hour too |
| | **Hold the physics** (toggle) | Everything moved by physics stands; the walker goes on | this run | |
| | **A step on** | One step of the physics (a sixtieth of a second) while held | this run | Greyed unless held |
| | **Keep in the world** | Gravity and speed written into the world's physics.txt | world | A text: may be written while it runs |
| | **Start again** | As Playtest's: the world as it was when it began; the windows for playing stay open | the run | |
| What moves, near the walker | the list | Each loose body within 60 m: what it is, how fast, its weight, at rest or tuned; each fixed body that kicks, turns or glides | | Nearest first; a click chooses one |
| The one chosen | a line | What it is, its kind, how high, how fast it goes and turns, what else its kind says (pushed, group, pinned ...) | | |
| | **Weight, Bounce, Grip, Damping** (- number +) | Set on this body now | this run | Weight and damping of what is loose only |
| | a line | Where it began, and how high it came up after each bounce, against the one before | | What bounce means, measured |
| | **Write into its kind** | Its numbers written into its kind's text (F6) when the world stops: every one of the kind is so from the next run | world | The document is not changed while the world runs |
| | **Keep for this one only** (new 2026-10-14, 7ak) | Its numbers kept as what this one thing says of itself (the Inspector's Physics part), when the world stops; the others of its design stay as they are | one thing | |
| | **Show colliders** (switch, new 7ak) | View > Colliders, while playing: each body's shapes where it is now | view | |
| in the world | a ring and a line | The one chosen, ringed; the way it goes, a line a second long | | |

### 8.20 Signals: what writes each fact, and what goes by it — `signalview.gd` (new 2026-10-14, amendment 7am)

Ctrl+K while editing; Window > Signals while editing or playing; the Inspector's
**Signals…**. Nothing here is kept: it is read from the world (the host's `signals`) each
time, and again by itself after each edit.

| Part | Control | What it does | Scope | Notes |
|---|---|---|---|---|
| top | **Read again** | Reads every thing's signals from the world again | | Also done after each edit |
| | **only facts called…** (text) | Only the facts whose name has this in it | view | |
| | **Draw the wiring** (switch) | Lines in the view from each thing that writes the chosen fact (orange) to each that goes by it (blue), a mark over each | view | Not saved |
| | a line | How many facts, things and designs, joints by name | | |
| what is wrong | a list | A fact gone by that nothing writes (with a written name like it: "did you mean"); a count asked for that what writes it never reaches (a sensor that only holds, asked for 3); something that goes by a fact and does nothing by it; `by` on what has no speed; `clears` with no `when`; a joint to nothing; a world too big to read whole | | **(show)** opens that fact |
| the facts | a list that folds | Each fact: how many write it and go by it (or that the clock or the weather writes it), and while playing its number now; under it each thing that writes (✎, orange) or goes by it (◉, blue), its line, "(own)" where it is its own over its design's | | A maker's things are one row: "woods: 30 post"; things called the same are told apart by where they stand; the texts' `put`, `clear` and `fact` lines too |
| | the right column (playing) | Whether each condition holds now; one that waits (`for S`) says how long it has held | | |
| foot | a line | The chosen fact's writers and readers; the chosen row's line, and where to change it (Inspector, F6, or a text) | | |
| | **Pick it** / **Open the text** | A thing's row: the thing picked in the view and shown in the Inspector. A text's row (sequences, talks, reactions, quests, acts): the texts' window (F10) opened on that text, the line chosen and scrolled to (7an) | one thing / one line | Editing only |

### 8.21 Sounds: what is written, listened to, and who sounds it — `soundview.gd` (new 2026-10-15, amendment 7as)

Window > Sounds, editing or playing (no key of its own). The sounds are the world's text
`sounds.txt` (F10); who sounds each is read from the world (the host's `signals`) and
the weathers each time the window is opened, the text is read again, or **Read again**.

| Part | Control | What it does | Scope | Notes |
|---|---|---|---|---|
| top | **How loud** (slider) | How loud everything is | this machine | Kept with the editor's settings (`user://editor.cfg`), never in a world |
| | **Quiet** (tick) | Nothing is heard | this machine | The same |
| | **Read again** | Reads who sounds what from the world again | | |
| | a line | How many sounds are written, how many things and weathers say one; heard only while the world runs | | |
| what is wrong | a list | A line of sounds.txt not understood (with its line); a sound said by a thing or a weather and not written; one wanted while playing and not found | | |
| the sounds | a list | Each sound written: its name, once or round and round and how long, how many say it | | Double-click: Listen |
| foot | a line | The chosen sound's parts, and who says it (each thing or weather, and its line) | | |
| | **Listen** / **Stop** | Plays the chosen sound once here (editing too); stops it | | |
| | **Open the text** | The texts' window (F10) on sounds, at the chosen sound's `sound NAME` line | one line | Editing only |
| | **Pick one that sounds it** | The first thing put down by hand that says the chosen sound: picked in the view and shown in the Inspector | one thing | Editing only; a weather's is said, not picked |

### 8.22 Character Studio: body, clips, movement, what it holds — `studio.gd` (new 2026-10-15, amendment 7at; authoring, 7au: section 13zj; reliable, 7av: 13zk; playback and panels, 7aw: 13zl; the workflow, 7ax: 13zm; Mixamo, 7ay: 13zn; in sight and what presses each move, 7az: 13zo; Buttons and Gear tabs, 7ba: 13zp; the Make tab, 7bf: 13zp — those sections say what changed since this table)

Characters > Character Studio, editing or playing (no key of its own). What it shows is
the world's text `characters.txt` (F10), `moves.txt` and the glTF files the
characters name; every change it makes is a document edit of characters.txt (Ctrl+Z;
kept with the world by Ctrl+S, never written to the project's file).

| Part | Control | What it does | Scope | Notes |
|---|---|---|---|---|
| top | **Character** (list) | Chooses the character shown | view | The characters written in characters.txt |
| | **Walk as this** | The walker is shown by this character and moves by its set; the game begins | world (a `walks` line) | Ctrl+Z takes it back |
| | **Make a copy** | A new character written as this one is (`NAME_2`) | world | |
| | **Open the text** | The texts' window (F10) on characters, at this character's line | one entry | Editing only |
| | **Read again** | Reads the glTF and FBX files again (after they were made anew on the disk) | | |
| what is wrong | a line | characters.txt lines not understood; what could not be made of the files | | Green when nothing is wrong |
| preview | a view of its own | The character alone on a floor marked in metres | view | Drag to go round it, the wheel nearer; **Front**, **Side**, **Back**, **Above** |
| **Body and bones** (tab) | a line | Its model, bones, size, map; bones not named; the retargeting rules; who the walker is | | |
| | a list | Godot's humanoid names, the bone that is each, its part | | |
| **Clips** (tab) | **Find** (field), **tags** (list) | Narrows the clips by name or word, or by one word | view | |
| | **Import clips...** | Downloaded clips (Mixamo's FBX, glTF) copied into characters/NAME/ and written as a library this character plays | world (characters.txt) and a folder of the project | Godot's own file window; files chosen, or the whole folder |
| | a list | Each clip: name, library, seconds, the words it is found by | | Choosing one shows it |
| | a line | The clip's library and file, length, once or round, stride, marks; **From** and **Licence** | | |
| | **Play**, **Round and round**, **Speed**, the scrubber, the time | Plays the chosen clip in the preview, loops it, slower or faster; dragged, steps through it | view | |
| **Movement** (tab) | **Its moves** (list), **Open the set** | Gives it another set of moves.txt; the set's line in F10 | world | |
| | the look (list) | Each thing the motor says, the clip that shows it, and how (the look says, the clip of its name, its kind's pose) | | |
| | **Show it with the clip chosen** | The row chosen is shown from now on by the clip chosen under Clips | world (its look) | A look shared by several characters changes for all of them: **Copy the look** first |
| | **Copy the look**, **Open the look** | A look of its own, like the one it had; its lines in F10 | world | |
| | the moves (list) | Each move of its set, how it is asked, its phases with their timing and clip, which phases strike | | Double-click: try it |
| | **Try the move** | Plays the move's phases one after another, each as long as the move says | view | |
| **Holds** (tab) | a line | What it holds, where, how turned, the places on it; every place on it | | |
| **Strikes, reactions, the rest** (tab) | a list | When each move strikes and with what (the move's timing, the clip's marks beside it); what begins upon being struck | | |
| | **Who can do what (F4)**, **Sounds**, **Movement numbers (F1)** | The windows for acts, what is heard, and the numbers | | |

### 8.19 A puzzle played — `puzzle.gd` (new 2026-10-14, amendment 7aj)

E at what offers **play** (the Falling blocks frame, or any kind that says `plays NAME`).
The walker stands and is not drawn; the eye is the puzzle's.

| Part | Control | What it does | Notes |
|---|---|---|---|
| top right | the panel | What it is called, points, level, cleared, best, the next piece; held or over; every key below, for the scheme in use | F3 changes the scheme |
| keys | Left Right (A D), Up Down (W S) / D-pad, left stick | Move the piece left, right, away from the eye, nearer, as the eye looks | Held, it repeats |
| | X / A | Spin (about up) | Nudged a cell aside if that frees it |
| | Z / X | Tip over (about the eye's left and right) | |
| | C / Y | Roll (about the way the eye looks) | |
| | Shift / RT | Fall faster, while held | |
| | Space / B | Drop all the way | Points for each cell |
| | Q E / LB RB | The eye a quarter round | |
| | P or Enter / Start | Hold; over, play again | |
| | Esc / Back | Leave | The walker goes on |
| the card (H) | | While a puzzle is played, its keys; otherwise one line about them | |

## 9. The bar while playing

| Name | Key | What it does |
|---|---|---|
| **Stop: back to editing** | Tab or Esc | The world stops; what happened in it is undone |
| **Movement** | F1 | Movement numbers |
| **Controls** | H | The card of controls |
| **Game v** | | Save the game (F5), Load the saved game (F9), An hour on (T), The next weather (N), The next eye (V), Keyboard or controller shown (F3) |
| **Environment** | | The Environment window (8.14), the game's hour and weather |
| **Playtest** | P | The Playtest window (8.15): new 2026-10-14 (7ag) |
| **Physics** | F2 | The Physics window (8.18): new 2026-10-14 (7aj) |
| **Journal** | J | The journal (8.16): new 2026-10-14 (7ai) |
| **Who can do what** | F4 | The hub |
| **Window v** | | As section 7 |
| **Checklist** | | |

---

## 10. Mouse and keys in the view, while editing

| Input | What it does |
|---|---|
| left click | Picks a thing (on a terrain tile's ground: the whole tile); with a thing armed, puts one down; with a tool, the tool's; while tiles are put down, puts one in the cell |
| left drag while tiles are put down | A tile in every cell it passes |
| left drag on a maker's box or ball | Moves the maker / sets its reach |
| right drag on what is picked | Moves it (Esc drops it) |
| right drag elsewhere | Moves the view |
| Shift + right click | Puts what is picked there |
| middle drag | Turns the view |
| wheel | Nearer, farther |
| W A S D, Q E, R F | Move, turn and tip the view |
| [ ] | Turn what is picked (a whole tile: in its cell); or the brush / reach of the tool in use; or the tile about to be put down |
| Delete | Take away the one thing picked (a whole tile: the tile and everything on it) |
| X | Detach what is picked (or, shaping, keep a drag to X) |
| 1 … 9 | Put down a rock, tree, bush, barrel, crate, sacks, fence, cart, lamp |
| O | Put down a rock (an old key) |
| Ctrl+D | Duplicate what is picked |
| Ctrl+Z / Ctrl+Y | Undo, redo |
| Ctrl+N, Ctrl+O, Ctrl+S, Ctrl+Shift+S | New, open, save, save as |
| Ctrl+M | Makers |
| Ctrl+G | Layout: a square or hex grid to build on |
| Ctrl+T | Tiles: pieces of land for a layout's cells |
| Ctrl+K | Signals: every fact, what writes it and what goes by it (new 2026-10-14, 7am) |
| F1, F4, F6, F7, F8, F10 | Movement numbers, hub, kinds, inspector, land, texts |
| F2, T, N | Next style, an hour on, next weather |
| I | What needs a look |
| P | Orbit about the pointer or the middle |
| Tab | Play |
| Esc | Ends a tool, a drag, or the engine's work |

**Keys that mean two things:** **[ ]** (turn a thing or a tile / brush radius / reach / the tile about to be put down), **X**
(detach / lock a drag to X), **N** (next weather; Ctrl+N new world), **- =**, **, .**
only inside the land tool.

---

## 11. Where one task is spread over several places

| Task | Where it can be done |
|---|---|
| Put one thing down | Place and move; Designs > Place one…; keys 1–9; Kinds and meshes > Put one down; Things > Put down a house |
| Select a maker | Makers list; Inspector list; context row "Select its maker"; Makers "Select its maker" |
| Move one thing | right drag; Shift + right click; "Move this one…" (bar, menu, Makers); Place and move arrows; Inspector east/south |
| Change how many trees | Makers > Where it makes its things > how many; Inspector > count; Environment > What grows: trees |
| Change a maker's other settings (spacing, variety, hills, a house's size) | Makers > the part named for the maker (words); Inspector (the engine's names, with **?**) |
| Lay a maker out again; freeze it | Makers > that part; Inspector > another seed / freeze |
| Open a world | World > Open a world…; Open again; Open an earlier save; the editor's start; `run.ps1 -World` |
| Say where things grow | The land > clear / let grow (all makers); Makers > keep to the level; Makers > Paint where… (one maker); a maker's reach (ring, number) |
| Shape something | context row "Shape by hand"; Things menu; Makers "Shape this one only…" and "Shape its design…"; Designs "Shape it in the view…"; Kinds and meshes "Shape it in the view"; Inspector "shape it by hand" |
| Undo | bar; World menu; Place and move; Ctrl+Z |
| Edit a design | Designs; Kinds and meshes; Inspector "edit this kind"; context row "Its kind" |

---

## 12. Names that are suspect, for review

*2026-10-13 (workflow pass, 13r): the window titles and menu names marked **done**
were renamed; the rest are as they were, for the user's review.*

| Name now | Why it is suspect |
|---|---|
| **Makers** | **Done in titles and menus**: *World outline (makers)*. Inside the window the word stays (the user's term) |
| **Kinds** vs **Designs** vs **sort** | **Done in titles**: *Objects in this world (designs)*, *Object text and meshes (kinds)*. "sort" is still both what a thing is and which variant of a design ("which of its sort") |
| **Inspector** / **node** / **generator** / **parameter** | Programming words for "this maker's settings" |
| **What it reads** / **Read by** | The engine's connections; a user would say what it follows or stands on |
| **The land, by hand** | **Done**: *Terrain: shape and paint the land* |
| **clear** / **let grow** | Sound like editing commands; they set where growth is allowed |
| **Paint** (land surface) vs **Paint where…** (a maker's area) | Two different paints. **Now side by side** under Paint on the toolbar, each said as what it puts down |
| **Make it mine** | **Done**: *Make a copy of my own* (as the library said already) |
| **Detach** | Unexplained anywhere |
| **What needs a look** | About hand edits that lost their place |
| **the usual** | What a maker makes with no designs given: depends on the style |
| **parts** | Shares of a mix, not pieces of a thing |
| **reach** | A radius; "size" or "radius" would be found sooner. Also the shaping strip's "reach" (how many corners move together): the same word for two things |
| **arrangement** | Chosen by the user for "what a maker made"; appears beside "maker" in most sentences, which makes them long |
| **Save the world** vs **Save the game** | Two different things saved, F5/F9 against Ctrl+S |
| **Environment** | **Done**: the menu is gone; terrain is Terrain, adding makers is Plants and Build, style/time/weather/wind is the Sky button's window |
| **Grow a design** | **Done**: *Tree Generator* |
| **Take away this one** / **Let go** | **Done**: *Delete*, *Deselect* |

---

## 13. What changed in the interface on 2026-10-11 (the consolidated pass)

So that this inventory can be compared with what was there the day before:

- **New:** Window > Colours (four); Environment > Style: NAME entries; Makers >
  "Where it makes its things" (how many, keep to the level, Paint where…, Rub out…,
  Nothing painted, brush); Makers > "This one …": its design, its size, Shape this
  one only…, Shape its design…, Add another … to this arrangement…; Makers > "Its
  arrangement": three more put-back buttons and "Add one thing by hand…"; Designs >
  "Shape it in the view…"; the shaping strip's first line, edges in the selection
  colour, and corner dots.
- **Changed behaviour:** a maker's reach, dragged or typed in Makers, keeps the
  maker's thickness (its count changes); dragging a reach measures on the maker's own
  level; a maker's ring is drawn on the land while dragged.
- **Changed words:** Designs "Change it (Kinds and meshes)" is "Change its text
  (Kinds and meshes)"; the notes under "Make it mine"; the notice after adding a wood;
  colour names in sentences follow the colours chosen.
- **Not changed:** every other name, menu and place.

---

## 13b. What changed in the capability-exposure pass (2026-10-11, later the same day)

- **New, and where:** the World heading says the world's file and \*; World > Open
  again, Open an earlier save of this world…, Show the folder; Environment > Change a
  wood, a meadow or a road (Makers), Open styles.txt, Shown at: dawn / noon / evening
  / night, Weather: NAME; Characters > Walk as the one Draw3D makes again; Makers >
  the part named for the maker (its settings in words, Lay it out another way, Freeze
  / Thaw); Makers > Its arrangement > Each change, one by one (Show, Put back);
  `run.ps1 -World FILE`.
- **Changed behaviour:** Save keeps what the file had first (`backups` beside it, the
  newest 12) and refuses to write if it cannot; the host writes a world to a part
  file and swaps it in; the editor starts on the world last worked on; a world that
  will not open leaves the editor with no file; an earlier save opens with no file;
  the X key says what detaching did; in Makers, with a maker selected, the
  description of what can be added takes only the lines it needs.
- **Changed words:** World > "Save the world" and "Save the world as…" say where and
  what happens; World > Makers and Things > Detach say more; Makers > "Its numbers
  (Inspector)" is "Ask what a change would do first (Inspector)"; the notice after
  opening and after saving.
- **Not changed:** every other name, menu, key and place. No wholesale renaming or
  regrouping was done: that waits for the review of sections 11 and 12.

---

## 13c. What changed in the island town pass (2026-10-11, later still)

- **Nothing in the interface was added, renamed or moved.** No menu, window, button
  or key is other than it was.
- **New outside the interface:** `editor\town.ps1` (builds `editor\town.json`),
  `editor\run.ps1 -Demo town`, `-Tour -Only town`; `editor\godot\town.txt` (the
  town's designs, as `cast.txt` is).
- **New words a kind may say** (typed in Kinds and meshes, F6; nowhere offered as a
  control): `flutters R H [V]`, `swims R H [LEAP]`, `hops R`, `perches R`,
  `sheer`, `water R H`, `tends SORT R MOVE [S]`. A line of one that is not
  understood is said under the text, as any line is.
- **In the town's own world**, Things > Designs lists about sixty designs under
  "Yours", and Makers lists 57 makers, 25 of them single things put down (each
  building, each person, each cat and dog, the pond's water). This is the long list
  section 8.1 warned of, in a real world.
- **Checklist:** written anew for the town.

Added to section 14's lists by this pass. **C (implemented, not reachable):** writing
the designs of `town.txt` into the world open (the build script does it; Kinds and
meshes has "Add the examples" for `cast.txt` only); building a town (a terminal).
**D (partly implemented):** a garden, a green, a pond's plants kept as a meadow that
makes nothing, to which things are added by hand: it works as a group and is called a
meadow; what flutters, swims, hops and perches goes about only while the world runs,
and stands where it was put while editing; pond water is seen through near and not
from far off; someone who tends things does a move at them and changes nothing.
**E (planned):** unchanged, and the house engine (E2), groups (D3) and behaviour (7g)
are what the town most shows the want of.

## 13d. What changed in the training ground pass (2026-10-11, last of the day)

- **Nothing in the interface was added, renamed or moved**, but for three lines of
  the controls card (Controls, H): "run" now also says what is gone over at a run,
  "crouch" that jump with no way asked is a backflip, and under "Hung from an edge,
  or held to a wall" "jump" says the step up a wall.
- **No number was added to Movement (F1).** What the new moves go by is written out in
  `godot\moves.txt` (`vault`, `wall_run`, `backflip`), which is not reached from the
  editor but through the hub's button that opens the file (F4).
- **New outside the interface:** `editor\run.ps1 -Demo course`, `-Tour -Only course`.
- **New words a kind may say** (typed in Kinds and meshes): `runs`, `drifts R H [V]`.
  **New words a move may say** (typed in `moves.txt`): `when running`, `when still`,
  `when low`, `upon low`.
- **In the town's world**, Things > Designs now lists about ninety designs under
  "Yours" and Makers 68 makers: the long lists of section 8.1, longer.
- **Checklist:** written anew for the course, the children and the tractor.

Added to section 14's lists by this pass. **C (implemented, not reachable):** which
moves the walker has, and every line of a move (a text file; the hub shows it and
opens it, and nothing in the editor changes it). **D (partly implemented):** what
drifts goes about only while the world runs, as 13c's four; a child at a toy does a
move and changes nothing; what is low enough to go over is felt for only by a body
whose set has a move that begins upon it (the walker's).

---

## 13e. What changed in the library pass (2026-10-11, after the training ground)

- **New window: Library** (8.2b), under **Things > Library: designs with pictures,
  kept for every world**, key **Ctrl+L** (said on the Keys card). Nothing else in the
  interface was renamed or moved.
- **Designs (8.2) is unchanged**, and a design brought in from the library is listed
  there under "Yours" like any kind of the world. The two windows overlap: "Place
  one…", "Give it to" and "Shape it in the view…" are in both. See section 11.
- **New outside the interface:** `editor\showcase.ps1` (builds the two showcase
  worlds), `editor\run.ps1 -Demo library`, `-Tour -Only library`, `-Tour -Only
  showcase`; the host flag `--library DIR`. A tour or a demo is given a copy of the
  library.
- **New word a kind may say** (typed in Kinds and meshes): `stretch X Y Z`.
- **A new style**, `city`, under Environment > Style.
- **Checklist:** written anew for the library and the two worlds.

Added to section 11 (one task in several places): **choosing a design to place** is
now in three windows (Place and move; Designs; Library), each listing a different
set: Place the code's things and the world's kinds by name; Designs the same with
sorts and makers; Library what is kept for every world, with pictures. **Making a
design of one's own** is in three: Designs > Make it mine (from one of the code's
things), Library > Make a copy of my own (from a library design or a kind), Kinds
and meshes > new kind (from nothing). Not merged in this pass: nothing is regrouped
until the user has reviewed this inventory (their standing direction).

Added to section 14's lists by this pass. **B (implemented before, newly reachable):**
a kind's colours, size and proportions without typing its text; a kind copied whole
under another name. **D (partly implemented):** customizing reaches colour, size and
proportion, not parts; a style reaches only designs written in its colours. **E
(planned, not existing):** materials; groups of designs; designs painted onto land;
a style that names designs for roles.

---

## 13f. What changed in L1, spatial layout (2026-10-12)

- **New window: Layout** (8.2c), under **Things**, key **Ctrl+G**, listed in Window.
- **Two buttons on the row under the bar** while a layout is snapped to: **Layout
  (Ctrl+G)** and **Stop snapping**; and the row begins with which layout that is.
  With nothing picked the row then says "Nothing picked: left click picks a thing."
  and leaves the view's keys out, for room.
- **Keys and mouse** (the Keys window) gained the line for Ctrl+G.
- **Changed behaviour, only inside a layout that is snapped to:** where a thing put
  down or moved ends up, how far the pad moves it, and by how much [ and ] turn it
  (section 8.2c lists every one). With no layout, or snapping off, nothing differs.
- **A host command gained an option**: `place` takes `yaw`.
- Nothing was renamed or regrouped.
- **Not reached from the interface:** the steps between two cells, and a cell's
  neighbours, are the host's (`cell`) and shown nowhere but as the ring in the
  showcase; they are for L2.

## 13g. What changed in L2, terrain tiles (2026-10-12)

- **New window: Tiles** (8.2d), under **Things**, key **Ctrl+T**, listed in Window.
- **Two buttons on the row under the bar**: **Stop putting down (Esc)** while tiles
  are being put down, and **Tiles (Ctrl+T)** then and while a whole tile is picked.
  The row says "Putting down the tile 'NAME' in 'LAYOUT'…" or "Picked: the tile
  'NAME' (column C, row R), with the N things on it…".
- **A click on a tile's ground picks the whole tile.** With one picked, [ ], a right
  drag, Shift + right click, the pad, Delete and Ctrl+D act on the tile (8.2d says
  each). A click on a thing on a tile picks that thing, as before.
- **Keys and mouse** (the Keys window) gained the line for Ctrl+T. **[ ]** now mean a
  fourth thing: the turn of the tile about to be put down.
- **Changed in the Layout window (8.2c):** "More than one thing to a cell" is
  **ticked by default** and means "the first goes to the middle, the next stands
  where it is put"; "Things in its cells go with their cells" is now "Things **and
  tiles** in its cells go with their cells", keeps each thing in its own cell when
  cells are made smaller, and **names what could not go** (its cell is no longer in
  the layout) in a message; the readout counts the layout's tiles.
- **Changed behaviour inside a layout that is snapped to:** in a cell that has a
  terrain tile, things are put and moved freely and turn by 15 degrees; and a second
  thing put in a cell stands where it is put. Only the first thing in an empty cell
  goes to its middle.
- **Makers lists a new sort of maker**, "Terrain tile", once a tile is put down; it
  is not in the list of makers to add (tiles are put down from Tiles).
- Nothing was renamed or regrouped.
- **Not reached from the interface:** a tile's text (`tile_def`); a tile put down
  where it does not join (`tile_put` with `anyway`); the ground of a tile saved
  (`tile_save` with `ground`); which turns join and why (`tile_fit` says more than
  the red cell does).

## 13m. What changed in the convenience pass (2026-10-13)

- **Context row, with one thing picked: "Look at it (Home)" and "Show it alone (/)".**
  The first brings the view to the thing, as near as shows the whole of it. The
  second draws only that thing, with the land (a house with all its parts); Shift+/
  draws every thing of the same design instead. Scope: this machine, until
  everything is shown again; never the world or its file.
- **Context row, whenever something is not drawn: "Show everything (/ or Esc)"**, and
  the row's words begin *NOT ALL IS SHOWN (…)*. Esc does it only when no tool is in
  use (a tool's own Esc comes first).
- **Window menu: a heading "Shown in the view"** with *Hide what grows* / *Show what
  grows again*, *Hide other things* / *Show other things again*, and *Show
  everything*. Not kept between starts. Play shows everything.
- **Keys:** Home, /, Shift+/ in the view while editing; listed under "The view" in
  Keys.
- **Usability notes.** A second preference-like group now sits in the Window list
  (section 7's note stands). "Look at it" exists twice, for a maker and for one
  thing, as two buttons with the same words. Sections 2, 7 and 10 above are not
  rewritten: this section is the record until the inventory is gone over.

## 13l. What changed in "paint on what is seen" (2026-10-12, amendment 7v)

- **The land, by hand (F8) > paint: a new row, "paints on".** Two choices: *what I
  see: a tile, or the land* (how it starts) and *the land, under any tile* (the tool
  as it was). Shown only while paint is chosen. Not kept between runs of the editor:
  it starts on "what I see" each time.
- **The blue line of that panel changes with it.** Other tools: "Works on: the land…"
  as before. Paint on what is seen: "Paints: what you see. On a terrain tile, the
  whole tile… Off the tiles, the land under the brush." Paint on the land: "Paints:
  the land, under any tile…".
- **The yellow line under the tools**: "paint: drag on a tile or on the land" when
  paint goes on what is seen.
- **Under the pointer**, on a tile, with paint on what is seen: "a tile is here
  ('name': ground): a stroke put down here makes the whole of this tile sand. The
  land under it, the other tiles and the tile's own things stay as they are." A tile
  that was painted says so, and what it is by its design.
- **In the view**: with paint on what is seen no brush is drawn on tiles; the tile
  under the pointer is tinted whole. Off the tiles the brush is drawn as ever. With
  paint on the land, stripes on tiles as before.
- **After a stroke** (green, under the tools): "That stroke painted the tile 'name':
  rock, the whole of it. The land under it is as it was. Ctrl+Z takes it back."
- **Tiles (Ctrl+T): a new part, "TILES IN NO CELL"**, above "THE TILE PICKED", shown
  only while there is one: how many, what that means, each with why (the first six),
  and two buttons: "Show the next one" (picks it and looks at it) and "Give the one
  picked the nearest empty cell".
- **Tiles > the tile picked** says "IN NO CELL: why" in place of its column and row,
  and "painted sand (grass by its design)" for a tile that was painted. "Replace
  with the one chosen" is not offered for a tile in no cell.
- **Messages**: putting a tile down or moving one onto a tile in no cell is refused
  with that tile's name; a layout change that leaves things behind now says that
  nothing was taken away and that Tiles lists the tiles in no cell.
- **Not in the interface** (the host has them): painting a tile forest or snow
  (`tile_paint`), and putting one tile over another on purpose (`stack`).
- **No control was renamed, moved or regrouped.** "paints on" is one more row of an
  existing panel; the names in it are the user's own words from the brief ("what I
  see", "the land under tiles") and can be changed with the rest when the inventory
  is reviewed.

## 13k. What changed in the user’s fourth look at tiles (2026-10-12)

- **No control was added, renamed or moved.** What a person sees: the sides of
  tiles no longer flicker as the eye moves, and tiles with no neighbour have no thin
  dark line round them from far off.
- **The checklist** has a new part, "Tiles as the eye moves".
- **Recorded for the interface, and not built** (amendments 7s to 7u): the paint
  tool is to say what it paints on ("what I see" or "the land under tiles"); a tile
  is to have a surface that can be painted; a brush is to say four things plainly
  (the tool, what it paints with, what surface, and add / replace / clear); from a
  thing or a design: Place, Paint with, Edit, Make a variant, Open its generator;
  the wheel over a panel is to work the panel; what is picked is to be let go or
  made right when the tool changes; a number changed is not to make its panel jump;
  a brush is to be sized in the view. None of these is to be done piecemeal: one
  coordinated pass, after the user has looked at this inventory.

## 13j. What changed after "the land under tiles" (2026-10-12)

- **The land, by hand (8.4)**: the stripes a tool is drawn in on a terrain tile's
  ground are **thin, a metre apart and fainter** (they were half the brush). Its
  outline, and the plain tint on bare land, are as they were. No control added,
  renamed or moved.
- **Nothing else a person sees changed**, but that a tile's ground no longer jumps
  for a frame while the land under it is shaped.
- **Not in the editor, and written down only** (amendment 7q): tools for ridges,
  shelves, terraces and the like, and generators of landforms and water. When they
  come they belong under **Environment**, beside "Shape and paint the land", and
  each is to say whether it changes the land or puts down a thing.

## 13i. What changed in "the land under tiles, and woods on tiles" (2026-10-12)

- **The land, by hand (8.4)**: a new line under the tool's name, in blue, **"Works on:
  the land…"**; the tool is drawn on a terrain tile's ground too, **in stripes** (plain
  on the land); the lines under the numbers now end, with the pointer on a tile, in
  **"a tile lies over the land here (its name, its ground): …"** and what the tool
  chosen does there. "under the pointer the ground is" now reads "the land is".
- **Makers > Where it makes its things (8.1)**, for a wood or a meadow: **Choose
  tiles…** (becomes **Done choosing tiles (Esc)**), a list **All tiles of…**, and, once
  it is on tiles, **Not on tiles: fill its reach again**, with two numbers, **uneven,
  tile to tile** and **clear of the tile's own things (m)**; "how many" reads "how
  many on each tile". The heading says "on N chosen tiles in 'layout'" or "on every
  tile (of a ground) of 'layout'".
- **A new mode of the left button**: while tiles are being chosen for a maker, a
  click on a tile adds it or takes it away, and picks nothing. Esc ends it. Nothing
  on the bar says the mode is on but the notice and the button's words.
- **Nothing renamed or moved.** The checklist is written anew.

## 13h. What changed in the pass after L2 (2026-10-12)

- **Nothing in the interface**: no control was added, renamed or moved.
- **What a person sees is different in one place**: a terrain tile's ground stays on
  the land when the land under it is shaped (before, the land came through it).
- The checklist has a ninth part for it. Two limits of the land's tools under a tile,
  and of what a wood's number means, are written under 14 D.

## 13n. What changed in tile elevation, first tier (2026-10-13)

- **Tiles window, under "the tile picked": a row "Above the land (m)"**: a number
  (0 to 40, steps of 0.5; set with Enter), **"On the land"** (0 again), and **"Those
  beside it as high"** (every tile beside the one picked made as high: one edit).
  Under it a note: Page Up and Page Down, and that the land's tools still shape the
  land under it. Scope: the one tile put down (kept with it in the world's file).
- **The words about the tile picked** now end "on the land" or "raised N m above the
  land".
- **Keys:** Page Up / Page Down with a tile picked, in the view while editing; listed
  under "The world" in Keys, by Ctrl+T.
- **A click that puts something down, or drags it, meets a tile's ground where it is
  seen** (it met the land under the tile). The land's tools still meet the land.
- **Usability notes.** Nothing about height is in the row under the bar for a tile
  picked: it is in the Tiles window and on two keys only. "Above the land" is a
  third thing the Tiles window says of the tile picked (turn, replace, height),
  beside save and the tiles in no cell: the window is long. The land's panel (F8)
  does not say that the tile over the pointer is raised.

## 13o. What changed in paint with one design, first tier (2026-10-13, amendment 7x)

- **Designs window, the row of buttons under a design chosen:** two more, after
  "Place one…": **"Paint with it…"** and **"Clear it…"** (each stays pressed while
  that brush is in hand; pressed again, the brush is put away). Not offered for a
  design with bones (a figure is put down one at a time).
- **Designs window, while a brush is in hand:** a part headed "The brush: painting
  with 'NAME'" (or "clearing"), with **"Put it away"**; **brush (m wide)**, **apart
  (m)**, **size from** and **to**; a tick, **"each turned its own way"**; and a
  note saying what a stroke leaves. Scope of these settings: the editor, until it
  is closed; they are not kept with the world or with the design.
- **In the view:** the brush is a ring on what is seen (the land or a tile), with
  the stroke so far as a line; the row under the bar says "PAINTING WITH 'NAME'…"
  or "CLEARING…", the width and how far apart. `[` and `]` change the width; Esc
  ends it. The left button is the brush's while it is in hand.
- **Makers window:** a new sort of maker in the list, **"Painted things"**, named
  "painted NAME". It has no settings of its own but its name; the box in its middle
  moves all of it. (What Makers offers for a wood's things added by hand it offers here: not tried for a painted maker.)
- **The bar:** Window > Designs now reads "everything there is to place, to paint
  with, or to give a maker".
- **Usability notes.** "Paint" now means three things in three places: the land's
  surface (Environment, F8), where a wood makes its things (Makers > "Paint
  where…"), and things of a design (Designs > "Paint with it…"). 7t means them to
  become one act; today they are three tools with one word. A stroke lands when the
  button is let go, not as it is dragged. The brush's settings are in the Designs
  window, which has to stay open to change them. Makers counts a painted maker's
  things as "added by hand".

## 13p. What changed after the hand test; groups and replace; Grow (2026-10-13, amendments 7y, 7z)

- **Every notice** (the plate at the bottom of the view) goes on as many lines as
  it needs and is never wider than the window.
- **The row under the bar** ends in "…" when it is longer than the window, with the
  whole of it said when the pointer rests on it; and says **"Works on: …"**, first,
  in every state that edits (the land or what is seen;
  cells of a layout; one tile; this one thing only; the whole maker; a brush's
  layer). While a tile is put down it says what a click would do in the cell under
  the pointer (empty; REPLACES the tile 'NAME'; that tile already; not put down
  and why). New button there: **"Put it down anyway"**, only while a tile was
  refused.
- **Tiles window:** a row **"Grid (snapped to): [list of layouts] · Layout…
  (Ctrl+G)"** under the set; a box above the pictures while a tile was refused
  (why, **"Put it down anyway"**, **"Show the cell"**, **"Never mind"**);
  "Replace with the one chosen" is now **"Replace with 'NAME'"**.
- **Layout window:** **"Put tiles in it…  (Ctrl+T)"** beside "Snap to this layout".
- **Designs window:** what is said of the one chosen is scrolled. The list begins
  with **"Groups: mixes of designs to paint with"**. A design has two more
  buttons: **"Replace it…"** and **"Add to a group…"** (or **"Add to the group
  'NAME'"**). Replacing, the brush's part has **"with [a design]"**. A group
  chosen shows its designs in rows (**share**, **size from**, **to**, **turned**,
  **x**), **"a design to add… · Add it"**, **"Paint with it…"**, **"Clear
  it…"**, **"Another group"**, **"Remove the group"**.
- **Things > "Grow a design: a tree made from a few numbers…"**: a new window,
  **Grow**. "Grow [a new tree | 'NAME', grown before]"; the picture (drag to
  turn); a row for each number (**form**, **height**, **trunk**, **lean**,
  **crown**, **starts**, **boughs**, **lumps**, **leaf**, **light**, **seed**); a
  name; **"Make it a design"** / **"Grow 'NAME' again"**; **"Another like it"**;
  **"Place one…"**; **"Paint with it…"**; **"Show it in Designs"**.
- **Words to look at when the inventory is reviewed:** "group" (a mix of designs)
  beside "arrangement" (what a maker made) and "set" (of tiles); "layer" is used
  in the documents for what a brush painted and nowhere in the editor; "grow" for
  generating a design beside "what grows" for vegetation on the land.

## 13q. What changed in tree generator 2 and the living environment (2026-10-13, amendment 7aa)

- **Grow** (Things > "Grow a design") is other:
  - **Pictures across the top**, one for each thing to begin from, its name in a
    colour: white a preset, green a design of this world that was grown, blue one
    of the library's, grey a tree that was not grown. A click loads its numbers (a
    grey one: the preset nearest it, said as that, with its picture put beside).
  - **The numbers under headings**: Form, Trunk, Boughs, Crown, Foliage, Colour,
    Seed. 31 of them; six that choose are lists (form, shape, foliage, bark, and
    the two leaf colours).
  - **"in the wind"** under the picture.
  - **"As a new design"** beside "Another like it", when what is shown is a design.
  - **A question** in place of the last row of buttons, when a design shaped by
    hand would be grown again: "Keep it as it is", "Make this a new design",
    "Grow it again".
  - The list "Grow: a new tree / 'NAME', grown before" is gone: the pictures are it.
- **Under the bar**, with a thing picked whose design was grown: **"Its numbers
  (Grow)"**.
- **Designs**, a grown design chosen: **"Its numbers… (Grow)"**.
- **Environment menu**: "Shown at: sunrise / noon / sunset / night" (it was dawn,
  noon, evening, night; sunrise and sunset are by the style's day); a new part
  **"Wind: none / N times as written"** with five entries, "No wind at all" to
  "Three times". **The menu is scrolled** when it is longer than the window.
- **Game menu** (playing): "To noon", "To sunset".
- **Words**: a tree's *form* is how it grows; a *preset* is numbers to begin from;
  a *design* is what was kept. "Conifer" is the name of one of the code's designs,
  not of a form: the form is "upright".
- **No key was added.**

## 13r. What changed in the workflow pass (2026-10-13, amendment 7ab)

The first pass that regroups and renames, at the user's asking ("Redesign the
editor's information architecture"; "Make terminology understandable"). Sections 1
to 7 above are rewritten for it; 8.13 and 8.14 are new. What moved, and what is
called something else:

| Was | Is now |
|---|---|
| bar: World, **Environment**, **Things**, Characters, Window | bar: World, **Edit**, **View**, Characters, Window; and a **toolbar** down the left: Select, Move, Paint, Erase, Terrain, Objects, Plants, Build, Sky |
| World > Makers; Environment > Change a wood… | Objects > **World outline**; Plants > Change a forest or a meadow… |
| World > Undo, Redo, What needs a look | **Edit** |
| World > Engine status | **View** |
| Environment > Shape and paint the land; Add land | **Terrain** > Terrain tools; Add land |
| Environment > Draw a line: a road | **Build** > Draw a road |
| Environment > Add woods / a meadow; What grows: … | **Plants** > Add a forest / a meadow; How many trees… / How much grass… |
| Environment > style, hour, weather, wind (a list to scroll) | **Sky** > the Environment window (rows of buttons) |
| Things > Place things; Library; Designs; Inspector; Kinds and meshes | **Objects** > Quick place; Asset Library; Objects in this world; Inspector; Object text and meshes |
| Things > Layout; Tiles | **Terrain** > Grid layout; Terrain tiles |
| Things > Grow a design | **Plants** > Tree Generator |
| Things > Put down a house; Add a street | **Build** |
| Things > Duplicate, Move, Shape, Take away, Detach | **Edit** > Duplicate, Move, Edit the shape, Delete, Detach |
| Window > orbit, what is shown, selection colours | **View** |
| context row: up to twelve buttons on up to three lines | the tool in hand, the sentence, six frequent buttons and **More**, on one line |

Window titles: *Makers: what this world is built of* → **World outline: forests,
streets, the land (makers)**. *Designs: what things are* → **Objects in this world
(designs)**. *Library: designs for any world* → **Asset Library: objects for any
world**. *The land, by hand* → **Terrain: shape and paint the land**. *Grow a
design: a tree from a few numbers* → **Tree Generator**. *Kinds and meshes* →
**Object text and meshes (kinds)**. *Place and move* → **Quick place and nudge**.
*Layout: a grid to build on* → **Grid layout: cells to build on**. *Tiles: pieces of
land for a layout's cells* → **Terrain tiles: pieces of land for a grid**. *Shape by
hand* → **Edit shape**.

Labels: *Take away this one (Delete)* → **Delete**. *Let go* → **Deselect**.
*Duplicate this one… (Ctrl+D)* → **Duplicate (Ctrl+D)**. *Move this one…* →
**Move**. *Shape by hand* → **Edit shape**. *Its numbers (Grow)* → **Tree
Generator**. *Its kind (F6)* → **Edit its text (advanced, F6)**. *View its maker
in Makers* → **Show its maker in World outline**. In Objects: *Make it mine* →
**Make a copy of my own**; *Its numbers… (Grow)* → **Open in Tree Generator…**;
*Change its text (Kinds and meshes)* → **Edit its text (advanced)**; *Give it to a
maker…* → **Give it to a forest or street…**; the headings *Brush groups*, *Yours:
objects of this world*. In the Tree Generator: *Make it a design* → **Save as an
object**; *Another like it* → **Another seed**; *As a new design* → **Save as a new
object**. Wherever a sentence named a window or a menu, it names the new one.

New controls: the toolbar; **More**; the tool's word at the head of the row; **Erase
objects** (8.13); **Environment** (8.14); in Objects, **moves in the wind** (a tick
box, for an object of this world without bones: its `gives` line with usual
numbers, or none); in the Tree Generator, under each picture what it is (preset,
saved tree, library tree, reference only), the weight of what is shown (HEAVY from
seven tenths of the most corners), and "NOT SHOWN: Too much for one tree…" with
what to lower.

**Not renamed, on purpose:** the words *maker*, *arrangement*, *thing*, *sort*,
*design* inside the windows' own sentences (the user's terms of 2026-10-11); the
Inspector; Characters and its windows; the parts and buttons of World outline, the
Asset Library, Grid layout, Terrain tiles and Terrain. Section 12's other suspects
(*What it reads*, *clear* / *let grow*, *the usual*, *parts*, *reach*) are as they
were, for the user's review. **Sections 8.1 to 8.12 and 9 to 11 describe those
windows by their earlier titles**; what each control does is unchanged.

## 13s. What changed after the workflow pass (2026-10-14, amendment 7ac)

| Where | What changed |
|---|---|
| Every tool | One tool in hand at a time: taking one up puts every other away (a brush, the eraser, a land tool, placing, tiles, a shape's corners, a layout being moved) |
| The wheel | Over a window, a list, the bar or the toolbar it is theirs; only over the world does it move the view |
| Erase (8.13) | Titled **Erase: objects, tiles, what is selected**; terrain tiles; all at once in the whole world or a layout, said first; what is selected; in parts that fold |
| Grid layout (8.2c) | The ring in the view; **Move with the pointer**; **Middle x, z**; *Move it to where the view looks* → **To the view's middle**; the note at the top shortened |
| Terrain (8.4) | In parts that fold; words with room kept for them |
| Objects, Characters | **Animals: horses, cows, goats** / **Farm animals** open the Asset Library on its new collection |
| Asset Library | A third collection, **Farm animals** (group Animals): horse, grey horse, cow, brown cow, goat |
| Build > street | A road with a street already: a new road is drawn; a road drawn now gets its street; what was made is said |
| Adding any maker | Says how many things it made, or that it made none here and why that may be |

Not changed: the words inside the other windows; the bar; the toolbar's buttons.

## 13t. What changed in adaptive modular construction, first tier (2026-10-14, amendment 7ad)

| Where | What changed |
|---|---|
| Build (toolbar list) | A heading **Build on a grid's cells** and four entries under it (2a): a wall, fence or hedge round cells; a path, yard or terrace; open or close gates; the Grid layout. Its tooltip names walls, fences, hedges, paths and yards |
| World outline (8.1) | A new maker under Buildings, **Wall, fence or hedge on cells**; with one chosen, the part **Its cells** and, under How it is built, its seed; no Move the whole maker for it |
| Every toolbar list | A row that gives the pointer a "click where" tool no longer puts the thing down at once under the list (the list's release reached the world) |

Not changed: the bar; the toolbar's buttons; the other windows.

After the user's hand test the same day: **height (m)** shows the height a structure stands at (not 0 for the usual) and **As usual** is beside it when it has a height of its own.

## 13u. What changed when modular construction was brought to a stopping point (2026-10-14, amendment 7af)

| Where | What changed |
|---|---|
| Build (toolbar list) | Under **Build on a grid's cells** (2a): a house, a fence, a hedge, a path or square and a pond, each a structure made as it says from the first click; **Walls inside the structure chosen**; the gates row now says gates and doors. The path row no longer opens World outline |
| World outline (8.1) | The maker is called **Structure on cells**. In **Its cells**: **Walls inside…**, **roof over its cells**, **where a floor stands high**; built as a building or a pond; garden soil as a floor; the walls inside said; the sides another structure builds said |
| A world | `editor\showcase-village.json` (`editor\showcase.ps1 -Which village`), and `run.ps1 -Demo village` |

Not changed: the bar; the toolbar's buttons; the other windows.

## 13v. What changed in "a living village", first tier (2026-10-14, amendment 7ag)

| Where | What changed |
|---|---|
| The bar while playing (9) | **Playtest  (P)**, opening the new window 8.15 |
| Keys while playing | **P**: the Playtest window (also on the card of controls, H) |
| Asset Library | A new shelf, **Village people** (group People): someone of the village, the baker, the smith, the shopkeeper, the gardener, a boy, a girl |
| Object text (F6) | A kind may say `goes HOUR at PLACE` (a structure by its name) or `goes HOUR at home` |
| A world | `editor\showcase-village.json` rebuilt with its people; `run.ps1 -Demo villagers` |

Not changed: the bar while editing; the toolbar; the other windows.

## 13w. What changed in "the village at work" (2026-10-14, amendment 7ah)

| Where | What changed |
|---|---|
| The Playtest window (8.15) | Each row says the work and what is carried; a **goods** panel and a **quests** line; more in what happened |
| While playing, bottom left | Above what is kept: the quest taken on and its step (and hint), and "You have 4 coins, 2 loaves" |
| While playing, the prompt | Beside something whose acts are refused for want of goods, why ("Buy a loaf (2 coins) at: You have no coins"); with someone passing offered too, both |
| While playing, E | Also: talk to (anyone of the village), bake bread in, put the flour into, take a sack of flour from, buy a loaf at, pick an apple from, sell an apple at, leave the bread on. A thing at hand is offered before someone passing |
| What is written (F10) | Three new texts: **goods**, **sequences**, **quests**; **acts** has goods lines (`makes`, `uses`, `gains`, `hands`), `for walker`/`for folk`, `talks` |
| Object text (F6) | A kind may say `goes HOUR do NAME` (a sequence), `stock NAME N [MOST]`, `shows NAME x y z`, `speaks WORDS` |
| Asset Library | Rural and island, new group **Village work**: bread oven, flour sacks, market stall, shop counter, long table. The apple tree holds four apples; the Village people can be talked to |
| A world | `editor\showcase-village.json` rebuilt with its work and the cook; `run.ps1 -Demo work` |

Not changed: the bar; the toolbar; the windows for editing.

## 13x. What changed in "the village answers" (2026-10-14, amendment 7ai)

| Where | What changed |
|---|---|
| While playing, E beside someone a talk is written of | A **talk panel** at the bottom: who, what they say, numbered answers (greyed, with why, when one cannot be given); 1 to 9, Up/Down (W/S) and E or Enter, a click; a controller's D-pad and A or Y; Esc or B leaves. The walker and they stand while it is held; the pointer is free |
| While playing, in the world | A gold **!** over whoever has a quest to ask now; a blue **▼** over where the quest's step is done |
| The bar while playing | New button **Journal (J)**: the Journal window (8.16) |
| Keys while playing | **J**: the journal (keys only; no controller button is free) |
| The controls card (H) | A line for J |
| The Playtest window (8.15) | Rows say a reaction ("sheltering from the rain, while the weather is rain or storm"); what happened has talks and reactions; it shows the last 200 lines (was 80) |
| Notices | Moved above the act prompt and above the talk panel when they would lie over them |
| What is written (F10) | Two new texts: **talks**, **reactions**; **sequences** take `face`, `move`, `go walker`, `go indoors`; **quests** take `at THING` under a step; conditions `near`, `quest`, `walker ...` |
| A world | `run.ps1 -Demo talk` (8 beats) |

Not changed: the bar while editing; the toolbar; the windows for editing.

## 13y. What changed in general-purpose physics (2026-10-14, amendment 7aj)

| Where | What changed |
|---|---|
| The bar while playing | New button **Physics (F2)**: the Physics window (8.18) |
| Keys while playing | **F2**: Physics. At a puzzle, its own keys (8.19) |
| While playing, E | Also: **kick** (a ball), **play** (a puzzle's frame) |
| The controls card (H) | A line for F2; while a puzzle is played, its keys only |
| Asset Library | New collection **Physics lab**: Balls, Ramps and walls, Moving parts, Puzzles (22 designs) |
| World outline, one thing put down | New setting **raised above where it rests (m)**: also in the Inspector as `raise` |
| Object text (F6) | A kind may say `bounce`, `grip`, `damp`, `pushed`, `fast`, `group`, `misses`, `pinned`, `kicks`, `turns`, `glides`, `plays`, and `solid ball` |
| What is written (F10) | Two new texts: **physics**, **puzzles**; **acts** has `sends` and `plays`, and the acts **kick** and **play** |
| Start again (Playtest, Physics) | The windows for playing that were open are open again after |
| A world | `editor\showcase-physics.json` (`editor\showcase.ps1 -Which physics`); `run.ps1 -Demo physics` (10 beats) |

Not changed: the bar while editing; the toolbar; the windows for editing.

## 13z. What changed in physics made authorable, second tier (2026-10-14, amendment 7ak)

| Where | What changed |
|---|---|
| Inspector (F7), a thing put down | New first part **Physics of this one** (8.5): five numbers beside the design's, **design's**, **its own lines**, the collider and a shape of its own, **show colliders** |
| View menu | New **Colliders** (on or off) |
| Physics window (F2) | New **Keep for this one only** and **Show colliders**; it is filled as soon as it opens |
| In the view | Colliders as coloured lines when asked for, editing or playing |
| Object text (F6) | A kind may say `senses NAME W H D [group N] [counts]`; `when NAME` now also holds for what kicks, turns or glides |
| Asset Library, Physics lab | New group **Sensors and signals**: Goal frame, Signal lamp (24 designs) |
| Demos | `run.ps1 -Demo own` (7 beats; the newest, so also `-Demo` alone) |

Not changed: the bar, the toolbar, World outline (a thing's own physics is not shown there), the keys.

## 13za. What changed in mechanisms (2026-10-14, amendment 7al)

| Where | What changed |
|---|---|
| Inspector (F7), a thing put down that is loose | New rows under its own lines (8.5): **its design also says**, **joined: …**, the joint's sort, what it is joined to (a list of what is near), its axis, where it holds, a motor, a stiffness |
| View > Colliders | Also rooms that blow (magenta) and joints (white) |
| Object text (F6) and one thing's own lines | New words `joins`, `blows`, `carries`, `hits`, `by`; `when` also holds for a joint's motor, `blows` and `carries` |
| Makers, renaming a thing | What was joined to it by its old name is joined to it by the new, in the same edit |
| Asset Library, Physics lab | New group **Mechanisms**: conveyor belt, parcel, chime tray, fan, updraft pad, paddle wheel, wheel stand, swinging link, light crate, sliding gate, weight on a spring (35 designs) |
| Demos | `run.ps1 -Demo mech` (7 beats; the newest, so also `-Demo` alone) |

Not changed: the bar, the toolbar, the windows, the keys. World outline does not show joints.

## 13zb. What changed in signals (2026-10-14, amendment 7am)

| Where | What changed |
|---|---|
| Window > **Signals** (Ctrl+K) | New window (8.20): every fact, what writes it and what goes by it, what is wrong, the wiring drawn, a thing picked from it; values and what holds while playing |
| Inspector (F7), Physics of this one | New **signals: …** line and **Signals…**; for a joint, **pick…** (what it is joined to, clicked in the view) and **click on it…** (where it holds) |
| Object text (F6) and one thing's own lines | `when` takes a condition: `NAME N` (at least N), `NAME >= <= > < = N`, `not NAME`, `… for S`; new words `clears NAME…` and `ticks NAME S`. Refused with the reason when wrong, of a design and of one thing |
| Conditions of texts (sequences, talks, quests: `fact NAME`) | Take the same comparisons: `fact ring 3` |
| Asset Library, Physics lab, Sensors and signals | New: swinging gate, timer post, reset post (38 designs) |
| Keys and mouse | The line for Ctrl+K |
| A controller | Heard only while the editor's or the game's window is in front (the user's decision, the same day) |
| Demos | `run.ps1 -Demo signals` (7 beats; the newest, so also `-Demo` alone) |

Not changed: the bar, the toolbar, the other windows. The Signals window is not on the bar
while playing (the row is full): Window > Signals.

## 13zc. What changed after signals: how hard, and rows to their rules (2026-10-14, amendment 7an)

| Where | What changed |
|---|---|
| Object text (F6) and one thing's own lines | New word `feels NAME [S]`: the world's NAME is how hard (m/s, to a tenth) the last strike at S or more (0.5) came. Refused with the reason when wrong |
| Inspector (F7), Physics of this one | The **signals: …** line says `blow (feels)`; the tooltip names `feels` |
| Window > Signals (8.20) | A `feels` row: "how hard it was last struck, m/s"; a count asked of it is never said to be out of reach. A text's row: the button is **Open the text**, which opens F10 at that line |
| The texts' window (F10) | Opened at a line (from Signals, or Who can do what), the line is chosen and scrolled to the middle once the window is laid out (before, the caret moved and the view stayed at the top) |
| Asset Library, Physics lab, Sensors and signals | New: strength pad (`feels blow 1`; 39 designs) |
| Demos | `run.ps1 -Demo signals` (8 beats: a strength tester struck soft and hard) |

Not changed: the bar, the toolbar, the other windows, the keys.

## 13zd. What changed in plants grown and colours varied (2026-10-15, amendment 7ao)

| Where | What changed |
|---|---|
| Toolbar, Plants list | New entry under the Tree Generator: **Plant Generator: grass, flowers, ferns, crops, berry bushes, fruit** (opens the same window on Flower; with bamboo since 7ap, 13ze) |
| The Generator window (`grow.gd`, id `grow`) | A row of **family buttons** across the top (Tree, Grass, Flower, Fern, Crop, Berry bush, Fruiting plant); the window's title follows: "Tree Generator", or "Plant Generator: Flower". The pictures are the chosen family's presets and grown designs. Below the numbers: **colours vary** (the shared control, below), **show a mix** (five side by side, the first in its own colours, the rest as painted things draw), **Set it on 'NAME'** (a saved design's variation), **Another mix** |
| The shared control `varybox.gd` ("colours vary") | The variation as words (editable; Enter sets it); a **role** (foliage, bloom, fruit, wood, walls, roof, stone, own, all) with **of** (percent of things), **hue ±**, **light ±**, **×** (role off); **some have**: the style's colours of the role to click (blooms, fruits, greens, roofs) |
| World outline (Makers), a forest, meadow or painted maker | New part **Colour variation** (folds): the control, what its group says under it, **Another mix**, **As its designs say** |
| World outline, one thing picked (forest, meadow, painted, or put down by itself) | **Its colours: …** in words, and **Another colour**, **Its design's own colours**, **Keep these colours** / **Back to the mix**; for a design that varies blooms or fruit, **its bloom:** / **its fruit:** swatches |
| Objects (Designs), a group | The group's **colours vary** control under what is said of it |
| Object text (F6) | New kind word `vary ROLE ...` (written by the Generator; editable) |
| Style files | New word `fruits C...` (the style's fruit colours); `fruit1..` usable wherever a colour is named |
| Demos and showcases | `run.ps1 -Demo nature` (5 beats); `editor\showcase.ps1 -Which nature` builds `editor\showcase-nature.json` |

Not changed: the bar, the other windows, the keys. The window id stays `grow` and the
Tree Generator's entry and title stay as they were (renaming waits on the review).

## 13ze. What changed after the plants: walked through, bamboo, drawn lighter (2026-10-15, amendment 7ap)

| Where | What changed |
|---|---|
| Toolbar, Plants list | The Plant Generator entry reads **Plant Generator: grass, flowers, ferns, crops, berry bushes, bamboo, fruit** |
| The Generator window (`grow`) | A family button **Bamboo** (between Berry bush and Fruiting plant); the Tree family's pictures have six more presets (Baobab, Dragon tree, Joshua tree, Flat-topped acacia, Cherry blossom, Twisted tree) and two more numbers (Trunk: **swell**; Foliage: **tufts** as a fifth foliage); a tree's **leaf** and **second** colours list the style's blooms after its greens; Flower's **form** has trumpet, drooping, rose, thistle, and five more presets (Lily, Bluebell, Rose, Peony, Thistle). Asked for a family its host does not grow, it says the host is older than the editor (it showed trees before) |
| Object text (F6) | `solid wade x y z w h d M`: a box gone through, not walked into, M slower; the Generator writes it for what grows low and thick. The inspector says it as "gone through N% slower" |
| Playing | The walker, and anyone, goes slower through a field or tall grass; what grows bends away from the walker and the four others nearest, and rises again after them. Nothing to set |
| Drawing | What grows is drawn lighter from farther off (fewer, bigger pieces). Nothing to set |
| `run.ps1` | Builds the host first when its sources are newer than it (and warns when an open editor holds it) |
| Demos, tours, showcases | `-Demo nature` has 7 beats (bamboo and stranger trees; walked through); `-Tour -Only acres` (new); the nature showcase gained a bamboo garden, a rose bed, stranger trees, thistles and bluebells |

Not changed: the bar, the other windows, the keys, the window ids.

## 13zf. A field that fills an outline (2026-10-15, amendment 7aq)

| Where | What changed |
|---|---|
| Toolbar, Plants list | New entry **Add a field: click its corners on the land; rows of a crop fill it…** (greyed with no land). It begins the line tool; Enter (or the row's end button) closes the outline and adds the field; with no crop in the world, wheat is grown first |
| World outline | New maker **Field** (Add…, under Growing things); its parts: **How the field is laid out** (a patch every (m) along a row, rows apart (m), rows turned from the first side, kept in from the edge, patches left out, keeps off roads, seed), **What it is made of**, **Colour variation**; its outline is a **Line** named "field outline" |
| Demos and tours | `-Demo field` (4 beats); `-Tour -Only field` |

Not changed: the bar, the keys, the other windows.

## 13zg. Signals finished: several joints a thing, what waits kept in a saved game (2026-10-15, amendment 7ar)

| Where | What changed |
|---|---|
| Inspector (F7), Physics of this one, a loose thing | New **joint N of M** chooser and **+ another joint** (8.5); the joint rows below change the one chosen |
| Object text (F6) and one thing's own lines | `joins` may be said more than once (a line each); one thing's own joints replace its design's all together; `no joins` takes them all |
| View > Colliders | Every joint of a thing drawn, not its first only |
| Signals window | The fact `waits` (what a saved game keeps of what waits) is named as the engine's, "a saved game (what waits)" |
| Asset Library > Mechanisms | **Bridge plank** and **Bridge post** (41 designs in the physics lab) |
| Saved game (F5 / F9) | Also keeps how long each condition has held (`for`) and how far each `ticks` is to its next; loaded, they go on from there |
| Demos and tours | `-Demo signals` (10 beats: new "Held at both ends", "Kept in a saved game"); `-Tour -Only bridge` |

Not changed: the bar, the keys, the other windows.

## 13zh. Sound, first tier (2026-10-15, amendment 7as)

| Where | What changed |
|---|---|
| Window (bar, editing and playing) | New **Sounds** (8.21) |
| What is written (F10) | New text **sounds** (`sounds.txt`): the sounds there are, as recipes; Apply and it is made again |
| `weather.txt` (F10) | A weather may say `sound NAME [N]`: breeze, overcast, rain, storm and snow do |
| Object text (F6) and one thing's own lines | New words `sounds NAME`, `hums NAME [R]`, `knocks NAME [S]` (said once; may be one thing's own; `no` them) |
| Inspector, its design also says | Shows a design's sound lines with its other words |
| Signals window | Says what `sounds` and `hums` do by a `when`; counts only things with a row |
| Asset Library > Physics lab | New group **Sound**: **Sound post**; lamps, the gate, chime, balls, crates, the strength pad, bridge planks and belts given sounds (42 designs) |
| Playing | The world is heard: what sounds, hums and knocks near the eye, and the weather. Editing is quiet |
| Demos and tours | `-Demo sound` (7 beats); `-Tour -Only sound`; tours are quiet |

Not changed: the bar's headings, the keys, the toolbar.

## 13zi. The Character Studio, first tier (2026-10-15, amendment 7at)

| Where | What changed |
|---|---|
| Characters (bar) | New **Character Studio…** (8.22) |
| Window (bar) | New **Character Studio** among the windows |
| What is written (F10) | New text **characters** (`characters.txt`): libraries, maps, characters, looks |
| `moves.txt` (F10) | New words: a phase `strikes PLACE R`, a move `upon struck`; new sets `samurai`, `folk_struck`; new moves slash, thrust, draw, struck |
| Playing as the samurai | The dash button (F, a click, B) cuts, at a run thrusts; greet (G) draws; the rest as anyone |
| Opening a world | Its texts are read at once (Play straight after plays by them) |
| Tours, `-Demo -Check`, measuring | The pointer is never captured, no real controller is heard, the window does not take focus |
| Demos and tours | `-Demo studio` (8 beats); `-Tour -Only studio`; `chartest.gd` headless |
| Then, the same day | Quaternius's library (CC0) in characters.txt: characters **quaternius** and **mannequin_ual** in the Studio's list; a look row may say `part FROM TO`, shown in the Movement tab's How column; the preview's **Front** and **Back** put the right way round, and it opens on the face |

Not changed: the bar's headings, the keys, the toolbar.

## 13zj. The Character Studio, authoring (2026-10-15, amendment 7au)

| Where | What changed |
|---|---|
| Studio, under the preview | New **Someone to strike** (tick), how far before it (m) and aside (m); a **timeline** of the move tried; a line of what the body is doing |
| Studio, preview | Shows a body of its own doing what is tried (moved by the game's motor), the blade's path, how far round it strikes; the eye follows it, from the side until moved by hand |
| Studio, Movement | Clicking a row of the ways of moving does it; clicking a move tries it; clicking a phase also opens it in the new tab; each list has a line saying what it is |
| Studio, new tab **Change a phase** | How it looks: Show it with the clip chosen, Part of the clip (from, to; Begin where the slider is, End where it is, The whole clip), The other way about, Fade in, a pose (a place of the body; tip, twist, lean). What the body does: Lasts, Goes forward, Turns round, Strikes with … within. Try it, Save as a new move, Open the move |
| Studio, Clips | A clip's text says which bones it was made for; mirrored copies are not listed |
| `moves.txt` (F10) | New phase word `spins DEG`; a strike is once a phase; new move `dash_cut` in the set `samurai` |
| `characters.txt` (F10) | A look row may say `mirror` and `bend BONE x y z`; `look samurai` has rows for `dash_cut` |

Not changed: the bar, the keys, the toolbar, the other tabs.

## 13zk. The Character Studio made reliable; combos (2026-10-15, amendment 7av)

| Where | What changed |
|---|---|
| Studio, under the preview | New row: **Play** (Pause), **Replay**, **Stop**, **Loop**, Speed with its value, what is playing; the Clips tab's Play, "Round and round" and Speed moved here |
| Studio, preview | The move's written path on the floor; a white ball at the end of the phase being changed, dragged to set how far it goes; someone to strike put before the move as it is asked |
| Studio, Movement | Moves list: new column **Ready to see**; a phase with no clip says so, with a clip that might do; new **Its clips only** |
| Studio, **Change a phase** | Rebuilt: phase N of M and what it does; Previous phase, Next phase, Play this phase, Play the whole move, Whole move clips only; Use the clip chosen / Use (a suggested clip); sliders with numbers, units and Reset for clip begins/ends, tip/twist/lean, lasts, goes forward, turns round; The whole clip; **Backwards**; Lead with the other side; Reset this part, Reset all; Strikes with; **More** (blend in, strike reaches, clip slider's place, the lines as written, Save as a new move, Open the move's text, Open the look's text) |
| Studio, new tab **Combos** (between Change a phase and Holds) | Saved combos (Combo, Moves, Ready to see); Add (a move, asked so many s after its window opens), Remove the last, Clear, Play the combo, Save the combo; what each link does; Let X cut into the end of Y |
| `moves.txt` (F10) | New entry `combo NAME` with `then MOVE [wait S]`; set and moves of the swordsman; combos cut_thrust, dodge_counter, three_cuts, dash_and_cut |
| `characters.txt` (F10) | Look word `backwards`; library and map `ual2`; character `swordsman`; look `swordsman` |

Not changed: the bar, the keys, the toolbar, the other windows.

## 13zl. The Character Studio's playback made explicit; its panels steadied; one brush agreed on (2026-10-15, amendment 7aw)

| Where | What changed |
|---|---|
| Studio, Play | Plays **the one thing chosen last**, in whichever list: a clip, a way, a move, a phase, a move's clips, a combo. Choosing something stops whatever played before it; choosing a clip shows it held at its start (Play plays it); a move, a way or a combo clicked plays at once, as before. Its tooltip says what it plays |
| Studio, the lists | Only the row of what Play plays is shown chosen; rows chosen in other lists are let go (so a click on one is always a choice) |
| Studio, under the preview | Two lines in fixed places, each one line (cut short with "…", the whole in a tooltip): **Play: what · playing / paused / stopped / finished / cannot play: why · once or looped · the phase now**; and what the body did. Nothing under them moves as they change |
| Studio, the preview and the tabs | Split by a line that is dragged to give either more room (`VSplitContainer`); the preview is smaller to begin with (it grows with the window) |
| Studio, the timeline (coloured bar) | A phase clicked opens it in Change a phase; the phase being changed is outlined in yellow; hovering a phase says what it does |
| Studio, **Change a phase** | **See it at** (slider): the phase's clip held at any moment of it, the time said; Play goes on from there. The controls in three parts that fold (Godot's `FoldableContainer`): **Look** (this character's animation only), **Timing** (the move, everyone with it), **Movement** (where the body goes, turns, strikes; the move, everyone with it); **More** folded |
| Studio, Movement and Combos | The paragraphs at their top made one line each; the rest in their tooltips |
| Studio, Movement | **Show it with the clip chosen** says which row and which clip ("Show 'stand' with Sword_Idle"); it changes the row of the look last clicked |
| Studio, someone to strike | Left at aside 0, for a move that dashes farther than they stand (2.5 m) they stand 0.7 m to its left, to be passed and cut |
| Objects | A row clicked while the paint brush is in hand: the brush paints with it from then on (said); one that cannot be painted puts the brush down (said why). Clearing and replacing keep what they were working on |
| Plant Generator | **Paint with it…** is pressed while it paints this plant ("Painting with 'X' (Esc ends it)"), and pressed again puts it down; another plant shown, or its numbers changed, puts its brush down (said); a line under it says what a brush taken up in Objects paints |
| `moves.txt` | `dash_slash`: a new phase **turn** (0.25 s, 180°) between the dash and the recovery, which lets cut_b in at once |
| `characters.txt` | look `swordsman`: `dash_slash/turn` and `dash_slash/recover` rows (the clip's part split at 0.62 s) |

Not changed: the bar, the keys, the toolbar, the other windows.

## 13zm. The Character Studio's workflow: tabs in the order of the work, one Edit, Save / Save as a copy / Discard, Where (2026-10-15, amendment 7ax)

| Where | What changed |
|---|---|
| Studio, the tabs | In this order: **Moves**, **Combos**, **Edit**, **Ways of moving**, then **Clips**, **Strikes, reactions, the rest**, **Holds**, **Body and bones**. Movement is split into Moves and Ways of moving; **Change a phase** is now **Edit** |
| Studio, every list | A row clicked is acted on just after the click (a combo row clicked while another played used to leave the other playing) |
| Studio, **Moves** | The set; **Add to the set** (any move of moves.txt the set has not) and **Take out**; the moves; **Edit this move** (or double-click a row), **New move**, **Play it**, **Its clips only**, **Open the set** |
| Studio, **Combos** | The list; **Edit this combo** (or double-click), **New combo**, **Play it**. Add, Remove the last, Clear, Play the combo and Save the combo moved into Edit |
| Studio, **Edit**, top | What is edited ("Editing the move 'cut_a' — the sets … have it", "Editing the combo 'x' — its move 2 of 3"); a line saying changed and not saved, or not; **Save**, **Discard changes**, **Save as a copy** with a name field, **Done**. Done or another thing opened with changes asks: Save, Discard, Keep editing |
| Studio, **Edit**, a combo | Its moves in order (a list), one chosen; **Play the whole combo**, **Play this move alone**, **Earlier**, **Later**, **Remove it**; **Asked … s after the move before lets it in**; **Add** a move **at the end** |
| Studio, **Edit**, a move | Its **Phases** in a row, one chosen; **Add a phase after it** (name field), **Remove this phase**; then the phase's Look / Timing / Movement / More, as in 13zl. More no longer has **Save as a new move** (Save as a copy at the top does it) |
| Studio, **Edit**, Look | **Choose a clip…**: the character's clips in a window of their own, found by name or word, each played in the preview as it is chosen; **Use it** or a double-click. **Use the clip chosen under Clips** stays beside it |
| Studio, **Ways of moving** | **Choose a clip for it…** (the same window) beside Show it with the clip chosen, Copy the look, Open the look |
| Studio, the timeline | A phase clicked opens it in Edit; a combo playing opens as the combo, at that move. While a combo is edited the whole combo is drawn and only the chosen move's phase outlined |
| Studio, under the preview | **Where**: a slider, the time ("0.94 / 1.88 s"), ◀ and ▶ (a thirtieth of a second); for whatever Play plays. A move or a combo is played again from its beginning to that moment |

Not changed: the bar, the keys, the other windows; the Look / Timing / Movement / More
controls themselves.

## 13zn. Mixamo's clips taken in: Import clips… (2026-10-15, amendment 7ay)

| Where | What changed |
|---|---|
| Studio, **Clips**, beside Find and the tags | **Import clips...**: Godot's own file window (inside the editor's), starting in Downloads. FBX or glTF files chosen and **Import the files chosen**, or **Import the whole folder**. Copied into `characters/NAME/`, written as the library NAME (rig found from the bones, from, licence, tags), added to the chosen character's clips; Ctrl+Z takes it back |
| Studio, **Read again** | Reads FBX files and folders of them again too |
| characters.txt | `library`'s `file` may be a folder (ending in /); `map mixamo` |

Not changed: the bar, the keys, the other windows, the other tabs.

## 13zo. The Character Studio: what plays stays in sight; buttons by colour; what presses each move (2026-10-15, amendment 7az)

| Where | What changed |
|---|---|
| Studio, top row | **Side by side** (new): the preview and its controls left, the tabs right, the window wider; off, as before. **Make a copy** is now **Copy**, **Open the text** is **Its text** |
| Studio, **Edit**, pinned at its top | What is edited, whether it is changed, **Save**, **Discard changes**, **Save as a copy**, **Done**; a combo's moves with **Play the whole combo**, **Play this move alone**, **Earlier**, **Later**, **Remove it**; the move's **Phases**; **◀ Phase**, **Phase ▶** (were Previous phase, Next phase), **Play this phase**, **Play the whole move**, **Whole move, clips only**. They stay in place while the rest scrolls |
| Studio, **Edit**, scrolled | **Asked with** (new): the button the move is pressed with in the game, or "by name only", and what that means; **Add a phase after it**, **Remove this phase**; a combo's **Asked … s after**, **Add**, what it says, and **In the game** (new: how a player presses it, step by step); then Look, Timing, Movement, More as before |
| Studio, **Moves** | The list has an **Asked with** column (orange: an earlier move with that button always wins, or nothing presses it). **Earlier in the set**, **Later in the set**, **The game's buttons** (a window: each button, what presses it, which of this character's moves it asks) |
| Studio, buttons | Coloured by what they do, in the workspace's colours: play green, edit blue, keep gold, undo or take away orange. Only those named in `CUE_OF` |

Not changed: the bar, the keys, the other windows, the other tabs.

## 13zp. The Character Studio: Buttons and Gear tabs; a phase's numbers in two columns (2026-10-16, amendment 7ba)

**Tabs, in order:** Moves, Combos, Edit, **Buttons** (new), **Gear** (new; Holds folded into it), **Make** (new in 7bf), Ways of moving, Clips, Strikes, reactions, the rest, Body and bones.

**Bar:** Characters gains *Buttons: keyboard and controller, moves on each…* and *Gear: weapons, things worn and carried…*. Each opens the Studio on that tab.

**Buttons tab** (`buttonstab.gd`):

| Control | Name | Does |
|---|---|---|
| Keyboard and mouse / Controller | `show_keys`, `show_pad` | which device's ways are shown and changed; the other kept as it is |
| list of buttons | `bindings` | Playing (on foot), Your own buttons, then The eye, At a puzzle, The game (folded); what presses each (• where you changed it, orange where another button shares it, the other's name in the tooltip); the moves it asks for this character |
| Change… / Add another… | `bind_change`, `bind_add` | the next key, click or controller button pressed is taken; Esc leaves it |
| Nothing / As Draw3D has it | `bind_clear`, `bind_reset` | none on this device / the built-in table's |
| moves on the button | `on_button` | this character's moves and combos asked with it, in the order tried |
| Only when, held | `when_pick`, `holding_pick` | the move's `when` (any time, on the ground, in the air, falling, crouched, crouched on the ground, running, standing still, moving), plus another button held |
| Earlier / Later / Take it off | `on_earlier`, `on_later`, `on_take_off` | its place in the set; out of the set |
| Plays by itself | `plays_by_itself` | a combo: one press plays it all |
| Put on this button | `put_pick`, `put_on` | any move or combo asked with this button, and in the set |
| Make a button / Remove it | `new_button_name`, `new_button_says`, `button_make`, `button_remove` | a button of your own |
| Save as a profile / Use this profile | `profile_name`, `profile_save`, `profiles`, `profile_load` | buttons and this set, to and from Draw3D's folder |

**Changed in 7bi (2026-10-16):** the game's own screens, apart from the editor's windows (`gameui.gd`).

| Where | Control | Name | Does |
|---|---|---|---|
| playing | quest step, goods (top right) | `hud_quest`, `hud_goods` | what the walker is on and has |
| playing | the things found, numbered (bottom left) | `hud_bar` | keys 1 to 9; gold in hand, green worn |
| playing | In hand / Put away | `hud_hand` | what is in hand, and whether drawn |
| playing | I inventory, Esc menu | `hud_hint` | the two screens' keys |
| Inventory (I, Back) | On you / In the pack | `inventory_rows` | a row per thing; Enter or A does the likeliest |
| Inventory | Put on (Hold), Take in hand, Take off, Use, Drop, Close | `inv_on`, `inv_hand`, `inv_off`, `inv_use`, `inv_drop`, `inv_close` | what is done with the one chosen |
| Inventory | what it is | `inventory_about`, `inventory_goods` | the thing chosen; goods |
| Menu (Esc, Start) | Go on, Inventory, Journal, Controls, Save the game, Load the saved game, Back to editing / Leave the game | `menu_on`, `menu_inventory`, `menu_journal`, `menu_controls`, `menu_save`, `menu_load`, `menu_leave` | the game held |
| the bar, playing | Stop: back to editing (Tab) | `stop` | was "(Tab or Esc)": Esc is the menu now |
| Library | Gear to find | — | a shelf of 18 designs, pictured as their gear |

**Changed in 7bh (2026-10-16):**

| Where | Control | Name | Does |
|---|---|---|---|
| everywhere | every box not ticked | — | drawn light, seen on the dark windows (`section.gd` `boxes_seen`) |
| Studio, Buttons | Lock on only while held | `lock_held` | shortened (was "Lock on only while the lock button is held (off: press, and press again)") |
| Studio, Buttons | Right stick: mirror left/right, mirror up/down | `mirror_x`, `mirror_y` | the eye turned the other way, in the game and the yard |
| Studio, Gear | Carried | `gear_carried` | one at a time / with the one in hand / always where it is |
| Studio, Gear | Keep for this model, Model's fit | `gear_fit_keep`, `gear_fit_use` | the nudge and turn kept for this model in every world (fits.txt) |
| Studio, Moves | draws and puts away: … | `draws_what` | what Draw and Sheathe act on |

**Changed in 7bg (2026-10-16):**

| Where | Control | Name | Does |
|---|---|---|---|
| Studio, the character's row | Rename | `rename_character`, `rename_field` | the character called by the name typed beside (Enter) |
| Studio, the character's row | Make… | `make` | the Make panel in place of the tabs (it is no longer a tab) |
| Studio, the character's row | Save world ● N / World saved | `save_world` | how many changes are not in the world's file; writes it (Ctrl+S) |
| Studio, the yard | the pad, sticks, arrow keys | — | the character's only; the right stick turns the eye |
| Studio, Moves | Draws it by itself first, with | `draw_first`, `draw_by` | the move beside it greyed while off |
| Studio, under the preview | status | `live_text` | NOT BEGUN says what the move wanted and what to tick |
| Studio, Edit | Phases | `phases` | numbered chips in the phases' colour, the chosen one outlined |
| Studio, Edit | Versions (N) | `versions`, `versions_remove` | each Save keeps one (versions.txt); one chosen gone back to; forget one |
| Studio, Moves | the list | `moves` | *open in Edit, saved* / *changes not saved*; nothing after Done |
| Studio, Make | ↺ beside each slider | `make_reset_size` … `make_reset_legs` | that part as its model has it |
| Studio, Make | Reset the whole body | `make_reset_body` | was *As the model is* |
| Studio, Buttons | profiles list | `profiles` | one chosen says how its buttons and moves differ |
| Studio, Buttons | and its moves | `profile_moves` | Use writes the profile's moves too (off: buttons only) |
| Studio, Gear | Turn round, Roll 90°, Tip 90° | `gear_turn_round`, `gear_roll`, `gear_tip` | the thing turned a half or a quarter, in one press |

**Changed in 7bf (2026-10-16): the Make tab** (`maketab.gd`, scrolls; after Gear). Each choice is a line of the character in characters.txt (Ctrl+Z, Ctrl+S):

| Where | Control | Name | Does |
|---|---|---|---|
| Make, top | about | `make_about` | every choice is a line; Copy first to make one's own |
| Make, Start from | Model | `make_model` | the models there are (each character's model, each library file with a skinned mesh): file, bones, whose |
| Make, Start from | Use this model / Copy with this model | `make_use_model`, `make_copy_model` | `model` and `rig` written; or a new character first, then those |
| Make, Body | Height, Build, Head size, Arm length, Leg length | `make_size`, `make_build`, `make_head`, `make_arms`, `make_legs` | sliders with their numbers: `size`, `shape build/head/arms/legs S`; seen as dragged, written 0.35 s after, as one edit |
| Make, Body | As the model is | `make_reset_body` | every `shape` line and `size` taken out |
| Make, Body | line under | `make_shape_text` | `shape` lines of single bones written by hand; how much longer the legs and stride are |
| Make, Colours | Surface, colour, No tint | `make_surface`, `make_colour`, `make_no_tint` | all of it or one material by name; `colour SURFACE r g b` (seen as chosen, written after); the line taken out |
| Make, Outfits | the list | `make_outfits` | each outfit and its gear, the first "starts in it"; a click puts it on the figure |
| Make, Outfits | Next outfit | `make_next_outfit` | the next outfit on the figure while what plays goes on (a move, a combo, the yard); nothing written |
| Make, Outfits | New outfit / Copy this one / Gear… | `make_new_outfit`, `make_copy_outfit`, `make_gear` | an empty outfit (or the worn one's gear), worn; the Gear tab |
| Make, Outfits | Name, Rename / Starts in this / Remove it | `make_outfit_name`, `make_rename_outfit`, `make_first_outfit`, `make_remove_outfit` | the outfit worn renamed, made the first, taken out with its gear |
| Gear | what it wears | `worn` | what it wears now: every outfit's and the worn outfit's; "(every outfit)" by a thing written before the first outfit; Put it on puts into the outfit worn |

**Changed in 7be (2026-10-16):**

| Where | Control | Name | Does |
|---|---|---|---|
| Studio, under the preview | Yard | `yard` | the training yard: the character steered with the game's buttons among the dummies of yard.txt; Esc or Yard again leaves |
| Studio, under the preview | the strip | `timeline` | a clip or a way shows its clip (no phases); the yard shows the move pressed |
| Studio, under the preview | status | `playing_text` | says "not the move edited" while something else plays; the banner dimmed then |
| Studio, Moves | Draws by itself, with | `draw_first`, `draw_by` | the set's `draws_by`: a move that wants it drawn draws it first |
| Studio, Moves | the list | `moves` | the move edited says *✎ editing*; Asked with says "spear in hand", "locked on" |
| Studio, Clips | sort | `clip_sort` | by name (whatever the capitals), by library, by seconds; a column's title too |
| Studio, Edit | phases | `phase_add_before`, `phase_add` | **+ Before**, **+ After** the phase chosen |
| Studio, Edit | beside Button | `hand_pick`, `locked_box` | what must be in the hand (`wields`); only locked on (`locked`) |
| Studio, Edit | In hand (Goes) | `phase_hand` | the phase draws it, puts it away, or takes another thing it carries |
| Studio, Edit (combo) | Only with in hand | `combo_hand`, `combo_locked` | the combo's `when`: a thing in the hand; locked on |
| Buttons | Always | `bindings` | *lock* listed (Y, middle click) |
| Buttons | Lock on only while held | `lock_held` | held, or press and press again |
| Gear | Cloth | `gear_swings` | the thing worn swings after the body (`sways`), or is stiff |
| The game | lock | — | Y or the middle click: locked on to the one nearest before the eye; the eye comes round |

**Changed in 7bd (2026-10-16):**

| Where | Control | Name | Does |
|---|---|---|---|
| Studio, above the tabs | edit banner | `edit_banner`, `edit_kind`, `edit_name`, `edit_rename`, `edit_save`, `edit_discard`, `edit_copy`, `edit_done`, `edit_state` | while a move or combo is edited, on every tab: EDITING MOVE/COMBO, its name (typed over and Enter, or Rename: renamed everywhere it is named), Save, Discard (was Discard changes), Copy (was Save as a copy; its name under Edit's Details), Done, saved or "● not saved". Edit's own head now only says what to do when nothing is edited; *Show Edit* is gone |
| Studio, Moves | the list | `moves` | each move folded until its arrow is clicked; the one edited unfolded and tinted |
| Studio, under the preview | status | `status` | what plays and how it goes on one line (were two) |
| Buttons, top | profile row | `profile_now`, `profile_save_over`, `profile_name`, `profile_save`, `profiles`, `profile_load` | Profile: NAME (the one in use); **Save** over it; **Save as new** (wants a name); a list and **Use** (were *Save as a profile*, *Use this profile* under the list) |
| Buttons, right | the button chosen | `chosen_button`, `button_call`, `button_rename` | Button: NAME; Rename: a button of one's own renamed with its moves; one of Draw3D's shown by the word typed (`label`), empty for its own name |
| Buttons, right | click box | `click_pad` | shown while a key is waited for: a mouse button is taken only here; a click elsewhere cancels |
| Buttons, list | headings | `bindings` | Asked by NAME's moves, Your own buttons, Always (moving about, the world), Not used by NAME (folded), The eye, At a puzzle, The game |
| Gear | buttons | `gear_in_hand` | **The one in hand**: of what it carries, the one chosen is drawn; the others `spare` |
| Gear | Gear there is | `gear_library` | now in columns, as tall as they are (✓: it has it); double-click puts it on |

**Gear tab** (`geartab.gd`, scrolls):

| Control | Name | Does |
|---|---|---|
| what it wears and carries | `worn` | thing, sort, where, put away at |
| Draw / put away | `gear_draw` | by its move if its set has one (draw_blade, sheathe_blade), else at once |
| Take it off / Its text | `gear_take_off`, `gear_text` | |
| Where / Put away at | `gear_place`, `gear_stow` | a place of `gear.txt` by name |
| Held in both hands (IK) | `gear_two_hands` | a thing of two hands: ticked, held before the body where gear.txt says (`two_hands_at`) and both hands brought to its grips by IK while drawn (the list says "both hands, PLACE"); unticked, `free_hand`: in the one hand (7bb, 7bc) |
| Nudge (cm), Turn (°), Reset | `gear_nudge_x/y/z`, `gear_turn_x/y/z`, `gear_reset` | the character's own adjustment where it is now (held or put away); seen at once, written after 0.35 s |
| Gear there is, Put it on | `gear_sort`, `gear_library`, `gear_put_on` | every thing by sort; a double-click puts it on |
| about | `gear_about` | what it is, hands, places, reach, weight, points, made of, from and licence; for two hands, the other hand's distance from its grip |

**Edit tab:**

- **Pinned:** Phases now has + Phase, its name and − Phase. *Whole move, clips only* is now *Clips only*.
- **Below the pinned part:**
  - See it at;
  - two columns. **Look** (`look_section`): what shows it, Choose a clip…, From Clips tab, Clip part from and to, Whole clip, Backwards, Mirrored, Button. **Timing and movement** (`movement_section`): Lasts, Blend in, Goes, Turns, Strikes;
  - Adjust pose: part, tip, twist, lean, ↺ part, ↺ all;
  - Details, folded: the explanations, the clip's seconds, the text.
- **Moves tab:** *The game's buttons* is now *Buttons…*, which opens the Buttons tab.

**Side by side:** the preview keeps its least width, and someone to strike moves to a row under Front / Side / Back / Above.

## 13zz32. Leaves under trees, snow with depth, rooftops and streets (2026-10-21, amendment 7dd)

| Where | Control | Name | Does |
|---|---|---|---|
| The room's bottom right, over the frame rate | the season; the weather | `season_cycle`, `weather_cycle` | a click: the next season (spring, summer, autumn, winter) or weather; a right click: the one before. Under every window |
| Environment window, What lies | Snow: a dusting, ankle deep, deep, knee deep | `deep_0.05`, `deep_0.22`, `deep_0.45`, `deep_0.70` | how deep snow lies at its fullest |
| The bar, Town | Power line | `cb_powerline` | a power line dragged along a street |
| The bar, Build | Raise | `cb_platform_brush` | a platform painted where the brush goes, one with any it meets |
| The view, over a power line | shorter, taller poles; how worn | `by_powerline_lower`, `by_powerline_higher`; `by_wear` | how tall its poles are; worn, they lean |
| The view, over a wall | its top: coping, battlements, plain | `by_wall_top_usual`, `by_wall_top_battlements`, `by_wall_top_plain` | what is along its top; bare where a wall stands on it |
| The view, over a patch of designs of the library (decor, crops) | still, gently, freely in the wind | `by_<brush>_wind_still`, `by_<brush>_wind_gently`, `by_<brush>_wind_freely` | each design's own way of moving in the wind, one step |
| The bar, Build > Windows | a click at the foot of a wall | | the door there, where it was clicked |

## 13zz31. Weather that lies, a town, platforms, windows by hand (2026-10-21, amendment 7dc)

| Where | Control | Name | Does |
|---|---|---|---|
| View menu | Frame rate, in the corner | (menu entry), Ctrl+Shift+F | frames a second and the slowest frame, bottom right, green, yellow or red; kept on this machine (`view.cfg`) |
| The view, over a wall, fence or hedge, or a gate or arch of it clicked | its gates (opens by itself when a gate or an arch is clicked) | `by_gates` | opens its gates' row under the strip |
| Under the strip (gates) | round, pointed, low, flat arches; open, a gate, a door, a grille in them; boards, pickets, farm, iron, two leaves; gates' colours; arches' stone | `by_arch_form_<form>`, `by_arch_fill_<what>`, `by_gate_form_<sort>`, `by_gate_colour_<hex>`, `by_arch_colour_<hex>` | how its arches and gates are made |
| The view, over a wall, fence, hedge, path, yard or garden; and a house's details | how worn | `by_wear`, then `by_wear_0`, `by_wear_3`, `by_wear_6`, `by_wear_9` | as new, a little worn, worn, falling apart |
| The view, over a path | up steep ground (fold); over water (fold) | `fold_steep`, `by_climb_form_<form>`; `fold_over_water`, `by_water_form_<form>` | steps of stone, wood, logs, a ramp; stepping stones, a boardwalk, a stone bridge, a footbridge |
| A house's details fold | its front; windows: by bays again | `by_front`, `by_front_<sort>`, `by_awning_colour_<hex>`, `by_sign_colour_<hex>`, `by_neon_colour_<hex>`; `by_windows_bays` | a shop's, a bar's front, garage doors, as the floors over it; their colours; windows put by hand taken off |
| Under the strip (windows) | also sash, factory, modern | `by_window_form_sash` and the rest | how its windows are made |
| The view, over a house | lift; lower | `by_lift_up`, `by_lift_down` | lifted off the ground, floating; down again |
| The view, over a platform | stone, grass edge; lower, higher; sharper, softer | `by_edge_<edge>`, `by_platform_lower`, `by_platform_higher`, `by_platform_sharper`, `by_platform_softer` | how high it is raised and how its edge is made |
| The bar at the right | Town: shop, apartments, townhouse, loft, bar | `cb_town`, `cb_shop`, `cb_apartments`, `cb_townhouse`, `cb_loft`, `cb_bar` | a town's building dragged out as a house is |
| The bar, Build | Windows; Platform | `cb_windows`, `cb_platform` | a window clicked onto a house's wall (its door near the wall's foot); a platform clicked round |
| The bar, Paths | Paving | `cb_paving` | paving where it is dragged, one with the paving it overlaps |
| The view, over grass | its sorts | `by_grass_sorts`, `by_grass_<sort>` | lawn, meadow grass, tussocks, dandelions, dandelion clocks, daisies, bluebells in its mix |
| The view, over a patch of flowers | also dandelions, clocks | `by_flowers_dandelion`, `by_flowers_dandelion-clock` | in its mix |
| The view, over a patch of decor | also the town street, back street, night street sets; rubbish bags, pallets, cones, barriers, a pavement sign, produce, café umbrellas, food carts, vending machines, phone boxes, newspaper boxes, parking meters, strings of lights, scaffolding, graffiti, street bins, hydrants, city benches | `by_decor_set_town`, `by_decor_set_backstreet`, `by_decor_set_night`, `by_decor_<design>` | in its mix |
| Environment window (now scrolls) | what falls: as the weather has it, rain, snow, hail, leaves, petals, ash, frogs, fish, meatballs, boxes, crates, the thing picked | `falls_weather`, `falls_<name>`, `falls_picked` | what falls in every weather instead of its own |
| Environment window | how much falls: light, as written, heavy, downpour | `much_<n>` | |
| Environment window | lies: never, slowly, as usual, quickly; stays: a minute, four minutes, twenty minutes, hours; footprints: none, half a minute, a minute and a half, ten minutes | `settles_<n>`, `keeps_<s>`, `prints_<s>` | how fast what falls lies, how long it stays, how long a print lasts |
| Environment window | now: bare, snowed, soaked, leafy | `lying_bare`, `lying_snowed`, `lying_soaked`, `lying_leafy` | at once (a look) |
| Environment window | season: any, spring, summer, autumn, winter | `season_any`, `season_<word>` | the world's season, given to every patch of plants too |
| Environment window, Weather | also drizzle, blizzard, hail, leaves | `weather_<name>` | |

## 13zz30. Context and the wild edges (2026-10-21, amendment 7db)

| Where | Control | Name | Does |
|---|---|---|---|
| The view, over anything chosen | a fold (a small ▾ in its corner) | `fold_<word>` | opens a row of what it holds under the strip; again, shuts it |
| Over a house | colours; materials and roof; storeys; roofs off; details; keep | `fold_colours`, `fold_materials`, `by_lower`, `by_higher`, `by_roofs_off`, `fold_details`, `fold_keep` | the house's row, folded |
| A house's details fold | windows, door, lanterns, porch, stair, climbs, furniture, what hangs | `by_windows`... `by_furnish`, `by_hung` | as before; `by_hung` opens what hangs on its walls |
| Under the strip (what hangs) | lanterns, baskets, banners, shields, wreaths, boxes; colours; any colour; none | `by_hung_<word>`, `by_hung_tint_<hex>`, `by_hung_any`, `by_hung_none` | on every side or taken down; their colour |
| Over a wall, fence, hedge, garden or pond | what grows (fold) | `fold_grows` | what grows along its foot, what climbs it, what hangs on it |
| Over a wall | its picture: also cobblestones, dressed stone, timber, concrete, tin, rusted iron; your own | `by_texture_<pic>`, `by_texture_import` | its picture; a picture of yours brought in |
| Over a path | cobbles, flagstones, paving, gravel, earth, stepping stones, boards | `by_floor_<word>` | its floor |
| Over a garden | what its beds are planted with (fold) | `fold_beds`, `by_beds_usual`, `by_beds_<word>` | flowers of its beds, any together |
| Over a painted patch | its season (fold); stiller, moving more; its colours (fold); new patch; keep (fold) | `fold_season`, `by_season_<word>`, `by_<brush>_wind_less`, `by_<brush>_wind_more`, `fold_colours`, `by_vary_<role>`, `by_vary_<role>_<hex>`, `by_vary_<role>_any`, `fold_keep` | spring to winter; the wind; any colour of each role |
| Under a patch's sorts | your own sorts; bring in your own | `by_<brush>_own_<design>`, `by_<brush>_import` | a model of yours in its mix, or taken out |
| Over a patch of decor | on walls (fold) | `fold_on_walls`, `by_hang_<word>` | what Decorate hangs when dragged along a wall |
| Over a patch of decor or life | sets (fold) | `fold_sets`, `by_<brush>_set_<word>` | sets put in and taken out of its mix: decor park, camp, laundry, wastes; life wild, wastes, wolves |
| Over grass | its colour (fold) | `fold_grass_colour`, `by_grass_any` | as before, and any colour |
| The bar, Water | Dock | `cb_dock` | dragged from a bank over water: a jetty and a boat |
| The bar, Paths | Path | `cb_path` | dragged now, not clicked |

## 13zz29. Its details beside it (2026-10-21, amendment 7da)

| Where | Control | Name | Does |
|---|---|---|---|
| The view, over a house | windows; door; lanterns; porch; stair | `by_windows`, `by_door`, `by_lamps`, `by_porch`, `by_stairs` | open their rows under the strip |
| Under the strip (windows) | square, arched, round, slit, leaded; shutters; flowers, plain, none; colours; flowers' colours | `by_window_form_<sort>`, `by_shutters`, `by_sill_<sort>`, `by_pane_colour_<hex>`, `by_flower_colour_<hex>` | how its windows are made |
| Under the strip (door) | boards, panelled, glazed, stable, arched; colours | `by_door_form_<sort>`, `by_door_colour_<hex>` | how its doors are made |
| Under the strip (lanterns) | one, both, none; light colours | `by_lamps_<sort>`, `by_lamp_colour_<hex>` | lanterns by its doors, lit in the dark |
| Under the strip (porch) | none, front, round; balcony none, front, round | `by_porch_<sort>`, `by_balcony_<sort>` | a porch round its foot, balconies over it |
| Under the strip (stair) | flight, ladder, wound; wood; rail | `by_stair_form_<sort>`, `by_stair_wood`, `by_stair_rail` | its stair inside |
| The view, over a wall, fence, hedge, path, yard, garden or pond | its sort; its colours; its picture; towers; lower, higher; wider, narrower; deeper, shallower; what grows along it; what climbs it; a pond's life | `by_fence_form_<sort>`, `by_hedge_form_<sort>`, `by_floor_<sort>`, `by_edge_<sort>`, `by_<word>` (stone, cap, wood, green, flower, floor, water colours), `by_texture`, `by_towers`, `by_<sort>_lower`, `by_<sort>_higher`, `by_path_wider`, `by_path_narrower`, `by_pond_deeper`, `by_pond_shallower`, `by_dress_<word>`, `by_climb`, `by_pond_life` | what it is made as and how it looks |
| The view, over a patch of life | quieter; louder | `by_life_quiet`, `by_life_loud` | how loud what sings and chirps in it is |
| The view, over decor | the lit set | `by_decor_set_lit` | lamp posts by a bench and pots |

## 13zz28. Decorated and alive (2026-10-21, amendment 7cz)

| Where | Control | Name | Does |
|---|---|---|---|
| Bar at the right, Paint | Crops; Decorate; Life | `cb_crops`, `cb_decor`, `cb_life` | paint a field; clutter, or on a wall ivy, or on a room's floor furniture; small life, or over a pond its life |
| The view, a plant brush on a wall | (the stroke) | | flowers bloom, bushes ivy, grass moss, waterside vines, up the sides dragged over; Rub out takes them off |
| The view, over a house | what climbs it; its furniture | `by_climb`, `by_furnish` | open their rows under the strip |
| Under the strip (climbs) | ivy, moss, vines, blooms; less, more; nine colours; none | `by_climb_<word>`, `by_climb_fewer`, `by_climb_more`, `by_climb_tint_<hex>`, `by_climb_none` | each on every side or off; how much; their colour |
| Under the strip (furniture) | home, shelter, store, camp; sparser, fuller; none | `by_furnish_<set>`, `by_furnish_fewer`, `by_furnish_more`, `by_furnish_none` | the set of furnish.txt; how many places used |
| The view, over a house or a patch | keep; kept | `by_keep`, `by_kept` | keep its look in your library; open the kept ones |
| Under the strip (kept) | each kept look | `by_kept_<n>` | gives it this |
| The view, over decor or life | its sets | `by_decor_set_<set>`, `by_life_set_<set>` | the whole mix at once |
| Bar at the top | Build | `menu_build` | every tool of the bar at the right; keep or copy the chosen look; the kept looks |
| Library | Kept looks (N); Remove | `kept_looks`, `kept_remove_<n>` | the looks you kept, where each is used; one taken out |

## 13zz27. Where a tower meets a house, and cattails' heads (2026-10-20, amendment 7cy)

| Where | Control | Name | Does |
|---|---|---|---|
| The view, over a house or tower that meets another | walled; open | `by_join_walled`, `by_join_open` | each keeps its walls; or one room where they meet. One change |
| The view, over waterside plants | their heads, in their colour | `by_heads` | opens the heads' row under the strip |
| Under the strip | head colours; any colour | `by_heads_tint_<n>`, `by_heads_any` | every head of the patch that colour |
| Under the strip | slimmer, plumper, smoother, fuzzier | `by_heads_slim`, `by_heads_plump`, `by_heads_smooth`, `by_heads_fuzzy` | every grass of the patch with heads grown again so |
| Under a mix's sorts | sliders, in each sort's colour | `by_<brush>_tune_<preset>` | opens the Plant Generator on this patch's design of it |
| Plant Generator, Grass | Heads: plump, fuzz, head_tint | | the same numbers, for any grass |

## 13zz26. Mixes, rocks and towers (2026-10-20, amendment 7cx)

| Where | Control | Name | Does |
|---|---|---|---|
| Paint | Waterside | `cb_waterside` | paints cattails, reeds and rushes |
| The view, over what a brush painted | what is in it | `by_<brush>_sorts` | opens its sorts under the strip (flowers, bushes, waterside, rocks) |
| Under the strip | each sort, in its colour | `by_<brush>_<preset>` | that sort in the mix or out; grown the first time |
| The view, over painted rocks | their colours | `by_rocks_colour` | opens the places' chips `by_rocks_place_<0-6>` and any colour `by_rocks_any` |
| The view, over painted rocks | their material, as the picture | `by_rocks_texture` | opens `by_rocks_texture_<picture>` (`_plain` for none) |
| The view, over painted rocks | less moss, more moss; smaller, bigger; other shapes; edges | `by_rocks_moss_less`, `by_rocks_moss_more`, `by_rocks_shorter`, `by_rocks_taller`, `by_rocks_shapes`, `by_rocks_edges` | all the patch's rocks grown again so, one change |
| The view, over bushes or waterside plants | lower, taller | `by_<brush>_shorter`, `by_<brush>_taller` | all its plants, one change |
| The view, over what a brush painted | new patch; copy; give | `by_<brush>_new_patch`, `by_<brush>_copy`, `by_<brush>_give` | a patch with a look of its own, then painting it; its look copied; the look copied given to this one |
| Look, a patch chosen | New patch, Copy, Give | `cb_new_patch`, `cb_copy_patch`, `cb_give_patch` | the same, with words |
| The view, over a chosen house | copy; give | `by_copy_look`, `by_give_look` | as Look's Copy and Give |
| The view, a chosen house's corners | green towers | (handles) | pressed and dragged, a round tower pulled out of that corner; a click, one of 3 m |
| Plant Generator | Rock (a family) | `rock` | its twenty knobs and nine presets |

**Changed:** Bushes beside them no longer replace one form with another; the sorts are a mix. Flowers' sorts moved into the row under the strip.

## 13zz25. Changed where it stands (2026-10-20, amendment 7cw)

| Where | Control | Name | Does |
|---|---|---|---|
| The view, over a chosen house | walls, roof, trim, timbers in their colours | `by_wall_colour`, `by_roof_colour`, `by_trim_colour`, `by_wood_colour` | opens that part's colours under the strip |
| Under the strip | colour chips; any colour | `by_<part>_<rrggbb>`, `by_<part>_any` | that part's colour, one change; the picker sets it when it closes |
| The view, over a chosen house | its walls' and roof's material, as the picture | `by_texture`, `by_roof_texture` | opens the materials under the strip: `by_texture_<picture>`, `by_roof_texture_<picture>` (`_plain` for none) |
| The view, over a chosen house | roofs off | `by_roofs_off` | roofs off; again, upper floors off too; again, back. For looking only |
| View | Roofs off, to look into houses; Upper floors off too | | the same |
| The view, over what a brush painted | Grass: shorter, the length, longer; ten colours | `by_grass_shorter`, `by_grass_taller`, `by_grass_tint_<n>` | the brush's grass grown again so; all of it follows |
| The view, over what a brush painted | Flowers: daisy, poppy, bluebell, tulip, lupin, foxglove; grass shorter, longer | `by_flowers_<sort>`, `by_flowers_shorter`, `by_flowers_taller` | a sort in the mix or out; the grass among them |
| The view, over what a brush painted | Bushes: shrub, wild rose, blueberry, bramble, low mat; lower, taller | `by_bushes_<form>`, `by_bushes_shorter`, `by_bushes_taller` | the brush's bush grown again so |
| The view, over what a brush painted | Trees: mixed, broadleaf, conifers | `by_trees_tree_mixed`, `by_trees_tree`, `by_trees_conifer` | what the trees brush paints |
| The view, over what a brush painted | thinner, thicker | `by_<brush>_fewer`, `by_<brush>_more` | its `dense` |
| Plant Generator, Grass | (knob) | `tint` | a colour of its own for its blades, as rrggbb; 0 the style's greens |

## 13zz24. One house, one outline (2026-10-20, amendment 7cv)

| Where | Control | Name | Does |
|---|---|---|---|
| The view, a chosen house | green + past each side's arrow | (line.gd `wings`) | held and dragged out, pulls a wing out of that side; along the side too, slides it, flush at an end for an L |
| The view, a house let go into another | (no control; Ctrl held keeps them apart) | | one house of one outline, the other's look kept; one Ctrl+Z |
| The view, over a chosen house | swatches | `by_palette_<n>` | its palette, one change |
| The view, over a chosen house | gable, hipped, flat | `by_roof_<form>` | its roof's form |
| The view, over a chosen house | storey down, the count, storey up | `by_lower`, `by_higher` | its storeys, 1 to 4 |
| Right side | Paint ‹ | `cb_paint` | Grass, Flowers, Bushes, Trees, Rocks (`cb_grass`…), Rub out (`cb_rub`): brushes dragged on the land, each painting into a maker of its own ("painted grass"…) |
| Inspector, a meadow or a wood | (setting) | `dense` | so many to each 2 m square painted; 0 shares its count out over what is painted |

## 13zz23. The construction bar (2026-10-20, amendment 7cu)

| Where | Control | Name | Does |
|---|---|---|---|
| Right side, while editing | Build ‹ | `cb_build` | opens its tools beside it: House, Tower, Wall, Fence, Hedge (`cb_house`…); pressed again, puts them away |
| Right side | Paths ‹ | `cb_paths` | Path, Yard, Garden |
| Right side | Water ‹ | `cb_water` | Pond (with life), Kerbed |
| Right side | Look ‹ | `cb_look` | the chosen house's sort (`cb_sort_<sort>`), its palette (`cb_palette_<n>`), Copy (`cb_copy_look`) and Give (`cb_give_look`) of a look; with no house chosen, says to click one |
| Right side | Erase | `cb_erase` | the eraser, as Erase on the left |
| A tool of a group | pressed | | taken up at once: the next press and drag on the land draws it; lit while in hand; pressed again, put down |

## 13zz22. Roofs that are one where houses meet (2026-10-20, amendment 7ct)

| Where | Control | Name | Does |
|---|---|---|---|
| The view, drawn houses that touch or overlap, an L/T/U-shaped house | (no control: it follows what is drawn and moved) | | each roof is not built inside the other house; they meet in a valley as one roof; worked out after every edit, undone with it |
| The view, a house whose door side comes inside another | (none) | | its door goes to the nearest wall still standing outside |
| Inspector, a drawn structure's "How it is built" (host `nodes` `structure`) | (read only) | `roof_near` | the houses near enough for their roofs to meet this one's |

## 13zz21. A house dragged by any part; a sort undone (2026-10-20, amendment 7cs)

| Where | Control | Name | Does |
|---|---|---|---|
| The view, a chosen drawn house | right button dragged on any part of it | (main.gd `dragging_whole`, line.gd `grab_whole`) | the whole house goes, as by its white cross; one edit |
| The view, a piece of a chosen drawn house | right click; toolbar Move | | says that a house goes as one and how to drag it; no longer moves one piece off it |
| World outline / Inspector, "A house of its sort" | the sort buttons | `house_<sort>` | each sets the plain house's look for what it does not say, so a house made a cottage and back is a house again |

## 13zz20. What is built where a house stands (2026-10-20, amendment 7cr)

| Where | Control | Name | Does |
|---|---|---|---|
| The view, any drawn wall, fence, hedge or house | (no control: it follows what is drawn and moved) | | a wall across a house stops at its faces; houses side by side share one wall; overlapping houses are one inside, the higher keeping its upper storeys; worked out after every edit, undone with it |
| World outline and the Inspector, a drawn wall, fence, hedge or house, under "How it is built" | part **Where it meets houses** (shown only when there is something to say) | `fold_meets`, `meets_note` | says, without changing anything: an end joined to a house (7cp), how many stretches are left out where a house stands, how many houses it overlaps that build the walls they share (host `nodes` `structure`: `joined`, `cut`, `overlaps`) |

## 13zz19. A handle's drag that was not kept (2026-10-20, amendment 7cq)

| Where | Control | Name | Does |
|---|---|---|---|
| The view, a drawn house's, a tower's or a height's handle | (let go) | | when what it was dragged to is not kept: the drag shown is let go (`drop`), the house's places and height stay as the host has them, and a note says "Not changed: … It is as it was" (before: nothing was said and the failed drag stayed shown) |

## 13zz18. Houses of their sorts, a stair's handle, a style's air and its palette (2026-10-20, amendment 7cp)

| Where | Control | Name | Does |
|---|---|---|---|
| Build > Draw on the land | **A cottage: drag it out…**, **A castle keep…**, **A round tower…**, **A shelter…**, **A ruin…** | (entries) | after the house's row: dragged out as a house is, made of that sort from the first frame (`house` and the sort's look and storeys, `makers.gd` `HOUSE_SORTS`) |
| World outline and the Inspector, a drawn house, under "How it is built" | part **A house of its sort: …** | `fold_house_sort` | open at first; says its sort |
| the same | a house · a cottage · a castle keep · a round tower · a shelter · a ruin | `house_house`, `house_cottage`, `house_castle`, `house_tower`, `house_shelter`, `house_ruin` | gives it that sort with a look and storeys to start from: the host's `params`, one step; the one it is, pressed |
| the same | a line saying where its stairs are | | the house's own choice, or the corner and wall said |
| the same, its stair said | **Let it choose where its stairs go** | `house_stair_own` | `stair_corner` 0, `stair_way` 0: the house chooses again; one step |
| The view, a drawn house of storeys chosen | an orange ball over each stair | (handle) | dragged toward a corner: the stair goes to it, along the wall of it nearer the pointer (`stair_corner`, `stair_way`), built again as it goes, one step; Esc puts it back |
| Window > Styles, a style of one's own | **Sun haze**, **Low haze at**, **Low haze**, **Distance**, **Shade colour**, **Soft shadows** (sliders) and **Shade colour** (a colour) | `style_sun_haze`, `style_low_haze_at`, `style_low_haze`, `style_distance`, `style_shade_colour`, `style_soft_shadows`, `style_shadow_colour` | the style's lines `haze`, `distance`, `shadow`, `soft`, written to styles.txt, the world given it again |
| Window > Styles, any style | part **Palette of '…'** | `style_palette` | every colour of the style by what it colours, in three groups (what is built, what grows, the ground and the water); says what a change reaches and where it is kept |
| the same | a swatch for each colour of each role | `palette_ROLE_K` (`palette_roofs_0`…) | in a style of one's own: a colour picker, the role's line written as it is chosen and seen at once; in one of the library's: shown, not changed |
| the same, a style of one's own | **+** · **−** beside a role of several colours | `palette_ROLE_more`, `palette_ROLE_less` | one more colour (the last one again, to change), or the last taken out |
| World > New world, an empty island or nothing | (no control) | | the new world begins in the lookout style (an edit like any; F2 or Window > Styles for another) |

## 13zz17. A drawn house's own look and its handles (2026-10-19, amendment 7cn)

| Where | Control | Name | Does |
|---|---|---|---|
| Build > Draw on the land | **A house: drag it out, then its handles (or click round any shape); its door in the first side** | (entry) | was "A house of any shape: draw round it…"; pressed on the land and dragged, a house is built from the corner pressed to the pointer, square to the view (to 15°), at least 3 m each way, staged once a frame; let go it is kept (one step) and chosen; Esc gives it up; clicks still put corners one at a time, Enter ends |
| The row under the bar, while dragging a house | "DRAWING '…': it is built as you drag…" | | what the drag is doing |
| World outline and the Inspector, a drawn or cell house, under "How it is built" | part **Its look: …** | `fold_look` | new for a house (a wall's is 13zz16): says how many colours of its own it has and the picture over its walls |
| the same | palette: choose one… · as the world's style · white cottage · stone house · red brick · timber boards · painted blue · mossy old | `look_palette` | its colours and pictures set together: the host's `params`, one step |
| the same | walls · trim · windows · wood · roof · footing: a colour picker each, and **x** / "(style)" | `look_wall_colour`, `look_trim_colour`, `look_pane_colour`, `look_wood_colour`, `look_roof_colour`, `look_stone_colour`, `look_back_…` | the parameter of that name, rrggbb, seen as it is chosen; one step each; "wood" is its doors, the timbers framing its walls and its floors' boards |
| the same | picture over its walls · picture over its roof: none · the project's textures | `look_texture`, `look_roof_texture` | `texture` (not over windows or doors), `roof_texture` |
| the same | roof made with: two slopes and gables · four slopes (hipped) · flat, with a rim | `look_roof_form` | `roof_form`; a house not square-cornered keeps its roof that rises to a point |
| the same | the pictures repeat every (m) · roof steepness · windows every (m) | `look_texture_repeat`, `look_roof_rise`, `look_window_every` | `texture_repeat`, `roof_rise` (1 as made), `window_every` (0: 2.2 m) |
| The view, a drawn house chosen | a white cross on the ground in its middle | (handle) | dragged: the whole house moves; built again as it goes, one step; Esc puts it back |
| The view, the same | a white ring off its first corner | (handle) | dragged round: the house turns about its middle, by 5° |
| The view, the same | an orange arrow out from each side | (handle) | dragged out or in: that side moves, the house longer or shorter (stops 1.5 m short of the side across) |
| The view, the same | a blue point over a side | (handle) | was over the middle of its line; dragged up and down: how high each storey is (all its storeys go together) |
| The view, the same | a blue point over its roof | (handle) | dragged up and down: how steep its roof is (`roof_rise`); not shown for a flat roof |
| The view, the same | orange dots, white "+", Alt + a click | (handles) | as a wall's (13zz15, 13zz16): a corner moved, one put in, one taken out (three stay at least) |
| The view, a drawn wall or house that goes round | its first orange dot | (handle) | its first place and its last move together now (they parted, and the line came open) |

## 13zz16. A drawn wall's look, its towers' handles, a place put in or taken out, snapping (2026-10-18, amendment 7cm)

| Where | Control | Name | Does |
|---|---|---|---|
| World outline and the Inspector, a wall, fence, hedge, pond or garden, under "How it is built" | part **Its look: …** | `fold_look` | folded at first; says how many colours of its own it has and its picture |
| the same | palette: choose one… · as the world's style · grey fieldstone · warm sandstone · red brick · whitewashed · mossy ruin · dark slate | `look_palette` | its colours, pictures and stone size set together: the host's `params`, one step |
| the same | stone · coping and caps · wood (wall, fence) · hedge (hedge) · tower roofs (with towers): a colour picker each | `look_stone_colour`, `look_cap_colour`, `look_wood_colour`, `look_green_colour`, `look_roof_colour` | the parameter of that name, rrggbb, seen as it is chosen (`livecolour.gd`); one step each |
| the same | **x** beside a colour, or "(style)" when it has none of its own | `look_back_…` | back to the world's style's colour |
| the same | picture over it: none · the project's textures | `look_texture` | `texture`; not over its gates |
| the same | the picture repeats every (m) | `look_texture_repeat` | `texture_repeat` |
| the same, a wall, pond or garden | a course of stones (m) | `look_course` | `course` (0: as made, 0.26 m) |
| the same, with towers | picture over its towers' roofs | `look_roof_texture` | `roof_texture` |
| the same, with towers | towers roofed with: a cone · a tall spire · a dome · a flat lid · battlements (no roof) · open (no roof) | `look_tower_roof` | `tower_roof` |
| the same, with towers | tower width (m) · towers over the wall (m) · roof steepness | `look_tower_wide`, `look_tower_rise`, `look_roof_rise` | `tower_wide` (0: by the wall), `tower_rise`, `roof_rise` |
| The view, a drawn wall with towers chosen | a green ring round each tower | (handle) | dragged out or in: how wide all its towers are; built again as it goes, one step; Esc puts it back |
| The view, the same | a green point over each tower | (handle) | dragged up and down: how far over the wall they stand; the same way |
| The view, a drawn wall, fence or hedge chosen | a white "+" half way between each two places | (handle) | pressed: a place put there and held; dragged with the wall built again; one step with the drag |
| The view, the same | Alt + a click on an orange dot | | takes that place out (two stay at least); one step |
| The view, the same | orange dots | (handles) | smaller than before (were as big as a road's); the blue lines over each of its pieces are no longer drawn while its handles show |
| Drawing a wall, fence or hedge, or dragging a place of one | (none) | | its ends, or the place, go onto another drawn wall near them (its end within 1.6 m, its length within 0.9 m); Ctrl held, they don't |
| The view, what a wall builds | a tower's roof | sort "tower roof" | a piece of its own now (picked, taken away, a picture of its own) |

## 13zz15. Walls drawn by dragging, with handles (2026-10-18, amendment 7cl)

| Where | Control | Name | Does |
|---|---|---|---|
| Build > Draw on the land | **A stone wall: drag it out, a tower where it turns** | (entry) | was "A stone wall, with a tower where it turns"; pressed on the land and dragged, the wall is built as the pointer goes (staged once a frame), let go it is kept (one step), Esc gives it up; a click without a drag still puts one place, Enter ends |
| Build > Draw on the land | **A fence: drag it out** | (entry) | was "A fence"; the same |
| Build > Draw on the land | **A hedge: drag it out** | (entry) | was "A hedge"; the same |
| The view, a drawn wall, fence or hedge chosen | orange dots on the land | (handles) | its line's places: dragged with the left button, the wall built again as it goes, one step on letting go; Esc puts it back |
| The view, the same | a blue point over its middle | (handle) | its height (the height parameter): dragged up or down, the same way |
| The row under the bar, while drawing | "DRAWING …: it is built as you go…" | | what the stroke is doing |

## 13zz14. Places drawn that know one another (2026-10-18, amendment 7ck)

| Where | Control | Name | Does |
|---|---|---|---|
| Build > Draw on the land | **A path: click along where it goes; it opens a gate or a door where it meets one…** | (entry) | new: a line that does not go round, built as nothing round it with gravel (`as none`, `floor gravel`) |
| Build > Draw on the land | **A pond with life: draw round it; a bank, reeds, lilies, fish, frogs…** | (entry) | new: `edge bank`, the set `lively` of ponds.txt (its designs brought from the library first) |
| Build > Draw on the land | **A pond with a stone kerb: draw round it…** | (entry) | was "A pond: draw round it" |
| Build > Draw on the land | **A garden: draw round it for beds, or along a wall for a border…** | (entry) | was "A garden planted in rows: draw round it"; along a line it is a border |
| Build > Draw on the land | **A paved yard: draw round it…** | (entry) | was "A paved yard or path: draw round it" |
| Any drawing | (none) | | one Ctrl+Z takes back all one drawing made (its designs, its line, the structure); the message says "Ctrl+Z takes both (and what it opened in what it crosses) back" |
| World outline | a drawn structure's name | | the house, stone wall, fence, hedge, path, border, garden, pond, yard (a number after it if taken); its line "line of …"; was "structure N" |
| World outline, a drawn path or border | "a path, W m wide" + number | `build_path_row`, `build_wide` | `wide` (0.4 to 8 m; 1.4 when not said) |
| World outline, a drawn path | (words) | `build_path_says` | what a path opens, and that it follows the path |
| World outline, a drawn border | (words) | `build_border_says` | what a border is |
| World outline, a drawn wall, fence, hedge, house or garden | **paths drawn across it open a way (a gate / a door)** | `build_opens` | `opens` 1/0 |
| World outline, the same | "Opened by NAMES, in piece(s): s0_3" | `build_opened_by` | what the host found (`opened`, `opened_by`); shown while it is opened |
| World outline, a drawn pond | its edge: a kerb of stone · a bank of earth and grass | `build_edge` | `edge` |
| World outline, a drawn pond | pond life: a set of ponds.txt… · lively · quiet · fish · none | `build_pond_set` | brings its designs from the library, then sets `life`, `bank`, `lively` |
| World outline, a drawn pond | in its water · along its bank (text) | `build_life`, `build_bank` | `life`, `bank`; Enter brings designs not in the world from the library |
| World outline, a drawn pond | how lively (0 to 1) | `build_lively` | `lively` |
| World outline, built as | "a pond: a kerb or a bank, water in it" · "a garden: an edging, beds planted in rows (along a line: a border)" · "nothing round it: only its floor (along a line: a path), or a room with no walls" | `build_as` | words only |
| Asset Library, Rural and island | group **Pond and water's edge** (10 designs) | | new |

## 13zz13. A controller first; the game's screens by a controller; the display's bars (2026-10-18, amendment 7cj)

| Where | Control | Name | Does |
|---|---|---|---|
| Playing, any world | the attack button: X or RT, a click | `attack` (controls.gd) | every blow: the fist's (`strike`), an axe's (`hew`), a sword's cuts; with what fires drawn, it fires |
| Playing | the controller's other buttons | `controls.gd` MAP | A jump, B dash (aside: dodge), Y do what is offered, LB lock on, RB run (held), LT crouch (and aim), right stick pressed put away, D-pad left use, D-pad right the next thing kept, D-pad up/down the eye, Back inventory, Start menu; the quick steps aside on Z and X only |
| Playing, the prompts and hints | the buttons named | | a controller's from the start when one is plugged in (`main.gd`), the keyboard's once a key is pressed |
| The menu, the ending, the controls page | D-pad or left stick, A, B | `gameui.gd` `pad` | from choice to choice, choose, go back, whatever Godot's own ui actions are; never out to the editor's windows |
| The same, and the inventory | what is chosen | `screens_theme` | a gold frame and gold words; pressed, gold on brown |
| The menu, under its buttons | "A choose  B back  D-pad or stick move" (or the keyboard's) | `menu_keys` | says how it is worked |
| The menu > Controls | the page | | only what the device in use has, by the buttons' names (no "(button 2)") |
| Playing, the display, top left | HEALTH, a bar and "3 / 5"; BREATH, a bar ("winded") | `fight_health`, rows `health`, `breath` | in place of "Health #####"; the stats' bars stand under them |
| Playing, over one hurt | a red bar facing the eye | `hp_mark` | in place of "##--" |
| Playing, the bottom | "unseen  seen 70%" | `watch_meter` | shown only where going unseen is part of the game (someone who catches by sight or spots, or an eye on a post) |
| Playing, the bottom, at an emptied chest | "The camp chest is empty." | | in place of "Search the camp chest? The camp chest has nothing." |
| Picked up, gear | "In hand: an axe. X: strike with it." | `items.gd` `first_use` | what is held goes into empty hands, what is worn onto a free place |
| gear.txt | `model PATH turn x y z` | `gearbook.gd` | a model file turned to lie as gear lies; gear models with no bones are made |
| things' traits | `lights … flame` | `things.gd` `_flame` | flames rising and flickering where it lights, in place of a glow (the beacon) |
| Asset Library, Woodland | Stone watchtower | `watchtower-keep` | a keep with a stair round its outside to its battlements; written by `editor/library/watchtower-keep.js` |
| Characters | the ranger | `character ranger` | Quaternius's ranger outfit and head (`characters/ranger.glb`, by `outfitmake.gd`) |
| Window > Styles | golden_valley | | the watchtower's style |

## 13zz12. A game's hour and its ending; the Forgotten Watchtower (2026-10-18, amendment 7ch)

| Where | Control | Name | Does |
|---|---|---|---|
| Inspector, HOW IT WORKS (the start), under "runs for, seconds" | a game begins at (hour; 0: morning) | `start_hour` | the Player start's `hour`: the hour a game begins at; 0 is the engine's morning (9) |
| the same | the game ends when | `start_ends` | the start's `ends`: a condition in cond.gd's words (`quest NAME done`, `fact NAME`, `has N GOOD`); one that is none is refused with why; empty, no ending. Applied when the field is left |
| the same | the ending says | `start_ending` | the start's `ending`: the words of the ending screen |
| World outline, the Player start | the same three rows | `set_hour`, `set_ends`, `set_ending` | as in the Inspector |
| Playing, when the start's `ends` holds | the ending screen: The end, the ending's words, how long it was played | `game_ending`, `ending_text` | holds the game; the world's fact `ended` is put (a saved game knows) |
| the ending screen | Keep exploring | `ending_on` | shuts it; it is not shown again this run (Esc does the same) |
| the ending screen | Play again from the start | `ending_again` | the world stopped and begun again at its start, as it was made |
| the ending screen | Back to editing / Leave the game | `ending_leave` | as the menu's |
| Playing, the display | the quest step (top right) | `hud_quest` | a step with its hint too long for the screen wraps within about a third of it (it ran off the left edge) |
| World > New world | Genre: The Forgotten Watchtower (a wilderness adventure) | `genre:watchtower` | the benchmark level, `editor/genres/watchtower.json` |
| Asset Library, Woodland (new shelf) | Fallen log, Tarp shelter, Beacon brazier, Waymark lantern, Stone tower wall, Stone parapet, Stone doorway, Stone tower corner | | new designs; People gains The old ranger, Animals that hunt The grey alpha |

## 13zz11. When this happens, do this (2026-10-18, amendment 7cg)

| Where | Control | Name | Does |
|---|---|---|---|
| Inspector, What it can do, Give it… (Does when something happens) | when this happens, does this | `ability_wired`, `ability_away_wired` | gives the line `on act do say Found something`; its x takes every `on` line away |
| Inspector, What it can do, under that row, one per `on` line | when · WHAT (a list: senses, hits, spots, act, answer) · WHICH (act and answer only; empty: any) · do · WHAT IS DONE (a list: put, give, say, sound, move) · its words · x | `wire_K`, `wire_K_when`, `wire_K_which`, `wire_K_does`, `wire_K_args`, `wire_K_needs`, `wire_K_away` | the line `on WHEN [WHICH] do WHAT ...`, written again as each is changed (one step of undo; the host refuses what is wrong, with why); a doing chosen begins with words of its own (put found, give coin 1, say Found something, sound chime, move wave); in orange when the thing does not say what it needs for that to happen (hits, senses, sees, offers, someone) |
| the same, below the rows | When this happens… (one more) | `wires_add` | one more row, for what is chosen |
| Inspector, What it can do, a row of an ability that is something happening (sees) | then… | `ability_NAME_then_WHEN` | beside the row (as while is beside what may be held): a row for when that happens is added |
| Playtest (8.15), under the quests | When this happened, done: WHO: LINE (N times) | `wires_view` | each `on` line done this run, by whom, how often |

The Inspector is put away while playing, so what was done is in Playtest, not on the row. (The same is so of 7cf's "(now: …)" below: it is never seen.)

## 13zz10. What it does only while something holds (2026-10-18, amendment 7cf)

| Where | Control | Name | Does |
|---|---|---|---|
| Inspector, HOW IT WORKS, What it can do (a row of an ability the folk do as they go about) | while · its condition (empty: always) | `ability_NAME_while` | it does that only while the condition holds (fact tamed, hour 6 20, near walker 4, weather rain, not …): the design says `while WORD CONDITION`; Enter or clicking away sets it, one step of undo; a condition that is none is refused with why |
| the same row, while playing | (now: yes) · (now: no, why) | `ability_NAME_held` | whether it holds for the one picked now |

## 13zz9. Modelled and asked for (2026-10-18, amendment 7ce)

| Where | Control | Name | Does |
|---|---|---|---|
| Objects (toolbar), Make one | Modelling: Godot's solid shapes added, cut out, extruded, spun and swept… | — | opens the Modelling window |
| Objects (toolbar), Make one | Assistant: ask for a change in words… | — | opens the Assistant window (also Ctrl+J, and Window) |
| Modelling window | the model · new model's name · New model ▾ (box with a door, arch, cave, vase, pipe, gable) | `model_which`, `model_name`, `model_new` | which model of modelling.txt is shown; a new one begun from one of those, one step of undo |
| Modelling window | Make it · Put one down · Open modelling.txt | `model_make`, `model_put`, `model_text` | Godot's solid baked into a mesh and a design of the model's name (one step of undo); made, then a click on the world puts one; the text in F10 |
| Modelling window | the picture | `model_view` | the solid as Godot's CSG nodes make it, turning by a drag, nearer by the wheel |
| Modelling window | smooth · walked into as it is shaped / as a box / not · Add a shape ▾ | `model_smooth`, `model_solid`, `model_add` | shading; what is walked into; another shape (box, ball, post, cone, ring, extrude, spin, sweep) at the end |
| Modelling window, a shape's row | add / cut out / keep where it meets · its numbers · ↑ · ✕ | `model_opN`, `model_lineN`, `model_upN`, `model_dropN` | what it does to what is above it; its line (Enter sets it); earlier; taken away. Each one step of undo |
| Inspector, HOW IT WORKS (a design made of a model) | Its shape: the model … · Change it in Modelling… | `works_model`, `works_model_open` | says it was made of Godot's solid shapes; opens Modelling on it |
| Assistant window | (the model in use) · Settings · Ideas ▾ · New conversation | `assistant_model`, `assistant_settings`, `assistant_ideas`, `assistant_new` | which model answers; its settings shown or hidden; a request to begin from put in the box; what was said forgotten |
| Assistant window, Settings | Model from · Address · Model · List models · API key · Remember… · Effort (Claude) · Tool calls · Test connection | `assistant_preset`, `assistant_address`, `assistant_model_id`, `assistant_model_list`, `assistant_list_models`, `assistant_key`, `assistant_remember`, `assistant_effort`, `assistant_tool_mode`, `assistant_test` | Claude (your key), LM Studio, Ollama or another OpenAI-compatible API; kept on this machine only with Remember, never in a world |
| Assistant window | what was said (each tool it used in a line: · to look, ✎ to draft) | `assistant_said` | the conversation |
| Assistant window, the draft | a change's sentence and its problems · ✕ · Apply (n) · Discard · Undo this | `assistant_changeN`, `assistant_dropN`, `assistant_apply`, `assistant_discard`, `assistant_undo` | nothing changed until Apply; Apply makes every change as one step of undo; Undo this (or Ctrl+Z) takes all of it back |
| Assistant window | the request box · Send · Stop | `assistant_request`, `assistant_send`, `assistant_stop` | Enter sends, Shift+Enter a new line; Ctrl+Z / Ctrl+Y in the empty box undo and redo the world |
| (the walker) | — | — | the scout walks unless a world says who: Quaternius's model; crouched, KayKit's sneak; backing away, aside, dodges with their own clips |

## 13zz8. Furnished and set out (2026-10-18, amendment 7cd)

| Where | Control | Name | Does |
|---|---|---|---|
| World outline, a structure (and the Inspector on a piece of it) | furnished with: a set of furnish.txt · nothing | `build_furnish_set` | a home, a shelter, a store room or a camp: its designs brought from the library, put against the walls inside and in the middle of each cell, every storey; or the furniture taken out |
| World outline, a structure | against the walls · in the middle | `build_furnish`, `build_furnish_middle` | the designs said by hand, as a maker's designs are said ("bed-rest, chair-rest*0.5"); set by leaving the field |
| World outline, a structure | of the places, used | `build_furnished` | 0 to 1: how many of the places for furniture are used |
| World outline, a structure | built as: nothing round it: only its floor, or a room with no walls | `build_as` | the old "nothing round it", named for a cave or a camp |
| World outline; Inspector | (a number changed) | — | the window stays where it was scrolled to; it went back to its top |
| Inspector, How it is done | stops you before it: turned to, stopped · this one | `done_stops_ACT`, `done_only_stops_ACT` | whether the walker, walking toward it, is turned to it and stopped before it (a cupboard) or goes through or past it (a door: acts.txt `passes`) |
| Characters menu | Game display: where each piece of what is on the screen stands… | — | opens the Game display window |
| Game display window | Show it here, while editing | `hud_preview` | every piece shown over the view with words of its own |
| Game display window | Add a font… · Open hud.txt | `hud_add_font`, `hud_text` | a font file copied into the fonts folder; hud.txt in the texts' window (F10) |
| Game display window, a piece's row | shown · corner · x · y · letters (or all of it) · colour · font · backing | `hud_shown_P`, `hud_at_P`, `hud_x_P`, `hud_y_P`, `hud_size_P` / `hud_scale_P`, `hud_colour_P`, `hud_font_P`, `hud_plate_P` | where the piece stands and how it looks; each change one step of undo, seen at once |
| Inspector, What it can do, Give it… | is covered all over with another design | `ability_covered` | `covered DESIGN EVERY [SIZE]`: grass, flowers, ivy, pebbles or boxes strewn all over its outside (the design brought from the library) |
| Inspector, Its shape | texture | `shape_texture_pick` | 42 pictures now (28 more from ambientCG: cardboard, moss, snow, sand, rust, dirt, gravel, carpet, plaster, wicker…) |
| Asset Library | shelves Furniture to use · To cover things with | — | a bed to sleep in, a wardrobe, chest, high shelf and crates to search, a chair, a table; grass tufts, small flowers, ivy leaves, pebbles, cardboard boxes |

## 13zz7. What anything can do, given to anything (2026-10-17, amendment 7cc)

| Where | Control | Name | Does |
|---|---|---|---|
| Inspector, HOW IT WORKS | What it can do | `abilities` | what the picked thing (put down by hand) can do, from the catalogue (abilities.txt) |
| Inspector, What it can do | Made usable as: NAME x | `abilities_uses`, `use_away_NAME` | each use it was made (a design made for it, or its own lines); pressed, that use taken away |
| Inspector, What it can do | one row per ability: its numbers · x | `ability_NAME`, `ability_NAME_FIELD`, `ability_away_NAME` | an ability it has, with its numbers (set by Enter or by leaving the field), and taken away |
| Inspector, What it can do | Give it… (under its groups) | `abilities_give` | any other ability given, with what it needs (a body that goes about, a rig) |
| Inspector, What it can do | The catalogue (abilities.txt) | `abilities_text` | abilities.txt opened (F10): every ability, its lines, its numbers as first given |
| Inspector, What it can do | (words in orange) | `abilities_missing` | words the engine has that the catalogue does not list yet |
| Inspector, How it is done | low down (under H m) · high up (over H m) | `done_ACT_low`, `done_ACT_high`, `done_only_ACT_low`, `done_only_ACT_high` | the move for a place low down or high up (acts.txt does_low, does_high), for every thing or this one |
| Inspector, How it is done | which way they lie | `done_lie_turn` | as it is, turned round, a quarter left or right: the side it is got on from, turned |
| Inspector, Its shape | texture · repeats every … m · Add a picture… | `shape_texture_pick`, `shape_texture_repeat`, `shape_texture_add` | a picture of the textures folder laid over the design; how often it repeats; your own picture copied in |
| Inspector, Its shape; Library, Customize | a part's colour square | `shape_N_colour`, `colour_WORD` | seen as it is chosen (written a moment after each change), not when the picker shuts |
| Every number field, and words in the Inspector, Makers, layouts, variation, designs, tree generator | (leaving the field) | — | sets what was typed, as Enter does; amber while typed and not yet set |
| Make it usable | a pet · a flyer | `usable_preset` | new and renamed uses (uses.txt) |
| Playing | jump at a climbable wall · the stick · crouch | — | held to it, climbed up, down and along, let go (moves.txt wall_climb) |
| Playing | jump under a climbable ceiling · the stick · crouch | — | hung under it, gone along, let go (moves.txt ceiling_crawl) |

## 13zz6. Anything made, and made to do anything (2026-10-17, amendment 7cb)

| Where | Control | Name | Does |
|---|---|---|---|
| Asset Library | Search (its own row) | `search` | every word typed found in a design's name, what it is called, its group, its shelf or what it says |
| Keys, editing | Ctrl+F | — | opens the Asset Library at its search |
| Objects menu, Make one | New shape: a box · a cylinder · a cone · a ball | — | a new design of one shape, put down by a click (`shapemaker.gd`) |
| Objects menu, Make one | Import a 3D model (glTF, FBX, OBJ)… | `import_model` | the file read, written as models/NAME.glb, a design of this world made of it and saved to your Library shelf (Imported); put down by a click |
| World menu | Export the game… | — | the world saved, then built in the background as build\NAME\NAME.exe (and its folder) and build\NAME.zip; told when done, the folder shown |
| Inspector, HOW IT WORKS, Make it usable | make it… (from uses.txt) | `usable_preset` | every use of uses.txt, yours among them: bed, seat, food, drink, medicine, container, chest, well, eat_at, weapon, creature, flyer, pet, none |
| Inspector, HOW IT WORKS, Make it usable | Save as a use… · The uses' text (uses.txt) | `usable_save`, `usable_text` | the thing's use lines saved as a use of your own (a name, what it is); uses.txt opened (F10) |
| Inspector, HOW IT WORKS, Make it usable | How it is done: each act's move · this one | `done_ACT`, `done_only_ACT` | the walker's move for that act (or the pose it rests in): for every thing that offers it, or for this one only (an act of its own) |
| Inspector, HOW IT WORKS | Its shape: the design 'NAME' | `shape_rows`, `shape_N_x/y/z/w/h/d`, `shape_N_colour`, `shape_N_remove`, `shape_add_box/cylinder/cone/ball` | the design's boxes, cylinders, cones and balls: where each stands, how big, its colour; taken away; more added |
| Playing, over what is at hand | ▼ (green) | `facing_mark` | the walker has turned to it: it is what E will do |
| Playing, the inventory | the line under the buttons | `inventory_hint` | the keys and a controller's buttons there: A or X uses or puts on, Y drops, right to the buttons, B shuts |
| Playing, the inventory (controller) | X · Y | — | X: what is likely (eat, drink, use, put on); Y: drop |

## 13zz5. Use anything (2026-10-17, amendment 7ca)

| Where | Control | Name | Does |
|---|---|---|---|
| The view, the green flag | (a click) | `pick_start` | chooses where a game starts and opens the Inspector at it |
| Inspector, HOW IT WORKS (the start) | across (x) · along (z) · facing | `start_x`, `start_z`, `start_yaw` | where a game starts and which way the walker faces |
| Inspector, HOW IT WORKS (the start) | Here: where the view looks · Face the way the view looks | `start_here`, `start_face` | the flag moved, or turned, as the view is |
| Inspector, HOW IT WORKS (the start) | seen through the eye · runs for, seconds | `start_eye`, `start_stamina` | what a game begins with |
| Inspector, HOW IT WORKS | Make it usable: make it… | `usable_preset` | a bed, a seat, food, a drink, medicine, a container, a place to drink at, a place to eat at, a weapon, nothing |
| Inspector, HOW IT WORKS | used, it does · its button says · not used up | `usable_effects`, `usable_as`, `usable_keeps` | the thing's own effects, in stats.txt's words |
| Inspector, HOW IT WORKS | Stats and statuses (stats.txt) | `usable_stats` | opens stats.txt (F10) |
| Characters menu | Stats and statuses … · What eating, drinking, resting do · Where a game starts, in the Inspector | — | stats.txt, goods.txt (F10); the start in the Inspector |
| Playing, top left | the stats' bars and the statuses | `stats_panel` | hunger, thirst, tiredness; bleeding 12s… |
| Playing, over things | ◆ marks | `notice_…` | what lies about to be taken, what holds something |
| Playing, the inventory | To eat, drink and use · Eat / Drink / Use | `good NAME` rows, `b_use` | goods with effects used one at a time; an item's effects |
| Playing, bottom right | the rounds and what loads them | `aim_ammo` | "Pistol 6 / 8   bullets 9" |
| Playing | the aim's mark | `aim_mark` | where the shot goes, as wide as its spread; red when empty |
| Controls | aim (right button, left trigger) | `aim` | aims what fires; the dash button fires, `use` (R) loads |

## 13zz4. A click shows how it works (2026-10-17, amendment 7bz)

| Where | Control | Name | Does |
|---|---|---|---|
| The view, left click on a thing | (a click) | `click_pick` | picks it and opens the Inspector on it; clicked again in the same place, what is behind it |
| View > Settings of this machine | A click on a thing opens the Inspector on it | `inspect_on_click` | on (the default): a click opens it; off: a click only picks (F7 opens it) |
| Inspector, top | HOW IT WORKS | `works` | what it is; its settings; what it does in play; what it goes by; where each is changed |
| Inspector, HOW IT WORKS | Its settings: 'NAME' | `works_settings` | a structure's own rows, the same as World outline's (built as, height, storeys, floor, roof, steps, gates, cells, walls inside, dress) |
| Inspector, HOW IT WORKS | What it does in play | `works_plays` | each line it and its working parts say, with its meaning; an act in words |
| Inspector, HOW IT WORKS | What it goes by: conditions and facts | `works_goes_by` | while / unless / needs / when / by / facts put / the eye's condition |
| Inspector, HOW IT WORKS | Its acts' text (F10) · Its design's text (F6) · Signals… · Its maker in World outline | `works_acts`, `works_kind`, `works_signals`, `works_maker` | where each is written and changed |
| A structure's settings (World outline and Inspector) | seen through: walls and roof go while the walker is behind or under them | `build_cutaway` | `cutaway 1` |
| A structure's settings | the eye inside it | `build_eye` | an eye of eyes.txt while the walker is in its cells; "not changed" for none |
| A structure's settings | when | `build_eye_when` | the condition for that eye, in `cond.gd`'s words; empty: always |

## 13zz3. Keys, drops, finds by chance (2026-10-17, amendment 7by)

| Where | Control | Name | Does |
|---|---|---|---|
| Inspector (F7), any thing of a kind: What is in it, what opens it | found by chance | `find_perhaps` | `perhaps GOOD N P`: N of a good in it one time in 1/P, drawn again each run; several with commas between; this one's own replace its design's all together |
| Inspector, the same | opens with | `find_key` | `key GOOD`: the good that unlocks it, kept and not used up |
| Inspector, the same | dropped by | `find_dropped_by` | `dropped_by NAME`: carried unseen by the one of that name until they fall, then lying beside them |
| Playing, E | "unlock the strongbox" · "search the crate" · "take what is carried by the bandit captain" | (offered) | new acts `unlock_key`, `search`, `search_unlocked`, `loot` |
| Playing | "Unlock the strongbox? You have no strongbox key." · "Search the crate? The crate has nothing." · "The bandit captain dropped a katana." | (told) | why not, and a drop |
| Library, Shelter | Crate of chance finds · Strongbox (opened by its key) · A door locked by a key | | new |
| Library, Creatures and enemies, Fighters | A bandit captain | | new: carries a key and coins |

## 13zz2. A boss of phases (2026-10-17, amendment 7bx)

| Where | Control | Name | Does |
|---|---|---|---|
| Inspector (F7), one of the folk who fights: How it sees, hears, goes and fights | patterns | `watch_pattern` | `pattern F MOVES`: its blows in turn below F of its health, a phase each; several with commas between; this one's own replace its design's all together |
| Inspector, the same | spent for, s | `watch_tires` | `tires S`: after its pattern it stands spent, the moment to strike it |
| Inspector, the same | weak | `watch_weak` | `weak behind N`, `weak tired N`: a blow of yours does N times the harm; both with a comma between |
| Playing | "You struck the warlord, from behind (N left)" · "…, while it is spent …" · "The warlord roars, and grows fiercer." | (told) | what a blow did, and a phase crossed |
| Library, Creatures and enemies, Fighters | A warlord | | new: a boss of three phases |

## 13zz. Barricades, locks, stamina (2026-10-17, amendment 7bw)

| Where | Control | Name | Does |
|---|---|---|---|
| World outline (Makers), the Player start | "How the game is seen at first, and run (2 settings)" | (section) | renamed from "How the game is seen at first (1 setting)" |
| World outline, the Player start | runs for, seconds (0: always) | `set_stamina` | `stamina`: seconds of running before the walker is winded |
| Playing | Breath ##########  (winded) | `fight_health` (a second line) | breath left, while the world sets a stamina |
| Playing | E at boards lying broken: "put up the boards" | (act `board_up`) | a piece of scrap; they stand again |
| Playing | E at a locked locker or door: "pick the lock of …" | (act `unlock`) | a lockpick used up; then "search" (`search_locker`) or "open" (`open_unlocked`) |
| Playing | E at a workbench: "make a lockpick at the workbench" | (act `make_lockpick`) | two pieces of scrap into a lockpick |
| Playing | the dash button, or a strike, at boards (`barricade`) | | one blow of so many |
| Library, Shelter (side-on) | Boards over a doorway · A locked locker · A locked door | | new |

## 13zy. Fights, building drawn on the land, houses of storeys (2026-10-17, amendments 7bt, 7bu, 7bv)

| Where | Control | Name | Does |
|---|---|---|---|
| Build (toolbar list) | *Draw on the land: click its places, Enter ends it (end by the first to go round)* | heading | new (7bu) |
| Build | **A stone wall, with a tower where it turns…** · **A fence…** · **A hedge…** · **A house of any shape: draw round it, its door in the first side…** · **A pond: draw round it…** · **A garden planted in rows: draw round it…** · **A paved yard or path: draw round it…** | `_draws` | the line tool; ended, a structure is added along the line, built as it says (towers, dressing, a door) |
| World outline (Makers), a drawn structure | "Drawn along its line: N sides, round" | `build` (section) | in place of "Its cells" |
| World outline, a drawn structure | **Its line's places…** | `build_line` | the line it follows in the Inspector, its places dragged on the land |
| World outline, a drawn structure | **Open or close gates…** / **Done with gates (Esc)** | `build_gates` | a click near a side: a gate in its middle (`line_gates`), or none again |
| World outline, a drawn structure | towers where it turns | `build_towers` | `towers` 1/0 |
| World outline, any structure | dressed along its foot: flowers · bushes · rocks · ferns · grass | `build_dress_WORD` | `dress` |
| World outline, any structure | built as: … a garden (an edging, beds planted in rows) | `build_as` | the sort `garden`, new |
| World outline, a building | storeys · stairwell | `build_storeys`, `build_stair` | `storeys` (1 to 6), `stair` (a cell's name; empty, its first) (7bv) |
| Inspector, a thing of a kind | How it sees, hears, goes and fights | `watch_head` | renamed from "How it sees, hears and goes" |
| Inspector | health · attacks · shoots · side · preys on · flees · keeps to, m · calls, m · rages below | `watch_health`, `watch_attacks`, `watch_shoots`, `watch_side`, `watch_preys`, `watch_flees`, `watch_defends`, `watch_calls`, `watch_enrages` | a line each, of its own (7bt) |
| Playing | Health ##### | `fight_health` | the walker's health, top left, while anything about can hurt (7bt) |
| Playing | (over someone hurt) ###-- | (a mark) | what they have left |
| Playing | the dash button, with someone to fight just before | (move `strike`) | a blow; otherwise the dash as before |
| Playing | E at a cupboard, toolbox, first-aid box | (acts `search_food`, `search_scrap`, `search_medicine`) | all of what it holds taken (7bv) |
| Playing | E beside one fallen | (acts `loot_food`, `loot_scrap`) | what they carried taken (7bt) |
| New world | Genre: Fights and creatures (the wilds) · Genre: A shelter seen side-on (2.5D) | `begins` | new genres |
| Library | Shelter (side-on) | (a collection) | new: things to search, rubble, a cot, a stove, a workbench, the dead at night |
| Library, Creatures and enemies | groups Fighters, Animals that hunt | | bandits, archer, brute; wolves, fox, deer, rabbits |

## 13zx. Connected worlds (2026-10-17, amendment 7bq)

| Where | Control | Name | Does |
|---|---|---|---|
| Inspector, In a level | leads to · worlds… · open it | `level_leads`, `level_worlds`, `level_open_world` | `leads WORLD [NAME]`; the world files there are; that world opened to edit |
| Inspector, In a level | gone through within, m | `watch_enters` | `enters R` |
| Playing, near a way | (the prompt) | — | "E  go to WORLD" |
| Playing, a world gone into | (words) | — | the world's name, a moment |
| New world | Genre: Connected worlds (a hex overworld) | `begins` | the overworld, its glade and the stealth level |
| Library | Ways between worlds | (a collection) | stone arch, cave mouth, signpost, marker of a hex |

## 13zw. Enemies by hand, rooms, cutaway (2026-10-17, amendment 7bp)

| Where | Control | Name | Does |
|---|---|---|---|
| Inspector, a thing of a kind | How it sees, hears and goes | `watch_head` | its design's (grey) and its own (cyan) |
| Inspector | sees, m · sees, wide ° · hears, m · sweeps, ° · sweeps, s | `watch_sees`, `watch_sees_wide`, `watch_hears`, `watch_sweeps`, `watch_sweeps_s` | numbers, each set as its own line; "design's" beside each |
| Inspector | hunts | `watch_hunts` | runs at the walker once seen and catches them |
| Inspector | alarm it raises · awake when | `watch_spots`, `watch_when` | `spots NAME`, `when COND` |
| Inspector | its round · give it a round / + place · - place · design's | `watch_round`, `watch_round_add`, `watch_round_take` | the marks on the world, dragged |
| Inspector, one that does not watch | give it sight and hearing | `watch_give` | `sees 12 90`, `hears 8` of its own |
| The view, a watcher picked | (yellow fan, blue ring) | `watch_sight` | how far it sees and hears |
| Inspector | In a level: room, version | `level_room` | `room NAME VERSION` |
| Inspector | show only version X of NAME / show every version | `level_show_only` | editing only, for looking |
| Inspector | seen through while the walker is behind or under it | `level_cutaway` | `cutaway` |
| Makers / Inspector, a structure | cutaway | (parameter) | 1: its building's walls and roof say `cutaway` |
| Playtest, The level | Rooms this run · Other rooms · Same each run · NAME: any/a/b | `rooms_line`, `rooms_roll`, `rooms_same`, `room_NAME` | the rooms of the run, chosen anew, kept, or fixed |

## 13zv. Side-on stealth: the eye, the watchers, a level, genres (2026-10-17, amendment 7bo)

| Where | Control | Name | Does |
|---|---|---|---|
| Playing | V | (the eyes) | now also `side`: side-on, A and D along, W and S a lane |
| Playing, a world with watchers | the line at the bottom | `watch_meter` | unseen / noticed / HUNTED, how well you are seen and why |
| Playing | the words at the top | `watch_banner` | seen, caught, hurt, a checkpoint |
| Playing | over a watcher | (a mark) | "?" noticed or searching, "!" hunting |
| Playing | on the ground | (the cones) | what each watcher sees, shortened by walls and by how you go |
| Playtest | The level: Side-on (2.5D) · Night · What each sees · each fact the watchers go by | `switch_eye`, `switch_night`, `switch_cones`, `switch_fact_NAME` | the level's switches while it runs |
| Playtest | the line under them | (words) | seen %, the last thing that happened, caught / hurt / broken |
| Playtest, Everyone | each watcher's row | (words) | [mode alert%] or [asleep] |
| World outline, the Player start | seen through the eye | `set_eye` | the eye a game begins with (eyes.txt): side for 2.5D |
| New world | the beginnings list | `begins` | now also "Genre: Side-on stealth (2.5D)" (genres.txt): a copy of a made world, no file |
| Library | Creatures and enemies | (a collection) | guard, the dead, stalker, hound, camera; grass, crate, acid, sludge, checkpoint; walls and platforms |

## 13zu. Styles (2026-10-17, amendment 7bn)

| Where | Control | Name | Does |
|---|---|---|---|
| Window > Styles; Sky > Styles…; Environment list | the window | `styles` (`styleview.gd`) | every style written, found, given, made one's own |
| Styles | Find a style | `style_find` | filters by name, says, after, tags |
| Styles | the list | `style_list` | ● the world's style; after which game; yours. Double-click gives it to the world |
| Styles | Give the world this style · No style | `style_use`, `style_none` | an edit, undone with Ctrl+Z |
| Styles | Try on a sample | `style_sample` | a copy of the island town, no file, given the style chosen |
| Styles | Open styles.txt | `style_text` | the file in a text editor |
| Styles | a name · Make my own from it | `style_copy_name`, `style_copy` | a copy, like it, yours, at the end of styles.txt |
| Styles, a style of yours | Tone; Exposure, Colour, Contrast, Bright, Cast, Glow, Shade, Corners, Grain, Flicker, Figures; Cast colour, Figures colour | `style_tone`, `style_exposure` … `style_figures`, `style_cast_colour`, `style_figures_colour` | each one line of the style, written; the world given it again |
| Styles, a style of yours | Take PART from STYLE · Take it | `style_take_part`, `style_take_from`, `style_take` | `takes PART from NAME` |
| Environment, Style | Styles… | `styles_window` | opens the Styles window |

## 13zt. The Character Studio: the shot camera (2026-10-17, amendment 7bm)

| Where | Control | Name | Does |
|---|---|---|---|
| The preview's row | Shot | `shot` | the preview through a lens and a frame, the thirds and safe areas over it; a move's shot taken when it is tried |
| Shot's row | Lens | `shot_lens` | mm, as on 35 mm film |
| Shot's row | the frame | `shot_frame` | 16:9, 2.39:1, 1.85:1, 4:3, 1:1, 9:16 |
| Shot's row | Guides · Follows it | `shot_guides`, `shot_follows` | the thirds and safe areas; the camera goes with the body |
| Shot's row | Keep as its shot · Forget it | `shot_keep`, `shot_forget` | `shot ...` written into the move chosen, or taken out |
| The preview's row | Above | `eye_above` | now from above (it was from under the floor) |

## 13zs. The Character Studio: the sixth hand test (2026-10-17, amendment 7bl)

| Where | Control | Name | Does |
|---|---|---|---|
| Clip library, its head | « | `library_fold` | folds the library to a strip at the left; the preview takes its width |
| Clip library, folded | » Clips | `library_open` | opens it again |
| Edit, In hand | blade as it lies there | `kept_box` | the thing in the hand turned toward how it lies where it is kept through the phase (`turn_to_stow`) |
| Edit | begins … %, ends … %; turn (°) x, y, z | `kept_from`, `kept_to`, `kept_x`, `kept_y`, `kept_z` (row `kept_amounts`) | how far turned at the phase's beginning and end; turned more on the way, the most at its middle |
| Walk, run, stand | rows walk_back, walk_left, walk_right, run_back, run_left, run_right | `look` tree | going that way while facing another; "(the walk, its legs turned)", "(the walk, played backwards)" until a clip is chosen |
| Walk, run, stand | All eight ways | `try_eight` | walks round every way facing ahead |
| Make, gait | Legs turned to the way it goes, facing another | gait `legs_turn` | how far the legs turn at most (degrees) when no clip goes aside |

## 13zr. The Character Studio after ActionForge (2026-10-16, amendment 7bk)

| Where | Control | Name | Does |
|---|---|---|---|
| Studio, left (side by side, workspace ≥ 1500 wide) | Clip library | `clip_library`, `library_find`, `library_word`, `library_grid` | every clip as a picture of the body in it; found by name or word; played as the pointer goes over; a click chooses it |
| Clip library | For the walk/run/stand row, For the phase edited | `library_use_way`, `library_use_phase` | the clip chosen given to the row chosen (and its start/stop) or to the phase being edited |
| Tabs | titles | (node names unchanged) | Moves, Combos, Edit a move, Buttons, What it carries, Walk, run, stand, Clips (list), When struck, Bones |
| Moves | What it is | `moves` column 1 | the move's `says` (moves.txt) and its length (was Timed) |
| Edit, In hand | hand to where it's kept | `stow_box` | the holding hand brought to where the thing is kept through the phase (`hand_to_stow`) |
| Edit | begins … % there, ends … %; move the spot (cm) right, up, ahead | `stow_from`, `stow_to`, `stow_right`, `stow_up`, `stow_ahead` | how far there at the phase's beginning and end; the spot moved |
| Preview | the yellow ball | `stow_ball` | the spot, while the phase edited says hand to where it's kept |
| Walk, run, stand | One part plays another clip: Play the chosen clip on it, Stop it | `way_over`, `way_over_off` | renamed from Over it, Lay the clip chosen over it, Take off |
| Edit, More posing tools | turn about the part itself, only at this moment, both sides at once; One part | `own_axes`, `key_here`, `both_sides`, `over_lay` | renamed (was its own axes, key at the playhead, both sides, Over it) |

## 13zq. The Character Studio: the animator pass (2026-10-16, amendment 7bj)

What is laid over the clips (`overlay.gd`), reached in the Studio. Names are the controls' `name`s.

| Where | Control | Name | Does |
|---|---|---|---|
| Edit, Adjust pose | part | `bend_bone` | now also each finger (Right thumb … Left little finger: its three bones bent as one) |
| Edit, the preview | a click on a joint | (the preview) | while a phase is edited: the part under the pointer chosen in `bend_bone` (a click, not a drag) |
| Edit, Details (top) | its own axes | `own_axes` | the turn about the part's own axes (`local`), not the body's |
| Edit, Details | key at the playhead | `key_here` | the turn said for this moment of the phase (See it at): `at P`; between keys it goes from one to the next |
| Edit, Details | both sides | `both_sides` | the other side's part turned the same, as in a mirror |
| Edit, Details | Copy pose, Paste, Paste flipped, Mirror L→R, R→L | `pose_copy`, `pose_paste`, `pose_paste_flipped`, `pose_mirror_lr`, `pose_mirror_rl` | a phase's turns copied and pasted (left for right), one side's given to the other |
| Edit, Details | Poses | `pose_pick`, `pose_lay`, `pose_name`, `pose_save` | a pose of one's own (`pose NAME` in characters.txt) laid on the phase; the phase's whole-phase turns kept as one |
| Edit, Details | Over it | `over_region`, `over_lay`, `over_off` | the clip chosen under Clips played over a part of the body while the phase is shown (`over CLIP REGION`), or taken off |
| Ways of moving | Over it | `way_region`, `way_over`, `way_over_off` | the same for the row chosen (a walk with a torch's arm) |
| Clips | play as the pointer goes over | `hover_play` | on: a clip plays in the preview as the pointer goes over its row |
| Clips | word buttons | `tag_chips` | every clip, and the words most clips are found by, each with its count: one click shows only those |
| Clips | the list | `clips` | a new column, **Leads to**: what a clip goes on into, by the names (a start to its loop, a loop to its end, a cut to its recovery) |
| Make | Bearing | `make_bear_posture`, `…_chest_turn`, `…_head_up`, `…_head_turn`, `…_head_tilt`, `…_shoulders`, `…_arms_out`, `…_arms_ahead` | how the body is held over whatever it plays (`bearing NAME DEGREES`; the arms only over standing and going) |
| Make | Gait | `make_gait_style`, `make_gait_stride`, `…_cadence`, `…_bounce`, `…_sway`, `…_arms`, `…_lean` | how it walks and runs (`gait NAME VALUE`); a style puts its numbers in and writes them |
| Make | Follow-through | `make_follow_head`, `…_chest`, `…_arms`, `…_hands`, `make_feel` | what comes after the body (`follow PART AMOUNT [feel F]`) |
| Make | Feet on the ground | `make_feet` | unticked: `feet off` |
| Make | Reset how it carries itself | `make_reset_carry` | every bearing, gait, follow and feet line taken out |

Edit's Details is now titled "Details — the pose's tools, what it all means, the clip's seconds, the text" (the tools at its top, so a phase's numbers stay in sight).

## 14. What the engine can do, and whether the editor reaches it

From an audit of the host's commands (`editor/host/src/lib.rs`, about 80 of them), the
makers' declared parameters (`shapes/land.rs`), the editor's scripts and `run.ps1`,
2026-10-11. "Reachable" means from a menu, a window or a button, with words that say
what it does.

### A. Reachable and discoverable before this pass

Worlds: new (three beginnings, seed, hills, size), open, save, save as, the guard on
leaving unsaved changes. Makers: add (land, line, street, house, furniture, woods,
meadow, flock, mooring, start, errand, finds, pen), select, rename, move, turn, reach,
connect to another land or line, duplicate, remove. A wood's or meadow's area: reach
on its own level, keep to a level, paint and rub out. What a wood, meadow or street
is made of: designs with shares. One thing: pick, move, turn, take away, duplicate,
another design, size, a shape of its own, one more for an arrangement; put back by
sort. Designs: list, place, give, make mine, shape. Kinds and meshes as text; meshes
dragged by corner and face. The land: raise, lower, smooth, flatten, paint, clear, let
grow, cliff, ledge, restore. Styles by name; an hour on; the next weather. Playing,
walking as a kind, where a game starts, saving and loading a game. Undo and redo of
all of it.

### B. Implemented before, newly reachable in this pass

| Capability | Was | Is now |
|---|---|---|
| Every maker's own parameters, explained | Inspector only, as `pitch`, `amp`, `keep_off`…, tooltip "a parameter of the generator" | Makers, in the part named for the maker: a name in words, what it does, sensible bounds |
| A house's width, depth, wall height and roof colour | the same | the same part, for a house put down alone |
| The land's seed, hills, size and road margin after it is made | the same | the same part, for the land |
| Lay a maker out again (next seed) | Inspector "another seed" | Makers "Lay it out another way" |
| Freeze / thaw a maker | Inspector | Makers |
| Each single change by hand, listed, shown, put back alone (`changes`, `restore` by record) | the host had the list; the editor showed only counts and "put back all of a sort" | Makers > Its arrangement > Each change, one by one |
| A weather by name; the world at a chosen hour | `weather.become` and the clock were only reached by "next" and "an hour on" | Environment menu |
| Making or changing a style | `styles.txt`, known only from documents | Environment > Open styles.txt |
| The built-in walker again | a second press of "Walk as this" | Characters menu |
| Starting on a named world | the script argument `--world`, which `run.ps1` did not pass | `run.ps1 -World FILE`; and the editor starts on the last world |
| Earlier versions of a world | none were kept | `backups`, and World > Open an earlier save |

### C. Implemented, still not reachable, or not suitable for the interface

| Capability | Why it is not exposed | Where it would belong |
|---|---|---|
| `compose` (several makers added as one edit) and `begin` with steps given; `recipe` (the steps a built-in beginning is) | Plumbing with nothing a person can author yet: there is no command that writes a world's makers out as steps, so there is nothing to save, name or put down again. Exposing it now would be a text box of JSON | World > New world ("begin from a composition of yours") and a "Save these makers as a composition" in Makers, when compositions are kept (E3 / the old W2) |
| `review`'s `accept`, `forget`, `dismiss` (what to do with a change by hand that landed elsewhere or found no place) | "What needs a look" (I) lists them as text only. The three commands are there, but what each should be called and when it is safe needs a pass of its own: `forget` deletes a record of the person's | A small window for "What needs a look", with a button on each row |
| The standalone game (`editor\export.ps1`, `run.ps1 -Game`) | Builds from `editor\world.json` only, takes minutes, and runs a script outside the editor | World > "Build a game from this world…", once it builds from the world open |
| Demos (`run.ps1 -Demo NAME`, 44 of them) and tour parts | Each replaces the world in the editor and drives it by script from the start; they are started with the editor, not inside it | A "Demos" list in the Checklist window that says the command; or a launcher |
| `check_upgrade`, `upgrade`, `upgrade_all`, `generator` (a maker's code changing under a world) | Only the prototype's test generator has a second version; nothing in a real world can be upgraded yet | Nowhere yet |
| `slow`, `hold`, `budget`, `history`, `check`, `scene`, `obstacle`, `-Villages`, `-Measure`, `-Weigh` | Measuring and test instruments | Engine status, if ever |
| `editor\ask.ps1` (another program asking the host) | A developer's door | — |
| The raw text parameters `of`, `each`, `added`, `on`, `what`, `variant`, `kind`, `wide` | Each has a proper control elsewhere (What it is made of, This one…, rests on); typed by hand they are easy to get wrong | They stay in the Inspector |
| The quick keys 1–9 and O, the orbit choice (P) | Reachable, but only said in Keys | A Preferences place |

### D. Partly implemented, or needing more work to be dependable

- **The land's tools under a terrain tile**: they shape and paint the land under a
  tile, the tile's ground follows, and the window now says so and shows the tool on
  the tile (13i). **Paint is still hidden by the tile**, by the user's decision
  (2026-10-12): a tool for a tile's own surface is to be another tool. A tool's
  outline is still hidden by a tree or a hut standing in front of it.
- **How many things a wood or a meadow makes** is about three in four of its number
  over a reach, and about its number where painted. **On tiles it is the number
  asked, on each tile** (13i). Tiles are chosen one click each; a region is not yet
  something with a name.
- **Detach** (X): works and is undoable, but a detached thing has no home in Makers.
- **A street given designs has no houses** (it is one or the other): known, recorded.
- **Freeze** keeps exactly what was made: land shaped under frozen things leaves them
  where they were, above or below the new ground. Said in the button's tooltip.
- **A new seed** on a wood, meadow or flock moves changes by hand onto other things
  (they are kept by name). Said when it happens; not prevented.
- **Shape by hand** from the row under the bar and the Things menu still does not say
  whether one thing or a whole design will change until the strip opens. The two
  buttons in Makers do.
- **What needs a look** shows text and offers nothing to do about it (see C).
- **The hour and the weather set while editing are a look**: a game starts in the
  morning in the first weather, whatever was shown.
- **A style is edited in a text file**; there is no way to choose a style for one
  region, or for one kind of thing.
- **The Makers window still does five jobs** (adding, listing, one thing, a maker, a
  maker's removal). This pass gave it two more parts; they fold. Its regrouping waits
  for the review this inventory is for.
- **The bar does not wrap**: in a window narrower than about 900 px its last buttons
  go off the edge. (Found in this pass; the World heading shortens itself to avoid it.)

### E. Planned, not existing

Named groups of designs (D3); things painted with a brush from a design or a group
(D4); an area drawn as an outline, or by slope, height, surface or distance from a
line (A1); one thing's colour or other numbers (D2); a style for a region, per-category
style choices, a style's lists edited in the editor (S1); one Variation System with
locks and previewed alternatives, and colour variation that keeps an authored palette
(principle R); a street that holds houses beside trees, rocks and decorations
(principle Q); the plant, house and decoration engines (E1 to E3); compositions kept
and reused; lines that are not roads; mazes and obstacle courses; behaviour (7g);
shaping tools beyond corners and faces (7f); docking; a Preferences place.
