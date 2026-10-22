# ActionForge: what it is, what we took, what to learn from it (2026-10-16)

ActionForge (https://actionforge.app, beta, by Hassan Charafeddine / CGstuff) is a browser
tool for previewing, adjusting, posing, layering and exporting character animation. It works
on Quaternius's human rig and animations, which are CC0. A second page, **Motion Lab**, makes
new walks from one walk.

**Licences.** The animations, rig and props are CC0: free for anything, sold games included,
with no credit needed (we credit anyway). The tool's code is "all rights reserved": no
copying, modifying or reverse engineering. So **we take its ideas, never its code.** Every
feature below would be built from Godot's own parts.

**Built (2026-10-16, amendment 7bj):** A1, A2, A3 and B4 to B7, at the user's word ("go all the way to b 7, whatever order lowers the cost"). B8 and C are not built. What was found on the way: no map named the finger bones, so no clip had ever moved a finger. That is why the hand stayed open on the sword.

## 1. The animations

**ActionForge's 85 clips are the two libraries we already have.** They are Quaternius's
Universal Animation Library 1 (`characters/quaternius/`, 45 clips) and 2
(`characters/quaternius2/`, 42 clips), with the same skeleton and bone names. They are
retargeted and renamed for the site. The site has no other genre sets.

Seven pieces were new, all CC0. I downloaded them to `editor/godot/characters/actionforge/` (`clips/` and `props/`)
(about 350 KB, with `License.txt`) and listed them in `editor/assets/CREDITS.txt`:

| File | What | Use for us |
|---|---|---|
| `fist.glb` | hand pose, fingers closed (tagged "grip") | **the sword grip**: fingers round the handle, the to-do left when characters were parked |
| `open_palm.glb`, `open_palm_natural.glb` | hand poses, open | empty hands, waving, pushing, reaching |
| `sword_heavy_combo.glb` | a 4.3 s heavy sword combo by Quaternius (not in our UAL 2 file) | a second combo for the swordsman |
| `Katana.glb`, `Training_stick.glb`, `magic_staff_01.glb` | props | gear to try grips on (we already have a katana) |

The hand poses move only the finger bones. Their bone names are UAL 2's (`index_01_l`, …), so
they lay over any clip on our characters without a map.

**Crouch and pick-up were never missing.** `Crouch_Idle_Loop`, `Crouch_Fwd_Loop`,
`PickUp_Table` and `Interact` are in UAL 1 and named in `characters.txt`. A move set that
does not list them simply cannot use them. That was the swordsman's case, mended in 7bi with
`stoop` and `reach`.

**More sets for other genres** (horse riding, two-handed and ranged weapons, ladders, driving,
emotes, creatures) must come from other sources:
- Quaternius's other packs;
- Kenney;
- Mixamo, which our importer already reads. Mixamo is free in sold games but may not be
  passed on as files.

Each would be checked against the rule: free for sold games, CC0 first, CC-BY when
credited, never non-commercial.

## 2. What ActionForge does well

| ActionForge | What it does | Draw3D today |
|---|---|---|
| **Clip grid** | A moving thumbnail per clip; search by name or tag; categories (aim, combat, gestures, interaction, locomotion, reactions); tick several; an arrow when a clip leads into another (Chest Open → …) | Studio lists clips by name; tags exist in `characters.txt` but are not a filter; no pictures |
| **Compose** | Layer clips by body region: the first is the base, each later one overrides the regions you give it (walk legs + lantern arms) | Not there: one clip plays the whole body |
| **Adjust** | Sliders added over any clip: arms up/down and in/out, posture, chest bend and twist, head; speed | Only speed and `stride` per clip |
| **Hand poses** | Finger-only poses that "the fingers take, and nothing else does" | Not there (the open hand on the sword) |
| **Props** | A prop in the left or right hand, or your own .glb dropped in | Ours is fuller: gear with grips, fits per model, put away, drawn, two hands |
| **IK** | Each limb pinned to root or hips; a pelvis handle; the prop's hand bone moved with a reach % | Two-handed grip IK only (`_arm_ik`); no feet on the ground |
| **Pose** | Click a joint, turn it, keyed on this frame; layers, X-mirror, snap, fingers, copy / paste / paste flipped, mirror L→R | Not there (keyframes await your decision) |
| **Picker** | A flat body and hand chart: click or drag over joints to pick them | Not there |
| **Import** | FBX/GLB on its rig, Mixamo, Unreal 4/5 or Reallusion; "root travels" or "hips carry it" | Mixamo only (7ay) |
| **Characters** | Your own rigged character, or an unrigged model rigged in the browser | Rigged models through `characters.txt`; no auto-rig |
| **Timeline** | Scrub; 0.25×–2×; loop; undo/redo; lock origin X/Y/Z/yaw (in place) | Play and loop in the Studio preview |
| **Stage** | A shot camera per animation: lens, frame shape, thirds and safe-area guides | Not needed yet (cutscenes later) |
| **Motion Lab** | One walk becomes many: Stroll, Brisk, Heavy or Sneak presets; sliders for stride, cadence, width, lift, bounce, sway, arms and lean; posture; follow-through (chest, head and arms lag; stiff, loose, heavy); all 8 directions generated; foot-plant curves; styles saved | One clip per gait; secondary motion only for capes etc. (`sway.gd`) |

Its usability habits are worth copying in the Studio too:
- every panel says in one line what it does;
- the side tabs open panels that stack, so several can be open at once;
- each panel has Reset;
- a hint under the view ("drag to orbit · scroll to zoom · space to play");
- "kept on this machine, nothing uploaded" stated where files go in.

## 3. What Draw3D could build from it, in order of worth

Each item is a tier of a named system (your rule), built from Godot's own parts.

**A. Small, and what you asked for first:**
1. **Fingers round the grip.** A hand pose (fist, open palm, natural) as a word on gear and
   on a move: `hand fist` and so on. It is laid over only the finger bones of the hand that
   holds the gear. It goes with the gear in hand and is let go when the gear is put away.
   It uses the downloaded poses; later, poses made in the engine.
2. **Bearing.** Adjust sliders kept per character: arms, posture, chest, head. They are
   added over every clip, so one walk serves a proud guard, a tired farmer and a hunched
   thief. In Godot that is one `SkeletonModifier3D` after the animation.
3. **A clip browser in the Studio.** Search, tag chips from `characters.txt`, the clip shown
   on the preview when hovered, and the arrow marks for clips that lead on.

**B. Medium:**

4. **Moves by body region.** A move can name a second clip for the upper body or the arms
   (walk + carry lantern, run + wave, idle + talk on the phone). Godot's AnimationTree blend
   filters do the masking. This multiplies the set we have more than any download would.
5. **Gait styles.** Procedural changes to the walk and run (stride, cadence, bounce, sway,
   arms, lean) with presets, kept per character. Follow-through (head and arms lag, loose or
   stiff) belongs in `sway.gd`, not a second system.
6. **Feet on the ground.** IK that plants the feet on slopes and steps, and lowers the
   hips. It is the most visible gain in our terrain-heavy worlds.
7. **The pose editor** (your open keyframe decision). Click a joint, turn it, key it, with
   mirror, paste flipped and a body picker chart. It would make the hand poses and new
   moves without leaving the engine.
8. **More import maps**: Unreal 4/5 and Reallusion beside Mixamo, and the choice of root
   travels or hips carry it.

**C. Large; needs your approval first:**

9. Rigging an unrigged model inside the engine.
10. Eight directions made from one walk (Motion Lab's "the eight").
11. A shot camera per animation, for cutscenes.

My recommendation: A1, A2, A3 and B4 as one pass. They are the grip you asked for, the
character in how a body stands, and twice the moves from the clips we have. They need nothing
new downloaded beyond what is now in `actionforge/`.
