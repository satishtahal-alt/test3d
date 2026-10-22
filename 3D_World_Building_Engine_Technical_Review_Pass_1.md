# 3D World-Building Engine --- Technical Review, Pass 1

## 0. Verdict

The vision is coherent and the core instinct is right: spend complexity
on world construction, style and atmosphere, not on photoreal rendering.
Four things need to change before it is buildable.

1.  **The document describes two products and treats the wrong one as
    the foundation.** One is a conventional runtime (render, physics,
    audio, input, scripting). The other is a *world compiler*:
    specification in, editable world out, with provenance. Your layered
    sketch puts procedural systems near the top as features on a
    runtime. They should be the centre, with the runtime as a
    deliberately modest backend.
2.  **"Baking" should be caching, not conversion.** If generated content
    is always a derived artifact keyed by a hash of its inputs, then
    procedural → cached → optimized → baked is one mechanism at
    different settings. Provenance cannot be destroyed because the
    source specification is never replaced.
3.  **The hard problem is stable identity across regeneration, not
    generation.** "Replace the farm animals without disturbing anything
    else" and "make the trees older but keep the one I moved" both
    depend on every generated thing having an identity that survives
    regeneration. This must be designed first.
4.  **The generator list is roughly ten products.** Terrain, vegetation,
    architecture, urban, creature, character, water, weather, style and
    traversal are each a multi-year specialism. The realistic plan is a
    strong generator *framework*, two generators built deep, and the
    rest shallow or deferred, with creatures last.

Two further corrections:

-   **Natural language does not belong inside the engine.** The engine
    should understand a typed, structured specification. An LLM sits
    outside and translates "make this neighbourhood run-down" into edits
    to that spec, through the same command API the editor uses. The
    engine stays deterministic and works without AI.
-   **"Generate, don't retrieve" is too strict.** Every reference game
    you list gets its look from hand-authored pieces assembled
    procedurally. Pure generation is strong for terrain, rocks, trees,
    building massing and scatter, and weak for a dumpster or an AC unit.
    The right model is procedural assembly of authored *kits and
    parametric recipes* that belong to a style.

## 1. Classification of your ideas

  -----------------------------------------------------------------------
  Category                            Items
  ----------------------------------- -----------------------------------
  **Fundamental**                     Retained provenance and selective
                                      regeneration. Authored and
                                      generated content in one pipeline.
                                      Semantic tags on generated
                                      geometry. Determinism. Runnable
                                      early, with a persistent lab scene.
                                      Stable API for scripts and tools.
                                      Stylized rendering with atmosphere
                                      as first-class.

  **Strong preferences (keep)**       Style propagating across
                                      generators. Scale as a world-design
                                      capability. Traversal-aware
                                      generation. Third-party physics,
                                      audio and platform. Bounded AI
                                      tasks under human architectural
                                      control.

  **Needs investigation**             Stable identity and overrides.
                                      Style as construction rule.
                                      Path-first traversal generation.
                                      How far scale reaches into
                                      generators. Whether any generation
                                      must run at runtime in shipped
                                      games.

  **Defer**                           Creature Engine. Urban interiors.
                                      Cloth tearing. Networking. Visual
                                      scripting. Lightmap baking. Plugin
                                      system. Crowds. Vehicles. World
                                      streaming (but chunk the data model
                                      now).

  **Reject or redesign**              Multiple rendering *paths* → one
                                      render graph with pluggable passes
                                      and shading models.
                                      Engine-understood natural language
                                      → external LLM editing a typed
                                      spec. Pure generation of everything
                                      → kits and recipes plus procedural
                                      assembly.
  -----------------------------------------------------------------------

## 2. Centre of gravity

**The engine is an incremental, deterministic world compiler with
retained source, attached to a small stylized runtime.**

What makes it different is four properties of the data model:

1.  **The specification is the source of truth.** Meshes, colliders,
    navmesh, traversal graph, audio zones and light probes are all
    compiled outputs.
2.  **Compilation is incremental.** Changing one parameter recompiles
    only what depends on it, fast enough to feel like direct
    manipulation.
3.  **Outputs carry meaning.** Every generator emits semantic roles
    (walkable, ledge, rope_anchor, flammable) alongside triangles.
4.  **Style is an input to compilation**, not a texture applied
    afterwards.

Traversal, scale, ecology, urban and creatures are all consumers of
those four properties.

The compiler should be buildable and testable **headless, with no
renderer**, emitting glTF and JSON. That keeps its identity independent
of the runtime.

## 3. Proposed architecture

### 3.1 Two sides, one direction of dependency

**Authoring / compile side** - Editor UI - CLI - optional in-game
authoring - AI/automation bridge - Command API - World Model - World
Compiler - domain generators - derivers - generation kernel

**Runtime side** - Gameplay framework and scripting - Runtime world /
ECS - render - physics - audio - animation - navigation - particles -
environmental fields

**Shared foundation** - Asset database - content-addressed derived-data
cache - Core: reflection, serialization, handles, jobs, math, hashing,
logging, profiling - Platform: window, input devices, audio device,
filesystem, time

Dependency rules: - Runtime never includes generator code. A fully baked
game links no generators. Runtime generation is opt-in. - Generators
never include renderer, physics or ECS headers. They emit neutral
artifacts. - Generators never call each other. They communicate through
declared inputs and outputs on the World Model. - Editor, CLI and AI
bridge never touch the World Model or ECS directly. They issue commands
and read queries. - Renderer and physics do not know about each other.
Debug drawing goes through a small Core interface.

### 3.2 World Model

A hierarchical graph of **spec nodes**. Each has a type, typed
parameters, a style reference, declared inputs and a stable ID.

-   A hand-placed imported mesh is a `literal` node.
-   Procedural terrain is a `terrain` node.
-   Prefabs and variants are spec subtrees with parameter overrides.
-   Overrides are a separate layer: parameter override, deleted, or
    detached from generator and converted to literal.

**Stable identity.** IDs are semantic paths derived from structure, not
enumeration order. Seeds are derived by hashing the path with the parent
seed, never drawn from a shared random stream. Scattered content uses
spatial identity such as cell coordinates plus index within cell.

When an override target disappears, it becomes an orphan shown by the
editor.

### 3.3 Compiler

Each generator is conceptually:

``` text
(params, style, seed, declared inputs, observer scale, detail budget) → artifacts
```

Artifacts include: - meshes - instance sets - colliders - semantic
tags - child spec nodes - field stamps - audio emitters

Cache keys hash all inputs plus generator version.

Bake stages become policy: - Live - Cached - Optimized - Baked/shipped

The source specification remains in the project even when shipped builds
contain only derived artifacts.

Derivers include: - collision - navigation - traversal annotation -
LOD - audio zones - probes - ambient-occlusion data

### 3.4 Generator cooperation

Cross-generator dependencies use shared fields and stamps rather than
direct calls.

Example: - building writes a flatten-footprint stamp - terrain evaluates
base terrain + stamps - vegetation reads final terrain and occupancy

A possible fixed stage order: 1. macro layout 2. terrain 3. water 4.
paths 5. structures 6. vegetation 7. props 8. derivers

The World Builder owns macro layout and shared anchors rather than
becoming a giant super-generator.

### 3.5 Style

Four layers:

  -----------------------------------------------------------------------
  Layer                   Purpose                 Wooden storybook
                                                  example
  ----------------------- ----------------------- -----------------------
  Substance               physical/gameplay       density, friction,
                          properties              footstep sound,
                                                  flammable, floats

  Construction            how things are made     planks, pegged joints,
                                                  span limits, end grain

  Surface                 shading and detail      hand-painted grain,
                                                  edge treatment

  Palette/form            colour, proportion,     warm ochres, chunky
                          irregularity            forms, slight wobble
  -----------------------------------------------------------------------

Generators first produce structural intent such as wall panel, beam,
roof surface, column, rock mass, trunk or fence run. A style-specific
realizer turns that intent into the final construction.

### 3.6 Scale

Scale is a context passed to compiler and runtime, not merely a player
transform.

-   Simulation can keep characters near a stable physical scale.
-   Generation receives observer scale and adjusts detail/semantic
    roles.
-   Movement metrics are character data consumed by
    traversal/navigation.
-   Perception changes with scale.
-   Camera-relative rendering and origin rebasing should be considered
    early.
-   Runtime growth/shrink may require artifacts at multiple observer
    scales.

### 3.7 Traversal

-   Generators annotate known traversal semantics.
-   A deriver analyzes geometry against movement metrics.
-   Output is a traversal graph that can also feed AI navigation.
-   Explicit routes become constraints between anchors.
-   Build the validator before attempting to generate "fun" routes.

### 3.8 Reactive environment

Use environment fields for: - wind - snow depth - wetness - heat -
trampling

Vegetation, particles, cloth, audio and gameplay can sample these
fields.

A substance interaction table can define interactions such as
fire/water/wind/cold against wood/stone/cloth/ice.

### 3.9 Gameplay boundary

-   **Engine systems:** simulation/presentation primitives
-   **Gameplay framework:** reusable patterns
-   **Script:** game-specific rules
-   **Data definitions:** content
-   **Generated content:** instances carrying semantic tags

Use hierarchical tags as shared vocabulary between generators and
gameplay.

## 4. Missing systems

Important missing systems identified:

-   reflection/type registry
-   command and undo
-   stable identity and overrides
-   derived-data cache
-   generation kernel
-   splines/regions/masks
-   constraint/layout solver
-   generation debugging
-   world validation
-   skeletal animation
-   camera
-   mood/environment state
-   decals
-   shader pipeline
-   job system
-   game UI/text
-   save games
-   game loop/time
-   hot reload
-   logging/console/debug draw/profiler
-   testing/headless mode
-   build/dependencies/packaging
-   crash handling/autosave
-   localization
-   world partitioning/streaming
-   networking
-   plugins/cinematics

## 5. Technology recommendations

### Core choices proposed for investigation

-   **Language:** C++20, with restricted style, handles across modules,
    sanitizers, warnings-as-errors.
-   **Platform:** SDL3 rather than GLFW.
-   **Graphics:** SDL3 GPU API or WebGPU; raw Vulkan/OpenGL not
    preferred.
-   **Shaders:** Slang.
-   **Rendering:** Forward+ with a thin prepass; one render graph with
    pluggable passes/shading models.
-   **Physics:** Jolt behind an engine interface.
-   **Cloth:** shader wind → spring-bone chains → simulated cloth; defer
    tearing.
-   **Audio:** miniaudio + engine event/bus/parameter layer; Steam Audio
    later; FMOD/Wwise as commercial alternatives.
-   **Scripting:** Luau, with bindings generated from reflection.
-   **ECS:** flecs, with EnTT as alternative.
-   **Editor:** Dear ImGui initially; editor communicates through
    commands.
-   **Game UI:** RmlUi.
-   **Animation:** ozz-animation.
-   **Navigation:** Recast/Detour.
-   **Mesh import:** glTF only initially; ufbx only if FBX becomes
    necessary.
-   **Geometry:** Manifold, meshoptimizer, FastNoise2, xatlas.
-   **Particles:** own CPU system first; GPU compute later.
-   **Serialization:** own source format plus binary artifacts and
    migrations.
-   **Profiling:** Tracy.
-   **Build:** CMake with pinned dependencies.

## 6. Ranked risks

1.  Too many generators before the data model is stable.
2.  Generated content becomes uneditable.
3.  Output is technically correct but aesthetically flat.
4.  Generators become tightly coupled.
5.  AI-assisted architecture drift.
6.  Nondeterminism.
7.  Regeneration/performance problems.
8.  World generation becomes difficult to debug.
9.  Generated spaces lack gameplay quality.
10. Creature Engine complexity.
11. Excessive abstraction.
12. Editor depends on internals.
13. Scripting API instability.
14. Third-party libraries dictate architecture.
15. Baking destroys provenance.
16. Multiple rendering paths.
17. Cloth/physics complexity.
18. Asset import.
19. Physics/rendering coupling.

## 7. Roadmap proposed

### Track C --- compiler/headless

World Model, stable IDs, hashed seeds, cache, overrides, toy generator,
glTF/JSON output, hash tests.

### Track R --- runtime/editor

1.  **Walking skeleton:** platform, graphics, renderer, camera, ECS,
    reflection, serialization, glTF, input, Luau, editor shell,
    commands/undo, save/load, text, one-shot audio, profiler, headless
    screenshot tests, export. Result: 3D Tetris + lab v0.
2.  **Physical world:** Jolt, fixed timestep, triggers, joints, queries,
    controller, debug draw, 3D audio, particles, follow camera. Result:
    Marble Run.
3.  **Tracks merge:** compiler drives runtime; provenance inspector,
    regeneration, overrides, splines/regions, scatter, rocks, first tree
    generator.
4.  **Landscape and mood:** terrain, paths, ecology, wind, instancing,
    LOD, sky, fog, mood, water.
5.  **Style:** four style layers, realizers, shading models, outlines,
    second style.
6.  **Architecture:** footprint/massing/floors/facade/roof, kits,
    decals, age parameter.
7.  **Inhabitants/traversal:** animation, traversal graph, metrics,
    validator, navmesh, animal AI, climbing, tiny-player test.
8.  **Urban:** streets, blocks, lots, buildings, infrastructure,
    vertical traversal, city lighting.
9.  **Reactive world:** environment fields, snow, fire, weather, cloth,
    GPU particles, generated audio zones.
10. **Scale-out:** optimized bake, world cells/streaming, saves,
    packaging, performance.
11. **Creatures:** procedural creature system.

## 8. AI-assisted workflow

Recommended controls: - build-enforced dependency rules - warnings as
errors - sanitizers - public headers as contracts - architecture
decision records for interface changes - headless verification -
screenshot regression - generator hash tests - bounded task briefs - one
current architecture document - version-control checkpoints

## 9. Creature Engine

Recommended eventual approach: - skeleton-first specification - typed
body parts and attachment sockets - mesh generated around the skeleton -
procedural motion - gait generators - IK - part state machines

Until then, use imported rigged animals placed through ecology rules.

## 10. Minimum serious foundation

Must exist: - reflection - versioned serialization - command/undo -
handles/asset database - World Model - stable IDs/overrides -
compiler/cache - runtime ECS - graphics layer - forward renderer -
action input - fixed timestep - script VM - editor shell through
commands - headless tests - build-enforced boundaries

Can initially be stubbed: - audio - game UI - jobs - localization - one
hard-coded style - constant scale context

## 11. Decisions requested for Pass 2

1.  Own runtime or existing engine?
2.  C++ or Rust?
3.  Which two generators go deep first?
4.  Does shipped content need runtime generation?
5.  Is the editor for one expert or other users?
6.  Which single style should be made beautiful first?
7.  Accept natural-language-outside-engine and kits/recipes corrections?
