# Engine architecture review 17

Pass 17: two slices in the order you set. Flags become marks on the record. Then a transaction in flight is shown as it goes, so a large project fills the screen while it is built instead of after.

Decisions recorded from your answer to Pass 16: rules 74–78 approved as they stand (no queue of committed edits until a workflow shows refusal is not enough); the 20 ms budget is the current responsiveness budget, and the invariant is that the editor never blocks on engine work; stopping stays at the owner, with the ~30 ms granularity at 420,000 things a known limitation; flags first, then progressive opening, before another broad pass on real content weight.

---

## 0. Outcome

**Flags.** A record that landed somewhere a person did not put it now says so itself, in the document, until it is accepted or taken back. In all four worlds the flagged owners were built again ten times and the review list was the same list; `accept` clears a flag in 0.4–0.7 ms. The review list is a list.

**Opening.** A large project now fills the screen while it is built. The editor is sent what the build has put down so far about ten times a second, draws it, and is brought to the finished world when the build lands.

| Things in the world | 21,000 | 105,000 | 420,000 | 105,000, all different |
|---|---|---|---|---|
| First things on screen, after the command | 42 ms | 64 ms | 134 ms | 489 ms |
| Half of the world on screen | (one look) | 0.44 s | 1.9 s | 0.80 s |
| Times the editor was shown the build | 1 | 6 | 35 | 6 |
| Landed: the world is there to be edited (Pass 16) | 0.18 s (0.17) | 0.80 s (1.07) | 4.4 s (3.6) | 1.16 s (1.30) |
| Everything drawn with its content (Pass 16) | 0.39 s (0.50) | 0.81 s (1.49) | 4.4 s (4.9) | 5.9 s (6.7) |
| Cells sent again when it landed, of all | 320 of 400 | 336 of 2,000 | 192 of 8,000 | 320 of 2,000 |
| Frame while the host loads | 5.6 ms | 0.6 ms | 2.6 ms | 3.8 ms |
| Upgrade: times shown / cells sent at the end | 2 / 80 | 17 / 112 | 206 / 16 | 21 / 64 |
| Stopped upgrade: world unchanged | yes | yes | yes | yes |
| Results landing on a state they had left | 0 | 0 | 0 | 0 |

Of the time to the first things, everything beyond about 40 ms (the 134 ms, the 489 ms) is the editor emptying the world it showed before, as in Pass 16.

The price is in the fourth row: at 420,000 things the build lands 0.8 s later than it did, because it does the store's work twice and shares the machine with an editor that is now receiving and drawing 38 MB of cells while it runs. With the editor capped at 60 frames a second, as it is on a screen, the same load lands 0.16 s later (3.09 s against 2.93 s). Against that, the first things are there in a tenth of a second instead of after 3 s or more, and the world is complete with its content sooner than before (half a second uncapped, a second capped), because content is made while the build goes on and not after it.

What is shown is not a world, and that is the whole of the design. Nothing was added to the transaction model: no world "so far", no split store transaction, no second basis. A build that is asked says what it has put down; the host shows it; and what counts is still only the world a transaction lands or the one it left. Stop, supersede, busy and the late check are untouched.

All suites pass: core 6, the prototype's 57 against the core and in the prototype, 11 native (2 new), 6 protocol (1 new, 1 extended). Every agreement check and every drag invariant held in all four worlds. No frozen test changed, and the prototype's kernel was not touched.

---

## 1. What changed

### Flags (core about 35 lines, host 10, editor 1)

- `doc::Record` has a `mark`: why the record waits to be looked at (`TargetChanged` or `Rematched`) and the path it had when it was marked. It is not written to a document that has none, so existing documents and every saved text in the suite are byte for byte what they were.
- `engine::persist` sets it when a build lands a record with one of those two labels, once: a record that is already marked keeps the mark it has. A record split into branches carries its mark to each.
- `cmd::accept` clears it (and, as before, writes the record against what it lands on now). `cmd::remove_record` takes the record and its mark with it. Nothing else touches it, and no build reads it: it is not in what a scope is remembered by.
- `Document::marked()` lists them. The host's `review` and the `flagged` count in every answer come from it, not from the labels of whichever build last ran. A flag says where the record is and where it was.

### Showing a transaction in flight (core about 45 lines, host 70, editor 25)

- `build::Watch` gained `ask()` and `so_far()`. Asked, a build answers once, when it has finished the owner it is on, with the store it started from and the owners built so far put into it. A build nobody asks keeps nothing.
- The host asks a committed transaction that has outlasted its exchange every 100 ms (`SHOW`), takes the answer as soon as it is given, sends the cells that differ from what the editor has, and a `SO_FAR` frame (kind 24) with the serial, progress and counts. While it has such an answer, that is what the editor is shown (`Session::sight`); every question is still answered from the committed world.
- When the transaction ends, however it ends, the answer is dropped and the editor is brought to the world that is, by the step that already existed. One addition to that step: a cell that is byte for byte what the editor has is not sent again. Two stores that were not made one from the other share no cells and may still hold the same ones, which is the case between what a build showed and what it handed back.
- Content named by what is shown is given to the workers while the build goes on, and sent when asked for, from the results the cache holds.
- The editor lays the cells it has collected when `SO_FAR` comes, as it does at `FINISHED`, and takes progress from it.
- A staged transaction in flight is not shown: the next one supersedes it.

### Tests

- `k10` (native): a `TargetChanged` mark survives its owner being built again three ways, the target coming back to where the record was written (the last build says `Matched`; the mark stays), and the owner being held under new code; a `Rematched` mark survives being stored under its new name and a rebuild with an empty cache; both are saved and opened; `accept` clears one and changes nothing shown, `remove_record` the other; no build brings either back.
- `k11` (native): a build of 48 villages is asked what it has put down, again and again, from another thread. Every answer is a subset of what the world will hold, they only grow, and the world is the one a build nobody looked at gives. Told to stop when a third is down, it hands back the world it started from, and the next build has nothing of it.
- `p03` (protocol) extended: flagged owners are built again by edits of their own and beside them, the editor reconnects, and the list is the same list; each flag says where it was and where it is.
- `p06` (protocol, new): a slow scene is shown part done (a strict subset of the things it will have); the host knows nothing of a shown thing when asked, reports no things in the world, and refuses `undo` as busy; stopped, the screen is empty again; run through, what is shown only grows, the cells at the end are the cells the scene always has, and only some of them are sent at landing; an upgrade shown half done and stopped puts back every cell it had replaced.
- `p05` and `p06` take turns (a lock), because the knobs that slow a build are the process's. `p03`'s `accept`, `dismiss` and `forget` now wait for their landing like the rest of that test, for the same reason Pass 16 gave for `p02`/`p03`.

No frozen test changed.

---

## 2. Rules added

79. **A flag is the record's.** A record that a build lands somewhere to be looked at is marked, in the document, with why and where it was. No build clears the mark and no build reads it; it goes when a person accepts the record where it is or takes the record back. What the last build made of a record (its label) is a different thing and is not what a review lists.
80. **What is shown of a transaction in flight is things to look at, not a world.** It is what the build has put down so far, on top of what it started from. Nothing is asked of it, nothing is done to it, and nothing is built from it. When the transaction ends, what is shown is the world it landed, or, stopped, failed or late, the world that was there. Rule 73 is untouched: there is no world in between.
81. **The editor is brought to a version by the cells that differ in what they hold.** Sharing is how the difference is found cheaply; a cell that is the same bytes is the same cell to the editor, whichever version made it.

---

## 3. Measurements

Scripted run (`editor\run.ps1 -Measure`), the editor drawing uncapped. `editor\godot\results-17.json`; the flags slice alone, before any of the second slice existed, is `results-17-flags.json`.

### Flags

| | 21,000 | 105,000 | 420,000 | all different |
|---|---|---|---|---|
| Flagged after the upgrade / in the inbox | 38 / 2 | 40 / 0 | 39 / 1 | 39 / 1 |
| Owners with a flag built again (two edits each of five) | 10 | 10 | 10 | 10 |
| Flagged after that, and the same list | 38, yes | 40, yes | 39, yes | 39, yes |
| `review` | 0.78 ms | 0.82 | 0.84 | 0.76 |
| `accept`, each: core / whole exchange | 0.26 / 0.41 ms | 0.37 / 0.53 | 0.50 / 0.71 | 0.46 / 0.67 |
| Flagged after accepting all | 0 | 0 | 0 | 0 |

The script makes 40 edits to windows before the upgrade; every one is either flagged or in the inbox afterwards, 40 in each world. In the flags-only run it was 40 / 0 everywhere. The split differs now because the script picks its windows by the order the editor received its cells in, and that order is now the build's; two of its picks then fall where a fingerprint cannot tell two candidates apart. It is the script's choice of windows that moved, not what the ladder does with a given one.

### Showing a load

The table in section 0. Where the build's time went, from the host (new in every answer as `phases`):

| | 21,000 | 105,000 | 420,000 | all different |
|---|---|---|---|---|
| Walk (owners planned and expanded) | 122 ms | 678 | 3,808 | 661 |
| of which answering "so far" | 2.5 ms | 61 | 451 | 69 |
| Store (the build's own transaction) | 11 ms | 64 | 449 | 64 |
| Whole transaction, host side (Pass 16) | 147 ms (157) | 756 (1,042) | 4,307 (3,548) | 741 (853) |
| Editor: laying what it was shown, in all | 0.02 ms | 24 | 159 | 6 |
| Cell bytes sent while building / at landing | 0.39 / 1.55 MB | 8.2 / 1.45 | 37.7 / 0.97 | 8.3 / 1.35 |

Answering costs what the store phase costs: every owner is put into a store twice, once to be shown and once in the transaction that counts. At 420,000 things that is 0.45 s of a 4.3 s load. The rest of the 0.76 s by which the load lands later is the walk itself running slower (3.36 s without the answering, against about 3.1 s in Pass 16) beside an editor that takes in 38 MB of cells and lays them 35 times while it runs. In the three smaller worlds the load lands when it did or sooner; run-to-run variation between passes is larger than the cost there (Pass 16's own capped and uncapped runs differ by a quarter).

### Showing an upgrade

| | 21,000 | 105,000 | 420,000 | all different |
|---|---|---|---|---|
| First changed cells on screen | 47 ms | 66 | 84 | 74 |
| Times shown | 2 | 17 | 206 | 21 |
| Run through (Pass 16) | 0.26 s (0.31) | 2.2 (2.5) | 24.4 (26.8) | 2.6 (3.4) |
| of which answering "so far" | 22 ms | 132 | 903 | 164 |
| Cells sent at the end, of all | 80 of 400 | 112 of 2,000 | 16 of 8,000 | 64 of 2,000 |
| Frame while it runs | 0.9 ms | 0.5 | 3.2 | 2.0 |
| Stopped (at 35–60 ms): world unchanged, host agrees | yes | yes | yes | yes |

An upgrade of 400 villages was 25 s of an unchanged screen with a counter. It is now villages changing over as they are done, and a stop puts every one of them back (`p06` checks the cells; the scripted run stops before the first look, as before).

### Everything else

| | 21,000 | 105,000 | 420,000 | all different |
|---|---|---|---|---|
| Drag step, cells to drawn (Pass 16) | 0.99 ms (1.18) | 1.02 (1.45) | 1.10 (1.44) | 1.05 (1.93) |
| of which core | 0.64 | 0.67 | 0.70 | 0.69 |
| Commit | 0.16 | 0.15 | 0.19 | 0.19 |
| Edit / move / turn | 1.0 / 1.2 / 0.9 | 1.1 / 1.3 / 1.0 | 1.5 / 1.4 / 1.0 | 1.3 / 1.3 / 0.9 |
| Wall height, 5 houses | 9.9 | 9.2 | 9.5 | 9.4 |
| Undo | 0.76 | 0.78 | 0.80 | 0.74 |
| Reconnect | 38 ms | 178 | 726 | 237 |
| Idle frame | 0.73 | 2.04 | 2.63 | 2.33 |

Nothing here was touched, and nothing is slower. These are nearer Pass 16's capped figures than its uncapped ones; I take that as the machine, not as an improvement.

### The same with the editor capped at 60 frames a second

`editor\run.ps1 -Measure -Capped`, `results-17-capped.json`. This is the editor as it is on a screen.

| | 21,000 | 105,000 | 420,000 | all different |
|---|---|---|---|---|
| First things on screen | 45 ms | 66 | 127 | 496 |
| Half of the world on screen | (one look) | 0.44 s | 1.47 s | 0.81 s |
| Times shown | 1 | 6 | 25 | 6 |
| Landed (Pass 16, capped) | 0.19 s (0.17) | 0.74 (0.81) | 3.09 (2.93) | 1.12 (1.35) |
| Everything drawn with its content (Pass 16, capped) | 0.40 s (0.52) | 0.75 (1.23) | 3.10 (4.15) | 5.9 (6.8) |
| Load, host side: walk / of which answering / store | 127 / 3 / 11 ms | 630 / 61 / 60 | 2,680 / 339 / 287 | 626 / 65 / 60 |
| Upgrade run through (Pass 16, capped) | 0.50 s (0.50) | 1.95 (1.87) | 21.9 (25.3) | 1.98 (2.38) |
| Stopped upgrade: world unchanged | yes | yes | yes | yes |
| Flagged / inbox, and kept through rebuilds | 38 / 2, yes | 40 / 0, yes | 40 / 0, yes | 38 / 2, yes |

Capped, the largest load lands 0.16 s later than it did (3.09 s against 2.93 s) and is complete with its content a second sooner (3.10 s against 4.15 s). The 0.76 s of the uncapped run was mostly the uncapped editor.

---

## 4. Findings

### Finding 1. A flag needed nothing but a place to be written (observation)

The label a build gives a record and the fact that a person has not looked at it were one thing, and are now two. The mark is thirty-five lines because everything it needed existed: `persist` already rewrote records where they landed, `accept` already existed, undo already swaps documents (a flag that is accepted and undone is a flag again, which `p03` checks). No build reads the mark, so it is in nothing a scope is remembered by.

One consequence worth saying aloud: a `TargetChanged` record whose target comes back to where it was is `Matched` again to the build, and still flagged. That is the rule you set (visible until a person resolves it), and `k10` holds it.

### Finding 2. What is shown did not have to be a world (confirmed)

Pass 16 expected this slice to need a store transaction per owner, a world that is "so far", and a word added to rule 73. It needed none of them. The store was already a value that costs nothing to copy and something to change in proportion to the change; a build can put what it has into one and hand it out without that store being anyone's basis. The transaction model has the same parts it had: at most one in flight, landing only on the state it left, stopped or superseded leaving saved work only.

What had to be decided instead is what the thing on the screen *is* while a build runs, and the answer is rule 80: things to look at. The host answers every question from the committed world, so a thing that is only shown cannot be described, picked into a command, or edited; `p06` asks and is told "not found", and an edit is refused as busy as before.

### Finding 3. Showing costs the store's work twice (measured, cost)

About 10 % of a load and 4 % of an upgrade at the top size (section 3). It is paid only by a committed transaction that outlasts its exchange and is asked; a drag step, an edit, anything within the budget, pays nothing. It could be taken back by making the build's own store transaction the incremental one and merging what each step changed into one account of what the build changed. That merge has cases (an owner built twice in one build, a thing added in one step and moved in the next) and it would sit in the middle of the build. I did not do it, and I recommend not doing it at this price.

### Finding 4. The build lands later because the editor is busy beside it (measured, cost)

Uncapped, at 420,000 things the walk runs about 0.3 s slower than in Pass 16 while the editor takes in and lays 38 MB of cells. Capped at 60 frames a second that part is gone: the load lands 0.16 s later than Pass 16's capped run, less than the 0.34 s the answering costs there. This is Pass 16's Finding 4 again in another place: the editor and the host share cores. Showing less often would give some of the rest back; 100 ms is a first number, not a tuned one.

### Finding 5. Content is ready sooner, though the build lands later (measured)

The workers are given what the shown cells name while the build goes on. With content slowed to 300 ms a piece, as the script does for loads, the world was complete with its content at 4.4 s instead of 4.9 s at 420,000 things, at 0.81 s instead of 1.49 s at 105,000, and at 5.9 s instead of 6.7 s where nothing repeats. A stopped build leaves that content made; it is the answer to a request, and rule 77 already says so.

### Finding 6. The world appears in build order, not near the viewer (observation, open)

Owners are built in the order of their rank (a node after everything it reads) and, within a rank, of their ids, which are hashes. So a large project appears scattered evenly across the map. For a person opening a project where they last stood, the wanted order is near first. The order within a rank is free: nothing depends on it. Giving the build a place to prefer is a small change to the queue's key. Not done: it is a decision about what "opening" is (section 7).

### Finding 7. Two things are not shown as they go (observation)

Nodes that are themselves things (hand-placed rocks, detached content) are put into the store after all owners are built, so they appear at landing; what a record attaches is shown with its owner. A staged transaction in flight is not shown, by rule 75: the next one supersedes it. Neither is exercised by the scripted run, whose loads have only generated content; this is from the code.

### Finding 8. The measuring script's choices follow the order cells arrive in (observation)

Section 3, flags. The script is unchanged; what it samples moved. Worth fixing in the script (sample by position, not by arrival) before the numbers of two passes are compared window for window. Not done.

### Finding 9. `p03` assumed an edit lands within its exchange (fixed, test only)

With `p06` slowing planning beside it, `p03`'s `forget` was answered before it landed and the test read the count too early. Rule 74 says a test must hold either way; `accept`, `dismiss` and `forget` there now wait for their landing, and the two tests that turn the slowing knobs take turns.

### Carried, unchanged

Stop granularity at the owner (your decision: a known limitation, about 30 ms at 420,000 things); `check_upgrade` and a generator swap's settling on the host's thread; a thread per transaction (0.25 ms); plans run once per facade side; square-on boxes around turned things; three turn-aware paths untested; content never let go; kept bytes as hex; non-empty inbox and `FAILED` content untested on the editor side. New, small: a mark's `from` path is not rewritten when its record moves between owners (detach, adopt), and a copied node's records keep their marks.

---

## 5. Remaining risks, by severity

1. **Real content weight (medium).** Unchanged, and now first by severity. Heavier owners make the stop slower (Pass 16, Finding 3) and each look at a build in flight staler by one owner.
2. **Things as Godot nodes (medium).** Unchanged.
3. **Memory where nothing repeats (medium).** Unchanged.
4. **Opening large projects (medium-low, was medium-high).** The screen fills from the first tenth of a second. What remains: the order is not the viewer's (Finding 6); the world cannot be edited until it lands, 4.4 s at 420,000 things; and every open is a build, because a project does not keep its cooked cells.
5. **Turn-aware paths nothing uses (medium-low).** Unchanged.
6. **A busy engine (low).** Unchanged from Pass 16.
7. **Draw units in Godot (low).** Unchanged.
8. **Flags that do not survive a build.** Closed.

---

## 6. What becomes necessary for the eventual engine

- **Opening without building.** The fastest open is not a faster build but none: a project that keeps its cooked cells beside it (as it keeps kept content) could show them at once and build beside them, replacing what differs. The step that does the replacing exists (rule 81). What does not exist is the rule for when kept cells may be trusted to be edited before the build has confirmed them. I would not start there.
- **An order of building that follows the viewer** (Finding 6), when projects are large enough for it to matter in practice.
- **Editing before landing** would need the world "so far" this pass avoided. Nothing yet says a person needs to edit a project in the seconds while it opens.
- **For what comes after world building** (characters, animation, physics, gameplay): the two rules of this pass are not about villages. A mark on a decision that a rebuild moved (rule 79) is what any authored override on generated or imported content will need, an animation retargeted onto a changed skeleton as much as a tint on a window. And "shown is not the world" (rule 80) is the line between a preview and state that the same systems will need when a simulation or a bake runs beside the editor.

---

## 7. Decisions for you

1. **Confirm rules 79–81.** In particular rule 80: what is shown of a transaction in flight can be looked at and nothing else. The alternative, a world that can be picked and edited while it is still being built, I recommend against; it is the split transaction Pass 16 described, and nothing has asked for it.
2. **The mark stays when the target comes back** (Finding 1). I read your decision that way and built it so. Say if a mark should clear itself when a later build finds the record plainly matched where it was written.
3. **Only the core has marks and "so far"; the prototype's kernel does not.** No frozen test changed and none of the 57 can see either. I treated both as new capability under the Pass 10 rule (new cases go in `native.rs`), not as a semantic difference between the kernels. Confirm, or say that the prototype should carry the mark too.
4. **Pay the store's work twice (10 % of a large load), or make the build's store transaction incremental** (Finding 3). I recommend paying.
5. **Build order.** Leave it (recommended for now), or let the editor name a place and have owners nearest it built first (Finding 6).
6. **Next.** By your order, real content weight is next: a broad pass on what heavier, more varied content does to build time per owner, stop latency, memory, content transfer and the frame. Before I start it I would want to agree what "real" is to mean for the pass (how heavy a house, how many distinct pieces), since the result depends on that choice more than on anything in the engine.

---

## Appendix. Running it

```
cd core
cargo test                                   # store 4, bridge 1, source 1

cd core/acceptance
cargo test                                   # scenarios 28, nested 11, cross 9, deferred 9, native 11

cd prototypes/identity
cargo test                                   # the same 57 in the prototype kernel

cd editor/host
cargo test --release -- --nocapture          # protocol 6
cargo build --release

editor\run.ps1                               # the editor, by hand; I lists what needs a look, Esc stops the host
editor\run.ps1 -Measure                      # section 3; writes editor\godot\results.json
editor\run.ps1 -Measure -Quick               # the smallest world only
editor\run.ps1 -Measure -Capped              # the same with the editor drawing 60 frames a second
```

`editor\godot\results-17.json` is the run in section 3; `results-17-capped.json` the capped one; `results-17-flags.json` the flags slice alone.
