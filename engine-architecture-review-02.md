# 3D World-Building Engine — Technical Review, Pass 2

Status: proposal for review. Not an implementation plan.
Technology facts (licences, feature lists, maturity) come from my own knowledge and were not re-checked online in this pass. Reconfirm each one at adoption.

---

## 0. What changed since Pass 1

Pass 1 was right about the data model and wrong about the framing. Seven revisions:

1. **The centre is not a "world compiler". It is a scene document with provenance built in.** Every object is a node with a stable identity. Generation is one way a node gets children; hand authoring and prefab instancing are the other two. A game that uses no generators never meets the generation system. The compiler is a subsystem, not the identity.
2. **The runtime is not "modest".** It is modest in *rendering ambition* only. Animation, physics, audio, UI, save games and scripting must be genuinely good, or ordinary games become awkward. Phase 2 of the new roadmap exists to prove that before any serious generator is written.
3. **Runtime generation is not optional in the long run.** Two of your reference games (Tiny Glade, Dorfromantik) *are* runtime generation. The editor's live regeneration and a Tiny Glade-style game are the same capability. Generators must be shippable as an optional module.
4. **Generator logic should mostly be script, not native code.** Tuning a facade grammar needs sub-second iteration. Native kernel, scripted recipes. This also makes generator versioning automatic.
5. **Audio is not stubbed.** Events, buses, parameters and 3D emitters exist from the first phase, and "audio as generator output" exists from the first generator.
6. **"No global illumination" was too categorical.** Tiny Glade's look depends on soft bounced light. The renderer needs a pluggable indirect-lighting term, starting cheap.
7. **Urban is designed in from the first building generator**, not appended after terrain.

One new foundational idea: **binding time**. Every parameter is resolved at one of three moments: when geometry is generated, when it is instantiated, or every frame. "Summer to autumn" is cheap or expensive depending entirely on where `season` is bound. This is the key to §2's question.

---

## 1. Reconciliation with the original vision

### 1.1 Strongly accepted

I still agree with all eighteen items. They are one idea seen from different sides, so I group them.

| Group | Items | Why it still holds |
|---|---|---|
| **Source is never replaced** | specs as source of truth; derived-data caching instead of destructive baking; retained source when optimized; deterministic seeds | Every one of your example edits ("older", "jungle animals", "autumn") is a change to source followed by partial recomputation. That only works if outputs are disposable. |
| **Things keep their identity** | provenance; stable identity | Without it, regeneration destroys user edits and gameplay references. Everything else is useless if this fails. §3 treats it in depth. |
| **Recompute only what changed** | incremental deterministic compilation | This is what makes generation feel like editing instead of batch processing, and it is what makes runtime generation possible. |
| **Outputs mean something** | semantic outputs; traversal annotation and validation; environment fields and stamps | These let independent generators cooperate, and let gameplay, audio and AI consume generated worlds without hand markup. |
| **The engine is driven through one typed door** | typed specs; command-oriented editor; LLM outside through a controlled API; headless compile and test | One mechanism gives undo, automation, testing, AI access and the editor boundary. It is also the only way an AI agent can verify its own work. |
| **Scope discipline** | kits plus recipes; stylized rendering; modular rendering; lab scene; late creature generation | Each removes a class of work that does not serve the visual target or cannot be done by one developer. |

Nothing here was weakened by the second look. Two were strengthened: *typed specs* (now proposed as schema-first, §10.2) and *kits plus recipes* (now the basis of the style system, §7.4).

### 1.2 Modified or qualified

| Pass 1 position | Pass 2 position | Why |
|---|---|---|
| World compiler is the centre; runtime is a small backend | Three peers (runtime, authoring model, generation) sharing a semantic layer; generation is optional per project | Your objection is correct. A compiler-centred engine makes Tetris pay for machinery it does not use. |
| Own runtime, conditional on headless compiler | Own runtime, same condition, with more confidence | The runtime needed is small *because many things are instance sets, not entities* (§4.3), and because the renderer is specialized. |
| C++20, close call | C++20, **medium confidence**, with a specific experiment to settle it | The case for Rust is stronger than Pass 1 admitted for the parts that are your own code. §10.1. |
| SDL3 over GLFW | Unchanged, high confidence | Every reference game is gamepad-first. §10.3. |
| Luau for gameplay | Luau for gameplay **and for generator recipes** | Iteration speed and automatic versioning. §5.6. |
| flecs, do not wrap | flecs for engine systems, **engine-owned API for scripts**; low stakes | Entity counts are low here, so ECS performance is not the driver. §10.5. |
| Slang | Unchanged | Pluggable shading models are the point of the style system. |
| Jolt | Unchanged, high confidence | |
| Landscape + architecture first | Landscape first, then **buildings-on-lots designed to urban requirements**, with one settlement layout layer that scales from village to district | Makes urban a continuation, not a restart. §8.1. |
| "Does any shipped game need runtime generation?" | Yes, eventually. Three shipping modes: fully baked, realize-at-load, full runtime generation | See revision 3. |
| Editor: utilitarian ImGui, UX priority is direct manipulation | Same tool, but parameters carry *tier* metadata from day one so a simplified creator view is a later skin, not a rewrite | Cheap now, expensive to retrofit. §9.1. |
| Audio stubbed early | Real audio model from phase 1 | §8.7. |
| No GI, no lightmaps | No ray tracing, no lightmaps; **indirect light is a pluggable term**, starting with sky/ground ambient plus derived occlusion, with a probe volume as a planned upgrade | §7.2. |

### 1.3 Still undecided, and what would decide it

| Decision | Do not lock because | What resolves it |
|---|---|---|
| C++ vs Rust | Depends on which language *you* can review, and on measured AI defect rates in each | The Phase 0 bake-off (§12): the identity prototype written in both, by the same agent, scored on defects found and review effort |
| SDL3 GPU vs WebGPU | Depends on whether a browser build matters to you | Your answer, plus the phase 1 renderer. Keep the engine's GPU layer WebGPU-shaped either way |
| Generators in Luau vs native | Luau may be too slow for terrain-scale work | Write the toy tree and fence generators both ways in phase 3 and measure edit-to-pixels time |
| Indirect lighting technique | Unknown how much of the target look needs bounce light | A look spike: one hand-built diorama lit three ways (§12, experiment E3) |
| Key scheme for layout generators | The quality of identity depends on what each generator keys on | The twelve identity scenarios in §3.9, run as tests on toy generators |
| How far path-first traversal can go | Open research | Validator first; then one urban block where props are placed to satisfy a planned route |
| Depth of scale awareness in generators | Every generator must be written for it | One station: the same rug, floorboards and grass at human and insect scale |
| Old generator versions: keep runnable, or freeze artifacts? | Storage and maintenance cost unknown | Measure artifact size on the lab island once it exists |
| Schema-first code generation vs in-language reflection | Tied to the language choice | Falls out of the bake-off |
| Voxel or mesh terrain for caves and overhangs | Heightfields cannot do them; voxels are a large commitment | Try "heightfield plus placed rock/arch meshes" on the coastal cliff first |

---

## 2. Runtime architecture, and the bake question

### 2.1 Three peers and a semantic layer

```
   AUTHORING MODEL                 GENERATION                    RUNTIME
   (document)                      (optional module)             (ships in every game)
   nodes, identity,        ──▶     evaluator, generators,  ──▶   entities, instance sets,
   overrides, prefabs,             derivers, kernel              render, physics, anim,
   shapes, styles                        │                       audio, fields, UI, script
        │                                ▼                              ▲
        │                         artifacts + manifest                  │
        └───────────────────────────────┬───────────────────────────────┘
                                        ▼
                                PACK (cooked world)

   SEMANTICS (shared by all three): tag vocabulary, substances, capability
   profiles, component and parameter schemas
```

- **Runtime** loads *packs*. It knows nothing about nodes, generators or the editor.
- **Authoring model** is the document. For an ordinary game it holds only authored nodes and prefab instances. Cooking it into a pack is a near-trivial one-to-one mapping.
- **Generation** turns generator nodes into artifacts. It depends on the authoring model and the kernel, never on the runtime.
- **Semantics** is small shared data: what `climbable` means, what `wood` is, how far the player jumps.

The editor runs all three in one process and keeps the pack updated incrementally in memory, so editing feels live. A shipped baked game links only the runtime. A shipped Tiny Glade-like game links all three minus the editor.

The precedent for document-then-cook is Unity's entity baking. Its known weakness is that two representations confuse users. The mitigation is that for authored nodes the mapping is transparent: the components in the document are the same reflected types as the components at runtime.

### 2.2 Can a generated world be reduced to efficient runtime data and still be selectively regenerated?

**Yes**, provided five things exist in the data model. None is exotic; together they are a build system for worlds.

**1. A manifest that records what each node read and wrote.** For every evaluated node:

```
node id
generator id + version
resolved parameters (hash)
inputs actually read:  style keys, context keys, field regions, assets, child briefs
outputs written:       artifact hashes, stamps, child node ids
state:                 live | locked | detached
```

Dependencies are recorded at the granularity of *keys actually read*, not "depends on the style". A generator that never asked for `season` is not affected when `season` changes. The evaluation context enforces this by refusing undeclared reads.

**2. Artifacts split by concern.** A tree is not one blob. It is trunk mesh, foliage mesh, collider, wind weights, audio traits. If the foliage changes and the collider's bytes do not, the collider's hash is unchanged.

**3. Early cutoff.** A downstream step reruns only if an input's *content hash* changed, not merely because an upstream step reran. Unchanged collider hash means navmesh, traversal graph and physics are untouched. This is the single most important property for "avoid rebuilding unrelated systems".

**4. Optimization is a per-cell deriver.** Merging, instancing and LOD building take the artifacts in one spatial cell and produce that cell's pack data. One node changing rebuilds only the cells it touches. The pack is content-addressed, so a patch to a shipped game is the set of changed cells.

**5. Binding time for every parameter.**

| Binding | Resolved | Cost of change | Example |
|---|---|---|---|
| **Generate** | When geometry is made | Regenerate affected nodes | leaf drop, branch shape, building age |
| **Instantiate** | When a cell is loaded | Rebuild instance buffers | per-instance tint, variant choice |
| **Frame** | Every frame, in shaders and systems | Nothing | season colour blend, wind, wetness, snow cover |

### 2.3 Your forest example, worked

1. *Generate a forest.* The vegetation node reads the moisture, slope and occupancy fields in its region, the species definitions, and the style's foliage keys. It emits a variant pool (say 12 pine meshes, 8 birch), an instance table, colliders for trunks, and audio traits.
2. *Bake most of it.* Nothing happens to the source. Artifacts are on disk; the manifest records their hashes.
3. *Ship an optimized representation.* The per-cell optimizer builds instance buffers, LODs and impostors. The pack contains those and a table mapping instance to node id and semantic record (species, age). No specs, no generators.
4. *Change summer to autumn.* What happens depends on binding:
   - Leaf colour is frame-bound through a phenology ramp in the species definition. **Nothing regenerates.** A global parameter changes.
   - Leaf *density* for birch (thinning canopy) is instantiate-bound through a variant set per season state. Cell instance buffers for cells containing birch are rebuilt.
   - If you want fallen leaves on the ground, that is generate-bound for the ground-cover node, which reads `season`. It regenerates.
5. *Regenerate affected data.* The manifest is queried for "who read `season` at generate or instantiate binding". Pines did not. Terrain did not. Buildings did not.
6. *Unrelated systems.* Trunk colliders have unchanged hashes, so collision, navmesh and traversal are not rebuilt. The ambience deriver read `season` (birdsong differs), so it reruns. That is correct.

**Limits, stated plainly:**

- A shipped game without the generation module cannot regenerate on the player's machine. It can only switch between states you built. Frame-bound and instantiate-bound changes still work.
- Fine-grained dependency tracking only helps if generators are written to read narrowly. A generator that reads "the whole style" defeats it. This needs a lint.
- Locked nodes whose generator version no longer exists have artifacts that cannot be reproduced. Those artifacts are *precious* and must live in version control, not in a disposable cache (§9.4).

---

## 3. Stable identity

This section is the architecture, not a detail of it. If you disagree with anything in this pass, I would most like it to be here.

### 3.1 Principle

**Identity comes from decisions, not from order.** An object's identity must be derived from the reason it exists ("the house on the left of path segment 3, second from the start"), never from when it was created in a loop ("house 17").

A corollary: **user edits create anchors.** The moment you touch a generated thing, the engine records enough about it that it no longer depends on the generator re-deriving the same key.

### 3.2 Three kinds of identity

| Kind | For | How the id is formed | Survives |
|---|---|---|---|
| **Assigned** | Authored nodes, prefab assets, assets | Random 128-bit id at creation, stored in the file | Rename, reparent, move |
| **Derived** | Generated nodes | `hash(parent id, key)` where the *key* is chosen by the generator | Any regeneration that produces the same key under the same parent |
| **Slot** | Scattered instances | `(layer, cell, slot index)` on a fixed candidate lattice | Density, mask and species-mix changes |

Machines use the hash. Humans see the path: `island/village/house@seg3.L.2/facade@south/window@f1.b2`.

### 3.3 Keys: the generator's most important design choice

Stability, best to worst:

1. **Anchor keys.** The thing exists because a user or a parent placed an anchor. `lighthouse@A_lighthouse`.
2. **Topological keys.** Position in a structure that itself has stable ids. `house@seg3.L.2`, `window@f1.b2`.
3. **Spatial keys.** Quantized location. `rock@cell(14,-3).slot5`.
4. **Ordinal keys.** Index in an output list. Forbidden except under a parent whose children are truly interchangeable.

Each generator declares its key scheme in its descriptor, and a test checks that small parameter changes move few keys.

### 3.4 Stable scatter

Each cell of each scatter layer has a fixed set of candidate slots. Each slot has a fixed position inside the cell and a fixed random threshold, both derived from `hash(layer seed, cell, slot)`. A slot is occupied when its threshold is below the local density.

- Raising density fills more slots. Existing instances do not move.
- Painting a mask removes instances under the brush and nowhere else.
- Changing the species mix re-rolls species per slot from a separate named stream, so positions hold.

This is what makes "the tree I moved" a meaningful thing to remember.

### 3.5 Seeds

`seed(node, stream) = hash(parent seed, key, stream name)`. Streams are named by purpose: `layout`, `shape`, `colour`, `wobble`. Adding a random draw to `colour` cannot shift `layout`. A per-node **reroll** is a salt stored as an override, so one house can be re-rolled without touching its neighbours.

### 3.6 Overrides

An override is a record in a layer above the generated result:

```
target id            derived or slot id
target path          for humans
fingerprint          type, parent, position, footprint, archetype
operations           what the user did
made against         generator id + version
```

Operations differ in how they are applied, and that matters more than anything else in this section:

| Edit | Stored as | Applied | Robustness |
|---|---|---|---|
| Change a parameter (red roof) | Parameter override | **Before** generation, as an input to that child | High |
| Move or rotate | **Pin**: a fixed member the parent generator must respect | **Before** generation, as a constraint on the parent | High; does not depend on keys |
| Delete | Tombstone, with a spatial exclusion as fallback | Before, as an exclusion | High |
| Add a child (bench by the door) | Authored node whose parent reference is a derived id, plus a stored world position | After | Medium |
| Sculpt the mesh, or anything the generator cannot express | **Detach**: the output is snapshotted as authored content; the generator sees an occupied region | n/a | Total, at the cost of future regeneration for that node |
| Reroll | Seed salt | Before | High |

The rule is **edits flow down as inputs wherever possible, and are patched on afterwards only when they cannot.** This puts a requirement on every layout generator: it must accept pinned members and exclusions as constraints. That is a framework-level contract (§5.1).

### 3.7 Node state

| State | Meaning | Regenerates when |
|---|---|---|
| **Live** | Still being designed | Any input you changed changes its key |
| **Locked** | You approved how it looks | Only on explicit request, with a preview |
| **Detached** | No longer generated | Never |

One asymmetry is deliberate: **changes you make** (parameters, style, terrain edits) apply automatically to live nodes. **Changes to generator code** never apply automatically to anything. They mark nodes *stale* and wait.

### 3.8 Regeneration is a transaction with a diff

Any regeneration that could affect overrides runs into a staging area first and reports:

- nodes changed, added, removed
- overrides matched exactly / re-matched by fingerprint / orphaned
- downstream artifacts invalidated

You commit it as one undoable command, or discard it. Small live edits (dragging a spline) skip the report when nothing with an override is affected.

### 3.9 Worked example: the village

> A procedural village contains 40 houses. The user manually modifies House 17. The village generator changes. What happens?

Setup: the settlement generator keys houses by path segment, side and ordinal: `house@seg3.L.2`. On "House 17" you changed the roof colour, moved it two metres, deleted its chimney, and placed a bench beside it.

Stored: a parameter override (roof), a pin on the village (this house, this footprint, here, ground-relative), a tombstone on `chimney@0`, and an authored bench with a derived-id parent.

| What changes | What happens |
|---|---|
| **You** change house count 40 → 45 | The pin is an input. The layout places 44 others around it. Keys of unaffected segments are unchanged. All overrides match exactly. |
| **You** raise the terrain under the village | The village read the elevation field there, so it is dirty. Houses re-seat. The pin is ground-relative, so House 17 keeps its plan position and gets the new height. |
| Generator update, **keys compatible** | The village is marked stale. Nothing changes until you upgrade it. On upgrade: same as the first row. |
| Generator update, **key scheme changed** (lots now come from block subdivision) | Exact id matching fails for everything. Then: (1) the generator may supply a key migration from the old manifest; (2) otherwise the pin, being geometric, still forces a house at that place, and the override is re-targeted to whatever key that house now has; (3) unpinned overrides (a red door on House 23) are re-matched by fingerprint: nearest house of the same archetype within tolerance, accepted automatically only if unambiguous, and flagged "re-matched" for review; (4) the rest go to the orphan inbox. |
| The new House 17 has no chimney | The tombstone has nothing to delete. It stays dormant, not an error. |
| House 17's slot no longer exists at all | The bench keeps its stored world position and is flagged. The modified house is offered as a detached object at its old location. |

Two things never happen: an edit is silently lost, or an edit is silently applied to a different object beyond the matching threshold.

### 3.10 Worked example: the forest

> A forest generator changes its algorithm. How does the engine determine what can safely regenerate and what must preserve existing output?

Setup: a live forest with three hand-moved trees, one deleted tree, and one hero oak you sculpted.

At edit time, before any generator change, the engine already did this:

- Each moved tree became an **explicit pinned instance** on the forest node (species, variant, transform), and its original slot got a tombstone.
- The deleted tree became a tombstone *plus an exclusion disc* at its position.
- The hero oak was detached and now writes a clearing stamp like any authored obstacle.

Now the algorithm changes and the candidate lattice with it, so every slot id is meaningless.

1. The forest is marked **stale**. Its existing artifacts stay in use. Nothing regenerates.
2. You request the upgrade. It runs into staging.
3. Pinned instances are geometry, not slots, so they persist unchanged.
4. Slot tombstones fail to match; their exclusion discs still apply. The deleted tree stays deleted.
5. The oak is untouched.
6. The diff shows the new forest. Downstream, collision and navmesh rebuild only for cells whose collider hashes changed; the ambience deriver reruns if species composition per zone changed.

So the answer to "what is safe" is: **nothing is regenerated by a code change without your request; when you request it, edits that were captured as geometry or constraints survive regardless of algorithm, and edits that could only be captured as ids are re-matched or surfaced.** The design work is in making sure as many edit types as possible fall into the first group.

### 3.11 Other changes

| Change | Detection | Response |
|---|---|---|
| Style key edited | Content hash of that key | Live nodes that read it regenerate; locked nodes flagged stale |
| Source asset re-exported (a kit piece from Blender) | Content hash | Same; sockets that disappeared invalidate placements that used them, reported by validation |
| Upstream field changed in a region | Region hash | Nodes that read that region become dirty; early cutoff stops propagation where outputs are unchanged |
| Parameter renamed in a generator | Parameters have stable field ids, not just names | Overrides follow the id |
| Node type schema changed | Per-type version number | Migration functions run on load; unknown fields are preserved, never dropped |
| A save game refers to a generated object that no longer exists | Missing id at load | Same orphan policy at runtime: drop the delta and log it. Hence: lock layout before shipping |

### 3.12 What is still uncertain here

- Whether fingerprint re-matching is good enough in practice or produces annoying false matches. Needs the scenario tests.
- Whether "pin" generalizes to every generator. It is natural for layouts and scatter, less obvious for a facade grammar (what does pinning one window mean when the bay count changes?). My current answer is topological keys plus parameter overrides for grammar children, and no pins. To be tested.
- How much of this the user must understand. The aim is that it surfaces only as badges ("edited", "stale", "orphaned") and one inbox.

---

## 4. The World Model

### 4.1 One kind of thing: the node

A **node** has an id, a type, a parent, a transform, components, tags, and parameters. What distinguishes nodes is where their *children* come from:

| Children are | Node kind | Example |
|---|---|---|
| **Authored** | Entity or group | a door with a script, a light, a camera |
| **Instanced** from an asset | Prefab instance | a lamp post placed twelve times |
| **Generated** from parameters | Generator node | a village, a forest, a building |

Prefab instances and generator nodes are the same mechanism: children computed from a source, with an override layer on top. A **variant** is a prefab whose source is another prefab plus overrides. Build the override layer once; phase 1 uses it for prefabs, phase 3 reuses it for generators. This is how ordinary-game features and world-building features share one foundation instead of competing.

A few node types exist to be *inputs* to generators: **anchors** (points), **paths** (splines), **regions** (closed outlines or painted masks). They are authored like anything else and are the main art-direction handles.

### 4.2 How the concepts relate

```
PROJECT
├─ Assets (immutable inputs, each with id + content hash)
│    meshes · textures · sounds · scripts · kits · species · substances
│    styles · moods · prefabs · recipes (generator presets) · capability profiles
│
├─ Worlds ("scenes")
│    └─ Node tree
│         authored nodes ── components (same types as runtime)
│         prefab instances ── asset ref + overrides
│         generator nodes ── type + params + style ref + inputs + state
│         input shapes ── anchors, paths, regions
│         environment node ── mood, season, weather, wind
│         override layers
│
├─ Manifest (per world; what was built from what)        ← small, versioned
└─ Artifact store (content-addressed derived data)        ← mostly disposable cache
         meshes · instance tables · colliders · field tiles · nav tiles
         traversal graph · audio zones · probes · impostors
                    │
                    ▼  cook (per spatial cell)
PACK ── what the runtime loads
         entity lists · instance buffers · merged geometry · field tiles
         nav + traversal · audio zones · id-to-semantics tables
```

### 4.3 Three weights of generated thing

Not everything generated becomes an entity. This is why entity counts stay low and the runtime stays simple.

| Weight | Runtime form | Has | Example |
|---|---|---|---|
| **Entity** | Row in the entity store | Components, scripts, physics body | door, animal, lamp, lever |
| **Instance** | Row in an instance table, drawn in batches | Id, transform, semantic record, optional collider | tree, rock, crate, fence post |
| **Merged** | Triangle range in a cell mesh | Id recoverable through an id buffer and a range table | wall panels, roof, terrain |

An instance can be **promoted** to an entity when gameplay touches it: a tree that is cut down, a crate that is pushed. The id is the same before and after, so save games and scripts are indifferent.

Grass is a fourth, lighter case: a density field expanded on the GPU, with no per-blade identity.

### 4.4 Tags, fields, gameplay

- **Semantic tags** are hierarchical names from one project-wide registry (`surface.walkable`, `traversal.ledge`, `substance.wood`, `plant.tree.conifer.pine`). They attach to nodes, to instances, and to *regions of a surface* (a ledge is an edge on a mesh, not a whole object).
- **Environment fields** are world-level layered grids. Authored masks (a painted "no trees here") are assets referenced by nodes. Generator stamps are outputs. Compile-time fields (elevation, shelter, moisture) are artifacts; runtime fields (current snow depth) are runtime state that starts from them (§8.5).
- **Gameplay objects** are ordinary authored entity nodes. They can be parented to generated nodes by derived id ("the door script on `lighthouse/door@main`"), with the same orphan handling as any override.

### 4.5 Worked example

> "Create a small coastal village on a rocky island with a lighthouse, pine forest, grass, several houses, a path from the beach to the lighthouse, and a small dock."

Conceptual specification (not syntax):

```
world "Lab Island"     style: storybook_coastal   seed: 4117   observer: human
│
├─ environment         mood: overcast_dawn · season: late_summer · wind: onshore, moderate
│
├─ island  (terrain)   outline: R_island · character: rocky · peak: 60 m
├─ sea     (water)     level: 0
│
├─ A_landing     anchor on the beach            ┐
├─ A_lighthouse  anchor on the north headland   │ authored inputs:
├─ A_dock        anchor on the east shore       │ the art direction
├─ R_village     region, sheltered side         │
├─ R_forest      region, inland slope           ┘
│
├─ path     (route)       from A_landing, through R_village, to A_lighthouse
│                         class: footpath · passable by: profile "human"
├─ lighthouse (building)  at A_lighthouse · archetype: lighthouse · height: 18 m
├─ village  (settlement)  in R_village · houses: 6 · archetype: fisher_cottage · age: 0.4
├─ dock     (structure)   at A_dock · length: 12 m
├─ pines    (vegetation)  in R_forest · mix: pine 0.8, birch 0.2 · density: medium
├─ grass    (ground cover) wherever suitable
│
└─ gameplay (authored)    player_start · door script on lighthouse/door@main
```

What is generated from it, in stage order:

| Stage | Produces |
|---|---|
| **Claims** | Lighthouse, village and dock reserve footprints. The village plans six lots along the path corridor. Children appear: `village/house@seg2.L.0` … each a building node with a brief (lot, archetype, age). |
| **Terrain** | Elevation from outline and character; then stamps: flatten under footprints, cut the path, shape the beach. Fields: elevation, slope, moisture, shelter, occupancy. Terrain tiles with colliders. |
| **Water** | Sea surface, shoreline distance field, shore foam mask. |
| **Routes** | Path geometry, wear stamp, keep-clear stamp, steps where slope exceeds the profile's limit. |
| **Structures** | Building massing → facades → roofs through the style's realizers and kits. Entities for doors and lamps. Ledge, climbable and walkable tags. Dock planks and posts. |
| **Vegetation** | Pine and birch instances by suitability; clearings around occupancy; ground cover density field; rocks. |
| **Props** | Nets, crates, a boat at the dock, a bench. |
| **Derivers** | Collision, navmesh, traversal graph, validation that the path is passable for "human", occlusion and probes, impostors. Audio: surf along the shoreline curve, wind-in-pines zone over R_forest, village ambience, a reverb zone inside the lighthouse. |

Then "replace pine with palm, style to tropical" changes two parameters. Terrain shape, path, lots and the dock are untouched; vegetation, materials, kit pieces and ambience regenerate.

---

## 5. Generator framework

### 5.1 What every generator has in common

A generator is a **descriptor** plus one or two **pure functions**.

Descriptor (data, reflected, visible to the editor and the AI API):

| Field | Purpose |
|---|---|
| id, version | Cache key and staleness |
| parameters | Typed, with ranges, defaults, documentation, **tier** (basic / advanced / expert), **binding** (generate / instantiate / frame), optional **handle** (how the editor manipulates it directly) |
| input slots | Shapes, fields (with read margin), child brief, assets, style keys, context keys (observer scale, profile, season) |
| output slots | Child nodes, stamps (which field, which stage), artifacts, tags, entities, audio emitters |
| stage | Where it sits in the fixed order |
| key scheme | How its children are named (§3.3) |
| constraints accepted | Pins, exclusions |
| cost class | Interactive or background |

Functions:

- **Plan**: parameters and inputs → child nodes, claims and stamps. Cheap. Decides *what exists and where*. All identity is established here.
- **Realize**: a leaf node → geometry and other artifacts. Expensive, parallel, depends on observer scale and detail budget.

The split matters for four reasons. Identity never depends on mesh code. Layout can be previewed instantly with boxes. A game can ship plan results and realize at load. And it resolves the building/terrain cycle (§5.3).

### 5.2 How each concern is handled

| Concern | Mechanism |
|---|---|
| **Declaring inputs** | Slots in the descriptor. The evaluation context serves only declared reads and records exactly which keys and regions were read. |
| **Declaring outputs** | Slots in the descriptor. Undeclared output is an error. |
| **Parameters** | Schema drives inspector, command validation, AI discovery, and hashing. |
| **Provenance** | Automatic. The evaluator writes the manifest record; the mesh builder stamps each face with the node id. A generator author writes no provenance code. |
| **Dependencies** | Derived from recorded reads, not hand-declared edges. |
| **Determinism** | Randomness only through named streams from the context. No clock, no thread id, no unordered iteration, no fast-math. Checked by running each generator twice in tests and comparing hashes. |
| **Caching** | Key = hash(generator id + version, parameters, values of everything read, kernel epoch). Stored by content hash, with early cutoff downstream. |
| **Style** | Structural intent is handed to realizers supplied by the style (§7.4). Generators read construction constraints (max span, unit size) as style keys. |
| **Scale** | Observer scale and detail budget are context keys. Features are declared with physical sizes and emitted only if they would be visible (§8.3). |
| **Semantics** | Mesh builder calls take tags; surface features (ledges, handholds) are first-class outputs alongside triangles. |

### 5.3 How generators cooperate

I examined the six options you listed.

| Mechanism | Verdict | Role |
|---|---|---|
| **Explicit stage ordering** | Use | The coarse backbone. Makes evaluation order obvious and debuggable. |
| **Fields and stamps** | Use | Spatial negotiation: "ground is flat here", "something occupies this", "keep this clear". |
| **Parent-to-child briefs** | Use | Hierarchical delegation: the village tells each house its lot, archetype and age. |
| **Anchors, pins, constraints** | Use | How users and parents fix things. |
| **Semantic queries** | Use, restricted | Read-only, and only over *completed earlier stages* ("nearest road edge", "is this on the shoreline"). |
| **Shared mutable world data** | Reject | Destroys caching and determinism. |
| **Events between generators** | Reject | Order-dependent, undebuggable. |
| **Iteration to a fixed point** | Reject | Unbounded cost; small changes ripple everywhere. |

**Recommended: stages for order, fields for space, briefs for hierarchy, constraints for intent, and read-only queries backward in stage order.** Generators never call each other and never know each other's types.

The one real circular dependency (buildings want flat ground; terrain wants to know where buildings are) is handled by the plan/realize split: structures *claim* footprints in an early stage, terrain is finalized with those claims as stamps, and structures are *realized* later on the final ground. No feedback, two passes.

Fixed stages: **claims → terrain → water → routes → structures → vegetation → props → derivers.** Vegetation has internal sub-stages (canopy before understory before ground cover) using the same field mechanism.

### 5.4 The World Builder

It is the generator at the top of the tree and nothing more. Its plan function partitions space into regions, creates the child generator nodes with briefs, and places shared anchors. It has no special access. A human can do its job by hand, which is exactly what the coastal example above is.

### 5.5 Guarding against ten products

- A generator is not allowed its own data structures for things the kernel has (fields, splines, mesh builder, scatter).
- A generator is not allowed its own editor panel. Its UI is generated from the descriptor plus handles.
- Depth is allowed in exactly two generators at a time. Everything else is a "toy" until promoted on purpose.

### 5.6 Where generator code lives

| Tier | Written in | Contains | Iteration |
|---|---|---|---|
| **Kernel** | Native | Mesh, fields, shapes, scatter, noise, hashing | Recompile |
| **Recipes** | Luau, hot-reloaded | Plan logic, grammar rules, realizer assembly | Save file, see result |
| **Specs and presets** | Data | Parameters, styles, species, kits | Instant |

Reasons: tuning needs sub-second turnaround; a script generator's version is simply the hash of its source, so cache invalidation cannot be forgotten; the same VM ships in games that need runtime generation; and the sandbox enforces "no undeclared reads" for free.

Exceptions written natively: terrain synthesis, erosion, scatter evaluation, anything per-vertex over large areas. Native generators need a manual version number, and golden-hash tests to catch an output change without a version bump.

This is medium confidence. The risk is script performance on mid-sized work such as facade assembly. The phase 3 toys measure it.

---

## 6. The generation kernel

Rule for the whole section: **promote on second use.** An algorithm lives inside the generator that needs it until a second generator needs it too.

### Core kernel — probably required

| Item | Why it is core |
|---|---|
| **Hashing and named random streams** | Identity and determinism depend on it everywhere |
| **Fields**: tiled 2D grids, sampling, stamp compositing, masks | The cooperation mechanism between generators |
| **2D shapes**: polylines, splines, polygons, offset, clip, union | Paths, lots, footprints, regions. Polygon clipping from a library (Clipper2) |
| **Stable scatter** on the candidate lattice | Used by vegetation, rocks, props, decals |
| **Noise** | Terrain, variation, wobble |
| **Mesh builder**: indexed mesh with attributes, per-face node id and tags, surface features, merge, transform, extrude, sweep along path, loft, revolve, bevel | Every realizer uses it; it is where provenance and semantics get attached automatically |
| **Instance tables** with semantic records | The "instance" weight in §4.3 |
| **Frames and scopes**: split, repeat, subdivide along an axis | The five or six operations that make split grammars. Needed by fences, facades, docks, stairs |
| **Kit placement**: sockets, snapping, fitting a piece to a slot | The kits-plus-recipes model |
| **Artifact types** with canonical serialization | Caching |

This is deliberately small: perhaps a dozen headers. Each item has at least two certain users in the first two deep generators.

### Useful later

| Item | Arrives with |
|---|---|
| 3D mesh booleans (Manifold) | Buildings with cut openings, ruins |
| Implicit surfaces and meshing | Rocks, organic forms, later creatures |
| Mesh simplification (meshoptimizer) and impostor baking | LOD at scale |
| Hydraulic erosion | Terrain quality pass |
| Graph utilities: planar graphs, shortest path, face extraction | Street networks, blocks |
| Straight skeleton | Roofs on arbitrary footprints. Notoriously fiddly; start with gable, hip, shed and flat on rectangular, L and T footprints |
| Branching structures: space colonization, L-systems | Lives in the tree generator until a second user appears (roots, coral, cables) |
| Local constraint solving, wave function collapse | Interior layouts, tile sets |
| UV unwrapping (xatlas) | Only if unique textures are ever needed; the default look avoids it (§7.3) |

### Probably unnecessary initially

| Item | Why not |
|---|---|
| A visual node-graph language for generators | That is building Houdini. Recipes in script, parameters in the inspector |
| A general constraint solver | Every early need is solved by a specific small algorithm |
| Volumetric or voxel terrain | Large commitment; try placed rock and arch meshes for overhangs first |
| A half-edge mesh structure | Builder-style construction rarely needs topology queries |
| 3D fields | 2D layered fields cover everything until true caves or volumetric clouds |
| A generic "procedural asset" plugin interface | No second party to serve yet |

---

## 7. Style, rendering and world materialization

### 7.1 What the reference games have in common, technically

| Trait | Seen in | What it asks of the engine |
|---|---|---|
| Colour carried by light and atmosphere over simple albedo | All | Lighting, fog and grading quality matter more than material complexity |
| Soft, bounced, coloured ambient light | Tiny Glade, Arise, Lana | An indirect term better than flat ambient |
| Heavy depth cueing: fog, haze, layered silhouettes | Lana, Sea of Solitude, Juggler's Tale, Submerged | Fog and sky as first-class, art-directed |
| Shallow depth of field, tilt-shift, vignette | Midnight Walk, Juggler's Tale, Dorfromantik, Tiny Glade | High-quality depth of field; miniature cues (also needed for scale, §8.3) |
| Stylized water as a major surface | Sea of Solitude, Submerged, Arise, Short Hike | A real water system, not a shader on a plane |
| Vegetation as shaped masses with wind | Short Hike, Tiny Glade, Lana | Instancing, custom normals, wind |
| Flat or gradient surfaces, little texture detail | Short Hike, Dorfromantik, Tiny Glade | Vertex colour and procedural shading; often no UVs |
| Deliberate low resolution or stepped animation | Short Hike, Midnight Walk | Render-scale and animation-stepping as style options (both nearly free) |
| Many small emissive lights, wet reflective ground | **Stray** | Clustered lights, emissive bloom, cheap reflections. Stray is the outlier and the reason forward shading must be clustered |

Two honest notes. Tiny Glade's softness comes from a custom real-time global illumination renderer, which is not cheap technology. And Stray is close to realistic rendering with stylized art direction. The engine should reach "Tiny Glade softness" and "Stray at night" by cheaper means, and that is the main rendering uncertainty.

### 7.2 Renderer specialized to this direction

**Structure**

- **Clustered forward shading** with a thin prepass (depth, normals, node id). Forward makes per-material shading models trivial; clustering gives Stray-style light counts; the prepass feeds outlines, occlusion and fog. MSAA with alpha-to-coverage for clean silhouettes and foliage. No dependence on temporal anti-aliasing, which smears stylized edges.
- **One render graph**; a style selects shading models and a post stack.
- **Shading model as a shader interface**: surface description in, light response out. First models: ramp-lit (gradient-controlled diffuse, shaped highlight, rim, tinted shadow), soft-lit (wrapped diffuse for the Tiny Glade feel), foliage, water, skin/cloth later.

**Lighting**

- Sun with stable cascaded shadows, soft filtering, **art-directed shadow colour** (stylized shadows are rarely black).
- Clustered local lights, a few with shadows.
- **Indirect light as a replaceable term.** Step 1: sky/ground gradient ambient × occlusion (per-vertex occlusion from a deriver, plus screen-space). Step 2, planned: a sparse probe volume, itself a derived artifact updated per cell when the world or mood changes. No ray tracing, no lightmaps.
- Contact shadows; blob shadows for small things.

**Outlines**, three sources, mixable:

1. Screen-space from depth, normal and id discontinuities. Default.
2. Inverted hull for characters.
3. **Generator-declared feature edges.** A generator knows which edges are true creases and silhouettes, so outlines can be placed deliberately instead of detected. This is an advantage a general engine does not have.

**Atmosphere** (where the budget goes)

- Art-directed gradient sky with sun, moon, stars and a cloud layer; a physically inspired model as an option, not the default.
- Height fog and distance haze from day one; volumetric fog and light shafts later.
- Colour grading per mood; a tonemapper chosen for colour richness (film-style curves that desaturate brights are the wrong default here).
- Bloom, depth of field, vignette, optional pixelation, optional painterly filter later.

**Water**: one system for sea, lakes and rivers. Surfaces, shoreline distance, flow maps and foam masks are *compiler outputs*. Summed waves, depth-ramp colour, refraction from a scene copy, sky plus cheap screen-space or planar reflections.

**Vegetation**: GPU-culled instances, dithered LOD cross-fade, impostors from a deriver, spherical or generator-supplied normals, hierarchical wind using per-vertex weights the tree generator emits exactly (it knows the branch hierarchy), bend from the trample field.

**Deliberately absent**: ray-traced anything, virtualized geometry, virtual texturing, temporal upscaling, photogrammetry support, a deferred path.

**Needed from you**: a target machine. "Steam Deck at 60 frames per second" versus "desktop GPU" changes several choices above.

### 7.3 A simplification worth taking

Generated geometry defaults to **vertex colour, world-space or triplanar procedural detail, and small tiling detail textures**. No unique UV unwrap, no unique texture bake. This matches most of the references and removes a slow, fragile step from every generator. Kit pieces authored in Blender may carry UVs and textures as usual.

### 7.4 World materialization

Your idea, restated: the world's substance changes construction, not just texture. Pass 1's four layers stand (substance, construction, surface, palette and form). What makes it buildable is three restrictions.

**1. A small structural vocabulary.** Generators never emit "plank" or "brick". They emit about a dozen structural primitives:

`panel` (wall, floor) · `beam/post` · `mass` (solid block) · `shell` (roof skin) · `opening frame` · `steps` · `run` (fence, rail) · `trunk/branch` · `foliage mass` · `ground surface` · `rock mass` · `liquid surface`

**2. Substances are parameterizations of a few realizer families**, not unique code each.

| Family | How it builds | Parameters | Covers |
|---|---|---|---|
| **Unit-stacked** | Repeated units with joints | unit size, bond pattern, joint type, gap, irregularity, edge wear | wood planks, brick, cut stone, ice blocks, sandstone |
| **Monolithic-carved** | One mass with treated edges | bevel, chipping, melt, slump angle, strata | rough stone, carved ice, packed sand, clay |
| **Soft-accreted** | Blobby implicit forms | blob size, softness, drip | snow, cloud, moss, foam |
| **Sheet** | Thin surfaces with folds | thickness, crease, drape | cloth, paper, leaf, metal sheet |

A wooden world is unit-stacked planks with pegged joints. An ice world is monolithic with melt-rounded edges and unit-stacked blocks for walls. A sand world is monolithic with slump. A cloud world is soft-accreted with its own shader. Four families times parameters is a tractable amount of code.

**3. Style reaches back into structure only through a short list of numeric constraints**, not arbitrary code: maximum span (wood inserts posts every 4 m; stone uses arches), minimum thickness, allowed roof forms, allowed opening shapes, wobble. Generators read these as style keys during planning.

Beyond geometry, a substance also supplies: surface shading, debris and dust particles, footstep and impact sounds, and its row in the reaction table (burns, melts, floats). That is how "wooden world" reaches particles, audio and gameplay without any of those systems knowing about styles.

A style then is: a default mapping from structural role to substance (walls wood, foundations stone), parameters for each, kits, palette, shading models, a mood set. Individual nodes can override the substance.

Authored kit pieces declare the structural role they fill. If a style lacks a piece, the parametric realizer is the fallback, and validation reports "unstyled fallback used here".

**Cost, honestly.** The framework is perhaps a few weeks. Each *excellent* style is content work measured in weeks to months, and that never goes away. Styles are things you make *with* the engine. I would plan for two in the first year, and treat "cloud world" as the hard test because it needs soft forms and different rendering.

---

## 8. Domain systems

### 8.1 Urban and architecture

**Method: a hybrid, with a different technique at each level.** No single technique works across the whole hierarchy.

| Level | Technique | Output |
|---|---|---|
| District → streets | Graph growth along fields and anchors | Street graph with classes (avenue, street, alley) |
| Streets → blocks | Face extraction from the graph, polygon offset for sidewalks | Block polygons |
| Blocks → lots | Recursive subdivision with frontage rules | Lot polygons with a street-facing edge |
| Lot → massing | Parametric volumes: extrude, setback, courtyard, roof form | Building volume, floors |
| Massing → facade | **Split grammar** producing *slots* (bay, window, door, shopfront, balcony position) | Slots with semantic roles |
| Slots → geometry | **Kit pieces or parametric realizers** from the style | Meshes, entities |
| Exterior infrastructure | **Rule-based attachment and routing on the facade's slot grid**: pipes run from roof to ground by path search; cables span between anchor points; fire escapes link window slots floor to floor | Pipes, vents, units, signs, cables, ladders |
| Weathering | Fields (wear, damp, grime) driving vertex colour, decals, puddles, growth | The "age" and "run-down" parameters |
| Traversal | Tags emitted by every step above | Ledges, rooftops, climbable pipes, gaps |

Constraint solving is used only locally (a fire escape must connect; a pipe must reach the ground). Wave function collapse is not the backbone; it produces noise-like results at city scale and is hard to art-direct.

**Why this gives urban equal status early:** a village and a city block are the same pipeline with a different layout generator at the top. The *building* generator is written once, to urban requirements, in the first architecture phase: party walls, multiple storeys, roof infrastructure zones, exterior attachment slots, traversal tags. The first content is a cottage and a lighthouse; the first design is for an apartment block.

**Stray-style traversal is more tractable in cities than in nature.** The facade is already a discrete grid of slots. A route can be planned on that grid first ("window ledge → air-conditioning unit → pipe → balcony → roof") and props then *placed to realize each step* within the character's metrics. The clutter that makes the alley feel real and the stepping stones that make it playable are the same objects.

**Interiors.** The hierarchy includes floors and rooms as data, but generating furnished interiors for every building is impractical and mostly unseen. Better interpretation: interiors exist only for buildings marked enterable, generated on demand from a room-layout step and interior kits, with windows elsewhere using a cheap fake-interior shader. Deferred beyond the first urban phase.

### 8.2 Traversal

**Capability profile** (data, in Semantics): body dimensions; walk (max slope, step height); jump (up, across, down); climb (which tags, reach); mantle and vault ranges; crawl height; squeeze width; swim; slide angle range; rope and zip use.

The profile must not drift from the real character controller. So it is *measured*: a headless calibration run drives the controller through test courses and writes the profile. One source of truth.

**Where traversal information comes from**

1. **Generator-declared features.** Ledges as tagged edges, ladders, handhold sets, rope anchors, slide surfaces. Exact and free, because the generator knows what it built.
2. **A geometric deriver** over collision geometry: walkable regions from the navmesh, candidate ledges from edge analysis, jump and drop links by sampling region boundaries against the profile. Catches what generators did not declare, and all authored geometry.
3. **Authored markup** for anything else.

**Traversal graph** (a derived artifact, tiled per cell, one filtered view per profile): nodes are walkable regions and features; edges are moves, each with the capability it requires, a cost, and entry and exit frames. The navmesh supplies the walkable part; the graph is the superset that includes climbing, jumping, sliding and ropes. AI agents use the same graph. Edges can carry runtime conditions (a door, a lowered bridge).

**Queries and validation:** reachability from A to B for a profile; reachable-area map; regions you can enter but not leave; which ability gates which region.

**Route constraints** ("a climbable route from the beach to the lighthouse") are a spec node: from, to, profile, allowed moves, difficulty. Processing:

1. *Corridor planning* in the routes stage, over terrain and claimed massing.
2. The corridor emits *requirements* as stamps and briefs: "a ledge chain along this cliff segment", "gap no wider than 2.5 m here", keep-clear volumes so vegetation and props do not block it.
3. Downstream generators honour them through their normal inputs.
4. The validator proves the route on the final graph.
5. On failure: a report, plus limited repair by inserting a piece from the style's *traversal kit* (ladder, crate stack, rope, fallen trunk).

No global backtracking solver. This yields routes that are **valid and plausible**. It does not yield routes that are **fun**; pacing, risk and reveal remain human work, done by placing anchors and tuning. I would not promise more than that.

At runtime the controller uses the declared features (grab *this* ledge) instead of probing raw geometry, which is more reliable than ray-tracing for ledges.

### 8.3 Scale

Scale is a **context**: world scale, observer scale, and a capability profile. It is not a transform on the player.

| Area | Mechanism |
|---|---|
| **Physics, collision** | Simulation is rescaled so the player stays roughly 1–2 units tall, the range physics engines are tuned for. Gravity and time scale are chosen for feel per context. |
| **Camera, perception** | Near plane, field of view, and **depth of field**: shallow focus is the strongest "this is tiny" cue, wide deep focus the strongest "this is vast". Fog distance, dust mote size. |
| **Traversal** | A different profile over the same geometry yields a different graph. A table edge is a cliff for one profile and a step for another. |
| **Procedural detail** | Generators declare features with *physical sizes* (wood grain 2 mm, plank gap 5 mm, carpet fibre 1 mm). A feature becomes geometry when it would be large on screen for the current observer, shading detail when smaller, nothing when smaller still. The same rug is a flat plane for a human and a field of stalks for an insect. |
| **Vegetation, props** | Weight shifts by scale: grass goes from GPU density field to collidable instances; a tree goes from instance to terrain-like landmark. |
| **Sound** | Pitch, reverb size, distance attenuation, footstep weight follow the context. |
| **Interaction** | What can be pushed, broken or climbed comes from profile strength against substance and size. |
| **Gameplay metrics** | Read from the profile; never hard-coded. |

**Foundational now** (cheap early, very expensive later):

- Camera-relative rendering.
- Positions in the document stored at high precision; no absolute world position in any shader.
- Observer scale and profile present in every generator context and in the cache key, even while constant.
- World scale as a property of the world.

**Not foundational:** the origin-rebasing implementation itself (design for it, build it with streaming), and grow/shrink transitions.

**Limits.** Only generators written to be scale-aware span scales, so a convincing insect-scale kitchen needs content made for that purpose. And growing or shrinking continuously across orders of magnitude is impractical; **discrete bands** (two or three realized scales with a transition effect) preserve the idea at a fraction of the cost.

### 8.4 Nature and ecology

**A species is an asset**, not a mesh:

| Part | Content |
|---|---|
| Taxonomy tags | `plant.tree.conifer.pine` |
| Growth recipe | Form as a function of age, health, exposure |
| Niche | Preferred ranges of elevation, slope, moisture, light; spacing; companions (mushrooms near birch, ferns in shade) |
| Phenology | Leaf colour ramp, leaf density, flowering and fruiting windows across the year |
| Reaction traits | Wind stiffness, flammability, snow load |
| Substances | Wood, leaf |
| Audio traits | Rustle character, what lives in it |
| Gameplay tags | Harvestable, climbable, cover |

A **community** (biome) is a weighted set of species in layers: canopy, understory, ground.

**How "pine, young, healthy, autumn, medium density" becomes pixels**

- Per species and age class the generator builds a **variant pool** (a dozen or so meshes), not a unique mesh per tree. Hero trees may be unique.
- Each instance stores variant, transform, tint and a **semantic record**: species, age, health. Queries such as "all young pines near water" work in the editor, in gameplay and through the AI API.
- Placement is field evaluation: suitability per species from the fields, canopy first, which writes shade and litter stamps that the understory reads.
- **Density** is a field parameter; with stable scatter it changes which slots are filled without moving anything.
- **Season** is bound as in §2.3: colour per frame, density per instantiation, ground litter per generation.

| Interaction | How |
|---|---|
| Instancing, LOD | Variant pools; impostors by deriver; cross-fade |
| Wind | Hierarchical bend from generator-supplied per-vertex weights, sampling the wind field |
| Snow | Shader accumulation on upward faces from the snow field; optional laden variants |
| Fire | Flammability from substance; instance promoted to entity when it ignites; spread through the heat field |
| Style | Foliage mass and trunk are structural primitives handed to realizers; a wooden world can have carved wooden trees |
| Scale | §8.3 |

Rocks: generator plus variant pool. Rivers: a spline that writes a carve stamp and a moisture stamp, with flow as a derived map. Ponds: fill of a terrain depression. Grass: density field expanded on the GPU.

Ecology here means **layered suitability**, not simulation of growth and competition over time. Simulated succession is a possible later generator and does not change the architecture.

### 8.5 Reactive environment

**Structure: a field service plus small independent phenomenon modules.**

The **field service** is a registry. Each field has a name, a schema (resolution, extent: camera-centred, per-cell or global; update rate; decay), writers and readers. It offers GPU binding for shaders and a low-resolution CPU query for gameplay and audio. It contains no phenomenon logic.

A **phenomenon module** is: the fields it owns, their update rule, and its injectors.

| Module | Owns | Written by | Read by |
|---|---|---|---|
| Wind | wind vector | weather state, gust noise, local injectors (fan, explosion, passing creature) | vegetation, cloth, particles, audio, fire |
| Precipitation and wetness | wetness | weather × **shelter mask** | materials, puddles, footsteps, audio, fire |
| Snow | snow depth | weather × exposure − trample − heat | materials, roofs, vegetation, footprints, movement |
| Trample | deformation | characters, objects | grass, snow, mud |
| Heat and fire | heat, burning set | fire sources, spread rule | lighting, particles, materials, vegetation, audio, snow |
| Fog | density | weather, water, terrain hollows | renderer, audio |

How it stays modular:

- **No phenomenon module includes another.** Snow does not call fire; it reads the heat field.
- **Cross-effects are data**: one substance × element table (wood + fire → burning; snow + heat → melt → wetness).
- **Every reader has a default** when a field is absent, so any module can be removed.
- **Static and dynamic are separate.** The compiler produces capacity and exposure layers (shelter mask from a top-down pass, drainage, flammable mass). The runtime holds state. Reactive systems therefore work on generated worlds with no markup, and on authored worlds through the same derivers.

Start with wind (analytic, global plus a few injectors, no texture), wetness (a global value times the shelter mask) and trample (one camera-centred render target). Those three already make a world feel responsive. Fire as a spreading simulation is the most expensive and comes last.

### 8.6 Creatures

Procedural creatures stay last. What must exist first, each useful on its own:

| Prerequisite | Also serves |
|---|---|
| **Rig description**: an asset mapping bones to semantic roles (spine chain, front-left leg chain, head, tail chain, wing chain) and sockets | Imported animals and the player character |
| **Pose pipeline** where clips are one source among several: IK, look-at, spring chains, additive layers | Foot placement on generated terrain, capes, tails |
| **Locomotion from limb roles and profile**, at first only to adapt imported animations to slopes | All animals |
| **Part state machines and hooks**: `wings: folded → deploying → deployed`, exposed as parameters, tags and events | Any animated gameplay object |
| **Body-plan schema**: segments, limb attachments, counts, proportions | It is only a node type |
| **Implicit surface mesher** in the kernel | Rocks, organic props |

**Imported creatures fit now.** A `creature` node references a rigged glTF, a rig description, abilities and a behaviour script. It exposes parts, sockets and hooks. Ecology, AI and gameplay are written against that interface.

**Procedural creatures later** are another implementation of the same node: body plan → skeleton → implicit surface mesh with skinning known by construction → rig description generated instead of authored → procedural gaits per limb role, spring-driven tentacles and tails, pose-state wings.

Your example sentence maps onto the body-plan schema directly (squat quadruped; head with four eye sockets; six tentacle chains around the mouth socket; two wing chains with an airborne-triggered state; rear feet with talons). An external LLM produces that structure; the engine never parses the sentence.

Expect motion that is charming and plausible, not hand-animated quality. No clip data exists for a novel body, and procedural gaits have a recognizable look.

### 8.7 Audio

Revised from Pass 1: not a stub.

**Engine-owned model, at event level:**

| Concept | Role |
|---|---|
| **Event** | A playable definition: layers, randomization, parameter curves, looping. Authored in the editor as an asset |
| **Emitter** | An event instance at a point, **along a curve**, or over an area |
| **Parameter** | Named value, global or per emitter (`wind_strength`, `time_of_day`, `wetness`, `speed`) |
| **Bus** | Mix groups, effects, sends, ducking |
| **Snapshot** | A mix state blended by mood or gameplay |
| **Acoustic zone** | Reverb character and openness for a volume |
| **Listener context** | Position, scale context, zone blend |

The backend interface is at this level, not at voice level, so the built-in backend (miniaudio) could later be replaced by FMOD without changing anything above it.

**Soundscape derivers** make generated worlds audible with no hand placement:

- **Ambience zones** from region composition: tree density → rustle intensity, species → bird set, distance to water → water layer.
- **Curve emitters**: surf along the shoreline, a river along its spline. One voice follows the listener's nearest point.
- **Scattered one-shots**: bird calls from random tree instances, creaks from wooden structures, drawing on the instance tables.
- **Acoustic zones**: interiors from the building generator; outdoors from an openness measure sampled at compile time.
- **Substance sounds**: footsteps, impacts, scrapes.
- **Runtime drive**: weather, wind field, time of day and wetness feed parameters.

Reverb starts as zone presets with blending, and occlusion as a ray test driving a filter. Propagation-based acoustics (Steam Audio) can replace both behind the same zone concept.

---

## 9. Workflow systems

### 9.1 Editor

**Principles**

1. Every action is a command. The console shows the command each click produced, which documents the API and gives an AI agent examples.
2. Direct manipulation first. Generators declare **handles** in their descriptors (this parameter is a spline, that one a height), so the viewport edits parameters without bespoke tools.
3. Progressive regeneration: an instant box-level preview from the plan phase, then realized geometry, refined in the background, cancellable.
4. Editing a generated thing just works and leaves a visible badge. The user never has to "convert" first.

**Views**

| View | Purpose |
|---|---|
| Viewport with overlays | Scene; fields; ids; traversal; physics; scale switch ("view as profile X") |
| Outliner | Node tree; authored, instanced and generated shown differently; generated children expand lazily |
| Inspector | Parameters by tier; override markers with revert; node state |
| **Why?** panel | Click any pixel → node → generator and version → resolved parameters, *each labelled with its origin* (default, preset, parent brief, style, override) → seed → inputs read → artifacts. Actions: reroll, pin, lock, detach, go to parent, show dependents |
| Regeneration | Dirty set, staged diff, timings, **orphan inbox**, stale nodes |
| Asset browser | Thumbnails rendered by the engine's own renderer in headless mode; **semantic types** (mesh, kit, species, substance, style, mood, recipe, prefab, sound event, profile); filter by tag; drag into the world to create the right node |
| Style editor | The four layers; live on the lab scene |
| Traversal debugger | Choose a profile; graph; reach map; click two points for a route; dead ends |
| Mood and time | Scrub time of day, season, weather |
| Validation | One list of problems from all validators |
| Play | Run the game in the editor on a copy of the world |

**Expert versus general user.** Three levels of exposure:

1. **Compose**: place presets, use handles, adjust the few *basic* parameters, pick styles and moods.
2. **Tune**: all parameters, overrides, style editing, provenance.
3. **Author**: write recipes, define species, build kits and styles.

Build level 2 for yourself, in Dear ImGui. Level 1 is a later presentation of the same descriptors, which is why parameter tiers go in from the start. Level 3 is files, scripts and an AI assistant. I would not build a polished non-programmer interface until someone other than you needs it.

### 9.2 AI and automation API

One protocol: structured messages over a local socket, also exposed as a command-line tool and wrapped as an MCP server so that assistants such as Claude can use it directly. The editor UI is a client of the same protocol.

| Group | Operations |
|---|---|
| **Describe** | List node types, generators and their parameter schemas with documentation; tag vocabulary; style keys; search the asset catalogue by semantic type and tag |
| **Query** | Tree and node contents; spatial queries; semantic queries over instance records; provenance of a node or a pixel; statistics and budgets |
| **Mutate** | A batch of commands applied as one transaction. **Dry-run** returns the diff and regeneration impact without committing |
| **Build** | Regenerate a node, region or world; report timings and staleness |
| **Observe** | Render from a camera, a turntable or top-down; id image; field image; contact sheet. Returns file paths |
| **Validate** | Run validators; structured results |
| **Vary** | Produce N variations (seed or parameter perturbation) into proposal layers |
| **Run** | Play headless for N seconds with scripted input; return logs and screenshots |
| **Undo / history** | The journal |

Properties that make it trustworthy:

- **Versioned schema.** Errors are structured and say what would have been valid.
- **Proposal layers.** AI edits land in a layer you accept or discard, with the same diff as any regeneration.
- **The journal is replayable.** A session is a script; a good session becomes a regression test.
- **No model inside the engine**, no keys, no network dependency. Everything an assistant does, a human can do by hand through the same commands.

Gameplay content is created the ordinary way: the assistant writes script and data files in the project, asks for a reload, and uses *Run* to check.

### 9.3 Laboratory scene and test games

**The lab is a test suite you can walk through.** It is built from *stations*. A station is a small node subtree with one purpose, a fixed camera, validators, a golden screenshot, artifact hashes and a performance budget. Stations share one island so that cross-system behaviour (wind across forest and cloth, sound across zones) is exercised too.

Rules: a feature is not done until it has a station; a station is added in the same change as the feature; stations stay small. That is what keeps it an engineering testbed instead of a showcase level.

**Games as architecture validation**, kept in the repository, built and run headless with scripted input on every change. They are the canaries for script API stability.

| Game | Proves |
|---|---|
| **3D Tetris** | Your list, plus standalone export and the whole authored-only path with no generators |
| **Marble Run** | Your list, plus fixed timestep and physics debug tooling |
| **A card game** (patience or a small deck-builder) | UI, text, picking and drag, turn and state logic, undo, save; no physics, no world. The best single test that the engine is not over-specialized |
| **A short walk** (hand-built kit scene, third-person character) | Character controller, camera, animation, audio zones, triggers, dialogue-sized UI |
| **A tile-laying game** (later) | Runtime generation shipped in a game |

### 9.4 Versioning

One rule covers everything: **anything that can influence an output has an identity and a version or content hash; the manifest records which were used; a change is classified, never silently applied.**

| Thing | Versioned by | On change |
|---|---|---|
| Engine | Release number recorded in the project | Project migration on open, after an automatic backup |
| Schemas (node types, components, parameters) | Integer per type; stable field ids | **Migrate** on load; preserve unknown fields |
| Generators | Source hash (script) or declared version (native), plus a "keys compatible" flag | Affected nodes become **stale**; live and locked nodes both wait for an explicit upgrade (§3.7) |
| Kernel and output-affecting libraries | A kernel epoch in every cache key | **Invalidate** cache; nodes stale |
| Styles, substances, species, kits, source assets | Content hash per key | Live readers **regenerate**; locked readers **warn** |
| Specifications | Text files in git | History, branching and review come from git |
| Reproducible artifacts | Content hash | Disposable cache, not in git |
| **Precious artifacts** (locked nodes on a generator version that no longer exists; detached snapshots) | Content hash | **Preserve**: stored in the project, in git large-file storage |
| Scripts | Declared script API level | Deprecated calls work for one level with a warning; the test games catch breaks |
| Third-party libraries | Pinned in the build | Upgraded deliberately; kernel epoch bumped if output can change |
| Save games | Save version plus world manifest hash | Deltas keyed by id; missing ids dropped and logged |

Open question from §1.3: whether to keep old generator versions *runnable*. For script generators this is nearly free (keep the old source file, addressed by hash). For native ones it is not. This is another argument for §5.6.

---

## 10. Technology comparison

Only decisions that shape the architecture. In each, "good technology" and "good for this engine" are separated.

### 10.1 Language: C++ or Rust

| Area | C++20 | Rust | Edge |
|---|---|---|---|
| Graphics API | SDL3 GPU, Dawn, Vulkan all native | wgpu is first-class and arguably the most pleasant modern GPU layer in any language | Rust, slightly |
| Physics | Jolt, native | Rapier is pure Rust and decent but narrower (no soft bodies, simpler character controller). Jolt through bindings is possible and immature | C++ |
| Animation | ozz-animation, native | No comparable standalone runtime; you would write sampling and blending yourself | C++ |
| Audio | miniaudio and Steam Audio are C; fine from both | Same, plus capable native crates | Even |
| Procedural geometry libraries | Manifold, Clipper2, meshoptimizer, Recast, xatlas, FastNoise2 are all C++ | Bindings or ports for some; gaps for others | C++ |
| Procedural geometry, **own code** | Fine | Enums with data, pattern matching, effortless data parallelism, stricter floating-point defaults all suit kernel and generator code | Rust |
| Editor | Dear ImGui with docking and a large widget ecosystem, native | egui is good and different; ImGui bindings lag | C++, modestly |
| Reflection and serialization | No language reflection you can depend on yet; needs macros or code generation | Derive macros make reflection, serialization and hashing nearly free | **Rust, clearly**, and this engine leans on reflection for parameters, commands, hashing and bindings |
| Build and dependencies | CMake plus a package manager; workable, tedious | Cargo. Module boundaries as crates are natural | Rust |
| Iteration speed | Slow full builds; mitigated by scripting | Slow builds too; link times hurt in large workspaces. Hot reload is worse | Even; both push iteration into script and data |
| Runtime performance and memory control | Full | Full | Even |
| Debugging on Windows | Excellent | Adequate | C++ |
| Scripting host | Luau is C++ | Luau through a mature binding | Even |
| Stable API for scripts and tools | Via schema and a C-level boundary | Same | Even |
| **AI-assisted coding** | Models write fluent C++. Mistakes surface at *run time* as memory or threading bugs that are hard to see in review | Models write good Rust. A large class of mistakes surface at *compile time*, where the agent fixes them itself. But agents under borrow-checker pressure reach for clones, reference counting or `unsafe` | Rust for own code; C++ once six libraries sit behind hand-written `unsafe` bindings |
| One-developer maintainability | Refactors can break silently | Refactors are far safer | Rust |

**What this adds up to.** The engine has two halves with opposite profiles. The *runtime* is integration-heavy: Jolt, ozz, Recast, ImGui, Luau, Slang, Manifold. That half favours C++. The *authoring and generation* half is your own code, heavy on reflection, hashing, serialization, parallelism and determinism. That half favours Rust.

A split (Rust for generation, C++ for runtime) looks clean and is a mistake for one developer: two toolchains, and component and parameter types defined twice across the boundary that the editor crosses every frame.

**Recommendation: C++20 in a restricted dialect. Confidence: medium.**

Because the library gap (physics with cloth, animation, navigation, editor UI) is real work you would otherwise write or bind, and because the reflection gap can be closed by schema-first code generation (§10.2), which you want anyway.

The dialect: handles instead of pointers across modules; no owning raw pointers; value types; interfaces only at module boundaries; strict floating-point flags; address and undefined-behaviour sanitizers and static analysis on every change; warnings as errors.

**What would flip it to Rust:**

- You read Rust more comfortably than C++. Human review of architecture is the control in your workflow, and it needs a language you can read.
- The bake-off (§12, E2) shows materially fewer defects or less review effort in Rust.
- You decide simulated cloth and a rich character controller are not near-term, which removes Jolt's main advantage.

It is worth knowing that Tiny Glade was built in Rust with a custom renderer and Bevy's entity system, by two people. The Rust path is proven for exactly this kind of game.

### 10.2 Reflection and serialization

Options for describing types once:

| Option | For | Against |
|---|---|---|
| Hand-written registration macros beside each type | Simple, no tooling | Easy to forget a field; nothing generated for scripts or tools |
| Parse C++ headers with a compiler front end | Types written once in C++ | Heavy, fragile tooling |
| **Schema-first**: components, parameters and commands defined in small schema files; a generator emits the C++ types, serialization, script bindings and type declarations, and the JSON schema the AI API serves | One source for five consumers; survives a change of language; trivially diffable | A code generator to own; discipline to use it only for boundary types |

**Recommendation: schema-first for components, parameters, commands and artifacts.** Internal types stay ordinary code. Medium confidence; it could be overkill, and the phase 1 Tetris build is where that shows.

**Formats.** *Custom* is right, but only the thin layer, on top of standard encodings:

| Use | Choice | Why not the alternative |
|---|---|---|
| Source files (worlds, specs, styles) | Strict JSON, deterministic key order, one file per top-level subtree | YAML is ambiguous; TOML is poor for deep trees; a bespoke text format costs tooling and AI familiarity |
| Artifacts and packs | Own chunked binary: header with type, version, hash; data laid out for direct loading | FlatBuffers and similar are good technology, but schema-first generation already provides evolution and layout, and adding a second schema system is not worth it |
| Hashing | A canonical byte form per type (defined float handling); a fast cryptographic hash for content addressing | |

### 10.3 Window and input: SDL3 or GLFW

GLFW is good technology that does less: windows, basic input, basic gamepad.

For this engine SDL3 wins on things that matter to shipping the reference genres: a large gamepad database with hot-plug, rumble and sensors (all reference games are gamepad-first); audio device handling; file dialogs and storage paths; a path to handhelds; and a GPU API in the same dependency. **SDL3, high confidence.** Only the platform module includes it.

### 10.4 Renderer and graphics API

**Custom renderer or an existing framework?**

| Option | Assessment for this engine |
|---|---|
| Filament | Excellent physically based renderer. Its fixed material model is the opposite of pluggable shading models. Good technology, wrong fit |
| bgfx | Broad platform reach, mature. Older-API abstraction, its own shader dialect, weaker compute path |
| The Forge and similar | Aimed at high-end production |
| **Custom renderer over a thin GPU API** | The renderer described in §7.2 is small and specific. Owning it is what lets style select shading models and lets generators supply outline edges and wind weights |

**Custom. High confidence.**

**Which GPU API under it?**

| | SDL3 GPU | WebGPU (Dawn or wgpu-native) | Vulkan directly |
|---|---|---|---|
| Backends | Vulkan, Direct3D 12, Metal | Same, plus browsers | One, plus translation for Apple |
| Capabilities | Compute, indirect draws, enough for §7.2. No ray tracing or mesh shaders (not wanted). Simple fixed binding model | Similar level, stricter | Everything |
| Validation | Relies on backend layers | **Strong, readable messages**, which helps AI-written rendering code | Good, verbose |
| Dependency weight | Already present | Heavy build or prebuilt binaries; C API still moving | SDK and helper libraries |
| Shaders | Needs one format per backend; Slang emits all | WGSL; Slang emits it | SPIR-V |
| Code volume | Small | Small | Several times larger |

For the C++ path: **SDL3 GPU, medium confidence.** If a browser build matters to you, choose WebGPU instead. On the Rust path wgpu is the obvious answer. In all cases the engine's own GPU layer (around thirty functions) is shaped like WebGPU so a second backend stays possible.

### 10.5 Entity architecture

The usual argument for a heavyweight ECS is performance at high entity counts. **That argument is weak here**: vegetation, props and building parts are instance tables and merged geometry (§4.3), so entity counts are in the thousands. The drivers are script-friendliness, runtime-defined components, hierarchy, change tracking and debuggability.

| Option | For this engine |
|---|---|
| **flecs** | Hierarchy, runtime-defined components (good for script components), observers, powerful queries, a live inspector, a C API. One principal maintainer. Its own reflection and prefab features would go unused, since the document owns those. AI assistants often mix up its major versions |
| EnTT | Lean, fast, stable API that assistants know well. No hierarchy; runtime components are awkward; template-heavy builds |
| Simple custom store (sparse sets) | Two thousand lines, total control, trivially debuggable, adequate at these counts. You then own queries and change tracking |

**Recommendation: flecs for engine systems, with a script-facing entity API owned by the engine so scripts never see it. Low stakes, medium confidence.** If the Tetris and Marble Run phases show you are using a tenth of flecs, replacing it with a simple store is a contained job *because* the script API is yours.

This revises Pass 1's "do not wrap the ECS": do not wrap it for native engine systems; do own the scripting surface.

### 10.6 Scripting

| Option | For this engine |
|---|---|
| **Luau** | Fast, sandboxed, gradual types with a real checker (catches AI-written script errors before running), optional native code generation, assistants know it well. Debugger and editor integration need some work |
| Lua 5.4 / LuaJIT | No types. LuaJIT's performance edge does not matter when heavy work is in the kernel |
| C# | A better language with better tools. Hosting the runtime complicates packaging, hot reload and sandboxing, and makes generators-in-script (§5.6) heavier |
| AngelScript | Capable; small community and thin AI familiarity |
| WebAssembly | Elegant sandbox; binding ergonomics are poor and the toolchain is another layer |

**Luau, high confidence for gameplay, medium for generator recipes.** Bindings and type declaration files are generated from the schemas.

### 10.7 Physics

| Option | For this engine |
|---|---|
| **Jolt** | Rigid bodies, constraints, a strong character controller, soft bodies for cloth, heightfield and mesh shapes, batch add and remove of static bodies (regeneration), a deterministic mode, optional double precision. Proven in large shipped games and adopted by Godot |
| PhysX 5 | Complete and large; advanced features lean on one GPU vendor |
| Bullet | Effectively in maintenance |
| Rapier | The Rust choice; narrower |

**Jolt, high confidence.** Behind an engine interface that owns the collision *description*, since colliders are compiler artifacts.

### 10.8 Animation

| Option | For this engine |
|---|---|
| **ozz-animation** | A toolkit, not a framework: sampling, blending, two-bone and aim IK, compression, offline tools. It imposes no graph, which suits a pose pipeline where procedural sources are equals (§8.6). Slow-moving, stable, one main author |
| Own sampler and blender | Feasible (clip sampling is not hard). You lose compression and vectorized blending |
| A full animation framework | Would fight procedural animation |

**ozz, with the pose graph and all procedural layers engine-owned. Medium-high confidence.** Low lock-in: the pose pipeline, not ozz, is the architecture.

### 10.9 Navigation

| Option | For this engine |
|---|---|
| **Recast/Detour** | Voxelizes any geometry, so it works on generated and authored worlds alike; tiled, matching per-cell regeneration; off-mesh links accept traversal edges. One mesh per agent size, so scale variety costs builds. Ground agents only |
| Navigation polygons emitted directly by generators | Tempting, since generators know what is walkable. Fails on authored geometry and where generators overlap |
| Grid or graph navigation | Fine for small games; poor at world scale |

**Recast/Detour as a per-cell deriver for ground agents; the traversal graph for everything else. Medium-high confidence.** Not needed until agents exist.

### 10.10 Audio backend

| Option | For this engine |
|---|---|
| **miniaudio** plus engine-owned model | Small, permissive, full programmatic control, which generated soundscapes need. You build event authoring in the editor |
| FMOD or Wwise | Excellent authoring tools and adaptive music. Proprietary licence, a second editor, and awkward if the engine is ever given to others. Events must be authored in their tool, though instances can be driven from code |
| SoLoud, SDL_mixer | Simpler than needed |

**miniaudio behind the event-level interface of §8.7, with FMOD possible as an alternative backend. High confidence in the interface, medium in never wanting FMOD.** What would change it: you working with a sound designer who lives in FMOD.

### 10.11 Shaders

| Option | For this engine |
|---|---|
| **Slang** | Modules and interfaces express "shading model" as a real language concept instead of preprocessor branches; emits every format SDL3 GPU and WebGPU need; reflection output drives material parameter binding. Mostly a superset of HLSL, so assistants can write it. A large tool, used at build and edit time only |
| HLSL with a cross-compiler | Proven; modularity by `#include` and macros |
| WGSL or GLSL directly | Weaker modularity; ties you to one API family |

**Slang, medium-high confidence.** Risk: generic-heavy shader code is hard to debug, so use interfaces for shading models and little else.

### 10.12 Asset interchange

- **glTF 2.0** for meshes, skeletons, animation. Kit metadata (sockets, structural role, substance, tags) travels as custom properties from Blender, with an engine sidecar file for anything else.
- **FBX** only through conversion or a small reader if a specific asset demands it.
- **USD**: the layered-override *idea* is used throughout this design; the library is far too large to adopt.
- KTX2 for shipped textures; EXR or 16-bit PNG for authored field masks; WAV, Ogg or Opus for sound.
- The compiler can **export** glTF, for debugging and for use elsewhere.

High confidence.

---

## 11. Revised architecture

### 11.1 Layers

```
 APPS        Editor │ Command-line and headless │ Game player │ Command server (socket, MCP)
             ──────────────────────────────────────────────────────────────────────────────
 AUTHORING   World Model │ Command API │ Generation framework │ Kernel │ Generators │ Derivers │ Style
             ─────────────────────────────── cook ─▶ PACK ─────────────────────────────────
 RUNTIME     Entity store │ Renderer │ Fields │ Particles │ Camera │ Game UI │ Gameplay framework │ Save
             ──────────────────────────────────────────────────────────────────────────────
 SERVICES    GPU layer │ Assets + artifact store │ Script VM │ Physics │ Audio │ Animation │ Navigation
             ──────────────────────────────────────────────────────────────────────────────
 SEMANTICS   Tags │ Substances │ Capability profiles │ Schemas
 CORE        Handles │ Math │ Hashing │ Jobs │ Log │ Profiling │ Reflection │ Serialization │ Files
 PLATFORM    Window │ Input devices │ Audio device │ Filesystem │ Time            (SDL3)
```

Dependencies point downward only. Authoring and Runtime are side by side: neither includes the other. They meet at the pack format.

### 11.2 Data flow

```
 human ─┐
 AI ────┼─▶ commands ─▶ World Model ─▶ dirty set ─▶ evaluator ─▶ generators (plan, realize)
 script ┘        ▲          │                           │              │
                 │          │                           ▼              ▼
              journal    overrides                  manifest      artifacts (content-addressed)
              + undo                                                    │
                                                         derivers ◀─────┤
                                                              │         │
                                                              ▼         ▼
                                                        per-cell cook ─▶ PACK ─▶ runtime world
                                                                                      │
                               validation, screenshots, queries ◀─────────────────────┘
```

### 11.3 Boundaries

| # | Module | Owns | Must not depend on |
|---|---|---|---|
| 1 | **Core layers** (Platform, Core, Semantics) | OS access; handles, math, hashing, jobs, logging; type schemas; tag registry, substances, profiles | Anything above. Semantics holds data definitions only, no behaviour |
| 2 | **Major modules** | Each of rows 3–18 is one build target with a public header set | Each other's private headers. Enforced by the build |
| 3 | **Data flow** (cook and pack) | Pack format; per-cell cooking; incremental in-memory packs for the editor | Editor; any specific generator |
| 4 | **World Model** | Nodes, ids, prefabs, variants, overrides, layers, input shapes, node state | Runtime, renderer, physics, ECS, any generator |
| 5 | **Generation framework** | Descriptors, evaluator, stages, dependency recording, cache keys, manifest, staging and diff. Kernel beside it. Generators and derivers as leaves | Runtime, renderer, physics, ECS, editor. Generators must not depend on each other. Kernel must not depend on the World Model |
| 6 | **Runtime** | Entity store, pack loading, instance tables, promotion, game loop, time | World Model, generation, editor |
| 7 | **Rendering** | Render graph, shading models, materials, lighting, atmosphere, water, vegetation drawing, post | Physics, audio, gameplay, World Model. Reads fields and packs only |
| 8 | **Physics** | Engine physics interface; bodies, queries, character controller, cloth simulation; the Jolt adapter | Renderer, audio. Jolt types never appear in public headers |
| 9 | **Animation** | Skeletons, rig descriptions, pose pipeline, state graphs, procedural layers; ozz adapter | Renderer (it produces poses, not draws), gameplay rules |
| 10 | **Audio** | Events, emitters, parameters, buses, snapshots, zones; backend adapter | Renderer, physics (occlusion queries arrive through a small interface), generators |
| 11 | **Input** | Devices → actions and axes, contexts, rebinding | Gameplay meaning of actions |
| 12 | **Scripting** | VM host, generated bindings, hot reload, sandbox for recipes | It must not expose flecs, Jolt or renderer types; only the engine's schema-defined API |
| 13 | **Editor** | Views, gizmos, handles, thumbnails | Any mutation path other than commands. Reads through queries |
| 14 | **Asset system** | Asset ids, import, content hashes, metadata, thumbnails; artifact store | Meaning of assets; generators |
| 15 | **Serialization and versioning** | Canonical forms, text and binary writers, per-type versions, migrations | Any specific type's semantics |
| 16 | **Derived-data cache** | Content-addressed storage, reproducible versus precious, garbage collection rooted at manifests | Knowing what a blob means |
| 17 | **AI / Command API** | Command and query schema, transactions, dry-run, journal, undo, socket and MCP front ends | Editor UI. No language model, no network beyond localhost |
| 18 | **Testing and headless** | Headless app, screenshot capture, golden hashes and images, scripted input, station runner, identity scenario suite | Editor UI. Must run without a window |

Fields are a runtime module (field service) and a kernel type (compile-time fields); they share only the sample format, defined in Core.

### 11.4 Circular-dependency hazards to watch

| Hazard | Rule |
|---|---|
| Buildings ↔ terrain | Claims before terrain, realization after (§5.3) |
| Traversal ↔ generation | Routes emit requirements downstream; validation is last; repair inserts kit pieces and never re-runs layout |
| Style ↔ structure | Style influences structure only through numeric constraints read during planning (§7.4) |
| Audio ↔ physics ↔ rendering | Each talks to fields and small interfaces in Core, never to each other |
| Editor ↔ everything | Commands and queries only |
| Runtime generation ↔ runtime | The generation module produces in-memory packs; the runtime still sees only packs |

---

## 12. Revised roadmap

Pass 1 ran a compiler track and a runtime track in parallel from week one. I now think that is wrong in one respect: it risks a compiler built before the engine has proven it can make an ordinary game. The revised order is: **settle the unknowns cheaply, build the spine, prove ordinary games, then add generation on a foundation that already has overrides and commands.**

### Phase 0 — Experiments (short, partly throwaway)

| | Experiment | Resolves |
|---|---|---|
| E1 | **Identity prototype**, headless, no graphics: nodes, derived ids, stable scatter, overrides, pins, staging and diff, on a toy village and toy forest. The scenarios of §3.9–3.11 as automated tests | Whether §3 works; the key schemes; fingerprint matching quality |
| E2 | **Language bake-off**: E1 written in both C++ and Rust by the same assistant from the same brief | Language, on evidence: defects, review effort, your comfort reading each |
| E3 | **Look target**: one small hand-made diorama (in Blender or any existing tool), lit and graded to the look you want, then the same with flat ambient, with occlusion, and with bounce | Indirect lighting requirement; first style; shading model list; target hardware |

E1 is the most important piece of work in the whole plan. If it goes badly, the design changes before anything expensive is built.

### Phase 1 — Engine spine → 3D Tetris

Built together because each is meaningless without the others: platform; core; schemas and code generation; serialization; World Model with *authored nodes and prefabs with overrides*; command API with undo, journal and socket; entity store; pack and cook (trivial mapping); renderer v0 (forward, one ramp-lit model, sun shadows, height fog, tonemap, id buffer); input actions; Luau with generated bindings; audio events, buses and parameters; minimal game UI and text; editor shell (viewport, outliner, inspector, asset browser with thumbnails, console); headless mode with screenshot tests; standalone export.

**Result:** Tetris as a standalone build; lab v0 with three stations.

### Phase 2 — Ordinary-game competence

Physics and character controller; fixed timestep; animation with an imported rigged character and rig description; cameras; particles; 3D audio with authored zones; save games; UI beyond text; debug drawing.

**Results:** Marble Run; a card game; a short walk through a hand-built scene made from a small kit.

This phase is the direct answer to "ordinary games must not become awkward". Nothing in it involves generation.

### Phase 3 — Generation spine (the minimum architecture)

Generator descriptors and the plan/realize evaluator; dependency recording; artifact store, manifest, early cutoff; derived ids and overrides on generated nodes (reusing the prefab override layer); stable scatter; fields and stamps; anchor, path and region nodes with handles; core kernel; collision deriver; id buffer and the *Why?* panel; staged regeneration with diff and orphan inbox; node states; validation framework; style skeleton with one style; observer scale and profile in every context; audio emitters as an output type; recipes in Luau.

Generators in this phase are **toys only**: a fence along a path, rock scatter, a simple tree, box houses on lots.

**Exit criteria. This is the answer to "what must exist before sophisticated generators":**

1. An override on one node survives regeneration of its siblings.
2. Same spec, same hashes, across runs and machines.
3. One changed leaf recompiles one node; an unchanged collider hash stops the navmesh from rebuilding.
4. Raising scatter density moves no existing instance.
5. A moved house survives a change of house count; a deleted tree stays deleted after the lattice changes.
6. A generator version change marks nodes stale and changes nothing until requested.
7. Clicking any pixel shows its full provenance chain.
8. The whole editor session can be driven and replayed through the socket.
9. A toy generator can be edited in script and seen in under a second.
10. A cooked pack of the toy world runs in the game player with no generation code linked.
11. Changing a frame-bound parameter regenerates nothing; changing a generate-bound one regenerates only its readers.
12. Tetris, Marble Run and the card game still build and pass, untouched.

### Phase 4 — Landscape and mood (first deep generator; vertical slice)

Terrain with stamps; water v1; paths; species and communities; vegetation rendering and wind; rocks; sky, fog, grading, depth of field; mood presets; soundscape derivers; indirect lighting step chosen by E3.

**Result:** the lab island is beautiful in one style, at dawn and at dusk, in summer and autumn, and it sounds like a place.

### Phase 5 — Buildings, settlement, second style, traversal

Building generator to urban requirements; kit pipeline from Blender; settlement layout (village); decals and weathering; traversal features, graph, profile calibration, validator; navmesh; the realizer families; a second style.

**Results:** the coastal village example end to end; "make it older"; the same island as a wooden world; a validated route from beach to lighthouse.

### Phase 6 — Urban

Street graph, blocks, lots; exterior infrastructure routing; route-first prop placement; clustered lights, emissive signage, wet ground reflections.

**Result:** one district block with rooftop routes, by day and at night in rain.

### Phase 7 — Reactive world

Field service and modules beyond the first three; weather; snow and wetness; fire; simulated cloth; GPU particles.

### Phase 8 — Scale and scale-out

Tiny and giant stations; detail bands; discrete grow/shrink; origin rebasing; cells and streaming; the generation module shipped in a game (tile-laying game).

### Phase 9 — Creatures

Prerequisites from §8.6 have accumulated since phase 2. Body-plan generator, implicit mesher, procedural gaits.

### How the pieces relate

| Question | Answer |
|---|---|
| Must exist first | Schemas, serialization, World Model with overrides, commands and journal, headless tests |
| Can be stubbed | Localization; jobs (single-threaded behind the real interface); style (one, behind the four-layer interface); origin rebasing; navigation until agents exist |
| Can be deferred | Networking, plugins, cinematics, interiors, creatures, volumetric fog, probes, visual scripting |
| Must be proven experimentally | Identity (E1), language (E2), look (E3), recipe performance (phase 3), route-first placement (phase 6), scale bands (phase 8) |
| Vertical slices | Tetris; the short walk; the lab island at phase 4; the coastal village; the district block |
| Built together | Model + commands + serialization + editor shell. Generator framework + cache + provenance UI + overrides. Terrain + paths + vegetation + fields. Buildings + kits + style. Every generator with its audio and traversal outputs |
| Kept isolated | Kernel (no engine dependencies). Generators (no runtime). Runtime (no World Model). Physics, renderer and audio from each other. Script API from library types |

I still give no dates. Phases 0–2 are well-understood work. Phase 3 is the novel part and may need a second iteration. Phases 4–6 are where quality, not architecture, sets the pace.

---

## 13. Guardrails against the failure modes you listed

| Failure mode | Mechanism that prevents it |
|---|---|
| Ten generator products | Two deep generators at a time; no private data structures or panels (§5.5) |
| Abstractions before need | "Promote on second use" (§6); toys before depth (phase 3) |
| Photoreal complexity | Explicit absent list (§7.2) |
| Opaque generation | Provenance is automatic and the *Why?* panel is an exit criterion |
| Destructive baking | Source is never replaced; cooking is a deriver |
| Lost edits | Edits become inputs; regeneration is a staged diff; orphan inbox |
| Unstable identity | Declared key schemes with tests; stable scatter; named seed streams |
| Generators coupled to render, physics, ECS | Build-enforced; generators see only kernel and context |
| AI writing arbitrary internals | Task briefs name modules in scope; public headers are the contract; boundary checks fail the build |
| Editor bypassing commands | The editor has no other write access; the journal replay test proves it |
| Libraries dictating architecture | Engine-owned interfaces for six services; no library types in public headers |
| Huge framework before anything runs | Tetris ships at the end of phase 1, before any generation code |
| Ordinary games awkward | Authored-only path is first-class; three ordinary games are permanent tests |
| Creatures too early | Phase 9; prerequisites are each justified by something else |
| Everything depends on everything | Fields and defaults: any phenomenon module can be removed |
| Hypothetical AAA requirements | A stated target machine; no feature without a station that needs it |
| Unmaintainable by one person | Small module count, schema-first boundaries, tests the assistant can run alone |

---

## 14. Explicit recommendations

| Decision | Recommendation | Confidence | Why |
|---|---|---:|---|
| Language | C++20, restricted dialect | Medium | Library fit for the runtime half; reflection gap closed by schema-first generation |
| Runtime architecture | Own runtime; authoring, generation and runtime as peers meeting at a pack format; generation optional and shippable | High | Keeps ordinary games simple and runtime generation possible |
| Graphics API | SDL3 GPU behind a WebGPU-shaped engine layer | Medium | Least work for the needed feature level |
| Window/input layer | SDL3 | High | Gamepad, audio device, handheld path |
| Renderer architecture | Custom; clustered forward with thin prepass; one render graph; pluggable shading models; replaceable indirect term | High (structure), Medium (indirect light) | Matches the style system and the references |
| ECS/entity architecture | flecs for engine systems; engine-owned script API; instance tables for mass content | Medium, low stakes | Entity counts are low; scriptability matters more than speed |
| Physics | Jolt behind an engine interface | High | Feature fit including cloth and character control |
| Animation | ozz-animation; engine-owned pose pipeline and rig descriptions | Medium-high | Toolkit that does not fight procedural animation |
| Audio | Engine-owned event-level model; miniaudio backend; soundscape derivers | High (model), Medium (backend) | Generated worlds need programmatic audio |
| Scripting | Luau, bindings generated; also used for generator recipes | High (gameplay), Medium (recipes) | Types catch AI errors; fast iteration; automatic versioning |
| UI | Dear ImGui for the editor; RmlUi for games | Medium-high | Fastest route to a capable editor; game UI not yet exercised |
| Asset format | glTF 2.0 in; engine pack out; kit metadata as custom properties | High | Sufficient and simple |
| Serialization | Schema-first; JSON source files; own binary artifacts; per-type versions and migrations | Medium-high | One definition serves engine, scripts, tools and AI |
| Procedural generation kernel | Small core (§6); promote on second use | High | Avoids both reinvention and a premature framework |
| World Model | Node tree with three child sources and one override mechanism; assigned, derived and slot ids | High (shape), Medium (details of matching) | Unifies prefabs and generation; the identity model |
| Editor architecture | In-process, commands only, descriptors drive UI, parameter tiers | High | Boundary by construction |
| AI integration | External; socket and MCP; dry-run, proposal layers, replayable journal | High | Deterministic and inspectable |
| Terrain/world generation | Heightfield terrain with stamps; layered suitability ecology; stable scatter | Medium-high | Proven techniques; overhangs unresolved |
| Architecture generation | Hybrid: parametric massing, split-grammar facades, kit and realizer filling, routed infrastructure | Medium | Each technique is proven; the combination under style and traversal is not |
| Creature generation | Late; skeleton-first; imported rigs behind the same node interface until then | Medium (approach), High (timing) | Prerequisites are independently useful |

**Resolving the medium and low confidence items**

| Item | Experiment or research |
|---|---|
| Language | E2 bake-off; your reading comfort |
| Graphics API | Your answer on browser builds; phase 1 renderer experience |
| Indirect light | E3 look target |
| ECS | Usage review after phase 2: how much of flecs is actually used |
| Audio backend | Whether a sound designer or adaptive music enters the picture |
| Recipes in Luau | Phase 3 timing of toy generators both ways |
| UI | The card game |
| Serialization (schema-first) | Whether it feels heavy during phase 1 |
| World Model matching | E1 scenario suite |
| Terrain overhangs | Coastal cliff with placed rock meshes in phase 4 |
| Architecture generation | The lighthouse and one apartment facade, in two styles, in phase 5 |
| Creature approach | A single procedural quadruped gait on an imported rig, any time after phase 2 |

---

## 15. Where I disagree with you, and where I was wrong

**Ideas I think are impractical as stated, with the interpretation I would keep**

| As stated | Problem | Better interpretation |
|---|---|---|
| "Generate … with a climbable route" as a guaranteed, good result | Valid routes are achievable; fun ones are a design problem | Route constraint + validation + kit-based repair; a human sets anchors and judges pacing |
| Any substance as a world material | Each excellent style is weeks to months of content | Four realizer families and a small vocabulary make it *possible*; plan two styles in year one |
| district → … → rooms → windows for every building | Interiors everywhere are mostly unseen cost | Rooms as data; interiors generated only for enterable buildings |
| Player that grows and shrinks, in general | Continuous change across magnitudes needs every generator at every scale | Discrete scale bands with a transition |
| "Genuine general-purpose 3D engine" | General in *architecture* is realistic. General in *breadth* (networking, mobile, consoles, virtual reality, 2D) is not for one developer | Define ordinary games as single-player 3D on desktop and handheld PC; keep the rest out explicitly |
| Urban with equal status *at the same time* as landscape | Two deep generators in parallel is the ten-products failure | Equal status in design from the first building; sequential in construction |

**Where Pass 1 was too conservative or too conventional**

- Calling the runtime "modest" and the compiler the centre.
- Stubbing audio.
- Ruling out global illumination categorically.
- Treating runtime generation as a question instead of an eventual requirement.
- Assuming generators are native code.
- Underweighting Rust for the half of the engine that is your own code.
- Listing technologies without tying each to something in this design.

**What I am least sure of in this pass**

1. Fingerprint re-matching quality (§3.12).
2. Whether recipes in Luau are fast enough.
3. Whether the cheap indirect term gets close enough to the Tiny Glade softness.
4. Whether four realizer families really cover the substances you care about, or whether "cloud" and "ice" each need their own.
5. The language.

**What I need from you for Pass 3**

1. Which language can you read comfortably, and do you want to run the bake-off?
2. Target machine and whether a browser build matters.
3. The first style, ideally as a handful of reference images for E3.
4. Whether the engine is ever intended for other people (affects audio backend, editor polish and licensing choices).
5. Your reaction to §3. If the identity model is wrong, everything built on it moves.

