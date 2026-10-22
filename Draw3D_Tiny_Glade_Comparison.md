# Tiny Glade and Draw3D's construction, compared by fact

Written 2026-10-18, after the user rejected 7ck as "not how Tiny Glade builds".

## Sources

**Read:**
- the video transcript the user gave
  (github.com/satishtahal-alt/test, `tinygladetranscript.md`);
- two Steam player guides, "Builder's Basics Guide" and "Tiny Glade – on the hunt for new tricks";
- the 80 Level interview with the developers.

**Not seen:**
- **The developers' talk** ("Rendering Tiny Glades With Entirely Too Much Ray Marching"): I could get only its title. Its subject is rendering.
- **Tiny Glade itself:** it has not been run on this machine.

**Draw3D facts** are from its code: `marks.gd`, `desk.gd`, `terrain.gd`, `build.rs` and `tileops.rs`.

## How Tiny Glade builds

The whole game rests on four rules.

1. **You draw by dragging, never by entering points.**
   - A wall or fence is a left-drag: drawn slowly it gets few control points, drawn fast it gets more.
     Two points make a straight wall; points close together make a rounded corner; points far apart make a
     gentle curve; a sharp turn makes a corner.
   - Paths, flowers, water and terrain are brushes with a size.
   - A building or tower is a click-and-drag that sets its size before you let go.

2. **Everything stays a shape with handles.** Right-click any structure to see them:
   - **Building:**
     - a cross that moves the whole building;
     - an arrow on each of the four sides that pushes that side in or out;
     - a corner arrow that turns the building;
     - on each side, a dot that raises the base onto stilts and a dot that sets the height;
     - an arrow that raises or lowers the roof, and circular arrows that turn the roof.
   - **Roof:** from the normal view, drag the roof itself to make it bulge out or curve in.
   - **Wall or fence:**
     - dots at the base that move each control point;
     - an arrow at the top for the height;
     - hovering just under the top cuts the wall in two;
     - two walls of the same kind can be joined.
   - **Stairs:** a dot per point, height and width arrows, and a "+" on the curve that adds a point.
   - **Terrain:** a dab raises the ground. Over the raised patch, dragging up or down sets its height and
     dragging sideways sets the steepness. Right-click picks up a patch to move it.

   While a handle is dragged, the structure is **regenerated live every frame**: the stones of the wall, its
   arches, the roof, the windows and its supports. The developers say the generation runs in real time on the
   GPU, with a separate generator for each kind of element. "Watching walls rearrange and build up in front of
   the player" is, in their words, the core pleasure.

3. **The context comes from overlaps, worked out live.**
   - **A path brushed across:**
     - a wall: a door, or an arch when the path is wide or crosses a corner;
     - a fence: an arch;
     - water: stepping stones;
     - a slope: steps;
     - a low building without a roof: stairs;
     - a building standing in water: a water grate.
   - **Arches:** the pillars are shaped by erasing bits of the path under the wall. Hover inside the arch to set
     its height; one height is shared by every opening in that building.
   - **Buildings:**
     - placed against each other, they snap into line and their roofs join;
     - raised, they grow supports that depend on how far they overhang;
     - moved over a stair platform, they take away its supports.
   - **Windows:**
     - dragged onto a wall: three side by side make a double, then a triple;
     - dragged low, or onto a building that meets another, they become doors;
     - on a flat roof, a trap door; near the roof, a dormer.
   - **Small things around windows and doors:** benches, pots and hay appear by them. Each is linked to its
     window or door: click that window or door for another one, or unlink the thing and move it.
   - **Chimneys:** one placed by a window becomes a metal chimney.
   - **Flowers:** the flower brush stays off paths; on water it lays lily pads.
   - **Ivy, moss and damage:** these grow by themselves.
   - **Snapping:** Ctrl turns it off.

4. **Every gesture is one step, undone and redone at once.** Hold Z to rewind, hold X to replay.
   The hammer removes a whole structure; the eraser removes part of one (a gap in a wall makes a ruin).

## What Draw3D does, side by side

| Tiny Glade | Draw3D today (from the code) |
|---|---|
| Walls, fences, hedges drawn by dragging; how fast you drag sets the points | Separate clicks, Enter to finish (`marks.gd` `press`: `draft.append`, one point per click) |
| A building dragged out to its size | A building drawn by clicking its corners, or by clicking grid cells |
| A live structure while you draw | A thin line while you draw; the structure appears only after Enter (planned by the host) |
| Handles on each side, height, roof, stilts, turn, move | **None on structures.** The only drag handles (`marks.gd` `grab`) move a maker's middle or its reach. Everything else is numbers in World outline |
| Live regeneration while a handle is dragged | Regeneration after a change of numbers, a round trip to the host (about 11 ms of planning per house, measured) |
| A path is a brush with a width | A path is a clicked line (7ck); roads are clicked lines |
| Path through a wall → door or arch, live, height draggable | Path across a drawn wall → a gate piece, after Enter (7ck). Not live; no arch shape or height handle; structures on cells not opened |
| Path over water → stepping stones; on a slope → steps | Neither |
| Windows are things you place and stack; low ones become doors | Windows are set by the generator; doors on cells come from clicking a side (`gates_begin`) |
| Small things linked to a window or door, swapped by a click | Dressing words along a line (`dress`), not linked to anything |
| Buildings snap, join roofs, grow supports | Joining only within one structure on cells (7af); drawn ones don't join |
| Water: a brush that digs below ground level | Ponds as structures (raised, with a kerb or bank); the land's own water level. No water brush |
| Terrain: dab, then drag its height and steepness | Raise, lower, smooth, flatten, cliff and ledge brushes (`terrain.gd`) with numbers for strength; no drag on the dab |
| Flowers: a brush that keeps off paths | Paint with a design (a brush); meadows keep out of what is built (7ck) |
| Eraser removes a part (ruins); hammer a whole | Erase brush for things and tiles; "take away one piece" of a structure by numbers |
| One gesture, one undo; hold to rewind | One drawing, one undo (7ck); Ctrl+Z one step at a time |
| Walk inside what you built | The walker walks in, through doors (7ag) |

**Bottom line:** Draw3D has some of the *rules of context*: a path opens a gate, plants keep clear, a pond gets
a bank. It has none of the *interaction*: drag to draw, handles on everything, live rebuilding while you drag.
Tiny Glade is the interaction. 7bu and 7ck both built on Draw3D's existing route of "click points, ask the host,
see the result", which is why both missed.

## Built since (7cl, 2026-10-18, at the user's word)

The first tier is built: walls, fences and hedges.
- **Round trip measured first.** One live step, the whole wall made again by the host and drawn by Godot, takes about 7–10 ms, worst 24 ms. So the preview is the real wall, with no stand-in.
- **Drag to draw:** the wall is built every frame as the pointer goes. It curves through the places the hand bends at, and a sharp turn makes a corner with a tower. Ending where it began, it goes round. One step of undo; Esc gives it up.
- **Handles:** dots on its places and a point over its middle for its height. Each is dragged with the wall rebuilt every frame, and each drag is one step of undo.

Then 7cm (the same day, at the user's "continue with the next step", with restyling in mind):
- **Its own look:** a palette, a colour for each part, a picture over it and over its towers' roofs, the size of its stones; towers roofed as a cone, a spire, a dome, a flat lid, battlements, or open.
- **More handles:** a green ring and point on its towers (wider, taller), built again as they are dragged; a "+" between places that puts one in; Alt and a click takes one out.
- **Snapping:** its ends, and a place dragged, go onto another drawn wall; Ctrl held, they don't.

Then 7cn (2026-10-19, at the user's "proceed with the next step"):
- **A house dragged out:** pressed at one corner and dragged to the other, built as it goes, square to the view.
- **Its handles:** a cross that moves it, a ring that turns it, an arrow on each side that pushes that side, a point for its storeys' height and one for its roof's steepness, all built again as they are dragged.
- **Its own look:** its walls, trim, windows, wood, roof and footing coloured, pictures over its walls and roof, its roof gabled, hipped or flat, its windows spaced.

Still missing from the table above:
- cutting a wall in two, and joining two (snapped walls meet but stay two);
- a house's stilts, its roof turned or bulged by dragging, houses that snap together and join their roofs;
- paths as a brush with live arches;
- windows placed on walls;
- small things linked to windows and doors.

`-Demo walls`, `-Demo looks` and `-Demo houses` show the built tiers. The amendments are 7cl, 7cm and 7cn.

## What a real Tiny Glade tier needs

1. **A structure that rebuilds live while it is dragged.** Everything else depends on this.
   - The host's planning is fast enough for one structure (about 11 ms).
   - Not yet measured: the full round trip during a drag (plan, send meshes back, build them in Godot, show
     them).
   - **Measure it first.** If it is too slow, the preview while dragging is built in Godot from the same rules
     (a light version) and the host builds the final one on release.
2. **Drag to draw:** walls, fences and hedges as left-drag strokes, simplified into control points by speed and
   turn as Tiny Glade does; buildings dragged out to a size.
3. **Handles on drawn structures.** A right-click shows:
   - a dot per control point, the top for height, cut and join (walls);
   - each side, height, base, roof height and turn, move and rotate (buildings).
   Every drag rebuilds live; letting go is one undo step.
4. **Paths as a brush with a width.** The places they cross are worked out live:
   - a door or an arch through a wall (shaped by its width);
   - steps on slopes, stepping stones on water;
   - one opening height per building, set by hovering in the arch.
5. **Windows as things put on walls:** stacking, low ones becoming doors, linked small things swapped by a
   click.
6. **Snapping** between buildings and walls, with Ctrl to turn it off; roofs that join.

The rules 7ck already has (keep-clear, opening a piece where a path crosses, one undo per gesture) carry over;
they become the "context" step of the live rebuild.

**About Bevy:** Tiny Glade began on Bevy and then replaced its rendering with their own (Vulkan, GPU
generation). Draw3D's host is already Rust, and the editor and play run in Godot. Moving to Bevy would replace
the whole editor and game side. Bevy's interaction model is not what makes Tiny Glade feel the way it does,
so I do not recommend it. Reading Bevy's code for ideas is free; it is MIT/Apache.

**Size:** steps 1–3 for walls alone are one milestone, and the first thing to judge by hand. Paths as a brush
with live arches are the next. Buildings with handles, then windows, follow. Each is materially larger than a
usual pass, so each needs the user's approval before it starts.
