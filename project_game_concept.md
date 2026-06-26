---
name: project-game-concept
description: Core design document for the Roblox MMORPG game being built by Duru and friend
metadata: 
  node_type: memory
  type: project
  originSessionId: 96d29ecd-db9e-4abd-b14a-829221d42b49
---

# Roblox MMORPG — Game Concept

## Genre & Target Audience
- MMORPG with PvE and PvP modes
- Target audience: children aged 8–13
- Visual style: TBD (cartoony/colorful likely fits audience)

## Server Structure
- 8 players per server
- Shared open world for farming
- Shared town hub: player spawns, portals to biomes, and player bases

## World Layout
- **Town Hub**: central area with spawns, portals, and bases
- **3 Biomes**: unlocked every 10 rebirths
  1. Grass/Forest
  2. Desert
  3. Hell (fire theme)
- **1 PvP Arena**: separate zone for duels
- Each biome has a **boss** at the end

## Combat
- Click-to-attack, melee only
- Weapon aesthetic: eye-catching for kids (details TBD — not basic swords)
- No item loss on death (PvE or PvP)

## Stats
- **Weapons:** Attack Power (flat)
- **Armor:** Armor = flat damage reduction
- **Mobs:** Attack Power, Health, Armor
- **Players:** No base stats — entirely gear-dependent
- Gear from previous biomes is outscaled by new biome gear

## Biome Layout (per biome)
- 4 monster camps, each dropping a specific rebirth material
- Currency drops from all mobs
- Boss at the end of the biome

### Biome 1 — Forest
- Rebirth materials: 4 crystal colors (Red, Blue, Yellow, Green)
- Camp 1 → Green Crystal, Camp 2 → Yellow Crystal, Camp 3 → Blue Crystal, Camp 4 → Red Crystal

### Biome 2 — Desert
- Rebirth materials: TBD (different material type, not crystals)
- Items and shop inventory distinct from Biome 1

### Biome 3 — Hell (Fire theme)
- Rebirth materials: TBD (different material type)
- Items and shop inventory distinct from previous biomes

Each biome has its own unique rebirth materials, items, and gear — previous biome gear is outscaled.

## PvE Progression
- Farm mobs → earn in-game currency + items
- Level up character
- Party system: players can group up to farm faster
- Boss fights: group up with friends to defeat

## Rebirth System
- Every 10 rebirths → unlock next biome
- Rebirth level displayed in player's base (prestige/social status)
- Defeating the biome boss is required to progress to next biome
- Cost scales with rebirth number: rebirth N costs N of each crystal + N×1000 currency
  - Example: Rebirth 1 = 1 red + 1 blue + 1 yellow + 1 green + 1,000 currency
  - Example: Rebirth 10 = 10 of each crystal + 10,000 currency
- 4 crystal types: Red, Blue, Yellow, Green (dropped from mobs/content)
- Each rebirth unlocks new purchasable items (weapons and armor) from shop

## Base System
- Every player has a base located in the shared town
- Other players can visit and view your base
- Display prestigious loot from boss kills
- Defeated bosses placed in base → generate passive in-game currency income
- Rebirth level visually shown in base

## PvP Arena
- 1v1 duels
- Level and gear score matter (not equalized)
- Optional betting system: players can wager in-game currency; winner takes all

## Economy
- In-game currency obtained from farming mobs in your current biome
- Currency used for rebirth (sacrificed)
- Currency used for PvP bets
- Passive income from boss trophies in base

**Why:** This is the foundational game design agreed upon in the initial interview.
**How to apply:** Use this as the source of truth when making design or implementation decisions.
