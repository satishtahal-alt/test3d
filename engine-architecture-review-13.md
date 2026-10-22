# Engine architecture review 13

Pass 13: the two content-model corrections. Place is out of content identity, and things have a turn.

---

## 0. Outcome

Both corrections are in and measured.

- **Place is no part of what a leaf looks like.** Things alike in all but place name one piece of content. 105,000 things now draw from 5 meshes (was 9 names for 6 meshes); with every village different, 401 (was 801 names for 501).
- **A thing has a turn about the vertical axis.** What is below it turns with it. The cooked member carries it in 4 bytes (92 bytes a member, was 88).

A side effect you did not ask for and should know about: draw units roughly halved, and the idle frame with them (section 3). I did no draw-unit work; fewer names per cell means fewer units.

All suites pass: core 6, the prototype's 57 tests against the core, 9 native tests (two new), the prototype's own 57. The scripted editor run agrees with the host after load, after edits with undo and redo, and after a reconnect, in two full runs. I did not drive the editor by hand.

The protocol slice is not started.

---

## 1. What changed

### Place out of content identity

- A leaf's request for content is its brief without `x`, `y`, `z` and `yaw`. Those four names in a brief are placement, and the realizer is never given them.
- Consequence for realizers: a look that depends on where a thing stands (a foundation that follows a slope, say) has to be given that as a field of its own. It will not be shared, which is correct.
- The cell's derived collider summary already read each member's place; it now reads its turn too.

### The turn

- A generated thing's turn is the field `yaw` in its brief, in degrees. A document node has a `yaw` of its own. Quarter turns are exact.
- **Frames carry it.** A thing stands and is turned in the frame of what it stands in; what is below it stands in its frame. The world store composes this when it flattens pieces into cells, so a cooked member's position and turn are in the world.
- **Turning is an edit like any other.** Turning a generated child is a recorded `Set` of `yaw`; turning a node sets its `yaw`. Neither plans or makes anything.
- **It survives leaving and coming back.** A detached child becomes a node with the turn it stood at; adopting it gives it back its place and turn in the scope's frame.
- Also made turn-aware, because they read places across frames: where a child stands (for attachments), what is in the way of a turned scope, the footprint of a turned obstacle or frozen child, where a publisher stands as a reader sees it, and the published list of children (a turn is listed where there is one).
- Not done: full rotation. Nothing needs it yet.

### Changes to the frozen suite

1. `x02` now expects that a moved house's garden follows and is **not** re-made (`(2, 0)` plans and realizes, was `(2, 1)`), with the comment saying why. As you decided.
2. The prototype's own kernel leaves place out of a leaf's request too (one line in `kernel/build.rs`), so that `x02` means the same thing in both kernels. The prototype has no turn; nothing in it changed for that.

### The host project and the editor

- `shapes.rs` has its own house: four sides are one facade, turned to face out. A facade always runs along its own x, so it is planned the same on every side and what is on it is made once. Windows and doors lost their "which way does the wall run" branch.
- The host has `turn` and `turn_node` commands, and says a thing's turn when asked what it is.
- The Godot side reads the 92-byte member, places each instance with its turn, and picks against the box around the turned mesh.

---

## 2. Rules added

66. **Where a thing stands and how it is turned are no part of what it is asked to look like.** `x`, `y`, `z` and `yaw` in a brief are placement; a realizer is not given them.
67. **A thing has a place and a turn about the vertical axis, in the frame of what it stands in.** What is below it stands in its frame. A node is a frame like any other.
68. **A turn plans nothing and makes nothing.** Like a move, it changes where things are and nothing else.

---

## 3. Measurements

Scripted editor run; times are command to drawn. Two full runs agreed within the ranges shown. "Before" is Pass 12.

| Things in the world | 21,000 | 105,000 | 420,000 |
|---|---|---|---|
| Edit one window | 0.8 ms | 0.8–1.0 ms | 1.4–1.5 ms |
| Move one house | 1.0 ms | 0.9–1.2 ms | 1.0 ms |
| Turn one house (21 things) | 0.8–0.9 ms | 0.6–0.7 ms | 0.6 ms |
| A village's wall height | 9 ms | 8–10 ms | 8 ms |
| Undo | 0.7 ms | 0.6–0.7 ms | 0.6 ms |
| First show | 0.20 s | 0.80–0.85 s | 3.3 s |
| Reconnect | 40–43 ms | 190–210 ms | 780–820 ms |
| Draw units | 1,760 | 8,800 | 35,200 |
| Idle frame, world in view | 1.5–1.6 ms | 10–12 ms | 24 ms |

| Before and after | Pass 12 | Now |
|---|---|---|
| Names of content, all villages alike | 9 (6 different meshes) | 5 (5 different) |
| Names of content, every village different | 801 (501 different) | 401 (401 different) |
| Mesh bytes, alike / different | 72 KB / 6.0 MB | 37 KB / 2.5 MB |
| Draw units at 105,000 things | 16,000 | 8,800 |
| Idle frame at 105,000 things | 38–45 ms | 10–12 ms |
| Idle frame at 420,000 things | 48–50 ms | 24 ms |
| First show at 105,000 / 420,000 | 1.0–1.1 s / 4.4–4.6 s | 0.8 s / 3.3 s |
| Wall-height change: new meshes | 8 | 4 |

- A turn costs less than a move because it changes fewer cells.
- All 20 test clicks on roofs hit at every size. Of 40 clicks on windows (25–29 of them standing turned), 35–37 hit; every miss was a door that stands in front of that window. The prototype's facade puts both at the middle of the south wall. That is the scene, not the picking.
- The frame with one instance per unit still equals the full frame. Draw units are still the whole frame cost; there are just fewer.
- The frame fell more than the unit count did (4× for 1.8× fewer units at 105,000). I did not look into why, and would not lean on the exact figure.

### The core

| 340,000 things | Pass 11–12 | Now |
|---|---|---|
| First build | 3.3 s | 3.0 s |
| Memory, settled | 808 bytes a thing | 808 bytes a thing |
| Edit one window | 0.4 ms | 0.4 ms |
| Move one house | 1.0 ms | 1.0 ms |
| Pieces of content named | 7 | 4 |

The turn added no memory: it fits in space the thing already had.

---

## 4. Findings

### Finding 1. A facade is now planned once per size, but still run once per side (observation)

The four sides of a house get the same parameters apart from length and the door, yet each is its own scope and its plan runs for each. Sharing plans between scopes that are asked the same thing would cut first build (3.0 s at 340,000 things, most of it plans). This was true before; the turn makes it visible because the sides are now truly alike. Not proposed for now: it touches how scopes own their records.

### Finding 2. Square-on boxes are loose around turned things (low)

Three places treat a turned footprint as the square-on box around it:

- what is in the way, as a generator asks it;
- the footprint of a turned obstacle or frozen child;
- picking in the editor.

For quarter turns they are exact. At 45° the box is up to 41% wider than the thing. A layout will leave a little too much room, and a click near a turned thing's corner can hit it when it should miss. Fine for now; exact footprints are a contained change when a real layout needs them.

### Finding 3. Generators do not know about turns unless they ask (observation)

A thing's `size` (in the cooked member, in the published list of children) is its size before it is turned. A generator that reads another's children and cares about turned footprints has to read `yaw`. The prototype's generators do not, and none of them turns anything, so the acceptance suite does not exercise this.

### Finding 4. What the tests cover and what they do not

Covered by tests: composition of turns through three levels and a turned root; turned things in the randomised store test (any sequence of transactions equals a build from nothing); turning a child at 90° and 30°; that a turn plans and makes nothing; detach and adopt of a turned house; turning a node; a fresh build giving the same cooked bytes after each.

Written but not covered: what is in the way of a turned scope; where a turned publisher stands as a reader sees it; an attachment whose target is turned (its offset stays in world axes, so it does not go round with the target). I would not trust these three until something uses them.

### Finding 5. The middle window stands behind the door (scene, low)

On the south wall the prototype's facade places a window and the door at the same spot. It was always so; boxes hid it and meshes show it. It is in the frozen generators, so I left it.

### Carried from Pass 12, unchanged

First show draws cells twice; content is never let go; kept bytes are written as hex; a generator has no look of its own.

---

## 5. Remaining risks, by severity

1. **The editor protocol beyond this slice (medium-high).** Dragging, late content, the review inbox, things as Godot nodes, a busy engine. Next.
2. **Opening large projects (medium-high).** 3.0 s of core time at 340,000 things; 3.3 s to first show at 420,000.
3. **Draw units in Godot (medium, was medium-high).** 10–12 ms at 105,000 things and 24 ms at 420,000, with the camera's range cutting off part of the largest world. Still one unit per piece of content per cell; a world with more variety per cell will have more.
4. **Real content weight (medium).** Unchanged: heavy meshes, textures and materials are untested.
5. **Turn-aware paths nothing uses yet (medium-low).** Finding 4.
6. **Memory where nothing repeats (medium).** Unchanged.
7. **Content identity (low, was medium).** The two rule-level problems are fixed. What is left is that a look depending on place must be given it by name.
8. **Wide scopes (medium-low).** Unchanged.

---

## 6. Decisions for you

1. **Confirm the second touch to the frozen prototype**: its kernel leaves place out of a leaf's request, so `x02` holds in both kernels. The alternative was to let the prototype's own run of `x02` fail.
2. **Go on to the protocol slice?** You set the order; nothing found here argues against it. Draw units stay after it.
3. **Nothing else.** Findings 1–3 need no decision now.

---

## Appendix. Running it

```
cd core
cargo test                                   # store 4, bridge 1, source 1

cd core/acceptance
cargo test                                   # scenarios 28, nested 11, cross 9, deferred 9, native 9
cargo run --release --example scale          # core measurements

cd editor/host
cargo build --release

editor\run.ps1                               # the editor, by hand
editor\run.ps1 -Measure                      # section 3; writes editor\godot\results.json,
                                             # shot.png, shot-close.png and shot-turned.png
```
