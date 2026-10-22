# Engine architecture review 12

Pass 12: one number type, and real geometry through the whole path.

---

## 0. Outcome

Both pieces are done and measured.

- **One number type.** Values have one kind of number. A parameter can be declared whole; a transaction that would give it any other number is refused.
- **Real geometry.** A realizer makes meshes. The engine names them, the cooked cell carries the names, the editor asks for them by name over the socket, and Godot draws them. Houses now have pitched tiled roofs, chimneys, walls, framed windows and doors (`editor/godot/shot-close.png`).

The path held without a change to the architecture. Three things came out of it that need your decision; the first two are rule-level (section 5, findings 1 and 2).

All suites pass: core 5, the prototype's 57 tests against the core, 7 native tests (two new), and the prototype's own 57. I had to touch the frozen suite in three small places; they are listed in section 1 so you can confirm them.

---

## 1. What changed

### One number type

- `Val` has one number, `Num`. `i(3)` and `n(3.0)` are the same value.
- A parameter declaration has a new field, `whole`. A plan is never run with a number that is not whole in such a parameter; the transaction is refused and nothing is left behind. This holds at the top (a node's parameters) and below it (a field set on a generated child).
- The contract check reports a declaration whose own default is not whole.
- The host no longer guesses which kind of number a value from the editor is.

### Touches to the frozen prototype

1. `ParamDecl` in the prototype has the `whole` field too, and ignores it. The prototype still has two kinds of number.
2. `gens.rs` says which of its parameters are whole (`count`, `seed`, `roof_color`, `axis`, `lattice_version`, `cells`) through one list by name. A shortcut, kept to one line per declaration.
3. One assertion in the test `d01` was true only for particular hash values ("every session but seed 5 ends with work outstanding"). Removing the integer changed how values are hashed, so it became false for different seeds. It now says "some session ends with work outstanding". Nothing else in the test changed.

### Content is bytes

- What a realizer makes is still outputs by name, but an output is now bytes. A text is its bytes, and reads and hashes as before.
- The kept store writes text as text and other bytes as hex. A new native test keeps a piece of content that is not text and reads the same bytes back.
- The core never looks inside an output.

### The host and the editor

- The project the host serves is in `editor/host/src/shapes.rs`: the prototype's generators, a mesh realizer, and a facade that also makes the wall its openings are set into.
- Two new kinds of frame: the editor sends names of content it has seen in cells and does not hold; the host sends each back as the realizer made it, or says why there is nothing (not made yet, not a mesh, failed).
- The Godot side keeps content by name. Things that name the same content are instances of one mesh. A cell is drawn as one MultiMesh per piece of content in it.
- Cells arrive first and are complete without content. When content arrives, only the cells that were waiting for it are drawn again.
- A thing whose content is not a mesh (a rock, something made by hand) is still drawn as a box.

---

## 2. Rules added

62. **There is one kind of number.** A parameter may be declared whole. A plan is not run with any other number in it; the transaction is refused.
63. **What a realizer makes is bytes by name.** The core does not know what a mesh is.
64. **Content crosses to the editor by name, on request.** A cell names it; the editor asks for what it lacks; things that name the same content share it.
65. **A world is complete before its content arrives.** Things are there, can be picked by id and edited; they are not seen until their content is.

---

## 3. Measurements: the editor with real meshes

Scripted run, same calls a mouse makes. A house is now 21 things (it was 17: four walls were added). Times are command to drawn, including fetching and building any new mesh.

| Things in the world | 21,000 | 105,000 | 420,000 |
|---|---|---|---|
| Edit one window | 0.8–1.2 ms | 0.9–1.3 ms | 1.6–1.7 ms |
| Move one house | 1.0 ms | 1.0–1.2 ms | 0.9–1.2 ms |
| A village's wall height | 10–12 ms | 9–13 ms | 9–16 ms |
| Undo | 0.7 ms | 0.6–0.8 ms | 0.6–0.7 ms |
| Click to thing | 0.10 ms | 0.24 ms | 0.83 ms |
| First show (build, send, fetch, draw) | 0.18 s | 1.0–1.1 s | 4.4–4.6 s |
| Reconnect | 45 ms | 210–220 ms | 880 ms |
| Idle frame, whole world in view | 3–11 ms | 38–45 ms | 48–50 ms |
| Draw units | 3,200 | 16,000 | 64,000 |
| Triangles placed | 1.3 million | 6.7 million | 26.6 million |

- All 20 test clicks at each size hit the intended thing.
- Editor and host agree after load, after edits with undo and redo, and after a reconnect; and the editor holds every mesh the cells name.
- A wall-height change makes 8 new meshes in 0.2–0.7 ms, sends 60 KB, and they are built in Godot in 2 ms.
- At the largest size the far end of the world is beyond the camera's range, so that frame time is not the whole world.

### Content

| World of 105,000 things | All villages alike | Every village different |
|---|---|---|
| Pieces of content | 9 | 801 |
| Meshes that are actually different | 6 | 501 |
| Size of all meshes | 72 KB | 6.0 MB |
| Triangles in all meshes | 780 | 64,932 |
| Time to make them | 0.3 ms | 25 ms |
| Time to fetch and build them in Godot | under 5 ms | about 60 ms |
| First show | 1.0–1.1 s | 1.2–1.3 s |

- A mesh here averages 87 triangles and 8 KB, and takes 0.03 ms to make. These are light. A finished stylized asset will be 10 to 100 times heavier.
- The mesh format is plain: 92 bytes a triangle, not compressed, no shared vertices across flat faces.

### The core

Unchanged from Pass 11 within noise (first build 3.3 s at 340,000 things; 808 bytes a thing settled; obstacle 0.19 ms).

---

## 4. What the spike shows

- **The path is sound.** Naming content by request, keeping it out of the world store, and cooking only names into cells carried real meshes with no change to the store, the cell format or the edit path.
- **Sharing works where the rules allow it.** 105,000 things draw from 9 meshes.
- **Edits stay fast.** The cost of a new mesh in an edit is the mesh, not the world.
- **Triangles are not the frame cost at this weight; draw units are.** See finding 3.

---

## 5. Findings

### Finding 1. Where a leaf stands is part of what it is asked to look like (rule-level, needs your decision)

A leaf's content is named by its whole brief, and the brief includes `x`, `y`, `z`: its place in its parent.

- Three identical windows along one wall are three pieces of content.
- Measured: 9 names for 6 different meshes; 801 names for 501.
- It gets worse with anything placed at unique positions. A fence of 100 posts would be 100 identical meshes, each made, sent and held, and none drawn as instances.
- Sharing between houses works today only because positions are relative to the parent and houses repeat.

I tried the fix: leave place out of the request. It is one line in the core. It breaks the acceptance test `x02`, which asserts the opposite: "a house is moved … one garden is re-made". So I took it back out. The suite is frozen and this is a change to what a test expects, not a missing case.

### Finding 2. A thing has no orientation (rule-level, needs your decision)

A thing has a place and a footprint size, and that is all. The cooked member has no rotation.

- Windows on east and west walls are different meshes from those on north and south, with the turn baked in.
- A turned copy of anything cannot share its mesh.
- Nothing can stand at an angle that is not baked into its own geometry.

This changes the cooked member (88 bytes today), so it is cheaper to decide before more of the editor reads that format.

### Finding 3. Draw units are the whole frame cost (adapter, medium-high)

I drew the same draw units with one instance each and compared.

| Things | Draw units | Frame, everything | Frame, one instance per unit |
|---|---|---|---|
| 21,000 | 3,200 | 3–11 ms | 2.8 ms |
| 105,000 | 16,000 | 38–45 ms | 37–44 ms |
| 420,000 | 64,000 | 48–50 ms | 51 ms |

- 6.7 million triangles cost nothing measurable on top of the units that carry them.
- Units doubled against Pass 11, because a cell now has one per piece of content (8) and not one per kind (5).
- Frame times vary between runs by several milliseconds; the comparison within a run is what counts.

So the draw-unit work you deferred does not depend on geometry weight, and is the next thing the Godot side needs.

### Finding 4. A first show draws the cells twice (adapter, low)

Cells arrive, then content, then the cells that waited are drawn again. At 105,000 things that second pass is 175 ms of a 1.0 s first show. Asking for content before drawing would avoid it. Edits are not affected.

### Finding 5. Content is never let go (medium-low)

- The editor keeps every mesh it was ever sent.
- The host's result store keeps every piece of content ever made in a session.

Neither grows fast in these tests (61 meshes after all edits). Both will with real work.

### Finding 6. Kept content that is bytes is written as hex text (low)

Frozen or detached geometry is written at twice its size. The kept store needs a way to write bytes as bytes before real meshes are frozen in quantity.

### Finding 7. A generator has no look of its own (observation)

A facade is a generator; what is seen of it is only its leaves. So walls had to become a leaf the facade makes. This reads as a rule that was already there. No change proposed.

### What the spike did not touch

- Heavy meshes: these are about 90 triangles each.
- Textures and materials: one material, colour per vertex.
- Level of detail, and not drawing what is far.
- Content that arrives late: the host still makes content inside the transaction.
- Collision from a mesh; picking uses the box around the mesh.

---

## 6. Remaining risks, by severity

1. **The editor protocol beyond this slice (medium-high).** Dragging, late content, the review inbox, things as Godot nodes, a busy engine. Unchanged, and now first.
2. **Draw units in Godot (medium-high).** Finding 3: 38–50 ms with the world in view.
3. **Opening large projects (medium-high).** 3.5 s of core time at 420,000 things, with every mesh shared.
4. **Content identity (medium).** Findings 1 and 2. Cheap to change now, costly once content is authored against them.
5. **Real content weight (medium, was high).** The path is proven; heavy meshes, textures and materials are not.
6. **Memory where nothing repeats (medium).** Unchanged at about 2,500 bytes a thing; meshes add 6 MB per 100 different villages at today's weight.
7. **Wide scopes (medium-low).** Unchanged.

---

## 7. Decisions for you

1. **Leave place out of what a leaf is asked to look like?** I recommend yes. It makes test `x02` expect that a moved garden is not re-made, so it is a change to the frozen suite and to the prototype's behaviour.
2. **Give things an orientation?** I recommend a turn about the vertical axis now (4 bytes in the cooked member), and a full rotation only when something needs it.
3. **Confirm the three touches to the frozen suite** in section 1.
4. **What comes next.** You said the next editor protocol slice. I would do decisions 1 and 2 first if you approve them, since both change what crosses the socket, then the protocol slice, then draw units.
5. **Where project code lives.** The mesh realizer and the walled facade are in `editor/host/src/shapes.rs`. That is fine for a spike; a real project will want its own place beside `core/` and `editor/`.

---

## Appendix. Running it

```
cd core
cargo test                                   # store 3, bridge 1, source 1

cd core/acceptance
cargo test                                   # scenarios 28, nested 11, cross 9, deferred 9, native 7
cargo run --release --example scale          # core measurements

cd editor/host
cargo build --release

editor\run.ps1                               # the editor, by hand
editor\run.ps1 -Measure                      # section 3; writes editor\godot\results.json,
                                             # shot.png and shot-close.png
```
