---
name: roblox-world-builder
description: Build organized, attractive, performant Roblox environments and gameplay spaces using Roblox Studio, with safe scripting and clear testing steps.
---

# Roblox World Builder Skill

## Goal
Create complete, easy-to-navigate Roblox environments that feel polished while remaining lightweight and maintainable. Prefer modular construction, clear player flow, readable landmarks, and simple interactions.

## Project safety
1. Inspect the existing Roblox place and Explorer hierarchy before making changes.
2. Preserve existing work. Never replace the place, delete folders, or rewrite scripts without understanding their purpose.
3. Work in a duplicate place or create a backup before major edits.
4. Make changes in small, testable stages.
5. Do not publish, change permissions, or expose a private experience unless the user explicitly asks.

## Scene organization
Use a predictable hierarchy, for example:
- Workspace
  - Map
    - Terrain
    - Paths
    - Buildings
    - Props
    - Nature
  - Gameplay
    - ThrowingArea
    - DogZone
    - UpgradeStations
  - SpawnLocation
- ReplicatedStorage
  - Assets
  - Remotes
- ServerScriptService
- StarterGui
- StarterPlayer

Use descriptive names. Avoid leaving hundreds of anonymous parts directly under Workspace. Group related static objects into Models and set PrimaryPart when it helps placement or scripting.

## Environment design principles
- Establish a clear spawn point and an obvious first objective.
- Use a strong focal point and landmarks so players understand where to go.
- Separate gameplay zones with paths, signs, color, lighting, and visual framing.
- Keep walkways wide enough for players and avoid decorative clutter in movement paths.
- Use consistent proportions, color palette, material language, and lighting.
- Add detail in layers: large shapes first, then medium props, then a small amount of accent detail.
- Ensure important interactive objects are visually distinct and reachable.

## Example: food-throwing dog park
For a simple food-throwing simulator, a useful layout is:
1. Spawn plaza with a short welcome sign and clear instructions.
2. Raised or marked throwing area with a safe, unobstructed aiming direction.
3. Dog feeding zone in front of the throwing area, with enough space for NPC movement.
4. Food/upgrade stalls placed to the side so they do not block the main loop.
5. Reward or coin indicator near the player-facing side of the map.
6. Perimeter fence, trees, benches, bins, lamps, flower beds, and other low-cost props to make the park feel complete.

Keep the core loop easy to understand: aim, throw food, dog reacts/eats, earn coins, buy an upgrade, repeat. Do not add complex systems unless requested.

## Parts, meshes, and collisions
- Prefer simple Parts, unions only when appropriate, and reusable MeshParts for custom assets.
- Anchor static environment pieces. Keep only necessary moving objects unanchored.
- Use CanCollide, CanTouch, and CanQuery intentionally; disable unnecessary interactions on decorative objects.
- Use simple collision geometry for decorative meshes.
- Avoid excessive unions, high-part-count detail, giant invisible colliders, and overlapping surfaces.
- Use Terrain for broad natural ground where it makes sense; use Parts for clean stylized paths and platforms.
- Check for gaps, snag points, spawn obstructions, and objects that players can get stuck inside.

## Performance
- Target low-end devices as well as desktop.
- Reuse meshes and materials; keep unique textures and transparent parts limited.
- Avoid excessive particle emitters, dynamic lights, shadows, and high-frequency effects.
- Keep NPC counts and update loops modest.
- Use streaming and collision settings appropriately for larger worlds.
- Test with Studio performance tools and multiple graphics levels when possible.

## Scripting rules
- Prefer small, clearly named scripts and ModuleScripts with one responsibility.
- Use server authority for currency, rewards, purchases, and important gameplay state.
- Validate all client requests on the server. Never trust a client-provided reward amount, target, or purchase result.
- Use RemoteEvents/RemoteFunctions only when needed and validate arguments, distance, cooldowns, and player state.
- Avoid infinite loops without task.wait(), runaway connections, and expensive per-frame work.
- Use task.wait/task.delay rather than deprecated wait/spawn/delay.
- Do not use loadstring, obfuscated code, hidden backdoors, or third-party free models with unknown scripts.
- Do not insert scripts from untrusted Toolbox assets. Inspect every script before use.
- Keep UI readable on different screen sizes and provide basic feedback for actions.

## Testing checklist
1. Run the experience in Play mode and confirm the player spawns correctly.
2. Walk the full map and test all paths, platforms, doors, and collision boundaries.
3. Check that interactive objects are reachable and prompts are clear.
4. Test gameplay with at least two players when multiplayer behavior matters.
5. Check Output for errors and warnings.
6. Check performance with a low graphics setting and avoid unnecessary visual effects.
7. Confirm no unintended scripts, external assets, or destructive changes were introduced.

## Delivery expectations
Summarize what was created and where it lives in Explorer. List scripts added or changed and explain their purpose. Include any manual steps still required, such as importing a mesh, setting a PrimaryPart, or testing a RemoteEvent. Do not claim something was tested or published unless it actually was.
