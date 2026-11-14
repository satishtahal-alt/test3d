# Draw3D lifecycle validation programme

Begun 2026-11-14. The aim: prove that a person can make a game with Draw3D by a dependable, repeatable sequence of workflows, one lifecycle phase at a time, and keep each proven workflow working.

## The documents

| | Document | What it is |
|---|---|---|
| A | [Lifecycle map](A_lifecycle_map.md) | The twelve phases, their workflows and dependencies, and the proposed benchmark game. |
| B | [Validation framework](B_validation_framework.md) | The workflow record, the statuses, the kinds of evidence, acceptance and regression rules. |
| C | [Progress and regression register](C_progress_and_regression_register.md) | Where every workflow stands, every open finding, the decisions waiting for you. |
| D | [Phase 1 report](D_phase1_project_and_world.md) | Projects, worlds, storage and asset ownership: what exists and what was found. |
| E | [Phase 2 report](E_phase2_player_and_controls.md) | The player, the moveset and the controls: the full trace of who and what is played. |
| F | [Hand-test instructions](F_hand_test_instructions.md) | Ten tests for you to do in the editor. |

## The evidence

`evidence/2026-11-14/`

- `ui-test-log.txt`: every step the UI test pressed, in order, and what it saw.
- `seen.json`: the same observations as data.
- 31 pictures, numbered in the order taken.

The test itself is `editor/godot/author_lifecycle.gd`, run with `editor\run.ps1 -Author lifecycle -Port 47930`. It presses the editor's own buttons on a sandbox copy of the project's files and changes nothing in the engine.

## Where it stands

- No engine code has been changed.
- 12 of 26 workflows in the first three phases were run through the UI. 10 work. 2 fail.
- Nothing is hand tested or validated yet.
- 29 findings are open; one is a blocker (a moveset chosen for the player is not the one played when a profile is in use).
- Nine decisions are waiting for you (register, section 4).
