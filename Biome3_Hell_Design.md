# Biome 3 — "The Inferno" (Hell) — Design Spec

**Status:** Approved economy/combat design. Build in progress.
**Date:** 2026-06-26
**Band:** Rebirth 20 → 30. Entry gated at **Rebirth 20+** (the hub's `Portal_HELL`, already stubbed/locked).
**Theme:** Hell. Dominant colour **red**. Big map, mirrors Forest/Desert scale.
**Location:** Sunken realm at **(0, −3000, 0)** — 3,000 studs below every surface biome, so isolation is guaranteed (depth + ring walls + heavy red fog). No other biome/hub/arena visible from here or vice-versa.

Calibrated against the live Forest + Desert economy and **verified to run exactly 2.0× the Forest grind** (Forest reb 0→10 ≈ 2.3 hrs active; Hell reb 20→30 ≈ **4.56 hrs** active — the longest, hardest biome, as requested).

---

## 1. The four tuning answers

| # | Question | Value |
|---|----------|-------|
| 1 | Gold for reb 20→21 | **13,000 gold** (schedule = `local_level × 13,000` → 130,000 at reb 29→30; 715,000 total) |
| 2 | Mob base stats (at reb 20) | **HP 340, Attack 36, Armor 5** (HP +3.5%/level → 480; ATK +3.5%/level → 51) |
| 3 | Crystal drop chance | **5% → 7%** (base 5%, +0.2%/level), **pity 30** dry kills |
| 4 | Gold per kill | **6× Forest rate** → 216 at reb 21, rising to 270 at reb 30 |

Crystal requirement is unchanged (1…10 of each per level, 55 per type). Gold cost is set at the **co-balance point** so gold is a serious sink you must farm for, while crystals remain the marginal final gate.

---

## 2. Gear is deferred (important calibration note)

Per direction, **no new weapons/shields are built yet** — players run the capped Crystal Scepter (~90 dmg/hit, ~69 after Hell mob armor) through the whole band. With flat DPS, steep mob HP would make late kills a 10-hit slog, so HP growth is held to **3.5%/level** (vs the 7% I'd use *with* a gear ladder). Result: kills stay **5 → 7 hits**, and the 2× length comes from tankier mobs + a heavy crystal/gold grind, not from unfun bullet-sponge fights.

**When the Hell gear ladder is added later**, revisit: raise mob HP growth toward 6–7% and the band can carry leaner, rarer-crystal mobs.

---

## 3. Mob curve & pacing (verified 2.00× forest)

Mobs scale to the player's rebirth, frozen at the band edges (same system as Forest/Desert, `GameData.Biomes[2]`).

| Player reb | Mob HP | Mob ATK | Hits/kill | Gold/kill | Drop % | Gold cost | gold_k / cry_k |
|-----------|-------|--------|-----------|-----------|--------|-----------|-----------|
| 21 | 352 | 37 | 6 | 216 | 5.2% | 13,000 | 60 / 61 |
| 25 | 404 | 43 | 6 | 240 | 6.0% | 65,000 | 271 / 281 |
| 30 | 480 | 51 | 7 | 270 | 7.0% | 130,000 | 481 / 507 |

`gold_k ≈ cry_k` at every level → gold and crystals are co-balanced (crystals gate by ~5%). **Total ≈ 4.56 hrs active** (≈ a month-plus of casual play). Seam: a reb-20 Hell mob (340 HP, Armor 5, 5-hit floor) is a clear step up from the desert's reb-20 ceiling (322 HP, Armor 3).

---

## 4. Why Hell becomes the farm

> **Hell ≈ 169,000 gold/hr** vs Desert 92,000 vs Forest 53,000 (back-farm). The 6× multiplier on tanky mobs makes Hell ~1.8× the desert's rate — the clear place to bank the big gold the new gear will cost. Biome ladder stays clean: 1× / 2.5× / 6×.

---

## 5. Crystals, camps & theme — "The Inferno"

Four distinct hues that read against a red world:

| Camp | Crystal | Colour (RGB) | Mob (cute-menacing demon) | Flavour |
|------|---------|--------------|------|--------|
| Cinder Hollow | **Magma** | 255,110,30 (orange) | **Emberling** (lava blob) | intro / baseline |
| Sulfur Pits | **Brimstone** | 235,215,70 (yellow) | **Gargling** (stone gargoyle) | tanky |
| Wailing Hollow | **Soulfire** | 90,205,235 (cyan) | **Wisp Wraith** (soul flame) | glass cannon |
| Demon's Maw | **Voidstone** | 170,80,220 (violet) | **Impling** (horned imp) | aggressive |

---

## 6. Treasure chest & minimap

- **Hell chest** (`GameData.HellChest`): separate slot tracker + remotes (like the desert chest), **GoldBase 900** (×6) → reward `900 × (1 + rebirth)`, **3–5 Hell crystals**, same Gold-Rush/Lucky buffs, own spawn near the arrival.
- **Minimap**: red palette, title "THE INFERNO", four camp markers + paths + player arrow, shown only inside the zone.

---

## 7. Implementation plan

**Data layer:** `HellConfig` module (mirrors DesertConfig; BIOME.Index=2, RebirthStart=20, CENTER (0,−3000,0)); `GameData` additive (4 Hell crystals, `HellCrystalOrder`, extend `AllCrystalOrder` to 12, `Biomes[2]` band metadata, `HellChest`); `ProfileService` gets `hellChestFoundSlot` (crystals auto-slot via `AllCrystalOrder`). `RebirthCost`/`BiomeForLevel` already generalize — `Biomes[2]` makes reb 21→30 cost Hell crystals + 13k/level.

**World layer:** sunken basin at (0,−3000,0) — charred ground, lava pools, obsidian spires, dark volcanic ring walls + heavy red fog (isolation), arrival dais, return pad. 4 camps with crystal nodes + `HellMob`-tagged mobs. `HellMobs` server script (copy of DesertMobs, biomeIdx=2, tag "HellMob"). `Portal_HELL` gated at reb 20 → arrival; return pad. Hell zone lighting (dark red atmosphere, ember particles). Add "HellMob" to the 4 weapon ClientAttack scripts. Hell chest (server+client) + minimap.

**Deferred:** Hell gear ladder (weapons/shields), Hell boss (gate reb 29→30?), per-camp stat flavour.
