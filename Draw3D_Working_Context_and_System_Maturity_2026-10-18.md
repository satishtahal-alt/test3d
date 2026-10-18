# Draw3D --- Working Context and System Maturity Overview

**Snapshot:** 2026-10-18\
**Project:** Draw3D --- Godot-hosted procedural 3D world-authoring
engine with a Rust semantic core\
**Repository:** https://github.com/satishtahal-alt/test3d

## Why this file exists

Use this as a context handoff if the current chat becomes unavailable.
It summarizes the latest project context, current development direction,
a broad maturity estimate for major systems, and the next benchmark. The
live repository `PROGRESS.md` remains the source of truth; check it
before recommending or implementing work.

This is a high-level assessment from documented progress, not an
independent audit or claim that every feature has been retested.
Maturity labels estimate the breadth and integration of the
implementation---not commercial readiness: - **Basic:** useful initial
implementation; limited breadth or integration. - **Intermediate:**
substantial working capability, but meaningful gaps remain. -
**Advanced:** broad or architecturally mature capability in its area;
still may need polish, integration, or testing. - **Mixed:** parts of
the system vary considerably in maturity.

## Current status and immediate work

As of the latest retrieved `PROGRESS.md` (2026-10-18, amendment 7cg),
the latest implementation is Tier 3 of the capability catalogue:
interactions/events such as sensed, struck, spotted, or acted upon can
trigger outcomes such as setting facts, giving items, dialogue, sound,
or movement through the Inspector's "What it can do" catalogue.

The preceding work includes condition-based behaviours
(follow/roam/hunt), solid-shape modelling tools, assistant-drafted edits
applied as a single undo step, improved character movement/animation
options, door traversal and interiors, authorable game display, creature
following/fleeing, reusable abilities, image textures, climbable
surfaces, inventory/equipment, ranged combat, health/status/effects, and
broader combat/wilderness and shelter/building capabilities.

**Current active benchmark:** Claude is working on a small but polished
third-person wilderness action-adventure, provisionally "The Forgotten
Watchtower." It should be a compact 5--10 minute vertical slice: forest
trail and landmarks, abandoned campsite/structure, fitting creatures and
combat, equipment pickup, an interaction-driven objective, and a clear
completion state. It must be authored through Draw3D's own systems
wherever possible, not implemented as a hard-coded showcase. Claude
should identify and fix only important, reusable capability gaps,
document minor issues, and keep testing risk-based.

Required deliverables include a repository document such as
`Draw3D_Wilderness_Adventure_Benchmark.md` with the overview, major
systems used, a practical guide to recreating the level through the
editor, capability gaps, and actual validation performed. A future idea
must also be recorded (not implemented in this pass): a live,
step-by-step guided recreation mode that loads a reference level and
guides a user through rebuilding it in the real editor, with user
confirmation at each step and a way to report missing tools/options.

The LLM integration is a separate workstream currently being explored.
Do not make it a prerequisite for the benchmark or conflate it with the
benchmark work.

## Major systems and maturity snapshot

  -----------------------------------------------------------------------------------------
  System                  Estimate                     What is documented / what remains
  ----------------------- ---------------------------- ------------------------------------
  Core architecture and   **Advanced foundations**     Rust semantic core with Godot as
  project model                                        host/view;
                                                       authoring/build/generation/runtime
                                                       separation, edit history/undo,
                                                       live/frozen/detached concepts and
                                                       runtime firewall are core
                                                       principles. Continue validating
                                                       end-to-end persistence and clean
                                                       authoring/runtime boundaries.

  Terrain and world       **Intermediate--Advanced**   Terrain creation and sculpting,
  editing                                              terrain paint and world editing
                                                       exist. Some UX/visual feedback and
                                                       the distinction between surface
                                                       paint and genuinely editable
                                                       land/shoreline remain areas to
                                                       improve.

  Procedural nature and   **Intermediate--Advanced**   Multiple plant families, vegetation
  environmental                                        generation, style/color variation,
  composition                                          environmental motion and nature demo
                                                       coverage are documented. Remaining
                                                       challenge is authored-looking
                                                       composition, role-based variation,
                                                       dense-world performance, and
                                                       ensuring nature integrates smoothly
                                                       with terrain and structures.

  Asset/design library    **Intermediate**             Design/library workflows, imported
  and placement                                        models, basic shapes, contextual
                                                       controls and arrangement features
                                                       exist. Drag/drop, selection/snap
                                                       quality, and smoother
                                                       placement/authoring workflows remain
                                                       important usability areas.

  Construction,           **Intermediate**             Modular structures, rooms, doors,
  structures and                                       roofs, steps, ponds, shared sides
  interiors                                            and multi-storey/shelter work are
                                                       documented. The workflow is useful
                                                       but still less cohesive than the
                                                       intended spatially painted, adaptive
                                                       Townscaper/Tiny Glade-like
                                                       construction experience.

  Style system            **Intermediate**             Style controls and a library of
                                                       game-inspired visual directions are
                                                       documented, with influences across
                                                       colour, lighting, atmosphere and
                                                       rendering. More depth, consistency
                                                       and influence over generated content
                                                       is still needed; style should affect
                                                       more than materials alone.

  Characters, models and  **Intermediate**             Character models, animation
  animation                                            selection, secondary motion
                                                       (hair/skirt/tail), movement
                                                       animation options and gear fitting
                                                       are documented. Consistency,
                                                       transition quality, broader reusable
                                                       animation authoring and a stronger
                                                       timeline/scrubbing workflow remain
                                                       likely development areas.

  Movement, navigation    **Intermediate**             Third-person movement, turning,
  and traversal                                        dodging/strafing, following/fleeing,
                                                       doors and climbable surfaces are
                                                       documented. Robust navigation across
                                                       changing terrain, stairs, interiors,
                                                       obstacles and varied movement states
                                                       needs real-world validation.

  Physics and mechanisms  **Basic--Intermediate**      Physics/mechanism demonstrations
                                                       include joints, fans, belts and hit
                                                       responses. The benchmark should
                                                       reveal how much is general-purpose
                                                       and authorable, rather than relying
                                                       on individual demonstration setups.

  Behaviours and reusable **Intermediate**             Reusable abilities, facts/conditions
  abilities                                            and behaviours such as follow, roam
                                                       and hunt are documented; behaviour
                                                       can be configured through the
                                                       Inspector. More complex combinations
                                                       and clear debugging/authoring of
                                                       behaviour state need validation.

  Interaction and         **Intermediate--Advanced     Tier 3 is the latest recorded
  event/outcome catalogue breadth**                    implementation. Events can trigger
                                                       facts, item grants, dialogue, sound
                                                       and movement. The key remaining
                                                       proof is whether creators can
                                                       configure and combine these reliably
                                                       in a complete game without bespoke
                                                       code.

  Combat, health and      **Intermediate**             Health, statuses/effects, ranged
  creature encounters                                  attacks, wind-up/dodge, takedowns,
                                                       factions, predator/prey, packs and
                                                       territory are documented. Balance,
                                                       enemy navigation, readable feedback,
                                                       animation fit and encounter
                                                       composition require benchmark-level
                                                       validation.

  Items, inventory and    **Intermediate**             Individual item identity, item
  equipment                                            facts, pickup, inventory,
                                                       gear/equipment states, fitting gear
                                                       to models, carried-item modes and
                                                       controller inventory are documented.
                                                       End-to-end usability and clean
                                                       integration with objectives/combat
                                                       need testing.

  Game UI, controls and   **Basic--Intermediate**      Authorable game display,
  export                                               HUD/inventory/menu and
                                                       keyboard/controller mappings, plus a
                                                       game export workflow are documented.
                                                       A real benchmark should verify
                                                       launch, controls, state feedback,
                                                       save/reopen where applicable, and
                                                       exported runtime behaviour.

  Objectives and complete **Basic--Intermediate /      Interactions and facts provide
  game progression        needs proof**                building blocks for progression, but
                                                       the benchmark is intended to test
                                                       whether they form a coherent,
                                                       reusable objective-to-completion
                                                       workflow in practice.

  LLM-assisted authoring  **Separate workstream / in   Do not assume an integrated LLM is
                          progress**                   complete or required for the
                                                       wilderness benchmark. The user is
                                                       separately exploring whether an LLM
                                                       can connect under their available
                                                       plan and how to integrate it.

  Testing and regression  **Intermediate**             There is a meaningful test suite and
  discipline                                           a risk-based policy. Required
                                                       approach: build + changed tests,
                                                       targeted end-to-end check, plausible
                                                       adjacent checks, broader tours only
                                                       when risk/evidence justifies them.
                                                       Do not claim unrun tours passed.
  -----------------------------------------------------------------------------------------

## What the big picture says

Draw3D already has broad foundations across world creation, nature,
construction, characters, behaviours, interaction, combat, items and
game presentation. The main uncertainty is no longer simply "does this
feature exist?" It is whether the systems combine into a polished,
coherent game that can be created, inspected, modified, duplicated and
exported through the intended editor workflow.

The maturity labels above are deliberately not an assertion that Draw3D
is commercially ready. Several systems have substantial breadth but need
integration, workflow refinement and validation. A benchmark should turn
those unknowns into evidence.

## Recommended sequence from here

1.  Finish the current wilderness benchmark without derailing into
    unrelated minor bugs.
2.  Play it and verify that its intended start-to-finish loop works.
3.  Use its recreation guide to attempt the major authoring steps in
    Draw3D, recording missing controls, unclear settings or hard-coded
    dependencies.
4.  Fix high-value reusable gaps exposed by that attempt; document
    minor/unrelated issues.
5.  Reassess maturity from observed results, then choose a contrasting
    second benchmark to test generality.

## Testing and collaboration rules

-   Treat `PROGRESS.md` as the live implementation record and inspect
    the current repository before starting work.
-   Do not reimplement completed roadmap items just because an older
    roadmap entry still exists.
-   Keep Draw3D separate from Draw5.
-   Prefer short, practical implementation prompts and
    targeted/risk-based testing.
-   Do not let small unrelated defects block a benchmark, but record
    their actual scope and conditions.
-   Clearly distinguish build results, unit tests, end-to-end workflow
    checks, broader tours, and visual/demo evidence.
-   A demo illustrates a capability; it does not by itself prove the
    underlying system is general-purpose.
-   Keep unresolved decisions and future ideas in repository notes so
    they survive context changes.
