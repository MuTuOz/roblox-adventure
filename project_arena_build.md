---
name: project-arena-build
description: "PvP Arena (medieval colosseum) built in Roblox Studio place v1 — location, structure, portal wiring"
metadata: 
  node_type: memory
  type: project
  originSessionId: 098530d3-ca94-46d7-8687-101d1706c523
---

# PvP Arena — Build State (as of 2026-06-21)

The PvP Arena destination for the [[project-game-concept]] MMORPG, reached via the PvP Arena portal in the [[project-town-hub-build]] hub. Built in Studio place "v1".

## Location & isolation
- Built as an **isolated skybox arena** centered at world **(3000, 500, 0)** — ~2305 studs from hub spawn, ~3155 from Forest, lowest part Y=380 (500 up in the sky).
- **Isolation mechanism:** `Workspace.StreamingEnabled = true` (default target radius ~1024). At 2305 studs (>2x radius) the arena never replicates to a client in the hub, so it is invisible from hub/spawn/other biomes as required. Nearest non-arena part is 2078 studs away — nothing bridges them.

## Style (per user brief)
Open-air medieval **colosseum**: simple-not-childish, medieval theme, relatively large, classy lighting, medieval decoration.

## Hierarchy: `Workspace.TownHub.Arena`
- `Structure` — `ArenaFloor` (sand cylinder, 112 dia, top Y=500, CanCollide), `CenterRing` (30-dia duel medallion), `Foundation` (224-dia base slab), `PodiumBlock`/`PodiumCap` ring (r=57, h=12, separates floor from seating).
- `Seating` — 6 concentric stone tiers (`Seat`+`Riser`), innerR=59 → outerR=99, rise 5.5/tier.
- `OuterWall` — `OuterBlock`/`Pilaster`/`Cornice` ring (R=104, H=42) + `Merlon` crenellations. Wall top ~Y=538.
- `Entrances` — 4 cardinal `Arch_South/North/East/West` (gold `Keystone`, piers, lintel, passage floor). South (-Z) is main arrival; wall blocks carved out at each opening.
- `Towers` — `Tower_1..4` at diagonals (cylindrical shaft h=58, bands, battlements, **cone roof mesh rbxassetid://1033714**, gold finial, team pennant).
- `Decor` — 24 hanging wall `Banner`s (red/blue/gold + cream crest) draped on OUTER face (R+1.9); `RoyalBox` at +Z (red drape, gold trim, 2 thrones, canopy); 4 `WeaponRack` (shield+crossed spears) on podium; 2 `Barrels` clusters on sand.
- `Lighting` — 20 `WallTorch` + 8 `Brazier` (tripod+bowl). Each `Flame` = orange Neon cone + `Fire` particle + warm `PointLight` (range 30-42) + ember `ParticleEmitter`. Fire/glow render at runtime, not in edit-mode captures.
- `Pads` — `ArenaArrival` (red Neon, 3000,500,-44, players land here) + `ArenaReturnPad` (cyan Neon, "Return to Town" billboard, 3022,500,-40).

## Scripts
- `ServerScriptService.HubPortals` (existing) updated: PvP portal prompt now "Enter / PvP Arena" → teleports to `ARENA_LAND = (3000,503,-44)`; added Touched handler on `ArenaReturnPad` → teleports back to `HUB_SPAWN (750,4,-76)`. Same pattern as the Forest portal.
- `StarterPlayer.StarterPlayerScripts.ArenaTorchFlicker` (LocalScript) — gentle Heartbeat flicker on all arena PointLights (dual-sine, ±~16% brightness) for a classy living-fire feel.

## Palette
Stone greys (120,114,102 / 96,90,80 / 151,144,131), TRIM (78,72,63), GOLD (170,138,72), roof dark-red (98,50,48), sand (196,176,134), banner red/blue/gold, FLAME (255,125,30).

**Why:** Records where/how the arena was built and how it ties to the portal so future sessions extend it (add spawns, combat, scoring) without rediscovering coords/wiring.
**How to apply:** Keep the arena in its isolated skybox slot; reuse `Arena.Pads` + `HubPortals` for teleport changes; flames/lights live under `Arena.Lighting`.
