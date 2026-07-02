---
name: game-balance-theorycraft
description: "Biome-1 balance theorycraft doc — full math analysis of Forest economy, combat, shop, monetization, and open questions (2026-06-23)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: adfcd13c-d2eb-4817-96a5-6f94b7b484c4
---

# Candy MMORPG — Balance Theorycraft (Biome 1: The Forest)

**Date:** 2026-06-23
**Scope:** **Biome 1 only — The Emerald Vale (Rebirth 1–10).** Harder mobs, mob tiers, and deeper gear belong to Biome 2+ and are listed in the "Deferred to Biome 2" appendix, not analyzed here.
**Goal:** MMORPG feel within the first biome (gearing up + getting richer toward the boss/graduation), skill-based-but-fair mob combat, sensible shop prices, and an ethical "pay-to-pass" acceleration layer.

All current numbers are pulled from the live scripts. Proposed values are flagged **[PROPOSED]**.

---

## 1. The current math, exactly as coded

### Stat model
A player's power is **only** the equipped weapon's `attackPower` plus the equipped shield's `armor`. Rebirth grants **no stat** — it is a prestige counter that gates which gear the shop will sell.

```
AttackPower = equippedWeapon.attackPower
Armor       = equippedShield.armor
MaxHP       = 100 (fixed)
```

### Damage formulas
```
Player → mob:  dmg = max(1, AttackPower − mobArmor)          -- mob armor = 2
Mob → player:  raw = max(1, mobAttackPower − playerArmor)    -- mob ATK = 15
               blocked = max(1, round(raw × 0.25))           -- block = −75%
```
Swing cooldown is **0.45s** client-side (server floor 0.28s) → ~2.2 hits/sec.
A player can be damaged **at most once every 0.45s total** (anti-one-shot gate), regardless of how many mobs surround them. Each mob swings every ~2.0s within 8.5 studs.

### Time-to-kill (basic slime, 60 HP, armor 2)
| Weapon | ATK | dmg/hit | hits | TTK @0.45s |
|---|---|---|---|---|
| Candy Cane | 12 | 10 | 6 | ~2.7s |
| Lollipop Hammer | 22 | 20 | 3 | ~1.35s |
| Chocolate Blade | 38 | 36 | 2 | ~0.9s |
| Crystal Scepter | 60 | 58 | **2** | ~0.9s |

### Incoming damage to player (mob ATK 15)
| Shield | armor | unblocked | blocked |
|---|---|---|---|
| Gummy | 3 | 12 | 3 |
| Cookie | 8 | 7 | 2 |
| Honeycomb | 16 | **1** | 1 |

### Forest economy
- **15 gold/kill.** **5% crystal drop** per kill → ~**20 kills per crystal** of a camp's color (each camp = one color).
- Rebirth N costs **N of *each* of the 4 crystals + 1000·N gold.** Forest covers **rebirth 1–10**; the boss gates 9→10.
- Boss (Verdant Warden, 2600 HP, armor 8, melee 22 / slam 40) drops **2500 gold + 5 of each crystal**, respawns every 45s.
- Trophy passive income: 5 gold/min, banks up to 2400 (8h).

---

## 2. The Biome-1 economy, worked out (Rebirth 1–10)

Crystals are the gate; each color needs cumulative 1+2+…+10 = **55 crystals** → ~20 kills each → **~1,100 kills per camp, ~4,400 kills total.** At a respawn-limited ~12 kills/min that's **~5–7 hours to reach Rebirth 10.**

| Quantity | Value over R1–10 |
|---|---|
| Total kills (crystal-bound) | ~4,400 |
| Gold earned (×15) | ~66,000 |
| Gold spent on rebirths (Σ 1000·N) | 55,000 |
| Gold surplus before shop | ~11,000 |
| Cost of all original gear* | ~15,950 |

\*Lollipop 750 + Chocolate 3,000 + Scepter 9,000 + Cookie 600 + Honeycomb 2,600.

**Key correction from the full-game pass:** scoped to the forest, **gold and crystals are roughly balanced** — the ~11k surplus doesn't quite cover all gear, so a player grinds a little extra for the gear they want. Gold is *not* meaningless here; the "drowning in gold" problem was a Rebirth-10+ artifact. Good news: the forest's original prices basically work.

**Per-rebirth ramp** (marginal kills for rebirth N ≈ 80·N): rebirth 1 ≈ 80 kills (~7 min), rebirth 5 ≈ 400 kills (~33 min), rebirth 10 ≈ 800 kills (~67 min). The later forest rebirths take ~10× the first — steep but survivable across only 10 levels. Pity protection and a small gold bonus (see §4) smooth it.

---

## 3. The real Biome-1 problems

### A. The Rebirth 6–10 gear tail
The last meaningful purchase is the **Crystal Scepter (R5)** and **Honeycomb (R3)**. Rebirths 6–10 currently add **no new power** — you're just accumulating crystals for the boss/graduation. That's five empty levels at the end of the biome. *(Fix: a weapon-upgrade gold sink, not more weapon tiers — §5.)*

### B. High weapons are overkill on trash; the boss is their only proving ground
Chocolate (38) and Scepter (60) both **2-hit** a 60-HP slime — identical in practice. The difference only appears against the **boss**. So the boss must carry the weight of justifying every purchase past Chocolate: keep its 2,600 HP so Scepter (50 hits) genuinely out-DPS-es Chocolate (~72 hits), making the boss the forest's DPS test and the reason to gear up.

### C. Combat requires almost no skill
Click auto-targets the nearest mob; basic mobs deal ~6 dmg per 2s while you kill them in <3s; blocking is never necessary. A player can faceroll the whole biome. *(Fix: parry, telegraphs, soft-aim, per-color mob behaviors — Fixes doc §2.)*

### D. Broken armor + drop variance
Subtractive armor makes **Honeycomb (16) reduce a 15-ATK mob to 1 damage** — effective invincibility in the back half of the biome, erasing any threat. And 5% drop is swingy (60+ dry kills happen). *(Fix: hybrid armor formula + 25-kill pity — Fixes doc §6, §5.)*

### E. The boss is a crystal shortcut
One ~25s boss kill gives **5 of each crystal = ~400 camp kills' worth**, respawning every 45s — anyone who can farm it skips most of the intended forest grind. *(Fix: drop to 2 each + per-player cooldown; keep it the climactic 9→10 gate — Fixes doc §4.)*

---

## 4. Recommended direction for Biome 1

Keep the forest **simple and self-contained.** Power growth across R1–10 comes from four sources that already exist or are cheap to add — **no global stat scaling and no mob tiers needed here:**

1. **Weapon ladder** — Candy → Lollipop → Chocolate → Scepter (R0–R5).
2. **Shield ladder** — Gummy → Cookie → Honeycomb (R0–R3).
3. **[PROPOSED] Weapon upgrade levels** — a repeatable gold sink (+5%/level) that fills the R6–10 tail.
4. **[PROPOSED] Light HP growth** — `MaxHP = 100 + 8·rebirth` (R10 → 180) purely so the boss fight scales with prestige. No damage multiplier in the forest.

**[PROPOSED] Smooth the reward ramp** so late-forest rebirths don't stall:
```
Gold/kill   = round(15 · (1 + 0.06 · rebirth))   -- R10 → ~24/kill
Drop chance = 0.05, with a guaranteed crystal every 25 dry kills (pity)
```
The "you feel powerful" payoff is deliberately saved for **stepping through the portal into Biome 2**, which you'll tune around a geared Rebirth-10 player.

---

## 5. Shop (Biome 1)

**Keep the original four weapons and three shields and their prices — §2 shows they're roughly gold-balanced over R1–10.** Do **not** add high-cost tiers to the forest (they'd break that balance and belong to Biome 2).

Fill the R6–10 tail with **weapon upgrade levels** instead:
- +5% weapon damage per level, up to +50% at level 10.
- Cost `= 250 · 1.6^level` (≈250 → ~25,000 cumulative for a full +10), a deep, optional gold sink that keeps gold meaningful and gives R6–10 a goal.
- Also fix the armor math so Honeycomb scales instead of flooring mobs to 1 (Fixes doc §6).

A few **gold consumables** (HP Gummy, Lucky Lollipop) round out the sinks.

---

## 6. Monetization — "pay-to-pass" (game-wide, forest-scoped products)

Sell **time and convenience, not locked power.** Every paid item must also be earnable with gold/crystals — payment buys the *skip*. For an 8–13 audience, F2P must stay fully viable.

**GamePasses (permanent):** VIP (+25% gold, +1 daily crystal) ~499 R$ · Lucky Charm (drop ×2) ~399 R$ · Auto-Collect ~149 R$ · 2× Trophy Income ~199 R$ · cosmetic bundles 99–249 R$.

**Developer Products (consumable — the pay-to-pass core, priced for forest gear):** Crystal Pack (+15 each) ~99 R$ · Gold Pouch (+25,000) ~99 R$ · Weapon Unlock Token (grant a forest weapon, skip req+gold) ~149–299 R$ · Rebirth Boost (pay your next rebirth's cost) scaling · 2× Loot 1h ~79 R$.

**Guardrails:** no loot boxes; generous free daily crystals; and — critically, because you have duels — **normalize stats in PvP** so spending and rebirth can never buy a duel win (Fixes doc §7).

---

## 7. Open questions (these lock the exact numbers)
- **Target minutes-per-rebirth** for a free player in the forest? (Current ramp: ~7 min at R1 → ~67 min at R10. Pick a feel and I'll tune gold/drop/pity to it.)
- Is **PvP** competitive or casual? Decides the monetization guardrails.
- Forest power purely from **gear + upgrades**, or do you also want the light HP growth in §4? (I've assumed yes to HP, no to damage scaling.)

---

## Appendix — Deferred to Biome 2+
These were in the original full-game analysis and are intentionally **out of scope for the forest**:
- **Mob tier system** (HP/ATK/reward multipliers, elites) — Biome 2 is where difficulty scales.
- **Deep weapon/shield tiers** (Gobstopper Maul, Rock-Candy Greatsword, Peppermint Bulwark, Caramel Aegis) — these are Biome 2 shop items, tuned to a geared R10 baseline.
- **Global damage multiplier from rebirth** — introduce alongside Biome 2's tougher mobs so the two scale together.
- **Warden Core / boss-gated top-tier currency** — a Biome 2 chase hook.
