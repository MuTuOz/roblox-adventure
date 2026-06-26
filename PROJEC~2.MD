---
name: project-town-hub-build
description: "Town Hub prototype built in Roblox Studio (place \"v1\") — location, layout, and conventions"
metadata: 
  node_type: memory
  type: project
  originSessionId: 96d29ecd-db9e-4abd-b14a-829221d42b49
---

# Town Hub — Build State (as of 2026-06-21)

First built area for the [[project-game-concept]] MMORPG. Created in Studio place "v1".

## Location & coordinates
- Hub centered at world **(750, 0, 0)** — placed clear of existing Forest content (which spans ~±500 around origin). Baseplate is 2048×2048, top at y=0.
- Forest biome WIP lives near origin (Trees, ForestProps, MonsterCamps: Bunny_Meadow, Frog_Pond, Chick_Coop, Piglet_Pen).

## Hierarchy
- `Workspace.TownHub` → folders: `Structure` (plaza, Monument), `Portals`, `Bases`, `Decor`, `Effects`.
- Scripts in `ServerScriptService`: `HubMonument` (crystal float/spin animation), `HubPortals` (proximity prompts + Forest teleport + return pad).

## Layout
- Round layered medallion plaza + central **Rebirth Crystal Monument** (white spire + 4 floating crystals Red/Blue/Yellow/Green, ties to rebirth lore).
- Spawn pad on south edge (CFrame ~750,1,-76).
- **Portal gallery** on north terrace: 4 portals left→right = Forest (open), Desert (locked, Rebirth 10), Hell (locked, Rebirth 20), PvP Arena (open). Models named `Portal_FOREST/DESERT/HELL/PvPARENA`; portal plane child is `PortalSurface`.
- **8 player bases** ring the plaza (`Base_1..8`) at radius 152, angles {130,170,210,250,290,330,10,50}, north gap reserved for portals. Each base has unique accent color, cottage w/ colored roof, glass-domed trophy podium, rebirth banner ("REBIRTH 0"). Attributes: BaseIndex, OwnerUserId.
- Decor: connector paths, 9 lampposts, 2 grand flag poles, perimeter trees, welcome arch reading "TOWN SQUARE / DESPAIR STUDIO".

## Conventions established
- Palette: warm sandstone/cream stone, gold trim, teal accent, wood; Neon for glows; Future lighting + skybox, warm daytime (ClockTime ~13).
- Forest portal teleports player to (-114, 8, -121); `ReturnToTownPad` in forest teleports back to hub spawn.

## Design-review fixes pass (2026-06-21, after CLAUDE.md brief)
- Houses redesigned: detailed cottages (stone base, trim, chimney, framed windows + flower boxes, porch lantern, overhanging ClayRoofTiles roof) with 3 silhouette variants (gable / cross-gable wing / corner turret), under each `Base_i.House`.
- Plots dressed: `Base_i.Yard` (walkway, hedges, tree, garden bed, potted plants, lamppost, LeafyGrass lawn patch).
- Trophy redesigned: `Base_i.TrophyDisplay` — stylized stone "Forest Warden" boss bust under glass dome, renamed from "BOSS TROPHY"; plate reads FOREST WARDEN / Boss Trophy.
- Monument glow: lowered light/bloom values + client `StarterPlayerScripts.HubMonumentGlow` does distance-based dimming (focal but not blinding up close).
- Signs hardened: entrance signs inset proud + double-sided SurfaceGui (Front+Back).
- Hub sealed + biomes hidden: `TownHub.Boundary` = circular stone rampart (radius 216, 36 segments, height 40 + merlons, 8 towers, hedge skirt, invisible anti-exploit lip). Portals are the only exit. Atmosphere haze bumped; `ServerScriptService.HubBounds` is a fall-safety teleport.

## Pass 2 (2026-06-21, second feedback round)
- Bases are now ENTERABLE 2-story mansions (`Base_i.Mansion`): hollow ground floor, 7-wide door opening, glass windows, chandelier interior light, columned portico + balcony, cupola crown, corner quoins. Plots enlarged to 46x46.
- Each mansion has a treasure `Chest` (Base_i.Mansion.Chest) with a `ChestLid`; `ServerScriptService.HubChests` adds an "Open" ProximityPrompt that tweens the lid open/closed.
- Name signs converted from SurfaceGui to **BillboardGui** (`Base_i.NameAnchor.NameTag`, and rebirth as `RebirthAnchor`) — root cause of the "names vanish up close" bug was a flat high-mounted SurfaceGui viewed edge-on; BillboardGui always faces camera. MaxDistance=0, LightInfluence=0.
- Monument dimming strengthened: `HubMonumentGlow` now also fades the Neon crystals' transparency and lowers Bloom on approach (not just the PointLights). Bloom Intensity ~0.25 / Threshold ~2.1.
- Yards re-dressed for the bigger plots; light decoration added between bases via `Decor.RingClusters` (trees, flower patches, benches).

## Pass 3 (2026-06-21, third feedback round)
- **Rebirth sign unblocked:** each `Base_i.Yard` had a decorative tree (Trunk+3 Leaves) whose foliage buried the rebirth banner. Relocated trees: each base now has TWO trees framing the gate at front corners — left at foundation-local (-19,-18), right at (19,-20.5). Rebirth assembly (`RebirthAnchor`+`Banner`+`BannerPole`) moved to local (14,-19), ~6.7 studs clear of the right tree.
- **Houses enlarged:** `Base_i.Mansion` uniformly scaled **1.22x** about its footprint-center-bottom pivot (kept grounded at minY). Footprint 33x30 → ~40x36 on the unchanged 46x46 foundation (~3-stud margin to walls). Done via per-part Size*scale + position offset*scale in each base's foundation-local frame (bases are identical instances, rotY increments of 40°).
- **Windows on all 4 sides:** only `WallBack` lacked windows. Added 2 framed back windows per house (`WinBack` glass + `WinBackBar` wood mullion cross + `WinBackSill`), oriented with wide dim along wall-local X, thin (1.3) along Z. NOTE: cloning a side window (WinR) put the thin axis on the wrong axis — build back-wall parts with explicit sizes, don't clone side windows.
- **Interior furniture:** each `Base_i.Mansion.Interior` folder holds a dining set on the rug (center local ~(0,*,7.5)): TableTop+4 TableLegs, 4 chairs (ChairSeat/ChairLeg/ChairBack, red Fabric cushions), Vase+FlowerStem+FlowerBloom centerpiece, a corner PotBase+PlantFoliage, and a Cabinet on the left wall. Floor top at local Y≈2.07.
- **Plaque shows player name:** gate `Plaque` SurfaceGui TextLabel now reads "<OWNER>'S ESTATE" (or "VACANT" when unowned) instead of generic "ESTATE". Driven by existing `OwnerName` attribute via new `ServerScriptService.BasePlaqueBinder` (binds plaque + NameTag subtitle, updates live on OwnerName change). Sample owner names set on bases for demo; Base_5 left vacant.

## Pass 4 (2026-06-23, base ownership wired up)
- **Auto base assignment:** new `ServerScriptService.BaseOwnership` (Script) gives every joiner the first vacant base (sets `OwnerUserId`+`OwnerName`); frees it immediately on leave (resets to 0/""/banner 0). Hub holds 8; a 9th player just gets no base. Handles players already in-game at script start (Studio playtest) via task.spawn.
- **Shared base helper:** new `ServerScriptService.BaseRegistry` (ModuleScript) = single source of truth for base lookup + banner writing: `AllBases()`, `IsVacant(b)`, `GetPlayerBase(plr)` (by OwnerUserId), `SetBaseRebirth(b,level)`, `Assign(b,plr)`, `Free(b)`. Both BaseOwnership and ShopRebirthService use it (ShopRebirthService's old duplicate lookup/banner code was removed) so they never fight over the same base.
- **Demo names cleared:** the hardcoded demo OwnerNames (Tuna/Murat/AquaWolf/…) were reset — all 8 bases now start VACANT at edit-time for clean live assignment.
- **NameTag title fix:** `BasePlaqueBinder` now also sets the **title** label to "BASE n" (from BaseIndex/name) authoritatively — older demo data had polluted some titles with owner names. Subtitle still = owner name or "Empty Plot"; plaque still = "<OWNER>'S ESTATE"/"VACANT".
- **Verified in playtest:** live player auto-assigned Base_1 (plaque "ORDEKMOZEN'S ESTATE", title "BASE 1", sub "ordekmozen"); real client RequestRebirth raised level & updated that base's banner (0→3); freeing a base reverts plaque→VACANT/sub→Empty Plot/banner→0 and marks it vacant=true; 7/8 vacant with one player.

**Why:** Records where/how the hub was built so future sessions extend it instead of rediscovering coordinates and naming.
**How to apply:** Build new biome/hub work relative to these coords and naming; keep the accent-color and rebirth-lock schemes consistent. All 8 bases are identical instances differing only by foundation CFrame — work in foundation-local space and loop 1..8 for any per-base change. For anything that needs a player's base or the rebirth banner, use `BaseRegistry` (don't re-implement lookup). See [[project-rebirth-shop-build]] for the rebirth/shop systems that drive the banner.
