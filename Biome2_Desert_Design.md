# Biome 2 — "The Gilded Wastes" (Desert) — Design Spec

**Status:** Approved economy/combat design. Build in progress.
**Date:** 2026-06-26
**Band:** Rebirth 10 → 20. Entry gated at **Rebirth 10+** (mirrors Forest's reb-9 boss gate as the prior step).
**Theme:** Desert. Dominant colour **sand yellow**. Big map, mirrors Forest scale.
**Isolation requirement:** Not visible from Hub / Forest / Arena, and none of those visible from here (distance + bounding walls + zone fog + skybox, same approach as `ForestBounds`).

All numbers below are **calibrated against the live Forest economy** (`GameData` + `ForestConfig`) and verified to run **~1.5× the Forest's grind length** (Forest reb 0→10 ≈ 2.3 hrs active; Desert reb 10→20 ≈ 3.4 hrs maxed / 4.0 hrs median active — i.e. real-world *weeks* of casual play).

---

## 1. The four tuning answers

| # | Question | Value |
|---|----------|-------|
| 1 | Gold for reb 10→11 | **4,500 gold** (schedule = `local_level × 4,500`, so 4.5k → 9k → … → 45k) |
| 2 | Mob base stats (at reb 10) | **HP 180, Attack 24, Armor 3** (HP +6%/level, ATK +3%/level over the band) |
| 3 | Crystal drop chance | **4.0% → 6.5%** (base 4%, +0.25%/level), **pity 30** dry kills |
| 4 | Gold per kill | **2.5× Forest rate** → 62/kill at reb 10, rising to 88 at reb 20 |

How the 1.5× was achieved (per direction): **crystal requirement unchanged**, slowdown comes entirely from **higher gold cost (×1.5)** + **rarer crystals** (lower drop %, higher pity). No change to how many crystals a rebirth needs.

---

## 2. Mobs — stats & scaling

Mobs scale to the **player's** rebirth and freeze at the band edges (identical system to Forest, via `GameData.MobLocalLevel`). Base values are the reb-10 floor ("weakest at reb 10"); the band end is "strongest at reb 20".

- **Base (local level 0 = reb 10):** Health 180, AttackPower 24, Armor 3, Level 11
- **HP scaling:** `180 × 1.06^L` (L = local level 0–10) → 180 at reb 10, 322 at reb 20
- **ATK scaling:** `24 × 1.03^L` → 24 at reb 10, 32 at reb 20
- **Armor:** flat 3

| Player reb | Mob HP | Mob ATK | Hits to kill (maxed / median Scepter) | Gold/kill | Crystal drop |
|-----------|-------|--------|------|-----------|--------------|
| 10 | 180 | 24 | 3 / 4 | 62 | 4.0% |
| 13 | 214 | 26 | 3 / 4 | 70 | 4.75% |
| 15 | 241 | 28 | 4 / 5 | 75 | 5.25% |
| 17 | 271 | 30 | 4 / 5 | 80 | 5.75% |
| 20 | 322 | 32 | 5 / 6 | 88 | 6.5% |

**Calibration rationale:** No new weapons exist for reb 10–20 yet (deferred — see §7), so player offense is effectively flat (capped Crystal Scepter ≈ 76 dmg/hit after mob armor). HP growth is therefore gentler than Forest (6%/level vs 10%) so kill-times creep modestly (3→5 hits) rather than ballooning. Mob damage stays far under player EHP growth (player HP 180→260, 50% mitigation from Honeycomb shield) — combat never becomes lethal, per the existing kid-friendly philosophy.

**Seam check:** reb-10 desert mob (180 HP / 24 ATK) is a clean step up from the last Forest mob (156 HP / 20 ATK) — tougher but still a 3-hit floor, not a wall.

---

## 3. Gold economy

- **Gold per kill:** `GameData.GoldPerKill(reb) × BiomeGoldMult`, where `GoldPerKill(reb) = 15 + reb` (existing) and **`DesertGoldMult = 2.5`**.
- **Rebirth gold cost:** `local_level × 4,500` (local_level = targetLevel − 10).
- **Total gold sunk reb 10→20:** 4,500 × (1+2+…+10) = **247,500 gold**.

**Farm incentive (the point of 2.5×):** Today every biome pays the same gold/kill, so there's no reason to leave Forest. With the multiplier, a reb-15 player earns **≈ 69,000 gold/hr in Desert vs ≈ 40,000/hr back-farming Forest (+73%)** — Desert becomes the clear gold farm, and it's also where rebirth crystals live. Keeps a clean biome ladder: Forest 1× / Desert 2.5× / Hell ~6×.

---

## 4. Crystals

Four **new** desert crystals (distinct hues, readable against sand-yellow). Each camp drops only its own type.

| Camp | Crystal | Colour (RGB) | Mob | Flavour |
|------|---------|--------------|-----|---------|
| Sunbask Flats | **Citrine** | 240,200,70 (gold-yellow) | **Sandling** (sand-blob golem) | intro / baseline |
| Ember Mesa | **Carnelian** | 210,95,45 (burnt orange) | **Dune Scarab** (armoured beetle) | tanky (optional +15% HP) |
| Oasis Hollow | **Turquoise** | 45,190,180 (teal) | **Cactite** (bouncing cactus) | spiky |
| Mirage Bluffs | **Amethyst** | 160,90,200 (violet) | **Sand Wisp** (dust imp) | glass cannon (optional −15% HP / +15% ATK) |

- **Drop chance:** base 4%, +0.25%/local level → 4.0% (reb 10) to 6.5% (reb 20).
- **Pity:** guaranteed crystal after **30** dry kills (raised from Forest's 25 to keep the lower drop % from collapsing onto pity).
- **Rebirth requirement (unchanged):** `local_level` of **each** of the 4 desert crystals → 1,2,…,10 each; **55 of each type** across the biome (same volume as Forest).
- Per-camp flavour modifiers in the table are optional; base build uses uniform stats like Forest.

---

## 5. Pacing (verified)

Gold is earned *while* crystal-farming, so time-to-rebirth = `max(gold_kills, crystal_kills) × time_per_kill`. The two are co-balanced at every level (neither trivializes the other).

| Rebirth step | Gold cost | Crystals (each) | ~Active minutes (maxed/median) |
|------|-----------|-----------------|-------|
| 10→11 | 4,500 | 1 | ~4 / 5 |
| 15→16 | 22,500 | 6 | ~23 / 27 |
| 19→20 | 45,000 | 10 | ~40 / 47 |
| **Total 10→20** | **247,500** | **55 each** | **~3.4 hrs / 4.0 hrs** |

"Active" = continuous optimized combat with maxed gear. Real calendar time is far longer (AFK, PvP, travel, RNG, under-gear, short kid sessions). ≈ **1.51× the Forest** by the same model.

---

## 6. Desert treasure chest (separate instance)

Reuse the `ChestSpawn` / chest system with a desert config block:

- **GoldBase 375** (= Forest 150 × 2.5) → reward `375 × (1 + reb)` ≈ 6,000 gold at reb 15 (~one level's worth).
- Crystal reward: 2–4 of a random **desert** crystal.
- Same 3-hour offline-aware slot, same buffs (Gold Rush +15% gold, Lucky ×2 drop), 12-min duration.
- Own spawn footprint near the desert arrival point; rolled independently of the Forest chest.

---

## 7. Implementation plan & notes

**Data layer (first build step):**
- `ReplicatedStorage.DesertConfig` — new module mirroring `ForestConfig` (CENTER, radii, 4 camps, mob base, boss TBD, desert lighting/fog, ARRIVAL/RETURN pads). Proposed CENTER **(7000, 0, 0)** — far from Hub (750), Forest (−3500), Arena (3000). Adjustable.
- `GameData` additive constants: `DesertGoldMult = 2.5`; desert `Crystals` set + order; desert mob scale reuses existing `MobScale` (HP 6%? — note: Forest uses 10%; desert needs its own `HPPerLevel`, so add per-biome scale or a Desert override of 6%); desert drop params (base 4%, +0.25%/L, pity 30); desert chest block; desert rebirth `GoldPerLevel = 4500`.
- **Biome-aware `RebirthCost`** (the critical change): compute `local_level = targetLevel − biomeStart` for the **count**, and select the crystal **set** + gold-per-level for the band the target falls in. Forest (1–10) behaviour must be unchanged.
- `ProfileService`: add storage for the 4 desert crystals; `DoRebirth` consumes the correct biome's crystals.

**World layer (staged, after data + review):**
- Terrain: big desert basin at CENTER, sand-yellow palette, dunes/mesas; flat camp pads.
- 4 camps with crystal nodes + mob spawners (DesertMobs script mirroring ForestMobs with desert stats/drops).
- Portal: gate entry at **reb 10+**, wired into existing `HubPortals` (connect to the existing portal system; confirm a free portal slot or add one). Arrival + return pads.
- **Occlusion:** bounding walls + zone fog + skybox + distance so biomes can't see each other (extend the `ForestBounds` approach to `DesertBounds`).
- Client: desert minimap, zone lighting, inventory UI rows for the 4 new crystals.
- Desert treasure chest spawner.

**Deferred (per direction — revisit after Biome 3 / Hell):**
- Interim mid-biome weapon + shield. For now reb 10–20 runs on capped Forest gear; the gentle 6%/level HP curve is tuned to make that feel fine.

---

## 8. Open items to confirm during build
- Desert CENTER coordinate (proposed 7000,0,0) and exact map size.
- Which Hub portal connects to Desert (free slot vs new portal).
- Whether to ship per-camp stat flavour or uniform stats for v1.
- Desert boss (analogous to Verdant Warden, gating reb 19→20?) — not yet designed; can mirror Forest boss later.
