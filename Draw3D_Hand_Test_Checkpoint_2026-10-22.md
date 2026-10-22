# Draw3D: hand-test checkpoint results sheet (2026-10-22)

**Purpose.** Hand-check the behaviour and gameplay work, plus a few outstanding checks, before the next milestone starts. This is step 0 of the sequence in [Draw3D_Roadmap_Progress_and_Next_Milestone_2026-10-22.md](Draw3D_Roadmap_Progress_and_Next_Milestone_2026-10-22.md).

**Rules.**
- Test the version named in section 0.
- Keep every uncommitted change.
- Change nothing during testing, unless Claude has named a specific blocker and we have agreed it.

**For every test, record:** Passed / Failed / Blocked / Not tested, and the first thing that differed.

**Before you test.** Answer the questions in **section A** first. **A1** decides what sections 4 and 1 can show.

---

## How to launch

Every test has a launcher in [checkpoint/](checkpoint/). **Double-click it in File Explorer:** `E:\AI apps\Draw3d\checkpoint\`. Each launcher runs `editor\run.ps1` with the options shown and keeps its console open when the run ends.

| Test | Launcher | Same thing from a PowerShell prompt in `E:\AI apps\Draw3d` |
|---|---|---|
| 0 | [0-editor.cmd](checkpoint/0-editor.cmd) | `editor\run.ps1` |
| 1 | [1-behaviours.cmd](checkpoint/1-behaviours.cmd) | `editor\run.ps1 -Demo behave` |
| 2 | [2-rules1.cmd](checkpoint/2-rules1.cmd) | `editor\run.ps1 -Demo rules1` |
| 3 | [3-guard.cmd](checkpoint/3-guard.cmd) | `editor\run.ps1 -Demo guard` |
| 4 | [4a-work.cmd](checkpoint/4a-work.cmd), [4b-talk.cmd](checkpoint/4b-talk.cmd) | `editor\run.ps1 -Demo work`, `editor\run.ps1 -Demo talk` |
| 5 | [5-village.cmd](checkpoint/5-village.cmd) | `editor\run.ps1 -Demo village` |
| 6 | [6-items.cmd](checkpoint/6-items.cmd) | `editor\run.ps1 -Demo items` |
| 7 | [7-cuts.cmd](checkpoint/7-cuts.cmd) | `editor\run.ps1 -Demo cuts` |

**How a demo runs.**
1. It leads the walker through its beats and puts a caption on each. It works inside its own window, not with your mouse.
2. After the last beat, **it is yours**: walk, open windows and try things by hand. The hand part matters most.
3. Close the window when you are done.
4. Each demo starts from a fresh playground, so your own world is not touched.

**Tours are not hand tests.** `-Tour -Only work`, `talk`, `village` and `items` are scripted checks. They drive the editor themselves and write a result file. Claude runs them at close-out (section C). You don't need to run them, but you may watch one if you like.

**Controls you will need.**

| Control | Keyboard | Controller |
|---|---|---|
| Play / stop playing | Play on the bar, or Tab | — |
| Act (do what is offered: open, pick up, talk) | **E** | **Y** |
| Attack | left mouse | X or right trigger |
| Behaviours and rules window | Ctrl+B | — |
| Inventory | I | — |
| Menu | Esc | — |

---

## Section 0. The starting version (answered by Claude)

**Branch and working copy.**
- Branch `m2-stable-terrain` in `E:\AI apps\Draw3d`, at commit `ced06f6` (the guard, 7di).
- Launch from this folder as it stands. Nothing has been stashed or reverted.
- The repository has **no git remote**, and this branch is not `main`. The GitHub links in your draft (satishtahal-alt/test3d, `main`) may show older documents, or none: the newest assessment is uncommitted. Use the local links in this sheet.

**Uncommitted changes that matter here.**

| File | Change | Effect on this checkpoint |
|---|---|---|
| `editor/godot/characters.txt` | The ranger's `moves walker` became **`moves samurai`**. The scout look gained `dash_cut/gather dash_gather`. | **Yes.** See A1. |
| `editor/godot/feel.cfg` | `walk_speed` went from 4.4 to **3.2**. | The walker is slower in every demo. Captions and waits may run ahead of it. |
| `editor/godot/weather.txt` | **188 lines removed and 12 added.** Gone: the file's whole explanatory header and the `falling hail` and `falling leaves` definitions. Wind strength went from 2.5 to 1.0, and `season autumn` was added. | Not in this checkpoint, but see A3. It looks like a window rewrote the file rather than a hand edit. |
| `editor/godot/styles.txt` | Your own style `my_lookout`. | None. |
| `editor/godot/modelling.txt` | A model `gable`. | None. |
| `editor/genres/watchtower.json` | The watchtower world re-saved. | None. |

**The ranger's `moves samurai` entry.** It is there, as shown above, and it does affect the tours:
- The samurai's move set has no `reach`, `wave` or `pet`.
- An act that needs a reach (talk, buy, open with a key) is therefore never offered to a walker playing the ranger.
- With your folder as it stands, `-Tour -Only work` and `-Tour -Only talk` fail: nothing at all is offered to the walker.
- With `characters.txt` and `feel.cfg` as committed, the same tours pass. This was recorded in PROGRESS for 7dg.
- Whether the entry is intentional is your call: question **A1**.

**Already known before you start** (recorded in PROGRESS; not new findings):
- The **`village` tour** fails at the committed code, for two reasons:
  - no door offers "open" to the walker;
  - a later beat cannot find Makers' "Walls inside…" button.

  Its repair is planned inside G4, the action model.
- The **`items` tour** fails at the committed code: the lantern never reaches the hand (`tour.gd:2013`). Its repair is planned inside G6.
- The **door's "try"** in the Behaviours demo needs no move, so it works even with the samurai set.

Status: answered above. Your questions: section A.

---

## Section A. Questions that need your reply before the next work

**A1. The ranger's `moves samurai`. Is it intentional?** Pick one:
- (a) **Keep it, and Claude gives the samurai set `reach`, `wave` and `pet`,** so talk, buy, pet and keyed doors are offered while you play the ranger. This is content only, in `moves.txt`, after the checkpoint.
- (b) **Revert it to `moves walker`.** Claude would keep a copy of your line in PROGRESS.
- (c) **Keep it as it is.** `work` and `talk` then stay failing by design, and their tours are rewritten to use another character.

If you want section 4 tested with offers working **now**, say "(b) for testing". Claude will change that one line before you test, keep a backup of the file, and put it back afterwards if you ask.

**A2. `walk_speed` 3.2 (it was 4.4).** Is it intended for every world? Pick one:
- (a) keep it;
- (b) put back 4.4;
- (c) a per-world or per-character speed later.

**A3. `weather.txt`.** Was the removal of its explanatory header and of the hail and falling-leaves weathers intended? If not, the likely cause is the Environment or season corner writing the whole file back. That would be a **defect**: Claude would restore the lost lines from git and find which window rewrote it.

**A4. If a gameplay test fails here, what happens next?** Pick one:
- (a) Claude mends only confirmed blockers for this checkpoint, then runs G3 (the recommended choice);
- (b) Claude mends everything you find before G3.

**A5. The order after the checkpoint.** Is G3 → PW0 → G4 → PW1 → G6 → G5 accepted? Or should PW0 (the saved-game repairs: renamed ids followed on load, orphaned facts listed, the guard's last-noticed place saved) come before G3?

**A6. The `village` and `items` tours.** Is it acceptable that they stay failing until G4 and G6? Or do you want them mended, or set aside with a clear "known, assigned to G4 / G6" label, before G3?

---

## 1. Editor: the Behaviours and rules window

Refresher: [Gameplay roadmap, G2](Draw3D_Gameplay_Roadmap_2026-10-21.md). Launcher: [1-behaviours.cmd](checkpoint/1-behaviours.cmd) (`-Demo behave`).

**What the demo shows**:
1. The world's behaviours are read, and mistakes in them are named by line.
2. A door's behaviour `my_door` is made in the window **without typing a line**.
3. That step is undone and done again.
4. It is given to one door; deer, fox and two doors are given theirs together.
5. A step's move opens in the Studio, and you return.
6. A behaviour is pasted in from text.
7. **The door is played:** tried while locked ("It is locked."), unlocked with its key, then it opens and shuts itself after 3 s.
8. The game is saved and loaded with the door unlocked.
9. What cannot behave is said.
10. The deer watches you and runs; the fox sees you and runs.
11. 200 behaving things are measured.
12. The world stops.

**By hand, after the last beat:**
1. **Ctrl+B** opens Behaviours and rules.
2. Click a **door** in the view. The window shows that door's behaviour.
3. Click through its states (shut / open) and their steps. Is each one understandable in words?
4. Change one choice, for example how long it stays open. Ctrl+Z, then Ctrl+Y.
5. Play (Tab). Walk to the door and press **E / Y**. It should say it is locked, or open.
6. Walk toward the deer: it should stop and face you, then run when you are close.

**Checks:**
- [ ] The window opens; no blank panel or error.
- [ ] What each behaviour does is understandable without reading a file.
- [ ] I can select entries and edit one choice; undo and redo work.
- [ ] The door answers as its behaviour says.
- [ ] The deer and the fox behave as described.

Status: ______ Notes: ___________________________________________

---

## 2. `rules1`: shared conditions, events and rules (G1), and autosave (X1)

Refresher: [Gameplay roadmap, G1 and X1](Draw3D_Gameplay_Roadmap_2026-10-21.md). Launcher: [2-rules1.cmd](checkpoint/2-rules1.cmd) (`-Demo rules1`).

**What the demo shows** (6 beats). A barrel is given seven lines:

```
when all: fact open; any: hour 6 20, weather rain
on hour 18 do put evening
on timer 1 do say Tick
on enters 4 do say Welcome
on leaves 4 do say Farewell
on fact bell do put answered
on act ring do put rung
```

1. **One condition language.**
   - Not open: it says so.
   - Open by day: it holds.
   - Open at 22:00 in clear weather: it does not hold, and says why ("none of them holds: it is 22:00, not between 06:00 and 20:00; nor the weather is clear, not rain").
   - Open at night in rain: it holds.
2. **At six.** The clock reaches 18:00 and `evening` is put.
3. **Every second, and the walker near.** "Tick" each second. "Welcome" once on coming within 4 m; "Farewell" once on leaving.
4. **A fact heard.** `bell` is put, and `answered` follows. The Playtest window lists what still waits, and why: "no one has done 'ring'".
5. **Wrong lines refused.** `on hour 25` and an `all:` with one part are refused, with what each needs.
6. **Written aside.** An autosave is written to `backups\autosave-NAME.json`, and the changes stay unsaved.

**By hand, after the last beat:**
1. Play. Walk to the barrel and away twice. Do you get one "Welcome" and one "Farewell" each time?
2. Open the **Playtest** window. Is "what waits, and why not yet" readable?
3. **The lamp check** (roadmap acceptance G1-b):
   - give any lamp or thing the line `on hour 18 do put lit`;
   - set the hour to just before 18:00 in the Sky window;
   - play: it should change at six.
4. **Autosave after a kill** (roadmap acceptance X1):
   - make any edit;
   - wait 3 minutes without saving (World > Autosave shows it is on);
   - end Draw3D from **Task Manager** (Godot), not the window's close button;
   - start [0-editor.cmd](checkpoint/0-editor.cmd).

   Expected: an offer **"Open the autosave"** from that time. Opening it should show your edit.

**Checks:**
- [ ] The beats run as captioned.
- [ ] A condition that should hold does; one that should not, does not, and says why.
- [ ] Welcome and Farewell come once each, by hand.
- [ ] Playtest's explanations are understandable.
- [ ] The lamp lights at six.
- [ ] After a kill, the autosave is offered and holds the edit.

Status: ______ Notes: ___________________________________________

---

## 3. The guard: patrol, investigate, raise the alarm, resume (7di)

Refresher: [Roadmap progress and next milestone](Draw3D_Roadmap_Progress_and_Next_Milestone_2026-10-22.md). Launcher: [3-guard.cmd](checkpoint/3-guard.cmd) (`-Demo guard`). That is the exact name; there is no `guard` tour.

**What the demo shows:**
1. The behaviour `guard_ways` is read.
   - Its states are patrol, investigate, answer and alarm.
   - A deer is said to lack what it needs (no round, cannot hear).
2. Three guards are put down and given `guard_ways`, each `with gives_up 2`.
3. **Patrol.** Each guard is on its round.
4. **Heard, looked, nothing.**
   - A noise like running steps, 7 m behind the first guard.
   - It says "What was that?", goes to where the noise was, and looks about.
   - It says "Nothing there." and resumes its round.
   - The others did not hear; **no alarm**.
5. **Seen: the alarm.**
   - A noise 12 m behind it, and the walker standing there.
   - Going to look, it **sees** them: "Intruder! To me!".
   - The fact `alarm` is put, and the others are told.
   - It chases (its legs hurry). A second noise elsewhere does not pull it off.
6. **The others answer.** The other two hurry to where the alarm was raised.
7. **Lost.**
   - The walker gets away.
   - The first guard searches where it last saw them, then returns to its round.
   - The alarm clears, and nobody is caught.
   - The second guard looks, says "Nothing there.", and returns.
   - The third gives up on the way (after 2 s) and returns.
8. The rounds go on.

**By hand, after the last beat:**
1. Play. Stay out of the guards' sight (about 14 m in front of them).
2. **Run** behind a guard, about 7–9 m away, out of its view. Running steps are noise; walking is quieter.
   - Expected: investigate, "What was that?", no alarm.
3. Let it find nothing: it should resume its round.
4. Step **into its view** within about 14 m.
   - Expected: "Intruder! To me!", and the other guards come.
5. Run away and break its line of sight.
   - Expected: it searches, then resumes its round; the alarm clears.
6. **Let one catch you** once. Expected (the rule you set): the catch is an event. What the guards do next is their behaviour's choice. Nothing is silently reset.

**Checks:**
- [ ] The guards patrol.
- [ ] A noise alone leads to investigating, **not** to the alarm.
- [ ] Being seen raises the alarm, and the others respond.
- [ ] Each returns to patrol once it is resolved.
- [ ] A catch behaves as an event, not a silent reset.

**Report** the first step that differed. Were you visible? Were you running? What did each guard do?

Status: ______ Notes: ___________________________________________

---

## 4. `work` and `talk`: existing character interactions

Refresher: [Gameplay roadmap, the three tours](Draw3D_Gameplay_Roadmap_2026-10-21.md). **Answer A1 first.**

**4a `work`.** Launcher: [4a-work.cmd](checkpoint/4a-work.cmd) (`-Demo work`).
- The village at work: folk going to their jobs by the clock, goods as facts, conditions, and work sequences such as the gardener picking apples.
- You walk up and are offered what they do: talk, buy.
- [ ] It starts, and the folk go about their work.
- [ ] Walking up to someone offers **talk** or **buy** (E / Y).
- [ ] Interruptions and reactions happen as captioned.
- [ ] No script or configuration error.

Status: ______ Notes: ___________________________________________

**4b `talk`.** Launcher: [4b-talk.cmd](checkpoint/4b-talk.cmd) (`-Demo talk`).
- Talks with answers, reactions, markers and the journal.
- [ ] It starts, and **talk** is offered by the cook and the others.
- [ ] Choosing an answer gives the expected response, and the journal records it.
- [ ] No error.

Status: ______ Notes: ___________________________________________

**What to expect.**
- **With A1 left as it is** (the ranger on samurai moves): **nothing is offered**. That is the configuration issue in section 0, not a new defect. Record "Blocked (A1)" and move on.
- **With A1 (b) "for testing"**: offers should appear. A missing offer is then a real finding.

---

## 5. `village`: doors and basic interaction

Refresher: [Gameplay roadmap, tour review](Draw3D_Gameplay_Roadmap_2026-10-21.md). Launcher: [5-village.cmd](checkpoint/5-village.cmd) (`-Demo village`).

**Expected to fail, in a known place:**
- no door offers "open" to the walker;
- Makers' "Walls inside…" is not found.

Both are assigned to G4. The aim is to record what you see, not to pass.

- [ ] The village opens with its houses, walls and **doors present**. They were empty before 2026-10-21's fix.
- [ ] The walker is **stopped by** a closed door, and does not pass through it.
- [ ] At a door, is "open" offered with E / Y? Record yes or no.
- [ ] Anything else unrelated (a UI or script error) is a new finding.

Status: ______ Notes: ___________________________________________

---

## 6. `items`: pickup and inventory

Refresher: [Gameplay roadmap, items](Draw3D_Gameplay_Roadmap_2026-10-21.md). Launcher: [6-items.cmd](checkpoint/6-items.cmd) (`-Demo items`).

**Expected:** the **tour** fails where the lantern never reaches the hand (assigned to G6). By hand, check whether the system itself works.

- [ ] The items are present (lantern and others).
- [ ] Walking up offers **pick up** (E / Y), and the item leaves the ground.
- [ ] **I** opens the inventory, and the item is there.
- [ ] Equipping or holding it: does it appear **in the hand**? Record yes or no. This is the tour's failing step.
- [ ] Using it (the lantern lit) behaves as captioned.

Status: ______ Notes: ___________________________________________

---

## 7. Construction regression: a cut fence keeps its posts (7df)

This one is still owed. 7df mended your finding that "posts were lost at cuts", and you have not seen the mend.

Launcher: [7-cuts.cmd](checkpoint/7-cuts.cmd) (`-Demo cuts`). Its last beat is "A fence cut in two keeps its posts".

**By hand, after the last beat:**
1. Build > **Fence**, and drag one out.
2. Click it. Beside it, press **Cut**.
3. Click one of the white **+** marks: two fences. Shift+click on a + cuts at once.
4. Look at the cut: **each new end has its own post**.
5. Ctrl+Z, then Ctrl+Y: the posts come back with the cut.
6. Optionally, do the same on a **wall**: each cut end keeps its pier.

**Checks:**
- [ ] Both new ends keep their posts or piers.
- [ ] Undo and redo keep the result.

Leave out `-Demo fences` unless the fence itself looks wrong. Do not start a general construction review.

Status: ______ Notes: ___________________________________________

---

## 8. Optional: your U4–U7 findings, as mended (2026-10-22)

Do this only if you have time; it can be deferred. Launch with [0-editor.cmd](checkpoint/0-editor.cmd), on any world.
- [ ] **Snow paint:**
  - Terrain > Paint says "Painted in this world now: snow N m²";
  - Erase lists "paint on the ground (snow N m²)" and removes it;
  - a roof, rock or crate over painted snow turns white.
- [ ] **Right click:**
  - on the selection, it stays selected;
  - away from it, nothing is selected;
  - with nothing selected, it puts away the tool in hand.
- [ ] **A part that spins:**
  - + / − on its speed visibly change it, in steps of 10;
  - no row of numbers is shown for "where".
- [ ] **A selected house:**
  - its handles are small and see-through;
  - corner + marks fade in only near the pointer;
  - none remain in play.
- [ ] A door drawn in a house is a door already, with "always open" or "always shut" as a choice.

Status: ______ Notes: ___________________________________________

---

## Section B. Checkpoint close-out (for Claude, after your results)

Give Claude this sheet, filled in, together with your answers to section A. Claude will then:
1. **Record** each test's outcome in PROGRESS.md.
2. **Sort what you found into four groups:**
   - confirmed defects;
   - configuration issues (A1–A3);
   - tour-script problems;
   - the already-known failures (village and items).
3. **Fix only** confirmed blockers that belong to this checkpoint.
4. **Run only** the relevant tests:
   - the host tests if the host changed;
   - `-Demo -Check` for the demos involved;
   - `-Tour -Only work`, `talk`, `village` and `items` as section C says.
5. **Report** what is now verified, what remains blocked, and what is deferred.

**Nothing from G3, PW0 or PW1 starts** until this is recorded and the next step is agreed (A4, A5).

## Section C. What Claude runs, so you don't have to

These run unseen and never take your mouse, keys or controller.

| Run | Purpose |
|---|---|
| `editor\run.ps1 -Tour -Only work` and `-Only talk` | Pass or fail after A1 is settled. |
| `editor\run.ps1 -Tour -Only village` and `-Only items` | Confirm they fail only at the known steps, and nowhere new. |
| `editor\run.ps1 -Demo behave -Check`, `rules1`, `guard`, `cuts` | The demos' own checks, after any fix. |
