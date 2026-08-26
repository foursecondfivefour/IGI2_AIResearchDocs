# Phase 3.4: Level & Mission System — IGI 2: Covert Strike

**Status:** ✅ REVERSED  
**Functions Documented:** 25+  
**Confidence:** HIGH

---

## Main Level Loader (FUN_00407420)

**Purpose:** Complete level/mission initialization pipeline

### Pipeline Sequence:
1. Set 3D Sound provider
2. Find mission by ID from config
3. Initiate camera
4. Initiate weapon handlers
5. Create new world
6. Register game materials
7. Init LOD Virtual Model System
8. Load COMMON resource archive
9. Load new resource archive
10. Load LOCATION resources
11. Load LEVEL MTP
12. Load common sounds
13. Load game sprites
14. Load texture resources
15. Load effect/lensflare textures
16. Load fonts
17. Load computer menu resource
18. Start game screen task
19. Start level task
20. Start sound system
21. Load lightmaps
22. Load heightmaps
23. Load objects
24. Load carry over data
25. Set up music
26. Load debugconsole
27. Load savegame
28. Unload Font Resource

---

## Resource Path Prefixes

| Prefix | Purpose | Example |
|--------|---------|---------|
| LOCAL: | Game install dir | LOCAL:textures/000_01_1.tex |
| MISSION: | Mission-specific | MISSION:graphs/graph%d.dat |
| SESSION: | Save games | SESSION:savegames/missions/ |
| COMPUTER: | HUD sprites | COMPUTER:objective_1.spr |
| LANGUAGE: | Localized text | LANGUAGE:objectives.res |
| STATUSSCREEN: | Status screen | STATUSSCREEN:weapon.spr |

---

## Mission System

### Global State (0x08350c50):
| Offset | Variable | Purpose |
|--------|----------|---------|
| +0x00 | DAT_08350c50 | Game context pointer |
| +0x04 | Mission active | 0=menu, 1=playing |
| +0x08 | Current mission ID | 1-14 |

### Key Natives:
| Native | Address | Purpose |
|--------|---------|---------|
| LevelLoad | 0x004F0E10 | Load level |
| LevelStart | 0x00415B30 | Start new level |
| LevelQuit | 0x00416550 | Quit to menu |
| LevelRestart | 0x00416FE0 | Restart level |
| MissionOpen | 0x00484E60 | Open mission |
| GraphOpen | 0x004F9FF0 | Open navigation graph |

---

## QTask/QVM Script System

### Script Functions:
| Function | Address | Purpose |
|----------|---------|---------|
| QscCompile | 0x004B8410 | Compile QSC→QVM |
| QvmLoad | 0x004B80B0 | Load QVM binary |
| QTaskHashTableSet | 0x004BAAC0 | Register task type |
| ScriptSetsymbolCxt | 0x004B8930 | Set symbol context |

### QTask Types:
- TASKTYPE_LEVELTIMER — Timer
- TASKTYPE_LEVELFLOW — Flow control
- TASKTYPE_SPAWNAREA — Spawn area
- TASKTYPE_SPAWNPOINT — Spawn point
- TASKTYPE_C4BOMBAREA — C4 bomb task
- TASKTYPE_SHADOWVOLUME — Shadow volume

---

## Objective System (0x00755378, 672 bytes)

| Offset | Field | Type |
|--------|-------|------|
| 0x00 | Objective Expression | string |
| 0x04 | Objective Failed Expression | string |
| 0x08 | Time for objective | float |
| 0x0C | Team | int |
| 0x10 | Order number | int |
| 0x14 | Is C4 required | bool |
| 0x18-0x6C | Objectives 1-8 | structured |
| 0x70 | Objectives Valid | bool |

**Registration:** FUN_004d1030 — DefineComputerObjective

---

## Trigger / Volume System

### Animation Triggers (0x4EC070+):
- HUMANANIM_TRIGGER_DEADBODY_SOUND
- HUMANANIM_TRIGGER_CLOSECOMBAT
- HUMANANIM_TRIGGER_THROWGRENADE
- HUMANANIM_TRIGGER_FEETSOUND
- HUMANANIM_TRIGGER_STOPANIMATION
- HUMANANIM_TRIGGER_DROPWEAPON
- HUMANANIM_TRIGGER_DRAW/OPENDRAWER

### Alarm System:
- AISquad_TriggerAlarm
- AISquad_TriggerEvent
- AlarmTriggerID
- nTimesAlarmTriggered

---

## Spawn Area System (0x007537e8, 132 bytes)

| Offset | Field | Type |
|--------|-------|------|
| 0x00 | nTicksToRespawn | int |
| 0x04 | nRespawnTime | int |
| 0x08 | zRespawnTime | float |
| 0x0C | SPAWNFREE/SPAWNCOST | enum |
| 0x10 | nID | int |

**Registration:** FUN_004840d0 — SpawnArea init

---

## Navigation Graph System (FUN_005b1ad0)

### Graph Node Properties (hash-keyed):

| Key | Type | Size | Purpose |
|-----|------|------|---------|
| nMaxNodes | int | 4 | Max capacity (0x400=1024) |
| nNodeID | int | 4 | Unique identifier |
| tNodePosition | float[3] | 12 | World position |
| vNodeGamma | float | 4 | Lighting factor |
| vNodeRadius | float | 4 | Detection radius |
| nNodeMaterial | int | 4 | Surface type |
| vNodeLightValue | float | 4 | Light intensity |
| nNodeOnTerrain | int | 4 | Terrain flag |
| nNodeCriteria | int | 4 | Flags (DOOR=1,VIEW=2,STAIR=4) |
| nLinkID1 | int | 4 | Edge target 1 |
| nLinkID2 | int | 4 | Edge target 2 |
| eLinkType | int | 4 | Edge type enum |

### Graph File Format:
- Magic: 0xFFEEDDCC
- Header: unknown, grid dims, node count
- Nodes: 12-byte position + edge list (uint16 target + cost)
- Path: MISSION:graphs/graph%d.dat

---

## Level Flow System (FUN_00524df0)

### Flow Events:
| Event | Purpose |
|-------|---------|
| FLOW_EVENT_RESTART_GAME | Restart |
| FLOW_EVENT_GAME | Progress |
| FLOW_EVENT_MAINMENU | Main menu |
| FLOW_EVENT_INTRO | Intro |
| FLOW_EVENT_QUIT | Quit |

### Flow Functions:
- LevelFlow_SetTimeOfDay
- LevelFlow_IsCountryUSA
- LevelFlow_LevelFailed
- LevelFlow_GetBreakCutSceneKey
- Flow_IsPlayCredits
- Flow_RequestEvent
- Flow-DrawInterpolate (smooth transitions)

---

## Mission Map List

### Level 1 (11 graphs):
Cutscene Areas, WaterTower, WatchTower, Anya HQ, etc.

### Level 2 (17 graphs):
First/Second Base, Watchtowers, SAM positions

### Level 8 (29 graphs):
Truck Alarm, HQ floors, APC Base, Barracks, etc.
