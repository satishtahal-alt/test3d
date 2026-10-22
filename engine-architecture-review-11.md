# Engine architecture review — Pass 11

**Checkpoint: the in-the-way rule, record order, memory, and the first slice of the editor adapter**

Date: 2026-10-06. Follows Pass 10 and the six decisions made after it.

---

## 0. Outcome

All four pieces of work are done and measured. Nothing in the core's model had to change to make the editor adapter work.

- **Something in the way is now the business of whoever asked about that place.** Adding an obstacle costs 0.10–0.15 ms at every world size. It was 8 ms at 17,000 things and 580 ms at 1,360,000.
- **A later edit of the same field replaces the earlier one.**
- **Memory is 1,060 bytes a thing in the test world, down from 3,151.** Once the record of the first transaction is let go it is 813. The two agreed reductions alone gave 2,004; two further small steps gave the rest (section 3). Where nothing in a world repeats, the figure is about 2,500.
- **The editor adapter works end to end, in a separate process.** Godot shows the world, turns a click into a thing, sends edits and goes back and forward. A window edit takes 0.65–1.2 ms from command to drawn. The Godot side is script only, with no native extension.
- **The prototype's 57 tests pass against the core, unmodified.** Five native tests cover what the prototype cannot ask. The prototype itself still passes its 57.

The largest untouched unknown is now real geometry. The largest new finding is on the Godot side: one draw unit per cell and kind gives a 26 ms frame when all 340,000 things are in view.

---

## 1. What changed, and where

| Part | Change |
|---|---|
| `core/src/plan.rs` | `blocked()` is gone. `in_the_way(place, size)` replaces it. |
| `core/src/build.rs` | Two indexes by place: what stands there, and who asked there. |
| `core/src/engine.rs`, `doc.rs` | One value per field of a child; edits apply in the order they were made. |
| `core/src/base.rs`, `names.rs` | `Brief`: named values as one small shared block. |
| `core/src/eval.rs` | Plans that read the same things are held once. |
| `core/src/store.rs` | `cells_changed`: the cells that differ between two versions of the world. |
| `editor/host/` | New. The engine as a process: 336 lines of Rust. |
| `editor/godot/` | New. The Godot side: 564 lines of GDScript in four files. |
| `editor/run.ps1` | Starts the host, runs Godot against it, stops the host. |
| `core/acceptance/tests/native.rs` | Two new tests, `k04` and `k05`. |

**The prototype crate was changed in two places, both because of the approved interface change:**

- `gens.rs`: the row, the facade and the scatter tile call `in_the_way` instead of `blocked`. This file is shared by path, so it is one text for the prototype and the core.
- `kernel/plan.rs`: two small methods (`in_the_way`, `tile_area`) that answer from the old list, so the prototype still compiles and still passes.

No test file was changed.

---

## 2. Rules added in this pass

52. **What is in the way is asked by place and remembered by place.** A generator asks what overlaps a footprint. The answer it got is remembered under that footprint, as a terrain sample is. Something put down, moved or taken away elsewhere is not a change to that plan.
53. **The build knows who asked where.** It keeps, by square of ground, what stands there and which generator nodes asked about it. A change to what is in the way visits the nodes that asked about that ground, and no others.
54. **Below the top, a scope is told only about what was detached from it.** Unchanged from before; it is now the same question with a shorter list behind it.
55. **One value per field of a child.** Setting a field again replaces the earlier record. When records reach one child by different roads, they apply in the order they were made, whatever order they are stored in.
56. **Named values in the world are small shared blocks.** A child nobody edited holds the brief its plan made, not a copy of it. An edit makes a block of its own.
57. **A plan is what it read.** Scopes that read the same things hold one plan and one list of reads between them. A plan that asks who it is belongs to that scope alone.
58. **A host is brought up to date from the version it has.** The engine process remembers which version of the world the editor was sent. One step serves every case: send the cells that differ between that version and the present one. A first connection, an edit, an undo and a reconnection differ only in the versions.
59. **The editor is sent cooked cells and keeps them as they are.** They are the bytes a shipped game would load. The editor draws from them and picks from them where they lie. It holds nothing per thing.
60. **A click becomes a command by asking.** The editor sends the id it hit; the engine says which node made it and under which path. The editor never builds a path or an id.
61. **Going back is taking an earlier version.** The engine process keeps the document and the world of each step. Undo builds nothing.

---

## 3. Measurements: the core

Release build, one thread, the same scenes as Pass 10. Full output: `core/acceptance/scale-results.txt` and `memory-results.txt`.

### Edits, in milliseconds

| Things in the world | 17,000 | 85,000 | 340,000 | 1,360,000 |
|---|---|---|---|---|
| Nothing changed | 0.015 | 0.015 | 0.017 | 0.018 |
| Edit one window | 0.33 | 0.36 | 0.38 | 0.46 |
| Move one house | 0.72 | 0.50 | 0.54 | 0.63 |
| A village's wall height (550 things) | 4.5 | 5.2 | 5.0 | 5.4 |
| **Add an obstacle far from everything** | **0.10** | **0.12** | **0.12** | **0.15** |
| **Add an obstacle on a house** | 0.58 | 0.50 | 0.58 | 0.64 |
| **Detach one window** | **0.33** | **0.38** | **0.41** | **0.47** |
| First build | 103 | 505 | 2,028 | 8,384 |

- **The two rows that grew with the world in Pass 10 are flat.** They were 8 / 35 / 221 / 580 ms.
- **An obstacle far away looks at no generator node at all.** One on a house builds one node, re-runs one plan, and changes about 18 things.
- **I broke the index on purpose** (nobody is told when ground changes). Test `k04` failed, as it should.

### Memory, in bytes a thing (85,000 things)

| Step | Total | Memory of work | Structure |
|---|---|---|---|
| Pass 10 | 3,151 | 1,386 | 1,251 |
| The two agreed reductions (rule 56) | 2,004 | 722 | 768 |
| Plans and version lists held once (rule 57) | **1,060** | 85 | 461 |

The total includes 247 bytes a thing for the record of the first transaction, which is let go at the next one. Without it the last row is **813**. At 1,360,000 things the engine holds 1.43 GB; it held 4.25 GB.

**I went two small steps past the two you agreed to.** Both are in the disposable cache only, about 40 lines together, and can be taken out without touching anything else. I did it because the two agreed steps alone left the figure at 2.5 times the target. Say if you would rather they were removed.

**The test world flatters this number.** It is 5,000 houses that are all alike. Measured on less kind worlds:

| World | Bytes a thing, settled |
|---|---|
| 100 villages of 50 houses, all alike | 813 |
| 100 villages of 50 houses, every village different | 877 |
| 5,000 villages of one house, all alike | 1,302 |
| 5,000 villages of one house, every house different | 2,485 |

- A world built from repeated parts is near the target. A world where nothing repeats is at three times the target.
- Many small generator nodes cost about 4.5 KB each on top of their things. About 3 KB of that is a second copy of the document node, held until the node is next edited.

**Where the rest is,** for the next step if one is wanted: the things are held twice in the store, once by generator and once by region (260); kind and fingerprint names are text in every child (about 150); a brief is 48 bytes a field (about 300 where not shared).

---

## 4. Measurements: the editor adapter

The engine runs in its own process. Godot 4.7.2 connects over a local socket. Every figure below was produced by the same calls a hand on the mouse produces. Full output: `editor/godot/results.json`; a picture: `editor/godot/shot.png`.

Milliseconds. "Core" is the transaction inside the engine process. "Whole" is from sending the command to the changed cells being drawn in Godot.

| Things in the world | 17,000 | 85,000 | 340,000 |
|---|---|---|---|
| Edit one window: core / whole | 0.50 / 0.65 | 0.58 / 0.75 | 1.04 / 1.21 |
| Move one house: core / whole | 0.66 / 0.94 | 0.65 / 0.95 | 0.61 / 0.92 |
| A village's wall height: core / whole | 5.3 / 6.6 | 5.9 / 7.2 | 5.0 / 6.1 |
| Add a rock far away: core / whole | 0.15 / 0.39 | 0.13 / 0.30 | 0.13 / 0.28 |
| Add a rock on a house: core / whole | 0.63 / 0.91 | 0.55 / 0.76 | 0.57 / 0.78 |
| Undo: core / whole | 0.002 / 0.42 | 0.004 / 0.40 | 0.002 / 0.35 |
| Click to thing (a ray through the cells) | 0.09 | 0.22 | 0.80 |
| Ask the engine what that thing is | 0.24 | 0.25 | 0.19 |
| First show: build, send, draw | 150 | 619 | 2,823 |
| Editor reconnects: send and draw everything | 25 | 132 | 510 |
| Bytes for the whole world | 1.5 MB | 7.5 MB | 30 MB |
| Idle frame, whole world in view | 1.6 | 13.3 | 26.0 |
| Draw units | 1,760 | 8,800 | 35,200 |

- **The process boundary costs 0.15–0.3 ms an edit.** A window edit sends one cell, 3.8 KB. Transport is not the risk, as Pass 9 guessed.
- **Script is enough on the Godot side, for now.** Reading a cooked cell and filling its draw buffers costs about 1.4 µs a thing in GDScript: half a second for 340,000 things, 0.06 ms for the cell a window edit changes.
- **Every click was right.** 20 of 20 at each size: the ray met the roof it was aimed at, and the engine named its node and path.
- **The two sides agree.** After the first show, after 55 edits with 20 undos and 10 redos, and after a reconnection, the editor holds the same number of cells, bytes and things as the engine.
- **Undo sends about six cells** and builds nothing.

---

## 5. Findings

### Finding 1. Too many draw units on the Godot side (adapter, medium)

The Godot side holds one MultiMesh for each kind of thing in each cell. With five kinds that is 35,200 draw units at 340,000 things, and a 26 ms frame when the whole world is in view. Pass 9 saw the start of this (6.8 ms with three kinds).

- This is the adapter's own problem. The engine and the protocol are not involved.
- The fix is coarser draw groups, or not drawing far cells in full. Neither is built.
- It will change again with real meshes, so I would not solve it before the geometry spike.

### Finding 2. What this slice of the adapter does not do (adapter, medium-high)

It draws, picks, edits by command and goes back. It does not yet do the following, and each needs something of the protocol:

- **Dragging.** A drag is many edits a second. At 0.7–1 ms each they fit in a frame. But each is now a committed transaction with an undo step. The engine can stage a transaction and throw it away; the host does not offer that to the editor yet.
- **Content arriving later.** The host runs the engine in the mode that makes content at once. Nothing is sent when content arrives.
- **Real content.** The editor draws a box by kind. It never asks for content by name. So a wall-height change sends 20 cells that look the same as before.
- **The review inbox.** Records that need a look, and badges, are not sent.
- **Things as Godot nodes.** Colliders, scripts and gizmos need some things to exist as nodes. Nothing here makes them.
- **A busy engine.** The host does one thing at a time. During a 2-second build it cannot answer a click.

### Finding 3. The adapter is tested only through Godot (low)

The measuring run checks agreement three ways, and it passes. There is no test of the protocol that runs without Godot.

### Finding 4. Loose ends in the core (low)

- **Numbers.** You decided after Pass 8 that values have one number type. The core still has two, as the prototype did. Values now cross the editor boundary as text, where the host has to guess which is meant. Cheap to fix now.
- **Very large obstacles.** An obstacle is entered in every 16 m square it covers. One the size of the world would be entered thousands of times. Not guarded against.
- **Kept versions.** The host keeps every version of a session for undo. Each costs what differs in it, but nothing ever lets one go.
- **Pinned children do not ask what is in the way.** A house moved by hand stays where it was put, even on a rock. This was so before; it is now visible in the measurements.

### Unchanged from Pass 10

- Opening a project is a full evaluation: 8.4 s at 1,360,000 things, with box content.
- An owner with 1,000 children costs 4–10 ms an edit.
- Real geometry has not been touched.

---

## 6. Remaining risks, by severity

1. **Real geometry (high).** The largest unknown never touched, and now the one thing between this and something that looks like the target. It reaches into content size, the kept store, opening time and the Godot side.
2. **The editor protocol beyond this slice (medium-high).** Finding 2. Dragging and late content are the two that could ask something of the engine.
3. **Opening and upgrading large projects (medium-high).** Unchanged.
4. **Draw units in Godot (medium).** Finding 1.
5. **Memory where nothing repeats (medium).** About 2,500 bytes a thing.
6. **Wide scopes (medium-low).** Unchanged.

The two risks that led Pass 10, the in-the-way rule and memory in the test world, are closed.

---

## 7. Decisions for you

1. **The shape of the adapter.** I chose, and recommend keeping:
   - a separate engine process, reached over a local socket;
   - script only on the Godot side, no native extension, until measurements ask for one;
   - cooked cells as the wire format, unchanged;
   - the engine process remembering which version the editor has.
2. **The two extra memory steps** (rule 57 and shared version lists). Keep, or take out.
3. **What comes next.** Candidates:
   - **A real-geometry spike through the whole path** (recommended): one realizer that makes a mesh, content fetched by name by the editor, drawn in Godot. It is the largest unknown, and there is now an end-to-end path to push it through.
   - **The next slice of the editor protocol:** dragging with staged transactions, late content, the inbox.
   - **Draw units in Godot.** I would leave this until after geometry.
   - **A cache on disk,** for opening large projects.
4. **One number type in values.** I recommend doing it now, before more of the editor depends on the guess.
5. **Where the adapter lives.** It is in `editor/host` and `editor/godot`, beside `core/`. Say if you want it elsewhere.

---

## Appendix. Running it

```
cd core
cargo test                                   # store 3, bridge 1, source 1

cd core/acceptance
cargo test                                   # scenarios 28, nested 11, cross 9, deferred 9, native 5
cargo run --release --example scale -- huge  # section 3, to 1,360,000 things (about 1.5 GB)
cargo run --release --example scale -- memory

cd prototypes/identity
cargo test                                   # the prototype itself: 57

cd editor/host
cargo build --release

editor\run.ps1                               # the editor, by hand
editor\run.ps1 -Measure                      # section 4; writes editor\godot\results.json
```

By hand: left click picks; right click moves the picked thing's house to that place; `O` puts a rock under the pointer; `X` detaches the picked thing; `Ctrl+Z` and `Ctrl+Y` go back and forward; `W A S D` and the wheel move the view.
