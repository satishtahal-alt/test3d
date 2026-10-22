# Engine architecture review 18

Pass 18: a bounded benchmark of content with real weight. No architecture was changed and nothing was optimised. The aim was to find where realistic content presses on what exists.

Decisions recorded from your answer to Pass 17: rules 79–81 approved; a flag stays until `accept`, a later plain match does not clear it; marks and "so far" stay in the real core only, the frozen prototype kernel is not touched; the doubled store work stays, no incremental store transaction; build order unchanged; the 38–40 flag-count difference in the measuring script is not chased unless a real invariant fails.

---

## 0. Outcome

**The core does not feel the weight.** Every edit, drag step, move, turn and undo costs the same 1–5 ms whether a window is 96 triangles or 708, and whether the world holds 5 different meshes or 36,503. Loading and rebuilding scale with the number of things and owners, as before. Every agreement check, drop and late-result check held in all nine worlds.

**Three things outside the core do feel it**, in this order of how soon a real project would meet them:

1. **Draw units in the editor.** The adapter draws one unit per mesh per region. With a modest kit of 355 meshes that is 20,707 units at 100 villages and 82,648 at 400, and the idle frame over the whole world is 38 ms and 175 ms. It is the number of units, not the triangles: the same frame with one instance per unit costs the same.
2. **Content is never let go.** Where parts differ from one another, every regeneration leaves the old parts in the host's memory and on the graphics card. The world with no two parts alike went from 2.8 GB to 4.4 GB of held content in one scripted session, and the host process to 5.2 GB.
3. **The editor takes content in without a limit per frame.** In the same world the editor's frames were 1.1–2.5 s long for about 20–30 s while 2.6 GB of meshes arrived. The host stays responsive; the editor does not.

The first is a problem for realistic content now. The second and third are severe only where nothing repeats, which I treat as a bound, not a target (decision 2).

All suites pass: core 6, the prototype's 57 against the core and in the prototype, 11 native, 7 protocol (1 new). No frozen test changed.

---

## 1. What was weighed

All content is project code in the host (`editor/host/src/shapes.rs`, `shapes/real.rs`); the engine knows none of it. It is stand-in geometry of the right weight, not art.

| Part | Triangles, plain | Triangles, real | Mesh bytes | Time to make |
|---|---|---|---|---|
| Roof (tiles laid one by one, gutters, chimney, dormer) | 96 | 1,608 | 134 KB | 0.10 ms |
| Wall, 6 m (plinth stones, timbers, brackets, quoins) | 36 | 924 | 85 KB | 0.06 ms |
| Window (glazing bars, shutters with slats, flower box) | 96 | 708 | 57 KB | 0.04 ms |
| Door (boards, nails, hinges, arch, steps, lantern) | 132 | 792 | 70 KB | 0.04 ms |
| Tree (broadleaf / pine) | – | 2,304 / 2,088 | 93 KB | 0.06 ms |
| Prop (barrel, crate, sacks, fence, cart, lamp post) | – | 156–480 | 10–25 KB | 0.004–0.01 ms |
| Figure (stand-in for a character) | – | 22,912 | 843 KB, plus 284 KB of skin and 4 KB of rig | 0.76 ms |

A real village is 50 houses (21 things each, about 13,000 triangles a house), a grove of 60 trees, 25 clusters of 6 props, and 5 figures: 1,265 things and about 0.94 million triangles. Trees, props and figures come from three new generators with one owner each per village, so a real world has four times the owners of a plain one.

Three kinds of variety:

- **alike**: every part of a kind is the same mesh (13 meshes in all).
- **kit**: 16 variants of each roof, wall, tree, prop and figure, two house heights, five roof colours (355 meshes, 43 MB).
- **own**: no two roofs, walls, trees, props or figures alike. Windows and doors stay shared, as kit pieces would in any project. (7,303 meshes at 20 villages, 36,503 at 100.)

Worlds: plain at 20 and 100 villages (the baseline, as Pass 17 but with content not slowed); real alike, kit and own at 20 and 100; real kit at 400. One run each, editor uncapped. `editor\godot\results-18.json`.

Not weighed: content that is slow to make. These parts take under a millisecond; a real importer or baker takes far longer. The existing knob that slows the realizer stands for that and was measured in Passes 15–17; it was off here so that real making cost was part of the result.

---

## 2. Measurements

### Weight

| | plain 100 | alike 100 | kit 100 | own 100 | kit 400 |
|---|---|---|---|---|---|
| Things | 105,000 | 126,500 | 126,500 | 126,500 | 506,000 |
| Owners | 100 | 400 | 400 | 400 | 1,600 |
| Different meshes | 5 | 13 | 355 | 36,503 | 355 |
| Mesh bytes | 0.03 MB | 1.4 MB | 43 MB | 2,621 MB | 43 MB |
| Other output bytes (colliders, skin, rig) | – | 0.3 MB | 4.4 MB | 138 MB | 4.4 MB |
| Triangles placed | 6.7 M | 94 M | 94 M | 94 M | 378 M |
| Cooked cells | 9.2 MB | 11.1 MB | 11.1 MB | 11.1 MB | 44 MB |
| Draw units | 632 | 1,656 | 20,707 | 37,530 | 82,648 |

### Opening

| | plain 100 | alike 100 | kit 100 | own 100 | kit 400 |
|---|---|---|---|---|---|
| First things on screen | 0.45 s | 0.18 s | 0.19 s | 0.90 s | 0.21 s |
| Half the world on screen | 0.91 s | 0.78 s | 1.12 s | never before landing | 2.8 s |
| Landed and editable | 1.39 s | 1.27 s | 1.6–1.9 s | 4.3–5.3 s | 6.2 s |
| of which walk (plan and expand) | 0.88 s | 0.96 s | 1.33 s | 2.58 s | 4.62 s |
| Content still owed at landing | 0 | 0 | 0 | 30,585 pieces | 0 |
| Everything drawn with its content | 1.39 s | 1.28 s | 1.7–2.0 s | 25–34 s | 6.5 s |
| Meshes sent to the editor | 0 | 1.4 MB | 43 MB | 2,621 MB | 43 MB |
| Editor time building meshes | 0 | 3 ms | 79 ms | 5.7 s | 95 ms |
| Editor frame while loading | 3.8 ms | 2.9 ms | 8–22 ms | 1,100–2,500 ms | 8 ms |

The two ranges are two runs of the same world. The late first things in plain 100 and own 100 (0.45 s, 0.90 s) are the editor and host putting away the previous, larger world before starting, as in Pass 17; they are not content weight.

### Idle frame

| | plain 100 | alike 100 | kit 100 | own 100 | kit 400 |
|---|---|---|---|---|---|
| Whole world in view | 2.1 ms | 15.7 ms | 38 ms | 97 ms | 175 ms |
| The same, one instance per unit | 0.65 ms | 1.7 ms | 37.5 ms | 96.5 ms | 174 ms |
| Draw calls | 633 | 1,657 | 20,708 | 37,531 | 82,649 |
| Standing in a street | 0.6 ms | 2.3 ms | 2.6 ms | 9.3 ms | 3.0 ms |
| Draw calls in the street | 93 | 242 | 2,172 | 3,558 | 2,172 |

At 20 villages: 0.7 ms plain, 3.6 ms alike, 4.2 ms kit (4,216 units), 13 ms own (7,541 units).

### Editing and regeneration

| Core / whole exchange, ms | plain 100 | alike 100 | kit 100 | own 100 | kit 400 |
|---|---|---|---|---|---|
| Edit a window | 0.7 / 1.2 | 0.9 / 1.6 | 0.9 / 1.8 | 1.3 / 3.2 | 1.4 / 2.6 |
| Move a house | 0.9 / 1.6 | 1.6 / 2.5 | 1.2 / 2.1 | 1.1 / 2.1 | 1.3 / 2.4 |
| Move a tree | – | 0.6 / 1.2 | 0.6 / 1.2 | 0.7 / 1.3 | 0.9 / 1.6 |
| Drag step | 1.0 / 1.5 | 1.2 / 1.8 | 1.0 / 1.6 | 1.2 / 1.7 | 1.3 / 2.0 |
| Undo | 0.01 / 1.1 | 0.01 / 2.2 | 0.01 / 3.4 | 0.01 / 4.1 | 0.02 / 4.4 |
| Regenerate a village (wall height) | 8 / 11 | 7 / 16 | 15 / 74 | 18 / 166 | 16 / 79 |
| pieces asked for, and MB fetched, each | 4 / 0 | 4 / 0.4 | 48 / 4.5 | 151 / 9.0 | 46 / 3.9 |
| Regenerate a grove (new seed) | – | 0.9 / 3.9 | 4.2 / 10.7 | 2.5 / 61 | 1.1 / 5.4 |
| Regenerate a troupe of figures | – | 0.7 / 2.2 | 0.6 / 2.2 | 2.3 / 25 | 0.4 / 2.0 |

Undo is slower in the real worlds because the steps it crosses are regenerations that touch about 20 cells, not because cells are heavier.

### Everything built again (generator upgrade), and reconnecting

| | plain 100 | alike 100 | kit 100 | own 100 | kit 400 |
|---|---|---|---|---|---|
| Upgrade, run through to landing | 2.4 s | 2.6 s | 2.2–2.7 s | 4.6 s | 31.3 s |
| New content it asked for | 0 | 0 | 0 | 7,072 at landing; 15,000 in all | 0 |
| Editor frame meanwhile | 2.8 ms | 16 ms | 62 ms | 579 ms | 241 ms |
| Stop, as the editor sees it | 28 ms | 66 ms | 100–260 ms | 1,470 ms | 700 ms |
| Stopped: world unchanged | yes | yes | yes | yes | yes |
| Reconnect: cells in | 78 ms | 129 ms | 111–138 ms | 132–150 ms | 450 ms |
| Reconnect: everything drawn | 0.2 s | 0.35 s | 3.2–3.6 s | 10.8–11.8 s | 13.7 s |

### Memory

| MB | plain 100 | alike 100 | kit 100 | own 20 | own 100 | kit 400 |
|---|---|---|---|---|---|---|
| Host, loaded | 189 | 240 | 277–314 | 665 | 3,250 | 943 |
| Host, after edits | 176 | 234 | 323 | 847 | 3,406 | 906 |
| Host, after building everything again | 329 | 373 | 499 | 1,190 | 5,170 | 1,609 |
| Content held by the host: loaded → end | 0 → 0 | 2 → 4 | 47 → 71 | 552 → 995 | 2,759 → 4,362 | 47 → 71 |
| Pieces held: loaded → end | 5 → 29 | 13 → 39 | 355 → 608 | 7,303 → 11,695 | 36,503 → 52,891 | 355 → 609 |
| Editor process, loaded | 771 | 778 | 976 | 780 | 1,800–2,030 | 1,968 |
| Video memory, loaded → after building again | 67 → 66 | 69 → 69 | 101 → 114 | 367 → 622 | 1,737 → 2,671 | 218 → 231 |

The editor process is about 565 MB before it holds anything.

---

## 3. Findings

### Finding 1. The core's costs follow things and owners, not weight (confirmed)

Edits are within a millisecond of the plain world in every real one. A load lands in time proportional to things: 126,500 real things in 1.3 s against 105,000 plain in 1.4 s; 506,000 in 6.2 s against Pass 17's 420,000 in 4.4 s. The naming of content (rule: a thing names what it looks like, and where it stands is no part of the name) is what keeps weight out of the transaction. Nothing here asks for a change.

### Finding 2. Where no two parts are alike, the build itself is 2.7 times slower (measured)

Own 100 walks in 2.6 s against 0.96 s for the same things alike. Two causes, both expected and both in project code's choices as much as the engine's: a house that looks at which house it is cannot share its plan with any other house, and 36,503 different requests have to be written out and named where 13 did. I did not separate the two. It is not a problem at this size.

### Finding 3. The editor's draw units are the first real limit (measured, problem)

Pass 14 chose a draw unit to be one mesh across a region of 4 × 4 cells. That was measured with five meshes. With 355, a region holds up to a few hundred units, and the frame over the whole world is all draw calls: 38 ms at 100 villages, 175 ms at 400, the same with every unit cut to one instance. Standing in a street it is 2–3 ms, because only nearby regions are in view. With all parts alike, 94 million triangles in 1,657 units draw in 16 ms.

So triangles are not the pressure at this weight; the count of separately drawn things is. This is in the adapter (Godot side), not the core, and not the protocol. Larger regions far from the viewer, not drawing far regions at all, or simpler stand-ins at distance would each address it. I changed nothing.

A side effect: while the idle frame is long, everything the editor does between frames is slow too. The upgrade's "stop" took 0.1–1.5 s as the editor saw it in the heavy worlds. The host stops within one owner's build as before (about 70 ms a village here, from 28.6 s of walk over 400 villages; it was about 50 ms in Pass 17). The rest is the editor waiting for its own frames. I could not separate the two more exactly from what the script records.

### Finding 4. Content is never let go, on either side (measured, problem where parts differ)

This has been on the carried list since Pass 13 as "content never let go". Now it has a size. Regenerating one village with its own parts leaves 151 old pieces behind (about 9 MB). The scripted session's edits took own 100 from 36,503 pieces to 37,891; building everything again took it to 52,891 and 4.4 GB, because the new house names gave every house a new variant. The host process ended at 5.2 GB and the graphics card at 2.7 GB. With a kit the same session goes from 47 MB to 71 MB, which is nothing.

The host's cache is "disposable: anything in it can be made again" by its own comment; nothing ever disposes of it. The editor likewise keeps every mesh it was ever sent until it reconnects.

### Finding 5. The editor does not bound what it takes in per frame (measured, problem where parts differ)

Opening own 100, the world landed at 4–5 s and was complete at 25–34 s. For that half minute the editor's frames were 1.1–2.5 s each. It asks for all the content a batch of cells names in one request, waits for the answer, and builds every mesh in it before drawing again. The host was never blocked; the invariant you set in Pass 16 (the editor never blocks on engine work) holds for transactions and is broken here by the editor's own intake. With a kit the same path costs 79 ms in all and is invisible.

### Finding 6. Regeneration is cheap in the core and paid for in content transfer (measured)

Regenerating a village is 7–18 ms of core in every world. The whole exchange is 11 ms plain, 74–79 ms with a kit and 166 ms with own parts, and the difference is fetching 4–9 MB of new meshes and building them. Groves and troupes regenerate in 2–11 ms with a kit, because every tree and figure they place already exists.

### Finding 7. Variation multiplies (observation)

My first kit was 32 variants, three house widths, four heights and five roof colours. At 20 villages that was 1,242 meshes and 154 MB, because each roof variant is a different mesh at every size and colour. Colour as part of the mesh is this project code's choice; a real project would tint, not remake. The engine cannot know that, and it will make what it is asked for. The kit used above (16 variants, two heights, five colours) is the reduced one.

### Finding 8. A move on a child of a generator that ignores pins does nothing, silently (observation, user-facing)

My new generators at first did not read their pins. Moving a tree was then accepted, recorded, committed as a step of history, and the tree stayed where it was, with no note. The new protocol test caught it and the generators now honour pins as the row does. But the engine let project code ignore an edit without saying so. A generator that never asks for its pins while its scope has some could be reported.

### Finding 9. A figure this heavy is nothing as a static thing (observation)

2,000 figures of 22,912 triangles each are in the 400-village world and are not visible in any number. That says only that the mesh, skin and rig bytes are carried like any other content. It says nothing about what characters will really cost: animation state per instance, skinning, and things that change every frame are not in this benchmark.

### Finding 10. `p04` assumed the last drag step was still building when the commit arrived (fixed, test only)

With a seventh test in the process the step sometimes landed first, and the test then read the step's report as the commit's. It now takes the commit's own answer in that case. Same class as Pass 17's `p03` fix.

### Carried, unchanged

Stop granularity at the owner; `check_upgrade` and a generator swap's settling on the host's thread; plans run once per facade side; square-on boxes around turned things; three turn-aware paths untested; kept bytes as hex; non-empty inbox and `FAILED` content untested on the editor side; build order not the viewer's; hand-placed things shown only at landing.

---

## 4. Remaining risks, by severity

1. **Draw units in Godot (medium-high, was low).** Finding 3. A kit of a few hundred meshes makes the overview unusable at 100 villages and above.
2. **Content lifetime (medium, was a carried note).** Finding 4. Harmless with a kit; unbounded growth with distinct content, in the host, the editor and the graphics card.
3. **Content intake in the editor (medium).** Finding 5. Only where thousands of meshes arrive at once.
4. **Things as Godot nodes (medium).** Unchanged, and Finding 9 says this benchmark did not touch it.
5. **Slow content (medium-low).** Not re-measured here. With a real importer, "everything drawn" follows making time, not transfer time.
6. **Opening large projects (medium-low).** 6.2 s to edit at 506,000 things; the screen fills from 0.2 s.
7. **Real content weight in the core (low, was medium).** Finding 1.
8. **A busy engine; turn-aware paths (low).** Unchanged.

---

## 5. What becomes necessary for the eventual engine

- **What is drawn together must follow the viewer**, not only the map: nearby as now, far away merged, simplified or not drawn. This is where levels of detail will live, and they are content like any other (another output of the same request).
- **A rule for letting content go.** The cache is already disposable by definition. What is missing is who says when: what no kept version names, least recently used, a budget.
- **A budget for content intake in the editor**, as the host has one for transactions.
- **Variation without new meshes**: tint and similar per-instance values carried by the thing and not by its content. The cooked member has no place for them today.
- **For characters and gameplay later**: this pass shows heavy static content is carried. It does not show anything about per-frame state, which is the next unknown of that kind.

---

## 6. Decisions for you

1. **What to address first.** I recommend the draw units (Finding 3), in the adapter only: it is the one pressure a realistic kit already meets, and it needs no change to the core or the protocol. Then content lifetime and bounded intake together as one slice (Findings 4 and 5), which does touch the host.
2. **Is "no two alike" a target or a bound?** I recommend a bound. A project of 36,503 different meshes is 2.6 GB of geometry before any texture; the realistic target is a kit of hundreds to a few thousand. If you agree, Findings 4 and 5 are medium, as ranked. If projects with mostly unique generated geometry are a target, they become the first priority, ahead of draw units.
3. **Should the engine report a generator that ignores a pin?** (Finding 8.) I recommend yes, as a note on the owner; it is small and it is the kind of silent failure a person cannot diagnose.
4. **Per-instance variation (tint) as part of a thing** (Finding 7). Not needed yet. Say if you want it scoped; it changes the cooked member, so it is a protocol decision.
5. **Keep the benchmark?** `editor\run.ps1 -Weigh` and the three generators are project code and can stay as the standing weight test, or be removed. I recommend keeping them: every later pass can be checked against the same nine worlds.

---

## Appendix. What changed, and running it

No engine behaviour changed. Added:

- `core/src/jobs.rs`: `Results::bytes()`, a count for reporting.
- `editor/host/src/shapes.rs`, `shapes/real.rs`: heavier parts, three generators (`grove`, `yard`, `troupe`), a `Profile` (real or plain, variety) that picks the registry and the realizer; rounded mesh primitives.
- `editor/host/src/lib.rs`: `scene` takes `content` and `variety`; `check` reports the host's memory, everything its cache holds, and making time.
- `editor/host/examples/weights.rs`: the table in section 1.
- `editor/host/tests/protocol.rs`: `p07` (a real project is served the same; variety changes meshes, not things; a tree can be moved); `p04` fix.
- `editor/godot/weigh.gd`, `-Weigh` and `-Only` in `run.ps1`; two counters in `world_view.gd`.

```
cd editor/host
cargo run --release --example weights        # what each part weighs
cargo test --release                         # protocol 7
cargo build --release

editor\run.ps1 -Weigh -TimeoutSec 3300       # all nine worlds; writes editor\godot\results-weigh.json
editor\run.ps1 -Weigh -Quick                 # the four 20-village worlds
editor\run.ps1 -Weigh -Only real_kit_100     # one world
```

`editor\godot\results-18.json` is the run in section 2 (for the two "own" worlds, the rebuild and reconnect figures are from a second run, `results-18-again.json`, after the script's stop was fixed to arrive in time). `shot-real_kit_20.png` and `shot-real_kit_20-close.png` show the content.
