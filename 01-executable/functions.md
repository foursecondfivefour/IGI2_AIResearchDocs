# Phase 1: Executable & Core PE Analysis

**Status:** 🟡 PARTIAL  
**Functions Documented:** 8 core functions  
**Confidence:** MEDIUM

---

## 1.1 Entry Point Analysis

**Address:** 0x0065d36d  
**Function:** `entry(uint param_1, int param_2)`

The entry point is a standard MSVC 6.0 CRT startup that:

1. Sets up exception handling (`FUN_00665468`)
2. Configures stack allocation (`FUN_00660fb0`)
3. Calls `GetVersionExA` for OS detection
4. Parses command-line arguments
5. Validates PE headers (checks for PE32 format at 0x4550)
6. Calls module initialization array at 0x006ab000
7. Sets up CRT handles (stdin/stdout/stderr)
8. Calls global destructors

### Key Globals Initialized at Entry:

| Address | Variable | Type | Description |
|---------|----------|------|-------------|
| 0x006bf6c4 | `_DAT_006bf6c4` | int | Runtime flag (param_2 value, 0-2) |
| 0x006bf6d0 | `_DAT_006bf6d0` | int | Stack size (param_1 >> 8) |
| 0x006bf6c8 | `_DAT_006bf6c8` | uint | Command line flags |
| 0x08352068 | `_DAT_08352068` | char* | Full command line (GetCommandLineA) |
| 0x08352060 | `_DAT_08352060` | void* | Process heap (HeapCreate) |
| 0x006bf708 | `_DAT_006bf708` | char* | Parsed arguments/env strings |
| 0x006bf6e4 | `_DAT_006bf6e4` | int* | Config key-value pairs |

---

## 1.2 CRT Initialization Functions

### FUN_00665468 — Exception Handler Setup
```c
void FUN_00665468(undefined4 param_1, int param_2);
```
Sets up structured exception handling frame (RtlUnwind-style).  
Pushes EBX, ESI, EDI, return address onto stack.

### FUN_00660fb0 — Stack Frame Management
```c
void FUN_00660fb0(void);
```
Standard MSVC 6.0 stack allocation for large frames (>0x1000 bytes).

### FUN_00663465 — Heap Creation
```c
undefined4 FUN_00663465(int param_1);
```
Creates the main process heap at `0x08352060` via `HeapCreate()`.

### FUN_00665299 — CRT Handle Setup
```c
undefined4 FUN_00665299(void);
```
Initializes stdin/stdout/stderr file handles. Sets up 64-file-handle table  
at `0x08351d00` (256 bytes, 64 entries of 8 bytes each: handle + attributes).

### FUN_00664ea2 — Configuration Parser
```c
undefined4 FUN_00664ea2(void);
```
Parses environment variables or command-line arguments.  
Format: `KEY=VALUE` pairs stored in array at `0x006bf6e4`.

### FUN_00664a2f — Global Finalization
```c
void FUN_00664a2f(void);
```
Calls global destructor array and cleanup handlers.

### FUN_00664c58 — Cleanup Handler
```c
void FUN_00664c58(void);
```
Called during shutdown if `_DAT_006bf710 == 1`. Calls cleanup at `0x006bf784`.

---

## 1.3 Module Initialization Array

**Location:** 0x006ab000 - 0x006ab028 (8 entries)

The game uses a function pointer array for module initialization. Called from `FUN_0065d167`:

```
Offset  | Address (Raw Value) | Notes
--------|---------------------|----------------------------
0x006ab000 | NULL (0x00000000) | Reserved/skipped
0x006ab004 | 0x00623089        | Init function 1
0x006ab008 | NULL (0x00000000) | Reserved/skipped
0x006ab00c | 0x00626270        | Init function 2
0x006ab010 | 0x00621e32        | Init function 3
0x006ab014 | 0x00624990        | Init function 4
0x006ab018 | 0x00626c09        | Init function 5
0x006ab01c | 0x0062754e        | Init function 6
0x006ab020 | 0x0062dd6f        | Init function 7
0x006ab024 | NULL (0x00000000) | Terminator
```

Called from `FUN_0065d167`:
1. First checks `0x006aee38` (early init callback)
2. Iterates through `0x6ab00c` - `0x6ab028` (module inits)
3. Then checks `0x6ab000` - `0x6ab008` (late inits)

---

## 1.4 Import Table Analysis

### KERNEL32.DLL (48 imports)
Core Windows API: memory management, file I/O, process/thread, time, locale.

| Function | Purpose |
|----------|---------|
| VirtualAlloc/VirtualFree/VirtualProtect/VirtualQuery | Memory management |
| HeapCreate/HeapDestroy/HeapAlloc/HeapFree/HeapReAlloc | Custom heap manager |
| CreateFileA/ReadFile/WriteFile/SetFilePointer/FindFirstFileA/FindNextFileA/FindClose | File I/O |
| LoadLibraryA/GetProcAddress/FreeLibrary | Dynamic loading |
| CreateThread/GetExitCodeThread/CreateMutexA/ReleaseMutex/CloseHandle | Threading |
| GetTickCount/QueryPerformanceCounter/QueryPerformanceFrequency | Timing |
| Sleep/GetLastError/SetLastError | System utilities |
| GetModuleHandleA/GetModuleFileNameA/SetCurrentDirectoryA | Module/working directory |
| GetCommandLineA | Command-line access |
| GetStartupInfoA | Startup configuration |
| GetVersionExA | OS version detection |
| GetLocaleInfoA/W, CompareStringA/W, LCMapStringA/W | Locale/collation |
| GetACP/GetOEMCP/GetCPInfo | Codepage handling |
| SetUnhandledExceptionFilter | Exception handling |
| TerminateProcess/ExitProcess | Process termination |
| GetStdHandle/GetFileType/SetHandleCount | Console handles |
| OutputDebugStringA | Debug logging |

### USER32.DLL
| Function | Purpose |
|----------|---------|
| MessageBoxA | Dialog/MessageBox |
| GetEnvironmentStrings/FreeEnvironmentStrings | Environment vars |

### GDI32.DLL
*(Import exists, usage TBD)*

### WINMM.DLL
| Function | Purpose |
|----------|---------|
| mixerSetControlDetails | Mixer controls |
| mciSendCommandA | MCI device control (CD, MIDI) |

### WININET.DLL (5 imports)
HTTP/Internet access:
| Function | Purpose |
|----------|---------|
| InternetOpenA | Internet session |
| InternetOpenUrlA | HTTP URL open |
| InternetReadFile | HTTP read |
| InternetSetFilePointer | HTTP seek |
| InternetCloseHandle | HTTP close |

### WSOCK32.DLL
*(Import exists — network socket support)*

### MSS32.DLL (Miles Sound System)
**37 AIL_* functions** — Full 3D audio integration.

### VERSION.DLL
| Function | Purpose |
|----------|---------|
| VerQueryValueA | Version info extraction |
| GetFileVersionInfoA/GetFileVersionInfoSizeA | File version query |

---

## 1.5 Export Table Analysis

**Total Exports:** 38 entries (entry + 36 C++ methods + ordinals)

### State Machine Initialization Functions

All exports follow the pattern:
- `CDAPFN{NNNN}_{ClassName}_InitState{StateName}` — Direct state init
- `CDAPFN{NNNN}_CDAPFN{NNNN}_X_{ClassName}_InitState{StateName}` — Vtable/virtual dispatch
- `Ordinal_{N}` — Ordinal export alias

### HumanPlayer States (10 exported):

| Export | Ordinal | State Type |
|--------|---------|------------|
| HumanPlayer_InitStateActivateDoor | 12 | Interaction |
| HumanPlayer_InitStateActivateCabinet | 11 | Interaction |
| HumanPlayer_InitStateActivateTerminal | 16 | Interaction |
| HumanPlayer_InitStateActivateStationaryGun | 15 | Combat |
| HumanPlayer_InitStateActivateGenericTBA | 13 | Object use |
| HumanPlayer_InitStateActivateC4BombTBA | 10 | Explosive |
| HumanPlayer_InitStateActivatePlaceExplosiveTBA | 14 | Explosive |
| HumanPlayer_InitStateSilentKill | 17 | Stealth |
| CDAPFN0506_X_HumanPlayer_InitStateActivateDoor | 3 | Vtable dispatch |
| CDAPFN0506_X_HumanPlayer_InitStateActivateCabinet | 2 | Vtable dispatch |
| CDAPFN0506_X_HumanPlayer_InitStateActivateTerminal | 7 | Vtable dispatch |
| CDAPFN0506_X_HumanPlayer_InitStateActivateStationaryGun | 6 | Vtable dispatch |
| CDAPFN0506_X_HumanPlayer_InitStateActivateGenericTBA | 4 | Vtable dispatch |
| CDAPFN0506_X_HumanPlayer_InitStateActivateC4BombTBA | 1 | Vtable dispatch |
| CDAPFN0506_X_HumanPlayer_InitStateActivatePlaceExplosiveTBA | 5 | Vtable dispatch |
| CDAPFN0506_X_HumanPlayer_InitStateSilentKill | 8 | Vtable dispatch |

### HumanSoldier States (2 exported):

| Export | Ordinal | State Type |
|--------|---------|------------|
| HumanSoldier_InitStateStandThrowGrenade | 18 | Combat |
| CDAPFN0506_X_HumanSoldier_InitStateStandThrowGrenade | 9 | Vtable dispatch |

### Class Hierarchy Deduction:

```
CDAPFN0506 (module/class namespace)
├── HumanPlayer (player character, 10 states)
│   ├── ActivateDoor — Interact with doors
│   ├── ActivateCabinet — Open cabinets/containers
│   ├── ActivateTerminal — Use computer terminals
│   ├── ActivateStationaryGun — Use mounted guns
│   ├── ActivateGenericTBA — Generic interaction object
│   ├── ActivateC4BombTBA — Place C4 explosives
│   ├── ActivatePlaceExplosiveTBA — Place other explosives
│   └── SilentKill — Stealth takedown
│
└── HumanSoldier (enemy NPC)
    └── InitStateStandThrowGrenade — Grenade combat state
```

**Note:** The exported state init functions at addresses 0x006b8090+ contain  
**CORRUPTED/ENCRYPTED CODE** — likely protected by a packing layer or  
custom code obfuscation. The ordinals (1-18) suggest a vtable-based  
dispatch mechanism.

---

## 1.6 PE Structure

### Memory Layout:
```
Segment    | Start      | End          | Size (approx) | Purpose
-----------|------------|--------------|---------------|----------
Headers    | 0x00400000 | 0x00400fff   | 4 KB          | PE headers
.rsrc      | 0x08677000 | 0x08679fff   | 12 KB         | Resources (icons, version)
.tdb       | 0xffdff000 | 0xffdfffff   | 64 KB         | Thread Local Storage block
.data1     | 0x08676000 | 0x08676fff   | 4 KB          | Initialized data
.data      | 0x06ab0000 | 0x0867559b   | ~27 MB        | Global variables, BSS
.rdata     | 0x06720000 | 0x06aaffff   | ~3.6 MB       | Read-only (strings, import table)
.text      | 0x00401000 | 0x0671ffff   | ~101 MB       | Code section
```

### Resources:
- **Icons:** 5 icons (Rsrc_Icon_1_400 through Rsrc_Icon_5_400)
- **Version Info:** VS_VERSION_INFO at Rsrc_Version_1_414
- **Group Icon:** Rsrc_GroupIcon_65_400

---

## 1.7 Next Steps

- [ ] Analyze each init function pointer (0x00626270, 0x00621e32, etc.)
- [ ] Resolve corrupted export functions — check if ordinals map to actual code
- [ ] Map DirectX 7 initialization (d3d7.dll likely loaded dynamically)
- [ ] Identify main game loop entry point
- [ ] Document all string-based function calls
