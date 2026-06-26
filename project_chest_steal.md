---
name: project_chest_steal
description: Base chest item storage + inspect other players + 500-Robux steal mechanic
metadata: 
  node_type: memory
  type: project
  originSessionId: 9a3a17ba-3b49-4692-825a-1992670efe23
---

Chest/inspect/steal system, added 2026-06-24 (Despair Studio). All live in the place.

- **Chest storage (persistent):** weapons/shields (NOT cosmetics) can be stored in a base chest. `ProfileService` gains `chest = {}` (set of ids), persisted in save/load (load also strips chest ids out of weapons/shields to avoid overlap). `M.DepositItem` (blocks the equipped weapon/shield), `M.WithdrawItem`, `M.OwnsItem`, `M.GrantItem`. Items leave the bag's GEAR list when stored and return on withdraw. Persists across logout via the profile DataStore.
- **Inspect:** the chest ProximityPrompt now opens a UI instead of just a lid. Owner → `OpenChest{own=true, bag, chest}` (deposit/withdraw panel). Non-owner → sets thief's `InspectingUserId` attribute + `OpenChest{own=false, ownerName, items, topId}` (items = owner's owned weapons+shields+cosmetics PLUS chest). `Inventories` module (`GetSnapshot(userId)`) reads live profile for online owners, or a registered `SetOffline` snapshot for offline (the test dummy). In a shared server everyone you'd inspect is online, so live snapshots cover real play.
- **Steal (500 Robux):** copies the inspected owner's most valuable item the thief doesn't already own; victim keeps theirs. Value = gear price, cosmetics robux*50 (`GameData.ItemInfo`). `MonetizationService.doSteal` reads `InspectingUserId`. Live = `GameData.StealProductId` (currently 0 — **create the 500 R$ Developer Product and paste its id**). Studio test = `Remotes.RequestSteal` (gated `RunService:IsStudio()`, free). `GameData.StealRobux=500`.
- **UI:** `StarterPlayerScripts.ChestUI` LocalScript (separate ScreenGui, DisplayOrder 40). IMPORTANT FIX: GUI buttons auto-activated on show (gamepad auto-select) — fixed via `GuiService.AutoSelectGuiEnabled=false`, `Selectable=false` on buttons, and a 0.3s open-debounce on close/steal.
- New remotes: `OpenChest`, `ChestMove`, `RequestSteal`.
- **Test (Studio only, `_StudioDebugGrant`):** tester gets 1e9 gold, both cosmetics, spare LollipopHammer+CookieGuard (to test depositing). Dummy "ordek" (ORDEK_ID=31415926) owns a vacant base (Base_2 in tests) via OwnerUserId/OwnerName attrs + `Inventories.SetOffline` with every item — inspect/steal target. Studio purchases are free so steal is unlimited.

Related: [[project_hub_systems]] [[project_rebirth_shop_build]] [[project_town_hub_build]]
