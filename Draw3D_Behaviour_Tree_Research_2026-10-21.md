# Draw3D — Behaviour Trees and State Machines: Research and Recommendation

2026-10-21. A research pass, not an implementation. It answers the brief "Behaviour Tree Integration Research and Workflow Recommendation". It was checked against the code on this date (branch `m2-stable-terrain`, after 7dg). It deepens section 4.1 of `Draw3D_Gameplay_Roadmap_2026-10-21.md`, which assessed LimboAI and Beehave in a paragraph.

**How claims are marked:**

| Mark | Meaning |
|---|---|
| **verified (code)** | Read in this repository today, at the file named. |
| **verified (source)** | Read today in the project's own repository, release page or documentation, linked at the end. |
| **unverified** | Likely, but not tried on this machine. Each is listed in section 8 with how to check it. |
| **proposed** | A recommendation. Nothing of it is built. |

---

## 0. Summary

1. **The deciding fact:** Draw3D's editor is a running Godot *game*, not the Godot editor. `run.ps1` starts the project with `--path`, and every window you use (`desk.gd`, `win.gd`, the Inspector, the Studio) is Draw3D's own. **verified (code)**
   - LimboAI's tree editor, task palette, blackboard plan editor and visual debugger are a Godot *editor plugin* (`editor/limbo_ai_editor_plugin.cpp`). They exist only inside the Godot editor. **verified (source)**
   - The same is true of Beehave's debug view and Godot State Charts' editing: their trees are nodes arranged in the Godot editor's scene dock.
   - So **no candidate plugin gives you a visual authoring UI inside Draw3D.** Each gives a *runtime* (tasks that tick) plus tools you would have to open the Godot editor to use.
2. **What Draw3D already has is closer to the goal than any plugin.** Behaviour is already content text, validated by the host, saved, undone, copied with a design, and shown as Inspector rows. One condition language with "why not" exists (G1, built). A priority-and-interrupt scheduler exists for the folk (`work.gd`). What is missing is states, a tree tier above them, the one entry point, and the animation link.
3. **Recommendation: a Draw3D-owned behaviour model, executed by a small GDScript runner, authored in Draw3D's own panel.** **proposed**
   - Borrow LimboAI's *design*: task status (running, succeeded, failed), dynamic priority selectors that abort a lower running branch, sequences, parallels, cooldowns, time limits, subtrees with parameter mapping, a blackboard. These are well-proven and small.
   - Do not take its code now. Keep **LimboAI as a named fallback executor** (the hybrid in section 3.3), to be tried only if the runner fails a measured budget or tree depth in G5.
4. **The smallest proof** is a door with locked/unlocked states and a creature with perception and reactions, authored only through the panel, with "Edit animation" from the creature's flee step opening the Character Studio at that move and returning to the same row (section 6).
5. **Eight decisions and five risks** need you (section 8). The first: confirm that behaviour stays text in the host's document, with no second home in Godot resources.

---

## 1. Current starting point (code-grounded)

### 1.1 Engine and constraints

| Fact | Where | Mark |
|---|---|---|
| Godot **4.7.2** stable, editor and console binaries in the project root; project features `4.7` | `Godot_v4.7.2-stable_win64*.exe`, `editor/godot/project.godot` | verified (code) |
| No addons or GDExtensions in the project (`editor/godot/addons` does not exist) | | verified (code) |
| The editor runs as a game: `Godot_v4.7.2-stable_win64_console.exe --path editor\godot -- --port …` | `editor/run.ps1` line 137 | verified (code) |
| Rust host owns the document, facts, saves, undo and validation of words; Godot runs per frame | memory "Runtime decisions"; `editor/host/src/lib.rs`, `shapes/kind.rs` | verified (code) |
| Licence rule: nothing GPL may ship inside a sold game; MIT is fine | memory "Engine not distributed" | verified (code) |
| Downloads need your explicit yes; new dependencies need approval when they are materially larger than a pass | memory "Reuse first, approval for big bets" | — |

### 1.2 Behaviour, events, conditions, actions and state today

| Capability | How it is represented | Where | Grade |
|---|---|---|---|
| **Facts**: the one world state, on a thing or on the world | host fact store, saved with the game; `state.gd` mirrors it | `state.gd`, `engine_core::state` | auto-checked |
| **One condition language** with `all:` / `any:` / `not` and a reason in words when it does not hold | `hour`, `has`, `room`, `there`, `reach`, `fact OP N for S`, `weather`, `near`, `quest`, `walker` | `cond.gd` (264 lines) | auto-checked (G1, `-Demo rules1`) |
| **Events**, raised in one place | `senses hits spots act answer` raised where they happen; `fact hour timer enters leaves` watched for | `events.gd` (187 lines); host `ON_HAPPENS` | auto-checked |
| **Trigger → action** lines on a design or one thing | `on TRIGGER [WHICH] do put/give/say/sound/move …` | `wires.gd`; host `kind.rs` `on_said`, `ON_DOES` | auto-checked |
| **Capability manifest (first form)**: which events a thing can raise, by the words it says | `ON_NEEDS`, `can_happen` | host `kind.rs` lines 391–407 | auto-checked |
| **Held abilities**: `while WORD CONDITION` on 11 behaviour words (follows roams shies tends idles greets runs hunts preys attacks flees) | per-instance, asked twice a second | `folk.gd` `_held`; host `HELD` | auto-checked |
| **Sequences** (do, go, wait, say, check, put, clear) with `needs … else` | `sequences.txt` | `work.gd`, `seqbook.gd` | tour `work` |
| **Reactions** with `when`, `while`, `urgent N`, `again M`: the most pressing that holds begins, interrupts, and work resumes at its step | `reactions.txt` | `work.gd`, `reactbook.gd` | tour `work` |
| **Schedules**: `goes HOUR at PLACE / do WORK` | per character | `folk.gd` | tours |
| **Acts** with requirements and "why not" | `acts.txt` | `acts.gd` | tours |
| **Perception**: sight cone, hearing, alarm; hard-coded phases (noticed, hunting, lost, searching) | `sees`, `hears`, `hunts`, `spots` words | `watch.gd` (1139 lines) | tours; **walker only** |
| **Fights**: health as blows, attacks, boss phases | words on a design | `fight.gd` | demos |
| **Quests, talks** | `quests.txt`, `talks.txt` over `cond.gd` | `quests.gd`, `talk.gd` | tours |
| **Explicit states, pages, behaviour trees** | — | searched: no `state NAME` word, no `rules.gd`, no `behaviours.txt` | **verified gap** |

**What this means:** Draw3D already has the *leaves* of a behaviour system (conditions, events, actions, moves, sequences) and one *composite* (the reactions scheduler: a priority selector with resume). It lacks a general way to compose them on any entity.

### 1.3 Entities, animation and persistence

| Topic | How it works | Mark |
|---|---|---|
| **Entities** | A *design* (kind) is words in the host's document. One placed *thing* may add or override words. A behaviour line is just another word line, so copying a design copies its behaviour, and undo covers it. | verified (code) |
| **Shared texts** | `moves.txt`, `poses.txt`, `sequences.txt`, `reactions.txt`, `uses.txt`, `abilities.txt`, `gear.txt` are project-wide content, read again when saved. The Studio keeps its edits in project files (7bk). | verified (code) |
| **Library** | `editor/library/*.txt` holds designs; `library/mine/` holds yours. There is no import of a behaviour package. `library.gd` has no import function. | verified (code) |
| **Animation model** | Movement first, animation second. A *move* (`moves.txt`) has *phases* with physics (`time`, `covers`, `launch`, `strikes`, `lets`, `yields`…). What a phase *looks like* is a clip named in `poses.txt` (`move/phase` → clip). Clips are read from glTF or FBX and retargeted by `clips.gd`. Playback goes through an `AnimationTree` with a blend tree built in code (`animated.gd` lines 931–984). | verified (code) |
| **Animation editing entry point** | `studio.gd` `edit_move(name, phase, ask)` opens a move at a phase in the Studio's Edit tab and asks about unsaved changes first. It is called today only from the Studio's own tests (`studioplay.gd`). | verified (code) |
| **Animation from behaviour today** | A reaction step `move MOVE` begins a move and waits for it. An `on … do move` line begins one. Neither links back to the Studio. | verified (code) |
| **Assistant** | Drafts changes as sentences, checks each before it is made, applies all as one undo step; the model is your choice and the editor works without it | `assistant.gd`, `assisttools.gd` | verified (code) |
| **Code route** | A Luau spike behind the host's generator trait, with counted interrupts (`BUDGET`) and opaque keys | `prototypes/luau-boundary` | verified (code); parked by decision D4 |

### 1.4 What works, what is incomplete, what must be added

- **Works:** conditions with reasons; events; `on` lines; reactions with priority and resume; sequences; moves with phases; the Studio's move editor; host validation and undo of every word.
- **Incomplete:** perception sees only the walker; reactions exist only for the folk; `watch.gd`'s phases are hard-coded; the manifest covers events but not actions or conditions; there is no "why it did not fire" for reactions.
- **Must be added:** states on any entity; a small tree tier; one entry point "Behaviour and rules…"; saved, named behaviour sets with parameters; apply to several; a running view; the link from a step to its animation.

---

## 2. LimboAI, examined

Version examined: **v1.8.1, released 2026-08-20.** Repository last pushed 2026-10-04, 3,066 stars, 62 open issues, MIT. **verified (source)**

### 2.1 What it is

| Area | Finding | Mark |
|---|---|---|
| **Form** | C++. Ships two ways: as an engine **module** (custom Godot builds and matching export templates from each release) or as a **GDExtension** (a `.dll` in `addons/limboai/bin/`). Projects can move between the two. | verified (source) |
| **Godot versions** | 1.8.x: GDExtension "Godot 4.6 or higher"; module "Godot 4.7". v1.8.1 updated its module builds to **4.7.2-stable**, this machine's version. The GDExtension zip is built against 4.6; its `.gdextension` file says `compatibility_minimum = "4.2"` with no maximum. | verified (source); **loading the 4.6 GDExtension in 4.7.2 is unverified** |
| **GDExtension limits** | No documentation tooltips. The `BBParam` property editor is unavailable. Older notes list export only to Linux, Windows, macOS and Web. | verified (source); export platforms for 1.8 unverified |
| **Behaviour trees** | A `BehaviorTree` resource (root task plus a `BlackboardPlan`) run by a `BTPlayer` node. Trees can be built in code: `BTTask.add_child`, `BehaviorTree.set_root_task`, `clone`. | verified (source) |
| **Status** | `FRESH`, `RUNNING`, `SUCCESS`, `FAILURE`; `abort()` exits a running task and resets it. | verified (source) |
| **Built-in composites** | sequence, selector, **dynamic** sequence and selector (re-check higher children every tick and abort a lower running one), parallel, random and probability selectors | verified (source, `bt/tasks/composites`) |
| **Built-in decorators** | always fail/succeed, cooldown, delay, for-each, invert, new scope, probability, repeat, repeat until failure/success, run limit, **subtree**, time limit | verified (source) |
| **Blackboard tasks** | check trigger, check var, set var | verified (source) |
| **Utility** | call method, console print, **evaluate expression**, fail, wait, random wait, wait ticks | verified (source) |
| **Scene** | **play / await / pause / stop animation** (on an `AnimationPlayer`), check / set agent property | verified (source) |
| **Animation task** | `BTPlayAnimation`: player, name, `await_completion` (seconds), `blend`, `speed`, `from_end`. Returns RUNNING while playing, SUCCESS when done, FAILURE if it cannot play. It targets an **`AnimationPlayer`**, not an `AnimationTree`. | verified (source) |
| **Blackboard** | Plans with typed variables; values overridden per `BTPlayer`; scopes to share data between agents; `BBParam` (a literal or a bound variable); subtree variable mapping (an export bug fixed in 1.5.1). | verified (source) |
| **State machines** | `LimboHSM` and `LimboState` nodes, nested; event-based transitions; `ANYSTATE`; `BTState` runs a tree as a state. The README says: **"State machine setup and initialization require code; there is no GUI editor."** | verified (source) |
| **Debugging** | A visual debugger for any tree in a running scene, with blackboard inspection since 1.7 — inside the Godot editor. `BehaviorTreeView` is a node to draw a tree in-game for custom tools. Performance monitors. | verified (source) |
| **Custom tasks** | A script extending `BTAction`, `BTCondition`, `BTDecorator` or `BTComposite`, with `_tick` (and optional `_setup`, `_enter`, `_exit`, `_generate_name`, `_get_configuration_warnings`). Found in `res://ai/tasks`, one category per folder. GDScript or C#. | verified (source) |
| **Tests and demo** | Unit tests for tasks and HSM; a demo project and tutorial. | verified (source) |

### 2.2 Against Draw3D's needs

| Need | LimboAI | Comment |
|---|---|---|
| Visual authoring **inside Draw3D** | **No.** The editor is `LimboAIEditorPlugin`, an `EditorPlugin`. A running project has no editor plugins. `BehaviorTreeView` only *draws* a tree. | The decisive mismatch. Draw3D would have to build its own authoring UI either way. |
| Behaviour as host-validated content, undone and copied with a design | **No.** Its home is `.tres` resources and `BTPlayer` nodes. | Usable only if Draw3D *compiles* its own text into LimboAI trees at play time (section 3.3). |
| Selecting existing actions and conditions without code | Only Godot-editor users choose from the palette. Every Draw3D action (do an act, go to a place, say, put a fact, begin a move) would need a custom `BTAction` script anyway. | The no-code layer is Draw3D's to build regardless. |
| Interruptions and priorities | **Strong.** Dynamic selectors, abort, time limits, cooldowns. | Worth copying as a design. |
| State machines without code | **No.** Transitions are added in code. | Draw3D's pages (G2) are already the no-code form. |
| Animation | Plays an `AnimationPlayer`. Draw3D plays moves through its motor and an `AnimationTree`. | A Draw3D `BTAction` "begin move" would be needed; the built-in animation tasks would go unused. |
| Persistence | A tree's running state is not saved by LimboAI; the blackboard is per instance. | Draw3D's facts already persist; running position would be re-derived on load, as `work.gd` does now. |
| Shipping | A native `.dll` (or a custom engine and templates) in every exported game; the export path ("Export the game", 7cb) must carry it. | MIT: licence is fine. Maintenance: one more binary to keep in step with each Godot upgrade. |
| Performance | C++ ticking, monitored. | Better than GDScript per node. Whether Draw3D needs it is unmeasured: the roadmap budget is 200 ruled entities under 0.5 ms a frame. |
| Maintenance | Active: four releases in 2026, prompt Godot support (4.7 module in June). One main maintainer. | Low-to-moderate risk. |

**Verdict on LimboAI:** an excellent behaviour-tree *runtime* and a fine *Godot-editor* tool. For Draw3D it would supply about the hardest 20% of the executor (composites, abort, status) and none of the authoring, the persistence, the action library or the animation link. It would also put behaviour in a second home unless Draw3D compiled into it.

---

## 3. Alternatives, compared

### 3.1 Candidates considered

Only the credible ones are kept. Each was checked on its repository today. **verified (source)**

| Candidate | Form | State today |
|---|---|---|
| **LimboAI** | C++ BT + HSM, Godot-editor plugin | v1.8.1, 2026-08-20; MIT |
| **Beehave** | GDScript BT, trees as nodes in a scene, debug view | v2.9.3, 2026-08-18; pushed 2026-09-20; MIT; README lists 2.10+ for Godot 4.5+ (4.6/4.7 not named) |
| **Godot State Charts** (derkork) | GDScript state charts as nodes: compound, parallel, history states; guards as expressions; delayed transitions; debugger | v0.22.5, 2026-06-26; MIT |
| **Behaviour Toolkit** (ThePat02) | GDScript BT + FSM as nodes | v2.0.4, 2025-09; pushed 2026-03; MIT. Smaller and slower-moving; not carried further. |
| **Godot native** | `AnimationTree` state machines (animation only); `Tree` and `GraphEdit` controls for building an editor in a running game | in 4.7.2 |
| **Draw3D-owned** | Pages (G2) plus a small tree tier, in host-validated text, run by `rules.gd` | proposed; most leaves exist |

**Ruled out:**
- `AnimationTree` state machines as the behaviour model. They switch animation, not decisions, and Draw3D's moves already own timing.
- Visual-scripting graphs such as Godot's removed VisualScript. They are code in another dress; the brief rules that out.

### 3.2 Comparison

| Criterion | LimboAI | Beehave | Godot State Charts | **Draw3D-owned (recommended)** | Hybrid: Draw3D authoring + LimboAI runtime |
|---|---|---|---|---|---|
| Authoring UI inside Draw3D | none (Godot editor only) | none (scene dock) | none (scene dock) | **built for it** (rows, then a tree view with `Tree`) | Draw3D's |
| Where behaviour lives | `.tres` / nodes | nodes in a scene | nodes in a scene | **host document**, like every word | host document, compiled to `BehaviorTree` at play |
| Undo, copy with a design, save with the world | no | no | no | **yes, already** | yes (source); compiled copy is throwaway |
| No-code selection of Draw3D's actions | needs a `BTAction` per action | needs a node script per action | needs scripts reacting to states | **the actions already exist as words** | needs a `BTAction` per action |
| Priorities and interruption | dynamic selector, abort | selector with interrupts | parallel and compound states | reactions already do it; generalise | LimboAI's |
| State machines | code only | no | **strong** | pages (G2) | pages, compiled to `BTState` or a selector |
| Reusable subtrees with parameters | `BTSubtree` + mapping | subtrees by scene | instanced scenes | `use BEHAVIOUR with …` (section 4) | compiled to `BTSubtree` |
| Debugging for *you* | Godot editor only; `BehaviorTreeView` for custom | Godot editor | Godot editor | **Playtest window, "why not" in words** | Draw3D's, fed from LimboAI's status |
| Animation | `AnimationPlayer` tasks | none | none | **moves and the Studio** | Draw3D action |
| Runtime speed | C++ | GDScript | GDScript | GDScript, event-driven at ≤5 Hz | C++ |
| Ships in a game | native `.dll` or custom engine | scripts | scripts | scripts | native `.dll` |
| Godot 4.7.2 | module verified for 4.7.2; GDExtension built for 4.6 (unverified here) | not named for 4.6/4.7 | not stated | native | as LimboAI |
| Integration cost | high (actions, UI, persistence, animation all still Draw3D's) | high, same | high, same | **moderate** (most leaves exist) | moderate + dependency + compiler |
| Maintenance burden | follow its releases per Godot version | low | low | **ours, and small** (a runner of a few hundred lines) | ours + theirs |

### 3.3 The hybrid, honestly

Draw3D keeps the words, the panel, the validation and the facts. At play, `rules.gd` would build a LimboAI `BehaviorTree` from each entity's lines (`set_root_task`, `add_child`) and give it to a `BTPlayer`. Each Draw3D action and condition would be one `BTAction` or `BTCondition` script calling the existing code.

| For | Against |
|---|---|
| Proven composites, abort and status semantics | A native binary in every built game, and one more thing to bring forward at each Godot upgrade |
| C++ speed if hundreds of agents are needed | Running-state debugging would come from LimboAI's objects, translated back into Draw3D words |
| | Two ways to say "the most pressing begins": LimboAI's and `work.gd`'s, until the folk move over |
| | The composites Draw3D needs number about six. Each is 30–80 lines of GDScript. |

**When to take it:** only if the G2/G5 measurement (`-Measure` with 200 entities) fails its budget after making the runner event-driven, or if trees grow deeper than the runner handles cleanly. Then the hybrid is a change of executor, not of model: the words, panel and saves stay.

### 3.4 Recommendation

**Build the Draw3D-owned model, borrowing LimboAI's semantics, and keep the hybrid as a measured fallback.** **proposed**

**Why, in order of weight:**
1. **Authoring is the goal, and no plugin provides it inside Draw3D.** All three plugins put their editors in the Godot editor.
2. **The source of truth is already settled.** Behaviour is text in the host document: validated, undone, copied, saved. A plugin would add a second home or need a compiler.
3. **Most of the leaves exist.** Conditions with reasons, events, actions, moves, sequences and a priority scheduler are built. What is missing is composition, and composition is the small part.
4. **Debugging must speak your words.** "The gate did not open: it is night (hour 6 20 does not hold)" comes from `cond.gd`. No plugin's debugger can say that inside Draw3D.
5. **No binary dependency** for a capability that can be measured first.

**Why LimboAI could still turn out best:** if behaviour needs to scale to hundreds of agents with deep trees, or if you later decide behaviour may be authored in the Godot editor by a developer and only *parameterised* in Draw3D. Neither is true today.

---

## 4. Proposed architecture

### 4.1 Who owns what

| Concern | Owner | Notes |
|---|---|---|
| **Authoring** | Draw3D panel **Behaviour and rules…** (Godot UI, `rulesview.gd`, new) | opened from the Inspector's HOW IT WORKS and the construction bar's beside strip (G2's entry point) |
| **Definitions** | Host document | Per design or per thing as now. Named behaviour sets in `behaviours.txt` (shared, like `uses.txt`) and `library/mine/` for yours. |
| **Validation and capability manifest** | Host `kind.rs` | Words, parameters, and which events, actions and conditions an entity is capable of (extends `ON_NEEDS` to actions and conditions) |
| **Execution** | Godot `rules.gd` (new), on `events.gd` | Event-driven; a tick no faster than 5 Hz for range conditions |
| **Running state** | Facts | `state` is a fact on the entity. A running step is not saved; on load the highest state that holds is entered again, as `work.gd` restarts sequences. |
| **Diagnostics** | Playtest window | Each rule, state and step: running, done, failed, and why not, in words |
| **Animation** | Moves (`moves.txt`), looks (`poses.txt`), the Studio | A behaviour step names a **move**; it never copies a clip |

### 4.2 The words (proposed, smallest form)

Three tiers. Each is the one below it, grown. You meet the simplest first.

**Tier 1 — rules** (exist): `on EVENT do ACTION`, `while WORD CONDITION`.

**Tier 2 — states as pages** (G2, as the roadmap has it):

```
state shut
  enter put open 0
  on act open when not fact locked do state open
  on act open when fact locked do say It is locked.
state open priority 1
  enter put open 1
  leave when timer 20
```

**Tier 3 — steps inside a state**, which is where a tree appears. Only six composites, after LimboAI's:

| Word | Meaning | After |
|---|---|---|
| `first` (the most pressing that holds) | dynamic selector: children by priority; a higher one that comes to hold aborts a lower running one | `BTDynamicSelector`; `work.gd` reactions |
| `then` | sequence: stops at the first that fails | `BTSequence`; `sequences.txt` |
| `together` | parallel: ends when the first or all end | `BTParallel` |
| `again N` / `until CONDITION` | repeat | repeat decorators |
| `at most S` | time limit | `BTTimeLimit` |
| `rest M` | cooldown | `BTCooldown` |

Leaves are the existing actions (`do ACT at THING`, `go PLACE`, `wait`, `say`, `put`, `move MOVE`, `face WHO`) and any `cond.gd` condition as a check. Every leaf answers running, done or failed, and failed always carries a reason.

**Reuse with parameters:** `use flee_from with from walker, far 12` takes a named set from `behaviours.txt`. Its parameters are declared there (`needs from: someone; far: metres = 12`). This is LimboAI's subtree mapping in Draw3D words.

### 4.3 Reuse across entity types without one universal class

- A named behaviour declares what it **needs**: the capabilities its steps use (sees, moves, opens doors, offers acts).
- The host's manifest says what each entity **has**.
- The panel offers a behaviour only where needs ⊆ has, and says what is missing otherwise ("a gate cannot flee: it has no legs"). A deer and a wolf share `flee_from`; a gate never sees it.

### 4.4 Several selected

As G2 already plans: the panel shows the intersection, marks mixed values, writes only rows you change, as one `together` undo step. **Individual overrides** stay: a thing's own line replaces the design's line of the same name, exactly as `while W` replaces an earlier `while W` today.

### 4.5 Animation and behaviour together

| Requirement | How | Owner | State |
|---|---|---|---|
| A step shows its animation | `move flee_run` row shows the move's name, its phases and their times, and a small moving picture from the clip library's drawn poses (`cliplibrary.gd` already draws a clip as three moments of bones) | Draw3D | proposed; parts exist |
| **Edit animation** from the step | the row's button calls `studio.edit_move(name, phase)`, which exists and already asks about unsaved changes | Draw3D | entry exists; not called from outside the Studio |
| Return without losing context | the panel keeps `{entity, state, step}` and the scroll position; the Studio's Edit head gets a **Back to behaviour** button while opened from there, which reopens the panel at that row | Draw3D | proposed |
| Stable references, no copies | a step names the move; the move names its clips through `poses.txt`. Editing the move changes it for everyone who uses it, said in the Studio's head ("used by 3 behaviours"). **Copy** stays an explicit choice in the Studio (it exists: Save/Copy/Discard, 7ax). | Draw3D | the naming exists |
| Playback, blending, timing | owned by the move's phases (`time`, `lets`, `yields`, `keeps`) and the `AnimationTree` blend in `animated.gd`. A step may add `at most S` and `speed K`. | Draw3D | phases exist |
| Completion and interruption | `move` answers running until its last phase ends, done after, failed if it cannot begin (with the move's reason). An aborting higher state ends the move by its `yields` rule, or cuts it if none. | Draw3D | `work.gd` already waits for a move (`MOVE_S`) |
| Movement with animation | the motor moves the body; a move's phases say how (`covers`, `pace`, `launch`). Navigation (`nav.gd`) moves it between places. | Draw3D | exists |
| Preview within the behaviour | **Try** in the panel runs the selected entity's behaviour in place for N seconds while the world stays in edit (as Playtest runs). The Studio's preview shows the move alone. | Draw3D | proposed |

**What Godot gives natively:** `AnimationTree` blending, `AnimationPlayer`, retargeting by `SkeletonProfileHumanoid`, `Tree` and `GraphEdit` controls. **What LimboAI would give:** play/await an `AnimationPlayer` clip. It does not know moves, phases or the Studio. **What Draw3D must build:** the step-to-Studio link, Back to behaviour, "used by", and Try. None of this exists yet as a round trip.

---

## 5. What you can do without code, and what still needs development

| Without code, once G2 and tier 3 are built | Still development work |
|---|---|
| Pick a thing, open Behaviour and rules…, and choose from only what it can do | A new **kind of event** (say, "the tide turns") |
| Give it states, each with conditions, priority, on-enter steps and `on` rows | A new **kind of action or leaf** (say, "dig a hole", "throw a rope") |
| Build steps with first / then / together / repeat / time limit / rest | A new **sense** (smell, a light meter) |
| Save a set by name with parameters; give it to any capable thing; to several at once | A new **composite** beyond the six |
| Override one thing's line; keep the rest from its design | Changes to how perception or navigation work inside |
| See, while playing, what each one is doing and why a rule did not fire | Speed work if the budget is missed |
| Jump from a step to its move in the Studio and back | |
| Ask the Assistant to draft a behaviour from existing words; check and apply it as one undo | |

**The line between them:** composing the existing vocabulary is yours, without code. Growing the vocabulary is a development pass, after which the new word is a choice in the panel like any other.

### 5.1 Importing behaviours (brief item 6)

**A. A behaviour composed from existing words** (no new executable code).

1. The Assistant, a file, or another world produces a `behaviour NAME` block: text only.
2. **Library > Import behaviour…** reads it. No copying files between folders.
3. The host validates every word against the manifest. Missing moves, sequences or sounds are listed by name, and the import offers to bring them along if the source has them.
4. It appears in the panel's list with its parameters as rows and its tree drawn.
5. Try it on a chosen thing, then save to `library/mine/`.

*Security:* none beyond what the Assistant already has. It is words checked by the host, applied as one undo; nothing runs that Draw3D did not already contain.

**B. A genuinely new capability** (a new leaf, sense or event).

This is code, so it is a development pass by Claude or another assistant, reviewed, tested by a demo with `expect`, and committed. It then appears as a word.

- **Not proposed:** loading GDScript or a `.dll` from a downloaded package. A script in a Godot project has the full power of the program. It could read files, reach the network, or break saves. That is the trust boundary.
- **The only sandboxed route** would be the parked Luau boundary (`prototypes/luau-boundary`: counted interrupts, opaque keys). If you ever want user-shareable *executable* behaviour, that is where it goes (decision D4, still yours), not into GDScript.

| | Composed behaviour | New executable capability |
|---|---|---|
| Who makes it | you, the Assistant, a shared file | a development pass |
| What is imported | text | code, reviewed and committed |
| Validation | host words + manifest + missing-dependency list | compile check, unit test, demo with `expect` |
| Version | `version N` line in the block; host refuses newer words it does not know, naming them | git |
| Risk | none new | full program power; never from an untrusted package |

---

## 6. Proof of concept: the door and the deer

**Scope:** the smallest thing that proves states, priority, perception, an animation link and the panel. It corresponds to G2's acceptance with the tree tier's first two composites added. It is not G2 in full: no apply-to-several, no templates library beyond one saved set.

### 6.1 The door

```
state shut
  enter put open 0
  on act open when not fact locked do state open
  on act open when fact locked do say It is locked.
state open priority 1
  enter put open 1
  leave when timer 15
on act unlock when walker has 1 key do put locked 0
```

**Shows:** states as pages; conditions with reasons; a locked door saying why; persistence (`state` and `locked` are facts, saved).

### 6.2 The deer

```
use graze with near home
state wary priority 1 when spots walker within 20
  first
    then
      check spots walker within 8
      move startle
      go away from walker 25 at most 10
    then
      face walker
      wait 3
leave when not spots walker for 10
```

**Shows:**
- perception driving a state;
- `first` choosing the more pressing branch and aborting the calmer one when the walker comes within 8;
- `then` with `at most`;
- `move startle` as a step with **Edit animation** → Studio at `startle` → **Back to behaviour**;
- a named set with a parameter (`graze`).

### 6.3 Engine work the proof needs

| Need | Exists? |
|---|---|
| `spots walker within R` for a creature, not only for enemies | partly (`watch.gd`); extend to any creature that `sees` |
| `go away from WHO D` | **no**: a new leaf, using `nav.gd` |
| `state` word, `rules.gd` | **no** |
| `first`, `then`, `at most` | **no**: about 200 lines together |
| panel with rows, Edit animation, Back | **no**; `edit_move` exists |
| "why not" per rule and step in Playtest | partly (G1 lists `on` lines) |

### 6.4 Acceptance

1. Authored entirely through the panel, never in F10 text, on a fresh gate and a fresh deer.
2. **Door:**
   - opens with the act key;
   - with `locked 1`, says "It is locked." and Playtest shows why it did not open;
   - after the key, opens;
   - after save, quit and load, the door is still unlocked.
3. **Deer:**
   - grazes;
   - the walker at 15 m: it faces and waits;
   - the walker at 6 m: the startle move plays and it runs;
   - 10 s out of sight: it goes back to grazing.
4. **Edit animation** from `move startle` opens the Studio at that move. Change its time, then **Back to behaviour**: the same row is selected, and **Try** shows the edited move.
5. Playtest shows the deer's current state and running step, and why `wary` is not current while it grazes.

---

## 7. Staged plan

Each stage is one pass, with a demo carrying `expect` checks, then your hand test. Stages B1–B3 are G2 of the roadmap, re-cut to include the tree tier and the animation link.

| Stage | Builds | Depends on | Acceptance | Automated tests | Your Godot test |
|---|---|---|---|---|---|
| **B0 Decide** | your answers to section 8 | — | answers recorded in an amendment | — | — |
| **B1 States** | `state`/`enter`/`leave`/`priority` words in host; `rules.gd`; state as a fact; Playtest shows current state and why not | G1 (built) | the door of 6.1, written as text | host unit (words), protocol (state saved and restored), `-Demo door -Check` | play the door, locked and unlocked; save and load |
| **B2 Steps** | `first`, `then`, `together`, `again/until`, `at most`, `rest`; leaves answer running/done/failed with reasons; `go away from`; creature `spots` | B1 | the deer of 6.2 as text | unit tests of each composite's abort and resume (headless GDScript, as `chartest.gd`); `-Demo deer -Check`; `-Measure rules` at 200 entities | watch the deer with the controller |
| **B3 Panel** | Behaviour and rules… from the Inspector and the beside strip; rows for states and steps; offers only from the manifest; a tree view with Godot's `Tree` control | B1, B2 | 6.4 items 1–3, authored in the panel | studio-style click script (`rulesplay.gd`: clicks like a person, `Controls.automated`); `-Tour -Only rules` | author both from scratch |
| **B4 Animation link** | Edit animation, Back to behaviour, "used by", Try | B3; Studio unparked for this entry only | 6.4 item 4 | click script: open from a step, change, back, same row; `-Demo deer` re-run | the round trip by hand |
| **B5 Reuse** | `behaviours.txt` sets with parameters and `needs`; apply to several; overrides; Import behaviour… with validation and a missing-dependency list | B3 | a deer and a wolf share `flee_from`; two gates given door rules at once; an imported block with a missing move is refused, naming it | protocol (import validation, `together`), demo | import a block the Assistant drafted |
| **B6 Migrate** (later) | `watch.gd` phases and `work.gd` reactions re-expressed as states and `first` | B2, G5 | tours `work`, `village` unchanged | existing tours | — |

**Stop point before B6:** if `-Measure rules` misses its budget in B2, try the LimboAI hybrid (section 3.3) behind `rules.gd` before going on. Probe P1 (section 8) should be run first.

---

## 8. Open decisions and risks

### 8.1 Decisions for you

1. **Behaviour's home.** Behaviour stays text in the host's document (recommended), and no part of it is authored in the Godot editor. Agreed?
2. **Tree tier.** States as pages plus the six composites of 4.2 (recommended), or pages only for now?
3. **Words.** Are `first`, `then`, `together` clear to you, or would you rather see "most pressing", "in order", "at once"?
4. **LimboAI fallback.** May I download the LimboAI v1.8.1 GDExtension (the release's GDExtension zip; I will state its size from the release page before downloading) for probe P1, into `downloads-untrusted/`, before B2? Or not until a budget fails?
5. **Studio unparking.** B4 touches the Studio's Edit head (a Back button, "used by"). The Studio is parked: is this exception allowed?
6. **Code route.** Imported behaviour stays text-only; new capabilities come only as development passes. Luau stays parked. Agreed?
7. **Proof subjects.** Door and deer (recommended), or a door and a guard, closer to the stealth genre?
8. **Order against the roadmap.** Replace G2 by B1–B3 (recommended, same intent), or do G2 as written first?

### 8.2 Risks requiring verification

| Risk | How to check | When |
|---|---|---|
| **P1:** LimboAI's 4.6 GDExtension loading in Godot 4.7.2 | after your yes: a headless probe project in `probes/` loads it and runs a two-task tree; log the result | before relying on the fallback |
| A GDScript runner too slow at scale | `-Measure rules`: 200 entities with three states and a `first` of three branches | B2 |
| `watch.gd`'s built-in phases and new states fighting over a creature | make `watch.gd` raise events only (`spots`, `lost`) and let rules decide; a demo where both are present | B2 |
| A step's move interrupted mid-phase leaving a pose | `yields` or cut, tested in the deer demo with the walker stepping in and out of 8 m | B2 |
| Saved games with a running step | a save in the middle of `wary`: on load the deer re-enters the highest state that holds; protocol test | B1, B2 |
| Studio window height in the 1280×720 tour (known gap since 7ax) | the B4 click script runs at the user's size, not the tour window | B4 |

---

## Sources

**In this repository (read 2026-10-21):**
- `editor/run.ps1`
- `editor/godot/project.godot`, `cond.gd`, `events.gd`, `wires.gd`, `work.gd`, `watch.gd`, `folk.gd`, `studio.gd` (`edit_move`), `cliplibrary.gd`, `clips.gd`, `animated.gd`, `assistant.gd`, `library.gd`
- `moves.txt`, `reactions.txt`, `sequences.txt`
- `editor/host/src/shapes/kind.rs` (`HELD`, `ON_HAPPENS`, `ON_NEEDS`, `ON_DOES`)
- `prototypes/luau-boundary`
- `Draw3D_Gameplay_Roadmap_2026-10-21.md`

**LimboAI:**
- [Repository and README](https://github.com/limbonaut/limboai)
- [Releases](https://github.com/limbonaut/limboai/releases)
- [Getting LimboAI (module vs GDExtension)](https://limboai.readthedocs.io/en/latest/getting-started/getting-limboai.html)
- [Custom tasks](https://limboai.readthedocs.io/en/latest/behavior-trees/custom-tasks.html)
- [Creating a state machine](https://limboai.readthedocs.io/en/latest/hierarchical-state-machines/create-hsm.html)
- [BTTask](https://limboai.readthedocs.io/en/latest/classes/class_bttask.html)
- [BehaviorTree](https://limboai.readthedocs.io/en/latest/classes/class_behaviortree.html)
- [BTDynamicSelector](https://limboai.readthedocs.io/en/latest/classes/class_btdynamicselector.html)
- [BTPlayAnimation](https://limboai.readthedocs.io/en/latest/classes/class_btplayanimation.html)
- [Asset Library listing (4.6+)](https://www.godotengine.org/asset-library/asset/4852)
- source folders `bt/tasks/*` and `editor/` and `gdextension/limboai.gdextension`, read through the GitHub API

**Alternatives:**
- [Beehave](https://github.com/bitbrain/beehave)
- [Godot State Charts](https://github.com/derkork/godot-statecharts)
- [Behaviour Toolkit](https://github.com/ThePat02/BehaviourToolkit)
