# Draw3D Master Roadmap — Final Working Roadmap Before Claude Review

> Working planning document for the future Draw3D engine.
>
> This revision incorporates the strongest findings from three independent LLM reviews (GLM, Gemma, and Gwen), plus the user's final personal review of the remaining candidates. Consensus findings and selected 2/3 findings are now explicit requirements. User-priority items are promoted where they directly strengthen Draw3D's intended identity. Lower-priority ideas remain deferred rather than becoming immediate implementation scope.

## Tracking

- ☐ Planned
- ◐ In progress
- ☑ Completed
- ⏸ Deferred
- ❌ Removed

**Documentation:** ☐ Not yet documented / ☑ Documented

### Working rules

- This is primarily a planning and capability-tracking document.
- Add, remove, clarify, consolidate, or move items as the engine vision develops.
- Completion means the capability we decided to build has been implemented; detailed testing history can remain with Claude/the development process.
- Documentation can be marked later, primarily as the engine matures.
- Use change notes on major sections/items when a requirement changes substantially.
- Keep stable item numbers where practical so requirements can be referred to consistently.
- Do not turn this into a detailed project-management system. Claude and ChatGPT can handle implementation details, testing, regression, and technical notes.
- Reviewer findings are requirements only when they fit the Draw3D vision. A review finding is not automatically a feature mandate.

---

# 1. Persistent Interactive World

**Status:** ☐ Planned  
**Documentation:** ☐

### Purpose

Build the foundation for a persistent authored world that can be edited, simulated, played, saved, and restored without confusing authored content with temporary runtime state.

### Capabilities

- ☐ Save and load the authored world.
- ☐ Define an explicit serialization/save format suitable for stable IDs, provenance, overrides, generated content, and authored content.
- ☐ Separate authored world data from runtime/gameplay state.
- ☐ Define explicit **World Data** vs **World State** responsibilities.
- ☐ Save and restore runtime/gameplay state independently of authored world data where appropriate.
- ☐ Preserve entity identity when saving/loading.
- ☐ Establish a clean **Preview → Simulate → Play** workflow.
- ☐ Support **Author → Preview → Simulate → Play → Inspect → Modify → Resume** as the broader workflow where practical.
- ☐ Support interactions and contextual prompts.
- ☐ Establish a contextual interaction layer that can expose appropriate actions based on object, actor, state, and situation.
- ☐ Inspect world objects and their properties.
- ☐ Support doors and interiors.
- ☐ Establish engine-owned colliders for world content.
- ☐ Support player start/spawn locations.
- ☐ Preserve authored changes when entering and leaving play.
- ☐ Allow temporary simulation state to be reset cleanly.
- ☐ Support deterministic re-entry into a scene where practical.
- ☐ Make runtime changes distinguishable from authored changes.
- ☐ Ensure gameplay changes such as opened doors, defeated enemies, collected items, unlocked areas, destroyed objects, and changed world conditions can become persistent world state when intended.

### Representative demo

A small village containing houses/interiors, doors, a chest, lamps, a well, a player start, and a persistent state change such as opening the chest or changing a world object. Save, exit, reload, and verify that authored content and runtime state remain correctly separated.

### Notes / open questions

- The authored world remains the source of truth for authored content.
- Runtime/gameplay state is a distinct layer rather than an accidental mutation of authored data.
- The save format must preserve the identity/provenance model rather than flattening it into opaque host scene data.
- Do not architect the engine around a single third-person game mode.

### Change notes

- Added explicit World Data vs World State and serialization requirements from the three-review consensus.

---

# 2. Characters, Creatures & Animation

**Status:** ◐ Character Studio, first tier (2026-10-15, amendment 7at): characters with bones of their own from glTF files, clip libraries retargeted to them, blended by the motor's words, a samurai set, a Studio window. (Before it, from the staged order: a rigged person posed from text, moves as data, sets per body, creatures.)  
**Documentation:** ☐

### Purpose

Create real player, NPC, and creature entities with reusable movement, animation, attachment, customization, and generation systems.

### Capabilities

- ☐ Player character entities.
- ☐ NPC entities.
- ☐ Creature entities.
- ◐ Rigging and skeletal animation support. (7at: glTF models and clip libraries read at runtime; retargeting by Godot's humanoid names.)
- ☐ Layered animation.
- ◐ Animation blending. (7at: stand/walk/run by speed, cross-fades, through Godot's AnimationTree, timed by the motor.)
- ☐ Animation state/transition system.
- ☐ Locomotion system.
- ☐ Walking, running, jumping, swimming, and other movement modes.
- ☐ Semantic movement capabilities rather than mesh-specific movement assumptions.
- ☐ Slopes and water interaction.
- ☐ Camera systems that are not locked to third-person.
- ☐ First-person camera support.
- ☐ Third-person camera support.
- ☐ Top-down/fixed camera support.
- ☐ Camera as a reusable capability layer that can support different gameplay types.
- ◐ Semantic attachment points. (Places on a rig since stage 5; on a character found from its map, 7at.)
- ◐ Equipment/prop attachment. (7at: a character holds glTF things at a bone, with places on them a move strikes with.)
- ☐ Generated characters.
- ◐ Imported characters. (7at: any glTF model with a humanoid skeleton and a map.)
- ☐ Generated creatures.
- ☐ Imported creatures.
- ☐ Semantic familiar-creature generation (for example, dog-like creatures).
- ☐ Semantic character archetypes (for example, pirate-like characters).
- ☐ Novel creature generation from specifications.
- ☐ Generated geometry remains editable.
- ☐ Procedural creature components.
- ☐ Animation/behavior hooks for generated creatures.
- ☐ Creature body structure connects meaningfully to attachments, animation, locomotion, collision/physics, and behavior.
- ☐ Support novel locomotion for unusual body plans rather than assuming every creature is bipedal.
- ☐ Creature behavior authoring works with generated and authored creatures.
- ☐ Character customization where it naturally fits the engine's authoring model.
- ☐ Procedural appearance/skin/material variation where useful.
- ☐ Progression from high-level creature design to component/shape editing to manual mesh editing.
- ☐ First audio integration, including footsteps and movement sounds.
- ☐ Authorable movement sets that can be assigned to characters/creatures rather than being automatic defaults for every entity.
- ☐ Reusable movement primitives such as jumps, dashes, rolls, climbs, slides, ledge transitions, and other traversal actions.
- ☐ Movement chains/combinations that can connect primitives into authored movesets.
- ☐ Movement parameters exposed at authoring level, including target distance, height, timing, arc, acceleration, momentum, recovery, and animation timing where appropriate.
- ☐ Ability to replace, interrupt, branch, or extend movement animation at meaningful points to create game-specific movement.
- ☐ Distinct movement styles for different genres or characters (for example ordinary jump, platforming jump, ledge jump, gap jump, parkour jump).
- ☐ Movement authoring is powerful enough for precise traversal while remaining approachable for ordinary character creation.

### Representative demo

A player, villagers, sheep, a dog, and at least one unusual generated creature moving through the world, including slopes, water, animation, attachments, behavior hooks, and different body/movement capabilities.

### Notes / open questions

The creature system should support both recognizable creatures and genuinely novel designs such as four-eyed creatures, tentacled creatures, or creatures with conditional wings. Generation should produce useful editable results rather than opaque one-shot meshes.

The semantic creature model should describe capabilities/body structure rather than merely producing a mesh. Animation, locomotion, collision, equipment, and behavior must be able to consume that semantic information.

### Change notes

- Strengthened animation, locomotion, layered animation, and creature-system integration based on reviewer findings.
- 2026-10-15 (amendment 7at): the Character Studio, first tier. A character is written in the world's `characters.txt`: a model (glTF), a map of its bones to Godot's humanoid names, clip libraries with their provenance and licence, a look (which clip shows what the motor says), its set of moves, what it holds. Clips are retargeted by the maps when a character is made, blended by Godot's AnimationTree as a mixer, and timed by the motor (phase progress, distance walked); the motor alone moves the body. Draw3D's poses baked to a glTF library; a mannequin on another skeleton; a samurai set (`strikes`, `upon struck`) with a katana. Characters > Character Studio: preview, bones, clips searched and auditioned, movement, holds, strikes; walk as. Automated runs leave the person's mouse and controller alone. Not there: an external clip library (proposed: Quaternius, CC0), layering, IK on clips, characters for the folk, damage.

---

# 3. Navigation, Behavior & Simulation

**Status:** ◐ First tier (2026-10-14, amendments 7ag, 7ah and 7ai)  
**Documentation:** ☐

### Purpose

Give characters and creatures the ability to understand and move through the world and to perform reusable behaviors without requiring a giant custom scripting project.

### Capabilities

- ◐ Generate walkable areas.
- ◐ Update navigation when the world changes.
- ◐ Support dynamic navigation changes caused by terrain, doors, destroyed objects, placed objects, weather, gameplay state, or other world changes.
- ◐ Pathfinding.
- ☐ Basic avoidance.
- ◐ Follow.
- ◐ Wander.
- ◐ Flee.
- ◐ Go to location.
- ☐ Stay/guard.
- ◐ Simple routines.
- ◐ World clock/time-of-day awareness.
- ◐ Behavior authoring.
- ◐ Reusable behavior composition.
- ☐ Preview → Simulate → Play behavior workflow.
- ☐ Deterministic simulation/test hooks.
- ◐ Environmental awareness.
- ◐ Interaction with gameplay objectives.
- ☐ Navigation communicates with world state and gameplay state.
- ☐ Persist navigation-relevant world changes such as opened/closed doors, destroyed obstacles, completed areas, and altered routes where appropriate.
- ☐ Behavior systems can consume semantic world/environmental information without hard-coding every subsystem relationship.

### Representative demo

Villagers follow daily routines, sheep graze and scatter, and a dog follows the player. Alter a world route or obstacle and verify navigation and behavior respond appropriately.

### Notes / open questions

Behavior should be reusable and composable rather than requiring one-off logic for every NPC. Navigation must be treated as a living capability that can respond to world changes rather than as a one-time baked map.

### Change notes

- Elevated dynamic navigation updates and behavior composition from review findings.
- 2026-10-14: first tier, in the village (amendment 7ag): ways made from what is solid as it stands (a shut door a wall, an open one a way), doors and gates as ways for whoever can open them (people open and shut them on their way; animals cannot, so a shut pen keeps them), places by name (a structure) in a day kept by the hour, where each is and what happened written down, and a Playtest window to watch it. Wander, follow and flee were there before as single habits (`folk.gd`). Behaviour authoring, composition, conditions and persistence of the changes are not begun.
- 2026-10-14, later (amendment 7ah): conditions (`cond.gd`) and sequences (`sequences.txt`, `work.gd`) as written words, a day's line that is work (`goes HOUR do NAME`), what cannot be done said and retried rather than waited on; the baker's bread and the gardener's apples in the village. Bodies of the folk step up onto a low lip, and ways are made only up what they can step. Reactions and profiles (7g M4c, M4d) not begun.
- 2026-10-14, later (amendment 7ai): reactions laid over work for everyone with a body (`reactions.txt`: who, when, while, how pressing, how often; greet this one otherwise than that one, shelter from rain, come to the walker who waits), with what they interrupt taken up again after; new steps `face`, `move`, `go walker`, `go indoors`; new conditions `near`, `quest`, `walker ...`. Profiles (M4d) not begun.
---

# 4. Environment, Weather, Effects & Audio

**Status:** ◐ Sound, first tier (2026-10-15, amendment 7as): sounds written as recipes, things that sound by their lines, a weather's sound, a Sounds window  
**Documentation:** ☐

### Purpose

Create a living environment in which weather, water, particles, audio, time, and environmental reactions communicate with one another.

### Capabilities

- ☐ Day/night cycle.
- ☐ Time-of-day lighting.
- ☐ Weather systems.
- ☐ Seasons.
- ☐ Rain.
- ☐ Snow.
- ☐ Puddles and surface responses.
- ☐ Snow accumulation on appropriate surfaces.
- ☐ Water/environment responses.
- ☐ Generalized particle/effect system.
- ☐ Arbitrary airborne/falling payloads.
- ☐ Support payloads such as leaves, debris, fruit, fish, frogs, or other objects.
- ☐ Configurable landing behavior: accumulate, bounce, settle, disappear, or custom response.
- ☐ Ambient world effects.
- ☐ Lightweight creature swarms.
- ☐ Butterflies, birds, fish, fireflies, insects, etc.
- ☐ Fire effects.
- ☐ Environmental interaction effects.
- ◐ Positional audio.
- ◐ Ambient audio.
- ☐ Music/audio regions.
- ◐ Environmental sound reactions.
- ☐ Semantic communication between weather, particles, audio, surfaces, NPCs, navigation, physics, and gameplay.
- ☐ Shared environmental semantics for effects such as wetness, snow, wind, temperature, season, and water state.

### Representative demo

A village transitions through time and weather: rain produces visible effects, puddles, sound, altered surfaces, and appropriate creature/NPC behavior.

### Notes / open questions

The particle/effect system should not be limited to conventional visual particles. It should be capable of producing lightweight world events and ambient life.

### Change notes

- Strengthened semantic environmental state and cross-system communication.
- 2026-10-15 (amendment 7as): sound, first tier. A sound is a recipe of tones and hiss written in the world's `sounds.txt` (sixteen there), made into what is heard when first wanted, in the editor and the exported game. Things say `sounds NAME` (once as their `when` comes to hold), `hums NAME [R]` (round and round while it holds, heard R m off) and `knocks NAME [S]` (at a strike, louder the harder); a weather says `sound NAME [N]`. Heard where a thing stands, while the world runs. Window > Sounds lists, plays and opens each, says who sounds it and what is wrong, and sets how loud (this machine's). Library group Sound (a sound post); lab designs given sounds. Not there: recorded sound, music, regions, footsteps, echo and occlusion, effects (particles) as content.

---

# 5. Rendering & Style System

**Status:** ☐ Planned  
**Documentation:** ☐

### Purpose

Create a flexible rendering and style framework capable of producing stylized atmospheric worlds without forcing the engine into one visual pipeline.

### Capabilities

- ☐ Modular rendering framework.
- ☐ Multiple rendering paths.
- ☐ Materials.
- ☐ PBR capability where useful.
- ☐ Stylized rendering treatments.
- ☐ Lighting.
- ☐ Shadows.
- ☐ Fog.
- ☐ Atmospheric effects.
- ☐ Sky/environment rendering.
- ☐ Explicit style definitions and style parameters.
- ☐ World-wide visual style definitions.
- ☐ Regional style overrides.
- ☐ Live style editing.
- ☐ Style influences procedural generation.
- ☐ Style influences generation choices/probabilities where appropriate.
- ☐ Style influences architecture proportions/detail.
- ☐ Style influences vegetation.
- ☐ Style influences characters and creatures.
- ☐ Style influences materials/palettes.
- ☐ Style influences lighting/fog mood.
- ☐ Style influences effects.
- ☐ Style influences rendering treatment.
- ☐ Style information is consumable by generators rather than existing only as a renderer setting.
- ☐ Support multiple meaningful visual styles without simply recoloring the same assets.

### Representative demo

The same island/world can be switched between several substantially different styles, affecting architecture, vegetation, materials, atmosphere, lighting, and generated content.

### Notes / open questions

The intended visual direction is stylized and atmospheric rather than photorealistic. Inspirations include Dorfromantik, A Short Hike, Townscaper, Sea of Solitude, A Juggler’s Tale, Stray, Arise, Submerged, Planet of Lana, and Midnight Walk.

Style is an authoring input and a generation input as well as a rendering concern.

### Change notes

- Made style architecture and style→generation communication explicit based on three-review agreement.

---

# 6. Large Worlds & Procedural Authoring

**Status:** ☐ Planned  
**Documentation:** ☐

### Purpose

Scale the engine from a small scene to large, editable, procedurally authored worlds while preserving identity, provenance, authored changes, and reuse.

### Capabilities

- ☐ Biomes.
- ☐ Regions.
- ☐ Terrain sculpting.
- ☐ Terrain painting.
- ☐ Rivers.
- ☐ Roads.
- ☐ Coasts.
- ☐ Hills.
- ☐ Caves.
- ☐ Dungeons.
- ☐ Route/path drawing.
- ☐ Region painting.
- ☐ Streaming/loading.
- ☐ Caching.
- ☐ Large-world editing.
- ☐ Responsive authoring at large scale.
- ◐ Seamless scale from small spaces to buildings, villages, regions, and large worlds where practical.
- ☐ Procedural generation.
- ☐ Generate → edit → regenerate.
- ☐ Generate variants.
- ☐ Freeze/bake generated content.
- ☐ Reuse generated content.
- ☐ Preserve identity across regeneration.
- ☐ Preserve provenance.
- ☐ Preserve authored overrides.
- ☐ Authoring studios.
- ☐ Asset browser.
- ☐ Save generated/edited content as reusable assets.
- ☐ Asset previews.
- ☐ Asset thumbnails.
- ☐ Better palettes and handles.
- ◐ Generator-driven architecture.
- ☐ Generator-driven roads.
- ☐ Generator-driven vegetation.
- ☐ Generator-driven creatures.
- ☐ Generator-driven characters.
- ☐ Generator-driven props.
- ☐ Generator-driven weather.
- ☐ Generator-driven encounters.
- ☐ Generator-driven populations.
- ☐ Generator-driven effects.
- ☐ Generator-driven gameplay content.
- ☐ Generator-driven styles.
- ☐ Generator dependencies can be represented explicitly.
- ☐ Generator composition/dependency graph.
- ☐ Generator inputs and downstream dependencies are discoverable.
- ☐ Generator versioning or equivalent change tracking.
- ☐ Controlled regeneration of dependent content.
- ☐ Generator controls exposed through appropriate authoring tools.
- ☐ Quality/validation checks for generated content.
- ☐ Support both procedural and manually authored workflows.
- ☐ Support imported content alongside generated content.
- ☐ Support anchor/attachment/relative positioning so manual edits can survive regeneration and upstream changes where appropriate.
- ☐ Avoid blindly rebuilding unrelated downstream content when upstream terrain or generator inputs change.
- ☐ Provide author-friendly generator previews and explanations before committing or regenerating content.
- ☐ Show, where practical, what will be regenerated and which authored overrides are expected to survive.

### Representative demo

A mainland containing two villages with different styles, forest, hills, river, and cave, where terrain and generated content can be edited and regenerated without destroying deliberate authored changes. Change an upstream generator input and verify only the appropriate dependent content changes.

### Notes / open questions

Changing land should not require blindly rebuilding everything. Furnishing should be reusable rather than repeatedly hand-authored. Streaming should preserve authored modifications.

Procedural generation is an authoring accelerator, not a requirement that every game be procedural.

The difficult part is not merely generation. The engine must understand dependencies, provenance, authored overrides, variants, regeneration scope, and the relationship between generated content and downstream authored work.

### Change notes

- Added generator dependency/composition/versioning/control requirements (selected 2/3 finding).
- Strengthened asset/content management, validation, and regeneration preservation.
- Added anchor/relative positioning as a review-derived requirement.
- 2026-10-14: modular construction on a layout's cells brought to a stopping point (amendment 7ad, 7af): structures that adapt to their cells (walls, fences, hedges, buildings with rooms, doors and roofs, ponds, paths, steps), shared sides built once, a village built of them (editor/showcase-village.json). Construction stops there; 7af recommends what to go to next across the engine.
- 2026-10-15: modular construction is **postponed, not cancelled**. The tool-led direction (Tiny Glade and Townscaper), its requirements, what exists and what remains, its dependencies, the decisions made and still open, and the acceptance criteria for resuming are kept in `Draw3D_Modular_Construction_Feedback_and_Next_Systems.md`. Not to be resumed until the user asks.

---

# 7. Items, Inventory, Equipment & UI

**Status:** ◐ First tier of goods, prompts and the quest line (2026-10-14, amendment 7ah); items with their own identity, equipment found and worn, the game's display, inventory screen and menu (2026-10-16, amendment 7bi)  
**Documentation:** ☐

### Purpose

Provide reusable item/equipment systems and a clear separation between editor UI and game UI.

### Capabilities

- ◐ Item definitions.
- ◐ Item instances.
- ◐ Containers.
- ◐ Inventory.
- ◐ Equipment.
- ◐ Equipment attachments.
- ◐ Trading.
- ◐ Saved game inventory/state.
- ◐ Game HUD.
- ◐ Inventory UI.
- ◐ Menus.
- ◐ Controller navigation.
- ☐ Input remapping.
- ◐ Prompts.
- ◐ Notifications.
- ☐ Dialogue UI foundation.
- ◐ Quest/objective UI foundation.
- ◐ Separate editor UI from game UI.
- ☐ Editor UI framework capable of hosting domain-specific tools consistently.
- ☐ Editor inspectors.
- ☐ Editor palettes.
- ☐ Generator controls.
- ☐ Style controls.
- ☐ Terrain tools.
- ☐ Creature tools.
- ☐ Behavior tools.
- ◐ Diagnostics.
- ☐ Asset browser.
- ☐ Natural defaults with deeper customization available.
- ☐ Content discovery and reusable-asset management should be part of the normal authoring workflow rather than an afterthought.

### Representative demo

A character finds an item, stores it, equips it, uses it, saves the game, and restores the same game state later.

### Notes / open questions

The editor should minimize the amount of technical knowledge required for ordinary tasks. Generator, style, terrain, creature, behavior, asset, and gameplay tools should feel like parts of one authoring environment rather than unrelated utilities.

### Change notes

- Strengthened content management and editor workflow requirements from the reviews.
- 2026-10-16 (amendment 7bi): items each with their own id (a thing that says `kept`; `gear ITEM` makes it a thing of gear.txt); the library's Gear to find (18 designs) picked up, kept, equipped: worn or taken in hand by the walker's character, as the Character Studio has gear, with the fit kept for the model; dropped and picked up again; the game's own layer apart from the editor's windows: a display while playing (quest, goods, what was found by key, what is in hand), an inventory screen (I or Back) and a menu (Esc or Start) that hold the game, keyboard, mouse and controller; all of it facts, so a saved game keeps it. The representative demo (find, store, equip, use, save, restore) is `-Demo inventory` and `-Tour -Only inventory`. Not yet: item numbers and wear, slots, others who equip, containers, a game's own menus as content.
- 2026-10-14 (amendment 7ah): goods as counts kept as facts (bread, apples, flour, coins: `goods.txt`), held by things that say `stock` and shown on them, carried by whoever has them; acts that make, use, take and hand them over (trading is an act with both); the walker's goods and the quest step shown while playing; a prompt that says why something cannot be done for want of goods; Playtest's goods panel. No equipment, no item instances apart from the things that already were, no inventory window.

---

# 8. Gameplay Framework

**Status:** ◐ First tier: one complete loop, talks and markers (2026-10-14, amendments 7ah and 7ai)  
**Documentation:** ☐

### Purpose

Provide reusable gameplay rules so complete game loops can be built without writing bespoke engine systems for each game.

### Capabilities

- ☐ Events.
- ◐ Conditions.
- ☐ Triggers.
- ◐ Interactions.
- ☐ Contextual interaction rules.
- ◐ Dialogue.
- ◐ Dialogue/narrative graph where useful.
- ◐ Quests.
- ◐ Objectives.
- ◐ Markers.
- ☐ Progression.
- ◐ Reusable game rules.
- ☐ Gameplay authoring tools.
- ◐ World-state integration.
- ◐ World-clock integration.
- ◐ Inventory integration.
- ◐ AI integration.
- ◐ Navigation integration.
- ◐ Interaction integration.
- ◐ Save/load integration.
- ☐ Weather/environment integration.
- ◐ Physics integration.
- ☐ Explicit physics ↔ gameplay ↔ simulation-state integration.
- ◐ Win/lose conditions.
- ◐ Small complete gameplay loops before the whole engine is finished.
- ☐ World-state changes can affect gameplay, navigation, AI, environment, and persistence through shared semantics rather than isolated special cases.
- ☐ Support persistent progression patterns such as unlocked areas, traversal gates, and changed spaces without requiring a separate Metroidvania-specific system.
- ◐ Leave room for lightweight in-world mini-games (such as cards, match-three, or simple board games) as a later gameplay extension rather than a core engine focus.

### Representative demo

Explore → meet NPC → receive objective → travel → interact → return → receive reward → observe a changed world state → save → reload → continue from the same gameplay state.

### Notes / open questions

Gameplay should begin appearing around the middle of the engine roadmap rather than being postponed until every lower-level system is complete.

The gameplay framework must be able to consume persistent world state and physics/simulation outcomes without forcing game-specific save logic into every subsystem.

### Change notes

- Added explicit physics/gameplay/save-state integration from the selected 2/3 review finding.
- Added contextual interaction and dialogue/narrative graph as review-derived candidates now considered part of the gameplay layer.
- 2026-10-14 (amendment 7ah): the first complete loop, in the village: meet the cook (talk), receive an objective and coins, travel, buy, deliver, return, be paid; the world changed (bread on the table, coins moved) and kept by a save as facts; a quest that fails when supper comes first. Quests and conditions are text (`quests.txt`, `cond.gd`). No dialogue choices, markers, journal or progression.
- 2026-10-14, later (amendment 7ai): talks with answers as text (`talks.txt`: points, what is said by condition, answers with conditions, going on, taking a quest, paying and being given goods, facts), greyed with why when they cannot be given; keyboard, mouse and controller; markers over who asks and where a step is done; a journal (J). No progression, no narrative graph editor beyond the text.
- 2026-10-14 (amendment 7aj): puzzles of blocks on a grid as rules (puzzles.txt) played at whatever says `plays NAME`: a 3D falling-blocks game, a flat one and a colour-matching one from one vocabulary; keyboard and controller, shown for the scheme in use; a best score kept as a fact.

---

# 9. Abilities, Combat, Advanced Movement & Physics

**Status:** ◐ Physics, first three tiers: authorable bodies (7aj); one thing's own physics, colliders drawn, sensors and what goes by them (7ak); mechanisms: joints between things put down, fans and belts, strikes counted, speeds by a number (7al, 2026-10-14); signals you can reason with: conditions, timers, resets, the wiring in a window, joints picked in the view (7am, 2026-10-14); how hard a thing was struck as a number, and Signals rows that lead to their rule (7an); several joints a thing, and what waits kept in a saved game (7ar, 2026-10-15): Signals a first version  
**Documentation:** ☐

### Purpose

Provide broad movement, physical interaction, ability, weapon, combat, and simulation capabilities that can support many genres without committing the engine to being a combat game.

### Capabilities

- ☐ Health/damage.
- ☐ Hit detection.
- ☐ Abilities.
- ☐ Ability costs.
- ☐ Cooldowns.
- ☐ Ability combinations.
- ☐ Hostile behavior.
- ☐ Death/defeat.
- ☐ Respawn.
- ☐ Dodge.
- ☐ Climb.
- ☐ Richer movement.
- ☐ Movement-domain transitions.
- ☐ Alternate movement modes.
- ☐ Vehicles.
- ☐ Weapons.
- ☐ Equipment interaction.
- ◐ Static rigid bodies.
- ◐ Dynamic rigid bodies.
- ◐ Joints.
- ◐ Forces.
- ◐ Gravity.
- ☐ Wind.
- ☐ Overlap tests.
- ◐ Triggers.
- ☐ Raycasts.
- ☐ Shapecasts.
- ☐ Cloth/fabric simulation.
- ☐ Cloth interaction with animated entities.
- ☐ Cloth interaction with colliders.
- ☐ Cloth constraints.
- ☐ Per-vertex cloth data.
- ◐ Physics-driven gameplay.
- ◐ Physics-driven puzzles.
- ☐ Parkour/momentum systems.
- ☐ Precision traversal/movement support.
- ☐ Support movement features important to traversal gameplay, such as acceleration/friction tuning, jump buffering, coyote time, air control, momentum preservation, responsive turning, and other genre-appropriate movement affordances.
- ☐ Naval/vehicle movement.
- ☐ Generic movement/ability/combat systems reusable across genres.

### Representative capability tests

- ☐ Physics platformer.
- ☐ Precision platformer/traversal test.
- ☐ Parkour/traversal game.
- ☐ Top-down combat.
- ☐ Vehicle gameplay.
- ☐ Naval gameplay.
- ◐ Physics puzzle.

### Notes / open questions

Combat is an engine capability, not a declaration that Draw3D is a combat-game engine. Movement and interaction systems should be general enough to support many genres without one-off engine code.

For traversal-heavy games, movement quality is not cosmetic polish. The movement system must support deliberate, precise, responsive play where timing, momentum, jumps, traversal chains, and recovery behavior are authorable and tunable.

Vehicles and characters should expose semantic movement capabilities that can connect to terrain, water, air, physics, navigation, input, AI, effects, and gameplay where appropriate.

### Change notes

- Added explicit precision-movement requirements after the three-review pass because this is especially important to the intended delivery/parkour direction.
- 2026-10-14 (amendment 7aj): physics as content. What a body is made of and how it moves is lines of its kind (bounce, grip, damping, a start speed, fast-moving, eight collision groups, a hinge to the world; and of fixed things: kicks, turns, glides; a ball collider); the world's gravity, speed and steps in physics.txt; anything put down can be raised; a Physics window watches and tunes bodies while playing (held and stepped); the library's Physics lab and a lab world built only of it; a puzzle of falling blocks as rules in puzzles.txt whose cleared blocks burst off as bodies. Godot's Jolt does the simulating. Per-thing physics, body-to-body joints, steady forces, raycasts and overlap triggers a person can use are not yet there (7aj's gaps).
- 2026-10-14, later (amendment 7ak): one thing put down may say its own physics over its design's (its own lines, in the kind's words, kept by the host and laid over the design's when it is made): the Inspector's Physics part shows each number beside the design's, sets and takes back, and gives a thing a collider of its own (a ball, box or post round it, sized there); colliders drawn as lines in the view while editing and playing (View > Colliders); the Physics window keeps a tuned number for one thing alone; sensors (`senses NAME W H D [counts]`) write a fact, and what kicks, turns or glides waits for one (`when NAME`): a goal lights a lamp and starts a spinner, all from library designs. Still not there: joints between bodies, fields, hits as facts, speeds by facts, own physics for what a maker makes. Next physics milestone: bodies joined, and the world acting on them.
- 2026-10-14, later still (amendment 7al): mechanisms. A thing that is loose may be joined (a hinge, a slider, a spring, a weld; limits; a motor) to the world, as its design says, or to another thing put down by hand, named in its own lines or chosen in the Inspector from what is near it (renamed, the joint follows; copied, the copy is joined too). Forces: `blows` (a fan, an updraft) and `carries` (a belt). Signals: `hits NAME` counts strikes, and `by NAME` drives any speed (a motor, turning, gliding, blowing, carrying) by a number of the world's. Library > Mechanisms (eleven designs); the lab's bell machine (a belt drops parcels into a chime that counts them, and a wheel speeds up, a gate opens and a fan blows two welded crates by the count), built from designs and lines; joints and fans drawn. Still not there: one joint a thing only, joints picked in the view, thresholds and timers on numbers, joints that break, the world's wind through fans, physics for what a maker makes.
- 2026-10-14, last (amendment 7am): signals you can reason with. `when` takes a condition (a count at least N, compared, not, and `for S`: once it has held so long), one reader for every door, lamp, moving part, fan, belt and motor and for the texts' conditions; `clears` takes facts back and `ticks` counts seconds, while a thing's `when` holds. Window > Signals: every fact, what writes it and what goes by it, values and what holds while playing, what is wrong (a name nothing writes, a count never reached), the wiring drawn in the view. A joint's other thing and anchor clicked in the view. The lab's batcher (belts, a chime, lamps, a gate, a timer and a reset post) runs in rounds, of library designs and lines alone. Still not there: and/or in one condition, what waits kept in a saved game, several joints a thing.
- 2026-10-14, after it (amendment 7an): `feels NAME [S]`, the world's NAME the speed of the last strike (a strength pad in the library; a strength tester in the lab: lamps at 3, 6, 9 m/s and a reset); a Signals row of a text opens F10 at its line.
- 2026-10-15 (amendment 7ar): Signals finished to a first version. A thing may be joined several ways (a `joins` line each; the Inspector chooses which and adds another; a bridge of planks held at both ends in the lab and the library); a saved game keeps what waits (how long a `for` has held, how far a `ticks` is to its next). The batcher and the strength tester tried by hand by the user. Still not there: joints that break, joint anchors and axes dragged in the view, and/or in one condition, signals far from the walker.

---

# 10. Genre Proof & Standalone Shipping

**Status:** ☐ Planned  
**Documentation:** ☐

### Purpose

Prove that the completed engine can support genuinely different games and can produce a self-contained build that can be played outside the development environment.

### Capabilities

- ☐ First-person mode.
- ☐ Third-person mode.
- ☐ Top-down mode.
- ☐ Fixed-camera mode.
- ☐ Game title/start flow.
- ☐ Game rules.
- ☐ Win/lose flow.
- ☐ Standalone build.
- ☐ External playtesting.
- ☐ Self-contained runtime.
- ☐ Multiple genuinely different small games.

### Genre gauntlet

- ☐ Third-person exploration.
- ☐ Top-down RPG/dungeon.
- ☐ Platformer.
- ☐ Precision platformer/traversal.
- ☐ Puzzle game.
- ☐ Card/strategy game.
- ☐ Delivery/parkour game.
- ☐ Naval battler.
- ☐ Physics-focused game.
- ☐ Book-of-Travels-style spatial diorama.
- ☐ Metroidvania-style game.

### For each genre test, record

- ☐ What existing systems handled naturally.
- ☐ What required special-case code.
- ☐ What authoring workflow was missing or inconvenient.
- ☐ What defaults were wrong or missing.
- ☐ Where systems failed to communicate.
- ☐ Which new capability should become reusable engine functionality.
- ☐ Which gaps were genuine engine limitations versus genre-specific preferences.

### Representative delivery/parkour concept

Movement mastery is the means of solving varied objectives rather than being the objective itself.

Potential objective types:
- ☐ Delivery.
- ☐ Multi-delivery.
- ☐ Retrieval.
- ☐ Steal.
- ☐ Rescue.
- ☐ Intercept.
- ☐ Recover.
- ☐ Boss retrieval.

Example boss encounter:
A boss has four tentacles, each holding one required item. The player must use traversal and movement abilities to reach and recover the items.

### Notes / open questions

The genre gauntlet should contain different kinds of games rather than becoming a collection of platformers.

Card/strategy is useful specifically because it tests whether the gameplay/UI/data model can support non-spatial rules without requiring Draw3D to become a generic UI application framework.

### Change notes

- Strengthened the platformer/traversal benchmark with precision movement.
- Kept card/strategy as a deliberate stress test rather than assuming it is automatically a core strength.

---

# Cross-Cutting Systems & Principles

These are not necessarily separate implementation phases. They are requirements that should remain visible throughout development.

## A. Stable Identity & Provenance

**Status:** ☐ Planned  
**Documentation:** ☐

- ☐ Stable IDs for world/content entities.
- ☐ Preserve identity across regeneration.
- ☐ Track generated provenance.
- ☐ Track authored provenance.
- ☐ Support overrides.
- ☐ Support nested generated content.
- ☐ Support detached/frozen content where appropriate.
- ☐ Avoid using host scene paths as the authoritative identity model.
- ☐ Preserve identity through save/load and standalone builds.

## B. Definition → Generation → Editing → Variant → Reuse

**Status:** ☐ Planned  
**Documentation:** ☐

- ☐ Define reusable content.
- ☐ Generate content from definitions.
- ☐ Edit generated results.
- ☐ Preserve deliberate edits.
- ☐ Create variants.
- ☐ Save results as reusable assets.
- ☐ Regenerate without unnecessarily destroying authored work.
- ☐ Make this workflow understandable to users.
- ☐ Define what happens when an upstream definition/generator changes.
- ☐ Preserve anchors/relationships where downstream authored content depends on generated content.

## C. Generated + Authored Coexistence

**Status:** ☐ Planned  
**Documentation:** ☐

- ☐ Generated objects and manually authored objects can coexist.
- ☐ Generated content can be manually adjusted.
- ☐ Manual content can coexist with generators.
- ☐ Imported content can coexist with generated content.
- ☐ Regeneration respects authored changes.
- ☐ Regeneration can identify what should be preserved, rebuilt, detached, or invalidated.

## D. Semantic Communication Between Systems

**Status:** ☐ Planned  
**Documentation:** ☐

Subsystems should communicate meaning rather than isolated implementation details.

Examples:
- ☐ Rain can affect particles.
- ☐ Rain can affect audio.
- ☐ Rain can affect surfaces.
- ☐ Rain can affect puddles.
- ☐ Rain can affect NPC behavior.
- ☐ Weather can affect navigation.
- ☐ Weather can affect gameplay.
- ☐ World time can affect lighting.
- ◐ World time can affect routines.
- ☐ Equipment can affect animation.
- ☐ Gameplay objectives can affect navigation.
- ☐ World state can affect navigation.
- ☐ Physics can affect animation/effects/gameplay.
- ☐ Physics/simulation outcomes can become persistent gameplay state when intended.
- ☐ Style can influence generators and rendering.
- ☐ Generated creature semantics can inform animation, locomotion, collision, equipment, and behavior.

### Interface principle

- ☐ Define explicit system boundaries/contracts for important cross-system relationships.
- ☐ Prefer shared semantic interfaces and dependency relationships over hard-coded knowledge of every other subsystem.
- ☐ Keep cross-cutting communication extensible enough that adding a new subsystem does not require rewriting unrelated systems.

## E. Procedural Generation as an Authoring Accelerator

**Status:** ☐ Planned  
**Documentation:** ☐

- ☐ Procedural generation should save authoring time.
- ☐ Procedural systems should not force procedural games.
- ☐ Manual workflows remain first-class.
- ☐ Imported content remains first-class.
- ☐ Generated results should be editable.
- ☐ Useful generated content should be reusable.
- ☐ Generation should expose meaningful controls without requiring users to understand internal generator implementation.
- ☐ Generated content should be validated before being treated as trusted content where practical.

## F. Natural Defaults + Deep Customization

**Status:** ☐ Planned  
**Documentation:** ☐

- ☐ Common operations should work without extensive configuration.
- ☐ Advanced controls should remain available.
- ☐ Minimize unnecessary setup/inconvenience.
- ☐ Avoid forcing users through technical details for ordinary tasks.
- ☐ Domain-specific authoring studios should expose useful controls directly.
- ☐ Generator controls should expose author-friendly concepts rather than only technical parameters.
- ☐ Common workflows should be tested by actually using the editor, not merely by checking that a feature exists.

## G. Automated Testing & Development Instrumentation

**Status:** ☐ Planned  
**Documentation:** ☐

- ☐ Automated deterministic tests.
- ☐ Scripted tours.
- ☐ Simulated input.
- ☐ Checkpoints.
- ☐ Replay where useful.
- ☐ Regression testing.
- ☐ Physical user input isolated from automated tests.
- ☐ Development diagnostics.
- ☐ Capability probes.
- ☐ Real-content testing instead of purely speculative benchmarks.
- ☐ Content validation/quality checks.
- ☐ Tests that exercise serialization and world-state restoration.
- ☐ Tests that exercise generator regeneration and authored override preservation.
- ☐ Tests that exercise important cross-system contracts.

Detailed test history can remain outside this roadmap.

## H. Performance & Streaming

**Status:** ☐ Planned  
**Documentation:** ☐

- ☐ Measure performance using real representative content.
- ☐ Streaming/loading.
- ☐ Caching.
- ☐ Large-world memory management.
- ☐ Editor responsiveness at large scale.
- ☐ Avoid premature optimization based only on synthetic assumptions.
- ☐ Investigate bottlenecks when actual content demonstrates them.
- ☐ Preserve responsiveness during authoring.
- ☐ Avoid unnecessary full-world rebuilds after local changes.

## I. Asset & Content Ecosystem

**Status:** ☐ Planned  
**Documentation:** ☐

- ☐ Asset/content browser.
- ☐ Save-as-asset workflow.
- ☐ Thumbnails.
- ☐ Previews.
- ☐ Reusable generators.
- ☐ Generator definitions and versions are discoverable.
- ☐ Reusable generated results.
- ☐ Imported assets.
- ☐ Generated assets.
- ☐ Editable assets.
- ☐ Content identity/provenance visible where useful.
- ☐ Content dependencies visible where useful.
- ☐ Discoverability of reusable content.
- ☐ Content validation/status information where useful.
- ☐ Consider import/export standards as an interoperability concern without turning Draw3D into a separate external asset pipeline.

## J. Host vs Draw3D Ownership

**Status:** ☐ Planned  
**Documentation:** ☐

Principle:

> **Borrow the host where it is infrastructure. Own the systems that define the engine’s identity.**

- ☐ Rust core remains host-independent where practical.
- ☐ Godot acts as host/infrastructure layer.
- ☐ Core world model is not dependent on Godot scene paths.
- ☐ Draw3D owns distinctive authoring/generation/identity systems.
- ☐ Draw3D can use Godot renderer/physics/animation/audio/UI/input where appropriate.
- ☐ Host-specific details remain behind adapters where practical.
- ☐ Avoid rebuilding infrastructure solely for the sake of owning it.

---


## K. Explicit Semantic Interface Contracts

Major systems should expose meaningful semantic information and consume shared capabilities without hard-coding every other subsystem. Important relationships should be inspectable and diagnosable.

Examples include World State ↔ Navigation, Style ↔ Generation ↔ Rendering, Physics ↔ Gameplay ↔ Save State, Equipment ↔ Animation, Weather ↔ Environment/AI, and Movement ↔ Character/Creature/Vehicle capabilities.

## L. Generated Content Validation

Procedural output should be able to report invalid or suspicious results, including broken references, impossible placements, missing attachments, unreachable spaces, navigation failures, collision problems, and generator outputs outside expected constraints. Validation should support authoring rather than becoming a separate enterprise-style validation system.

## M. Movement as an Engine Signature

Movement is a reusable authoring capability, not a mandatory behavior. Draw3D should make it possible to build simple movement easily and sophisticated movement without one-off engine code. Precision, responsiveness, animation integration, traversal chaining, movement domains, and recovery behavior should be treated as engine-quality concerns because they strongly affect the player's perception of games made with Draw3D.

## N. Authored Movement, Action, Interaction and Presentation

Added 2026-10-08 as a standing architectural requirement. It extends M and the review decisions "contextual interactions tied to actor/object/state" and "rich character/creature animation and movement authoring". It does not reorder the development order, and it is not a framework to be built ahead of the cases that need it.

> **Draw3D should make authored movement, action, interaction, and presentation reusable across entities rather than requiring bespoke engine code for every new character, creature, object, or game mechanic.**

These are foundational parts of the engine's creative authoring model, not late-stage polish. They need not all exist early, but the architecture must leave the place for each to become a first-class system.

- ☐ **Move sets are assignable content.** A move set is the whole movement/action vocabulary of an entity (human, parkour, samurai, armored, orangutan, duck, horse and rider, a vehicle where the same principles hold): its plain locomotion, its numbers, its moves, and how each looks. Assigning a different set to an entity needs no engine code.
- ☐ **A move is deeply authorable.** Start and entry conditions, phases, timing, acceleration, velocity, momentum kept or cancelled, gravity and gravity curves, air control, trajectory, steering, impulses, contacts and anchors, contact windows, interruption, branching, chaining, transitions, landing, follow-through; body orientation, pose, limb positions, look direction, anticipation, secondary motion, effects/audio/trails; and the gameplay consequence it produces.
- ☐ **What a move does and how it looks stay connected.** The movement authority and the presentation may be technically distinct, but they are authored against each other: a jump has compression, push-off, flight, landing, recovery, not only an upward velocity.
- ☐ **Interactions are authored actions: Actor + Target + Action.** The target exposes capabilities (push, pull, pick up, carry, put down, throw, open, mount...). The actor supplies an action that fits its body. The action establishes the spatial relationship (approach point, interaction point, grip/contact point, relative orientation, direction, attachment or constraint) and then controls behavior and presentation (reach, grip, brace, exertion, release, back to locomotion). Physics may take part underneath; an incidental collision is not the interaction model.
- ☐ **The same model holds across the world:** crates, barrels, doors, switches, levers, weapons, tools, NPCs, creatures, horses, vehicles, ropes, ladders, ledges, items. Different actors resolve the same capability differently.
- ☐ **Interaction presentation is not optional polish.** Picking up, putting down, opening, pushing, wielding, using, climbing each can express approach, contact, the act, release, and return.
- ☐ **A central Movement / Action hub in the editor:** for every entity, the movement and action capabilities it has, from which the right specialized editor opens. A discoverability and management layer, not one editor for everything.
- ☐ **Human authoring and LLM authoring use one representation.** A move set is structured, inspectable content reached through the Draw3D content/API system, not generated code.

Added 2026-10-08 (second direction, with stage 9 under way):

- ☐ **Creatures take part in the same interaction model.** A creature in the world is not decoration: each kind eventually offers at least one meaningful interaction that suits it (sheep, cat, dog: pet; horse: pet, mount; cow: pet or interact; NPC: talk, interact). They may be simple at first. They are not a separate category of object: they are targets of the same Actor + Target + Action model, and real cases for it.
- ☐ **Collision is told apart from intentional interaction.** Low-level physics (walking into a crate moves it) stays as physics. Player + Crate + Push, Player + Door + Open, Player + Sheep + Pet each resolve into an authored action: approach, contact/grip/position, body movement, the behavior itself, physics where it takes part, and animation.
- ☐ **Everything built as an example is something the user can make and change.** Claude-made content (a sheep's move set, a parkour or samurai set, a vehicle, a creature interaction, a weather effect, an animation) demonstrates the framework; it is never the only way to use it. The user can eventually inspect it, duplicate it, modify it, make another version, borrow pieces of it, or make their own from scratch. For move sets in particular: create a set, copy one, borrow selected moves from another, edit a move, create new moves, change movement behavior, change animation/presentation, assign sets to entities. The same for interactions and their animations.
- ☐ **Keyboard/mouse and gamepad are both first-class in demos and playtest scenes.** Clear keyboard controls, clear gamepad controls with the actual buttons and axes documented, and an easy way to switch the displayed/tested scheme, as one shared facility rather than per-demo documentation, so the same demo can be played with one and then the other and compared.

Method: prototype → use real cases → discover the abstraction → extract it into reusable content and systems. Neither a framework first, nor one-off demos retrofitted later. Where each of these stands, the seams being kept for them, and which real case is expected to teach which piece, is recorded in `PROGRESS.md` ("Moves, actions and interactions: where they stand").

## P. Composable Makers: the Thing Made, and What Arranges It

Added 2026-10-10 as a standing architectural requirement. It makes B, C, E and I concrete for world building, and it is the frame the specialised engines (trees, rocks, water, houses, decoration, gardens) are to be built in. It does not ask for those engines now.

> **Almost anything that can be placed or made by hand in a world should be able to have a generator of its own, and generators should work together through reusable, composable systems. The thing being made is kept apart from the system that arranges it.**

- ☐ **World objects and instances.** Individual things that are placed, inspected, moved, turned, customised, made twice and taken away, whether a hand or a maker put them there.
- ☐ **Makers.** Editable nodes of a world that make things, change them, or arrange them. A maker is a thing of the world itself: listed, added, named, placed, connected, made twice, taken out, with its settings kept, undone and saved like anything else. Not every thing needs a maker, and not every maker needs to be complicated.
- ☐ **Connections.** What one maker reads from another is said outright, by named inputs and outputs, with the checks each kind of input needs. A street reads a line the way any maker reads any other: no pairing is a private mechanism, and no world arrangement is written in code to join two makers.
- ☐ **What arranges is apart from what is arranged.** A street arranges houses; a house engine makes and varies a house; a decoration system supplies and arranges what stands about them. Each can be edited without the others being lost: changing the street does not throw away a house's own windows or what was put down by hand beside it.
- ☐ **Specialised engines** (to come, each its own milestone): tree and vegetation design, rock design, water bodies with their own shapes and levels, house design, decoration and furniture, gardens. Each is used through the same makers, connections and placement, so that none needs a world arrangement of its own.
- ☐ **Reusable compositions** (to come): several makers and assets as one thing that is placed, customised and used again elsewhere without losing its structure (a pond with its bank, its plants, its frogs and its fishing spot).
- ☐ **No world depends on one arrangement written in code.** What a demonstration world begins as is a composition said to the same commands an editor has.
- ☐ A creator's workflow, for any of it: **Define → Generate → Edit → Create Variations → Reuse** (principle B).

## Q. Design, Save, Group, Generate or Paint, Customise, Reuse

Added 2026-10-10 as a standing architectural requirement, after the first world was built by hand from makers. It says what principle P's "thing made" is to become. The aim is not photorealism: it is stylised things with a readable shape, made deliberately, varied meaningfully, under the creator's control. It does not ask for the engines now; the order they come in is below, and where each stands is in `PROGRESS.md`.

> **What a thing is, is designed and kept apart from where things are. A creator designs a recognisable thing, saves the design, groups designs, paints or generates them into a world, changes any one of the results by itself, and uses all of it again elsewhere.**

**The parts, and what each is responsible for** (logical responsibilities: not each a separate engine or plugin):

- ☐ **Designs.** A reusable description of one sort of thing: a flower, a tree, a tuft of grass, a rock, a house, a bench. It has parts a person recognises (a flower has petals, a centre, a stem and leaves; a tree a trunk, branches and foliage; a house floors, a roof, windows and doors) and numbers that shape them. Built-in designs are starting points that already read as what they are, never shapeless; a creator's own are made from them or from nothing, and are not limited to a list of species or styles the code knows.
- ☐ **Groups.** A named set of designs with how much of each: a meadow's palette, a street's house styles. A group is given wherever a design is.
- ☐ **Makers.** What arranges: scatters over an area, lines up along a line, or puts down under a brush. A maker is given a design or a group; it does not contain the code that draws a petal or a window. (Principle P.)
- ☐ **Instances.** Each thing in the world is itself: chosen, moved, turned, changed and taken away by itself, whichever maker put it there. What was done to one is kept by name and survives its maker making its things again; it can be seen and taken back.
- ☐ **Arrangements kept.** Several makers and designs as one thing put down again: a garden, a pond, a house plot, a village.
- ☐ **Where a maker makes its things is its own, and is said apart from what it makes.** How far it reaches, the level it keeps to, the ground it was painted over: a creator says "here, and not there" without rebuilding anything else, and what stands follows the land as it is. (Begun 2026-10-11: a reach, a level, a painted area for a wood and a meadow. A general selection of terrain, by slope, height, surface or another maker's edge, is to come.)
- ☐ **A maker that arranges structure holds more than one sort of thing.** A street stands houses, and beside them trees, rocks, fences and whatever else it is given: a design added to a maker is added, and takes nothing away. Putting one thing in place of another is an operation of its own, asked for and said. (Today a street given designs has no houses: recorded as owed.)
- ☐ **A style says what things are made of, not only their colours.** The look of a world reaches geometry and composition: which trees, what lies under them, what a meadow is strewn with; later materials, ground cover and the palette of a whole environment. It does so by the same designs and groups a creator gives a maker by hand, never by a private list in code. (Begun 2026-10-11: one style says what a wood and open ground are made of.)
- ☐ **A style is chosen for each sort of thing, and they are mixed.** The trees of one style with the houses of another and the ground of a third: a world's look is a choice for each category (and in time for one region, or one maker), not one switch for everything. (Not begun: today a world has one style, and a maker is given other designs by hand to differ from it.)
- ☐ **What things do** (later): bending in wind, the seasons, weather on what grows, giving way to what walks through. Added as the engines for it mature, to designs that were made so it could be.

**Three kinds of difference, never confused:** *another design* (a flower with other petals) is a new asset; *variation of one design* (taller, paler, older, turned) is a number a maker or a brush gives each instance, and the instance is still that design; *where and how many* is the maker's.

**One thing may differ from what its maker makes** (begun 2026-10-11): given another design, a size, or a design that is its own and no other's; and a maker's arrangement may be given one more thing by hand. Each is the maker's to keep, by the thing's name, through any change of its numbers, and each is listed and taken back. A duplicate of a thing is a thing by itself and belongs to no arrangement; "one more of this maker's" is another action and is never a mode of it.

**Requirements that hold for every engine:**

- ☐ Stable identity and provenance for everything generated (A): which maker, which design, under what name.
- ☐ Changes by hand that survive compatible regeneration (C), listed and reversible.
- ☐ Designs and groups usable in other worlds (I): kept in files a world brings in, not in one scene.
- ☐ Generation that makes again only what changed (H), and never destroys what a person authored.
- ☐ Every step usable with no language model (E).
- ☐ Painting and generating side by side: a brush for the precise and local, a maker for the broad; the same designs and groups under both.

**The order, by what depends on what:**

1. ◐ Choosing reliably; one instance worked on by itself (W1b, W1c; D2's first part 2026-10-11: one thing's design, size and own shape, and one more for an arrangement).
2. ◐ A design as an asset a maker is given (D1, 2026-10-11); designs, makers, instances and groups told apart everywhere (groups: not begun).
3. ◐ Basic reusable designs: what the code draws today, as designs to start from (D1; and a first family made to be told from another by its outline, the conifers and what lies under them, 2026-10-11).
4. ☐ Plant and tree design: flowers, then trees, bushes, rocks.
5. ☐ Grass as blades and tufts with a real silhouette; vegetation painted onto the land.
6. ☐ Plant groups, and a meadow generated from one.
7. ☐ Trees distributed, and woods with more to them (kinds by ground and height, clearings, edges).
8. ☐ House design; a neighbourhood whose houses are each changed by themselves.
9. ☐ Decorations, gardens and arrangements kept and reused.
10. ☐ Wind, seasons, weather and physical response, as rendering, animation, physics and weather mature.

## R. Variation: One System for Designs, Groups, Populations and Compositions

Added 2026-10-11 as a standing direction. Nothing of it is built; it is written down so that what is built before it does not close it off. Today each maker varies what it makes in its own way (a seed, which of a sort, shares); the engines to come (plants, houses, creatures) would each grow another. They are to share one.

> **Whatever a creator made can be asked for again, otherwise: looked at beside what there is, taken or refused, with the parts they like held as they are.**

- ☐ **Variation of one design.** Its numbers given ranges; an instance takes a value in them from its own name, the same every time (principle Q, "variation of one design").
- ☐ **Variation of a group or a composition.** Which designs, in what shares; a forest composition asked for again with other trees, shrubs and rocks.
- ☐ **Variation of population and structure.** How many, how they are laid out, and how they stand to one another (a clearing, an edge, a path between).
- ☐ **Locks.** A design, a property, an instance or a part of a composition held as it is while the rest is made again: the flowers kept, the trees, shrubs and rocks varied. A lock is of the thing by its name, as a change by hand is.
- ☐ **Alternatives shown before they are taken.** Several, side by side or in turn, each accepted or rejected; refusing all leaves what there was untouched. Nothing a creator has is destroyed to look at another.
- ☐ **Colour varied as one system, over an authored palette.** A design keeps the palette its maker gave it; a population of it takes coherent variants of that palette (lighter, warmer, an autumn few) in proportions a creator sets, the same for trees, houses and creatures; and any one instance can be given a palette wholly its own. Never a colour jitter that loses the authored base. (First tier built 2026-10-15, amendment 7ao: `vary` lines by role over the style's palette, set on a design, a group, a maker and one thing, drawn by each thing's own seed and kept; see `PROGRESS.md`.)
- ☐ **Reproducible.** A seed names an alternative; weighted choices and constraints (never this beside that, at least one of those) where a kind of thing needs them.
- ☐ **Living things too.** A fauna preset that gives different but fitting kinds of animal from one forest to another, each animal its own variant, with how many and where: the same system, over creatures and behaviour profiles.

It rests on what exists: stable names (A), changes by hand kept by name (C), designs and groups as assets (Q), and a maker's parameters asked about before they are set (`preview`). It is not to be built inside one engine and copied to the next.

# Development Order

This is the proposed implementation order for Claude to review. It is deliberately dependency-driven, but not rigid: real content can cause a phase to move or split.

### Phase 1 — Persistent World Foundation

1. ☐ Persistent Interactive World
   - World Data vs World State
   - stable identity and serialization/save format
   - Preview → Simulate → Play
   - contextual interaction foundation
   - authored/runtime separation

### Phase 2 — Character, Creature & Movement Foundation

2. ☐ Characters, Creatures & Animation
   - semantic entities
   - animation/layering
   - reusable movement primitives and authorable movesets
   - precision movement foundations
   - attachments and creature body semantics

3. ☐ Navigation, Behavior & Simulation
   - dynamic navigation
   - behavior composition
   - world-clock/environment awareness
   - persistence of navigation-relevant world changes

### Phase 3 — Living World & Visual Identity

4. ☐ Environment, Weather, Effects & Audio
5. ☐ Rendering & Style System
   - establish style early enough to influence representative content
   - Style ↔ Generation ↔ Rendering relationship tested, even before full procedural authoring is complete

### Phase 4 — Procedural Authoring & Content Ecosystem

6. ☐ Large Worlds & Procedural Authoring
   - identity/provenance
   - generated + authored coexistence
   - generator dependencies
   - anchor/relative positioning
   - regeneration scope
   - generator authoring UX
   - content validation
   - asset/content management

7. ◐ Items, Inventory, Equipment & UI
   - reusable asset/content workflows
   - editor/game UI separation

### Phase 5 — Gameplay & Persistent Simulation

8. ◐ Gameplay Framework
   - events/conditions/interactions
   - world-state-driven progression
   - physics/gameplay/save-state integration
   - small complete gameplay loops
   - dialogue foundation; narrative graph can mature later

### Phase 6 — Advanced Physical & Traversal Capability

9. ☐ Abilities, Combat, Advanced Movement & Physics
   - advanced precision traversal
   - parkour/momentum
   - vehicles and multi-domain movement
   - physics-driven gameplay/puzzles
   - cloth and richer simulation

### Phase 7 — Genre Proof & Shipping

10. ☐ Genre Proof & Standalone Shipping
   - representative genre gauntlet
   - precision platformer/delivery-parkour
   - naval/vehicle
   - Metroidvania-style persistent progression
   - Book-of-Travels-style spatial world
   - standalone build and external playtesting

### Deliberately Later / Triggered by Real Need

These should be considered, not forgotten, but should not drive early architecture unless a real benchmark requires them:

- ☐ Large-scale editor optimization and aggressive world streaming optimization. First make small/medium worlds work correctly; then scale and measure.
- ☐ Full asset import/export interoperability beyond the minimum host boundary.
- ☐ Generator version migration tooling beyond basic provenance/version tracking.
- ☐ Full dialogue/narrative graph tooling.
- ☐ In-world mini-game framework (cards, match-three, board games, etc.).
- ☐ Maze and obstacle-course generators. Specialised makers, each built on the shared ones when those are ready (lines, areas, designs, groups, connections, variation): a maze is walls and ways arranged by a rule, a course is obstacles along a line with a difficulty. Neither is to be made a general feature early, nor a world arrangement written in code.
- ☐ Card/strategy as a full genre target; retain it as a boundary/stress test.

### Sequencing clarification

Several contracts must exist early even if their mature tooling arrives later:

- World Data ↔ Runtime/Game State
- Serialization ↔ Identity/Provenance
- Identity/Provenance ↔ Regeneration
- Generator Dependencies ↔ Downstream Authored Content
- Anchors ↔ Regeneration/Editing
- Style ↔ Generation ↔ Rendering
- World Changes ↔ Navigation
- Physics ↔ Simulation ↔ Gameplay ↔ Save State
- Semantic Communication ↔ Explicit Interfaces
- Generated Content ↔ Validation
- Movement ↔ Character/Creature/Vehicle capabilities
- Authoring Principles ↔ Actual Editor Workflow

These are not necessarily separate phases. They should be exercised through small representative content as early as practical.

### Important sequencing principle

Do not wait until the end to make the engine feel like a game. Small meaningful gameplay loops should appear during development once the foundational world, characters, movement, navigation, and interaction systems exist.

# Major Checkpoints

## Checkpoint A — Persistent World

- ☐ World saves and loads.
- ☐ Serialization preserves identity/provenance.
- ☐ Authored and runtime state are separate.
- ☐ Preview/Simulate/Play workflow exists.
- ☐ Basic contextual interaction works.
- ☐ Player starts work.
- ☐ Interiors/doors work.

**Documentation:** ☐

## Checkpoint B — Living World

- ☐ Characters and creatures exist.
- ☐ Animation and locomotion work.
- ☐ Navigation works and responds to world changes.
- ☐ Basic behaviors work.
- ☐ World time works.
- ☐ Environment/weather begins communicating with world systems.

**Documentation:** ☐

## Checkpoint C — Authorable World

- ☐ Procedural terrain/world tools work.
- ☐ Generated and authored content coexist.
- ☐ Identity/provenance/overrides work.
- ☐ Generator dependencies/regeneration behavior are understandable.
- ☐ Style system works.
- ☐ Style influences generation as well as rendering.
- ☐ Assets/content can be saved/reused/discovered.
- ☐ Generated content can be validated.
- ☐ Larger worlds can be edited.

**Documentation:** ☐

## Checkpoint D — Game Framework

- ☐ Inventory/equipment works.
- ☐ UI framework works.
- ☐ Gameplay events/conditions/objectives work.
- ☐ Contextual interactions work.
- ☐ Dialogue/narrative flow works where required.
- ☐ Save-game state works.
- ☐ Physics/simulation state can participate in persistent gameplay state where appropriate.
- ☐ A small complete gameplay loop works.

**Documentation:** ☐

## Checkpoint E — Broad Capability

- ☐ Advanced movement works.
- ☐ Precision traversal movement works.
- ☐ Physics capabilities are mature enough for representative games.
- ☐ Abilities/combat/equipment work.
- ☐ Vehicles/alternate movement can be authored.
- ☐ Multiple genre capabilities work without one-off engine architecture.

**Documentation:** ☐

## Checkpoint F — Engine Proof

- ☐ Genre gauntlet completed.
- ☐ Capability gaps documented.
- ☐ Missing workflows/defaults identified.
- ☐ Reusable engine improvements made from benchmark results.
- ☐ Standalone build works.
- ☐ External playtesting completed.
- ☐ Logical how-to/manual documentation produced.

**Documentation:** ☐

---

# Central Development Question

At every major pass, ask:

> **Can this engine express the next meaningful thing naturally, or are we solving around a limitation that should become an engine capability?**

Additional questions:

- Does this capability work with generated and authored content?
- Can it communicate with the other systems it should logically affect?
- Is the common workflow easy?
- Can the result be edited and reused?
- Does it remain useful outside the current demo?
- Are we building an engine capability or a one-off game feature?
- What did real content reveal that the architecture did not?
- What happens to this capability when world state is saved, restored, regenerated, or streamed?
- Does a world change propagate to every subsystem that semantically depends on it?

Before each major stage, for authored movement and action (principle N):

- Does this stage introduce a system that will make authored movement/actions harder to implement later?
- Are we hard-coding behavior that should eventually be data/content?
- Are movement, animation, interaction, physics, and entity capabilities being kept composable?
- Is there an obvious future integration point for reusable move sets?
- Is there an obvious future integration point for intentional interactions?
- Are we preserving enough semantic information to later create proper contact/attachment/animation relationships?

And for every major system added (the engine, not a collection of demos):

> **Can the user eventually inspect it, edit it, create another one, duplicate it, recombine pieces of it, and use the same underlying capability for something that was not specifically anticipated?**

Not every editor or authoring workflow is needed at once. But no foundational system should support only the one example being demonstrated, such that making it authorable later would need a major rewrite.

---

# Final Personal Decisions From Review Candidates

The following decisions were made after reviewing the three independent reports:

### Promoted to explicit engine priorities

- ☐ Precision movement and traversal feel as a reusable, authorable capability.
- ☐ Rich character/creature animation and movement authoring.
- ☐ Generator dependency awareness and a basic dependency/composition model.
- ☐ Anchor/relative-position preservation through regeneration.
- ☐ Basic generated-content validation and diagnostics.
- ☐ Generator authoring UX: preview, parameter control, regeneration visibility, and override-survival visibility.
- ☐ Camera as a flexible capability layer: first-person, third-person, top-down, fixed/diorama, and transitions.
- ☐ Contextual interactions tied to actor/object/state.
- ☐ Dynamic/persistent navigation consequences of world changes.
- ☐ Multi-domain movement semantics for characters, creatures, and vehicles.

### Included, but intentionally later / lighter

- ☐ Asset import/export interoperability beyond the core Godot boundary.
- ☐ Large-scale editor optimization and deeper streaming strategies.
- ☐ Generator version migration tooling.
- ☐ Full narrative/dialogue graph tooling.
- ☐ In-world mini-games.
- ☐ Full card/strategy genre support.

### Guiding rule

If a candidate capability can be introduced as a small, clean foundation that improves the engine's architecture or authoring experience without adding unnecessary complexity, prefer establishing the foundation early and expanding it only when real content proves the need.

# Personal Change Log

Use this only for major changes to the roadmap. Do not turn it into a detailed implementation history.

| Date | Section / Item | Change | Reason |
|---|---|---|---|
| 2026-10-07 | Roadmap review | Incorporated three independent LLM reviews; added consensus findings and selected 2/3 findings. | Strengthen architecture without blindly expanding scope. |
| 2026-10-08 | Cross-cutting principle N; Central Development Question | Added authored movement, action, interaction and presentation as a standing requirement, with six questions to ask before each major stage. Order unchanged. | Hands-on use of stage 8: pushing by collision works mechanically but is not the interaction model wanted. |
| 2026-10-08 | Cross-cutting principle N; Central Development Question | Added: creatures take part in the interaction model; collision told apart from intentional interaction; everything built as an example is user-makeable and editable; keyboard and gamepad both first-class with a shared way to switch what is shown; the "inspect, edit, duplicate, recombine" question. Stage 9 order confirmed: creatures, vehicle, weather. | Direction given with stage 9 under way. |
| 2026-10-09 | Amendment file (`Draw3D_Roadmap_Amendment_2026-10-09.md`) | Principle O (content as data); terrain and geometry editing made concrete; six edit-preservation mechanisms; editor workspaces; next-phase order M1–M4. Master unchanged. | Reconciliation after the nine-stage runtime; verified audit (`Draw3D_Audit_and_Next_Milestone_2026-10-09.md`). M1 approved and built the same day. |
| 2026-10-10 | Cross-cutting principle P; amendment 7d, 7h | Added composable makers as a standing requirement: the thing made kept apart from what arranges it; makers as things of the world; connections said outright; specialised engines and reusable compositions named as later milestones built on them. The basic editor first, the specialised engines after. | Direction given before W1: every capability so far worked only inside the one arrangement the playground is. W1 (makers added, connected, made twice and taken out in the editor) built the same day. |
| 2026-10-10 | Cross-cutting principle Q; amendment 7i | Added design apart from distribution as a standing requirement: designs with recognisable parts, groups, makers and brushes that are given them, instances changed by themselves, arrangements kept; the three kinds of difference; a ten-step order. Nothing of the engines is built; step 1 is begun (W1b). | The user's hands-on review of W1 (2026-10-10): generated things are coarse and cannot be designed; one house of a street could not be told from its street. |
| 2026-10-11 | Cross-cutting principles Q and R; "Deliberately later"; amendment 7j | Q: where a maker makes its things is its own (reach, level, painted area); a structural maker holds houses and other things together, with replacement explicit; a style says what things are made of; one thing may differ from what its maker makes. New principle R: one shared Variation System (designs, groups, populations, compositions; locks; alternatives previewed; seeds, weights, constraints; fauna). Maze and obstacle-course generators named as later specialised makers. Of this only the first rungs of Q are built (see `PROGRESS.md`). | The user's consolidated brief of 2026-10-11 after trying D1: trees off a plateau, design editing unclear, styles that only change colour, and the directions to keep. |
| 2026-10-11 | Cross-cutting principles Q and R; amendment 7k | Q: a style is chosen for each sort of thing, and they are mixed. R: colour varied as one system over an authored palette. 7k: a world’s file is never written over without being kept; a maker’s parameters shown under words where it is edited; commands no control reaches are listed. | The user’s brief before hands-on exploration: expose what exists, keep worlds safe, keep the long-term requirements in view. |
| 2026-10-12 | Amendment 7o | Added structured spatial layout as a second way of building beside freeform editing and makers, in three milestones: L1 spatial layout (square and hex cells, built), L2 tile sets and placement rules, L3 intelligent modular construction. A layout says where and is not content; one module answers every question of cells. | The user's direction of 2026-10-11: tile-based worlds, grid and hex gameplay, and adaptive modular construction, built in stages on one foundation. |
| 2026-10-12 | Amendment file, 7q | Added, as a direction and with nothing of it built: a family of specialised construction tools (ridge, ridge round a mountain, a shelf on a wall, terraces, plateaus, passes, walkable slopes), some making a structure apart from the land with air under it, and reusable generators over tiles, a region or the map (mountain ranges, hills, water bodies and shores, plants, what lives in water) whose results stay editable, saved and reused. Construction, style and simulation kept apart. It widens principle Q and 7i ("construct" beside "generate") and builds on layouts, tiles, the land, the library and makers. Ten dependencies in order, the first being one question of height with tile elevation. Master unchanged. | The user's brief of 2026-10-12, third of the day. |
| 2026-10-12 | Amendment file, "7p, continued" | After the user's third look at tiles. What still flickered while land under tiles was shaped was two things in what the view draws while a mesh is awaited (a ground standing as it stood drawn where its tile now is; one cell of the view changed a frame before its neighbour), both mended in the view; the stripes of a tool on a tile made lighter. Master unchanged. | The same brief. |
| 2026-10-12 | Amendment file, 7p | After the user's second look at tiles. Decided: the land's tools work on the land beneath a tile and say so, and are never turned onto a tile without its being said (a tile's own surface is to be another target); decoration aimed at tiles comes before a tile's own height; a tile's height is never to be written into the land's layer. Built: a tool drawn on a tile's ground and the target named; the four causes of the tearing that remained, each mended in what is drawn (no world or mesh of the land changed); a wood or a meadow told which tiles to make its things on, the first tier of decoration aimed at tiles; where each tile stands worked out in one place, which a tile's height is to be said in. How height and water are asked today is set out there, for elevation. Master unchanged. | The user's brief of 2026-10-12, after trying the pass before. |
| 2026-10-12 | Amendment file, "7o, continued" | After the first use of tiles by hand: the underlying terrain, a tile's elevation and a tile's surface are three things to keep apart. Recorded as to come: raised tile groups and platforms, landforms, water and shorelines, transitions between neighbouring tiles, decoration aimed at tiles or regions; with what in the code leaves a way to them and six risks to decide before a tile is given height (one height assumed everywhere; the land's layer is not the place for a tile's height; one water level). Built in that pass: a tile's ground drawn through the land's own triangles, so it lies on the land however the land is shaped. Master unchanged. | The user's closeout brief of 2026-10-12. |
| 2026-10-12 | Amendment file, 7o | L2 built: terrain tiles. A tile is a maker put down in a cell of a layout (its ground and authored things), kept in sets of files beside the library; sides say what they meet and a set says what may meet; neighbours of another ground blend; cells hold several things. What L2 still owes (ground with height, rules that choose, layers, edges as places) and a region of cells selected and turned together are recorded there. Master unchanged. | The user's L2 brief. |
| 2026-10-12 | Amendment file, 7r | After the user's fourth look at tiles. Tiles flickered as the eye moved because each tile's ground went 2 cm past its side onto its neighbour's: two sheets in one place, drawn one or the other dot by dot wherever they were not one colour. No ground goes past its side now (the two are cut at it through the same triangles and told the same heights), and a tile's rim is lit as its ground. Recorded and not mended: the colour two tiles disagree about near a corner of three or four grounds (one colour for a place, designed), and tiles found standing in one cell after a layout was changed. |
| 2026-10-12 | Amendment file, 7s to 7u, and more of 7q | Added, as traced and as directions, with nothing of them built: what paint paints on (the land, a tile's own surface kept apart from the land, later other faces), chosen and shown; the user's one way to paint and create (what to paint with: a material, a design or a group; where; add, replace or clear; standing as the surface asks), set against what stands today and given an order; the editor's workflow beside Godot, Unity and Unreal, with what to take and what to leave, and five requirements for the interface; vegetation cleared about a change to the land, stood again, and standing as its surface asks. |
| 2026-10-12 | Amendment file, 7v | After the user's five decisions on 7r to 7t. Built: paint goes on what is seen by default (a tile where one lies, the land elsewhere), with "the land, under any tile" as the other choice, said in the panel and never redirected silently; a tile put down has a surface of its own (a whole tile painted a ground, kept with that tile, not in the land and not in its text; worlds written before are unchanged until a tile of them is painted); a tile's colour is one mix over every cell about a place, so tiles agree at corners (7r's notch), and toward a cell with no tile it turns wholly into the land as the land is coloured (7r's hard rim); a tile is in a cell only when it stands in the middle of it as big as it, so a layout made smaller takes none away and puts no two in one cell (how the user's world came to that is reproduced by a test), tiles in no cell are listed in Tiles and given a cell or removed by hand, and none is put down on one unasked. Not built: tile elevation, paint within a tile, paint with a design. Master unchanged otherwise. |
| | | | |

---

# Notes / Ideas to Incorporate Later

Use this area for ideas that are important but not yet placed into the roadmap.

- ☐
- ☐
- ☐
- ☐
- ☐

---

# Roadmap Review Checklist

Before handing this roadmap back to Claude as a major implementation specification:

- ☐ Remove obsolete ideas.
- ☐ Consolidate duplicates.
- ☐ Clarify vague requirements.
- ☐ Add important missing capabilities.
- ☐ Confirm development order.
- ☐ Confirm representative demos.
- ☐ Mark completed capabilities.
- ☐ Mark deferred capabilities.
- ☐ Resolve major open questions.
- ☐ Decide what should become explicit engine requirements.
- ☐ Review the separate Personal Review Candidates list before finalizing.
- ☐ Mark the sections that need documentation.
- ☐ Produce a clean implementation plan from the finalized roadmap.
