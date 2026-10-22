# Engine architecture review — Pass 19: draw units

Scope, as set: reduce draw-unit pressure in the adapter only, without touching the content or identity model, and measure the Pass 18 worlds before and after. No Rust changed in this pass: not the core, not the host, not the protocol. The frozen suite and the native and protocol tests were not re-run because nothing they cover changed.

---

## 0. Outcome

The draw-unit pressure is gone for kit worlds. With the whole world in view:

| 100 villages unless noted | Pass 18 | now |
|---|---|---|
| Kit: draw calls | 20,708 | 2,891 |
| Kit: idle frame | 38 ms | 15.6 ms |
| Kit, 400 villages: draw calls | 82,649 | 2,936 |
| Kit, 400 villages: idle frame | 175 ms | 57.5 ms |

A kit world now draws in the same time as a world whose parts are all alike (15.7 ms for 94 million triangles). What is left in those frames is triangles, not units.

It costs something: edits are 0.2–1.2 ms slower in the editor, undo and reconnect are slower, and the editor holds somewhat more memory in the lighter worlds. Section 3 has the numbers. Worlds where no two parts are alike do not benefit, as expected: every mesh is in one place, so there is nothing to draw together.

One fault unrelated to draw units was found by the benchmark and fixed in the adapter (section 4, Finding 5).

---

## 1. What changed

Before, a draw unit was one mesh across a region of 4 × 4 cells, everywhere. Units were therefore meshes × regions, whatever the viewer could see.

Now regions come in three levels: 4, 16 and 64 cells on a side (128 m, 512 m, 2 km). Each larger region is made of whole regions of the level below. Every place is drawn by exactly one region: the largest one around it that the eye is farther from than twice its side. So:

- near the eye, small regions, which are left out when out of view, as before;
- far from the eye, large regions, which are few.

A region's units are laid only while it is the one drawn. When a cell changes, the regions over it at every level are told which units no longer hold. Those being drawn are laid again at once; the others when they are next drawn. A large region's instance list is its child regions' lists end to end, so laying it again reads only what changed and copies the rest.

What did not change: a cell arrives whole and replaces what was there; a thing names its content; content is fetched and kept by name; picking reads the cooked cells. The adapter still hears nothing about single things. One level (`-Levels 4`) is exactly the old behaviour and is kept as the control.

The two numbers (the levels, and "twice its side") are the adapter's own. I tried "once its side" first; it drew distant 512 m regions whole from the street and cost about 1 ms there (3.6 ms against 2.2–2.7). At twice, the street is as it was.

A new check runs in every world: from three camera places, after loading, after edits, after the upgrade and after reconnecting, the regions drawn hold every instance the cells hold, each once, with nothing owed. It held everywhere (54 of 54).

---

## 2. Method

- **Before:** the Pass 18 results (`results-18.json`), and a control in this build with one level, for four worlds (`results-19-flat.json`). The control reproduces Pass 18's unit counts exactly and its frames within noise.
- **After:** all nine worlds, one uncapped run each (`results-19.json`). The heaviest world (no two alike, 100 villages) was run four more times because of Finding 5; its row is from the last run.
- Two measurements were added: the eye flying from over the whole world down to the street (twice), and a drag with the whole world in view, where the largest regions are the ones being re-laid.

Single runs: differences under about 15% are not findings.

---

## 3. Measurements

### Draw units and idle frame (100 villages unless noted)

| | plain | alike | kit | no two alike | kit, 400 |
|---|---|---|---|---|---|
| Units laid after loading: before → now | 632 → 202 | 1,656 → 534 | 20,707 → 7,524 | 37,530 → 36,850 | 82,648 → 9,657 |
| Draw calls, whole world in view | 633 → 44 | 1,657 → 115 | 20,708 → 2,891 | 37,531 → 36,627 | 82,649 → 2,936 |
| Idle frame, whole world in view | 2.1 → 1.9 ms | 15.7 → 15.7 ms | 38 → 15.6 ms | 60–97 → 57–88 ms | 175 → 57.5 ms |
| Idle frame, over the first villages | 2.1 → 1.8 ms | 15.5 → 15.6 ms | 39 → 15.6 ms | 97 → 56–86 ms | 176 → 57.9 ms |
| The same, one instance per unit | 0.65 → 0.47 ms | 1.7 → 0.6 ms | 37.5 → 9.9 ms | 96.5 → 72–84 ms | 174 → 14.2 ms |
| Idle frame, standing in a street | 0.6 → 0.6 ms | 2.2 → 2.2 ms | 2.7 → 2.5 ms | 9.3 → 7.5–10 ms | 3.3 → 3.4 ms |
| Draw calls in the street | 93 → 93 | 242 → 242 | 2,172 → 2,172 | 3,558 → 3,558 | 2,172 → 2,172 |

At 20 villages nothing changes (4,215 units, 4.1–4.6 ms with a kit): the whole world is within 2 km of the eye from every place measured.

The "no two alike" ranges are run-to-run spread in that world, not an effect of this change; the control in this build measured 60 ms where Pass 18 measured 97.

### Opening

| | plain | alike | kit | no two alike | kit, 400 |
|---|---|---|---|---|---|
| First things on screen | 0.35 s (as before) | 0.07 → 0.15 s | 0.16 → 0.16 s | 0.09–0.9 s | 0.19 → 0.19 s |
| Landed and editable | 1.39 → 1.14 s | 0.96–1.27 → 1.02 s | 1.33–1.9 → 1.38 s | 4.3–5.3 → 3.9–5.4 s | 5.5–6.2 → 4.9 s |
| Everything drawn with its content | 1.39 → 1.14 s | 0.96–1.28 → 1.03 s | 1.67–2.0 → 1.41 s | 25–34 → 21–37 s | 5.9–6.5 → 5.0 s |

Where two "before" figures are given, they are the control and Pass 18. Opening is unchanged or slightly faster (fewer units to lay). I would not claim the improvement from single runs.

### Editing and regeneration (whole exchange, ms; control → now)

| | alike | kit | kit, 400 |
|---|---|---|---|
| Edit a window | 1.5 → 1.9 | 1.7 → 1.9 | 2.7 → 3.2 |
| Move a house | 1.6 → 2.0 | 1.9 → 2.2 | 2.3 → 3.4 |
| Move a tree | 1.1 → 1.3 | 1.2 → 1.3 | 1.6 → 2.2 |
| Drag step | 1.3 → 1.8 | 1.4 → 1.7 | 1.8 → 2.0 |
| Drag step, whole world in view | 1.3 → 1.8 | 1.4 → 1.8 | 1.9 → 3.0 |
| Undo | 2.0 → 2.2 | 3.3 → 4.2 | 3.7 → 6.0 |
| Regenerate a village | 13 → 14 | 50 → 52 | 50 → 59 |
| Regenerate a grove (new seed) | 3.8 → 3.8 | 6.4 → 7.4 | 5.7 → 7.7 |

Core time is unchanged in every row (0.5–1.0 ms for edits, 7–8 ms for a village). The added time is the adapter laying larger units.

### Everything built again, and reconnecting (control → now)

| | alike | kit | kit, 400 |
|---|---|---|---|
| Upgrade, through to landing | 2.1 → 2.0 s | 2.1 → 2.2 s | 21.5 → 21.6 s |
| Editor frame meanwhile | 16 → 16 ms | 67 → 27 ms | 261 → 67 ms |
| Stop, as the editor sees it | 32 → 42 ms | 104 → 64 ms | 185 → 124 ms |
| Reconnect: everything drawn | 0.29 → 0.30 s | 3.4 → 3.7 s | 13.7 → 16.5 s |

### Memory (MB, after loading; control → now)

| | alike | kit | kit, 400 |
|---|---|---|---|
| Editor process | 702 → 859 | 970 → 930 | 1,977 → 1,343 |
| Editor's own allocations | 133 → 168 | 295 → 215 | 992 → 431 |
| Video memory | 68 → 105 | 100 → 144 | 223 → 224 |

Plain 100: editor process 771 → 785. Host memory is unchanged in every world, as it must be.

### The eye moving (whole world down to the street, 120 frames)

| | kit, control | kit, now | kit 400, control | kit 400, now |
|---|---|---|---|---|
| Worst frame, first time | 83 ms | 48 ms | 362 ms | 58 ms |
| Worst frame, second time | 42 ms | 16 ms | 195 ms | 59 ms |
| Units laid afterwards | 20,707 | 10,983 | 82,648 | 15,324 |

---

## 4. Findings

### Finding 1. Draw units no longer limit a kit world (measured)

Draw calls with the whole world in view fell 7 times at 100 villages and 28 times at 400. The count now follows meshes × the few large regions in view, and no longer grows with the world: 2,891 at 100 villages, 2,936 at 400. The frame with one instance per unit fell from 174 ms to 14 ms at 400 villages.

### Finding 2. What remains in the far view is triangles (measured)

A kit world at 400 villages still takes 57.5 ms with everything in view, and that is 378 million triangles: all-alike at 100 villages draws 94 million in 15.7 ms, the same rate. Grouping cannot reduce this. Only simpler stand-ins at distance can, and those are content (another output of the same request), so it is not an adapter-only matter. At 100 villages the frame is 15.6 ms and this does not bite yet.

### Finding 3. "No two alike" is not helped, and cannot be by grouping (measured, expected)

36,850 units for 36,503 meshes. You set this as a bound, not a target; it stays one.

### Finding 4. The cost is in edits seen from afar, undo, and reconnect (measured)

- **Edits** are 0.2–0.5 ms slower at 100 villages and 0.2–1.2 ms at 400. A change under a 2 km region copies and uploads that region's whole list for each mesh touched. The worst measured is a drag with the whole 400-village world in view: 3.0 ms a step against 1.9.
- **Undo** at 400 villages: 6.0 ms against 3.7, for the same reason over about 20 cells.
- **Reconnect** is 10–20% slower with a kit (16.5 s against 13.7 s at 400 villages). Content arrives in many frames, and each re-lays the large regions being drawn. This is the same path as Pass 18's Finding 5 (unbounded intake) and would be bounded with it.
- **Editor memory** rose in the all-alike world (process +157 MB, own allocations +35 MB, video +37 MB) and fell sharply in the 400-village kit world (−634 MB). The rise is the instance lists kept at three levels, and units of both sizes existing once the eye has moved. I have not accounted for all of the +157 MB: the process figure is a working set and includes allocator high-water marks.
- **Units laid and not shown** stay until their region is emptied. After flying the eye across the 400-village world, 15,324 units existed and about 3,000–9,700 were drawn. Nothing lets the others go. It is small (instance data only, 48 bytes each) and I left it.

None of these comes near a frame. All are in the adapter.

### Finding 5. After a stop, the editor waited for content nothing named (found by the benchmark, fixed, adapter only)

In the heaviest world one check failed in three of four runs: after a stopped upgrade, the editor was still waiting for about 1,000 pieces of content while the host said it owed nothing. The cells were identical to before the stop; the world was unchanged.

The cause: the stopped build's cells had been shown, and had named new content, which the editor asked for. When the build was withdrawn, the host rightly stopped making that content. The editor kept waiting for it on behalf of cells it no longer held. Nothing was drawn wrongly, but "is everything here?" could never become true.

The fix is in the adapter: a cell that is replaced or removed stops waiting for what it named. After it, the check passed in the one run made; I cannot show from one run that the race occurred in that run, only that the stale waits can no longer exist. The control with the old grouping passed in its one run, so I believe this is timing, not this pass's change, and that it was latent in Pass 18. The host still keeps the names in its own "owed" list after a stop; that is harmless (they are sent if they are ever made) but it is the host-side half of the same looseness and belongs with content lifetime.

### Carried, unchanged

Content never let go; unbounded content intake in the editor; the pin note (approved, not yet built — see decisions); stop granularity at the owner; square-on boxes around turned things; the rest of Pass 18's list.

---

## 5. Remaining risks, by severity

1. **Content lifetime and intake (medium).** Now the first item. Unchanged from Pass 18, and reconnect's slowdown here adds a little to it.
2. **Triangles in the far view (medium-low, new as a separate item).** Finding 2. Bites at 400 villages, not at 100.
3. **Things as Godot nodes; slow content; opening large projects (as Pass 18).**
4. **Draw units (low, was medium-high).** Addressed for kit worlds. The two numbers are tuned on these worlds only: one layout, one machine.
5. **Edits under large regions (low).** Finding 4. Would grow with denser worlds or larger top regions.

---

## 6. Decisions for you

1. **Accept the levels as the adapter's way of drawing together?** I recommend yes, with 4 / 16 / 64 cells and "twice its side" as defaults, on the understanding that they are adapter settings and not rules of the engine.
2. **What next.** I recommend the pin note first (you approved it; it is small and touches the core, so it should be its own reviewed step), then content lifetime and bounded intake as one slice, including the host's "owed" list after a stop. The alternative is far stand-ins for triangles, which I would defer: it needs a content decision and only matters at 400 villages.
3. **Finding 5's fix** is already in. Say if you would rather it had waited for review; it is ten lines in `world_view.gd`.

I have not started anything beyond this.

---

## Appendix. What changed, and running it

- `editor/godot/world_view.gd`: regions in levels, chosen from the eye each time it moves; lists and units laid on demand; `drawn_once()` and `only_one()` for the scripts; a replaced cell forgets what it waited for.
- `editor/godot/main.gd`: the label shows units shown and laid; no list of cells to lay later.
- `editor/godot/weigh.gd`: `--levels`, `--near`; units shown per view; the drawn-once checks; the fly-through; the far drag; the parts of the after-stop check recorded separately.
- `editor/godot/measure.gd`: its grouping sweep sets one level.
- `editor/run.ps1`: `-Levels`, `-Near`.

```bash
editor\run.ps1 -Weigh
```

```bash
editor\run.ps1 -Weigh -Levels 4
```

Results: `editor/godot/results-19.json` (now), `results-19-flat.json` and `results-19-own-flat.json` (one level, this build), `results-19-near1.json` (once its side), `results-19-near2.json` (the three-world trial of twice), `results-18.json` (before).
