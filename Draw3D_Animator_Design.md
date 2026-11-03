# The Animator: one editor for how anything moves (2026-10-29)

The user, after the posing pass failed by hand for the third time: keep the Character Studio
and the Animation window as they are; make a new way to edit animations, moves and combos;
edit the animation of anything animated; pose a part of a move by dragging; edit every move
and phase of a combo; choose the buttons; preview, test live, save, and have the game use it.
"From simple walking to complex combos. This is the bare minimum."

## Why the old tools failed

The runtime is sound (keyed `bend BONE x y z local at P` on a look row is played by the game,
the reach maths works). What failed is the host: the posing handles lived inside the Studio's
8,000 lines of shared state and were shown only when seven things all held at once (Simple
mode, a move card opened with ✎, Pose pressed, a move and phase in `editing`, a humanoid
rig, a sequence built, nothing refreshed since). Any one missing and the handles were gone,
with no word why. The Animation window was a separate narrow window, and a flock's sheep
(made by a maker, not placed) gave it no design, so it opened empty.

## Principles (from the research: Cascadeur, Blender, Akeytsu, Unreal Montages, Dustloop
frame charts, MUGEN hitboxes, Godot 4.6+ IK)

1. One full-size window, its own preview and its own state. No hidden switch: what is chosen
   is posed, and the handles are always on the figure while a frame is shown.
2. Three columns: **What** (anything that animates), **Its actions** (getting about, moves,
   combos), and the big **preview** with the **phase strip** under it and the **settings** of
   the chosen phase beside it.
3. The phase strip (Dustloop frame chart / Unreal Montage): every phase of the move as a
   coloured cell as long as it lasts, in frames; the hit window red; key diamonds on it; a
   playhead to drag; a cell's right edge dragged sets its time. A combo shows its moves as
   cards above, each card opening its own phases in the strip.
4. Posing (Cascadeur AutoPosing, Akeytsu hot points): big handles on hands, feet, chest,
   head and what is held, dragged with IK; a small dot on every other bone, turned by
   dragging (any skeleton, not only people). Each drag is a key at that frame; between keys
   it goes from pose to pose. Ghosts of the frames before and after; the path of the chosen
   part. Mirror, copy and paste a pose, take a key away, clear the phase's keys.
5. Additive, never destructive (Unreal additive layers): keys lie over the clip; the clip is
   not changed, and clearing the keys gives it back.
6. Settings by forms, grouped in plain words: Timing, Animation, Movement, Hit, Reaction,
   Effects. Every phase word moves.txt knows has a field.
7. Buttons: the move's button picked from a list that leads with the controller's button;
   held buttons and when it can begin, in words.
8. Play (the move as the game plays it, on the spot, again and again, slow motion) and
   **Test with the controller** (the training yard inside the same preview: dummies, the
   real motor, the real buttons). Changes are kept as they are made (each one Undo-able) in
   the world, and the game uses them at the next press; Ctrl+S writes the world's file.

## Stages

- **A1 (this pass): characters.** What: every character. Actions: getting about (stand,
  walk, run, jump, fall, land, crouch…) and every move of its set, and its combos. Phase
  strip, posing on any bone, phase forms, the move's button, combos as cards (add, remove,
  reorder, button), Play, Test with the controller. Opened from Window, the right click
  ("Animate…"), the Inspector, and the Studio.
- **A2: creatures with clips of their own and bodies of bones** (dog, deer, sheep, horse):
  the same posing and strip, keys stored per way of going on the design, played by
  ownclips.gd / figure.gd over the clip or pose. The flock's animals found from the maker.
- **A3: mechanisms** (doors, propellers, wheels, `turns`, `glides`, wings): keyed turns and
  slides of a part over time.
- **A4: pose library, hit boxes drawn on active frames, trajectory of the weapon tip,
  breakdown slider and easing words.**
