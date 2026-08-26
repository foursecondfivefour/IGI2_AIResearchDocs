# Reverse Engineering Project IGI 2 — Structs & Data Types

**Status:** ⚠️ INITIAL  
**Structs Defined:** 0  
**Confidence:** —

---

## Pending Struct Definitions

### HumanPlayer (estimated)

```c
struct HumanPlayer {
    // Base class (Entity/Character)
    Vector3 position;          // 0x000 - World position
    Vector3 angles;            // 0x00C - Euler angles (yaw, pitch, roll)
    Vector3 velocity;          // 0x018 - Movement velocity
    float health;              // 0x024 - Current health
    float maxHealth;           // 0x028 - Maximum health
    int state;                 // 0x02C - Current state machine state
    int previousState;         // 0x030 - Previous state
    void* stateTable;          // 0x034 - State machine vtable
    void* vtable;              // 0x038 - Class vtable pointer
    
    // Combat
    int currentWeapon;         // 0x03C - Currently equipped weapon ID
    int ammo[5];               // 0x040 - 5 ammo types
    int kills;                 // 0x054 - Enemy kill count
    int detected;              // 0x058 - Detection level (0=undetected)
    
    // Animation
    int animFrame;             // 0x05C - Current animation frame
    float animProgress;        // 0x060 - Animation blending progress
    
    // Interaction
    int interactTarget;        // 0x064 - Current interactable object
    int stealthMode;           // 0x068 - Stealth mode flag
};
```

### HumanSoldier (estimated)

```c
struct HumanSoldier {
    // Base class (Entity/Character)
    Vector3 position;          // 0x000 - World position
    Vector3 angles;            // 0x00C - Euler angles
    Vector3 velocity;          // 0x018 - Movement velocity
    float health;              // 0x024 - Current health
    float maxHealth;           // 0x028 - Maximum health
    int state;                 // 0x02C - AI state
    int enemy;                 // 0x030 - Current target enemy
    void* stateTable;          // 0x034 - AI state table
    void* vtable;              // 0x038 - Class vtable pointer
    
    // AI
    int alertLevel;            // 0x03C - Alertness (patrol/search/combat)
    int patrolIndex;           // 0x040 - Current patrol waypoint
    int lastKnownEnemyPos;     // 0x044 - Last seen player position
    float sightRange;          // 0x048 - Vision cone range
    float hearingRange;        // 0x04C - Hearing distance
    int hasWeapon;             // 0x050 - Equipped weapon
    
    // Combat
    int currentWeapon;         // 0x054 - Weapon ID
    int fireCooldown;          // 0x058 - Weapon fire timer
    int grenadeCount;          // 0x05C - Grenades remaining
};
```

### GameState (estimated singleton)

```c
struct GameState {
    int missionNumber;         // 0x000 - Current mission (1-14)
    int missionObjective;      // 0x004 - Current objective index
    int objectiveComplete;     // 0x008 - Bitmask of completed objectives
    int missionFailed;         // 0x00C - Mission failed flag
    int missionCompleted;      // 0x010 - Mission success flag
    int gamePaused;            // 0x014 - Pause flag
    int gameOver;              // 0x018 - Game over flag
    int timeRemaining;         // 0x01C - Mission timer
    int score;                 // 0x020 - Player score
    int stealthRating;         // 0x024 - Stealth score
};
```

### Vector3

```c
struct Vector3 {
    float x;                   // 0x000
    float y;                   // 0x004
    float z;                   // 0x008
};
```

### Rectangle / Screen Bounds

```c
struct Rect {
    int left;                  // 0x000
    int top;                   // 0x004
    int right;                 // 0x008
    int bottom;                // 0x00C
};
```

---

## Known Data Addresses (TBD Types)

| Address | Current Type | Suspected Type | Purpose |
|---------|-------------|----------------|---------|
| 0x006bf6c4 | undefined4 | int | Runtime flag |
| 0x006bf6d0 | undefined4 | int | Stack size |
| 0x006bf6c8 | undefined4 | uint | Command line flags |
| 0x006bf6cc | undefined4 | int | Return address data |
| 0x006bf6d4 | undefined4 | int | Extra return data |
| 0x006bf708 | undefined* | char* | Parsed args |
| 0x006bf6e4 | undefined* | int* | Config KV pairs |
| 0x006bf710 | undefined | int | Cleanup flag |
| 0x08352060 | undefined4 | void* | Process heap |
| 0x08352068 | undefined4 | char* | Command line |
| 0x08351cec | undefined4 | int | Handle count |
| 0x08351d00 | undefined4 | void* | Handle table |
| 0x006aee38 | undefined4 | code* | Early init callback |
| 0x006aecd4 | undefined4 | int | Early init flag |
| 0x006bf898 | undefined4 | int | Environment cache |
| 0x006c0798 | undefined4 | HINSTANCE | Module handle |
| 0x006c0790 | undefined4 | HWND | Window handle |
| 0x006c06f8 | undefined | bool | Exit flag |
| 0x006c07d4 | undefined | int | Running flag |
| 0x006c07d8 | undefined | int | Pause flag |
| 0x0073fb4c | undefined4 | uint | FPU control word |

---

## Next Steps

- [ ] Analyze vtable data at 0x006aa4a0 for struct layouts
- [ ] Reverse struct fields from function parameter patterns
- [ ] Apply struct types in Ghidra
- [ ] Document all member variable usage patterns
