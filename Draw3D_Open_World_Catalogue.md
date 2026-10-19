# Draw3D: open-world styles, mechanics and demos, against what exists

Written 2026-10-17 (amendment 7bp), from the user's direction "Draw3D — Open-World Styles, Mechanics Library and Demo Priorities". That direction asks for a catalogue first: what exists, what is partial, what is missing, and which capabilities unlock the most genres. It does not ask for everything to be built. This is that catalogue. The games it names are references for reusable styles and mechanics, not games to recreate.

**E** exists (built, with a demo), **P** partial (parts exist, or one tier), **N** not built. "Where" is the file or window where it is authored.

## 1. Mechanics, by family

### World generation

| Mechanic | Now | Where, or what is missing |
|---|---|---|
| Land from a seed, shaped by hand, freeze/thaw of what a generator made | E | Land maker, Environment tools, Inspector (freeze) |
| Square and hex layouts; terrain tiles on cells; tile elevation | E | Layout, Terrain tiles (L1, L2, 7w) |
| Structures on cells: walls, fences, buildings, rooms, roofs, ponds | E | Structure maker (L3) |
| Woods, meadows, fields, painted vegetation, plant and tree generators | E | makers; Paint; Plant/Tree Generator |
| Biome families: a style's vegetation, ground and buildings together | P | a style says trees, strewn, ground colours, buildings (`styles.txt`); no biome as a region rule |
| Regions as things (an outline any maker can fill) | N | owed since 7q |
| **Connected overworld → local playable environments** | E (first tier, 7bq) | `leads WORLD NAME`, `enters R`; a hex overworld of tiles whose markers lead into local worlds (genre "Connected worlds"); each world kept as left while running. Items and a saved game of the whole journey go across (7br). Not yet: followers |
| Rooms re-rolled per run | E (first tier) | `room NAME VERSION` (7bp, `rooms.gd`); versions chosen per run; no reordering of rooms |
| Persistent regional state | E (first tier) | each world of a journey kept as left while the world runs (7bq), and all of them in one saved game (7br) |

### Traversal

| Mechanic | Now | Where |
|---|---|---|
| Jump, long jump, roll, dash, dodge, vault, wall run, wall jump, backflip | E | `moves.txt`, the training ground |
| Climbing, ledge hang, wall grip | E | `moves.txt` (climb, ledge_hang, wall_grip) |
| Side-on 2.5D, depth lanes | E | eye `side` (7bo) |
| Swimming | P | the motor's `afloat`; water entry and exit still owed |
| Gliding | N | `jet` and `hover` exist as moves; no glide |
| Sliding (slopes, slippery ground) | P | `slippery` (7bo); no slide down slopes as a move |
| Vehicles | P | boat, ridden animals (`bodied`); no wheeled or flying vehicle |

### Survival

| Mechanic | Now | Where |
|---|---|---|
| Weather (rain, snow, wind, fog), day and night | E | `weather.txt`, `clock.gd`, styles |
| Hunger, thirst, fatigue, temperature, exposure | N as such; P by parts | facts that tick and clear (`ticks`, `clears`), goods, conditions. No meters, no cold from weather or shelter |
| Injuries, energy, fuel | N | |

### Nature and wildlife

| Mechanic | Now | Where |
|---|---|---|
| Plants, colour variation, growth brushes | E | Plant Generator, `grow.gd` |
| Animals that roam, graze, flee from being near, are petted, are ridden | E | `folk.gd` (roams, shies, idles), farm animals in the Library |
| Birds, fish, insects (flutter, swim, hop, perch) | E | `wild.gd` |
| Harvesting, growth over time | N | waits as a gameplay layer |
| Predator and prey, habitats, territory, temperament | E (first tier, 7bt) | `preys`, `flees`, `defends`, `calls`, `side` (`watch.gd`); wolves, deer, a fox and rabbits in the library; the wilds genre |
| Time-based activity | P | `goes HOUR` (folk), `when dark` (watchers) |

### NPCs and factions

| Mechanic | Now | Where |
|---|---|---|
| Schedules, work sequences, goods carried, doors opened | E | `goes`, `sequences.txt`, `work.gd` |
| Talk with answers, reactions, quests, journal | E | `talks.txt`, `reactions.txt`, `quests.txt` |
| Patrols | E | `patrols` (7bo), edited on the world (7bp) |
| Trading | P | goods as facts can be given and taken; no shop or prices |
| Relationships, recruitment, followers, factions, territorial control | P | sides that fight each other and call their own (`side`, `preys`, `calls`, 7bt); followers (`follows`); no attitudes that change, no recruitment |

### Stealth and perception

| Mechanic | Now | Where |
|---|---|---|
| Vision, hearing, concealment, light, alarms, searching, pursuit, being caught | E | `watch.gd` (7bo); each enemy's settings in the Inspector (7bp) |
| Distractions (something thrown, a noise made on purpose) | P | every sound at a place is heard; nothing to throw yet |

### Combat and creatures

| Mechanic | Now | Where |
|---|---|---|
| Melee: strikes, combos, lock-on, weapons drawn and put away | E | Character Studio, `moves.txt`, `gear.txt` |
| Enemies that hunt and catch | E | `hunts` (7bo) |
| Enemies that fight back (attack patterns), health, being hit | E (first tier, 7bt) | `health`, `attacks`, the walker's health and blow (`strike`), dodging, takedowns, bodies found (`fight.gd`); no patterns of several moves, no blocking |
| Ranged weapons, projectiles | P | enemies shoot (`shoots`, 7bt); the walker's own firing awaits approval (Gear plan) |
| Weak points, status effects, bosses and phases | P | `enrages` (7bt); `pattern` (blows in turn, a phase each by health, a roar between), `tires` (spent after), `weak behind`/`weak tired` (7bx: the warlord); no weak points as parts of a body, no status effects |
| Creature assembly | P | kinds of parts on rigs (person, sheep); Creatures and enemies library (7bo); no creature parts as gameplay (weak points) |
| Enemies shown by clip-animated characters | N | the zombie clips of Universal Animation Library 2 are here |

### Inventory and equipment

| Mechanic | Now | Where |
|---|---|---|
| Pickups, inventory, gear worn and carried, loadouts | E | `items.gd`, `gameui.gd`, `gear.txt`; gear dropped by the fallen (`dropped_by`), keys kept and not used up (`key`, `needs key`), what is found by chance (`perhaps`), all a thing has taken at once (`gains everything`) (7by) |
| Crafting materials, recipes | P | goods as facts; work sequences make goods; an act makes goods of others at a station, a lockpick of scrap at a workbench (7bw); no recipe window; nothing made as an item |
| Weight, durability | N | |

### Construction and crafting

| Mechanic | Now | Where |
|---|---|---|
| Building in the editor (structures, walls, rooms) | E | L3; drawn on the land as Tiny Glade does it, towers, dressing, ponds and gardens drawn round (7bu); houses of storeys seen side-on (7bv) |
| Building by the player while playing (shelters, settlements) | N | modular construction postponed by the user; boards broken down are put up again (7bw), nothing new is built |
| Traps | P | hazards (`harms`), pressure plates (`pressed`), mechanisms |
| Player-built vehicles and contraptions | N | |

### Physics

| Mechanic | Now | Where |
|---|---|---|
| Joints, motors, fans, belts, loose bodies, mechanisms, signals | E | Physics milestone (7aj to 7ar), Signals window |
| Breakables | E (first tier) | `breaks` (7bo): gone, a noise, a count; `barricade N` (7bw): so many blows, the folk batter it when it is in their way, it lies where it stood and is put up again with scrap |

### Persistence

| Mechanic | Now | Where |
|---|---|---|
| Save and load a game, facts, broken things, where NPCs stand | E | F9, `state.gd` |
| Discovered locations | P | the journal says what happened; no map of places found |
| Settlement state over time | N | |

### Progression and events

| Mechanic | Now | Where |
|---|---|---|
| Day and night, weather, quests, conditions | E | |
| Seasons | P | styles summer and autumn as looks; no season as time passing |
| World and regional events, knowledge-based unlocks | N | conditions could carry them; nothing schedules an event |

## 2. Styles

The direction asks for styles that are more than material swaps. A Draw3D style already says buildings (wall, roof, storeys, pitch), what grows (trees and strewn designs, how dense), the ground, the light through the day, fog, post-processing, figures' tint, and may take another style's buildings (`styles.txt`, Window > Styles). It does not yet say proportions or silhouettes of creatures, or terrain shape.

| Direction's style | Nearest now | What is missing |
|---|---|---|
| 1 Storybook wilderness | `summer`, `evergreen`, `dreamfield` | painterly ground |
| 2 Graphic fantasy adventure | `bright_blade` | landmarks as a generation rule |
| 3 Atmospheric survival | `last_light`, `highland` | survival mechanics to go with it |
| 4 Nature-reclaimed dystopia | `rust_belt` | ruins and overgrowth as designs |
| 5 Gothic illustrated | `gothic_ward` | distorted proportions |
| 6 Cyberpunk noir | `neon_noir`, `security_feed` | signage, dense urban kit |
| 7 Surreal organic fantasy | `dreamfield` (in part) | giant roots, glowing fungi as designs |
| 8 Layered 2.5D | `bright_side`, `moonlit_stealth` | (the side eye and lanes exist) |
| 9 Minimalist monochrome | `silent_reel` | none, beyond more level pieces |

## 3. The five integration demos

| Demo | What exists for it | What it is missing |
|---|---|---|
| **A** Connected-biome survival (hex overworld) | hex layouts, tiles, styles, weather, items, folk, wildlife; the overworld and local worlds (7bq) | survival needs |
| **B** Physics exploration (BotW/TotK) | the whole physics milestone, climbing, mechanisms | gliding; building contraptions while playing; elemental reactions |
| **C** Side-on stealth and creature ecology | the stealth level (7bo), rooms and cutaway (7bp), enemies set by hand, predator and prey and fights (7bt), a shelter of storeys (7bv) | enemies as clip-animated characters |
| **D** Living village and settlement | the village at work, talks, quests, goods (7ag to 7ai) | shops with prices; building while playing; settlement change over time |
| **E** Faction and creature wilderness | perception, patrols, creatures library; sides, fights, packs, prey (7bt): the wilds genre | resource sites held by someone; attitudes that change |

Demo C is the nearest: it is the existing stealth level, now with rooms and a house.

## 4. What unlocks the most, in the order recommended

Each is a bounded step on what exists. The first three are each larger work and need the user's approval before they are built.

1. **Composable behaviour, second tier** (built, first tier: 7bt). The perception words of 7bo applied beyond enemies: `flees` (prey), `hunts KIND` (a predator of another kind, not only the walker), `defends R` (territory), and enemies that strike back with a move. All of these go through `folk.gd` and `watch.gd`, not a new AI. This unlocks wildlife ecology (C, E), combat (E) and factions later. It is the direction's own "composable behaviour system".
2. **Connected worlds** (built, first tier: 7bq). A thing that leads to another world, such as a door, a cave or a hex tile, saying `leads WORLD [PLACE]`. A saved game keeps each world's facts apart. A hex overworld is then an ordinary world of hex tiles whose tiles lead into local worlds. This unlocks A and E, and every "regions with their own rules" game. It is the largest of these, and the direction's own "connected overworld/local-world system".
3. **Survival needs as meters.** Needs that tick, such as hunger, cold and fatigue, as facts. They show on the HUD and are eased by goods, shelter or fire. Cold could come from weather and the hour, and shelter from 7bp's "under a roof" test. This unlocks A and the survival styles. It is small: mostly words and a HUD row on systems that exist.
4. **Small traversal gaps:** a glide, water entry and exit, and a slide.
5. **Enemies as clip-animated characters:** the zombie and crouch clips already downloaded.

Not recommended yet: building while playing (postponed by the user), vehicles of new kinds, bosses.

## 5. What 7bp built, from this list

- **Enemies set by hand.** In the Inspector, a picked enemy shows how far and how wide it sees, how far it hears, whether it hunts, a camera's sweep, its alarm and when it is awake. Each setting starts as its design's and becomes this enemy's own once changed. Its patrol is marks on the world, dragged; what it sees and hears is drawn on the ground.
- **Rooms that come out otherwise each run:** `room NAME VERSION`.
- **Walls and roofs seen through:** `cutaway`, and a structure's `cutaway` parameter.
- **The frame rate held:** the cause of the inventory freezing the computer.
- **The stealth level** given rooms and a house (Demo C's start).

## 5. After 7bt to 7bv

What it would take to make games of the kind of Tiny Glade, This War of Mine and Into the Dead: Our Darkest Days, and what the enemy section still lacks, is in `Draw3D_Genre_Readiness_2026-10-17.md`, with an order recommended.
