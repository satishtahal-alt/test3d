# 3D World-Building Engine — Technical Review, Pass 3

Status: architecture review. No code, no implementation plan.
Builds on [Pass 2](engine-architecture-review-02.md). Where this pass contradicts Pass 2, this pass wins; every contradiction is listed in §0.
Technology facts (library features, binding maturity) are from my own knowledge and were not re-checked online. Items that most need checking are marked *unverified*.

---

## 0. Outcome of the stress test

### 0.1 The central test

> Can this architecture let us build a normal handcrafted 3D game, a procedurally generated world, and a hybrid of the two without creating three separate engines or allowing procedural generation to infect the entire runtime?

**Yes, after two structural changes to Pass 2 and under four conditions.** Neither change is large, but both were real gaps.

**Change 1 — "Build" is its own module, not part of Generation.** Pass 2 drew three peers (authoring, generation, runtime) and left cooking and derivers (collision, navmesh, audio zones) inside generation. A hand-built game needs those too. So there are four parts:

```
 AUTHORING MODEL ──▶ BUILD (always present) ──▶ PACK ──▶ RUNTIME
 (document)          incremental evaluator,
        ▲            artifact store, derivers, cook
        │                    ▲
        └── GENERATION ──────┘
            (optional) generator host, kernel, plan store, staging and matching
```

Generation is one *producer* that feeds Build. Authored geometry is another. A game with no generators links Authoring + Build in the editor and only Runtime when shipped.

**Change 2 — the stage list was wrong as a single line.** `claims → terrain → water → routes → structures → …` cannot express "the village lays out lots along the path" because the path came after the claims. Everything, not only buildings, needs the plan/realize split. The schedule becomes a *planning half* on base terrain followed by a *realization half* on final terrain (§5).

**The four conditions**

1. Prefab overrides and generator edits are one mechanism with one state model (§1, §2).
2. The runtime knows exactly four things that originate in generation: an opaque id, tags, semantic records, and the three weights with promotion. Nothing else (§6).
3. Authored objects take part in generation only through the same fields any generator uses (an authored rock writes an occupancy stamp). That is the whole hybrid mechanism.
4. The three ordinary test games are built with the generation module *not linked*, and the build enforces it.

Where the answer is weaker than "clearly yes": the hybrid case depends on users being able to predict what an edit will do (§1.3). The model is precise; whether it is *learnable* is a prototype question, and it is the second-ranked risk in §11.

### 0.2 What this pass changes from Pass 2

| Pass 2 | Pass 3 | Section |
|---|---|---|
| Overrides target a derived id | Edits are stored on the owning authored root and address targets by **relative key path**. The hash is an index, not the address | §1.2 |
| Every edit is an "override", most applied before generation | Three classes: **input edits**, **attachments**, **ownership changes**, each with a closed vocabulary | §1.3 |
| States: live / locked / detached | Modes: **live / frozen / detached**, the same three for prefab instances. "Locked" and "baked" are one thing: frozen | §2 |
| Open question: keep old generator versions runnable? | No. Frozen artifacts are always stored. Old versions are never re-run | §2.4 |
| Cache key computed from declared inputs | **Verifying traces**: reads are recorded while running and re-checked later. Writes are declared; reads are recorded | §3.1 |
| Observer scale in every cache key | Only in the trace of units that read it | §3, §10.1 |
| Binding time is a property of a parameter | Binding time is **which class of evaluation unit is allowed to read a value** | §3.3 |
| One stage list | Plan layers, then realize layers. The **field registry is the schedule** | §5 |
| Generators declare input slots | Minimum contract is nine fields; input slots are tooling metadata | §4 |
| Generator code changes always wait for you | Per node: **hold** (default in worlds) or **follow** (default in the lab, and while authoring a recipe) | §9.6 |
| Traversal graph filtered per profile at build time | Edges store measured requirements; profiles filter at query time | §10.2 |
| Three peers | Four parts (above) | §6 |

---

## 1. Identity and provenance

### 1.1 Five things that must not be confused

Most override systems rot because these five blur into each other. Each gets one definition, one place it is stored, and one rule about who may read it.

| | Answers | Stored | Written by | Read by | If you deleted it |
|---|---|---|---|---|---|
| **Identity** | *Which thing is this?* | Assigned ids: in the document. Derived and slot ids: nowhere, recomputed | Engine | Everything | References dangle |
| **Generator input** | *What was asked for?* | Document | User, or a parent's plan (as a brief) | Generators | Output changes |
| **Provenance** | *Why does it exist and look like this?* | Build manifest (derived data) | Evaluator, automatically | Editor, tools, AI API. **Never a generator, never the cook** | No output changes. You lose the explanation |
| **Artifact** | *What was produced?* | Content-addressed store | Evaluator | Derivers, cook | Rebuilt from inputs (unless frozen, §2) |
| **Attachment** | *What did the user add that generation does not know about?* | Document | User | The assembly step, after generation | That addition disappears; nothing else changes |

Three rules follow, and they are the tests to apply whenever a new feature is proposed.

**Identity carries no data.** An id never encodes position, type or version. Anything descriptive belongs to provenance or to a fingerprint (§1.5).

**Provenance is write-only from generation's point of view.** No plan function, realize function, deriver or cook step may read it. The check is literal: delete the manifest, rebuild, and every artifact hash must be identical. If something needs provenance to compute a result, that something is really an input and must be moved into the document. Pass 2's "key migration from the old manifest" violated this and is replaced in §1.5.

**An attachment is invisible to the generator of its target.** If the generator would need to see it, it is an input instead.

### 1.2 Identity, refined

The three kinds stand: assigned, derived, slot. Four refinements.

**Derived ids hang from the nearest authored ancestor, by key path.**
`id = hash(owner id, key path)` where *owner* is the authored node whose children are computed (a generator node or a prefab instance) and the key path is the chain of keys below it: `house@seg3.L.2 / facade@south / window@f1.b2`.

Consequences:

- Reparenting or renaming the owner changes nothing, because its assigned id is unchanged.
- **Copying** the owner gives the copy a new assigned id; every derived id below changes, but every key path is the same. So anything that addresses by key path copies correctly for free.
- The 128-bit hash is what the runtime, save games and the id buffer use. The path is what the document uses.

**Prefab instances are the same case.** A node inside a prefab has an assigned id *in the asset*. In an instance, its id is `hash(instance id, inner id)`: a derived id whose key happens to be an anchor key of the best kind. Nested prefabs and prefabs that contain generator nodes need no extra rule.

**Slot ids belong to a lattice, and a lattice is forever.** `slot id = hash(layer id, lattice version, lattice cell, slot index)`. The lattice cell size is unrelated to the streaming cell size, which is a cook setting and can change freely. Each scatter layer declares its frame:

- *World-anchored* (default for ecology): the region is a mask over a fixed world lattice. Moving the region reveals different trees; the ones still inside do not move.
- *Node-local* (moss on a rock, crates on a dock): the lattice moves with the node.

**Runtime-spawned objects** get assigned ids from a separate id space (a reserved bit plus a per-save counter), so they can never collide with document ids.

### 1.3 The edit boundary

Pass 2 said edits should become inputs "wherever possible". That is too eager. The right test is not *can* it be an input but:

> **Should any other generated content respond to this edit?**

If you move a house, the path, the terrain and the trees must respond: input. If you put a script on a door, nothing should respond: not an input. If you sculpt a wall, the neighbours should treat it as a fixed object but nothing can re-derive it: ownership change.

That gives three classes, each with a **closed** vocabulary. Closed is the point: new edit kinds are an engine change, never something a generator or tool invents.

**Class A — Input edits.** Stored on the owner, addressed by key path, delivered to generators as part of their inputs.

| Operation | Meaning | Delivered to |
|---|---|---|
| `set` | Parameter value for a keyed child | That child, as a layer over its brief |
| `pin` | "A member with this footprint and archetype exists here" (in the owner's frame; height optionally ground-relative) | The generator that lays out that member |
| `exclude` | "Nothing of yours in this region", optionally naming a key | Same |
| `add` | A pin with no source key: an extra member the layout did not plan | Same |
| `reroll` | Salt for one key and one named stream | That child |

**Class B — Attachments.** Stored on the owner, addressed by key path, applied by the engine after generation. One flat layer, last write wins per field; an attachment can never target another attachment's effect.

| Operation | Meaning |
|---|---|
| `attach` | An authored node (with its own assigned id and subtree) parented to a generated node. Stores a fallback frame in the owner's space |
| `component` | Add a component, or set one property, on a generated node. Forces that node to entity weight at cook |
| `tag` | Add or remove a semantic tag |

References from elsewhere (a quest script naming `lighthouse/door@main`) are not edits, but they are checked by the same orphan machinery.

Attached authored nodes are ordinary authored nodes, so if flagged as obstacles they write occupancy stamps like any other. They are invisible to their *target's* generator, not to the world. Their stamps enter the schedule just after their target's layer, so they can affect only later layers. A bench attached to a house clears grass; a sign attached to a tree cannot move the path.

**Class C — Ownership changes.** Freeze and detach (§2). The only way to change generated *geometry* by hand is to detach it. There is deliberately no "mesh patch" operation. That is what prevents patch-on-patch.

**How the editor chooses.** The user never picks a class. The choice is mechanical:

1. The gesture maps to a parameter of the target or an ancestor (through a declared handle) → `set`.
2. It moves, deletes or duplicates a member of a generator that declares it accepts pins and exclusions → `pin`, `exclude`, `add`.
3. It adds something the generator does not read → attachment.
4. Anything else (sculpting; moving one window in a grammar that takes no pins) → detach the smallest enclosing leaf, and say so before doing it: *"This makes the south facade yours. The building will keep it as is."*

Step 4 is the honest one. It is where the model says no to "every edit is an input".

**Two forms of delete.** "Remove it and leave the gap" is `exclude`. "Not this one, give me another" is `reroll`. A delete stored only as a key (Pass 2's tombstone) is wrong on its own: the generator would still flatten the ground and bend the path for a house that is not there. The key is kept only as a hint for matching.

### 1.4 Edit records

```
owner          assigned id of the generator node or prefab instance
path           key path of the target, relative to the owner
class + op     as above
payload        value, frame, footprint, attached subtree, …
against        generator id, key epoch, and a fingerprint of the target when the edit was made
```

Records are stored in the owner's document file, sorted by path, one per line, so that text merges are clean.

The **fingerprint** is a small descriptive summary the plan produces for every keyed child: archetype, position in the owner's frame, footprint size, parent key. It exists for one purpose, which is the next section.

### 1.5 The matching ladder

Whenever a regeneration could change what an edit refers to, each record is resolved by the first rule that applies:

| Step | Rule | Result label |
|---|---|---|
| 0 | The record is geometric (`pin`, `exclude`, `add`): it does not need a target | *applies* |
| 1 | Its path exists in the new plan and the fingerprint agrees within tolerance | *matched* |
| 2 | Its path exists but the fingerprint disagrees (the "same name" is now a different kind of thing, or somewhere else) | *matched, target changed* — applied, flagged for review |
| 3 | The generator declares a **key migration**: a pure function from old key path to new key path(s), per key epoch | *migrated* |
| 4 | Exactly one new child has a matching fingerprint within tolerance | *re-matched* — applied, flagged |
| 5 | None of the above | *orphaned* — to the inbox, with the old content offered as a detached object at its old place |

Never: an edit silently lost, or silently applied by step 4 when two candidates qualify.

The migration in step 3 is a function over *keys*, shipped with the generator. It does not read the old manifest. Everything the ladder needs is in the document (the records and their fingerprints) plus the new plan.

### 1.6 Stress cases

"Stale" means: flagged, nothing changes until an explicit upgrade, which runs as a staged transaction with a diff.

| Case | What happens | Weak point |
|---|---|---|
| **Algorithm changes, keys unchanged** | Stale → upgrade. All records resolve at step 0 or 1 | None |
| **Generator version changes** | Three declared kinds: *output only*; *parameters changed* (parameter field ids migrate `set` records); *key epoch bumped* (ladder steps 3–5) | Authors forgetting to bump the key epoch. Caught by a contract test that diffs key sets across versions |
| **Hierarchy change by the user** (reparent the generator node) | Nothing. Ids hang from the owner's assigned id | None |
| **Hierarchy change by the generator** (a `storey` level appears between house and window) | A key epoch bump. The migration is a path rewrite: `house/window@fN.bM → house/storey@N/window@bM` | Needs a migration to be written; otherwise step 4 |
| **User drags a generated child out of its parent** | Not a reparent. It is detach + move: the child becomes authored, and the parent gets an exclusion where it was | Must be explained in the UI |
| **Moving a generated parent** | Edits are in the owner's frame, so they travel. A generator that works in its local frame (a building) does not re-plan; only its stamps move, dirtying terrain at the old and new places. A generator that reads world fields (village, forest) re-plans, with pins as constraints | World-anchored scatter: trees do not travel with the region. A separate "move with contents" command converts the move into pin offsets. Slightly surprising either way |
| **Deleting a generated child** | `exclude` with footprint, key kept as hint | — |
| **Changing density** | Slot thresholds: more slots fill, none move. Capacity is not a limit if the lattice is hierarchical (each level adds slots without disturbing coarser ones) | With a minimum-spacing rule, occupancy of a slot depends on higher-priority neighbours. Under *spatially varying* density a local increase can knock out a nearby lower-priority instance. The disturbance is local and small, not zero |
| **Changing biome boundaries** | World lattice: positions hold. Species is re-drawn per slot from the community now covering it. A `set` on such a slot resolves at step 2 (*target changed*: it was a birch, now a pine) | Correct but generates review items along a moved boundary |
| **Changing seeds** | Anchor and topological keys do not contain the seed, so paths survive and records match by name. Shapes change. For slots the positions change, so slot-addressed `set` records land on step 2. Geometric records are unaffected | A world reseed is by design a different world; the engine preserves intent, not appearance |
| **Changing generator parameters** | The normal case. Pins constrain; keys of untouched parts are stable | Depends entirely on key scheme quality (§1.8) |
| **Changing a prefab used by generated content** | The prefab asset is read by facet: bounds, sockets, content. A placement generator re-plans only if bounds or sockets changed; otherwise only cells containing instances are re-cooked. Edits inside such an instance address `generated path / inner id` and orphan in the ordinary prefab way if the inner node is deleted | — |
| **Replacing the generator with a substantially different one** | The node keeps its assigned id. `pin`, `exclude`, `add` are framework vocabulary and transfer unchanged. `set` records transfer only where the new generator declares a parameter as superseding an old field id. Everything else: steps 4–5. The old output is offered as a frozen copy for comparison | Expect a large inbox. This is a rebuild that preserves placements, not a seamless swap |
| **One object splits into several** | Migration is one-to-many. `set` records fan out only if the migration says so. `attach` goes to the nearest fragment by fingerprint, flagged. A `pin` on the old object stays as a footprint constraint and is flagged | Never fully automatic. Acceptable: this is rare and deserves review |
| **Several merge into one** | Many-to-one. `attach` records all move to the merged node. Conflicting `set` records are a conflict the user resolves. Exclusions are unioned | — |
| **A key changes** | A rename: step 3 if declared, else step 4 | — |
| **Deleting and recreating a generator node** | The edits belong to the node and are deleted with it; undo restores them. A newly created node has a new id and inherits nothing. "Replace generator" is a different command that keeps the id | References from other nodes orphan. Correct, but users will do this by accident once |
| **The generator type itself is removed from the project** | The node cannot be evaluated. It behaves as frozen at its last output, which the store must therefore still hold (§2.4) | — |
| **Baking, unbaking** | §2 | |
| **Copying a generated root** | New assigned id, records copied verbatim (relative paths). The copy is live and independent | — |
| **Copying part of a generated region** (three of the forty houses) | There is no owner to carry them, so the copies are detached content with provenance pointing at the source | Cannot stay generated. Stated in the UI |
| **Moving a generated root across the world** | Same as "moving a generated parent" | — |
| **World partitioning** | Ids never contain a streaming cell. An object that crosses a cell boundary keeps its id. The document is split into files by top-level subtree, never by cell. Edit records live with their owner. Scatter is evaluated per lattice cell on demand and is never materialized as nodes | A single generator spanning the whole world (terrain, a continent of forest) must be *tiled* (§3.2) or its plan becomes the bottleneck |

### 1.7 Does anything assume single-process, local editing?

Mostly not, and it is worth knowing exactly where it does.

| Aspect | Assumes one writer? | Note |
|---|---|---|
| Assigned ids | No | Random 128-bit; two people can create nodes independently |
| Derived and slot ids | No | Deterministic; two machines compute the same ids. A future networked game would get agreement on generated-object ids for free |
| Edit records | Per owner, yes | Sorted one-per-line storage makes git merges usually clean. Two people pinning the same house conflict, as they should |
| The build manifest and artifact cache | Yes: local derived state | Must never be merged. It is rebuilt. This is why nothing irreproducible may live only there |
| Frozen snapshots | No | They are project files (§2.4) |
| Staleness | Evaluated locally | Two machines can disagree about what is stale until both have the same sources. Harmless |
| Bit-identical generation across machines | **Assumed for cache sharing, not for correctness** | See §9.2. Frozen content is stored, not recomputed, so a machine with slightly different floating point still opens the same approved world |
| Undo journal | Per session | — |
| Runtime-spawned ids | Yes | A per-save counter. Networking would need an authority here. Out of scope; the fix would be local to the runtime |

So: the model supports asynchronous collaboration through version control. It does not support simultaneous live editing, and nothing in it would have to be torn out to add an authority for runtime ids later.

### 1.8 What remains uncertain

1. **Key scheme quality.** Every row above that says "keys of untouched parts are stable" is a promise the *generator author* keeps, not the engine. The engine can only test it (perturb each parameter, measure how many keys change).
2. **Fingerprint tolerance.** Too loose gives wrong re-matches; too tight fills the inbox. Needs real cases.
3. **Step 2 volume.** Biome and seed changes may flag hundreds of slot edits as "target changed". Batch review ("accept all in this region") is probably required.
4. **Grammar children.** I still have no good meaning for "pin this window". Current answer: grammars take `set` and `exclude` on topological keys, never `pin`; moving a window detaches the facade.

---

## 2. Freeze semantics

### 2.1 Vocabulary first

"Bake" is used for two unrelated things, and that is a source of confusion in every engine that has both.

- **Cook**: produce the pack the runtime loads. Always derived, always repeatable, never a state of anything. A fully generated, fully live world is cooked every time you press play.
- **Freeze**: stop re-evaluating a piece of computed content and keep what it currently is. A state in the document.

I use only those two words from here on. "Bake a region" in your sense is *freeze*.

### 2.2 The state model

A node whose children are computed has a **source** and a **mode**. That is all.

| | Source: prefab asset | Source: generator |
|---|---|---|
| **Live** | Ordinary prefab instance | Ordinary generated node |
| **Frozen** | Instance that ignores later changes to the prefab | "Baked" node |
| **Detached** | Unpacked prefab | Generated content you now own |

A node with no source is *authored*. Your six terms map as follows:

| Your term | Source | Mode | Edit records |
|---|---|---|---|
| authored | none | — | — |
| prefab instance | prefab | live | possibly |
| generated | generator | live | none |
| generated + overridden | generator | live | some |
| frozen / baked | either | frozen | kept, dormant |
| detached | recorded as history only | detached | dissolved into content |

"Generated + overridden" is not a state; it is live with records. Likewise *stale*, *has orphans*, *reproducible* are **facts the engine computes and shows as badges**, not states. Three modes, and nothing else is a mode.

| | Live | Frozen | Detached |
|---|---|---|---|
| Content is | Recomputed when inputs change | The stored result of the last evaluation | Authored nodes and authored mesh assets |
| Parameters | Editable | Read-only ("thaw to edit") | Gone; shown as history |
| Input edits | Applied | Kept, not applied | Dissolved: already reflected in the content |
| Attachments | Applied | Applied (they work by id, and ids hold) | Become ordinary authored children |
| Geometry editable by hand | No | No | Yes |
| The parent generator sees it as | A member it controls | **A pinned member**: it counts as one of the forty houses and the layout works around it | **An obstacle**: an occupied region; it is not one of the forty |
| Has a key in the parent | Yes | Yes | No |
| Reacts to generator changes | Stale badge, then upgrade | Informational badge only | Nothing |

The one-line distinction to teach: **frozen is the generator's output, kept; detached is yours now.**

### 2.3 What survives

**Ids never change across any transition.** On detach, a node's derived id value becomes its assigned id. Scripts, save games and attachments cannot tell the difference. This is the single rule that makes the state model safe to use casually.

On **freeze**, everything survives: ids, key paths, tags and semantic records, the provenance record (generator, version, resolved parameters, seeds, hashes of what was read), all edit records, and the artifacts. What is suspended is only the ability to react.

On **detach**, ids, tags, semantic records and geometry survive. The provenance record is copied into the document as read-only history. Key paths are kept in that history (they are needed for reconnecting, §2.6). Parameters and input edits stop having any effect.

### 2.4 Storage: frozen means stored

A frozen node's snapshot is two things: its **plan result** (the children it had and the briefs it gave them) and the **artifact hashes** of every frozen evaluation unit below it. Both are written into the project, and the artifacts they name are moved from the disposable cache to project storage.

I recommend doing this for *every* frozen node, not only for those that can no longer be reproduced. "Reproducible" can silently become false: a kernel fix, a compiler upgrade, a library update. If frozen meant "we could recompute it", approved content would be exposed to all three.

This settles a Pass 2 open question. Old generator versions are never re-run, so they never need to stay runnable. The cost is storage, which needs measuring on real content (§12).

It also defines a **release build**: freeze everything, record the snapshot. A later patch built from that snapshot has identical ids, so existing save games remain valid.

### 2.5 Partial freezing

Mode is **per evaluation unit**: a node's plan, a leaf's realization, or one tile of a tiled generator (§3.2). The freeze command defaults to "this node and everything below"; the per-unit granularity is what makes the useful combinations fall out without special cases.

| Situation | Meaning |
|---|---|
| **Child frozen, parent live** | The child is a pinned member of the parent. The parent re-plans around it and does not send it a new brief. If the parent's new plan would no longer produce that key at all, the child is kept and flagged ("approved content the layout no longer asks for") |
| **Parent frozen, child live** | The parent's plan is fixed: same children, same briefs. The child can still be re-tuned through its own parameters and edits. This is "layout approved, still iterating on the houses", and it is probably the most-used combination |
| **Some tiles of a forest or terrain frozen** | Those tiles keep their stored instance tables or heights. Live neighbours read the frozen tiles' edges as fixed boundary input |
| **A spatial region frozen** ("bake this valley") | Shorthand: freeze every unit whose bounds fall inside, and the intersecting tiles of tiled generators |

The weak point is seams. A frozen terrain tile beside a neighbour regenerated with very different parameters will show a join, softened but not removed by the boundary blending. I would report it in validation and not try to solve it.

### 2.6 Thaw, and reconnect

**Thaw** (frozen → live) is always a staged regeneration with a diff against the snapshot. If generator and inputs are unchanged and the build is deterministic, the diff is empty. If the generator has changed since, thaw *is* the upgrade; there is no "live at the old version". You may look at the diff and re-freeze instead.

**Reconnect** (detached → live) is where honesty is required. Hand changes to geometry cannot be converted back into parameters, and I would not attempt to infer them. Two forms are possible, and only two:

1. **Adopt.** Reconnecting a detached *parent* makes it live again while every detached child becomes a **frozen child with its current content as the snapshot**. The generator now controls the layout and works around what you made. Nothing is lost. Children can then be thawed one at a time.
2. **Discard.** Thawing one of those children regenerates it. Its hand edits are lost, and the diff says exactly that before you commit. Attachments re-match through the key path kept in its history.

So a region can always be reconnected to its generator. An individually sculpted object can be reconnected only by giving up the sculpt.

### 2.7 Your questions, directly

| Question | Answer |
|---|---|
| What survives baking? | Everything except reactivity (§2.3) |
| What if the generator later changes? | Nothing happens. A badge says a newer version exists |
| Can a baked region be regenerated? | Yes: thaw, review the diff, commit or re-freeze |
| Can part of a hierarchy be baked while the parent stays generated? | Yes, and the reverse (§2.5) |
| Can it be reconnected later? | Frozen: yes, losslessly. Detached: yes by adoption, or by discarding hand edits (§2.6) |

---

## 3. Selective regeneration

### 3.1 The mechanism: evaluation units and verifying traces

Everything that computes is an **evaluation unit**: a pure function with recorded inputs and hashed outputs.

| Unit | Granularity | Example |
|---|---|---|
| `plan(node)` | One node | The village decides its lots |
| `realize(leaf)` | One leaf node | One house becomes meshes |
| `tile(node, t)` | One tile of a tiled generator | Terrain tile (14, −3); forest lattice cell |
| `derive(deriver, cell)` | One cell | Navmesh tile; collision set; ambience zone |
| `cook(cell, part)` | One part of one cell | The instance buffers of cell (3, 2) |
| `import(asset)` | One source asset | A glTF kit piece |

Asset import is on the list on purpose: it is the same machinery, and it is why Build exists without Generation.

Pass 2 described a cache key computed from inputs. That cannot work as stated, because a generator's reads are only known *after* it runs (which field tiles it samples depends on where its children ended up). The correct scheme, well known from build systems, is the **verifying trace**:

```
trace(unit) = code version
              + list of (what was read, hash of the value read)
              + list of (output name, hash of the output)
```

A unit is up to date if its code version is unchanged and every recorded read still has the same hash. Then its recorded outputs are reused without running it. Checking is cheap: a reverse index from "thing that can change" to "units that read it" gives the candidate set, and only candidates are verified.

So the first two requirements become one asymmetric rule: **writes are declared, reads are recorded.** The engine needs to know what a unit may write before it runs (to schedule it). It learns what the unit read by serving every read through the evaluation context, which is the only door.

### 3.2 What counts as a read

Granularity of reads is the whole game. Each kind has a defined unit of change.

| Read | Unit of change |
|---|---|
| Own parameter | One parameter field |
| Brief from parent | The whole brief value. Briefs are small |
| Style key, context key (season, detail band, profile) | One key |
| Asset | One facet: bounds, sockets, content |
| Field | One tile of one field, at the layer it was read |
| Input shape (anchor, path, region) | The shape |
| Input edits | The records delivered to this unit |
| Backward query ("nearest road edge within 30 m") | **The result**, not the data searched. Verification re-runs the query, which is cheap, and compares |
| Artifacts in a cell (for derivers and cook) | The list of (id, artifact hash, transform) overlapping the cell plus a margin |

Hashing query *results* instead of the queried data is what keeps a prop generator from re-running every time any road anywhere changes.

**Tiled generators.** A generator whose output is naturally a grid (terrain, scatter, ground cover) is evaluated per tile, not per node. Without this, one forest node covering the island is one unit and every edit near it re-runs the lot. With it, painting a clearing dirties the tiles under the brush.

### 3.3 Binding time as access control

Pass 2 treated binding time as a label on a parameter. More precisely, it is **which class of unit may read a value**:

| Tier | Readable by | A change costs |
|---|---|---|
| Generate | plan, realize, tile | Re-running those readers |
| Instantiate | cook only | Re-cooking affected cell parts |
| Frame | runtime systems and shaders only | Nothing |

A generator parameter declares its tier and the context enforces it: a realize function that asks for an instantiate-tier value gets an error, not a value. A shared value such as `season` has no single tier. It may be read at any tier, and the trace shows who read it where. "What will this change cost?" is therefore a query over traces, answerable before the change is made. That is what the dry-run in the command API reports.

### 3.4 How the five requirements interact

```
 change ──▶ reverse index ──▶ candidate units ──▶ verify trace ──┬─ all reads equal ─▶ reuse outputs
            (1: manifest)                                        │
                                                                 └─ a read differs ─▶ run
                                                                                       │
            binding tier decided which units could                                     ▼
            possibly be candidates (5)                             outputs hashed per concern (2)
                                                                                       │
                                              output hash unchanged ─▶ stop here (3: early cutoff)
                                              output hash changed   ─▶ downstream units become candidates
                                                                        cook and derive only the
                                                                        cells and tiles touched (4)
```

Each requirement covers a different failure:

| Without | You get |
|---|---|
| Recorded reads (1) | Either stale output or "rebuild everything to be safe" |
| Concern separation (2) | A colour change invalidates the collider because they were one blob |
| Early cutoff (3) | Every upstream re-run floods downstream even when nothing changed |
| Locality (4) | One house re-cooks the world |
| Binding tiers (5) | A season slider that regenerates geometry |

**Concern separation in practice.** `realize` is one function (a tree's collider comes from the same skeleton as its mesh, so splitting the function would duplicate work) but it writes **named outputs**, hashed separately:

`visual mesh` · `collider` · `surface features` (ledges, handholds) · `stamps` · `instance table` · `semantic records` · `audio traits` · `anchors`

Derivers subscribe to output names. The navmesh deriver reads colliders and never sees a visual mesh.

### 3.5 Example A: one more house

*The village's `houses` parameter goes from 6 to 7.*

| Step | Result |
|---|---|
| `plan(village)` | **Reruns** (it read `houses`). Emits seven briefs. Five are byte-identical to before, one lot is narrowed, one is new |
| `plan(house)` × 7 | Five verify clean and are **reused**. Two run |
| `realize(house)` × 7 | Five **reused**. Two run |
| Claims and flatten stamps | Changed in two terrain tiles |
| `tile(terrain)` | Two tiles **rerun**. All others verify clean: their recorded field tiles are unchanged |
| Water, route corridor | **Untouched**. Nothing they read changed |
| `tile(forest)` | The lattice cells under the new footprint rerun (occupancy changed). A few trees disappear |
| `derive(collision)`, `derive(navmesh)` | The two cells containing changed colliders |
| `derive(ambience)` | Reruns for one zone (tree count changed); the quantized result is the same as before, so **cutoff**: no audio data changes |
| `cook` | Two cells: merged building geometry, instance buffers. The runtime swaps two chunks |
| Lighthouse, dock, every other cell | Never considered |

### 3.6 Example B: wider planks

*The style key `wall.unit_width` goes from 0.18 m to 0.22 m.*

| Step | Result |
|---|---|
| Any `plan` | **None rerun.** Plans read structural constraints (`wall.max_span`), not unit width |
| `realize(house)` for buildings with wooden walls | **Rerun.** Their `visual mesh` output changes |
| Their `collider` output | Byte-identical (walls collide as slabs, not planks). **Cutoff** |
| Their `surface features` output | Identical. **Cutoff** |
| Collision, navmesh, traversal graph, audio | **Untouched** |
| Stone buildings, terrain, vegetation | Never candidates: they did not read the key |
| `cook` | Merged-geometry parts of cells containing wooden walls. Instance parts of the same cells are not rebuilt |

A third variant, for contrast: a roof tint ramp read at frame tier changes a material constant and touches no unit at all.

### 3.7 What necessarily regenerates broadly

| Change | Why it is broad | Can it be narrowed? |
|---|---|---|
| World seed | Everything reads it, directly or through a parent seed | No. By definition |
| Global terrain shape parameters, sea level | Every terrain tile reads them, and nearly everything reads terrain | Only if the change is spatially local (a region mask on the parameter) |
| Kernel change that can alter output (kernel epoch) | In every code version | No. Hence: be slow to change the kernel, and keep frozen content stored |
| Lattice version of a scatter layer | Every slot id changes | No. Hence: a lattice is forever (§1.2) |
| Style swap | Many keys at once | All `realize` units rerun, but **no plans** unless structural constraints differ, and colliders mostly cut off. Broad but shallow |
| A change early in the schedule over a large area | Later layers read its fields | Cutoff still applies tile by tile |
| A generator written to read broadly ("give me the whole style") | Its trace contains everything | Yes: a lint that reports the widest readers |
| Derivers that are global by nature: reachability, route validation, light bounce across cells | Their answer depends on the whole | Make them hierarchical (per cell, then a cheap global stitch) or run them on demand |
| Cook settings (streaming cell size, LOD policy) | Every cook unit | All cook, **no generation** |

The practical consequence: the things you tune most (style, species, facade rules, density, season) are all narrow or shallow. The things that are broad (seed, base terrain, lattice) are decided early and rarely.

---

## 4. The generator contract

Pass 2's descriptor mixed what the engine needs for correctness with what tools need for comfort. Separated, the correctness part is small.

### 4.1 Required for correctness

| # | Field | Why the engine cannot work without it |
|---|---|---|
| 1 | **Id** | Stable name. Edit records and node files refer to it |
| 2 | **Code version** | Source hash for recipes (automatic); declared integer for native generators. Enters every trace |
| 3 | **Key epoch**, and optionally a key migration function | Tells the matching ladder when exact path matching is no longer trustworthy (§1.5) |
| 4 | **Parameters**: for each, a stable field id, type, default, enforced range, and **binding tier** | Hashing, edit records that survive renames, access control |
| 5 | **Layer** in the schedule | Ordering (§5) |
| 6 | **Fields written** | Scheduling, and knowing when a field is final |
| 7 | **Constraints honoured**: any of `pin`, `exclude`, `add` | Tells the editor whether a move can be an input edit or must be a detach (§1.3) |
| 8 | **Child schema**: which node types it may create, with their brief types | Type-checks briefs; lets `set` records be validated |
| 9 | **Functions**: `plan`, `realize`, or `tile` | — |

And four obligations on the code, each checked by an automatic contract test that every generator gets without writing it:

| Obligation | Test |
|---|---|
| Deterministic | Run twice, compare all output hashes |
| All identity is established in `plan`; `realize` creates no children | Enforced by the context: the call does not exist there |
| Keys unique among siblings, and stable | Perturb each parameter slightly; report the fraction of keys that change |
| Declared constraints are actually honoured | Inject a random pin and a random exclusion; check the output |

### 4.2 Produced by the engine, never written by the author

| Item | Source |
|---|---|
| Read manifest | Recorded by the context |
| Produced artifact types | Observed from what was written through the kernel builders |
| Provenance | Recorded by the evaluator; node ids stamped onto faces by the mesh builder |
| Fingerprints | Computed from each child's type, frame and footprint |
| Seeds | Derived from keys and stream names |

You asked for a minimum contract for "dependency manifest", "produced artifact types" and "provenance". The minimum is zero: a generator author writes none of it. That is a design goal in itself, because anything hand-declared will eventually be wrong.

### 4.3 Optional metadata, for tools only

Display name and documentation · parameter **tier** (Compose / Tune / Author) · UI ranges and steps · **handles** · presets · **cost class** · expected inputs (which shapes it wants, so the editor can offer to create them) · key scheme description · thumbnail.

Two demotions from Pass 2 deserve a note. **Cost class** is a scheduling hint only; the engine measures real times and trusts those. **Input slots** are a convenience for the editor ("this generator wants a region"), not a correctness mechanism, because the trace already records what was really read.

Removing all optional metadata must leave every output hash unchanged. That is the test for whether something belongs in §4.1 or §4.3.

---

## 5. Generator cooperation

### 5.1 The stage list fails its own example

Pass 2's order was `claims → terrain → water → routes → structures → vegetation → props → derivers`, and its coastal village example said the village "plans six lots along the path corridor" during claims. But the path is a route, three stages later. Three more cases break the same way:

- A river must carve terrain, but water comes after terrain.
- A bridge must know where the river is, but routes come after water and structures after routes.
- A route wants to avoid buildings; buildings want to front the route.

Every one is the buildings-and-terrain cycle again, and has the same cure: **decide where things go on the base terrain, then build them on the final terrain.** Applied to everything, not just buildings, the schedule has two halves.

```
 PLANNING HALF  (cheap; vector data; reads base terrain)
   P0  base       base elevation before any stamp; sea level; world partition
   P1  regions    biome and district regions; reserved areas
   P2  networks   route corridors, street graphs, river courses
   P3  parcels    a) fixed: anchored single structures claim footprints
                  b) layout: settlements and blocks subdivide around networks and fixed claims

 REALIZATION HALF  (expensive; geometry; reads final fields)
   R1  terrain    base + all stamps (flatten, cut, carve) → final elevation and its derived fields
   R2  water      surfaces, shoreline distance, flow
   R3  routes     path and road surfaces, steps, bridges
   R4  structures massing → facades → roofs; published anchors and surface features
   R5  vegetation canopy → understory → ground cover
   R6  dressing   props, decals, weathering, lights, spawn points
   R7  derivers   collision, navigation, traversal, audio, lighting data
```

`props` is renamed *dressing* because weathering, decals and lights belong there too.

**What this cannot do, stated plainly: emergent features cannot influence layout.** A pond that appears because the final terrain happens to have a hollow is discovered in R2, after every plan has run. A route will go straight through it. Anything a planner must respect has to be *declared* in the planning half, as a region or a network. I think that restriction is right for an art-directed engine, since a pond nobody asked for should probably not reroute a road, but it is a real limit and it should be known.

### 5.2 Are these stages sufficient, and where do they extend?

Sufficient for everything in the vision so far. I checked: interiors (a hierarchy inside one structure node, not a global stage), creature populations (spawn points in dressing), reactive capacity maps (derivers), soundscapes (derivers), traversal requirements (§10.2).

**The stages are fixed and owned by the engine. Generators cannot add one.** What is extensible is the **field registry**, and that is where the real schedule lives:

```
field        elevation   shade          keep_clear    occupancy
type         height      scalar 0..1    mask          mask
written at   R1          R5.1           P2            P3, R4
compose      (single)    max            union         union
default      —           0              empty         empty
```

Each field declares the layer at which it is written and how overlapping writes combine. Layers have sub-layers (`R5.1` canopy, `R5.2` understory, `R5.3` ground cover). The rules are then two lines:

- A generator may **write** only fields of its own layer.
- A generator may **read** only fields of strictly earlier layers.

Canopy-before-understory is not a special vegetation feature; it is `shade` being written at R5.1 and read at R5.2. Adding a new kind of interaction means registering a field at a layer, which is data. No dependency graph is solved, the check is static at registration and enforced at run time by the context, and **a cycle cannot be expressed**.

### 5.3 The four channels

| Direction | Channel | Carries | Rule |
|---|---|---|---|
| **Down** the tree | **Brief** | A typed value from a parent's plan to one child: its lot, archetype, age, door side | The child cannot see its parent or its siblings. A user `set` edit is a layer over the brief: one mechanism, and the *Why?* panel labels each value with its origin |
| **Sideways**, through space | **Fields and stamps** | "Flat here", "occupied here", "keep clear here", "shaded here" | Stamps are vector records (shape, profile, value) emitted by plans and rasterized per tile on demand. Combination is by the field's declared rule, so it does not depend on evaluation order |
| **Sideways**, by name | **Anchors** | Named frames with tags, published by a plan or placed by the user: `door@main`, `mooring`, `chimney_top` | Queryable by later layers |
| **Backward** in the schedule | **Queries** | A closed set supplied by the engine: sample field; nearest point on a network; anchors by tag within a radius; claims in an area; instance records in an area; surface features in an area | Read-only; earlier layers only; the result is what gets recorded (§3.2) |

There is **no upward channel**. A parent never receives anything from a child. If the village needs each house's door to face the street, the village decides the door side and puts it in the brief. If a later generator needs to know where the doors ended up, it queries the published anchors.

There are deliberately no queries over another generator's *meshes*. Props sit on published anchors and surface features, not on ray casts against whatever happens to be there. That one restriction keeps generators from depending on each other's geometry.

### 5.4 Conflicts

| Conflict | Handling |
|---|---|
| Siblings under one parent | Cannot happen by construction: the parent hands out disjoint lots in the briefs |
| An anchored structure inside a settlement's region | Fixed claims (P3a) are written before layouts (P3b) read them. The settlement plans around the lighthouse |
| Two stamps overlapping in one field | The field's compose rule. For priority-based fields the order is: authored, then pinned, then by explicit priority, then by id. Never by evaluation order |
| Two *authored* intentions that contradict (two anchored buildings overlapping; a pin inside an exclusion) | **Not resolved.** Both are built, and validation reports an error at the place. The engine never silently picks between two things the user asked for |
| A generator cannot satisfy its constraints (no room for seven houses) | It produces what it can and reports the shortfall as a validation item with a number: "placed 5 of 7" |

### 5.5 What prevents circularity

Not discipline: structure.

1. Fields have fixed layers; reads are strictly earlier. Checked statically.
2. Briefs flow down a tree.
3. Queries go backward only.
4. Nothing flows up.
5. Validation and repair run last. Repair may insert pieces (§10.2) but may never trigger a re-plan.

If a real need appears that these cannot express, the answer is a new field at a suitable layer, or moving a decision earlier into a plan. It is never iteration.

---

## 6. Runtime representation

### 6.1 Seven things

| Thing | What it is | Lives in | Who can see it | In a shipped baked game |
|---|---|---|---|---|
| **Scene document** | Authored nodes, sources, parameters, input shapes, edit records, frozen snapshots | Project files | Authoring, Generation, Build | Absent |
| **Plan data** | The expanded tree below generator nodes: derived nodes, briefs, stamps, anchors, fingerprints | Plan store (derived) | Generation, editor | Absent |
| **Artifacts** | Typed, content-addressed outputs, split by concern | Artifact store | Build | Absent |
| **Runtime entity** | Row in the entity store with components | Pack → memory | Runtime, scripts | Present |
| **Instance** | Row in an instance table: transform, variant, id, record index | Pack → memory | Runtime | Present |
| **Merged / density** | Triangle ranges in a cell mesh; GPU-expanded density fields | Pack → memory | Runtime | Present |
| **Semantic record** | Typed columns per instance or node: species, age, substance, … | Pack → memory | Runtime, scripts, audio | Present |

### 6.2 The pipeline

```
 SOURCE DOCUMENT
     │  plan units (Generation)          authored nodes skip this box entirely
     ▼
 PLAN DATA ── expanded tree, stamps, anchors
     │  realize and tile units (Generation)     │  import units (Build): authored meshes, kits
     ▼                                          ▼
 ARTIFACTS ── meshes · colliders · instance tables · surface features · field tiles · records
     │  assembly (Build): apply attachments; resolve prefab instances; choose weight
     │  derive units (Build): collision, navigation, traversal, audio zones, lighting data
     │  cook units (Build): per cell and part — merge, batch, LOD, id tables
     ▼
 PACK
     │  mount global tables; stream cell chunks
     ▼
 RUNTIME WORLD ── entities · instance sets · merged meshes · fields · nav · audio
     │
     ▼  renderer · physics · audio · animation · scripts · save
```

**What a pack contains**

- *Per cell, in separately replaceable parts*: entity list (components in the same schema types as the document) · instance tables · merged meshes with a range table · density fields · collider sets · static field tiles · navigation and traversal tiles · audio emitters and zones · lighting data.
- *Global*: id index (id → cell, kind, row) · semantic record tables · entity templates (cooked prefabs, for spawning) · assets · tag registry · world settings.
- *Development builds only*: a sidecar mapping id → key path and provenance, for the *Why?* panel during play. Stripped from release.

### 6.3 What the runtime is allowed to know

This is the firewall. The runtime knows:

1. **An opaque 128-bit id** per object. It cannot tell assigned from derived.
2. **Tags and semantic records.**
3. **Three weights, and promotion.** An instance archetype may name an entity template to become when gameplay touches it.
4. **Chunk replacement.** A cell part can be swapped for a new version; objects whose ids persist keep their runtime state.

It does not know: generator, key, key path, brief, edit record, mode, plan, layer, stamp, trace, manifest. None of those words may appear in a runtime header, and the build can check that mechanically.

Chunk replacement is not a concession to generation. The editor needs it for every edit, including moving a hand-placed lamp, so it exists in an authored-only engine anyway. It is also the entire interface a Tiny Glade-style game uses: the generation module, shipped in the game, produces new cell parts; the runtime swaps them in.

**Save games** store deltas by id: components changed, instances promoted or destroyed, runtime-spawned entities. Nothing in a save refers to generation.

### 6.4 A game with zero procedural generation

| Step | What happens |
|---|---|
| Document | Authored nodes and prefab instances. No generator nodes, no plan data |
| Build | Import units for assets. Assembly resolves prefab instances and their overrides. Static meshes are batched per cell by the same cook that merges generated walls: this is ordinary static batching. Collision cooking, and navmesh if wanted, are the same derivers |
| Pack | Usually one cell. Entity list, entity templates, a few merged meshes |
| Editor | Keeps the in-memory pack current through chunk replacement. The user does not experience a "build" |
| Shipped game | Runtime only. Generation is not linked, and neither is Build |
| What the developer meets | Nodes, prefabs, overrides, components, scripts. The words *generator*, *plan*, *stage*, *freeze* never appear |

Tetris pays for: an id on each entity, and a cook step it cannot see. That is the full tax.

One thing it *gains* from the shared foundation: freezing and detaching prefab instances, and the orphan inbox when a prefab changes under its instances. Those are useful in any engine.

### 6.5 The hybrid

A hand-built village square inside a generated town needs no third system:

- Authored nodes flagged as obstacles write `occupancy` and `keep_clear` stamps at layer P3a. Generators plan around them.
- Authored anchors and paths are generator inputs.
- Generated content can be frozen or detached piece by piece and edited as authored content, with ids intact.
- Authored objects can be attached to generated ones.

Everything on that list is a mechanism already defined above. That is the evidence for the "yes" in §0.1.

---

## 7. The C++ / Rust bake-off

### 7.1 A bias to admit first

The Pass 2 proposal (write the identity prototype in both languages) is **biased toward Rust**. The prototype is entirely own-code: ids, hashing, serialization, tree transformations, diffing. That is the half of the engine where Rust is strongest. The half where C++ is strongest, integration with Jolt, an animation runtime, Recast, a docking editor UI, is not exercised at all.

So the bake-off has two parts, and neither alone decides.

### 7.2 Part A — the identity prototype

Headless library, command-line driver, no graphics. Target size: roughly three to four thousand lines per language. If it grows past that, the scope was wrong.

**Contents**

| Element | Minimal form |
|---|---|
| Document | Node tree in JSON. Authored nodes; one prefab asset (a lamp post with two children); prefab instances; generator nodes |
| Generator `row` | Places box "houses" along a polyline. Keys by segment and ordinal. Honours `pin`, `exclude`, `add`. Each house is a child with a brief |
| Generator `scatter` | Stable lattice scatter inside a polygon with a density. Slot ids. Tiled |
| Leaf `house` | `realize` writes two outputs: a visual box list and a collider box |
| One deriver | Per-cell collision summary. Exists to demonstrate early cutoff |
| Cook | Per-cell listing of entities and instances with ids |
| Edits | `set`, `pin`, `exclude`, `reroll`, `attach` |
| Modes | Live, frozen, detached; thaw; adopt |
| Evaluator | Verifying traces, per-concern output hashes, early cutoff |
| Staging | Regeneration into a staging area; structured diff; commit or discard |
| Versions | Bump `row` to a new code version (stale); bump its key epoch with and without a migration (the ladder) |

**Scenarios.** Written once, language-neutral, as JSON command scripts with expected JSON results. Both implementations must pass the same files.

| # | Scenario | Tests |
|---|---|---|
| 1 | Save, load, save: byte-identical | Serialization |
| 2 | Build twice: identical hashes | Determinism |
| 3 | Prefab instance with an override; change the prefab | Derived ids for prefab children |
| 4 | `set` on one house; change the house count | Stable derived identity |
| 5 | `pin` one house; change count and polyline | Pin as constraint |
| 6 | `exclude` one house; regenerate | Exclusion |
| 7 | Raise scatter density | No existing instance moves |
| 8 | `exclude` one scatter instance; change the lattice version | Geometric edits survive |
| 9 | `attach` a node to a house; regenerate siblings | Attachment by path |
| 10 | Change a parameter read only by `realize` visual output | Collider hash unchanged; deriver not rerun |
| 11 | Change one house's parameter | Exactly one realize unit runs |
| 12 | Code version bump | Stale; nothing changes until upgrade |
| 13 | Key epoch bump with migration | Records migrate |
| 14 | Key epoch bump without migration | Fingerprint re-match; one deliberate ambiguity goes to the inbox |
| 15 | Staged regeneration, then discard | Document byte-identical to before |
| 16 | Freeze the row; change its parameters; thaw | Nothing changes while frozen; diff on thaw |
| 17 | Freeze one house under a live row; change count | Frozen child acts as a pin |
| 18 | Detach one house; change count | Acts as an obstacle; id unchanged |
| 19 | Reconnect by adoption | Detached children become frozen children |
| 20 | Copy a generator node | New ids, same paths, edits carried |
| 21 | Delete the manifest and rebuild | All artifact hashes identical (provenance is not an input) |

A bonus worth trying, not a requirement: both implementations produce the *same hashes as each other*. That would show the canonical serialization is truly specified and not an accident of one implementation.

### 7.3 Part B — integration probes

Four small probes, each half a day to a day, in **both** languages. They are not a prototype; they answer "does this work at all, and how ugly is it?"

| Probe | Question |
|---|---|
| **Luau host** | Expose six kernel functions and a handle type to Luau. Run the `row` generator's plan as a recipe. Hot-reload it. Deliberately misuse a stale handle from script and see what happens |
| **Physics** | Jolt: step a character controller on a heightfield and a triangle mesh. From Rust this means bindings, whose completeness I have not verified, or Rapier, whose character controller is simpler |
| **Animation** | Load a skinned glTF, sample and blend two clips, apply a two-bone IK. In C++ with ozz. In Rust with whatever exists, or measure the cost of writing it |
| **Editor UI** | A docking layout with a tree view, a property grid generated from a schema, and a 3D viewport texture. Dear ImGui in C++; egui or ImGui bindings in Rust |

### 7.4 Protocol

1. Write the scenario files and a short written spec first. Neither implementation sees the other.
2. Same assistant, same brief, separate fresh sessions. Record the wall time, the number of build-fix cycles, and every point where you had to intervene.
3. After both pass: three **change requests**, applied to each. This is the most informative step, because the engine will be changed far more often than it is written.
   - Add a new edit operation end to end.
   - Change a stored type and write its migration.
   - Make `realize` run on a thread pool.
4. **Seeded defects.** A separate session inserts five plausible bugs into each codebase (an iteration-order dependency, a stale handle, a missed field in a hash, a data race, an off-by-one in the ladder). Count how many are caught by the compiler, by the tests, by sanitizers or Miri, and by you reading the diff.
5. **Reading test.** You read one three-hundred-line slice of each (the matching ladder is a good choice) and explain it back. Your own verdict on which you could review week after week counts as data.

### 7.5 What is observed, per requirement

| Requirement | Where it shows | My expectation before the test |
|---|---|---|
| Memory ownership | Staging keeps two versions of a tree alive; handles from script | Both end up with ids and arenas instead of pointers. Rust forces it; C++ permits skipping it |
| Tooling integration | Part B, editor probe | C++ ahead |
| Reloadable generators | Part B, Luau probe | Even. Recipes are script in both, so native hot reload matters much less than usual |
| Scripting integration | Part B, Luau probe: glue lines per function, handle safety, error quality | Roughly even; Luau is native C++, Rust has a mature binding |
| Debugging | Step through a failing scenario on Windows; inspect a node map | C++ ahead |
| Ecosystem | Part B, physics and animation probes | C++ clearly ahead |
| Iteration speed | Incremental build time after a one-line change; test suite time | Even; both acceptable at this size, which says little about size ×50 |
| Performance | Time for scenario 7 at a million slots | Even |
| GPU integration | Not tested. Desk judgment: both have a good path (SDL3 GPU; wgpu) | Even |
| Long-term maintainability | The three change requests; seeded defects | Rust ahead |
| Clean native API to Luau | Luau probe; how one schema type becomes serialization + hash + binding | Rust ahead on boilerplate through derives, unless schema-first generation closes the gap in C++ |

### 7.6 Decision rule

Set before running, so the result cannot be rationalized afterwards.

**Choose Rust if all of these hold:**

1. You read the Rust slice at least as comfortably as the C++ slice.
2. Part A shows a material difference: roughly half the defects surviving to review, or clearly cheaper change requests.
3. The physics probe gives a character controller you would ship, and you accept writing or adopting a smaller animation runtime.
4. The editor probe is not a material step down.

**Choose C++ if any of these hold:**

1. A Part B probe fails materially (no acceptable physics or editor route).
2. Part A shows a small difference. A tie goes to C++, because of libraries, debugging and editor tooling.
3. You read C++ more comfortably.

**If the result is split** (Rust wins A clearly, C++ wins B clearly), which is the outcome I think most likely: choose by which half is the bigger risk. My judgment is that the authoring and build half is the novel, error-prone part and the runtime half is well-trodden, which leans Rust; but the runtime half is where one developer most needs to *not* write things, which leans C++. I do not have a confident prior, and that is why the test is worth running.

One thing would settle it without a test: if you can comfortably read only one of the two, use that one.

---

## 8. Rendering: what must be decided now

The aim is not a renderer design. It is the short list of decisions that other modules will build against, so that the renderer can change later without them noticing.

### 8.1 Decide now

| # | Decision | Why now |
|---|---|---|
| 1 | **The renderer consumes a per-frame snapshot.** Each frame, visible state is extracted into an immutable render view: cameras, draw lists referring to renderer handles, light list, environment block, field bindings. The renderer never reads the entity store, the pack or the document | This *is* the replacement boundary. It also makes headless rendering, thumbnails and later threading straightforward |
| 2 | **GPU resources are owned by the renderer only.** Everything else holds generation-checked handles. CPU-side mesh and texture data is owned by the asset system; the renderer uploads on demand and may drop and re-create (streaming, device loss) | Ownership is the hardest thing to retrofit |
| 3 | **Pack geometry is renderer-agnostic**: vertex streams with declared meanings, not GPU buffer blobs. Fixed vocabulary: position, normal, two colour sets, optional UV, wind weights, feature-edge flags. Plus named *custom streams* declared in the schema; a shading model declares which it needs and the cook validates | Generators write this vocabulary. Changing it later means touching every generator |
| 4 | **Ids reach the GPU only as 32-bit indices** into a per-cell table. The prepass writes the index; the CPU resolves it to the 128-bit id | Affects pack format, picking and outlines |
| 5 | **A material is: shading model id + parameter block + texture references + substance reference.** No shader graphs. Generators emit material references and per-vertex data, never shader code | Keeps generation independent of the shading language |
| 6 | **Shading model as an interface with a fixed shape**: the engine owns the lighting loop (lights, shadows, indirect, fog) and calls the model for surface response. A model cannot fetch lights itself | Lets lighting change without rewriting models, and models be added without touching lighting |
| 7 | **Indirect light through one function**, roughly `irradiance(position, normal) → colour, occlusion`. Behind it: a constant sky and ground gradient, or per-vertex data from a deriver, or a probe volume. The pack reserves a per-cell "lighting data" part whose content is opaque to everything except the active provider | The technique is unknown (experiment E3); the *seam* must not be |
| 8 | **Forward shading with a depth, normal and id prepass; MSAA assumed** | Forward versus deferred shapes the shading-model interface and the anti-aliasing story. Clustering of lights is an implementation detail behind it and can arrive when a night city needs it |
| 9 | **A render graph exists, as code.** Passes declare what they read and write; the graph orders them and owns transient targets. The pass set is engine code. A style selects among passes and sets parameters | Having passes declare resources is cheap now and painful to add later. Making the graph data-driven or user-extensible is not needed |
| 10 | **Camera-relative rendering, reverse depth, linear HDR internally** | Nearly free now; each is a renderer-wide change later |
| 11 | **Water, vegetation and atmosphere get data contracts, not implementations.** Water: a surface type with its own artifacts (shoreline distance, flow, foam mask). Vegetation: drawn from pack instance tables with per-instance variant and tint. Atmosphere: one environment parameter block driven by mood | The Build side needs to know what to produce |
| 12 | **SDL3 GPU on the C++ path; wgpu on the Rust path.** A browser build is out of scope, which removes the reason to consider WebGPU under C++ | Unblocked by your scope decision |

### 8.2 Renderer replacement

With the above, replacing the renderer means re-implementing "snapshot in, image out" plus the resource manager. The contract that survives is: the snapshot schema, the geometry vocabulary, the material description, the indirect seam, and the artifact contracts of item 11. Shading models, being shader code, are rewritten. That is the right place for the cost to fall.

I would not abstract further for hypothetical backends. The engine's own GPU layer stays thin and WebGPU-shaped, as in Pass 2, and that is sufficient insurance.

### 8.3 Check early in the first renderer (*unverified*)

Three capabilities of SDL3 GPU that §8.1 relies on and that I have not confirmed against current documentation: storage buffers readable in fragment shaders (for light lists), indirect multi-draw (for GPU-culled instances), and texture readback (for id picking). If any is missing, the fallback is the WebGPU library under the same engine layer, which is a contained change.

### 8.4 Safe to leave undecided

The indirect technique itself · shadow filtering · light clustering details · water wave model · cloud rendering · outline technique mix · the post stack and tonemapper · volumetric fog · LOD and impostor scheme · GPU culling strategy · transparency sorting · particle rendering · threaded rendering · whether the graph ever becomes data-driven.

Each of these sits entirely behind one of the twelve decisions.

---

## 9. The Luau boundary

### 9.1 Four places code or data can live

| | Native | Luau gameplay script | Luau generator recipe | Data only |
|---|---|---|---|---|
| **Holds** | Everything per-element (vertices, samples, slots, bodies); all services; the evaluator; identity; anything threaded | Game rules, behaviour, UI logic, sequencing | *Decisions*: what exists, where, which rule applies, how parts are assembled | Parameters, presets, styles, species, substances, kit metadata, profiles, tag and field registries, audio events, input maps |
| **Runs** | Always | In the game | In Build (and in games that ship generation) | Never |
| **May call** | — | Entity, input, audio, physics-query, UI, save APIs | The kernel and the evaluation context. **Nothing else** | — |
| **May not** | — | Touch the document or the kernel (in a baked game they are not there) | See entities, time, input, files, other nodes, or any state that outlives the call | — |
| **State** | Owns all of it | In components, never only in script variables | None. A pure function | Is state |
| **Versioned by** | Kernel epoch (output-affecting) and API level (signatures) | Gameplay API level | Source hash of the recipe and everything it requires | Content hash |

The line between recipe and data: **if it has no control flow, it is data.** The line between recipe and native: **a recipe makes decisions; the kernel touches elements.** A recipe that loops over vertices or field samples is in the wrong tier.

Gameplay and recipes run in **separate environments with different API surfaces**. They share the language and nothing else. A Tiny Glade-style game does not let gameplay script call generators; gameplay issues *commands to the document* (move this anchor, set this parameter), exactly as the editor does.

### 9.2 Determinism

What recipes may not use: wall-clock time, `math.random` (replaced by named streams), the `os` library, iteration over tables keyed by reference (order depends on memory addresses; an ordered map is provided), shared mutable globals (the environment is frozen).

**How deterministic must it be?** Three levels, and I recommend promising only the first two.

| Level | Promise | Needed for |
|---|---|---|
| Same binary, same inputs → same bytes | **Yes, always.** Tested on every generator | Caching, early cutoff, the whole of §3 |
| Same ids on every machine | **Yes.** Ids come from keys, never from floating-point results | Save games, collaboration, patches |
| Same bytes on every machine and compiler | **Goal, not promise** | Sharing caches between machines |

Two design rules make the second level hold even when the third does not:

- **Existence is decided on quantized values.** Whether a slot is occupied, how many bays a facade has, which species is drawn: integer or fixed-point comparisons. Floating point decides *shape* only. A last-bit difference then changes a vertex by a hair, never the set of objects.
- **Frozen content is stored, not recomputed** (§2.4).

Transcendental functions (`sin`, `pow`) are the usual source of cross-machine differences. Recipes get them from the kernel so that at least one implementation is used everywhere. Whether Luau's native code generation gives bit-identical arithmetic to its interpreter I have not verified; test it before enabling it for recipes.

### 9.3 Performance

Budget, as a rule of thumb to be measured: a plan in about a millisecond; a realize making on the order of hundreds to a few thousand kernel calls. The kernel API must be shaped so that this is enough, which means batch operations ("repeat this piece along this edge at this spacing") and not element callbacks ("call my function per vertex"). A script callback per element is the one pattern that makes this approach slow.

Terrain synthesis, scatter evaluation and anything else inherently per-element stays native, as Pass 2 said.

### 9.4 Memory ownership

- Kernel objects created during an evaluation unit (mesh builders, shapes, field views) live in an **arena owned by that unit**. Luau holds small handles (index plus generation). When the unit ends the arena is released and every handle is dead. The script garbage collector never controls a kernel object's lifetime.
- Gameplay scripts hold entity handles (id plus generation), never pointers. Component access is by copy or by a proxy validated on each access.
- Using a dead handle is a script error with a clear message, never undefined behaviour. This is one of the things the Luau probe in §7.3 tests.

### 9.5 API stability and serialization

- Both script APIs are generated from the schemas, together with type declaration files so the Luau type checker catches misuse before anything runs.
- Two independent numbers: the **kernel epoch** changes when output could change; the **API level** changes when a signature does. A recipe's trace includes the epoch; its header declares the level.
- **No arbitrary script tables are ever stored.** Everything that crosses out of a recipe goes through a kernel builder or is a schema-typed value (briefs, semantic records). Everything a gameplay script wants saved is in a schema-typed component. Script-defined components are declared with a schema, at load time.
- The test games and the lab stations are the canaries. If they break, the API changed.

The real stability risk is the **recipe-facing kernel API**. Every recipe ever written depends on it, and changing a signature means touching all of them. Mitigation: very few recipes until the API has survived two deep generators.

### 9.6 Hot reload, and a correction to the staleness rule

Pass 2 said generator code changes never apply automatically. Pass 2 also promised sub-second recipe iteration. Those conflict.

Resolution: each generator node has an **upgrade policy**.

- **Hold** (default in worlds): a code change marks the node stale and waits.
- **Follow** (default in the lab; switchable per node while you are authoring a recipe): a code change re-evaluates immediately.

One flag, no new state. Frozen nodes ignore it.

Gameplay hot reload replaces functions and keeps component data. State kept only in script variables is lost on reload, which is the practical reason for the "state lives in components" rule. That rule also gives save games for free.

### 9.7 Debugging

Purity pays for itself here.

- A recipe error leaves the node showing its **last good output** with an error badge. A broken recipe never breaks the world.
- Every evaluation unit can be **replayed in isolation** from its recorded inputs. "Why did this house come out wrong" becomes: replay that one unit with tracing on.
- Print output, debug shapes and timings are captured per unit and shown on the node.
- A step debugger for Luau is real integration work and can wait. Replay covers most of the need for recipes; gameplay scripts will want it sooner.

### 9.8 Access restrictions

| | Recipe | Gameplay |
|---|---|---|
| Files, network, process | None | None; saving through the save API |
| Time | None | Game time only |
| Other nodes or entities | Only what the context serves | Through the entity API |
| CPU and memory | Budgeted per unit; exceeded → error, last good output kept | Per-frame budget warning |
| Globals | Frozen | Per-module |

---

## 10. Scale and traversal

### 10.1 Scale

**Pass 2 conflated three things.** Separated:

| Concept | What it is | Discrete or continuous |
|---|---|---|
| **Capability profile** | What a body can do: sizes, jump distances, climbable tags | Any number of profiles; continuous values |
| **Detail band** | The smallest physical feature worth emitting as geometry | **Discrete**: a few bands, each roughly eight times finer than the last (for example: landmark, human, small, tiny) |
| **Scale context** | Runtime parameters: camera near plane and depth of field, gravity and time feel, audio pitch and reverb size, physics scaling | **Continuous**, and can be interpolated |

A cat (Stray) is a different *profile* at the human *band* with a larger detail budget near the camera. An insect is a different profile and a different band. Keeping these apart is the main correction.

**Are discrete bands sufficient?** For artifacts, yes, and I am fairly confident. A grow or shrink transition interpolates the scale context continuously while the artifacts switch band once, behind a transition effect. What bands cannot do is a smooth, uninterrupted zoom from human to insect with geometry refining all the way. That needs continuous level of detail across three orders of magnitude from every generator, and I would not attempt it.

**Is one band per world enough?** No, and Pass 2 implied it was. An insect-scale garden inside a human-scale house has both in view. So the detail band is an input **per evaluation unit**, chosen by the cook for each cell from where that band is needed. Artifacts at different bands are separate cache entries. Pack cell parts are keyed by (cell, band).

**What foundational data must know about scale**

| Must | Why |
|---|---|
| All lengths are metres; the document stores positions at double precision | One unit system |
| Generators declare features with **physical sizes**, and `realize` receives a detail band | The only way one generator serves two bands |
| The band is a context key, recorded in traces **only for units that read it** | Pass 2 put it in every cache key, which would have doubled all work for no reason |
| Profile is separate from band | Above |
| Pack parts keyed by (cell, band) | Mixed-band scenes |
| The runtime has one scale-context structure that camera, physics, audio and controller read | No hard-coded metrics |
| Camera-relative rendering | Precision at both extremes |

| Must not | Why |
|---|---|
| Ids | The same table is the same object at every band |
| The document's nodes and transforms | Scale is context, not content |
| Layout plans, normally | Where the houses are does not depend on who is looking. Exception: content that exists only at a fine band (individual grass blades as instances) is a separate scatter layer active only there |

**Uncertain:** physics at small scale. The Pass 2 idea (rescale the simulation so the body is about a metre tall) means one physics world per band and a hand-over during transitions. I think it is right and I have not seen it tested in this form. It is a late prototype and nothing foundational depends on it, since collider artifacts are in metres regardless.

### 10.2 Traversal: six separate things

The confusion to avoid is treating "the engine found a route" as one fact. It is six different kinds of statement with different sources and different reliability.

| # | Layer | The question | Source | Status |
|---|---|---|---|---|
| 1 | **Geometric possibility** | Could a body with these measurements physically make this move here? | A deriver over collision geometry | Objective but imperfect: it finds ugly moves nobody intended and misses some the controller can really do |
| 2 | **Generator-declared affordance** | Was this built to be used this way? | Surface features emitted with the geometry: ledge edges, ladders, handholds, each with a frame | Intent. Still **verified against (1)**: a declared ledge blocked by a crate is marked obstructed |
| 3 | **Authored markup** | What does the designer assert? | A human | Highest priority, and it works in both directions: *add* a link, or **forbid** one. Removing accidental routes matters as much as adding intended ones |
| 4 | **Capability requirement** | What does this move demand? | Measured from (1)–(3) | Each edge stores **measurements**: gap 2.1 m, rise 0.8 m, surface tag `pipe`. Not a yes or no |
| 5 | **Route generation** | Can the world be arranged so that a route of this kind exists from A to B? | A route node in the planning half | Produces valid and plausible routes. Can fail, and says so |
| 6 | **Design judgment** | Is it good? | A human | Not computed |

**A correction on (4).** Pass 2 built one filtered graph per profile. Storing the measurements on each edge and filtering at query time is better: one graph serves every profile, and a character who gains an ability mid-game needs no rebuild. Candidate edges are generated up to an **envelope profile**, the project's maximum capabilities.

**A safety margin.** A move at exactly the profile's limit succeeds some of the time. Route generation and validation plan at a fraction of capability (a tunable, say 85%), so that "valid" means reliably doable.

**How (5) fits the schedule of §5.** A route node plans its corridor at layer P2. It cannot send briefs to a cliff or a building, because they are not its children. So its requirements travel sideways like everything else:

- a `keep_clear` stamp along the corridor;
- **demand anchors**: tagged frames such as `traversal.need.ledge_chain` or `traversal.need.gap ≤ 2.5 m`, which structure and terrain generators may query and honour if they declare that they can.

After the derivers have built the graph, validation checks the route. Unmet demands are repaired by inserting a piece from the style's traversal kit (ladder, crate stack, rope, fallen trunk). Those pieces are generated children of the route node, keyed by demand, so they have stable identity like anything else. If repair fails, it is a validation error with a location.

Stress cases:

| Case | Outcome |
|---|---|
| A demand lands on frozen content | Cannot be honoured there. Repair kit, or an error suggesting a thaw |
| A demand conflicts with a user exclusion | Reported; not resolved (§5.4) |
| The layout changes | The corridor is keyed to its anchors and re-plans; repair pieces follow their demands |
| Movable objects are part of a route | The edge carries a runtime condition |
| The controller is changed and the profile no longer matches | The calibration run (Pass 2) rewrites the profile; routes are re-validated; failures are listed |

**On (6): metrics, not scores.** The engine will not judge fun and should not pretend to. What it can do is measure, and let the designer set limits:

route length and time at profile speed · sequence and mix of move types · longest stretch without a safe stop · greatest fall height · whether the goal is visible from the start and from each rest point · number of alternative routes · amount of backtracking.

A designer may turn any of these into a validator ("no fall over six metres on the main route"). The engine checks the rule. Whether the rule makes a good route is the designer's call.

---

## 11. Architecture risk register

Ranked by the product of impact and difficulty of changing later, since those are the risks that build corners. *Confidence* is my confidence in the current answer.

| # | Risk | Likelihood | Impact | Hard to change later | Confidence | Mitigation, and the early signal |
|---|---|---|---|---|---|---|
| 1 | **Key schemes in real generators are less stable than the model assumes.** The engine guarantees matching given good keys; good keys are each generator author's job. Grammars and graph-based layouts are the hard cases | Medium | Very high | Very hard: every existing world's edits depend on them | Medium | Identity prototype; the key-churn contract test. Signal: the inbox fills during ordinary tuning |
| 2 | **The edit boundary is correct but not learnable.** Users cannot predict whether a gesture becomes an input, an attachment or a detach | Medium-high | High | Hard: it is the document format | Medium | The editor announces every detach before doing it. Signal: you are surprised by your own tool in the first week |
| 3 | **Formats that are forever**: hash function, canonical serialization, key path syntax, lattice definition. All are baked into every id | Medium (that one is regretted) | High | Very hard | Medium-high | A format version in the document header; a lattice version per layer; migrate by full rewrite while there is one user. Decide them deliberately, once |
| 4 | **The recipe-facing kernel API churns.** Every recipe depends on it | Medium-high | High | Hard, and harder with each recipe | Medium | Few recipes until two deep generators have used it; API level and epoch. Signal: a third generator needs kernel changes that break the first two |
| 5 | **Incremental evaluation produces stale results.** A missed read means output that is silently wrong | High, at some point | High | Medium | Medium | The context is the only door, so a read cannot be missed by accident. A "paranoid" mode that rebuilds from scratch and compares, run routinely by tests |
| 6 | **The schedule is too rigid.** A case appears that truly needs feedback between layers | High that cases appear | Medium | Medium-hard: every generator assumes the order | Medium-high | The rule "declare it in planning or it cannot influence layout" (§5.1). Signal: a generator wanting to read a later layer's field |
| 7 | **The pack firewall leaks.** Gameplay starts needing key paths or generator concepts; or a runtime-generation game pulls the document into the runtime | Medium | High | Hard | Medium-high | Mechanical check on runtime headers; test games built without Generation; gameplay drives generation only through document commands |
| 8 | **Scope exceeds one developer.** The generation spine is novel and may need a second iteration before anything beautiful appears | High | High | Not applicable | — | Ordinary games first; toys before depth; explicit exit criteria. Decide beforehand what result would make you stop |
| 9 | **Wrong language** | Low-medium | High | Very hard | Medium | §7 |
| 10 | **Ordinary game development degrades as generation grows** | Medium | High | Medium | High | Three permanent test games; build-enforced separation |
| 11 | **Stable scatter looks worse than free scatter**, or spacing rules erode stability (§1.6) | Medium | Medium | Hard for existing worlds (lattice is forever) | Medium | Identity prototype extended with a visual check of scatter quality before any world depends on a lattice |
| 12 | **Schema-first generation grows into a reflection framework** | Medium | Medium | Hard | Medium | Boundary types only; a line budget for the generator tool. Signal: internal types being pushed through it |
| 13 | **The cheap indirect term does not reach the target look** | Medium | Medium-high | Medium: the seam exists | Medium | Look experiment before the renderer is built |
| 14 | **Cross-machine results differ** | High | Low-medium by design | Hard | Medium-high | Quantized existence; stored frozen content; two-machine comparison in tests |
| 15 | **AI-assisted changes erode module boundaries** | High | Medium-high | Medium | Medium | Boundaries enforced by the build, not by convention; task briefs scoped to modules |
| 16 | **Frozen artifact storage grows large** | Medium | Low-medium | Easy | Medium | Measure; fall back to storing only what cannot be reproduced |
| 17 | **Traversal profile and controller drift apart; planned routes feel unreliable** | Medium | Medium | Medium | Medium | Measured profile; safety margin |
| 18 | **A needed GPU feature is missing from SDL3 GPU** | Low-medium | Medium | Medium: thin layer | Medium | Early capability check (§8.3) |

**The corners.** Risks 1, 2, 3 and 4 are the ones where a wrong early choice is embedded in every piece of content made afterwards. All four are exercised by the identity prototype and the Luau probe, which is the argument for doing those before anything else. Risk 8 is the one most likely to end the project, and no architecture addresses it.

---

## 12. Updated decision table

Every open item from Pass 2 (§1.3 and §14), plus the ones this pass opened.

### Decide now

| Item | Decision |
|---|---|
| Scope | Single-player 3D, desktop and handheld PC |
| Four-part structure | Authoring, Build, Generation (optional), Runtime; meeting at the pack |
| State model | Source + mode (live, frozen, detached), shared by prefabs and generators; ids never change across modes |
| Edit model | Three classes with closed vocabularies; relative key-path addressing; the matching ladder |
| Provenance rule | Never an input; verified by deleting the manifest |
| Frozen content | Always stored in the project |
| Evaluation model | Units with verifying traces; writes declared, reads recorded; per-concern output hashes |
| Schedule | Planning half and realization half; field registry defines order; no upward channel; no iteration |
| Runtime firewall | Id, tags, semantic records, weights with promotion, chunk replacement. Nothing else |
| Terms | *Cook* and *freeze*; "bake" retired |
| Units and precision | Metres; double-precision document; camera-relative rendering |
| Detail band versus profile versus scale context | Three separate concepts |
| Traversal graph | Edges store measurements; filtered by profile at query time |
| Window and input | SDL3 |
| Renderer shape | Custom; forward with prepass; the twelve decisions of §8.1 |
| Graphics API | SDL3 GPU (C++ path) or wgpu (Rust path). Browser target dropped |
| Physics | Jolt on the C++ path |
| Gameplay scripting | Luau, in an environment separate from recipes |
| Source format | Strict JSON, sorted, one record per line where merges matter |
| Audio model | Engine-owned, event level |
| Asset interchange | glTF 2.0 |

### Prototype before deciding

| Item | Prototype | What decides |
|---|---|---|
| Language | §7, both parts | The rule in §7.6 |
| Key schemes and fingerprint tolerance | Identity prototype scenarios 4–8, 13, 14 | Inbox size under ordinary edits |
| Whether the edit boundary is learnable | Drive the prototype by hand for a day | Whether its choices surprise you |
| Stable scatter quality and spacing | Prototype plus a plotted image | Visible lattice pattern; churn under varying density |
| Recipes in Luau, and kernel API granularity | Luau probe, then two toy recipes | Kernel calls and time per realize; glue volume |
| Schema-first generation versus in-language derivation | Falls out of the language test | Boilerplate per boundary type |
| Indirect lighting technique | Look experiment E3 from Pass 2 | Side-by-side images |
| Per-tile freeze seams | First real terrain | Whether boundary blending is acceptable |
| SDL3 GPU capabilities | First renderer | §8.3 |
| Achievable cross-machine determinism | Same build on two machines | Whether caches can be shared |
| Frozen storage cost | First sizeable frozen region | Megabytes per region |
| Hold versus follow as defaults | Use in the lab | Annoyance |

### Defer safely

Each of these sits behind a boundary already drawn, so deciding late costs nothing.

| Item | Protected by |
|---|---|
| Entity store (flecs or a simple custom store) | Engine-owned script API. Start with either; review after the ordinary-game phase |
| Audio backend (miniaudio, possibly FMOD) | Event-level interface |
| Animation library | Engine-owned pose pipeline. Depends on language |
| Navigation library | Not needed until agents exist |
| Game UI library | The card game will decide |
| Shader language (Slang or plain HLSL) | Shading-model interface. Decide at the first renderer |
| Indirect lighting beyond the seam | §8.1 item 7 |
| Light clustering | §8.1 item 8 |
| Overhangs and caves | Try placed rock meshes on a heightfield first |
| Architecture generation method | First building generator |
| How far route-first placement goes | First urban block |
| Physics at small scale | Late prototype |
| Origin rebasing, streaming | Camera-relative rendering and cell-keyed packs are already in |
| Interiors, creatures | Nothing foundational depends on them |
| A simplified creator-facing editor | Parameter tiers are metadata already |

### Probably not needed

| Item | Why |
|---|---|
| Keeping old generator versions runnable | Frozen content is stored (§2.4) |
| WebGPU and a browser build | Out of scope |
| A separate graph per traversal profile | Measurements on edges |
| Observer scale in every cache key | Only readers record it |
| Inferring parameters from hand-edited geometry on reconnect | Adopt or discard (§2.6) |
| A mesh-patch edit operation | Detach instead |
| Iteration or events between generators | §5.5 |
| User-defined stages | Field registry |
| Queries over other generators' meshes | Anchors and surface features |
| Hand-declared dependency edges or provenance | Recorded automatically |
| Continuous scale | Bands plus an interpolated context |
| A data-driven or user-extensible render graph | Code is enough |
| A second schema system for artifacts (FlatBuffers and similar) | Schema-first generation covers it |
| Voxel terrain, a general constraint solver, a node-graph editor, a plugin interface | As in Pass 2; nothing in this pass revived them |
| Identity that anticipates networking | Derived ids already agree across machines; only runtime-spawned ids would need an authority |

---

## 13. What should be answered before implementation begins

In order. Each is small. Together they cover risks 1, 2, 3, 4, 9, 11 and 13 of the register.

| # | Question | How | Blocks |
|---|---|---|---|
| 1 | Does the identity, edit and freeze model hold up on the twenty-one scenarios? | §7 Part A, in whichever language is quicker to start; it is throwaway | Everything |
| 2 | Is the edit boundary predictable when *you* use it? | A day driving that prototype by hand through its command line | The document format |
| 3 | Do stable-scatter instances look natural, and how much does spacing disturb them? | A plotted image from the prototype at several densities | The lattice definition, which is permanent |
| 4 | Which language? | §7 in full, by the rule in §7.6 | All engine code |
| 5 | Is a recipe in Luau fast enough, and what does the kernel API need to look like? | Luau probe plus the `row` plan as a recipe | The kernel API |
| 6 | What does the target look require of indirect light? | Pass 2's look experiment: one diorama lit three ways | Renderer v0 priorities |
| 7 | Which hash, which canonical form, which key path syntax? | A one-page written decision after question 1, before any real project file exists | Every id ever created |

And three things only you can supply, carried over from Pass 2 and still open:

1. **Which of C++ and Rust you read comfortably.** If only one, question 4 is answered.
2. **A target machine.** "Steam Deck at 60" and "desktop GPU" give different answers in §8.
3. **Reference images for the first style.** Question 6 cannot be run without them.

### Where I am least sure in this pass

1. Whether detach as the only route to hand-edited geometry is too blunt. It is clean, and it may feel punishing for small tweaks.
2. Whether "emergent features cannot influence layout" will hold once real terrain produces things worth reacting to.
3. Whether the measured recipe budget survives a real facade grammar.
4. The split outcome I predict for the language test, and what to do with it.
5. Per-tile freezing next to live tiles. The semantics are clear; the visual result may not be acceptable.

