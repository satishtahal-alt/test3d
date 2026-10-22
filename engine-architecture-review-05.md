# Pass 5: Visual Target, Godot Feasibility Probe, and Whether Approach B Survives

Date: 2026-10-05. Follows Pass 4. Probe code and raw results are in `probes/`.

## 0. Short answer

- **Approach B still makes sense, and the evidence for it is now stronger.** None of the 22 reference images needs a custom renderer. Most of what makes them distinctive is generated geometry, palette, light and atmosphere.
- **The probe passed on every question it could test.** Replacing one generated chunk in Godot costs well under a frame for meshes and instance tables, leaves everything else untouched, and never gets saved into the host scene.
- **One real problem surfaced that was not visible on paper: triangle-mesh colliders.** They cost about 0.65 ms per 1,000 triangles, always on the main thread, and I found no way to move that cost off it. This is manageable with a rule (section 3.2), not a reason to leave Godot.
- **The Rust binding was not tested.** This machine has no Rust toolchain and no Python, so neither the Rust binding nor the official C++ binding can be built without downloading tools, which I did not do without your say-so.
- **The probe changed my recommendation on integration.** The core should talk to the host through one flat "change list", and should start as a separate process beside the editor. That removes the binding as a risk and makes the language choice independent of Godot.

## 1. What the reference images actually demand

I grouped the 22 images by the game they resemble. "Stock" means a Godot feature that only needs configuring.

| Family (images) | What makes the look | Where the work is |
|---|---|---|
| Tiny Glade (4) | Brick-by-brick walls, leafy blob canopies, dense grass, very soft warm bounce light, tilt-shift blur | Generators (bricks, ivy, canopies) and instance tables; soft global illumination is the one renderer-dependent part |
| Townscaper (2) | Module selection on an irregular grid, flat pastel colour, calm reflective water | Almost entirely the generator; one water shader |
| Dorfromantik (3) | Painted ground patches, cone and blob forests, tiny houses, ink-like edges | Generator plus instance tables; edge darkening is a small post effect |
| A Short Hike (5) | Low-resolution pixelated render, banded cliffs, cone trees, light shafts, toon water | Stock low-resolution scaling plus four short shaders |
| Planet of Lana (3) | Painterly rock with overhangs and arches, layered silhouettes, hanging vines, painted sky | Non-heightfield rock generation and painterly materials; framing is art direction |
| Sea of Solitude (2) | Coloured fog, foam crests as chunky geometry, rain, glowing accents | Stock fog and particles; foam is generated geometry |
| Stray (2) | Photoreal materials, hundreds of hand-made props, decals, many neon lights, wet reflections | Mostly authored content density, not renderer features |
| Low-poly campfire (1) | Flat-shaded shapes, one warm light, pixel particles | Stock |

Two observations follow.

- **Stray is the outlier.** Godot has the renderer features (many lights, decals, screen-space reflections, volumetric fog). What it shows is a quantity of bespoke props that no approach here produces procedurally. I am treating it as a mood and lighting reference, not a content target. Tell me if that is wrong.
- **The cheapest families share one recipe:** generated meshes with per-vertex or per-instance colour, a toon-style lighting ramp, a depth-based water shader, fog and a low sun.

### 1.1 Your feature list, sorted by where the difficulty lives

| Feature | Core (ours) | Host (Godot) | Difficulty is in |
|---|---|---|---|
| Procedural trees, vegetation | Skeleton growth, mesh, leaf cards or blobs, detail bands | Instancing, wind in a vertex shader, alpha-to-coverage | Generator quality; distant-tree stand-ins |
| Dense forests | Placement, instance tables per cell | MultiMesh per cell | Easy; measured below |
| Grass fields | Density and colour fields | Instanced blades per cell | Instance counts near the camera; needs its own small test |
| Rocks, cliffs, terrain | All geometry, including overhangs | Draws it | Generator; collider cost for non-heightfield shapes |
| Rivers, lakes, ocean | Surface mesh, flow direction, shoreline data | One water shader, stock reflections | Shader work; mirror-like reflections need an extra camera pass |
| Rain, snow, mist, dust, leaves, embers | Where and how much (fields) | Stock GPU particles, fog volumes | Configuration |
| Wind | A wind field as data | Global shader parameters | Essentially free |
| Atmospheric lighting | Time-of-day and mood presets as data | Stock sky, fog, glow, tone mapping, global illumination | Art direction; global-illumination quality ceiling |
| Stylized materials | Per-element colour and material ids | Custom shaders, low-resolution scaling, outline post effect | A handful of shaders per style |
| Variation and composition | All of it (planning half) | Nothing | Ours, in every approach |
| Structures, modular architecture | All of it | Instance tables, primitive colliders | Ours, in every approach |
| Creatures | Body plan, rig, skin weights | Skeleton, inverse kinematics, animation playback | Procedural animation; hard in any engine; defer |

### 1.2 Where custom technology is genuinely needed

Nowhere does the target need a custom renderer. These are the specific places stock Godot will not be enough:

1. **Soft global illumination at Tiny Glade quality.** Godot's real-time option for worlds that change is coarser and can leak light. This is the one place a custom renderer would visibly win. I recommend accepting the ceiling.
2. **Mirror-calm water reflections.** Needs a second camera pass; a known, small technique.
3. **Ink edges and similar full-screen effects.** A post effect reading depth and normals; small.
4. **Very dense near-camera grass.** May need a specialised path; unknown until tested.
5. **Distant stand-ins for generated trees and structures.** The core must emit several detail bands; Godot switches between them by distance. This is core work that Pass 3 already planned.

### 1.3 Look sketch

`probes/godot-probe/probe_look.gd` builds a terraced island, 6,000 trees, a lake, a few houses and drifting leaves from one script, using four short shaders and stock atmosphere. Output: [look_wide.png](probes/godot-probe/look_wide.png) and [look_shore.png](probes/godot-probe/look_shore.png).

- It is an untuned sketch: overexposed, with crude cone trees. It is not evidence of final quality.
- It is evidence of cost. Banded cliffs, shoreline foam, haze, light scattering and particles took about 150 lines of shader and settings.
- With real-time global illumination, volumetric fog and ambient occlusion on, a frame took 2.7 ms at 1920×1080 (295,000 terrain triangles, 18,000 tree instances). With those off, 1.9 ms.

## 2. Probe design

Environment: Godot 4.7.2 stable, Vulkan, Forward+ renderer, Jolt physics, RTX 4080 Laptop GPU, vertical sync off. Baseline frame time for the test world was 1.06 ms.

| Probe | File | Question |
|---|---|---|
| 1 | `probe.gd` | Cost and side effects of replacing one chunk's mesh, collider and instances, by size and method |
| 1b | `probe_b.gd` | Collider alternatives, sleeping bodies, whether collider builds can move off the main thread |
| 2 | `probe_c.gd` + `core-native/` | A native headless core driving Godot through a change list; determinism against a headless run |
| 3 | `probe_look.gd` | The look sketch above |

The test world is 16×16 cells of 32 m, each with a 2,048-triangle terrain mesh, a collider and 100 tree instances.

## 3. Probe results

### 3.1 Mesh chunk replacement

Median main-thread time to replace one chunk's mesh in place:

| Triangles | 512 | 2,048 | 8,192 | 32,768 | 131,072 |
|---|---|---|---|---|---|
| Mesh replace (ms) | 0.07 | 0.10 | 0.28 | 1.0 | 4.8 |

- The frames after a replacement stayed at baseline, except at 131,072 triangles (2–3 ms for one frame).
- Updating the existing mesh in place and assigning a new mesh cost the same.
- Driving the rendering server directly, with no nodes at all, cost the same as using nodes.
- Instance tables are cheaper still: 10,000 instances in 0.19 ms, 50,000 in 0.93 ms.

What happened to everything else, after 200 replacements of one cell:

- The other 255 cells were bit-for-bit unaffected (same objects, same GPU resources, same transforms).
- The replaced cell kept its node, its transform, visibility layers, material, metadata, an attached child light, and every outside reference to it.

The rule that follows: **the adapter keeps one host object per core unit for its whole life and only swaps its contents.**

### 3.2 Collider replacement — the one real problem

| Collider type | Size | Median (ms) |
|---|---|---|
| Triangle mesh | 2,048 triangles | 1.1 |
| Triangle mesh | 8,192 triangles | 4.4 |
| Triangle mesh | 32,768 triangles | 19 |
| Triangle mesh | 131,072 triangles | 85 |
| Height field | 65×65 samples (same area as 8,192 triangles) | 0.22 |
| Height field | 257×257 samples | 2.9 |
| Boxes on one body | 1,000 boxes | 0.85 |

- Triangle-mesh colliders cost about fifteen times more than the mesh they shadow.
- **The cost cannot be moved to a worker thread through Godot's interface.** Jolt builds the shape when the body enters the world, on the main thread. I tried building through nodes and through the physics server directly from a worker; both times the full 86 ms landed on the main thread.
- Height fields are about twenty times cheaper for the same area. Primitive shapes are cheaper again.

Design rules this forces on the core. They fit Pass 3, which already made the collider a separate named output:

1. Terrain that is a height field ships a height-field collider.
2. Structures ship primitive or convex colliders, not their render mesh.
3. Anything that truly needs a triangle-mesh collider (arches, overhangs) is split so that no single unit exceeds roughly 5,000 collision triangles, and uses a simplified collision mesh.

Untested: running physics on its own thread, which might hide the cost.

### 3.3 Physics behaviour across a replacement

- A raycast sees the new collider in the same frame.
- **A sleeping body is left hanging in mid-air when the ground under it is lowered.** It falls only when something wakes it.
- A body that ends up below raised ground falls through the world.

So the adapter must wake, and if necessary lift, dynamic bodies inside a replaced unit's bounds. This is a few lines, but it must be a stated adapter duty.

### 3.4 Localized regeneration, end to end

With the native core in a separate process (probe 2), one edit that adds a hill:

| Units affected | Core generates (ms) | Round trip incl. transport (ms) | Host applies (ms) | Frame containing the edit (ms) |
|---|---|---|---|---|
| 4 | 0.34 | 0.54 | 0.69 | 2.6–4.1 |
| 9 | 0.9–1.9 | 1.3–2.1 | 1.4 | 3.7–4.5 |
| 25 | 2.6–5.4 | 3.5–6.3 | 3.8 | 8.5–11 |
| 36 | 4.9–6.6 | 6.0–7.8 | 5.3 | 12–14 |

- Only the affected units were touched in the host, every time.
- All 256 host objects kept their identity across the twelve edits.
- The following frames stayed at baseline. Nothing became unstable.
- A small edit fits inside one frame at 240 frames per second; a 36-cell edit fits inside one frame at 60.

For scale, the same toy generator written in GDScript took about ten times longer than the native one and dominated the edit. Recipes must not run in the host's script language; Pass 3 already said so.

### 3.5 Persistence

Nodes the adapter creates without an owner are not written when the scene is saved: of 261 live nodes, the saved scene contained 2 (the root and one deliberately authored marker). Godot therefore cannot become the store of generated content by accident.

### 3.6 Not tested

- The adapter running inside the Godot editor rather than in a running game.
- Bridging Godot's undo history with the core's.
- The Rust binding and the official C++ binding (tools not installed).
- An exported game shipping the core beside it.
- Large worlds with chunks streaming in and out.

## 4. The authoring/runtime boundary

### 4.1 The boundary is one data format

Probe 2's core has no Godot headers and no Godot types. Its entire interface is:

- **In:** edits, as commands.
- **Out:** a *change list* — a flat block of bytes listing units to create, replace or remove. Each unit carries an opaque id from the core and its named outputs (render mesh, collider, instance table).

The same sequence of edits gave the same world hash (`b33b1987352d6d77`) whether the core was driven from Godot or run headless from a command line. The adapter, about 60 lines of script, holds a table from core id to host object and nothing else.

Because the change list is plain bytes, the same core can be reached three ways without changing it:

| Route | Used for | Depends on a Godot binding? |
|---|---|---|
| Separate process, local socket | The editor | No |
| In-process library | Shipped games that generate at run time | Yes, a thin one |
| No core at all, cooked Godot resources | Shipped games with frozen worlds | No |

Transport over the local socket ran at about 1.5 GB per second; a 54 MB change list (all 256 units at 8,192 triangles) arrived in about 36 ms. It is not the bottleneck.

### 4.2 What lives where

| Lives in the core | Lives in Godot |
|---|---|
| The world document: generator roots, inputs, edit records, frozen snapshots | Authored scenes and prefabs, as files the core refers to by stable resource id |
| Identity: ids, keys, fingerprints, the matching ladder | A table from core id to host object, rebuilt on load, never saved |
| Planning, realization, the Build cache and its traces | The realized meshes, colliders and instance tables, held in memory only |
| Live, frozen and detached state | Materials, shaders, lighting, atmosphere, particles |
| Undo history for world edits | Camera, gizmos, panels, viewport picking |
| Recipes and their sandbox | Gameplay scripts, physics, audio, animation, UI |
| Cooking to shippable resources | Export and packaging |

Three flows cross the boundary:

- **Picking.** Godot reports which host object and triangle or instance was hit. The core maps that to an element. Godot never works out identity.
- **Attachments.** A light or script attached to a generated element is an edit record in the core, addressed by key path. The adapter re-parents the host node when the unit is replaced.
- **Hand-authored content.** A zero-generation game is an ordinary Godot project. The core is not involved.

### 4.3 Problems Godot creates that were not apparent on paper

1. **Triangle-mesh collider cost on the main thread** (section 3.2). Handled by rules in the core.
2. **Physics bodies do not notice replaced ground** (section 3.3). Handled by an adapter duty.
3. **Two undo histories.** Godot's editor has its own. World edits must be undone by the core, with Godot's history holding only a marker that calls it. Untested; this is the item I would test next in the editor.
4. **Instance tables are culled as a whole.** One table per cell, as in the probe, is the unit of culling. Per-instance culling is not available.
5. **In-process native libraries on Windows are awkward to reload** while the editor has them open. The separate-process route avoids this.

None of these makes Godot the source of truth. All are adapter or core rules.

## 5. Rust or C++, after the probe

**What I could not test.** The machine has the Microsoft C++ compiler but no Rust toolchain and no Python. The Rust binding for Godot needs the Rust toolchain. The official C++ binding needs Python and its build tool. Both need downloads, so neither was built. The stand-in core is C++ only because that compiler was already present; it is not a vote.

**What the probe changes.** In Pass 4 the weak link for Rust was the community binding for Godot. With the change-list boundary:

- In the editor, the core is a separate process and needs no binding at all.
- In a shipped game, the binding carries a handful of functions that pass bytes. If the Rust binding disappointed, a small C++ shim over the same C interface would replace it without touching the core.

So the binding is no longer a reason to choose either language. The choice rests on which language is better for the core's own code: hashing, serialization, tree diffing, a parallel evaluator, a recipe sandbox. Pass 4's reasoning for Rust on that ground stands, still at medium confidence.

**Friction to expect with Rust, as far as I can state it without having run it:**

- One toolchain install (a few hundred megabytes). The C++ linker it needs is already here.
- Embedding the Luau recipe language from Rust goes through a wrapper library that compiles Luau's C++ source, so the C++ compiler stays in the build either way.
- Whether the Rust binding builds cleanly against Godot 4.7.2 is unknown until tried.

The identity prototype is the right place to settle this, because it is exactly the kind of code the core is made of.

## 6. Development time saved against capability lost

| Borrowed from Godot | Evidence from this pass | What is lost |
|---|---|---|
| Renderer, atmosphere, particles, post effects | Look sketch at 2.7 ms per frame with everything on; no reference image needs more | Tiny Glade-grade soft bounce light; per-instance culling |
| Physics | Colliders replace cleanly per unit | Triangle-mesh builds are main-thread; needs the collider rules |
| Editor shell, viewport, gizmos, asset import, export | Not exercised here | Two undo histories to bridge |
| Scene model for hand-authored content | Unowned nodes are never saved | Two scene models, as Pass 4 said |
| Gameplay scripting, audio, animation, UI | Not exercised here | Nothing found |

What stays ours, and what the probe says about it:

- The core ran headless, deterministic, and unaware of the host.
- Selective regeneration reached the screen in 1–14 ms for 4–36 units.
- The adapter stayed small and held no world state.

**Verdict.** Approach B holds, in a slightly sharper form: *the core is a headless program that emits change lists; Godot is one consumer of them.* The custom technology that remains is the core, an adapter, the editor panels, and roughly four to eight shaders per visual style. The largest remaining effort is unchanged from Pass 4: writing good generators and styles.

I would not lock B in permanently yet. The two things that could still overturn it are the editor-side adapter (undo, picking, gizmos) and near-camera grass density. Neither looks likely to.

## 7. Identity prototype (not started)

It stays a small, independent, headless program with no Godot in it, answering one question: can identity and user intent survive regeneration without becoming a special-case machine?

- It runs the 21 scenarios from Pass 3 section 7 Part A as automated tests.
- It includes the enforcement mechanisms from Pass 4 section 9: the single typed-key child call with no index constructor, key-handing layout helpers, duplicate-key hard errors, generated perturbation tests with key-set snapshots, key-diff diagnostics, plan-only preview with ladder counts, key-scheme change detection by re-planning stored inputs, declared identity contracts, and fingerprints alongside keys.
- Its pass condition is structural as well as functional: the 21 scenarios must pass through the general ladder, with a count of any scenario-specific branches reported. More than a few means the model has failed, whatever the tests say.
- It doubles as the language trial. Pass 3 planned to write it in both languages; with development time as a constraint I would write it once, in Rust, and fall back to C++ only if that goes badly.

## 8. What I need from you

1. **May I install the Rust toolchain on this machine?** It is needed both to check the Godot binding and to write the identity prototype in Rust. If you would rather not, I will write the prototype in C++ with the compiler already here, and the Rust question stays open.
2. **Is "separate process first" acceptable** for the editor, with in-process linking added only when a shipped game needs run-time generation?
3. **Which reference family is the first style target?** I suggest the A Short Hike and Dorfromantik family first because it is the cheapest to reach, then Tiny Glade.
4. **Is Stray a mood reference or a content target?** I have assumed mood.

## Appendix: running the probes

- `probes/run.ps1 -Project godot-probe -Scene res://probe.tscn` (also `probe_b`, `probe_c`, `probe_look`). Results are written beside the scripts as `results*.json`.
- `probes/core-native/build.bat` builds the stand-in core; `corecli.exe replay 16 32 12` runs it headless.
- The probes are throwaway measurement code, not the start of the engine.
