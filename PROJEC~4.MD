---
name: project-forest-biome-build
description: "Forest biome (\"The Emerald Vale\") built in Roblox Studio place v1 — isolated grass valley, 4 crystal slime camps, portal wiring, minimap, zone lighting"
metadata:
  node_type: memory
  type: project
  originSessionId: a2d04e43-2f75-47ff-857b-687933a7519f
---

# Forest Biome — "The Emerald Vale" (as of 2026-06-21)

Biome 1 (Forest) for the [[project-game-concept]] MMORPG, reached via the Forest portal in the [[project-town-hub-build]] hub. Built in place "v1". Isolation pattern mirrors [[project-arena-build]].

## Location & isolation
- Huge grass valley centered at world **(-3500, 0, 0)**; terrain floor top at y=0. ~3750 studs from hub (750,0,0), ~6000 from arena.
- **Isolation = 4 layers:** (1) `StreamingEnabled=true` (~1024 radius) → hub/arena never replicate at this distance; (2) client zone fog `FogEnd≈830`; (3) terrain **mountain ring** (~30 peaks, r≈1000, up to y≈185); (4) invisible collision **wall ring** (`Boundary`, 40 segs, r=870). Verified in playtest: hub/arena not visible from valley.
- **Portal-only exit:** invisible walls + `ServerScriptService.ForestBounds` (teleports to ARRIVAL if a player in the forest region falls below y=-50). Return portal is the only way out.

## Single source of truth
- `ReplicatedStorage.ForestConfig` (ModuleScript) holds ALL constants: CENTER, ARRIVAL `(-3500,8,70)`, RETURN_PAD `(-3500,2,150)`, HUB_SPAWN `(750,4,-76)`, ZONE_R=1300, POND `(-3380,0,280)`, the 4 CAMPS (id/name/crystal/mob/pos/color/glow), MOB_STATS, and the forest LIGHTING profile. Edit camps/coords/lighting here; all scripts read it.

## Hierarchy: `Workspace.ForestBiome`
- `Structure` — `AncientTree` (hero centerpiece, fireflies, hanging vines), `ArrivalDais` (teal rune dais where Forest portal lands), 8 `StandingStone`s, `ReturnPortal` (stone arch + neon portal plane), `BiomeSign` ("THE EMERALD VALE").
- `Camps` — `Camp_GREEN/YELLOW/BLUE/RED`, each: ground rim disc (camp color), `CrystalNode` (faceted Glass+Neon crystal cluster on rock base, glow PointLight + sparkles), campfire, 2 banners, tent, crates, `Sign` (BillboardGui), and `Mobs` folder of 6 slimes. Camp attributes: CampId, CampName, CrystalName, CrystalColor.
- `Flora` — `Trees` (328 stylized: round oak / conifer / broad; dense tree-wall ring at r720-808 for occlusion), `Bushes`, `GroundCover` (grass tufts, flowers, mushrooms), `Rocks`, `Logs`.
- `Boundary` — 40 invisible CanCollide wall parts (r=870).
- `Decor.Paths` — stone tile roads glade→each camp, laid on **flattened graded corridors** (terrain carved flat to y≈2 with shoulder grading; all flora cleared within ~30 studs of centerline). **Design rule (user): roads must never cross hills or any object** — keep corridors clear when scattering props or re-sculpting. `Pads`, `Effects` reserved.

## Camps & mobs (crystal mapping per game concept)
- GREEN "Mossy Hollow" → Verdant Crystal, **Mossling Slime** (leaf trait); at (-3076,0,424)
- YELLOW "Sunlit Meadow" → Amber Crystal, **Sunpip Slime** (antennae); at (-3924,0,424)
- BLUE "Misty Spring" → Tidal Crystal, **Dewdrop Slime** (water droplet); at (-3924,0,-424)
- RED "Emberwood" → Ember Crystal, **Cinder Slime** (flame tuft); at (-3076,0,-424)
- **Slimes = same stats, different phenotype** (per user): each tagged `ForestSlime`, attributes MaxHealth=60, **AttackPower=15**, Armor=2, Level=1 (identical), plus CrystalDrop=camp id. Gel ball body + Neon core + cute eyes/smile + per-camp trait. Combat IS built (see below).

## Scripts (added this build)
- `ServerScriptService.HubPortals` (existing) updated: `FOREST_LAND=(-3500,8,70)`, return pad position `(-3500,1.5,150)`. Round-trip teleport playtested OK.
- `ServerScriptService.ForestBounds` — fall-safety (above).
- `StarterPlayer.StarterPlayerScripts.ForestZoneLighting` (LocalScript) — zone-detects (XZ dist to CENTER < ZONE_R); tweens Lighting+Atmosphere between FOREST profile (green fog/haze) and hardcoded DEFAULT (hub) profile, toggles `Lighting.ForestCC` ColorCorrection. Keeps hub/arena lighting untouched. **Global Lighting/Atmosphere left at hub defaults** (fog off / FogEnd 9000, Atmosphere Density .39 Haze 1.9) — do NOT change globally; per-zone is client-side.
- `StarterPlayerScripts.ForestMinimap` (LocalScript) — bottom-right "EMERALD VALE" minimap: 4 colored camp markers (G/Y/B/R) + names, glade/portal center marker, paths, live player arrow; only Enabled in forest zone.
- `StarterPlayerScripts.ForestSlimeIdle` (LocalScript) — cheap client bob/tilt for all `ForestSlime`-tagged mobs via stored `BasePivot`+`Phase` attributes.

## Notes / conventions
- Retired old origin WIP forest (deleted `Workspace.Trees` 120 + `Workspace.ForestProps` 227 simple tree models). Baseplate (origin ±1024) and hub untouched.
- Terrain material colors set globally (only the forest uses Terrain): Grass (86,152,64), Rock (120,120,112), Water teal.
- Props are ground-snapped via downward Raycast vs Terrain — reuse that when adding props.
- Display name "The Emerald Vale"; internal folder "ForestBiome".

## Combat, weapon, shield, inventory (built 2026-06-22)
- **Data:** `ReplicatedStorage.GameData` (ModuleScript) is the tuning source of truth: DropChance 0.05, RespawnTime 30, ForestGoldPerKill 15, regen (1/s in combat, 10/s out, CombatTimeout 15), PickupLifetime 60, **MobMeleeRange 8.5, MobAttackInterval 2.0, BlockReduction 0.75**, Crystals, **Weapons.CandyCane** (atk 12), **Shields.GummyShield** (armor 3). Starter weapon=CandyCane, starter shield=GummyShield.
- **Player:** 100 HP; **ATK & DEF come only from items** (CandyCane 12 atk + GummyShield 3 armor). `ServerScriptService.PlayerInit` sets HP, clones the weapon Tool from `ServerStorage.Weapons` by display name, auto-equips, routes `EquipItem` to weapon/shield, sets `Blocking` attr from `PlayerBlock` remote, resets Blocking on spawn/death.
- **Profiles:** `ServerScriptService.ProfileService` (ModuleScript) — per-player crystals/weapons/shields/equipped/gold, ComputeStats (atk+armor from equipped weapon+shield → player attributes), Sync via `InventorySync`, best-effort DataStore (off in Studio, pcall-guarded). NOTE: requiring ProfileService from an MCP execute gives a *fresh* module copy (empty profiles) — verify via player attributes, not require.
- **Weapon:** `ServerStorage.Weapons["Candy Cane"]` Tool = small white shaft + red diagonal stripes + rounded hook; Handle has soft **Hit** sound (rbxassetid 9120112642) + **Block** boing (86557022977865). `ClientAttack` LocalScript: left-click attack (cooldown 0.45, finds slime under mouse or nearest in 20), **guards: no attack while dead or blocking**; right-click hold = block (fires `PlayerBlock`). `ShieldRig` Script builds the **blue Gummy Shield (pink heart)** welded to HRP off-hand on Equip, tweens it to a raised pose while Blocking.
- **Mobs (`ServerScriptService.ForestMobs`):** `MobHit` remote (guards dead/distant attackers), damage→health bar + **white floating damage numbers** + spark + **recoil** (sets LastHitTime/HitDirX/Z attrs; `ForestSlimeIdle` reads them to lurch the slime back = backlash #10). Death → pop + slime-splat sound + **+15 gold** + 5% physical crystal drop + 30s in-place respawn. **Retaliation is per-mob staggered** (each slime attacks ~every 2s only within 8.5 studs) with a **0.45s per-player i-frame** so a 6-slime camp can't burst you (fixes one-shot #4); damage = max(1, mobAtk−playerArmor), ×0.25 while blocking; fires `PlayerHit` → client red flash + camera shake (#9).
- **Crystal drop pickup (#11):** glowing Neon gem + upward light beam + ground glow ring + sparkles + label, bobs/spins, walk-over `Touched` → AddCrystal + chime (93529351909119) + Toast. Despawns after 60s.
- **HUD (`StarterPlayerScripts.ForestInventoryUI`):** top-left ATK/DEF + HP bar, gold pill, bottom-left **INVENTORY button (shortcut "I")** opening a panel with crystals + gold + gear (weapons & shields, click to equip). Also hosts the `PlayerHit` red-flash + shake.
- **Remotes** (`ReplicatedStorage.Remotes`): MobHit, InventorySync, EquipItem, Toast, **PlayerBlock, PlayerHit**.
- **Arena return is now a portal** (`Workspace.TownHub.Arena.ReturnPortal`): stone arch + cyan Neon plane over the existing `ArenaReturnPad` (touch logic still in `HubPortals`); old flat pad billboard disabled.

**Why:** Records where/how Biome 1 was built and how it ties to the portal/minimap/lighting so future sessions add combat, harvesting, the biome boss, and rebirth hooks without rediscovering coords/wiring.
**How to apply:** Extend via `ReplicatedStorage.ForestConfig`; keep the forest in its isolated slot; reuse `ForestSlime` tag + camp attributes for combat/drops; add the Forest **boss** (game concept: each biome ends in a boss) in a far clearing or beyond a 5th path. Desert/Hell biomes should copy this isolation+config+zone-lighting pattern at their own far coords.
