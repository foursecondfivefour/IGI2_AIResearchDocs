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

## 1.1 WinMain/CRT Startup

**Address:** 0x00404280  
**Function:** `WinMain_CRTStartup(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nShowCmd)`

The WinMain entry point that handles:

1. **Mutex Check** — Creates "InnerloopGameMUTEX" to prevent multiple instances
   - Error: "InnerloopGame is already running." if mutex exists
2. **COM Initialization** — `CoInitialize(NULL)`
3. **Window Class Registration** — WNDCLASSA "Innerloop" class
   - Icon: 0x7f00, Cursor: 0x7f00, Menu: "Innerloop"
   - Background: COLOR_WINDOW+1 (0x7)
   - WindowProc: LAB_004040a0
4. **Window Creation** — CreateWindowExA (448x480, centered)
   - Error: "Couldn't open window" if fails
5. **Cursor Hidden** — ShowCursor(0)
6. **FPU Control** — Sets FPU precision to 53-bit (0xc00 mask)
7. **DirectX Version Check** — Requires 5.0+ (0x801)
   - Error: "This application requires DirectX version %d.%d or greater to run."
8. **Config Callbacks** — 11 config subsystems registered via FUN_0041d9e0:
   - Window, WinInput, Track, Editor, Server, Player, Password, Zbias, Config, SessionDir, FPSLock
9. **Config Strings** — "filesys.cfg", "Innerloop", "MR_NOT_NAMED", "networkconfig.cfg"
10. **Global State** — DAT_006c0638..DAT_006c09dc (48+ entries)
11. **Game State Init** — Calls FUN_00401210 (Game_ApplicationInit)
12. **Main Message Loop** — PeekMessageA → GetMessageA → TranslateMessage → DispatchMessageA
    - Calls FUN_00401440 (Game_MainLoop) per frame
    - Sleep(2) or Sleep(0) based on DAT_006c07d4 flag
13. **Cleanup on Exit** — ShowCursor(1), UnregisterClassA, CoUninitialize, ReleaseMutex, CloseHandle

### Key Globals Managed by WinMain:

| Address | Variable | Purpose |
|---------|----------|---------|
| 0x006c0798 | param_3 | Command line / instance handle |
| 0x006c0794 | hWnd | Window handle |
| 0x006c06f8 | gameRunning | Game state flag (non-zero = running) |
| 0x006c07d4 | sleepMode | 1=Sleep(2), 0=Sleep(0) |
| 0x006c07f0 | forceQuit | Force quit flag |
| 0x006c07d8 | fullscreen | Fullscreen flag |
| 0x006c0638..0x006c0700 | configCallbacks | 48-entry callback table |

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

---

## 1.8 Function Classification: Game vs CRT/Library

### CRT/Library Functions (0x00650000 - 0x00670000) — **MUSOR**

These are Visual Studio 2003 Release CRT functions — **NOT game code**. They are part of msvcrt.dll or statically linked runtime.

| Address | Name | Library | Purpose |
|---------|------|---------|---------|
| 0x0065cc30 | FUN_0065cc30 | CRT | sprintf-like formatted output |
| 0x0065d28e | FUN_0065d28e | CRT | Exit/cleanup handler |
| 0x0065d550 | _strncpy | CRT | String copy with truncation (VS2003 Release) |
| 0x0065d674 | _abort | CRT | Abort with raise(0x16), exit(3) (VS2003 Release) |
| 0x0065d690 | _strncmp | CRT | String comparison (VS2003 Release/Debug) |
| 0x0065ccb9 | _sscanf | CRT | String parsing |
| 0x0065cd97 | _malloc | CRT | Memory allocation |
| 0x0065cced | _free | CRT | Memory deallocation |
| 0x0065cf40 | _longjmp | CRT | Long jump |
| 0x0065ce36 | FUN_0065ce36 | CRT | Error handling |
| 0x0065ce8d | FUN_0065ce8d | CRT | Error handling |
| 0x0065ceb0 | FUN_0065ceb0 | CRT | Error handling |
| 0x0065cfb9 | FUN_0065cfb9 | CRT | Error handling |
| 0x0065d1cc | FUN_0065d1cc | CRT | Exit processing |
| 0x0065d167 | FUN_0065d167 | CRT | Module init array caller |
| 0x00660ff0 | __setjmp3 | CRT | Setjmp/longjmp support |
| 0x0066201d | FUN_0066201d | CRT | Low-level I/O |
| 0x00664ab7 | FUN_00664ab7 | CRT | Error reporting |
| 0x00664514 | __security_check_cookie | CRT | Stack cookie validation |

**Total CRT functions:** ~20 identified (likely more unanalyzed)

### Game Engine Functions (0x00400000 - 0x00650000) — **GAME CODE**

These are the actual game engine functions. Key high-frequency functions:

| Address | Name | Xrefs | Category | Purpose |
|---------|------|-------|----------|---------|
| 0x0040a8f0 | QTask_RegisterType | 1962 | QTask | Register QTask type with handler table |
| 0x0040a990 | QTaskList_Add | 846 | QTask | Add QTask to linked list |
| 0x0040aa70 | QTask_Destroy | 209 | QTask | Destroy QTask with pre/post callbacks |
| 0x0040a870 | QTask_Create | 239 | QTask | Create new QTask entity |
| 0x00406920 | Game_UpdateInput | 120 | Core | Main game tick/update loop |
| 0x00402f80 | String_ParseToken | 107 | String | Parse strings with vtable dispatch |
| 0x00401000 | ParseString | 3 | String | Quoted string parsing |
| 0x00401210 | Game_ApplicationInit | 1 | Core | Application initialization |
| 0x00401440 | Game_MainLoop | 1 | Core | Main loop with setjmp |
| 0x00401490 | Game_ProcessGameState | 1 | Core | State dispatch |
| 0x00401600 | Game_LoadLevel | 1 | Level | Level loading |
| 0x00401970 | Game_UpdateInput | 2 | Input | Input processing |
| 0x00401a30 | Game_ProcessEvents | 2 | Core | Event processing |
| 0x00401b20 | Game_RunFrame | 1 | Core | Frame processing |
| 0x00402c00 | Game_ResetTiming | 7 | Core | Timing reset |
| 0x00402cf0 | Game_CalculateFrameTime | 12 | Core | Frame time calculation |
| 0x004031c0 | FUN_004031c0 | 45 | Config | Config parsing |
| 0x004036b0 | FUN_004036b0 | 35 | Config | Config validation |
| 0x00403af0 | FUN_00403af0 | 22 | Config | Config loading |
| 0x00403bb0 | FUN_00403bb0 | 11 | Config | Config saving |
| 0x004040a0 | FUN_0040aca0 | 330 | QTask | QTask type creation |
| 0x0040ad00 | FUN_0040ad00 | 20 | QTask | QTask factory |
| 0x0040ad60 | FUN_0040ad60 | 16 | QTask | QTask management |
| 0x0040adc0 | **QTaskType_ClearActive** | 234 | QTask | Clear active flag at offset in_AX*0xc |
| 0x0040ade0 | **Game_GetDebugFlag** | 176 | Core | Return global debug flag at DAT_006c0bf0 |
| 0x0040adf0 | **QTaskType_Register** | 372 | QTask | Register new QTask type |
| 0x0040af30 | **QTaskType_Dispatch** | 375 | QTask | Dispatch task type cleanup |

## 1.9 Round 7 — QTask System Deep Dive & File System (2024-08-27)

### New Renamed Functions

| Address | New Name | Xrefs | Module | Purpose |
|---------|----------|-------|--------|---------|
| 0x0040adf0 | **QTaskType_Register** | 372 | QTask | Register new QTask type. 0x34 bytes/type. 0xd slots. Copies 0x200-byte state arrays from parent |
| 0x0040af30 | **QTaskType_Dispatch** | 375 | QTask | Dispatch task type cleanup. Clears active flag, calls destructor |
| 0x0040aca0 | **GameEvent_Alloc** | 330 | Event | Allocate game event slot. 0xc bytes/event, 0x20 max. Type + data |
| 0x0040b2f0 | **QTaskList_FindByType** | 313 | QTask | Find task in linked list by type (short at offset 0x12/9*4) |
| 0x0040b3f0 | **QTaskType_IsDerivedFrom** | 206 | QTask | Check type inheritance. Traverses base chain (0x1a stride). Base=0x200 |
| 0x0040adc0 | **QTaskType_ClearActive** | 234 | QTask | Clear active flag at offset in_AX*0xc |
| 0x0040b350 | **QTaskList_FindByTypeID** | 183 | QTask | Find QTask in linked list by type ID, checks offset 9*4 |
| 0x0040b380 | **QTaskList_FindByTypeOffset** | 28 | QTask | Find QTask by type at offset 0x24 (36 bytes) |
| 0x0040bb00 | **QTaskList_DestroyAll** | 114 | QTask | Destroy all tasks, calls QTask_Destroy on each, resets list |
| 0x0040ade0 | **Game_GetDebugFlag** | 176 | Core | Return global debug flag at DAT_006c0bf0 |
| 0x0040c6d0 | **FileSys_Read** | 41 | FileSys | File read. Device handler table (0x90 bytes/device). Max 10 retries |
| 0x0040cf20 | **FileSys_Close** | 37 | FileSys | Close file handle, call device close callback |
| 0x0040d020 | **CommandLine_Process** | 32 | Core | Parse CLI args. 1024-byte buffer. Callback at DAT_00704b50 |
| 0x0040c330 | **FileSys_ParsePath** | 35 | FileSys | Parse Windows path (C:\...). Find device from ':' char |

### QTask System Architecture

The QTask system is the core task/scheduler framework with **~2000+ cross-references** across 10+ functions.

**Type Registry:**
- 0x34 bytes per type entry (callback pointers, state arrays, flags)
- 0xd (13) maximum active type slots
- 0x200-byte state arrays copied from parent type during inheritance
- Base type marker: 0x200 (terminates inheritance chain)

**Type Entry Layout (0x34 bytes):**
```
Offset  Size  Field
0x000   0x04  Callback table pointer (param_2 in Register)
0x004   0x04  Type ID (param_1, clamped to max 4)
0x008   0x02  Base type / parent type ID
0x010   0x04  Unknown flag/counter
0x014   0x04  State size (param_1 + 3 & ~3, aligned to 4)
0x018   0x04  Unknown
0x01C   0x04  Unknown
0x020   0x04  Unknown
0x024   0x04  State data (0x200 bytes total across all types)
0x...   ...   ... (continues to 0x34)
```

**Active Type Flags:**
- DAT_08498828: Active/in-use flags (1 = registered, 0 = free)
- DAT_08498829: Allocation flags

**Task List:**
- Linked list structure with next pointers
- Type stored at offset 0x12 (9*4 = 36 bytes from task start)
- Multiple find variants: by type, by type ID, by type at offset

### File System Architecture

**Device Handler Table:**
- Location: DAT_007048d4
- 0x90 bytes per device entry
- 4 device types (1=stdin, 2=stdout, 3=stderr, 4=file)
- Device name table: DAT_007048f8 (0x90 bytes each)

**Path Parsing (FileSys_ParsePath):**
1. Extract device letter from ':' character position
2. Validate device type (1-4 range)
3. Check device present flag (DAT_007048d4 + type*0x90)
4. Dispatch to device-specific handler table
5. Copy remaining path to device buffer

**FileSys_Read Operation:**
1. Get current device from FileSys_ParsePath result
2. Calculate device offset: type * 0x90
3. Check if device active (DAT_007048b0 + offset != 0)
4. Allocate buffer via FUN_0040bb30
5. Call device read callback (DAT_007048b0 + offset)
6. Retry loop: max 10 attempts with callback at DAT_00704888
7. Error: "Error while reading from file %s" on failure

### Game Event System

**GameEvent_Alloc (0x0040aca0):**
- 0xc bytes per event entry
- 0x20 (32) maximum events
- Stores: type (uint2), active flags (2 bytes), data (uint4)
- Location: DAT_00700820..DAT_007009a2
- Error: "Unable to allocate event" → abort()

### CRT Function Confirmed in Game Code

FUN_0055b820 (0x0055b820-0x0055b877) — **Pooled Memory Allocator**
- Below CRT boundary (0x00650000) = game code
- Allocates 0xc byte blocks via FUN_0054c240
- Manages pool with offset tracking (in_EAX+4, in_EAX+8)
- Calls FUN_00445270 when pool full (flush/commit)
- Stores param_1 at offset 0x08 in block
- Also allocates via FUN_00445460 when unaff_EDI != 0

## 1.10 Round 8 — Core Systems: DList, FatalError, Network, Render (2026-08-27)

### New Renamed Functions

| Address | New Name | Xrefs | Module | Purpose |
|---------|----------|-------|--------|---------||
| 0x00445540 | **DList_RemoveNode** | 852 | DList | Doubly linked list node removal. next->prev = prev, prev->next = next |
| 0x00446000 | **FatalError_Handler** | 553 | Core | Fatal error with "Fatal error:\n\n" + message, callback dispatch |
| 0x0041d470 | **Server_LogMessage** | 347 | Network | Timestamped server logging [%02d:%02d:%02d], max 0x1e messages |
| 0x00445460 | **DList_InsertNode** | 217 | DList | Doubly linked list node insertion. node->prev/next, update neighbors |
| 0x00425740 | **ResourceManager_Load** | 143 | Resource | GLOB:/ and LOCA:/?ENGLISH/ paths, String_ParseToken, FileSys_Read |
| 0x00427730 | **Render_DrawParticles** | 162 | Render | D3D particle/effects rendering, FUN_00589f80, FUN_0058a160, FUN_005f8040 |
| 0x004194c0 | **QTaskList_FindByTypeAndCallback** | 183 | QTask | Find QTask by type, then traverse with callback at offset 0x28 |
| 0x00448530 | **D3D_ErrorString** | 129 | Render | D3DERR_* and E_* HRESULT to string converter, switch table |
| 0x00430730 | **SoundDef_Find** | 100 | Audio | Sound definition with GameEvent_Alloc("FindSoundDef"), QTask_Create |
| 0x00435820 | **Performance_GetCounter** | 231 | Core | QueryPerformanceCounter wrapper, divides by frequency, *1000 for ms |
| 0x00419300 | **Network_SendPacket** | 75 | Network | Network packet sender, 8180-byte buffer, "Sending: %d, %c%c%c%c" |
| 0x004378e0 | **EventDispatcher_Dispatch** | 61 | Event | Event handler table dispatch, DAT_08350bb4+8, DAT_083997e0 |
| 0x0041f310 | **QTaskList_Add** | 60 | QTask | Array-based task list append with overflow check (in_EAX[1] <= *in_EAX) |
| 0x0041f340 | **QTaskList_Remove** | 56 | QTask | Array-based task list removal by value, linear search, shift |

### Doubly Linked List (DList)

**Core data structure** with 852+ cross-references — the most used pattern in the binary.

**DList_RemoveNode (0x00445540):**
```c
void DList_RemoveNode(DListNode *node) {
    node->next->prev = node->prev;
    node->prev->next = node->next;
    node->next = 0;
    node->prev = 0;
}
```

**DList_InsertNode (0x00445460):**
```c
void DList_InsertNode(DListNode *node, void *next, void *prev) {
    node->prev = *(void**)(param_1 + 8);  // prev
    node->next = param_1 + 4;             // next
    **(void***)(param_1 + 8) = node;      // prev->next = node
    *(void**)(param_1 + 8) = node;        // prev = node
}
```

### Fatal Error System

**FatalError_Handler (0x00446000):**
- 256-byte buffer at DAT_084a01a0
- Formats "Fatal error:\n\n" + user message
- Calls error display callback at DAT_0073fb38
- Calls error handler at DAT_0073fb30 (may abort or show dialog)

### Server Logging

**Server_LogMessage (0x0041d470):**
- Timestamp: [%02d:%02d:%02d] using _gmtime
- Writes to file device (DAT_08350c0c)
- Server log: "Server: %s" with max 0x1e messages
- Traffic suppression after 0x1e messages

### Resource Manager

**ResourceManager_Load (0x00425740):**
- Path formats: "GLOB:/filename", "LOCA:/?ENGLISH/filename"
- Maintains linked list at DAT_0070b304
- Uses String_ParseToken for path parsing
- FileSys_Read for actual loading
- Error: "Failed to load resource: '%s'"

### Network System

**Network_SendPacket (0x00419300):**
- 8180-byte stack buffer
- Sends to DAT_08350c6c+0x40 (server address)
- Logs: "Sending: %d, %c%c%c%c, to %x, %d bytes, Guaranteed"
- Packet type: first 4 bytes as characters
- Calls FUN_004437f0 for actual send

### Rendering System

**Render_DrawParticles (0x00427730):**
- D3D rendering calls: FUN_00589f80 (DrawPrimitive), FUN_0058a160 (DrawIndexedPrimitive)
- Screen dimensions: DAT_083521c4 (width), DAT_083521c8 (height)
- Uses FUN_005f8040 for state setup
- Particle count from param_1[4]+6 (short at offset 6)
- 162 xrefs = core render loop

### Performance Counter

**Performance_GetCounter (0x00435820):**
- QueryPerformanceCounter wrapper
- Frequency stored at DAT_0072bc40/44 (64-bit)
- Converts to milliseconds: (counter / frequency) * 1000
- FatalError if no performance counter available

### Event Dispatcher

**EventDispatcher_Dispatch (0x004378e0):**
- Handler table at DAT_08350bb4+8
- Callback table at DAT_083997e0
- Event IDs at (&DAT_0072bc58)[event_id]
- Dispatches via indirect call with (short)puVar3[9] * 0x1ff offset

### QTaskList (Array-Based)

**QTaskList_Add (0x0041f310):**
- Array at in_EAX+2 (pointer to task pointers)
- Count at *in_EAX, capacity at in_EAX[1]
- Overflow check: in_EAX[1] <= *in_EAX
- Error: "QTaskList is full" → FatalError_Handler

**QTaskList_Remove (0x0041f340):**
- Linear search for unaff_EDI value
- Shift remaining elements down
- Decrement count

### Coverage Update

| Metric | Before | After | Change |
|--------|--------|-------|--------||
| Renamed functions | 830 | 844 | +14 |
| Coverage | 15.0% | 15.3% | +0.3% |
| DList functions | 0 | 2 | +2 |
| Network functions | 0 | 2 | +2 |
| Render functions | 0 | 2 | +2 |
| Event functions | 0 | 1 | +1 |
| Performance functions | 0 | 1 | +1 |
| Resource functions | 0 | 1 | +1 |


### Coverage Update

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Renamed functions | 816 | 830 | +14 |
| Total functions | 5507 | 5507 | 0 |
| Coverage | 14.7% | 15.0% | +0.3% |
| QTask functions identified | 8 | 14 | +6 |
| FileSys functions identified | 0 | 3 | +3 |
| Event system functions | 0 | 1 | +1 |

### Boundary Analysis

**CRT/Game Boundary:** ~0x00650000

Functions below 0x00650000 are game engine code. Functions above are CRT/library code.

**Key Evidence:**
1. **Compiler identification:** CRT functions marked as "Visual Studio 2003 Release"
2. **Calling conventions:** CRT uses `__cdecl`, game uses `__thiscall`/`__fastcall`
3. **Function patterns:** CRT has standard library patterns (strncpy, sscanf, malloc, free)
4. **Debug patterns:** 0xcdcdcdcd in QTask_Destroy confirms MSVC debug heap usage
5. **Security cookie:** __security_check_cookie at 0x00664514 confirms MSVC runtime

### Summary Statistics

| Category | Address Range | Functions (est.) | Purpose |
|----------|---------------|------------------|---------|
| Game Engine | 0x00400000 - 0x00650000 | ~4800 | Core game logic |
| CRT/Library | 0x00650000 - 0x00670000 | ~200 | C runtime, standard library |
| IAT/Thunks | 0x00670000 - 0x00680000 | ~200 | Import address table |
| Data/Strings | 0x00680000 - 0x006b0000 | N/A | Global data, strings |
| Export Table | 0x006b8000 - 0x006b9000 | 38 | Exported C++ methods |
| Resources | 0x08677000+ | N/A | Icons, version info |

## 1.11 Round 9 — Memory, Event, Config, String Systems (2026-08-27)

### New Renamed Functions

| Address | New Name | Xrefs | Module | Purpose |
|---------|----------|-------|--------|---------||
| 0x0040ac10 | **EventDispatcher_DispatchCallback** | 46 | Event | Callback lookup via DAT_083997e0 + type*0x1ff, linked list traversal |
| 0x0040dea0 | **String_ParseInput** | 46 | String | 2052-byte stack buffer, parse/convert pipeline, 0x800 flag |
| 0x004451b0 | **Memory_ValidateBlock** | 47 | Memory | Alignment check ((uVar1-1)&uVar1==uVar1), bounds validation, aligned return |
| 0x00445270 | **MemoryPool_Allocate** | 33 | Memory | 0xc-byte header + block pool, fills from callback table |
| 0x004335d0 | **Config_LookupString** | 49 | Config | Case-insensitive __stricmp search, 0x44-byte entries, 0x44 max |
| 0x00425a10 | **ResourceManager_LoadAll** | 64 | Resource | Calls ResourceManager_Load(1) to load all resources |
| 0x004637c0 | **RenderTask_Cleanup** | 30 | Render | Callback at unaff_ESI+0x24, DList_RemoveNode, reinsert at unaff_ESI+0x30 |
| 0x0040e300 | **TaskCallback_Dispatch** | 120 | QTask | Indexed callback lookup, FUN_005697f0 or FUN_005661c0 |
| 0x0040baa0 | **PooledAllocator_Alloc** | 161 | Memory | Wrapper around FUN_0055b820 (pool allocator), returns 0 if null |
| 0x0041f3f0 | **Memory_Allocate** | 27 | Memory | Wrapper around FUN_0054c240, allocates size*4+8 bytes |
| 0x0041f410 | **Memory_Deallocate** | 27 | Memory | 0xcdcdcdcd debug fill, DList_RemoveNode, _free |
| 0x00463680 | **RenderObject_Create** | 27 | Render | Allocates via FUN_00463630, initializes 13 fields, DList_InsertNode |

### Memory Management System

**Memory_ValidateBlock (0x004451b0):**
- Alignment check: `((uVar1-1) & uVar1) == uVar1` (power of 2)
- Bounds validation: checks unaff_EDI - param_2 < size
- Returns aligned address or 0
- Sets DAT_08350b24 = 1 on validation failure

**MemoryPool_Allocate (0x00445270):**
- Allocates 0xc-byte header + block
- Header: {callback_ptr, block_ptr, size}
- Fills pool with callbacks from unaff_ESI+0x14 table
- Updates counter at unaff_ESI+8

**Memory_Allocate (0x0041f3f0):**
- Simple wrapper around FUN_0054c240
- Allocates unaff_ESI*4+8 bytes
- Returns allocated pointer

**Memory_Deallocate (0x0041f410):**
- MSVC debug pattern: fills freed memory with 0xcdcdcdcd
- Decrements DAT_079fd620 (active allocation counter)
- Calls DList_RemoveNode to unlink from tracking list
- Calls _free from CRT

**PooledAllocator_Alloc (0x0040baa0):**
- Wrapper around FUN_0055b820
- Returns 0 if in_EAX == 0
- 161 xrefs = heavily used allocation path

### Event Dispatcher System

**EventDispatcher_DispatchCallback (0x0040ac10):**
- __thiscall member function
- Traverses linked list via in_EAX+0x20
- Looks up callback at DAT_083997e0 + type*0x1ff
- Calls callback with (iVar1, param_2)

**TaskCallback_Dispatch (0x0040e300):**
- 120 xrefs = core task system
- If param_1+4 != 0: calls FUN_005697f0(&local_4, 4, 1)
- Otherwise: indexed lookup at *(param_1+8) + in_EAX*4

### String Parsing System

**String_ParseInput (0x0040dea0):**
- 2052-byte stack buffer (local_804)
- 36-byte temp buffer (local_828)
- Calls FUN_0056d450 (strlen), FUN_0056e2d0 (parse), FUN_005670c0 (convert)
- Sets DAT_0849f31c = 0 on completion

### Config System

**Config_LookupString (0x004335d0):**
- Case-insensitive search via __stricmp
- Table at unaff_EBX+0x2fc, 0x44 bytes per entry
- 0x44 entries max (short at unaff_EBX+0x26be)
- Returns value at unaff_EBX+0x33c + iVar2*0x44

### Render System

**RenderObject_Create (0x00463680):**
- Allocates via FUN_00463630
- Initializes 13 fields (offsets 0-0xd)
- param_1 at offset 6, param_2 at offset 0xc, param_3 at offset 5
- DAT_08350abc at offset 3
- Sets flag at offset 0xd = 1
- Inserts into DList

**RenderTask_Cleanup (0x004637c0):**
- Calls callback at unaff_ESI+0x24
- DList_RemoveNode to unlink
- Reinserts at unaff_ESI+0x30 with DList_InsertNode
- Resets flag at unaff_ESI+0x34

### ResourceManager

**ResourceManager_LoadAll (0x00425a10):**
- Simple wrapper: calls ResourceManager_Load(1)
- 64 xrefs = common operation

### Script VM System (Round 10)

**ScriptVM_AddParameter (0x005772d0) — 1754 xrefs:**
- Add parameter to script VM context
- 0x14-byte entry: [value:4][type:4][type_name:4][flag:1][pad:3]
- "Too many parameters" / "Undefined parameter type '%s'" checks
- Increments counter at in_EAX

**ScriptVM_ReadParameter (0x005697f0) — 518 xrefs:**
- Read parameter from script VM context
- param_3=1: returns string pointer (allocates via FUN_00567b80)
- param_3=2: returns raw value
- param_3=3: strncpy to buffer
- Returns 1 on success, 0 on failure

**ScriptVM_EvaluateExpression (0x005661c0) — 303 xrefs:**
- Int expression evaluator, recursive descent
- Opcodes: 0=const/call, 1=add, 2=sub, 4=unary-, 5=mul, 6=div
- 8=bitwise OR, 9=XOR, 10=AND, 0xb=shl, 0xc=shr
- 0xd=bitwise NOT, 0xe-0x14=cmp (==,!=,<,<=,>,>=), 0x14=!0
- 0x18=pop, 0x19=assign (with const/syntax error checks)
- 0x1b=logical OR, 0x1c=logical AND, 0x1d=ternary ?:
- Calls FUN_0056a6f0 (cleanup), FUN_0056a670 (alloc), FUN_00565d70 (assign)

**ScriptVM_EvaluateExpressionFloat (0x00566730) — 229 xrefs:**
- Float10 (extended precision) expression evaluator
- Same opcode set as ScriptVM_EvaluateExpression
- Returns float10, casts via (float10)(double)
- Division by zero sets DAT_0849f31c = 1

### Symbol Table System

**SymbolTable_Add (0x0056a3c0) — 1101 xrefs:**
- Add symbol to registry
- Allocates memory, copies string name
- Sets 6 params, registers via DAT_07ab6c10 callback table
- Callback at DAT_07ab6c10+0xc and DAT_07ab6c10+0x18

**SymbolTable_Remove (0x0056a4c0) — 666 xrefs:**
- Remove symbol from registry
- "Undefined symbol" error if not found
- Array remove + memmove in DAT_07ab6bf4
- Clear DList ref in DAT_07ab6be0
- DList_RemoveNode, fill 0xcdcdcdcd, _free

### Memory Pool System

**MemoryPool_Free (0x0055b880) — 214 xrefs:**
- Pool free with reference counting
- param_1!=0: decrement ref count (*(param_1+4)--), DList remove
- param_1==0: full free with 0xcdcdcdcd fill, DList_RemoveNode, _free

### Coverage Update

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Renamed functions | 844 | 857 | +13 |
| Coverage | 15.3% | 15.5% | +0.2% |
| Memory functions | 3 | 8 | +5 |
| Event functions | 1 | 2 | +1 |
| String functions | 0 | 1 | +1 |
| Config functions | 0 | 1 | +1 |
| Render functions | 2 | 4 | +2 |

### Round 10 Update

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Renamed functions | 857 | 864 | +7 |
| Coverage | 15.5% | 15.7% | +0.2% |
| Script VM functions | 0 | 4 | +4 |
| Symbol table functions | 0 | 2 | +2 |
| Memory pool functions | 0 | 1 | +1 |

### Round 11 Update

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Renamed functions | 864 | 868 | +4 |
| Coverage | 15.7% | 15.8% | +0.1% |
| Memory allocator functions | 1 | 2 | +1 |
| DList functions | 2 | 3 | +1 |
| Script VM string functions | 1 | 2 | +1 |
| Pattern matching functions | 0 | 1 | +1 |

### Round 12 Update

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Renamed functions | 868 | 872 | +4 |
| Coverage | 15.8% | 15.9% | +0.1% |
| Script interpreter functions | 2 | 6 | +4 |

### Script Interpreter System (Round 12)

**ScriptInterpreter_Execute (0x0054a2a0) — main script execution loop:**

Core script interpreter with stack-based execution.

**Script context structure (param_1):**
```
Offset 0:  script source pointer
Offset 1:  memory block (Mem_Alloc result)
Offset 2:  callback result (FUN_0040dda0 result)
Offset 3..0x12: 0x10 script frames (PooledAllocator_Alloc)
Offset 0x13: frame count
```

**Features:**
- Tokenizes input via FUN_005903c0 (max 0x7f tokens)
- Handles `.isExists` checks
- Handles `this.` calls (calls ScriptVM_DispatchCallback)
- Skips `//` comments to newline
- Stack-based: 0x10 frames max, 0x100 depth
- "Script: Stack overflow" / "Script: Stack underflow" errors
- Calls Mem_Alloc for memory, PooledAllocator_Alloc for frames
- Calls FUN_00566f00 on callback result cleanup

**ScriptVM_DispatchCallback (0x0054a060) — script callback dispatch:**

Looks up and executes callbacks from DAT_083997e0 table.

**Lookup:** `DAT_083997e0 + (DAT_006b6724 * 0x1ff) + (ushort)[context+0x24] * 0x1ff`

**Fallback:** Searches DAT_079f9600[0x1000] for matching function pointer, calls FUN_0040b120(index) + FUN_0065cc30()

**ScriptContext_Cleanup (0x0054a160) — context teardown:**

Frees all allocated resources in script context.

**Cleanup order:**
1. Clear 0x13 script frames (DList_RemoveNode, prepend to DAT_0849f314)
2. Free memory block at offset 0 (DList_RemoveNode + 0xcdcdcdcd fill + _free)
3. Free memory block at offset 1 (same pattern)
4. Call FUN_00566f00 on frame[2] (callback result)

**ScriptVM_EvaluateExpressionEntry (0x0054a5f0) — expression evaluation entry:**

Entry point for float expression evaluation.

**Flow:**
1. If context+4 != 0: call FUN_005494f0
2. If context+4 still != 0: skip to return
3. Otherwise: call ScriptInterpreter_Execute()
4. If context+8 != 0: return ScriptVM_EvaluateExpressionFloat(context+8)
5. Otherwise: return 0.0

### Custom Memory Allocator (Round 11)

**Mem_Alloc (0x0054c240) — 667 xrefs:**

Core memory allocator with alignment and debug features.

**Header structure (before aligned pointer):**
```
Offset -0x10: original malloc ptr
Offset -0xc: refcount (always 0)
Offset -0x8: aligned pointer
Offset -0x4: total size (header + payload)
Offset 0: requested size
```

**Features:**
- Aligns to `in_EAX` boundary (typically 0x10 = 16 bytes)
- Fills payload with 0xabababab (debug pattern)
- Tracks allocations via DList_InsertNode
- Increments DAT_079fd620 (allocation counter)
- "Mem_Alloc() failed to allocate %d bytes" error with total allocation stats

### Pattern Matcher (Round 11)

**PatternMatcher_Match (0x0056ec80) — 65 xrefs:**

Wildcard pattern matching system for string matching.

**Pattern syntax:**

| Pattern | Meaning |
|---------|---------|
| `$` | Literal string |
| `.` | Any single character |
| `?` | Optional match |
| `*` | Repeat/zero-or-more |
| `[chars]` | Character class |
| `[a-z]` | Character range |
| `[~^chars]` | Negated class |
| `^` | Anchor (start) |
| `{...}` | Group/block |
| `(...)` | Sub-expression |

**Output:** 5-int tokens per match slot, 0x40 pattern slots max.

### DList Context Operations (Round 11)

**DList_RemoveFromContext (0x005520e0) — 89 xrefs:**

Remove node from context-specific DList.

**Context structure:**
- Offset 0x58: prev pointer
- Offset 0x5c: next pointer
- Offset 0x60: parent reference
- Calls FUN_005500c0 callback on removal
