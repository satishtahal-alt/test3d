# Engine architecture review 16

Pass 16: the busy engine. A transaction that outlasts its exchange, a build that can be stopped, an editor that keeps drawing, and a world that stays coherent whatever ends when.

---

## 0. Outcome

The editor no longer freezes on anything the host does. A transaction runs on a thread of its own; the host answers within a budget of 20 ms whether it is done or not, and goes on answering everything that reads, and `stop`, while it runs. The cells of a transaction that outlasted its answer come when it lands, with a `FINISHED` frame. The editor drew frames at its idle cost while the host built the largest world and while it upgraded it.

| Things in the world | 21,000 | 105,000 | 420,000 | 105,000, all different |
|---|---|---|---|---|
| Load: editor blocked for | 27 ms | 53 ms | 121 ms | 513 ms |
| Load: frame while the host builds | 2.2 ms | 0.5 ms | 1.0 ms | 3.9 ms |
| Load: cells drawn after (Pass 15, blocked throughout) | 0.17 s (0.20) | 1.07 s (0.90) | 3.6 s (3.5) | 1.3 s (1.4) |
| Upgrade: editor blocked for | 33 ms | 28 ms | 36 ms | 30 ms |
| Upgrade: frame while the host builds | 0.57 ms | 0.54 ms | 2.7 ms | 0.56 ms |
| Upgrade: edit meanwhile refused as busy | yes | yes | yes | yes |
| Upgrade: stopped, after being asked | 9.6 ms | 11.9 ms | 24.8 ms | 10.6 ms |
| Upgrade: world unchanged after the stop | yes | yes | yes | yes |
| Upgrade: run through (Pass 15) | 0.31 s (0.23) | 2.5 s (1.9) | 26.8 s (24.8) | 3.4 s (2.5) |
| Results landing on a state they had left | 0 | 0 | 0 | 0 |

Of the time the editor is blocked for a load, everything beyond the first 25 ms or so (the 121 ms at 420,000 things, the 513 ms in the last column) is the editor emptying the world it showed before, which used to be inside "first show"; the host's answer itself comes within the 20 ms budget.

Cancellation is a correctness rule, not a button. A build looks for the request between owners and ends with the world it started from; a stopped transaction leaves nothing behind but saved work. A staged transaction in flight is superseded by whatever comes next, stopped and discarded, as the next staged edit always replaced the last. While a committed transaction runs, anything that would change the committed state is refused as busy. A result lands only on the state it was started from; the host counts results that came for any other state, and the count was zero in every run.

All suites pass: core 6, the prototype's 57 against the core and in the prototype, 9 native, 5 protocol (2 new). Every agreement check and every drag invariant held in all four worlds. No frozen test changed.

---

## 1. What changed

### The core (about 60 lines)

- `build::Watch`: a handle on a running build. It is told to stop, and it says how far the build is (owners built, owners still queued). `build_until` looks at it between owners; told to stop, it ends with `Outcome::stopped` and the world it started from. `build` is `build_until` with a watch nobody holds.
- `engine::stage_until`: `stage` with a watch; a stopped transaction ends in the error `STOPPED`, with nothing changed.
- `Engine::start`: a transaction on a thread of its own, built from copies of the document, the registry, the cache handle and the world, so that the engine goes on answering while it runs. It hands back a `Working`: `wait(budget)` for its result, `stop()`, and dropping it discards whatever it ends in.
- `Generator: Send + Sync`, and the registry holds its generators as shared pointers and is `Clone`: a build holds the code it was started with, whatever is set afterwards. No generator changed; the frozen `gens.rs` builds the registry with an empty map literal and did not notice.

### The host (`editor/host/src/lib.rs`, about 150 lines)

- Every edit goes through one `edit`: start the transaction, wait for it up to the budget, land it if it ended, otherwise answer with `working`. `scene`, `generator` and `upgrade_all` go through it too; nothing builds on the host's own thread any more except `check_upgrade`.
- One `Flight` at most: its serial, its op, whether it is staged, the committed-state version it left, and whether a commit is to follow it. `land` refuses a result for another version (`late`, counted), commits or stages the rest, and refreshes it with content that arrived meanwhile.
- The rules in `command`: reads (`status`, `review`, `check`, `nodes`, `stop`, the knobs) go ahead beside anything; `commit` beside a staged flight marks it to be committed when it lands; anything else beside a staged flight supersedes it; anything else beside a committed flight is refused as `busy: 'op' has been running for N s`.
- `finish`: lands a transaction that ended since the last frame, syncs its cells, sends `FINISHED` (kind 23) with everything a `DONE` says plus `serial`, `op`, `stopped`. It runs before every frame is handled and on every turn of the serve loop, so an answer is always about the state after whatever had landed.
- Every answer carries `working`: null, or the flight's serial, op, elapsed time and progress.
- Knobs for the measuring script and the tests: `budget` (ms; 0 sends every transaction down the late path) and `slow` gained `plan_ms` (a house takes this long to plan, so a build can be watched in a world small enough for a test). A generator swap's rebuild is no longer a step of history, as before.

### The editor (`editor/godot`, about 80 lines)

- One `_side` for frames the host sends on its own account (names, cells, content, `MADE`, `FINISHED`), called from every loop that waits on the socket and from `listen` each frame. Cells of a transaction that outlasted its answer collect in `_later` and are laid once, at `FINISHED`.
- `working` is what the last answer said the host is doing; while it is not empty the editor asks `status` four times a second, shows op, elapsed time and progress in the label, and Esc sends `stop`.
- The measuring script waits for a working command with `finish()`, drawing frames, and measures the frame while the host loads and while it upgrades; it starts an upgrade, refuses an edit, stops it, checks the world is what it was, then runs it through.

### Tests (`editor/host/tests/protocol.rs`, 2 new)

- `p04`: with budget 0 a move is answered first and lands after; the answer shows nothing of it; what lands is the same cells and the same one step of history as the move within the budget; a drag of ten staged steps answered before they are built supersedes nine of them, the commit waits for the last, and the drag is the move it ended at.
- `p02` and `p03` now take a transaction's result from `FINISHED` when the host answered before it was done. The tests run beside one another in one process, and the knobs that slow a build (`slow`) are the process's, so a scene or an upgrade in one test may outlast its exchange because of another; by rule 74 the test must hold either way, and does.
- `p05`: with slow planning a new scene is watched (`status` shows op and progress), an edit beside it is refused as busy and undo too, a read goes ahead; stopped, it ends in about 100 ms with nothing built, and the same scene built again is the same world; four staged edits of 160 ms each supersede one another, the commit waits for the last, what is committed is that step, superseded transactions are never reported as finished, and no result ever came late.

---

## 2. Rules added

74. **A transaction is answered within the exchange budget, done or not.** Done, it is answered with its cells, as always. Not done, the answer says what is in flight, and its cells and `FINISHED` come when it ends. The budget is a measured number: with a budget of zero every transaction takes the second path and the world is the same.
75. **At most one transaction is in flight, and whether it is staged decides what may happen beside it.** Reads and `stop` go ahead beside anything. A staged transaction is superseded by whatever else comes: stopped and discarded, as the next staged edit always replaced the last; a commit meanwhile applies to it when it lands. While a committed transaction runs, anything that would change the committed state is refused as busy, and the refusal says what is running.
76. **A result lands only on the state it left.** The committed state has a version; a transaction remembers the one it was started from and is discarded for any other. Rule 75 makes that never happen; the host counts it anyway and reports the count. Content arriving is not a change of state: a landed result is refreshed with it.
77. **Stopping is a request a build honours between owners, and a stopped build leaves nothing behind but saved work.** It ends with the world it started from. What it wrote to the memo is checked before it is trusted, as everything in the memo is; requests it made for content are answered or dropped by whoever next wants them.
78. **A build holds the code it was started with.** Generators are read-only and shared; replacing one is a change of the committed state, refused while a committed build runs, and a superseded staged build goes on with the generators it had until it stops.

---

## 3. Measurements

Scripted run (`editor\run.ps1 -Measure`), the editor drawing uncapped as in every pass before. `editor\godot\results-16.json`.

### The busy engine

The table in section 0. Progress when the stop was asked for, about 35–60 ms in: 4 of 20, 4 of 100, 7 of 400 and 3 of 100 owners built. A stop lands within one owner's build: 10–12 ms in the three smaller worlds, 25 ms at 420,000 things, where an owner is about 30 ms. After every stop the editor's fingerprint of its cells was what it was before the upgrade began, and the host agreed with it cell for cell.

### Everything else, with the transaction on its own thread

| | 21,000 | 105,000 | 420,000 | all different |
|---|---|---|---|---|
| Drag step, cells to drawn (Pass 15) | 1.18 ms (0.84) | 1.45 (0.99) | 1.44 (1.07) | 1.93 (1.04) |
| of which core (Pass 15) | 0.76 (0.51) | 0.84 (0.53) | 0.88 (0.58) | 1.24 (0.59) |
| Commit | 0.24 | 0.38 | 0.22 | 0.25 |
| Edit / move / turn | 0.9 / 1.9 / 1.0 | 1.9 / 1.8 / 1.1 | 1.5 / 2.1 / 1.9 | 1.8 / 2.0 / 1.5 |
| Wall height, 5 houses (Pass 15) | 13.1 (10.4) | 14.5 (8.9) | 13.8 (9.1) | 12.4 (11.0) |
| Undo | 0.9 | 1.5 | 0.9 | 0.8 |
| Reconnect | 44 ms | 178 | 765 | 280 |
| `accept`, each | 1.1 | 1.4 | 1.5 | 1.3 |
| Idle frame | 0.64 | 2.40 | 2.90 | 1.96 |
| Load, host side (Pass 15) | 0.16 s (0.13) | 1.04 s (0.63) | 3.5 s (2.5) | 0.85 s (1.15) |

All agreement checks held after load, edits, review and reconnect; every drag invariant held; 20 of 20 roofs picked, windows as before. Every command the small world has besides load, generator swap and upgrade landed within its exchange.

### Where the time went

Three things are slower than in Pass 15, and they have two causes.

- **A thread per transaction: about a quarter of a millisecond.** The drag step's core time rose by 0.25–0.3 ms in three worlds (0.65 in the fourth, which also had the slowest commit). That is the spawn, four handle clones and a channel.
- **The editor drawing beside the build.** Loads at 105,000 and 420,000 things took 1.4–1.7 times the host time of Pass 15; the upgrades 1.1–1.4; the wall-height edits, which make new content on the workers while the editor draws, 1.3–1.6. In this process alone, with nothing drawing, a 100-village scene costs the same on its own thread as on the calling one: 560–630 ms over ten rounds in both orders (`cargo run --release --example thread`). The editor in the measuring run draws uncapped, at 300–1,500 frames a second, and now does so while the host builds; before, it was blocked and cost nothing.

The same run with the editor capped at 60 frames a second (`editor\run.ps1 -Measure -Capped`, `results-16-capped.json`):

| | 21,000 | 105,000 | 420,000 | all different |
|---|---|---|---|---|
| Load, host side: capped / uncapped / Pass 15 | 0.15 / 0.16 / 0.13 s | 0.78 / 1.04 / 0.63 | 2.8 / 3.5 / 2.5 | 0.88 / 0.85 / 1.15 |
| Upgrade, host side: capped / uncapped / Pass 15 | 0.13 / 0.28 / 0.21 s | 1.8 / 2.4 / 1.8 | 25.2 / 26.7 / 24.3 | 2.3 / 3.4 / 2.4 |
| Drag step core: capped / uncapped / Pass 15 | 0.63 / 0.76 / 0.51 ms | 0.66 / 0.84 / 0.53 | 1.04 / 0.88 / 0.58 | 0.97 / 1.24 / 0.59 |
| Wall height: capped / uncapped / Pass 15 | 10.0 / 13.1 / 10.4 ms | 9.7 / 14.5 / 8.9 | 12.5 / 13.8 / 9.1 | 11.7 / 12.4 / 11.0 |
| Stop lands after | 49 ms | 49 ms | 49 ms | 49 ms |

Capped, the upgrades cost what they cost in Pass 15 and the loads 1.1–1.25 times it; the rest of the difference was the editor. The thread itself is the 0.1–0.4 ms on a drag step. One thing the capped run shows that the uncapped one hid: a stop lands in 49 ms everywhere, because the editor looks at the socket once a frame and asks for progress between frames, so the stop is seen, sent and its end noticed on the editor's cadence, not the host's. The host's part is the 10–25 ms above.

---

## 4. Findings

### Finding 1. The transaction model needed no change (observation)

Stage, commit and drop were already copies and swaps. Running the build on another thread was cloning four handles and sending the result back; landing it was the commit that already existed. The whole slice is about 60 lines in the core, 150 in the host and 80 in the editor, and no frozen test moved. The thing that made this cheap was decided in Pass 9: a world is never changed, a transaction is the next one.

### Finding 2. Supersede, not queue, is the right rule for staged edits (confirmed)

Ten drag steps answered before they were built: nine superseded, the commit waited for the tenth, and the drag committed to the move it ended at, exactly as a synchronous drag does. Nothing had to remember a queue or a basis beyond the committed version. A superseded build releases the shared memo lock within one owner, which is how long the next step waits to start.

### Finding 3. Stopping is as fine as an owner, and no finer (observation, risk)

At 420,000 things an owner is about 30 ms of build and a stop lands in 25 ms. With real content an owner will be heavier, and the stop will take as long as the heaviest owner. A finer stop means looking at the watch inside `expand`, per scope; that is one line, but a scope that stops must hand back `None` like a failed plan does, which the owner loop already treats as nothing built. Not done; nothing today needs it.

### Finding 4. A build costs the same on its own thread, and the editor drawing beside it is not free (measured)

In this process, with nothing else running, a 100-village scene built on the calling thread or on a thread of its own costs the same (560–630 ms either way, over ten rounds in both orders). In the editor run the loads at 105,000 and 420,000 things took 1.4–1.7 times Pass 15's host time, because the editor is no longer blocked while they run: it draws at hundreds of frames a second, uncapped, and takes cores from the host. Section 3 has the same run with the editor capped at 60 frames a second, as an editor on a screen is.

### Finding 5. The late path is the ordinary path with a budget of zero (confirmed)

`p04` runs the same move both ways and gets the same cells and the same one step of history. The protocol has one more frame and one more field, not a second way of doing things. Every frame loop on the editor side now takes the host's own frames (`_side`) wherever it is waiting, which also closes a gap Pass 14 left: content owed from before could already arrive in the middle of a `DESCRIBE`, and cells now can too.

### Finding 6. What still runs on the host's thread (observation)

`check_upgrade` is a build and runs on the host's thread, refused while another build runs but itself blocking for its length (it is a dry run of one upgrade, so one owner and its readers). A generator swap first finishes all content still being made, synchronously, before the rebuild goes to its thread; in the runs nothing was outstanding. `undo`, `redo`, `drop`, `HELLO` and the cell sync are not builds and stay where they are. A panicking generator on the transaction thread now ends as an error (`the transaction ended without a result`) instead of ending the host; not exercised by a test.

### Finding 7. A thread per transaction costs a tenth to a third of a millisecond (measured)

A drag step's core time went from 0.5–0.6 ms to 0.63–0.66 ms with the editor capped and 0.76–0.88 uncapped, in the two smaller worlds: spawning the thread, cloning four handles and the channel. Noticeable in the number, not to a hand on the mouse. A long-lived transaction thread would take it back for perhaps forty lines; not done.

### Carried, unchanged

Flags as persistent marks (decided after Pass 15, not yet built: the next protocol slice); plans run once per facade side; square-on boxes around turned things; three turn-aware paths untested; content never let go; kept bytes as hex; counts per reply walk the owners; non-empty inbox and `FAILED` content untested on the editor side; cells going away through the region path untested.

---

## 5. Remaining risks, by severity

1. **Opening large projects (medium-high).** The editor now draws while a project loads, but it shows nothing of it until the build lands: 4 s of empty world at 420,000 things. Showing a project as it is built (cells landing owner by owner) would need the store transaction to be split, which it is not.
2. **Real content weight (medium).** Unchanged, and Finding 3 adds: the stop is as slow as the heaviest owner.
3. **Things as Godot nodes (medium).** Unchanged.
4. **Memory where nothing repeats (medium).** Unchanged.
5. **Flags that do not survive a build (medium-low).** Decided, not yet built.
6. **Turn-aware paths nothing uses (medium-low).** Unchanged.
7. **A busy engine (low, was medium-high).** The editor does not freeze; a long transaction can be watched and stopped; the world stays coherent under supersede, stop and late results, with the checks in `p04`/`p05`. What remains is Finding 3 and Finding 6.
8. **Draw units in Godot (low).** Unchanged.

---

## 6. What becomes necessary for the eventual engine

What this slice shows is enough for an editor on one machine; what it leaves for later, in the order it is likely to be needed:

- **A stop inside an owner**, when owners get heavy (Finding 3). Per scope is one line in `expand`; inside a generator's plan it needs the generator's cooperation through its context, which is the only place a generator sees the outside anyway.
- **Showing a build as it lands**, if opening large projects stays the top risk after real content: owners finishing in order, each put to the store as it does, with the editor sent cells along the way. That is a store transaction per owner instead of per build and a world that is "so far"; rule 73 (a world stands for the document as the transaction leaves it) would need a word about the in-between.
- **One long-lived transaction thread** instead of one per transaction, if the quarter millisecond starts to matter (Finding 7).
- **Nothing about queues.** A committed edit while a committed transaction runs is refused, and the refusal says what is running and for how long. Until a case shows that a queue is what a person wants there, this is the simpler and the more honest answer.

---

## 7. Decisions for you

1. **Confirm rules 74–78**, and in particular the shape of rule 75: a staged transaction in flight is superseded by anything but a read, a committed one refuses anything but a read and `stop`. The alternative, queueing committed edits behind a long one, I recommend against until a case asks for it.
2. **The budget of 20 ms.** A frame of the editor's is about 16 ms; with 20 the drag, every edit and the small world's commands land within their exchange, and only loads, upgrades and generator swaps take the late path. Say if you want it shorter, so that nothing ever blocks the editor for more than a frame.
3. **Stop granularity.** Leave it at the owner (recommended), or take the one-line per-scope check now.
4. **Next.** The agreed order is complete. By severity, the next is opening large projects (risk 1), which is either the progressive show above or accepting that a project opens in a few seconds and spending the pass on real content weight instead. Before either, the flag marks decided after Pass 15 are a small protocol slice I would do first, since they are decided and the review list is otherwise not a list.

---

## Appendix. Running it

```
cd core
cargo test                                   # store 4, bridge 1, source 1

cd core/acceptance
cargo test                                   # scenarios 28, nested 11, cross 9, deferred 9, native 9

cd prototypes/identity
cargo test                                   # the same 57 in the prototype kernel

cd editor/host
cargo test --release -- --nocapture          # protocol 5
cargo build --release

editor\run.ps1                               # the editor, by hand; Esc stops what the host is doing
editor\run.ps1 -Measure                      # section 3; writes editor\godot\results.json
editor\run.ps1 -Measure -Quick               # the smallest world only
editor\run.ps1 -Measure -Capped              # the same with the editor drawing 60 frames a second

cd editor/host
cargo run --release --example thread 100     # a build on the calling thread against one on its own
```

`editor\godot\results-16.json` is the run in section 3; `results-16-capped.json` the capped one.
