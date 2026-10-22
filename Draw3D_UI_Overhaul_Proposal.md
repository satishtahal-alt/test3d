# Draw3D — a proposal for one consistent editor UI (2026-10-15)

Written after the Character Studio playback fix and its usability pass (amendment 7aw),
from the editor as it is now. Nothing here is built except where it says so. It asks for
decisions; it does not redesign anything by itself. `Draw3D_UI_Inventory.md` remains the
list of every control; this is about how they are organised.

## 1. What is wrong now, by example

**Windows are as tall as everything in them, and taller than the screen.** `win.gd`
makes a window "never smaller than what it holds needs", and most panels stack fixed
minimum sizes:
- Makers: 132 + 110 + 190 px of minimums before any of its parts (`makers.gd`).
- Library: 250 + 230 px (`library.gd`).
- Objects: 120 + 200 px (`designs.gd`).
- The Character Studio was 260 px of preview, 300 px of tabs and rows between them:
  more than a 720 px screen, so its tabs ran off the bottom.

So resizing is often the only way to reach a control, and a window cannot be made
smaller than its worst case.

**Feedback text sits in the flow and moves everything under it:**
- The Studio's "Done: …" and status lines wrapped to two or three lines and pushed the
  tabs down. *Fixed this pass.*
- Objects shows the brush's settings only while the brush is in hand. A comment in
  `designs.gd` records that showing them always "pushed what is under them out of a
  window on a small screen".
- The bar's context row is the good precedent: one line, cut short with "…", the whole
  of it in a tooltip (`desk.gd`).

**Panels keep their own "chosen" while one tool acts.** There is one brush
(`marks.gd`, tool `strew`), but:
- Objects has its own chosen design, and the Plant Generator its own shown plant.
- Choosing in either list did not change the brush. After painting crates, choosing a
  plant preset left crates on the brush, and only the long context row said so ("…
  PAINTING WITH crate…"). *Fixed this pass*, see section 4.
- The Studio had the same fault inside one window: Play replayed the last thing tried
  while another list showed something else chosen. *Fixed this pass.*

**One task, many doors** (inventory section 11):
- "Put one thing down" has five routes.
- "Shape something" has six.
- "Select a maker" has four.

Each route shows its own settings and words, so a person learns several partial
pictures of one thing.

**Offered without regard to whether it makes sense:**
- Objects offers "Paint with it…" for every design without bones, including the
  physics library's pendulum (`library/physics.txt`). A pendulum is pinned 2.5 m up and
  swings when loose; a field of them painted on the ground is not what anyone means.
  Its painting was not tried by hand here; the code offers it like a rock.
- The Plant Generator's "Paint with it…" first makes the plant a design, as a side
  effect, before the brush is taken up.

**Explanations always on screen:**
- The Studio's Movement, Combos and Change a phase tabs opened with three-line
  paragraphs. *Shortened this pass; the detail went into tooltips.*
- Makers' parts and Objects' detail still do this.

**Lists without context:**
- Long lists (Objects' designs, Library shelves, Makers) scroll the chosen row out of
  sight, while the detail below names it.
- Search is in some lists (Studio clips, Library) and not others (Makers, Objects).

## 2. The system proposed

1. **Four levels, each with one job.**

   | Level | What it is | Examples |
   |---|---|---|
   | The workspace | The view, plus the bar (world, edit, view, play, and the tool in hand) | — |
   | Tool categories | The left toolbar. A click takes up a tool or opens its short list | — |
   | Contextual panel | One at a time: the settings of the tool in hand, or of what is selected | Today's Inspector and brush settings |
   | Authoring windows | Floating, opened on purpose | Character Studio, Signals, Sounds, Physics, Library, Plant Generator |

   Secondary dialogs (New world, a question asked) are modal and small.

2. **One active target, shown everywhere.**
   - What is in hand is one piece of state, the one `marks.gd` already keeps: the tool
     (place, paint, clear, replace, erase), the asset or group, and its settings.
   - The bar's chip says it in a few words: `PAINT · crate · 6 m`.
   - Every browser marks the asset in hand.
   - Choosing in any browser while a tool is in hand either retargets that tool or puts
     it down, and says which.
   - No panel keeps a private "chosen" that a tool reads later.
   - The Studio's "Play: …" line (7aw) is the same idea for playback.

3. **Choose, then configure, then apply — kept apart.**
   - *Choose* happens in a browser.
   - *Configure* happens in the contextual panel: the asset's own settings (size,
     variation) and the tool's settings (brush width, spacing).
   - *Apply* happens in the view: click to place, drag to paint.
   - Generators (Plant Generator, Tree Generator) make assets: their result lands in the
     browser, chosen, and painting always goes through the one brush.

4. **One panel standard** (the pattern the Studio now follows):
   - **Height:** at most the room under the bar. Content scrolls inside one
     ScrollContainer, never nested scrolls.
   - **Order:** primary controls first; advanced settings in a folded
     `FoldableContainer`.
   - **Status:** a fixed one-line status area at the top or bottom, cut short with "…",
     the whole text in a tooltip. It does not grow.
   - **Explanations:** in tooltips or a "?" toggle, not paragraphs in the flow.
   - **Sizing:** a remembered size per window, and "Put every window back" restores it.

5. **One list pattern for long lists:**
   - search at the top, filter chips (tags, shelves), then the list (Tree or ItemList,
     with thumbnails where there are pictures);
   - the chosen row kept in view;
   - a short summary of the chosen item pinned under the list, not scrolled away.

6. **One workflow for clips, moves, combos and phases** (built in the Studio):
   - Selecting chooses what Play plays.
   - The timeline is where a phase is chosen.
   - A phase is edited in three labelled parts:
     - **Look**: this character's animation only;
     - **Timing**: the move, for everyone with it;
     - **Movement**: the move, for everyone with it.
   - A scrubber looks at any moment of the phase.

7. **Reuse Godot's own controls:** TabContainer, FoldableContainer (Godot 4.5+, used
   now), H/VSplitContainer, Tree, ItemList, PopupMenu, tooltips and themes, through the
   project's existing `desk.gd`/`win.gd` windows. No new UI framework. Godot has no
   docking at runtime, so docking (section 3, item 9) would be ours to build.

**What should stay separate, because the tasks differ:**
- Character authoring (Studio) and world building.
- Physics, Signals and Sounds, which each watch a running world.
- The play-mode bar.

They share the panel standard and the target model, not a window.

## 3. In order

### Low risk, soon (one pass, or two)
1. **Windows bounded by the screen** (`win.gd`): a window's height is at most the room
   under the bar, and its content scrolls in one ScrollContainer. This is the single
   change that most reduces "maximise to use it".
2. **A fixed status line helper** (one line, "…", tooltip) used by Objects, Makers,
   Library and Plant Generator for their feedback, as in the Studio.
3. **The bar's chip names what is in hand:** `PAINT · crate`, `PLACE · house`,
   `ERASE · painted`. Each browser marks the in-hand asset in its list.
4. **Fold** Makers' and Objects' long explanations and advanced parts into
   FoldableContainers and tooltips.
5. **Offer painting only where it means something:** not for pinned or loose physics
   kinds such as the pendulum; say why instead ("place it one at a time").

### Medium (a pass or two each, after the above)
6. **One asset browser:** Objects, Library and the plant presets as tabs or filters of
   one window, with one selection and Place / Paint acting on the shared target. The
   Plant Generator becomes "Make a plant…", whose result appears in it.
7. **A contextual panel docked at the right edge** for the tool in hand: brush and
   placement settings, in place of each window's own copy of them.
8. **One main route per task** in inventory section 11, the others kept as shortcuts
   that open the same panel. This needs the user's review of the inventory first (a
   standing rule).

### Larger, each a decision of its own
9. **A docked layout** (browser left, contextual panel right, timeline bottom) in place
   of floating windows. Godot has no runtime docking, so it is a component to build and
   maintain.
10. **A real animation timeline** (keys, curves, layers) beyond the Studio's phase
    scrubber. It overlaps the parked animation work: IK, layering, keyed bends.
11. **The renaming pass** the inventory was written for: still waiting on the user's
    review.

## 4. Done in this pass, as a first step toward the above

- **Character Studio:**
  - a play target;
  - a fixed status area;
  - a split between the preview and the tabs;
  - a clickable timeline with the phase being changed outlined;
  - Look / Timing / Movement / More sections;
  - a phase scrubber;
  - shorter always-visible text.
- **Objects:** a row clicked while the paint brush is in hand moves the brush to it, or
  puts the brush down if it cannot be painted, and says so (`designs.gd`
  `_brush_follows`).
- **Plant Generator:**
  - Its "Paint with it…" now says when it is painting ("Painting with 'X' (Esc ends
    it)") and puts the brush down when pressed again.
  - Showing another plant, or changing its numbers, puts its brush down and says so.
  - A brush taken up in Objects is named on a fixed line in the window
    (`grow.gd` `_say_brush`).
