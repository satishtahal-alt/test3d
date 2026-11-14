# F. Initial Testing Instructions: Phase 1 and Phase 2 by hand

Draw3D lifecycle validation programme. Written 2026-11-14.

Ten short tests you do yourself in the editor. They cover the same ground as Claude's UI test, plus the parts a script cannot reach: the real start-up, the Recent rows, a controller, the title, and "Come back where I was".

Every step uses a button or menu that exists today. Nothing asks you to open a text file.

## Before you start

1. **Close any world you care about**, saved.
2. **Make a folder for the benchmark** outside Draw3D's folders, for example `Documents\Draw3D games\Benchmark`. Tests 3 and 5 save into it.
3. **Start Draw3D by hand:** `run.bat`, or `editor\run.ps1` with nothing after it.
4. Keep this page open beside it. For each test, write what you saw under "Note".
5. Two tests touch things shared by all your worlds. They are marked **⚠** and say exactly how to leave everything as it was.

**How to answer.** For each numbered check, write **Yes**, **No**, or **Unclear**. "Unclear" is a result: it means the editor did not make it plain.

---

## Test 1. Starting up (L1.1)

1. Start Draw3D by hand.

| Check | Yes / No / Unclear |
|---|---|
| 1a. A window headed **Begin** is shown, with nothing but sea behind it. | |
| 1b. It offers New world…, Open a saved world…, Project…, Characters…, Animate and moves…. | |
| 1c. Under WORKED ON LATELY you recognise each world, and can tell which copy each one is. | |
| 1d. The bar's World heading says the world is new and has no file. | |
| 1e. You could tell from this window alone how to begin a game. | |

**Look closely at:** the times shown beside each recent world. Claude's test saw them five hours off. Are yours right?

Note:

---

## Test 2. A new blank world (L2.1)

1. In Begin press **New world…**.
2. Leave the list on **A plot of land (blank)**.
3. Press **Make the new world…**, then **Yes: replace the world in the editor**.

| Check | Yes / No / Unclear |
|---|---|
| 2a. A small plot of land appears, with the Terrain window open. | |
| 2b. The message tells you what to do next. | |
| 2c. Anything on screen tells you this world has no place for a game to start. | |
| 2d. Anything on screen tells you who the player will be. | |

4. Open the World menu and read its last lines.

| Check | |
|---|---|
| 2e. It says there is 1 unsaved change though you have done nothing. Does that seem right to you? | |

Note:

---

## Test 3. Saving (L2.5)

1. **World › Save as…**

| Check | Yes / No / Unclear |
|---|---|
| 3a. Which folder does the dialog open in? Write it under Note. | |
| 3b. Is that where you would want a new game to go? | |

2. Go to your Benchmark folder, name the file `benchmark`, and save.
3. Change anything small (drag a little land out with Extend land), then **World › Save** (Ctrl+S).
4. Open the World menu again.

| Check | |
|---|---|
| 3c. The title bar and the World heading show `benchmark`. | |
| 3d. **Restore a backup… (1)** is offered. | |
| 3e. **World › Show in folder** opens your Benchmark folder, with a `backups` folder in it. | |

Note:

---

## Test 4. Close and reopen (L2.6)

1. **World › New world…**, choose **Nothing at all**, make it.
2. **World › Recent: Benchmark**.

| Check | Yes / No / Unclear |
|---|---|
| 4a. You were asked about unsaved changes in the empty world you never touched. | |
| 4b. Benchmark opened as you left it. | |
| 4c. You can see your plot straight away, without flying the view to it. | |

3. Close Draw3D altogether and start it again. In Begin, press **Benchmark** under WORKED ON LATELY.

| Check | |
|---|---|
| 4d. It opened, the same again. | |

Note:

---

## Test 5. A game, and a second game from it (L1.2, L1.3, L2.8)

1. **World › Begin: new, open, worked on lately…**
2. Press **＋ Keep Benchmark here as a game**.
3. Under GAMES, press **New game from it…**.
4. Save it in the same Benchmark folder as `benchmark_two`.

| Check | Yes / No / Unclear |
|---|---|
| 5a. Benchmark is listed under GAMES. | |
| 5b. The save dialog opened in a sensible folder. | |
| 5c. The title now shows `benchmark_two`. | |
| 5d. Is `benchmark_two` a second world of the same game, or a second game? Does the editor tell you? | |

Note:

---

## Test 6. The game's worlds (L1.4)

With `benchmark_two` open:

1. **World › Project: what the game is made of…**
2. Press **Worlds** on the left.

| Check | Yes / No / Unclear |
|---|---|
| 6a. Both `benchmark` and `benchmark_two` are listed. (Claude's test: only the open one was.) | |
| 6b. The other worlds listed are ones you think of as part of this game. | |

Note:

---

## Test 7. Who the world plays (L3.1, L3.5)

1. Open `benchmark` (World › Recent).
2. **World › Project…**, the **Overview**.

| Check | Yes / No / Unclear |
|---|---|
| 7a. Who does PLAYABLE CHARACTER name, before you choose anyone? Write it. | |
| 7b. Did you expect a new world to have that player? | |

3. Beside **The game plays as**, open the list, type `swordsman`, press Enter.
4. Close the Project window. Press **Play (Tab)**.

| Check | |
|---|---|
| 7c. You are the swordsman. | |
| 7d. W A S D move, Space jumps, a left click strikes. | |
| 7e. The same with a controller: stick, A, X. | |

5. Tab to stop. **Ctrl+S**.

Note:

---

## Test 8. The player's moveset (L3.2) ⚠

This is the test Claude's run failed. It uses the ranger, who has a testing profile.

1. In `benchmark`: **World › Project… › Overview › The game plays as**, type `ranger`, Enter.
2. Press **Set up the player →**, then the **Moves** tab.

| Check | Yes / No / Unclear |
|---|---|
| 8a. What does the list beside **Its moves** show? Write it. | |
| 8b. What set does the heading over the list of moves name? Write it. | |
| 8c. What does **Moves by** say in the Details on the right? Write it. | |
| 8d. Do the three agree? | |

3. Open the **Its moves** list, type `swordsman`, Enter.
4. **If a box headed "This is shared" appears, press Create separate version.** ⚠ This keeps the change to the benchmark world only. Do not press Update shared asset.

| Check | |
|---|---|
| 8e. Were you asked? | |
| 8f. After choosing, do the list, the heading and Moves by agree now? | |

5. Close the Project window. **Play (Tab)**. Press **H** for the controls card. Left click a few times.

| Check | |
|---|---|
| 8g. Do the moves you get belong to the swordsman's set, or to the ranger's testing set? | |
| 8h. Does anything on screen explain why? | |

6. Tab to stop. Then **World › Project… › Controls & Input**, the first tab.

| Check | |
|---|---|
| 8i. Does this page explain it? | |
| 8j. Would you have found this page on your own? | |

7. **To leave things as they were:** if you were *not* asked at step 4, press **Undo** (Ctrl+Z) until the Moves list shows what it showed at 8a.
8. **Ctrl+S**, close Draw3D, start it, open `benchmark`, Play.

| Check | |
|---|---|
| 8k. The same character and the same moves as at step 5. | |

Note:

---

## Test 9. Buttons (L3.3)

Claude's run did not change a button, so this is new ground. The names in step 2 are from the notes of 2026-10-24, not from pressing them today: if a name does not match what you see, write what you found instead. In `benchmark`:

1. **World › Project… › Controls & Input › Game buttons & layouts**.
2. Choose the **Jump** button. Add a second key to it with **Press a key…**, and press `J`.
3. Play. Press J.

| Check | Yes / No / Unclear |
|---|---|
| 9a. You could find where to do it without help. | |
| 9b. J jumps. Space still jumps. | |
| 9c. The page said where the change was kept: this game, or every game. | |

4. Stop. Ctrl+S. Close Draw3D, start it, open `benchmark`, Play, press J.

| Check | |
|---|---|
| 9d. J still jumps. | |

5. Open `benchmark_two`, Play, press J.

| Check | |
|---|---|
| 9e. Does J jump there too? Did you expect it to? | |

6. Now with the ranger as the player (test 8), look at the same page.

| Check | |
|---|---|
| 9f. Is it clear which buttons the profile decides and which this page decides? | |

Note:

---

## Test 10. An older world, and coming back (L3 trace) ⚠

**⚠ Do not save in this test.** Opening an older world changes it in memory. Saving would write that into its file.

**10a. An older world's player.**

1. **World › Open…** and open `editor\showcase-physics.json`.
2. **World › Project… › Overview**.

| Check | Yes / No / Unclear |
|---|---|
| 10a-1. Who is the PLAYABLE CHARACTER? (Its file names nobody. Claude's test: the Skyfarer.) | |
| 10a-2. Is that who you remember playing this world as? | |
| 10a-3. Scroll the right-hand side for **THIS WORLD'S OWN VERSIONS**. How many? Did the editor tell you this happened on opening? | |

3. Play, look, stop. **Leave without saving** when you open the next world.

**10b. Coming back where you were.**

1. Open `benchmark`. Play. Walk well away from where you began. Tab.
2. Play again.

| Check | |
|---|---|
| 10b-1. Did play begin where you stopped, or where it first began? | |
| 10b-2. Did anything say so? | |
| 10b-3. Is that what you want a game's start to do while you are making it? | |

**10c. The character left over.**

1. With `benchmark` open and the ranger its player, open `editor\dioramas\driftwood_island.json`. Do not press Play.
2. Find the player figure standing in the world.

| Check | |
|---|---|
| 10c-1. Is the figure the castaway (this world's player) or the ranger (the last world's)? | |

3. **Leave without saving.**

Note:

---

## When you are done

Tell me, for each test, the checks you answered **No** or **Unclear**, and anything in your notes. That is the hand-test result for L1, L2 and L3. A workflow moves to "Hand tested" only for the tests you completed.

What these tests do **not** cover, and so stays open: generated worlds, worlds from a map, genres, restoring a backup, the Player start, joining worlds, importing a model, making a new character or moveset, and exporting.
