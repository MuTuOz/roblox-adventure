---
name: project_hub_systems
description: "Town hub QoL + monetization — owner-only chests, base spawn, escalator ring, Robux cosmetics"
metadata: 
  node_type: memory
  type: project
  originSessionId: 9a3a17ba-3b49-4692-825a-1992670efe23
---

Hub features added 2026-06-23 (all live in the place):

- **Owner-only chests:** `HubChests` now gates each base chest's "Open" prompt — `prompt.Triggered` checks the triggering player's UserId against the base's `OwnerUserId` attribute; non-owners get a "This isn't your chest!" toast. Chests are at `Workspace.TownHub.Bases.Base_N.Mansion.Chest`.
- **Spawn on own base:** `PlayerInit.spawnAtBase` redirects HUB spawns only (guard: within ~300 studs of x=750,z=-50) so forest(-3500)/arena(3000) respawns are untouched. Uses `BaseRegistry.GetPlayerBase`; `baseSpawnCFrame` raycasts to plaza/path ground in front of the yard (excludes the base model + characters; clamps groundY>10 to 1). Faces the plaza.
- **Sideways escalator:** `Workspace.TownHub.BaseEscalator` = SINGLE-lane octagonal moving walkway ring (160 parts), center (750,~0.5,-17), R=92, CCW, speed 40. Look (2026-06-24, no glass): dark DiamondPlate tread (Conveyor attr) with big yellow Neon direction CHEVRONS pointing travel direction, SOLID dark-metal balustrade + black handrail on the INNER (monument) side, low outer metal skirt + yellow safety stripe; OUTER side open so players board from their base. Driven by `ServerScriptService.HubEscalator` which re-applies each Tread part's `Conveyor` Vector3 attribute to `AssemblyLinearVelocity` every Heartbeat (anchored parts need this). (Previously a 2-lane pink/mint candy ring; redesigned 2026-06-24 to one proper escalator.) Hub layout: 8 bases ring at r~140-151; 4 portals NORTH at r~165; services (ShopKiosk r52, RebirthAltar r43) inside; one arch SOUTH at r101-105.
- **Robux cosmetics (monetization):** `GameData.Cosmetics` (AngelWings=wings, SweetHalo=halo), `CosmeticOrder`. Price 80 Robux ≈ $1 USD (2026 buy rate ~81.5 R$/$). Sold as Developer Products — **productId is 0 (placeholder); user must create the products on the Creator Dashboard and paste real assetIds**. **Perks: wings `walkSpeedMult=0.30` (+30% WalkSpeed, 16→20.8); halo `goldBonus=0.10` (+10% gold per mob kill, applied in ForestMobs.die via the `GoldBonus` player attribute, NOT boss gold).** Multiple cosmetics can be worn at once — profile stores `equippedCosmetics` SET (table id->true), `EquipCosmetic` toggles. `CosmeticUtil.Apply` rebuilds one `CosmeticRig` with all equipped rigs + sums speed/gold perks + sets WalkSpeed=16*(1+ΣwalkSpeedMult). `CosmeticService` wears on spawn + `EquipCosmetic` remote. `MonetizationService` ProcessReceipt → GrantCosmetic (auto-equips). Shop UI rows show `perk` text; buy button prompts purchase or EQUIP/EQUIPPED toggle per item. New remotes: `UpgradeWeapon`, `EquipCosmetic`. Test helper `ServerScriptService._StudioDebugGrant` (Studio-only) grants 1e9 gold + both cosmetics equipped.

Wing visuals are basic procedural neon parts — swap for catalog meshes if fancier look wanted. Related: [[project_balance_system]] [[project_town_hub_build]] [[project_rebirth_shop_build]]
