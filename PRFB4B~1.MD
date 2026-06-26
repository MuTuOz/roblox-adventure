---
name: project_treasure_chest
description: "Timed per-player treasure chest world event in the forest (3h real-time slots, find-to-open, weighted loot)"
metadata: 
  node_type: memory
  type: project
  originSessionId: 9a3a17ba-3b49-4692-825a-1992670efe23
---

Timed treasure chest, built + verified 2026-06-24 (Despair Studio). Live in the place.

- **Model:** `GameData.Chest` config — `SlotSeconds=3*3600`, `GoldBase=150`, `BuffDuration=720` (12min), `GoldBuffMult=0.15`, `LuckBuffMult=2`, spawn `SpawnNear=14/SpawnFar=42/SpawnSide=24` (chest spawns just IN FRONT of the forest ARRIVAL point, in the open glade, visible on entry — changed 2026-06-24 from a 180-720 whole-map radius which was too hard to find), loot weights gold 0.55 / crystals 0.25 / buff 0.20.
- **Offline-aware timer:** chest "slot" = `floor(os.time()/SlotSeconds)`, so the 3h timer runs whether the player is online or not. Each slot's location is DETERMINISTIC from `Random.new(userId*31 + slot*1000003)` (angle+radius around forest CENTER, ground-snapped via raycast). Miss it → next slot's chest appears at a new spot. `ProfileService.chestFoundSlot` (persisted) tracks the last opened slot; `GetChestFoundSlot`/`SetChestFound`.
- **Per-player without instancing:** chest is CLIENT-rendered (`StarterPlayerScripts.TreasureChestClient`) so each player only sees their own — works now in the shared world and survives future per-player biome instancing. Server (`ServerScriptService.ChestSpawn`) is authoritative: sends `TreasureState{available,pos,slot,justSpawned}`, validates the open (`OpenTreasure` remote, must be within 28 studs), rolls loot, sets chestFoundSlot. A 30s watcher re-sends state on slot rollover.
- **Loot (one weighted roll, NO Robux, deliberately small "little edge"):** gold = `150*(1+rebirth)`; OR 1 crystal each of 2 distinct random colors; OR a 12-min temp buff (`BuffType`+`BuffUntil` attributes) — "gold" (+15% gold/kill) or "luck" (x2 crystal drop). `GameData.BuffActive(plr,kind)` checks os.clock() expiry; ForestMobs.die reads it (gold mult + drop-chance mult). NO chest-exclusive cosmetic (user cut it).
- **Findability/UX:** client chest has a tall yellow Neon light-beam beacon + PointLight + "Treasure!" billboard + a "Search" ProximityPrompt (1s hold). On spawn the player sees a center-top warning banner ("⚠ A treasure chest appeared in the forest! Go find it.", `TreasureWarn` ScreenGui). Open → sparkle burst + chest removed + reward Toast.
- New remotes: `TreasureState`, `OpenTreasure`.
- **Verified in playtest:** chest built at seeded forest pos with prompt; warning banner shown; open granted +150 gold and removed chest; second open same slot did nothing (one-per-slot guard); gold buff raised per-kill gold 16→18 and expired correctly.

Related: [[project_forest_biome_build]] [[project_balance_system]] [[project_hub_systems]]
