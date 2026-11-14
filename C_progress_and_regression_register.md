# C. Validation Progress and Regression Register

Draw3D lifecycle validation programme. Last updated 2026-11-14.

Where every workflow stands, what is open against it, and what must be run again when something changes. Statuses and evidence kinds are defined in the framework (document B). Workflow ids are from the lifecycle map (document A).

## 1. Summary

| | Count |
|---|---|
| Workflows in L1 to L3 | 26 |
| Validated | 0 |
| Hand tested | 0 |
| Refined | 0 |
| Claude tested, works | 10 |
| Claude tested, fails | 2 |
| Discovered | 4 |
| Listed only | 10 |
| Workflows listed in L4 to L12 | 58, none investigated |
| Open findings | 29 (1 Blocker, 18 Major, 8 Minor, 2 Questions) |

Nothing is validated. Nothing has been hand tested. No fix has been made.

## 2. Workflow status

### L1. Project

| Id | Workflow | Status | Last test | Evidence | Open findings | Next |
|---|---|---|---|---|---|---|
| L1.1 | Start and choose where to begin | Claude tested | 2026-11-14 | UI test P1.1 | – | Hand test 1 (the real start-up was not covered) |
| L1.2 | Keep a world as a game | Claude tested | 2026-11-14 | UI test P1.5 | F-L1-01, F-L1-04 | Hand test 5 |
| L1.3 | New game from an existing one | Claude tested | 2026-11-14 | UI test P1.7 | F-L1-02 | Hand test 5 |
| L1.4 | See what the game is made of | Claude tested (fails) | 2026-11-14 | UI test P1.8 | F-L1-03 | Decision on project folders |
| L1.5 | Set the game as a whole | Listed | – | – | – | Investigate |
| L1.6 | Find, move, back up a game's files | Discovered | – | Code | F-L1-01, F-L1-05, F-L1-08 | Decision on project folders |
| L1.7 | Check the game has everything | Discovered | – | Code | F-L1-06, F-L1-07 | Test with a missing file; export in L12 |

### L2. World

| Id | Workflow | Status | Last test | Evidence | Open findings | Next |
|---|---|---|---|---|---|---|
| L2.1 | A new blank world | Claude tested | 2026-11-14 | UI test P1.2, P1.6 | F-L2-01, F-L2-02 | Hand test 2 |
| L2.2 | A new generated world | Listed | – | – | – | Investigate |
| L2.3 | A new world from a map file | Listed | – | – | – | Investigate |
| L2.4 | A new world from a genre | Listed | – | – | – | Investigate |
| L2.5 | Save a world | Claude tested | 2026-11-14 | UI test P1.3, P1.4 | F-L1-02 | Hand test 3 |
| L2.6 | Close, open, go back | Claude tested | 2026-11-14 | UI test P1.6 (Open dialog only) | F-L2-03, F-L2-04, F-L2-05 | Hand test 4 (Recent and Begin rows) |
| L2.7 | Restore an earlier save | Listed | – | An older tour part, `keeping` | – | Investigate |
| L2.8 | Duplicate a world | Claude tested | 2026-11-14 | UI test P1.7 | – | Hand test 5 |
| L2.9 | Where and how a game starts | Listed | – | – | F-L2-01 | Investigate before L3 is accepted |
| L2.10 | The world's look, hour, weather | Listed | – | – | – | With L4 |
| L2.11 | Join several worlds | Listed | – | – | F-L1-03 | Investigate |
| L2.12 | Bring one world into another | Listed | – | – | – | Investigate |

### L3. Player

| Id | Workflow | Status | Last test | Evidence | Open findings | Next |
|---|---|---|---|---|---|---|
| L3.1 | Choose who the world plays | Claude tested | 2026-11-14 | UI test P2.1; runtime P2.4 | F-L3-01, F-L3-06, F-L3-07 | Hand test 7 |
| L3.2 | Give the player a moveset | **Claude tested (fails)** | 2026-11-14 | UI test P2.2; runtime P2.4 | **F-L3-02**, F-L3-03, F-L3-04 | Decision; then fix |
| L3.3 | Lay out the buttons | Discovered | 2026-11-14 | Pages read (P2.3); default keys in play | F-L3-11, F-L3-12 | Test changing a binding; hand test 9 |
| L3.4 | Model, rig, animations | Discovered | – | Code; shipped characters seen in play | – | Test an import |
| L3.5 | Play and check | Claude tested | 2026-11-14 | Runtime P2.4, twice | F-L3-13, F-L2-01 | Hand test 7 to 9, with a controller |
| L3.6 | Save, close, reopen holds | Claude tested | 2026-11-14 | UI test P1.4, P1.6, P1.7 | – | Hand test 8 |
| L3.7 | A new character or moveset | Listed | – | – | – | Investigate |

### L4 to L12

Listed in document A. 58 workflows; none investigated for this programme.

## 3. Open findings

Full text in documents D and E. C = confirmed, L = likely (read in code, not reproduced).

| Id | Sev. | In one line | C/L | Decision or fix |
|---|---|---|---|---|
| F-L3-02 | Blocker | A moveset chosen on the player's page is not played when a profile is in use | C | Awaiting your review |
| F-L1-02 | Major | Save dialog opens in the engine's folder | C | Awaiting |
| F-L1-03 | Major | Project › Worlds lists only the engine's folders | C | Awaiting |
| F-L1-04 | Major | Games and recent lists drop moved files silently | L | Awaiting |
| F-L1-05 | Major | Imports go to the engine's folders, not the game's | L | Awaiting |
| F-L1-06 | Major | Missing pictures and sounds are silent; no check before Play | L | Awaiting |
| F-L1-07 | Major | Exported game carries no profiles | L | Confirm in L12 |
| F-L1-08 | Major | Two files named `hold_the_light.json`; demo-built games live in the demo folder | C | Awaiting |
| F-L2-01 | Major | No Player start in a new world; the start drifts with each play | C | Awaiting |
| F-L2-05 | Major | Old worlds silently get 45 to 58 own versions on opening | C | Awaiting |
| F-L3-01 | Major | Worlds with no choice play the Skyfarer; shown as "Assigned" | C | Awaiting |
| F-L3-03 | Major | Profiles are engine-wide, by character name | C | Awaiting |
| F-L3-04 | Major | Moveset change writes the shared library; wording wrong | C | Awaiting |
| F-L3-05 | Major | Previous world's character still worn while editing | C | Awaiting |
| F-L3-07 | Major | Two characters marked as played; file order decides | C | Awaiting |
| F-L3-08 | Major | Six deciders of the player, three of the moveset, mostly unnamed in the UI | C | Awaiting |
| F-L3-10 | Major | "Come back where I was" restores class and facts; unsaved worlds share one | L | Hand test 10b |
| F-L3-14 | Major | Class on the title overrides the choice; no UI test can reach the title | L | Extend the test harness |
| F-TEST-01 | Major | The author harness cannot choose from lists over twelve items; older author scripts would stop there | C | Awaiting |
| F-L2-02 | Minor | Untouched new world has 1 unsaved change | C | Awaiting |
| F-L2-03 | Minor | View is 150 m out after opening | C | Awaiting |
| F-L2-04 | Minor | Recent-list times five hours off | C | Awaiting |
| F-L2-06 | Minor | Long message covers the weather buttons | C | Awaiting |
| F-L3-06 | Minor | Two tooltips name `characters.txt` and `walks`, wrongly | C | Awaiting |
| F-L3-11 | Minor | Two things called a profile | C | Awaiting |
| F-L3-12 | Minor | Four places edit buttons | C | Awaiting |
| F-L3-13 | Minor | Controls card covers status text in play | C | Awaiting |
| F-L1-01 | Question | No project: the engine folder is the only one | C | Your decision (document D, section 9) |
| F-L3-09 | Question | Which character a new game begins with | – | Your decision (document E, section 9) |

## 4. Decisions waiting for you

From document D, section 9:

1. Does a new game start with a copy of the engine's characters and moves, or link to them?
2. Are profiles the game's, the character's, or both?
3. Should the engine's sample worlds and your games be kept apart now, before project folders?
4. Which existing worlds are your games, and which are samples?

From document E, section 9:

5. Should a profile belong to a game?
6. Should the player's page be the one place that decides character, moveset, gear and buttons?
7. Should a new world start with nobody chosen, or with a named starting character?

Also:

8. Is the benchmark game in document A, section 4, the one to use?
9. Are the proposed acceptance criteria in documents D and E the ones to accept against?

## 5. Regression register

### 5.1 The regression set

Empty. A test joins the set when its workflow is validated.

### 5.2 Tests that exist and will join it

| Test | Covers | How to run | Last run | Result |
|---|---|---|---|---|
| `author_lifecycle` | L1.1 to L1.4, L2.1, L2.5, L2.6, L2.8, L3.1 to L3.3 (read), L3.5, L3.6, and four older worlds | `editor\run.ps1 -Author lifecycle -Port 47930` (a free port; see the note below) | 2026-11-14 | 53 steps, 0 stuck, 0 script errors. Two workflows fail on what it saw (L1.4, L3.2) |

**Before any run:** check no other session is using the port. On 2026-11-14 two other sessions were running demos on 47870 and 47880.

### 5.3 What to run again, when

| If this changes | Run again |
|---|---|
| `startview.gd`, `newworld.gd`, `main.gd` (start-up, new, open, save) | `author_lifecycle` P1.* |
| `assets.gd`, `assetask.gd`, `main.gd` (library and texts) | `author_lifecycle` P2.*, P2.5 (older worlds) |
| `charbook.gd`, `player.gd`, `builds.gd`, `profilebook.gd`, `profileplay.gd` | `author_lifecycle` P2.* |
| `projectview.gd` (Overview, The player), `worldspage.gd` | `author_lifecycle` P2.1 to P2.3, P1.8 |
| `controls.gd`, `controlspage.gd`, `presses.gd` | `author_lifecycle` P2.3, P2.4 |
| `listsearch.gd`, `desk.gd`, `author.gd` | The whole of `author_lifecycle` |

### 5.4 Older checks, not part of this programme

The tour parts (`keeping`, `makers`, `world`), the demos (`inputs`, `assets`, `hubplayer`) and the older author scripts exist and cover neighbouring ground. They were not run for this programme and are not counted as evidence here. F-TEST-01 means the older author scripts need their list-choosing step mended before they can be relied on.

## 6. Log

| Date | What |
|---|---|
| 2026-11-14 | Programme begun. Lifecycle map, framework, register written. L1 to L3 investigated in code. `author_lifecycle` written and run: 53 steps, 0 stuck. 29 findings recorded. No engine code changed. |
| 2026-11-14 | Accident: a first run collided with another session's demo on port 47870 and overwrote the start of its log. Stopped; rerun on 47930. |
