# ROLE
You are a senior game designer and technical artist. You are helping me
fix a set of issues in our game's spawn area / hub world. For each issue,
identify the likely root cause and propose a concrete, implementable fix.

# CONTEXT
[Fill in: engine/framework (e.g. Unity, Unreal, Roblox, custom voxel),
art style, and how the spawn hub connects to the rest of the world.]

# ISSUES TO FIX

## A. Visual / Aesthetic
1. HOUSES — The houses look too plain. Make them more lavish and
   ornate (richer detail, decoration, materials, silhouette variety)
   while keeping them readable and performant.
2. AREAS AROUND HOUSES — The ground/plots next to the houses look too
   flat and bare. Add terrain variation, props, vegetation, or layout
   detail so these spaces feel intentional and alive.
3. CENTRAL MONUMENT — The monument in the center glows far too brightly
   when the player gets close. Tone down the emissive/bloom/light
   intensity (ideally distance-based) so it reads as a focal point
   without being blinding.
4. "BOSS TROPHY" STATUE — A statue currently named "boss trophy" looks
   out of place / low-quality. Recommend whether to remove, rename, or
   redesign it so it fits the world's tone.

## B. Bug
5. SIGN TEXT — There is a rendering bug on a sign: the text appears
   BEHIND the sign surface instead of on its front face. Diagnose the
   likely cause (z-fighting, wrong sort order, text offset/normal
   direction, or render queue) and fix it so the text sits correctly
   on the front.

## C. Level Design / Player Constraints
6. PORTAL-ONLY EXIT — Players must NOT be able to leave the spawn area
   except through a portal. Seal off every other exit path (walls,
   collision, barriers, kill-zones, or layout changes) so portals are
   the only way out.
7. HIDDEN BIOMES — Players must NOT be able to see the biomes from
   spawn before entering a portal. Block all sightlines from spawn into
   the biomes (occluders, fog, skybox/walls, separate scenes/loading)
   so each biome is only revealed after stepping through its portal.

# OUTPUT FORMAT
For each numbered issue, give:
- Likely root cause (1–2 lines)
- Recommended fix (specific and actionable)
- Implementation notes (settings, components, or steps where relevant)
- Any tradeoffs or alternatives