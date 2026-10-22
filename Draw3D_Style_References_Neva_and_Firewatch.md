# Draw3D — Visual Style References: Neva and Firewatch

**Recorded:** 2026-10-19  
**Status:** Strategic visual-direction reference. These are desired capabilities and guidance, not claims about what Draw3D currently supports.  
**Purpose:** Add two complementary references to Draw3D's style-library direction. *Neva* informs expressive, painterly, nature-rich visual storytelling; *Firewatch* is especially valuable for stylized lighting, color scripting, atmospheric depth, and lush forest environments.

These references are for extracting reusable visual rules, not copying proprietary assets or reproducing either game's exact artwork. Draw3D should express the principles through its own style presets, procedural generators, asset library, and host-rendering capabilities.

---

## 1. Neva — Painterly, expressive nature and storybook composition

### Visual identity

*Neva* is a reference for an illustrated, painterly world in which natural forms, expressive silhouettes, character animation, and palette shifts carry much of the emotional tone.

Useful traits to study and support:

- **Painterly surface language:** brush-like edges, softly irregular contours, deliberate texture and mark-making, and a hand-crafted feel rather than uniform plastic materials.
- **Expressive silhouettes:** characters, creatures, plants, and environmental shapes remain readable at a glance. Shape language is designed, not merely realistic.
- **Stylized organic forms:** trees, grass, flowers, branches, rocks, and terrain can be simplified and exaggerated while remaining visually rich.
- **Illustrated composition:** foreground framing, layered midground forms, distant silhouettes, and deliberate negative space guide the eye.
- **Palette as storytelling:** dominant hues and color relationships can shift by region, season, mood, or narrative phase rather than remaining globally fixed.
- **Animation with character:** movement feels expressive and intentional, with posing, anticipation, follow-through, and personality rather than only generic looping motion.
- **Nature as an active visual presence:** plant masses, wind movement, falling leaves or petals, drifting particles, and environmental changes contribute to mood and composition.

### Draw3D systems suggested by this reference

- **Shape language:** angular, rounded, elongated, delicate, chunky, and asymmetrical tendencies for assets and vegetation.
- **Surface treatment:** clean flat color, painterly texture, soft gradients, visible brush-like variation, and controlled edge irregularity.
- **Palette roles:** sky, distant land, near land, foliage, flowers, character accents, highlights, shadows, and atmospheric color.
- **Composition presets:** foreground framing, layered landscape bands, silhouette-led staging, and open spaces emphasizing characters or landmarks.
- **Organic variation:** authored families of plant/rock shapes, controlled silhouette variation, color-role variation, and non-uniform placement.
- **Animation style profiles:** restrained sway, expressive secondary motion, deliberate timing, and per-part motion where the asset supports it.
- **Mood/region transitions:** palette, vegetation, lighting, and atmosphere can change together to establish a new emotional phase or biome.

### What to extract

1. How silhouettes remain distinct against detailed or textured backgrounds.
2. How foliage and terrain become painterly masses without looking like generic low-poly props.
3. How palette and composition change the emotional reading of the same kind of landscape.
4. How character motion and environmental motion share a consistent visual rhythm.
5. How to preserve authored irregularity while supporting procedural generation and reusable asset families.

**Potential applications:** painterly fantasy landscapes, illustrated adventure worlds, storybook biomes, expressive character-led exploration, stylized ruins reclaimed by nature, and scenes where environmental art communicates mood as strongly as geometry.

---

## 2. Firewatch — Stylized lighting, color scripting, and lush environments

### Why this reference is especially important

*Firewatch* is a major reference for making a relatively simplified visual world feel rich through **lighting, color, composition, atmospheric perspective, and carefully layered vegetation**. Its appeal is not simply “low-poly trees.” The overall image depends on silhouettes, sun direction, warm and cool color masses, haze, depth, and environmental density.

Study it as a complete visual system rather than reducing it to a tree model or a single orange palette.

### Visual identity

- **Designed, simplified forms:** terrain, cliffs, tree crowns, trunks, and distant mountains use deliberate, legible shapes instead of relying on photorealistic surface detail.
- **Strong color blocking:** large color regions establish the composition before small details are added.
- **Warm/cool contrast:** warm sunlight and sunlit foliage contrast with cooler shadows, shaded forest, and distant atmospheric colors.
- **Time-of-day color scripting:** dawn, midday, late afternoon, sunset, overcast conditions, and dusk coordinate light, sky, environment response, and atmosphere.
- **Atmospheric perspective:** distant terrain and vegetation lose contrast and detail and shift toward the color of the atmosphere, producing a sense of scale.
- **Layered forest depth:** foreground trunks and foliage, readable midground clearings, and overlapping distant tree/mountain silhouettes make forests feel deep rather than like a flat wall of repeated assets.
- **Lush environmental composition:** vegetation forms coherent masses and clusters, with openings, edges, clearings, paths, and sightlines—not just evenly distributed trees.
- **Cinematic framing:** branches, trunks, ridges, and terrain frame views toward landmarks or distant horizons.
- **Controlled detail:** silhouettes and lighting carry the image; extra texture or geometry supports readability rather than competing with it.

### Lighting and atmosphere capabilities to investigate

Draw3D's style system should eventually configure lighting and atmosphere as a coordinated preset, instead of treating them as unrelated sliders.

- **Directional sunlight:** direction, elevation, warmth/coolness, intensity, and softness.
- **Shadow quality and character:** readable cast shadows, suitable softness, stable shadowing across terrain and foliage, and settings appropriate to style and performance budget.
- **Sky and ambient illumination:** sky gradient or sky model, ambient color, environment contribution, and a useful balance between direct sunlight and shade.
- **Fog and aerial perspective:** distance-based color blending, density, height distribution where supported, and independent control over how quickly distant objects lose contrast.
- **Volumetric or simulated light shafts:** visible beams through canopy gaps when the host renderer and performance budget support them; a convincing non-volumetric approximation may be an acceptable fallback.
- **Exposure and tone mapping:** preserve bright sky and sunlight while keeping shaded vegetation legible; avoid washed-out images or crushed shadows.
- **Color grading:** coordinated warm highlights, cooler shadows, restrained saturation in the distance, and scene-wide palette shaping.
- **Contact and local shading:** objects sit convincingly on the ground and overlap clearly, without relying on exaggerated black outlines.
- **Weather/time variants:** haze, clear air, overcast softness, dusk warmth, or other variations modify the scene coherently.

Use capabilities available in Godot and the project's actual rendering path. Do not assume every advanced effect is already available or must be implemented immediately. Inventory the host renderer and existing Draw3D settings first, then test which effects materially improve the result.

### Lush forest and vegetation capabilities to investigate

A convincing forest needs more than a high tree count. Support:

- **Distinct plant roles:** canopy trees, narrow/tall trees, broad crowns, saplings, shrubs, groundcover, grasses, ferns, flowers, deadwood, and locally appropriate forms.
- **Layered density:** canopy, understory, and ground layer can be authored or generated separately, with controls for density and height.
- **Clustered distribution:** groves, edges, thickets, clearings, sparse patches, and natural transitions instead of uniform random scatter.
- **Terrain-aware placement:** slope, elevation, moisture/biome, paths, clearings, and user-defined exclusion areas influence plant placement.
- **Sightline-aware composition:** intentional gaps and vistas keep forests dense without hiding every landmark or making navigation unreadable.
- **Stable variety:** controlled species, silhouette, scale, rotation, and color variation reduce repetition while preserving a coherent style.
- **Foreground framing:** selected branches, trunks, and foliage can frame the camera view without blocking gameplay or interaction.
- **Environmental motion:** shared wind direction, gusts, phase variation, and asset-specific responses make vegetation feel like one coherent environment rather than identical loops.
- **Performance-aware populations:** use instancing, level of detail, culling, and suitable simplified representations where available; reserve costly detail for areas where it matters.

### What to extract

1. How light direction and shadow placement reveal hills, trees, and clearings.
2. How warm sunlight and cool shadows create depth and mood without photorealistic materials.
3. How fog and atmospheric color make distant layers recede.
4. How forest density is balanced with openings, landmarks, and readable navigation.
5. How a small number of strong color and silhouette decisions make a scene feel art-directed.
6. How the same environment changes under different times of day and weather.
7. How lighting, palette, vegetation, and camera composition work together as one style.

**Potential applications:** lush wilderness exploration, forest adventure games, mountain and canyon environments, outdoor narrative games, stylized survival worlds, cinematic trails and lookout points, and natural landscapes needing strong visual quality without photorealistic assets.

---

## 3. Shared lessons for Draw3D's Style system

The references overlap in authored shape language, deliberate palettes, layered composition, and expressive environments, but should remain distinct styles.

| Style dimension | Neva-inspired direction | Firewatch-inspired direction |
|---|---|---|
| Main visual emphasis | Painterly illustration, expressive nature, emotional storytelling | Lighting, color scripting, atmospheric depth, cinematic wilderness |
| Shape language | Hand-crafted, expressive, varied, painterly contours | Clear designed silhouettes, bold simplified masses |
| Surface language | Brush-like texture and illustrated irregularity | Clean, controlled surfaces with detail subordinate to composition |
| Vegetation | Painterly plant masses, expressive forms, mood-driven variation | Layered forests, strong canopy silhouettes, clustered density and sightlines |
| Palette | Can shift strongly with narrative mood and region | Warm light, cool shadow, atmospheric distance, time-of-day palettes |
| Lighting | Supports illustrated mood and focal hierarchy | Primary scene-shaping tool: sunlight, shadow, sky, and haze work together |
| Depth | Layered illustration and compositional separation | Atmospheric perspective and overlapping forest/terrain silhouettes |
| Motion | Expressive timing and secondary motion | Subtle environmental motion supporting immersion and mood |

### Architectural principle

A style should be a coherent configuration across multiple systems—not just a material preset. Depending on the style, it may influence:

- asset geometry and proportions;
- silhouettes and shape variation;
- materials and surface treatment;
- color roles and variation;
- vegetation species, density, clustering, and placement;
- direct light, ambient light, shadows, sky, fog, and color grading;
- wind and secondary motion;
- particles and subtle environmental effects;
- camera framing and depth treatment;
- generation rules and composition defaults.

Not every style needs to change every dimension. Let each style declare which dimensions it uses and provide sensible defaults for the rest. Geometry, construction, gameplay semantics, and simulation should remain separate from visual presentation wherever practical.

### Recommended reusable style-preset structure

1. **Identity:** name, version, tags, reference games, intended uses.
2. **Shape language:** proportions, silhouette tendencies, facet/edge treatment, irregularity.
3. **Palette grammar:** named color roles, ranges, contrast rules, variation rules.
4. **Materials:** surface response, roughness, texture treatment, stylization rules.
5. **Lighting:** sun, ambient/environment light, shadow character, sky, exposure/tone mapping.
6. **Atmosphere:** fog/aerial perspective, distance desaturation, haze, optional light shafts.
7. **Environment:** vegetation families, density, clustering, groundcover, biome relationships.
8. **Motion:** wind response, ambient animation, secondary motion, variation and synchronization.
9. **Composition:** foreground/midground/background guidance, sightlines, landmark emphasis.
10. **Quality/performance variants:** high, balanced, and lightweight options where useful.
11. **Preview/test scene:** a repeatable scene making the style's distinctive features easy to assess.
12. **Status:** reference-only, prototype, implemented, or visually verified. Never present an unimplemented style as complete.

---

## 4. Suggested validation scene: one environment, multiple style presets

When the Style system is ready for a meaningful visual test, use one small but varied environment containing rolling/sloped terrain, a forest edge and dense grove, foreground trees and understory, a grassy clearing with flowers and groundcover, rocks or a cliff face, a path leading toward a visible landmark, a distant mountain or layered terrain silhouette, one character/scale reference, and a few moving foliage assets.

Render the same authored environment under at least these conditions:

1. **Firewatch-inspired late-afternoon light:** warm directional sun, cooler shade, clear silhouettes, lush layered forest, visible aerial perspective.
2. **Firewatch-inspired hazy distance:** softer contrast and atmospheric blending across distant layers.
3. **Neva-inspired painterly treatment:** illustrated surface language, expressive organic silhouettes, and a mood-led palette.
4. **Neutral baseline:** helps identify which changes come from the style rather than the underlying world.

The goal is not to claim visual parity with either game. Prove that style settings transform presentation coherently while preserving the same world layout, editable objects, and gameplay-relevant structure.

### Visual acceptance checks

- Tree and terrain silhouettes remain readable at near, middle, and far distances.
- The forest has believable layers and intentional openings rather than uniform repeated scattering.
- Lighting gives terrain and foliage form; shadows remain useful rather than crushed or noisy.
- Distant objects recede through atmosphere instead of merely becoming smaller.
- Warm/cool relationships remain coherent across terrain, foliage, sky, and props.
- Vegetation variation looks designed, not like random per-instance color noise.
- The scene retains useful navigation and gameplay visibility.
- The style remains coherent in a still view and during camera movement.
- The same world data remains editable and functional when switching styles.
- Performance is measured before enabling expensive effects across large populations.

---

## 5. Implementation guidance and boundaries

- Treat these as **style-library and visual-quality requirements**, not an instruction to interrupt the current construction completion pass.
- When implementation resumes, inventory existing Style, material, lighting, fog, vegetation, wind, and rendering systems before proposing new infrastructure.
- Prefer extending shared systems and defining reusable data-driven presets over adding special-case logic named after either reference game.
- Do not assume a style can automatically convert arbitrary geometry into painterly artwork; some styles may require suitable assets, geometry, textures, or generators.
- Do not equate high quality with maximum detail. Lighting, silhouettes, palette, composition, and vegetation structure can produce a strong image without expensive geometry everywhere.
- Record which capabilities are implemented, which are only configured by a preset, and which have been visually verified in a running scene.
- Exact imitation of protected assets, characters, maps, or artwork is out of scope. The goal is to learn general art-direction principles and create original Draw3D outputs.

## Summary

**Neva** contributes a reference for painterly environmental art, expressive silhouettes, mood-led palettes, and characterful organic motion. **Firewatch** contributes a particularly valuable target for coordinated lighting, warm/cool color relationships, atmospheric perspective, cinematic composition, and lush, layered forests.

Together they reinforce the broader Draw3D requirement: styles must shape the whole rendered experience—including environment generation, vegetation, lighting, atmosphere, materials, and motion—while remaining reusable and separate from the underlying world and gameplay systems.
