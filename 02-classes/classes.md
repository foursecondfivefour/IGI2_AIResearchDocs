# Phase 2: Class Hierarchy & Data Structures

**Status:** 🟡 PARTIAL  
**Classes Identified:** 2 (HumanPlayer, HumanSoldier)  
**Functions Documented:** 3 (entry, WinMain, matrix math)  
**Confidence:** MEDIUM

---

## 2.1 Main Game Entry Point

**Address:** 0x00404280  
**Function:** `undefined * FUN_00404280(HINSTANCE hInstance, HINSTANCE hPrevInstance, int nCmdShow, int nShowCmd)`

This is the **WinMain** entry point — the main game engine initialization and message loop.

### Initialization Flow:

1. **Mutex Check** — Creates `InnerloopGameMUTEX` to prevent multiple instances
2. **COM Initialization** — `CoInitialize(0)` for ActiveX/OLE support
3. **Window Class Registration** — Registers "Innerloop" window class
4. **Configuration Loading** — Loads multiple config files:
   - `filesys.cfg` — File system configuration
   - `Window` — Window settings
   - `WinInput` — Windows input settings
   - `Track` — Tracking settings
   - `Editor` — Editor mode config
   - `Server` — Server settings
   - `Player` — Player settings
   - `Password` — Password settings
   - `Zbias` — Z-buffer bias settings
   - `Config` — General config
   - `SessionDir` — Session directory
   - `FPSLock` — FPS lock settings
5. **DirectX Check** — Verifies DirectX >= 3.1
6. **Window Creation** — Creates main game window (640x480 or system default)
7. **Main Loop** — Message pump with PeekMessage/DispatchMessage

### Key Globals Initialized at WinMain:

| Address | Name | Type | Purpose |
|---------|------|------|---------|
| 0x006c0798 | DAT_006c0798 | undefined4 | HINSTANCE (module handle) |
| 0x006c0790 | DAT_006c0790 | undefined4 | Window handle |
| 0x006c07d4 | DAT_006c07d4 | int | Game running flag |
| 0x006c07d8 | DAT_006c07d8 | int | Pause flag |
| 0x006c0804 | DAT_006c0804 | code* | Callback function pointer |
| 0x0073fb4c | DAT_0073fb4c | undefined4 | FPU control word |
| 0x006c06f8 | DAT_006c06f8 | undefined | Game exit flag |

### Window Style:
```
WS_OVERLAPPED | WS_CAPTION | WS_SYSMENU | WS_MINIMIZEBOX
= 0x10CE0000
```

### Window Class:
```
Class Name: "Innerloop"
Icon: IDI_APPLICATION (0x7f00)
Cursor: IDC_ARROW (0x7f00)
```

---

## 2.2 Matrix Math Functions

**Address:** 0x006280e7  
**Function:** `void FUN_006280e7(float *param_1, undefined4 param_2)`

This function generates a **4x4 transformation matrix** from Euler angles.

```c
/* Input: param_2 likely contains Euler angles (x, y, z) as float3 */
/* Output: param_1 receives 4x4 rotation matrix (column-major) */

void GenerateRotationMatrix(float* outMatrix, float* angles) {
    float cx, cy, cz, sx, sy, sz;
    // Angles are stored on stack with -2.0 scale
    // (likely converting degrees to radians or similar)
    
    // Standard 4x4 rotation matrix construction:
    // [R00  R01  R02  0]
    // [R10  R11  R12  0]
    // [R20  R21  R22  0]
    // [Tx   Ty   Tz   1]
    
    // Matrix is stored column-major (DirectX style)
}
```

---

## 2.3 HumanPlayer Class

**Module:** CDAPFN0506  
**Base Address:** Vtable at 0x006aa4a0

### Exported State Machine Functions:

| Ordinal | Function | Address | Purpose |
|---------|----------|---------|---------|
| 1 | X_HumanPlayer_InitStateActivateC4BombTBA | 0x006b8aa8 | Vtable dispatch |
| 2 | X_HumanPlayer_InitStateActivateCabinet | 0x006b8668 | Vtable dispatch |
| 3 | X_HumanPlayer_InitStateActivateDoor | 0x006b8558 | Vtable dispatch |
| 4 | X_HumanPlayer_InitStateActivateGenericTBA | 0x006b8998 | Vtable dispatch |
| 5 | X_HumanPlayer_InitStateActivatePlaceExplosiveTBA | 0x006b8bb8 | Vtable dispatch |
| 6 | X_HumanPlayer_InitStateActivateStationaryGun | 0x006b8888 | Vtable dispatch |
| 7 | X_HumanPlayer_InitStateActivateTerminal | 0x006b8778 | Vtable dispatch |
| 8 | X_HumanPlayer_InitStateSilentKill | 0x006b8cc8 | Vtable dispatch |
| 10 | HumanPlayer_InitStateActivateC4BombTBA | 0x006b8a20 | Direct call |
| 11 | HumanPlayer_InitStateActivateCabinet | 0x006b85e0 | Direct call |
| 12 | HumanPlayer_InitStateActivateDoor | 0x006b84d0 | Direct call |
| 13 | HumanPlayer_InitStateActivateGenericTBA | 0x006b8910 | Direct call |
| 14 | HumanPlayer_InitStateActivatePlaceExplosiveTBA | 0x006b8b30 | Direct call |
| 15 | HumanPlayer_InitStateActivateStationaryGun | 0x006b8800 | Direct call |
| 16 | HumanPlayer_InitStateActivateTerminal | 0x006b86f0 | Direct call |
| 17 | HumanPlayer_InitStateSilentKill | 0x006b8c40 | Direct call |

### State Categories:

**Interaction States (4):**
- ActivateDoor — Open/close doors
- ActivateCabinet — Open storage containers
- ActivateTerminal — Use computer terminals
- ActivateGenericTBA — Generic interactable objects

**Combat States (2):**
- ActivateStationaryGun — Use mounted/gun emplacements
- SilentKill — Stealth takedown animation

**Explosive States (2):**
- ActivateC4BombTBA — Place C4 explosives
- ActivatePlaceExplosiveTBA — Place other explosives

**Dispatch Mechanism:**
The "X_" prefix functions are vtable dispatch thunks that call the actual state init functions.

---

## 2.4 HumanSoldier Class

**Module:** CDAPFN0506  
**Class Type:** Enemy NPC AI

### Exported State Machine Functions:

| Ordinal | Function | Address | Purpose |
|---------|----------|---------|---------|
| 9 | X_HumanSoldier_InitStateStandThrowGrenade | 0x006b8118 | Vtable dispatch |
| 18 | HumanSoldier_InitStateStandThrowGrenade | 0x006b8090 | Direct call |

### Combat State:
- **StandThrowGrenade** — Enemy soldier throws grenade at player

---

## 2.5 Class Hierarchy (Deduced)

Based on export patterns and naming conventions:

```
CDAPFN0506 (module namespace)
│
├── HumanPlayer (player character)
│   ├── StateMachine
│   │   ├── m_nCurrentState (int)
│   │   ├── m_nPreviousState (int)
│   │   └── m_pStateTable (StateEntry*)
│   │
│   ├── States (10 total):
│   │   ├── ACTIVATE_DOOR = 12
│   │   ├── ACTIVATE_CABINET = 11
│   │   ├── ACTIVATE_TERMINAL = 16
│   │   ├── ACTIVATE_STATIONARY_GUN = 15
│   │   ├── ACTIVATE_GENERIC_TBA = 13
│   │   ├── ACTIVATE_C4_BOMB = 10
│   │   ├── ACTIVATE_PLACE_EXPLOSIVE = 14
│   │   └── SILENT_KILL = 17
│   │
│   └── Vtable (16 entries at 0x006aa4a0):
│       ├── pfnInitActivateDoor
│       ├── pfnInitActivateCabinet
│       ├── pfnInitActivateTerminal
│       ├── pfnInitActivateStationaryGun
│       ├── pfnInitActivateGenericTBA
│       ├── pfnInitActivateC4Bomb
│       ├── pfnInitActivatePlaceExplosive
│       ├── pfnInitSilentKill
│       └── ... (8 more vtable entries)
│
├── HumanSoldier (enemy NPC)
│   └── States:
│       └── STAND_THROW_GRENADE = 18
│
└── [OTHER CLASSES] (not exported, TBD)
```

---

## 2.6 State Machine Architecture

### Pattern Analysis:

All state init functions follow:
```c
void __fastcall ClassName_InitStateStateName(StateData* this, int stateParam);
```

### Vtable Structure (at 0x006aa4a0):

```c
struct HumanPlayer_VTable {
    void (*pfnInitActivateC4Bomb)(void);       // Ordinal 1
    void (*pfnInitActivateCabinet)(void);       // Ordinal 2
    void (*pfnInitActivateDoor)(void);          // Ordinal 3
    void (*pfnInitActivateGenericTBA)(void);    // Ordinal 4
    void (*pfnInitActivatePlaceExplosive)(void);// Ordinal 5
    void (*pfnInitActivateStationaryGun)(void); // Ordinal 6
    void (*pfnInitActivateTerminal)(void);      // Ordinal 7
    void (*pfnInitSilentKill)(void);            // Ordinal 8
    void (*pfnX_HumanSoldier_Grenade)(void);    // Ordinal 9
    void (*pfnUnknown_A)(void);                 // Ordinal 10
    void (*pfnUnknown_B)(void);                 // Ordinal 11
    void (*pfnUnknown_C)(void);                 // Ordinal 12
    // ... more entries
};
```

---

## 2.7 Configuration File Format

Based on WinMain analysis:

### filesys.cfg
File system paths for game assets (models, textures, sounds, levels)

### Window
Window resolution, fullscreen mode, border style

### WinInput
Keyboard/mouse input configuration

### Track
Camera tracking parameters (FOV, distance, smoothing)

### Editor
Editor mode flags, gizmo settings, grid size

### Server
Multiplayer server configuration

### Player
Player preferences (sensitivity, bind key mappings)

### Password
Network password/authorization

### Zbias
Depth buffer bias for shadow rendering

### Config
General game settings (audio, video, difficulty)

### SessionDir
Multiplayer session directory path

### FPSLock
Frame rate limit (30/60/120/unlimited)

---

## 2.8 Key Functions Called by WinMain

| Function | Address | Purpose |
|----------|---------|---------|
| FUN_0041d9e0 | 0x0041d9e0 | Config file parser |
| FUN_00446360 | 0x00446360 | DirectX version check |
| FUN_00401210 | 0x00401210 | Game loop state |
| FUN_00446780 | 0x00446780 | Graphics initialization |
| FUN_00401440 | 0x00401440 | Frame render/update |
| FUN_00401490 | 0x00401490 | Window cleanup |

---

## 2.9 New AI Types Discovered (2026-08-27)

### HUMANAI_TYPE_C1_NORMAL_SOLDIER
**Address:** 0x006b9830 (string pointer)  
**Xrefs:** 8  

A new AI type string discovered in the binary. This is likely the type identifier for standard C1 (Covert 1) soldier AI. The naming convention suggests:
- **C1** — Covert 1 (James "Jimmy" Riggs, the player character)
- **NORMAL_SOLDIER** — Standard enemy soldier type

### Other String References
| Address | String | Xrefs | Purpose |
|---------|--------|-------|---------|
| 0x006bc0e8 | "Jones" | 9 | NPC name / character reference |
| 0x006aed04 | "Sunday" | 6 | Day of week (part of date/time system) |

---

## 2.10 Known Data Addresses

### Global Configuration Block (0x006c0600-0x006c0900)
All zeroed at load time, initialized by WinMain:

| Offset | Size | Purpose |
|--------|------|---------|
| 0x006c0638 | 192 bytes | Config value array (0-initialized) |
| 0x006c06fc | 4 bytes | Config data pointer |
| 0x006c0700 | 4 bytes | Config data pointer |
| 0x006c0790 | 4 bytes | Window handle |
| 0x006c0794 | 4 bytes | Window title |
| 0x006c0798 | 4 bytes | HINSTANCE |
| 0x006c079c | 16 bytes | "filesys.cfg" |
| 0x006c07a8 | 16 bytes | "Innerloop" |
| 0x006c07d0 | 4 bytes | Flag (0) |
| 0x006c07d4 | 4 bytes | Game running (1) |
| 0x006c07d8 | 4 bytes | Pause flag (1) |
| 0x006c07e0 | 4 bytes | Flag (1) |
| 0x006c07e8 | 4 bytes | Flag (0) |
| 0x006c07ec | 4 bytes | Flag (0) |
| 0x006c07f0 | 4 bytes | Exit flag (0) |
| 0x006c07f8 | 4 bytes | Flag (0) |
| 0x006c0804 | 4 bytes | Callback function |

### FPU State (0x0073fb4c)
- Initial: `0x10ce0000` (single precision)
- Runtime: `0x96080000` (modified for game math)

---

## Next Steps

- [ ] Find the actual vtable entries (0x006aa4a0) and link to functions
- [ ] Discover more classes beyond HumanPlayer/HumanSoldier
- [ ] Reverse the state machine data structures
- [ ] Identify all class member variables from usage patterns
- [ ] Map vtable layouts for each class
