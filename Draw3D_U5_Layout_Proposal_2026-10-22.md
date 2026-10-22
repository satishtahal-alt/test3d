# Draw3D: U5 layout proposal, one place per job (2026-10-22)

**Status: built 2026-10-22,** after the user said "complete U5". The questions of section 5 were taken as recommended: (1) yes, left = whole world, right = drawn and built; (2) Roads on the right bar; (3) the Build menu kept as the right bar's index; (4) moved outright, no links; (5) a Style button on the toolbar opening the Styles window (no new gallery). The left Sky is called Weather. What was built is in the inventory, section 13zz41.

~~Status: a proposal for your review. Nothing in it is built.~~

Your U0(b) answer was: do not merge the toolbars yet, but decide after reviewing a proposed layout. This document is that layout.

U5 is built only after you answer the questions in section 5. Until then:
- every tool stays where it is today;
- every control keeps its id, so tours and demos that open it by name still work.

Sources: the audit's F5 and section 6 rule 8 ("one place per job"), the inventory (`Draw3D_UI_Inventory.md`), and the code as it is now:
- `desk.gd` `TOOLS`, `GROUPS`, `_paint`, `_structures`;
- `buildbar.gd` `GROUPS`.

---

## 1. Today: which jobs have more than one entry point

| Job | Entry points today | What differs between them |
|---|---|---|
| **Build** | (a) the left toolbar's **Build** ‹ list (roads, lots, streets, walls, fences, hedges, houses of each sort, grid structures);<br>(b) the right bar's **Build**, **Town**, **Paths**, **Water**;<br>(c) the top bar's Build menu (7cz), which lists what (b) has | (a) has roads, lots, streets and grid structures, which (b) does not. (b) has the drawn sorts with their looks and handles. (a) and (b) both draw houses, by different words |
| **Paint** | (a) the left toolbar's **Paint** ‹ list: ground (F8), objects (a design or a group), where plants may grow;<br>(b) the right bar's **Paint**: grass, flowers, bushes, trees, rocks, crops, decor, life, climbers | The two say "paint" for three different things: the ground's surface, scattering objects, and the right bar's plant and decor brushes |
| **Erase** | (a) the left toolbar's **Erase** (the Erase window: brush, a whole layout, the world, what is selected);<br>(b) the right bar's **Erase** (what is built, by brush or all at once) | Two windows that overlap; neither says the other exists |
| **The world's look** | (a) **Sky** on the left toolbar, which opens the window titled "Time and weather";<br>(b) Window > **Styles**;<br>(c) the right bar's **Look** (one house's palette, sort and colours);<br>(d) the style row inside Time and weather | Four names for overlapping things. "Look" is the only one about one house |

The jobs that already have **one place each**, and stay as they are: Select, Move, Terrain shaping, Objects, Plants (generators), the Inspector, Behaviours, Character Studio.

---

## 2. The proposed layout

```
 Top bar:  World   Edit   View   Characters   Window        (Build menu: removed, or "Build ▸" opens the right bar)

 Left toolbar (general tools)                         Right bar (building, as a player builds)
 ┌──────────────────┐                                  ┌──────────────────┐
 │ Select           │                                  │ Build      ‹     │  houses, towers, walls, fences, hedges
 │ Move             │                                  │ Town       ‹     │  shops, apartments, bars
 │ Erase            │  ← the one Erase window          │ Roads      ‹     │  NEW group: road, lot, street,
 │ Terrain   ‹      │  shape, and Paint ground         │                  │  grid walls/fences/floors
 │ Objects   ‹      │                                  │ Paths      ‹     │
 │ Plants    ‹      │  generators, forests, meadows    │ Water      ‹     │
 │ Time & weather   │  (was "Sky")                     │ Brushes    ‹     │  was right-bar "Paint": plants, decor,
 │ World style      │  (was Window > Styles)           │                  │  life, and "paint with an object/group"
 └──────────────────┘                                  │ Look       ‹     │  one house's look; "Follow world style" first
                                                       │ Erase  → opens the one Erase window, set to "what is built"
                                                       └──────────────────┘
```

**The rule behind it:**
- **Left** is what works on the whole world: selecting, moving, erasing, the land, objects, plants, time and weather, the world's style.
- **Right** is what a player builds and decorates by drawing, the way Tiny Glade lays it out. That is the precedent you found works, from 7cv onward.

---

## 3. Each control: where it is now, where it goes

The ids are kept. A tool that moves leaves a **link** behind in its old place for one release, labelled "→ now in …", unless you say otherwise (question 4).

| Control (id) | Now | Proposed | The old place |
|---|---|---|---|
| `structures` (left "Build ‹") | left toolbar | removed from the left. Its roads, lots and streets go to the right bar's new **Roads** group (`cb_roads`), and its drawn sorts are already under Build | a link "Build is on the right bar →" for one release |
| Build menu (top bar) | top bar | removed; or kept as "Build ▸", which only opens the right bar | |
| `paint` (left "Paint ‹") | left toolbar | split by what it paints:<br>• "Paint ground" becomes a tool of **Terrain** (it already is, as F8);<br>• "Paint with an object/group" and "where plants grow" go to the right bar's **Brushes** | the left button is removed. F8 still paints the ground |
| `cb_paint` (right "Paint") | right bar | renamed **Brushes**: the same brushes, plus painting with an object or a group | |
| `erase` (left) | left toolbar | **the one Erase** | |
| `cb_erase` (right) | right bar | opens the same Erase window, its choice set to "what is built" | its own shelf is removed |
| `environment` (left "Sky") | left toolbar | renamed **Time & weather** (the window's title already is, since U1) | |
| Window > Styles | Window menu | **World style** on the left toolbar: a gallery with a picture per style, previewed on hover, Customise colours… (a copy named "Lookout (mine)") | still listed under Window |
| style row in Time and weather | that window | removed from there. It says "World style →" | |
| `cb_look` (right "Look") | right bar | stays: one house's look only. Its palette list begins with "Follow world style" | |

---

## 4. What it costs, and what it risks

- **Size:** interaction work, mostly in `desk.gd` (`TOOLS`, `GROUPS`, `_paint`, `_structures`) and `buildbar.gd` (`GROUPS`, a Roads group, the Erase hand-over). Small engineering: the Styles gallery (thumbnails of a style's palette, hover preview).
- **Tours and demos:** they pick tools by id (`desk.pick`). The ids are kept, and a removed button's id is redirected to its new place, so `pick("structures")` opens the right bar's Roads. The acceptance check lists every control id before and after the change, and names each one that moved.
- **Muscle memory:** your hand tests since 7cu used the right bar. The left Build and Paint lists are the ones that move.
- **Not touched:** the Inspector, the Behaviours window, the Studio, Terrain's shaping tools, Objects, Plants.

---

## 5. Questions for you (U5 waits on these)

1. **The split:** left = whole world, right = what is drawn and built. Is that the rule you want?
2. **Roads, lots and streets:** move them to a Roads group on the right bar? Or keep a small left "Layouts" list for them?
3. **The top bar's Build menu:** remove it, or keep it as "Build ▸", which only opens the right bar?
4. **Links in the old places:** keep "→ now in …" links for one release, or move things outright?
5. **World style:** a toolbar button with a gallery, as drawn here? Or only rename Window > Styles?

Answered, U5 is built as one pass. It ends with:
- the control-id comparison;
- a rewrite of the checklist;
- inventory section 13zz41;
- your hand test of T4, T5 and T8 from a fresh world.
