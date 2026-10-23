

A 3D engine usually has three distinct layers:

| Layer | Purpose | Emberwatch evidence |
|---|---|---|
| Runtime simulation | Terrain, objects, characters, physics, AI, combat, world state | Present: terrain tools, construction, behaviours, combat, regions, clock, endings.  |
| Authoring tools | Let a maker create and edit that simulation efficiently | Partially present, with major friction: text-only rules, awkward placement, no slope warnings, no form editors.  |
| Game-product systems | Make a shipped game usable: menus, saves, settings, audio, scenes, localization, builds | Mostly absent or minimal: only F5/F9 save/load and a game-start object are described.  |

Your instinct is correct: the missing pieces are not all “advanced engine features.” Many are the ordinary connective tissue every game needs before it feels like a game rather than a sandbox.

## Project lifecycle map

Think of a game project as moving through these phases. Each phase needs engine support—not necessarily a polished visual editor at first, but a dependable, inspectable workflow.

### 1. Project creation

When someone starts a game, they should be able to create a project, not merely a world file.

Missing or underdeveloped systems:

- **Project concept**: a folder/package containing worlds, scenes, scripts/data, assets, audio, UI, settings, save-schema definitions, build configuration, and metadata.
- **Project browser**: see all worlds, scenes, prefabs/templates, materials, textures, models, sounds, fonts, localization tables, and data files.
- **Asset registry**: know which assets exist, where they are used, which are unused, and which are missing or broken.
- **Dependency view**: “If I delete this texture/model/sound, what breaks?”
- **Project settings**: display, input, physics, rendering quality, audio mixer, save format/version, default scene, company/game name, version number.
- **Templates**: empty project, first-person game, third-person game, top-down RPG, survival game, etc.
- **Project validation**: detect duplicate IDs, missing references, invalid data, orphaned assets, and incompatible save versions.

The workflow shows world files such as `emberwatch.json`, but there is no described project-level view of the game’s assets, data, dependencies, or reusable settings. 

### 2. World and scene authoring

The engine already has strong world-building tools: terrain sculpting, painting, structures, vegetation, water, styles, object placement, and named places. 

But a game needs more than one continuous world. It needs **scenes** and controlled transitions.

Missing systems:

- **Scene management**: title screen, main menu, loading screen, gameplay scene, pause menu, cutscene, game-over screen, credits, settings screen.
- **Scene graph / hierarchy**: a clear tree of scenes, subscenes, entities, UI, cameras, lights, audio emitters, triggers, and spawn points.
- **Scene transitions**: fade, wipe, loading screen, asynchronous loading, transition callbacks.
- **Multiple levels/maps**: travel between areas, streaming or load/unload of large worlds.
- **Prefabs / reusable compositions**: “guard tower,” “campfire,” “village entrance,” “enemy patrol,” saved as reusable objects.
- **Spawn points and player starts**: not just one game start, but named starts for new game, continue, fast travel, respawn, and testing.
- **Level validation**: unreachable areas, missing navigation, too-steep slopes, objects intersecting terrain, missing collisions, lights without shadows, missing audio listeners.
- **Level metrics**: object count, draw calls, triangle count, texture memory, audio memory, AI count, physics bodies.

The workflow explicitly records authoring friction: the editor does not show when a slope is too steep, structures can block paths, placement can be rotated incorrectly, and panels can obscure the viewport. These are exactly the kinds of problems a level-validation and better inspector system should catch earlier. 

### 3. Game flow and state

This is probably the biggest missing category you identified.

A game is not just a world; it is a sequence of states:

```text
Boot → Title → Main Menu → New Game / Continue → Loading → Gameplay ⇄ Pause → Save/Load → Ending → Credits
```

Missing systems:

- **Application state machine**: boot, title, menu, loading, playing, paused, cutscene, dialogue, inventory, map, game over, credits.
- **Title screen system**: logo, background scene, menu buttons, version text, “Continue” enabled only when a save exists, “New Game,” “Load,” “Settings,” “Quit.”
- **Continue system**: automatically identify the most recent valid save, show its metadata, and resume it.
- **Save slots**: multiple saves, autosaves, quicksaves, manual saves, save metadata such as location, playtime, timestamp, chapter, screenshot.
- **Save compatibility**: versioned save data, migration from older saves, safe handling of missing fields.
- **Checkpoint system**: respawn points, auto-checkpoints, death/respawn flow.
- **Pause semantics**: what stops, what keeps animating, whether audio continues, whether the world simulates.
- **Game session data**: playtime, deaths, items collected, quests completed, settings, achievements, and user preferences.
- **Flow scripting**: “after the player enters the square, start dialogue; after dialogue, unlock gate; after gate, trigger raid.”

The workflow has a game-start object, a time-scale field, an end condition, and conditional endings, but no described menu/scene-flow layer or “Continue” path. 

### 4. UI and interaction

The engine appears to have an editor UI, but a game also needs its own runtime UI framework.

Missing systems:

- **UI authoring**: buttons, panels, lists, tabs, sliders, dropdowns, tooltips, dialogs, HUD elements.
- **UI navigation**: mouse, keyboard, gamepad, touch; focus order; back/cancel behavior.
- **HUD**: health, stamina, mana, objective, compass, minimap, notifications, interaction prompts.
- **Menus**: pause, settings, inventory, crafting, map, journal/quest log, character sheet, controls remapping.
- **Toasts and notifications**: “Quest updated,” “Item acquired,” “Objective complete.”
- **UI theming**: fonts, colors, scalable UI, resolution independence, safe areas for consoles/mobile.
- **Accessibility**: text scaling, colorblind modes, subtitles, reduced motion, remappable controls, screen-reader-friendly labels.
- **UI debugging**: show layout bounds, focus path, input events, hidden/overlapping elements.

The Texts window is currently a central authoring surface for goods, acts, recipes, stats, quests, talks, and endings—but the workflow says these have no form editors and are typed as text. That is workable for a programmer, but painful for iteration and dangerous for non-programmers. 

### 5. Data authoring

The engine’s text-based approach is interesting and may be powerful, but it needs proper tooling around it.

Missing systems:

- **Structured data editors**: forms for quests, dialogue, items, recipes, stats, abilities, behaviours, endings, and world facts.
- **Schema definitions**: define fields, types, defaults, ranges, required values, and references.
- **Validation with useful errors**: “Quest `find_herb` references missing item `moonflower`,” not merely “red lines under text.”
- **Search and refactor**: find every reference to an item, character, quest, sound, or scene; rename safely.
- **Diff-friendly data**: stable IDs, deterministic serialization, readable diffs.
- **Localization**: externalized strings, language tables, pluralization, font support, right-to-left support.
- **Balancing tools**: tables for damage, health, economy, XP, loot, spawn rates, and difficulty.
- **Content dependencies**: quest requires item; dialogue requires quest state; ending requires facts.

The workflow confirms that quests, talks, acts, goods, recipes, and stats are authored through raw text and only checked when applied.  That makes the engine programmer-friendly but not yet maker-friendly.

### 6. Audio management

You specifically called this out, and it is a major gap.

A basic sound engine can play a sound. A game engine needs an **audio content workflow**.

Missing systems:

- **Audio asset browser**: list sounds/music, preview them, inspect format, duration, loudness, loop settings, and usage.
- **Sound events / cues**: named events such as `footstep_dirt`, `door_open`, `quest_complete`, `ui_click`, `wolf_howl`.
- **Event assignment UI**: select an object or animation/event, assign a cue, set volume, pitch, random variation, distance attenuation, loop, delay, and cooldown.
- **Audio buses/mixer**: Master, Music, SFX, UI, Ambience, Voice; per-bus volume and mute.
- **Listener and emitters**: 2D UI sounds, 3D positional sounds, ambient loops, reverb zones.
- **Music system**: layered music, exploration/combat/danger states, crossfades, intensity control.
- **Dialogue audio**: voice lines, subtitles, timing, interrupt rules.
- **Audio debugging**: show active voices, current buses, emitter positions, occlusion, CPU/DSP load, and missing audio references.
- **Validation**: “This door has no open sound,” “This music track is unused,” “This cue references a missing file.”

Right now, the workflow describes sounds being assigned by Claude during programming, but no visible system for browsing, assigning, auditing, or managing them. 

### 7. Input and controls

The engine likely has input at some level, but it needs a player-facing and developer-facing control system.

Missing systems:

- **Input actions**: `Move`, `Jump`, `Interact`, `Attack`, `OpenInventory`, `Pause`, `QuickSave`.
- **Input mapping UI**: remapping for keyboard, mouse, and gamepad.
- **Contextual input**: different mappings in gameplay, menus, dialogue, vehicles, and UI.
- **Input prompts**: dynamically show “E — Open” or gamepad-button equivalents.
- **Deadzones and sensitivity**: mouse sensitivity, controller stick deadzones, invert axes.
- **Input replay/debug**: visualize current actions and pressed devices.

The workflow mentions `E` for gates and `Tab` for play, but does not describe a general input-action system or remapping workflow. 

### 8. Debugging and testing

This is another area you correctly identified. The engine needs to make the game observable while it is running.

Missing systems:

- **Debug overlay**: FPS, frame time, CPU/GPU timing, memory, entity count, draw calls, physics bodies, active AI, audio voices.
- **Entity inspector at runtime**: click an object while playing and inspect its state, components, variables, AI state, inventory, relationships, and recent events.
- **God mode**: invincibility, infinite resources, no clip, fly camera, teleport, time control, spawn items, unlock doors, complete quests.
- **Cheats and test scenarios**: “Give all items,” “Set quest stage,” “Teleport to location,” “Spawn raid,” “Skip to dawn.”
- **Scenario runner**: define a test such as “Start at the north gate, trigger raid, verify the player can reach the square.”
- **Logging console**: filterable logs by system—AI, audio, save, UI, combat, quest.
- **Breakpoints/watch values**: pause when a variable changes or a condition becomes true.
- **Screenshot/video capture**: for bug reports and marketing.
- **Automated smoke tests**: launch game, load world, walk to a point, save, reload, verify state.
- **Crash/error reporting**: readable errors with scene, entity, save version, and recent log context.

The authoring workflow already shows why this matters: bugs were discovered by playing—raiders stopping too far away, gates facing incorrectly, cliffs blocking routes, and steep terrain preventing movement.  A debug/test layer would surface many of these before manual play.

### 9. Player progression and systems

These are more game-design systems than pure engine systems, but a general-purpose engine should offer reusable building blocks.

Missing or likely incomplete systems:

- **Inventory**: items, stacks, weight, equipment slots, containers, pickups, drop/destroy rules.
- **Stats and attributes**: health, stamina, strength, resistances, modifiers, buffs/debuffs.
- **Abilities/skills**: cooldowns, costs, targeting, animation/event hooks, upgrades.
- **Quest system**: objectives, stages, prerequisites, rewards, tracking, failure states, journal entries.
- **Dialogue system**: nodes, choices, conditions, speaker portraits, subtitles, voice, interrupts, quest hooks.
- **Economy**: currency, shops, prices, buy/sell, loot tables, crafting recipes.
- **Faction/reputation**: hostility, alliances, faction memory, consequences.
- **AI behavior tooling**: behavior trees, state machines, blackboards, perception, navigation, squad behavior.
- **Interaction system**: prompts, hold-to-interact, multiple interactables, ownership/permissions.
- **Physics/gameplay events**: damage, collision, triggers, volumes, projectiles, explosions.
- **Progression persistence**: what exactly is saved: player state, world state, quest state, NPC state, opened doors, killed enemies, inventory, time, settings.

The workflow does show a meaningful rule/behaviour foundation: behaviours imported from text, kinds, facts, endings, combat, barricades, regions, and a game clock.  The gap is less “no systems” and more “systems are not yet exposed as manageable, inspectable, reusable game-development tools.”

### 10. Presentation and cinematics

A game needs controlled presentation moments, not just a live simulated world.

Missing systems:

- **Camera system**: gameplay camera, cinematic camera, camera blending, shake, target follow, collision handling.
- **Cutscenes**: scripted sequences, dialogue, camera moves, character animation, skippable playback.
- **Animation events**: footstep sounds at animation frames, attack hit timing, door-open sound, UI triggers.
- **Post-processing**: bloom, color grading, vignette, depth of field, motion blur, quality presets.
- **Lighting workflow**: light baking or realtime lighting controls, light probes, shadows, performance budgets.
- **Material/material instance workflow**: reuse materials, tweak colors, manage textures, avoid duplicate assets.
- **VFX**: particles, decals, trails, impact effects, weather, fire, smoke.
- **Rendering profiles**: low/medium/high/ultra settings and automatic quality scaling.

The workflow includes world styles, terrain painting, water, crops, forests, and structures, but does not describe a broader presentation pipeline for cameras, animation events, VFX, or quality settings. 

### 11. Settings, accessibility, and platform concerns

These are boring until they are missing.

Missing systems:

- **Settings persistence**: video, audio, controls, gameplay, accessibility.
- **Video settings**: resolution, fullscreen/windowed, VSync, frame cap, quality preset, shadows, antialiasing.
- **Audio settings**: master/music/SFX/UI/voice volume, mute, output device.
- **Accessibility settings**: subtitles, text size, colorblind support, reduced motion, hold-to-toggle, aim assist.
- **Language selection**: UI language, voice language, subtitle language.
- **Platform abstraction**: keyboard/mouse, gamepad, touch, different screen aspect ratios, cloud saves where relevant.
- **First-run experience**: graphics detection, control explanation, language choice, safety/accessibility prompts.

### 12. Build, release, and maintenance

The final phase is turning the project into something a player can run.

Missing systems:

- **Build profiles**: Windows, macOS, Linux, web, console targets where applicable.
- **Asset packaging**: include only used assets, compress textures/audio, generate asset bundles/packages.
- **Build validation**: missing files, unresolved references, debug-only code, incompatible save versions.
- **Versioning**: game version, engine version, content version, save version.
- **Patching**: update content without breaking saves.
- **Crash reporting and logs**: useful logs from player machines.
- **Analytics**: optional, privacy-respecting telemetry for crashes, progression blockages, performance, and feature usage.
- **Documentation generator**: document actions, text commands, data schemas, and editor workflows automatically.
- **Sample projects**: small complete examples demonstrating menus, saves, audio, quests, combat, and UI.

## A practical priority order

If I were sequencing this, I would not start with the most sophisticated systems. I would build the ones that make every future feature easier to test and ship.

### Tier 1 — Make it a game

1. **Project system**: one project contains worlds, scenes, assets, data, settings, and builds.
2. **Scene/state manager**: title, menu, loading, gameplay, pause, game over, credits.
3. **Title screen with Continue**: detect latest valid save, show metadata, load it.
4. **Save-slot manager**: autosave, quicksave, manual slots, save metadata, save versioning.
5. **Runtime UI framework**: menus, HUD, prompts, notifications, settings screens.
6. **Audio event manager**: named cues, assignment UI, buses, preview, missing-reference checks.
7. **Input action system**: named actions, remapping, contextual prompts.

### Tier 2 — Make authoring humane

8. **Structured editors** for quests, dialogue, items, recipes, stats, abilities, and endings.
9. **Asset browser and dependency/usage view**.
10. **Prefab system** for reusable structures, encounters, props, and UI.
11. **Inspector improvements**: search, categories, meaningful object names, quick rotation/placement, contextual actions.
12. **Level validation**: slope, navigation, missing references, overlapping objects, unreachable areas.
13. **World/scene diffing and version-friendly serialization**.

### Tier 3 — Make development observable

14. **Runtime debug overlay** and entity inspector.
15. **God mode / cheat console / scenario runner**.
16. **Log console with filters** and useful error messages.
17. **Performance profiler**: frame time, systems, memory, entities, audio, rendering.
18. **Automated smoke tests** for load, play, save, reload, quest progression, and ending conditions.

### Tier 4 — Make it shippable

19. **Settings persistence and accessibility options**.
20. **Localization**.
21. **Build packaging and validation**.
22. **Crash reporting, logs, and optional analytics**.
23. **Documentation and sample projects**.

