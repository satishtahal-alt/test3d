# Pass 6: Rust/Godot Integration Test and the Identity Prototype

Date: 2026-10-05. Follows Pass 5. Code is in `probes/rust/`, `probes/godot-rust-probe/` and `prototypes/identity/`.

## 0. Short answer

- **Rust integration with Godot is clean.** The binding built on the first attempt, loaded in Godot 4.7.2, and produced the same world hash as every other route. Rust stays the choice for the core; I found nothing that argues for the C++ fallback.
- **The identity model holds.** All 21 scenarios pass through one set of commands, one matching ladder and one build. The kernel contains no scenario-specific or generator-specific branch, and a test enforces that.
- **The model needed six corrections to get there.** None is a special case; each is a rule Pass 3 was missing or had wrong. Two of them would have caused silent data corruption in a real engine (sections 2.2 and 2.3).
- **One open problem remains that the prototype exposed and did not solve:** a key migration changes the 128-bit ids of the affected children, so anything outside the document that stored those ids (a save game) would break.
- **The prototype does not cover nested generators.** Every generated child is one level below its owner. That is where I would look next before trusting the model fully.

## 1. Rust and Godot

### 1.1 What was built

| Piece | Location | Size |
|---|---|---|
| Rust port of the stand-in core, no dependencies, with a command-line front end | `probes/rust/probe-core` | about 230 lines |
| Godot bridge using the Rust binding (`godot` crate 0.5.5) | `probes/rust/godot-bridge` | 55 lines |
| Godot project that loads the bridge and runs the Pass 5 adapter against it | `probes/godot-rust-probe` | the same adapter script |

The bridge is the entire binding surface: six methods that pass edits in and change-list bytes out.

### 1.2 Results

| Question | Result |
|---|---|
| Toolchain install | One installer, Rust 1.99.0; it used the Microsoft linker already on the machine |
| First build of the binding | Succeeded first try; 2 minutes 6 seconds from clean, 16 crates |
| Loads in Godot 4.7.2 | Yes. The binding is built against the 4.6 interface and Godot accepted it |
| Same result as other routes | Yes: `b33b1987352d6d77` from Rust headless, Rust inside Godot, C++ headless and C++ beside Godot |
| Call overhead | 0.03–0.6 ms per edit on top of the core's own time (one copy of the bytes) |
| Edit to applied, 4 units | about 1.1–1.5 ms; 36 units about 11–13 ms. Same as the separate-process route |

### 1.3 Friction, exactly

- **A project must be opened once by the editor (or imported from the command line) before Godot notices a new native library.** A plain run of a fresh project does not load it.
- **The binding trails Godot by a version.** It targets the 4.6 interface; anything added in 4.7 is not reachable from Rust until the binding catches up. The bridge uses nothing recent, so this does not matter here.
- **A clean build takes two minutes**; later builds take seconds.
- **The library is 3.8 MB** for a 55-line bridge. Irrelevant on desktop.
- **Not tested:** reloading the library while the editor is open, and exporting a game that includes it.

With "separate process first" for the editor, none of this is on the critical path. The in-process route is confirmed to work for when a shipped game needs it.

## 2. The identity prototype

### 2.1 What it is

A headless Rust library with no Godot in it, about 1,900 lines plus 575 lines of tests. Two dependencies, both for JSON. Run it with `cargo test` in `prototypes/identity`.

| Part | Contents |
|---|---|
| Kernel | Ids, typed keys, the document, edit records, the matching ladder, the evaluator with recorded reads, modes, staging, commands, contract tests |
| Generators | `row` (houses along a polyline), `scatter` (lattice instances in a region, tiled), `prefab` (a prefab instance expressed as a generator), one leaf with separate visual and collider outputs, and two deliberately faulty generators |
| Tests | The 21 scenarios and 7 enforcement checks |

It covers everything in the Pass 3 plan: `set`, `pin`, `add`, `exclude`, `reroll`, `attach`, `tag`; live, frozen, detached; thaw and adopt; staged regeneration with a diff; code-version and key-epoch changes with and without migration.

### 2.2 Results

All 28 tests pass.

| # | Scenario | Result |
|---|---|---|
| 1 | Save, load, save | Byte-identical; a fresh engine on the loaded file produces the same pack |
| 2 | Build twice | Second build runs zero units; all hashes identical |
| 3 | Prefab override, then change the prefab | Part ids unchanged; override still applies after the part moves |
| 4 | `set` on a house, change count | No existing house moves; the edit survives count going to 0 and back |
| 5 | `pin`, change count and polyline | Pinned house keeps its id and place |
| 6 | `exclude` a house | Gap stays empty through regeneration and reseeding; siblings untouched |
| 7 | Raise scatter density | Instances added, none moved or removed; a region change re-evaluates only affected tiles |
| 8 | Scatter exclusion, new lattice | All instances change; the excluded spot stays empty |
| 9 | `attach` to a house | Follows the house through sibling regeneration and a move |
| 10 | Visual-only parameter | Visual hashes change, collider hashes do not, the deriver does not run |
| 11 | Edit one house | Zero plan units and exactly one realize unit run |
| 12 | Code-version bump | Nothing changes until upgrade, even with the cache deleted |
| 13 | Key-epoch bump with migration | `set`, `tag` and `attach` records migrate; the pin applies; inbox empty |
| 14 | Key-epoch bump without migration | One record re-matched by fingerprint and flagged; the deliberately ambiguous one goes to the inbox |
| 15 | Stage, then discard | Document byte-identical |
| 16 | Freeze a row | Unaffected by a generator change and a lost cache; thaw shows a diff |
| 17 | Freeze one house under a live row | Counts as one of the row's members and is left alone |
| 18 | Detach one house | Same id; the row treats it as an obstacle |
| 19 | Reconnect by adoption | Same ids; sculpted content and hand placement survive |
| 20 | Copy a generator node | New ids, same key paths, edits carried, copy independent |
| 21 | Delete the manifest and rebuild | Every artifact hash identical |

**On the structural pass condition.** The kernel has zero branches on a scenario or on a generator's name; a test fails if any kernel file mentions one. Two things work with no code of their own, which is the strongest sign the model is general:

- A prefab instance is just a generator whose plan reads an asset.
- A frozen child is just an implicit pin. The `row` generator contains nothing about freezing.

The kernel does rely on two conventions I should name: a child's brief carries its position and footprint under fixed field names, and a tiled generator's region is a parameter called `region`.

### 2.3 Corrections the prototype forced on Pass 3

1. **A seventh outcome: dormant.** Pass 3 sent every record whose target was missing to the inbox. Drag a count slider down and back up and every override on the removed houses would be gone. Now, within one key epoch, a record whose key is currently absent stays in place and applies again when the key returns. The inbox is only for records whose key namespace has changed.
2. **Names are authoritative only inside one key epoch.** Pass 3's ladder tried "path exists" before "migration". After an epoch bump an old name can coincide with a new, unrelated one. The ladder now checks the epoch first: same epoch means match by name; different epoch means migrate, then fall back to fingerprint. Fingerprints only ever bridge epochs.
3. **Detaching must retire the key.** Pass 3 said a detached child keeps its id and has no key in the parent. If you then move the detached house away, the generator refills that lot with the same key, and two objects share one id. The owner now keeps a list of retired keys that the kernel refuses to hand out; adoption releases them.
4. **"Which candidates exist" must not depend on how keys are spelled.** My first `row` ranked lots by hashing the key text. Renaming keys in a new epoch reshuffled which lots were filled: a pure rename silently moved houses. The kernel's layout helper now supplies a ranking token that is independent of the key's spelling. This is exactly the kind of mistake a generator author would make, and the fix lives in the kernel helper, not in author discipline.
5. **"Nothing changes until upgrade" needs the last output stored in the document.** When a generator changes under a live node, the old output lives only in the cache. Delete the cache and it is gone, and old code is never re-run. The engine now moves the last output into the document at the moment it detects the change.
6. **Anchor keys are trusted by name without a fingerprint check.** A prefab part that moves is still that part. Applying the fingerprint test to it produced false "target changed" flags.

Two smaller inconsistencies in Pass 3:

- Scenario 16 changes a frozen node's parameters, but section 2.2 says they are read-only. I kept read-only; the scenario thaws and changes the parameter in one staged transaction.
- A stale live node still accepts parameter edits that have no effect until upgrade. It should be read-only like a frozen node, or prompt for the upgrade.

### 2.4 Open problems the prototype exposed

1. **A key migration changes ids.** An id is a hash of the owner and the key path, so renaming `house@a.lot3` to `plot@a.3` gives the same house a new id. Edit records are fine, because they hold paths. A save game or gameplay script holding the old id is not. Pass 3's "ids never change" is true across freeze and detach but false across a key epoch. The likely fix is for an upgrade to emit an old-id-to-new-id table that the cook ships; I have not built or tested it.
2. **Excluding a member of a count-driven generator fills a lot elsewhere.** Delete one of six houses and the row places a sixth somewhere else to keep the count. That is consistent, but may not be what a user means by delete.
3. **Fingerprint tolerance is a guess** (one metre, 25 percent of size). It needs real cases.

### 2.5 Identity enforcement, as built

| Mechanism | How it works in the prototype | Test |
|---|---|---|
| One canonical child call | `ctx.child(key, …)` is the only way to create a generated child | all |
| Typed keys | Three kinds: anchor (from an already-stable id), topological (only kernel layout helpers can create one), interchangeable (the explicit escape) | e2 |
| No index-to-key constructor | Enforced by the compiler: the constructor is private to the kernel module | e2 |
| Deterministic-key helpers | `lots_along` and `slots` hand out keys together with positions; the generator never sees the index | s4, s7 |
| Duplicate-key detection | Hard error; the transaction is rejected and leaves nothing behind | e1 |
| Interchangeable children cannot be edited individually | `set`, `attach`, `tag` on one are refused; geometric edits still work | e2 |
| Identity regression tests | Generated from the manifest: each parameter is perturbed and key churn compared with what the author declared (stable, monotone, may change) | e2, e3 |
| Explicit identity contract | Per-parameter key effect and the list of key kinds the generator may emit | e3 |
| Diagnostics when a key scheme changes | Stored inputs are re-planned under the new code; a changed key set without an epoch bump is reported, as is a migration that leaves keys without a target | e4, s13 |
| Version compatibility check | Classifies an upgrade as output-only, epoch with migration, or epoch without | s12–s14 |
| Preview before regeneration | Plan-only dry run: keys added and removed, and a count of records per ladder outcome; touches nothing | e5 |
| Fingerprints beside keys | Flag "same name, different thing" within an epoch; re-match across epochs; refuse when two candidates fit | e6, s14 |

The limit stated in Pass 4 stands: the engine cannot know that a stable key still *means* the same thing. Fingerprints catch that only when position, size or type also changed.

### 2.6 What the prototype does not validate

- **Nested generation.** No generator's child is itself a generator, so multi-segment key paths, briefs passed down several levels and partial freezing at depth are untested. This is the largest gap.
- **Split and merge migrations.** The one-to-many code path exists but no scenario exercises it.
- **Attachments acting as obstacles for later layers**, and the planning schedule generally. There is one layer.
- **Staging at scale.** The prototype stages by copying the whole document and cache. A real engine needs something cheaper.
- **Per-slot edits on scatter** (`set` on one tree).
- **The second language.** Pass 3 planned the prototype in both; it exists only in Rust.
- **Scenarios as language-neutral scripts.** They are Rust tests.

## 3. Rust, judged on this work

This is the kind of code the core consists of, so it is the fairest evidence available.

- The library compiled on the first attempt at about 1,500 lines.
- 24 of 28 tests passed on the first run. Of the four failures, one was a genuine design error (correction 4 above), two were mistakes in the tests, and one was a comment tripping the no-special-cases check.
- The module-privacy rule that keeps generators from minting index keys is a compiler guarantee, which is hard to get as cleanly in C++.
- No friction worth recording beyond compile time.

I recommend settling on Rust for the core. The one thing still untested on the Rust side is embedding Luau for recipes.

## 4. Where this leaves the plan

| Assumption behind Approach B | Status |
|---|---|
| Godot can replace generated chunks cheaply and locally | Confirmed (Pass 5) |
| The core can stay host-independent | Confirmed; four routes give one hash |
| Rust can be the core language with Godot as host | Confirmed |
| Identity and intent survive regeneration without special cases | Confirmed for one level of generation; open for nested |
| Godot can deliver the first visual family | Supported by a sketch, not by finished art |
| The adapter works inside the Godot editor (undo, picking, gizmos) | Untested |

## 5. Decisions for you

1. **Extend the prototype to nested generators before anything else?** I recommend it: a village that plans houses that plan facades, with freezing and edits at each level. It is the remaining place where special cases could appear.
2. **Id stability across key migrations.** Accept an alias table shipped by the cook, or require that migrations never happen after a game has shipped saves? This is a policy choice, not only a technical one.
3. **Is Rust now settled for the core?**

## Appendix: running it

- `prototypes/identity`: `cargo test` runs the 21 scenarios (`s01`–`s21`) and the enforcement checks (`e1`–`e7`).
- `probes/rust`: `cargo build --release`, then copy `target/release/godot_bridge.dll` to `probes/godot-rust-probe/bin/` and run `probes/run.ps1 -Project godot-rust-probe`.
- Cargo is at `%USERPROFILE%\.cargo\bin`; a new terminal picks it up automatically.
