# Draw3D — Roadmap Progress and the Next Milestone

Written 2026-10-22, after commit `ced06f6` (the guard, 7di) and the usability passes U1–U7. This is an assessment and a plan; nothing was built for it and no implementation file was changed.

The roadmap it measures against is [Draw3D_Master_Roadmap_Final.md](Draw3D_Master_Roadmap_Final.md), sections 1–10 and principles A–R, with the amendments in [Draw3D_Roadmap_Amendment_2026-10-09.md](Draw3D_Roadmap_Amendment_2026-10-09.md) (up to 7di). The master roadmap's own ☐/◐ marks are mostly stale: they were last set around 2026-10-15 and many ☐ items have since been built. Every estimate below comes from the code, the tests, and [PROGRESS.md](PROGRESS.md) as it stands today, not from those marks.

---

## 1. Executive summary

1. **Draw3D is past the "collection of systems" stage and into the "authoring platform with gaps" stage.**
   - Every one of the ten roadmap sections has working code.
   - Most of it is authored as content (text the host validates, edited through windows), not as code.
   - Weighted across the roadmap, roughly **55–60% of the intended capability is delivered**. That figure hides a wide spread: the persistent-world foundation and identity are about 80–85%; large-world streaming is about 15%.
2. **The foundations are sound and well tested.**
   - The document, undo, identity and save model are proven by about 185 host tests and in daily use since M1.
   - Authored world and runtime state are separate by design: the document versus facts.
   - Ids are hashes of key paths, so they survive regeneration.
3. **Hand evidence is the weakest link.** It is far behind breadth.
   - **Shaped by repeated hand tests:** construction, terrain, plants, physics signals, and the U1–U3 usability passes.
   - **Mostly never tried by hand:** gameplay (behaviours, the guard, combat, items since 7cb, quests), the environment's newer weather and sound, connected worlds, and export.
   - Two tour parts fail even at the committed code (`village`, `items`).
4. **Can the user make a small game without Claude writing code?**
   - **Yes, for a first draft** of a small exploration, stealth or village-errand game. That means:
     - land, construction and plants;
     - creatures with behaviours composed from states and steps;
     - doors and locks, physics machines, quests and talks written as text;
     - a HUD, a style, an eye, sounds.
   - **Not yet for a game that holds up.** The reasons:
     - the vocabularies (events, steps, actions, abilities) are closed lists, so each new kind of mechanic still needs code;
     - quests and talks are raw text;
     - items, crafting and combat numbers are shallow;
     - the player cannot build or change the world while playing;
     - the exported game exits with a fault code and cannot carry connected worlds;
     - no genre has met the user's polish bar.
5. **Persistent, scalable open worlds:** more exists than the phrase suggests, and less than an open world needs.
   - **What exists:**
     - a durable record of runtime state (facts by stable id);
     - discrete regions with a lifecycle (connected worlds, held in memory and saved together as a journey);
     - distance tiers for drawing;
     - per-cell navigation rebaked when what is solid changes;
     - a crude simulation LOD: things far from the player freeze.
   - **What is missing:**
     - unloading by distance inside one world;
     - durable records of things the *player* adds;
     - reconciling a saved game with a world edited or regenerated since (id migrations are not applied to facts);
     - catch-up simulation for frozen settlements;
     - any view of region activity.
6. **Recommendation: a small end-to-end proof first,** called **PW1 "A region that remembers"** (section 6).
   - One world of 3×3 regions, where far regions are really unloaded.
   - Six kinds of player change survive unload, reload, save and load, and a world regenerated in between.
   - One scheduled villager and one timer catch up while away.
   - A Regions overlay shows what is happening.
   - It is preceded by a small saved-game integrity step (**PW0**) and by G3, the vertical slice already designed.
   - Expand to a seamless benchmark only after PW1's measurements say it holds.

---

## 2. How to read this

**Evidence grades.** These are the same four the gameplay roadmap uses. A higher grade is never claimed from a lower one.

| Grade | Meaning |
|---|---|
| **built** | The code exists. Nothing more is claimed. |
| **auto-checked** | A Rust unit or protocol test, a tour part, or a demo with `expect` checks passes. |
| **seen** | A picture from a checked run was looked at. |
| **hand-tested** | The user tried it, as PROGRESS records. |

**Completion.** A completion figure is the share of the section's *intended* capability that a user could rely on today. A system that exists in code but is untried, or works only in its demo, counts for part. Optional advanced items are not counted against a section whose core is mature.

**Confidence.**
- **High:** checked in code here, and backed by tests or hand use.
- **Medium:** checked in code or in PROGRESS, with thin evidence.
- **Low:** inferred, or depends on behaviour nobody has seen in motion.

**Authoring independence** asks how much the user can configure, combine, extend and reuse through Draw3D's own windows and texts, without Claude changing code:
- **High:** new uses need no code.
- **Medium:** configuration and composition need no code; new kinds of thing do.
- **Low:** mostly fixed, or reachable only as text with no checks.

---

## 3. Roadmap progress, section by section

### 3.1 Summary table

| # | Roadmap section | Done | Evidence | Authoring independence | Priority | Confidence |
|---|---|---|---|---|---|---|
| 1 | Persistent Interactive World | **~80%** | auto-checked, hand-tested | High | **Now** (saved-game integrity, PW0) | High |
| 2 | Characters, Creatures & Animation | ~55% | auto-checked; Studio hand-tested 4× | Medium | Later (parked by the user) | Medium |
| 3 | Navigation, Behaviour & Simulation | ~60% | auto-checked; little hand | Medium–High | **Now** (G3, then G5) | Medium |
| 4 | Environment, Weather, Effects & Audio | ~65% | auto-checked, seen; almost no hand | Medium | Later (recorded sound and fog when a game needs them) | Medium–Low |
| 5 | Rendering & Style | ~55% | auto-checked; user found the look "generic" | Medium–High | Later (style pass, before showing a game) | Medium |
| 6a | Procedural authoring (makers, designs, generators, construction) | ~70% | hand-tested repeatedly | High | Mature: do not reopen | High |
| 6b | Large worlds and streaming | ~15% | engine scale measured; Godot scale not | Low | **Next architectural milestone** (PW1) | High |
| 7 | Items, Inventory, Equipment & UI | ~60% | auto-checked; items tour broken | Medium | Soon (G6) | Medium |
| 8 | Gameplay Framework | ~55% | auto-checked; first loop hand-played 2026-10-14 | Medium | **Now** (G3, G4, G8) | Medium |
| 9 | Abilities, Combat, Movement & Physics | ~50% (physics ~75%, combat ~35%) | physics hand-tested; combat barely | Medium | Physics: mature. Combat: G7 when a game needs it | Medium |
| 10 | Genre Proof & Standalone Shipping | ~30% | genres built; one played and rejected | Medium | After PW1 and G3–G6 | Medium |

**Cross-cutting principles:**

| Principle | Done | Note |
|---|---|---|
| A Identity and provenance | ~85% | Key-path ids, migrations and things kept by name. Saved-game facts do not follow id migrations (§5.1). |
| B, C, P, Q Define → generate → edit → reuse; generated and authored together | ~70% | Hand edits survive regeneration by name. Designs and groups are given to makers. A library. |
| D, K Semantic communication | ~45% | Facts are the shared bus, with events (G1) and a capability manifest. Many links are still one-off. |
| E, F Defaults and depth | ~60% | Presets, "What it does", the U1–U7 passes. Advanced settings stay as text. |
| G Testing and instrumentation | ~70% | Host tests, about 125 demos (newer ones with `expect`), 70+ tour parts, Playtest and Signals. Several tours fail. |
| H Performance and streaming | ~25% | The engine is fast (85,000 things, 22 MB, edits in tens of µs). Godot cost at scale is unmeasured; tall grass drops the frame rate. No streaming. |
| I Asset and content ecosystem | ~65% | Library with pictures and search; kept looks; glTF, FBX and OBJ imports; credits. |
| J Host versus Draw3D | ~85% | The Rust host owns state and saves; Godot draws and simulates. Held to throughout. |
| L Validation of generated content | ~50% | Readers say what is wrong (kind words, behaviours, Signals, Studio audits). No reachability or placement checks. |
| M, N Movement and authored action | ~55% | Moves as data, the Studio, acts with why-not. Action definitions are spread (G4). |
| R Variation | ~30% | Colour variation, first tier. No alternatives shown, no locks. |

**Checkpoints** (master roadmap, "Major Checkpoints"):

| Checkpoint | State |
|---|---|
| **A** Persistent World | Met. |
| **B** Living World | Met in code. Hand evidence ends at the village play of 2026-10-14, which left five faults open. |
| **C** Authorable World | Mostly met. Missing: regions as things, style influencing every category, validation of generated content, editing large worlds in Godot. |
| **D** Game Framework | Built. Not proven by a hand-played slice since G1 and G2. |
| **E** Broad Capability | Partial. Physics and traversal are good; combat, vehicles and abilities are thin. |
| **F** Engine Proof | Early. |

### 3.2 Section detail

#### 1. Persistent Interactive World — ~80%, high confidence

- **Goal:** an authored world that is edited, played, saved and restored, with authored content and runtime state never confused.
- **Accomplished:**
  - **World data and world state are separate:**
    - the world's document is saved by the host, with backups before each save and autosave every 3 minutes (X1);
    - runtime state is *facts* about stable ids, in their own layer ([core/src/state.rs](core/src/state.rs)). No edit, undo or rebuild touches a fact.
  - **Play and stop:** play never writes into the document, and stop restores the authored world.
  - **Saved games** (F9) hold doors, broken things, items (where each is, dropped or equipped), the fallen, behaviour states, timers and waits (7ar), and where NPCs stand.
  - **Interaction:** acts with why-not (`acts.txt`), `offers`, doors and interiors, engine-owned colliders, a player start, a contextual prompt, and the Inspector on any click (7bz).
- **Evidence:**
  - core state tests;
  - protocol tests p01–p12, p18, p27–p30, p69, p90–p91 (worlds written by older code open built), p95 (autosave);
  - tours `saves`, `delivery`, `door`;
  - in daily use since M1.
- **Remaining:**
  - A saved game loaded over an edited or regenerated world reports `document_changed` and a count of missing things. It does **not** apply the document's id migrations to facts: [editor/host/src/lib.rs](editor/host/src/lib.rs) `load` never calls `resolve_id`. It also does not reconcile or list orphaned facts.
  - Some runtime state is not saved: a guard's `noticed` place (7di); what loose physics bodies were doing beyond where they rest.
  - "Deterministic re-entry" is partial: rooms re-roll per run by a seed (7bp).
- **Independence:** high. Nothing here needs authoring.
- **Priority:** now, as the small step PW0 (§6.2).

#### 2. Characters, Creatures & Animation — ~55%, medium

- **Accomplished:**
  - **The motor:** step-up, climb (including walls and ceilings), vault, wall run, backflip, wade, eight ways.
  - **Moves as data** (`moves.txt`), with combos.
  - **The Character Studio:** glTF, FBX and Mixamo import; retargeting by maps; phases; versions; buttons and profiles; gear on 21 places; IK for two hands; lock-on; a practice yard; the Make tab for body, colours and outfits.
  - **Creatures:** creature kinds on rigs, with gaits; capes that sway (`sway`).
  - **Eyes:** behind, first person (`unseen`), top-down (by tip), side-on 2.5D with lanes, and a shot camera. All are authorable in `eyes.txt` and chosen per world.
- **Evidence:** `chartest.gd` (192 headless checks); tours `motor`, `moves`, `climb`, `studio`; four Studio hand tests (7bd–7bh).
- **Remaining:**
  - generated or novel creatures (body plans that drive locomotion);
  - faces and expressions (tier 2, waiting on the user's choice of assets);
  - fingers on a grip;
  - water entry and exit;
  - a glide;
  - real controller and real Mixamo-file validation;
  - gear seen only on Studio characters, not on the host's figure.
- **Independence:** medium. Moves, combos, gear placement, eyes and bodies are authored. New body plans and new move mechanics need code.
- **Priority:** later. The user parked characters on 2026-10-16.

#### 3. Navigation, Behaviour & Simulation — ~60%, medium

- **Accomplished:**
  - **Navigation:** Godot's server, baked one cell at a time on a thread, only for cells asked for, and rebaked when what is solid in a cell changes ([editor/godot/nav.gd](editor/godot/nav.gd)). A shut door is a wall; doors are ways for those who can open them.
  - **The folk:** schedules (`goes HOUR at/do`), work sequences, and reactions with priority and resume (7ag–7ai).
  - **Perception:** sight, hearing, alarm, search, patrols ([editor/godot/watch.gd](editor/godot/watch.gd)).
  - **Ecology:** predator and prey, sides, packs (7bt).
  - **Behaviours (G1, G2):**
    - one condition language with `all:`/`any:` and why-not (G1);
    - declared events (10 kinds) and a capability manifest;
    - Draw3D's own behaviour system: states with priority, seven building blocks, about 20 steps, parameters, a library, and the Behaviours window, reached with Ctrl+B or from the Inspector (7dh, [editor/godot/rules.gd](editor/godot/rules.gd));
    - the guard: patrol, hear, investigate, alarm, return (7di).
- **Evidence:**
  - demos `behave` 14/14, `guard` 9/9, `rules1` 6/6 with `expect`;
  - measured: 206 behaving things at 0.34 ms a frame on average;
  - the user tried the behaviour window in U1–U3.
- **Remaining:**
  - Perception reports only the player, not other entities or stimuli (G5).
  - No navigation links for ladders or climbable walls; no swimming.
  - Crowds and stairs while carrying are open faults.
  - The folk cannot open a door that behaves.
  - **Simulation LOD is a freeze.** A roaming creature, a watcher or a ruled thing farther than about 75 m is left exactly as it is, and wakes at 60 m with no catch-up ([editor/godot/folk.gd](editor/godot/folk.gd) `_left_be`, `SLEEPS`). Folk with schedules are never put to sleep, so they all simulate at full rate, everywhere.
- **Independence:** medium–high. States, steps, conditions and templates are composed in a window. A new step, event or sense needs code.
- **Priority:** now (G3), then G5.

#### 4. Environment, Weather, Effects & Audio — ~65%, medium–low

- **Accomplished:**
  - **Weather and time:** day and night; weather with one shared wind; seasons.
  - **What lies:** snow (with depth and shelter under roofs), rain, leaves and ash, all lying where they fall; prints; anything falling (frogs, crates).
  - **Water and sky:** rippled water and foam; sunsets and volumetric light; chimney smoke.
  - **Life:** ambient birds, fish and insects ([editor/godot/wild.gd](editor/godot/wild.gd)).
  - **Sound:** sounds written as recipes, heard where they are made, and a weather's sound (7as).
  - **Reactions:** folk shelter from rain (7ai).
- **Evidence:** tours `weather`, `living`, `sound`; demos `seasons`, `weathers`, `living`. Almost no hand evidence. PROGRESS records that nobody has listened to the sound yet.
- **Remaining:**
  - recorded sound, music and music regions, footsteps;
  - fog controls;
  - temperature and exposure;
  - weather acting on navigation and gameplay;
  - seeing all of it in motion.
- **Independence:** medium. Weathers, sounds and wind are text and windows. New effect kinds need code.
- **Priority:** later. Take each item when a game needs it.

#### 5. Rendering & Style — ~55%, medium

- **Accomplished:**
  - **Styles** ([editor/godot/styles.txt](editor/godot/styles.txt), Window > Styles): eight after named games, plus lookout and storybook.
  - **What a style says:**
    - picture grading;
    - palette roles;
    - buildings;
    - what grows (the `evergreen` style gives the same woods conifers);
    - light through the day;
    - fog;
    - `takes` from another style.
  - **Looks and materials:** per-part colours and materials; textures; kept looks in the user's library.
- **Evidence:** demo `styles`; the user's verdict on the built look: "generic, little charm" (7cs).
- **Remaining:**
  - a style per category, mixed, and per region (principle Q);
  - style reaching creatures and characters;
  - the style pass the construction direction document orders next.
- **Independence:** medium–high.
- **Priority:** later, but before any game is shown to others.

#### 6a. Procedural authoring — ~70%, high

- **Accomplished:**
  - **Land:** terrain sculpt and paint (five surfaces, including snow); layouts of square or hex cells; terrain tiles with their own elevation.
  - **Makers and designs:** makers as things of the world; designs and groups; a library of 100+ designs.
  - **Generators:** tree, plant and rock, with the recipe kept in the design; brushes with mixes; fields that fill an outline.
  - **Hand edits kept by name** through regeneration; freeze and thaw.
  - **Drawn construction** in the manner of Tiny Glade: walls, houses of eleven sorts, roofs that join, paths, ponds, docks, platforms, a town; cut, join, copy a look.
  - **Several things chosen at once** (7df).
- **Evidence:** about 40 protocol tests (p19–p94); many tours; the user's hand tests 7cq–7da and the tile passes.
- **Remaining:**
  - regions as things: an outline any maker can fill (7q);
  - biomes as rules over a region;
  - rivers;
  - a generator dependency graph shown to the user;
  - two construction routes (cells and drawn);
  - 7db–7df untried by hand.
- **Independence:** high.
- **Priority:** mature. **Do not reopen for general polish.** Take gaps only when a game hits them.

#### 6b. Large worlds and streaming — ~15%, high

- **What exists:**
  - **Engine scale** ([core/scale-results.txt](core/scale-results.txt)): 85,000 things in 22 MB, first build 46 ms, one edit about 34 µs, one village remade in about 1 ms.
  - **Drawing in Godot:** cells grouped into draw regions of 4, 16 and 64 cells, each drawn whole beyond a distance. Meshes are fetched when a cell names them and let go when none does ([editor/godot/world_view.gd](editor/godot/world_view.gd)).
  - **Lighter meshes** for what grows, far off (7ap).
- **What is missing:**
  - **The host sends every cell of the world, and Godot holds all of them.** Nothing is unloaded by distance: no things, no physics bodies, no simulation.
  - Godot's cost for a large, fully dressed world is unmeasured.
  - Tall grass drops the frame rate. The user has asked for that as its own later pass.
- **Independence:** low (nothing to author yet).
- **Priority:** the next architectural milestone (§5).

#### 7. Items, Inventory, Equipment & UI — ~60%, medium

- **Accomplished:**
  - **Items:** each item with its own id; Gear to find (18 designs); taken in hand, worn, dropped; carried across worlds (7br).
  - **Goods:** counts held as facts.
  - **The game's own screens** ([editor/godot/gameui.gd](editor/godot/gameui.gd)): HUD, inventory, menu, journal, all worked by keyboard, mouse and controller. The HUD is laid out in a window (`hud.txt`).
  - **Survival meters:** stats and statuses (`stats.txt`).
  - **Uses:** "Make it usable" presets for a bed, food, a container, a door, a locked door.
  - **Controls:** buttons remapped in the editor and kept for every project.
  - **The editor shell:** one workspace, about 29 windows, the Library, and catalogues.
- **Evidence:** demos `inventory`, `usable`, `anything`, `presets`. The `items` tour fails at the committed code (the lantern never reaches the hand, `tour.gd:2013`). The user tried 7ca and 7cb by hand.
- **Remaining:**
  - Items and goods are two models: no stacks of a kept item.
  - No crafting recipes or stations as a model.
  - No transfers between containers.
  - No weight or slots.
  - What is worn shows only on Studio characters.
  - Only the player has stats.
- **Independence:** medium.
- **Priority:** soon. G6 also mends the items tour.

#### 8. Gameplay Framework — ~55%, medium

- **Accomplished:**
  - **Rules:** facts as world state; G1 conditions and events; `on EVENT do ACTION` rows; states (7dh).
  - **Quests and talk:** quests and talks with answers (`quests.txt`, `talks.txt`); journal and markers.
  - **Economy:** goods, and trading as acts.
  - **Puzzles:** block puzzles as rules (`puzzles.txt`).
  - **Endings:** a game's hour and ending set on the start (7ch).
  - **Loops:** the village loop played by hand on 2026-10-14; genres from New world (wilds, stealth, shelter, watchtower, connected worlds).
- **Evidence:** tours `work`, `talk`, `errand`. Both fail in the working tree only, because the user's uncommitted `characters.txt` gives the ranger `moves samurai`, which has no reach, wave or pet. Demos `rules1`, `wired`, `held`.
- **Remaining:**
  - quests and talks edited as raw text only;
  - no branching endings, reputation or progression;
  - action definitions spread across four files (G4);
  - no shops with prices;
  - G3, the slice that would prove G1 and G2 together, not yet built.
- **Independence:** medium.
- **Priority:** now (G3, then G4; G8 after).

#### 9. Abilities, Combat, Advanced Movement & Physics — ~50%, medium

- **Physics (~75%, hand-tested):**
  - each thing's own physics and colliders;
  - joints (several per thing);
  - fans and belts;
  - sensors, `hits` and `feels`;
  - signals with conditions, timers and resets; the Signals window.
- **Movement:** vault, wall run, climb, wade, slippery ground.
- **Abilities:** a catalogue that gives any ability to any thing (`abilities.txt`).
- **Combat (~35%, barely hand-tested):**
  - health as a number of blows; attacks; dodge; stamina;
  - takedowns; enemies that shoot;
  - boss phases; barricades;
  - aiming and firing.
- **Evidence:** demos `mech`, `signals`, `fights`, `boss`; the user's hand test of the batcher and the strength tester; the shelter "reasonable for now".
- **Remaining:**
  - health as points with armour;
  - abilities with costs and cooldowns;
  - statuses on entities other than the player;
  - wheeled and flying vehicles;
  - glide;
  - cloth beyond a swinging cape;
  - verification of precision-platforming feel;
  - a ball sent flying by walking into it.
- **Independence:** medium. Physics is high.
- **Priority:** physics is mature. Do combat (G7) when a target game needs it.

#### 10. Genre Proof & Standalone Shipping — ~30%, medium

- **Accomplished:**
  - World > Export the game: one program, and a zip.
  - Genres as worlds from New world.
  - The wilderness benchmark, with a record of its gaps.
  - Genre-readiness notes.
- **Evidence:**
  - The user played the watchtower with a controller and rejected it (7ch); it was reworked (7cj) and not tried again.
  - The built game exits with a Windows fault code after it has done everything (Known gaps).
- **Remaining:**
  - a clean exit;
  - connected worlds in a build;
  - a title and start flow as content;
  - one game played to the polish bar;
  - the genre gauntlet's records.
- **Priority:** after PW1 and G3–G6. One genre should be played to the polish bar before the gauntlet is widened.

---

## 4. Readiness to author real games

### 4.1 By domain

Key: ● yes, ◐ partly, ○ no.
- **Available:** it exists.
- **Authorable:** set up in Draw3D's windows or texts.
- **Composable:** combined with other systems into new gameplay.
- **Reusable:** saved and applied across objects or worlds.
- **Reliable:** works consistently in real use, by hand evidence.

| Domain | Available | Authorable | Composable | Reusable | Reliable | What blocks independence |
|---|---|---|---|---|---|---|
| World construction, terrain, vegetation, layout | ● | ● | ● | ● | ◐ | 7db–7df untried by hand. Regions as things missing. |
| Characters, movement, animation | ● | ● | ◐ | ● | ◐ | New body plans and move mechanics need code. Characters are parked. |
| Combat foundations | ◐ | ◐ | ◐ | ◐ | ○ | Health is blows. No costs or cooldowns. Barely hand-tested. |
| NPC behaviour, perception, schedules, reactions | ● | ● | ◐ | ● | ○ | Perception is of the player only. The step and event lists are closed. No hand test since G1 and G2. |
| Items, equipment, inventory, resources, crafting | ● | ◐ | ◐ | ◐ | ○ | Two item models, no recipes, broken tour. |
| Quests, progression, rules, triggers, sequences | ● | ◐ | ● | ◐ | ◐ | Text only. No progression. G3 not built. |
| Physics mechanisms and interactive objects | ● | ● | ● | ● | ● | Mature first version. |
| Sound, lighting, weather, effects, styles | ● | ● | ◐ | ● | ◐ | Not seen in motion. No recorded sound or music. The look is "generic". |
| Cameras (eyes) | ● | ● | ● | ● | ◐ | No transitions authored by condition beyond zones. |
| Save, load, persistence | ● | ● (nothing to author) | ● | ● | ◐ | No reconciliation after an edit. Player-built content is not possible. |
| Connected worlds and journeys | ● | ● | ◐ | ● | ○ | Untried by hand. Not in an exported game. |
| Shipping a game | ◐ | ● | — | — | ○ | Fault code on exit. Single world only. |

### 4.2 What the user can build today without Claude writing code

- A small world: land shaped and painted, a village drawn as in Tiny Glade, woods, plants, ponds, paths.
- Doors and locked doors from presets; containers, beds and food; machines from joints, fans, belts and sensors, wired by signals.
- Creatures and guards given behaviours composed in the Behaviours window:
  - states with priorities, conditions and events;
  - steps: go, flee, patrol, look about, say, give, set a fact;
  - saved to the library and given to every one of a type, or one alone.
- Folk with daily schedules, work sequences and reactions; quests and talks with answers, written as text with checks.
- A HUD, an eye (third person, first person, top-down, side-on), a style, sounds from recipes, weather.
- A genre start, a player start with an hour and an ending, saved games, and connected worlds.

That is enough for the first draft of a 15–30 minute exploration, village-errand, or light stealth game.

### 4.3 What still forces code

1. **Closed vocabularies.** Each of these is a list held in code:
   - events (`events.gd`; host `kind.rs`);
   - behaviour steps (`rulebook.gd`);
   - acts;
   - abilities (`abilities.txt` lines map to GDScript);
   - kind words.

   A mechanic that needs a new verb ("chop", "plant", "mount", "throw", "repair", "trade at a price") needs Claude. This is by design: words are checked, and a scripting route (Luau) is parked. It is still the single largest dependence.
2. **Actions are not one definition** (G4). Duration, interruption, animation, sound and effect live in four places, so a new interaction is assembled by hand or by code.
3. **Items, crafting and economy** (G6). There are no recipes, stacks or prices.
4. **The player changing the world while playing.** The player cannot build, place, dig or plant. Settlement and survival games depend on it.
5. **Structured editing of quests and talks.** Text is authorable but error-prone at scale.
6. **Combat numbers** (G7): armour, damage kinds, cooldowns, statuses on enemies.
7. **New UI screens.** A shop, a crafting screen or a map are not content.

### 4.4 Candid overall assessment

**Draw3D can author small, coherent prototypes without code today, faster than a general engine would.** Building the world is its strongest part.

It cannot yet author a small game that a stranger would play to the end and call finished. Three reasons:
- **Reliability is unproven.** Most gameplay passes since 2026-10-17 have never been played by hand. Two tour parts fail at the committed code.
- **Each new genre still meets a missing verb.** Combat depth, crafting, building while playing and shops are examples.
- **No game has met the polish bar.** The watchtower was rejected, and its rework was never tried.

The gap is narrower than the breadth of open items suggests. Most of it is three foundations (actions, items, player-made changes) and a hand-test debt, not missing engines.

---

## 5. Near-complete areas, and remaining blockers

### 5.1 Close to their goal: targeted work only

| Area | What closes it | Size |
|---|---|---|
| Persistent world foundation | PW0: apply id migrations to saved facts on load; list orphaned facts; save `noticed` | Small |
| Physics, mechanisms, signals | Nothing now. The ball-flying fault when one is next touched. | — |
| Terrain, tiles, plants, generators | Nothing now. | — |
| Drawn construction | The user's hand test of 7db–7df. Mend only what that finds. | Small |
| G1 conditions and events, X1 autosave | One hand test each: a lamp at dusk, a talk answer with `all:`, a kill-and-restart | Small |
| G2 behaviours | The construction bar's button; mixed values with several chosen; G3 proves the rest | Small |
| Editor usability U1–U7 | A hand test of U4–U7 and U5's layout | Small |
| Tours | Mend `village` inside G4 and `items` inside G6, as planned. Rewrite `library` as a protocol test. | Small |

### 5.2 Substantially built, but not enough hand evidence to call complete

- weather that lies, snow, sound, the town (7dc, 7dd, 7as);
- items and the inventory since 7cb;
- fights, the boss, barricades beyond the shelter;
- stealth and side-on;
- connected worlds and journeys (7bq–7bs);
- the guard (7di);
- the assistant;
- the reworked watchtower;
- export.

None should be called done until played. None needs rework before it is played.

### 5.3 Genuine blockers, ranked by what they stop

1. **Player-made changes are not durable records** (blocks settlement, survival and building games). This is PW1.
2. **One action model** (G4). It blocks every new interaction without code, and the village tour.
3. **Items, stacks, recipes** (G6). It blocks crafting and survival loops, and the items tour.
4. **Perception of any entity** (G5). It blocks ecology, factions and NPCs reacting to each other.
5. **The exported game's fault code, and single-world export.** It blocks shipping anything.

### 5.4 Optional, not blockers

The roof curve, stilts, a radial toolbar, cloth, generated creatures, card and strategy games, vehicles of new kinds, faces, generator version migration tooling.

---

## 6. The next architectural milestone: persistent, scalable open worlds

### 6.1 What the implementation already supports

| Requirement | What exists | Where | Grade |
|---|---|---|---|
| **Region lifecycle** | (a) **Connected worlds** are discrete regions. Walking into one sets the current world aside whole (its engine, history, facts, running state) and takes up the other as it was left. Going back restores it exactly. (b) **Draw regions** of 4, 16 and 64 cells, drawn whole beyond a distance; meshes fetched when named and let go when not. | host `Away`, `travel` ([lib.rs](editor/host/src/lib.rs)); [travel.gd](editor/godot/travel.gd); [world_view.gd](editor/godot/world_view.gd) | auto-checked (tour `journey`); not hand-tested |
| **Durable records of runtime state** | Facts about stable ids, each change numbered, saved apart from the document: doors, broken things, item whereabouts, the fallen, states, waits, where NPCs stand. A journey save holds every world's facts in one file. | [core/src/state.rs](core/src/state.rs); host `save`/`load`, `save_journey` | auto-checked; hand (F9 since stage 1) |
| **Restoration across save/load** | Loaded facts become the state, and the editor hears only what changed. Facts about things that are gone are counted. | host `load` | auto-checked |
| **Restoration across unload/reload** | Only between connected worlds, held in memory. Nothing inside one world is unloaded, so nothing needs restoring. | — | — |
| **Generated, authored and player content together** | Ids are hashes of key paths, so a regenerated child keeps its id when its key survives. Hand edits are kept by name. Migrations record renamed ids. **But** saved-game facts are not passed through migrations, and orphaned facts are only counted. The player cannot add content. | [core/src/idmap.rs](core/src/idmap.rs), [kept.rs](core/src/kept.rs) | auto-checked (document side); gap (saved-game side) |
| **Simulation LOD** | Binary sleep: roamers, watchers and ruled things beyond about 75 m freeze, and wake at 60 m. `wild.gd` leaves far life alone. Scheduled folk are never asleep. Worlds set aside are frozen; only the hour goes with the player. | [folk.gd](editor/godot/folk.gd) `_left_be`; [rules.gd](editor/godot/rules.gd); [wild.gd](editor/godot/wild.gd) | built |
| **Navigation after construction or terrain change** | Per-cell rebake on a thread when what is solid changes. Doors are links. Changes made in the editor, and doors opened in play, are covered. Runtime building does not exist. | [nav.gd](editor/godot/nav.gd) | auto-checked |
| **Debugging and visibility** | Playtest (facts, behaviours, why not yet), Signals (writers and readers of each fact), host `held` (cells, bytes, things, meshes), frame rate, `-Measure`. Nothing per region. | various | built |

**In short:** the *data model* for persistence is right (facts by stable id, kept apart from the document) and already proven across worlds. What is missing is (a) *lifecycle inside one world*, (b) *records of things the player adds*, (c) *reconciliation when the world changes under a save*, and (d) *simulation that continues while unloaded*. None of these needs a replacement architecture. Each extends a mechanism that exists.

### 6.2 Recommendation: prove it small first

Do not begin with a seamless large-world benchmark. Today's whole-world-loaded model works at village scale. Its limits (Godot nodes and bodies, scheduled folk simulated everywhere, facts growing without bound) should be measured on a small world built to reveal them, before anything is designed for kilometres.

#### PW0 — Saved-game integrity (small; before PW1)

- **Scope:**
  - Pass each loaded fact's id through `resolve_id` (the document's own migrations).
  - List orphaned facts (about things no longer there) in the load message and in Playtest, never applying them to another thing.
  - Save `noticed` and any other behaviour memory that is a place.
- **Acceptance:** protocol tests in which:
  - a saved game is loaded after a maker's key is renamed by migration, and the facts follow;
  - a maker is regenerated with fewer things, and the facts about removed things are listed, not lost silently and not misapplied;
  - the guard demo is loaded mid-investigation and goes to its `noticed` place.
- **Why first:** PW1's restoration criteria rest on it, and it closes Section 1.

#### PW1 — "A region that remembers" (the proof of concept)

**The world.** One authored world of about 384 × 384 m, divided into 3 × 3 *regions* of 128 m. A region is a fixed square of the existing cell grid; it is not a new spatial model. It contains:
- a hamlet of drawn houses with three scheduled folk;
- a wood, generated;
- a field with a timer-driven crop (a fact that ticks);
- a guard with `guard_ways`;
- loose crates.

**Region lifecycle**, decided in Godot by distance from the player:

| State | Where | What runs |
|---|---|---|
| **Active** | the player's region and its eight neighbours | full simulation and drawing |
| **Far** | beyond them | Godot frees the region's things, bodies, nav regions and units. The host keeps the document and the facts, unchanged. |

Coming back near, the region's cells are laid again and every fact about its things is applied, as a loaded game is. This uses what `travel` does between worlds, at the scale of a region within one world.

**Player-made changes, the six cases, each a durable record:**
1. a tree of the wood chopped (a generated thing; fact `where gone`);
2. a door broken or left open (exists);
3. an item dropped (exists);
4. **a crate or barricade put down by the player from the inventory.** This is the new record: a *placed in play* list of design, place and its own id, kept with the facts, never in the document;
5. goods moved by a villager (exists);
6. a crate pushed far by physics (where it rests, saved).

**Simulation LOD**, a first tier:
- A scheduled villager in a far region is not simulated. On becoming active again, it is put where its day says it should be at the current hour (its `goes` line), with what its `do` work would have produced, *or* it is put at its last fact if the schedule says it is between places.
- A crop's ticking fact advances by the time elapsed.
- Nothing finer than this.

**Navigation:**
- The placed crate or barricade is solid. Its cell rebakes, and the folk route round it.
- After unload and reload, and after save and load, the same.

**Debugging:**
- View > Regions draws each region's state on the ground.
- A Regions panel lists, per region:
  - its state;
  - things laid;
  - facts about its things;
  - awake and asleep entities;
  - mesh bytes;
  - the last load and unload with their milliseconds.
- The run log records each transition.

**Acceptance criteria** (a demo `regions` with `expect` checks, plus one hand test with a controller):

| # | Criterion | Measure |
|---|---|---|
| 1 | Far regions are truly unloaded | Walking corner to corner, node and body counts in Godot return within 5% of the start. Mesh bytes held go down when regions go far. |
| 2 | Loading a region does not hitch | No frame over 33 ms at a region boundary on this machine, measured over 10 crossings. |
| 3 | All six changes survive unload and reload | Each checked by fact and by what is drawn after walking away and back. |
| 4 | All six survive save, quit and load | The same checks after F5, stop, start, load, at the committed host. |
| 5 | A regenerated world keeps what it can | Re-seed the wood between save and load. The chopped tree stays chopped if its key survives. Facts about trees that no longer exist are listed; none lands on another tree. |
| 6 | Catch-up | Leave at 07:00 and return at 10:00: the miller is at the mill (by its `goes 8 at mill`), within one cell. The crop's stage has advanced by three hours of ticks. |
| 7 | Navigation follows the player's change | A barricade put in a lane: a villager's route goes round within 2 s, and still does after reload. |
| 8 | No regression | Tours `journey` and `saves`; demos `guard`, `behave`, `rules1`; host tests. |
| 9 | Visible | The Regions panel's numbers agree with criteria 1–2 during the demo. |

**Dependencies:** PW0; G1 and G2 (built); facts; `world_view` tiers; per-cell nav; items' drop path (the items tour is broken, so the demo must check drops itself or G6's repair comes first); behaviours' sleep (`rules.gd`).

**Risks:**
- Godot node churn on load: building things, bodies and nav for a 128 m region in one frame. Mitigated by spreading it over frames, as `world_view` already budgets mesh intake at 4 ms a frame.
- Loose physics bodies settling differently after reload. Mitigated by saving where they rest and freezing them until their region is active.
- Facts growing without bound in a long game. Measure them in the Regions panel before designing compaction.
- Scheduled folk catch-up being wrong mid-sequence. Kept to "where the day says", not replayed.
- The two construction routes (cells and drawn) both have to be solid for navigation. Only drawn construction is in scope.
- Players placing things clashes with the editor's undo. Placed-in-play records are facts, never document edits, as every runtime change already is.

**Evidence needed to expand to a seamless benchmark** (PW2):
- criteria 1–3 hold at 128 m regions, with load times that scale linearly when the region is doubled;
- one measurement of a dressed region's Godot cost (drawn houses, plants, snow, shadows), which is also the tall-grass question the user has deferred;
- the host's own scale at a 2 × 2 km document (the engine test suggests about 50–100 MB; confirm);
- the user's hand test: walking away and back feels seamless;
- whether the host should also stop *sending* far cells. Today it sends all of them; at 2 km it may need to send by region.

**PW2 (only after PW1 passes):** a 2 × 2 km world with three settlements and a road, travelled on foot, with simulation LOD tiers for distant settlements (production and population as facts per settlement, advanced coarsely), and building while playing as its first real user.

### 6.3 How this serves the games in view

- **Open-world action adventures** (exploration, camps and dungeons, enemies that stay defeated, chests that stay opened) need criteria 1–5: regions that unload, and changes that stay. Connected worlds already give dungeons and interiors. PW1 gives the overworld itself.
- **Persistent settlement and survival games** (This War of Mine, Tiny Glade with play, a hex-overworld survival game) need things the player adds that stay (case 4), settlements that go on while the player is away (criterion 6), and navigation that follows what was built (criterion 7). PW1 is the first tier of each; building while playing (§7, step 7) is its first real user.
- **Everything authored stays authorable.** Regions are an engine mechanism, not something a creator has to set up. Their only setting would be the region size in World settings, and the Regions panel is a diagnostic. This keeps to the rule that demos must be authorable: nothing in PW1's world is built by script that the user could not build by hand.

---

## 7. Recommended sequence

Each step is one or two passes, with a demo with `expect` checks, and is marked done only after a hand test (the gameplay roadmap's rule).

| # | Step | Why at this point | What it unlocks | Done when |
|---|---|---|---|---|
| 0 | **Hand-test checkpoint** (the user's, about 1 hour): G1 lamp and talk, the behaviours window on a gate, the guard, U4–U7 and U5's layout, a cut fence, autosave after a kill. **And decide** on the ranger's `moves samurai` in `characters.txt`, which breaks the `work` and `talk` tours. | Everything since 7df is untried. Hand tests have moved this project more than any other evidence. | Confidence in G1 and G2 before building on them. | The user's findings recorded; any blocker mended. |
| 1 | **G3: the bell at the gate** (already designed) | Proves conditions, events, states, persistence and group-apply across a thing and a person. Small. | Trust in the behaviour model; the pattern every later G-step reuses. | Demo `bell` with day, night and save/load; hand test with a controller. |
| 2 | **PW0: saved-game integrity** | Small. Closes Section 1. PW1 depends on it. | Saves that survive editing and regeneration. | The three protocol tests in §6.2. |
| 3 | **G4: one action model**, with the village tour mended | The largest single reducer of "needs code": every interaction becomes one definition used by the player, AI and rules. | New interactions authored as content; NPCs using behaving doors. | The village tour passes; a locked chest says why; an NPC opens a gate through the same action. |
| 4 | **PW1: a region that remembers** | Its foundations are now proven (facts, G3, G4's place action). It is the architectural step the open-world direction depends on, and it is better made before items and building multiply what must persist. | Overworld persistence; the measurements that decide PW2. | Criteria 1–9 in §6.2 and the user's walk. |
| 5 | **G6: items, stacks, recipes, containers**, with the items tour mended | Uses G4's actions and PW1's placed-in-play record (crafted things put down). | Survival and crafting loops. | The gameplay roadmap's G6 acceptance: wood → torch → equip → lit, saved and loaded. |
| 6 | **G5: perception of any entity, navigation links, group response** | After PW1, so asleep and awake entities are defined. | Ecology, factions, guards that respond to each other. | Deer flee a wolf; three guards answer one alarm. |
| 7 | **Building while playing, first tier** (needs the user's approval: they postponed it in 7bp) | Reuses drawn construction through the host, recorded as placed in play. PW1 already proves persistence and navigation for it. | Settlement and shelter games. | A wall and a hut built in play persist, are routed round, and survive save and load. |
| 8 | **One genre to the polish bar**, then the export's fault code and multi-world export | Proves the platform end to end and finds the next real gaps. | Something to show and sell. | The user's verdict with a controller; a clean exit code. |
| 9 | **Decide PW2** from PW1's measurements, including the deferred tall-grass pass | Only then is the cost of seamless scale known. | Large worlds. | A written go or no-go with the numbers. |

**Deferred deliberately:**
- G7 combat depth and G8 structured quests: do them when the chosen genre in step 8 needs them. G8 may move before G5 if stories matter first.
- The style pass (before showing a game, so alongside step 8).
- Characters and faces (parked).
- Generated creatures, cloth, new vehicles, card and strategy games, generator version tooling, the radial toolbar, the roof curve.

**Not to be reopened:** construction, terrain and tiles, plants and generators, physics and signals. Take their gaps only when a step above runs into one.

---

## 8. Evidence sources and confidence notes

**Read for this assessment:**
- [Draw3D_Master_Roadmap_Final.md](Draw3D_Master_Roadmap_Final.md): the framework. Its marks are stale.
- [Draw3D_Roadmap_Amendment_2026-10-09.md](Draw3D_Roadmap_Amendment_2026-10-09.md): 7bq, 7br, 7bp, 7bo, 7dh, 7di.
- [PROGRESS.md](PROGRESS.md): "Now", the newest Done entries (7df–U5), Known gaps.
- [Draw3D_Core_Capabilities_Audit_2026-10-21.md](Draw3D_Core_Capabilities_Audit_2026-10-21.md), [Draw3D_Gameplay_Roadmap_2026-10-21.md](Draw3D_Gameplay_Roadmap_2026-10-21.md), [Draw3D_Open_World_Catalogue.md](Draw3D_Open_World_Catalogue.md).

**Checked in code for this assessment:**

| Claim | Where it was checked |
|---|---|
| Facts are a separate layer, saved with the document's hash | [core/src/state.rs](core/src/state.rs) |
| The journey save, `Away` and `load` | [editor/host/src/lib.rs](editor/host/src/lib.rs) around lines 693, 1227, 5074 |
| Saved-game load does not apply id migrations (`resolve_id` is not used anywhere in the host) | [editor/host/src/lib.rs](editor/host/src/lib.rs); [core/src/idmap.rs](core/src/idmap.rs) |
| Ids are hashes of key paths | [core/src/idmap.rs](core/src/idmap.rs) header |
| Draw regions and tiers; content let go | [editor/godot/world_view.gd](editor/godot/world_view.gd) lines 120–185 |
| The host sends all cells (no eye radius in the host) | grep of [lib.rs](editor/host/src/lib.rs) |
| The sleep distance (60/75 m) and scheduled folk never sleeping | [editor/godot/folk.gd](editor/godot/folk.gd) `_left_be` |
| Per-cell nav rebake | [editor/godot/nav.gd](editor/godot/nav.gd) |
| Eyes: first person, top-down, side-on | [editor/godot/eyes.txt](editor/godot/eyes.txt) |
| Engine scale | [core/scale-results.txt](core/scale-results.txt) |
| Export does not carry other worlds | [editor/export.ps1](editor/export.ps1) has no world list |

**Counted here:**
- `#[test]` in the host: 86 unit and 99 protocol, which matches PROGRESS's latest runs (86 and 99 at U5);
- about 125 demo names in [editor/godot/demo.gd](editor/godot/demo.gd) `PARTS`.

**Not run for this assessment:** no tests, tours or demos were run. Every pass or fail cited is PROGRESS's most recent record.

**Confidence notes:**
- **Percentages** are judgements on one consistent scale, not measurements. A difference under about 10 points between sections means nothing.
- **Sections 4 and 9 (combat)** are rated low to medium, because almost everything there has been seen only as readouts and stills.
- **6b (15%)** is high confidence. The absence of unloading inside a world was checked directly in code.
- **"Saved facts do not follow migrations"** was checked by search in the host. It is possible the editor reconciles elsewhere; PW0's first test settles it either way.
- **The working tree** has uncommitted changes to `characters.txt`, `feel.cfg`, `styles.txt`, `weather.txt`, `modelling.txt` and `watchtower.json`. Assessments of the `work` and `talk` tours depend on which version is meant (§3.2, section 8).
