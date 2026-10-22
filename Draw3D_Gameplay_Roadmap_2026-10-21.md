# Draw3D — From Construction to Reusable Gameplay: Roadmap

2026-10-21. Written after pass 7df (construction stabilisation, multi-selection). It builds on `Draw3D_Core_Capabilities_Audit_2026-10-21.md` and was checked against the code on that date.

**How the evidence is graded:**

| Word | Meaning |
|---|---|
| **built** | The code exists. Nothing more is claimed. |
| **auto-checked** | A host unit test, a protocol test, a tour part or a demo with its own `expect` checks passes. |
| **seen** | A picture from a checked run was looked at. |
| **hand-tested** | You tried it yourself. |

No feature is called done on the strength of a button or a demo alone.

---

## 0. Summary

1. **Construction stabilisation (section 1) is built and auto-checked; parts are seen.**
   - Cut walls and fences keep their end posts.
   - Wear now shows at every level, piece by piece, on tall and low pieces alike.
   - Worlds saved before a generator changed open built again.
   - Several things can be chosen and moved, turned, lifted or deleted together.
   - None of it is hand-tested yet.
2. **The gameplay foundation already exists in pieces, and the pieces do not share a model.** There are:
   - two condition languages (`cond.gd` words, and the facts-based `when` of `state.gd`);
   - one trigger table (`on … do …`, 5 triggers × 5 actions);
   - a priority-and-interrupt scheduler for the folk (`work.gd` reactions);
   - acts with requirements (`acts.txt`);
   - quests, talks, stats, items, fights and perception, each with its own words.
   - The roadmap's first job is to make these one model rather than add a seventh.
3. **Recommended order:**
   1. G1, one condition and event model;
   2. G2, states and rules on any entity, through one entry point;
   3. G3, a small vertical slice across two entity types;
   4. then interactions (G4), AI (G5), items (G6), combat (G7) and quests (G8), each re-based on G1–G2.
   - X1, autosave, is small and independent, and runs beside G1.
   - The brief's order A–F is kept. A is split into G1/G2, and the slice comes before B, for the reason given in section 5.
4. **No new dependency is recommended now.**
   - LimboAI and Beehave were assessed (section 4). Both are good and MIT-licensed, but they assume behaviour lives as nodes in a scene tree.
   - Draw3D keeps behaviour as content text in the host's document, which is saved, undone and validated there. Their ideas are borrowed (status of a running step, a blackboard, a state machine), not their code.
5. **Seven decisions are yours (section 10).**

---

## 1. Construction stabilisation (pass 7df)

### 1.1 Defects, root causes and fixes

| Defect you saw | Root cause found in the code | Fix | Evidence |
|---|---|---|---|
| Most wear options showed no change; the one that did looked like regularly spaced missing sections | A piece's mesh is built from its brief without its place (`core/src/build.rs` `PLACE`), so `r(brief, k)` gave every piece of a wall or fence the same numbers. Every piece was worn identically, the same pales gone beside every post. Counts were also rounded per piece (`round(wear·w·h/3.5)` is 0 for a 2.5 m piece at wear 0.3). On a dry-stone wall, the cracks and hollows were drawn *inside* the stones, which stand 0.5–3 cm proud. | `worn_apart` gives each worn piece one of seven seeds by where it stands (`land/build.rs`). Counts come by chance where under one (`some`). Dry-stone walls wear their own way: top stones gone, a stone fallen out of a face leaving a dark hollow, moss low down, stones lying at the foot. Fences grey, and pales go, break short with a ragged top, or lean; a rail comes away at one end and falls. Hedges thin and brown in places. Posts grey and lose caps. | unit `what_is_worn_shows_at_every_wear_and_stays_its_height`; protocol `p89` (every wear level gives a different set of meshes, worn pieces differ); demo `fences` pictures **seen** |
| Wear wrongly scaled or invisible on low fences | Rails were placed at fixed heights (0.28 m and 0.4 m and up), so a 0.5 m fence had a rail at its top. Crack steps were fixed at 12–26 cm and broke off at once on a low wall. Hollows were 30×20 cm and were skipped wherever they did not fit. | Rails, crack steps, stains and hollows are scaled by the piece's height. | unit test bounds (nothing worn rises above the piece); `p89` low fence and low wall; demo `fences` beat 5 **seen** |
| Cut walls and fences lose their end posts, or the posts become invisible | Not reproduced in three scripted cuts: straight, bent and cut twice, with undo and redo. The host makes a post at every cut end. One real cause was found: a cut could leave a 0.3 m gap, while wall piers are 0.62 m across and hedge piers 0.88 m. The two end piers then overlapped and one hid the other. | `cut_line` leaves at least a pier's width plus a hand (wall 0.9, fence 0.4, hedge 1.2, power line 0.6). It keeps the cut 0.25 m from the places either side, so each half keeps its end there, and it refuses a stretch too short to cut. | `p89` (posts at all four ends, after undo, redo, write and open); demo `fences` and `cuts` with `expect` checks |
| *Found while repairing tours:* the showcase village opened with every structure empty | Structures went to code version 3 on 2026-10-17 (`afb0fe3`). A world file holds no made things, so a maker written by older code opened "stale, no stored output, upgrade required" and showed nothing. Nothing ever upgraded it. **This hits any world saved before that date: yours, the showcases, and a built game.** | The host's `open` upgrades every stale live maker as part of opening, carrying hand edits as an upgrade does. The editor says which makers were upgraded. | protocol `p90` (a code change between write and open), `p91` (the real showcase village has its doors and gate) |

**Still needs your eye:**
- whether the new wear looks convincing;
- whether the lost posts you saw happen again in your own way of cutting. If they do, tell me the steps; the scripted ones do not lose them.

### 1.2 Regression coverage added

| Layer | Test | What it asserts |
|---|---|---|
| host unit | `what_is_worn_shows_at_every_wear_and_stays_its_height` | Fence, dry-stone and hedge, 0.5 m and 1.2 m, seven seeds, wear 0.3, 0.6 and 0.9: each differs from new, and stays within its height. Stones lie at the foot of at least half of much-worn walls. |
| protocol | `p89` | A fence and a wall (worn, with a gate) cut as the editor cuts: a post or pier at all four ends, the other half as worn. Undo/redo and write/open keep it. Fence, wall, low fence, low wall and hedge: new pieces are one mesh, worn pieces are not, and each wear level is a new set. |
| protocol | `p90`, `p91` | A world written before a code change opens built; the village has its doors. |
| protocol | `p92` | House, joined wall and a placed thing moved by 5 m: each moves 5 m, the joined wall once. One undo step; a quarter turn about a point; a lift where possible, the rest named; write/open keeps it; an empty group is refused. |
| editor demo with checks | `-Demo fences -Check`, `-Demo several -Check`, `-Demo cuts -Check` | Demos now print `demo: problems [...]` from `expect(...)` calls, so a checked demo is a pass/fail check of the editor as a person drives it. |

**Not provable by these tests:**
- whether wear *looks* weathered rather than broken;
- whether the frames around several chosen things read clearly on a busy scene;
- feel: drag latency, snapping.

These are in the manual plan (section 8).

### 1.3 Tours reviewed

| Tour part | State before | Now | Verdict |
|---|---|---|---|
| `village` | Failed since 2026-10-17 | The root cause, empty structures, is fixed: doors stop the walker again. It still fails because no door offers "open" to the walker, and a later beat cannot find Makers' "Walls inside…" button (`build_walls`). | **Repair inside G4.** The door that answers an interaction is G4's first validation example. Mending it there means mending it on the shared model, not twice. |
| `items` | Failed since 2026-10-16 | Script error at `tour.gd:2013`: the lantern was never moved into the hand (`th.moved` has no entry), so taking it out failed. | **Repair inside G6** (items), for the same reason. |
| `library` | Failed | "Shelves" and "found by name" fail. The Library window changed in 7bl (folds), and the tour reads the old layout. | **Retire or rewrite.** It checks a window layout, not a capability. Replace it with a protocol test of `library` listing plus a demo with `expect`. |
| `ui` | Fails on the Studio window only | Unchanged | The Studio is parked by you. **Keep, with the Studio beat skipped** until the Studio resumes. |
| `style` | Fails on time passing on this machine | Unchanged | Machine-timing dependent. **Rewrite** its clock beat against the clock, not wall time. |
| `kinds`, `flock` | Intermittent | Unchanged | Keep. Record each failure exactly (risk-based testing rule). |
| drawn construction | No tour part | The demos `fences`, `several` and `cuts` now carry their own checks | Use these as the drawn-construction check, rather than a fourth copy in tour.gd. |

### 1.4 Multi-selection (built and auto-checked; not hand-tested)

**How you use it:**

| Gesture | Effect |
|---|---|
| Shift+click | Adds what is under the pointer, or takes it out. The one chosen before becomes the first. |
| Click without Shift | Back to choosing one, exactly as before. |
| Esc or Deselect | Lets the several go. |

**What can be one of the several:**
- a drawn house, wall, fence, hedge, path or pond, whole;
- a thing put down by itself;
- one thing of a maker that makes many, such as a tree of a wood.

The ground, a tile and a structure on a layout's cells say why they cannot be.

**What is done to them together:**

| Action | How | Note |
|---|---|---|
| Move | right-drag any one of them; the frames follow before letting go | |
| Turn | `[` and `]`, about their middle | |
| Lift | Page Up and Page Down | Only where a maker has a height of its own (`raise`). The rest are named. |
| Delete | Delete | |

Each is **one host edit and one undo step**, through the new host op `move_group`. A wall joined to a house, both chosen, moves once, because `follow_joins` leaves an end the edit itself put. A thing whose maker is also chosen goes with the maker.

**Shared and individual operations:**
- Shared: where they are and how they are turned.
- Individual: each one's look, sort and line places. The status bar says "click it alone".
- No look or setting is written across several yet; that is G2's "apply to compatible" (section 6), done through the entry point.

**Where it lives:**
- `editor/godot/group.gd`;
- `main.gd` (Shift+click, right-drag, keys);
- `desk.gd` (the status line);
- the host's `move_group` in `lib.rs`.

---

## 2. Code-grounded gap analysis (gameplay)

Each row was read in the file named. "Verified gap" means the code was searched and the capability is not there.

### A. Events, conditions, states, behaviour rules

| Exists | Where | Grade |
|---|---|---|
| Facts: the one world state, on a thing or the world, kept by the host and saved | `state.gd`; `engine_core::state` | auto-checked (protocol) |
| Conditions in words: `hour`, `has`, `room`, `there`, `reach`, `fact OP N for S`, `weather`, `near`, `quest`, `walker`, `not` | `cond.gd` (used by work, quests, talks, reactions, stats, goods, folk) | auto-checked by tours |
| A second condition form on things: `when FACT [OP N] [for S]` (stands open, lights) and `while WORD CONDITION` (7cf) | `state.gd` `cond`; host `kind.rs` | auto-checked |
| Triggers to actions: `on TRIGGER [WHICH] do ACTION`, triggers `senses hits spots act answer`, actions `put give say sound move` | `wires.gd`; host `kind.rs` `ON_HAPPENS`/`ON_DOES` | auto-checked (`-Demo wired`) |
| Priority and interruption, for the folk only: reactions, "the most pressing that holds begins; work resumes from its step", `again`, `while` | `work.gd`, `reactions.txt` | tour `work` passes |
| Rows in the Inspector for `on` lines | `inspect.gd` `_fill_wires` | built |

**Verified gaps:**
1. **Two condition languages.** `cond.gd` and `state.gd` `when` differ in words and in who is asked. Quests and talks use the first, things the second.
2. **No and/or.** There is only `not`, and `weather A B` means any of them.
3. **No explicit states.** Searched: no `state`, `page` or `mode` word for a thing or a creature. What looks like state is a fact, such as `open`, or hard-coded phases in `watch.gd` (seen, hunting, lost, searching).
4. **Priority and interruption exist only for the folk's reactions**, not for things or creatures.
5. **Events are five triggers.** There are no events for: a fact changed; entering or leaving an area; an hour or a timer; a thing used by an item; health reaching zero; a quest step.
6. **Diagnostics:** the Playtest window counts `on` lines fired. Nothing says why a rule did *not* fire, though `cond.gd` already returns a human reason for a failed condition. That is the seed of it.

### B. Interactions and actions

| Exists | Where |
|---|---|
| Acts as content (come, do, ride, leave, hold); requirements by goods (`makes uses gains hands`); `for walker/folk`; why not, said | `acts.gd`, `acts.txt`, `actbook.gd` |
| Offers on things (`offers ACT PLACE`); doors and gates `opens`; uses presets (bed, seat, food, drink, medicine, container) | host `shapes.rs` `traits`, `uses.txt`, Inspector "Make it usable" |
| Mechanisms (joins, blows, carries, hits) and signals | `physbook.gd`, `signalview.gd` |

**Verified gaps:**
- An action has no declared duration, interruption, condition list (beyond goods), animation, sound or effect bundle as one definition. Those are spread across acts, uses, wires actions and stats effects.
- `lock`/`unlock` exists only as a fact plus `when` (7bw).
- There are no `inspect`, `equip` (as an act), `cook`, `gather`, `chop`, `repair`, `trade`, `feed`, `tame` or `mount` definitions. `ride` covers boarding.
- The door is not offered in the village tour (see 1.3).

### C. Characters, navigation, perception, AI

| Exists | Where |
|---|---|
| Navigation: Godot's NavigationServer, baked a cell at a time on a thread, from what is solid; a shut gate is a wall | `nav.gd` |
| Schedules (`goes HOUR at PLACE / do WORK`), sequences, reactions | `folk.gd`, `work.gd` |
| Perception: `sees R DEG H`, `hears R`, `hunts`, alarm, lost/search; creatures | `watch.gd` |
| Movement: walk, run, vault, wall-run, climb walls and ceilings (7cc), wade | `motor`, `moves.txt`, `abilities.txt` |

**Verified gaps:**
- Perception reports only *the walker*. `watch.gd` sees and hears the walker, not other entities or stimuli.
- Patrol, follow, flee, investigate and return-home are not reusable behaviours. Some are bespoke inside `watch.gd` or `folk.gd` ("follows whom", 7cd).
- No group responses beyond an alarm.
- Swimming as navigation is not supported: nav treats deep water as no way.
- Climbing exists for the walker's motor, not for navigation.

**Known open faults (PROGRESS Known gaps):**
- stairs while carrying;
- crowds;
- held poses;
- physics pushes.

### D. Items, inventory, equipment, crafting

| Exists | Where |
|---|---|
| Items with their own ids, found, kept, held, equipped (`gear ITEM`); goods counted; HUD and inventory (I) | `items.gd`, `goods.gd`, `gameui.gd` |
| Containers by use preset; drops and chance finds (7by); keys (7by) | `uses.txt`, `fight.gd`, `finds` |

**Verified gaps:**
- No recipe or crafting station model; searched for `recipe`/`craft`. Acts' `makes/uses` at a thing is the nearest: the bread oven.
- Items and goods are two models (one-of-a-kind and counted). Stacks of a kept item do not exist.
- No transfer between containers other than acts' `hands`.
- The items tour fails (1.3).

### E. Combat, damage, abilities, effects

| Exists | Where |
|---|---|
| `health N`, `attacks MOVE R DAMAGE S`, fallen state kept; boss phases (7bx); stamina, barricades (7bw); strike strength as a fact (7an) | `fight.gd`, `strike.gd` |
| Stats and statuses with durations and stacking; effects on eat/drink/act/rest | `stats.gd`, `stats.txt` |
| Abilities catalogue (give any ability to any thing) | `abilities.gd`, `abilities.txt` |

**Verified gaps:**
- Health is blows to fall, not points with armour or resistance.
- Statuses apply to the walker only; `stats.gd` is "how the walker is".
- Abilities have no costs or cooldowns as one definition.
- Only the walker has stats.

### F. Objectives, quests, dialogue, progression

| Exists | Where |
|---|---|
| Quests as steps in `quests.txt`, state in the fact `quest.NAME`; talks with answers, `if`, `takes`, `gives`, `pays`; journal; markers | `quests.gd`, `talk.gd`, `journal.gd` |

**Verified gaps:**
- No branching outcomes beyond done/failed.
- No optional objectives, no reputation, no timed encounters.
- They are edited as text only, with no rows (audit foundation 4).
- Their conditions are `cond.gd`'s. This is good: they move to the shared model with it.

---

## 3. Architecture

### 3.1 What is kept

The runtime decisions of 2026-10-07 stand:
- the Rust host owns the document, the facts and the saves;
- Godot runs per frame.

**Behaviour stays content:**
- lines of a design or of one thing, read by the host's `kind.rs` (which validates the words);
- shown and edited as rows;
- saved, undone and copied like every other parameter.

Nothing here adds a scripting runtime. The Luau prototype (`prototypes/luau-boundary`) remains the code route, and choosing it is decision D4.

### 3.2 The shared model (G1–G2)

Five parts, each the generalisation of something that already exists.

**1. Facts** stay the world state.
- An entity's **state** is the fact `state` on it.
- Its persistence, undo of authored rules, and save/load come for free from the existing fact store.

**2. One condition language,** written as `cond.gd` writes it, with the `when`/`fact` forms folded in:
- `all:` / `any:` groups and `not`;
- asked of *someone*: `{about, at, opens}`, extended to any entity, not only the folk and the walker;
- it answers *why not* in words. That is the diagnostics.

**3. Events,** a closed, declared list:
- today's five, plus: `fact` (a fact changed), `enters`/`leaves` (an area or a range of an entity), `hour`, `timer`, `used ITEM`, `falls` (health at 0), `quest STEP`;
- each event says who it is *from* and *about*, so one rule can react to another entity's event (the keeper hears the bell);
- events are raised where they already happen (acts.gd, watch.gd, fight.gd, quests.gd), in one place (`events.gd`).

**4. Rules,** in the form of RPG Maker's event pages (section 4):
- an entity has **states**, and each state is a page: conditions, a priority, on-enter actions, `on EVENT` rows, while-in actions, and an exit on a condition or an event;
- the highest-priority state whose conditions hold is current, as `work.gd` reactions already decide "the most pressing";
- a higher state interrupts a lower one; on leaving it, the lower resumes.
- The deer, the guard and the door are three entities with three pages each, not three systems.

**5. Capabilities.**
- The host already knows which words a design and a single thing may say (`{"op":"words"}`).
- This becomes a **capability manifest**: for each entity type, which events it can raise, which actions it can take, and which conditions can be asked of it. A door raises `act open`; a creature raises `spots` and `falls`.
- The entry point lists only what the selection is capable of. With several chosen, it shows the intersection, marks mixed values, and only writes the rows you change. No unrelated line is touched.

### 3.3 Ownership and boundaries

| Concern | Owner | Why |
|---|---|---|
| Rule text, validation, undo, copy to another thing | host (`kind.rs`, document) | already how `on`/`while`/`when` work |
| Facts, saves, the world clock | host (`engine_core::state`) | already |
| Evaluating rules each frame, raising events, running actions | Godot (`rules.gd`, new; `events.gd`, new) | per-frame and physics-bound; `wires.gd` and `work.gd` already run there |
| Navigation baking, physics | Godot | already |

**Performance budget:**
- rules are evaluated on change: an event, or a fact change heard through `state.changed`, plus a timer tick no faster than 5 Hz for range conditions, as `work.gd`'s reactions are now;
- target: 200 ruled entities under 0.5 ms a frame on this machine, measured by a `-Measure` part added in G2.

---

## 4. Targeted research (what changed a decision)

### 4.1 Behaviour tree or state-machine libraries for Godot

**Investigated:**
- **LimboAI**: C++ (module or GDExtension), MIT, behaviour trees plus hierarchical state machines, an editor and a visual debugger. Its Asset Library versions are labelled for Godot 4.4–4.5; v1.6.0 is listed for 4.4.
- **Beehave**: GDScript, MIT, behaviour trees with a blackboard, tested by CI. 2.9.2 is listed for Godot 4.5.

**For Draw3D:**
- Pro: mature tasks and debuggers.
- Con: both store behaviour as node or resource trees in Godot scenes. Draw3D's behaviour must live in the host document as per-thing content: copied with a design, undone, validated, and saved with the world. A second home for behaviour would split the source of truth.
- Con: LimboAI's binaries for Godot 4.7.2 (this machine's) are not confirmed.

**Recommendation: do not adopt now.** Borrow:
- RUNNING/SUCCESS/FAILURE for a step;
- the blackboard as facts;
- hierarchical states as states with priority.

Revisit if G5 shows that trees deeper than pages are needed.

**Licence and maintenance:** both are MIT (compatible with games you sell). Beehave's GDScript has no binary-compatibility risk if it is ever needed.

### 4.2 How creators attach behaviour without code

**Investigated:**
- **RPG Maker event pages:** at most one page active; a page is active when all its conditions hold; with several eligible, the higher page wins; *self switches* are an event's own state; triggers are action button, touch, autorun and parallel.
- **UEFN direct event binding:** devices expose *events* and *functions*, and a creator binds one device's event to another device's function in the device's options; Verse is the code counterpart, not a replacement.
- The **multi-object inspector** convention in Godot and Unity: only common properties are shown, and mixed values are marked.

**Recommendation:**
- States as pages, with priority and conditions: this is the RPG Maker model.
- `on EVENT [from WHO] do ACTION [to WHOM]`, with the *to* chosen by picking in the view: this is UEFN binding, and is how the keeper hears the bell.
- Multi-select shows the intersection and writes only what changed.

All three map onto what exists: facts, `work.gd` priority, `on … do …`, and Inspector rows.

### 4.3 Navigation

**Investigated:** Godot's navigation-mesh baking. It is ReCast-based, recommended on a thread for runtime rebakes, and parsed source geometry can be cached and baked again from it. `NavigationLink3D` exists for connections a mesh cannot express.

**For Draw3D:** `nav.gd` already bakes per cell on a thread from what is solid, so a shut gate is a wall.

**Recommendation:**
- G5 adds links for ladders and climbable walls, and wading or swimming costs, rather than a new pathfinder.
- Cache parsed geometry per cell if rebakes show in the budget.
- Doors stay "solid when shut", which is already right.

---

## 5. Dependency map

```
            X1 autosave ─────────────── (independent; beside G1)

  G0 construction (7df, done) ──► multi-select ──┐
                                                 ▼
  G1 events + one condition language ──► G2 states/rules + entry point ──► G3 slice
        │                                   │
        ├──► G8 quests/talks (conditions)   ├──► G4 interactions/actions ──► G6 items/crafting
        │                                   ├──► G5 perception/nav/AI behaviours ──► G7 combat
        └──► diagnostics (why not) ─────────┘
```

**Why:**
- G2 needs G1's single condition model, or each state would carry its own.
- G3 proves G1 and G2 before G4–G8 multiply on them.
- G4 precedes G6 because pickup, equip and use are actions.
- G5 precedes G7 because combat needs targets from perception.
- G8 depends only on G1, so it could move earlier if you want stories before AI.

**The one change from the brief's order:**
- Priority A is split into G1 and G2, and the slice G3 comes before B.
- The slice needs only what exists in B (acts, offers, a door), so B can then be designed knowing what the shared model needs from an action.

---

## 6. Milestones

Each is small enough for one pass, a demo with `expect` checks, and a hand test.

**Status (2026-10-21, 7dg).** The user said to continue the roadmap without answering section 10, so its recommended answers were taken: the order as written, states as pages, Luau parked, the three tours as in 4, autosave every 3 minutes, Shift+click. **G1: built, auto-checked** (`-Demo rules1`, host unit and protocol tests), not hand-tested. **X1: built, auto-checked** (p95, the `rules1` beat; the restart offer is for a hand test), not hand-tested. G2 is next. A milestone row is marked done only after the hand test.

### G1 — One condition language, declared events, why-not

| | |
|---|---|
| **Purpose / result** | Any condition anywhere (quests, talks, reactions, a thing's `when`/`while`, `on` lines) is written the same way, can say `all:`/`any:`, and says why it does not hold. Events are one declared list that any entity can raise or listen to. |
| **Exists** | `cond.gd` (words, why-not), `state.gd` `cond` (`when`), `kind.rs` `ON_HAPPENS`, `wires.gd` |
| **Gaps (verified)** | two languages; no and/or; five events; no "why it did not fire" |
| **Scope** | 1. `cond.gd` takes the `when`/`fact` forms and `all:`/`any:`/`not`; `state.gd` `cond` calls it (one evaluator). 2. `events.gd`: one place where events are raised, with `from` and `about`; `wires.gd` subscribes to it; new events `fact`, `enters`, `leaves`, `hour`, `timer`. 3. The host's `kind.rs` validates the new words and lists them in `{"op":"words"}` with each entity type's capabilities (the manifest). 4. The Playtest window shows, for a selected entity, each rule and why it did or did not fire. |
| **Non-goals** | states (G2), new actions (G4), any visual editor beyond the rows that exist |
| **Acceptance** | (a) every existing tour or demo that uses conditions passes unchanged (`work`, `talk`, `wired`, `held`); (b) `on hour 18 do put lit` lights a lamp at six; (c) `when all: fact open; any: hour 6 20, weather rain` evaluates as written; (d) a rule that did not fire is listed with the reason |
| **Tests** | host unit: validation of new words, the manifest; protocol: words listing per type; demo `rules1` with `expect`; tours `work`, `talk` re-run |
| **Persistence / undo / perf** | rules are text lines (saved and undone as now); event dispatch is O(listeners); timer ticks at ≤5 Hz |
| **Risks** | Folding `when` into `cond.gd` changes the timing of `for S`. Kept by moving `state.gd`'s since-when bookkeeping with it. |
| **Shown before done** | the demo, plus your hand test of one lamp and one talk |

### X1 — Autosave and recovery (beside G1)

| | |
|---|---|
| **Purpose** | A crash loses at most a few minutes. |
| **Exists** | backups on save only (`main.gd` `keep_before_writing`); run logs kept |
| **Scope** | Every N minutes (default 3) while unsaved: write to `backups/autosave-<name>.json` without touching the world's file. On start after an unclean exit, offer "Open the autosave from HH:MM?" One setting, in World. |
| **Non-goals** | autosave of saved games; cloud |
| **Acceptance** | kill the editor mid-edit; restart; the offer appears; the opened autosave has the edit |
| **Tests** | protocol `write` to the autosave path; a scripted kill-and-restart check (the run already detects an unclean exit through its logs) |
| **Risk** | writing a large world every 3 minutes on the main thread. Measure first: `write` time at showcase size. |

### G2 — States and rules on any entity, through one entry point

**Status (2026-10-21, 7dh).** Being built as the native behaviour and rules system (amendment 7dh; the research `Draw3D_Behaviour_Tree_Research_2026-10-21.md` stages B1–B4 and the first of B5, which are this milestone re-cut, not a replacement of it). Built and auto-checked: states with `when` and priority, the six building blocks, the entry point (Behaviours and rules…, from the Inspector and the Window menu), templates with parameters, several chosen, the door and the deer. Not yet: the construction bar's button. Not tried by hand.

**Status (2026-10-22, 7di).** The guard example is built as the behaviour `guard_ways` (patrol, hears, investigate, alarm, back to the round), on `watch.gd`'s events, with the general steps and event it lacked (amendment 7di). Auto-checked by `-Demo guard`; not tried by hand.

| | |
|---|---|
| **Purpose / result** | Select a door, a deer, a guard or a lever; open **Behaviour and rules**; give it states (pages) with conditions, priority, on-enter actions and `on` rows; apply the same to several compatible things at once. |
| **Exists** | facts; G1; `work.gd` priority and resume; Inspector rows; multi-select (7df) |
| **Gaps** | no states, no pages, no priority for things or creatures, no common entry point |
| **Scope** | 1. Words: `state NAME [priority P] [when CONDITION]`; under it, `enter ACTION`, `on EVENT do ACTION`, `leave when CONDITION`. The current state is the fact `state`. 2. `rules.gd` decides the current state on change, runs on-enter, and resumes a lower state when a higher one ends. 3. **The entry point:** one **Behaviour and rules…** button. It appears in the Inspector's HOW IT WORKS, which every click already opens (the 7bz rule), and in the beside strip of the construction bar for structures. Both open the same panel. It lists only the events, actions and conditions the selection's capability manifest allows. With several chosen, it shows their shared rules, marks mixed ones, and writes only the rows you change, as one undo step (`together`). 4. Templates: a state set can be saved by name (`behaviours.txt`, as `uses.txt` saves uses) and given to any capable entity. |
| **Non-goals** | the full contextual bar; graph views; new AI movement (G5) |
| **Acceptance** | the three examples run, authored only through the panel. **Door:** shut → (act open, and `not fact locked`) → open; a locked door says why. **Deer:** graze → (spots walker within 12) → flee → (out of sight 10 s) → return → graze. **Guard:** patrol → (hears) → investigate → (spots) → alarm, then back to patrol. Two doors selected and given the door rules in one step; each still has its own colour. |
| **Tests** | protocol: the words validated, a state written to two things by one `together`; demo `states` with `expect` for all three; tour `work` unchanged |
| **Persistence / undo / perf** | the current state is a fact, so it is saved; authoring is undone as text; the budget is measured with `-Measure` at 200 entities |
| **Risks** | Overlap with `watch.gd`'s built-in phases. Mitigated by making them *events* (`spots`, `lost`), not states, so watch.gd stays the sense and the rules decide what follows. |
| **Shown before done** | the demo, plus your hand test: build a gate, give it the door states from the panel, play |

### G3 — Vertical slice: "the bell at the gate"

| | |
|---|---|
| **Purpose** | Prove G1 and G2 across two different entity types, with persistence and authoring. |
| **The loop** | 1. The walker comes to a drawn wall's gate (a structure's gate thing) and rings the bell beside it (a placed thing with `offers ring`). 2. Event `act ring`, from the bell. 3. The gate's rule: `on act ring from bell when hour 6 20 → state open`. At night it stays shut and says "Come back by day". 4. The keeper (an NPC of the folk) has the rule `on act ring from bell → state answer`: walk to the gate, say a line, `give lantern` once (`not fact given`), then return to post (state ends; patrol resumes). 5. The walker passes through and has a lantern. 6. Save, quit, load: the gate is open, the lantern kept, the keeper does not give again. 7. In the editor: click the gate, then **Behaviour and rules…**, and its state pages are there. Click the keeper: the same panel, other capabilities. Select two gates with Shift and give both the rule. |
| **Proves** | one event model across a thing and a person; conditions with time; states on two entity types; an outcome for the player; persistence through facts; one entry point; group apply |
| **Does not prove** | combat, crafting, navigation at scale, many simultaneous rules (performance is measured in G2, not here), branching dialogue |
| **Acceptance** | demo `bell` with `expect` for each step, day and night; save/load inside the demo; your hand test with a controller |

### G4 — Interactions and actions (Priority B)

| | |
|---|---|
| **Purpose** | One action definition used by the player's key, by AI and by rules. |
| **Exists** | `acts.txt` (come, do, ride, hold; goods requirements); uses presets; `on` actions |
| **Scope** | `actions.txt` (the successor of `acts.txt`): `needs` (any G1 condition), `takes` (seconds), `interrupted by`, `move`, `sound`, `effect`, `then` (facts, goods, items, state). acts.txt lines are read as actions unchanged (no rewrite). First set: open, close, lock, unlock, pick up, drop, use, inspect, sit, rest, talk, operate. **The village tour's door is mended here.** |
| **Non-goals** | cook, craft (G6); mount beyond `ride` |
| **Acceptance** | the village tour passes; a locked chest says why not; an NPC's rule can do `do open at gate` through the same action |
| **Tests** | tour `village` (repaired); demo `actions`; protocol validation |

### G5 — Perception, navigation and reusable AI behaviours (Priority C)

| | |
|---|---|
| **Exists** | `watch.gd` (walker only), `nav.gd`, `folk.gd` days, `follows` |
| **Scope** | 1. Perception of any entity or stimulus: events `sees WHO`, `hears NOISE`, raised for listeners by tag (`creature`, `folk`, `walker`, `loud`). 2. Behaviour templates as G2 state sets with parameters: patrol (route or places), follow, flee, investigate, guard, return home, wander. 3. Navigation: `NavigationLink3D` for ladders, stairs and steps; wading cost; swimming only where a body says `swims`. 4. Group response: an alarm event heard by tag within R. |
| **Acceptance** | deer flee the walker *and* a wolf; three guards respond to one alarm; a cat follows the walker up stairs while carrying is mended or recorded |
| **Risk** | crowds (known gap): measure 30 bodies |

### G6 — Items, inventory, equipment, crafting (Priority D)

| | |
|---|---|
| **Scope** | One item definition (gear.txt + item facts) for pickup, inventory, equip and use, through G4 actions. Stacks for kept items of a kind. `recipes.txt` and a station capability (`crafts at`), using acts' `makes/uses`. Transfers between containers. **The items tour mended.** |
| **Acceptance** | gather wood → craft a torch at a bench → equip → it lights; saved and loaded |

### G7 — Combat, damage, abilities, effects (Priority E)

| | |
|---|---|
| **Scope** | Health as points, with armour and resistance by damage kind (blows stay as a simple preset); statuses on any entity with stats; abilities with cost, cooldown and conditions (G1) and effects. Combat states via G2 (attack, stagger, flee, fallen). |
| **Acceptance** | a sword and a bow against a guard and a wolf; poison stacks and ends; a boss phase is a state |

### G8 — Objectives, quests, dialogue, progression (Priority F)

| | |
|---|---|
| **Scope** | Quests and talks as rows over G1 conditions and G3 events; branching outcomes (several `done` endings); optional steps; rewards through actions; a reputation fact per person or group; timed encounters (timer events). |
| **Acceptance** | a quest with two endings chosen by a talk; reputation changes a greeting; a timed escort fails at its limit; all of it survives save/load |

---

## 7. First implementation batch (after this pass)

This is G1 plus X1. In order:

1. **`cond.gd` as the one evaluator.**
   - Add `all:` / `any:` groups.
   - Fold in `fact NAME OP N for S`, taking over `state.gd`'s since-when tracking.
   - Make `state.gd` `cond` call `cond.gd`.
   - Re-run tours `work`, `talk` and `held`, and demos `wired` and `held`.
2. **`events.gd`.**
   - Raise today's five triggers from their places, plus `fact`, `hour` and `timer`.
   - Have `wires.gd` subscribe.
3. **Host `kind.rs`.**
   - Validate `on fact|hour|timer|enters|leaves`, and `all:`/`any:`.
   - Extend `{"op":"words"}` with a capability manifest per entity type.
4. **Playtest why-not:** a selected entity's rules, each with fired or why not.
5. **Demo `rules1` with `expect` checks;** protocol tests for the new words.
6. **X1 autosave.**
   - Measure `write` time on `showcase-village.json` first.
   - Then build the timer and the restart offer.
7. **Records:** PROGRESS, amendment `## 7dg`, checklist, UI inventory, memory.

---

## 8. Manual testing plan

### After 7df (now)

1. **Draw a fence, then Cut and click a +.** Look at both cut ends close up.
   - Expected: a post at each end and a clear gap.
   - Repeat on a wall: a pier at each end, about a metre apart.
2. **Choose a fence, then How worn.** Step through the four levels.
   - Expected: each level visibly more broken, never a repeating pattern from post to post.
   - Repeat on a stone wall: stones gone from the top, dark hollows, moss, stones at its foot.
   - Repeat on a fence made low (Lower, three times): still visibly worn, with nothing sticking up above it.
3. **Open an old world** (any saved before 2026-10-17).
   - Expected: its houses and walls are there, and a message names the makers upgraded.
4. **Click a house, Shift+click a wall drawn from its side, Shift+click a barrel.**
   - Expected: three yellow frames and a status line.
   - Right-drag the wall: all three follow, and the frames show where they will land.
   - `]` turns them.
   - Page Up says the house and wall cannot be lifted.
   - Delete removes all three; Ctrl+Z brings all three back.
   - A plain click chooses one again.

### After G1

- Give a lamp `on hour 18 do put lit`, then play to dusk: it lights.
- Write a talk answer `if all: has 2 coin; not quest bread done`. It is offered only when both hold. The Playtest window says why it was not offered.

### After G2

- Make a gate a door with states from the panel. Play: shut, then opened with E; locked, it says why.
- Select two gates with Shift and give both the same states. Each keeps its colour.

### After G3

- Ring the bell by day: the gate opens, the keeper walks over, speaks and gives a lantern, then returns.
- Ring at night: the gate stays shut, with the reason.
- Save, quit, load: the gate is still open, the lantern is kept, and the keeper does not give again.

### After G4–G8

One scenario each, as in their acceptance rows, played with the controller (polish bar).

---

## 9. Progress-record plan

Each milestone keeps the existing records:

| Record | What it gets |
|---|---|
| **PROGRESS.md** | One section per pass. "Checked" lists tests by name, with counts. Demos are listed with their `expect` results. Every claim carries one of the four grades. |
| **Amendment** (`Draw3D_Roadmap_Amendment_2026-10-09.md`) | `## 7dX`: the decisions made and why, and what changed from this roadmap. |
| **This roadmap** | A milestone's row is marked done only after your hand test. Until then: built / auto-checked / seen. |
| **Known gaps** (in PROGRESS) | Every unexplained failure, exactly as seen: tour part, line, values. |
| **UI inventory, checklist, memory** | As every pass. |

---

## 10. Decisions for you

1. **Order.** G1 → G2 → G3, then G4…G8, with X1 beside G1. Or move G8 (quests) before G5?
2. **States as pages** (RPG Maker style: highest eligible page wins) as the one behaviour form. Or do you want behaviour trees as well, later (LimboAI, Beehave)?
3. **The code route:** keep Luau parked (rows only), or plan it after G3?
4. **The village, items and library tours:** repair inside G4 and G6, and retire `library` in favour of a protocol test and a demo. Is that agreed?
5. **Autosave:** every 3 minutes while unsaved, kept beside backups. Is that right for you?
6. **Multi-select gesture:** Shift+click. Ctrl+click is also common. Should both work?
7. **The slice:** "the bell at the gate". Is that the example you want, or one closer to a genre you plan to sell first?

---

**Sources consulted for section 4:**
- [LimboAI on the Godot Asset Library](https://godotengine.org/asset-library/asset/edit/21207)
- [LimboAI docs](https://limboai.readthedocs.io/en/v1.1.0/)
- [Beehave README](https://cdn.jsdelivr.net/gh/bitbrain/beehave@godot-4.x/README.md)
- [UEFN direct event binding](https://dev.epicgames.com/documentation/fortnite/direct-event-binding-in-unreal-editor-for-fortnite)
- [UEFN device interactions in Verse](https://dev.epicgames.com/documentation/en-us/uefn/coding-device-interactions-in-verse)
- [RPG Maker MZ help, event pages](https://rpgmakerofficial.com/product/MZ_help-en/01_09_03.html)
- [Event page limits (HimeWorks)](https://himeworks.com/2014/10/understanding-and-surpassing-event-page-limits/)
- [Godot navigation meshes](https://docs.godotengine.org/en/4.2/tutorials/navigation/navigation_using_navigationmeshes.html)
