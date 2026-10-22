# Engine architecture review — Pass 10

**The kernel in the real core, end to end.** Checkpoint after porting the identity kernel into `core/`.

Date: 2026-10-06. Authoritative over earlier passes where they differ.

---

## 0. Outcome

The kernel now lives in the core, and a transaction costs what it changes.

- **Acceptance.** All 57 tests of the identity prototype pass against the core. The test files and the generators are the prototype's own files, compiled from where they are and not edited.
- **End to end.** Editing one window takes 0.4–0.8 ms in a world of 17,000 things and in a world of 1,360,000 things, and does the same work in both: 3 scopes entered, 1 leaf asked for, 1 thing changed. The prototype took 129 ms and 4.6 s for this edit at 17,000 and 340,000 things.
- **Kept content** is beside the document. Freezing a village of 850 things grows the document by 144 bytes.
- **Two findings need a decision before more is built on this.** Both are in section 5.
  1. Anything that can be in the way is shown to every top-level generator. Adding one obstacle costs 0.36 ms per generator node: 580 ms at 1,600 villages. This is a rule of the model, not a defect of the port.
  2. The core holds about 3,100 bytes per thing: 4.2 GB at 1,360,000 things. About 260 of those bytes are the things; the rest is the structure above them and the memory of work.

Nothing editor-side was started. The prototype crate was not changed.

---

## 1. What exists now

```
core/                      crate engine-core (was world-core)
  src/base keys ladder idmap contract     ported unchanged
  src/plan                 generator interface; assets and inputs are looked up, not copied
  src/doc                  the document, as a value that is free to copy
  src/kept                 kept content, beside the document          (new)
  src/jobs                 requests, finished content, the worker pool
  src/eval                 what the build remembers
  src/build                document -> world, from the world before   (rewritten)
  src/engine               transactions and commands
  src/store pmap names     the world store from Pass 9; names as numbers
  tests/                   store (3), bridge (1), source (1)
  acceptance/              crate core-acceptance
    src/kernel/            the core under the prototype's names
    tests/native.rs        3 tests the prototype could not ask
    examples/scale.rs      the measurements in section 4
```

About 4,700 lines in `core/src`. `base`, `keys`, `ladder`, `idmap`, `contract` and most of `plan` and the commands are the prototype's code. `build`, `eval`, `doc` and `kept` are new.

### How the acceptance suite works

`core/acceptance` is a crate whose library is named `identity_prototype`. Its four test targets point at the prototype's test files by path, and its `gens` module is the prototype's `gens.rs` by path. So the same source that tests the prototype tests the core.

The tests ask their questions in the prototype's shape: `world.entities`, `world.pack`, `last_diff.moved`. The core has no such maps. A shell of 430 lines (`acceptance/src/kernel`) reads the core's world out into that shape after every transaction. It computes nothing itself: every value comes from a core query.

The shell also checks two things on every transaction of every test, which the prototype never needed because it rebuilt everything every time:

1. **The world reached by changing the last world is, thing for thing, the world a fresh build of the same document gives.**
2. **The list of changes the core reports is exactly the difference between the two worlds.**

I verified that these checks bite. With remembered scopes made to ignore their signature, 6 of the 9 cross-generator tests fail, 5 of them on check 1. With owners made to ignore changes in what they read, 8 tests fail across two suites.

### The slices

The evaluator was ported as one piece, because one `expand` handles every depth. It was validated suite by suite, in the order the prototype grew:

| Slice | Prototype suite | Result |
|---|---|---|
| Flat generators, records, the ladder, modes, staging | `scenarios.rs` | 28 of 28 |
| Nested generation, at every depth | `nested.rs` | 11 of 11 |
| Generators that read generators | `cross.rs` | 9 of 9 |
| Content made in the background | `deferred.rs` | 9 of 9 |

All four passed on the first complete build of the core. I do not take that as proof of much by itself; the mutation checks above are why I trust it.

---

## 2. Rules added in this pass

Rules 1–41 stand. These are what the real core needed beyond them.

**42. A world is built from the world before it.** A world remembers the document it was built from. The build finds the nodes that differ between that document and the new one, and looks at nothing else. A transaction that changes nothing costs 13–29 µs at every size.

**43. The document is a value that is free to copy.** Nodes are shared between a document and its copies. Staging copies the document; finding what differs between two documents does not look at the nodes they share. Undo is swapping a document.

**44. Every scope is an evaluation unit.** A scope is one generator at one place in the tree, with everything below it. It is remembered whole, under one hash of everything inside its owner that can reach it: its parameters, the records routed to it, what is frozen, retired or detached below it. If the hash is what it was, the scope is not entered. Pass 8 had this for owners only; it is the same rule one level down, and it is why a window edit enters 3 scopes of 251.

**45. What a plan reads from outside its owner is checked, not hashed.** Assets and inputs are not part of a scope's hash. A remembered scope records which of them its plans read and what they were; it is reused only if those still hold. So a terrain edit away from every house reaches no village, without any village being hashed against the terrain.

**46. The build never makes content.** A leaf names its content by the request for it and moves on, in every mode. Without background workers, what is still missing is made once at the end of the transaction. So content arriving changes no thing and causes no rebuild, and equal requests share one answer in the synchronous case too: the first build of 1,360,000 things ran the box realizer 7 times.

**47. Kept content is named by the document and held beside it.** Freezing, holding, detaching and hand-editing put content into a kept store that is only ever added to. The document holds names. A frozen thing keeps the name it had while live, so freezing changes nothing in the world, to the byte. Content that did not exist when it was kept is kept as nothing (rule 31 unchanged).

**48. The cache is shared, not copied.** A staged transaction uses the cache of the engine it was staged from. What a discarded transaction leaves behind is only saved work, because everything in the cache is checked before it is trusted. A world met with a cache other than the one it was built with is built whole.

**49. A scatter tile is one thing.** Its content is the table of its instances. Instances are not things: a forest costs the store one thing per tile.

**50. Derived data is pulled.** A cell's collision summary is made when asked for, from the cell and the content its things name, and remembered by what it read. A change in looks alone does not make it run again.

**51. Names are numbers taken from the names.** Kinds and tags are stored as numbers derived from the text, so a cooked cell is the same in every session.

One consequence worth knowing: the engine writes back to the document after a build (record paths, generator versions). The world's basis is the document *before* that. So the transaction after a node is created, thawed or upgraded looks at that node once more. It costs one remembered-scope check.

---

## 3. Where the core deliberately differs from the prototype

None of these changed a test. Each is a place where I chose, and you may want to choose differently.

- **"Realize" counts things, not runs.** The prototype ran the realizer once per leaf. The core runs it once per distinct request. The tests assert counts such as "11 realized", which are counts of leaves whose content changed, so the core reports that under the same name and reports actual runs separately.
- **A cooked cell holds content names, not content hashes.** So a cell is the same before and after its content arrives. A final cook for shipping must wait for content; an editor does not.
- **`Document::save()` writes one text holding the document and the kept content it names.** The tests round-trip a project through one string, so this stayed. The split form is `Document::text()` plus `Kept::save_dir()`, and that is the one the native tests exercise.
- **Two prototype tests read the kernel's source** for generator names and depth counters. In the acceptance crate they read the shell. `core/tests/source.rs` applies the same check to the core.
- **A fix in an existing core test.** `tests/store.rs` overflowed when computing a seed in a debug build. It now wraps, which gives the values the release build always used.

---

## 4. Measurements

Release build, one thread, the prototype's generators with box content. Every figure is a whole transaction: copy the document, run the command, build, write back, commit. Small figures are the median of 21 runs on different targets. Full output: `core/acceptance/scale-results.txt`.

A village is 50 houses; a house is 17 things.

### Cost of an edit, in milliseconds

| Things in the world | 17,000 | 85,000 | 340,000 | 1,360,000 |
|---|---|---|---|---|
| Nothing changed | 0.013 | 0.012 | 0.029 | 0.020 |
| Edit one window | 0.37 | 0.39 | 0.75 | 0.61 |
| Move one house (17 things) | 0.71 | 0.51 | 0.97 | 0.76 |
| One house fewer (85 things) | 0.43 | 0.49 | 1.01 | 0.81 |
| A village's wall height (550 things) | 4.6 | 4.9 | 6.6 | 5.5 |
| Move a whole village (850 things) | 0.30 | 0.35 | 0.74 | 0.68 |
| Add a hand-placed prop | 0.08 | 0.10 | 0.20 | 0.14 |
| Freeze a village | 2.0 | 1.8 | 8.0 | 2.1 |
| **Add an obstacle** | **8.0** | **35** | **221** | **580** |
| **Detach one window** | **7.7** | **36** | **217** | **561** |

- **The work is identical at every size.** A window edit builds 1 owner, enters 3 scopes, takes 52 whole, asks for 1 leaf, changes 1 thing in 1 cell. The native test `k01` asserts the counts are equal in worlds of 3 and 48 villages.
- **The time is not perfectly flat.** It roughly doubles from 17,000 to 340,000 things while the work stays the same. I attribute that to memory being touched further apart, and the 340,000 column was the noisiest: an earlier run gave 0.48 ms for the window edit.
- **Where a window edit's 0.4 ms goes.** Mostly into checking the 50 sibling houses, at about 4 µs each. The store commit measured in Pass 9 is 20–60 µs of it.
- **The last two rows grow with the world.** See finding 1.

### Cost of whole-world operations

| Things | 17,000 | 85,000 | 340,000 | 1,360,000 |
|---|---|---|---|---|
| First build | 98 ms | 489 ms | 2.7 s | 8.7 s |
| Open with no cache | 102 ms | 517 ms | 2.8 s | 8.7 s |
| A generator is replaced (all held) | 35 ms | 173 ms | 0.9 s | 2.7 s |
| Upgrade all | 90 ms | 462 ms | 2.4 s | 7.7 s |
| Document, written | 0.10 MB | 0.25 MB | 0.80 MB | 3.0 MB |
| Document, save / open | 0.1 / 0.2 ms | 0.3 / 0.4 ms | 1.9 / 2.4 ms | 4.0 / 4.8 ms |
| Memory held | 55 MB | 266 MB | 1.06 GB | 4.25 GB |

These are about 6.4 µs per thing for a full build.

### One owner with 1,000 children

| Edit one window | Move one house | Wall height (all 1,000 houses) | Freeze it (17,000 things) |
|---|---|---|---|
| 4.0 ms | 8.5 ms | 118 ms | 41 ms |

An edit costs about 4 µs for each sibling of each scope on the way down. With 50 siblings that is invisible; with 1,000 it is 4–8 ms.

### Linked generators (terrain, 100 villages, 100 garden nodes; 90,000 things)

| Edit one window | Move one house | Move a whole village |
|---|---|---|
| 0.55 ms | 1.0 ms | 1.3 ms |

A moved house re-plans its village and that village's gardens, and re-makes one garden. No other reader is looked at.

### Kept content

- Freezing a village: the document grows by **144 bytes**. Seven entries go into the kept store, because 850 things name only a handful of distinct contents.
- The whole kept store after the session: 30 files, 7 KB, written in about 11 ms.
- Native test `k03`: a project saved as one text and one directory, opened in a fresh engine with no cache, gives the same cooked bytes, and the frozen village is not planned again. Opened without the directory, it says the stored output is missing and builds the rest.

### Where the memory is (85,000 things)

| Part | Bytes per thing | Can it be thrown away? |
|---|---|---|
| The things, by region (the store) | 260 | No |
| The structure: every generated child with its brief | 1,230 | No: commands need it |
| Memory of work: plans, what they read, scope records | 1,383 | Yes |
| Record of the last transaction (after a first build) | 247 | Yes, transient |
| Document, kept store, registry | 6 | No |

---

## 5. Findings

### Finding 1. Everything in the way is seen by everyone (model, high)

In the model since Pass 3, a top-level generator is shown every obstacle in the document, in its own frame. The prototype rebuilt every owner on every transaction, so this cost was hidden. In the core it is the one edit whose cost grows with the world.

- Adding or moving an obstacle, or detaching anything (a detached thing is an obstacle), makes every generator node check what it sees. The row generator reads what is in the way, so every village re-plans.
- Measured: 0.36 ms per generator node. 580 ms at 1,600 villages.
- Nothing below the top scope of each owner is entered, and no thing changes. The cost is the looking.

The general rule this is missing is the one already used for terrain and for scatter coverage: **read by place, so that a change somewhere else is not a change to you.** Options:

- **(a) What is in the way is read by place.** A generator asks what is in the way near a footprint. The trace records the places it asked about, exactly as `sample` does for terrain. An obstacle elsewhere re-plans nothing. This changes the generator interface: `blocked()` becomes a query with a place. The prototype's generators, and therefore the acceptance suite's, would need that one call changed.
- **(b) A generator node declares its reach.** The kernel filters by a declared region. Simpler for authors, but a reach is a second source of truth about where a generator works.
- **(c) Leave it.** Acceptable only if worlds have few top-level generator nodes.

I recommend (a). It is the existing rule applied once more, and it is cheaper to change the interface now than after more generators exist.

### Finding 2. 3,100 bytes per thing (representation, high)

The store is 260 bytes a thing, as measured in Pass 9. Everything above it is twelve times that.

- **Why.** A generated child carries its brief as a map of named values: about 750 bytes for six fields, mostly the map's own overhead. It is held twice, once in the plan that made it and once in the structure, because an edit may change the copy. Fingerprints and keys are text. Twenty thousand identical houses hold twenty thousand identical plans.
- **This is representation, not architecture.** No rule requires it. But at this size a world of a few hundred thousand things fills a gigabyte, so it bounds what can be tried.
- **What would reduce it, in order of effort:**
  1. Share the brief between the plan and the structure when no edit touches it. Small change, about 25% off.
  2. A compact map for briefs and parameters, with field names as numbers. About 50% off what remains.
  3. Drop memory of work for regions not being edited, and make it again on demand. Removes up to 1,383 bytes a thing at the cost of re-planning when returning to a region.
  4. Let equal scopes share one result, with ids derived on demand. The largest saving and the largest change: the store would no longer hold one record per thing. I would not start here.

### Finding 3. Opening a project is a full evaluation (medium-high)

The cache lives in memory only. Opening a 340,000-thing project takes 2.8 s and a 1,360,000-thing project 8.7 s, with box content. Replacing a generator holds everything made with it (0.9 s at 340,000) and upgrading re-plans it (2.4 s). Real realizers will add to the first of these.

Nothing in the design prevents a cache on disk, since everything in it is checked before use. It has not been built. Building by region on demand is the other route, and the store already hands out cells.

### Finding 4. Wide scopes and long record lists (medium-low)

- A scope with 1,000 children costs 4–8 ms per edit below it, from hashing each child's parameters to learn that it did not change. A cheaper test exists (same plan, same brief, nothing addressed below) and is not implemented.
- A node is copied when it is edited, with all its records. A village with thousands of edits pays for that on each new one. Not measured.

### Finding 5. A quirk inherited from the prototype (low, but it is a correctness question)

Records are stored sorted by path and then by id, and ids are text: `r10` sorts before `r9`. Two `Set` records on the same field of the same child are applied in stored order. So which one wins can change between the transaction that adds the second and the next rebuild. No test exercises it and the core behaves exactly as the prototype does. It should be decided rather than left: either the later record replaces the earlier (as a second pin already does), or order is by creation.

### Not covered by this pass

- Real geometry. Everything is still boxes.
- Stopping a job that is already running. Scatter tiles stay synchronous, as decided.
- Reading below the top level across generators. Unchanged from Pass 8.
- The kept store is never trimmed in memory; saving writes only what the document still names. Kept trees are stored whole, so two nearly identical frozen villages share their content but not their structure.
- Requests made by a preview that is thrown away stay in a small table until answered or the cache is dropped.
- Hashing goes through JSON text, as in the prototype. It is a large part of the 6.4 µs per thing.
- The Godot store probe was only re-checked for compilation against the renamed crate. Its numbers are Pass 9's.

---

## 6. Remaining risks, by severity

1. **The in-the-way rule (high).** Finding 1. A decision, then a small change.
2. **Memory per thing (high).** Finding 2. Bounds world size until addressed.
3. **Real geometry (medium-high).** Unchanged, and now the largest unknown that has never been touched.
4. **Opening and upgrading large projects (medium-high).** Finding 3.
5. **Editor protocol beyond drawing (medium).** Unchanged. The core now offers what an adapter needs for drawing: changed cells, content by name, what is still being made.
6. **Wide scopes, long record lists (medium-low).** Finding 4.
7. **Running-job cancellation, scatter tiles (medium-low).** Unchanged.
8. **Deep cross-generator reads (medium-low).** Unchanged.

The risk that led Pass 9, the evaluator in the real core, is closed for edits. It is open for whole-world operations (findings 2 and 3).

---

## 7. Decisions for you

1. **The in-the-way rule.** I recommend option (a): read by place. It changes one call in the generator interface and in the prototype's generators.
2. **A memory budget.** I recommend naming one (for example 800 bytes a thing all-in at a million things) and doing steps 1 and 2 of finding 2 against it before any large-content test. Without a number there is no way to say when this is done.
3. **What comes next.** Three candidates:
   - **The editor adapter**, separate process, drawing only at first. The core is now proven end to end for edits, which was the condition you set.
   - **A real-geometry spike** in the core: one realizer that makes meshes, to see what content does to memory, to the kept store and to opening time.
   - **Findings 2 and 3 first.**

   I recommend the in-the-way change, then the editor adapter, with the memory work before the geometry spike. The adapter is where the remaining protocol questions are, and none of them depend on memory per thing.
4. **Record order (finding 5).** I recommend that a later `Set` on the same field replaces the earlier one.
5. **The acceptance arrangement.** The prototype's tests are now the core's tests, by path. I recommend freezing the prototype crate as it is: its tests and generators are the suite, and its kernel is the reference the suite was written against.
6. **Three choices I made that you may want to reverse** (section 3): "realize" counts things, a cooked cell holds content names, and `save()` stays a one-text bundle next to the split form.

---

## Appendix. Running it

```
cd core
cargo test                                   # store 3, bridge 1, source 1

cd core/acceptance
cargo test                                   # scenarios 28, nested 11, cross 9, deferred 9, native 3
cargo run --release --example scale          # section 4, to 340,000 things
cargo run --release --example scale -- huge  # adds 1,360,000 (needs about 5 GB)
cargo run --release --example scale -- memory

cd prototypes/identity
cargo test                                   # the prototype itself, unchanged: 57
```
