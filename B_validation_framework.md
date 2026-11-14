# B. Workflow Validation Framework

Draw3D lifecycle validation programme. Written 2026-11-14.

This is the standing method. Every workflow in the lifecycle map (document A) is documented, tested, accepted and kept working by these rules. The register (document C) holds where each workflow stands.

## 1. What a workflow is

A workflow is one thing a game maker sets out to do, with a clear start and a result they can see. "Give the player a moveset" is a workflow. "The Moves tab" is not: it is a place a workflow passes through.

Each workflow has an id from the lifecycle map (L3.2 is the second workflow of lifecycle phase 3).

## 2. The workflow record

One record per workflow, always with these thirteen parts, in this order.

| # | Part | What goes in it |
|---|---|---|
| 1 | Name and stage | The id, the name, the lifecycle phase. |
| 2 | Intended outcome | What the maker has at the end, in one or two sentences. |
| 3 | Start and dependencies | What must already be true, and which earlier workflows provide it. |
| 4 | UI steps | The menus, windows, tabs and buttons, in the words the editor shows, in order. |
| 5 | Expected result | What should be seen in the editor and in play. |
| 6 | What exists, and its limits | What the code does today, with file names. Anything that needs a text file, a script or a workaround is said here in plain words. |
| 7 | Claude's test and evidence | How it was tested, on what date, and where the log and pictures are. |
| 8 | Functionality and integration | Did each step do what it should, and did the result hold in play, after saving, and after reopening. |
| 9 | Usability and accessibility | What a maker would find confusing, slow or hidden. Keyboard-only and small-window problems go here. |
| 10 | Missing or awkward controls | Controls that are absent, in the wrong place, or need a workaround. |
| 11 | Acceptance criteria | The numbered conditions that must all hold. Agreed with you before the workflow can be validated. |
| 12 | Hand test | The steps you perform, and what you judge. |
| 13 | Regression tests | Which earlier workflows must be run again when this one changes, and which checks do it. |

Rules for writing a record:

- **Only what exists.** A step is written only if the control is in the editor today. A step that could not be done is written as "cannot be done through the UI", with what was tried.
- **A related feature proves nothing.** "The Studio has a Play as this button" does not mean "the world plays that character". The record says which was checked.
- **Workarounds are named.** Editing a `.txt` file, running a script, or setting something from code is recorded as a gap, never as a step.

## 3. Statuses

A workflow has exactly one status. It only moves forward when the evidence for the next status exists.

| Status | Meaning | Evidence needed |
|---|---|---|
| **Discovered** | The existing workflow has been read from the code and the UI, and written as a record. | Parts 1–6 of the record. |
| **Claude tested** | Claude has attempted the workflow through the UI and checked the result. | A UI test log and pictures; parts 7–10. Failures are recorded, not hidden: a workflow that was tested and failed is "Claude tested (fails)". |
| **Refined** | The problems found were fixed, with your authorisation, and the UI test was run again and passed. | The fix, the rerun log, and the regression checks it called for. |
| **Hand tested** | You have completed the workflow yourself, unaided. | Your result, dated, with anything you noted. |
| **Validated** | Every agreed acceptance criterion is met, by Claude's test and by your hand test. | Both, and no open blocking finding. |

Two more marks, used beside a status:

- **Regressed**: a validated or hand-tested workflow that a later change broke. It goes back to "Claude tested (fails)" until fixed and retested.
- **Blocked**: it cannot be tested yet because a workflow it depends on fails.

## 4. Kinds of evidence, kept apart

These are five different things. A record says which one each claim rests on, and never swaps one for another.

| Kind | What it is | What it can prove | What it cannot prove |
|---|---|---|---|
| **Automated test** | A Rust test of the host, or a headless script check. | The system underneath does what its code says. | That a person can reach or use it. |
| **Scripted demonstration** | A demo (`run.ps1 -Demo`). It may call functions directly. | That a capability exists and can be shown. | That the UI path works. It is never counted as a UI test. |
| **UI test** | An "author" run (`run.ps1 -Author`): only clicks, keys and drags sent as the mouse and keyboard send them, on what is visible. | That the buttons and windows do the job when pressed in order. | Feel, clarity, and anything the run's set-up switches off (see section 6). |
| **Runtime test** | Play begun and the result measured: who is played, which moves begin, how far the body went. | That the configuration reaches the game. | That it feels right. |
| **Hand test** | You, with the editor. | Usability, discoverability, feel, and that it works on your real files. | Nothing is assumed beyond what you did. |

A workflow is not "Claude tested" on a demo alone, and not "Validated" without a hand test.

## 5. Testing dimensions

Each record judges the dimensions that apply to it and says "not relevant" for the rest.

| Dimension | The question |
|---|---|
| Functionality | Does each step do what it says? |
| Usability | Can it be done without confusion or wasted steps? |
| Discoverability | Would a maker find it without being told where it is? |
| Accessibility | Does it work at 1280×720, by keyboard, and are words plain? |
| Integration | Does the result reach play, saving, reopening and the other windows that show it? |
| Reliability | Does it give the same result every time? |
| Completeness | Can the whole job be done, or only part? |
| Flexibility | Can a different game make a different choice here? |
| Visual quality | Where it produces something seen. |
| Performance | Where it could be slow. |

A workflow that works but is confusing or hard to find does not pass.

## 6. How Claude tests through the UI

**The tool.** The repository already has an author harness (`editor/godot/author.gd`). It finds a button by the words it shows, clicks it with a real mouse event, types into fields key by key, and writes every step to a log. It cannot call the host or write a file. The validation scripts are named `author_lifecycle*.gd` and are checks only; they change nothing in the engine.

**The sandbox.** Your files are never used. Before the first step the run copies the project's texts into its own folder and keeps its worlds, its list of recent worlds and its profiles file there. This set-up is the one thing done outside the UI, and every log says so.

**Known blind spots.** A scripted run differs from a run by hand in these ways. Whatever depends on them is marked "not covered by the UI test" and is left to the hand test.

| Difference | Effect |
|---|---|
| "Come back where I was" is off in scripted runs. | The place, class and carried things kept from the last play are not restored. |
| The title card and the class picker are off. | A game that begins with a title or a choice of class is not exercised from its title. |
| Your own buttons (`%APPDATA%\Draw3D\controls.txt`) are not read. | The keys tested are Draw3D's defaults and the project's layout. |
| No controller. | Keyboard and mouse only. |
| The editor is put on "nothing open" by the script. | The real start-up path is run by hand only. |
| The system's own file pickers (used by a few imports) cannot be clicked. | Those are hand-test only. The world Open and Save dialogs are Draw3D's own and are clicked. |

**Play it through.** The standing rule from 2026-11-13 holds here: a game is not handed over for a hand test until a run has played it from the title as a player. Parts checked on their own are written "checked alone, not in play".

## 7. Findings

Every problem found gets an id (F-L3-04 is the fourth finding of lifecycle phase 3) and one severity.

| Severity | Meaning |
|---|---|
| **Blocker** | The workflow cannot be completed through the UI, or the result is wrong in play. |
| **Major** | It can be completed, but a maker would likely go wrong or lose work. |
| **Minor** | It works; wording, placement or an extra step is in the way. |
| **Question** | A decision for you, not a fault. |

Each finding states what was seen, how to see it again, where it comes from in the code if known, and whether the cause is **confirmed** (reproduced) or **likely** (read from code, not yet reproduced). A likely cause is never written as fact.

## 8. Acceptance

A workflow is validated when all of these hold:

1. Its acceptance criteria were agreed with you before testing for acceptance.
2. A UI test completes every step with no step marked "stuck".
3. The result holds in play, after save, and after close and reopen.
4. No step needed a text file, a script, or code.
5. No open Blocker or Major finding.
6. You completed the hand test and accept it.
7. Its regression tests pass.

## 9. Fixing and refining

1. Findings are presented to you with a proposed fix and its size.
2. Nothing is changed until you authorise it.
3. The fix follows the standing rules: the data drives it, the data has a window, nothing is set for one game's sake.
4. The UI test is run again, then the regression set.
5. The record and the register are updated the same day.

## 10. Regression

- Every validated workflow keeps one UI test that can be rerun unattended. Together they are the **regression set**.
- A change to a file listed in a workflow's part 13 means that workflow's test runs again before the change is called done.
- A later stage that exposes a fault in an earlier workflow reopens that workflow. Its status drops, and the register records why.
- The register lists, for each validated workflow, the date and result of its last run. A test that was not run is never written as passed.

## 11. The two cycles

- **Cycle A, the baseline.** Each lifecycle phase in order. A phase is finished when every workflow the benchmark game needs from it is validated.
- **Cycle B, depth.** Any workflow may be returned to and improved. The same record, statuses, acceptance and regression rules apply. An improvement that breaks a validated workflow is not done.

## 12. The benchmark game

One small game grows through the phases. It exists to exercise workflows, not to be a product. Its scope at each phase is set in the lifecycle map (document A, section 4). Nothing is added to it that the editor's UI could not make; a part that cannot be made is a finding, and the work is to add the control to the engine first.
