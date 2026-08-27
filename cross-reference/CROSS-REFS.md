# Cross-References — IGI 2: Covert Strike

Complete mapping of how all systems interconnect.

**Status:** 🟡 STRUCTURAL — cross-reference analysis  
**Verified:** 2026-08-26 via Ghidra MCP

---

## System Interconnections

### AI ↔ Rendering
- AICameraInfo → Camera System (GetHumanCameraInfo at 0x6910a8)
- AI sight cones → Render view frustums
- Graph nodes → Terrain lighting (vNodeGamma, vNodeLightValue)

### AI ↔ Weapons
- Grenade throw probability in AI config → Grenade system
- AI actions (OffensiveCombat) → Weapon fire functions
- AIFunction_SetGunnerID → Stationary gun control

### AI ↔ Level System
- Graph navigation used by all AI types (1024 nodes)
- Spawn areas → AI patrol paths
- Triggers → AI event system

### AI ↔ Audio
- Sound events trigger AI reactions (hearing range config)
- AI sound definitions → Miles Sound System streams

### Weapons ↔ Rendering
- Weapon model overlay → 3D model rendering
- Sniper overlays → Texture filtering system
- Muzzle flash → Sprite rendering
- Projectile models → MEF model loader

### Weapons ↔ Level System
- Weapon carry-over between missions
- C4 bomb objectives → GMUpdate packets
- Grenade physics → Level collision detection

### Rendering ↔ Level System
- MEF models → Level object rendering
- Lightmaps → Terrain shading
- BSP trees → Forest rendering
- Heightmaps → Terrain generation

### Rendering ↔ Audio
- 3D sound positioning matches 3D model positions
- Camera position → Listener position for 3D audio

### Level System ↔ Audio
- Sound files loaded during level init (21-step process)
- Sound triggers → Event system
- Music changes on level transitions

### QVM ↔ All Systems
- Script files define AI behavior (TASKTYPE_*)
- Mission scripts control objectives
- Config scripts set game parameters
- HUD elements drawn via QVM rendering calls

---

## Function Call Graphs

### Main Game Loop (0x00404280 WinMain)
```
WinMain (0x00404280)
├── DirectX init (0x00446360)
├── Config parsing (0x0041d9e0 × 13 files)
├── MenuManager_Init (0x00433C50)
├── Level loader (0x00407420)
│   ├── AI init (0x00462c80)
│   ├── Weapon system init
│   ├── Rendering init
│   ├── Audio init (0x00542570)
│   └── Level loading (28 steps)
└── Main loop: PeekMessage → DispatchMessage
    └── Frame update (0x00401440)
        ├── Input processing (0x004262C0)
        ├── AI update (0x005699C0)
        ├── Render frame
        └── Audio update
```

### State Machine Dispatch
```
FUN_004a07c0 (state dispatcher)
└── State entry table at 0x006b8558
    ├── InitStateActivateDoor → Player-Door interaction
    ├── InitStateActivateCabinet → Player-Container
    ├── InitStateActivateTerminal → Player-Terminal
    ├── InitStateActivateStationaryGun → Player-Mounted Gun
    ├── InitStateActivateGenericTBA → Generic interaction
    ├── InitStateActivateC4BombTBA → C4 placement
    ├── InitStateActivatePlaceExplosiveTBA → Explosive placement
    └── InitStateSilentKill → Stealth takedown
```

### QVM Execution Flow
```
QvmLoad (0x004B80B0)
└── QVM Interpreter (0x005699C0)
    ├── Opcode dispatch (0x00568B74 jump table)
    ├── Expression eval (0x005661C0 int, 0x00566730 float)
    ├── AIFunction calls (0x00698238+)
    └── QTask handlers
        ├── TASKTYPE_HUMANSOLDIER
        ├── TASKTYPE_LEVELTIMER
        ├── TASKTYPE_SPAWNAREA
        └── 88+ task types
```

### Resource Loading Pipeline
```
FUN_0040c6d0 (dispatcher)
├── ILFF archive parsing
├── MEF model loading (0x004F7AD0)
├── TEX texture loading (0x004F8790)
├── LMP lightmap loading (0x0053EB70)
├── FNT font loading
├── SPR sprite loading
├── IFF animation loading (0x004F8930)
└── RES resource loading
```

---

## Data Structure Cross-References

### Entity Struct (0xd80 bytes)
Used by:
- Entity update (0x004b0a00)
- AI behavior
- Rendering (position/rotation)
- Physics (collision detection)

### HumanPlayer Input (0x2B0 bytes × 10 players)
Used by:
- Input processing (0x0052c400)
- Menu system (0x004262c0)
- Gameplay controls
- Config (mouse sensitivity, invert)

### NetManager (0x006a25f0+)
Used by:
- Network tick (0x006a25f0)
- Game mode state machine
- Spawn management
- Score tracking

---

## String References by System

### Common Strings (cross-cutting)
- `"Error occurred during rendering"` → Rendering + Error handling
- `"LOCAL:common/textures/linefilter1.tga"` → Rendering + Resources
- `"Starting sound"` → Audio + System
- `"Unknown command"` → Console + Input

### System-Specific Strings
| System | String Count | Range |
|--------|-------------|-------|
| AI | 80+ | 0x00694000-0x00698924 |
| Weapons | 50+ | 0x0068ca00-0x0068ccbc |
| Rendering | 60+ | 0x00683500-0x0068ff90 |
| Level | 40+ | 0x0068b000-0x0068d000 |
| Audio | 37+ | 0x00672330-0x0068a7b8 |
| Menus | 100+ | 0x00696000-0x006a1700 |
| Config | 70+ | 0x006a0cf0-0x006a59b0 |
| Networking | 40+ | 0x0069a000-0x006a3024 |
| QVM | 30+ | 0x00689540-0x00689924 |
| Resources | 130+ | 0x006a5000-0x006a7000 |

---

## Key Global Data Dependencies

| Global | Used By | Purpose |
|--------|---------|---------|
| 0x006c07f0 | Audio init, Network init | System init flags |
| 0x00704884 | Resource loading | File type dispatch |
| 0x0071a604 | Menu system | Screen stack |
| 0x0073fb4c | Rendering | FPU control |
| 0x006b8558 | State machine | AI interaction table |
| 0x006a25f0 | Networking | NetManager |
| 0x006a2dac | Networking | GM message strings |
