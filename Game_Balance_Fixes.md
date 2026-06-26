# Candy MMORPG — Implementation Fixes (Biome 1: The Forest)

**Companion to `Game_Balance_Theorycraft.md`.** Concrete, code-level fixes for the **Forest biome / Rebirth 1–10 only.** Mob tiers, deep gear, and global damage scaling are intentionally **deferred to Biome 2** (see the appendix). Everything here drops into your existing scripts (`GameData`, `ForestConfig`, `ForestMobs`, `ProfileService`, `PlayerInit`, weapon `ClientAttack`, `ForestBoss`).

> **Why no mob scaling in the forest:** the world is an 8-player shared instance (mob HP is global) **and** the forest is the starter biome — it doesn't need to scale. Power growth over R1–10 comes from gear + weapon upgrades + a little HP; the difficulty reset happens when you enter Biome 2. Keep the forest simple.

All values are starting points — tune against your target "minutes per rebirth."

---

## FIX 1 — Light HP growth (no damage scaling in the forest)

Just enough HP growth that the boss fight scales with prestige. **No global damage multiplier in Biome 1** — gear and upgrades carry power; scaling damage only makes 60-HP trash die faster.

**`GameData` — add:**
```lua
GameData.HPPerRebirth = 8   -- R10 -> 180 max HP (biome 1)
function GameData.PlayerMaxHP(rb) return GameData.PlayerBaseHP + GameData.HPPerRebirth*(rb or 0) end
```
**`PlayerInit.onCharacter`:**
```lua
local rb = Profile.GetRebirth(plr)
hum.MaxHealth = GameData.PlayerMaxHP(rb)
hum.Health    = hum.MaxHealth
```
**`ProfileService.DoRebirth` — after `p.rebirth += 1`, push new HP live:**
```lua
local ch  = plr.Character
local hum = ch and ch:FindFirstChildOfClass("Humanoid")
if hum then
  local frac = hum.Health/hum.MaxHealth
  hum.MaxHealth = GameData.PlayerMaxHP(p.rebirth)
  hum.Health    = hum.MaxHealth*frac     -- or full as a reward
end
```

---

## FIX 2 — Combat needs skill (the forest's biggest engagement fix)

### 2a. Active parry (timed block)
**`GameData` — add:** `GameData.ParryWindow = 0.25` and `GameData.ParryStun = 1.25`.

**`PlayerInit` PlayerBlock handler — stamp when the block began:**
```lua
Remotes.PlayerBlock.OnServerEvent:Connect(function(plr, state)
  local ch = plr.Character
  local hum = ch and ch:FindFirstChildOfClass("Humanoid")
  local blocking = (state==true) and hum~=nil and hum.Health>0
  plr:SetAttribute("Blocking", blocking)
  if blocking then plr:SetAttribute("BlockStart", os.clock()) end
end)
```
**`ForestMobs` mob-hits-player block — add the perfect-block branch:**
```lua
local blocked = tgt:GetAttribute("Blocking")==true
local bs = tgt:GetAttribute("BlockStart") or -1e9
local perfect = blocked and (now - bs) <= GameData.ParryWindow
local dmg
if perfect then
  dmg = 0
  mob:SetAttribute("StunUntil", now + GameData.ParryStun)   -- reward: stun the attacker
elseif blocked then
  dmg = math.max(1, math.floor(raw*(1-GameData.BlockReduction)+0.5))
else
  dmg = raw
end
if hum and dmg>0 then hum:TakeDamage(dmg) end
```
In the mob attack loop, skip mobs whose `StunUntil > now`. Early block = 75% cut (safe but passive); **on-reaction block within 0.25s = full negate + stun** (skill payoff).

### 2b. Telegraphed heavy attack
In the mob retaliation loop, sometimes wind up a dodgeable heavy hit instead of an instant bonk:
```lua
if math.random() < 0.30 and not mob:GetAttribute("Winding") then
  mob:SetAttribute("Winding", true)             -- client shows a wind-up telegraph on this attribute
  task.delay(0.6, function()
    mob:SetAttribute("Winding", false)
    -- re-check range at strike time; heavy = raw*2, dodgeable by stepping out, parryable
  end)
end
```
A small client LocalScript watches `Winding` and flashes a ring so players can step out or parry.

### 2c. Soft-aim instead of auto-target
**Weapon `ClientAttack` fallback target** — require the player to roughly face the mob:
```lua
local look = hrp.CFrame.LookVector
-- inside the nearest-candidate loop, before accepting a candidate:
local to = (m.PrimaryPart.Position - hrp.Position)
if to.Magnitude > 0 and look:Dot(to.Unit) < 0.5 then continue end   -- ~60° cone; ignore mobs behind you
```
Kid-friendly (no precise clicking) but rewards positioning.

### 2d. Per-color mob personalities (cheapest big win for the forest)
Add a behavior flag per camp in `ForestConfig.CAMPS` and branch in the mob loop. Same stat budget, four different threats:
- **RED (Cinder)** — `moveSpeed×1.3`, attacks more often. Aggressive rusher.
- **BLUE (Dewdrop)** — ranged spit projectile; close on it or dodge.
- **YELLOW (Sunpip)** — slightly tankier, slow, hits a bit harder.
- **GREEN (Mossling)** — splits into two weak slimes on death.
This is what makes the forest feel hand-crafted instead of four identical farms.

---

## FIX 3 — Rebirth 6–10 gear tail + gold sink (weapon upgrades)

Fills the empty back half of the biome with a repeatable goal, **without** adding new weapon tiers.

**`GameData` — add:**
```lua
GameData.Upgrade = { MaxLevel=10, DmgPerLvl=0.05, CostBase=250, CostGrowth=1.6 }
function GameData.UpgradeCost(level) return math.floor(GameData.Upgrade.CostBase * GameData.Upgrade.CostGrowth^(level or 0)) end
```
**`ProfileService`** — add `p.upgrades = {}` to the default profile (save/load it), and:
```lua
function M.UpgradeWeapon(plr, id)
  local p = profiles[plr]; if not p or not GameData.Weapons[id] or not p.weapons[id] then return false end
  local lvl = (p.upgrades[id] or 0)
  if lvl >= GameData.Upgrade.MaxLevel then return false, "Max level" end
  local cost = GameData.UpgradeCost(lvl)
  if (p.gold or 0) < cost then return false, "Not enough gold!" end
  p.gold -= cost; p.upgrades[id] = lvl+1
  plr:SetAttribute("Gold", p.gold); M.ComputeStats(plr); M.Sync(plr)
  return true
end
```
**`ComputeStats`** — fold the upgrade bonus into the equipped weapon:
```lua
if w then
  local ulvl = (p.upgrades[p.equippedWeapon] or 0)
  atk = atk + math.floor((w.attackPower or 0) * (1 + GameData.Upgrade.DmgPerLvl*ulvl))
  arm = arm + (w.armor or 0)
end
```
A full +10 weapon is +50% damage (~25,000 gold cumulative) — a deep optional sink that keeps gold meaningful through R6–10. Add a couple of **gold consumables** (HP Gummy heal 50%, Lucky Lollipop drop×2 for 10 min) as secondary sinks.

---

## FIX 4 — Boss: the forest's climax/gate, not a crystal shortcut

**`GameData` — retune:**
```lua
GameData.BossGoldReward     = 2500
GameData.BossCrystalReward   = 2     -- was 5: still a great haul, no longer skips the grind
GameData.BossPlayerCooldown  = 300   -- per-player 5-min reward lockout (anti chain-farm)
```
**`ForestBoss` on-defeat** — gate rewards by a per-player timestamp:
```lua
local last = plr:GetAttribute("BossRewardAt") or -1e9
if os.clock() - last >= GameData.BossPlayerCooldown then
  plr:SetAttribute("BossRewardAt", os.clock())
  Profile.AddGold(plr, GameData.BossGoldReward)
  for _,id in ipairs(GameData.CrystalOrder) do Profile.AddCrystal(plr, id, GameData.BossCrystalReward) end
end
```
Keep the **2,600 HP** so weapon tier matters here (Scepter ≈50 hits vs Chocolate ≈72 hits) — the boss is the forest's DPS proving ground and the Rebirth 9→10 graduation gate into Biome 2.

---

## FIX 5 — Drop variance + gentle reward ramp

**`GameData` — add:**
```lua
GameData.GoldRebirthBonus = 0.06   -- +6% gold/kill per rebirth (R10 -> ~24/kill)
GameData.PityThreshold    = 25     -- guaranteed crystal after 25 dry kills
```
**`ForestMobs.die` — replace the flat reward block:**
```lua
if killer and killer.Parent then
  local rb   = Profile.GetRebirth(killer)
  local gold = math.floor(GameData.ForestGoldPerKill * (1 + GameData.GoldRebirthBonus*rb))
  Profile.AddGold(killer, gold)
  Remotes.Toast:FireClient(killer, {text="+"..gold.." Gold", color=Color3.fromRGB(255,215,0)})

  local pity = (killer:GetAttribute("CrystalPity") or 0) + 1
  if math.random() < GameData.DropChance or pity >= GameData.PityThreshold then
    spawnCrystalDrop(mob:GetAttribute("CrystalDrop"), pos)
    killer:SetAttribute("CrystalPity", 0)
  else
    killer:SetAttribute("CrystalPity", pity)
  end
end
```
Pity caps the worst dry streak at 25 kills; the small gold bonus eases the late-forest rebirth ramp without inflating the economy.

---

## FIX 6 — Fix the armor formula (keep original shop tiers)

**Keep the four forest weapons and three shields and their prices — they're roughly gold-balanced over R1–10.** The one required combat fix: subtractive armor floors mob damage to 1 at Honeycomb. Replace it with diminishing returns so armor scales smoothly and never grants immunity.

**`ForestMobs` (and anywhere a player takes damage)** — replace `max(1, mobATK − playerArmor)` with a shared `GameData.Mitigate` (constant `ArmorK = 16`):
```lua
function GameData.Mitigate(dmg, armor)
  local a = armor or 0
  return math.max(1, math.floor(dmg * (1 - a/(a + GameData.ArmorK)) + 0.5))
end
-- usage: local raw = GameData.Mitigate(mob:GetAttribute("AttackPower") or 15, arm)
```
Mitigation at forest shields (K=16): Gummy 3 → −16%, Cookie 8 → −33%, Honeycomb 16 → **−50%**. Honeycomb halves damage — strong, not invincible (it used to floor mobs to 1) — so block/parry and positioning still matter at the top of the biome.

> **IMPLEMENTED 2026-06-23** — every fix in this doc is now live in the place EXCEPT: telegraph VFX client script (2b) and per-color mob personalities (2d); the PvP gold-bet duel system (needs a new DuelService + player-vs-player damage, which doesn't exist yet); and monetization (needs real Roblox product/gamepass IDs from you). Weapon upgrades are wired through a new `UpgradeWeapon` remote with an **UP** button on weapon slots in the bag. Verified in a live playtest: R0 stats correct, +3 upgrade raised Candy Cane 12→14 ATK with correct gold cost, and a rebirth scaled MaxHP to 140 at R5.

---

## FIX 7 — Monetization ("pay-to-pass") + PvP fairness (game-wide)

### 7a. Products — a `MonetizationService` server script
```lua
local MPS = game:GetService("MarketplaceService")
local PRODUCTS = {   -- Developer Products (consumable): id -> grant
  [0000001] = function(plr) for _,id in ipairs(GameData.CrystalOrder) do Profile.AddCrystal(plr,id,15) end end, -- Crystal Pack
  [0000002] = function(plr) Profile.AddGold(plr, 25000) end,                                                    -- Gold Pouch
  [0000003] = function(plr) Profile.GrantWeapon(plr, "CrystalScepter") end,                                     -- Weapon Unlock Token
  [0000004] = function(plr) Profile.PayNextRebirth(plr) end,                                                    -- Rebirth Boost
}
MPS.ProcessReceipt = function(info)
  local plr = game.Players:GetPlayerByUserId(info.PlayerId)
  if not plr then return Enum.ProductPurchaseDecision.NotProcessedYet end
  local grant = PRODUCTS[info.ProductId]
  if grant then grant(plr); Profile.Save(plr); return Enum.ProductPurchaseDecision.PurchaseGranted end
  return Enum.ProductPurchaseDecision.NotProcessedYet
end

local VIP_PASS = 0000010
local function hasPass(plr,id) local ok,owns=pcall(function() return MPS:UserOwnsGamePassAsync(plr.UserId,id) end); return ok and owns end
-- in AddGold/on join: if hasPass(plr, VIP_PASS) then gold = math.floor(gold*1.25) end
```
Catalog & prices are in the theorycraft doc §6. Every paid weapon/shield must **also** be earnable with gold — payment buys the *skip*, never an exclusive.

### 7b. Keep PvP pay-to-win-proof (you have duels)
Normalize stats at duel start so spending and rebirth can never buy a win:
```lua
-- on duel start, snapshot then override both fighters:
plr:SetAttribute("PvP_SavedATK", plr:GetAttribute("AttackPower"))
plr:SetAttribute("AttackPower", GameData.PvP.FixedAttack)   -- e.g. 40
plr:SetAttribute("Armor",       GameData.PvP.FixedArmor)    -- e.g. 8
hum.MaxHealth = GameData.PvP.FixedHP                        -- e.g. 150
-- on duel end, restore the saved values + recompute
```
Money and grind speed up PvE; cosmetics flex everywhere; duels stay pure skill — fair for an 8–13 audience.

---

## Suggested rollout order (forest)
1. **Armor formula (Fix 6)** + **light HP growth (Fix 1)** — small, foundational.
2. **Pity + gold ramp (Fix 5)** — smooths the grind.
3. **Skill layer (Fix 2)** — parry, telegraph, soft-aim, per-color behaviors. Start with mob personalities (2d).
4. **Weapon upgrades + consumables (Fix 3)** — fills the R6–10 tail / gold sink.
5. **Boss retune (Fix 4)** — climax & graduation gate.
6. **Monetization + PvP normalization (Fix 7)** — last, on top of a loop that's already fun for free.

Give me your target **minutes-per-rebirth** and whether PvP is competitive or casual, and I'll lock exact multipliers and implement these in Studio in this order.

---

## Appendix — Deferred to Biome 2+
Out of scope for the forest, saved for when you build harder biomes:
- **Mob tier system** (per-mob HP/ATK/reward multipliers, elites).
- **Deep weapon/shield tiers** (Gobstopper Maul, Rock-Candy Greatsword, Peppermint Bulwark, Caramel Aegis), tuned to a geared R10 baseline.
- **Global damage multiplier from rebirth** — introduce with Biome 2's tougher mobs so both scale together.
- **Warden Core / boss-gated top-tier currency** — a Biome 2 chase hook.
