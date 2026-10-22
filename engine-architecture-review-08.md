# Engine architecture review — Pass 8 (checkpoint)

Date: 2026-10-05. This pass ran the three probes agreed after Pass 7 and stops here, before any editor-side work, for review.

- Probe 1: generators that read another generator's output.
- Probe 2: timing of staging and document cost at larger scale.
- Probe 3: a Luau generator behind the same boundary as the Rust ones.

Code: `prototypes/identity` (kernel about 2,620 lines, 48 tests passing) and a new `prototypes/luau-boundary` (wrapper 288 lines, one 67-line script, 6 tests passing). Nothing here is the engine.

---

## 0. Short answer

1. **Cross-generator reads did not change the shape of the kernel.** They fit as one more kind of recorded read. Evaluation order comes from the document alone. An edit spreads one reader at a time and stops where nothing it read has changed. Seven new general rules came out of it (section 3); the most important one is about identity: a key built from another generator's id is followed through the id migration table.
2. **Scale is the one real problem found.** The design avoids the right work (editing one window among 340,000 things runs 2 units), but in this prototype every transaction still costs time proportional to the whole world: about 0.12 s at 17,000 things, 0.6–0.9 s at 85,000, 3.7–4.8 s at 340,000. I measured where the time goes and removed the largest part with one general rule (an owner is itself a skippable unit). The rest needs a world store built from shared per-owner pieces. That is designed, not demonstrated.
3. **Luau works behind the boundary.** The row generator ported to Luau produces the same keys, ids, positions and, with one declaration, a byte-identical cooked world, through an unchanged kernel. Key opacity survives as a run-time type check. The cost was 1.4–1.6× the Rust plan. Four boundary rules came out of it.
4. **The foundation is ready to commit to**, with two things explicitly left open: the world store and cook pipeline (because of point 2), and anything asynchronous (never tested). Section 7 lists both sides.

---

## 1. What was built and what happened

### 1.1 Probe 1: generators reading generators

Scene: `terrain --height--> village --children--> gardens`.

- **Terrain** has no children. It publishes a height field, one entry per lattice cell.
- **Village** is the row generator from before. It now reads the ground height under each house it places.
- **Gardens** reads the village's list of children and makes one garden behind each house, keyed by that house's id.

Nine new tests (`tests/cross.rs`). What they show:

| Question | Result |
|---|---|
| What decides evaluation order? | The document. A node is built after the nodes its inputs name. Nodes were created readers-first on purpose; the order came out terrain, village, gardens. |
| Cycles? | Refused when the connection is made, and refused again by the build if a document arrives with one. A refused transaction leaves the document untouched. |
| Terrain edited where no house stands | 1 unit ran (the terrain's plan). The village is not re-planned. |
| Terrain edited under one house | 2 plans ran (terrain, village), 0 realize units. One house and the 16 things in it rise. Gardens are not told: the list they read is unchanged. |
| A house moved | Village re-plans, gardens re-plan, 1 garden is re-made. Nothing inside any house runs. |
| A village parameter that leaves the list of houses unchanged | Stops at the village. |
| An edit deep inside a house | 1 realize unit. The village's plan does not run, so no reader hears of it. |
| The whole village node moved | Houses and gardens follow. The house that stood on a bump leaves it. |
| Terrain generator changes version | Terrain is held, and what it published is held with it in the document. Readers see no change, with or without the cache. |
| Village renames its houses, migration declared | Check before upgrade reports 102 ids renamed in the village and 6 in its reader. After upgrade every garden is the same garden under a new id; edits on a garden followed it, labelled Migrated; nothing went to the inbox. |
| Same rename, no migration declared | Reported: 102 ids and 6 reader ids without a successor. The garden edit was re-matched by fingerprint and flagged. |
| Reader frozen during the rename, thawed later | It follows the rename when thawed, because the table is in the document. |
| Deterministic? | Same document without a cache, and the saved document loaded elsewhere, give the same cooked world. The same upgrade on a second engine gives identical tables. |

### 1.2 Probe 2: scale

`cargo run --release --example scale huge`, results in `prototypes/identity/scale-results.txt`. Section 4 has the numbers.

### 1.3 Probe 3: Luau

`prototypes/luau-boundary`: `LuauGen` implements the kernel's `Generator` trait by running a script (mlua 0.10 with Luau, built from source by cargo with no extra setup). The kernel has no Luau-specific code. Section 5 has the results.

### 1.4 Changes to existing behaviour

So that nothing is hidden in the diff:

- `build` takes the previous world as an argument. It uses it for one thing: saying where ids went.
- The id migration table is now produced by the build, not by a pass after it.
- The row generator gained one optional input (ground height). Without a connection it behaves exactly as before; the 39 earlier tests pass unchanged.
- Entities carry a height. The cooked line format has one more number.
- The contract test now runs every plan twice (rule 21).
- The cache holds one entry per owner (rule 20).

---

## 2. What is now proven

"Proven" means: implemented in the prototype, exercised by a test, and the test would fail if the claim were false. It does not mean proven at production scale or with real geometry.

**Identity**

- Ids survive regeneration, parameter changes, pins, exclusions, freezing, detaching and adoption, at three levels of nesting, through one recursive code path.
- A generator author cannot mint a positional key by accident, in Rust (compiler) or in Luau (run-time type check).
- Key renames are recorded as an explicit, deterministic old-id → new-id table, known before commit, including for nodes that read the renamed one.
- Edits are never dropped silently: every record ends as applied, flagged, waiting, or in the inbox.

**Regeneration**

- Work is selective across nesting and across generator links. The unit counts in section 1.1 are asserted, not observed once.
- The cache is not an input: deleting it changes no result, including across links and held versions.
- A generator version change alters nothing until an explicit upgrade, and "upgrade all" is a single transaction whose order follows from the document.

**Structure**

- One generator can read another without the kernel gaining a second evaluation mechanism.
- A scripting language can sit behind the generator boundary without the kernel knowing.
- The document stays small: 0.74 MB for a world of 340,000 things, saved in 3 ms. It is proportional to what was authored, not to what was generated.

---

## 3. Architectural changes discovered

Continuing the numbering from Pass 7 (rules 7–13).

**14. There are no implicit reads between generators.** One generator sees another only through an input its manifest declares and the document connects. Evaluation order is therefore a property of the document, known before anything runs, and a cycle is an error at the moment it is made. The one world-level read that remains is the existing one: a top-level scope is told about authored and detached obstacles.

**15. Only owners have inputs.** Below an owner, everything arrives through the brief (rule 11). The village reads the terrain and writes a height into each house's brief; the house generator never learns terrain exists. Height turned out to be like position: carried down the tree, told to nobody, so raising a house re-plans nothing inside it.

**16. A generator is seen only through what it publishes.** That is two things: values it exports by name, and a list of its children made by the kernel, by id. Readers never see keys. This is what keeps a reader from depending on how another generator spells its names.

**17. A read across a link is a read like any other.** It is recorded with a hash, whole or entry by entry. The terrain field is read per cell, which is why a terrain edit elsewhere costs the village nothing. No new invalidation mechanism was needed.

**18. A stored node goes on publishing what it published, and staleness does not spread.** A frozen or held node's exports are stored in the document with its content. A node that reads a stale node is not itself stale. When the source is upgraded, its readers change in the same transaction and the change is in the diff, exactly as if a parameter had changed.

**19. An id inside a key is followed through the id table.** This is the identity finding of the pass. The garden's key contains its house's id. When the village renames its houses, house ids change, so garden keys change, although the garden generator changed nothing. Three consequences, all now implemented:

- A reader of a renamed node gets its own table entry in the same transaction.
- Records on the reader follow by declaration (Migrated), not by fingerprint.
- A key that names an id with no successor can never match again. It is not left waiting as Dormant; it falls to fingerprint and is flagged.

The lookup uses the document's whole table, not only this transaction's, which is why a reader that was frozen during the rename still follows when thawed.

**20. Units nest: an owner is itself an evaluation unit.** If nothing an owner can see has changed (its state, inputs, obstacles, the project's generator versions), none of its scopes is visited. Found by the scale probe; see section 4. It should hold at every scope, not only the top; the prototype does it for owners only.

**21. The contract test runs every plan twice.** A plan must be a function of what it reads. Hidden state cannot be seen at the boundary in any language; two runs with different answers can. This is now part of the identity contract check for all generators.

**Smaller findings, not yet rules**

- **Archetype names and generator ids share one namespace.** My garden generator first emitted children of archetype "garden"; the kernel expanded each as a nested garden generator. It was harmless only by luck. A generator that emits its own id with parameters would recurse without end. This needs an explicit decision: a separate namespace, or a depth guard, or both.
- **A reader sees what its source has now.** Detach a house and its garden disappears (the edit on it waits, and returns on adoption with the same id). The village also fills the gap with another house, which gets its own garden. Consistent with the rules, but worth your eye: see decision 1.
- **Two generators that must avoid each other form a cycle.** There is no solver. Such a pair has to be resolved by a parent generator or by giving one priority. I think this is the right limit for this engine, but it is a limit.

---

## 4. Performance and scaling results

Release build, this machine, one thread. A scene is K villages of 50 houses; a house is 17 things. Generators here are trivial (boxes described as text), so these numbers measure the engine's bookkeeping, not content generation.

### 4.1 Cost of one transaction, after rule 20

| Scene | Things | Nothing changed | Edit one window | Move one house | First build | Document |
|---|---|---|---|---|---|---|
| 2 villages | 1,700 | 11 ms | 16 ms | 17 ms | 29 ms | 0.01 MB |
| 20 villages | 17,000 | 121 ms | 129 ms | 125 ms | 303 ms | 0.04 MB |
| 100 villages | 85,000 | 718 ms | 941 ms | 637 ms | 1.7 s | 0.19 MB |
| 400 villages | 340,000 | 4.8 s | 4.6 s | 3.7 s | 8.2 s | 0.74 MB |
| 1 village of 1,000 houses | 17,000 | 124 ms | 243 ms | 248 ms | 345 ms | under 0.01 MB |
| 20 villages + terrain + gardens | 18,000 | 332 ms | 368 ms | 378 ms | 632 ms | 0.06 MB |

Units actually run: 2 for the window edit and 5 for the house move, at every size.

### 4.2 What this says

**The work avoided is right; the cost is not.** A transaction costs roughly 7–14 µs per thing in the world, whatever was edited. At 17,000 things an edit takes an eighth of a second. That is too slow to drag something, and 85,000 things is not a large world.

**Where the time goes** (100 villages, nothing changed):

| Part | Before rule 20 | After |
|---|---|---|
| Walk: visiting every scope to verify its trace | 574 ms | 44 ms |
| Assemble: turning trees into entities | 174 ms | 277 ms |
| Cook: per-cell listings | 211 ms | 254 ms |
| Clone the cache for staging | 35 ms | 32 ms |
| Diff the two worlds | 14 ms | 22 ms |
| Remainder (mostly freeing the previous world) | about 230 ms | about 90 ms |

(Assemble and cook did not get slower; run-to-run noise on this machine is about ±30%.)

Before rule 20, verifying that a unit need not run cost about as much as running it, because these units are trivial. Skipping a whole unchanged owner removed that. What remains is everything that rebuilds the world's flat tables from scratch on every transaction.

**Three specific findings**

1. **The world must be made of shared per-owner pieces.** Assembly, cook, diff and the freeing of the old world are all proportional to the world because the world is rebuilt as one flat table. If each owner's entities are an immutable piece shared between the old world and the new one, and each cooked cell is redone only when its members changed, these become proportional to the edit. Nothing in the rules works against this: rule 14 is exactly what makes an owner's result independent of other owners. But it is not demonstrated.
2. **The owner is too coarse a unit on its own.** In the single 1,000-house village, an edit costs 120 ms more than a no-op, because the one changed owner is walked whole. Rule 20 has to apply at every scope.
3. **Inputs must be passed by reference.** The linked scene is 2.7× slower than the unlinked one of the same size. The cause is mechanical: the 128 × 128 terrain field is copied and hashed once per reader per transaction. A field must be hashed once at its source and shared.

**What is fine**

- Document size and save/load time. Undo by keeping previous documents is affordable at any of these sizes (not tested as a feature).
- Selectivity itself.
- First build and reopening without a cache take the same time (8–11 s at 340,000 things, with trivial generators). With real geometry this will be much longer, so the cache should be kept on disk between sessions. That was already the plan; this confirms it matters.

---

## 5. The Luau boundary

### 5.1 Results

| Question | Result |
|---|---|
| Same identities? | Yes. The row generator in Luau, run through an editing session (move, delete, deep edit, count, pitch, colour), gives the same keys, ids and positions as the Rust one, three levels down, with Rust house and facade generators nested under it. |
| Same content? | With whole-numbered parameters declared in the manifest: the cooked world is byte for byte identical. Without: identity is untouched, but every roof hashes differently (see rule 23). |
| Identity contract | The same contract check passes on the script, and the key-scheme snapshot equals the Rust generator's. |
| Forged key (`ctx.child("crate@run.lot0", ...)`) | Transaction refused: "error converting Lua string to userdata", with script name and line. Document untouched. |
| Duplicate key | Refused by the kernel's existing check. |
| Illegal key characters | Refused with an error. (In the kernel this is an assertion; reached from a script unguarded it would have crashed the process.) |
| `os.clock`, `math.random` | Not present. |
| A plan that never returns | Stopped by a budget counted in VM interrupts, so it stops at the same point on every machine. |
| Hidden state in the script | Not visible at the boundary. Caught by rule 21. |
| Selective regeneration | Unchanged. Reads go through the same context, so traces work as before. |
| Cost | Plan of 2,000 houses: Rust 12 ms, Luau 17–19 ms (1.4–1.6×). Most of the time is in kernel helpers either way. |

### 5.2 Boundary rules

**22. Opacity becomes a run-time type check, and a failed check is a refused transaction.** Keys, lots and pins cross as opaque values. A script can hold one and hand it back; it cannot build one from text. What the Rust compiler refused is now refused when the script runs, before commit, with a line number. That is later than compile time, but it is still before anything is saved.

**23. Values that cross the boundary need declared number types.** Luau has one number type. The kernel's values keep integers and reals apart and they hash differently, so a value that merely passed through a script could change its hash and make identical content look changed. Declaring which parameters are whole-numbered fixed it completely. The cleaner fix is in the kernel: see decision 3.

**24. A script's text is its version.** There is no compiler to bump a version number. I used a hash of the source as the code version, so any edit to a script makes it a changed generator, and nodes made with the old text are held until upgraded. That is correct for a shipped project and wrong for someone writing the script, who wants to see each change at once. An authoring mode that upgrades automatically is needed; it is a workflow decision, not an architectural one.

**25. Nothing ambient may be in a script's reach, and its budget must be counted, not timed.** Clocks and random sources are removed; the stop condition is a count. A wall-clock limit would make the same document build on one machine and fail on another.

### 5.3 Not covered by the spike

Tiles and key migrations written in a script; a nested generator written in Luau (only the top-level one was); hot reload; Luau's static type checker, which could give back some of the compile-time checking; debugging. Manifest strings are leaked to satisfy `&'static str` in the trait; the real trait should own its strings.

One thing the boundary does not improve: an anchor key accepts any text, so a careless author can still build one from a loop counter. That is equally true in Rust. The defence is the same in both: the contract test.

---

## 6. What remains uncertain

Stated plainly, because the prototype's success can make these easy to forget.

- **Everything is synchronous.** Every build here runs to completion inside the transaction. Real content generation (meshes, terrain tiles) will take long enough that planning must answer at once and realization must finish later, in the background, with cancellation when the next edit arrives. Nothing in the prototype tests whether staging, diffs and the ladder survive that split. I believe they do, because the ladder and the diff of identities need only plans, not realized content; but that is a belief.
- **Everything is flat boxes.** Positions are 2D, content is a text line. No rotation, no real meshes, no materials, no collision shapes. Fingerprints and their tolerances are untested against real geometry.
- **The world store.** Section 4, finding 1. Designed, not built.
- **The editor protocol.** The core emits a diff of ids; nothing yet carries it across a process boundary to Godot, and nothing yet measures what that costs per edit.
- **Reading deeper than the top level.** A reader gets a node's exports and its top-level children. "A path from the garden to the house's door" needs the door, which is two levels down and decided by the facade generator. The mechanism would be the same (a kernel-made list, by id), but it is not built, and it raises the question of how a reader navigates another generator's subtree without depending on its key names.
- **Identity contract for inputs.** The contract test perturbs parameters. It does not yet perturb inputs, so a generator whose keys churn when its input changes harmlessly would pass.
- **Split and merge migrations.** The code follows one-to-many renames, including through rule 19. Still exercised only lightly.
- **Scatter instances.** Their ids are not in the id table, and they are not addressable by readers.
- **Undo and redo.** Should be cheap, since the document is small and every change is a transaction. Not built.
- **Luau beyond the plan function.** Section 5.3.

---

## 7. Remaining risks, ranked by severity

| Rank | Risk | Severity | Why | What would retire it |
|---|---|---|---|---|
| 1 | Transaction cost proportional to world size | High until fixed, low once fixed | Measured: 0.12 s per edit at 17,000 things. The cause and the fix are understood, and no rule opposes the fix, but it is unproven. | Build the world store as shared per-owner pieces with per-cell cook; re-run `examples/scale`. Target: an edit in a 340,000-thing world costs what it costs in a 1,700-thing one. |
| 2 | Asynchronous realization | High | Untested, and it touches staging, which everything goes through. | A probe where realize units are slow and run after commit; check that diffs, the ladder and cancellation still behave. |
| 3 | Editor protocol and Godot adapter | Medium-high | The whole Approach B bet rests on a diff of ids being enough for a host to stay in sync. Only the earlier Rust/Godot test supports it. | The editor-side adapter, on a small scene, as planned. |
| 4 | Real geometry | Medium | Fingerprints, obstacles, pins and tolerances are all tuned to boxes on a plane. | First real generator (a house with actual meshes) through the same tests. |
| 5 | Reading below the top level across generators | Medium | Likely needed early by real content (doors, road edges of nested streets). Same mechanism, unbuilt. | Small extension of the children list, plus one test. |
| 6 | Script authoring workflow | Medium-low | Rules 23 and 24 need decisions; tiles, migrations and nested generators in Luau are untried. | Decide 3 and the authoring mode; port one more generator. |
| 7 | Generators that constrain each other | Medium-low | A design limit, not a defect. It may surprise content authors. | Write one "parent resolves two children" example to confirm the pattern is comfortable. |
| 8 | Archetype / generator namespace, recursion | Low | Found by accident; easy to fix either way. | A decision and ten lines. |
| 9 | Id table growth and lookup | Low | The table is re-read on every build; at 102 entries per upgrade that is nothing, at a shipped game's scale it wants an index. | Index it when the real core is written. |
| 10 | Split/merge, scatter ids, fingerprint tolerances, attachment order | Low | Unchanged from Pass 7. Nothing in these probes gave a reason to look. | Content will say. |

Risks 1 and 2 are both about the same part of the system: what happens between "the document changed" and "the host sees the new world". Everything upstream of that (identity, records, the ladder, modes, migration, links) held up under every test I could think of.

---

## 8. What can be committed to as the foundation

**Commit to these.** Each has been attacked by at least one test designed to break it.

1. A headless Rust core that owns the document and emits changes; Godot as host through an adapter; separate process first.
2. The document is the only thing saved. It holds inputs and decisions, never generated content, except where a human approved or a version is being held.
3. Identity: assigned ids for what people make, ids chained per level for what generators make, typed keys, positional keys minted only by kernel helpers.
4. Edits as records in a closed vocabulary, written against every segment of their path, resolved by the ladder, never dropped silently.
5. Modes: live, frozen, detached. Detach severs one level. A generator change holds the whole owner until an explicit upgrade; "upgrade all" is one transaction.
6. The id migration table: explicit, deterministic, in the document, reported before commit, followed by keys that contain ids.
7. One evaluation context as the only door, recorded reads, verifying traces, units that nest.
8. One recursive expansion. A nested generator sees only its brief. Only owners have inputs.
9. Links between generators as declared inputs in the document; order from the document; exports by name and children by id.
10. Every change is a staged transaction with a diff that can be inspected or discarded.
11. Identity contracts declared in the manifest and checked by perturbation, for the whole subtree, with the plan run twice.
12. Luau as the generator scripting language behind the same trait, with opaque keys, declared number types, no ambient state, counted budgets.

**Do not commit yet.**

- The shape of the world store, the cook pipeline, and the diff format. Risk 1 says these must be designed around shared per-owner pieces from the first line, and the diff format should be expressed in the same pieces. The prototype's flat tables are the wrong model to copy.
- Anything about scheduling, threads, or background work. Risk 2.
- The exact export API (what "children" contains, how deep a reader can look). Risk 5.
- The value type that crosses boundaries. Decision 3.

**About the prototype itself.** It should not become the engine. Its kernel is a specification by example and its 54 tests are acceptance tests the real core must pass. The real core should be written fresh around the world store that risk 1 calls for.

---

## 9. Decisions for you

1. **A reader sees what its source has now.** Detaching a house removes its garden (the edit on the garden waits and returns on adoption). The alternative is that detached children stay in the list a reader sees. I recommend keeping the current behaviour: a detached house has left the village, and a garden for it should then be a deliberate act. But this is a content-authoring judgement and yours to make.
2. **Staleness does not spread to readers** (rule 18). Upgrading a source changes its readers in the same transaction, shown in the diff. The alternative, holding every reader until it too is upgraded, is safer in a narrow sense and much more tedious. I recommend keeping it as is.
3. **Number types in values.** Either values keep integers and reals apart and every boundary must declare which is which (what the spike did), or values have one number type and "whole-numbered" is a constraint on a parameter. I recommend the second: it removes a class of false "content changed" results at every boundary, including save files and the editor protocol.
4. **What comes next.** My recommendation:
   - Do not extend this prototype to fix risk 1. Make the world store requirement one of the real core.
   - Do one more small probe before or alongside the adapter: asynchronous realization (risk 2), because it is the last thing that could change the transaction model.
   - Then the editor-side adapter on a deliberately small scene, with the change protocol defined in per-owner pieces.

I have not started any of this.

---

## Appendix: running it

```
cd prototypes/identity
cargo test                                   # 48 tests: 28 flat, 11 nested, 9 cross-generator
cargo run --release --example scale          # add "huge" for the 340,000-thing scene

cd ../luau-boundary
cargo test --release -- --nocapture          # 6 tests; l06 prints the Rust/Luau timing
```

Files of this pass:

- `prototypes/identity/src/kernel/idmap.rs`: the id table and "ids inside keys" (rule 19).
- `prototypes/identity/src/kernel/build.rs`: evaluation order, inputs, exports, the owner as a unit.
- `prototypes/identity/src/gens.rs`: `TerrainGen`, `GardenGen`, the row's ground input.
- `prototypes/identity/tests/cross.rs`: x01–x09.
- `prototypes/identity/examples/scale.rs` and `scale-results.txt`.
- `prototypes/luau-boundary/`: `src/lib.rs`, `scripts/row.luau`, `tests/boundary.rs`.
