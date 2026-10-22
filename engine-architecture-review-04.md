# 3D World-Building Engine — Pass 4: Technology and Development-Time Trade-offs

Status: analysis for a decision. No code, no implementation plan.
Builds on [Pass 3](engine-architecture-review-03.md). The architecture of Pass 3 §1–§5 (identity, edits, freeze, selective regeneration, generator contract, schedule) is taken as settled and is not reopened.
Facts about third-party technology are from my own knowledge and were not re-checked online. The ones a decision would rest on are listed in §10.3 and should be confirmed by the probe in §10.2 before committing.

---

## 0. Short answer

1. **The stack I recommended in Passes 2 and 3 is the slow one.** In it, well over half of the total work is infrastructure that is not distinctive: renderer, editor shell, entity runtime, script bindings, animation, audio engine, game UI, asset import, export. None of that is what makes this engine worth building.
2. **The Pass 3 architecture already contains the escape.** The runtime firewall (§6.3 there) says the runtime knows only ids, tags, semantic records, three weights and chunk replacement. A runtime that knows so little can be *someone else's engine*. The distinctive half (document, identity, Build, Generation, kernel, recipes) can be a self-contained native library that drives an existing engine through a thin adapter.
3. **Recommended: the balanced approach, with Godot 4 as the host.** Own the core; borrow the body. This removes most of the non-distinctive work, keeps every decision of Pass 3 §1–§5, and is not a dead end: the adapter boundary is exactly the one a custom runtime would later plug into.
4. **This changes the language answer.** Under the custom stack, C++ led because of Jolt, the animation library, navigation and the editor UI toolkit. Under the balanced approach the host supplies all of those, and what remains is own-code: hashing, serialization, tree diffing, a parallel evaluator. That is where Rust is stronger. My recommendation becomes **Rust for the core, at medium confidence**, with C++ as the fallback if the host binding disappoints (§7).
5. **What no approach shortens:** writing good generators and making excellent styles. That work is identical in all three approaches and is the real long pole. The reason to pick a faster stack is to *reach* that work sooner, not to avoid it.

---

## 1. Where the effort actually is

Relative sizes only. "Distinctive" means: would this engine be pointless without our own version of it?

| Work package | Distinctive? | A: custom stack | B: own core, hosted | C: inside the host |
|---|---|---|---|---|
| Window, input, platform | No | Low | None | None |
| GPU layer, shader toolchain | No | Moderate | None | None |
| Renderer (lighting, shadows, fog, post, water, vegetation drawing) | Partly: the *look* is, the plumbing is not | **Very high** | Low–moderate: shaders and a few effects | Low |
| Editor shell (docking, viewport, gizmos, inspector, asset browser, undo, play-in-editor) | No | **High** | None | None |
| World-building UI (*Why?* panel, regeneration diff, inbox, handles, field overlays) | **Yes** | Moderate | Moderate | Low–moderate |
| Entity store, game loop, save games | No | High | Low | None |
| Gameplay scripting host and bindings | No | High | None | None |
| Physics and character controller | No | Moderate | None | None |
| Animation runtime, state graphs, IK | No | High | Low | None |
| Audio engine and event model | Partly: soundscape derivers are | Moderate–high | Low | Low |
| Game UI and text | No | Moderate | None | None |
| Asset import (glTF, textures, sound) | No | Moderate | Low | None |
| Standalone export and packaging | No | Moderate | None | None |
| Schema, code generation, serialization | Partly | Moderate–high | Moderate (core types only) | Low |
| **Document, identity, edits, freeze, matching** | **Yes** | Moderate | Moderate | Moderate, smaller |
| **Build: evaluator, traces, artifact store, derivers** | **Yes** | High | High, can be staged | Low: coarse |
| **Generation kernel** | **Yes** | High | Moderate–high | Moderate: host built-ins |
| Pack, cook, runtime loading | — | Moderate | Moderate: the adapter | None |
| Command and AI API, journal | **Yes** | Moderate | Moderate | Low |
| Headless tests, lab stations | Yes | Moderate | Moderate | Low |
| **Generators, kits, styles, species (content)** | **Yes** | **Very high** | **Very high** | **Very high** |
| **Overall** | | **Very high** | **Moderate–high** | **Moderate** |

Two things to read from this table.

- Approach A's three largest non-content items (renderer, editor shell, runtime with scripting) are all "No" or "Partly" in the second column.
- The last content row does not move. If approach A takes years, most of those years are spent before the first beautiful generated place exists. That is the practical argument against it, more than the total.

---

## 2. Three approaches

### 2.1 Approach A — Maximum control

**Stack:** the Pass 2–3 recommendation. Own runtime; own clustered-forward renderer on SDL3 GPU with Slang; Jolt, an animation toolkit, miniaudio and Recast behind engine interfaces; Luau for both gameplay and recipes with generated bindings; Dear ImGui editor; schema-first code generation; own pack format.

**What the extra complexity buys**

| Capability | Why only A gives it fully |
|---|---|
| Renderer built around generator output: an id buffer, generator-declared outline edges, exact wind weights, a replaceable indirect term | You own every pass |
| One override mechanism for prefabs *and* generators | You own the scene model |
| Per-cell, content-addressed packs and byte-level patches | You own the load path |
| Per-instance GPU culling and density-field expansion tuned to your data | You own the draw path |
| No dependency on anyone's roadmap; no host concepts in a shipped game | — |
| A tiny shipped runtime | — |

**Savings available inside A** (an "A-lite"): plain HLSL instead of Slang; hand-written type registration instead of schema-first generation; a simple entity store instead of flecs; ImGui for game UI as well; an FMOD backend instead of an own event model. Together these take A from very high to high. They do not change its character: most of the work is still not distinctive.

**Verdict:** the best *engine*, and the approach most likely to collide with your statement that a multi-year build is not viable. I no longer recommend starting here.

### 2.2 Approach B — Balanced: own core, borrowed body

**Idea.** Split along the firewall that Pass 3 already drew.

```
 CORE  (ours; a native library plus a command-line tool; runs headless)
   document · identity · edit records · freeze snapshots · matching ladder
   Build: evaluator, traces, artifact store, derivers
   Generation: kernel, recipe host (Luau), plan store, staging
   command and AI API, journal
                 │
                 │  artifacts: meshes, instance tables, colliders, field tiles,
                 │  surface features, semantic records, audio zones  +  chunk replacement
                 ▼
 ADAPTER  (ours; thin; the only code that knows both sides)
                 ▼
 HOST ENGINE  (Godot 4)
   renderer · physics (Jolt) · audio · animation · game UI · input · navigation
   gameplay scripting · editor shell · asset import · export
```

**What we still write:** everything in the core; the adapter; the world-building panels as editor plugins; the *look* as shaders and a few post effects on the host's renderer; a small audio-event layer if the host's bus model proves too plain.

**What we stop writing:** every "None" in the B column of §1.

**How the Pass 3 concepts land**

| Pass 3 concept | Under B |
|---|---|
| Authoring → Build → Generation → Runtime | Unchanged. Runtime is the host |
| Runtime firewall | Becomes the adapter boundary. Still mechanically checkable: the core has no host headers |
| Pack | In the editor: in-memory chunks handed to the adapter. For shipping: either host-native resources written by the adapter, or core artifacts loaded by the adapter at run time (needed anyway for games that ship generation) |
| Three weights | Entity → a host node. Instance → a batched-instance object per cell and archetype. Merged → one mesh per cell part. Promotion → swap an instance for a node with the same id |
| Authored nodes, prefabs | **The host's own scene nodes and scene instances**, with the host's own override mechanism. The core's document covers generator roots, their inputs, edit records and snapshots |
| Attachments | A host node carrying (owner id, key path, fallback frame); the adapter re-parents it after each regeneration |
| Gameplay scripts | The host's scripting language. Luau remains for recipes only, inside the core |
| Editor | The host's editor plus our panels. Generator parameters appear in the host inspector, driven by descriptors. Handles are host gizmos |
| Commands, undo | Core commands, registered with the host's undo system so one Ctrl-Z stack exists |
| Headless build and tests | The core's own CLI, with no host involved. This is what keeps the AI-assisted loop fast |

**What is lost or weakened, honestly**

| Loss | Severity |
|---|---|
| One unified override model for prefabs and generators. Prefabs use the host's; frozen and detached exist only for generated content | Mild. It was elegant more than necessary |
| Control of renderer internals. No custom global-illumination technique without modifying the host. Per-instance culling is per batch, not per instance | Mild on your hardware; see §8 |
| A pixel-exact id buffer. Picking generated objects is done by the core against its own geometry instead | Mild |
| Byte-level, per-cell patches of shipped games | Mild; irrelevant until something ships |
| Independence. Host upgrades can break the adapter; host bugs are not ours (but the source is available) | Moderate, permanent |
| Two scene models side by side: host nodes and core nodes. The outliner must present them coherently | Moderate. This is the main design problem B introduces |
| Speed of chunk replacement is bounded by how fast the host can create meshes and colliders | **Unknown.** Decides whether a Tiny Glade-style live game is comfortable. Must be probed |
| Two scripting languages (host language for gameplay, Luau for recipes) | Mild |

**Verdict:** the primary candidate.

### 2.3 Approach C — Development-time optimized

**Idea.** Build the generation system *as content of the host*: an add-on, with native extension code only for hot loops.

| Pass 3 concept | Under C |
|---|---|
| Document | The host scene tree and its text resource files |
| Generator node | A host tool node that rebuilds its children in the editor |
| Generated children | Real host nodes, created on the fly and not saved |
| Recipes | Host scripts. No Luau |
| Kernel | Mostly the host's built-ins: polygon offset and clipping, mesh construction tools, noise, curves, batched instances, navigation baking. Plus a community heightfield-terrain extension |
| Edit records, ladder, modes | Implemented as in Pass 3, as a resource on the generator node |
| Freeze | Save the generated subtree as a host scene file. Detach = the host's "make local" |
| Build | Per-node caching keyed on a parameter hash and hand-declared dependencies. No traces, no per-concern cutoff, no tiles at first |
| Pack, firewall | None. The runtime *is* the generated host nodes |

**What is lost**

| Loss | Consequence |
|---|---|
| Recorded reads. Recipes can touch anything in the scene | Selective regeneration becomes "regenerate this node and hope the declared dependencies are right". Stale output becomes possible |
| The firewall | Gameplay can and will reach into generator nodes. Runtime generation ships the whole tool |
| Sandbox and enforced determinism | Results can drift with host versions (its noise, its triangulator) |
| Independent headless core | Tests need the host in headless mode: slower, coarser |
| Scale | Everything is a node. Comfortable for a diorama, a village, a small island. Not for a district or a large landscape |
| Recipe performance | Host script is slower than Luau and runs on the main thread |
| Per-cell cooking | No streaming story |

**What survives:** the identity model, the edit classes, the matching ladder, the three modes, the planning and realization schedule, fields and stamps. Those are logic, not infrastructure.

**The real danger of C** is your own core question: *"…without turning the system into an unmanageable special-case machine."* Without the context as the only door, each generator reads what it likes, and coupling grows quietly. C is the approach most likely to produce a pile of clever tool scripts.

**Verdict:** the fastest route to something beautiful on screen, and a legitimate way to *learn* (key schemes, the look, what recipes need from a kernel). As the final architecture it gives up selective regeneration at scale, which is one of the things the engine is for.

### 2.4 Considered and rejected as the generation system

| Option | Why not |
|---|---|
| Houdini (through its engine plug-in), Blender Geometry Nodes | Mature, powerful procedural tools. They have no concept of identity that survives regeneration, no edit records, and cannot run in a shipped game. They replace exactly the part that is distinctive and keep none of its properties. Useful as *asset* sources (kit pieces), not as the world system |
| Unreal's built-in procedural framework | Same objection, inside a host that is also the wrong fit (§5) |
| A voxel world plug-in | Solves a problem Pass 3 chose not to have |

---

## 3. Side-by-side comparison

| Area | A: custom | B: own core, Godot host | C: inside the host |
|---|---|---|---|
| **Renderer / graphics** | Own clustered forward renderer. Everything possible, everything to build | Host's clustered forward renderer: MSAA, shadows, volumetric fog, several global-illumination options, sky shaders, post stack already exist. The look is custom surface shaders with a custom light function, which is close to Pass 3's "shading model" interface, plus a few custom post passes | Same as B |
| **Physics** | Jolt, integrated and wrapped by us; controller written by us | Jolt, already integrated; rigid bodies, character body, soft bodies available | Same as B |
| **Audio** | Own event model on a small backend | Host buses, 3D players, effects. A thin event and parameter layer on top; soundscape derivers still ours. FMOD integration exists as a community extension if needed | Host audio, directly |
| **Animation** | Toolkit plus own pose pipeline and state graphs | Host animation tree, blend spaces, skeleton modifiers for procedural layers. Procedural gaits written as modifiers | Same as B |
| **Procedural geometry** | Own kernel plus native libraries (polygon clipping, booleans, simplification) | Same own kernel, same libraries, in the core. Unchanged | Host built-ins; own code where they run out |
| **Procedural generation** | Full Pass 3 | Full Pass 3 | Pass 3 identity and schedule; coarse Build |
| **Editor** | Own, in Dear ImGui | Host editor plus our plug-in panels | Host editor plus tool scripts |
| **Asset pipeline** | Own importers, thumbnails, kit metadata | Host import; kit metadata from glTF custom properties read by the adapter | Host import |
| **Serialization / data model** | Schema-first for everything | Schema-first only for core types; host formats for the rest (text-based, so diffs and AI edits work) | Host formats |
| **Runtime** | Own | Host, behind the adapter | Host, no boundary |
| **Tooling / debugging** | Own profiler hooks, debug draw, replay | Host profiler, debugger, remote scene inspector for the game side; core replay and traces for the generation side | Host tools only |
| **AI-assisted development** | Large native codebase; assistants do well inside clear module boundaries, but there is a great deal of it | Small, pure, headless core with fast tests: ideal for an assistant. Host-side work is scripts and shaders in a widely known engine | Fastest to write; weakest verification |
| **Dependency maturity** | Ten or so libraries, each mature, integration ours | One large, mature, open-source dependency, plus a handful of geometry libraries | One dependency |
| **Windows development** | Fine | Fine; the host is developed primarily on desktop | Fine |
| **Scalability to weaker hardware** | Best: we decide every cost | Host has lighter rendering back ends, but our shaders will target the full one; quality settings scale lighting and fog. Adequate | Worst: node count is the limit |
| **Long-term maintenance** | Everything is ours, forever | Core and adapter are ours; track host releases | Least code; most exposure to host changes |
| **Distinctive capabilities kept** | All | All of Pass 3 §1–§5; renderer-specific extras reduced | Identity and workflow; not selective regeneration at scale, not the firewall |
| **Relative effort before the first deep generator** | Very high | Moderate | Low |
| **Relative total effort** | Very high | Moderate–high | Moderate |

---

## 4. The trade-offs, classified

*Time saved ↔ capability, quality or flexibility lost.*

### 4.1 Essentially free

Large saving, almost nothing lost for this engine.

| Compromise | Saves | Why it costs nearly nothing |
|---|---|---|
| Host editor shell instead of our own | High | Docking, viewport, gizmos, inspector, asset browser are generic. Our panels are the distinctive part and are kept |
| Host physics | Moderate | It is the same library we chose |
| Host animation runtime | High | The pose pipeline we wanted (clips as one source among several) is available as skeleton modifiers |
| Host game UI, input, text, export, packaging | High in total | No distinctive requirement in any of them |
| Host gameplay scripting; drop Luau bindings for gameplay | High | Recipes are where Luau mattered (sandbox, hashing, speed) |
| Host asset import | Moderate | glTF in was the plan anyway |
| Host navigation baking | Low–moderate | Same underlying library |
| Drop the GPU layer and shader toolchain | Moderate | — |
| Drop the entity-store decision | Low | — |
| Run the look experiment in the host, now | — | The diorama lit three ways (Pass 2's E3) needs no engine of ours. It becomes a few days of work instead of a prerequisite |
| Not designing around handheld constraints | Moderate | See §8 |

### 4.2 Reasonable trades

Real saving, real loss, and the loss is recoverable later.

| Compromise | Saves | Loses | Recoverable? |
|---|---|---|---|
| Host renderer with custom shaders, instead of our own | **Very high** | Own indirect-light technique; per-instance culling; exact id buffer; total control of pass order | Yes: the adapter is where a custom renderer would attach |
| Host scene model for authored nodes and prefabs | High | Unified override mechanism; freeze and detach for prefabs | Partly |
| Host resources for shipped content instead of our pack | Moderate | Fine-grained patching; smallest possible runtime | Yes |
| **Stage the Build system**: traces per node first; tiles, per-cell cook and per-concern cutoff when a world is big enough to need them | Moderate | Early incremental performance | Yes, *provided the evaluation context is the only way to read anything from the first day*. Recording can be coarse at first. A door that was never there cannot be added later |
| Thin event layer over host audio instead of a full event model | Moderate | Authoring depth for adaptive sound | Yes |
| Start the kernel from existing libraries (polygon clipping, noise, simplification) with thin wrappers | Moderate | Nothing notable | — |
| Hand-written registration for the few core types, schema generator later | Low–moderate | Automatic consistency between serialization, hashing and bindings | Yes |
| Worlds up to a few kilometres without large-coordinate support | Low | Very large continuous worlds | Yes: origin shifting in the adapter |

### 4.3 Would fundamentally undermine the engine

Do not take these, however much time they save.

| Shortcut | What it breaks |
|---|---|
| Recipes that read the scene directly instead of through the evaluation context | Selective regeneration, provenance, determinism. All of Pass 3 §3 |
| Identity taken from host node names or paths | Everything in Pass 3 §1 |
| Saving generated content into host scene files as the source of truth | Non-destructive regeneration. This is destructive baking by another name |
| An external procedural tool as the generation system | Identity, edits, runtime generation (§2.4) |
| A core that cannot run without the host | The fast headless test loop, and with it the AI-assisted workflow |
| Gameplay code that refers to generator concepts | The firewall; ordinary games become entangled |
| A closed-source or very heavy host | The ability to fix or work around the one dependency everything rests on (§5) |
| Skipping the planning and realization split to get geometry sooner | Identity would depend on mesh code |

---

## 5. Which host

The host must supply: a clustered forward renderer with custom surface *and* lighting shaders, MSAA, fog and post; Jolt-class physics; skeletal animation with procedural hooks; batched instancing; meshes and colliders created at run time; an extensible editor; a native extension interface; text-based project files; headless operation; open source.

| Host | Fit | Decisive points |
|---|---|---|
| **Godot 4** | **Best** | Permissive open-source licence; meets every item on the list; small and fast to iterate; text scene and resource formats suit version control and AI assistance; native extension interface in C++ officially and Rust through a community binding; assistants know its scripting language well. Weak points: per-batch instance culling; renderer internals are harder to extend than its shaders; large-coordinate support needs a custom build |
| **Bevy** (Rust) | Partial | A hackable modular renderer and a good entity system, all in Rust. But no mature editor, so the largest "free" saving in §4.1 disappears, and it has frequent breaking releases. It is approach A with a head start, not approach B |
| **Unreal** | Poor | Enormous capability, most of it photoreal machinery the vision excludes. Custom shading models need engine modification. Binary assets and long compile times work against a one-person, AI-assisted loop |
| **Unity** | Poor | Technically workable with a native plug-in core. Closed engine source, so problems at the adapter boundary cannot be fixed; licensing history is a risk for a long project |
| **Flax, Stride, O3DE** | Not enough advantage | Each is capable. Smaller communities, thinner AI familiarity, and nothing they do better than Godot for this purpose |

**Godot, at medium-high confidence**, subject to the probe in §10.2.

### 5.1 Why this is not a dead end

The route from B to A stays open because of where the line is drawn:

- The core never includes a host header. It emits renderer-agnostic artifacts and chunk replacements, as Pass 3 §6 and §8.1 specified.
- The adapter is the *only* host-specific code. A custom runtime, should one ever be justified, is a second adapter.
- Styles are partly host-specific (shaders). That is the same cost Pass 3 §8.2 already accepted for renderer replacement.

What does get locked in under B is gameplay code and authored scenes of any *game* made on it. That is true of every engine.

---

## 6. What Pass 3 keeps and what it drops under B

| Pass 3 section | Status |
|---|---|
| §1 Identity, edit classes, matching ladder | Kept in full |
| §2 Live, frozen, detached | Kept for generated content. Prefabs use the host's mechanism |
| §3 Evaluation units, traces, binding tiers | Kept. Introduced in stages (§4.2) |
| §4 Generator contract | Kept |
| §5 Planning and realization schedule, field registry | Kept |
| §6 Runtime representation | The firewall becomes the adapter. Pack format becomes an internal artifact format plus adapter output |
| §7 Bake-off | Part A kept. Part B's four probes collapse into one host probe (§10.2) |
| §8 Renderer decisions | Replaced by a short list of what we need from the host renderer. Items 3–5 and 11 (geometry vocabulary, ids as indices, material description, water and vegetation data contracts) remain as the adapter's input format |
| §9 Luau boundary | Recipes: kept. Gameplay: host language |
| §10 Scale and traversal | Kept. Physics-per-band becomes a host question for later |
| §11 Risks | Risks 9 (language), 13 (indirect light), 18 (GPU feature) shrink. A new one appears: host dependency and chunk-replacement speed |

---

## 7. Language, in the same practical frame

Evaluated on technology only. The third column is the point: most criteria that favoured C++ were about libraries the host now provides.

| Criterion | C++ | Rust | Under B, does it still matter? |
|---|---|---|---|
| Graphics ecosystem | All native | Excellent own GPU layer | **No.** Host renders |
| Physics ecosystem | Jolt native | Narrower native option; Jolt through bindings | **No.** Host |
| Audio | Fine | Fine | No |
| Animation | Mature toolkit | Would be written by hand | **No.** Host |
| Editor and tool development | Dear ImGui, native | Good alternative, different | **No.** Host editor; panels are host-side |
| Procedural geometry libraries | Polygon clipping, booleans, simplification, noise: all native | Pure-Rust polygon boolean and noise libraries exist; bindings for simplification and booleans exist | Yes. C++ modestly ahead on booleans |
| Procedural geometry, own code | Fine | Sum types, pattern matching, easy data parallelism | Yes. Rust ahead |
| Dependency management | CMake plus a package manager; tedious | One tool; trivial | Yes. Rust ahead |
| Serialization, hashing, reflection of own types | Macros or a code generator we write | Derives; a fast content hash native to the ecosystem | **Yes, heavily.** The core is made of this. Rust clearly ahead |
| Debugging and profiling on Windows | Excellent | Adequate | Yes. C++ ahead |
| AI-assisted coding | Fluent; errors surface at run time | Fluent; a large class of errors surface at compile time, where an assistant fixes them unaided | Yes. Rust ahead for a core whose bugs are silent wrong results |
| Iteration speed, build times | Slow full builds | Slow full builds; fine for a small workspace | Roughly even: the core is small, and tuning happens in recipes |
| Host integration | Official extension library; same language as the host, so its source can be read and stepped into | Community binding: capable and used in shipped games, but not yet at a stable major version | Yes. C++ ahead |
| Luau hosting | Native | Through a mature binding | Even |
| Parallel evaluator without data races | By discipline and sanitizers | By construction | Yes. Rust ahead |
| One-developer maintainability | Refactors can break silently | Refactors are much safer | Yes. Rust ahead |
| Integrating native third-party code | Direct | Through bindings; routine for C interfaces, more work for C++ ones | Yes, but the list of such libraries is now short |
| Expected complexity for *this* engine | A: best. B: fine | A: poor (six bindings). B: good | — |

**Result by approach**

| Approach | Language | Reasoning |
|---|---|---|
| A | **C++** | Integration-heavy. Unchanged from Pass 2 |
| B | **Rust for the core** (medium confidence); adapter in Rust through the community binding, or a small C++ adapter over a C interface if that binding proves a liability | The core is pure own-code where compile-time checking, derives and safe parallelism pay most, and the one remaining integration is a narrow boundary |
| C | The host's scripting language, with C++ for hot loops | Follow the host |

**What would move B back to C++:** the host binding failing the probe; needing to patch or step through the host's own source often; or a hard dependency on a C++-only geometry library early. None is unlikely enough to dismiss, which is why this is medium confidence.

Under B the Pass 3 bake-off shrinks to its Part A, and its bias toward Rust is no longer a flaw: the prototype now *is* representative of what will be written in that language.

---

## 8. Hardware

An RTX 4080 laptop GPU with 12 GB removes most of the reasons the custom renderer was attractive:

- The host's clustered forward path with MSAA, volumetric fog and its real-time global-illumination options run comfortably at this level. Pass 3 planned a cheap indirect term *because* cost was assumed to matter; here a ready-made, better one can simply be switched on.
- Per-batch instance culling (one batch per cell and archetype) wastes some GPU work compared with per-instance culling. On this hardware that is unlikely to matter at the world sizes in view.
- 32 GB of system memory makes the in-memory artifact store and plan store simple.

**Scalability to weaker hardware** is kept by things already in the architecture and independent of the host: cells, instance tables instead of nodes, detail bands, impostors as a deriver, and quality settings for lighting and fog. I see no compelling architectural reason to design around handheld limits. The one habit worth keeping is counting instances and draw batches per cell from the start, because that number, not shader cost, is what a weak machine runs out of first.

---

## 9. Making unstable identity hard to create by accident

Your principle: *the generator author defines semantic identity, but the engine makes unstable identity difficult to create accidentally.* All eight mechanisms you listed are feasible, cheap, and belong in the smallest prototype, because they are what stops the special-case machine from forming. Most of them work because `plan` is cheap and pure: it can be run speculatively, as often as needed, without building anything.

| Mechanism | How | Strength |
|---|---|---|
| **Canonical child-identity API** | A child can be created only through one context call that takes a **typed key**, never a string or a bare number. Key constructors name their kind: *anchor*, *named*, *along an edge (edge id, side, position)*, *grid cell*, *lattice slot*. There is no constructor that takes a loop index. An ordinal exists only as an explicitly named escape that requires the child type to be declared interchangeable | Strong. The unstable choice is not the default and cannot be made silently |
| **Deterministic-key helpers** | The kernel's layout primitives hand out keys themselves. "Distribute along this edge at this spacing" returns positions *with* keys that stay put when the edge lengthens. Split and repeat scopes derive keys from the rule path, with a declared policy for where counting starts (from the start, the end, or the centre). In a grammar the author never invents a key at all | Strong for grammars and regular layouts, which are the bulk of cases. Does not help free-form layouts |
| **Duplicate-key detection** | A hard error during planning; the node keeps its last good output and shows the two colliding children | Complete |
| **Identity regression tests** | Generated for every generator, not written: perturb each parameter slightly and report the share of keys that change; inject pins and exclusions and check they are honoured. Plus stored key-set snapshots for lab stations | Strong. Catches drift when a recipe is edited |
| **Diagnostics when keys change** | Every re-plan produces a key diff (kept, added, removed, moved). It is compared with the size of the input change; "one parameter moved by two percent and seventy percent of keys changed" is a warning that names the parameter | Strong, and it works on real worlds, not only tests |
| **Preview of identity changes** | A plan-only dry run, then the matching ladder, before anything is realized: *412 matched, 3 target changed, 1 orphaned*. This is the first screen of the staged transaction | Complete. Cheap because planning is cheap |
| **Generator-version compatibility checks** | When a recipe's source changes, the engine re-plans stored inputs with the new code and compares the key sets against the stored plan results. Same keys: the change is output-only. Different keys without a key-epoch bump: flagged before it can be applied. So the engine **detects** a key-scheme change instead of trusting the author to declare one | Strong. Removes the "forgot to bump the epoch" failure from Pass 3 |
| **Identity contract in the manifest** | Small and declarative: for each child type, the key kinds allowed and whether children are interchangeable; which of pin, exclude and add are honoured. Everything else about the key scheme is *observed* by the tests above, not declared | Enough. A longer contract would be documentation that drifts |

**What the engine cannot do.** It can make keys stable; it cannot know whether a stable key still *means* the same thing. `house@seg3.L.2` is perfectly stable when the path is rerouted, and may now be on the other side of the village. That case is what fingerprints and the "matched, target changed" rung of the ladder are for. The prototype should include at least one scenario of this kind, since it is the residue no API design removes.

---

## 10. Recommendation and next steps

### 10.1 Recommendation

**Approach B: own core, Godot 4 as host, Rust core.** Borrow from approach C one thing only: use the host immediately, before any core exists, for the look experiment and for throwaway generator sketches, purely to learn. Keep approach A as a destination that stays reachable through the adapter, and do not build toward it.

| Decision | Recommendation | Confidence |
|---|---|---|
| Overall approach | B | High that A is too slow; medium-high that B beats C as the lasting structure |
| Host | Godot 4 | Medium-high, pending the probe |
| Core language | Rust | Medium |
| Recipes | Luau, inside the core | Medium, unchanged |
| Gameplay scripting | Host language | High |
| Build system | Full design of Pass 3 §3, delivered in stages; evaluation context as the only door from the start | High |
| Renderer | Host renderer; our shaders and post effects | Medium-high |

### 10.2 Order of work

1. **You choose the approach.** Everything below assumes B.
2. **Host probe** (small, throwaway). Five questions, each a yes or no:
   - Can a cell part (one merged mesh, one instance batch, one collider set) be replaced fast enough to feel live while dragging a handle?
   - Does a custom light function give the ramp-lit and soft-lit looks, with per-vertex data (colour sets, wind weights, edge flags) arriving intact?
   - Can an editor plug-in show a descriptor-driven inspector, a viewport handle and a dockable panel, with undo going through one stack?
   - Does the Rust binding hold up: build, hot reload in the editor, passing large arrays without copying pain?
   - Can a skeleton be posed procedurally each frame from extension code?
3. **Look experiment** in the host: the diorama, lit three ways. Needs your reference images.
4. **Identity prototype**: Pass 3 §7 Part A, reduced to the smallest thing that answers your core question, with the §9 mechanisms included. Headless, no host. Under B it is written in the core's language and, if it goes well, it is the seed of the core and not throwaway.

Steps 2 and 3 are independent of 4 and of each other.

### 10.3 Facts to confirm before committing (*unverified*)

- Godot: current state of its inverse-kinematics and skeleton-modifier support; custom post passes that can read depth and normals; whether scene nodes now carry persistent unique ids; cost of creating collision shapes at run time.
- The Rust binding for Godot: release maturity and how closely it tracks host releases.
- Rust bindings for mesh booleans, should those be needed early.
- The community heightfield-terrain extension, if approach C sketches use it.

### 10.4 Where I am least sure

1. **Chunk-replacement speed in the host.** If it is slow, runtime-generation games suffer and the editor feels sluggish. It is the one result that could push back toward A, or toward a custom mesh path inside the host.
2. **Two scene models in one editor.** I believe a combined outliner makes it coherent. I have not seen it done for exactly this case.
3. **Rust for the core.** The analysis points there once the host absorbs the libraries; the binding is the weak link.
4. **Whether B's "moderate–high" is low enough for you.** The infrastructure shrinks a great deal. The content work does not, in any approach.
