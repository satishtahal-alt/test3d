# Character Studio — research record (2026-10-15)

What was looked at before the Character Studio's first tier (amendment 7at) was built:
Godot's own systems, free and open assets, and what Draw3D already had. What was used,
what was not and why, and what each candidate would cost to bring in. Licences were
read from the publishers' pages on 2026-10-15. One file was then downloaded with the
user's yes (below: "Brought in").

## What Draw3D already had (and kept)

| Already there | Where | How the Studio uses it |
|---|---|---|
| A motor that alone moves a body, by intent, and says what it does in words (`doing`, `phase`, `progress`, `side`) | `motor.gd`, `move.gd` | The clips are chosen by those words and timed by them; the motor is still the only authority on position |
| Moves written as text, sets assigned by name | `moves.txt`, `moveset.gd` | The samurai is a `set` and four moves; two new words: `strikes PLACE R`, `upon struck` |
| Poses written as text for a person's 17 bones | `poses.txt`, `poses.gd`, `figure.gd` | Baked into a real clip library (`basic.glb`), and the language used to author the samurai's clips |
| A figure API others rely on (places on the body, holding at a place, a hand reaching) | `figure.gd` | `animated.gd` extends it, so acts, carrying and reaching work on a clip-shown character |
| Texts kept by the host with the world, undone, saved | `book.gd`, `written.gd`, host `text` | `characters.txt` is one more such text |
| The window system, F10 for texts | `desk.gd`, `win.gd`, `textview.gd` | The Studio is a window; "Open the text" goes to F10 at the entry |

## Godot 4.7.2 systems (verified present with a headless probe, 2026-10-15)

| System | Used? | Why |
|---|---|---|
| `GLTFDocument` / `GLTFState` (runtime glTF read and write) | **Used** | The editor runs as a game, so files must be read at runtime; also writes the baked assets (`charmake.gd`) |
| `AnimationTree` with `AnimationNodeBlendTree`, `Blend2`, `TimeSeek`, `Animation` nodes, manual process | **Used** as the mixer | Blends stand/walk/run by speed and cross-fades clips. Its state machine is **not** used: the motor already decides what happens, and a second decider would break the seam "the motor decides, the figure shows" |
| `SkeletonProfileHumanoid` | **Used** | Its 56 bone names are the shared vocabulary every map translates to; the Studio lists them with their groups |
| `AnimationLibrary`, `Animation` track interpolation | **Used** | Retargeted clips are ordinary Godot animations |
| `BoneAttachment3D` | **Used** | What a character holds (the katana) |
| `RetargetModifier3D` (live retarget, parent → child skeleton) | Evaluated, not used yet | Needs both skeletons named by one profile and a hidden source skeleton per character; baking once at load gives one skeleton, deterministic clips and exact checks. Worth revisiting if live retargeting of procedural motion is wanted |
| Import-time retargeting (`BoneMap` in the `.import` options) | Not usable here | Works only on files imported by the Godot editor; Draw3D's users load files in the running editor |
| `TwoBoneIK3D`, `FABRIK3D`, `CCDIK3D`, `ChainIK3D`, `SplineIK3D`, `LookAtModifier3D`, `SpringBoneSimulator3D` | Present, not used yet | Candidates for feet on slopes, a second hand on the hilt, looking at a target, hair and cloth. Draw3D's own two-bone reach (`figure.reach`) and `sway.gd` stay until a case replaces them |
| Root motion (`AnimationMixer.root_motion_track`) | Deliberately not used | The motor moves the body. Root motion could later *measure* a clip (how far a roll goes) to fill in a move's numbers, without moving anything |

## Compatibility facts found by testing (not in the documentation we read)

1. **Godot's glTF reader turns `.` into `_` in names**, of bones and of clips. A person's
   `upper_arm.L` is `upper_arm_L` once read from a file; a clip `slash.cut` becomes
   `slash_cut`. Maps now find a bone under either name (`clips.gd` `bone`), and Draw3D
   names clips with `_` (a phase `slash/cut` → clip `slash_cut`).
2. **A second `generate_scene` from one `GLTFState` renames every node** (`Hips_2`). A
   character is made by duplicating the first scene made of a file.
3. **Tracks that hold a bone at its rest are dropped** on import (a pose that leaves the
   hands alone has no hand tracks). Missing tracks are read as the rest.
4. **Clips are resampled to 30 fps** on import. Fine for the motor-timed playback.
5. A skinned mesh, its skin and its animations written by Godot's glTF writer read back
   whole (checked: 22 bones, 61 clips, all pointing their limbs within 3.4° of the source
   after retargeting, the worst in the spine where the bone counts differ).

## Retargeting requirements (as built)

- Both sets of bones **stand alike** when not posed (Draw3D's people and the mannequin
  stand arms down). A set made in a T says how its bones turn to stand so:
  `stance LeftUpperArm 0 0 -75` (degrees) in its map.
- Both **face the same way** (Draw3D: −z). A set facing +z says `faces +z`.
- Bones are matched **by Godot's humanoid names**; a bone in only one map is not moved by
  the other's clips; unnamed bones keep their rest.
- The hips' **height** is scaled by how much taller one is; **lengths are not carried**
  (a longer arm reaches farther). A walk's **stride** is scaled by height, so the clip
  plays faster on shorter legs and feet are let down onto the ground each frame.

## Free and open candidates

| Candidate | Source | Licence | Format | Godot 4.7 | What it would give | Limits | Cost to bring in |
|---|---|---|---|---|---|---|---|
| **Quaternius Universal Animation Library** | quaternius.com/packs/universalanimationlibrary.html; quaternius.itch.io/universal-animation-library; opengameart.org/content/universal-animation-library | CC0 (commercial use, redistribution allowed) | glTF, FBX, OBJ (free tier); .blend and Godot exports are paid extras | glTF reads at runtime | Locomotion in 8 directions, jog, sprint, push, crawl, swim, sit, deaths; a humanoid rig made for retargeting (also Mixamo-compatible). Free "Standard": 45 clips; full: 120+ | Its v2.0 renamed the rig's bones (map must match the version); T-pose or not to be checked; naming to be read from the file | **Small**: one `library` entry, one `map` (and `stance`/`faces` if needed), look lines. About an hour once downloaded |
| **Quaternius Universal Animation Library 2** | opengameart.org/content/universal-animation-library-2 | CC0 | as above | as above | 130+ more: parkour, combat combos, farming, zombie locomotion: the very sets the brief names next | Same rig family; check the version | **Small** after the first |
| Kenney Animated Characters 1–3 | opengameart.org (Kenney), kenney.nl | CC0 (credit asked, not required) | FBX (glTF in some packs) | FBX needs conversion; glTF reads | Four skins on one rig; a second, differently built body to prove retargeting | Three clips only (idle, jump, run) | Small; useful as a test model, not as a library |
| GDQuest Mannequiny | github.com/gdquest-demos/godot-3d-mannequin | Project MIT; character CC-BY 4.0 (credit: "Mannequiny CC-By 4.0 GDQuest, Luciano Muñoz, and contributors"); a commenter disputes it | glb | Godot 3 project; the glb itself reads | A rigged mannequin with a few clips | Attribution required; licence to be confirmed | Small; attribution in the library's `licence` line |
| GDQuest Godot 4 3D Characters (Sophia, Gobot...) | opengameart / GDQuest repo | Listed CC-BY 4.0; an unverified July 2026 report says changed to CC-BY-NC-SA 4.0 | glb | yes | Polished characters | Possibly non-commercial: **not to be used** until the repo's licence is read | — |
| Mixamo | mixamo.com | Free to use in projects; raw files may not be redistributed on their own; Adobe sign-in | FBX | read at runtime by Godot 4.7's FBXDocument (amendment 7ay) | The largest humanoid library | Cannot ship in Draw3D's repository; the user downloads for their own games | Built (7ay): `map mixamo`, a folder of FBX files as one library, the Studio's Clips > Import clips…; checked on script-made Mixamo-like files only |

## Decisions taken in this slice

- **Prove the pipeline with Draw3D's own assets first**, made by script into glTF files
  (`charmake.gd`): a clip library from the existing poses, a samurai library from new
  pose text, a mannequin on a different skeleton, a katana. They go through exactly the
  path a downloaded file would (glTF read at runtime, mapped, retargeted, blended). No
  external file was downloaded: that needs the user's yes, and is proposed below.
- **The motor stays the authority**; clips never move the body; clip time follows the
  move's progress or the distance walked.
- **One new text** (`characters.txt`) beside moves and poses, kept by the host with the
  world; no host code changed.

## Brought in (2026-10-15, with the user's yes)

| | |
|---|---|
| File | `universal_animation_librarystandard.zip`, 14,541,205 bytes (size read from the server before downloading; the download matched) |
| Source | https://opengameart.org/sites/default/files/universal_animation_librarystandard.zip (Quaternius's upload, page opengameart.org/content/universal-animation-library) |
| Licence | CC0 1.0 Universal, public domain dedication (its `License.txt`, kept beside the file) |
| Kept | `editor/godot/characters/quaternius/AnimationLibrary_Godot_Standard.glb` (6,671,104 bytes) and `License.txt`; the Unity and Unreal FBX, pictures and the zip were removed |
| Contents | 46 clips (45 and a T-pose), a 53-bone Rigify rig (`root`, `DEF-hips`, `DEF-spine.001-.003`, fingers, toes), Quaternius's own skinned model |
| As read by Godot 4.7.2 | Bone names keep their dots here (unlike files Godot itself wrote); made in a T; faces +z, its left at +x; root bone turned -90 degrees about x (Blender's up); `_RM` clips move the root (root motion) |
| Map | `map ual` in characters.txt: `faces +z`, `stance LeftUpperArm 0 0 -86`, `stance RightUpperArm 0 0 86`, 22 bones named |
| Checked | Its clips carried to the mannequin and to a person: every limb within 4.7 degrees of the source's direction; arms hanging when idle; its own model worn; walking, the jump and the roll shown; the cut's quickest hand in the move's cut phase |
| Cost | About two hours: one library entry, one map, one look, and three small engine changes the file showed were needed (`part` of a clip; stance in the body's frame; a +z model turned round) |

UAL 2 (parkour, combat combos, farming; CC0) is the same family and would need only its own
library line and look rows, after its bone names are checked against `map ual` (its
v2.0 renamed the rig).

## Combat sources, and the RPG pack (2026-10-15, authoring pass, 7au)

Looked at for sword, katana and martial movement that is natural and whole (a step, the
weight, the follow-through, the recovery). Prefer a few good clips adapted (part, mirror,
bend, phase timing, spins) over many poor ones.

| Source | What | Licence, as read | Rig, fit | Cost | Verdict |
|---|---|---|---|---|---|
| Quaternius RPG Characters (Nov 2020), the user's Drive link | 6 characters (Cleric, Monk, Ranger, Rogue, Warrior, Wizard), .gltf 2.4-3.1 MB each, FBX, Blend, OBJ, "Humanoid Rig Versions" | CC0 1.0 (its License.txt, read in Drive's viewer); the folder is the one quaternius.com/packs/rpgcharacters.html links | Quaternius's rig; a humanoid version also given | about an hour once on disk | **Blocked**: Google Drive "Quota exceeded" on every file, 2026-10-15. The user's own Download all is needed |
| Quaternius UAL 2, Standard (free) | about 70% of 130+ clips: melee and armed combos, split into hits with their recoveries, parkour, farming, fishing | CC0 (OpenGameArt, itch.io) | Same family as UAL 1 (`map ual`); its v2 bone names to be checked | a library line and look rows; a download needs the user's yes (universal_animation_library_2standard.zip, 10.3 MB on OpenGameArt) | **Recommended next**: combos already split into phases match how moves are made |
| Mixamo (Adobe) | many sword, katana and great-sword packs, motion capture | free; Adobe's FAQ (community post, 2022): no fee, files may not be given away loose, may ship built into a game | mixamorig bones: one map | the user's Adobe account and download, per clip; a map | Good quality; usable for Draw3D's games (not redistributed loose; Draw3D is not distributed). Terms to be read live before shipping |
| MoCap Online free sample packs | a one-handed sword pack (attacks, blocks, movement) | terms not read | its own rig | unknown | Look at the terms first |
| CMU motion capture, converted libraries | general movement, some martial | free to use, not to sell as animations | various | a map | No weapons |
| Mesh2Motion | an open alternative to Mixamo's rigging, with Quaternius's clips | assets CC0 (said); code licence unread | | | Nothing new for combat |

What is still missing for a samurai of quality, and the best means: a cut that steps
in with the right foot and turns through (UAL 2's armed combos or Mixamo, then `part`,
`mirror` and `bend`); the left hand kept on the hilt and feet planted through a turn (IK:
Godot's TwoBoneIK3D, not built); a guard held while running (layering, not built).

## Input mapping in other engines (2026-10-16, for 7bd)

Looked at to answer the user's hand test of 7ba: how other engines let a person put actions on buttons, name them, and keep profiles.

- **Unreal Engine, Enhanced Input.** An *Input Action* is a named thing the game asks for. *Input Mapping Contexts* put keys on actions, and can be swapped by situation (on foot, in a vehicle). *Triggers* say when an action fires: pressed, held, tapped, *Chorded Action* (only with another action held), *Combo*. *Player Mappable Key Settings* give each mapping a fixed name for saving and a **display name** shown to the player.
  - Draw3D's counterparts are a move's `asks` (the action); its `when` (triggers, `holding` for a chord); and now `label` (the display name).
  - Not taken: contexts as objects. The character's set already decides which moves a button can ask.
- **Unity, Input System.** *Action Maps* hold *Actions*, each with *Bindings*. The rebinding UI uses `PerformInteractiveRebinding()`, usually with `WithControlsExcluding("Mouse")` so that a stray mouse movement or click is not taken, and `WithCancelingThrough("<Keyboard>/escape")`.
  - Taken: the mouse is left out of capture unless the person clicks the box shown for it, and Esc cancels (as before).
- **Godot, InputMap.** Project Settings lists actions with a "listen for input" dialog. Actions are renamed in the list. There is no display name apart from the action's name.
- **What all three share:** actions on the left and the selected action's bindings and conditions on the right (the Buttons tab already works this way); a name shown apart from the name kept; capture that is hard to trigger by accident; and the set of actions shown fitting the situation, which here means the character's moves first.

Sources: [Unreal Engine, UInputAction](https://dev.epicgames.com/documentation/unreal-engine/API/Plugins/EnhancedInput/UInputAction); [Unreal Engine, PlayerMappableKeySettings](https://docs.unrealengine.com/5.2/en-US/PythonAPI/class/PlayerMappableKeySettings.html); [Unreal Engine, FEnhancedActionKeyMapping](https://dev.epicgames.com/documentation/unreal-engine/API/Plugins/EnhancedInput/FEnhancedActionKeyMapping); [Satisfactory modding docs, Enhanced Input](https://docs.ficsit.app/satisfactory-modding/latest/Development/Satisfactory/EnhancedInputSystem.html); [Unity Input System, rebinding extensions](https://docs.unity.cn/Packages/com.unity.inputsystem@1.8/api/UnityEngine.InputSystem.InputActionRebindingExtensions.html); [a rebinding guide](https://uhiyama-lab.com/en/notes/unity/unity-input-rebinding-guide/); [rebinding cancelled at once by mouse jitter](https://bugnet.io/blog/fix-unity-input-system-rebind-operation-cancelled-immediately).

## Universal Animation Library 2, brought in (2026-10-15, amendment 7av)

Downloaded with the user's yes from opengameart.org/content/universal-animation-library-2
(universal_animation_library_2standard.zip, 10,257,419 bytes; CC0 1.0 in its License.txt).
Kept: `Unreal-Godot/UAL2_Standard.glb` (8,061,600 bytes) and `License.txt`, in
`editor/godot/characters/quaternius2`. Not kept: the FBX, the .blend, the female
mannequin, the setup pictures.

What it is: 43 clips on 65 bones named as Unreal's mannequin names them (pelvis,
spine_01..03, upperarm_l...), not UAL 1's Rigify names: a map of its own (`ual2`), the
same stance (a T, facing +z). Armed: Sword_Regular_A, _B, _C and their recoveries, a
whole combo, Sword_Dash_RM, Sword_Block; Melee_Hook; Hit_Knockback; Shield; NinjaJump;
Slide; ClimbUp_1m_RM; farming; zombie. Where each cut is fastest (the right hand, by
`ual2look.gd`): A 0.17-0.31 s, B 0.19-0.33, C 0.58-0.72, the dash cut 0.20-0.37.
