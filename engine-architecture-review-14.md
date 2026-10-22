# Engine architecture review 14

Pass 14: the second slice of the editor protocol. Dragging as staged transactions, content that arrives after its cells, and the review list.

---

## 0. Outcome

The slice is in and measured. Three things cross the socket that did not before:

- **A staged edit.** Any edit command may say `stage: true`. The editor is shown the world it would make; the document, the history and the committed world know nothing of it. The next staged edit replaces it, `commit` makes it one edit, `drop` leaves nothing. A drag is sixty of these and one commit, at 0.75–0.95 ms a step in every world size.
- **Content that comes later.** The host runs the engine in the mode that asks for content and does not wait for it, with worker threads making it. Cells arrive at once, naming content; what the editor asks for before it is made is owed and sent when made, unasked. Nothing else changes when it arrives.
- **What needs a look.** `review` lists inbox items, edits that landed somewhere other than where they were made (flagged), and the build's notes. `accept`, `forget` and `dismiss` deal with them; each is an edit, and undoable.

Also: the protocol now has tests that run without Godot (`editor/host/tests/protocol.rs`, three tests), which Pass 11 noted as missing.

All suites pass: core 6, the prototype's 57 against the core, 9 native, 3 protocol. The scripted editor run agrees with the host after load, after edits with undo and redo, after the review, and after a reconnect, in all four worlds, in two full runs. Every drag invariant held in every world. I did not drive the editor by hand beyond the one check in section 1.

Draw-unit work is not started.

---

## 1. What changed

### The host (`editor/host`)

- It is a library with a thin `main`. `Session` owns the engine, a `Pool` of workers, the staged transaction if there is one, the versions for undo, and what the editor has been sent and is owed.
- The editor's frames are read on a thread of their own. Between frames (every 5 ms at most) the host takes in what the workers finished, refreshes the committed world and the staged one, tells the pool what is wanted now, and pushes owed content.
- `WANT` first takes in what the workers have finished: content made by then is sent, not owed.
- Every `DONE` says `staged`, `waiting` (content asked for and not made), `inbox`, `flagged`, `badges`, `can_undo`, `can_redo`.
- New commands: `commit`, `drop`, `review`, `accept`, `forget`, `dismiss`, `check_upgrade`, `upgrade`, `upgrade_all`; and two for the measuring script, `generator` (another version of the prototype's row generator, as a stand-in for project code changing under a running editor) and `slow` (section 3).
- A committed edit, undo, redo, a reconnect and a generator swap all drop a stage.

### The core

- `cmd::remove_record` (take an edit back) and `cmd::dismiss` (let an inbox item go). Nothing else.

### The editor (`editor/godot`)

- Right button held on a picked thing's house and moved: one staged move a frame; let go: `commit`; Esc: `drop`. A click without moving moves at once, as before.
- Frames the host sends on its own are taken between commands (`listen`, each frame) and wherever the editor is waiting for an answer.
- Content the host has not made yet stands as a plain box of its kind until it arrives.
- `I` lists what needs a look; the label shows the counts always.

---

## 2. Rules added

69. **What the editor is shown and what the document holds may differ by one staged transaction.** A stage is built from the committed world, is replaced whole by the next, and leaves nothing when dropped. Committing it is one edit, the same edit as doing the last step directly.
70. **A cell never waits for content.** Content asked for and not made is owed; it is sent when it is made, and its arrival changes nothing but content.
71. **What needs a look crosses the socket as the document and the world already hold it:** inbox items, resolutions that are flagged, the build's notes. Nothing is worked out for the editor that the core does not keep. Dealing with one is an edit like any other.

---

## 3. Measurements

Scripted editor run; times are command to drawn unless said. Two full runs agreed within the ranges shown.

| Things in the world | 21,000 | 105,000 | 420,000 | 105,000, all different |
|---|---|---|---|---|
| Drag, one staged step (cells to drawn) | 0.75–0.8 ms | 0.8–0.85 ms | 0.9 ms | 0.85–1.5 ms |
| of which core | 0.54 | 0.52–0.56 | 0.56–0.58 | 0.5–1.0 |
| Drag, the commit | 0.16–0.18 ms | 0.17–0.18 ms | 0.19 ms | 0.2–0.3 ms |
| Edit one window | 0.9 ms | 0.9 ms | 1.5 ms | 1.2–1.5 ms |
| Move one house | 1.0 ms | 1.0 ms | 1.1 ms | 1.1–1.5 ms |
| Turn one house | 0.8 ms | 0.7–0.75 ms | 0.75–0.8 ms | 0.75–1.2 ms |
| A village's wall height, cells drawn | 8.9 ms | 8.9 ms | 8.9 ms | 9.4 ms |
| Undo | 0.75 ms | 0.7 ms | 0.7 ms | 0.75 ms |
| First show (cells, with placeholders) | 0.19 s | 0.87 s | 3.6 s | 1.28 s |
| Reconnect | 42 ms | 207 ms | 858 ms | 260 ms |
| `review` | 0.6 ms | 0.6 ms | 0.7 ms | 0.6 ms |
| `upgrade_all` after a generator change | 0.22 s | 1.8 s | 20.3 s | 1.9 s |
| Idle frame, world in view | 1.6 ms | 11–13 ms | 23–24 ms | 11 ms |

Drag invariants, checked in every world: one undo step for sixty staged moves; a dropped drag of twenty steps leaves the editor byte for byte where it was, and the host agrees; the world a drag commits to is byte for byte the world of a direct move to the same place.

### Content that comes later

At this content weight (5 meshes for a world of houses alike, 401 when every village differs, ~90 triangles each), **content is made before the editor can ask for it.** In every edit of every run the host owed nothing by the time the editor asked: a wall-height change asks for 4–5 new pieces and gets them in the same exchange, 2 ms after the cells. The path for content that is owed and pushed is taken in two ways:

- The protocol test (`p02`) with no slowing: 25 of 25 pieces owed at the first ask, pushed in 1.1 ms, no cell changed.
- The load in the scripted run, with each piece slowed by 300 ms (`slow`, a knob for the script and nothing else): 5 of 5 pieces owed at first show in the three alike worlds, 401 of 401 in the different one, all pushed. First show to all content: 0.5 s at 21,000 things, 1.2 s at 105,000, 4.6 s at 420,000, 6.5 s for the 401 different pieces (that is the 300 ms, over 15 workers). `shot-owed.png` is the world as it stands between the two: placeholders.

The undo of a wall-height change asks for nothing: that content was made.

### The review

The scripted run swaps in a row generator that names houses differently and declares no migration, then upgrades every village.

- Every village is held and badged until upgraded: 20 / 100 / 400 notes.
- `check_upgrade` says "key epoch bumped, no migration".
- The upgrade rebuilds everything the villages own: 42,000 / 210,000 / 840,000 things changed, every cell sent again. 0.22 s, 1.8 s, 20.3 s. The editor waits for it.
- Afterwards 40 edits are flagged, all Rematched (found by fingerprint under the new name), none orphaned, inbox empty, notes gone. Accepting all 40 takes them off the list; the host agrees with the editor afterwards.
- `accept` is 0.5 / 2.3 / 17 ms of core time on average. The mean hides a first transaction that looks at every upgraded owner once more (1.5 / 3.5 / 11 owners built on average over the 40, which is 20/40 + 1, 100/40 + 1, 400/40 + 1): the first transaction after a build writes down which generators each node was built with, as `k01` says. The window edits after the load show the same (2.25 / 5.95 / 21 owners on average over 20). This was so before this pass.

### Picking

Unchanged: 20 of 20 roofs at every size; 35–37 of 40 windows, every miss the door standing in front of the middle south window.

### The core

Not re-measured: the core changed by two commands.

---

## 4. Findings

### Finding 1. The host still does one thing at a time (medium-high)

A 20-second upgrade of 400 villages is a 20-second wait with the editor frozen on its exchange. The same holds for a 3.6-second first show. Cells stream, but the editor waits for `DONE` before it does anything else, and the host is inside one transaction. Content is the only work that happens apart from transactions now. What a long transaction needs is for the host to answer questions and show progress while it runs, and for the editor to not block on an exchange. Neither is built. This is the "busy engine" of Pass 11, now with a number on it.

### Finding 2. An upgrade is a full rebuild of what it upgrades (observation)

840,000 things changed at 420,000 things: every thing was rebuilt twice over (the held version and the upgraded one, both in the transaction's delta). That is what an upgrade is; the point is that it costs the same as a first build and lands in Finding 1.

### Finding 3. Late content is not seen at this weight (observation)

Every piece in these worlds is made in less time than the editor takes to apply the cells that name it. The pushed path is tested, but a real project will have content that takes seconds; nothing here says how that feels. The placeholder is a plain box of the thing's kind, which for a window is a 1 m box in the air: crude, and the only choice made.

### Finding 4. What the tests cover and what they do not

Covered: stage shown and not kept; drop leaves nothing; a drag commits to the world of the direct move; a committed edit ends a stage; commit with nothing staged is an error and not a change; content asked for and not made is owed and pushed, with no cell changing; an edit that needs new content gets cells at once and content after; undo asks for nothing already made; the review lists flagged edits after an upgrade without migration; accept, forget and dismiss take them off the list and are undoable.

Not covered: a stage built before content arrives, then refreshed (the host does it; the core's `d05` covers the engine; no protocol test has the timing); an inbox with something in it (every edit in these scenes was rematched, none orphaned, so `dismiss` is tested only for its error path and `review` has only ever listed an empty inbox); a failed piece of content as the editor sees it (`FAILED` is sent; nothing in Godot shows it); a stage over a world with gardens reading villages (the linked scene is not in the run).

### Finding 5. Counts in every reply walk the world (low)

`flagged` and `badges` in every `DONE` walk every built owner's resolutions and notes: 0.6–0.7 ms at 400 villages, paid on every edit. Fine for now; it grows with owners, not with things.

### Finding 6. A generator swap starts history afresh (decision made, to confirm)

Earlier versions were built with code that is gone, so I clear undo and redo when the registry changes. The alternative, keeping them and letting the core hold them as stale, I did not try.

### Carried, unchanged

Plans run once per facade side; square-on boxes around turned things; three turn-aware paths untested; content never let go; kept bytes as hex; first show draws cells twice.

---

## 5. Remaining risks, by severity

1. **A busy engine (medium-high, was part of "protocol beyond this slice").** Finding 1. Long transactions freeze the editor; there is no progress, no cancel, no answering while building.
2. **Opening large projects (medium-high).** 3.6 s to first show at 420,000 things, and an upgrade costs the same again.
3. **Draw units in Godot (medium).** Unchanged: 11 ms at 105,000 things, 24 ms at 420,000. Next.
4. **Real content weight (medium).** Unchanged; Finding 3 adds that late content has not been felt.
5. **Things as Godot nodes (medium).** Colliders, scripts and gizmos: the one item of the Pass 11 list not touched by this slice.
6. **Turn-aware paths nothing uses (medium-low).** Unchanged.
7. **Memory where nothing repeats (medium).** Unchanged.
8. **The editor protocol as a shape (low, was medium-high).** Staging, late content and the review fit the protocol as it was: new commands and one new frame kind, no change to cells or content. I would not expect the shape to change for the two items left.

---

## 6. Decisions for you

1. **Confirm: a generator change clears undo and redo** (Finding 6).
2. **The placeholder for content not yet made:** a plain box of the thing's kind (now), nothing, or the last content the thing had under its previous name. I would keep the box until real content weight shows what a wait looks like.
3. **Order from here.** You set draw-unit work next. Finding 1 is the first thing a hand on the mouse would hit in a large project; I would still do draw units first, as agreed, since the frame cost is paid every frame and the stall only on upgrades and opens, but say if you want the busy engine first.
4. **Where the protocol tests live:** `editor/host/tests/protocol.rs`, beside the host. Say if you want them elsewhere.

---

## Appendix. Running it

```
cd core
cargo test                                   # store 4, bridge 1, source 1

cd core/acceptance
cargo test                                   # scenarios 28, nested 11, cross 9, deferred 9, native 9

cd editor/host
cargo test --release                         # protocol 3
cargo build --release

editor\run.ps1                               # the editor, by hand
editor\run.ps1 -Measure                      # section 3; writes editor\godot\results.json,
                                             # shot.png, shot-close.png, shot-turned.png, shot-owed.png
editor\run.ps1 -Measure -Quick               # the smallest world only
```
