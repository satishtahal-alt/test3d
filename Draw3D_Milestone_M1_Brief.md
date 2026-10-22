# Milestone M1 — Kinds as content (implementation brief)

Status: **approved 2026-10-09 with conditions; built the same day, at checkpoint C4, waiting on the user.** Baseline commit `1d92092`
(repository put under git on 2026-10-09; `.gitignore` excludes Rust targets, Godot binaries,
the exported build, logs, run results, autosaves, tour and demo screenshots).
Rationale and audit: `Draw3D_Audit_and_Next_Milestone_2026-10-09.md` (Deliverable C).

## Acceptance checklist (the user's seven conditions, each with its check)

| # | Condition | Checked by |
|---|---|---|
| 1 | A kind is created and edited through a visible editor workflow, the result previewed, and validation errors understood | Kind panel (F6): list, new, duplicate, delete, text editor, apply; problems shown line by line under the text; every placed thing of the kind is remade on apply (the world is the preview). Tour part `kinds`; manual script steps 2–5 |
| 2 | The creature's behaviour is data-driven through existing capabilities, not hard-coded for it | The creature is only `kind.NAME` text: `rig sheep`, `someone`, `roams`, `shies`, `idles`, `moves sheep`, `offers pet back`. No Rust or GDScript names it. Demo `kinds` and tour `kinds` make it by command and play it; `grep` shows no new kind name in code |
| 3 | Kinds persist in the project document, survive save/reopen, can be placed, and the runtime uses them | Protocol test p17 (write, scene, open: kinds and placed things back); tour `kinds` (Ctrl+S path: save as, reopen); the thing is pushed, the creature roams and is petted while the world runs |
| 4 | Undo/redo covers the authoring operations and is tested | p17: set, edit, delete, duplicate each undone and redone; p16: place by kind undone and redone. Ctrl+Z in the editor goes through the same host command |
| 5 | Built-in kinds keep working without a wholesale migration | No built-in kind is migrated; `PLACEABLE` and `real.rs` untouched except `Placed` gaining a `kind` parameter. Existing suites: host protocol p01–p15, shapes unit tests, whole tour once at checkpoint C4 |
| 6 | Authored content is available through the documented host interface, and the export path is verified | Host commands `kinds`, `kind` (get/set/delete/duplicate) documented in `lib.rs`; `ask.ps1 kinds` from beside the editor (p17 checks the beside rule); `export.ps1` then `run.ps1 -Game -Demo kinds -Check` in the built game |
| 7 | The manual verification script is complete and ready | Section "Manual verification" below, rewritten for what was built; automated tests are not treated as proof of usability |

Architectural constraint honoured: the kind text is the T1/T2 representation. A kind holds
a parts list today; a later `mesh NAME` line (M3) names an editable mesh asset in place of
or beside the parts, and the realizer bakes either to `MSH1`. Nothing in the data model
assumes geometry is always rebuilt from primitives: parts, mesh assets and (M2) terrain
layers are all document content read by the realizer. M2, M3, M4 remain separate.

## Goal

A user creates a new kind of thing in the editor, as text the document keeps, shapes it
from parts, gives it affordances and behaviour, places it, plays with it, saves, reopens,
exports — with no change to Rust or GDScript. Two proofs: a pushable prop, a simple
bodied creature that roams, shies and can be petted.

## Design (fixed before coding)

**Kind text** (one entry per kind; same idiom as `styles.txt`; blank line ends an entry):

```
kind NAME [like OTHER]
  size W D                         footprint in metres (default 0.8 0.8)
  part block x y z w h d COLOUR    a box centred at x,y,z
  part post x y z r h COLOUR       a cylinder standing on y
  part blob x y z rx ry rz COLOUR  an ellipsoid
  part limb x1 y1 z1 r1 x2 y2 z2 r2 COLOUR   a tapered tube
  part face x y z nx ny nz w h COLOUR        a flat quad
  solid auto | box ... | post ... | hull ... (today's collider lines)
  rig none                         (a rig name is parsed and refused in M1)
  <any trait line in today's vocabulary: offers, point, loose, kept, held, lights, does,
   someone, bodied, roams, shies, idles, greets, moves, body, weighs, called, opens, when,
   takes, pressed, goes>
```

COLOUR is `rrggbb` or a style colour name (`wall`, `trim`, `stone`, `wood`, `bark`,
`rock`, `cliff`, `sand`, `dirt`, a roof or pane by number `roof1`, `pane2`) so kinds follow
the world's style. Anything not understood is reported with its line and left out; the
text is never refused.

**Storage:** document asset `kind.NAME` → the entry's text (`Document.prefabs`). Edits are
document edits: undo, history, Ctrl+S, Save As, export all come for free.

**Realization:** archetype `kind` with brief `kind NAME`; `leaf_assets("kind")` names
`kind.NAME` so a change re-realizes only that kind's leaves. Outputs: `mesh` (MSH1 from
the parts via `Mesh::block/limb/blob/face`), `collider`, `traits` (trait lines verbatim,
`called` defaulted to the name). No `rig`/`skin` in M1.

**Placement:** `Placed` gains a string parameter `kind` (empty = use `what`). `placeable`
answers `{ "builtin": [...], "kinds": [...] }`.

**Commands (host):**

| op | fields | does |
|---|---|---|
| `kinds` | — | names, each with its problems (from the last realize) |
| `kind` | `name` | the text |
| `kind` | `name`, `text` | set (document edit) |
| `kind` | `name`, `delete: true` | delete; refused while placed things use it (say which) |
| `kind` | `name`, `as` | duplicate as `as` with `like name` |

Reads allowed from `--beside`; writes editor-only (as today's rule).

## Tasks and checks

| # | Task | Files | Targeted check |
|---|---|---|---|
| 0 | `git init` + initial commit (user decides) | root | — |
| 1 | Kind parser and problems | `editor/host/src/shapes/kind.rs` | unit tests: parse, `like`, bad line reported |
| 2 | Realizer for `kind`; style colour names | `shapes.rs` (`shape_in`, `solid`, `traits`, `made_from`) | unit test: three parts → triangle count; auto collider; traits verbatim |
| 3 | `Placed` by kind; `placeable` shape | `shapes/land.rs`, `lib.rs` | protocol p16: place, describe, undo, redo |
| 4 | `kinds` / `kind` commands; beside rules | `lib.rs` | protocol p17: set, undo, save/open round trip, delete refused while used |
| 5 | Palette: kinds + "New kind…" | `palette.gd` | tour `kinds` |
| 6 | Kind panel | `kindview.gd` (new), `main.gd`, `controls.gd` (F6) | tour `kinds`: edit → instances change; problems shown |
| 7 | Inspector panel | `inspect.gd` (new), `main.gd` (F7 / on pick) | tour: owner params listed; `amp` edited moves trees |
| 8 | Open… / Save As… | `main.gd` | tour `world`: save as, reopen |
| 9 | Play proofs as content only | `demo.gd` `kinds`; tour `kinds` | acts `could`: push and pet offered and done |
| 10 | Export with kinds | `export.ps1 -Check` | built game shows them |
| 11 | PROGRESS.md slice; hub shows the panel as "where it is written" for kinds | docs, `hub.gd` | tour `hub` |

Checkpoints: C1 after 1–2, C2 after 3–4, C3 after 5–8, C4 after 9–11 (whole tour and all
suites once, screen on).

## What was built (2026-10-09)

Everything in the task table through task 11, except as noted under "Deferred". One
widening against the proposal: a kind may say `rig person | sheep | skiff` (a rig the
engine has), and each part may say `bone B`. Without a rig nothing can be someone
(`folk.gd` gives a body only to content with a rig and a skin), so the creature proof
needs it. Poses are those written for that rig's kind of body. Rigs of the user's own
remain deferred (M3).

| Where | What |
|---|---|
| `core/src/plan.rs`, `build.rs` | `Generator::leaf_assets_of(archetype, brief)`: a leaf's assets may follow its brief; the build names the leaf's content from them |
| `editor/host/src/shapes/kind.rs` | the kind text: read, `like` resolved, built into a mesh in the world's look, collider, traits, rig and skin; three unit tests |
| `shapes.rs`, `shapes/land.rs` | archetype `kind` realized from the document's `kind.NAME` texts; `Placed` puts one down by name and reads its footprint |
| `editor/host/src/lib.rs` | commands `place` by `kind`, `placeable` with `kinds`, `kinds`, `kind` (get, set, delete, duplicate), `node`; kinds read from beside the editor, written only by it |
| `editor/host/tests/protocol.rs` | p16, p17 |
| `editor/godot/kindview.gd` (F6) | list, New, Duplicate, Delete, Put one down, text, Apply (Ctrl+Enter), problems line by line, status |
| `editor/godot/inspect.gd` (F7) | what is picked, its node, its parameters set by Enter, the way to its kind |
| `palette.gd`, `main.gd` | kinds in the palette after the code's things; `new kind...`; Ctrl+O open, Ctrl+Shift+S save as; the window and the top line say the world's file; the palette follows every change to the kinds |
| `things.gd`, `hub.gd`, `hubview.gd` | a thing says `kind NAME`; the hub says it is written in the kinds panel |
| `tour.gd` (`-Tour -Only kinds`), `demo.gd` (`-Demo kinds`) | the kinds part of the tour, and the demo |

## Manual verification (for the user; nothing in Rust or GDScript is changed)

Start: `editor\run.ps1` (the playground, or the world you last wrote with Ctrl+S).

1. **A kind, written.** Press **F6**. Press **New**: a kind `thing1` appears in the list,
   its text to the right, and `thing1  (kind)` at the end of the palette. Replace the
   text with:
   ```
   kind drum
   	size 0.9 0.9
   	part post 0 0 0 0.4 0.8 wood
   	part block 0 0.82 0 0.86 0.04 0.86 trim
   	loose 30
   	offers push side
   	point side 0 0.4 0.5 from 0 1
   	point side 0 0.4 -0.5 from 0 -1
   	point side 0.5 0.4 0 from 1 0
   	point side -0.5 0.4 0 from -1 0
   ```
   Press **Apply** (or Ctrl+Enter). Expect: "Kind 'drum' written", nothing under the
   text, `drum` in the list and the palette. Choose `thing1`, press **Delete**.
2. **A wrong line, said.** Add a line `part wheel 1 2 3` and apply. Expect: under the
   text, `line 11 (part wheel 1 2 3): 'wheel' is not a part: block, post, blob, limb or
   face`, and the status still says 2 parts: the rest held. Take the line out, apply.
3. **Put down, and the preview.** Press **Put one down**; the panel closes and the
   palette says `placing: drum`. Click the ground three times: three drums stand on the
   land, in the style's wood. Press `just pick`. Press F6, change `0.4 0.8 wood` to
   `0.3 1.4 stone`, apply: all three are tall and grey at once. **Ctrl+Z**: wood again.
   **Ctrl+Y**. Press F2 twice: the drums follow the style's wood and trim.
4. **Pushed.** Tab. Walk into a drum: it goes. Come at a side: "E take hold of the drum";
   E, push and pull it, E lets go. Esc back to editing.
5. **Someone new.** F6, New, replace the text with:
   ```
   kind hopper
   	size 0.8 1.2
   	rig sheep
   	part blob 0 0.55 0 0.3 0.28 0.45 c8d8f0 bone hips
   	part blob 0 0.8 -0.55 0.14 0.14 0.14 e0e8f8 bone head
   	part limb -0.13 0.5 -0.29 0.05 -0.13 0.03 -0.29 0.035 606070 bone lower_fore.L
   	part limb 0.13 0.5 -0.29 0.05 0.13 0.03 -0.29 0.035 606070 bone lower_fore.R
   	part limb -0.13 0.5 0.29 0.05 -0.13 0.03 0.29 0.035 606070 bone lower_hind.L
   	part limb 0.13 0.5 0.29 0.05 0.13 0.03 0.29 0.035 606070 bone lower_hind.R
   	someone
   	roams 6
   	shies 4 2.6
   	idles graze
   	moves sheep
   	body 0.3 0.8
   	weighs 40
   	offers pet back
   ```
   Apply, put two down near the drums. Tab. Expect: they stray about; run at one, it
   bolts; crouch (C) and come slowly: "E pet the hopper"; E strokes it, hand on its
   back, and it stands for it. Esc.
6. **Variants.** F6, choose `drum`, **Duplicate**: `drum-1`, like drum. Add a line
   `part block 0 1.0 0 0.3 0.3 0.3 aa2222` and `called the red drum`, apply, put one
   down: a drum with a red block on top. Change the drum's wood to stone: the red drum
   follows (it is like the drum).
7. **Inspected.** Pick a drum, press **F7**: kind `kind`, made by `drum` under `kind#it`,
   generator `placed`, parameters `what`, `variant`, `yaw`, `kind`. Type `90` into
   `yaw`, Enter: it turns. Ctrl+Z. Press **edit this kind (F6)**: the panel opens on
   `drum`. Pick a tree, F7: generator `woods`, its parameters. The note says they are
   knobs of the generator, not the thing's shape.
8. **Kept with care.** F6, choose `hopper`, Delete: refused, "2 of 'hopper' are put down".
   Choose `drum`, Delete: refused, "drum-1 like 'drum'".
9. **Saved and opened.** **Ctrl+Shift+S**, write `kinds-test.json` beside the editor
   folder. The window's title and the top line say `kinds-test.json`. Close the editor.
   `editor\run.ps1`, **Ctrl+O**, open it: everything as left; F6 lists the three kinds;
   the palette has them.
10. **From beside the editor.** In another terminal, `editor\ask.ps1 kinds`: the three,
    each with how many are put down. `editor\ask.ps1 kind name=drum`: its text.
11. **In the built game.** Ctrl+S (so `editor\world.json` is this world), then
    `editor\export.ps1`, then `game.bat`: the drums and hoppers are there, pushed and
    stroked as in the editor. (Also checked by script: `editor\run.ps1 -Game -Demo
    kinds -Check`.)

What to say afterwards: whether writing a kind as text in this panel is a way you would
work, before handles and a mesh editor exist (M3); whether the words are the right
words; what you tried to write that it could not say.

## Non-goals (held to)

Direct mesh editing; terrain; rigs of the user's own; migrating built-in kinds; rules
as data; visual part handles; written texts into the document (M2).

## Deferred from M1, and why

| Deferred | Why |
|---|---|
| Rigs of the user's own (`bone` lines) | poses, treads and places on a rig are a body of work (M3); the engine's three rigs cover a person, a four-legged creature and a vehicle |
| Mesh assets (`mesh NAME`) | M3; the text has the place for the line |
| Freeze/thaw and regenerate buttons in the inspector | M2 (generators workspace); the inspector shows the mode and says parameters are read-only when frozen |
| Kinds as children of generators (a flock of hoppers) | the generators are Rust; M4 makes what they strew data too |
| Renaming a kind | duplicate as the new name, delete the old; a rename that follows the placed things is a later convenience |
