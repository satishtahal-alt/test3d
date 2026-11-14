# E. Phase 2 Investigation Report: player character, moveset and controls

Draw3D lifecycle validation programme. Investigated 2026-11-14. Covers lifecycle phase L3.

Nothing in the engine was changed. The workflow was traced end to end in the code, then run through the UI, then checked in play, after saving, and after reopening.

## 1. The short answer

- **Choosing who the world plays works.** One list on Project › Overview sets it, it holds in play, after saving, after reopening, and in a copy of the world.
- **Assigning a moveset does not work for five characters.** For any character with a profile "in use" (the ranger, the castaway and the three scavengers), the moveset chosen on the player's page is ignored in play. The profile's set is used instead. The page shows both names at once and does not say which wins.
- **A world you never gave a player to already has one.** It is the Skyfarer, because the shared library marks that character as the default. This reaches old worlds too: a world saved with no player character now plays the Skyfarer.
- **The causes of "the wrong character" and "unexpected movesets" are found.** Six different things can each decide who is played, and three can each decide the moveset. Only one of each is set on the page a maker would use. Section 6 lists every cause with its evidence.
- **Buttons were read and checked in play, not changed.** Changing a binding through the UI is not yet tested.

## 2. How this was investigated

| Method | What | Evidence |
|---|---|---|
| Code reading | Every place that sets or reads the played character, the set of moves and the buttons. | File names below. |
| UI test | `editor\run.ps1 -Author lifecycle`, steps P2.1 to P2.5: real clicks and keys on a sandbox copy. 0 steps stuck. | `evidence/2026-11-14/ui-test-log.txt`, pictures `06` to `15`, `20`, `21`, `25` to `31`. |
| Runtime test | In play: who is worn, which set the body moves by, distance walked on a held key, height on Space, the move begun on a left click. | The same log. |
| Older worlds | Copies of four worlds saved by earlier versions were opened and played: Emberwatch, My world, Driftwood island, Showcase physics. | Step P2.5. |
| Not done | No hand test. No controller. No binding was changed. The title card, the class picker and "Come back where I was" are off in scripted runs, so they were read in code only. | Marked below. |

## 3. What decides who is played

Traced in `player.gd` `wear_walker`, `charbook.gd` `walker`, `builds.gd` `_apply`, `main.gd` `_dress`. Listed strongest first. The first one that applies wins.

| # | Decider | Where it is kept | Where a maker sets it | Scope |
|---|---|---|---|---|
| 1 | **A class in force.** The class chosen as the game begins, if this world's Player start offers classes. Its "plays as" character wins. | The class: the world's text. The choice: the run (a saved game, or "Come back where I was"). | Project › Gameplay › Builds and classes; chosen on the title card in play. | This world, this run |
| 2 | **The world's own choice.** | The world's file (`assigned`). | Project › Overview › **The game plays as**. Also Character Studio › Play as this; a character's page › Play as … in this world. | This world |
| 3 | **The first character marked as the one played** in the world's copy of the characters. | The shared library file, copied into each world. | **No control found.** Today the library marks the Skyfarer. | Every world with no choice of its own |
| 4 | **A kind of this world to walk as** ("Play as this" on a person put down). It decides the body only where no character is marked at 3. | The world's file. | Right-click a person › Play as this. | This world |
| 5 | **The character marked as the fallback.** | The shared library file. | No control found. | Every world |
| 6 | **Someone the engine makes.** | Code. | – | – |

Two more things change who you see without being a choice:

- **"Come back where I was."** When play stops, the run's facts (the class, what is carried, health) and the place are kept on this computer and put back at the next play of the same world file. Read in code, not tested: for a world with no file yet, the key is empty, so any unsaved world would be given the last unsaved world's kept run.
- **The body left over from the last world.** Confirmed in the test: after another world is opened or a new one begun, the body and its set stay those of the previous world until Play is pressed. After opening Driftwood island the world's player was the castaway while the body still worn was the ranger.

## 4. What decides the moveset

| # | Decider | Where it is kept | Where a maker sets it | Scope |
|---|---|---|---|---|
| 1 | **A class in force**: its "moves". | The world's text. | Project › Gameplay › Builds and classes. | This world, this run |
| 2 | **A profile in use** for that character: its set. It also replaces what the character carries, adds abilities, and lays out the controller. | `editor/godot/profiles.txt`. Never in a world. | Window › Profile & buttons; the Animator's profile row. | **Every world of every game**, by the character's name |
| 3 | **The character's own set.** | The library, or the world's own version of the character. | Project › The player › **Moves** › Its moves. | Every world, or this world if made unique |

Profiles in use today: `ranger_testing` (the ranger), `castaway_survivor` (the castaway), and one each for the three scavengers.

## 5. What decides the buttons

`controls.gd` `device_ways`. Strongest first, for each game button on each device.

| # | Layer | Where it is kept | Where it is set |
|---|---|---|---|
| 1 | The game says this button is not used | The world's `inputs` text | Project › Controls & Input |
| 2 | The player's own remapping, in a built game | The player's folder | The game's Settings screen |
| 3 | **The profile in use of the character played.** It lays out the whole controller: a game button it does not name is on no controller button. | `profiles.txt` | Window › Profile & buttons |
| 4 | The project's layout in use (Classic, Parkour, Combat) | The world's `inputs` text | Project › Controls & Input › Game buttons & layouts |
| 5 | Your own buttons on this computer | `%APPDATA%\Draw3D\controls.txt` | Character Studio › Buttons |
| 6 | Draw3D's table | Code | – |

In the editor, 4 comes before 5. In a built game, 5 comes before 4. On top of all of these, presses (taps, holds, chords) take a button for themselves while they could happen.

There is also an older, separate kind of "profile": button sets saved in `%APPDATA%\Draw3D\profiles\` (two are there: `First_Profile`, `swordsman`), used from the Studio's Buttons tab. Two different things are called a profile.

Four places edit buttons: Project › Controls & Input, Project › The player › Buttons, Window › Profile & buttons, and the Animator's move box.

## 6. The trace: why the wrong character or moveset appears

Each cause, how sure it is, and the evidence.

### C1. A world with no choice of its own plays the Skyfarer. **Confirmed.**

- A brand-new blank plot: nothing chosen, and the Overview already said "PLAYABLE CHARACTER: Skyfarer", with a green "Assigned" (P1.2, P2.1, picture `06`).
- Showcase physics: its file names no played character. Opened today, it plays the Skyfarer (P2.5).
- Why: the shared library marks the Skyfarer as the one played (it was made the flying default on 2026-11-04). When any world opens, what the library has that the world's copy lacks is added to it, mark included.
- 39 of the 40 world files have no choice of their own, so each plays whichever character the library, or its own old copy, marks.

### C2. A profile in use overrides the moveset the page shows. **Confirmed.**

- On the benchmark world the ranger was chosen as the player. Project › The player › Moves showed "Its moves: **Brawler**" in the list, and under it "Its moves — the set **Ranger testing**" (picture `09`).
- "Swordsman" was chosen in that list. The message said the moves were set. The right-hand Details still said "Moves by: Ranger testing" (picture `10`).
- In play the body moved by `ranger_testing`: a left click began `warrior_cut_1`, a move of that set. The set chosen on the page was never used. The same after saving and reopening.
- Emberwatch: its file has its own ranger with the set `walker`. Opened today and played, it moves by `ranger_testing` (P2.5). A world made before the profile existed now has a different moveset.
- The profile also replaced what the ranger carries: a katana and a crossbow in play (picture `14`).

### C3. The moveset change went into the shared library, for every world. **Confirmed.**

- The same choice of "Swordsman" was written into the library's file as well as the world, with no question, and the world was not marked unsaved.
- The question "Update shared asset / Create separate version / Cancel" is asked only when another world on this computer uses the character. In the sandbox none did. On your real files many worlds use the ranger, so the question would appear; that case was **not tested**.
- The message said "kept with this world (Ctrl+S) and in the shared file every world starts from", which describes the old model, not the library.

### C4. Opening an old world freezes dozens of its characters and moves. **Confirmed.**

- Emberwatch 56, My world 58, Driftwood island 50, Showcase physics 45 entries became "this world's own versions" on opening, with no message.
- From then on those entries never follow a fix made in the library, while the entries beside them do. One world can hold a mix of old and current versions of related moves.

### C5. Two characters marked as played: the order in the file decides. **Confirmed that it happens; which wins is by order.**

- After opening, Emberwatch and My world each had three marked characters: ranger, scout (fallback) and skyfarer. The ranger won only because it comes first.
- `testworld.json` has ranger and skyfarer both marked on disk.
- A world whose own ranger happens to match an older shipped version would follow the library instead, lose its mark, and play the Skyfarer. Likely; not reproduced on a specific world.

### C6. The previous world's body is still worn while editing. **Confirmed (state), not pictured.**

- New empty world after the benchmark: player should be the Skyfarer; body worn was still the ranger.
- Driftwood island: should be the castaway; body worn was the ranger.
- Showcase physics: should be the Skyfarer; body worn was the castaway.
- Play corrects it. Until then the figure standing in the world while you edit is likely the wrong one.

### C7. A class chosen on the title overrides the world's choice. **Read in code; not tested.**

- By design a class in force wins over "The game plays as". The Overview says so in words when one is in force.
- This is what happened in Hold the Light on 2026-11-13 (the page said the scout, the game began as the ranger). A guard was added that day: a class counts only if the world's start offers it.
- The class picker is off in scripted runs, so the UI test cannot reach it.

### C8. "Come back where I was" restores the last run's class and facts. **Read in code; not tested.**

Off in scripted runs. For worlds with no file the key is empty (section 3).

### C9. Two copies of one game. **Confirmed (files).**

Two different `hold_the_light.json` files, in `editor/` and `editor/godot/demo/`. Which one is opened decides what you see.

### C10. The exported game has no profiles. **Read in code; not tested.**

`profiles.txt` is not copied by the export. An exported game would play the character's own set and the layout's buttons, not what was tested in the editor.

## 7. Workflow records

### L3.1 Choose who the world plays

| Part | |
|---|---|
| 2. Outcome | This world plays the character chosen, and no other world is changed. |
| 3. Start | A world open. |
| 4. UI steps | World › Project: what the game is made of… › Overview › **The game plays as** › type the name in the list's search box › Enter. |
| 5. Expected | The row above says the character. Play begins as them. It holds after save and reopen. |
| 6. What exists | `projectview.gd` (overview), `main.gd` `assign_played`, `assignedbook.gd`. The same is done by Character Studio › Play as this, and a character's page › Play as … in this world. |
| 7. Test | UI test P2.1, P2.4, P1.6, P1.7. Pictures `06`, `07`, `14`. |
| 8. Function | Chose Ranger from 53. The row changed. One unsaved change. In play the ranger was worn. After save, close and reopen: the ranger. In the copy: the ranger. The file holds the choice. |
| 9. Usability | The list is long and opens as a search box, which works. Before anything is chosen the row says "Assigned" in green (C1). Three places set the same thing. Two tooltips still say it is "written in characters.txt as `walks`", which is no longer true and shows a file name and a code word. |
| 10. Missing | No "nobody chosen yet" state. No way to see or change which character new worlds start with. |
| 11. Acceptance (proposed) | (a) A new world says plainly that it plays the engine's starting character until one is chosen. (b) One choice sets it. (c) It holds in play, after save, after reopen, in a copy. (d) No other world changes. (e) The page names anything that overrides the choice. |
| 12. Hand test | Document F, test 7. |
| 13. Regression | `author_lifecycle` P2.1, P2.4. Rerun on changes to `charbook.gd`, `player.gd`, `builds.gd`, `assets.gd`, `projectview.gd`. |

**Status: Claude tested.** Works; open findings F-L3-01, F-L3-06.

### L3.2 Give the player a moveset

| Part | |
|---|---|
| 2. Outcome | The player moves by the set chosen. |
| 3. Start | L3.1 done. |
| 4. UI steps | Project › Overview › **Set up the player →** › tab **Moves** › **Its moves** › type the set's name › Enter. |
| 5. Expected | The list and the heading name the same set. In play, the set's moves happen. |
| 6. What exists | `projectview.gd` `_moves_tab`. It writes the character's `moves` line. `charbook.gd` `_with_profile` then replaces it with the profile's set when one is in use. `builds.gd` replaces it again with a class's. |
| 7. Test | UI test P2.2, P2.4. Pictures `09`, `10`, `14`, `15`. |
| 8. Function | **Fails for a character with a profile in use.** The list showed Swordsman, the heading and Details showed Ranger testing, and play used Ranger testing. The change was also written to the shared library (C3). Not tested for a character with no profile. |
| 9. Usability | One page shows two different answers to "which moves". Nothing says a profile is deciding. The page that does say so is Project › Controls & Input › The character played. |
| 10. Missing | No statement of what decides the set. No control here to take the profile off or change its set. |
| 11. Acceptance (proposed) | (a) The set shown is the set played. (b) Where a profile or a class decides, the page says so and leads there. (c) Changing it for this game does not change another game without asking. (d) It holds after save and reopen. |
| 12. Hand test | Document F, test 8. |
| 13. Regression | `author_lifecycle` P2.2, P2.4. |

**Status: Claude tested (fails).** Blocker F-L3-02.

### L3.3 Lay out the buttons

| Part | |
|---|---|
| 2. Outcome | Each key and controller button does what the maker chose. |
| 4. UI steps | Project › **Controls & Input** › The character played / Game buttons & layouts / Presses / Wheels / Test. Also Project › The player › Buttons, and Window › Profile & buttons. |
| 6. What exists | Section 5. `controlspage.gd`, `controls.gd`, `profileview.gd`, `presses.gd`. |
| 7. Test | UI test P2.3: the pages were opened and read. **No binding was changed.** Pictures `11` to `13`. In play (P2.4): W walked 2.1 m in about a second; Space rose 1.06 m; a left click began an attack. |
| 8. Function | The first tab said "Ranger plays with the profile 'Ranger testing' in every world" and "✓ Checked: each press does one thing". The layout in use was "Classic — the game's own". The keys worked in play. |
| 9. Usability | Four places edit buttons. With a profile in use the game's Attack button has no key or controller button of its own; the profile's own presses take its place, which the card shown in play does list. |
| 10. Missing | Not yet known: changing a binding was not tested. |
| 11. Acceptance (proposed) | (a) A button changed in one place is changed in play. (b) It holds after save and reopen. (c) The page says which layer it was written to. (d) The same result on a controller. |
| 12. Hand test | Document F, test 9. |
| 13. Regression | `author_lifecycle` P2.3. The older demo `inputs`. |

**Status: Discovered**, with the default keys checked in play.

### L3.4 Resolve the model, the rig and the animations

Read in code only. A character names a model file, a bone map, clip libraries and a look that says which clip shows each move. All are library entries; the model and clip files are in `editor/godot/characters/`. In the test the ranger and the mannequin were drawn and animated in play (picture `14`), which shows the shipped characters resolve. Importing a model or animations of your own was not tested. **Status: Discovered.**

### L3.5 Play and check

| Part | |
|---|---|
| 4. UI steps | **Play (Tab)** on the bar. Controls (H) shows the card. Tab stops. |
| 7. Test | UI test P2.4, twice. Pictures `14`, `15`, `20`, `21`. |
| 8. Function | Play began. No card of the game was over it. The controls card listed the keys and the profile's presses. Walking, the jump and the attack happened. Tab stopped play. |
| 9. Usability | In play at 1280×720 the controls card covers two lines of status text at the top left (picture `14`). With no Player start, play begins where the last play stopped (document D, F-L2-01). |
| 11. Acceptance (proposed) | (a) Play begins as the chosen character with the chosen moves. (b) Each assigned action happens on its button. (c) Stopping returns to editing with nothing changed in the world. |

**Status: Claude tested**, keyboard and mouse only.

### L3.6 Save, close, reopen: the setup holds

| Part | |
|---|---|
| 7. Test | UI test P1.4, P1.6, P2.4 (after reopening), P1.7. |
| 8. Function | After save, a new world, and reopening: the same character, the same set in play, the same distances and moves. The copy the same. |
| Note | "The same set" was the profile's set both times. The set chosen on the page was saved in the file and ignored both times. |

**Status: Claude tested.** Holds, including the fault of L3.2.

## 8. Findings

| Id | Severity | Finding | Cause |
|---|---|---|---|
| F-L3-01 | Major | A world with no choice of its own plays the library's marked character, today the Skyfarer, and the Overview calls that "Assigned". Old worlds with no player now play the Skyfarer. | Confirmed (C1) |
| F-L3-02 | **Blocker** | For a character with a profile in use, the moveset chosen on Project › The player › Moves is not the one played. The page shows both names and does not say which wins. | Confirmed (C2) |
| F-L3-03 | Major | Profiles are engine-wide and apply by character name to every world of every game, including worlds made before the profile existed. | Confirmed (C2, Emberwatch) |
| F-L3-04 | Major | Changing the player's moveset writes the shared library for every world, with wording that describes something else. With other worlds present a question is asked; that path is untested. | Confirmed (C3) |
| F-L3-05 | Major | The previous world's character stays worn while editing after another world is opened or begun. | Confirmed (C6) |
| F-L3-06 | Minor | Two tooltips say the choice is "written in characters.txt as `walks`": wrong since 2026-11-08, and a file name and code word in the UI. | Confirmed (code) |
| F-L3-07 | Major | Two characters can be marked as played in one world; the order in the file decides. | Confirmed (C5) |
| F-L3-08 | Major | Six things can decide who is played and three the moveset. The Overview names only the class override. | Confirmed (code, test) |
| F-L3-09 | Question | Which character should a new game begin with, and should the maker be asked? | – |
| F-L3-10 | Major | "Come back where I was" restores the last run's class and facts; unsaved worlds would share one kept run. | Likely (code) |
| F-L3-11 | Minor | Two different things are called a profile: character profiles in the engine's folder, and button sets in `%APPDATA%`. | Confirmed (files) |
| F-L3-12 | Minor | Four places edit buttons. | Confirmed (code) |
| F-L3-13 | Minor | In play the controls card covers the status lines at the top left. | Confirmed (picture 14) |
| F-L3-14 | Major | The class chosen on the title overrides the world's choice; the scripted test cannot reach the title, so this path has no UI test. | Read in code |

See also document D: F-L1-07 (export carries no profiles), F-L1-08 (two copies of one game), F-L2-05 (old worlds frozen on opening).

## 9. Directions for the fix (for review, not built)

The desired outcome is: create a world, assign a character, assign a moveset, configure controls, with no workaround. What stands in the way is not a missing button. It is that the answer is spread over several stores with different scopes. Three questions settle most of it.

1. **Should a profile belong to a game?** Today it is engine-wide. If it becomes the game's, C2, C3 and C10 are answered together, and it joins the project-folder decision in document D.
2. **Should the player's page be the one place that decides?** One page, "The player of this game", stating the character, the set, what is carried and the buttons as they will be in play, and naming whatever overrides each (a class, a profile) with a link to it.
3. **Should a new world start with no player, or a named starting one?** Either "nobody chosen yet", or "the engine's starting character", said as such and changeable in the editor.

Smaller, independent of those:

- Put the right body on when a world opens (C6).
- Say on opening an old world that N entries were kept as its own, with the list (C4).
- Refuse or flag two marked characters (C5).
- Correct the two tooltips and the moveset message.

## 10. What is recommended next

1. Review sections 6 and 8.
2. Do the hand tests for L3 (document F, tests 7 to 10). Tests 8 and 10 are the two I most want your eyes on: the moveset that does not take, and an old world's player.
3. Answer the three questions in section 9, together with the four in document D.
4. Then authorise the fixes, smallest first. Each is retested by the same run, and the run becomes the first entry of the regression set.
