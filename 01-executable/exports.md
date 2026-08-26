# IGI 2: Covert Strike — Export Table

**Status:** 🟡 PARTIAL  
**Total Exports:** 38 (1 entry + 16 class methods + 16 ordinals + 5 version)  
**Confidence:** HIGH

---

## Entry Point

```
entry → 0x0065d36d
```

---

## C++ Class State Machine Exports

All exports follow the naming convention:
```
CDAPFN{NNNN}_{ClassName}_InitState{StateName} → {Address}
CDAPFN{NNNN}_CDAPFN{NNNN}_X_{ClassName}_InitState{StateName} → {Address}
Ordinal_{N} → {Address}
```

### Module: CDAPFN0506

This namespace/class contains player and soldier state initialization functions.

---

### HumanPlayer Class (10 states)

The `HumanPlayer` class manages the player character's state machine.

#### Interaction States

| Export Name | Address | Ordinal | Description |
|-------------|---------|---------|-------------|
| CDAPFN0506_HumanPlayer_InitStateActivateDoor | 0x006b84d0 | 12 | Activate/interact with doors |
| CDAPFN0506_HumanPlayer_InitStateActivateCabinet | 0x006b85e0 | 11 | Open cabinets/storage |
| CDAPFN0506_HumanPlayer_InitStateActivateTerminal | 0x006b86f0 | 16 | Use computer terminals |

#### Combat States

| Export Name | Address | Ordinal | Description |
|-------------|---------|---------|-------------|
| CDAPFN0506_HumanPlayer_InitStateActivateStationaryGun | 0x006b8800 | 15 | Use mounted/stationary guns |
| CDAPFN0506_HumanPlayer_InitStateSilentKill | 0x006b8c40 | 17 | Stealth takedown animation |

#### Object/Item Interaction

| Export Name | Address | Ordinal | Description |
|-------------|---------|---------|-------------|
| CDAPFN0506_HumanPlayer_InitStateActivateGenericTBA | 0x006b8910 | 13 | Generic interactable object |
| CDAPFN0506_HumanPlayer_InitStateActivateC4BombTBA | 0x006b8a20 | 10 | Place C4 explosives |
| CDAPFN0506_HumanPlayer_InitStateActivatePlaceExplosiveTBA | 0x006b8b30 | 14 | Place other explosives |

#### Vtable Dispatch Functions (X_ prefix)

These are likely the virtual method table entries or thunk functions:

| Export Name | Address | Ordinal | Notes |
|-------------|---------|---------|-------|
| CDAPFN0506_CDAPFN0506_X_HumanPlayer_InitStateActivateCabinet | 0x006b8668 | 2 | Vtable dispatch |
| CDAPFN0506_CDAPFN0506_X_HumanPlayer_InitStateActivateDoor | 0x006b8558 | 3 | Vtable dispatch |
| CDAPFN0506_CDAPFN0506_X_HumanPlayer_InitStateActivateGenericTBA | 0x006b8998 | 4 | Vtable dispatch |
| CDAPFN0506_CDAPFN0506_X_HumanPlayer_InitStateActivatePlaceExplosiveTBA | 0x006b8bb8 | 5 | Vtable dispatch |
| CDAPFN0506_CDAPFN0506_X_HumanPlayer_InitStateActivateStationaryGun | 0x006b8888 | 6 | Vtable dispatch |
| CDAPFN0506_CDAPFN0506_X_HumanPlayer_InitStateActivateTerminal | 0x006b8778 | 7 | Vtable dispatch |
| CDAPFN0506_CDAPFN0506_X_HumanPlayer_InitStateSilentKill | 0x006b8cc8 | 8 | Vtable dispatch |
| CDAPFN0506_CDAPFN0506_X_HumanPlayer_InitStateActivateC4BombTBA | 0x006b8aa8 | 1 | Vtable dispatch |

---

### HumanSoldier Class (1 state)

| Export Name | Address | Ordinal | Description |
|-------------|---------|---------|-------------|
| CDAPFN0506_HumanSoldier_InitStateStandThrowGrenade | 0x006b8090 | 18 | Enemy soldier throws grenade |
| CDAPFN0506_CDAPFN0506_X_HumanSoldier_InitStateStandThrowGrenade | 0x006b8118 | 9 | Vtable dispatch |

---

## Version Info Functions (from .rsrc)

```
VerQueryValueA → 0x0061f0b0
GetFileVersionInfoSizeA → 0x0061f0b6
GetFileVersionInfoA → 0x0061f0bc
```

---

## Known Issues

### Corrupted Export Code

The state machine init functions at `0x006b8090` and `0x006b84d0` contain **unparseable instruction data**:

```
WARNING: Control flow encountered bad instruction data
WARNING: Instruction at (ram,0x006b8212) overlaps instruction at (ram,0x006b8211)
halt_baddata()
```

**Possible causes:**
1. **Packed/encrypted exports** — the function bodies are encrypted and decrypt at runtime
2. **Custom calling convention** — non-standard code layout
3. **JMP thunk** — these addresses may actually be jump tables or data, not code
4. **Vtable data** — the ordinals might reference vtable entries that are data, not code

**Recommended approach:**
- Search for the actual function bodies by looking for callers of these ordinal exports
- The real code may be located elsewhere, with these exports being thin thunks

---

## Class Hierarchy (from exports)

```
CDAPFN0506 (export namespace)
│
├── HumanPlayer
│   ├── State: ActivateDoor
│   ├── State: ActivateCabinet
│   ├── State: ActivateTerminal
│   ├── State: ActivateStationaryGun
│   ├── State: ActivateGenericTBA
│   ├── State: ActivateC4BombTBA
│   ├── State: ActivatePlaceExplosiveTBA
│   └── State: SilentKill
│
└── HumanSoldier
    └── State: StandThrowGrenade
```

**Pattern:** All states follow `InitState{StateName}` naming, suggesting a state machine pattern where each state has its own initialization function.

---

## Next Steps

- [ ] Find actual code bodies for these exported functions (search callers)
- [ ] Discover more classes — these 2 classes are likely a subset
- [ ] Identify vtable layouts from ordinal references
- [ ] Cross-reference with research data for expected state machines
