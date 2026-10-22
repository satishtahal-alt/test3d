# Milestone M2 — The land by hand (implementation brief, as built)

Status: **built 2026-10-09, at its last checkpoint, waiting on the user.** Begun on the
user's word ("proceed to M2") after the M1 hands-on report was answered (`ab95eee`).
Scope: `Draw3D_Roadmap_Amendment_2026-10-09.md` §3.1, §3.3, §4 and §5 item 2. This brief
was written with the work, not before it: what it says is what was built and checked.

Commits: `25947ef` (the layer in the host), `02ef715` (the editor's tools, the climb),
`6294d01` (what is written, in the document and in the editor), and the last of M2
(the inspector, the road's line, the demo, this brief).

## What M2 was to do, and how each part is checked

| # | From the amendment | Built as | Checked by |
|---|---|---|---|
| 1 | A persistent sculpt layer: height deltas kept by the land, document content, inside what generators read | For each patch of ground touched, an asset `land.LAND.GX.GZ`: how far each sample (a metre apart) was raised, to the millimetre, and what its surface was made. The land publishes which patches have a part; whoever stands something on the land reads the parts with the shape | protocol `p19`; tour `land` |
| 2 | Patch-local rebuild: a land change must not remake all patches | A patch is made from its own part and nothing else (the part reaches one sample past the patch on every side). One stroke inside a patch makes 1 patch again of 144; over an edge, 2 | `p19` (`remade` 1, then 2); tour (`patches_made_at_most` 2) |
| 3 | Brushes: raise, lower, smooth, flatten, with radius and strength; undoable strokes | Those four, and `erase` (back toward the land as its parameters make it). A drag is the stroke so far staged again and again and one commit: one step of undo | `p19`; tour (a drag by mouse events: staged while held, one step when let go, Ctrl+Z, Ctrl+Y, Esc lets it go) |
| 4 | Two stamps: cliff, ledge | `cliff`: ground raised behind a line, sheer at the line, easing down behind and at its ends. `ledge`: a level shelf behind a line. Each turned the way its edge looks out. What is raised sheer by hand (more than 2 m between samples a metre apart) is a face: the mesh's foot stands under its top, so it is a wall with an edge | `p19` (heights before, on and behind the line; the face); tour (put down with clicks) |
| 5 | Paint layer: surface class per sample, read by the ground's colour and by generators | `paint`: grass, rock, sand, dirt, or as it was. The ground is drawn so; the woods and the meadow do not grow on rock, sand or dirt | `p19` (fewer grow, the same again when unpainted); tour |
| 6 | Regenerating the land's own parameters keeps the layers; the user may clear them | No parameter of the land touches the layer: other hills, and what was done is as far over them as it was. `sculpt clear` takes all of it away (the panel's button), undone like any edit | `p19`; tour (`still_over_them_m`, cleared and undone) |
| 7 | Feature line editing for the route | The road's `polyline` is shown on the world while the inspector shows its node: a place is taken hold of with the left button and let go where it is to be (one edit), a place added after it, or taken away. The land, the houses and what stands before them follow | `p19` (a place moved by `param`, undone); tour `knobs` (dragged with mouse events, 0.4 m from where it was let go; added; taken away) |
| 8 | Freeze/thaw and generator parameters in the editor; "what will change" | The inspector lists every node (one that makes nothing to pick is chosen there); freeze and thaw; another seed; and `?` beside a parameter asks the host what the value typed would do, without doing it (`preview`) | `p19` (frozen woods stand while the hills change; thawed, they follow; preview counts); tour `knobs` |
| 9 | The written texts into the document; in-editor text editing with problems, undoable | The moves, poses, acts, eyes and weathers are assets `written.NAME`: writing one is an edit, undone and done again, in the world's file. Still written while the world runs and from beside the editor. A panel (F10) shows and writes them, with what their reader did not understand under the text | protocol `p14` (rewritten for the new rule), `p15` unchanged; tour `penned` |
| 10 | Proof: sculpt a cliff with ledges, regenerate the woods, the cliff stands and is climbable | The tour and the demo do exactly that: the walker jumps at the face, catches the ledge's edge 3.04 m up and climbs, then the top's at 6.16 m, by `ledge_hang` and `climb` as they were written for the yard's wall. Nothing was written for it | tour `land` (`climbed`); `editor\run.ps1 -Demo land` |

## How it is made

**Where it is kept.** `Document.prefabs`, like styles and kinds. A patch's part is text:
a line `h` and 35 heights in millimetres for each row, a line `s` and 35 letters for
each row's surfaces (`.` as it was, `g r s d`); a row nothing was done to is the letter
alone, a part nothing is done to any more is not kept. 35 = the patch's 33 corners and
one sample past it on every side, so a sample near an edge is in up to four parts and
whoever writes it writes each (`Layer::put`). That is what lets a patch be made from one
asset.

**Who reads it.** The land's plan asks for every patch's part by name (so it publishes
which there are, and plans again when one appears). A patch's content is named from its
brief, the style's ground and its own part (`Generator::leaf_assets_of`, added in M1).
Every generator that stands something on the land (`Ground::read`) reads the published
shape and each part it lists: a stroke asks each of them again where things stand, and
only what moved is put elsewhere. Their meshes are not made again.

**What a stroke costs** (measured, release build, the playground): about 12 ms for the
host to answer a step of a drag, about 30 ms until the patch's new mesh has arrived;
1 or 2 patches made each step. The 12 ms is the generators on the land planning again
(the woods and the meadow sample the whole island).

**Height between samples.** What lies between them, except across a sheer step, where
the ground stays low until 5 cm from the higher sample. The mesh moves the lower corner
there, so the mesh and `Shape::height` agree and the face is upright to within a degree
or two.

**Host commands.** `sculpt` (`tool`, `dabs`, `radius`, `strength`, `height`, `surface`,
`yaw`, `depth`; or `clear`), `ground` (heights and surfaces at places; read-only, also
from beside the editor), `preview` (a parameter's value tried and not kept). `param`
takes a list (a line of places). `text` is now a document edit.

**Editor.** `terrain.gd` (F8), `textview.gd` (F10), `line.gd` (the road's places),
`inspect.gd` (rewritten: node list, `?`, freeze/thaw, another seed, line buttons).

## Manual verification (nothing in Rust or GDScript is changed)

Start: `editor\run.ps1 -Fresh` (the playground as it is made). Persistence is Ctrl+S and
Ctrl+O; F5 and F9 are the saved game only.

1. **A brush.** Press **F8**: the panel at the bottom right. Press **raise**. Move the
   pointer over the grass behind the village: a yellow ring shows where the brush would
   fall. Hold the left button and drag: the ground rises under it as you go. Let go.
   **Ctrl+Z**: the whole stroke is gone. **Ctrl+Y**: back.
2. **Wider, harder.** `]` three times and `=` twice, drag again: wider and higher. `[`
   and `-` the other way. Hold the button still: it goes on rising where it is.
3. **The other brushes.** **lower**: a dip. **smooth** over a rough place: it evens out.
   **flatten**, begun on a level place and dragged over a hill: the hill is cut to that
   level. **erase** over anything you made: back to the land as it was.
4. **Esc.** Start a drag and press Esc before letting go: nothing of it is kept. Esc
   again: no tool, and the left button picks as it did.
5. **A cliff.** Press **cliff**. The outline is a rectangle with a short mark on the
   side its edge looks out from; `,` and `.` turn it. Set **cliff high** 6 and **cliff
   back** 14, turn it so the mark points toward you, click on open ground. A grey face
   6 m high stands there, sloping away behind.
6. **A ledge on it.** Press **ledge**, **ledge up** 3, **ledge back** 2, the same
   turning; click at the foot of the face, in its middle. A shelf is cut 3 m up.
7. **Painted.** Press **paint**, choose **rock**, drag over the cliff's top: it is grey,
   and any grass or bush on it is gone. Choose **as it was**, drag over part of it: green
   again there, and what grew is back.
8. **Climbed.** Tab. Walk to the foot of the face under the ledge, jump at it holding
   forward: the hands catch the ledge's edge. Space: up onto it. Again at the face
   behind: the top. Esc.
9. **The woods made again.** **F7**; in the list at its top choose **woods**. In `seed`
   type 9 and press **?**: the note says how many trees would go, come and stay; nothing
   has changed. Enter: the trees are elsewhere, the cliff as it was. **freeze**; choose
   **island**, type 9 in `amp`, Enter: the hills change, your cliff is as far over them
   as it was, and the frozen trees stand where they stood (some in the air or the
   ground). Choose **woods**, **thaw**: they stand on the land. Ctrl+Z three times.
10. **The road.** In the list choose **road**: orange marks on the road's four places.
    Take hold of the last with the left button, drag it 20 m to one side, let go: the
    road bends, the level ground and the houses follow. Ctrl+Z. **add a place**: a fifth
    mark; drag it; **take the place away**.
11. **What is written.** **F10**. Choose **moves**; find `move jump`; under it change
    `launch jump_height ` to `launch jump_height*2 `; **Apply**. Tab, jump: twice as
    high. Esc, **Ctrl+Z**: "Taken back: 'moves' is as it was before", and `moves.txt`
    on disk is as it was too. F10 again, add a line `	wings 2` under a move, Apply:
    the line is said under the text as not understood; the rest holds. Ctrl+Z.
12. **Kept with the world.** **Ctrl+Shift+S**, write `land-test.json`. Close the editor.
    `editor\run.ps1`, **Ctrl+O**, open it: the cliff, the ledge, the paint and the road
    as you left them. F8, **take away all that was done by hand**: the land as it was
    made; Ctrl+Z: your cliff again.
13. **Shown.** `editor\run.ps1 -Demo land` (or `-Demo` alone): the same, led.

What to say afterwards: whether a 1 m lattice is fine enough for the land you want to
shape; whether a face at 2 m per metre is the right line between a slope and a wall;
whether the brush follows the hand closely enough (each step is about 30 ms behind);
whether the stamps want other numbers or other shapes; and whether Ctrl+Z over a text
(which also rewrites its file) is what you expect.

## Not built, and why

| Not built | Why |
|---|---|
| **Bake** of the layer | There is one layer and nothing to bake it into: the land's own height is a function of its parameters. `clear` is built. A bake wants a second layer, or a land whose base is itself samples (later) |
| Stamps `slope`, `hill`, `valley` | The amendment's M2 line names two (cliff, ledge); `raise`, `lower` and `flatten` make the others by hand |
| Feature lines other than the road (rivers, coasts, boundaries) | Nothing reads one yet. The road is the first, and the handles (`line.gd`) are for any parameter that is a list of places |
| A finer lattice, overhangs, caves | A height at each sample: one height per place. A face is as near upright as 5 cm in a metre. Caves and overhangs are meshes (M3) |
| Faces along a line that is not square to the lattice | A cliff turned 30° has a stepped edge, a metre a step. It is a wall all the same |
| The texts' files and the document made one | A file is still read when it is saved and when the editor starts, and then replaces what the world has. Applying in the panel, and Ctrl+Z over a text, write the file too, so the two agree; opening another world does not write files |
| Navigation following a stroke while the world runs | Strokes are made while it stands; ways are found when it starts |

## Decisions taken while building, for the user to confirm or change

1. **F9 stays a saved game** (the user's decision, 2026-10-09): what has happened is
   loaded, the edited world is not put back. Ctrl+S and Ctrl+O are the project.
2. **A text there was none of is no step of history** (it is the text's beginning), so
   Ctrl+Z after the editor starts cannot take the moves away.
3. **Writing a text is allowed while the world runs**, though it is now a document edit
   and no other edit is: a jump made higher while it is played is still the jump.
4. **A saved game does not go by the texts**: a game saved before a move was rewritten
   loads without "the world has been edited".
5. **Ctrl+Z over a text writes its file** where there is one (the keys and the palette's
   button; not a script's `undo`).
6. **Nothing grows on rock, sand or dirt that was painted**; a rock may lie on rock.
7. **A sheer step is 2 m between samples a metre apart.** Less is a slope.
