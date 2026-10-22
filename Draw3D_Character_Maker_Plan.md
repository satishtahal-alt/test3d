# Draw3D: making a character, and what fights it (plan, 2026-10-16)

The user's hand test of 7bd asked for character creation as "the next step": choosing a
character's eyes, face, head and body type, its starting gear, and cycling through clothing
to see how it looks while it performs its moves in the Character Studio. It also asked that
enemy types (with their own moves and reactions to being hit, and not only people: four
legs, many legs), an expression system for faces, and physics for what is worn be kept on
the list. This file says what exists, what can be built without new assets, and what needs
a decision first. Nothing in it is built except where it says so.

## What exists (2026-10-16)

- A character is a glTF model with bones (`characters.txt`: `model`, `rig`, `size`), its
  clips, its look (which clip shows what), its set of moves, and what it wears and carries
  (`gear` lines; `gear.txt` has 33 things made of simple parts, and places on a body).
- The models are the mannequin (Draw3D's own) and Quaternius's Universal Animation Library
  models: neither has face shapes (blend shapes) or eyes of its own; each is one material.
- The Studio's Gear tab puts things on and takes them off; `spare`, `two_hands_at`, IK for
  the second hand; since 7be, cloth that swings (`sways`).
- Enemies: anyone with a body is struck by a move that `strikes`, and begins its set's
  `upon struck` move (thrown back). Since 7be the Studio's yard (`yard.txt`) puts dummies
  on a field: standing, jumping, cutting back; a target on a post.
- Kinds (the host's own bodies, `cast.txt`) already have rigs for a person, a four-legged
  creature and a vehicle, `sway` bones and meshes; the Studio works on characters
  (humanoid maps) only.

## Tier 1: the body and the clothes, with what is there (no new assets)

Buildable in one pass, in a new Studio tab ("Make"), every choice a line of the character
so it is written, undone and kept with the world as the rest is:

1. **Proportions by bone.** `shape BONE scale` lines (or named sliders writing them):
   height, build (chest and hips wide), head size, arm and leg length. Godot scales the
   bones of the skeleton; clips still play (a longer leg strides farther, which `stride`
   already allows for). The figure's points and treads follow the bones.
2. **Colours.** `colour SURFACE r g b`: the model's materials tinted (skin, clothes) where
   a model has more than one surface; the mannequin gets a second material for clothes
   only if its file is changed, so first a tint over the whole and the gear's own colours.
3. **Outfits.** `outfit NAME` blocks of gear lines: the starting gear is the first; Next
   outfit cycles in the preview while a move or the yard plays. A game gives a character
   another outfit by name.
4. **Start from**: a list of the character models there are (the mannequin, Quaternius's
   two), and Copy (it exists) to make one's own.

## Tier 2: face, eyes, hair, expressions (needs a decision: assets)

None of this can be done well on the models there are: they have no face to move. One of
these has to be chosen, which is a choice of assets and so the user's (the standing rule:
reuse first, approval for big bets):

| Source | What it gives | Licence | Cost here |
|---|---|---|---|
| **MakeHuman / MPFB 2** (Blender) base meshes and its exported humans | Body and face shapes as shape keys (sliders), eyes, eyebrows, hair, clothes; a facial rig option | Assets made with it CC0; MPFB itself GPL as a tool (Draw3D is not distributed, the games are: only exported assets go into games) | An exporter path (Blender to glTF with shape keys), a map for its skeleton, shape keys driven by sliders. The largest, and the most like a "character creator" |
| **Quaternius Ultimate Modular Characters** | Swappable heads, hair, bodies, outfits on one skeleton | CC0 | Parts swapped by `part` lines; no face shapes (expressions by textures or none) |
| **VRoid / VRM models** | Anime-style faces with expression blend shapes (joy, angry, blink, A I U E O) | Per model (VRoid Studio output is the author's) | A VRM importer (glTF extension), a map; expressions come with it |

Recommendation: Quaternius's modular characters first (same family as the clips in use,
CC0, swaps cover "cycle through clothing" and head and hair), then MakeHuman for sliders if
modular parts are not enough. Asked of the user before anything is downloaded.

**Expressions**, once a model has blend shapes: a look row says what face a move shows
(`cut_a/cut face angry 0.8`), blended in and out as clips are; standing, a blink now and
then and eyes that look at what the body is locked on to (`lockon.gd` says who).

## Gear and clothes that look like things (asked 2026-10-16, after 7bg; nothing built)

The user's fourth hand test found the boots are two blocks: every thing in `gear.txt` is
made of simple parts (boxes, rods, cones) by Draw3D itself, as a placeholder. Asked for,
"for the future", not now:

1. **A weapon designer.** A sword made and changed by numbers: blade length and breadth,
   its curve and point, the guard, the handle and pommel; two blades (a double-ended sword).
   A shield's shape (round, kite, tower) and size. Like the tree generator (7aa): a generator
   makes a design (`gear.txt` item with a `recipe`), its numbers kept so it can be changed
   again, hand edits protected. Rigid things (swords, axes, shields, hats, helmets, packs)
   need no new assets: buildable as a generator of parts or of one mesh.
2. **Clothes that bend with the body** (shoes, trousers, shirts, capes, hats changed in
   shape, size and colour). A rigid part cannot bend at a knee; clothes have to be skinned
   to the body's bones. Two ways, for the user to choose:
   - **Grown from the body**: a garment made from the body's own mesh where it covers
     (feet for shoes, legs for trousers), pushed out a little and given the body's bone
     weights, so it bends as the body does; its numbers say where it starts and ends, how
     loose, its colour. No assets; fits any of the models there are; looks simple, not
     tailored.
   - **Modular clothes from a pack** (Quaternius's modular characters, CC0): outfits made
     for one skeleton, swapped; better looking, but only what the pack has, and its own
     body.
3. **Fingers round a grip.** The hand closes on what it holds: a grip pose per thing
   (finger bones bent, as a look row's `bend` does), held while it is drawn, on models
   with finger bones (UAL2's has them; the mannequin has none). Small, and the first of
   these to build.
4. **A starting fit per model**: built (7bh), the Gear tab's *Keep for this model*
   (`fits.txt` in Draw3D's own folder).

Faces, eyes, hair and expressions are Tier 2 above: they need a model with a face to move,
which is an asset choice.

## The user's answers after 7bh (2026-10-16): the to-do, parked until the user comes back

The user said the characters, moves and combos have come far enough for now ("a significant
step in the right direction"), asked for these to be kept as the to-do, and moved on to the
next major section of the roadmap. Nothing here is built. When the characters are taken up
again, this is the order:

1. **Fingers round the grip, first.** Tried by hand on UAL2's model: the hand does not
   close on the sword (on another model it looked right). A grip pose per thing held
   (the finger bones of the hand that holds it bent, by numbers on the gear line, with a
   preset "closed on a handle"), held while drawn and let go when put away; the Gear tab
   shows and nudges it. On models without finger bones, nothing. Check on UAL2 first.
2. **A starting library of assets: the download is approved** (the user, 2026-10-16).
   What may be taken: things free to use in sold games (CC0 first; CC-BY when credited);
   never non-commercial or "no games" licences. The user does not redistribute the assets
   by themselves; they are for the engine and the games made with it. **Every asset
   downloaded gets a line in a credits file** (`editor/assets/CREDITS.txt`: what, from
   where, author, licence, date) so each game can credit what it uses, even where CC0
   does not ask it. Each pack goes into its own folder, untouched, and is read by Draw3D
   from there. A reasonable amount, as a starting library:
   - characters: a humanoid with **face shapes** (blend shapes for expressions and
     visemes) and a skeleton the Studio's maps can read (see 3); Quaternius's modular
     characters (heads, hair, bodies, outfits on one skeleton);
   - weapons: swords, axes, spears, shields, bows (Quaternius, Kenney and the like, CC0);
   - hats, helmets, packs;
   - clothes, preferably modular on the same skeleton as the characters.
   Wanted with them, as the user said: **a way to change them in the engine** so they fit
   other characters (sized and nudged per model, as fits.txt already does for gear;
   coloured; for clothes, refit to another body by its bones), not only used as they come.
3. **One character that does everything** (the user's end goal, which the models chosen
   so far did not keep in view): a model that plays the clips of the move sets, wears
   gear and clothes, and shows expressions. What this needs, said plainly: the clips are
   **not** tied to UAL2's model. The Studio plays a library's clips on any humanoid
   through its map (`map ual`, `map mixamo`, the humanoid profile), so a model *with* a
   face can play the same moves. UAL2's model and the mannequin have no face shapes, and
   shapes cannot be added to them here without a modelling tool (Blender is not on this
   machine; it could be, as a tool, if the user agrees). So the way is: choose a model
   that has face shapes (MakeHuman/MPFB export, a VRM model, or another CC0 humanoid
   with shape keys), map its skeleton, and check that the moves, the gear fits and the
   grip all carry over. Then Tier 2's expressions (above) are built on it.
4. **Expressions**: a face per move phase (angry on a strike, a wince when struck),
   blinks, eyes that look at what is locked on; as Tier 2 says.
5. **The weapon designer** and **a shield's shape and size** (above, "Gear and clothes
   that look like things", 1).
6. **Clothes made and changed in the engine** (shoes, trousers, shirts, capes, hats:
   shape, size, colour): from the pack's modular clothes refitted, or grown from the
   body (above, 2); the user wants an editor for them either way.

## Enemies, creatures, reactions

- **Enemy types** are characters with sets of their own: what they can do is their set,
  how they react is their `upon struck` moves. First tier built (7be): the yard's dummies.
  Next, within what exists: `upon struck` told how the blow came (from behind, heavy, a
  blade or a blunt thing) so a set has several reactions; a dummy's line says which set.
- **Behaviour** (an enemy that closes in, circles, attacks in an opening) is the NPC
  behaviour system (amendment 7g, `reactions.txt`, `sequences.txt`): to be extended, not
  paralleled, when the user asks for it.
- **Not only people.** The Studio shows characters with a humanoid map. A creature's rig
  (four legs, many legs, a tail) needs maps of other profiles in `characters.txt` and a
  Studio that does not assume hands and feet: the creature engine's work, planned apart.

## Physics of what is worn

- Built (7be): `sways` on a thing of `gear.txt` (a cape): it hangs from where it is worn and
  swings after the body on a spring (`sway.gd`, the same as a kind's `sway` bones), never
  into the body; stiff when not said (a shield, a helmet). The Gear tab's "Cloth: swings
  after the body" writes it.
- Later: real cloth (Godot's SoftBody3D) for a long cape or a flag, colliding with the
  body's capsules; a weight that bounces (a pouch); ragdolls for the struck (Godot's
  physical bones). Each a step of its own.

## Layout

The user found the Studio's side-by-side, drag-to-resize layout "a huge improvement" and
suggested it where other windows are convoluted. Kept as a held finding: the next pass that
touches a crowded window (Makers, Terrain, Layout) offers the same split.
