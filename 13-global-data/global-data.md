# Phase 10: Global Data Structures — IGI 2: Covert Strike

**Status:** ✅ REVERSED  
**Data Areas:** 13 documented  
**Confidence:** HIGH

---

## Critical Data Areas

### 1. AI State Machine Jump Table (0x006b8558-0x006b8ce8)
**Size:** 14 entries × 32 bytes = 448 bytes

**Entry Structure (32 bytes each):**
```c
struct StateEntry {
    DWORD dwPrimaryFn;     // [0:4] Primary function pointer
    DWORD dwParam;         // [4:8] Integer parameter (0x84, 0x00, 0x20)
    DWORD dwSecondaryFn;   // [8:12] Secondary function pointer
    DWORD dwActionType;    // [12:16] Action type flag (0 or 2)
    DWORD dwMode;          // [16:20] Mode flag (0 or 2)
    DWORD dwReserved;      // [20:24] Reserved (0)
    DWORD dwEnabled;       // [24:28] Enabled flag (1)
    DWORD dwReserved2;     // [28:32] Reserved (0)
};
```

**State Entries:**
| Address | State | Primary Fn | Secondary Fn | Type |
|---------|-------|-----------|-------------|------|
| 0x006b8558 | InitStateActivateDoor | 0x004a0933 | 0x004a0be0 | 2 |
| 0x006b85e0 | InitStateActivateCabinet | 0x004a0933 | 0x004b03e0 | 0 |
| 0x006b8668 | X_HumanPlayer_InitStateActivateCabinet | 0x004a0933 | 0x004a0c00 | 2 |
| 0x006b86f0 | InitStateActivateTerminal | 0x004a0933 | 0x004a0c20 | 0 |
| 0x006b8778 | X_HumanPlayer_InitStateActivateTerminal | 0x004a0933 | 0x004a0c20 | 2 |
| 0x006b8800 | InitStateActivateStationaryGun | 0x004a0933 | 0x004a0be0 | 2 |
| 0x006b8888 | X_HumanPlayer_InitStateActivateStationaryGun | 0x004a0933 | — | 2 |
| 0x006b8910 | InitStateActivateGenericTBA | 0x004a0933 | 0x004a0be0 | 0 |
| 0x006b8998 | X_HumanPlayer_InitStateActivateGenericTBA | 0x004a0933 | — | 2 |
| 0x006b8a20 | InitStateActivateC4BombTBA | 0x004a0933 | 0x004a0be0 | 2 |
| 0x006b8aa8 | X_HumanPlayer_InitStateActivateC4BombTBA | 0x004a0933 | — | 2 |
| 0x006b8bb8 | InitStateActivatePlaceExplosiveTBA | 0x004a0933 | 0x004a0be0 | 2 |
| 0x006b8cc8 | InitStateSilentKill | 0x004a0933 | 0x004a0be0 | 2 |

**Note:** "(X)" variants handle cross-team/ally interactions. All primary pointers → FUN_004a07c0 (quaternion/matrix math for spatial alignment).

---

### 2. Input Action / Key Binding Table (0x006ba1e4)
**Size:** ~256 bytes

**Entry Structure (32 bytes):**
```c
struct KeyBinding {
    WORD wActionID;       // [0:2] Action ID
    WORD wModifierFlags;  // [2:4] Modifier flags (0x0002, 0x0001, 0xFFFF)
    DWORD dwFnPtr;        // [4:8] Function pointer
    BYTE szKey[24];       // [8:32] Key name string
};
```

**Strings:** `"Movement"`, `"MoveForward"`, `"MoveBackwards"`, `"MoveLeft"`

---

### 3. Weapon System Globals (0x0068ca3c-0x0068ca94)
**Strings table:**
- `"iActiveWeapon"` — Current weapon ID
- `"WeaponTypeList"` — All weapon types
- `"AmmoTypeList"` — All ammo types
- `"WeaponItem"`, `"AmmoItem"` — Current weapon/ammo
- `"GetWeaponObject"` — Weapon getter
- `"Weapon_PositionOverlay"`, `"Weapon_CreateProjectile"`
- `"Weapon_DeActivate"`, `"Weapon_Activate"`, `"Weapon_Zoom"`
- `"Weapon_Reload"`, `"Weapon_Release"`, `"Weapon_GetPrecision"`

---

### 4. Mission Types (0x0068c024)
- `"[IGI] "` — Chat prefix
- `"MISSION_TYPE_HINDHELICOPTER"` — Helicopter mission
- `"MISSION_TYPE_BOMB"` — Bomb mission
- `"MISSION_TYPE_MISSION_KEYBOARD"` — Keyboard mission
- `"MISSION_TYPE_MISSION_ANTITANK"` — Anti-tank mission

---

### 5. Debug Console Commands (0x00685b68)
- `"players"`, `"rules"`, `"info"`, `"basic"`, `"status"`, `"echo"`
- `"Rain"`, `"SpliceObjSegment"`
- `"Unknown command: '%s'"`
- `"Enter ? <commandname> for i"`

---

### 6. HUD/UI Strings (0x0068b888)
- `"COMPUTER:h_player.spr"` — Player sprite
- `"eStatus"`, `"nTicksLeft"`, `"Time In Seconds"`
- `"Pause Timer Expression"`, `"Start Timer Expression"`
- `"MissionTimerDrawTask"`

---

### 7. Physics/Animation Constants (0x006835b8)
**Type:** Float array (~64 floats)  
**Values:** Alternating 0.5f, 1.0f, and physics coefficients  
**Xrefs:** 108 — significant usage  
**Purpose:** Rigid body physics or animation blending coefficients

---

### 8. Texture/Surface Format Table (0x006ab274-0x006ab574)
**Structure per entry (~32 bytes):**
```c
struct TextureFormat {
    DWORD dwFormatIndex;  // [0:4] Format index (0x15-0x34)
    DWORD dwWidth;        // [4:8] Width/flags (0x20 = 32)
    DWORD dwHeight;       // [8:12] Height/flags (0x40-0x41 = 64-65)
    DWORD dwAlpha;        // [12:16] Alpha/flags
};
```

**Four-character codes:** `"SYVP"`, `"YVP2"`, `"DXTD"`, `"DXTE"`, `"DXTF"` — Direct3D surface formats

---

### 9. Direct3D API Dispatch Table (0x0067218c)
**Size:** 96 entries × 4 bytes = 384 bytes  
**Xrefs:** 209 — heavily used  
**Type:** Function pointer table for d3d8.dll, d3dx8.dll imports

---

### 10. Configuration/Flag Arrays (0x006b0860)
**Xrefs:** 88  
**Content:** Word array with values 0x0000, 0x0001, 0x0002, 0xFFFF; floats 0.5f, 0.8f; ints 12, 1, 16, 20, 4, 8, 24, 28  
**Purpose:** Game difficulty/settings configuration

### Texture Loading Params (0x006b09fc)
**Xrefs:** 188  
**Content:** `"Error occurred during rendering"`, `"LOCAL:common/textures/linefilter1.tga"`, words 32, 28, 274, 514, 8

### Alpha Blend Config (0x006b6718)
**Xrefs:** 164  
**Content:** `"abcdefghijklmnop...z"` (alpha table), 1.0f repeated, words 0x0002, 0x0001, 0xFFFF

### Sound Control Coefficients (0x006b7cd8)
**Xrefs:** 113  
**Content:** `"Start sound"`, `"Stop sound"`, `"Move sound"`, floats 0.5f, 2.0f

---

## Runtime Globals (BSS)

| Address | Size | Purpose | Xrefs |
|---------|------|---------|-------|
| 0x006bf6c0 | ~472 bytes | Runtime config | BSS |
| 0x006c0638 | ~712 bytes | WinMain-initialized globals | BSS |
| 0x006c0790 | ~256 bytes | Additional runtime globals | BSS |
| 0x00704884 | ~256 bytes | File type dispatch table | BSS |
| 0x0070f464 | 4 bytes | Flag/state | 317 |
| 0x0070f46c | 4 bytes | Flag/state | 159 |
| 0x0071a604 | ~256 bytes | Menu screen stack | BSS |
| 0x0072bc48 | ~256 bytes | Game state array | BSS |
| 0x0073fb4c | 4 bytes | FPU control word | BSS |
| 0x08350000+ | ~12KB | Runtime-allocated data | BSS |

---

## Entity/Actor Struct (0xd80 bytes ≈ 3456 bytes)

Accessed via FUN_004b0a00:

| Offset | Purpose | Type |
|--------|---------|------|
| 0x52c-0x530 | Linked list pointers | DWORD* |
| 0x540 | Entity type/ID | DWORD |
| 0x544 | State flags (set to 3) | DWORD |
| 0x564 | Active flag | DWORD* |
| 0x568 | Reference to sub-entity | DWORD* |
| 0x6e0 | Position/rotation Y | float |
| 0x700 | Camera reference | DWORD* |
| 0x74c | Animation state pointer | struct* (+0x7c = 10 floats) |
| 0xcb4 | Secondary position | float/DWORD |
| 0xc18 | World position | float* |
| 0xc1c | World rotation | float* |
| 0xc2c-0xc30 | State reset | DWORD |
| 0xc34 | Action timer | DWORD |
| 0xc38 | Action type | DWORD |
| 0xd34 | Action parameter (0x14) | DWORD |
| 0xd68 | Reference pointer | DWORD* |
| 0xd80-0xd88 | Action parameters | DWORD |

---

## Key Addresses Summary

| Priority | Address Range | Content |
|----------|--------------|---------|
| CRITICAL | 0x006b8558-0x006b8ce8 | AI State machine (14 entries) |
| HIGH | 0x006ba1e4-0x006ba2a0 | Input/action dispatch |
| HIGH | 0x0068ca3c-0x0068ca94 | Weapon system strings |
| HIGH | 0x006b0860-0x006b095c | Settings/flags config |
| HIGH | 0x006ab000-0x006ab600 | Texture format tables |
| MEDIUM | 0x00685b68-0x00685c80 | Debug console strings |
| MEDIUM | 0x0068b888-0x0068b8d0 | HUD/UI strings |
