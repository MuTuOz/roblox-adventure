---
name: project-inventory-ui-icons
description: "ForestInventoryUI HUD tweaks — gold pill moved to top-center, item gear icons are stylized vector mini-pictures"
metadata: 
  node_type: memory
  type: project
  originSessionId: 4b3aab28-3ee4-407e-b7db-bc11dd243ebc
---

ForestInventoryUI (StarterPlayer.StarterPlayerScripts, cloned live into Players.<name>.PlayerScripts) UI changes, 2026-06-24:

- HUD gold pill moved from top-left to **top-center** (AnchorPoint 0.5,0; Position 0.5,0,0,8) so the Roblox topbar/menu button no longer covers it.
- Added `makeItemIcon(parent, id, data, kind, sizePx)` helper (above `gearSlot`) that draws a distinct stylized emblem per item from nested Frames via a `px(...)` shape helper — no image assets exist for items, only `color` + shield `visual` blocks in [[project_rebirth_shop_build]] GameData. Per-id art: CandyCane (striped cane), LollipopHammer (pink swirl), ChocolateBlade (sword), CrystalScepter (gem scepter); shields draw a crest from `visual.face/rim/style/emblemColor`; cosmetics get a sparkle; fallback gem.
- `gearSlot` (BAG, incl. equipped item) and shop catalog rows both call `makeItemIcon` instead of the old flat colored circle/square. BAG icon 38px, SHOP icon 52px.

Also disabled Roblox's default Backpack/hotbar GUI (`StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.Backpack,false)` next to the Health one at top of script). Reason: the held-weapon hotbar slot showed a blank gray box because `Tool.TextureId` is empty on the `ServerStorage.Weapons.<name>` templates (cloned by PlayerInit.giveWeapon, see [[project_weapon_swap_fix]]), and TextureId can ONLY take an uploaded image asset — it cannot render frame-based vector icons. No item images exist in the project, so user chose to hide the hotbar entirely; BAG menu is now the sole gear UI. Weapon still equips/works.

Verified in playtest: compiles clean (only the usual DataStore StudioAccessToApisNotAllowed warning), icons render in both BAG and SHOP, gold centered, hotbar slot gone. See [[project_hub_systems]] for related HUD/monetization work.

2026-06-24 follow-up: Rebuilt the **Lollipop Hammer 3D model** (ServerStorage.Weapons."Lollipop Hammer") to match the round-lollipop icon. Was a candy-cane shape (brown Handle stick + 5 pink cube segments climbing it + 8 pink balls forming a hook curl, all WeldConstraint'd to Handle). Now: kept Handle (0.42x3.4x0.42 brown stick) + scripts (ClientAttack, ShieldRig) + Grip (unchanged) + RequiresHandle; deleted all old candy parts/welds; added LollipopHead (Cylinder, Size (0.5,2.2,2.2) so round faces point along handle local X / sideways, NO rotation — a flat disc), 4 concentric Swirl cylinders (white/deep-pink, nudged along +X), and a StickCap ball at the bottom (local y=-1.7). Reattached each with a fresh WeldConstraint(Part0=Handle). Gotcha learned: Roblox Cylinder's circular faces are on its LOCAL X axis; do NOT rotate it to face sideways. Icon reverted to original round-lollipop (disc+swirl+stick). If user wants a 3D-round head instead of flat disc, swap LollipopHead to a Ball.
