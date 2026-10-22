# Modular construction: feedback, the direction it is to take, and what resuming it needs

**Status: resumed by the user on 2026-10-18** (brief "Integrated, Context-Aware Modular
Construction", Tiny Glade's interaction design as the reference). The first pass after
resuming is amendment 7ck: its audit, what it changed and what it leaves are in section 6
below. It was postponed on 2026-10-15; drawn structures (7bu) were built on 2026-10-17
under the user's autonomous brief of that day. This file is the canonical record of the direction,
so the work can be resumed without reconstructing the discussion. It was first written
2026-10-14 from the user's brief of that day and rewritten 2026-10-15 from their second
brief, which asked for the requirements, what exists, the dependencies, the decisions
and the acceptance criteria to be kept here. What was built is recorded in the
amendment (`Draw3D_Roadmap_Amendment_2026-10-09.md`, 7ad and 7af) and in `PROGRESS.md`
("Adaptive modular construction, first tier" and "Modular construction brought to a
stopping point").

## 1. The intended experience, and the principles it follows

A **tool-led, spatial building workflow** in the manner of Tiny Glade and Townscaper,
not a collection of separate procedural building systems. The person chooses a
construction tool, draws or drags in the world, and gets a coherent result that they
can then refine in place.

What the user found (2026-10-14): the construction tools built (7ad, 7af) are
functional groundwork, but they feel like disconnected, basic systems. Each is a maker
with settings, chosen and set one at a time, rather than one way of building.

The requirements for the milestone when it resumes:

1. **Spatial authoring first.** Structures are made by direct interaction in the world:
   dragging out a footprint, drawing a wall, a fence, a path. Text and property panels
   are a way to refine, not the main way in.
2. **Integrated construction.** Walls, fences, hedges, floors, roofs, openings, steps,
   ramps and paths work together as parts of one coherent structure, not as isolated
   generators.
3. **Context-sensitive results.** Construction adapts to corners, joins, neighbouring
   pieces, enclosed areas, terrain, changes of elevation and openings. A shared
   boundary never gives duplicate or conflicting geometry.
4. **Editable output.** What is made stays understandable and editable after it is
   made: footprint, dimensions, pieces, openings and appearance are adjusted without
   throwing the structure away and making it again.
5. **A reusable design language.** Pieces come to use the engine's style and design
   systems, not looks written in code. People make and reuse their own architectural
   pieces and styles.
6. **Terrain integration.** Slopes, raised ground, paths, ponds and several levels are
   accounted for. Dependencies on terrain elevation and water are named and waited on,
   never covered by shortcuts that would have to be replaced.
7. **Persistence and safety.** Structures save and reopen with their arrangement and
   edits whole. Making something again never silently overwrites a deliberate change by
   hand.
8. **General purpose.** Different architectural styles and kinds of space, not one
   village or one demonstration house.
9. **Demonstration and validation.** On resuming, a meaningful playable or explorable
   example is built, and the authoring workflow itself is tested, not only whether the
   meshes look right.

## 2. What exists, and what remains (the local repository, 2026-10-15)

Checked against the code (`editor/host/src/layout.rs`, `land::build` in
`editor/host/src/shapes/land.rs`, the Build list in `editor/godot/desk.gd`, World
outline's "Its cells" in `editor/godot/makers.gd`) and the amendments. Compare each
item with the code again when resuming: this list is a snapshot.

**Built** (7ad, 7af; the village showcase `editor\showcase-village.json`):

- A **structure on cells**: one maker (`structure`) whose arrangement is cells of a
  square or hex layout, built `as` wall, fence, hedge, building, pond or none (a floor
  alone: a path, a street, a square, a terrace).
- **The rim** of its cells built round, holes included; piers at the joints of a wall
  or fence; a hedge's corners where it turns.
- **Gates and doors** said by a side (`cell side`) and kept by those words; a gate on a
  side that is not on the rim waits rather than being lost.
- **Walls inside** between two of its cells (rooms, pens, a maze), doors in them.
- **Buildings**: the house generator's facades on the rim (windows by bays, a door
  where a gate is said), corner posts, a footing where the ground falls away.
- **Roofs** over each rectangle of its cells (an L is two roofs), ridge the longer way:
  none, tiles, slate, moss, thatch.
- **Steps or a ramp** wherever a floor stands more than a step above what is beside it
  and no wall is in the way (`steps: none` turns them off).
- **Floors**: paving, gravel, boards, garden soil.
- **Raised ponds**: a kerb, a bed, water at one level that things swim in.
- **Shared sides built once**, by the structure that is more of a building (`rank`),
  then the first made; a gate either says in it is a gate of the one that builds it.
- **Hand changes kept by piece name** (each piece is named by its cell and side).
- **In the editor**: Build lists a house, a wall, a fence, a hedge, a path or square, a
  pond, each made as it says from the first click; Its cells (paint cells, walls
  inside by clicks, gates and doors by clicks, roof, steps, height).
- **What other systems read**: a structure is a place by its name (`goes 7 at the
  square`); doors and gates are ways because they open; pieces are things with traits.

**Not built** (owed when it resumes, roughly in the order use will ask):

- The **tool-led workflow** itself: a footprint dragged out, a wall drawn along a line,
  the result adapting as it is drawn. Today cells are painted, then words are set.
- **Storeys and stairs inside** (a layout above a layout: 7o's levels).
- **One roof over an L or a U** (valleys, hips), and a chimney placed once.
- **Openings placed by the person** (windows and doors on a facade's own handles).
- **Pieces as library designs** (a wall piece, a pier, a gate of the person's own) and
  **pieces chosen by their neighbours** (an end, a corner piece, an arch).
- **Paths along a line** rather than cell by cell, and paths that join.
- **Ponds and streams in the land**, which need water with a level of its own.
- **Styles reaching the pieces** (dry stone, brick): today the pieces take the style's
  stone, wood and greens, and the stone wall's look is code.

## 3. Known limitations, dependencies and a likely order

**Limitations of what stands** (from 7ad and 7af):

- Every structure is built again whenever its rim changes; one of hundreds of sides is
  a few hundred pieces, measured nowhere beyond the tours' sizes.
- A side shared by two buildings gets one facade, whose windows look into the
  neighbour.
- A gate's leaf is hung for a person's height, not the wall's.
- Steps can reach into a neighbouring structure's cell.
- A structure added from Build is named "structure N".

**Dependencies on other systems** (not to be faked inside construction):

- **Terrain elevation**: a tile's own height (7w) is built and distinct from the land;
  floors follow it. Several levels need 7o's layout above a layout.
- **Water with a level of its own** (7q, seventh): ponds and streams in the land wait
  for it. A pond is not to be made as a sheet laid on a cell.
- **A region as a thing** (7q, second) for a footprint that is not cells.
- **The design and style systems** (D1, the library, principle R's variation) for
  pieces of the person's own and styles that reach geometry.
- **Navigation and behaviour** already read structures by place and doors by trait;
  whatever a new tool makes must keep that.

**A likely order when it resumes:** (1) the tool-led workflow over the structure there
is: drag a footprint, draw a wall or path, with the result shown while dragging; (2)
openings placed by hand on a facade; (3) pieces as library designs, then chosen by
their neighbours; (4) one roof over an L; (5) paths along a line; (6) storeys and
stairs, after layouts above layouts; (7) ponds in the land, after water with a level.

## 4. Decisions made, and decisions still the user's

**Made, and to be kept** (7ad, 7af, 2026-10-14; the user tried 7ad and found it
adequate):

- One maker for walls, fences, hedges, floors and buildings, told `as`; a structure
  follows its cells, not its node.
- All cell arithmetic lives in `layout.rs`; every piece is named by cell and side; a
  hand change is kept by that name.
- A shared side is built once, by `rank`, then the first made.
- A building's sides are the house generator's facades; its walls stand at its floor.
- Steps appear by themselves where a floor is high.
- A roof is laid over rectangles of cells.
- A pond is raised, not dug: the land layer is never dug by a structure.
- A painting or dragging tool is a new way of telling a structure its cells, not a new
  kind of structure.
- Regeneration never silently overwrites a deliberate hand change (the engine's rule,
  7aa and M3).

**Still the user's to decide when it resumes:**

1. Whether the tool-led workflow keeps cells underneath (a footprint dragged becomes
   cells of a layout made for it), or construction also works free of any layout. This
   affects architecture and what existing worlds keep: it should be decided first.
2. Whether a building's look stays the house generator's facades or becomes pieces of
   the person's own from the library first.
3. How far a structure adapts to its neighbours by itself (a path meeting a door turns
   to it; a garden wall meeting a house joins it) and how much is said by hand.
4. Which architectural styles the first example should cover, and what the playable
   example is to be.

## 5. Acceptance criteria for resuming

The milestone is begun again only when the user asks. It is accepted when:

1. A person makes a house, a garden wall, a path and a pond **by choosing a tool and
   dragging in the world**, with no text typed and no property panel needed to get a
   coherent result.
2. Pieces **meet coherently**: corners, joins, shared sides and openings give no
   doubled or conflicting geometry, and a path that meets a door, a wall that meets a
   house, join.
3. The result **adapts to terrain**: slopes, raised tiles and steps, without digging
   the land and without shortcuts around the missing water level.
4. Anything made is **edited afterwards**: footprint, size, an opening, a piece, its
   look, without throwing the structure away.
5. **A hand change survives** making the structure again, saving and reopening.
6. At least **two styles** are shown, and pieces of the person's own can be used.
7. A **playable or explorable example** is built only with the editor's own tools, and
   the people and animals of the village systems use it (doors as ways, places by
   name).
8. The **authoring workflow is tested end to end** (a protocol test and a tour part),
   apart from looking at the meshes, and the existing village and worlds still open the
   same.

## 6. Resumed 2026-10-18: the audit, and the first pass (amendment 7ck)

The user's brief of 2026-10-18 ("Integrated, Context-Aware Modular Construction") asked
for construction to be one spatially aware workflow: choose a tool, draw or drag, the
engine interprets the action in its context, the result stays editable and undoable. It
named buildings, gardens and vegetation, ponds and water, paths and connections,
selection and undo, and a coherent style; it asked for an audit first and the most
valuable vertical slice, not breadth.

### What was there when work began (checked in the code, not the roadmap)

- **Drawn structures (7bu)** in `editor/host/src/shapes/land/build.rs`: a wall, fence,
  hedge, house of any shape, pond, garden or paved yard drawn with the line tool
  (Build > Draw on the land), in pieces of no more than 3 m named by side and piece,
  towers, dressing along the foot, gates by a click (`line_gates`), roofs that fit.
  **Each planned alone**: a structure knew only its own line.
- **Structures on cells (7ad, 7af)** know each other, but only through shared cells
  (`rank`, `tileops::settle_makers`), not through space.
- **Growth** (woods, meadow, field) kept off the land's roads, water and bare ground,
  and nothing else: a meadow over a drawn house grew grass inside it, on its wall's
  foot and in its doorway. The brush (`stroke`, 7x-7y) kept off water only. A
  structure's own dressing was put in front of its own gates.
- **There was no path along a line**: a paved drawn line had to go round (a yard); an
  open line built as nothing made nothing.
- **Ponds**: a drawn pond was a stone kerb, a bed and water at one level (raised, not
  dug: decided in 7af), with rocks, grass and ferns at its foot. **The earlier pond with
  frogs, fish, lilies, reeds, ducks and dragonflies is the island town's** (commit
  612ef72). It was made by the script `editor/host/examples/town.rs`, which dug the land
  with `sculpt` and put every lily, fish and frog by hand; those designs lived only in
  `editor/godot/town.txt`, not in the library. No tool could make it: that is why the
  ponds made with the tools were simpler. The life itself (`swims`, `hops`, `flutters`
  in `wild.gd`) and a drawn pond's water (its `water R D` trait) already worked
  together; nothing put the life there.
- **Undo**: drawing a structure was two steps (its line, then the structure); the
  editor said "Ctrl+Z twice". `together` (7ce) existed and only the Assistant used it.
- **Selection**: a click picks one piece and chooses its maker in World outline (W1b);
  Delete takes that piece away alone and the maker keeps the gap by its name; a drawn
  structure moves by its line. This works and was not changed. A drawn structure was
  named "structure N".

### The friction points, in the order they cost a person most

1. Drawn things ignored each other: a path through a wall stayed blocked; plants grew
   through houses, paths and doorways.
2. No path along a line; no border along a wall.
3. A pond made with the tools had no life, and the life there was could only be put by
   a script.
4. One gesture, two undo steps; names that say nothing ("structure 3").
5. A garden drawn round a house laid soil under the house.

### What 7ck changed (all general, no showcase logic)

- **One shared picture of what is built where**: after every edit the host works out,
  from the structures drawn on the land, each one's inside (house, pond, garden, paved
  yard), every wall's foot, every path, and every way through (a gate or a door and
  2.6 m of ground either side), and writes it into the document as one text
  (`shapes::land::BUILT`, `built.clear`, as tile heights are written as `tiles.raised`).
  Woods, meadows, fields, a structure's dressing, a garden's rows, a pond's bank life and
  the brush keep clear of it. A maker's own places are not kept from itself (a wall's
  flowers stand by its foot), but nothing is planted in a way through, its own included.
- **Paths that open what they cross** (`tileops::settle_drawn`): a path is a line drawn
  and built as nothing round it with a floor (Build > "A path"), a strip that lies on
  the ground in mitred pieces. Where it crosses a drawn wall, fence, hedge or garden
  edging, the piece it crosses gets a gate; where it crosses a house's wall or ends
  within 1.2 m of its side, a door. These are derived parameters (`opened`,
  `opened_by`), apart from the gates the person asked for (`line_gates`), which are
  never touched. Move the path and the openings move; take it away and they close;
  "paths drawn across it open a way" unticked (`opens 0`) keeps a structure shut.
- **A border**: a garden drawn along a line and not round is a strip of soil planted in
  rows, with no edging along its middle.
- **A garden round a house** plants round it and lays no soil under it.
- **A pond with life**: `edge bank` (a low broad bank of the land's own grass running
  down under the water, in place of the stone kerb; still standing on the land, never
  dug), `life` (designs in its water), `bank` (designs along its bank), `lively` (0 to
  1). In the water, what swims is put under the top by as much as it swims over its
  place, what floats on the top, the rest on the bed. Sets in `editor/godot/ponds.txt`;
  the designs are the library's new group "Pond and water's edge" (rural.txt: lily
  pad, water lily, reeds, bulrushes, frog, goldfish, duck, dragonfly, two butterflies),
  brought into the world when a set is chosen.
- **One drawing is one undo step**: the designs brought, the line and the structure
  (`makers.gd` `drawing_mark` / `drawing_done`, host `together`). A drawn structure is
  named for what it is: house, stone wall, path, border, garden, pond, yard.
- **In World outline**, a drawn structure shows its path's or border's width, "paths
  drawn across it open a way" and what opened it ("Opened by path, in piece s0_5"), a
  pond's edge, its pond-life set, its two lists of designs and how lively it is.

### Decisions taken to build it (the user's to overrule)

1. A path opens what it crosses **by itself**, with a per-structure switch to keep it
   shut, rather than asking each time. Predictable (the opening is where the path is),
   reversible (it follows the path), and the person's own gates are never touched.
2. **A path ending within 1.2 m of a house's side opens a door there**; ending near a
   wall, fence or hedge does not (only crossing does).
3. **A path across a house goes in by one door and out by another.**
4. **What a person put is not taken away** when something is built over it: what was
   painted with the brush, or put down by hand, stays where a house is drawn later (the
   brush keeps clear only as it paints). Generated growth (woods, meadow, field,
   dressing) does move out of the way.
5. **Ponds stay raised** (7af): the bank is a mound on the land, its inner slope under
   the water. Digging is still a decision owed (below).
6. Only **drawn** structures took part at first; **structures on cells were added the
   same day at the user's word** (their insides, walls' feet and gateways from `over`
   and `rim`). They keep their own cell-based joining (7af); a path drawn on the land
   does not open a gate in a structure on cells (that is still by clicks).

### Still owed, in the order use will ask

1. ~~Structures on cells in `BUILT`~~: done the same day (worlds on cells now open
   with nothing grown in their houses). Still owed of it: a drawn path opening a gate
   in a structure on cells.
2. **A path that turns to meet a door** (its end drawn to the nearest door or gate), and
   paths that join one another (a fork, a crossing) with one surface.
3. **A pond dug into the land**: it needs a structure to own a lowering of the land, or
   water with a level of its own in the land (7q, seventh). The user's call: 7af says
   never dug.
4. **Steps where a path climbs** a slope too steep to walk.
5. **A garden painted** with the brush as its outline, not only drawn.
6. **Styles reaching the pieces**: the bank and the path take the style's colours; a
   wall of the person's own pieces (dry stone, brick) does not exist yet.
7. **A preview while drawing**: the line is drawn as it is clicked; what it makes appears
   on Enter.
