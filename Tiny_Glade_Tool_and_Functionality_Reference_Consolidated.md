# Tiny Glade — Tool and Interaction Reference
**Scope:** Functional inventory of Tiny Glade's world-authoring tools, their controls, contextual behaviors, generated outcomes, and supporting editing systems.

**Reference date:** 19 October 2026  
**Purpose:** Describe Tiny Glade itself. This is not a Draw3D proposal or a feature wish list.

## How to read this document

Tiny Glade has eleven main tools in its traditional toolbar:

1. Add structure
2. Add path
3. Add decor
4. Change appearance
5. Erase
6. Raise ground
7. Add flowers
8. Add trees
9. Add body of water
10. Photo mode
11. Set time

Several functions are contextual rather than separate top-level tools. For example, roof creation is closely tied to building geometry; a window placed low on a wall can become a door; a path crossing water can create stepping stones; and the stairs tool can produce stairs, platforms, ladders, bridges, and walkways according to route geometry and supports.

### Evidence labels

- **Documented:** explicitly described in official developer patch notes or official game material.
- **Guide-documented:** described in a community tool guide or reference. Exact edge cases may vary by game version.
- **Context-dependent:** the outcome depends on geometry, placement, slope, nearby structures, or other conditions.
- **Not fully specified:** the public sources reviewed do not enumerate every handle, numerical limit, or rule. The document does not invent one.

Tiny Glade receives updates, so this document distinguishes a tool's broad behavior from exact keyboard bindings. Inputs can be rebound in settings. Where an interaction depends on a specific default binding, that is stated as a default rather than a universal rule.

---

# 1. Add Structure

The structure tool family creates the main architectural geometry. The creator sketches a footprint or line; the game generates connected walls, surfaces, roof geometry, openings, and supporting details where applicable.

## 1.1 Square / rectangular building

### What it creates
- Rectangular or square building masses.
- Enclosed architectural footprints.
- Buildings that can be combined with walls, paths, windows, doors, stairs, and decorations.
- Flat or raised structures, depending on the available editing operation.

### Basic interaction
1. Select the structure tool and the rectangular building option.
2. Draw or place the building footprint.
3. Adjust the footprint and height using the available on-object editing controls.
4. Let the game generate the enclosing walls and compatible roof form.
5. Select the structure again to reposition or edit it.

### Geometry and editing
- **Footprint:** defines the building's horizontal outline and proportions.
- **Height:** changes the vertical scale of the building.
- **Rotation:** supported structures can be rotated in their editing/focus mode.
- **Elevation:** structures can be raised or lowered where the editing gizmo provides elevation control.
- **Roof adaptation:** roof geometry follows the footprint rather than requiring the player to place each roof piece manually.
- **Openings:** windows and paths can modify walls to produce windows, doors, or passages.
- **Style:** material/color customization can change the building's appearance without replacing its footprint.

### Context-sensitive outcomes
- Window placements on the wall can create openings.
- A window that touches the ground or the bottom of a wall can turn into a door. Holding the disable-snapping key (Left Ctrl by default) prevents this conversion.
- Paths drawn through suitable building geometry can produce a passage or arch.
- Raised structures can create elevated entrances and may interact with stairs or railings.
- Roof and wall geometry must adapt around openings rather than simply placing a flat texture over them.

### Important limits of the public documentation
The public notes do not list every numeric footprint limit, every drag handle, or every combination of footprint and roof geometry. Do not infer exact dimensions from this inventory.

## 1.2 Round building / tower

### What it creates
- Round or cylindrical building forms.
- Towers and circular architectural masses.
- Structures that can be combined with wall runs, roofs, stairs, windows, and decorations.

### Basic interaction
1. Choose the round-building option.
2. Place or draw the round footprint.
3. Adjust its diameter/scale and height using the available editing controls.
4. Add windows, doors, roofs, paths, and stairs as appropriate.

### Geometry and context
- The curved wall surface is treated as a building surface for compatible window and decoration interactions.
- Stairs can attach to buildings and snake around towers; corner/turning sections can automatically produce small platforms.
- A route attached to a tower can therefore create a spiral-like or wrapping circulation path without the player modeling every stair individually.
- Roof and wall styles can be changed using appearance customization.

## 1.3 Freehand wall

### What it creates
- Straight, angled, or curved wall runs defined by a sequence of nodes/control points.
- Enclosure walls, courtyard boundaries, partitions, ruined walls, and defensive walls.

### Drawing and editing
- Click to place successive points/nodes along the wall path.
- Adjust the shape by editing control points.
- Change the wall's height through the available wall controls.
- Rotate freehand walls in focus mode; the September 2026 update added rotation for freehand walls and fences.
- Walls can be snapped to other walls while drawing.
- The wall-snapping preview follows the shape of the wall being targeted; a 2025 patch improved this preview from dashed straight guides to a solid line and a dot following the wall.
- Wall segments can be split or removed to make gaps, openings, and ruins.

### Terrain behavior
- A freehand-wall customization option controls whether the wall follows terrain slope or maintains a constant maximum height. This option was documented in a September 2024 preview update.
- Which mode is appropriate depends on whether the creator wants a wall that follows the ground or a more uniform wall top.
- Zero-length wall/fence curves were a known crash edge case and were fixed in a July 2025 patch; this demonstrates that very short/degenerate segments are a special case, not a recommended drawing outcome.

### Snapping
- Wall-to-wall snapping is supported.
- Snapping indicators communicate the target wall shape.
- Holding the disable-snapping key temporarily suppresses snapping behavior.
- Exact snap distance and every wall endpoint condition are not specified in the public notes.

## 1.4 Fence

### What it creates
- Lower boundary structures for gardens, yards, enclosures, and property edges.
- Freehand fence lines.
- Wooden fence variants using vertical planks in the September 2026 release.

### Drawing and editing
- Draw a fence line through a series of points.
- Adjust its route with control points.
- Snap to a wall while drawing where compatible.
- Rotate a freehand fence in editing/focus mode.
- Change supported appearance/material options.
- Delete or erase unwanted sections.

### Relationship to walls and paths
- Fences can be used as a lighter alternative to a full-height wall.
- A path drawn through suitable wall/fence geometry can create a gate/opening.
- Wall/fence snapping is intended to make connected runs easier to construct.
- The game has special handling for path-plus-wall gates; the September 2026 notes mention making these gates double-sided so they render correctly from inside and outside.

## 1.5 Roof generation and roof editing

Roof generation is part of the structure workflow, not one of the eleven top-level tools.

### General behavior
- Roof geometry is generated to fit a suitable building footprint.
- Roof forms adapt to rectangular and round structures.
- Roofs can be edited as part of the structure's focus/editing interactions.
- Appearance customization can change supported roof colors/materials.
- Flat roofs can be used as surfaces to which stairs can attach.
- Roofs now participate in the game's indirect-lighting acceleration structure, improving indoor lighting behavior (September 2026 update).

### Roof-related contextual interactions
- Stairs can attach to flattened roofs.
- Roof edges and wall openings must be reconciled with the rest of the generated building.
- The game fixes specific geometry edge cases over time; for example, a September 2026 patch fixed stitch bricks not covering gable-roof sides.

### Not fully specified
Public documentation does not enumerate every roof handle or every roof shape's full adjustment range. Avoid assuming that all roof geometries are independently editable in the same way.

## 1.6 Wooden walls and stone foundations

Introduced in the September 2026 update.

### Variants
- Wooden walls with horizontal planks.
- Wooden walls with vertical planks.
- Stone foundations.
- Wooden fences with vertical planks.
- Wooden stairs, selected through the appearance/customization tool.

### Geometry-aware behavior
- Window holes are cut through wooden planks.
- Wooden and brick framing is generated around window openings.
- Foundation height can be adjusted; the developers specifically addressed the case where the stone foundation is raised to a large height.
- Wooden walls use compatible audio and material treatment.
- Wooden stairs can visually match wooden buildings.

These are construction variants, not merely a flat material texture swap: the system must generate the correct plank layout, openings, frames, and foundation transitions.

---

# 2. Add Path

The path tool family creates surface routes and also acts as a contextual editing tool. Drawing across other geometry can create openings, crossings, and transitions.

## 2.1 Ground path / road

### What it creates
- Paths over the ground.
- Winding routes and courtyard paths.
- Paths connecting buildings and outdoor areas.
- Path surfaces that can use different supported appearances.

### Drawing
- Paint/draw a continuous stroke along the intended route.
- Curved routes can be drawn by steering the stroke.
- The path follows the surface/terrain interaction supported by the game.
- The stroke is restricted to the glade's playable area, but a September 2026 update changed boundary handling so strokes can continue visually beyond the glade edge for a short distance instead of ending abruptly.

### Surface variants
- Three cobblestone variants were added with the July 2025 stairs update.
- Five additional floor patterns were added in that update: two stone and three wood patterns.
- Surface appearance can be changed through the customization tool.
- Paths/floors can visually coordinate with stairs and buildings.

### Editing and removal
- Draw over a route only where the tool's normal painting rules permit.
- Use undo to reverse a stroke.
- Hold the disable-snapping key (Left Ctrl by default) to temporarily switch supported raster tools, including paths/cobblestone, into quick eraser mode.
- Use the Erase tool for broader removal.

## 2.2 Path interactions with walls and buildings

Path drawing can alter structures where the stroke intersects suitable geometry.

### Through a building
- A path through a building can create a doorway or passage.
- A wider stroke can create a larger opening or arch, depending on the geometry and stroke.
- This is contextual behavior; it is not equivalent to a universal boolean-cutting tool for every possible surface.

### Through a wall or fence
- A route can create an opening or gate.
- Suitable path-plus-wall interactions can generate an arch/gate form with supporting side geometry.
- The September 2026 update fixed path-plus-wall gates to render double-sided.
- Snapping can help the route meet a wall or fence.

### Across water
- A path drawn through water can generate stepping stones.
- The stones are a contextual result of the path/water interaction rather than individually placed stone assets.
- Water and path strokes can be edited/erased separately according to the tool behavior.

### Door conversion and snapping
- A separate interaction exists in which a low window placement becomes a door.
- Holding Left Ctrl by default disables snapping and prevents windows from turning into doors.
- This is a specific exception to the usual contextual conversion behavior.

## 2.3 Cobblestone

Cobblestone is a path surface variant rather than a separate top-level toolbar tool.

### Behaviors
- Three cobblestone variants are available.
- The path tool can create cobblestone routes.
- The surface can be customized.
- Cobblestone interacts with path strokes and the quick-eraser behavior.
- Paths and cobblestone are designed to meet surrounding geometry and can visually continue past the glade boundary after the September 2026 update.

## 2.4 Stairs / walkways / bridges / platforms / ladders

The stairs tool arrived in the July 2025 update. The developers explicitly designed it to cover stairs, bridges, and walkways.

### Creating a route
1. Select the stairs tool.
2. Click to place individual nodes along the intended route.
3. Continue placing nodes to define direction, curvature, and changes in elevation.
4. Right-click existing stairs (default focus/edit interaction) to open detailed editing mode.
5. Adjust the route and its supported options.
6. Change supported color/material options and railings.
7. Place clutter on top of suitable stairs.

### Route geometry and automatic outcomes
- **Steep route:** can become ladder geometry.
- **Moderate slope:** generates stairs.
- **Nearly level route:** can become a platform or walkway.
- **Elevated route between separated surfaces:** can function as a bridge/walkway.
- **Route around a tower:** the stair path can snake around the tower and automatically generate small platforms around corners.
- **Route onto a building:** stairs can attach to buildings.
- **Route onto a flattened roof:** stairs can attach to the roof.
- **Route along a wall:** stair nodes can attach to walls; attached nodes are indicated in the UI.
- **Unsupported route:** may use a stylized magical levitation effect rather than conventional supports.
- **Supported route:** can be supported by a building, wall, or other supported stairs.

These are geometry-dependent outcomes. The game does not treat the tool as a single fixed staircase mesh.

### Nodes and editing
- The route is node-based: the player clicks to place individual nodes.
- Detailed editing is opened by right-clicking the existing stair structure (default interaction).
- A node attached to a wall displays an attachment indicator/link.
- The player can detach stairs from a wall; a tutorial hint was added for this operation.
- The public update notes establish node-based editing and detailed focus mode but do not document every handle's exact dimensions or hotkey.

### Width and overall shape
- The stairs tool has detailed editing controls and is designed to permit route customization.
- Width/shape adjustments are visible through the tool's editing mode in-game; exact numeric ranges and the full handle-by-handle mapping are not enumerated in the cited patch notes.
- Do not treat any specific width increment or handle axis as a verified numeric specification unless checked in the current game build.

### Elevation and slope
- Node placement defines the route's vertical course.
- The route's slope determines whether generated geometry is stairs, platform/walkway, or ladder.
- Elevation can be used to connect different heights or create a raised walkway.
- The result is regenerated when the route is edited.
- A level route does not automatically imply that every shape will form an arch; bridge/walkway behavior depends on the endpoints, support situation, and route geometry. The documented official description specifically confirms stairs, bridges, walkways, platforms, and ladders, but does not specify a universal automatic “arch” threshold.

### Attachments and snapping
- Stairs can attach to walls.
- Stairs can attach to buildings and flattened roofs.
- The tool can snake around towers.
- Attached wall nodes show a visual indicator.
- The September 2026 update made wall-attached supports check that enough wall exists.
- Supports can be deleted when they are unwanted.
- When all conventional supports are removed, the game can show a magical levitation effect.
- Other buildings, walls, or supported stair sections can act as structural support.

### Materials, railings, and surface details
- Color/material customization is supported.
- Railings can be customized.
- Stone and wooden stair styles are available; wooden stairs were added with wooden building support in September 2026.
- Clutter can be placed on top of stairs.
- Curved railing segments were improved in September 2026 to follow curved stair geometry more accurately.

### Special interaction
- The July 2025 update added a hidden roller-coaster feature involving a cart placed on stairs.
- The advanced variant supports multiple carts.
- The cart has controls to accelerate, slow down, and lean left/right using WASD in the documented secret mode.

### Edge cases fixed in updates
- Platforms over rectangle corners could sometimes be assembled at the wrong angle; this was fixed in September 2026.
- Incorrect ladder meshes on steep terrain near gates were fixed.
- Stair styles could fail to restore after undo when stairs of different styles merged; this was fixed.
- Wall-attached supports now check that sufficient wall is present.
- Rare stair-editing crashes were fixed in September 2026.

## 2.5 Floor surfaces

The game includes floor-pattern variants accessed through customization.

### Documented patterns
- Two stone floor patterns.
- Three wood floor patterns.
- A pre-existing floor pattern that received a visual update.
- Three cobblestone variants for paths.

### Interactions
- Floor patterns can be inherited by snapped shapes; a July 2025 patch fixed a case where the floor pattern was not inherited.
- Flat roofs can receive stair attachments.
- Floor/roof openings can interact with trapdoors.
- Trapdoors do not emit light, so the customization menu does not offer them window-light customization.

---

# 3. Add Decor

Decor includes windows and architectural details such as lanterns, flags, and chimneys. Some decor is placed as an individual object; other detail is generated automatically by the building system.

## 3.1 Windows

### Types
- Square/rectangular windows.
- Arched windows.
- Narrow slit/arrow-slit windows.

### Placement
- Place a window on a suitable wall surface.
- Position and orient it using the available placement controls.
- Window placement is aware of the wall/building geometry.
- Windows can be placed on wooden walls; the construction system cuts holes through planks and generates compatible frames.
- Window placement can trigger automatic contextual details such as flower boxes or nearby clutter.

### Contextual conversion into doors
- Windows touching the ground or bottom of a wall can become doors.
- This interaction was introduced in the September 2024 update.
- Holding the disable-snapping key (Left Ctrl by default) prevents the window-to-door conversion.
- Elevated door/window arrangements can interact with railings and stairs according to the surrounding geometry.
- The official 2024 update notes specifically mention additional “chemistry” such as railings when elevated and stairs when in water.

### Editing and appearance
- Windows can be recolored with the customization tool.
- The Style Clone tool can copy a window's style to another window one by one.
- The Style Clone tool can flood-color all windows on a structure.
- Windows on wooden walls require generated plank cutouts and frame geometry.
- Trapdoors are not offered window-light customization because they do not emit light.

### Related generated details
- Window placement can cause flowers, pots, or other suitable clutter to appear nearby.
- The exact set of generated clutter depends on the available content and mods.

## 3.2 Lanterns

### Placement
- Place lanterns on suitable building or environmental surfaces.
- Adjust their position using the object's focus/editing mode.
- Change their supported color through customization.

### Lighting behavior
- Lanterns contribute local light.
- Since the September 2026 renderer update, lanterns illuminate fog through the updated participating-media/froxel rendering system.
- Lighting and fog effects therefore interact; the lantern is not only a decorative mesh.

## 3.3 Flags

### Placement and customization
- Place flags on suitable structures.
- Reposition them in focus/edit mode.
- Change supported colors/appearance.
- Flags can be used as repeated accents across buildings and stairs.

### Contextual grouping
- Closely related flags can form connected decorative bunting in suitable arrangements.
- Flags can be placed on stairs where placement is supported.
- The September 2026 update corrected flag audio for flags on stairs.

## 3.4 Chimneys

### Placement
- Place chimneys on compatible roof/building locations.
- Adjust their position in the supported editing mode.
- Change supported appearance through customization.

### Context
- Chimneys are generated/placed in relation to the building surface rather than modeled as a separate full building.
- Their fit depends on the roof/building geometry.
- Public patch notes do not enumerate every chimney-specific handle or variant.

## 3.5 Trapdoors and surface openings

- Suitable openings on flat surfaces can become trapdoors.
- Trapdoors interact with floor/roof geometry.
- Trapdoors have distinct audio for wooden and stone floors (September 2026 update).
- Trapdoors do not emit light and do not expose window-light customization.
- The available sources do not establish that every roof/floor surface supports every trapdoor arrangement.

## 3.6 Clutter

Clutter includes small decorative objects that may be generated around windows or placed/moved by the player.

### Automatic placement
- Clutter can appear around windows and other appropriate architectural areas.
- The generated set depends on the game's clutter assets and installed clutter mods.
- Clutter helps make a building look occupied without requiring the player to place every small object.

### Clutter focus mode
- The player can enter focus mode for a clutter item and move/customize it.
- The available clutter can be cycled.
- A dedicated clutter picker UI was added in September 2026.
- Clutter can be placed on stairs.
- The September 2026 Workshop update allows mods to add new clutter items.

### Limitations documented by developers
- The clutter picker did not initially include search/filter controls.
- Clutter items cannot be placed on top of other clutter items as a general supported behavior.
- A modded clutter item can affect existing glades if the mod author updates or removes it.

---

# 4. Change Appearance / Customization

The customization tool changes the visual style of existing geometry. It is separate from the tool that initially creates the object.

## 4.1 Color customization

- Recolor supported buildings.
- Recolor walls and fences.
- Recolor stairs.
- Recolor windows.
- Recolor flags and lanterns where supported.
- Change supported roof/floor appearances.
- Use the contextual color menu to expose options relevant to the selected object.
- The color menu's options depend on which part of the object was selected.

The September 2026 update improved the color menu so that its secondary ring remains aligned with the primary ring, and only displays special color options when the selected segment is affected.

## 4.2 Construction/material style

### Supported examples
- Stone/brick-like structures.
- Wooden walls with horizontal planks.
- Wooden walls with vertical planks.
- Stone foundations.
- Wooden fences with vertical planks.
- Wooden stairs.
- Different floor patterns and cobblestone variants.

### Geometry-aware material behavior
- Wooden walls require plank cutouts around windows.
- Window frames adapt to wooden and brick construction.
- Foundation geometry responds to foundation height.
- Stair material can be changed to match the wooden building style.
- Surface changes may affect audio and rendering details as well as color.

## 4.3 Plaster wear

- A customization control adjusts the amount of plaster peeling.
- The option was added in the July 2025 stairs update.
- It changes the worn/aged appearance of supported plaster surfaces.
- This is a surface-condition control, not a geometry-placement tool.

## 4.4 Floor and path patterns

- Select among supported stone and wood floor patterns.
- Choose among three cobblestone variants.
- Pattern inheritance for snapped shapes is supported; a patch fixed an inheritance bug.
- Pattern choice affects the visual surface rather than the route's core path.

## 4.5 Style Clone

Added in September 2026.

### Source types
- Buildings.
- Walls.
- Stairs.
- Fences.
- Windows.

### Operation
1. Choose Style Clone under the customization category.
2. Select an object to copy style from.
3. Select the target object.
4. Transfer the compatible style.
5. For windows, either recolor individual target windows or flood-color all windows on a structure.

### Scope
- Copies supported appearance/style data, not the source object's full geometry or location.
- Compatible target types matter; the public notes list the supported source/target families but do not enumerate every property transferred by every pair.
- Style clone is a consistency tool for reusing appearance across existing objects.

---

# 5. Erase

The Erase tool removes parts of the world. It is distinct from the quick-eraser shortcut available while painting certain raster tools.

## 5.1 Erasing structural objects

- Remove buildings and other supported selected objects.
- Remove walls or wall sections.
- Remove fences or fence sections.
- Remove decorations.
- Remove paths.
- Remove water.
- Remove trees.
- Remove painted vegetation.

## 5.2 Erasing walls and fences

- Erase selected wall/fence geometry to make gaps.
- Create broken or ruined-looking structures by removing sections.
- Remove individual segments rather than always deleting the whole composition.
- Wall/fence control-point editing and erasure are separate operations.

## 5.3 Erasing paths, plants, and water

- Use the appropriate erase tool to remove strokes/painted areas.
- The quick-eraser behavior can temporarily turn the active path/cobblestone/plant/water painting tool into an eraser.
- Default modifier: Left Ctrl, the disable-snapping key.
- This avoids switching toolbar tools for a small correction.
- Undo is an alternative when the entire last operation should be reversed.

## 5.4 Selected-object deletion

- Objects with focus mode can be deleted using the Delete key, which is configurable.
- The September 2026 update added this as a general editing shortcut.
- The default focus interaction is right-click, but controls can be rebound.

---

# 6. Raise Ground

The Raise Ground tool changes terrain elevation.

## 6.1 Painting terrain

- Select the raise-ground tool.
- Paint over an area of terrain to raise it.
- Shape hills and elevated areas.
- Adjust the resulting terrain using the available tool controls.
- Use the terrain shape as a base for buildings, paths, trees, flowers, and water.

## 6.2 Terrain as a context for other tools

- Paths can be drawn over terrain and respond to surface height.
- Stair routes can connect different elevations.
- Water can occupy low-lying areas.
- Trees and flowers can be placed across the resulting terrain.
- Building placement can be combined with raised ground.
- Terrain can be shaped around the edges of ponds and structures.

## 6.3 Public documentation limits

The sources reviewed confirm the Raise Ground tool and its role in terrain editing, but do not provide a complete public enumeration of its brush-radius handles, falloff modes, numerical elevation limits, or all terrain blending rules. Those details should be checked in the current game UI rather than guessed.

---

# 7. Add Flowers / Meadow Plants

The flower tool paints vegetation across areas rather than requiring each plant to be placed individually.

## 7.1 Flower painting

- Paint colorful flowers in a meadow or garden region.
- Create broad patches and borders.
- Build up the visual density of a planted area.
- Combine flowers with grass, trees, paths, buildings, and water.

## 7.2 Grain plants

The September 2026 update expanded meadow plants to include grain plants.

- Paint grain plants into meadow areas.
- Use them as a different vegetation type from colorful flowers.
- Mix different plant types to create more varied ground cover.

## 7.3 Density and repeated painting

- Painting over the same plant can make it bushier/denser.
- Repeated strokes can build up a patch rather than replacing the whole patch with a single object.
- The game distributes the resulting plant geometry across the painted region.
- Grass rendering was also improved in the September 2026 update.

## 7.4 Quick eraser

- Hold Left Ctrl by default while painting plants to temporarily erase the painted vegetation.
- This behavior is part of the quick eraser for raster tools.
- It allows small corrections without switching the toolbar to Erase.

## 7.5 Performance and rendering context

The developers described a new vegetation authoring/rendering system in the September 2026 update, introduced because the old grass system did not scale to larger glades. The new system supports more types of plants and a more detailed grass appearance.

---

# 8. Add Trees

The tree tool places trees as environmental objects.

## 8.1 Placement

- Select the tree tool.
- Place trees in suitable locations.
- Use repeated placements to populate the environment.
- Combine trees with paths, buildings, water, and painted flowers.
- Use the tree eraser to remove unwanted trees.

## 8.2 Terrain interaction

- Trees can be placed in environments with different terrain elevations.
- A December 2024 patch fixed trees hovering above the ground when grabbed from elevated terrain.
- This indicates that tree placement/movement needs to account for terrain height.

## 8.3 Editing

- Grab/move trees in supported object editing mode.
- Delete or erase trees.
- Use trees as repeated composition elements in the scene.

## 8.4 Limits of the documented tool

The sources reviewed do not establish a complete built-in control set for selecting arbitrary species, setting exact spacing/density, editing growth stages, or assigning a numeric population distribution. Those capabilities should not be assumed merely because other vegetation tools offer painting.

---

# 9. Add Body of Water

The water tool paints bodies of water and water channels.

## 9.1 Painting water

- Paint a pond or pool.
- Draw water channels through suitable terrain.
- Adjust the outline through the painting interaction.
- Combine water with terrain shaping, vegetation, paths, and structures.

## 9.2 Water level and terrain

- Water occupies a surface relative to the surrounding terrain.
- Low terrain and raised banks can frame the water body.
- Water can be combined with terrain sculpting to create pond basins and channels.
- The public notes reviewed do not enumerate all water-depth handles or numeric level ranges.

## 9.3 Path/water interaction

- Draw a path through water to create stepping stones.
- Use this behavior to create crossings or a route across a shallow water feature.
- The stones are generated contextually from the path-water interaction.

## 9.4 Water erasing

- Erase painted water with the Erase tool.
- Hold Left Ctrl by default while painting water to temporarily erase the active water stroke.
- The September 2026 update fixed black water rendering in cases where no water bed was present.

## 9.5 Water vegetation and ambient life

- The environment can include aquatic details such as lily-pad-like vegetation.
- Ducks are present as ambient environmental life.
- The September 2026 update improved duck ripple rendering.
- These are environmental details, not a general animal-placement or behavior-authoring tool.

---

# 10. Photo Mode

Photo mode is a presentation/camera system rather than a world-construction tool.

## 10.1 Camera presentation

- Switch into a presentation-oriented view.
- Reposition and frame the scene.
- Zoom and change viewing angle using the camera controls.
- Inspect the world from different viewpoints.
- Hide or reduce editing UI for screenshots where supported.

## 10.2 Image controls

Community guides document controls such as:
- Tilt-shift/depth-of-field presentation.
- Image filters.
- Brightness.
- Contrast.
- Saturation.
- Camera framing and composition.

The exact controls available may depend on the current build. The official sources reviewed do not enumerate every slider or numerical range, so this section does not claim exact values.

## 10.3 Camera modes and inspection

- Camera navigation allows the creator to inspect the composition from different angles.
- The September 2026 update fixed a camera-rotation issue when the camera was high up.
- The game also has a first-person ghost/noclip mode, mentioned in the July 2025 patch notes; it is a navigation mode, not a separate world-authoring tool.

---

# 11. Set Time / Environment

The time tool controls the world-wide lighting and atmospheric presentation.

## 11.1 Time of day

- Change the time of day.
- Change the sun/light direction and the resulting scene lighting.
- Keep time fixed or allow it to progress over time.
- Use time-of-day changes to preview the same architecture under different lighting.

## 11.2 Fog

Added as a controllable feature in September 2026.

- Adjust fog density.
- Adjust fog height.
- Adjust how closely fog clings to the ground.
- Lock the selected fog settings or allow the environment to change them over time.
- Local lantern lighting can illuminate fog using the updated renderer.

## 11.3 Snow

- In winter glades, adjust the snow control.
- Snow is a seasonal environment control, not a general terrain-sculpting brush.
- Its availability is tied to winter glades.

## 11.4 Lighting and environment interaction

- Time of day affects the appearance of all structures and vegetation.
- Fog affects depth and atmospheric visibility.
- Lanterns illuminate fog.
- Snow changes the winter scene.
- Photo mode can be used to present the resulting environment.

---

# 12. Shared editing and interaction systems

These systems affect multiple tools and should be understood alongside the eleven top-level tools.

## 12.1 Focus/edit mode

- Right-click is the default focus/edit interaction for many objects.
- Focus mode exposes object-specific controls and gizmos.
- Stairs enter detailed editing mode by right-clicking the existing stair structure.
- Selected objects can be manipulated through their available handles.
- The September 2026 update fades unnecessary gizmos when they would obscure small shapes or wall sections.
- Objects with focus mode can be deleted using the configurable Delete key.

## 12.2 Snapping

- Snapping helps align supported objects and construction strokes.
- Walls can snap to existing walls.
- Stairs can attach to walls, buildings, and flattened roofs.
- The stair UI shows an indicator when a node is attached to a wall.
- Holding Left Ctrl by default disables snapping.
- The same modifier prevents window-to-door conversion.
- The same modifier temporarily turns supported painting tools into erasers.
- Exact snap radius and all snapping priorities are not enumerated in the public patch notes.

## 12.3 Group selection and transformations

Added in September 2026.

### Selection
- Right-click one building.
- Hold Left Ctrl and right-click additional buildings/walls/fences to add them to the group, using the documented default controls.

### Group operations
- Move selected buildings, walls, and fences together.
- Rotate the group.
- Elevate/lower the group.
- Bookmark the group so it can be selected again without repeating the entire selection process.

### Individual freehand-wall/fence rotation
- Freehand walls and fences can be rotated in focus mode.
- This was added alongside group rotation.

## 12.4 Style reuse

- Style Clone copies compatible appearance from buildings, walls, stairs, fences, and windows.
- For windows, the tool supports applying a color individually or flood-coloring all windows on a structure.
- This is separate from group transforms: it copies style, not spatial position.

## 12.5 Clutter picker

- Opens a UI to select available clutter.
- Supports easier selection when many clutter items are installed.
- Can cycle through available clutter.
- Mods can add clutter items through Workshop support.
- Search/filter was not available in the initial September 2026 picker implementation.

## 12.6 Quick toolbar

- Press Spacebar or the mouse Back button by default to open the compact toolbar around the cursor.
- Release the key/button to choose the highlighted tool.
- The binding can be changed in settings.
- The goal is to reduce cursor travel to the main toolbar.

## 12.7 Input hints and remapping

- Context-sensitive input hints can be shown on the left side of the screen.
- Hints depend on the current tool or highlighted object.
- They can be disabled in settings.
- Key and mouse-button bindings are configurable.
- The September 2026 input system permits an action to be bound to a key, a mouse button, or both.

## 12.8 Undo, autosave, and backups

- Undo/redo supports correcting construction operations.
- Background autosaving and backups were introduced with the July 2025 stairs update.
- A UI for loading game backups was added.
- The September 2026 update added a button to open a saved glade's folder and UI for importing beta saves.
- Saves are automatically upgraded when loaded by newer versions, according to the July 2025 update notes.
- The same notes warn that older game versions may not load saves that have been upgraded by newer versions; backups are therefore important.

## 12.9 World boundary behavior

- The glade has a boundary that restricts the effective world area.
- Strokes such as path and water can continue beyond the edge visually after the September 2026 update, but their effects remain restricted to the playable area.
- Path/cobblestone geometry may continue slightly beyond the boundary for a more natural edge.

## 12.10 Workshop and mod support

Added in September 2026.

- Workshop mods can add new clutter items.
- Supported asset-replacement mods can be shared through Workshop.
- Clutter mods load automatically.
- Other glade mods must be enabled per glade in the Mods menu.
- This is not an unrestricted scripting/plugin system: the developers explicitly state that Workshop does not permit arbitrary new doodling tools or wholly new gameplay modes.
- The developers note that subscribed mods consume memory even when not used in a given glade, and that removing/updating a modded item can affect glades using it.

---

# 13. Cross-tool interaction matrix

| Action / source tool | Target or condition | Documented/contextual result |
|---|---|---|
| Draw path | Ordinary terrain | Creates a ground path/road. |
| Draw path | Through suitable building geometry | Can create a doorway or passage. |
| Draw path | Wide passage through suitable building geometry | Can create a larger opening/arch, depending on geometry. |
| Draw path | Through suitable wall/fence | Can create an opening or gate. |
| Draw path | Across water | Can generate stepping stones. |
| Place window | Low on wall / touches ground | Can turn into a door. |
| Place window | Hold Left Ctrl (default) | Prevents window-to-door conversion by disabling snapping. |
| Draw stairs | Steep route | Can generate ladder geometry. |
| Draw stairs | Sloped route | Generates stairs. |
| Draw stairs | Nearly level route | Can generate a platform/walkway. |
| Draw stairs | Between elevated surfaces | Can function as a bridge/walkway. |
| Draw stairs | Around a tower | Can snake around the tower and create corner platforms. |
| Draw stairs | Near/onto a building | Can attach to the building. |
| Draw stairs | Near/onto flattened roof | Can attach to the roof. |
| Draw stairs | Node snaps to wall | Node is shown as wall-attached; it can be detached. |
| Remove stair supports | No remaining conventional support | Stylized levitation effect can appear. |
| Place stairs | On a wooden building | Wooden stair style can match the construction. |
| Change building style | Wooden wall | Planks, foundations, and window framing use compatible geometry. |
| Place window | Wooden wall | Planks are cut around the opening and compatible framing is generated. |
| Paint flowers/grain | Paint over same plant | Plant becomes bushier/denser. |
| Paint path/cobblestone/plants/water | Hold Left Ctrl (default) | Temporarily erases using the active raster tool. |
| Select multiple buildings/walls/fences | Group selection | Move, rotate, or elevate as a group. |
| Use Style Clone | Compatible structure | Copies supported appearance/style. |
| Use Style Clone on window | Other windows | Recolor one at a time or flood-color all windows on a structure. |
| Set time/fog | Lanterns in fog | Lanterns illuminate fog in the updated renderer. |
| Paint stroke | Across glade boundary | Effective changes stay restricted; some stroke geometry can continue visually beyond the edge. |

---

# 14. Source references

The descriptions above are based primarily on official developer patch notes reproduced by SteamDB, with community guides used for tool names and general interactions.

1. **Tiny Glade Wiki — Tools**  
   https://tinyglade.wiki.gg/wiki/Tools  
   Lists the eleven top-level tools.

2. **Tiny Glade Wiki — Building menu**  
   https://tinyglade.wiki.gg/wiki/Building_menu  
   Describes the building and decoration groups.

3. **Official Tiny Glade Steam page**  
   https://store.steampowered.com/app/2198150/Tiny_Glade/  
   General description of the game and its construction approach.

4. **29 July 2025 — “The stairs update is finally here!”**  
   https://steamdb.info/patchnotes/19397164/  
   Official update notes covering node-based stairs, platforms/ladders, attachments to walls/buildings/flattened roofs, corner platforms around towers, supports, customization, clutter on stairs, roller-coaster mode, cobblestone variants, floor patterns, autosave/backups, and related UX changes.

5. **31 July 2025 — Patch 1.14.1a**  
   https://steamdb.info/patchnotes/19428041/  
   Covers wall snapping indicators, stair-node wall attachment UI, and related fixes.

6. **23 September 2026 — “Wood, Flowers, Workshop & more”**  
   https://steamdb.info/patchnotes/25477972/  
   Official update notes covering wooden walls, stone foundations, wooden stairs, meadow flowers and grain plants, controllable fog, snow controls, Workshop support, clutter picker, group transforms, freehand wall/fence rotation, Style Clone, quick toolbar, quick eraser, configurable input, deletion, and other editing changes.

7. **24 September 2026 — Patch 1.16.1**  
   https://steamdb.info/patchnotes/25506810/  
   Includes stair-editing fixes and rendering/compatibility fixes.

8. **25 September 2026 — Patch 1.16.2(a)**  
   https://steamdb.info/patchnotes/25529800/  
   Includes a fix for the interaction between disable-snapping and group selection.

9. **27 June 2025 — Beta stairs/cobblestone/floors notes**  
   https://steamdb.info/patchnotes/19024915/  
   Earlier beta description of node-based stairs, detailed editing, platforms/ladders, wall/roof attachments, and floor variants.

10. **Switchblade Gaming — Tiny Glade beginner's guide**  
    https://www.switchbladegaming.com/cozy-games/tiny-glade-guide/  
    Secondary guide for general tool behavior. Community-guide details should be treated as lower confidence than explicit official patch notes.

---

## 15. Completeness and verification notes

This document catalogs the eleven main tools and the major contextual editing systems described in the sources reviewed. It is not a reverse-engineered specification of the game's internal code.

The following details are not exhaustively specified in the public sources used here:
- Exact numeric limits for each tool.
- Every gizmo/handle and its exact axis or sensitivity.
- All snapping distances, priorities, and tie-breaking rules.
- Every possible stair width, elevation, and slope threshold.
- Every roof geometry mode and its full control set.
- All terrain-brush radius/falloff settings.
- All water-depth and shoreline parameters.
- Every clutter asset and every generated-detail rule.
- Every combination of structural opening, window, path, stair, and terrain interaction.

Where those details matter, inspect the current in-game UI and record the precise behavior with screenshots or a reproducible test. They are deliberately not guessed in this reference.

---

# 16. Community-Guide Discoveries: Advanced Interactions and Construction Tricks

**Purpose:** This supplement consolidates additional interactions observed in two player-guide transcripts, including details not fully specified in the core tool inventory. It is an extension to the reference, not a claim that every trick is guaranteed in every game version. Items below are marked **Guide-reported** when they come from demonstrations rather than official specifications. Geometry thresholds, exact controls, and version-dependent behavior should be verified in the current build before treating them as guaranteed.

## 16.1 Building shapes, doors, and openings

- **Guide-reported — Grouped windows and doors:** Up to three adjacent windows may visually combine into a wider window arrangement; up to three adjacent doors may similarly combine into a wider doorway. Exact grouping depends on placement and snapping.
- **Guide-reported — Widening a door into an arch:** A path/opening interaction can create a large doorway. After deselecting and hovering/focusing the resulting opening, handles can reportedly raise it and widen it. Widening far enough may convert the doorway into an arch rather than a rectangular opening.
- **Guide-reported — Building placed across water:** Placing a building shape across a water channel can generate an archway with gate-like details, creating a water passage beneath the structure.
- **Guide-reported — Wall drawing direction affects windows:** Windows placed on freehand walls may appear on the wall's exterior side according to the direction in which the wall was drawn. If the opening or window appears on the wrong side, test reversing the wall's start/end direction.
- **Guide-reported — Wall style and wear:** The wall tool's appearance/paint options can reportedly switch between a regular/straight wall and a ruined look, as well as vary the amount of visible wear. Windows placed on freehand walls may look like open cut-outs and differ from windows placed on building walls.
- **Guide-reported — Perfect circular wall/barrier:** Place a cylinder as a circular guide, draw a wall around it, then use the smallest wall-eraser setting and target the highlighted wall/roof area to remove the roof and retain a circular wall-like shell. The bottom can then be adjusted/closed. This is a demonstrated workaround, not a documented dedicated circular-wall tool.
- **Guide-reported — Curved shell from a building:** Erasing or cutting parts of a round building can leave a curved wall-like shell. The edited result may no longer behave like a normal snapped building.
- **Guide-reported — Stacked round-building supports:** Overlapping or resizing stacked round structures may make some supports disappear or allow selective control over them. Treat this as an emergent geometry trick, not a stable construction rule.

## 16.2 Fence, wall, and path combinations

- **Guide-reported — Fence-to-arch workflow:** Draw a path beneath a fence to alter its opening/shape, then adjust the fence height and cut it into sections. This can create repeated tall arch-like fence forms more efficiently than building each separately. Width and height thresholds are not specified.
- **Guide-reported — Fence-derived bridge or stair-like pieces:** Fence beams and path placement can be combined to create small bridge-like or stair-like structures.
- **Guide-reported — Partial fence supports:** Carefully placing a fence at three points and routing a path beneath only part of it may create a partial support. The guide describes this as finicky and position-dependent.
- **Guide-reported — Wall/path opening differences:** A path interacting with a stone wall versus a fence can produce different arch/opening results. Wall-top settings, such as flat tops versus railings, may alter the geometry around an opening. Different wall modes may also vary in whether the top follows terrain or stays level.
- **Guide-reported — Path through a building boundary:** A path overlapping a building can change an opening from a door-like form to an arch or passage.
- **Guide-reported — Water and wall details:** The water tool may carve toward a local low point, potentially producing holes when used on raised terrain. A path placed across water may create stepping stones or rocks; highlighting/adjusting the path can make them protrude enough to act as a crossing. Path placement near submerged walls has reportedly produced grate-like details.
- **Guide-reported — Stone arches as reusable shapes:** Stone arch pieces can be layered or reused visually as furniture, throne arms, garden edging, and similar decorative structures.

## 16.3 Stairs, ladders, and floating structures

- **Guide-reported — Converting stairs to a ladder:** Focus/edit a stair structure, use the right-click interactions to select its endpoint/handle, and move the relevant point inward; the route may convert into ladder geometry. Exact input sequence can vary with the current editing state.
- **Guide-reported — Stair shape and finish:** The guide demonstrates changing railing/border options from full railings to lower borders or flat, borderless stairs. A handle can expand or contract the overall stair route. The route's circular/continuation handle can be used to extend the path, and the result can be edited into a loop-like shape.
- **Guide-reported — Removing a stair base:** Using the hammer/removal interaction on the lower stair section can remove that base while leaving the upper stairs apparently floating.
- **Guide-reported — Floating cylinder platform:** Place a cylinder on the ground, raise it to the desired height, adjust its roof and make its floor very thin, then place another same-size cylinder directly on top. The shared/intersecting floor may disappear. Expanding the shapes can cause floors to return, so this is a geometry-dependent workaround; the guide reports that squares do not behave the same way.
- **Guide-reported — Raised stairs as display surfaces:** Stair/platform geometry can be expanded or arranged to provide surfaces for lanterns, flowers, or other small decorative objects where direct placement on a bench or wall is unavailable.

## 16.4 Windows, doors, clutter, and small props

- **Guide-reported — Window/door decorations:** Doors may cycle through decorations such as a bell, wreath, or no decoration. Some window types may cycle through flower boxes, shutters, or hanging fabric; not every window type necessarily supports every option.
- **Guide-reported — Clothesline effect:** Hanging fabric on nearby windows can visually connect into a clothesline-like arrangement.
- **Guide-reported — Stained-glass effect:** Colored square blocks placed behind a window pane may create a stained-glass-like appearance, especially at night. This is an improvised visual effect rather than a dedicated stained-glass setting.
- **Guide-reported — Clutter detaching and duplication:** When clutter appears attached to a window or building, right-clicking it may detach it so it can be moved. The clutter can then be duplicated and cycled through available object variants using the visible picker/arrows. This allows contextual clutter to be reused as individually arranged props.
- **Guide-reported — Chimney beneath a window:** Placing a chimney under a window may produce a narrower metal chimney variant, even with a small window. The guide uses this behavior to suggest an improvised fire-pit-like object; exact placement compatibility should be tested.
- **Guide-reported — Lantern placement:** A lantern placed directly on the ground may appear without a pole, unlike a lantern mounted on a structure. Lantern brightness reportedly has several settings, and lantern light color can be changed.
- **Guide-reported — Flags:** Flags can extend outward from their attachment point; their color and emblem/design can be cycled separately.
- **Guide-reported — Flower brush behavior:** Holding the flower tool may place smaller flowers, while a subsequent click or alternate interaction can switch to larger bushes. Confirm the exact click/hold behavior in the current version.
- **Guide-reported — Decorative details in water:** Flowers can be placed in water, and paths can add stones/rocks to water. These can be combined to decorate ponds or create a stepping-stone crossing.

## 16.5 Terrain, plants, and seasonal changes

- **Guide-reported — Terrain brush direction:** The ground tool is described as behaving somewhat like a sculpting brush: dragging inward/outward changes the ground profile and can also lower it. The transcript does not establish precise brush modes, falloff, or numeric settings.
- **Guide-reported — Season switching:** To change a glade's season, the guide returns to the main menu, loads the glade being edited, then uses a leaf icon to choose a season. Seasonal changes affect snow, trees, and flowers. This is a workflow reported by the guide, not a full specification of season controls.
- **Guide-reported — Preserve selected clutter between glades/themes:** Switching to a theme/glade with different natural clutter (the guide names the Olden theme as a source of rocks and logs) while locking selected clutter may preserve those selected pieces. Confirm persistence behavior before relying on it in a long build.

## 16.6 Color customization and material detail

- **Guide-reported — Separate color channels:** The guide demonstrates changing basic wall color, wooden outline color, fence color, window border/frame color, and interior/window light color independently. It also demonstrates separate lantern-light and flag color/design customization.
- **Guide-reported — Colorful material combinations:** Wall and wood-outline colors can be deliberately contrasted (for example, bright walls with dark outlines). This is a visual customization capability rather than a preset limitation.
- **Guide-reported — Stair sparkles:** Changing a stair color may also change the color of its surrounding sparkle/levitation effect.

## 16.7 Photo mode, camera, and presentation

- **Guide-reported — Multiple camera viewpoints:** Camera mode includes ground-level walking, elevated/flying views, free camera movement, and a top-down/aerial view. A central square/region can be resized to focus attention on an object or area.
- **Guide-reported — Camera effects:** The guide demonstrates a tree-peeking/occlusion view, a mode that presents buildings more like 3D models, and blur/painting-like effects. The exact names and purpose of every mode are unclear from the transcript.
- **Guide-reported — Lighting and celestial composition:** A sun/time control changes color and intensity; time of day can be adjusted, and the sun/moon direction can be moved to frame the moon in a screenshot. A vignette-like focus effect and color filters are also mentioned.
- **Guide-reported — Clean screenshots:** A settings/UI option reportedly controls whether the game logo appears in screenshots. Hide or disable the logo/UI when a clean image is needed; verify the current menu label.
- **Guide-reported — Rewind/time-lapse workflow:** Undoing and redoing a completed construction can create a simple time-lapse presentation, but new edits may overwrite redo history. Treat this as a capture trick, not a safe substitute for saving a copy.
- **Guide-reported — Toolbar and keyboard shortcuts:** The toolbar may be repositioned, and hotkeys reportedly cycle tool options or hide UI. Exact bindings should be checked in the current settings/input hints.

## 16.8 Additional environment and presentation interactions from the earlier guide

- **Guide-reported — Lantern brightness levels:** Lanterns may offer dim/low/bright settings in addition to color customization.
- **Guide-reported — Photo filters:** Color filters and controls for saturation, contrast, and brightness are reported. These are presentation controls and should not be confused with changes to the authored world's underlying materials.
- **Guide-reported — Moon placement:** The sun/moon direction control can be used to move the moon into or out of a chosen camera composition.
- **Guide-reported — Theme-specific clutter:** Some glades/themes appear to supply distinct loose clutter, including rocks and logs. Locking selected items before switching themes may preserve them, but this needs in-version verification.

## 16.9 Verification checklist for these discoveries

For each guide-reported behavior, record **Proven**, **Partially proven**, **Not reproduced**, or **Not tested** when checking the current game version. Capture the tool selected, input sequence, shape/material used, and resulting geometry. Prioritize tests that unlock reusable construction patterns; do not treat every exploit-like combination as a requirement for complete parity.

Recommended first verification pass:
1. Test three-part window/door grouping and door widening into an arch.
2. Test wall drawing direction, wall wear settings, and circular-wall workaround.
3. Test fence/path arching, cutting, and partial supports.
4. Test cylinder stacking for a floating platform and stairs-to-ladder conversion.
5. Test clutter detach/duplicate/cycle, chimney-under-window, and floor lantern placement.
6. Test flowers and stepping stones in water, plus building-over-water archway.
7. Test season switching and whether locked clutter survives theme/glade changes.
8. Test separate color channels, flag designs, stair sparkle colors, and lantern brightness.
9. Test camera modes, moon/sun positioning, screenshot logo settings, and undo/redo time-lapse workflow.

