---
name: project-mob-chase-ai
description: "Forest slimes now chase players, attack in melee, and lunge forward on attack (added 2026-06-23)"
metadata: 
  node_type: memory
  type: project
  originSessionId: c060dd4f-04a4-4c97-bdb4-635eb46c64e1
---

Added 2026-06-23: Forest slimes (tag "ForestSlime") were static and only bonked players who walked into MobMeleeRange (8.5). Now they actively chase.

KEY ARCHITECTURE: slimes have NO Humanoid. They are anchored part-models positioned every frame by the client `StarterPlayer.StarterPlayerScripts.ForestSlimeIdle`, which reads the `BasePivot` attribute (a CFrame) and adds bob/tilt/hit-backlash/lunge offsets. So all "movement" is done server-side by writing the `BasePivot` attribute; the client animation rides on top.

New attributes per mob:
- `HomePivot` = fixed spawn CFrame (captured at init from original BasePivot). Respawn + leash anchor.
- `BasePivot` = LIVE position (now movable). Chase writes this.
- `Aggro` = bool, true while actively pursuing.
- `LungeTime`/`LungeDirX`/`LungeDirZ` = set when the mob attacks; client renders a forward surge.

Tuning in `ReplicatedStorage.GameData`: MobAggroRange=28, MobMoveSpeed=9 (player walkspeed 16, so escapable), MobLeash=45 (from home), MobStopDist=6 (stops in melee range, doesn't stack on player), MobLungeDist=3.2, MobLungeTime=0.18.

Logic in `ServerScriptService.ForestMobs`:
- `homeOf(mob)` + `nearestPlayer(pos,maxDist)` helpers; `initMob` captures HomePivot.
- New "MOVEMENT BRAIN" task.spawn loop (Heartbeat-rate): nearest living player within aggro AND within leash of home -> step BasePivot toward them (face travel dir) until MobStopDist; else walk back to HomePivot. Skips stunned mobs (parry StunUntil).
- Attack loop (existing retaliation) now also sets Lunge* attributes pointing at the target.
- `die()`/respawn now use HomePivot and reset BasePivot to home on respawn.

Targeting = nearest player, all slimes pile on (per user choice). Playtested: chase 13->6 studs then holds, lunge fires in melee, leash returns home when player flees to 90 studs.

FOLLOW-UP FIX (2026-06-23): two bugs surfaced — couldn't hit slimes once they chased outside their camp, and slimes often didn't attack while approaching. Root cause: the SERVER never moved the model during chase (only the client pivots for visuals), so `mob.PrimaryPart.Position` stayed at the camp. Both the MobHit range check (>20 reject) and the retaliation loop's target detection used that stale PrimaryPart position. Fix: added `livePos(mob)` helper (reads BasePivot.Position as truth); MobHit and retaliation `mp` now use livePos; and the movement brain now also calls `mob:PivotTo(newBase)` each step so server PrimaryPart tracks the chase (client still re-pivots each frame for bob/tilt, no visual conflict). Verified: server track gap 0.00, lunge/attacks fire on approach, hits land on a slime 20 studs out of camp.

Related: [[project_forest_biome_build]], [[project_balance_system]], [[game_balance_theorycraft]].
