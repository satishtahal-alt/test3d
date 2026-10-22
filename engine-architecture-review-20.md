# Engine architecture review — Pass 20

2026-10-07. Two things, in the order you set: the move-pin note (core, small), then content lifetime and bounded intake (adapter and host). One part of the second was not built and needs your decision (section 5).

## 0. Summary

- **Pin note: done.** A move or an added member that a layout does not follow is now said, on the owner and on the record. One new native test, `k12`. The frozen suite is unchanged and passes on both kernels.
- **Editor lets content go: done.** A mesh that no cell names any more is freed, with the draw units made from it. The editor now holds what the world names and nothing else, in every world.
- **Bounded intake: done.** Content is asked for a few names at a time, the answer is not waited for beyond what a frame can spare, and meshes are built under a per-frame budget. The 1.1–2.5 s frames of Pass 18 are now at most 0.11–0.15 s in the same world.
- **Cost:** everything being drawn takes longer to complete. About 2× in a kit world at 100 villages (1.4 s → 2.7 s), and 2.2× where no two parts are alike (21 s → 48 s).
- **Not built: letting go in the host's cache.** It depends on what undo history holds, which is a decision about undo, and it needs a small core change. It is a real growth path for a kit world: a hand on a slider adds about 4 MB a step.
- **New finding:** the host hashes every finished mesh on its main thread, 1.0 s in one stall in the heaviest world.

No rule is added. The pin note could be written as one (section 1) if you want it in the list.

## 1. The pin note

### What it does

A move is a pin, and a pin is given to project code. Until now, if that code did not follow it, the move was accepted, kept as a step of history, and nothing was said.

Now, after a scope is planned, each pin and each added member recorded on it is checked against the plan:

- A move is followed when the child it names stands where it was put.
- A pin with no name to go by (an added member, or a move whose child's name is retired) is followed when some child stands there.

One that was not followed gives:

- a note on the owner, listed with the other notes in `review` and counted in every answer. For example: `fence: 'post#b' was moved to (9.5, 3) and the layout left it at (4, 0)`, or `… and the layout made nothing there`;
- the same text as the note of that record's resolution.

### What it does not do

- The move is not refused, dropped, flagged or sent to the inbox. It is the person's edit and stays as written; its label stays `Applies`.
- It is not a mark. It is a fact about the world as built, like "approved content is no longer asked for by the layout": it is there for as long as the layout does not follow the pin, and gone when the layout follows it or the move is taken back. A mark would have outlived the fault.
- The engine does not judge why. A layout that snaps a move to a grid is reported with where it left the thing. That is accurate, and may be noise for a generator that snaps on purpose; no such generator exists yet.

### Scope

`core/src/build.rs`, in `expand`: about 25 lines. No change to the protocol, the document, the host or the editor: the host already sends notes, and the editor already shows them. The prototype kernel is not changed (new capability in the real core only, as with marks in Pass 17).

Cost is one lookup per pin in a scope that is walked; a scope that is reused replays its note. I did not measure it separately; the edit timings below include it.

### Tests

`k12_a_pin_the_layout_does_not_follow_is_said` in `core/acceptance/tests/native.rs`, with two generators that exist only in that test:

- a layout that never asks for its pins: the thing does not move, the note is on the owner and the record, the record is kept without a mark, the inbox is empty;
- the note is still there after a rebuild that changes nothing, after an edit elsewhere, and in a build of the same document from nothing (which also gives the same cells);
- an added member the layout makes nothing for is reported; one added to the row generator is not;
- taking the records back takes the notes away;
- a layout that puts a moved thing on the nearest whole metre is reported with where it is; a move it can follow exactly replaces the first and the note goes.

Suites after all of this pass's changes: frozen 57 on the core and 57 on the prototype kernel, native 12, core 6, protocol 7. All pass.

## 2. The editor lets content go

Each cell records the content it names, and the editor counts the cells naming each piece. When a count reaches zero the piece is let go the next time units are laid: its mesh, its entry, and its draw units wherever they were laid, including in regions not drawn at the moment. A cell replaced by one that names the same content lets nothing go. If a cell names the piece again later, it is asked for again; the host has it.

Content that arrives for something no cell names any more is dropped on arrival.

The host now stops owing content when no cell it has sent names it (the remainder noted in Pass 19, after a stopped build).

Measured, pieces of content the editor holds (after load / after edits / after the slider / after building everything again):

| world | Pass 19, after building again | now |
|---|---|---|
| kit, 100 villages | 606 | 355 / 603 / 652 / 655 |
| own parts, 20 villages | 7,620 | 7,303 / … / 7,624 |
| own parts, 100 villages | 52,893 (Pass 18) | 36,503 / 36,816 / 36,817 / 36,817 |

The editor's count now equals what the host says the world names, in every world and at every check.

Video memory where no two parts are alike, after building everything again: 2,651 MB → 1,732 MB at 100 villages, 625 → 374 MB at 20. In kit worlds it was never a problem and is unchanged.

Cost: going back needs what was let go. Undo where parts differ is 7.5 → 20 ms at 100 villages (4 → 14 ms at 20), because the old parts are fetched again. With a kit it is 4.2 → 4.5–5.3 ms. I kept no spare content for undo; this is the cost of not doing so.

A remainder: regions that are not drawn keep small "this key is out of date" marks for content that has since been let go, until they are next drawn. A few bytes each; not measured as a problem; listed in section 6.

## 3. Bounded intake

Three changes in the editor:

1. **Asking.** At most 8 names in a request, one request outstanding. The answer is waited for only as long as the frame can spare; if the host is busy it arrives later and is taken like any frame the host sends on its own account.
2. **Building.** A mesh that arrives waits in a queue and is built when its turn comes, under a time budget per frame. Nothing more is asked for while more than 8 MB waits.
3. **Drawing again.** Cells that were waiting for content are redrawn under the same budget, once for however many pieces arrived for them meanwhile.

The budget is 4 ms each for asking, building and redrawing, or an eighth of the current frame time if that is more. These are adapter defaults.

### Worst frame while content arrives

| world | Pass 18 | now |
|---|---|---|
| own parts, 100 villages, opening | 1,100–2,500 ms | 113 ms |
| own parts, 100 villages, reconnecting | not recorded | 136 ms |
| kit, 100 villages, opening | not recorded | 42–92 ms |
| kit, 20 villages, opening | 47–70 ms (this pass, before the change) | 27 ms |

In the own-parts world a frame with nothing happening already costs 84 ms from the whole view, so 113 ms is intake adding about a third.

The kit figure at 100 villages is 42 ms with a fixed 4 ms budget and 92 ms with the scaled one, one run each. I believe the longest frame there is cells landing, not content, but I did not separate them.

### Time until everything is drawn

| | Pass 19 | now |
|---|---|---|
| kit 100, opening | 1.4 s | 2.7 s |
| kit 100, reconnect | 3.7 s | 2.2 s |
| kit 100, regenerate a village (each of five) | 52 ms | 74 ms |
| own 100, opening | 21 s | 48 s |
| own 100, reconnect | 12 s | 61 s |
| own 100, everything built again | 14 s | 43 s |
| own 100, frame while that goes on | 716 ms | 201 ms |

This is the trade: frames stay short and the whole takes longer. Where no two parts are alike it is 2–5× longer. That world is the stress bound you set, so I accepted it; the budget is two numbers if you want it weighted the other way.

The exchange for a regeneration itself is shorter (kit 100: 52 → 25–40 ms; own 100: 142 → 59–80 ms), because it no longer waits for its content.

### Edits

Single edits and drag steps are 0.3–1 ms slower than Pass 19 in most worlds (kit 100 drag step 1.7 → 2.1 ms; kit 400 2.0 → 3.9 ms). Each world was run once, so I do not claim more than "somewhat slower"; the cause is the counting of names when a cell is replaced.

## 4. Findings

### 4.1 The host hashes finished content on its main thread (measured, new)

When workers finish content, the host takes it in on its main thread, and taking it in hashes every byte of every output, under the lock on the results. In total that is about 4.6 ms a megabyte: 1.2 s for the 243 MB of a kit session, 30 s for the own-parts session. The longest single stall was 70 ms with a kit at 20 villages, 408 ms at 100, and 1,020 ms with own parts.

Any request that arrives during a stall waits behind it. That is why asking in small rounds was not enough on its own and the editor had to stop waiting for answers. A drag step that arrives while content is being taken in still waits.

The fix is to hash on the worker that made the content. It changes a type in the core's job interface, so I did not do it unasked.

### 4.2 Every step of history names its old content (measured)

After a 40-step slider on one village with a kit: the host holds 2,324 pieces and 237 MB, of which the world names 652 and 71 MB. 161 MB is named only by undo history. After the generator swap clears history, 1–2 MB is named by history and the host still holds all 237 MB: about 165 MB is named by nothing at all.

In the own-parts world at 100 villages the host ends at 58,775 pieces and 4.9 GB, 5.8 GB of process, for a world that names 36,817.

### 4.3 The core assumes held content stays held (by reading, not measured)

A scope that is reused from the last build is not asked again whether its content is still held. If the host dropped content and a later edit brought a scope back to a state the core remembers, the world would name content that is gone and nothing would ask for it. Scatter tiles do check. So letting go in the host needs either the remembered scopes cleared when content is dropped (one cheap re-walk per owner afterwards) or a check on reuse.

## 5. Decisions

1. **What does undo history hold?** This decides how the host can let content go.
   - **(a) History holds its content, and history is bounded by memory.** When content named only by history passes a budget, the oldest steps are dropped. Undo is always immediate; undo depth shrinks under heavy edits. Needs a way to drop results in the core's cache (three small additive methods) and the reuse fix of 4.3. I recommend this: it is the common editor behaviour, it changes no world semantics, and it is the smaller change.
   - **(b) History holds no content; it is made again on going back.** Unlimited depth, but an undo of a heavy step shows placeholders while parts are remade, and the core must learn that a world can name content that is no longer held. That is a larger change to the core.
   - Under either, content named by nothing (the 165 MB above) can be dropped.
2. **Hash content on the worker?** (4.1) I recommend yes, as part of the same slice as decision 1, since both touch the core's results.
3. **Intake budget.** Keep "4 ms, or an eighth of the frame" as the adapter default? I recommend yes for now. It favours short frames over time to completion.
4. **The pin note as a rule.** If you want it in the list: "An instruction the engine passes to project code and project code does not follow is reported on the owner, for as long as it is not followed. It is never refused or dropped for that reason."

## 6. Not done, and limits of what was measured

- Host cache letting go: not built (decision 1).
- `results-20.json` is all nine worlds with the fixed 4 ms budget. The scaled budget was run afterwards on kit 100 and own 100 only (`results-20-real_kit_100.json`, `results-20-real_own_100.json`); figures marked "now" for those two worlds are from those runs. The other seven worlds were not re-run with the scaled budget.
- In the nine-world run, own 100 did not agree with the host after reconnect. Its reconnect took 120 s, which is the benchmark's waiting limit, so the check ran before everything had arrived. With the scaled budget it took 61 s and agreed. I take the first as the time limit, on that evidence only.
- One run per world. Differences under about 15% are not findings.
- Cells are still taken in all at once when a build lands or the editor reconnects (557 ms of one exchange at 400 villages on reconnect). Only content intake is bounded. Not addressed; no measurement yet says it must be.
- Marks left in undrawn regions for content let go (section 2).
- The pin check was not run against a turned scope or a pin below the top scope in a test; the code handles the path prefix, and `k12` covers the top scope only.

## Appendix: files changed

- `core/src/build.rs` — pin check in `expand`.
- `core/acceptance/tests/native.rs` — `k12` and its two generators.
- `editor/godot/world_view.gd` — `named`, `loose`, `drawn_at`, `_let_go`, `_unname`; `arrived`, `redo`, `build_some`; `take_wanted(most)`; `take_content` queues.
- `editor/godot/main.gd` — `fetch` in rounds without blocking, `_fetched`, `FETCHED` as a side frame, budget in `_process`.
- `editor/godot/measure.gd`, `weigh.gd` — worst frame, slider phase, new memory fields.
- `editor/host/src/lib.rs` — `owed` pruned in `sync`; `check` reports history-only content and time taking content in.
