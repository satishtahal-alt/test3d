# Draw3D: what it would take to make games like these

Written 2026-10-17, after the user's request to finish the enemy section, then build Tiny Glade-style modular building with decor, ponds and gardens, then modular 2.5D buildings like This War of Mine and Into the Dead: Our Darkest Days, and "imagine what it would take for the engine to build similar games". Amendments 7bt (fights and creatures among themselves), 7bu (building drawn on the land) and 7bv (houses of storeys seen side-on) built what is marked **now**. The rest is in the order recommended. "Size" is a guess: S a few hours, M a day or two, L several days.

Nothing here recreates those games. The question asked was what the engine needs so that a person could author games of their kind from content: kinds, lines, styles, library designs and makers, with no code per game.

## 1. The enemy section (action, stealth, survival)

| What such games need | Draw3D now | Missing, in order | Size |
|---|---|---|---|
| Health, being hurt, falling | **now**: `health N [NAME]`; the walker's five, shown while anything can hurt; downed back to the last checkpoint, whole; fallen lie there, also in a saved game; what they carry taken from them where they lie (`stock` and the acts `loot_food`, `loot_scrap`, offered `while fallen`; 7by: `loot`, all of it at once); gear dropped as they fall (7by: a thing that says `dropped_by NAME`) | What the folk carry drawn on them; things made while the world runs | S |
| Melee that can be read and avoided | **now**: `attacks MOVE R DAMAGE S`, the blow wound up long enough to see; a dash, dodge or roll steps out of it; a moment of grace after a hit | Blocking and parrying (a move whose phase `guards`); a hit reaction by the side struck | M |
| Ranged enemies | **now**: `shoots R DAMAGE S SPEED`, a shot that flies and is stopped by cover, hurts whoever it meets except its own side | Shooters that keep their distance and take cover; the walker's own bow (the Gear plan's firing, awaiting approval) | M |
| Stealth kills | **now**: struck before it has noticed you (from behind, or before it knows you are there): it falls at once, hardly heard | A takedown move of its own (a held grab), carrying a body away | M |
| Groups | **now**: `side`, `calls R`; one that sees one of its side lying comes to look and calls | Flanking and surrounding (each hunter to a place of its own round the walker, as followers already keep apart); morale of a group | M |
| Factions fighting each other | **now**: `preys NAMES`, sides that never hunt each other: the dead and the bandits fight without the walker | Attitudes that change (a side made friendly by a quest fact); territory held by a side, shown | M |
| Animals among themselves | **now**: `preys`, `flees R NAMES`, `defends R`, prey caught; a pack calls; hurt badly, it runs; noticed by sound as well as sight | Hunger that drives the hunting (needs, below); herds that move together | M |
| Bosses and phases | **now** (7bx): `pattern F MOVES` (its blows in turn, a line a phase by its health; a roar crossing into the next; the fact `phase`), `tires S` (spent after), `weak behind N`, `weak tired N`; new moves lunge, slam, roar, pant; the warlord in the wilds; `enrages F` as before | Weak points as parts of its body; a harm of its own for each move; patterns chosen by distance or a fact | M |
| Status effects | none | Stunned, slowed, burning, poisoned: facts on a body that its motor and health read | M |
| Enemies drawn by animated characters | the zombie clips of Universal Animation Library 2 are here; folk are posed figures | A kind that says `character NAME` drawn by the walker's animated character code | L, the catalogue's step 5 |

## 2. Tiny Glade (building by drawing, the world dressing itself)

| What it needs | Draw3D now | Missing, in order | Size |
|---|---|---|---|
| Walls drawn freely, fitting the ground | **now**: Build > Draw on the land: a stone wall, fence or hedge along any line, pieces of no more than 3 m, a pier at each joint, stepping with the ground | A curve through the places (the line is straight between them now) | S |
| Towers where a wall turns | **now**: `towers`, a round tower with a cone roof at every sharp turn and at open ends | Battlements; towers that can be made taller by hand | S |
| Houses of any footprint | **now**: a house drawn round: facades with windows by bays, corner posts, a floor, a door in its first side, and a roof that fits (a house's roof over each oblong of an L, T or U; a roof to a point for a slanting outline) | Hipped roofs over any outline (a straight skeleton); dormers and chimneys; windows placed by a click | L |
| Height dragged | **now**: height and storeys in World outline | Dragged in the view (a handle on the wall's top) | M |
| Gates and doors where clicked | **now**: Open or close gates… and a click: a gate in the middle of that side | A gate anywhere along a side, not only its middle; an archway (a gate with no leaf, arched) | M |
| Ponds and gardens | **now**: a pond drawn round (kerb, bed, water, rocks, grass and ferns at its edge); a garden drawn round (an edging, soil, flowers and bushes in rows) | A pond dug into the land (the land lowered under it), reeds in the water, lily pads; vegetable rows of a crop (the Field maker does this for a crop already) | M |
| The world dressing itself | **now**: `dress` flowers, bushes, rocks, ferns, grass along a structure's foot | Ivy and moss up walls; lanterns on piers; paths worn where people walk | M |
| Live feedback while drawing | none: the structure is made when the line is ended (Enter) | Drawn as the pointer moves, before the click (the host makes a draft structure each change) | L |
| Bridges | none | A structure drawn across water as a bridge (a deck, rails, piers in the water) | M |

## 3. This War of Mine (a shelter seen in cross-section)

| What it needs | Draw3D now | Missing, in order | Size |
|---|---|---|---|
| A house cut open, floor over floor | **now**: a building of up to six storeys on cells, a facade per storey, floors over the first, stairs from floor to floor into the next cell along; seen side-on with its near walls simply gone (`cutaway`) | Ladders and holes in floors (a lighter way up); rooms not yet entered kept dark | M |
| Searching | **now**: a cupboard, toolbox and first-aid box that hold goods; E searches (`search_food`, …), heard 5 m off (7bw: an act's `heard R`); what is found by chance (7by: `perhaps GOOD N P`, drawn again each run, kept with a saved game; `search`, all of it) | A table of finds shared by many things | S |
| Rubble and locked doors | **now**: rubble that `breaks`; a locked door and a locked locker, opened by a lockpick used up (7bw: `unlock`, `open_unlocked`, `search_locker`); keys (7by: a thing's `key GOOD`, an act's `needs key`, kept and not used up; a door locked by a key, a strongbox) | A key as an item of gear, seen | S |
| Crafting at stations | **now**, as goods: a workbench makes a lockpick of two pieces of scrap (7bw: an act's goods lines done one after another) | What is made as an item (gear.txt: a shovel, a crowbar), not only goods | M |
| Needs: hunger, sleep, sickness | none | Survival needs as meters (the catalogue's step 3, awaiting approval) | M |
| Day and night as phases | partly: the clock, `when dark`, night raids by the dead | A day that is skipped to evening; a night of scavenging elsewhere (connected worlds exist) | M |
| Several survivors to switch between | none (character switching is the user's to say) | Who is walked as, chosen while playing | L, the user's decision |

## 4. Into the Dead: Our Darkest Days (side-scrolling through a city of the dead)

| What it needs | Draw3D now | Missing, in order | Size |
|---|---|---|---|
| Side-on exploration of buildings | **now**: side-on lanes, houses of storeys, cutaway | A street of such houses made by a maker (a side-on street of buildings of storeys) | M |
| The dead that hear and swarm | **now**: hearing, hunting, calls, the dead at night (`when dark`) | Hordes (many cheap bodies: a body without nav planning for far ones) | L |
| Barricades | **now** (7bw): `barricade N`, struck by the walker and by whoever hunts and finds it in the way; broken, it lies flat; `board_up` puts it up again with scrap | Boards put up where there were none; doors the dead batter | S |
| Hiding and fleeing | **now**: `hides` (crouched in it, unseen), dodging, stamina (7bw: the Player start's `stamina`, Breath shown); spent by a dodge (1 s) and a blow (0.5 s), and no dodge winded (7by) | Stamina of the folk | S |
| Travel between places | **now**: connected worlds and a hex overworld (7bq) | A map of places found | S |

## 5. What would advance the engine most, in order

Each is a step on what exists, not a new system beside it.

1. **Survival needs as meters** (already recommended, awaiting approval). It is what three of the four games above turn on, and it is mostly facts that tick and a HUD row.
2. **Things that can be struck and broken by the folk** (barricades, doors, crates): built as 7bw (`barricade N`), with locks, a lockpick made at a workbench, searching heard and stamina.
3. **Live drawing**: the structure made as the line is drawn, which is what makes Tiny Glade feel as it does. It is the largest of these, and wants the host to make a draft without an edit.
4. **Attack patterns and phases**: built as 7bx (`pattern`, `tires`, `weak`, the warlord). It was built in `fight.gd` rather than as work sequences: a pattern is only moves in turn, and a work sequence's steps (go, wait, do) would have carried nothing it uses.
5. **Enemies drawn by animated characters**: the catalogue's step 5.
6. Smaller: ladders, a gate anywhere along a side, a curve through a drawn line, searching that is heard, locked doors and keys, stamina, drops.

Of section 5, item 2 and four of item 6 (searching heard, locked doors, crafting a lockpick, stamina) are built (7bw), and item 4 (7bx). Items 1, 3 and 5 are each larger work, so they wait for the user's word. What is left of item 6 is small: ladders, a gate anywhere along a side, a curve through a drawn line. Keys, drops and finds by chance are built (7by).
