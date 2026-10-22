# Draw3D — verified implementation audit and the next bounded milestone

2026-10-09. Written from the repository as it stands (code outranks summaries). Companion
files: `Draw3D_Roadmap_Amendment_2026-10-09.md` (Deliverable D, roadmap) and
`Draw3D_Milestone_M1_Brief.md` (Deliverable D, implementation brief). The master roadmap
`Draw3D_Master_Roadmap_Final.md` is not changed. Nothing has been implemented.

Status words used in the "Verification" column:

- **editor** — implemented and usable through the editor by hand
- **text/command** — implemented, reached only through text files, host commands or scripts
- **core only** — the core (or host) has it; no authoring workflow
- **prototype** — exists as a demo or a stand-in
- **tested, not hand-verified** — covered by Rust tests or the scripted tour; not played
- **planned** — not implemented
- **unknown** — not verified in this audit

Evidence pointers are `file:function` or `file:line`.

---

## Deliverable A — verified implementation audit

### A.1 The architecture as it actually is

Three programs, one boundary each:

1. **Core** (`core/src`, ~5,900 lines, headless): `doc.rs` the document (nodes, generator
   state, edit records, modes Live/Frozen/Detached, an asset map `prefabs`, an inbox);
   `build.rs` incremental build with read-tracking; `kept.rs` the kept store (content the
   project owns: frozen, detached, hand-made, saved with the document); `state.rs` facts
   (runtime state, apart from the document); `plan.rs` the `Generator` trait, `Contract`,
   `Ctx`; `engine.rs::cmd` the edit vocabulary.
2. **Host** (`editor/host/src/lib.rs`, 1,848 lines, plus `shapes.rs`, `shapes/real.rs`,
   `shapes/land.rs`, `shapes/style.rs`, 3,400 lines): the core as a process on a local
   socket; ~50 commands (`lib.rs:937` `run`); history with a memory bound; staged edits;
   **all project content** (every generator, every mesh, every collider, every affordance
   line) lives here as Rust code; the written texts (moves, poses, acts, eyes, weather)
   are kept here by name but not read here.
3. **Godot editor/game** (`editor/godot`, ~45 scripts): draws cells, picks, sends
   commands; the motor, figure, acts, things, folk, weather, clock, hub, demos, tour. The
   same scene is the standalone game (`export.ps1`).

The decisions recorded in `PROGRESS.md` and the 21 review passes hold in the code: Rust
owns state and saves, Godot runs per frame, the document is the only thing edited, runtime
facts are apart from it, content is named by request hash and made off-thread.

### A.2 The audit table

| # | Capability | Intended end state (brief) | Actual implementation (evidence) | How a user reaches it today | Verification | Key missing pieces / risks | Depends on |
|---|---|---|---|---|---|---|---|
| 1 | Persistent world document, stable ids, provenance | Projects, named worlds, Save As, identity through regen and save/load | `doc.rs` `Document`/`Node`/`GenState`; ids derived from key paths (`base.rs:derive_id`); `Mode` Live/Frozen/Detached; `History` on detached nodes; `Document::save` bundles document + kept content (`doc.rs:551`); host `write`/`open` wrap it with the written texts as `world.json` (`lib.rs:537-570`) | Ctrl+S writes **one fixed file** `editor/world.json`; reopened on start. No New/Open/Save As, no named worlds | tests (core, acceptance, prototype ~76; host protocol 15); tour `world` part; hand: unknown beyond play | No project management at all; the walker is not in the document (asked of the host by `made`, stage 5 limits); written texts are in `world.json` but not in the document (no undo, no history) | — |
| 2 | Generators: create, edit, regenerate, dependencies, upgrade | User-authored generators; parameter control; regeneration visibility | `plan.rs` `Generator` trait with `Contract`, `inputs`, `plan`, `migrate`; registry of Rust structs (`land.rs:933` 12 playground generators + prototype ones); read-tracking so a changed input re-runs only readers (`eval.rs`); `cmd::set_param`, `connect`, `check_upgrade`, `upgrade` (`engine.rs`); host ops `param`, `generator` (swaps the prototype row generator only), `check_upgrade`, `upgrade` | **Nothing in the editor sends `param`** (only `tour.gd`). No way to create a generator, see its parameters, or connect inputs | tests; tour (`param` three times) | Generators are Rust code; parameters are invisible; "regenerate selected part" = undo or param change via script; changing a land parameter remakes all 144 ground patches (`PROGRESS.md` known gaps) | 1 |
| 3 | Overrides, pins, local edits on generated content | Local edits survive regeneration; different mechanisms for different intents | Closed `Op` vocabulary: `Set`, `Reroll`, `Pin`, `Add`, `Exclude`, `Attach`, `Tag` (`doc.rs:50`); records with `against` steps, marks, inbox; `review`/`accept`/`forget`/`dismiss` | Right-drag moves (Pin, staged then committed), `[ ]` turns, Delete excludes; `I` lists what needs a look. `set` (Op::Set on a child field) is in the host, not sent by the editor | tests (the bulk of the 57 prototype tests); tour; hand: moving and deleting have been used in demos | No per-child field editing in the editor; no visible "what will survive" preview; a record on an ordinal child is refused (`engine.rs:420`) and the editor does not explain it | 1, 2 |
| 4 | Freeze / thaw / detach / adopt / copy / sculpt | Explicit freeze/bake, detach, re-adopt, variants | `cmd::freeze`, `thaw`, `freeze_child`, `thaw_child`, `detach_child`, `detach`, `adopt`, `copy_node`, `sculpt` all in `engine.rs:531-834`; host exposes only `detach`, `freeze`, `thaw` (`lib.rs:1067-1080`) | **X = detach** the picked thing. Freeze/thaw: host command only, no key, no protocol test. Adopt, copy, freeze_child, sculpt: core only | core tests cover all; host freeze/thaw untested at protocol level; editor: detach by hand in tour only | `sculpt` is a stand-in: it only sets a `visual` text on a detached node (`engine.rs:726`), no geometry; detach is an all-or-nothing escape, not a local-edit mechanism | 1, 3 |
| 5 | Placing objects | Place any kind, including user kinds | `PLACEABLE` table of **18 fixed kinds** (`land.rs:874`); each placed thing is its own `Placed` generator node reading the land for height (`land.rs:897`) so it follows the land | Keys 1–9 and the palette (`palette.gd`); move/turn/delete/undo | editor; tour; demos | A new kind is a row in `PLACEABLE` plus Rust geometry and traits; a placed thing cannot have a field (e.g. `tint`) changed | 2 |
| 6 | Entity / kind definitions (geometry, collider, affordances, rig, behaviour words) | Persistent, editable definitions; new creatures and props without engine code | Hard-coded in the host: geometry `shapes.rs:shape_in` matches on archetype string and `form` number to Rust functions in `real.rs`; colliders `shapes.rs:solid` + `real.rs:prop_solid`; affordances and behaviour words `shapes.rs:traits` + `real.rs:prop_traits` (text lines: `offers`, `point`, `loose`, `someone`, `roams`, `moves NAME`…); rigs `real.rs:person_rig/sheep_rig/skiff_rig`. **The Godot side consumes all of this as text/bytes generically** (`things.gd`, `acts.gd`, `figure.gd`, `solid.gd`): no per-kind code there | None. "What a thing says of itself is the host's code" (`PROGRESS.md`, hub limits); the hub (F4) shows it and stops there | tested, not hand-verified (p10 etc.); hand: sheep and boat placed from the palette in demos | Creating a dog, a new prop, a different door = editing `real.rs`/`shapes.rs`. The consumer side is already data-shaped; only the producer is code. The document's `prefabs` asset map and `PrefabGen` (`gens.rs:181`, registered via `nested_registry`) exist but the host realizer makes nothing of a `part` archetype (falls to a green slab, `shapes.rs:489`) | 5, 7 |
| 7 | Editable geometry | Semantic creation → component/shape editing → direct mesh ops | A `Mesh` is a triangle soup (`shapes.rs:62`; bytes `MSH1` positions, normals, colours, indices, `shapes.rs:286`) built by primitives `face`, `block`, `limb`, `blob`, `corner`; no topology, no edit ops. The kept store persists arbitrary named outputs by hash (`kept.rs` `Stored::Made(Outputs)`, `put_hand_made`) and the project file carries them | None | planned; `sculpt` is text-only | No editable mesh representation, no edit ops, no editor tools. **Persistence of edited geometry is architecturally covered** (kept store + `Content.made`); the representation and the ops are not | 6 |
| 8 | Terrain | Sculpt (raise/lower/smooth/flatten, cliffs, ledges), paint, paths/rivers/coasts, local edits preserved through regen | `land.rs:Shape::height` is a pure function of `seed`, `amp`, `half`, `road` polyline, `clear`; `Land` exports `shape` (`land.rs:309`); 32 m patches of 32×32 faces; **every generator that stands something samples `ground.height(at)`** and `settle` re-reads it (`land.rs:266`), so dependents follow terrain changes already | None in the editor (parameters only via `param`, only from the tour) | tested, not hand-verified (land follows params in tour) | **Parameter-only. No sculpt layer, no paint, no feature lines beyond the one route polyline.** Any land change remakes all 144 patches. A persistent height-delta layer has an obvious home (inside the exported `shape`) and dependents would follow it without change | 2 |
| 9 | Styles | World and regional styles; live editing; style drives generation | Three styles in `styles.txt` (summer, autumn, highland); F2 cycles; kept as document assets `style.*` (`lib.rs:1039`), undoable, saved; regeneration scoped by asset group (`shapes.rs:made_from`) | **editor** (F2) + text file, live reload | tour; demo; hand partly | No regional style; editing only in the text file; no panel. Note: this is the one content system already in the document (undo, save, scoped regen) — the pattern to copy | 1 |
| 10 | Time of day, weather, falling things | Full environmental range, accumulation, interaction with the world | `clock.gd`, `weather.gd`, `weather.txt` (6 weathers, `falling` lines generalised: rain, snow, ash in tour); wind/sway/swell to shaders; weather as facts | T, N keys; `weather.txt` live reload | tour; demo; hand: wind in motion not yet seen | No accumulation, no `when wet` consequences, one sway formula; weather is whole-world | — |
| 11 | Movement: motor, moves, poses, sets | Assignable sets, authored moves, editor hub | `motor.gd`, `moveset.gd`, `move.gd`, `moves.txt` (four sets, wall grip/ledge hang/climb/wall jump/quick step/dodge written), `poses.txt` per body kind; host keeps texts (`written`, `text`, `read`) and the hub (`hub`) | Text files with live reload; F1 numbers; F4 hub shows and opens the file; `editor\ask.ps1` for programs | tour (`moves`, `climb`, `hub`, `written`); demos; **hand: almost none (PROGRESS "Waiting on the user"); controller never plugged in** | Which set a body has is a word in Rust `traits`; new words are code; texts are not undoable; nothing edited in the hub | 6 |
| 12 | Acts (Actor + Target + Action) | Authored interactions for everything | `acts.txt` (open, shut, pick_up, put_down, take, deliver, push, pet, board), `acts.gd`, `actbook.gd`; `offers ACT PLACE` and `point` lines in traits; `could` rule shared by offer and hub | Text file; E in play; hub | tour (`acts`, `hands`); demos; hand: not yet | What offers what is Rust; one offer taken; no choice UI; consequences limited to facts and five verbs | 6, 11 |
| 13 | Creatures and NPCs | Reusable perception/patrol/chase behaviours; creature studio | Sheep ×4 (`roams`, `shies`, `idles graze`, `offers pet back`), keeper (`goes HOUR`, `greets`), nav (`nav.gd`), folk (`folk.gd`) | Palette places a sheep or a boat | tour; demos; hand: not yet | Behaviour vocabulary is five words interpreted in GDScript; no perception/chase/attack; new creature kind = Rust | 6, 11 |
| 14 | Items, inventory, equipment | Equipment affecting movement/gameplay; inventory UI | Lantern, hatchet; `kept`, `held`, `does`, `lights`; list at bottom-left; 1–9, Q, R | play keys | tour; demo; hand: not yet | No stats, no equipment effects, no UI beyond a list | 6 |
| 15 | Gameplay rules, quests, events | "When event and conditions, do actions" as data | Facts (`state.rs`); trait words `when NAME`, `takes`, `pressed`; the errand is `Errand` generator + words | play | tour; demo | No rule format; every new rule is a trait word in Rust or GDScript | 6 |
| 16 | Physics | Rigid bodies, joints, forces, triggers, raycasts, shape casts | Godot physics via `loose.gd` (`loose KG`), plate (`pressed`), push/pull as acts; Godot raycasts used internally (`grasp.gd`) | play | tour; demo | No authoring of joints/forces/triggers; cloth absent | — |
| 17 | Runtime state save/load | Save-game apart from world | `State` facts, serial numbers, `save`/`load` slots, `document_changed` check | F5 / F9 | tour (`saves`); demos | Older saves report "edited since" after each stage (known) | 1 |
| 18 | Standalone export | Build verified on another machine; controller | `export.ps1`, `build/Draw3D-game`, `-Check` runs the delivery in the built game | `game.bat`, `run.ps1 -Game` | script-checked on this machine only | Never run elsewhere; no controller tried | — |
| 19 | Inspection / property inspector | Hierarchy, selection, properties, source/generated/authored status, params | Host `describe` returns kind, pos, yaw, size, origin (generated / pinned / authored / detached / attached), owner, path (`lib.rs:1363`); `nodes` lists nodes; `review` lists flagged/inbox/notes | One line of text at the top (`main.gd:say`): "picked: kind owner path (origin)"; `I` for the review list | editor; tour | **No panel, no params shown or editable, no hierarchy, no traits shown for the picked thing** | 1, 2 |
| 20 | Editor workspaces and asset library | Coherent workspaces; asset browser; variants; previews | None. Palette (18 fixed buttons), hub (read-only), feel panel (numbers), controls card | — | — | Everything | 19 |
| 21 | Program / AI access | One representation for humans and LLMs | `--beside` door; `engine-host --ask`; `editor\ask.ps1`; reads and writes texts and the hub, refuses world edits (`lib.rs:beside_may`) | terminal | tests (p15); tour | Cannot edit the world from beside; no key | — |
| 22 | Undo / history | Undo everything authored | Host versions with a 256 MB content bound; Ctrl+Z/Y | editor | tests; tour | Written texts (moves, acts…) are not undone (known gap) | 1 |
| 23 | Rendering and style controls | Modular paths, shadows, atmosphere | One project shader for everything since the weather; sky/fog/sun per `at HOUR` lines; lamps | styles.txt | demo | Not audited in depth here; no regional/material authoring | 9 |
| 24 | Automated verification | — | Rust tests: core 6, acceptance 13, host protocol 15, prototype 57 (~91); tour with `failed: 0` on 2026-10-08 23:22; 30 demos with `-Check` | `run.ps1 -Tour`, `cargo test` | — | Host protocol tests flaky when run together (known); the tour is not a usability proof | — |

### A.3 The working audit from the brief, checked

| Statement in the brief | Verdict |
|---|---|
| Headless core with documents, ids, read tracking, undo, pins/overrides, freeze/detach, upgrades, review marks, cancellable builds, bounded history | **Accurate.** Undo and bounded history are the host's (`lib.rs`), not the core's; the rest is core |
| Godot host/editor displays the world | Accurate; it also runs the whole game side (motor, acts, folk, weather) |
| ~18 object kinds, move/rotate/delete, undo/redo, several styles, time of day, weather, world saving, play mode with save/load | Accurate (18 in `PLACEABLE`, 3 styles, 6 weathers) |
| Moves, poses, acts, cameras, weather, styles partly authorable through text + live reload | Accurate, and more than "partly": every one of those is text-first with live reload; only the *consumer* vocabulary is code |
| Tour passed; most capabilities not hands-on tested; controller untried; standalone unverified elsewhere | Accurate. `PROGRESS.md` "Now" lists six demos waiting for the user's hand |
| ~29 demos | 30 names in `run.ps1` |
| No property inspector or workspaces | **Accurate** (one text line) |
| Host commands for params exist but the editor does not expose them | **Accurate** (`param` sent only by the tour) |
| Generator creation/editing outside the playground unverified | **Stronger than that: there is no such workflow.** Generators are Rust structs |
| Freeze/thaw not exposed through a complete editor workflow | Accurate: host commands without a key or a protocol test; detach has a key |
| Objects, geometry, affordances, behaviour hard-coded in Rust | **Accurate** for the producer side (`real.rs`, `shapes.rs`). The consumer side (Godot) is already generic text/bytes |
| Mesh-building operations unclear | **Resolved: not implemented.** Meshes are triangle soups built by Rust primitives; `sculpt` is a text stand-in |
| Terrain parameterized, no sculpt layer | **Accurate** |
| No asset library, creature studio, animation authoring | Accurate (poses are text) |
| Behaviour composition, rules, inventory, equipment, dialogue, quests, combat, cloth, genre demos incomplete | Accurate |
| Rendering/style controls limited | Accurate |
| No end-to-end proof that a user can create new content without engine code | **Accurate for things.** A user *can* already author a new move, act, pose, eye, weather or style without engine code (text). A user **cannot** author a new kind of thing, creature, prop, affordance set, terrain feature or rule |

### A.4 Can the architecture hold user-authored geometry and entity definitions as data?

Checked, not assumed:

- **Yes for persistence.** `Kept` stores named `Outputs` (mesh, collider, traits, rig, skin as bytes/text) by content hash; `Content.made` names them; `Document::save` writes them into the project file; `put_hand_made` exists (`kept.rs`, `engine.rs:340`). An edited mesh or a user kind's outputs can be persisted today without a new store.
- **Yes for scoped regeneration.** `Document.prefabs` is an asset map; a generator reads `asset/NAME` through `Ctx::read`, which is tracked (`build.rs:861,1093,1555`); a changed asset re-runs only what read it. The style system uses exactly this (`leaf_assets`). A kind definition stored as an asset gets undo, save and scoped regeneration for free.
- **Yes for consumption.** Godot reads traits, colliders, rigs and meshes as text/bytes with no per-kind code.
- **Missing, concretely:** (1) a realizer path from *data* to `Outputs` — today `shape_in` dispatches on archetype to Rust functions; (2) a `Placed` entry that names a kind by name instead of a `PLACEABLE` index; (3) an edit vocabulary for per-instance geometry/terrain edits — `Op` has no "replace content" or "height delta" op; `sculpt` requires detaching first and only sets text; (4) patch-local terrain invalidation.
- **Conclusion:** nothing structural stands in the way, and this audit names the four additions that do. Items 1–2 are a host change plus a small core-side asset convention; item 3 needs one new record op or a kept-content convention; item 4 is build/generator work on the land.

---

## Deliverable B — reconciled requirements and roadmap gaps

Mapping the brief to `Draw3D_Master_Roadmap_Final.md` (sections 1–10, principles A–N).

### B.1 Already stated adequately in the roadmap

- Persistent world, World Data vs World State, identity/provenance (1, A). Built.
- Movement/action/interaction as authored content (M, N). Built as text; the gap is "what a thing says of itself".
- Style → generation → rendering (5). Built for whole-world style.
- Generator dependencies, regeneration scope, anchors, validation (6, B, C, L). Core built; no authoring UX.
- Host vs Draw3D ownership (J). Held.
- The genre gauntlet and the parkour/delivery benchmark (10). Unchanged.

### B.2 Stated too vaguely to be actionable (resolved in the amendment)

1. **"Terrain sculpting / terrain painting"** (6) — resolved as: a persistent sculpt layer in the land's exported shape; brushes raise, lower, smooth, flatten, with strength and radius; stamps for cliff, ledge, slope; a paint layer for surface class; feature lines (path, river, coast, boundary) as polylines the land and dependents read. Staged across M2 and later. A height-function parameter panel is **not** terrain sculpting.
2. **"Generated geometry remains editable" / "progression to manual mesh editing"** (2, 6) — resolved as three tiers with a data model each: T1 semantic (a kind from words/parameters), T2 component/shape (a parts list the user adds to, removes from, moves, resizes, recolours), T3 direct mesh (an editable mesh asset with vertex/face move, extrude, bevel, revolve/sweep/loft, baked to `MSH1` on realize). T1+T2 in M1, T3 in M3.
3. **"Preserve authored overrides / regeneration respects authored changes"** (6, B, C) — resolved as six distinct mechanisms, each with its home: parameter change (generator params); protected local edit (a record: Pin/Set/Exclude/Attach, and a new content-override record for geometry); sculpt layer (terrain, and later per-kind shape deltas); freeze (approved subtree kept as-is); detach (independent node, re-adoptable); save-as-kind/variant (a new asset derived from an instance). Detach alone is not the answer and the amendment says so.
4. **"Authoring studios", "asset browser", "generator controls exposed"** (6, I) — resolved into a named workspace list with the minimum each needs (amendment §4).

### B.3 Missing from the roadmap

- **Content as data (new principle O):** a persistent kind-definition format (geometry recipe, collider, affordances, rig, behaviour words, style hooks) that the realizer interprets; the built-in kinds become examples of it over time.
- **Gameplay rules as data:** event/condition/action records (8 mentions "events/conditions" but not as authored data).
- **Project management:** New, Open, Save As, named worlds (1 says "save and load" only).
- **The written texts belong in the document:** so they are undoable and part of history (today they are beside it).
- **Per-instance field editing of generated children** in the editor (the host has `set`).
- **Editor must be usable without an LLM** is stated (N: one representation) but no editor-first workflow is listed for kinds or rules.

### B.4 Architectural foundations to establish early (before polished tools)

1. Kind definitions as document assets, realized from data (M1). Everything else that is "a new thing" depends on it.
2. A terrain sculpt layer inside the land's export, with patch-local rebuild (M2). Required from the beginning because every generator samples `ground.height`; a layer added later is cheap, a layer that lives outside the export would break the dependency model.
3. A content-override record op (or kept-content convention) for per-instance shape edits that survive regeneration without detaching (M3, but the op shape should be decided in M1's design note).
4. Written texts moved into the document (M2 or M4): small, removes a known undo gap, needed before an in-editor text editor is trustworthy.

### B.5 Safely postponed

Cloth; joints/forces authoring; dialogue/narrative graph; vehicles beyond the skiff; regional styles; accumulation weather; asset thumbnails; import/export; mini-games; large-world streaming optimisation; a controller remap panel; lock-on and further movement (ready in principle, not an authoring gap).

### B.6 Conflicting recommendations resolved

- *"Prototype → real case → extract"* (N) vs *"establish foundations early"* (guiding rule): both hold. M1 extracts the kind definition from the eighteen real cases that exist; it is not a framework ahead of cases.
- *Movement next* (PROGRESS "Next" 1–2: lock-on, vault, shimmy) vs *authoring next* (this brief): authoring first. The movement items are each a few lines of text on a motor that already exists and none of them changes what a user can author; they can be taken up any time the user's hands-on pass asks for them.
- *Text files as the authoring route* vs *the editor as the authoring route*: keep text as the representation (one representation for people and programs, N), put the editor on top of it (panel edits the same text, with live result and problems shown), and move the texts into the document so the editor's edits are undoable.

---

## Deliverable C — the next bounded milestone

### C.1 Candidates compared

| Candidate | What it proves | Rework risk if done first | Depends on |
|---|---|---|---|
| (a) Expose existing capabilities (inspector, params, freeze/thaw, Save As) | Visibility; parameter editing of Rust generators | Low, but **proves no authoring**: it exposes knobs on code-defined generators, which the brief warns against mistaking for authoring | — |
| (b) Data-driven kind definition with editable (T2) geometry, placed, edited, saved, reopened | **The transition from code to content**, with the editor panel and asset plumbing every later workspace reuses | Low: it uses the asset map, kept store and realizer seams that exist; no core data-model change | a minimal inspector/definition panel (a slice of (a)) |
| (c) Terrain sculpt layer | Direct world shaping; edit survival across regen of vegetation | Medium: needs patch-local invalidation first (a build change) and a brush UI; teaches nothing about kinds | the panel from (a)/(b) for tool settings |
| (d) Minimal editor workspace + reusable-content proof | Same as (b) when the content is kinds; otherwise a shell | — | (b) |

**Choice: (b) first, as M1, with the slice of (a) it needs (a picked-thing inspector, parameter display and editing for the owner node, Open/Save As).** Then (c) as M2 using the same panel and asset path. (b) before (c) because the kind panel and the "asset in the document, realized from data, undoable, saved" plumbing is the foundation M2's brush settings, M3's mesh assets and M4's rules all reuse, and because (b) is the capability the brief says is absent: a thing the user makes without Rust. (c) is not blocked by (b); it is sequenced after it to avoid building two panels.

Entity definitions must become data before meaningful editor work on *content*: an inspector over Rust-defined kinds can only show, never author. M1 makes the definition data, then the editor edits that.

### C.2 M1 — "Kinds as content": scope

**One sentence:** a user creates a new kind of thing (a prop, or a simple bodied creature) in the editor as text the document keeps, shapes it from parts, gives it affordances, places it, plays with it, saves the world, reopens it, builds the standalone game, and nothing in Rust or GDScript changed.

**In scope**

1. A **kind definition** text format (`kind NAME` entries, same idiom as `moves.txt`/`styles.txt`), stored as document assets `kind.NAME` (like `style.*`): parts (block, post, blob, limb, face: position, size, colour or style colour name), `solid` (auto box/post, or explicit lines in today's collider format), `rig none|person|sheep|skiff` (M1: `none` only for new kinds; a rig name is parsed and refused with a message), affordance and behaviour lines verbatim in today's `traits` vocabulary (`offers`, `point`, `loose`, `kept`, `held`, `lights`, `someone`, `roams`, `shies`, `moves`, `body`, `weighs`, `called`…), `like OTHER` for variants, `size w d` for the footprint.
2. **Realizer from data**: archetype `kind` reads `asset/kind.NAME` (tracked), builds `Mesh` with the existing primitives, collider, traits. A kind edit re-realizes only the leaves made from it.
3. **Placing user kinds**: `Placed` accepts a kind name; `placeable` lists built-in kinds and document kinds; the palette shows both.
4. **Host commands**: `kinds` (list, with problems), `kind` get / set / delete / duplicate (all edits to the document → undo, history, save). Also reachable from `--beside` for programs.
5. **Editor**: a Kind panel (list, text editor for the chosen kind, Apply, New, Duplicate, Delete, problems shown; placed instances update live); an **Inspector panel** for the picked thing (describe fields, origin, owner, path; owner node parameters listed and editable via `param`, labelled as generator parameters; traits text shown; "edit its kind" when it is a user kind); **Open… / Save As…** file dialogs for world files (the fixed `editor/world.json` remains the default).
6. **Persistence**: kinds travel in the document (Ctrl+S / Save As, reopen); `export.ps1` carries them unchanged (world.json already carries the document).
7. **Tests, tour part, demo**: see C.5–C.6.

**Non-goals (deferred, with reasons)**

- Direct mesh editing (vertex/face, extrude, bevel, revolve) — needs an editable mesh asset format and tools; M3.
- Terrain sculpting — M2.
- New rigs/bones for user kinds and skinning — needs a rig/skin authoring step; a user creature in M1 is unrigged (it moves as a body and stands still visually, like the keeper in edit view today). M3/M4.
- Changing built-in kinds' geometry (house, tree, sheep) through the panel — they stay Rust in M1; M1 proves the path, the migration of built-ins to kinds is M2+ background work.
- Rules/events as data; inventory UI; perception behaviours — M4.
- A visual (drag-handle) part editor — M1 edits parts as text with a live result; handles come with M3's tools.
- Moving the written texts into the document — M2.

### C.3 Implementation tasks (ordered, each with a targeted check)

| # | Task | Where | Check |
|---|---|---|---|
| 0 | Checkpoint: the repository is not under version control. `git init` and a first commit of the tree as it stands (user's call; the brief asks for a commit plan) | root | `git status` clean |
| 1 | Kind text format and parser: `shapes/kind.rs` — entries, `like`, parts, `solid`, `size`, `rig`, pass-through trait lines; problems as strings (never refuse the text) | host | Rust unit tests: round trip, `like`, a bad line reported with its number |
| 2 | Realizer: archetype `kind` → `Outputs` from the asset; `made_from("kind")` returns the kind's asset name so edits re-realize only its leaves | host `shapes.rs` | Rust test: a kind of three blocks → mesh triangle count, auto collider box, traits text verbatim |
| 3 | `Placed` takes `kind NAME` (string param) beside `what`; `placeable` answers `{builtin: [...], kinds: [...]}` | host `land.rs`, `lib.rs` | protocol test p16: place a user kind, describe it, undo, redo |
| 4 | Host commands `kinds`, `kind` (get/set/delete/duplicate) as document edits; allowed from `--beside` (read: `kinds`, `kind` get; writes: editor only) | host `lib.rs` | protocol test p17: set, undo restores, save/open round trip, problems reported |
| 5 | Palette: built-in and user kinds; "New kind" button opens the panel | `palette.gd` | tour `kinds`: palette lists the new kind |
| 6 | Kind panel (`kindview.gd`): list, TextEdit, Apply (sends `kind set`), New, Duplicate, Delete, problems; live update of placed instances (cells arrive as for any edit) | godot | tour `kinds`: a block added in the text changes the mesh's triangle count on the placed thing |
| 7 | Inspector panel (`inspect.gd`): describe fields; owner parameters via `nodes` + `param` with typed fields; traits text; "edit kind" | godot | tour: a land `amp` changed from the panel moves a tree's height; a placed kind's traits shown |
| 8 | Open… / Save As… (Godot `FileDialog`), default path unchanged; title shows the world file | `main.gd` | tour `world`: saved under a new name and reopened |
| 9 | Play proof: a user kind with `loose 25` + `offers push side` + `point side…` is pushed with E; one with `someone roams 6 shies 4 2 moves sheep body 0.3 0.8 offers pet back point back…` roams, shies and is petted | godot (no code; content) | tour `kinds` (acts `could` says the walker can push/pet it); demo `kinds` |
| 10 | Export: `export.ps1 -Check` with a user kind in `world.json`; the game shows it | `export.ps1` | script check |
| 11 | PROGRESS.md slice; roadmap amendment ticks; hub shows "where it is written" for a kind (the panel), closing the hub's known stop | docs, `hub.gd` | tour `hub` |

Regression policy: targeted tests per task (the protocol test and the tour part named); the whole tour and all suites once at the end of the pass, screen on, as established.

### C.4 User-visible outcomes

- Palette: a "New kind…" button and the user's kinds beside the built-in ones.
- A Kind panel (suggested key F6) where a kind is written, duplicated, applied, and its problems read.
- An Inspector panel (suggested key F7 or shown on pick) with the picked thing's origin, owner, path, parameters (editable), traits.
- Open… / Save As… and the world's file name in the title.
- Ctrl+Z undoes a kind edit; the placed things follow.

### C.5 Acceptance criteria

1. With the editor open and **no Rust or GDScript edited**, the user creates kind `crate2` (parts, `loose`, `offers push side`, `point side` ×4, `called`), places three, pushes one in play with E, saves as `editor/saves/worlds/kinds-test.json`, closes, reopens that file: the three stand where they were and push again.
2. The same for a bodied kind (`someone`, `roams`, `shies`, `offers pet back`): it roams, shies, is petted.
3. Editing the kind's text (a part resized) updates every placed instance within a second and nothing else is re-realized (host `check`: pieces made equals the number of kind leaves).
4. Ctrl+Z after Apply restores the previous shape of all instances; Ctrl+Y re-applies.
5. `export.ps1 -Check` passes with the kinds in `world.json`; the built game shows them.
6. The inspector shows a picked tree's owner parameters; changing `amp` on the land through it moves the trees (parameter editing, labelled as such).
7. `editor\ask.ps1 kinds` lists the kinds from a second program.
8. Tests: new Rust tests pass; protocol p16, p17 pass one at a time; tour part `kinds` passes; whole tour `failed: 0` at the end of the pass.

### C.6 Manual verification script (for the user)

*Superseded 2026-10-09 by the script in `Draw3D_Milestone_M1_Brief.md` ("Manual
verification"), written for what was built. Kept here as proposed.*

1. `editor\run.ps1`. Press the palette's **New kind…**. In the panel, paste:
   ```
   kind drum
     size 0.9 0.9
     part post 0 0.45 0 0.42 0.9 wood
     part block -0.5 0.45 0 0.06 0.8 0.06 trim
     solid auto
     loose 30
     offers push side
     point side 0 0.5 0.55 from 0 1
     point side 0.55 0.5 0 from 1 0
     point side 0 0.5 -0.55 from 0 -1
     point side -0.55 0.5 0 from -1 0
     called the drum
   ```
   Apply. Expect: no problems listed; `drum` appears in the palette.
2. Choose `drum`, click the ground three times. Expect three drums standing on the land.
3. Change `0.42` to `0.6`, Apply. Expect all three fatter at once. Ctrl+Z: thin again. Ctrl+Y.
4. Tab, walk to a drum: "E push the drum". Push it, let go. Esc back to editing.
5. Duplicate `drum` as `drum2`, change `wood` to `stone`, Apply, place one. Expect a grey drum beside the brown ones.
6. In the panel add a kind `critter` with `someone`, `roams 6`, `shies 4 2`, `moves sheep`, `body 0.3 0.8`, `weighs 40`, `offers pet back`, `point back 0 0.6 0 from 0 1`, `called the critter`, and a few parts. Place two, Tab: they stray; run at one: it bolts; crouch near one: "E pet the critter".
7. Save As… `kinds-test.json`. Close the editor. `editor\run.ps1`, Open… that file. Expect everything as left. Pick a drum: the inspector says kind `drum`, origin `generated`, owner `placed`, and lists its traits.
8. `editor\export.ps1 -Check`, then `game.bat`: the drums and critters are there.
9. In another terminal: `editor\ask.ps1 kinds`. Expect `drum`, `drum2`, `critter`.
10. Pick a tree. In the inspector set the land's `amp` from 6 to 9. Expect the hills to rise and the trees with them. Ctrl+Z.

### C.7 Checkpoint plan

- **C0** `git init`, commit the tree (if the user agrees).
- **C1** after tasks 1–2 (kind format and realizer, Rust tests green).
- **C2** after tasks 3–4 (host commands, protocol tests p16–p17 green one at a time).
- **C3** after tasks 5–8 (editor panels; tour part `kinds` green).
- **C4** after tasks 9–11 (play proof, export, docs); whole tour and all suites once; demo `kinds` handed to the user for the manual script.

### C.8 Deferred from M1 and why

| Deferred | Why |
|---|---|
| Direct mesh editing (T3) | Needs an editable mesh asset and tools; M1 proves the pipeline it would bake into |
| Terrain sculpt layer | Needs patch-local invalidation first; its own milestone (M2) on the same panel plumbing |
| Rigged user creatures | Rig + skin authoring is a separate step; unrigged bodies prove the behaviour/affordance side |
| Built-in kinds migrated to kind text | Large, mechanical; do it kind by kind in the background once the format has survived M1 |
| Written texts into the document | Separate, small, M2 |
| Visual part handles | Comes with M3 tools; text + live result is enough to prove authoring |
| Choice among offers, rules as data, inventory UI | M4 |

---

## Deliverable E — honest final status

**Genuinely working today (editor or play, by hand or by tour):** an island world generated, picked, moved, turned, deleted, undone, styled (3 styles), timed, weathered, saved as a world file and reopened; play mode with a motor, a rigged walker, doors, a delivery errand, items, a keeper, sheep, a boat, loose physics, cameras, wall grip and climbing; runtime facts saved and loaded; a standalone Windows build that runs the same on this machine; moves, poses, acts, eyes and weather authored as text with live reload and reachable by command from another program.

**Exists only in code, tests or demos:** generator parameters, freeze/thaw, per-child field edits, review/accept, upgrade (core and host, no editor path); `sculpt`, `adopt`, `copy_node` (core only); the prefab generator (registered, never realized); most play features have not been played by the user; the controller has never been plugged in; the build has not run on another machine.

**The critical missing authoring capability:** a user cannot make a new kind of thing. Every object, creature, prop, affordance set, rig and behaviour word is Rust in `real.rs`/`shapes.rs`; terrain is a parameter set; geometry is a baked triangle list; there is no inspector, no parameter panel, no project management. The consumer side (Godot) is already data-driven and the persistence side (kept store, asset map, scoped regeneration) is already in place, which is why the gap is narrower than it looks.

**Recommended next milestone:** M1 "Kinds as content" (C.2): kind definitions as document assets realized from data, placed from the palette, edited in a panel with live result, saved, reopened and exported, with a bodied creature and a pushable prop as the two proofs, plus the inspector and Open/Save As it needs. Then M2 terrain sculpt layer, M3 editable mesh assets, M4 rules as data.

**Evidence that Draw3D has crossed from runtime demo to authoring engine:** the manual script in C.6 completed by the user with zero changes to Rust or GDScript; the same kinds shown by `ask.ps1` to a program; Ctrl+Z over a kind edit; the kinds present in a standalone build; and, after M2, a hand-sculpted cliff surviving a regeneration of the woods around it.
