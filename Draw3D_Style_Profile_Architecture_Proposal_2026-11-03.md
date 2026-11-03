# Draw3D — Style profiles that change how things are made (proposal, 2026-11-03)

The brief: "Advanced Procedural Style Architecture Research and Proposal". Draw3D should
make worlds that look genuinely different, from realistic to painterly to exaggerated
cartoon to low poly. Each style should change how a tree, rock, house or road is
*constructed*, not only its colours and light. This document is the research and the
plan. **Nothing has been built.** Implementation waits for your review.

I worked from the code as it stands on `m2-stable-terrain` (edf1141). Where a claim
points at a file, the path is under `editor/`. The Godot facts are for **4.7.2 stable,
Forward+ on Vulkan**, checked against the 4.3–4.7 release notes and the docs.

Each recommendation is marked with one of four words:

- **Exists**: it works today.
- **Reuse**: keep it as it is and build on it.
- **Modify**: change what is there.
- **Build**: new work.

---

## 0. The short version

1. **The current system is a good foundation.** The style system already does several
   things right:
   - styles are one text format, saved into the world;
   - each group of a style is its own asset, so a change remakes only what reads it;
   - low poly already changes construction in four builders, with the per-design
     `styled` word and `form_on` by sort;
   - colliders and identity are never touched by a style.

   None of this needs replacing.
2. **What is missing is the link from a style into the generators.** `grown::grow`
   (`host/src/shapes/grown.rs:701`) takes no style. A grown tree is built the same
   way under every style, then decimated after the fact by `Mesh::formed`. Today a
   style can only *coarsen* a mesh. It cannot build something richer or differently
   structured.
3. **The mesh format has no texture coordinates** (`MSH1`: position, normal, colour,
   index; `host/src/shapes.rs:934`). Realism needs bark and rock textures laid along
   the surface, and leaf cards with cut-out pictures. That needs UVs. This is the one
   format change the plan needs.
4. **Proposal:**
   - Keep `styles.txt` and its groups.
   - Add one new kind of group, a **family profile** (`make.trees`, `make.rocks`,
     `make.built`, `make.land`, `make.water`, `make.paths`, `make.plants`). Each holds a
     few shared shape-language axes plus words only that family understands.
   - Pass the profile into each generator, which interprets it in its own way. Most
     often that means choosing a different *builder* (how a crown, a facade or a rock is
     put together), not just different numbers.
   - The design's recipe stays the person's: *what* the thing is. The style's family
     profile is *how* it is made in this world.
5. **First proof: realistic trees,** end to end.
   - UVs added to the format.
   - A bark and leaf-card surface set from Poly Haven (CC0).
   - A tree builder ported from EZ-Tree's algorithm (MIT).
   - Host-made LODs.
   - The same world under no style, low poly and realistic: low poly must come back
     exactly as it is today, and hand-shaped designs must stay untouched.
6. **Performance is planned before looks.** Realistic geometry is affordable only with:
   - host-made LODs (Godot does not make LODs for meshes built at run time);
   - per-region far meshes with visibility ranges;
   - MultiMesh grass;
   - shared materials, so Godot's automatic instancing keeps working.

---

## 1. The current style architecture

### 1.1 What a style is

| Piece | What it is | Where | Verdict |
|---|---|---|---|
| A style | `style NAME` lines in `godot/styles.txt` (96 styles: 16 low poly, 36 copies people made, `my_*`) | parsed by `host/src/shapes/style.rs:368` | **Reuse** |
| Inheritance | `like NAME` (lines replace by key); `takes GROUP from NAME` (one whole group from another style) | `style.rs:341`, `:387` | **Reuse**: the base for variants and intensity |
| Groups | build, wild, ground, village, growth, air, look (32 words), about, form | `style.rs:112` | **Reuse**; add family profiles beside them |
| Saved with a world | the style's name, plus each group as an asset `style.GROUP` in the world's document; undoable | `lib.rs:5918`, `style.rs:426` | **Reuse**: a world keeps working if the library changes |
| What remakes what | `made_from(archetype)` lists the style groups each sort of thing reads; the result cache checks what each build read | `shapes.rs:1875`, `core/src/eval.rs` | **Reuse**: the key to changing a style without remaking everything |
| Copies of your own | a library style is read-only; editing makes `my_NAME` (`own_or_copy`), shown under its parent | `styleview.gd` | **Reuse** |
| Finish | paint, settle, edges, harmony, warm/cool, ao | `godot/finish.txt`, `finish.gd` | **Modify**: it is not saved in the world and the host never sees it; fold it into the style |
| Moods | another style's look laid over the world's while a condition holds | `moods.gd` | **Reuse** (look only) |

### 1.2 How low poly changes geometry today (keep all of it)

- **`form` group** (`style.rs:501`): `form none|faceted|soft|angular`, `simpler`,
  `uneven`, `bolder`, `face_tint`, `form_on rocks trees plants built props ground water`.
- **After-pass** `Mesh::formed(sort, seed)` (`shapes.rs:563`). It runs *after* the
  collider is made:
  - merges corners by sort-sized cells, retrying if too much area is lost;
  - pushes corners with noise (`uneven`) and swells crowns and trunks (`bolder`);
  - gives each face its own tint and flat or creased normals.
- **Swapped builders** under a form:
  - trees → `real.rs tree_formed`;
  - roofs → `roof_formed` when `simpler` ≥ 0.3;
  - `land.rs rock` → three chunks;
  - `land.rs conifer` → stacked cones.
- **Draw-time only:** land facets (`world_view.gd facet`, 2/4/8 m), per-face ground tint,
  and a faceted water mode.
- **Overrides:**
  - the world's style;
  - `form_on` by sort;
  - the design word `styled no|faceted|soft|angular` (`kind.rs`, resolved in
    `shapes.rs:2050`);
  - one thing, through the Inspector (it gets a design of its own first).
- **The proof it is safe:** the demo's triangle count goes 2,111,152 → 242,611 under Low
  poly → 2,111,152 when the style is taken off.

**Verdict.** Low poly is **Exists / Reuse**. Under the new architecture, `form` becomes
the low poly entry of the shared shape language (§4.3). `Mesh::formed` stays as the
general "make it in fewer faces" interpreter. The four swapped builders become the
low poly builders in their families' builder tables. Low poly's output must not change
by a single triangle (an acceptance test in every phase).

### 1.3 Which generators shape geometry, and which only colour it

| Family | Where it is made | What shapes it | Does a style reach it? | Biggest gap for realism |
|---|---|---|---|---|
| Grown trees | `grown.rs tree()` :1015; 37 knobs, 14 presets | rich: form, trunk (height, taper, swell, lean, bend), boughs (levels, forks, angle, curve, wander), crown shape, foliage kind and density | after-pass only; `grow` takes no style | fixed tessellation (trunk 7–9 sides, lumps 6×4); no UVs; no leaf cards; no bark relief |
| Built-in trees and conifers | `real.rs tree` :546, `land.rs conifer` :3225 | only `variant` | yes: swapped builders | a handful of fixed shapes |
| Grown plants (grass, flower, fern, crop, bush, bamboo, fruiting, cactus, crystal, mushroom) | `grown/plants.rs`, `grown/cave.rs` | rich forms and knobs | after-pass only | no detail knob; no cards; no UVs |
| Built-in plants | `land.rs:3129` | `variant` | after-pass | fixed tufts and blobs |
| Rocks (grown) | `grown/rocks.rs:62`, icosphere plus cuts | facets, rough, pieces, detail (≤3 levels), edges | after-pass; built-in rock swapped | no strata, cracks or erosion; detail capped |
| Cliffs, crags, block sides | `land.rs` stamps `cliff`/`ledge`; `cut_side`; `land/rim.rs`; `build.rs:4185` crag | the land's heights | **no** (they read `style.ground` only) | no rock faces: a cliff is steep land, coloured |
| Terrain | `land.rs ground()` :2538 (32 m patches, 1 m grid), `Shape::hills` value noise, hand layers, open-world plan (`land/world.rs`) | heights; colour by steepness, shore and roads | draw time only (facets) | no surface material layers, no micro-relief, no erosion look |
| Buildings, walls, roofs | `shapes.rs` house :2253; `land/build.rs` structures (`makes` :4202, `shape` :4532) | rich: storeys, roof forms, window, door, fence and hedge forms, porches, `wear` | roof pitch and storeys from the style; flat lighting; `roof_formed` | no material relief (planks, stones, tiles); wall detail is merged, not redesigned |
| Roads, paths, railways | `build.rs roadway` :5660, `pathway` :6397; rails drawn in `train.gd` | lanes, width, paving | colours only | flat strips with no edges, ruts or verges |
| Water | `build.rs riverwater` :5080, dug ponds; the sea is a Godot shader | outlines | shader only (`water_tones`, faceted) | no shore geometry, foam bands or flow |
| Imported models | `models.gd` | none (the maker's own) | painted plain under low poly | cannot be remade; can only be chosen per style (§7.4) |
| Characters | glTF made offline (`charmake.gd`, `lowpolymake.gd`) | none | a colour tint (`figures`) | no style link (§9) |

### 1.4 Materials, shaders and the environment

- **One shader for every host mesh.** It is a string in `world_view.gd` (PAINT :221),
  plus GLASS and SEA. It takes albedo from vertex colour and wind from `1 − COLOR.a`.
  It can show an optional object-space triplanar texture, when a design says
  `texture FILE`; there are 43 jpgs. There are **no `.gdshader` files**, no normal or
  roughness maps, and no addons or GDExtensions. **Reuse** one shared shader with uniforms.
  **Modify** it to take material slots and texture sets (§4.5).
- **Environment** (`clock.gd picture()` :661, `_light()` :881): sky colours by the hour,
  sun, fog and height haze, volumetric fog for mist and golden hour, SSAO from `shade`,
  glow, tonemapper, grade and cast. SSIL, SDFGI and SSR come from *graphics quality*
  (`graphics.gd over()` :502), not the style. **Reuse.**
  **Modify**: let a style *ask for* effects, which quality can still deny (§4.6).
- **Post-process** `treat` (13 modes), film, ink outlines and the finish sheet.
  **Reuse** for the painterly and cartoon styles.

### 1.5 Saving and regeneration

- The world file is one JSON document. Generated meshes are **not** saved. They are
  remade from parameters and the style's assets into an in-memory cache. Grown designs
  *are* saved, as mesh text with their recipe line (`grown tree version=2 preset=… made=HASH k=v…`).
- A grown design shaped by hand is known by its `made=` hash, and regrowing it is
  refused without `by_hand: discard` (`lib.rs:4435`). **Reuse**: this is the protection
  the new architecture relies on.
- **Today a style change never regrows a grown design.** It only re-colours it and runs
  the after-pass. The new architecture changes this, deliberately and safely (§4.4).
- **A gap found:** the open-world generator grows `ow-FORM-s/-l` designs with
  `by_hand: "discard"` (`worldgen.gd:1806`), so a hand edit to them is lost when their
  numbers change. Not part of this work, but worth mending when the realistic world
  generator is touched.
- **Cost of a style change:** about 15 s for the first form change on town-wastes, then
  1–5 s. Meshes are rebuilt on every load.

### 1.6 What to keep, what to adapt, what is new

- **Keep:**
  - the text format, groups as assets, and `made_from` invalidation;
  - `like` and `takes`;
  - the `styled` design word and `form_on`;
  - the after-pass;
  - the four low poly builders;
  - colliders made before any shaping;
  - the one shared paint shader;
  - the per-hour air and the look words;
  - copies of your own.
- **Adapt:**
  - generators take a family profile;
  - the mesh format gains UVs and material slots;
  - the shader reads material slots;
  - the style can ask for GI and reflections;
  - finish words move into the style;
  - `form_sort` grows into "which family profile applies".
- **New:**
  - family profiles and their builder tables;
  - surface sets (texture sets per style);
  - host-made LODs and far meshes;
  - per-region style (late, optional);
  - the realistic builders themselves.

---

## 2. Godot 4.7 capabilities that matter

V = verified in the 4.7 docs or release notes. L = likely (long-standing, not
re-checked). U = unverified.

### 2.1 Geometry, LOD and instancing

| Feature | 4.7 | Use here | Caveat |
|---|---|---|---|
| ArrayMesh from packed arrays | V | the host's bytes already go this way (`world_view.gd _build`) | SurfaceTool and MeshDataTool are too slow in GDScript for big meshes; keep generation in Rust |
| Automatic mesh LOD | V, **imported scenes only** | — | meshes made at run time get **no** automatic LOD |
| LODs given by hand: `add_surface_from_arrays(..., lods)` | V | **the main route:** the host simplifies and sends index lists per LOD (it already does this for swaying plants, `Mesh::lighter`) | the LOD keys are only roughly distances; tune them by eye |
| `ImporterMesh.generate_lods` at run time | API V, run time U | fallback only | an import-time class; test in an exported game first |
| `lod_bias`, viewport `mesh_lod_threshold` | V | quality presets; hero things keep detail | — |
| Visibility ranges (begin/end, fade, `visibility_parent`) | V | far meshes per region: a merged region mesh at distance | alpha fades sort badly; use dither or hysteresis |
| MultiMesh (`set_buffer`, colours, custom data) | V | grass, small rocks and far trees, from host buffers | culled as one box: one per cell |
| Automatic instancing (Forward+) | V | many things of one design with one material are drawn as one | needs a shared material: per-thing variation must go into `instance uniform`s or vertex colour, never a material of its own |
| Occlusion culling (`ArrayOccluder3D` from points) | V | towns and cliffs: the host can send occluder boxes | little gain on open land |
| Precision | V | 4 km worlds fit in single precision | origin shifting only if jitter is seen |

### 2.2 Surfaces

| Feature | 4.7 | Use here |
|---|---|---|
| Vertex colour as albedo | V | today's whole palette system; keep it as the tint over textures |
| Texture2DArray in a ShaderMaterial | V | one terrain shader with N layers chosen by weights (host-made splat) |
| Triplanar mapping | V (already in PAINT) | rock and cliff faces without UVs; a stopgap for bark |
| Normal maps, roughness, AO, height (in a ShaderMaterial) | V | needed for realism: a 2-texture "albedo+height, normal+roughness" packing keeps memory down |
| `instance uniform` (16 per shader) | V | per-thing wear, moss and tint without breaking instancing |
| Global shader uniforms | V | wind, season, snow line and wetness, once per frame (needs a project-setting entry) |
| Alpha scissor or hash (leaf cards) | V | cut-out leaves stay opaque-pass, so they instance and shadow correctly |
| Backlight or transmission (custom light or `BACKLIGHT`) | L | light through leaves: important for realistic and painterly foliage |
| Specular occlusion (4.5) | V | cheap realism, worth turning on |
| Bent normals (4.5) | V | needs baked maps; little use for generated meshes |
| Stencil (4.5) | V | outlines, see-through cutaways; not needed here |
| CompositorEffect | V | outline, paper and ink styles at higher quality than the canvas `treat` |
| Ubershaders (4.4) | V | hide shader-compile hitches; every new material variant risks one, so keep variants few |
| Vertex shading (4.4) | V | retro styles |

### 2.3 Light and atmosphere

| Feature | 4.7 | Use here |
|---|---|---|
| SDFGI | V | the only real-time GI for generated worlds. It leaks and shifts cascades on fast moves. Offer it as a realistic option, never required. HDDAGI, its replacement, is an unmerged pull request (U, not in 4.7) |
| VoxelGI, LightmapGI | V | baking needed; LightmapGI bakes in the editor only. Not for generated worlds; VoxelGI perhaps for one interior |
| SSAO and SSIL | L | already wired; SSIL plus SSAO is the cheap GI for big outdoor worlds |
| SSR (rewritten in 4.6, half or full resolution) | V | water and wet stone |
| ReflectionProbe (octahedral in 4.6, cheaper) | V | interiors and the cabin |
| Volumetric fog and FogVolume | L | already used; limited range, so keep depth and height fog for distance |
| Tonemappers incl. AgX (4.4; adjustable 4.6) | V | AgX for realistic skies and sunsets, chosen per style (`tone agx` is already accepted) |
| Glow before tonemapping (4.6) | V | glow looks shifted in 4.6; retune per style |
| SMAA (4.5), TAA, FSR 2.2 | V | realistic foliage shimmers without TAA or FSR 2; FSR 1 is used today when scaling below 100% |
| AreaLight3D (4.7) | V | cabin windows and interiors |
| Directional shadows, 4 splits | L | 8192 already set; shorten max distance for big worlds, and turn off shadows from grass (Light3D shadow caster mask, 4.4) |
| Texture streaming, contact shadows | **no** | 4.8 only; streaming also needs imported `.stex` files |

### 2.4 Godot-native, Draw3D-specific, or native extension

- **Godot-native, use directly:**
  - ArrayMesh LOD lists, visibility ranges, MultiMesh and instancing;
  - Texture2DArray, alpha scissor, instance and global uniforms;
  - AgX, SSR, SSIL, SDFGI, volumetric fog, SMAA, FSR 2;
  - occluders.
- **Draw3D-specific, to build:**
  - all generation (Rust host);
  - LOD simplification and far meshes (Rust);
  - splat maps (Rust);
  - texture sets as style data;
  - the material-slot shader;
  - family profiles.
- **Native extension:** not needed. A GDExtension (godot-rust was probed and works) only
  pays if sending very large buffers over the socket proves slow. Measure first; it is a
  big bet that needs your approval.
- **Possible new dependency:** `meshopt` (Rust bindings to meshoptimizer, MIT) for
  simplifying LODs. `Mesh::lighter` exists but only handles swaying plants. Approval
  needed (§11, decision D2).

---

## 3. Open-source projects and techniques

Nothing was downloaded or run; repository pages, licences and papers were read. The
split matters: **Rust owns generation, Godot draws.** An addon that builds meshes in
GDScript would duplicate the host, so most are "learn from", not "install".

| Project | What it does | Worth studying | Godot fit | Licence | Recommendation | Conflict |
|---|---|---|---|---|---|---|
| **EZ-Tree** (dgreenheck, TypeScript) | recursive trees in levels: angle, children, length, radius, taper, twist, gnarl, force; leaf cards (single or crossed); LODs from one skeleton | the best single model for trees that change with a style | n/a (port it) | MIT | **Port the algorithm to Rust**, keep the notice | none once ported |
| **Weber & Penn 1995** | parametric species trees: stem levels, curve, split, down-angle, crown envelopes | realistic species shapes; `grown.rs` already resembles it | paper | ideas free | **Port the algorithm** (Blender's Sapling is GPL: copy nothing) | none |
| **Space colonization** (Runions et al. 2007) | branches compete for points in a crown envelope | natural crowns; with a blob envelope, the puff crowns of painterly and cartoon trees | paper | ideas free | **Port the algorithm** | none |
| proctree (BSD-3) | tiny recursive tree | cheap far trees | dormant | BSD-3 | learn; maybe port | none |
| **Tree3D** (JekSun97/gdTree3D) | C++ GDExtension tree generator: wind in vertex colour, seasons, growth | its wind encoding matches ours | Godot 4.5+, active | MIT | learn only | builds meshes in Godot: duplicates the host |
| **Gaea 2** | node-graph procedural generation, grid and tile | the graph-as-recipe idea for later | 4.4+, "early development" | MIT | learn architecture | GDScript generation |
| **Building Generator** (the nearest found is Home Builder, wikicode96) | walls, floors, roofs placed in the editor, LODs baked | its editing flow | 4.1+ | MIT | learn only | duplicates `build.rs` |
| **CGA split grammar** (Müller et al. 2006) | building mass → facade → floors → bays → window or door by split rules | a style-neutral facade grammar where the style chooses the terminal pieces and proportions | paper | ideas free | **Port the algorithm** (the house facade is already bay-based, `WalledFacade` :2132) | none |
| **Foliage3D** (caphindsight) | chunked quadtree, one MultiMesh per mesh per chunk, LOD map (mesh / impostor / none), jittered lattice seeded by position, threads keep the old chunk until the new one is ready | **the best match for scattering at scale** | Godot 4.7 | MIT (addon folder only) | **learn the architecture**; do not adopt | needs Terrain3D |
| ProtonScatter | scatter by a modifier stack | the stack as a scatter recipe | Godot 4, active | MIT | learn the UX | places instances in Godot |
| Spatial Gardener | paint props on any surface, octree LOD | painting UX | 4.2+, slowing | MIT | learn only | owns its data |
| Terrain3D | terrain plus a foliage instancer | **shader and instancer reference only** | active | MIT | reference only; **our terrain stays** | would replace our terrain |
| Zylann HTerrain | heightmap terrain with a grass layer | grass sway shader | 4.6+ | MIT | lift shader ideas | shader only |
| **GodotGrass** (2Retr0) | per-blade grass after the Ghost of Tsushima talk | blades widened toward the camera, bent by height; compute placement | Godot 4 | MIT | **reuse or port the shader** | low |
| Ghost of Tsushima grass (GDC 2021) and Horizon Zero Dawn placement (GDC 2017) | dense procedural grass; density maps plus rules decide what grows | "style picks the set, density decides the place" | talks | — | learn | — |
| **Sky3D** (TokisanGames) | day and night, moon phases, stars, clouds | a realistic sky | 4.3+, active | MIT (Milky Way texture needs credit) | reuse the shaders, or keep as reference | overlaps `clock.gd`/`clouds.gd`; drive it from our hours, don't let it own them |
| **Material Maker** | node-graph procedural textures, PBR export | authoring bark, rock and painterly texture sets **offline** | 1.4 on Godot 4.4 | MIT | **use as an offline tool**; port nodes as reference | don't embed |
| Waterways | river flow and foam baking | flow-map bake algorithm | Godot 3 (a .NET port for 4.2 has no baking) | MIT | port the algorithm later | C# port: avoid |
| WFC (mxgmn) | constraint tiling | street and roof tile layouts | C# | MIT (samples excluded) | later, via a Rust crate | none |
| **Poly Haven** | textures, HDRIs, models | realistic surface sets | — | CC0 | **use as sources** (already usable in Draw3D) | none |
| godotshaders.com | community shaders | toon, water, foliage | varies | per shader: CC0, MIT or GPL | CC0 and MIT ones only, after reading each page | skip GPL and Shadertoy ports |
| The Grove, Tree-It, Buildify, Blender rock addons | — | ideas only | — | commercial, proprietary or GPL (Buildify unverified) | skip the code | — |

**Rocks:** no strong open generator with a clean licence was found. The best route is
our own: extend `grown/rocks.rs` with plane cuts, strata bands, cavity colour and
erosion-like bevels (§5.3).

---

## 4. Target architecture

### 4.1 One idea

> **A design says what a thing is. A style says how things are made in this world.**
> The style speaks a small shared shape language plus words for each family. Each
> generator interprets them in its own way, usually by choosing a different builder,
> not just different numbers.

Today the style holds palette, light, look and a low poly `form`. The new layer sits
between the design's recipe and the generator:

```
design recipe (what: an oak, 9 m, spreading, 7 boughs)  ─┐
                                                          ├─► family generator ─► mesh (+UV, slots, LODs)
style family profile (how: realistic: fine bark,          │        │
  leaf cards, 3 branch levels, gnarl 0.4, …)             ─┘        └─► after-pass (low poly formed, if asked)
surface set (bark_oak, leaf_cards_broad, …) ─────────────────────► Godot material slots
air + look (light, fog, GI asks, tone, post) ────────────────────► clock.gd / graphics.gd (as today)
```

### 4.2 Layers of a style profile

A style keeps its existing groups and gains these. Each is a separate asset, so
`made_from` still remakes only what reads it.

| Layer | Group(s) | Status |
|---|---|---|
| Identity | `about` (says, after, tags, yours) **+ `version N`, `variant_of NAME`** | **Modify** |
| Palette | build, wild, ground (role colours), limit_* | **Reuse** |
| Shared shape language | **`shape`** (§4.3), which absorbs `form` | **Build** (form kept as is) |
| Family profiles | **`make.trees`, `make.plants`, `make.rocks`, `make.land`, `make.built`, `make.paths`, `make.water`, `make.figures`** | **Build** |
| Surfaces | **`surfaces`**: which texture set fills each material slot, texture size, detail strength, weathering | **Build** |
| Growth and composition | growth (pines, woods, meadow, trees, strewn) | **Reuse** (it already chooses designs per style) |
| Light and air | air (`at` hours, day, moon) | **Reuse** |
| Picture | look (tone, grade, shade, haze, distance, rim, outline, treat, …) **+ the `finish.txt` words** | **Modify**: finish moves in |
| Rendering asks | **`render`**: `gi none|ssil|sdfgi`, `reflections none|ssr|probes`, `ao`, `aa` | **Build**. Quality caps every ask (§4.6) |

### 4.3 The shared shape language

A few axes every family understands. Each family maps them to its own construction.
They are numbers from 0 to 1, so blending between styles and intensity sliders are
simple:

| Axis | Meaning | Low poly today | Realistic | Cartoon (exaggerated) |
|---|---|---|---|---|
| `facets` | 0 smooth surfaces ... 1 flat faces | 1 (faceted) | 0 | 0 |
| `round` | how much shapes swell toward blobs and soft bevels | 0.3 | 0.1 | 0.9 |
| `uneven` | irregularity and asymmetry | 0.35 | 0.5 | 0.15 |
| `exaggerate` | proportions pushed: fat trunks, big heads, squat rocks (today's `bolder`) | 0.35 | 0 | 0.85 |
| `detail` | how many secondary parts (twigs, cracks, trim, pebbles) | 0.2 | 0.9 | 0.3 |
| `resolution` | tessellation: sides, rings and subdivisions | 0.15 | 0.8 | 0.6 |
| `edges` | sharp ... bevelled ... worn | sharp | worn | bevelled |
| `wear` | weathering, chips, moss, dirt in creases | 0 | 0.6 | 0.1 |

The existing `form`, `simpler`, `uneven`, `bolder`, `face_tint` and `form_on` lines stay
valid. They are read as the low poly settings of this language (`form faceted` →
`facets 1`; `simpler` → low `resolution` and `detail`; `bolder` → `exaggerate`). Old
worlds load unchanged. **Low poly keeps its own after-pass**: `facets > 0` with `form_on`
still runs `Mesh::formed` exactly as today.

### 4.4 Family profiles: the specialised part

Every family has its own words, a **builder table**, and a function in the host:

```
fn profile_for(family, style_lines) -> FamilyProfile      // parse, with defaults
grow(f, values, name, &FamilyProfile)                     // generators gain one argument
```

A profile is three things:

1. **A builder choice.** This is the structural change, the part a material can never
   do. Examples:
   - trees: `crown lumps|cards|clusters|puffs|cones`;
   - rocks: `rock blob|cut|strata|boulderfield`;
   - built: `facade plain|timbered|masonry|stylised`.

   Low poly's `tree_formed`, `roof_formed` and the formed rock and conifer become the
   `lumps`/`cones`/`slab`/`chunks` builders in these tables.
2. **Tessellation and detail.** These follow from the shared axes but can be set
   directly: trunk sides, branch rings, leaf-card count, rock subdivision, plank count
   on a wall, path edge stones.
3. **Recipe biases.** Multipliers or offsets on the design's own knobs that a style may
   push, for example realistic `gnarl ×1.6` and `taper +0.1`, cartoon `trunk ×1.8` and
   `crown ×1.3`. Biases never replace the person's numbers; they lean them. A design can
   refuse biases (`styled as_made`, the existing `styled no`).

Example family profile words (illustrative, to be settled in Phase 1):

```
make.trees   crown cards   levels 3   twigs 0.6   bark ridges 0.5   leaf cards 220 crossed   gnarl 1.6   lean_to_light 0.3
make.rocks   rock strata   cracks 0.5   bevel worn   sink 0.3   pebbles 0.4   cavity 0.6
make.land    relief 0.4   cliffs faces   scree 0.5   verge 0.6   layers grass_lush dirt rock_granite sand_wet
make.built   facade timbered   planks 1   stones 1   tiles 1   trim_depth 0.6   sag 0.2
make.paths   edge stones|verge|kerb   ruts 0.4   profile 0.05
make.water   shore foam_band   reeds 0.5   flow 1   depth_tint 1
make.figures proportions real   shading pbr   set realistic
```

**Why not one generic parameter list?** A tree, a facade and a river share almost
nothing structurally. The shared axes give a style a single coherent *intent*. The
family words carry what only a tree, a rock or a facade means. Each family adds words
without touching another's.

**Regrowing under a style (Modify).** Today a style never regrows a grown design. Under
the new architecture:

- A grown design's realised mesh is grown at realise time from its recipe plus the
  world's family profile. The kept mesh text is still the design as grown under no
  profile, so the library and copies keep working.
- The cache key includes only the family profile's own asset (`style.make.trees`), so a
  rocks change remakes no tree.
- **A hand-shaped design** (its `made=` hash does not match) is **never regrown**. It
  gets only colours, surfaces and the after-pass, exactly as today.
- `styled as_made` keeps any design as grown, whatever the style.

### 4.5 Surfaces: material slots and texture sets

- **Mesh format `MSH2` (Build).**
  - Adds an optional UV block and a per-vertex *material slot* byte (bark, leaf card,
    rock, plaster, wood, roof tile, metal, glass, ground). Slots could also be per
    triangle, if surfaces are split.
  - `MSH1` is still read, as slot 0 with no UV.
  - Generators write UVs where they know them: along the bark ring and up the trunk,
    on the leaf card, on planks and tiles. Where they do not, triplanar stays.
- **Shader (Modify).**
  - PAINT gains texture arrays per slot: albedo+height and normal+roughness, packed
    two per layer.
  - Vertex colour stays the palette tint over the texture, so every existing palette,
    variation and `limit_*` keeps working.
  - Alpha scissor for leaf cards.
  - Each style chooses what its slots hold; still one shader, a handful of variants.
- **Surface sets (Build)** are style data, not code:

```
surfaces   size 1024   detail 1   wear 0.5
  bark       bark_pine  bark_oak
  leaves     leaf_cards_broad  leaf_cards_needle
  rock       rock_granite  rock_moss
  ground     grass_lush dirt_path rock_scree sand_wet     (terrain layers, in this order)
  plaster    plaster_white   wood  planks_weathered   tiles  roof_clay
```

  Texture sets live in a library folder (`godot/textures/sets/NAME/`), each with a
  `set.txt` saying where it came from and its licence (Poly Haven CC0, Material Maker,
  our own). Low poly's set is "none": vertex colour only, as today.
- **Terrain layers (Build).**
  - The host sends a splat (4–8 weights per patch corner) from slope, height, wetness,
    roads and paint.
  - Godot blends the layers with triplanar on steep faces and a height-based blend at
    edges.
  - The ground's walkable surface is unchanged.

### 4.6 Environment: the style asks, quality decides

- The style's `render` group *asks* for GI, reflections, AO and AA.
- `graphics.gd` already layers trying, player, game and quality. It grants each ask up
  to the current quality and says in words what it held back (for example "Realistic
  asks for SDFGI; held back at Medium").
- AgX, sky, fog, haze, volumetric mist, sun and fill stay in `air` and `look`, as today.
- This keeps a realistic world playable on Low quality without a second style.

### 4.7 Saving, versioning, duplicating, applying

- **Saved as today:** the world keeps the style's name plus every group as an asset.
  Family profiles and surfaces are more such assets. A world does not depend on the
  library staying the same.
- **Versioned:** `about` gains `version N`. A world records the version it copied. When
  the library's style is newer, Window › Styles says so and offers **Update** (per group,
  with Keep or Put back) or **Keep mine**. Old words are migrated by the parser, never by
  rewriting the world unasked.
- **Duplicated and shared:**
  - `own_or_copy` stays.
  - **Export** writes one `.style.txt` (the style's flat lines plus a list of texture
    sets it needs).
  - **Import** reads one, with the texture sets copied into the library.
- **Applied to an existing world:**
  - the style change is one undoable edit (as now);
  - layout, placement, colliders and hand edits are untouched;
  - only what reads a changed group is remade.
- **Applied to a new world:** the New world window's style choice (it exists) now
  includes the realistic family.

### 4.8 Determinism and seeds

- **Placement and construction use different seed streams.** Where a thing stands, its
  turn and its design choice come from the maker's or world's seed, as today.
- How a design is built under a profile uses the design's own seed hashed with the
  *family* only, never the style's name. So switching styles changes a tree's shape but
  not where it stands. Switching back gives the identical mesh, a cache hit.
- **The growth group remains the one style lever that changes *which* designs a wood
  is made of.** That is deliberate. Its pick is seeded by the maker, so the same style
  always gives the same wood.
- **Colliders:**
  - Low poly still makes the collider before the after-pass.
  - Regrown trees and rocks under a profile make their collider from the profile's
    trunk and body (a realistic oak's trunk is still where the oak is).
  - The collider shape may change slightly between styles. Paths, nav and placement
    use the land and footprints, which do not change.
- **Hand edits:**
  - Terrain tiles, kept structure pieces and hand-shaped designs are never touched by a
    style change.
  - Open-world regions that are locked stay locked.
  - Mend the `worldgen.gd:1806` discard while there.

### 4.9 Per-region style (late, optional)

Today a style is the world's. Region styles ("the swamp is painterly, the town
realistic") would need the realiser to look up a style by place: each region asset
names a style, and leaves read their region's groups. The `made_from` dependency model
already supports that, with one more key. Proposed as Phase 9, only after the realistic
proof. Until then, the existing per-design `styled` override is the escape.

---

## 5. Realistic: the first proof

### 5.1 The test environment: `realtest`

- **Where:** a 512 m world, made through the editor's own makers and kept in the
  library, so it is authorable (demos must be authorable).
- **Terrain:** rolling hills, one valley, a 25 m cliff band with scree below, a lake with
  a shore and reeds.
- **Trees:** three varieties (a broadleaf oak type, a conifer, a birch type), in woods
  and on their own at a forest edge.
- **Undergrowth:** bushes, ferns, dense grass in a clearing, wildflowers.
- **Rocks:** boulders, a rock outcrop, the cliff face itself.
- **A cabin:** timber walls, a stone chimney, a shingle roof, a porch.
- **Paths:** a dirt path with ruts and verges, and a gravel track to the cabin.
- **Light:** a late-afternoon hour, AgX, haze, a morning-mist variant.
- **Views:** the same four camera views under no style, Low poly and Realistic, one
  contact sheet each.
- **Demo:** `-Demo realtest` in demos.txt.

### 5.2 What exists and what each part needs

| Part | Exists today | Realistic needs | Size |
|---|---|---|---|
| Hills, slopes | value-noise hills, brushes, open-world plan, 1 m grid | terrain material layers (splat, texture arrays, triplanar); macro and micro relief from the host (light erosion-like channels in `make.land relief`); far land merged (exists) | **Build**, medium |
| Cliffs | steep land coloured as cliff | cliff **faces**: rock-strata geometry laid on steep bands (a structure the land maker places, made by the rock family with `rock strata`), scree at the foot | **Build**, large |
| Trees ×3 | grown trees (37 knobs), conifers | a realistic tree builder: 3–4 branch levels, twigs, crossed leaf cards in clusters, bark UVs, root flare, 8–16 trunk sides near, LODs to an impostor far | **Build**, large (the PoC) |
| Bushes, ferns | grown families | leaf-card builders (fern fronds as cards, bush as card clusters on a twig skeleton) | **Modify**, medium |
| Grass | grown grass designs, meadow strewing, turf shader | dense blade grass by MultiMesh per cell, host buffers, GodotGrass-style shader (blades widened toward the camera, bent by wind and treads) | **Build**, medium |
| Rocks | icosphere rocks with cuts | strata, cracks, worn bevels, sinking into the ground, pebbles round them, cavity colour; rock textures | **Modify**, medium |
| Cabin | structures with timber looks, roof forms, porches, `wear` | planks and log courses as geometry; shingles or tiles as rows (UV'd); stone chimney of stones; sag and wear; window depth | **Modify**, medium–large |
| Paths | flat `pathway` strips | a cross-section profile (crowned, rutted), verges, edge stones, dirt splat on the land | **Modify**, small–medium |
| Water, shore | flat sheets, sea shader, water_tones | shore foam band, wet sand splat, depth colour from the land below (exists in part), reeds strewn by the shore, SSR | **Modify**, medium |
| Light and air | full hour system, AgX accepted, volumetric mist | a realistic style's hours; `render` asks (SSIL, optional SDFGI, SSR); FSR 2 or TAA for foliage | **Modify**, small |

**Materials alone will not get there.** Compare the realistic and low poly versions
of the same world: the differences are the twig layers, leaf cards, cliff faces, plank
courses and grass blades. All of those are geometry and builder work.

### 5.3 How each realistic builder works (summary)

- **Trees.** EZ-Tree's level structure, ported to Rust, inside the existing tree family:
  - the 37 knobs map onto it (levels, forks, angle, curve, wander already exist);
  - add per-level twist, gnarl, a growth pull (toward light, down for weeping), twig
    levels, leaf-card clusters and root flare;
  - space colonization as an alternative crown builder (`crown colonize`) for dense
    natural crowns;
  - bark UVs run round and up; leaf cards carry slot `leaves` with alpha scissor;
  - normals of cards bent toward the crown's centre, so crowns shade as a volume
    (painterly styles use more of this).
- **Rocks.** From the icosphere:
  - plane cuts for fractured faces;
  - strata (displace along a style's layer direction, slightly stepped);
  - worn bevels;
  - cavity darkening into vertex colour;
  - a sinking base that meets the land;
  - a pebble ring.
- **Cliff faces.** The land maker finds steep bands (slope above the style's limit) and
  lays rock-strata pieces along them. These are a kind of structure with colliders off;
  the land's own collider stays. The land below gets a scree splat.
- **Cabin.** A finer facade grammar on top of the bay facade:
  - wall → courses (logs or planks) → openings (window frame with depth, sill) → trim;
  - roof → rows of shingles or tiles, with UVs.

  The style's `make.built` chooses the terminal pieces. Low poly keeps `roof_formed` and
  its flat walls.
- **Grass.**
  - The host sends per-cell instance buffers: positions from the meadow's own seed,
    density from paint and splat.
  - Godot draws one MultiMesh per 32 m cell with a blade mesh of 3–5 segments.
  - Distance thins the grass, and far land carries a grass-tinted splat.

---

## 6. Low poly in the new architecture

- `form`, `simpler`, `uneven`, `bolder`, `face_tint` and `form_on` keep their meaning.
  They are parsed into the shared axes with exactly the values they mean today.
- The four low poly builders become entries in their families' builder tables
  (`tree_formed` → `crown lumps`, conifer → `crown cones`, rock → `rock chunks`,
  `roof_formed` → `roof slab`).
- `Mesh::formed` remains the after-pass for anything with `facets > 0` and its sort in
  `form_on`.
- The surface set is "none" (vertex colour), so low poly never loads a texture.
- **Acceptance in every phase:** `-Demo lowpoly -Check` passes. Its triangle counts are
  unchanged: 2,111,152 → 242,611 → 2,111,152 on town-wastes.

---

## 7. More styles and reusable presets

### 7.1 A style is a profile, not a switch

- There will be no `if style == "botw"`.
- A reference game becomes an **original** style by naming what we see in it, then
  setting the shared axes, the family builders, the surfaces, and the light and look.
- The `after` line stays honest ("after painterly open-world games: their soft colour
  fields, not anyone's art"), as the library already does.

### 7.2 Four reference directions translated

| Reference | What we see | Parameters (axes, palette, light) | Needs different builders or techniques | Needs assets |
|---|---|---|---|---|
| **Realistic** (after Red Dead Redemption 2) | dense, true-scale detail; materials that read as wood, bark, mud; long atmospheric depth; heavy, warm or overcast light | facets 0, detail 0.9, resolution 0.8, uneven 0.5, wear 0.6; AgX; haze and distance; SSIL or SDFGI asks | realistic tree, rock and cliff builders; blade grass; plank and shingle facades; rutted paths; terrain layers | PBR texture sets (Poly Haven CC0), leaf atlases |
| **Painterly stylised** (after Breath of the Wild) | soft colour fields, simplified masses, brushed grass, crowns as rounded clusters, rim light, distance turning blue | facets 0, round 0.6, detail 0.4, exaggerate 0.2; low-contrast grade; rim; strong distance tint; `treat paint` light or none | `crown puffs` or `clusters` with sphere-bent normals; a brush-stroke grass card; a soft-ramp light function (a `shading` mode) | brush-stroke texture sets (Material Maker, our own) |
| **Atmospheric stylised realism** (after Ghost of Tsushima) | fields of one vivid colour (red leaves, white pampas, gold grass), wind made visible, deep volumetric light, particles | realistic axes with `detail 0.7`; a strong limited palette in `wild`; heavy mist and volumetric hours; strong wind in `weather.txt` | dense blade grass with strong sway; leaf-card trees whose colour comes in large masses (vary.rs roles by region); falling-leaf particles (exist) | leaf atlases, grass blade textures |
| **Exaggerated cartoon** (after Super Mario Odyssey) | big rounded shapes, saturated colours, clean bevels, chunky proportions, glossy toy materials | facets 0, round 0.9, exaggerate 0.85, detail 0.3, resolution 0.6, edges bevelled; saturated palette; low roughness, clearcoat (energy-conserving in 4.7) | `crown puffs` (smooth sphere-blobs), `rock blob`, a `stylised` facade with rounded trims, smooth subdivision on built pieces | few: vertex colour plus a gloss ramp |
| **Low poly** (today) | as §6 | as §6 | the existing builders | none |

### 7.3 Variants and intensity

- **Variants** are `like` styles, as today: `realistic`, `realistic_painterly`,
  `stylised_realism`, `realistic_overcast`.
- **Intensity** blends a style's numbers with its parent's (or with "as made") by one
  slider from 0 to 1:
  - numbers in between are interpolated;
  - a builder choice flips at 0.5;
  - the blend is written as `intensity N` in `about`, not as new copies of every value.
- **Takes** already composes styles ("Realistic's trees with Cartoon's palette").
  Extend it to the new groups: `takes make.trees from realistic`.

### 7.4 What parameters cannot do

- **Imported models** keep their geometry. A style can only *choose* between model
  variants tagged for it. Allow a design's model line to name per-style files, for
  example `model cabin.glb lowpoly=cabin_lp.glb`.
- **Hand-shaped designs** keep their shape: only surfaces and colours follow.
- **Characters with bones** follow by choosing a matching look set (§9), never by
  remeshing.

---

## 8. Generators to adapt or extend

| Generator | Change | Kind | Phase |
|---|---|---|---|
| `style.rs` | parse `shape`, `make.*`, `surfaces`, `render`, `version`, `intensity`; map `form` words into the axes; validation; extend `takes` | **Modify** | 1 |
| `shapes.rs` `made_from`, `form_sort` | list the new groups per archetype; `form_sort` becomes "which family" | **Modify** | 1 |
| `shapes.rs` `Mesh`, `bytes()`; `world_view.gd _build` | UV block, material slot; `MSH2`; reading both formats | **Modify** | 2 |
| `world_view.gd` PAINT | material slots, texture arrays, alpha scissor, normal+roughness | **Modify** | 2 |
| `grown.rs` `grow`, `tree()` | profile argument; builder table; realistic tree builder (EZ-Tree port); leaf cards; bark UVs | **Modify + Build** | 3 |
| `real.rs tree`, `land.rs conifer` | entries in the tree builder table | **Modify** | 3 |
| LODs (`Mesh::lighter`) | general simplifier for every family (meshopt, if approved), impostor cards for trees | **Modify + Build** | 4 |
| Region far meshes | merged far mesh per 500 m region, visibility ranges | **Build** | 4 |
| `grown/rocks.rs`, `land.rs rock` | strata, cracks, bevels, sinking, pebbles | **Modify** | 5 |
| Cliff faces | the land maker places strata pieces on steep bands | **Build** | 5 |
| `land.rs ground()`, land colour | splat weights per corner; relief | **Modify** | 5 |
| `grown/plants.rs` | card builders for fern and bush; blade grass by MultiMesh (host buffers plus Godot MultiMesh) | **Modify + Build** | 6 |
| `land/build.rs` structures and house facades | courses, shingles and tiles with UVs, window depth, sag | **Modify** | 7 |
| `build.rs` roadway, pathway | profile, verges, edge stones, splat | **Modify** | 7 |
| `build.rs riverwater`, sea shader | foam band, wet-sand splat, reeds, SSR | **Modify** | 7 |
| `graphics.gd`, `clock.gd` | `render` asks with quality caps; AgX; FSR 2 option; specular occlusion; SMAA | **Modify** | 2 (small) |
| `finish.txt` → style | move the finish words into `look` | **Modify** | 1 |
| `worldgen.gd` | the realistic recipe; stop discarding hand edits at :1806 | **Modify** | 8 |
| `models.gd` | per-style model variants | **Modify** | later |

---

## 9. Character Studio compatibility

Not built now. The interface it will need:

- **`make.figures`** in the style says three things:
  - `proportions real|stylised|chibi`: a band of head-to-body ratio and limb thickness;
  - `shading pbr|toon|flat`;
  - `set NAME`: which look set to prefer (`realistic`, `lowpoly_blocky`, …).
- **Character Studio** tags each look set and clothes item with the styles it suits, and
  offers "fits this world's style" first.
- **The world's figures** use the set the style asks for, if the character has one.
  Otherwise they use their own, and the editor's log says so.
- The existing `figures` colour words and `limit_figures` stay.
- Skinned meshes are never remeshed. A style chooses a set; it does not reshape bones.
- **Realistic characters** need realistic look sets with UVs and textures. That is
  Character Studio's own later work. The `MSH2` format is not involved (characters are
  glTF).

---

## 10. Performance

### 10.1 Where realism costs

| Cost | Low poly today | Realistic, naive | Realistic, planned |
|---|---|---|---|
| A tree, near | 300–800 tris | 15–40k tris (twigs, cards) | 8–15k near, 2–4k mid, 300 far, 2-tri impostor beyond ~150 m |
| Woods of 5,000 trees | about 2.5M tris | far too many | about 1.5M visible with LODs and region far meshes |
| Grass | turf shader plus strewn designs | millions of blades | MultiMesh per 32 m cell, 30–60k blades near the eye, thinned by distance; no shadows from grass |
| Draw calls | one shared material, instanced | one material per texture: thousands | a few slot materials, shared; automatic instancing kept; per-thing variation in instance uniforms |
| Texture memory | none | dozens of 2–4k sets | texture arrays of 1k (2k for hero sets), 2 textures a layer (albedo+height, normal+roughness); about 120–250 MB for a realistic style |
| Shader cost | one light shader | full PBR per slot plus alpha cards | alpha scissor (no blending), few variants, ubershaders hide compile hitches |
| Generation time | 15 s first form change | a realistic tree takes 10–50 ms to grow; 30 designs is fine, rebuilding every instance is not | designs grow once per (recipe, family profile), and instances share them (as now). Grass is buffers, not meshes. Work goes to the host's existing jobs, off the frame |
| Memory (host) | meshes in a memory cache | bigger meshes ×3–4 with UVs and LODs | measure with `scale -- memory`; a disk cache if needed (below) |
| Corner limit | 20k per mesh, 19k grown | a realistic oak can pass 19k | keep the safeguard; split a tree into trunk and crown meshes rather than raise it |

### 10.2 Mitigations

1. **Host-made LODs for every family**, sent with the mesh. Godot will not make them
   for run-time meshes.
2. **Tree impostors.** At first, a crossed-card or billboard picture rendered once per
   design. Octahedral impostors later, if needed.
3. **Region far meshes:** one merged mesh per 500 m region (land, woods as impostor cards,
   buildings as boxes), with visibility ranges. This fits the persistent-world regions.
4. **Shared materials only**, so automatic instancing works. Variation goes in vertex
   colour and instance uniforms.
5. **Grass as MultiMesh buffers** per cell, shadows off (Light3D shadow caster mask).
6. **Quality caps on the style's asks.** Realistic on Low quality: no SDFGI, SSIL off,
   1k textures, shorter grass distance, impostors sooner.
7. **A disk cache** of realised meshes keyed by (recipe, profile hash, code version).
   This one is optional and needs your yes (D4). Today every load rebuilds; the first
   load of a realistic world could be slow.
8. **Generation is editor-time.** The game uses the same host, so a built game still
   realises on load. The disk cache, or baking at export, would remove that cost.
9. **Measure first.** Each phase states its budget on `realtest` and on the 4 km
   open-world demo (`-Demo openworld`): frame time, triangles drawn, draw calls, host
   memory, first-change time. **Building substantial worlds must still work.** No phase
   is accepted on a small scene alone.

---

## 11. UI that this needs (and only this)

All of it lives in the existing Window › Styles, the bar's Style button, the Inspector,
and New world. There is no layout overhaul, and labels are plain words of 1–3
(familiar words first).

| Need | Control | Where |
|---|---|---|
| Pick a world's style | the existing list, grouped by family (Realistic, Painterly, Cartoon, Low poly, Others) | Styles, New world |
| Preview a change | the existing live change with **Keep / Put back**; **Try on a sample** (exists); a **Compare** strip of the same view under 2–4 styles, as pictures | Styles |
| Strength | one **Strength** slider (intensity, §7.3) | Styles, top |
| Characteristics | the shared axes as sliders: **Flat faces, Rounded, Uneven, Exaggerated, Detail, Smoothness, Edges, Wear** | Styles › Shapes |
| Family settings | one folding part per family (**Trees, Plants, Rocks, Land, Buildings, Paths, Water, People**): a builder picker ("Crowns: Lumps / Cards / Puffs / Cones") and the family's few words | Styles › Shapes › family |
| Surfaces | a picker per slot from the library's texture sets, with a thumbnail; **Texture size** | Styles › Surfaces |
| Rendering asks | **Light bounce: None / Fast / Full**, **Reflections**, with "held back at this quality" said in words | Styles › Light |
| Override on one design | the existing **As the style says / Kept as made / …** row, extended with the styles' family builders | Styles (picked thing), Inspector |
| Save and duplicate | the existing **Customise (makes a copy)**; **Export…** and **Import…** | Styles |
| Version | **Update / Keep mine** when the library's style is newer | Styles, top |

Every control writes style lines through the existing debounced writer. Nothing is
typed by hand.

---

## 12. Phased roadmap

Each phase is a normal pass and ends with a demo, host tests where the risk is, and the
low poly check.

| Phase | What | Depends on | Acceptance |
|---|---|---|---|
| **1. Profile plumbing** | `shape`, `make.*`, `surfaces`, `render`, `version`, `intensity` parsed; `form` mapped into the axes; `grow` and the builders take a `FamilyProfile`; `made_from` lists the new groups; finish words move into the style; header of `styles.txt` brought up to date | — | **No visible change anywhere**: every existing demo's pictures are identical; `-Demo lowpoly -Check` passes with the same triangle counts; a host test shows a change to `make.rocks` remakes no tree |
| **2. Surfaces** | `MSH2` (UV, slot); PAINT with slot texture arrays, normal+roughness, alpha scissor; surface sets in the library with licence notes; `render` asks with quality caps; AgX, specular occlusion, SMAA, FSR 2 offered | 1 | `MSH1` meshes render identically; a test design with UV'd planks shows its texture; the quality caps say what they held back; no new shader-compile hitch over 100 ms on first sight (measured) |
| **3. Realistic trees (the PoC)** | the realistic tree builder (EZ-Tree port, MIT notice); leaf cards; bark UVs; a broadleaf, conifer and birch preset under `realistic`; `styled as_made`; hand-shaped designs refused for regrowing | 1, 2 | **§13** |
| **4. Scale** | LODs for every family (meshopt if approved); tree impostors; region far meshes with visibility ranges; grass MultiMesh pipeline started | 3 | the open-world demo under Realistic trees holds its frame budget (set at Phase 3's measurement; target ≤ 16.7 ms on this machine at High); host memory within 1.5× of today |
| **5. Rocks, cliffs, ground** | rock strata, cracks and bevels; cliff faces on steep bands; terrain splat and layers; relief | 2, 4 | the cliff and scree read as rock in the `realtest` views; land editing speed unchanged (brush stroke time measured before and after) |
| **6. Plants** | fern and bush card builders; blade grass (MultiMesh, GodotGrass-style shader) | 2, 4 | a clearing of grass at near range in budget; walking through grass still bends it (`treads`) |
| **7. Built, paths, water** | plank and log courses, shingles and tiles, window depth; path profile and verges; shore foam, wet sand, reeds | 2 | the cabin in `realtest` reads as timber and stone up close; low poly houses unchanged |
| **8. Realistic proof world** | `realtest` complete; the open-world generator's realistic recipe; the `worldgen.gd` discard mended | 3–7 | the four views look convincingly realistic to you (your call); same world under Low poly unchanged; style change time on `realtest` stated |
| **9. More styles** | painterly (`crown puffs`, soft ramp, brush cards), exaggerated cartoon (blobs, bevels, gloss), stylised realism; intensity slider; Compare strip; Export and Import | 3–8 | the same `realtest` world under five styles shows different *shapes* (contact sheet), not only different colours |
| **10. Regions, figures** | per-region styles (optional); `make.figures` with Character Studio look sets | 9 | a region boundary shows two styles with no seam in placement; figures pick the style's set when they have one |

### Decisions needed from you

- **D1. The mesh format change (`MSH2` with UVs and material slots).** I recommend yes:
  realism needs it, and it is backward compatible.
- **D2. The `meshopt` crate** (MIT, meshoptimizer) for LODs in the host, as against
  extending `Mesh::lighter` by hand. I recommend `meshopt`: it is the standard tool and
  saves weeks.
- **D3. Texture sets downloaded from Poly Haven** (CC0). About 10 sets at 1k, roughly
  40–60 MB; I'll name each file and its size before downloading. Plus your own from
  Material Maker later.
- **D4. A disk cache of realised meshes.** Optional. I recommend deciding after Phase 4's
  measurement.
- **D5. Porting EZ-Tree's algorithm** with its MIT notice, rather than writing a tree
  builder from the Weber & Penn paper alone. I recommend the port: it is the faster
  route.
- **D6. Sky3D:** use its shaders, or keep our sky. I recommend keeping ours and reading
  Sky3D only as a reference, because `clock.gd` and `sunsets.gd` already own the hours.

---

## 13. The smallest meaningful proof of concept

**What:** Phases 1 and 2 (the minimum of each) plus Phase 3, on one family: trees.

**Build:**

1. Profile plumbing for `make.trees` only, plus the shared axes, with `form` mapped.
2. `MSH2` with UVs and a slot byte. PAINT with two slots (bark, leaf cards), alpha
   scissor and normal+roughness.
3. Two surface sets: `bark_oak` and `leaf_cards_broad`, from Poly Haven CC0 (D3), in
   the library with a licence note.
4. A realistic tree builder (`crown cards`), a port of EZ-Tree's levels inside
   `grown.rs`. A style `realistic` (like `summer`) with `make.trees crown cards levels 3
   …` and its surfaces.
5. Host LODs for the tree only (`Mesh::lighter` extended, or meshopt).
6. Demo `-Demo styletrees`: one wood plus one tree alone, from the island town copy,
   under no style, Low poly and Realistic. Close, mid and far views.

**It proves the architecture when:**

- The realistic oak is **built differently**: more branch levels, twigs, leaf cards, a
  trunk of 12+ sides near. It is not the same tree re-coloured. The demo checks the
  structure (levels, card count), not only pictures.
- Under Low poly the same design comes out **identical to today** (the triangle count
  and the existing lowpoly demo pass).
- Under no style it is **identical to the grown design** as kept.
- A **hand-shaped tree design is not regrown**. Only its surfaces change.
- A tree's **place and turn do not move** between styles. Switching back is a cache hit
  (the time is stated).
- A rock and a house are **not remade** when only `make.trees` changes (host test).
- **Frame budget:** the wood of the demo holds the frame time at High, measured, and
  the LOD switch is visible only on inspection.
- Your verdict on the pictures, then a hand test in the Styles window.

**Not in the PoC:** terrain layers, cliffs, grass blades, buildings, the Compare strip,
regions. They come after the architecture has shown it works.

---

## Sources

- **Code surveyed:**
  - `host/src/shapes/style.rs`, `shapes.rs`, `grown.rs`, `grown/*.rs`, `real.rs`,
    `land.rs`, `land/build.rs`, `land/world.rs`, `kind.rs`, `vary.rs`, `lib.rs`;
  - `godot/styles.txt`, `finish.txt`, `palettes.txt`;
  - `world_view.gd`, `clock.gd`, `graphics.gd`, `styleview.gd`, `models.gd`,
    `worldgen.gd`, `project.godot`.
- **Earlier records:**
  - `Draw3D_Low_Poly_Advanced_Style_2026-10-30.md`, `Draw3D_Style_Library.md`,
    `Draw3D_Style_References_Neva_and_Firewatch.md`;
  - `Draw3D_Godot_4.8_Assessment_2026-10-29.md`,
    `Draw3D_Rendering_Variety_Traversal_2026-10-24.md`.
- **Godot:**
  - release pages 4.3–4.7 (godotengine.org/releases/4.x);
  - docs: Mesh LOD, ImporterMesh, Visibility ranges, Occlusion culling, Optimizing 3D,
    Introduction to GI, Compositor, Shading language.
- **Projects:**
  - github.com/dgreenheck/ez-tree, github.com/JekSun97/gdTree3D,
    github.com/gaea-godot/gaea, github.com/caphindsight/Foliage3D;
  - github.com/HungryProton/scatter, github.com/dreadpon/godot_spatial_gardener,
    github.com/TokisanGames/Terrain3D, github.com/TokisanGames/Sky3D;
  - github.com/Zylann/godot_heightmap_plugin, github.com/2Retr0/GodotGrass,
    github.com/RodZill4/material-maker, github.com/Arnklit/Waterways;
  - github.com/mxgmn/WaveFunctionCollapse, github.com/wikicode96/home-builder,
    polyhaven.com/license.
- **Papers and talks:**
  - Weber & Penn 1995;
  - Runions et al. 2007 (algorithmicbotany.org);
  - Müller et al. 2006 (CGA shape grammar);
  - Guerrilla Games, GPU procedural placement in Horizon Zero Dawn (GDC 2017);
  - Ghost of Tsushima grass (GDC 2021).
- **Unverified:** Buildify's licence, the Blender rock addons' licences, Breath of the
  Wild's rendering specifics, run-time `ImporterMesh.generate_lods` in exported games,
  and the merge status of HDDAGI.
