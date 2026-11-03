# Draw3D: gameplay loop tracker

One tracker for gameplay loops. It merges the user's "Gameplay Loops & Mechanics Audit.md"
(its rows and genre index, rated here) with Draw3D_Gameplay_Loop_Analysis_2026-10-25.md
(the loop-by-loop walk; folded into section B below and then removed). Keep it current
every pass that touches a loop. First rated 2026-10-25, after the rewards pass.

## How it is rated

| Rating | Meaning |
|---|---|
| Works | The whole loop runs end to end, and a demo or tour shows it (`run.ps1 -Demo NAME`) |
| Rough | It runs, but a link is manual, fragile or thin |
| Missing | The engine cannot do it without new code |
| N/A | Not a target (no multiplayer, no rhythm game planned) |

**A** after a rating means it can be authored through the interface, with no code (forms,
pickers, Inspector, Project). The rating is the engine's, from code and demos. The
**Hand** column is for the testing phase: the user's own verdict, filled in as they play.
It is blank, not "untested". The user has played much more than the old records said.

The demo names are the `-Demo` parts (PROGRESS.md Demos table). "this pass" means the
rewards pass of 2026-10-25.

---

## A. The audit, rated

### 1. Locomotion and traversal

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Basic movement | Works A | motor.gd, moves.txt; walk/run blend by stick; `-Demo motor`, `course` | |
| Jumping | Works A | jump, land, `falls` hurt (game.txt `uses falls`); `course` | |
| Sprint and stamina | Works A | stamina stat, run costs it, regen; `uses stamina`; `fights` | |
| Crouch and slide | Works A | crouch, slide, low body under things; `-Demo slide` | |
| Climbing | Works A | climbable designs, wall climb, mount top; `climb`, `parkour` | |
| Ledge grab and vault | Works A | vaults, one-hand side vault (this pass), ledge hang; `parkour` | |
| Swimming and diving | Works A | swim, dive, air meter (`uses air`); `-Demo kitpass` diving | |
| Fall and recovery | Works A | hard landing and roll; foes get up again (`gets_up S F N`: onto a knee, then stood; 2026-10-26, `frontier`); a fall broken when flung (`breaks_fall`) and getting up as a move of the set (`recovers`), picked in the Combo graph (2026-10-27, `counters`); no physics ragdoll | |
| Environmental traversal | Works A | ladders as nav links, climbing, boats, vehicles; ziplines ridden down (`zipline X Y Z`, refused if blocked; `hazards`); rising and ferry platforms ridden (`glides`; `storyline`); no monkey bars | |
| Fast travel | Works A | what says `waypoint` (a signpost, every station) is found as you come near; at a found one the others are listed and one chosen takes you there (2026-10-26, `marketday`) | |
| Grapple / hookshot | Works A | grapple to a ledge or point as traversal (this pass), pulls foes too; `parkour` | |
| Wall-run / air dash | Works A | wall run, ceiling crawl, dash, glide, super jump; `parkour`, `rendering` | |

### 2. Combat

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Basic attack | Works A | moves `hits` with phases (windup, strike, recover); `fights`, `combatpage` | |
| Combo strings | Works A | combos, Combo graph, combo screen; dropped when struck; `comboscreen` | |
| Super meter and finishers | Works A | a bar filled by blows that land, more in a combo, letting a class's super go (`when super`); a move that finishes a foe at so much health or knocked down, played instead (builds.txt, moves.txt; 2026-10-27, `builds`) | |
| Heavy / charged attacks | Works A | `heavy`, held presses; `actions` | |
| Blocking and parrying | Works A | block, guard, parry window, `blocks/parries` for foes; `foes` | |
| Dodging | Works A | dodge, roll, untouchable frames (player.gd `untouchable`), stamina | |
| Hit reactions | Works A | stagger, knockback, launch, knockdown, juggle; `airjuggle`, `powerbomb` | |
| Lock-on / targeting | Works A | lockon.gd, switch with the right stick; `foe()` targeting | |
| Projectiles and ranged | Works A | shots.txt, arrows fly along their path and stick (this pass), ammo (`uses ammo`, this pass); `-Demo range` | |
| Status effects | Works A | stats.txt statuses with time, ticks, cures; `abilities`, `powers`; stunned (does nothing) and confused (fights its own), from the dazing and bewildering charms (2026-10-27, `stunconfuse`) | |
| Resource-gated abilities | Works A | powers `costs breath N`; `uses costs`; `powers` | |
| Cooldowns | Works A | powers `every N`, HUD shows it; `powers` | |
| Stagger / poise | Works A | `poise N recovers S`: harm adds up and falls away; past N the guard breaks (staggered) (2026-10-26, `frontier`) | |
| Enemy tells and telegraphs | Works A | phases, `at` strike time, boss patterns; `boss`, `foes` | |
| Counter / reversal | Works A | parry, then a counter within 1.2 s lands two and a half times as hard, more with the skill Riposte (2026-10-26, `hazards`); a move that `counters` another after a parry, a grab turned round (`reversible`), for the player and foes (2026-10-27, `counters`) | |
| Environmental kills | Works A | traps, spikes, falls, `crashes`; a fire pit and an acid pool harm whoever is in them (`hazard N`); fuel drums explode and set each other off (`explodes R D`); the kill is the player's if they struck or chased it in (2026-10-26, `hazards`) | |
| Friendly fire / allegiance | Works A | sides: a blow of one's own side staggers but does not hurt; `fights` | |
| Death and respawn | Works A | Game Shell "You fell", back to the last safe place or a screen; `gameshell` | |
| Boss fights | Works A | phases by health, patterns, drops, a reward on falling; `boss`; a boss of many legs and a stinging tail, legs and tail set in the Inspector (the sand queen, 2026-10-27, `manylegs`) | |

### 3. Grabs, throws and carrying

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Grab an enemy | Works A | `seizes`, grasp.gd; `powerbomb` | |
| Carry the held enemy | Works A | carried body follows; `powerbomb` | |
| Throw / slam | Works A | powerbomb, throws, `crashes`; impact harm; `powerbomb`, `airjuggle` | |
| Carry objects and props | Works A | loose.gd carry and throw, kept as they are; `loose` | |
| Escape / mash-out of grabs | Works A | seized, any button pressed again and again breaks free (`holds_fast N`), the seizer reels; a foe's grab `grab_hold` (2026-10-26, `frontier`) | |
| Grab-vs-grab | Missing | no tie-break between two grabs | |
| Enemy as a shield | Works A | a foe seized from behind (`seizes before`) and held up: shots and blows aimed at the holder strike the held (`shields`); bracing and cover with peeking out to a side (`braces`, `peeks`) (2026-10-27, `cover`) | |

### 4. Interaction and world objects

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Interact prompt | Works A | acts with prompts (E / A); notice marks; `usable` | |
| Doors and keys | Works A | doors as nav links, locks, keys, lockpicks; `finds`, `lockwalk` | |
| Switches and levers | Works A | signals, wires (`on act … do put`), mechanisms; `signals`, `wired` | |
| Containers and loot | Works A | search, chests remember being emptied (facts saved); `finds` | |
| Destructibles | Works A | barricades and placed walls broken by foes (`breaks`); drums broken by a blow or a shot explode (2026-10-26); a crate, a pot, a pillar broken by blows: `sturdy N` (cracks first), `pieces`, loot left lying, `mends H` (2026-10-27, `breakables`) | |
| Pressure plates and triggers | Works A | `senses` volumes, physics plates, puzzles.txt; `mech`, `signals` | |
| Pushable / movable objects | Works A | physics push, carry; plate puzzles; `physics` | |
| Talk to NPC | Works A | talks.txt with answers, outcomes as facts; `talk`, `questform` | |
| Readables and lore | Works A | signs and talks read; goods that `teaches` read; notes in sets (lore.txt) read where they stand (`lore NOTE`), the journal's codex, a set read whole gives and makes a fact; Project › Lore to find edits them as cards (2026-10-27, `sands`) | |

### 5. AI and NPCs

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Basic enemy AI | Works A | watch.gd sight/hearing, approach, attack, react; `foes`, `stealth` | |
| Patrol routes | Works A | patrol, investigate, return; `guard` | |
| Pathfinding and navigation | Rough | navmesh with doors and ladders as links; storeys work; a chaser stuck twice plans a way round (2026-10-26; the demo's chaser went round by its usual way, so this path is not yet shown); can stick on cluttered drawn bases | |
| Alert states | Works A | unaware, suspicious (?), hunting (!), searching, calm; marks sized by a setting (this pass) | |
| Group coordination | Works A | foes hunting one go round them to their own places on a ring, only `pack N` strike at once and the rest wait wider, shooters keep apart (pack.gd); ways begun from where the ways are (2026-10-27, `packs`) | |
| Fleeing and surrender | Rough A | `flees` when low; no surrender or spare choice | |
| Companion AI | Works A | `companion [S]`: follows, fights your foes, is down not dead and helped up (E), helps the player up when they fall (2026-10-27, `packs`; Ash beside the start in the wastes town) | |
| NPC schedules | Works A | Routines (Ctrl+H), `goes HOUR at PLACE`; a row only while a condition holds (`goes 6 … if weather rain`, Routines' *only if*, 2026-10-26, `storyline`); `routines`, `village` | |
| Factions and reputation | Rough A | sides decide who fights whom; standing at shops (known at 5, a friend at 15; a quest done counts at every shop) lowers prices and is a condition (`standing N at SHOP`; 2026-10-26, `marketday`, `storyline`); no standing with a side | |
| Escalating alarms | Works A | `calls`, the bell at the gate, alarm to whom; `bell` | |
| Crowd variety | Works A | people chosen together given looks of their own (hair, beard, outfit pieces, cloth colours, face at rest, height), drawn from who each is; *Vary their looks* (2026-10-26, third brief, `vary`) | |
| Watching anyone | Works A | the right click on any of the folk or a creature: *Follow NAME*, the eye with them as they go; moving brings it back (2026-10-26, third brief, `follow`) | |

### 6. Quests and narrative

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Accept a quest | Works A | talks offer, journal, markers; `talk`, `questform` | |
| Track progress | Works A | quest steps with counts (`has N`, facts); journal updates | |
| Complete and reward | Works A | turn-in once, facts change the world; experience for a quest: its Experience field (`earn N`, 2026-10-27, `genreform`), or by `on … do` | |
| Quest states | Works A | new, taken, done, failed, ended; conditions ask each | |
| Branching quests | Works A | answers set facts, later steps read them | |
| Objective types | Rough A | fetch, talk, reach, use, kill (by facts); letters posted, made, felled (by kind or name), a place found, standing at a shop (2026-10-26, `storyline`); escort and defend as behaviours (`defend`, presets `bodyguard`, `escorted`, `holds_ground`, 2026-10-26); a quest's *It fails when* card and *＋ An escort* (2026-10-27, `escortghost`); harvested, broken, foes finished, made better at an anvil, a level, a class (2026-10-27, `breakables`) | |
| Escort quest | Works A | Project › a quest › ＋ An escort: a step to bring them, a hint while left behind, fails when they fall, they follow while it is taken; one change (2026-10-27, `escortghost`) | |
| Timed quests | Rough A | fail at an hour (`supper begins`); no visible countdown | |
| Quest chaining | Works A | `quest NAME done` as a condition | |
| Journal / log UI | Works A | journal (J), track; `talk` | |
| Hidden discovery | Works A | regions found, `senses` can start a quest; a region's finds come up when searching in it (Regions › Found when searching here; 2026-10-26, `marketday`) | |
| Dialogue trees | Works A | talks with conditions on answers; `talk` | |
| Cutscenes | Works A | Project › Cutscenes: shots of who, how long and the words; each shot's camera, fades and a fact it sets; ▶ Play it plays the game and the scene and comes back (2026-10-26, third brief, `scenes`); played once when a condition holds, or by a wire (`do scene`); the camera kept out of walls, the display put away, skipped by any button (2026-10-26, `storyline`) | |
| Lore breadcrumbs | Rough | facts and journal lines; no codex | |

### 7. Progression and loot

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| XP and levelling | Works A | rewards.gd: earned by fights, picking and crafting, levels curve, "Level N" (this pass); `-Demo rewards` | |
| Skill trees | Works A | Project › Skills and the game menu's Skills: points a level, skills that need others, giving health, running, a harder counter, money off; learnt as facts (2026-10-26, `storyline`); a skill raises a power's level (`raises POWER N`; a level `when learnt`), Ember bolt to two shots then three that home (2026-10-26, third brief, `skillrace`) | |
| Stat allocation | Works A | attributes raised with points at the start and each level (builds.txt), spent on the game menu's Character page, added to what skills give; Project › Builds (2026-10-27, `builds`) | |
| Equipment effects | Works A | gear changes blows; `items`, `kitpass` | |
| Loot drops | Works A | kinds' `loot GOOD N CHANCE`, icons, pickups drawn to you (this pass); `rewards` | |
| Item rarity | Works A | rarities with colour and strength, `rarity random` drawn as first found (quality.txt); Project › Items, rarity and enchanting (2026-10-27, `builds`) | |
| Random affixes | Missing | | |
| Buffs and debuffs | Works A | statuses with icons and time; potions | |
| Consumables | Works A | goods used (eat, drink, health potion), count falls | |
| Crafting | Works A | recipes at benches; recipes learnt and wanting a level or a bench tier (this pass); `crafting`, `rewards` | |
| Upgrade / enchant | Works A | a bench upgraded a tier; gear made better (+1, +2…) and enchanted at an anvil (`forges`, "improve") for goods, an enchantment putting a status (2026-10-27, `builds`) | |
| Vendors | Works A | buying and selling by talks and acts (`village`, `drygulch`); a shop window (`sells`, `stocks`): its prices, what it buys, standing (2026-10-26, `marketday`) | |
| Character builds | Works A | starting classes chosen at the Player start (warrior, ranger, bruiser: gear, set, combos, a finisher, a super), attributes, skills of one class (builds.txt, 2026-10-27, `builds`) | |

### 8. Inventory and equipment

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Pickup and storage | Works A | goods and items kept in saves; `items` | |
| Stacking and limits | Works A | goods stack; the pack holds free, so many things or so many kg (Project › The game's flow; a good's *One weighs*), full refuses picking up, gathering and buying (2026-10-26, `frontier`) | |
| Equip and slots | Works A | gear on the body's places, seen; stats from gear | |
| Hotbar / quick use | Works A | a hotbar of places 1-9 (keys; LB+Left/Right/Up on a pad) filled by what is carried or set in Controls & Input › Hotbar; the wheel's Weapons and Use lists, Y puts a slot on the hotbar; the inventory's On the hotbar (2026-10-27, `hotbar`) | |
| Drop, trade, store | Rough A | chests store (`stores`), a horse's saddlebags (2026-10-26); a trader who swaps (`trades GIVE N GET M`: the desert caravan, 2026-10-27, `sands`); drop thin | |
| Item degradation | Works A | `wears N`, "wearing out", breaks, mended at a bench (this pass); `uses wear` | |
| Compare items | Works A | the inventory compares the chosen thing with what is carried of its sort: strength, rarity, upgrades, enchantment (2026-10-27, `builds`) | |

### 9. Economy and world simulation

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Currency loop | Rough A | coins dropped and paid by talks; prices are words, not a system | |
| Shop stock and refresh | Rough A | what a shop stocks runs out (`stocks GOOD N MOST`; `marketday`); restocking over the days not checked by a demo | |
| Resource nodes | Works A | felled and mined, reseeded and caught up with time (PW1); `harvest` | |
| Farming loop | Works A | fields watered, fed, picked (`gathering`); beds tilled with a hoe, sown, watered with a can, grown by the hours, harvested with a sickle: ten crops in Project › Crops (2026-10-27, `farm`); farmed by a worker (`workers`); Furrowfield, three field hands through tilling, sowing, watering and harvest, seed boxes opened into packets (2026-10-27, `furrowfield`) | |
| Day / night | Works A | sky, light, night spawns by hour conditions; `seeing` | |
| Weather | Works A | weather.txt with sound and wind; affects cold; weather by place (fronts) and by region; weathers of your own from the panel; lightning, thunder a guard hears, hail that hurts; dust storms with tumbleweeds, a gale that jostles; a calendar of seasons (2026-10-26, `skies`); what lies when it changes: snow kept, melted or cleared (Environment › What lies, or the step's own), prints by who makes them (`treads`) (2026-10-27, `snowclear`, `treads`) | |
| Territory control | Missing | regions have sides in the form, nothing captures them | |
| Supply and demand | N/A | not planned | |
| Persistent world changes | Works A | PW0/PW1: felled stays felled, dead stay dead, through saves | |

### 10. Stealth

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Vision and detection | Works A | light and cover read; `stealth`, `perceives` | |
| Noise | Works A | running loud, crouching quiet, `hears`; `guard` | |
| Stealth takedown | Works A | takedown from behind; body stays; `stealth` | |
| Hide bodies | Works A | bodies carried; one of a side finding another fallen raises the alarm (`frontier`); a body in tall grass or a haystack is not seen (2026-10-26, `marketday`) | |
| Disguises | Missing | | |
| Alarms and search | Works A | alarm, search, calm after time; `bell`, `guard` | |
| Lockpicking | Works A | a lock picked by an act wanting a lockpick; no minigame; `finds` | |

### 11. Platforming and puzzles

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Precision jumps | Works A | course with checkpoints, fall and back; `course` | |
| Moving platforms | Works A | a rising and a ferry platform (`glides`) carry who stands on them; ridden up 5.4 m (2026-10-26, `storyline`) | |
| Timing challenges | Rough A | mechanisms on ticks (`clears/ticks`); hazards thin | |
| Multi-step puzzle | Works A | puzzles.txt, signals in order; `mech` | |
| Physics puzzle | Works A | own physics, plates, boxes; `physics` | |
| Boss-as-puzzle | Rough A | phases by conditions possible; no demo | |
| Checkpoint and retry | Works A | checkpoints, autosave; `gameshell` | |

### 12. Building, management and strategy

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Unit selection and orders | N/A | not planned | |
| Turn-based combat | N/A | not planned | |
| Resource production chain | Works A | `produces OUT N from GOOD M every MIN holds K` (produce.gd): wheat stook, millstone, log pile, baker's oven, saw bench (Library › Village work); put in and take out (E); what it waits for in HOW IT WORKS; Millbrook's miller carries by his day (2026-10-27, `chain`) | |
| Base building | Works A | build mode while playing, joins and stacks, attaches to what stands, decks on posts (this pass); `base`, `rewards` | |
| Worker assignment | Works A | jobs (jobs.txt, Project › Workers): hours, place, things tended or beds farmed, harvest to the player or a stall, a wage; given in Project, the Inspector, or in play ("give work to"); placed with work; Mae farms at Millbrook (2026-10-27, `workers`); a job picked on the card over a person while editing (`furrowfield`) | |
| Automation (conveyor) | Rough | a conveyor kind moves things; no machine chain | |
| Tech / research tree | Rough A | recipes learnt, bench tiers (this pass); no tree view | |
| Fog of war | Works A | the map dark until walked (areas.gd `fog`) | |
| Population / needs | N/A | not planned beyond folk needs | |
| Win / lose conditions | Works A | endings.txt, You fell | |

### 13. Racing, sports, score

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Race with checkpoints | Works A | gates in order (`course RACE N`, Abilities › is a gate of a timed course): the lowest starts the clock, one out of turn is told, the highest stops it; a clock at the top, a light over the next gate (2026-10-26, third brief, `skillrace`) | |
| Laps and position | N/A | | |
| Time trial and ghosts | Works | a course times the run and keeps the best (`race_best.RACE`, `race_runs.RACE`, kept by a saved game, asked by conditions; 2026-10-26, `skillrace`); the best run's ghost beside the next, ahead or behind at each gate, kept on this machine, can be turned off (2026-10-27, `escortghost`) | |
| Rules and fouls | N/A | | |
| Score and combo | Works A | a chain of hits: each landed within the seconds set keeps it going, a move new to the chain scores more, ×N and its points on the display (its own piece in the display designer); ended, into the facts `score` and `best_chain` with experience; broken by a blow, half kept; set in Project › Builds, THE CHAIN card (2026-10-29, fourth pass, `hitchain`) | |
| Match flow | N/A | | |
| Rubber-banding | N/A | | |

### 14. Cards and turn-based: N/A, all rows

### 15. Rhythm: N/A, all rows (music-driven hazards could come later through wires)

### 16. Investigation and social

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Evidence discovery | Rough A | facts and journal | |
| Connect the dots | Missing | | |
| Interrogation | Rough A | talk answers gated by facts and goods | |
| Accusation / verdict | Rough A | answers with outcomes | |
| Social simulation | Missing | | |

### 17. Survival

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Needs meters | Works A | hunger, thirst, tiredness, cold, radiation; each optional (`uses needs`) | |
| Gather, craft, sustain | Works A | fell, mine, craft; tools wear and are mended (this pass) | |
| Day/night survival | Rough A | night threats by hour conditions; a bed slept in till morning and woken by after a fall (`until`, `wakes_here`, 2026-10-26); shelter matters little | |
| Base defence | Works A | walls battered and broken (`breaks`), turrets turn and spend ammo; the raid a preset on Project › Bases and raids (2026-10-26, `frontier`) | |
| Death penalties | Rough | back to the last safe place, or the bed last slept in; nothing dropped | |
| Cooking | Works A | at anything that `crafts fire`: cooked meat and fish, a stew, herbal tea, boiled water; raw may make you sick (2026-10-26, `frontier`) | |
| Seasons and a calendar | Works A | Environment › The calendar: seasons go by with the days; winter snows, freezes ponds, harvests less (`season` a fact; 2026-10-26, `skies`) | |
| Batteries and light | Rough A | what `drains battery S` (a flashlight) uses one up every S seconds lit; batteries bought and found by region (2026-10-26); no demo of a light going out | |
| Zone exposure and protective gear | Works A | a region's *What being in it does* (a stat an hour); worn gear `shields STAT F`: a gas mask half, a hazmat suit three quarters, found in the wastes' camp (2026-10-26, `frontier`). Before this pass only drums and weather raised radiation, and nothing kept it off | |
| Infection | Works A | statuses that `turns` worse untreated (infected, sick, fevered); a bite `inflicts infected`; herbal tea or a bed ends the first stage, antibiotics any (2026-10-26, `frontier`) | |
| Unsafe water and purifying | Works A | dirty water from troughs and ponds may make you sick; boiled at a fire or made clean with a tablet at a bench (2026-10-26) | |
| Generators, fuel and power | Works A | `generates N reach R burns F` burns its own fuel by the hour; what `draws N` within reach is lit or fires while power is spare, the nearest first; dry, it stops (power.gd, 2026-10-26, `frontier`). Before, a generator was a model | |
| Firearms and ammunition | Works A | reload and `uses bullet` were there; rounds found in crates, bought in Dry Gulch, pressed from scrap and gunpowder at a bench (2026-10-26) | |
| Mounts | Works A | a saddled horse: tack drawn on its bones, ridden with the hands on the reins, saddlebags (2026-10-26, `frontier`); petted at its head, ridden from its side, hold E for the other; the rider seated and galloping (third brief, `riding`); at a gallop the rider stays in the saddle and the hands keep to the reins; camels ridden in the far desert (2026-10-27, `riding`, `sands`) | |

### 18. Multiplayer: N/A, all rows (Draw3D is single-player)

### 19. Meta progression

| Loop | Engine | Where / evidence | Hand |
|---|---|---|---|
| Save / load | Works A | slots, Continue, autosave; `gameshell` | |
| Checkpoints | Works A | | |
| Permadeath and runs | Missing | | |
| Unlocks | Rough A | facts last in a save, not across games | |
| Achievements | Missing | | |
| Difficulty settings | Works A | Project › The game's flow › *How hard*: easy, normal, hard scale foes' health and the harm to the player (game.txt `difficulty`; 2026-10-26, `storyline`) | |
| New Game+ | Missing | | |
| Tutorials / onboarding | Works A | lessons (tutorials.txt, Project › Tutorial) shown one at a time in play with the device's buttons, done by doing them or put by, kept as learnt; quest hints, HUD prompts (2026-10-27, `tutorial`) | |

### 20. Presentation and feedback

| Check | Engine | Notes | Hand |
|---|---|---|---|
| State visibility | Works A | HUD in hud.txt, bars optional per game | |
| Feedback on action | Works A | sounds, effects, rising +XP and pickups | |
| Success / failure clarity | Rough | tells say why; a few silent refusals remain | |
| Retry flow | Works | | |
| Audio | Rough A | positional, weather, hits; music by region; no reactive music | |
| Camera | Rough | follows; walls handled; tight interiors still clip; demos now frame both fighters (this pass) | |
| Remapping and accessibility | Rough A | Controls & Input pages, layouts switched in play (this pass); no colour-blind settings | |

---

## B. The loops walked (from the 2026-10-25 analysis, updated)

What a loop needs: a reason to start, an action, a response, a reward or a cost, and a way
back that is not the same as the first time.

| Loop | What was missing that morning | Now |
|---|---|---|
| Fight | a lasting reward | **Done**: experience, levels, loot by chance, trails; skill points to spend in a tree (2026-10-26) |
| Survive | cooking; a bed worth more | **Done 2026-10-26**: cooking at a fire, a bed slept in till morning and woken by; infection, unsafe water and zones too |
| Gather and make | tool wear; unlocks; a goods sink | **Done this pass**: wear and mending, recipes learnt, by level and by bench tier, a better bench as a sink |
| Base | placed walls battered; raid rule; turret turning and upkeep | Walls battered (`breaks`), turrets turn, ammo upkeep, attach to what stands, decks on posts: **done**. A raid offered with a base: **done 2026-10-26** (Project › Bases and raids) |
| Sea | fire back from a boat; sinking; boarding | Unchanged: missing |
| Move | run timer; grapple traversal | Grapple: **done**. A run timer with a best kept: missing |
| Explore | a find tied to a region; fast travel | **Done 2026-10-26**: a region's finds; travel points found as you come near, travelled between (`marketday`) |
| People | standing / reputation | Standing at shops: **done 2026-10-26**. Standing with a side: missing |

## C. Recommended next, for the user to choose

1. ~~Cooking and a bed~~ and 2. ~~a raid offered with a base~~: done 2026-10-26.
3. ~~A run timer with a best kept, and checkpoints in order~~: a timed course, done 2026-10-26 (third brief). Its ghost, done 2026-10-27 (`escortghost`).
4. ~~Skill points to spend~~: a skill tree, done 2026-10-26; power levels raised by skills done the same day (third brief).
5. ~~Grab escape~~ (done 2026-10-26); a held body as a shield (grabs, the engine's speciality).
6. ~~Difficulty settings~~: done 2026-10-26.

---

## D. What the assistant can do for these loops

Programmed this pass (godot/assisttools.gd; `-Demo rewards` beat 7 shows it):

- `words` topic `rewards` and one topic per sort (good, recipe, quest, talk, item, power,
  shot, stat, effect, act): it reads each file's own words before writing.
- `get_entry` / `write_entry`: it drafts any of those entries whole, checked by the reader
  the game uses, so a wrong word is said and mended before the person sees it.
- `game_setting`: uses (what the game leaves out), earns, levels.
- `give` already gives a design `loot`, `xp`, `ammo`, `tier`, `on … do`, `while …`.

The simpler tasks it can now do from a sentence:
- foes' drops and experience;
- a recipe, with level, tier or learnt;
- a good that teaches a recipe;
- a potion;
- gear that wears or leaves a trail;
- a quest with steps and a reward;
- a talk with answers;
- a power or a shot;
- a status;
- turning bars and costs off for a game;
- a bench put down;
- a wire for a raid at an hour (`on hour 20 do put horde`).

Complex tasks it could take on with what it has, by many drafted steps:
- a whole quest line across talks, quests and facts;
- a base with walls, a gate and turrets, and its raid wire;
- a boss with phases (kind lines and a power);
- a small dungeon (model, build, things put down, a key and a lock).

Added 2026-10-26: the tool `plan`: asked for something big, it plans first; the steps shown as a checklist in its panel, each drafted and applied alone, *Next step* for the one after (`storyline`).

These would want new tools first:
- placing folk with routines (no tool reaches Routines yet);
- regions (areas.txt);
- the HUD;
- characters and move sets;
- checking a loop by playing it (it cannot play).

## E. Estimates asked for (parked, not built)

**Meshes for drawn houses' hedges and roofs.** Hedges are made by the host from parts
(build.rs, four `hedge_form`s), and roofs from generated faces with a `roof_texture`.

- *Hedges from meshes*: about one pass.
  - A `hedge_form mesh` look.
  - The host lays a design's mesh along each piece, stretched to its length.
  - Two or three CC0 hedge models brought in.
- *Roofs from meshes* (tiles, shingles, thatch as geometry): one to two passes.
  - Rows of a tile mesh along each roof face.
  - The risk is the hips and valleys of irregular drawn outlines.
  - A normal-mapped tile texture on the faces gets most of the look in a fraction of the
    time, and could come first.

**Faces from pictures.** A face made from a photo needs a face-landmark model (as
MediaPipe's), which Godot does not have. It would be brought in as ONNX, a large bet for
the precision it gives. Recommended instead: the picture shown behind the face editor, so
the twelve shapes and their knobs can be matched by eye. That is under a pass, and as
precise as the shapes allow.

## F. Alert marks: what others do (research for this pass's marks setting)

What was found:
- Tenchu shows awareness as a meter in a corner, not over heads.
- Skyrim's popular detection-meter mod puts one small eye over the screen's centre that
  fills.
- Mark Brown's "School of Stealth" (GMTK) argues the readable states matter more than
  their size: unaware, suspicious, alert, each shown distinctly and briefly.

Draw3D keeps its ? and ! over heads, with three changes:
- they are smaller;
- they no longer hide what they are over;
- they are sized by the `marks` piece's scale in Window > Game display.

Outlines of foes within reach are a setting (hud.txt `reach R`) and an ability
(`keen_sense`).

Sources:
- strategywiki.org/wiki/Tenchu:_Stealth_Assassins/Gameplay
- nexusmods.com/skyrimspecialedition/mods/63057
- amara.org, the GMTK "School of Stealth" transcript

Icons downloaded this pass: 7Soul1, "496 RPG icons" (opengameart.org, CC0), credited in
editor/assets/CREDITS.txt.
