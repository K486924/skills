---
name: blender-roblox-assets
description: Create clean, optimized, Roblox-ready low-poly 3D assets in Blender, with reliable pivots, naming, materials, and export settings.
---

# Blender to Roblox Asset Skill

## Goal
Create polished, readable, lightweight 3D assets for Roblox experiences. Prefer a cohesive low-poly/stylized look, clean silhouettes, simple materials, and assets that import predictably into Roblox Studio.

## Before modeling
1. Read the user's request and identify the asset, intended in-game scale, use, and visual style.
2. Inspect the existing Blender scene before changing it. Do not delete or overwrite unrelated objects.
3. If the user asks for a set, establish a shared style, scale, palette, and naming convention.
4. Ask only when a missing detail would materially change the result; otherwise choose sensible game-ready defaults.

## Modeling standards
- Use clean, intentional geometry. Favor bevels and silhouette over tiny decorative detail.
- Keep meshes low-poly and efficient. Avoid dense subdivision, hidden internal faces, unnecessary modifiers, and excessive separate objects.
- Apply object transforms before export: rotation and scale should be clean; use Ctrl+A > Rotation & Scale where appropriate.
- Set origins/pivots intentionally. Put the origin at the natural placement point (usually bottom-center for props, hinge for doors, center for symmetric pickups).
- Use consistent real-world/game scale. Treat 1 Blender unit as 1 stud unless the project specifies otherwise.
- Use descriptive object names such as `DogHouse_Body`, `FoodBowl_Main`, and `ParkBench_Seat`; avoid names like `Cube.001`.
- Keep object hierarchy understandable. Join static pieces when sensible, but keep independently animated or interactive parts separate.
- Use weighted normals or smooth-by-angle only when they improve the silhouette. Keep flat faces flat.
- Avoid paper-thin surfaces, open accidental holes, coplanar flicker, and overlapping geometry.

## Materials and appearance
- Use a small, consistent palette with readable color contrast.
- Prefer simple Principled BSDF materials and solid colors. Avoid complex node networks, procedural dependencies, transparency, and texture-heavy materials unless specifically requested.
- Give materials clear names (for example `M_GrassGreen`, `M_WoodWarm`, `M_MetalDark`).
- Use a restrained roughness range; avoid unnecessary metallic effects.
- Do not rely on Blender-only effects that will not transfer to Roblox.
- If textures are necessary, pack or deliver them and explain how to import them.

## Roblox performance
- Optimize for low-end devices: modest triangle counts, few materials, limited object count, and no needless micro-detail.
- Reuse meshes/materials for repeated props where possible.
- Avoid excessive transparency, high-resolution textures, and many unique material slots.
- Do not create huge monolithic meshes when the user may need to reposition or reuse parts.
- Keep collision geometry simple; recommend simple Roblox collision fidelity for decorative meshes.

## Export workflow
1. Save the working `.blend` file.
2. Select only the intended export objects.
3. Apply transforms and verify object origins.
4. Export FBX for Roblox Studio unless the task specifically calls for OBJ.
5. Use selected objects only; apply unit scale consistently; avoid exporting cameras, lights, and unrelated scene objects.
6. Use forward/up axes compatible with the target Roblox import workflow and verify orientation in Studio.
7. Export textures separately when required and keep filenames stable.
8. Never claim an export succeeded unless the file exists and has been checked.

## Deliverables
For each asset task, provide:
- The editable Blender project when requested.
- Exported FBX/OBJ files.
- Any required texture files.
- A short manifest listing asset name, approximate scale, pivot location, and intended use.
- Import notes for Roblox Studio, including any recommended collision setting.

## Safety and project integrity
- Inspect before editing. Preserve the user's existing scene and source files.
- Do not overwrite an existing asset without checking or making a backup.
- Do not run destructive cleanup, shell commands, or scripts unrelated to the requested asset.
- Do not add scripts or executable content to a model unless explicitly requested.
- If a requirement cannot be met, explain the limitation rather than silently substituting a different output.

## Final quality checklist
- [ ] Correct silhouette and proportions
- [ ] Consistent scale and intentional pivot
- [ ] Clean names and hierarchy
- [ ] Applied transforms
- [ ] No obvious holes, flicker, or stray geometry
- [ ] Lightweight mesh and material count
- [ ] Roblox-compatible export checked
- [ ] Source project and deliverables preserved
