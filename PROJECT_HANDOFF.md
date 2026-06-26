# Roblox MMORPG — Developer Handoff

**Studio (Despair Studio) · Project: shared-world MMORPG for ages 7–13**
Place: `v1` · Last updated: 2026-06-21

This doc is a self-contained brief so a new developer can understand the design and pick up the build. Part 1 is the game design. Part 2 is the current state of the build in Studio (what exists, where, and how it works).

---

# PART 1 — Game Design

## Genre & audience
- MMORPG with PvE and PvP. Target audience: kids ~7–13. Visual style: bright, stylized low-poly fantasy — colorful but not babyish.

## Server & world structure
- **8 players per server**, shared world.
- **Town Hub** — central social area: spawns, portals to biomes, and every player's base.
- **3 Biomes**, unlocked every 10 rebirths: 1) Forest, 2) Desert, 3) Hell (fire). Each biome has a **boss** at the end.
- **1 PvP Arena** — separate zone for 1v1 duels.

## Combat & stats
- Click-to-attack, **melee only**. Weapons are meant to look eye-catching (not basic swords). **No item loss on death** (PvE or PvP).
- Weapons = flat **Attack Power**. Armor = flat **damage reduction**. Mobs have Attack/Health/Armor. Players have **no base stats** — entirely gear-dependent. New-biome gear outscales old gear.

## Biome layout (per biome)
- **4 monster camps**, each dropping a specific rebirth material. Currency drops from all mobs. Boss at the end.
- **Biome 1 (Forest)** rebirth materials = 4 crystal colors: Camp1→Green, Camp2→Yellow, Camp3→Blue, Camp4→Red.
- Desert & Hell use their own distinct materials/items/gear (TBD).

## Rebirth system
- Every **10 rebirths** unlocks the next biome. You must **defeat the biome boss** to progress.
- Cost scales: rebirth **N** costs **N of each crystal + N×1000 currency** (e.g. Rebirth 1 = 1 of each + 1,000; Rebirth 10 = 10 of each + 10,000).
- Each rebirth unlocks new purchasable weapons/armor from the shop. Rebirth level is shown at the player's base (prestige).

## Base system
- Every player has a **base in the shared town**; others can visit it.
- Display **boss trophies** from kills → defeated bosses generate **passive currency income**. Rebirth level shown visually.

## PvP Arena
- 1v1 duels; level and gear score matter (not equalized). Optional **betting**: wager currency, winner takes all.

## Economy
- Currency from farming mobs in your current biome → spent on rebirth (sacrificed) and PvP bets. Passive income from base trophies.

---

# PART 2 — Current Build (Town Hub)

The Town Hub is the first area built. It's a prototype but fairly complete and polished.

## World map / coordinates
- Baseplate is 2048×2048, top at **y = 0**.
- **Town Hub** is centered at **(750, 0, 0)** — built clear of the Forest WIP.
- **Forest biome** content sits near the **origin (0,0,0)** (Trees, ForestProps, MonsterCamps: Bunny_Meadow, Frog_Pond, Chick_Coop, Piglet_Pen).
- The hub is fully enclosed by a circular **rampart wall at radius ~216** — portals are the only way out, and the wall hides the biomes from spawn.

## DataModel hierarchy
```
Workspace
└─ TownHub (Folder)
   ├─ Structure   → plaza discs, Monument (spire + 4 floating crystals), spawn pad
   ├─ Portals     → Portal_FOREST / Portal_DESERT / Portal_HELL / Portal_PvPARENA
   │                 (each portal's glowing plane child is "PortalSurface")
   ├─ Bases       → Base_1 … Base_8  (each = Mansion + Yard + TrophyDisplay + gate/sign/banner)
   ├─ Decor       → Paths, Lampposts, Trees, RingClusters, welcome arch
   └─ Boundary    → rampart wall segments, towers, merlons, hedge skirt, invisible anti-climb lip

ServerScriptService
├─ HubMonument   → animates the monument crystals (float + rotate)
├─ HubPortals    → portal ProximityPrompts; Forest portal teleports to the forest + return pad
├─ HubBounds     → fall-safety: teleports anyone below y=-20 back to spawn
└─ HubChests     → "Open" prompt on each mansion chest; tweens the lid open/closed

StarterPlayer/StarterPlayerScripts
└─ HubMonumentGlow → client: dims monument lights/bloom + fades the neon crystals by camera distance
                     (focal beacon from afar, soft up close)
```

## Layout & key features
- **Central Monument** — white spire ringed by 4 floating crystals in the rebirth colors (Red/Blue/Yellow/Green). Ties to the rebirth-crystal lore. Animated.
- **Spawn pad** — south edge of the plaza (~`750, 1, -76`); players face the monument and portals.
- **Portal gallery** (north terrace), left→right: **Forest** (open), **Desert** (locked — "Rebirth 10"), **Hell** (locked — "Rebirth 20"), **PvP Arena** (open). Locked ones are barred. Forest portal is functional (teleports to the forest; a "Return to Town" pad teleports back).
- **8 player bases** ring the plaza at radius **152**, angles `{130,170,210,250,290,330,10,50}` (north gap reserved for the portal avenue). Each base:
  - An **enterable 2-story mansion** (`Base_i.Mansion`): hollow ground floor, door opening, glass windows, chandelier interior light, columned portico + balcony, cupola crown.
  - A **treasure chest** inside (`Base_i.Mansion.Chest`, lid = `ChestLid`) with an "Open" prompt.
  - A **trophy display** (`TrophyDisplay`) — stylized "Forest Warden" boss bust under a glass dome (passive-income feature placeholder).
  - An **estate gate** with a **BillboardGui nametag** (`NameAnchor.NameTag`) and a **rebirth banner** (`RebirthAnchor`).
  - A dressed **yard** (`Yard`): walkway, hedges, trees, garden, lamps.
- **Lighting**: Future technology, warm daytime (ClockTime ~13), skybox, Atmosphere haze, mild Bloom.

## Conventions to keep
- **Per-base accent color** (Base 1→8): red, orange, amber, green, teal, blue, purple, pink — used on roofs, trim, banners, nametag stroke. Easy player identification.
- **Portal lock scheme**: Forest open, Desert = Rebirth 10, Hell = Rebirth 20, PvP open.
- Palette: warm sandstone/cream, gold trim, wood, with **Neon** only for glows. Build new structures relative to the hub center and reuse the `lookAt(pos, center)` local-frame pattern so things face inward.
- **Name/number labels should be BillboardGuis**, not flat SurfaceGuis mounted high (those vanish when viewed edge-on up close).

## Known next steps / TODO
- **Ownership**: assign `Base_i` to players on join (set `OwnerUserId` / `OwnerName`); show the real player name on the nametag and live rebirth count on the banner.
- **Trophies → income**: drive `TrophyDisplay` from which bosses the player has beaten; wire passive currency.
- **Biomes**: build Desert, Hell, and the PvP Arena. Decide whether biomes stay in this place (streamed in on portal use) or become **separate places via TeleportService** (cleanest for hiding + performance; needs a call on shared vs per-party biomes).
- **Core systems** not yet built: combat, mob camps + drops, rebirth flow, shop, currency, party system, betting.
- Minor polish: the marble mansion exterior reads a little bright; can be toned down.

---
*Generated as a handoff summary. The live source of truth is the `v1` Studio place.*
