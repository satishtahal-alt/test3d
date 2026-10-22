# Draw3D: mechanics reference (from the game style library)

Written 2026-10-17 (amendment 7bn). This file covers the gameplay of the games in `Draw3D_Style_Library.md`, kept apart from their looks. Nothing here was built because a game has it. Each mechanic is traced to the **underlying reusable system** it would need and placed against what Draw3D has now.

As with the style catalog, the 30-game reference document was not provided, so this covers the eight priority games named in the brief. The sources are listed in the style library.

**Classes:**

- **E (Existing):** Draw3D can author this now.
- **R (Reusable, expand):** a system exists that would carry it with more words or tiers.
- **N (New, investigate):** no system yet, and one is needed.
- **U (Unverified):** in the brief or the sources too thinly to plan on.

## Built since (2026-10-17, amendment 7bo)

The user approved the two recommended systems and the reference's mechanics, cheapest first. What is now **E (existing)**, and where:

| Mechanic | Games | Now | Where |
|---|---|---|---|
| Side-on 2.5D movement on a line, lanes | ItD, GF, SS, SR, WR, DP | E | eye `side` (`eyes.txt`: `round`, `plane lanes`); the Player start's `eye` |
| Sight cones, hearing, being seen, light and darkness | ItD, SS, WR, DP | E | `sees`, `hears` (`watch.gd`); visibility by crouch, stillness, cover, dark |
| Noise draws enemies | ItD | E | steps, landings, breaking, every sound at a place, alarms |
| Enemies that patrol, hunt, search, catch | ItD, WR, SS | E | `patrols`, `hunts`; caught back to a checkpoint |
| Security cameras | DP | E | `sees`, `sweeps`, `spots NAME` (an alarm fact others hear) |
| Camouflage when still | DP | E (as a share of visibility) | stillness lowers how well one is seen |
| Hiding spots | WR, ItD | E | `hides` |
| Hazards | DP, GF | E | `harms` |
| Slippery surfaces (sludge) | DP | E | `slippery` (the motor's `slip`) |
| Destructible objects | ItD, SR | E (first tier) | `breaks`: struck or dashed into; no pieces kept |
| Checkpoints | all | E | `checkpoint` |
| A stalker only in the dark | WR | E | `when dark` on any watcher |
| Survivor needs, a barricade that wears | ItD | E by parts, no level shows it | ticks, clears, work, goods |

Since 7bp also **E**: rooms re-rolled per run (`room NAME VERSION`, first tier: a version per room, not reordered), cutaway walls (`cutaway`). Each enemy's settings are in the Inspector. The open-world direction is catalogued in `Draw3D_Open_World_Catalogue.md`.

Still **N**, for the user: wheeled and flying vehicles, character switching mid-level, guardians and encounter phases, magnetism and light beams, ink, pixel figures, enemies shown by clip-animated characters (the zombie clips of Universal Animation Library 2 are here for it).

The level that shows them: World > New world, "Genre: Side-on stealth (2.5D)" (`editor/genres/stealth-side.json`; `run.ps1 -Demo stealth`).

## Per game

### Into the Dead: Our Darkest Days (PikPok): verified from previews

- **Core loop:** keep a group of survivors in a shelter; scavenge by day; hold the barricade; manage the team. Several people can be sent out, but not to the same place.
- **Movement:** walking side-on on small 2.5D maps; sneaking.
- **Interactions:** scavenging containers; repairing the barricade.
- **Combat:** fight, sneak past or dodge the undead.
- **Simulation:** the barricade wears down over time; the survivors' needs.
- **Distinctive:** a flashlight and noise draw the undead.

| Mechanic | Class | Draw3D system |
|---|---|---|
| Walk, sneak, dodge | E | motor, `moves.txt` (crouch, dodge, roll) |
| Side-on 2.5D movement on a line | N | a constraint on the motor plus a side eye (`eyes.txt` has follow eyes, none held to a plane) |
| Containers to search, loot | R | items (7bi), goods as facts (7ah), acts (`acts.txt`) |
| A barricade that wears and is repaired | R | facts with clears and ticks (7am signals), work sequences |
| Noise and light attract enemies | N | perception: hearing and sight cones; `sound.gd` makes sound but nothing listens |
| Sending a team out | N | squad orders / NPC tasking on top of `goes HOUR at PLACE` (7ag) |
| Survivor needs (hunger, morale) | R | facts and ticks; no needs model yet |
| Cutaway buildings | U / N | a camera-driven cut of walls and roofs (rendering); not in the sources |

### GREENFIELD (Six Foot Giraffe): verified from announcement and listings

- **Core loop:** explore a dream island; reach five energy stations; gain suit abilities; solve puzzles; defeat guardians.
- **Movement:** platforming; a jetpack; vehicles on land, air and water.
- **Puzzles:** energy-themed (combustion, hydro, wind, magnetic, light).
- **Progression:** abilities gate new areas.

| Mechanic | Class | Draw3D system |
|---|---|---|
| Platforming, jumps | E | motor, moves, ledge, `when` words |
| Jetpack | R | gear `jetpack` exists in `gear.txt`; flight is a set's `air` way, so needs a move set for powered flight |
| Vehicles: land, water, air | R | ridden things go by their own set (`acts.gd`, `afloat ahead`, the boat); air vehicles are not done |
| Energy puzzles (wind, magnetism, water, light) | R | physics mechanisms (joins, blows, carries: 7al), signals (7am); `blows` is wind already; magnetism and light beams are new words |
| Abilities gate areas | R | facts plus `when` and conditions; an ability list is not modelled |
| Guardian bosses | N | encounter scripting: phases and arenas (see the shared map below) |

### Silent Sadie (Kapra Games): one listing; most of the brief unverified

- **Core loop:** puzzle platformer with stealth, in 1920s Hollywood.
- **Movement:** acrobatic (vaudeville).

| Mechanic | Class | Draw3D system |
|---|---|---|
| Acrobatic moves | E | `vault`, `wall_run`, `backflip`, combos, the Character Studio |
| Stealth | N | perception (shared with the other stealth games) |
| Slapstick, expressive animation | U / R | moves with phases, reactions to being struck (`upon struck`), the shot camera; "slapstick" as a system (falls, props that react) is not specified |
| Vintage film presentation | E (as a style) | `silent_reel`: `film`, `grade` |

### Steel Rats (Tate Multimedia): verified from reviews

- **Core loop:** ride through 2.5D industrial levels destroying machines.
- **Movement:** motorbike physics: ramps, wall-riding, flips.
- **Combat:** a chainsaw front wheel; weapons; switching between four characters; ultimates.

| Mechanic | Class | Draw3D system |
|---|---|---|
| A rideable bike | R | ridden things (`acts.gd`) with a `ground ahead` set; a bike's physics (lean, wheelie, air control) is new |
| Vehicle physics on ramps | N | wheeled vehicle bodies (Jolt `VehicleBody3D` or own; physics milestone 7aj–7ar is rigid bodies and joints) |
| Wheel or vehicle as a weapon | R | `strikes PLACE R` on a move; a vehicle's strike shape |
| Character switching | R | play as (`playas`), one walker; switching mid-level is new |
| Destructible machines | R | physics `hits`, `breaks` not built; facts on things |

### Neon Blood (ChaoticBrain Studios): verified from reviews

- **Core loop:** adventure in a dystopian city; investigation; dialogue; some combat.
- **Interactions:** talking, examining.

| Mechanic | Class | Draw3D system |
|---|---|---|
| Dialogue with choices | E | talks with answers (7ai), journal |
| Investigation, clues | R | facts, quests (7ai), markers |
| Turn or real-time combat | R | strikes and reactions; turn-based is new |
| Pixel figures in 3D | N | a figure rendering mode (sprite or low-res), not a mechanic |

### Withering Rooms (Moonless Formless): verified from reviews

- **Core loop:** survive a mansion-asylum that rearranges on death; build a character; escape a dream loop.
- **Combat:** melee and spells; clunky by reviews.
- **Progression:** many builds; New Game+.
- **Distinctive:** the house changes each time you die.

| Mechanic | Class | Draw3D system |
|---|---|---|
| Rooms rearranged procedurally per run | R | layout on cells (L1–L3), makers; a seed per run and a re-roll on death are new |
| Death loop, new run | N | run state kept apart from the world (saved game vs facts) |
| Builds: gear, spells | R | gear, items, inventory; spells are moves plus effects (effects not built) |
| Hiding from stalkers | N | perception plus hiding spots (acts) |
| Detailed interior props | R | the library; L3 rooms; "fill a room by style" is new |

### Darwin's Paradox! (ZDT Studio): verified from reviews

- **Core loop:** an octopus escapes a factory to the sea: platforming, puzzles, stealth.
- **Movement:** swimming; sticking to walls and ceilings; slippery sludge.
- **Interactions:** ink against cameras; camouflage while still.

| Mechanic | Class | Draw3D system |
|---|---|---|
| Swimming | E | the motor's swimming and floating |
| Climbing any surface, ceilings | R | `wall`, `ledge`, `gripped` words; full surface-sticking is new |
| Surfaces that change grip (sludge) | R | ground kinds and their numbers; a "slippery" surface word is new |
| Camouflage when still (not seen) | N | perception: a visibility value that a body's state lowers |
| Ink blinds cameras | N | perception plus a blocking effect |
| Security cameras | N | perception: a sight cone on a thing |
| An unusual body (tentacles) | R | the Character Studio (any rig); procedural tentacles are new |

### Cloven Blade: not found

The brief's words only: "colourful fantasy, agile combat and acrobatic movement". Agile melee and acrobatics are E/R: moves, combos, lock-on, the eight ways (7bl), `vault`, `wall_run`, `backflip`. Nothing can be verified.

## Shared capabilities, mapped against Draw3D and its roadmap

These are the systems the games share. A tick in a column means the game uses that system.

| Capability | ItD | GF | SS | SR | NB | WR | DP | Draw3D now | Proposed tier |
|---|---|---|---|---|---|---|---|---|---|
| **Perception and stealth:** sight cones, hearing, light and darkness, being seen, hiding | ✓ | | ✓ | | | ✓ | ✓ | none (the `dark` fact exists; `sound.gd` plays sounds) | **first:** senses on a thing (a sight cone, a hearing radius) as `when` words; a body's visibility from light and stillness. Four of the seven games need it. |
| Context-sensitive movement and traversal | ✓ | ✓ | ✓ | ✓ | | | ✓ | strong: motor, moves, when-words, Studio | grip surfaces (sludge, ceilings); a 2.5D plane |
| Vehicles and machinery | | ✓ | | ✓ | | | | boat, ridden things, mechanisms | wheeled and flying vehicle bodies |
| Combat, enemies, bosses | ✓ | ✓ | | ✓ | ✓ | ✓ | | moves, strikes, reactions, lock-on, gear | enemy AI that fights back; encounter phases |
| Environmental puzzles, switches, doors, hazards | | ✓ | ✓ | | | | ✓ | physics, mechanisms, signals, puzzles, doors | hazard words (hurts on touch), magnet and light beams |
| Destructible objects | ✓ | | | ✓ | | | | none | `breaks` on a thing: pieces, a fact |
| Scavenging, inventory, crafting, survival | ✓ | | | | | ✓ | | items, inventory, goods as facts, work | needs (hunger and morale) as ticking facts; crafting recipes from goods |
| Procedural assembly, interconnected exploration | ✓ | ✓ | | | | ✓ | | layout L1–L3, makers, doors as links | a seed per run; re-rolled rooms |
| Character animation, interaction poses, effects | ✓ | | ✓ | ✓ | | ✓ | ✓ | the Character Studio, acts, poses, sway, shot camera | effects (particles); procedural limbs |
| Side-on 2.5D presentation | ✓ | ✓ | ✓ | ✓ | | ✓ | ✓ | none | a side eye plus a plane constraint: six of seven games are 2.5D |

**Two findings stand out:**

1. Six of the seven verified games are played side-on in 2.5D.
2. Four need perception and stealth.

Neither exists in Draw3D. Both are reusable and system-level, not one-off, and they are the two capabilities this catalog recommends investigating first. Neither is built in this milestone. Each needs the user's approval as a milestone of its own (by the rule that larger bets are flagged first).
