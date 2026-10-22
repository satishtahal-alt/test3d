# Engine architecture review — Pass 9

Date: 2026-10-05. Follows Pass 8 and the seven decisions given after it.

This pass has three parts:

- **Part A** — the asynchronous realization probe (in the identity prototype).
- **Part B** — the world store, built in a new core crate, not in the prototype.
- **Part C** — the checkpoint: scaling, incremental transactions, cancellation, memory, and the Rust ↔ Godot boundary.

Nothing editor-side has been started.

Decisions recorded from the last message, now treated as settled:

1. A detached house loses its garden (a reader sees what its source has now).
2. Staleness does not spread to readers.
3. Whole-numbered is a constraint on a parameter, not a separate kind of value.
4. Asynchronous realization is probed before anything else. (Part A.)
5. No editor yet.
6. The world store is built in the real core. (Part B.)
7. Then a checkpoint. (Part C.)

---

## Part A — Asynchronous realization

### A.1 What was built

About 250 lines were added to the prototype kernel (`src/kernel/jobs.rs` is new; `build.rs`, `eval.rs` and `engine.rs` were extended), with 9 new tests (`tests/deferred.rs`, d01–d09) and one measurement (`examples/deferred.rs`).

The prototype now has two ways to run:

- **Synchronous**, as before: a leaf's content is made inside the transaction.
- **Deferred**: the plan still runs inside the transaction, but a leaf whose content is not there yet becomes a *job*. The transaction commits without it. Jobs run on worker threads and their results come back later.

The split follows the line the design already had. The plan decides identity, keys, place and size. Realization decides only what a leaf looks like.

### A.2 What is proven

All of these are tests, not arguments.

- **Order of arrival does not matter (d01).** The same editing session was replayed with results arriving in six different scrambled orders, including results for requests that had been withdrawn and results delivered twice. In every case the final world is byte-for-byte the synchronous one.
- **The document never depends on what has been made (d01, d08).** After every transaction the saved document is identical to the synchronous one, whether or not any content existed at that moment.
- **Before anything is made, everything is already there (d02).** Every thing has its id, path, place, size and tags. Every edit record has landed where it lands. Only content is missing. When content arrives, the diff contains content changes and nothing else.
- **Edits by identity do not wait (d03).** Moving a house and editing a window work with nothing made.
- **A withdrawn request needs no undoing (d04, d09).** Changing a parameter before the content exists just changes what is wanted. Old work that finishes anyway is kept and changes nothing. On undo it is used at once and not asked for again.
- **A result that arrives during a staged transaction is not lost (d05).** This was a real hazard; see rule 29.
- **A failed job does not take anything down (d06).** A realizer that panics leaves one thing without content, with a reason shown. Everything else is unaffected.
- **Real threads give the same world (d08, d09).** Four workers, edits going on while they work, same result.

### A.3 Rules found

Rules continue the numbering from Pass 8.

**Rule 26 — Only content waits.**
The plan runs inside the transaction. A unit may be deferred only if nothing reads its output except the cook and the derivers. It follows that a plan never reads realized content: what one generator publishes to another (rule 16) is built from plans alone. This held already; it is now a rule because breaking it would move identity into the background.

**Rule 27 — Content is named by what was asked for.**
A job's name is the hash of its realizer's version and everything it was given. It is not named by which thing asked, or when. Three things follow, and none needed code of its own:

- A late, repeated or unwanted result is still simply the answer to that request.
- Cancelling is taking a request out of the queue. There is nothing to roll back.
- Equal requests are one piece of content. In the measured scene 60,000 leaves needed 601 pieces. This is instancing, found without looking for it.

**Rule 28 — The store of results is add-only and outside transactions.**
Staging copies the document and the cache. It must not copy the results, or committing a staged transaction would discard what arrived while it was open. Results are shared by every copy.

**Rule 29 — Content is shown because it is there, not because it arrived.**
After a commit, the committed world may be waiting for something the store already has. The check is "does the world want anything the store holds", made after every commit and every delivery. An event-driven version ("rebuild when a result arrives") loses results in exactly the staged case of d05.

**Rule 30 — A result never changes the document.**
Arrivals rebuild the view. They create no undo step and do not mark the file as changed.

**Rule 31 — The document never holds a promise.**
Freezing, detaching and adopting copy content into the document. They are refused while that content is not made ("N parts are still being made"). It is enough that the part in question is made; the rest of the scene can still be waiting. For the same reason a generator is replaced only after what the old one was asked for is finished: d07 shows that otherwise the held output has holes that nothing can ever fill, because the old code is gone and results do not write to the document.

**Rule 32 — A job that fails is a state of the thing.**
The transaction that asked for it has already committed, so it cannot be refused. The thing keeps its identity and place, has no content, and says why. The failure is stored as the answer to that request, so it is not retried; a different request is a different job. A thing in this state cannot be frozen.

**Rule 33 — What is built from unmade content knows it.**
A cooked cell that contains something not yet made is marked provisional. A cook for shipping requires nothing provisional.

### A.4 Measurements

Leaves were given an artificial cost of about 2 ms each. Eight worker threads.

| Scene | Leaves | Different pieces | Transaction, deferred | Everything made after | Main thread, showing arrivals |
|---|---|---|---|---|---|
| 2 villages (1,700 things) | 1,200 | 13 | 22 ms | 92 ms | 25 ms |
| 20 villages (17,000) | 12,000 | 121 | 220 ms | 676 ms | 230 ms |
| 100 villages (85,000) | 60,000 | 601 | 1,126 ms | 3,534 ms | 1,249 ms |

For comparison, the synchronous build of the 2-village scene took 2,002 ms, because it makes each leaf where it stands and shares nothing.

Three things to read from this:

- **Sharing by request did most of the work**, not the threads. That benefit does not depend on deferral and should be kept in any mode.
- **The workers were never the bottleneck.** They finished 601 pieces in well under a second. The main thread then needed 1.2 s to show them, because in the prototype showing an arrival is a rebuild of the world.
- **Cancellation had almost nothing to cancel** for the same reason: by the time the main thread finished a transaction, the queue was empty. With real geometry this will be different; with this world store it is not.

So the probe did not find a new problem. It found the Pass 8 problem (transaction cost proportional to the world) again, from a second direction, and it says precisely what the world store must do about arrivals: see B.1.

### A.5 What this probe did not cover

- **Stopping a job that is already running.** Here a running job is left to finish. Stopping one needs places inside the realizer where it can stop. Luau has one already (the interrupt counter of rule 25). Rust realizers will need an explicit check.
- **Scatter tiles.** They stayed synchronous. A tile produces instances with ids, so by rule 26 it is closer to a plan than to content. Whether tiles can be late is open.
- **Order of work.** Jobs run in the order asked. Nearest-to-camera-first is a scheduling policy on top and changes no rule.
- **Forgetting results.** The store only grows. Forgetting an answer is safe by rule 27 (it can be asked again), except for content the document keeps.
- **Generators on worker threads.** Only leaf realizers ran on workers. Plans did not. A Luau VM cannot move between threads, so script realizers need one VM per worker.

### A.6 One thing noticed about kept content

Frozen, held and detached content is stored in the document together with its realized output. With boxes that is a few bytes. With real geometry it would put meshes inside the document.

The consequence for the design: kept content must be stored beside the document, by request name, in a store that is *not* disposable, unlike the cache. The document then holds the names. This does not change any rule; it changes where bytes live. It is listed again in Part C.

---

## Part B — The world store, in the real core

### B.1 What it had to do

Pass 8 found that every transaction cost time proportional to the whole world, because the prototype rebuilt its flat tables each time. Part A found the same cost again when content arrives. So the store had four requirements:

1. A transaction costs what it changes, not what the world holds.
2. Staging and discarding cost nothing.
3. Content arriving costs nothing in the store.
4. What the host is told is as small as the change.

### B.2 What was built

A new crate, `core/` (crate name `world-core`, a placeholder). It is fresh code. It does not depend on the prototype and knows nothing about Godot, files or threads.

- `src/pmap.rs` (about 90 lines): a map that can be copied for nothing and changed at the cost of the change.
- `src/store.rs` (about 370 lines): the world store.
- `tests/store.rs`: 3 tests. The main one runs 8 random sequences of 160 transactions each and checks every commit five ways.
- `tests/bridge.rs`: 1 test. It replays an editing session in the identity prototype and feeds every transaction's output into the store. The prototype is a test dependency only.
- `examples/scale.rs`: the measurements in B.5, with an allocator that counts live bytes.

### B.3 The shape

The world is kept twice, for two readers.

- **Pieces.** A tree with one piece per evaluation unit, holding what that unit made, placed in the unit's own frame. A piece is never changed after it is built.
- **Cells.** Flat lists of placed things, one list per 32 m square. This is what a host draws, what is streamed and what is cooked. A cell is never changed either; a changed cell is a new cell.

A transaction puts or removes top-level pieces. Committing it walks the old and new trees together, enters only where they differ, and rebuilds only the cells the differing things stand in. The result is a new world that shares everything untouched with the old one. The old world stays whole.

### B.4 Rules found

**Rule 34 — A piece of the world is what one evaluation unit made.**
A unit that did not run again hands back the same piece, and "the same piece, in the same place, under the same root" is skipped without being opened. This is rule 20 (units nest) carried into storage: the unit of skipping work and the unit of sharing memory are the same unit. It is also what removes the "one big owner" problem from Pass 8: editing one window in a 1,000-house owner copies three small pieces, not the owner.

**Rule 35 — A piece holds things in its own frame.**
World positions are worked out on the way down. Moving a house therefore shares the house's piece untouched; only its 17 entries in the cells change.

**Rule 36 — A version of the world is a value.**
It is never modified. Staging is building the next version; discarding is dropping it; an older version can be kept at the cost of what differs.

**Rule 37 — A thing names its content by request, and the store holds only the name.**
This is rule 27 applied to storage. Whether the content exists yet is the business of whoever keeps the results. So an arrival changes nothing in the store: no cell is rebuilt and no delta is produced. The 1.2 s the prototype spent showing arrivals (A.4) becomes zero here by construction. Each commit also reports which content names came into use and which fell out of use, so the job queue is kept in step without ever scanning the world.

**Rule 38 — Equal is unchanged, even when nothing is shared.**
Where pieces are not the same object, the store compares by value. A village made again from nothing, with identical results, produces an empty delta and touches no cell. So "deleting the cache never changes a result" extends to the host: it never even hears about it.

**Rule 39 — The host is told which cells changed, and takes those cells again.**
The delta lists changed things for whoever needs them, but for drawing the unit of exchange is the cell. This is the same unit as streaming and as the cook. What the editor is sent and what the runtime loads are then the same kind of thing. Part C measures this in Godot.

**Rule 40 — A thing that moves between roots has changed; it has not been removed and added.**
Detaching a house takes it out of the village's piece and makes it a root. It keeps its id, so the host is told about one change of owner, not a deletion and a creation.

**Rule 41 — Two things cannot share an id.**
A commit that would cause it is refused and leaves nothing behind.

### B.5 Measurements

Release build. A village is 50 houses of 17 things. Times for single edits are means over 200 different targets.

| Things | First build | Nothing changed | Edit one window | Move one house | One village made again | Memory |
|---|---|---|---|---|---|---|
| 17,000 | 9.8 ms | 0.1 µs | 20 µs | 49 µs | 1.0 ms | 4.7 MB |
| 85,000 | 46 ms | 0.1 µs | 34 µs | 60 µs | 1.0 ms | 22 MB |
| 340,000 | 205 ms | 0.1 µs | 53 µs | 81 µs | 1.4 ms | 89 MB |
| 1,360,000 | 995 ms | 0.1 µs | 84 µs | 96 µs | 2.8 ms | 353 MB |

For comparison, the prototype's "edit one window" was 129 ms at 17,000 things and 4.6 s at 340,000. Those figures include its evaluator; see B.7.

Other results at 340,000 things:

- Staged and thrown away: 2.4 µs.
- An edit followed by cooking the cells it touched: 58 µs. Unchanged cells keep their cooked bytes.
- What a window edit sends: 100 bytes. A whole village made again: 88 KB, encoded in about 10 µs.
- Cooking everything: 14 ms for 30 MB.
- Keeping every version for undo: about 108 KB a version, nearly all of it fixed overhead of the map and not the edit.
- One owner of 1,000 houses: a window edit is 37 µs. The Pass 8 penalty for a coarse owner is gone.
- 17,000 single-house owners: a window edit is 110 µs.

Memory is about 260 bytes a thing: about 110 in pieces and about 150 in cells and the index.

### B.6 What is proven

- **Any sequence of transactions gives the world built from scratch** from the same pieces, including identical cooked bytes. 1,280 random transactions, each checked.
- **Each delta is exact.** Applying the deltas in order to an independent copy reproduces the world. Nothing is reported that did not change.
- **Cells not named in a delta are the very same objects as before.**
- **Earlier versions stay whole** after later commits.
- **Content in use is tracked exactly** without scanning.
- **The store holds what the prototype builds.** Across a 27-transaction session (moves, deletions, tags, attachments, parameter changes, freezing, detaching, adopting, copying, a key-renaming upgrade, scatter) the store holds the same things at the same places, and finds the same additions, removals and changes that the prototype found by comparing whole worlds.

### B.7 What the store does not solve

The store is the downstream half of a transaction. The upstream half is the evaluator: deciding which units to visit and running them. In the prototype that half still looks at every owner on every transaction (44 ms at 100 owners after the Pass 8 fix). The store makes it *possible* for a transaction to cost only what it changes. It becomes true end to end only when the evaluator in the real core hands back the same piece for a unit it did not run, and finds the units to visit from what the transaction touched. That is the main item in Part C.

Also not in the store yet:

- Edits applied after generation (tags) are expected to arrive as copied pieces along one path. The kernel that does this is not written.
- The store is single-threaded. Because versions are values, readers on other threads are safe by construction, but that is not exercised.
- Memory per thing has not been worked on. It could be roughly halved; there is no reason to do so yet.

---

## Part C — Checkpoint

### C.1 Scaling

- **World store:** solved and measured to 1.36 million things. Edits cost tens of microseconds and grow very slowly with the world.
- **Document:** small throughout (0.74 MB at 340,000 things, from Pass 8).
- **Evaluator:** not yet. See B.7. This is where the remaining world-proportional cost lives.
- **First build:** about 0.7 µs a thing in the store. First builds will be dominated by generation and realization, not storage.

### C.2 Incremental transactions

- A transaction on the store is proportional to the change, with exact deltas.
- Staging and discarding are free. Previews can be staged on every mouse move.
- Deleting the cache produces no delta (rule 38).
- Still to be shown end to end with a real evaluator in front of it.

### C.3 Cancellation

- Cancelling is removing a request from a queue (rule 27). Nothing is rolled back, because an unanswered request was never part of anything.
- The store tells the queue what came into use and fell out of use at each commit (rule 37).
- A late result is harmless and is kept.
- Not covered: stopping a job that is already running, and scatter tiles (A.5).

### C.4 Memory

- About 260 bytes a thing in the store: 89 MB at 340,000 things, 353 MB at 1.36 million.
- About 108 KB for each extra version kept.
- Results are never forgotten yet. Forgetting is safe by rule 27 except for content the document keeps.
- Kept content (frozen, held, detached) must live beside the document by name, in a store that is not disposable (A.6). This is a design consequence, not yet built.

### C.5 The Rust ↔ Godot boundary

A small probe (`probes/godot-store-probe`, `probes/rust/store-bridge`) put the store behind one Godot class, in-process. Godot holds one MultiMesh per cell and kind. After each transaction it asks which cells changed and takes those cells again, whole (rule 39). This is a measurement, not the adapter.

| Things | Loading everything: core + Godot | Edit one window: core / Godot | Move a house: core / Godot | Remake a village: core / Godot |
|---|---|---|---|---|
| 17,000 | 8 ms + 16 ms | 27 µs / 0.04 ms | 58 µs / 0.10 ms | 0.9 ms / 0.28 ms |
| 85,000 | 49 ms + 48 ms | 48 µs / 0.05 ms | 93 µs / 0.11 ms | 1.3 ms / 0.34 ms |
| 340,000 | 220 ms + 198 ms | 62 µs / 0.05 ms | 107 µs / 0.13 ms | 2.0 ms / 0.32 ms |

What this shows:

- **Keeping Godot in step costs well under a millisecond per edit at every size.** The frame containing an edit was no longer than an idle frame.
- **A window edit moves about 2.3 KB to Godot** (one cell, three kinds). A separate process would add a copy of that; transport is not the risk.
- **The host needs no per-thing bookkeeping to draw.** It holds cells.

What it does not show:

- Anything but drawing. Picking a thing by clicking, gizmos, per-thing nodes for colliders or scripts, and the review inbox are not covered. These need ids on the Godot side, and that is the real design work of the adapter.
- Draw-call strategy. 23,300 MultiMeshes gave a 6.8 ms idle frame at 340,000 things. Fine for a probe; a matter for the adapter.
- A separate process. The probe was in-process.

### C.6 What is now proven

Added since Pass 8:

1. Realization can run in the background without touching identity, the document or determinism (rules 26–33, 9 tests).
2. Content named by request gives late-result safety, free cancellation and sharing of equal content in one mechanism.
3. A world store exists whose transactions cost what they change, to 1.36 million things, with exact deltas and free staging (rules 34–41).
4. The store agrees with the prototype on a full editing session.
5. Godot can be kept in step by cells for well under a millisecond per edit.

Test totals: identity prototype 57, Luau spike 6, core 4.

### C.7 Architectural changes discovered in this pass

1. **Content is named by request** everywhere: in the job queue, in the store, and (to come) in kept content beside the document. In the prototype, content hashes lived inside things; that was the cause of both rebuild-on-arrival and content in the document.
2. **The world is a value made of shared pieces, one per evaluation unit.** The prototype's flat tables are replaced, as Pass 8 said they must be.
3. **The cell is the unit of exchange** for drawing, streaming and cooking alike.
4. **Showing content is driven by state, not by arrival events.**
5. **Operations that copy content into the document wait for that content**, and a generator is replaced only after its outstanding work is done.

None of these contradicts a rule from earlier passes.

### C.8 What remains uncertain, ranked by severity

| # | Risk | Severity | Why | What would settle it |
|---|---|---|---|---|
| 1 | The evaluator in the real core | High | The kernel must be rewritten to produce pieces, and its own per-transaction cost must follow the change. Until then the end-to-end claim rests on the store alone. | Port the kernel into the core in slices, with the prototype's tests as acceptance tests, and repeat the Pass 8 scale table end to end. |
| 2 | Real geometry | Medium-high | Everything is still boxes. Fingerprints, tolerances, mesh sizes and kept-content storage are untested. | One real archetype (a wall with openings) through plan, realize, store and Godot. |
| 3 | The editor protocol beyond drawing | Medium | Picking, gizmos, per-thing nodes, inbox, separate process. | The editor-side adapter on a small scene. |
| 4 | Running-job cancellation and scatter tiles | Medium-low | Not covered by Part A. | A realizer with stop points; a decision on whether tiles may be late. |
| 5 | Reading below the top level across generators | Medium-low | Unchanged from Pass 8. | A use case that needs it. |
| 6 | Lower | Low | Script authoring workflow, mutually constraining generators, archetype and generator name collision, id table growth, split and merge, memory per thing, draw calls. | Left alone unless something points at them. |

Pass 8's top risk (transaction cost proportional to the world) is resolved for the store and remains, in smaller form, as risk 1. Pass 8's second risk (asynchronous realization) is resolved except for item 4.

### C.9 What can be committed to

Everything in Pass 8 section 8, plus:

- Realization is deferred; plans are not (rule 26).
- Content is named by request (rules 27, 37).
- Results are add-only and outside transactions; showing is state-driven (rules 28, 29).
- Arrivals never change the document; the document never holds a promise (rules 30, 31).
- The world store's shape: a value made of shared pieces per evaluation unit, with cells as the flat view (rules 34–36, 38).
- The cell as the unit of exchange with a host (rule 39).

Still not to be committed to:

- Exact binary formats, cell size, map fan-out and memory layout.
- Which thread plans run on.
- The export API between generators.
- The editor protocol beyond "changed cells".
- The crate name and layout of `core/`.

### C.10 Decisions for you

1. **Next step.** I recommend risk 1: port the kernel into the core, in slices, each slice passing the matching prototype tests. A first slice would be document, identity and an evaluator that produces pieces for one generator, with the Pass 8 scale table repeated end to end. The editor adapter comes after at least that slice, because it would otherwise be built on a stub. The alternative is to start the adapter now on the store with hand-made pieces; it would show something on screen sooner but tests risk 3 before risk 1.
2. **The cell as unit of exchange (rule 39).** It ties the editor protocol to the runtime format. I recommend confirming it.
3. **Kept content beside the document (A.6, C.4).** A second, non-disposable store next to the save file. I recommend confirming the direction; nothing needs building until real geometry.
4. **Scatter tiles stay synchronous for now.** I recommend yes.
5. **The `core/` crate.** Confirm the location and give it a name, or tell me to keep `world-core` for now.

---

## Appendix — where things are and how to run them

```
cd prototypes/identity
cargo test --release                     # 57 tests
cargo run --release --example deferred   # table in A.4

cd ../../core
cargo test --release                     # 4 tests
cargo run --release --example scale      # table in B.5 (saved in scale-results.txt)

cd ../probes
./run.ps1 -Project godot-store-probe     # table in C.5 (results_e.json)
```

- Async probe: `prototypes/identity/src/kernel/jobs.rs`, `tests/deferred.rs`, `examples/deferred.rs`.
- World store: `core/src/pmap.rs`, `core/src/store.rs`, `core/tests/store.rs`, `core/tests/bridge.rs`.
- Godot probe: `probes/rust/store-bridge/src/lib.rs`, `probes/godot-store-probe/probe_e.gd`.
