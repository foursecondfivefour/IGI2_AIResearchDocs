# Phase 3.1: AI System — IGI 2: Covert Strike

**Status:** 🟡 PARTIAL — 30+ functions renamed  
**Functions Documented:** 30+  
**Confidence:** HIGH  
**Verified:** 2026-08-26 via Ghidra MCP

---

## Architecture Overview

The AI system uses a **task-based entity architecture** with QScript-driven behaviors and graph-based pathfinding.

### Core Initialization

| Address | Function | Purpose |
|---------|----------|---------|
| 0x00462c80 | `HumanAI_Init` | Main AI subsystem — loads default.qsc, registers events/actions |
| 0x004eefe0 | `HumanSoldier_Init` | Registers TASKTYPE_HUMANSOLDIER entity type |
| 0x0045e380 | `RegisterAIActions` | Registers 80+ AI action behaviors |
| 0x00477120 | `InitAIDetection` | Registers detection system (LineOfSight, GetDetectionInfo) |
| 0x0056a4c0 | `UnregisterAIAction` | AI action cleanup |

---

## AI Entity Types

| String Address | Entity Type | Description |
|---------------|-------------|-------------|
| 0x006903e8 | TASKTYPE_HUMANSOLDIER | Main enemy AI |
| 0x0068e930 | TASKTYPE_HUMAN | Base human entity |
| 0x0068e1b0 | TASKTYPE_HUMANSHADOW | Stealth/shadow variant |
| 0x0069a3a0 | Bodyguard | Special guard type |

---

## AI Squad System

### Squad Globals (0x0069424c area)

| Address | Variable | Purpose |
|---------|----------|---------|
| 0x00694104 | AISquad_nEvent | Current squad event ID |
| 0x00694114 | AISquad_nSquadState | Current squad state |
| 0x0069422c | SquadType | Squad classification |
| 0x0069424c | AISquad | Base squad struct |
| 0x00697a90 | nPatrolPathStackPos | Patrol path stack |
| 0x00696124 | CurrentPathIndex | Path traversal index |

### Squad Functions (at 0x0069424c table)

| Function | Offset | Purpose |
|----------|--------|---------|
| AISquad_TriggerEvent | +0x08 | Trigger squad event |
| AISquad_PlaySoundAtReceiversPos | +0x20 | Play AI sound |
| AISquad_ReceiverHasTarget | +0x40 | Check target presence |
| AISquad_DefaultHandler | +0x5c | Default event handler |
| AISquad_GetDistanceToEvent | +0x74 | Calculate event distance |
| AISquad_GetEventSpecificInt32 | +0x90 | Get event integer param |
| AISquad_GetCurrentPatrolID | +0xb0 | Get patrol path ID |
| AISquad_ThrowGrenade | +0xc8 | Squad grenade throw |
| AISquad_Patrol | +0xe0 | Execute patrol |
| AISquad_MoveToNode | +0xf0 | Navigate to graph node |
| AISquad_ManStationaryGun | +0x104 | Operate turret |
| AISquad_TriggerAlarm | +0x120 | Trigger alarm state |
| AISquad_ReceiverLookAtEvent | +0x14c | Look at event directly |
| AISquad_HasTarget | +0x168 | Check valid target |
| AISquad_GetSquadType | +0x17c | Get squad classification |
| AISquad_SwitchState | +0x194 | State transition |
| AISquad_GetEvent | +0x1a8 | Get current event |
| AISquad_GetState | +0x1bc | Get current state |

---

## AI State Machine (10 States)

| State | String Address | Description |
|-------|---------------|-------------|
| 0 | 0x006945b8 | AISquadState_Danger |
| 1 | — | AISquadState_LookSearchInDirection |
| 2 | — | AISquadState_MoveSearchInDirection |
| 3 | — | AISquadState_Patrol |
| 4 | — | AISquadState_Leapfrog |
| 5 | — | AISquadState_Move |
| 6 | — | AISquadState_SearchArea |
| 7 | — | AISquadState_Inquire |
| 8 | — | AISquadState_HoldArea |
| 9 | — | AISquadState_Idle |

---

## AI Event System (26 Events)

### Detection Events (0x00697d48 area)

| Event | Description |
|-------|-------------|
| HUMANAI_DETECTIONEVENT_CANNON_RANGE | Large weapon detection |
| HUMANAI_DETECTIONEVENT_GUNSHOT_RANGE | Rifle gunshot |
| HUMANAI_DETECTIONEVENT_GUNSHOT_PISTOL_RANGE | Pistol gunshot |
| HUMANAI_DETECTIONEVENT_GUNSHOT_SILENCED_RANGE | Silenced weapon |

### General AI Events (0x00697fe0 area)

| Event | Description |
|-------|-------------|
| AIEVENT_LOSTSIGHTOFENEMY | Lost visual contact |
| AIEVENT_SEENSOMETHING | Suspicious movement |
| AIEVENT_BACKUP_REQUEST | Called backup |
| AIEVENT_DEATH_CRY | Death vocalization |
| AIEVENT_FRIENDLY_GUNSHOT | Heard ally fire |
| AIEVENT_ENEMYDETECTION | Detected enemy with LOS |
| AIEVENT_EXPLOSION | Heard/seen explosion |
| AIEVENT_BULLETIMPACT | Heard bullet impact |
| AIEVENT_DISTRESSCALL | Heard distress call |
| AIEVENT_GRENADESHOUT | Grenade warning |
| AIEVENT_ALERT | Alert triggered |
| AIEVENT_SQUADCOMMAND | Squad received command |

---

## AI Actions (7+ Actions)

| Action | String Address | Purpose |
|--------|---------------|---------|
| AIAction_DefensiveCombat | 0x006985a0 | Defensive stance |
| AIAction_OffensiveCombat | 0x006985bc | Offensive stance |
| AIAction_CombatIdle | 0x00698670 | Combat standby |
| AIAction_RunToCover | 0x006985e8 | Move to cover |
| AIAction_EnemyDetection | 0x0069862c | Scan for enemies |
| AIAction_FriendlyDetection | 0x006985fc | Detect friendlies |
| AIAction_Patrol | 0x006987e8 | Execute patrol route |

---

## Patrol Path Commands (13 Commands)

| Command | Purpose |
|---------|---------|
| PATROLPATHCOMMAND_SET_UPPERBODY_ANIM | Set upper body animation |
| PATROLPATHCOMMAND_WAIT_FOR_EXPRESSION | Wait for expression |
| PATROLPATHCOMMAND_SETSPEED | Set movement speed |
| PATROLPATHCOMMAND_QUIT | Quit patrol |
| PATROLPATHCOMMAND_END | End of path |
| PATROLPATHCOMMAND_LOOKATNODE | Look at path node |
| PATROLPATHCOMMAND_CROUCH | Crouch command |
| PATROLPATHCOMMAND_RUNTO | Run to target |
| PATROLPATHCOMMAND_WALKTO | Walk to target |
| PATROLPATHCOMMAND_DELAY | Wait/delay |
| PATROLPATHCOMMAND_ANIMATION | Play animation |

---

## AI Configuration API (30+ Functions)

| Function | Parameter Range | Purpose |
|----------|-----------------|---------|
| AIFunction_SetAlarmAccess | BEFORE/AFTER COMBAT | Alert access control |
| AIFunction_SetGunnerID | xx | Assign gunner |
| AIFunction_SetPercentageFiredPerBurst | 0-100 | Burst accuracy |
| AIFunction_SetEvasiveActionProb | 0-100 | Evasive maneuver chance |
| AIFunction_SetMinRangeAccuracy | 0-100 | Close-range accuracy |
| AIFunction_SetMaxRangeAccuracy | 0-100 | Long-range accuracy |
| AIFunction_SetAccuracyMaxRange | 1+ | Max accuracy distance |
| AIFunction_SetMinDelayBetweenFiring | seconds | Fire rate limit |
| AIFunction_SetTrackingTimeout | seconds | Target tracking timeout |
| AIFunction_SetTrackingMaxDistance | meters | Max tracking range |
| AIFunction_SetGrenadeThrowProb | 0-100 | Grenade usage chance |
| AIFunction_SetGrenadeDetonationTimer | seconds | Grenade fuse |
| AIFunction_SetGrenadeLandingReactionProb | 0-100 | Grenade reaction |
| AIFunction_SetCloseCombatDamage | xxx | Melee damage |
| AIFunction_SetHitPoints | xxx | Health/HP |
| AIFunction_SetHitRecoilThreshold | 0-100 | Recoil tolerance |
| AIFunction_SetCombatViewCone1Alpha/Length | degrees/m | Combat vision angle/range |
| AIFunction_SetIdleViewCone1Alpha/Length | degrees/m | Idle detection |
| AIFunction_SetInvestigateViewCone1Alpha/Length | degrees/m | Investigation cone |
| AIFunction_SetIdleAnimationFrequency | seconds | Idle animation rate |
| AIFunction_SetAnimSpeedFactor | 0-2 | Animation speed multiplier |
| AIFunction_SetHearingRangeFactor | — | Hearing modifier |

---

## AI Tuning Tables (0x006a21c8 area)

| Table | Range | Purpose |
|-------|-------|---------|
| DetectionToAttackTime | 0-100 | Time before attack |
| MaxTrackingTime | 0-500 | Max tracking duration |
| MaxTrackingDist | 0-500 | Max tracking distance |
| PercentFirecPerBurst | 0-100 | Rounds per burst |
| MinFireInterval | 0-1 | Min fire interval |
| InvView1/2Length | 0-500 | Investigate view range |
| InvView1/2Alpha | 0-180 | Investigate view angle |
| IdleView1/2Length | 0-500 | Idle detection range |
| IdleView1/2Alpha | 0-180 | Idle detection angle |
| HitPoints | 0-100 | Enemy HP |
| GrenadeThrowProb | 0-100 | Grenade usage |
| GrenadeTimer | 0-100 | Grenade detonation |

---

## Key Data Addresses

| Address | Variable | Purpose |
|---------|----------|---------|
| 0x00696234 | vDetectionRadius | AI detection radius |
| 0x00694c14 | nDetectionTime | Detection duration |
| 0x00694c24 | isLastDetection | Latest detection flag |
| 0x00694c34 | isDetection | Detection active flag |
| 0x006a21c8 | AI tuning tables | Combat/behavior tuning |
