# Engine architecture review — Pass 21

2026-10-07. The rest of the content-lifetime slice, as you decided it after Pass 20: undo history holds its content within a memory bound, the host lets go of content nothing names, and content is hashed by the worker that made it. The pin note is recorded as a rule.

## 0. Summary

- **Hashing is off the host's main thread.** The longest stall taking content in was 1,020 ms in the heaviest world; it is now 4 ms. With a kit it is 0.2–0.3 ms.
- **The host lets content go.** At the end of a session it holds what the world names, plus what history holds. Kit world at 100 villages: 2,324 pieces and 237 MB before, 659 and 76 MB now. Own parts at 100 villages: 58,775 pieces and 4.9 GB before, 37,119 and 2.8 GB now.
- **History holds its content within a bound**, 256 MB by default. The 40-step slider with a kit fits (161 MB, every step kept). Where no two parts are alike it does not: 17 steps are left of 137.
- **The fault I described from reading in Pass 20 was real**, is reproduced by a test, and is fixed (3.2).
- **A second fault turned up in measurement and is fixed:** content that arrived while the host answered a request was not pushed to the editor until more content arrived. It comes from Pass 20's small requests, not from letting go (5.1).
- **Costs.** A step that drops history costs about 3.5 ms more in the core. Process memory does not come back as far as the content does (5.2).
- Rules 82–84. Two new tests, `k13` and `p08`. All suites pass.

## 1. Decisions recorded

From your answer to Pass 20:

1. Undo history holds its content, within a memory bound. Over the bound the oldest history goes, with its content. Nothing is made again on going back, and there are no placeholders.
2. Content is hashed off the host's main thread, in this slice.
3. The intake budget stays: 4 ms, or an eighth of the frame.
4. The pin note is a rule (rule 82 below).

## 2. Rules added

82. **An instruction passed to project code that project code does not follow is said.** A move or an added member that a layout does not honour is reported on the owner and on the record, for as long as it is not followed. It is never refused, dropped or flagged for that reason: it is the person's edit and stays as written.
83. **Finished content may be let go only by whoever keeps the worlds, and only when none it keeps names it.** The core never lets content go on its own account. A world that comes to name content that was let go asks for it like content that was never made: nothing may take for granted that content it once saw is still held.
84. **A version that can be gone back to holds its content.** Going back or forward makes nothing and waits for nothing. History is bounded by the content it holds apart from the present world; over the bound the oldest steps go, each with the content only it named. A step that is gone cannot be gone back to; nothing is remade to pretend it can.

Rule 83 replaces a sentence in the core that was no longer true ("only ever added to"). Rule 84 is about the session, not the core: the core has no history. The size of the bound is the host's default, not a rule.

## 3. What was built

### 3.1 Hashing on the worker (core, `jobs.rs`)

A worker now hands back content already hashed (`Pool::take_done`), and the engine takes it in as it is (`Engine::take_in`, `Results::admit_done`). Taking a result in is an insert into a map.

`Pool::take` and `Engine::deliver` still exist with the types they had, because the frozen suite uses them. They do the old work on the calling thread.

### 3.2 Letting content go (core, `jobs.rs`, `build.rs`)

`Results` gained three methods and a counter:

- `let_go(names)` and `keep(named)` remove content: by name, or everything a test does not vouch for.
- `size_of(names)` and `bytes()` say how much is held. `bytes()` is now a count kept as content comes and goes, not a walk over everything.
- `times_let_go()` counts the times anything was removed.

The core calls none of the removing methods. They are for the host (rule 83).

The fault described in Pass 20 section 4.3 was real. `k13` reproduces it: with a remembered scope and its content let go, the world named content that nothing held and nothing asked for. The fix is the cheaper of the two I listed: the build notes `times_let_go()` beside its remembered scopes, and when the number has moved it forgets them all before building. Every owner that is then built walks its scopes once and asks for what they name. Plans stay remembered, so nothing is planned again. Its cost is measured in section 4.4.

### 3.3 History within a bound (host, `lib.rs`)

Each step of history now carries two lists: the content it names that the next step toward the present does not, and the reverse. They come from the transaction's own account of what it came to name and ceased to name, so keeping them costs nothing to compute.

From those the session keeps one set, `apart`: the content history names and the present world does not. It is updated on every commit, undo and redo by the two lists of the step concerned, never by walking a world.

- **The bound.** After a commit, while the bytes held under `apart` are over the bound, the oldest step is dropped. Its "only here" list is checked against every version still kept, and what none names is let go from the core's results. Default 256 MB (`HISTORY`); the command `history` sets it.
- **An edit after going back** drops the steps that could have been gone forward to, the same way.
- **A generator swap** clears history as before, and now lets go of everything history held.
- **Undo and redo** move names between the world and `apart`. Nothing is let go and nothing is made.

The bound counts content bytes. It does not count what a version itself holds (its cells and trees, shared with its neighbours). That is section 6, first point.

### 3.4 Content nothing names (host)

Some content is named by no version at all: what a staged step asked for before the next step replaced it, what a stopped build made, what arrived after its cells were withdrawn. This is not history and is not deterministic, so it is kept out of the bound.

It is looked for when the bytes held have grown by 64 MB or a quarter since the last look, and never while a transaction is in flight (that transaction may come to name anything). Everything is kept that the committed world, the staged world, what the editor is shown, or `apart` names.

### 3.5 Tests

- Native `k13_content_let_go_is_asked_for_again`: content named by one thing is let go when its house leaves the row; the house comes back from remembered scopes; the content is made again and the world is cell for cell what it was. Fails without the fix of 3.2. Also checks that the byte count follows, and that an answer is never replaced.
- Protocol `p08_history_holds_its_content_within_a_bound`: twelve wall-height steps, each leaving walls only history names; three staged steps dropped, leaving content nothing names. With the bound set to half of what history holds: the oldest steps go, what is held is exactly what the world and history name, the session's running set equals one counted from the versions, every remaining undo finds its content there, the next undo is refused, and a value whose content was let go is made again. The whole session is run twice and must leave the same history.

Suites: frozen 57 on the core and 57 on the prototype kernel, native 13, core 6, protocol 8. All pass. The prototype kernel is unchanged.

## 4. Measured

Same nine worlds and script as Pass 20, plus one run of the kit world with the bound set to 64 MB. One run per world. "Before" is Pass 20; for kit 100 and own 100 that is the scaled-budget run, for the others the fixed-budget run (see Pass 20 section 6). `results-21.json`, `results-21-kit_100-history64.json`.

In every world, at every check, the editor agrees with the host, nothing is owed, and the session's running count of what history holds equals a count made from the versions themselves.

### 4.1 Taking content in

| world | longest stall before | now |
|---|---|---|
| kit, 20 villages | 70 ms | 0.2 ms |
| kit, 100 villages | 408 ms | 0.3 ms (run on its own) |
| own parts, 100 villages | 1,020 ms | 4 ms |

In total over the own-parts session at 100 villages it was 30 s of the main thread. It is now 172 ms summed over all nine worlds.

### 4.2 What the host holds

Pieces and megabytes of finished content at the end of the session (after the edits, the slider, a generator swap and building everything again):

| world | before | now | the world names |
|---|---|---|---|
| kit, 20 villages | 2,418 / 243 MB | 647 / 74 MB | 633 |
| kit, 100 villages | 2,324 / 237 MB | 659 / 76 MB | 656 |
| kit, 400 villages | 2,397 / 241 MB | 657 / 76 MB | 648 |
| own parts, 20 villages | 17,618 / 1,564 MB | 7,772 / 595 MB | 7,622 |
| own parts, 100 villages | 58,775 / 4,928 MB | 37,119 / 2,815 MB | 36,819 |

What is left over the world's own count is a few pieces history names, and in the own-parts worlds 14–29 MB that nothing names and has not been looked for yet (3.4: the threshold was not reached).

### 4.3 History

Right after the 40-step slider, before history is cleared:

| world | bound | steps kept | steps dropped | history holds |
|---|---|---|---|---|
| kit, 100 villages | 256 MB | 137 | 0 | 161 MB |
| kit, 100 villages | 64 MB | 15 | 122 | 61 MB |
| own parts, 20 villages | 256 MB | 17 | 120 | 248 MB |
| own parts, 100 villages | 256 MB | 17 | 120 | 247 MB |

Where no two parts are alike, one slider step is about 7 MB of walls nothing else shares, and the hundred edits before it had already put 126 MB in history. So the bound is reached and the slider leaves 17 steps to go back through. With a kit the default bound holds everything this script does.

Undo itself is unchanged: 4.8 ms with a kit at 100 villages (4.5 before), 25 ms with own parts (23 before; that cost is the editor fetching what it let go, Pass 20).

### 4.4 Times

| | before | now |
|---|---|---|
| kit 100, opening until all drawn | 2.7 s | 1.9 s (1.7 and 2.2 in two other runs) |
| kit 100, reconnect | 2.2 s | 2.3 s |
| kit 100, drag step | 1.2–2.1 ms | 2.0 ms |
| kit 100, slider step, whole exchange | 24.8 ms | 30.6 ms |
| kit 100, slider step, core | 7.1 ms | 8.3 ms (7.4 and 7.5 in two other runs) |
| kit 100, slider step, core, bound 64 MB | — | 11.6 ms (10.9 in another run) |
| own 100, opening until all drawn | 48 s | 54 s |
| own 100, reconnect | 61 s | 56 s |
| own 100, everything built again | 43 s | 38 s |
| own 100, regenerate a village until drawn | 2.4 s | 1.9 s |
| own 100, worst frame while content arrives | 113 ms | 122 ms |

- **Dropping history costs about 3.5 ms a step in the core** (7.4–8.3 ms without, 10.9–11.6 ms with). That is the remembered scopes being forgotten each time content is let go (3.2): the village is walked again instead of taken from memory. It is paid only by steps that drop something.
- The slider exchange as a whole is about 5 ms slower than the Pass 20 scaled-budget run and 9 ms faster than the Pass 20 nine-world run. I cannot say which way it moved from single runs.
- Nothing else differs by more than run-to-run variation. Opening the own-parts world did not get faster, although 30 s of main-thread hashing went away: that time was not what it waited on. The editor's intake budget is.

## 5. Findings

### 5.1 Content taken in during a request was not pushed (measured, fixed)

In the first run of this pass the own-parts world at 20 villages waited 120 s after the slider, which is the script's limit, and did not agree with the host. The host owed the editor five pieces, held all five, and was not sending them.

Cause: the host takes in finished content in two places, every 5 ms on its own account and at the start of answering a request for content. Only the first then looked through what it owed. Content that happened to arrive in the second stayed owed until some later content arrived by the first. During a load something always does. After a small edit nothing may.

This is Pass 20's fault, not this pass's: asking eight names at a time made requests frequent enough for it to happen. It did not show there by luck. The fix is a flag in the host, set wherever content is taken in. After it the same phase takes 1.7 s and agrees, and nothing is owed at any check in any world. The first run is kept as `results-21-before-owed-fix.json`.

No test covers it. The window is a few milliseconds wide and I did not find a way to hit it on purpose without adding a knob to the host. I can add one if you want the case pinned.

### 5.2 Process memory does not follow the content down (measured)

Own parts at 100 villages: after everything is built again the host holds 2.8 GB of content and the process holds 5.0 GB (5.8 GB before). It was 3.5 GB before that step and 5.0 GB after it. While everything is being built again, the old world's content and the new are both held until the new world lands and the old is let go; I take the rise to be that, and the process keeping the memory afterwards, but I did not confirm it. At 400 villages with a kit the process is 1.6 GB after building again, as before; content there is 76 MB, so that is the worlds, not content.

So the bound is on content held, and the high-water mark of the process is set by the largest single step. I did not look into returning memory to the system.

### 5.3 A request withdrawn before it was made leaves history naming content that was never made (by reading)

If an edit is replaced by the next before its content was made, the version in history names content that does not exist. Going back to it asks for that content then, and the editor shows the placeholder until it comes. That was already so before this pass. It is the one case where going back is not immediate. `p08` settles each step before the next, so it does not exercise it. Section 7, decision 2.

## 6. Not done, and limits

- **The bound counts content bytes only.** A version also holds what differs in its cells and trees. Ten thousand moves with a kit would hold ten thousand versions and no content apart from the world. I did not measure what a version costs; there is no limit on the number of steps.
- **Nothing tells the person that history was dropped.** The undo count in the status goes down, and undo is refused at the end. No note is given.
- **Looking for content nothing names** (3.4) is exercised by `p08` only. In the benchmark the threshold was never reached, so 14–29 MB of it is still held at the end in the own-parts worlds.
- **The table of requests asked and not answered** is not cleared for requests that were withdrawn. It holds a request's values, not content. Not measured.
- **A single step bigger than the bound** empties history, including that step. Building everything again in the own-parts world would do so; in the script it follows a generator swap, which clears history anyway.
- One run per world; differences under about 15% are not findings. The 64 MB run is the kit world at 100 villages only.
- 5.1 has no test.

## 7. Decisions

1. **Confirm rules 82–84**, in particular 83: the core never lets content go, the host does, and a world that names it again asks again.
2. **Content withdrawn before it was made (5.3).** Either (a) leave it: going back to such a step waits for its content, as now; or (b) the host keeps making what a kept version names, so that history really holds it. I recommend (a) until it is seen to matter: (b) spends workers during a drag on content nobody is looking at.
3. **The default bound, 256 MB.** It holds everything the script does with a kit, and 17 slider steps where no two parts are alike. It is a host default; say if you want another number, or a bound on steps as well (section 6, first point).
4. **What next.** Candidates, in the order I would take them:
   - tell the person when history is dropped (small, adapter and host);
   - cells taken in all at once on reconnect and landing (557 ms of one exchange at 400 villages, Pass 20), the last unbounded intake;
   - a knob and a test for 5.1.

   Far stand-ins and per-instance tint stay deferred as you decided.

## Appendix: files changed

- `core/src/jobs.rs` — `Results` as a struct with `admit_done`, `let_go`, `keep`, `size_of`, `times_let_go` and a running byte count; `Blob::size`; workers hash; `Pool::take_done`.
- `core/src/engine.rs` — `Engine::take_in`; `deliver` goes through it.
- `core/src/eval.rs`, `core/src/build.rs` — remembered scopes forgotten when content has been let go.
- `core/acceptance/tests/native.rs` — `k13`.
- `editor/host/src/lib.rs` — `Version`, `apart`, `HISTORY`, `trim`, `release`, `sweep`, `forget`, command `history`; `check` reports them; `arrived` flag (5.1).
- `editor/host/tests/protocol.rs` — `p08`.
- `editor/godot/weigh.gd`, `editor/run.ps1` — new memory fields, `-History`.
