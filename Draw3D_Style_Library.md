# Draw3D: game-inspired style library

Written 2026-10-17 (amendment 7bn). This is the catalog of styles made after games' looks. Each style here is also a style in `editor/godot/styles.txt`, so it can be used straight away. In the editor, open Window > Styles (or Sky > Styles…) to find one by a word, give it to a world, try it on a sample town, or make your own copy and change it.

The mechanics of the same games are in a separate file, `Draw3D_Mechanics_Reference.md`.

**The reference document of 30 games was not with the request.** It was not attached, and it was not found anywhere on this machine (the project, Downloads, Documents, Desktop). This catalog therefore covers the eight games the brief named as priorities. The other 22 slots are left in the index for when the document is added to the project folder. Every source detail below was found by web search on 2026-10-17; the sources are listed at the end.

## How to search this catalog

Every entry has a **Tags** line. These are the words the Styles window finds a style by: it matches the name, what the style says, what it is after, and its tags. The same words work in a text search of this file.

| Style | After | Tags | Confidence in the source |
|---|---|---|---|
| `siege` | Into the Dead: Our Darkest Days | survival horror town dusk cinematic grim | medium |
| `dreamfield` | GREENFIELD | surreal dream island bright pastel glow | low–medium |
| `silent_reel` | Silent Sadie | vintage film monochrome 1920s slapstick cinematic | low (setting only) |
| `rust_belt` | Steel Rats | industrial rust smog city vehicles grim | medium |
| `neon_noir` | Neon Blood | neon noir cyberpunk city night rain futuristic | medium |
| `gothic_ward` | Withering Rooms | gothic horror victorian interior candle night | medium |
| `factory_deep` | Darwin's Paradox! | industrial underwater factory sewer creature stealth | medium |
| `bright_blade` | Cloven Blade | fantasy colourful bright combat acrobatic | **none: game not found** |
| 22 more | the 30-game reference document | — | not provided |

**Confidence** means how much of the game's look the sources confirmed. A style's colours and numbers are always this project's own choices made from those descriptions. Nothing was taken from the games themselves: no art, names, shapes or textures. Only the general look is followed.

## What a style is made of in Draw3D

A style is one entry of `styles.txt`, kept with a world as eight parts. Each part remakes only what reads it, so a change to the haze rebuilds nothing.

| Part | Words | What it governs |
|---|---|---|
| build | `wall trim stone wood roofs panes pitch` | buildings: materials and colours, roof shape |
| village | `storeys` | building proportions |
| wild | `bark leaves greens blooms fruits rock` | vegetation and stones: colour |
| growth | `pines woods meadow trees strewn` | what woods and open ground are made of, and how thick |
| ground | `grass cliff sand seabed dirt` | the terrain's colours |
| air | `day water moon at HOUR sky sun fog fill` | lighting, sky, haze and fill light by the hour |
| **look** (new) | `tone grade cast glow shade film figures` | the picture: tone mapping and exposure; saturation, contrast and brightness; a colour cast; bloom; ambient occlusion; vignette, grain and flicker; a colour over characters |
| **about** (new) | `says after tags yours` | what the style is, what it is after, what finds it, and whether a person made it |

**Combining styles:** `takes PART from NAME` takes one part as another style has it, for example `takes air from silent_reel`. The rest stays the style's own, so two styles combine part by part, not as a blend that muddies both.

**Variants:** `like NAME` is the original plus your own lines. Make my own in the Styles window writes such a copy, marked `yours`, at the end of the file. The window's controls and Take change only styles marked `yours`, so the original stays as it was.

## The styles

Each entry follows the brief's list of fields. A field the sources did not cover says what was assumed, and the open questions are listed under it.

### siege — after Into the Dead: Our Darkest Days (PikPok)

- **Genre, references:** a side-scrolling shelter survival game. You scavenge small 2.5D maps where stealth matters, while the shelter's barricade wears down; a flashlight or noise draws the undead. *(previews, Steam Deck HQ, Thumb Culture)*
- **Palette:** colour drained away (grade saturation 0.5), cold grey-blue cast (`cast 9fb0b8 0.25`); walls of weathered beige and grey, boarded wood, dark roofs; olive and khaki vegetation. **Variation:** roofs and panes each have four close greys and browns. Nothing is saturated.
- **Shape language:** low, plain houses (pitch 0.35); ordinary town proportions. *(assumed)*
- **Materials:** weathered paint and timber, by colour only (Draw3D has no texture or wear layer yet).
- **Lighting:** a short day (7–17); a low cold sun that warms toward evening; long shadows; strong ambient occlusion (`shade 2.6`); a little glow; vignette and light grain (`film 0.5 0.15 0`).
- **Terrain and vegetation:** sparse woods (0.6), a meadow almost without flowers, dry olive grass.
- **Camera:** the game is seen from the side. Draw3D's eyes are in `eyes.txt`; a side-on eye that follows along a line is not built (see the mechanics reference).
- **Characters:** a grey-green colour over people (`figures 8a9090 0.15`).
- **Suits:** abandoned towns, outbreak and siege scenarios, stealth scavenging.
- **Open questions:** the exact palette and time-of-day progression; the look of cutaway interiors (the brief mentions cutaway buildings, but the sources found did not describe them).

### dreamfield — after GREENFIELD (Six Foot Giraffe)

- **Genre, references:** a story-driven puzzle platformer, solo-developed. The dreamlike island has five stations, each built around an energy (combustion, hydro, wind, magnetic, light or photonic). There are vehicles for land, air and water, an exo-suit, a jetpack, and guardians. Sources describe its art as "beautifully surreal", and its release is listed as 2026. *(Gematsu, Film Stories, GamingOnLinux, listings)*
- **Palette:** pastel walls, roofs of cyan, pink, gold and periwinkle; teal-green leaves with pink and blue among them; a sky too blue by day and pink at dusk; a glowing teal sea.
- **Shape language:** steep roofs (pitch 0.55). *(assumed)*
- **Lighting:** ACES tone mapping at 1.15 exposure; strong bloom (`glow 1.4 0.85`); light shade (0.8); a violet night with bright moonlight.
- **Terrain and vegetation:** a meadow with many flowers (0.35); pale sand.
- **Camera:** sources call it a side-scroller.
- **Characters:** no colour over them.
- **Suits:** dream worlds, puzzle islands, gentle exploration.
- **Open questions:** the real palette. The "underwater woman" and the station designs are content, not style. How an energy theme should change a biome is a question for the mechanics side.

### silent_reel — after Silent Sadie (Kapra Games)

- **Genre, references:** a puzzle platformer with stealth, set in 1920s Hollywood. The heroine is an acrobat from a vaudeville family who wants to act. It is not on sale yet and has no release date. *(GG.deals listing)*
- **Palette:** black and white (`grade 0 1.4 1.05`), with the warm tint of old film stock (`cast e8d0a8 0.25`).
- **Shape language and materials:** the world's own. Only how it is shown changes, apart from a few lighter wall and roof tones.
- **Lighting and post:** Reinhard tone mapping; hard contrast; strong vignette, grain and flicker (`film 0.65 0.5 0.4`).
- **Camera, characters, animation:** the brief says expressive animation and slapstick. In Draw3D these belong to the Character Studio (moves, phases, shot camera), not to a style.
- **Suits:** silent-film comedy, period pieces, flashbacks.
- **Open questions:** whether the game itself is monochrome. The sources say only "1920s Hollywood", so the film look is this project's reading of "vintage cinematic presentation" in the brief. Intertitles and frame-rate stutter are not built.

### rust_belt — after Steel Rats (Tate Multimedia)

- **Genre, references:** a 2.5D motorbike action game (2018). Its front wheel is a chainsaw; you switch between characters; reviews praise its "crisp 2.5D graphics" and moody visuals and criticise its camera. *(TheSixthAxis, GamingTrend, Thumb Culture, COGconnected, Xbox Tavern)*
- **Palette:** red-brown brick, grey corrugated iron, rust roofs, a furnace-coloured sky, and an orange cast (`cast d08a4a 0.25`).
- **Shape language:** flat industrial roofs (pitch 0.2).
- **Lighting:** thick orange haze all day; bloom for sparks (`glow 0.7 1.0`); `shade 2.2`.
- **Terrain and vegetation:** few trees (0.4), dull olive grass, dark earth, a grey-brown sea.
- **Suits:** industrial cities, vehicle levels, post-industrial ruins.
- **Open questions:** the palette is inferred from the industrial setting. The motorbike and its physics are mechanics.

### neon_noir — after Neon Blood (ChaoticBrain Studios)

- **Genre, references:** a pixel-art adventure (originally planned as a JRPG). Pixel characters stand against a dingy, neon-lit 3D cityscape, with hand-drawn cutscenes. The art is praised; the gameplay reviews are mixed (OpenCritic 66). *(RPGFan, Checkpoint Gaming, OpenCritic)*
- **Palette:** dark walls and roofs; panes and awnings in pink, cyan, violet and yellow; a purple haze; a dark sea.
- **Shape language:** flat roofs (0.1) and tall houses (storeys 4.5–6).
- **Lighting:** a short, dim day (9–15) and a long night; ACES tone mapping; strong bloom (`glow 1.8 0.75`); a violet cast; strong saturation and contrast.
- **Characters:** a lilac colour over people (`figures b0a0ff 0.2`).
- **Suits:** cyberpunk cities, night levels, noir stories.
- **Open questions:** emissive signs (the panes take the colours but give no light), rain, and wet reflections are not built. Pixel-art figures in a 3D world are a rendering question (a figure rendering mode), not a style colour.

### gothic_ward — after Withering Rooms (Moonless Formless)

- **Genre, references:** a 2.5D horror RPG in a Victorian mansion and asylum (Mostyn House) that changes each time you die; a dream loop. Reviews praise its Victorian look and atmosphere. *(RPGamer, GodisaGeek, PlayStation Country, GOG)*
- **Palette:** oxblood walls, dark panelled wood, green-grey stone, candle-gold panes, a dark sea.
- **Shape language:** steep roofs (0.8); tall houses (storeys 4–5).
- **Lighting:** a short, grey day (8–16); a long night under a cold moon; a candle-warm cast; deep corners (`shade 3`); vignette with a little grain and flicker.
- **Terrain and vegetation:** dense, dark woods (1.3), more pines (0.6).
- **Characters:** a dim warm colour over people.
- **Suits:** haunted houses, asylums, gothic horror.
- **Open questions:** the brief stresses interiors and detailed props. Interiors are rooms built on cells (L3), and props are library designs; the style colours them but cannot fill them. Candles as light sources are lamps (`lights … when dark`) that a design must carry.

### factory_deep — after Darwin's Paradox! (ZDT Studio, published by Konami)

- **Genre, references:** a 2.5D platformer released 2 April 2026. Darwin, an octopus, escapes an industrial complex back to the sea: swimming, hiding, camouflage, ink against cameras, sticky tentacles on walls and ceilings, slippery sludge. One review finds the factory floors and sewers drab. *(Checkpoint Gaming, The Outer Haven, GameSpew)*
- **Palette:** wet concrete and pipe greys, green-tinged lamplight, slime greens; a teal cast, as if seen through water.
- **Shape language:** flat roofs (0.15).
- **Lighting:** a teal-grey day, bloom on lamps (`glow 0.8 0.9`), `shade 2`.
- **Terrain:** the seabed and water in deep teal.
- **Suits:** factories, sewers, labs, an escape to the sea.
- **Open questions:** the game's own palette may be less teal than this. Camouflage and wall-sticking are mechanics.

### bright_blade — after Cloven Blade (not found)

- **No source was found** for a game titled "Cloven Blade". The closest results were *Cloven* (an early itch.io movement shooter) and *HALLOW BLADE* (a dark-fantasy first-person game), and neither matches. The style is made only from the brief's words: "colourful fantasy, agile combat and acrobatic movement".
- **Palette:** saturated greens, flowers everywhere, roofs of red, blue, gold, green and violet, white walls, warm gold light.
- **Lighting:** ACES tone mapping; strong saturation (1.4); a little bloom; light shade.
- **Suits:** high fantasy, action and adventure.
- **Open question:** what game the brief meant. Give its developer or a link, and the entry will be checked.

### Styles for a genre (added 2026-10-17, amendment 7bo)

Five more, each after a kind of game rather than one game, for the mechanics built in 7bo. Nothing here is taken from any game. Each is in `styles.txt` with its `says`, `after` and `tags`, and is found in Window > Styles by those words.

| Style | For | What it is | Tags |
|---|---|---|---|
| `moonlit_stealth` | stealth, the side-on stealth level | a clear moonlit night: deep blue shadows, cold silver light, warm lamps; the city's buildings (`takes build from city`) | stealth night moonlit blue dark sneaking |
| `security_feed` | what a camera sees | grey-green, nearly colourless, grain, flicker, dark corners | cctv camera surveillance monochrome green grain stealth |
| `toxic_works` | factories of puzzle-platformers, the level's acid | yellow-green haze, concrete and rust, thick shade | industrial toxic factory haze acid green grime |
| `bright_side` | side-on platformers | clean pastel walls and greens, a sky always fine, soft light, little shade | platformer side-scroller bright pastel cheerful 2.5d |
| `last_light` | survival of day and night | a long amber dusk, colour drained, the edges closing in; siege's buildings (`takes build from siege`) | survival dusk amber abandoned bleak apocalypse |

Verified: nothing to verify (they are after genres, in words). Checked: the host's style tests read `styles.txt` whole with them in it.

### The other 22

To be added from the 30-game reference document once it is in the project folder. Each will get the same entry, a style in `styles.txt`, and a row in the mechanics reference.

## Verified and unverified

| Detail | Status | Source |
|---|---|---|
| Into the Dead: Our Darkest Days: side-view shelter survival, 2.5D scavenging, stealth, flashlight and noise draw attention, a barricade that wears | verified (previews) | pikpok.com, steamdeckhq.com, thumbculture.co.uk |
| …cutaway buildings, dynamic lighting | **unverified** (in the brief only) | — |
| …its palette | **unverified** | — |
| GREENFIELD: solo developer Six Foot Giraffe, story-driven puzzle platformer, five energy stations, vehicles on land, air and water, exo-suit, jetpack, guardians, surreal art | verified (announcement, listings) | gematsu.com, filmstories.co.uk, gamingonlinux.com |
| …release (listed as 2026), "free", Unreal Engine 5 | **unverified** (listings disagree) | gamepressure.com, steamdata.ai |
| Silent Sadie: Kapra Games, puzzle platformer with stealth, 1920s Hollywood, vaudeville acrobat | verified (one listing) | gg.deals |
| …monochrome or film look, slapstick, expressive animation | **unverified** (in the brief only) | — |
| Steel Rats: 2.5D motorbike action, chainsaw wheel, character switching, praised visuals and sound, criticised camera | verified (reviews) | thesixthaxis.com, gamingtrend.com, thumbculture.co.uk |
| Neon Blood: ChaoticBrain Studios (Madrid), Meridiem Games, pixel characters in a neon-lit 3D city, hand-drawn cutscenes, mixed reviews | verified (reviews) | rpgfan.com, checkpointgaming.net, opencritic.com |
| Withering Rooms: Moonless Formless, 2.5D horror RPG, Victorian mansion-asylum that changes on death, dream loop, build variety, clunky combat | verified (reviews) | rpgamer.com, godisageek.com, gog.com |
| Darwin's Paradox!: ZDT Studio and Konami, 2 April 2026, octopus escape, camouflage, ink, sticky climbing, sludge, factories and sewers | verified (reviews) | checkpointgaming.net, theouterhaven.net, gamespew.com |
| Cloven Blade: anything | **not found** | — |
| Every palette, number and proportion in `styles.txt` | this project's own reading, not measured from any game | — |

## Implementation report (what was reused, added and deferred)

**Reused, not rebuilt:** the style system of `styles.txt` and `shapes/style.rs`, and the way a world keeps it (its parts as assets). Also the air-by-hour lighting (`air.gd`, `clock.gd`), designs for woods and meadows (`trees`, `strewn`), the restyle edit with Undo, the Environment window, F2, and the host's style reading. The authoring, build and runtime boundaries are unchanged: a style is text a person writes (authoring), the host keeps it in the document and rebuilds only what reads the changed part (build and generation), and Godot shows the light and the picture (runtime).

**Added:**

- in the host:
  - two parts, `look` and `about`;
  - `takes PART from NAME`;
  - the `air` answer carries the picture;
  - the `style` answer says what each style is.
- in Godot:
  - the picture applied to the world's Environment (tone mapping, exposure, adjustments, a colour-correction cast, glow, SSAO);
  - a film sheet for vignette, grain and flicker;
  - a colour pass over every character and creature.
- the Styles window: find, give to the world, no style, try on a sample, open the text, make your own, picture controls, and take a part from another style.
- eight styles after games.
- the demo `-Demo styles`.

**Deferred** (each needs a decision or a larger system):

- materials beyond colour: roughness, wear, and textures per style;
- emissive signs and windows (neon);
- rain and wet surfaces;
- outlines and toon shading;
- pixel-art figures in 3D;
- a side-on 2.5D camera along a line;
- interiors filled by style;
- a style's own library of props and designs (only `trees` and `strewn` say what things are made of);
- the 22 games of the missing document.

## Sources

Found by web search, 2026-10-17:

- [PikPok: Into the Dead: Our Darkest Days](https://pikpok.com/games/into-the-dead-our-darkest-days/), [Steam Deck HQ preview](https://steamdeckhq.com/news/preview-into-the-dead-our-darkest-days/), [Thumb Culture preview](https://www.thumbculture.co.uk/into-the-dead-our-darkest-days-pc-preview/amp)
- [Gematsu: GREENFIELD announced](https://www.gematsu.com/2023/09/atmospheric-story-driven-puzzle-platformer-greenfield-announced-for-pc), [Film Stories](https://filmstories.co.uk/news/greenfield-is-a-story-driven-puzzle-platformer-with-some-beautifully-surreal-artwork/), [GamingOnLinux](https://www.gamingonlinux.com/articles/18982/), [Gamepressure](https://www.gamepressure.com/games/greenfield/z367ae)
- [GG.deals: Silent Sadie](https://gg.deals/game/silent-sadie/)
- [TheSixthAxis: Steel Rats](https://www.thesixthaxis.com/2018/11/20/steel-rats-review/), [GamingTrend](https://gamingtrend.com/reviews/road-rage-against-the-machines-steel-rats-review/), [Thumb Culture](https://www.thumbculture.co.uk/steel-rats-review)
- [RPGFan: Neon Blood](https://www.rpgfan.com/review/neon-blood/), [Checkpoint Gaming](https://checkpointgaming.net/reviews/2024/12/neon-blood-review-bleeding-out/), [OpenCritic](https://opencritic.com/game/17757/view)
- [RPGamer: Withering Rooms](https://rpgamer.com/review/withering-rooms-review/), [GodisaGeek](https://www.godisageek.com/reviews/withering-rooms-review), [GOG](https://www.gog.com/game/withering_rooms)
- [Checkpoint Gaming: Darwin's Paradox!](https://checkpointgaming.net/reviews/2026/04/darwins-paradox-review/), [The Outer Haven](https://www.theouterhaven.net/darwins-paradox-review/), [GameSpew](https://www.gamespew.com/2026/03/darwins-paradox-review/)
- Cloven Blade, searched and not found: [Cloven on itch.io](https://jaxx888.itch.io/cloven), [Gematsu: HALLOW BLADE](https://www.gematsu.com/games/hallow-blade)
