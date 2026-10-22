# Personal Tiny Glade Feedback — Draw3D

This document records my personal observations from playing Tiny Glade. The intention is to share what I have learned from its user experience and building tools so that these findings can be taken into consideration as Draw3D continues to develop.

This is feedback to enrich the direction we are already taking, not a request to change course or start over. The recent progress with modular building, quick customization, and context-sensitive placement is finally moving in the right direction. I want to build on that work.

Please keep this document in the project files and refer to the relevant sections when working on construction, terrain, vegetation, decoration, and other related systems. Use these observations as guidance where they fit naturally into the existing architecture and implementation. Continue building on what already works rather than duplicating features or introducing unnecessary complexity.

## 1. Modular building and quick customization

One of Tiny Glade's strongest features is how quickly you can create and customize a building without navigating lots of menus.

- You can place a basic house and drag handles along its walls to extend its length and width.
- Dragging around the corners allows you to rotate the building.
- A handle where the walls meet the roof allows you to adjust the building's height.
- Right-clicking a building reveals its available controls together, including movement, rotation, dimensions, and roof adjustments.
- Roofs can be raised or flattened by dragging up and down. Dragging left and right changes the roof's shape, including convex and concave curves.
- Roof orientation can be changed to alter which side faces the front of the building.
- A slider can adjust the proportions of different materials, such as wood and brick, with immediate visual feedback.

The important thing is that a basic structure can be placed quickly and then refined directly in the world. The user should not have to construct every wall and roof manually to get started.

## 2. Context-sensitive placement

Tiny Glade frequently changes what a tool creates depending on where you use it.

- Placing a lantern on a wall creates a wall-mounted light, while placing it on the ground creates a post lantern.
- Using a window tool near the bottom of a wall creates a door.
- Placing windows next to one another can automatically join them into larger, more elaborate window arrangements.
- Repeated placement can produce different sizes or variations of windows, including decorative details.
- Placing paths through walls transforms openings into archways, sometimes with supporting posts.
- Paths adapt to terrain: on slopes they can create rocky steps, while paths through water can form rocky crossings.
- Stairs and other structures recognize nearby walls and elevations and adapt their placement accordingly.
- Towers can be placed against buildings and fit into the structure naturally.

This is particularly useful because the user expresses an intention through placement, and the engine works out an appropriate result. The tool does not always need to produce the same object regardless of context.

## 3. Combining and editing structures

Buildings and other objects can be modified after placement without losing the convenience of modular construction.

- Buildings can be resized, rotated, moved, and adjusted through direct manipulation.
- Multiple compatible placements can join seamlessly, such as connected windows, banners, paths, trees, and terrain platforms.
- Separate sections can sometimes be created deliberately, including by holding Control when drawing additional terrain platforms.
- Fences can be edited using control points, curved sections, and posts. Sections can be detached and later snapped back into place.
- Stairs can be extended, curved, attached to walls, adjusted for elevation, and edited using intermediate control points.
- Right-clicking can finish a drawing operation or reveal additional controls, depending on the current tool. It can also deselect objects when clicking on the ground.

The key observation is that connected objects should feel like a coherent structure while remaining individually editable where needed. Joining, separating, snapping, and adjusting should be natural parts of the same workflow.

## 4. Windows, doors, roofs, and architectural details

The building tools provide a useful range of details without requiring the user to model everything individually.

- Different window shapes support repeated placement, with adjacent windows sometimes combining into larger arrangements.
- Windows can have sills, plants, decorative coverings, and other details.
- Some windows can be placed at wall corners to create built-in openings.
- Doors can be placed through the window tool and can form wider arrangements through repeated placement.
- Chimneys can be positioned on roofs or walls, moved after placement, resized, and potentially rotated.
- Chimneys can emit smoke, which is a good example of how environmental effects can make a building feel alive.
- Roofs support changes in height, curvature, shape, and orientation.
- Towers provide similar direct controls for dimensions, material proportions, rotation, and placement.

These are examples of how a small number of intuitive tools can produce a wide variety of architectural results.

## 5. Materials, colours, and appearance

Quick customization is another area I particularly like.

The paintbrush tool allows you to select an object and choose colours, materials, and textures. Walls can have different appearances, including vertical wooden boards, brick, mixed materials, and scaffolding-like finishes. Roofs can be recoloured as well.

Colour palettes provide a quick visual preview as you move across the available colours. Different parts of an object, such as the wall and its frame, can have their own colour choices.

There is also a bucket-style tool that copies the appearance or style of one building and applies it to another.

This is important for Draw3D's style system. The goal is to make it easy to produce a consistent look across a world and then customize individual objects without repeatedly opening complicated settings. The existing Draw3D material and style systems should be enriched in this direction.

## 6. Terrain shaping and vegetation

Tiny Glade's terrain tools show how terrain, buildings, and decoration can work together.

- You can paint raised platforms into the ground.
- Dragging up and down adjusts the height, while dragging horizontally changes the shape and smoothness of the edges.
- Connecting compatible platforms allows them to blend together.
- Buildings placed on raised terrain adapt to the changed ground level.
- Additional controls allow terrain platforms to be moved or adjusted after placement.
- Flowers, reeds, shrubs, bushes, and other vegetation can be painted into the environment and blend naturally with surrounding growth.
- Trees can be placed individually or connected into joined arrangements, while still allowing separation where required.
- Water can be painted into the ground to create ponds, complete with surrounding rocks and vegetation.

Draw3D has already made good progress with vegetation and nature placement. These observations are intended to help refine that work, particularly the ease of painting, blending, joining, and editing elements directly in the world.

## 7. Paths, stairs, fences, and walls

These tools demonstrate how a small set of construction operations can support many different results.

**Paths:** Different styles include dirt, cobblestone, rocks, and cracked stone. Paths can be painted, resized, and blended into surrounding ground. Painting around a doorway can create a wider paved standing area. Paths can also interact with walls, water, and slopes.

**Stairs:** Stairs can be drawn between the ground and walls, extended to different heights, and adapted to changes in elevation. Their ends can be repositioned, their curves adjusted, and intermediate control points added or removed.

**Fences:** Fences can be drawn as continuous sections, curved, edited through their control points, split into separate sections, and snapped back together.

**Walls:** Walls can be drawn and resized, with additional controls available after placement.

The useful principle here is that the tools are not isolated modelling operations. They recognize their surroundings and help the user create connected environments.

## 8. Lighting and environmental presentation

Tiny Glade includes simple ways to preview the environment under different lighting conditions.

- A lantern can immediately provide illumination when placed on a building or the ground.
- A time-of-day control lets you move between day and night.
- The sun's position can be adjusted to preview the direction of light and shadows.
- Chimneys can produce smoke, and vegetation and architectural details help make the environment feel more complete.

These are useful references for Draw3D's lighting, shadows, particles, and environmental presentation. They reinforce the importance of making a world feel finished, not merely structurally complete.

## 9. Interaction patterns worth considering

Across these tools, several patterns recur:

- Direct manipulation through handles and mouse dragging.
- Dragging vertically to change height or elevation.
- Dragging horizontally to change shape, curvature, or proportions.
- Right-clicking to reveal additional controls or finish an operation.
- Immediate visual feedback while adjusting values.
- Context-sensitive results based on where an object is placed.
- Automatic joining, adaptation, and snapping between compatible elements.
- A simple initial result, with more advanced controls available when needed.

These patterns make the interface approachable without sacrificing flexibility. They are worth considering as Draw3D's existing tools are refined and expanded.

## Closing note

These are my initial findings from Tiny Glade, based on actually using its tools. I am sure there are many more advanced behaviours I will discover over time.

For now, please treat this as personal user-experience feedback to inform the work already underway. The recent direction with modular construction and quick customization is encouraging. The intention is to help strengthen that approach by considering how Tiny Glade makes building, editing, and decorating feel intuitive and interconnected.

Please continue with the existing roadmap and implementation priorities. Consult these observations when they are relevant to the system being developed, and use your technical judgment to determine how best to incorporate the underlying ideas into Draw3D.
