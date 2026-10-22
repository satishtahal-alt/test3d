# Draw3D — weapons, equipment, actions and controls: audit and staged plan

2026-10-16, amendment 7ba. The user asked for reusable weapons, equipment, accessories, actions, input mappings and loadouts: first an audit, then straightforward first steps, with larger pieces put forward for approval. This file records:

- what was found;
- what was built in this pass;
- the stages proposed after it;
- the decisions that need the user.

Reading guide:

- **Script-checked** means a headless check proves it (`chartest.gd`, `studioplay.gd`), or the studio tour ran it and took a picture.
- **Hand** means a person used it. Nothing in this pass has been used by hand.
- Visual quality (grip, weight transfer, trajectory) is never claimed from a passing check.

## 1. Audit: what exists

| Capability | Where | State |
|---|---|---|
| Character definitions: model, bone map to Godot's humanoid profile, clip libraries, look, set, points | `characters.txt`, `charbook.gd`, `animated.gd` | Implemented, script-checked |
| Clips carried between skeletons (baked, rest-relative) | `clips.gd` | Implemented, script-checked; seen in tour pictures, never by hand in motion |
| Libraries: Draw3D poses, Quaternius UAL 1 and UAL 2 (CC0), Mixamo import | `characters/`, `Import clips…` | UAL 1 and 2 implemented. Mixamo checked only on generated FBX files; no real Mixamo file was ever on the disk |
| Moves, phases, the shared runner, `when`, `lets` windows, input buffer, `rests`, `upon` | `moves.txt`, `moveset.gd`, `move.gd`, `motor.gd` | Implemented, script-checked |
| Combos | `combo` in `moves.txt` | Before this pass, played only by the Studio. Since this pass, also asked in the game (below) |
| Strikes from a place on the body or on what is held; being struck | `strike.gd`, `upon struck` | Implemented, script-checked |
| A thing held at a bone (`holds FILE at BONE`) | `characters.txt` | Implemented (the samurai's katana). Kept as the older form; `gear` replaces it for new work |
| Things in the world that are kept, held and used | `things.gd` (`kept`, `held`, `does MOVE`; keys 1–9, `out`, `use`) | Implemented (stage 6), tour-checked. A separate system from a character's gear: nothing joins the two yet |
| Input: one table into Godot's InputMap; keyboard and controller always both heard; the controls card | `controls.gd` | Implemented. Before this pass it was code only, with no remapping |
| Persistence: texts kept with the world by the host, entry-level document edits, Ctrl+Z | `book.gd`, `written.gd`, host | Implemented; `gear.txt` uses it unchanged |
| Two-hand grip, IK on clip-shown figures | — | Missing. Godot 4.7 has `TwoBoneIK3D`, `FABRIK3D`, `CCDIK3D` and `LookAtModifier3D` natively (checked with `ClassDB`). `figure.gd` has its own two-bone reach, for posed figures only |
| Ranged firing, aiming, projectiles | — | Missing. Loose physics things exist (`loose.gd`) and could be fired |
| Equipment slots, inventory rules, loadouts drawn from the world inventory | — | Missing |
| Weapon properties used by gameplay (reach, weight, damage) | — | Missing. `reach` and `weight` are written on items; no move reads them yet |
| Clips for other weapons in the libraries on disk | UAL 1: Pistol (aim, idle, reload, shoot), Spell (enter, idle, shoot, exit), punches. UAL 2: Shield (idle, dash, one-shot), Sword_Block, OverhandThrow | Available; no moves use them yet |

## 2. Built in this pass (7ba)

All of it is script-checked. Nothing has been used by hand, and no real controller was plugged in.

### Buttons

- **What presses each button.** Characters > Buttons, or the Character Studio's Buttons tab:
  - A **Keyboard and mouse / Controller** toggle. The two are kept apart, and switching loses nothing.
  - **Change…** and **Add another…**: press the key, click or controller button wanted. Esc leaves it as it was.
  - **Nothing**, and **As Draw3D has it**.
  - Two buttons on the same key are marked in orange, with the other's name in the tooltip. Both are still asked; it is not refused (jump and long jump already share Space).
- **Buttons of your own** ("attack", "heavy"). A move or combo can be put on one.
- **Where buttons are kept.** Your buttons are yours, for every project. `controls.gd` reads `controls.txt` in Draw3D's own folder (`%APPDATA%\Draw3D`) over its built-in table, and every change is written at once. A scripted run uses a folder of its own and never reads or writes yours.
- **The new `sheathe` button:** B on the keyboard. No controller button is free, so give it one in the Buttons tab.

### Moves and combos on a button, and when

- **On a chosen button**, the Buttons tab shows this character's moves and combos asked with it, in the order they are tried. From there you can:
  - choose **Only when**: any time, on the ground, in the air, falling, crouched, crouched on the ground, running, standing still, moving;
  - choose **held**: run, crouch, or a button of your own held down at the same time;
  - move one **Earlier** or **Later**;
  - **Take it off**;
  - **Put** any move or combo on the button.
- **New when-words:**
  - `holding BUTTON` (another button held as this one is pressed);
  - `drawn`, `sheathed`;
  - `with ITEM_OR_SORT` (carrying or wearing a thing, or a thing of a sort);
  - `following` (pressed only inside a window that the move going opens, never from rest). It ends the 7az gap: the second cut no longer starts by itself in the first cut's rest.
- **Combos in the game.** A combo can be in a set as a move is, with `asks BUTTON` and `when`:
  - one press begins its first move;
  - each next move is its button again in the window of the move before;
  - with **Plays by itself** (`plays by itself`), one press plays the whole combo.

### Profiles

**Save as a profile** keeps your buttons and the chosen character's set, with every move and combo it needs (including what those let in, or are written `like`), in `%APPDATA%\Draw3D\profiles\NAME.txt`. **Use this profile** in any world or project makes the buttons yours and writes the moves into that world (Ctrl+Z each).

### Gear (`gear.txt`, `gearbook.gd`)

- **21 places on a body**, named as a person would:
  - the hands, the left forearm;
  - the left and right hip;
  - the back, from the right or the left shoulder, and the small of the back;
  - the chest, the thighs;
  - the head, the face, the neck;
  - the belt, the waist, the right of the waist;
  - the pack, the shoulders, the feet.

  A place names a humanoid bone and a spot in the body's own frame, so it suits any rig mapped to Godot's profile. Hand places are in the hand bone's own frame.
- **34 things**, each of a sort (40 sorts) and each made of simple parts (box, rod, cone, ball, disc, ring), except the katana:
  - **Blades:** katana, short sword, dagger, greatsword.
  - **Long shafts:** spear, staff, halberd.
  - **Heavy at one end:** axe, war hammer, club, wand.
  - **Ranged:** bow, crossbow, slingshot, pistol, rifle, shotgun, grappler.
  - **Shield and tools:** round shield, shovel, pickaxe.
  - **Worn:** hat, helmet, glasses, amulet, backpack, jetpack, pouch, belt, quiver, cape, gloves, boots, hover shoes.

  A thing can say:
  - `model` (a glTF file instead of parts), `grip`, `second` (the other hand's grip);
  - `point` (a place a move strikes or shoots from);
  - `worn`, `stows`, `pair` (worn on both sides);
  - `reach`, `weight`, `size`, `colour` (a tinted variant through `like`);
  - `from`, `licence`.
- **On a character:** `gear ITEM [at PLACE] [x y z] [turn x y z] [stowed PLACE [x y z] [turn x y z]]`.
  - The nudge and turn are this character's own, written apart from the thing, so the thing stays as written for everyone.
  - The swordsman now carries the katana this way: drawn in the right hand, put away at the left hip.
- **Drawn and put away:**
  - The motor keeps `drawn`.
  - A phase that `draws` or `sheathes` changes it; the figure moves what is carried between the hand and its stowed place.
  - The swordsman's `draw_blade` and `sheathe_blade` are the sheathe button either way (the sheathe is the draw clip played backwards).
  - The cuts want `drawn`.
- **Moves that need gear**, as examples:
  - `jet`: jump again in the air with a jetpack worn, and it flies up for 1.5 s;
  - `hover`: jump as it falls with hover shoes, and it glides down.

  Without the gear, neither begins.

### The Gear tab (Character Studio)

What the character wears and carries is listed. For the thing chosen you can:

- choose **Where** and **Put away at** by place name (no bone names, no coordinates);
- **Nudge** in cm and **Turn** in degrees: seen at once and written a moment later;
- **Reset** the nudge and turn;
- **Draw / put away**: by the character's own move if its set has one, otherwise at once;
- **Take it off**.

Below it is every thing in `gear.txt`, filtered by sort, with **Put it on**. For a two-handed thing it says how far the other hand is from its second grip: the mannequin's left hand stands 74 cm from the spear's.

### Edit tab, without scrolling

A phase's numbers are in two columns. Most numbers have a short slider beside them; the clip part is two plain numbers.

- **Look:** what shows the phase, Choose a clip…, the clip part from and to, whole clip, backwards, mirrored, and the button the move is asked with.
- **Timing and movement:** lasts, blend in, goes, turns, strikes with and how far.

Under the columns, the pose has one row (part, tip, twist, lean, reset). The long explanations are folded under Details, and + Phase and − Phase sit beside the phases.

Measured: a phase's numbers need 254 px, against 265 available with the Studio at 900 px; side by side, the whole Edit tab is in sight at 1280×720 (tour picture). The Edit tab's least width is 619 px. The Studio's least width stays 703 px, and the Gear tab scrolls.

### Checks

- **chartest, 119 of 119.**
  - Gear: every thing made; the katana's place in the hand and at the hip; the sheathe button both ways; no cut while it is put away; worn things where they belong; pairs; the pistol at the thigh; the jetpack move only with a jetpack.
  - Combos and buttons: a combo on its own button, by itself and pressed; `holding`; the rest-press gap closed.
  - Bindings: kept apart per device, written and read back, InputMap, reset.
- **studioplay, 93 of 93.**
  - Buttons tab, by clicks and by keys and buttons pushed into Godot (not the computer's): a key and a controller button captured, Esc, a button made, a combo put on it and made to play by itself, when and held changed, a profile saved and used.
  - Gear tab: put away by the move, nudged and written, reset, put on and taken off, the spear's gap.
  - Edit: all of a phase's numbers in sight; Play stays put with Details open and scrolled.
- **Studio tour:** clean, with pictures 19–27.
- **Moves tour:** clean.

## 3. Proposed stages after this

| Stage | What | Depends on | Size |
|---|---|---|---|
| G2 | Hands that hold: the other hand brought to a two-handed thing's `second` grip by Godot's `TwoBoneIK3D` on clip-shown figures; a grip correction per bone map, so hand places suit Quaternius's own model and Mixamo's rig as they suit the mannequin | 7ba | One pass. **Needs approval (A1)** |
| G3 | Handles in the Studio's preview: the chosen thing dragged along its axes and turned by rings, as `handles.gd` does for meshes; Front/Side/Back already there | 7ba | One pass. Fits the current architecture; can go ahead when asked |
| G4 | Moves that suit their gear: a thing's sort names the moves that fit it (a starting set per sort); `reach` and `weight` scale a move's numbers (a heavy sword's wind-up and recovery longer); moves for the clips on disk (pistol aim, shoot and reload; spell; shield block; overhand throw) | G2 for two-handed moves | One or two passes |
| G5 | Ranged: a phase word that fires a loose thing from a point (`shoots POINT THING SPEED`), aiming from the eye, a grappler's line pulling the body (`to` a hit point) | `loose.gd`, G4 | One or two passes. **Needs approval (A2)** |
| G6 | Gear in the world's inventory: a kind may be a thing of `gear.txt`; picked up, it joins the walker's loadout; equipment slots by place; a game's own rules (stacking, durability) optional and off by default | `things.gd`, 7ba | Two passes. **Needs approval (A3)** |
| G7 | Profiles carry gear (the things and the character's `gear` lines) and a gear library shared across projects (beside `editor/library`) | 7ba | Small. Waits on decision A4 |
| G8 | Input beyond one button: sequences (down, forward, press), chords pressed together, and button icons | 7ba | One pass, when wanted |

## 4. Decisions for the user

### A1. Two-handed grip by IK (G2) — approved, smallest version built 2026-10-16 (7bb)

Built: the other hand brought to a two-handed thing's second grip by `TwoBoneIK3D` (with
a pole) while drawn; `free_hand` turns it off; the Gear tab's box writes it. Within reach
the palm lands on the grip (1.5 cm). From the clips on disk the grip is beyond the other
arm's reach (they hold things in one hand, the hand at the side): the hand comes as far as
the arm goes. **A1b — built the same day (7bc):** a thing's `two_hands_at PLACE` holds it before the body,
both hands brought to its grips by IK; every two-handed thing of gear.txt lands both
hands within 0.4 cm on the mannequin. **A1c, for the user:** a hold that moves with a
move's phases (a cut with a greatsword), or clips made for two hands. Proposed then: a two-handed stance (the held hand also brought before
the body by IK to a hold place, per thing) or clips made for two hands. The grip
correction per rig (part of G2) is not built.

As proposed:

- **Benefit:** the spear, greatsword, rifle, crossbow and staff are held with both hands through every clip, rather than the left hand hanging (74 cm off today).
- **Effort:** one pass, using Godot's own `TwoBoneIK3D` and adding nothing of our own.
- **Risks:**
  - each rig's hand needs its grip turned right;
  - IK over a clip can bend an elbow the wrong way without a pole target;
  - only a person looking can judge it.
- **Alternatives:** clips made for two hands (UAL has few); posing the left arm per phase with `bend` (tedious, and drifts).
- **Smallest version:** the left hand only, only while drawn, only for things with a `second` grip, with on/off per character.
- **Delays:** nothing in progress.

### A2. Ranged firing (G5)

- **Benefit:** bows, guns, slings and the grappler do what they are.
- **Effort:** one or two passes.
- **Risks:**
  - aim from a third-person eye is a design choice;
  - many loose things per second cost frames.
- **Smallest version:** a phase fires one loose ball from a point, at a speed, along the facing; hits are reported as strikes are.
- **Alternatives:** an instant ray hit (no projectile).

### A3. Gear in the inventory and slots (G6)

- **Benefit:** things found in the world become what a character wears and carries.
- **Effort:** two passes, touching the host's save of what is kept.
- **Risk:** RPG rules leaking into every world. Keep them optional.
- **Smallest version:** a thing kept that is gear can be put on or taken off from the 1–9 keys, at its usual place.

### A4. Where gear lives

- **As built:** `gear.txt` is the world's, as `moves.txt` is, and the host keeps it with the world.
- **The choice:**
  - also keep a shared gear library across projects, as the asset library is shared;
  - and/or carry gear in profiles.
- **Recommended:** profiles carry it (small), and a shared gear library comes with G6.

### A5. How a character stands with its sword put away

Today a look has one `stand`, so the swordsman keeps the sword stance with the sword at the hip.

- **Smallest version:** look rows may say `stand/sheathed`, shown while sheathed.
- **Effort:** small.

## 5. The next smallest high-value task

Try the Gear and Buttons tabs by hand (the checklist walks through both), and say yes or no to A1. Two-handed grip is what most of the weapons in the library are waiting on, and Godot already has the IK node for it.
