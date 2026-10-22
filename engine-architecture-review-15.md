# Engine architecture review 15

Pass 15: draw units. What the editor draws together, measured as the whole frame, and one core defect the measurement exposed.

---

## 0. Outcome

The idle frame is 2–3 ms in every world instead of 11–24 ms, with no change to the core, the protocol, the cells or the content. What changed is one number on the editor side: a draw unit was one piece of content in one cell; it is now one piece of content across a region of 4 × 4 cells. The cell stays the unit of change and of transfer.

| Things in the world | 21,000 | 105,000 | 420,000 | 105,000, all different |
|---|---|---|---|---|
| Draw units, before → after | 1,760 → 132 | 8,800 → 632 | 35,200 → 2,512 | 8,800 → 1,784 |
| Idle frame, before → after | 1.6 → 0.7 ms | 11.0 → 2.1 ms | 23.7 → 3.1 ms | 14.3 → 1.9 ms |
| A drag step, cells to drawn | 0.84 ms | 0.99 ms | 1.07 ms | 1.04 ms |

The measurement also found a core defect: **every staged edit after a build or an upgrade rebuilt every owner** (4.5 ms a step at 105,000 things, 15 ms at 420,000, until something was committed). The world kept the document as it was *before* the transaction wrote its results back, so the next transaction took those writes for edits. Fixed in the core (rule 73); one frozen test (`x05`) is adjusted, and section 6 asks you to confirm that.

All suites pass: core 6, the prototype's 57 against the core and in the prototype, 9 native, 3 protocol. The scripted run agrees with the host after load, edits, review and reconnect in all four worlds; every drag invariant held; 20 of 20 roofs picked, windows as in Pass 13.

---

## 1. What changed

### The editor (`editor/godot/world_view.gd`)

- Each cell keeps, for each piece of content it names, its own list of instance transforms, placed from the corner of the region the cell is in. A draw unit (one `MultiMeshInstance3D`) is those lists laid end to end over the region. `region` is the number of cells on a side; 4.
- When a cell arrives, only the units of its region that the cell names, or named before, are laid again. When the cells of one exchange are all in, the units are laid once (`lay`), not once per cell; the same when content arrives for cells that waited for it.
- `rebuild()` lays everything again, for changing the region size; the measuring script uses it to sweep 1, 2, 4, 8.
- A unit's instance count is set only when it changes (setting it reallocates).

### The core

- `World::settle_basis` and its use in `engine::stage`: after a transaction has written its results back into the document (where records landed, which versions were used, id tables), the built world stands for *that* document. Without it, the next transaction, staged or committed, saw every owner as changed. The rename table the world carries is recomputed when migrations were written.

### The host

- A `DONE` reply's counts (`changed`, `owners_built`, …) are the shown transaction's: the stage's while there is one. Before they were the last committed one's, which hid the defect above behind its numbers.

### The suite

- `x05_a_rename_upstream_is_followed_by_the_keys_built_from_it`: after an upgrade, the test did a no-op rebuild and expected the migrated records to read as plain matches. A no-op rebuild now builds nothing, so the labels are the upgrade's until the owner is built again; the test now builds the owner again (a parameter set to its own value) and checks the same thing. It passes in both kernels.

---

## 2. Rules added

72. **What is drawn together is the viewer's choice, and it is not the cell.** The cell is the unit of change and of transfer. A draw unit is one piece of content over a region of cells; a cell that changes lays again only its region's units that it names or named. The region is a number chosen by measurement, not a structure anything else depends on.
73. **A world stands for the document as the transaction that built it leaves it.** What a build writes back (where records landed, versions used, id tables) is not an edit. A transaction that changes nothing builds nothing, after a build or an upgrade as much as after an edit.

---

## 3. Measurements

Scripted run (`editor\run.ps1 -Measure`), camera over the whole world for frame times. "Units only" is the same units with one instance each: the cost of the units without their triangles.

### The sweep: region size against the frame and against a cell change

| Region (cells a side) | 1 | 2 | 4 | 8 |
|---|---|---|---|---|
| **21,000 things** units | 1,760 | 504 | 132 | 36 |
| idle frame / units only | 1.64 / 1.58 ms | 0.67 / 0.47 | 0.67 / 0.37 | 0.70 / 0.38 |
| drag step, editor side (of which laying) | 0.10 (0.02) ms | 0.12 (0.04) | 0.17 (0.09) | 0.36 (0.27) |
| **105,000** units | 8,800 | 2,504 | 632 | 166 |
| idle / units only | 11.0 / 11.3 | 2.7 / 2.3 | 1.9 / 0.55 | 1.9 / 0.41 |
| drag step editor (laying) | 0.19 (0.04) | 0.22 (0.07) | 0.34 (0.18) | 0.48 (0.33) |
| **420,000** units | 35,200 | 10,024 | 2,512 | 636 |
| idle / units only | 23.7 / 23.8 | 4.2 / 4.1 | 2.9 / 1.1 | 2.8 / 0.39 |
| drag step editor (laying) | 0.21 (0.04) | 0.22 (0.08) | 0.32 (0.18) | 0.79 (0.64) |
| **105,000 all different** units | 8,800 | 3,912 | 1,784 | 850 |
| idle / units only | 14.3 / 14.5 | 5.0 / 4.8 | 2.1 / 2.1 | 1.8 / 0.95 |
| drag step editor (laying) | 0.19 (0.04) | 0.22 (0.06) | 0.28 (0.12) | 0.35 (0.18) |

Reading it: the frame is the units until about 1,000 of them; below that the triangles show (420,000 things: 1.8 ms of 2.9 is triangles, 26.6 million placed, most culled) and then Godot's own floor (0.37 ms with 36 units). Going from 4 to 8 buys 0.1 ms of frame and costs 0.2–0.5 ms on every cell change. 4 is the default.

### The rest, with region 4

| | 21,000 | 105,000 | 420,000 | all different |
|---|---|---|---|---|
| Drag step, cells to drawn | 0.84 ms | 0.99 | 1.07 | 1.04 |
| Commit | 0.21 | 0.18 | 0.22 | 0.22 |
| Edit / move / turn | 0.9 / 1.1 / 0.8 | 0.9 / 1.1 / 0.7 | 1.2 / 1.2 / 0.9 | 1.7 / 1.7 / 1.2 |
| Wall height (5 houses; editor side) | 10.4 (1.3) | 8.9 (1.1) | 9.1 (1.2) | 11.0 (1.0) |
| Undo | 0.8 | 0.7 | 0.8 | 0.8 |
| First show | 0.20 s | 0.90 s | 3.5 s | 1.44 s |
| Reconnect | 40 ms | 181 | 851 | 254 |
| `upgrade_all` | 0.23 s | 1.9 s | 24.8 s | 2.5 s |
| `accept`, each (Pass 14) | 0.46 (0.69) | 0.64 (2.5) | 1.46 (16.3) | 1.16 (2.5) |
| `review` | 0.6 ms | 0.6 | 0.9 | 0.9 |
| Relaying everything (`rebuild`) | 21 ms | 103 | 427 | 119 |

First show, reconnect and upgrade are within Pass 14's range: laying once per exchange costs nothing more than the per-cell units did. Core-side times moved 15–25 % between today's runs on unchanged paths (the 420,000 upgrade: 19.9–24.8 s); the editor-side and frame numbers did not.

### The defect, before and after

Staged steps right after a load, before any commit, at 105,000 / 420,000 things: core 4.5 / 15.3 ms a step, every owner built, every thing in the delta. After the fix: 0.53 / 0.56 ms, one owner. `accept` after an upgrade, which paid the same revisit on its first transaction: 16.3 → 1.5 ms at 420,000.

---

## 4. Findings

### Finding 1. The basis defect (fixed; medium, was hidden)

Pass 14 recorded "the first transaction after a build revisits every owner once" as a known effect. It was not once: it was every transaction until one was committed, because a stage is built from the committed world and dropped. A first drag in a freshly opened large project would have cost 15 ms a step in the core alone. The frozen suite did not catch it because the prototype kernel rebuilds every owner's ladder in every transaction anyway; `k01` in the native tests worked around it with an explicit rebuild. The fix is six lines and one rule (73); the suite adjustment is section 6.

### Finding 2. A flag is a build's view, not a record's state (decision needed)

Rule 71 says the review lists what the world holds, and what the world holds for a record is the label the last build of its owner gave it. Before the fix, any edit rebuilt every owner and every `Rematched` turned into `Matched`: the review list emptied itself on the next unrelated edit. After the fix the labels stay until the owner is built again, for any reason, and then the flags go the same way. Neither is what a reviewer wants. If "landed somewhere else, not yet looked at" is to survive builds, it is a mark on the record (set when a build persists a `Rematched` or `TargetChanged` landing, cleared by `accept`), not a label. Small core change; I did not make it.

### Finding 3. Triangles begin to show at 420,000 things (observation)

With 2,500 units the frame is 1.1 ms of units and 1.8 ms of triangles. Pass 12's "triangles are free" held while the units were the frame; it no longer quite does at this size, and real content will be heavier per thing. Still nowhere near a problem, and a region's units are culled as one, so a bigger region costs more triangles in view.

### Finding 4. Draw units are now independent of the cell (observation)

The cell size was chosen in Pass 9 for transfer and change granularity, and until now it was also the draw granularity. It no longer is, so it can be retuned for transfer alone if that ever matters.

### Finding 5. Changing the grouping is a full relay (observation)

`rebuild` costs 100–430 ms at 105,000–420,000 things. Anything that changes the grouping at run time (a level-of-detail scheme, a region size that follows the camera) pays this or does it incrementally. Nothing planned needs it.

### Finding 6. Not covered

Cells going away (a cell with nothing left in it) through the region path: nothing in the run empties a cell. Content arriving for a cell whose region has other cells still waiting: exercised by the slowed load, not asserted. Negative cell coordinates: exercised by the far obstacles.

### Carried, unchanged

Plans run once per facade side; square-on boxes around turned things; three turn-aware paths untested; content never let go; kept bytes as hex; counts per reply walk the owners; non-empty inbox and `FAILED` content untested on the editor side.

---

## 5. Remaining risks, by severity

1. **A busy engine (medium-high).** Unchanged: a 25-second upgrade and a 3.5-second first show freeze the editor. Now the first thing a hand on the mouse hits in a large project.
2. **Opening large projects (medium-high).** Unchanged.
3. **Real content weight (medium).** Unchanged; Finding 3 adds that triangles are no longer free at the top size.
4. **Things as Godot nodes (medium).** Unchanged.
5. **Flags that do not survive a build (medium-low, new).** Finding 2.
6. **Turn-aware paths nothing uses (medium-low).** Unchanged.
7. **Memory where nothing repeats (medium).** Unchanged.
8. **Draw units in Godot (low, was medium).** 2–3 ms at 420,000 things, by one number.

---

## 6. Decisions for you

1. **Confirm the `x05` adjustment and rule 73.** The test checked an intended rule (migrated records are plain matches once stored under the new name) through an accident of the prototype (every transaction rebuilt everything). It checks the same rule now through an owner that is actually built again.
2. **Flags: a mark on the record, or a build's view?** Finding 2. I recommend the mark, done in the next protocol slice; the review list is otherwise not a list.
3. **Region 4 as the default.** From the sweep; say if you want it exposed as a setting (it is a variable, not a setting).
4. **Next: the busy engine.** The agreed order is done (content identity → orientation → protocol slice → draw units). Risk 1 is next by severity: the host answering, showing progress and cancelling while a long transaction runs, and the editor not blocking on an exchange. Its shape touches the protocol (a transaction that is not done when `DONE` would be) and the core (a build that can be stopped), so I would want your view on it before building.

---

## Appendix. Running it

```
cd core
cargo test                                   # store 4, bridge 1, source 1

cd core/acceptance
cargo test                                   # scenarios 28, nested 11, cross 9, deferred 9, native 9

cd prototypes/identity
cargo test                                   # the same 57 in the prototype kernel

cd editor/host
cargo test --release                         # protocol 3
cargo build --release

editor\run.ps1                               # the editor, by hand
editor\run.ps1 -Measure                      # section 3; writes editor\godot\results.json
editor\run.ps1 -Measure -Quick               # the smallest world only
```

`editor\godot\results-15.json` is the run in section 3.
