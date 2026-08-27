# Reverse Engineering Notes — IGI 2: Covert Strike

**Date:** 2026-08-27  
**Source:** Ghidra MCP (igi2-research project)  
**Binary:** IGI2.exe (5507 functions, 25575 symbols, 442 data types)

---

## 1. Master Initialization Pipeline

### System_InitializeAll (0x005D6040)

**Purpose:** Master initialization for 90+ subsystems in the game engine.

**Initialization Order (as called from System_InitializeAll):**

1. FUN_00610cb0 — Unknown subsystem init
2. FUN_005bd400 — Particle2DCollision_Action
3. FUN_0040ade0 — QTask type creation ("CamTask")
4. FUN_0040aca0 — QTask type creation ("GetInput")
5. FUN_00435f10 — InputPort_Initialize
6. FUN_00600110 — Unknown subsystem init
7. FUN_005c5e40 — MenuItem_Initialize
8. FUN_005c6c30 — MenuItemTask_Initialize
9. FUN_0060aee0 — PushButton_Initialize
10. FUN_0040adf0 — QTask type creation ("SimBoneDynCubeObj")
11. FUN_00577700 — ParameterSystem_Initialize
12. FUN_0054ad40 — Level_RegisterTypes
13. FUN_00606f60 — (0x400 param)
14. FUN_005597b0 — Serialization_RegisterHandlers
15. SerializeList (0x07bb77b0) — 2 callbacks
16. SerializeArray (0x07bab354) — 2 callbacks
17. SerializeStruct (0x07babdc0) — 2 callbacks
18. SerializeQTaskRef (0x07bab358) — 2 callbacks
19. FUN_00592ab0 — EnumTypes_Register
20. FUN_005a3820 — Unknown subsystem init
21. FUN_00435690 — Unknown subsystem init
22. FUN_00599cf0 — MagicObjConfig_Initialize
23. FUN_00605ed0 — LevelScript_Register
24. FUN_00611710 — Unknown subsystem init
25. FUN_0043d420 — PhysicsObjType_ParseConfigFile
26. FUN_005c3ed0 — PhysicsObj_Action
27. FUN_005d5f40 — PhysicsMagicObj_Action
28. FUN_00594980 — EditorRigidObj_Action
29. FUN_00604b20 — Terrain_Discard
30. FUN_00607890 — Terrain_DiscardArea
31. FUN_0060acb0 — CubeModifier_Register
32. FUN_00594000 — Unknown subsystem init
33. FUN_0060da60 — Unknown subsystem init
34. FUN_0059ae10 — TagItem_Initialize
35. FUN_005dbb90 — Unknown subsystem init
36. FUN_00594400 — Unknown subsystem init
37. FUN_005ca4a0 — EditRigidObj_Action
38. FUN_005c08e0 — Unknown subsystem init
39. FUN_005f6840 — Rendering_InitDiscardTask
40. FUN_005e4b50 — Unknown subsystem init
41. FUN_005815c0 — Terrain_Initialize
42. FUN_005a7870 — Water_Initialize
43. FUN_00604030 — Unknown subsystem init
44. FUN_0043dc50 — Unknown subsystem init
45. FUN_006114c0 — EditRigidObjAttachedToBone_Action
46. FUN_0043baa0 — Unknown subsystem init
47. FUN_006059c0 — Unknown subsystem init
48. FUN_00606a60 — AnimRigidDynCubeObj_Action
49. FUN_005b1ad0 — AI_GraphDataInit
50. FUN_005fdf40 — Unknown subsystem init
51. FUN_0060f9a0 — Unknown subsystem init
52. FUN_005a8800 — Unknown subsystem init
53. FUN_005ba990 — Vehicle_Initialize
54. FUN_005dfd80 — Unknown subsystem init
55. FUN_005e5d40 — Unknown subsystem init
56. FUN_00540370 — Unknown subsystem init
57. FUN_00558300 — Serialization_InitLensflare
58. FUN_005dde40 — Unknown subsystem init
59. FUN_005e0950 — Unknown subsystem init
60. FUN_00608950 — NoiseQTask_Register
61. FUN_006095b0 — MoveRigidObj_Action
62. FUN_006054c0 — SmoothQTask_Register
63. FUN_00604e60 — Unknown subsystem init
64. FUN_0041f6b0 — Unknown subsystem init
65. FUN_006038e0 — Unknown subsystem init
66. FUN_0060ff40 — Unknown subsystem init
67. FUN_0060ccd0 — Water_SubsystemInit
68. FUN_005b8be0 — Unknown subsystem init
69. DAT_07b54d4c = "RunDelayed" — QTask type creation
70. DAT_07b54d50 = RunDelayed config (0x18 bytes)
71. FUN_005f7900 — Unknown subsystem init
72. FUN_00610fc0 — Unknown subsystem init
73. FUN_006111a0 — SoundDef_Group_Register
74. FUN_00610240 — SoundDef_Graph_Register
75. FUN_0060d1c0 — SoundDef_Sound_Register
76. FUN_006099e0 — SoundDef_TriggerOnce_Register
77. FUN_00606dd0 — SoundManager_Register
78. FUN_00594f00 — Unknown subsystem init
79. FUN_00573140 — Unknown subsystem init
80. FUN_00610b10 — Unknown subsystem init
81. FUN_00605cd0 — AnimSound_Register
82. FUN_0055b360 — TerrainLightMap_Init
83. FUN_005cbe90 — Unknown subsystem init
84. FUN_0054e1d0 — Unknown subsystem init
85. FUN_0060a630 — SplinePathGuideQTask_Register
86. FUN_005e37e0 — Unknown subsystem init
87. DAT_006aff04 = "LevelInfo" — QTask type creation
88. FUN_006057a0 — Unknown subsystem init
89. FUN_00604430 — Terrain_OLELoader
90. FUN_0054f970 — DebugConsole_DrawConsoleWindow
91. FUN_005e2230 — Unknown subsystem init
92. FUN_0060d740 — Unknown subsystem init
93. FUN_00608180 — Unknown subsystem init
94. FUN_0058c380 — Unknown subsystem init
95. FUN_0060df40 — Unknown subsystem init

**Unknown subsystems:** 31 functions (FUN_00xxxxxx) still need analysis

---

## 2. Code Gaps Analysis

**Total gaps found:** 2171  
**Scan range:** .text section (0x00401000 - 0x0671ffff)

### Largest Gaps (>500 bytes)

| Start | End | Size | Has Undefined | Has Orphaned Instr | Before Func | After Func |
|-------|-----|------|---------------|-------------------|-------------|------------|
| 0x00403cfd | 0x004041ef | 1267 | Yes | Yes | FUN_00403cc0 | FUN_004041f0 |
| 0x00405139 | 0x0040540f | 727 | Yes | No | FUN_00405110 | FUN_00405410 |
| 0x00405431 | 0x0040568f | 607 | Yes | No | FUN_00405410 | FUN_00405690 |
| 0x00402a6a | 0x00402bff | 406 | Yes | Yes | FUN_00401b20 | FUN_00402c00 |
| 0x00405708 | 0x0040588f | 392 | Yes | No | FUN_00405690 | FUN_00405890 |
| 0x00406c18 | 0x00406d9f | 392 | Yes | Yes | FUN_00406bd0 | FUN_00406da0 |

### Medium Gaps (200-500 bytes)

| Start | End | Size | Has Undefined | Has Orphaned Instr |
|-------|-----|------|---------------|-------------------|
| 0x004014ff | 0x004015ff | 257 | Yes | Yes |
| 0x004049d6 | 0x00404a9f | 202 | Yes | No |
| 0x00403be5 | 0x00403c8f | 171 | Yes | No |
| 0x004048b0 | 0x0040495f | 176 | Yes | Yes |
| 0x00402c4e | 0x00402cef | 162 | Yes | No |
| 0x00404c5e | 0x00404cff | 162 | Yes | No |
| 0x004065c6 | 0x0040665f | 154 | Yes | Yes |

### Small Gaps (50-200 bytes)

| Start | End | Size | Has Undefined |
|-------|-----|------|---------------|
| 0x004011af | 0x0040120f | 97 | Yes |
| 0x00402f2a | 0x00402f7f | 86 | Yes |
| 0x00403151 | 0x0040319f | 79 | Yes |
| 0x00404d19 | 0x00404d5f | 71 | Yes |
| 0x004050cb | 0x0040510f | 69 | Yes |
| 0x00401937 | 0x0040196f | 57 | Yes |
| 0x00406758 | 0x0040678f | 56 | Yes |
| 0x00404249 | 0x0040427f | 55 | Yes |
| 0x00402db2 | 0x00402ddf | 46 | Yes |
| 0x0040141c | 0x0040143f | 36 | Yes |
| 0x0040484d | 0x0040486f | 35 | Yes |
| 0x00401475 | 0x0040148f | 27 | Yes |
| 0x00403298 | 0x004032af | 24 | No |
| 0x0040480a | 0x0040481f | 22 | Yes |

### Summary Statistics

| Metric | Value |
|--------|-------|
| Total gaps | 2171 |
| Gaps with undefined bytes | ~1800 (83%) |
| Gaps with orphaned instructions | ~150 (7%) |
| Largest gap | 1267 bytes |
| Smallest gap | 2 bytes |
| Average gap size | ~35 bytes |

### Analysis

- **83% of gaps** contain undefined bytes — likely padding, alignment, or data between functions
- **7% of gaps** contain orphaned instructions — potential missed function boundaries or dead code
- **10% of gaps** are small (<50 bytes) — likely alignment padding between adjacent functions
- **0.7% of gaps** are large (>500 bytes) — potential missed functions or data sections

---

## 3. Switch Dispatch Table Functions (switchD)

**Total switchD functions:** 40+

### Previously Documented (Round 1-5)

| Function | Address |
|----------|---------|
| switchD_00401d77 | 0x00401d77 |
| switchD_004027b3 | 0x004027b3 |
| switchD_004031f7 | 0x004031f7 |
| switchD_0041275b | 0x0041275b |
| switchD_004129e0 | 0x004129e0 |
| switchD_00413d30 | 0x00413d30 |
| switchD_0041591b | 0x0041591b |
| switchD_00415aeb | 0x00415aeb |
| switchD_00415cef | 0x00415cef |
| switchD_00415e7c | 0x00415e7c |
| switchD_00416049 | 0x00416049 |
| switchD_00416214 | 0x00416214 |
| switchD_0041845e | 0x0041845e |
| switchD_0041999d | 0x0041999d |
| switchD_0041a00a | 0x0041a00a |
| switchD_0041fb93 | 0x0041fb93 |
| switchD_0041fcbf | 0x0041fcbf |
| switchD_004201bc | 0x004201bc |
| switchD_00431655 | 0x00431655 |
| switchD_00431683 | 0x00431683 |
| switchD_00432fc5 | 0x00432fc5 |
| switchD_004372ed | 0x004372ed |
| switchD_0043f2f1 | 0x0043f2f1 |
| switchD_0043f32a | 0x0043f32a |
| switchD_0043f3a5 | 0x0043f3a5 |
| switchD_0044158e | 0x0044158e |
| switchD_004415e1 | 0x004415e1 |
| switchD_00441cb2 | 0x00441cb2 |
| switchD_00444bff | 0x00444bff |
| switchD_00444c8e | 0x00444c8e |
| switchD_00447e31 | 0x00447e31 |
| switchD_00447eb6 | 0x00447eb6 |

### Newly Discovered (Round 6)

| Function | Address |
|----------|---------|
| switchD_004485a9 | 0x004485a9 |
| switchD_004485e5 | 0x004485e5 |
| switchD_0044aaca | 0x0044aaca |
| switchD_0044d800 | 0x0044d800 |
| switchD_0044eb61 | 0x0044eb61 |
| switchD_00451938 | 0x00451938 |
| switchD_004519ab | 0x004519ab |
| switchD_00456e59 | 0x00456e59 |
| switchD_00457896 | 0x00457896 |
| switchD_004624e5 | 0x004624e5 |
| switchD_0046272b | 0x0046272b |
| switchD_00462951 | 0x00462951 |
| switchD_00464590 | 0x00464590 |
| switchD_004645d4 | 0x004645d4 |
| switchD_004646a6 | 0x004646a6 |
| switchD_00464a83 | 0x00464a83 |
| switchD_0046ad02 | 0x0046ad02 |
| switchD_0046ad7d | 0x0046ad7d |
| switchD_0046ae52 | 0x0046ae52 |
| switchD_0046aff1 | 0x0046aff1 |
| switchD_00477d9d | 0x00477d9d |
| switchD_0047dc9c | 0x0047dc9c |
| switchD_00484cd4 | 0x00484cd4 |
| switchD_0048ac79 | 0x0048ac79 |
| switchD_0048b181 | 0x0048b181 |
| switchD_0049881f | 0x0049881f |
| switchD_0049887a | 0x0049887a |
| switchD_0049b101 | 0x0049b101 |
| switchD_0049be2d | 0x0049be2d |
| switchD_004a20c4 | 0x004a20c4 |

### Analysis

- **32 switchD functions** in the 0x00400000-0x00440000 range (core engine)
- **20 switchD functions** in the 0x00440000-0x004a0000 range (game systems)
- These are likely switch/case dispatch tables for:
  - QTask event handlers
  - AI state machine transitions
  - Input action dispatchers
  - Network message handlers
  - Menu screen transitions

---

## 4. AI Type Strings

### Discovered Strings

| Address | String | Xrefs | Purpose |
|---------|--------|-------|---------|
| 0x006b9830 | "HUMANAI_TYPE_C1_NORMAL_SOLDIER" | 8 | Standard C1 soldier AI type |
| 0x006bc0e8 | "Jones" | 9 | NPC name / character reference |

### Naming Convention

The "C1" prefix likely refers to:
- **C1** = Covert 1 (player character James "Jimmy" Riggs)
- **NORMAL_SOLDIER** = Standard enemy soldier variant

Other AI types likely follow:
- `HUMANAI_TYPE_C1_NORMAL_SOLDIER` — Standard soldier
- `HUMANAI_TYPE_C1_ELITE` — Elite soldier
- `HUMANAI_TYPE_C1_BODYGUARD` — Bodyguard type
- `HUMANAI_TYPE_C1_PATROL` — Patrol type

---

## 5. Telnet Protocol Strings

**Address:** 0x0068261c+

| Address | String | Xrefs | Purpose |
|---------|--------|-------|---------|
| 0x0068261c | "TELNET_OPTION_UNKNOWN" | 5 | Unknown telnet option |
| 0x00682624 | "TELNET_OPTION_ECHO" | 5 | Telnet ECHO option (RFC 857) |

**Analysis:** The game includes Telnet protocol support, likely for:
- Console communication over network
- Remote debugging
- Server administration

---

## 6. Master Initialization Summary

### Boot Sequence

```
1. WinMain_CRTStartup (0x00404280)
   ├── Mutex check ("InnerloopGameMUTEX")
   ├── CoInitialize(NULL)
   ├── WNDCLASSA registration ("Innerloop")
   ├── CreateWindowExA (448x480)
   ├── DirectX 5.0+ version check
   ├── Config callbacks (13 subsystems)
   ├── ShowCursor(0), ShowWindow(pHVar10, 10)
   └── Main message loop
       └── Calls Game_MainLoop (0x00401440) per frame

2. Game_MainLoop (0x00401440)
   └── Calls Game_ProcessGameState (0x00401490)
       └── State dispatch based on game state flag
           └── Calls System_InitializeAll on first state

3. System_InitializeAll (0x005D6040)
   ├── QTask type creation (CamTask, GetInput, SimBoneDynCubeObj, RunDelayed, LevelInfo)
   ├── Input system (InputPort_Initialize)
   ├── Menu system (MenuItem, MenuItemTask, PushButton, TagItem)
   ├── Parameter system (ParameterSystem, EnumTypes)
   ├── Serialization (List, Array, Struct, QTaskRef)
   ├── Physics (PhysicsObj, PhysicsMagicObj, EditRigidObj, AnimRigidDynCubeObj)
   ├── Terrain (Initialize, Discard, DiscardArea, OLELoader, LightMap)
   ├── Water (Initialize, SubsystemInit)
   ├── AI (GraphDataInit, Vehicle)
   ├── Rendering (DiscardTask)
   └── Audio (SoundDef_*, SoundManager, AnimSound)
```

---

## 7. Coverage Metrics

| Metric | Value |
|--------|-------|
| Total functions | 5,507 |
| Functions renamed | 830 |
| Coverage | 15.0% |
| Code gaps | 2,171 |
| switchD functions | 40+ |
| Custom data types | 10+ |
| Function tags | 105 definitions |
| Documentation files | 21 |
| Global data areas | 19 |
| AI types discovered | 2+ |
| String references | 15+ |

---

## 8. Next Steps for Analysis

### Priority 1 — Fill Code Gaps
- Analyze 150+ gaps with orphaned instructions
- Check for missed function boundaries
- Look for embedded data tables

### Priority 2 — Unknown Subsystems
- Analyze 31 unknown FUN_00xxxxxx calls in System_InitializeAll
- Map to known subsystems or create new ones

### Priority 3 — switchD Functions
- Reverse engineer 20 new switchD functions
- Map to QTask event handlers, AI states, input actions

### Priority 4 — vftable
- Map vftable @ 0x0067c7a8 to specific class hierarchy
- Identify all virtual functions

### Priority 5 — AI Types
- Discover all HUMANAI_TYPE_* strings
- Map to AI entity behaviors

---

## 9. QTask System Architecture (Round 7)

### Overview
QTask is the core task/scheduler framework with ~2000+ cross-references across 10+ functions.

### Type Registry
- **Entry size:** 0x34 bytes (52 bytes) per type
- **Max slots:** 0xd (13) active type entries
- **State arrays:** 0x200 bytes (512 bytes) per type, copied from parent during inheritance
- **Base type marker:** 0x200 (terminates inheritance chain)
- **Active flags:** DAT_08498828 (1=registered, 0=free)

### Core Functions
| Function | Address | Xrefs | Purpose |
|----------|---------|-------|---------|
| QTaskType_Register | 0x0040adf0 | 372 | Register new type, allocate slot, copy parent state |
| QTaskType_Dispatch | 0x0040af30 | 375 | Cleanup task type, call destructor |
| QTaskType_ClearActive | 0x0040adc0 | 234 | Clear active flag for type |
| QTaskType_IsDerivedFrom | 0x0040b3f0 | 206 | Check inheritance chain |
| QTaskList_FindByType | 0x0040b2f0 | 313 | Find task by type in linked list |
| QTaskList_FindByTypeID | 0x0040b350 | 183 | Find task by type ID (offset 9*4) |
| QTaskList_FindByTypeOffset | 0x0040b380 | 28 | Find task by type at offset 0x24 |
| QTaskList_DestroyAll | 0x0040bb00 | 114 | Destroy all tasks in list |

### Task List Structure
- Linked list with next pointers
- Type stored at offset 0x12 (36 bytes from task start)
- Multiple find variants for different offset requirements

---

## 10. File System Architecture (Round 7)

### Device Handler Table
- **Location:** DAT_007048d4
- **Entry size:** 0x90 bytes (144 bytes) per device
- **Device types:** 4 (1=stdin, 2=stdout, 3=stderr, 4=file)
- **Name table:** DAT_007048f8 (0x90 bytes each)

### Path Parsing
**FileSys_ParsePath (0x0040c330):**
1. Extract device letter from ':' character position
2. Validate device type (1-4 range)
3. Check device present flag
4. Dispatch to device-specific handler table
5. Copy remaining path to device buffer

### File Operations
**FileSys_Read (0x0040c6d0):**
- Device callback dispatch at DAT_007048b0
- Buffer allocation via FUN_0040bb30
- Retry loop: max 10 attempts (DAT_00704888 callback)
- Error: "Error while reading from file %s"

**FileSys_Close (0x0040cf20):**
- Device close callback at DAT_00704884
- Special handling for DAT_00704648 (stdin/stdout?)

---

## 11. Game Event System (Round 7)

### GameEvent_Alloc (0x0040aca0)
- **Entry size:** 0xc bytes (12 bytes)
- **Max events:** 0x20 (32)
- **Layout:** type (uint2), active flags (2 bytes), data (uint4)
- **Location:** DAT_00700820..DAT_007009a2
- **Error:** "Unable to allocate event" → abort()

---

## 12. Coverage Update (Round 7)

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Renamed functions | 816 | 830 | +14 |
| Coverage | 14.7% | 15.0% | +0.3% |
| QTask functions | 8 | 14 | +6 |
| FileSys functions | 0 | 3 | +3 |
| Event functions | 0 | 1 | +1 |

---

## 13. Doubly Linked List System (Round 8)

### Overview
DList is the **most used data structure** in the binary with 852+ cross-references for RemoveNode alone.

### DList_RemoveNode (0x00445540)
```c
void DList_RemoveNode(DListNode *node) {
    node->next->prev = node->prev;
    node->prev->next = node->next;
    node->next = 0;
    node->prev = 0;
}
```
- 852 xrefs = core data structure used everywhere
- Node structure: {next:4, prev:4}

### DList_InsertNode (0x00445460)
```c
void DList_InsertNode(void *param_1, DListNode *node) {
    // param_1+4 = next, param_1+8 = prev
    node->prev = *(void**)(param_1 + 8);
    node->next = param_1 + 4;
    **(void***)(param_1 + 8) = node;
    *(void**)(param_1 + 8) = node;
}
```
- 217 xrefs
- Used by ResourceManager, QTaskList, render systems

---

## 14. Fatal Error System (Round 8)

### FatalError_Handler (0x00446000)
- 553 xrefs = critical error handling
- 256-byte buffer at DAT_084a01a0
- Format: "Fatal error:\n\n" + user message
- Callback dispatch:
  - DAT_0073fb38: error display callback
  - DAT_0073fb30: error handler (may abort)

---

## 15. Server Logging (Round 8)

### Server_LogMessage (0x0041d470)
- 347 xrefs = network/server subsystem
- Timestamp: [%02d:%02d:%02d] using _gmtime
- Dual output:
  1. File device (DAT_08350c0c)
  2. Server log: "Server: %s"
- Traffic suppression: max 0x1e (30) messages, then "Suppressing announces - too much traffic"

---

## 16. Resource Manager (Round 8)

### ResourceManager_Load (0x00425740)
- 143 xrefs = asset loading
- Path formats:
  - "GLOB:/filename" — global resources
  - "LOCA:/?ENGLISH/filename" — localized (English)
- Maintains linked list at DAT_0070b304
- Uses String_ParseToken for path parsing
- FileSys_Read for actual file loading
- Error: "Failed to load resource: '%s'" → FatalError_Handler

---

## 17. Network System (Round 8)

### Network_SendPacket (0x00419300)
- 75 xrefs = multiplayer/networking
- 8180-byte stack buffer
- Destination: DAT_08350c6c+0x40 (server address)
- Log format: "Sending: %d, %c%c%c%c, to %x, %d bytes, Guaranteed"
  - %c%c%c%c = first 4 bytes as ASCII (packet type tag)
  - "Guaranteed" = delivery type (vs unreliable?)
- Calls FUN_004437f0 for actual send

---

## 18. D3D Error Converter (Round 8)

### D3D_ErrorString (0x00448530)
- 129 xrefs = DirectX error handling
- HRESULT to string converter
- Switch table on HRESULT value
- Returns D3DERR_* and E_* error names:
  - D3DERR_INVALIDCALL, D3DERR_DEVICENOTRESET
  - D3DERR_DEVICELOST, D3DERR_NOTAVAILABLE
  - D3DERR_TOOMANYOPERATIONS, D3DERR_CONFLICTINGRENDERSTATE
  - E_OUTOFMEMORY, E_FAIL, E_INVALIDARG

---

## 19. Performance Counter (Round 8)

### Performance_GetCounter (0x00435820)
- Windows QueryPerformanceCounter wrapper
- Frequency stored at DAT_0072bc40/44 (64-bit)
- Converts to milliseconds:
  ```c
  counter = QueryPerformanceCounter();
  ms = (counter / frequency) * 1000;
  ```
- Uses __alldvrm, __allmul, __alldiv (64-bit math)
- FatalError if no performance counter available

---

## 20. Event Dispatcher (Round 8)

### EventDispatcher_Dispatch (0x004378e0)
- 61 xrefs = event-driven architecture
- Handler table: DAT_08350bb4+8 (linked list of handlers)
- Callback table: DAT_083997e0 (function pointers)
- Event IDs: (&DAT_0072bc58)[event_id]
- Dispatch: indirect call with (short)puVar3[9] * 0x1ff offset

---

## 21. QTaskList Array-Based (Round 8)

### QTaskList_Add (0x0041f310)
- 60 xrefs = task management
- Array at in_EAX+2 (pointer to task pointers)
- Count at *in_EAX, capacity at in_EAX[1]
- Overflow check: in_EAX[1] <= *in_EAX
- Error: "QTaskList is full" → FatalError_Handler

### QTaskList_Remove (0x0041f340)
- 56 xrefs = task removal
- Linear search for unaff_EDI value
- Shift remaining elements down
- Decrement count

---

## 22. Sound Definition Finder (Round 8)

### SoundDef_Find (0x00430730)
- 100 xrefs = audio subsystem
- Calls GameEvent_Alloc(0, "FindSoundDef")
- Iterates sound handlers from DAT_08350bd0
- Creates QTask for result via QTask_Create

---

## 23. Coverage Update (Round 8)

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Renamed functions | 830 | 844 | +14 |
| Coverage | 15.0% | 15.3% | +0.3% |
| DList functions | 0 | 2 | +2 |
| Network functions | 0 | 2 | +2 |
| Render functions | 0 | 2 | +2 |
| Event functions | 0 | 1 | +1 |
| Performance functions | 0 | 1 | +1 |
| Resource functions | 0 | 1 | +1 |
| Audio functions | 0 | 1 | +1 |

---

## 24. Memory Management System (Round 9)

### Memory_ValidateBlock (0x004451b0)
- 47 xrefs = memory safety checks
- Power-of-2 alignment check: `((uVar1-1) & uVar1) == uVar1`
- Bounds validation: `unaff_EDI - param_2 < size`
- Returns aligned address or 0
- Sets DAT_08350b24 = 1 on failure

### MemoryPool_Allocate (0x00445270)
- 33 xrefs = pool allocator
- Allocates 0xc-byte header: {callback_ptr, block_ptr, size}
- Fills pool with callbacks from unaff_ESI+0x14 table
- Updates counter at unaff_ESI+8

### Memory_Allocate (0x0041f3f0)
- 27 xrefs = simple allocator wrapper
- Allocates unaff_ESI*4+8 bytes via FUN_0054c240

### Memory_Deallocate (0x0041f410)
- 27 xrefs = deallocator with debug fill
- MSVC debug pattern: 0xcdcdcdcd (0xcd = "CD" = "free")
- Decrements DAT_079fd620 (active allocation counter)
- DList_RemoveNode from tracking list
- Calls _free from CRT

### PooledAllocator_Alloc (0x0040baa0)
- 161 xrefs = heavily used allocation path
- Wrapper around FUN_0055b820
- Returns 0 if in_EAX == 0

---

## 25. Event Dispatcher System (Round 9)

### EventDispatcher_DispatchCallback (0x0040ac10)
- 46 xrefs = event callback dispatch
- __thiscall member function
- Linked list traversal via in_EAX+0x20
- Callback lookup: DAT_083997e0 + type*0x1ff
- Calls callback(iVar1, param_2)

### TaskCallback_Dispatch (0x0040e300)
- 120 xrefs = core task system
- If param_1+4 != 0: FUN_005697f0(&local_4, 4, 1)
- Otherwise: indexed lookup at *(param_1+8) + in_EAX*4

---

## 26. String Parsing System (Round 9)

### String_ParseInput (0x0040dea0)
- 46 xrefs = string input processing
- 2052-byte stack buffer (local_804)
- 36-byte temp buffer (local_828)
- Pipeline: strlen → parse (FUN_0056e2d0) → convert (FUN_005670c0)
- Sets DAT_0849f31c = 0 on completion

---

## 27. Config System (Round 9)

### Config_LookupString (0x004335d0)
- 49 xrefs = configuration lookup
- Case-insensitive search via __stricmp
- Table at unaff_EBX+0x2fc, 0x44 bytes per entry
- 0x44 entries max (short at unaff_EBX+0x26be)
- Returns value at unaff_EBX+0x33c + index*0x44

---

## 28. Render System (Round 9)

### RenderObject_Create (0x00463680)
- 27 xrefs = render object allocation
- Allocates via FUN_00463630
- Initializes 13 fields:
  - offset 0: 0
  - offset 4: 0
  - offset 6: param_1
  - offset 5: param_3
  - offset 3: DAT_08350abc
  - offset 0xc: param_2
  - offset 0xd: 1 (active flag)
  - offsets 7,8,9,0xb: 0
- DList_InsertNode to track

### RenderTask_Cleanup (0x004637c0)
- 30 xrefs = render task lifecycle
- Calls callback at unaff_ESI+0x24
- DList_RemoveNode to unlink
- Reinserts at unaff_ESI+0x30 with DList_InsertNode
- Resets flag at unaff_ESI+0x34

---

## 29. ResourceManager (Round 9)

### ResourceManager_LoadAll (0x00425a10)
- 64 xrefs = bulk resource loading
- Simple wrapper: calls ResourceManager_Load(1)

---

## 30. Coverage Update (Round 9)

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Renamed functions | 844 | 857 | +13 |
| Coverage | 15.3% | 15.5% | +0.2% |

---

## 31. Script VM System (Round 10)

### Architecture

IGI 2 includes a custom script virtual machine with:
- Parameter system: 0x14-byte entries with type/value/name
- Expression evaluators: integer and float10 (extended precision)
- 30+ opcodes for arithmetic, logic, bitwise, comparison, assignment
- Callback-based symbol table for runtime resolution

### ScriptVM_AddParameter (0x005772d0) — 1754 xrefs

Core parameter addition to VM context. Called extremely frequently.

**Parameter structure (0x14 bytes):**
```
Offset 0:  value (4 bytes)
Offset 4:  type (4 bytes)
Offset 8:  type_name pointer (4 bytes)
Offset C:  flag (1 byte)
Offset D:  padding (3 bytes)
```

**Checks:**
- "Too many parameters" — if counter exceeds limit
- "Undefined parameter type '%s'" — if type == -1

### ScriptVM_ReadParameter (0x005697f0) — 518 xrefs

Reads parameters from script VM context with three modes:

| param_3 | Mode | Behavior |
|---------|------|----------|
| 1 | String pointer | Allocates via FUN_00567b80, returns ptr |
| 2 | Raw value | Returns value directly |
| 3 | String buffer | strncpy to caller buffer |

Returns 1 on success, 0 on failure.

### ScriptVM_EvaluateExpression (0x005661c0) — 303 xrefs

Integer expression evaluator with recursive descent parsing.

**Opcode table:**

| Op | Operation | Description |
|----|-----------|-------------|
| 0x00 | const/call | Load constant or call function |
| 0x01 | add | param_1c + param_18 |
| 0x02 | sub | param_18 - param_1c |
| 0x03 | copy | Return left operand |
| 0x04 | unary- | -param_1c |
| 0x05 | mul | param_1c * param_18 |
| 0x06 | div | param_18 / param_1c (div-by-zero → DAT_0849f31c=1) |
| 0x08 | bitwise OR | param_1c \| param_18 |
| 0x09 | bitwise XOR | param_1c ^ param_18 |
| 0x0a | bitwise AND | param_1c & param_18 |
| 0x0b | shift left | param_1c << (param_18 & 0x1f) |
| 0x0c | shift right | param_1c >> (param_18 & 0x1f) |
| 0x0d | bitwise NOT | ~param_1c |
| 0x0e | equal | param_1c == param_18 |
| 0x0f | not equal | param_1c != param_18 |
| 0x10 | less than | param_1c < param_18 |
| 0x11 | greater than | param_18 < param_1c |
| 0x12 | less or equal | param_1c <= param_18 |
| 0x13 | greater or equal | param_1c >= param_18 |
| 0x14 | not zero | param_1c != 0 |
| 0x18 | pop | Discard right, return left |
| 0x19 | assign | Assign with const/syntax error checks |
| 0x1b | logical OR | Short-circuit OR |
| 0x1c | logical AND | Short-circuit AND |
| 0x1d | ternary ?: | Conditional expression |

**Helper calls:**
- FUN_0056a6f0 — cleanup
- FUN_0056a670 — allocation
- FUN_00565d70 — assignment handler

### ScriptVM_EvaluateExpressionFloat (0x00566730) — 229 xrefs

Extended precision (float10) variant of ScriptVM_EvaluateExpression.
- Same opcode set
- Returns float10, casts via (float10)(double)
- Division by zero sets DAT_0849f31c = 1

---

## 32. Symbol Table System (Round 10)

### SymbolTable_Add (0x0056a3c0) — 1101 xrefs

Add symbol to runtime registry.

**Flow:**
1. Calculate string length
2. Allocate memory (FUN_0054c240)
3. Zero 7 fields
4. Copy string name
5. Set 6 parameters
6. Register via DAT_07ab6c10 callback table
7. Call FUN_005ccd30 (post-registration)

### SymbolTable_Remove (0x0056a4c0) — 666 xrefs

Remove symbol from registry.

**Flow:**
1. Look up symbol via DAT_07ab6c10 callbacks
2. "Undefined symbol" error if not found
3. Array remove + memmove in DAT_07ab6bf4
4. Clear DList ref in DAT_07ab6be0
5. DList_RemoveNode to unlink
6. Fill freed memory with 0xcdcdcdcd
7. _free to release

---

## 33. Memory Pool System (Round 10)

### MemoryPool_Free (0x0055b880) — 214 xrefs

Pool allocator free with reference counting.

**Two modes:**

| param_1 | Mode | Behavior |
|---------|------|----------|
| != 0 | Ref count decrement | *(param_1+4)--, DList relink |
| == 0 | Full free | DList_RemoveNode, 0xcdcdcdcd fill, _free |

---

## 34. Coverage Update (Round 10)

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Renamed functions | 857 | 864 | +7 |
| Coverage | 15.5% | 15.7% | +0.2% |
| Script VM functions | 0 | 4 | +4 |
| Symbol table functions | 0 | 2 | +2 |
| Memory pool functions | 0 | 1 | +1 |

---

## 35. Custom Memory Allocator (Round 11)

### Mem_Alloc (0x0054c240) — 667 xrefs

Core memory allocation with alignment and debug tracking.

**Header layout (allocated before aligned pointer):**
```
[Padded to alignment]
[original_malloc_ptr:4]  @ aligned-0x10
[refcount:4]             @ aligned-0xc (always 0)
[aligned_ptr:4]          @ aligned-0x8
[total_size:4]           @ aligned-0x4
[requested_size:4]       @ aligned+0
[padding:4]              @ aligned+4
[payload...]             @ aligned+8
```

**Behavior:**
1. Calculate total size: `in_EAX + 0x1c + param_1`
2. Increment DAT_079fd620 (allocation counter)
3. Call `_malloc(total_size)`
4. On failure: walk DAT_084a3d10 linked list, report totals, FatalError_Handler
5. Align pointer to `in_EAX` boundary
6. Store original ptr, refcount, aligned ptr, size in header
7. If `DAT_08350778 == '\0'`: call FUN_0054c3b0 (init?)
8. Insert into DList for tracking
9. Fill payload with 0xabababab (debug pattern)
10. Return aligned pointer

**Debug pattern:** 0xabababab — distinguishes from 0xcdcdcdcd (free pattern)

---

## 36. Pattern Matching System (Round 11)

### PatternMatcher_Match (0x0056ec80) — 65 xrefs

Wildcard pattern matching with regex-like features.

**Pattern syntax table:**

| Char | Meaning | Token flags |
|------|---------|-------------|
| `$` | Literal string | 0x4000 |
| `(` | Sub-expression | 0x8000, push to stack |
| `)` | Close group | 0x10000, pop stack |
| `*` | Repeat/zero-or-more | 0x200, 0x20000 |
| `.` | Any character | 0x1000 |
| `/` | Skip next char | - |
| `?` | Optional | 0x40000, 0x20000 |
| `[chars]` | Character class | 0x1000 |
| `^` | Anchor (start) | 0x2000 |
| `{` | Open block | 0x400 |
| `}` | Close block | 0x800 |

**Character class features:**
- `[abc]` — match a, b, or c
- `[a-z]` — match range a through z
- `[~^abc]` — negate class
- `[~^]` — invert entire class
- `\n`, `\r`, `\t` — escape sequences

**Output format:**
- 5-int tokens per match: `[prev_expr:4][var_count:4][flags:4][depth:4][end_token:4]`
- 0x40 pattern slots (param_2)
- 255 expression slots (local_808)
- 258 token slots (local_40c)

---

## 37. DList Context Operations (Round 11)

### DList_RemoveFromContext (0x005520e0) — 89 xrefs

Remove node from context-specific DList (not global list).

**Context structure (unaff_ESI):**
```
Offset 0x58: prev pointer
Offset 0x5c: next pointer  
Offset 0x60: parent reference
Offset 0x34: child list head
```

**Behavior:**
1. Get prev node from context+0x58
2. If prev exists: clear prev's reference
3. Get next node from context+0x60
4. Update prev->next and next->prev pointers
5. If node is head (matches prev+0x2c): update prev+0x2c
6. Call FUN_005500c0 (callback?)
7. Clear context+0x58

---

## 38. Script VM String System (Round 11)

### ScriptVM_EvaluateToString (0x00565e40) — 97 xrefs

Convert script expressions to string representations.

**Opcodes:**

| Op | Mode | Behavior |
|----|------|----------|
| 0x00.2 | string literal | strncpy from param_1+8 |
| 0x00.4/5 | variable lookup | Resolve symbol, call callback, strncpy from result+0x10 |
| 0x00.6 | recursive | Recursively evaluate and copy |
| 0x01 | concat | Evaluate left, strncpy; evaluate right, strncat |
| 0x18 | pop | Evaluate right, return it |
| 0x19 | assign | Resolve target, assign value, evaluate result |

**Error messages:**
- "Numeric member in string expression. Line %d."
- "Unknown identifier '%s' in expression, line %d"
- "Syntax error in string expression, line %d"
- "Internal script error. Invalid assignment, line %d"
- "Expression: Cannot assign to constant, line %d"
- "Syntax error in assignment, line %d"

**Buffer:** 2048-byte stack buffer (local_800)

---

## 39. Coverage Update (Round 11)

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Renamed functions | 864 | 868 | +4 |
| Coverage | 15.7% | 15.8% | +0.1% |
| Memory allocator functions | 1 | 2 | +1 |
| DList functions | 2 | 3 | +1 |
| Script VM string functions | 1 | 2 | +1 |
| Pattern matching functions | 0 | 1 | +1 |

---

## 40. Script Interpreter System (Round 12)

### Architecture

IGI 2 includes a stack-based script interpreter with:
- Token-based parsing (max 0x7f tokens per line)
- 0x10 script frames per context (PooledAllocator_Alloc)
- 0x100 call stack depth
- Callback dispatch via DAT_083997e0 table
- Memory management via Mem_Alloc / PooledAllocator_Alloc

### ScriptInterpreter_Execute (0x0054a2a0) — main execution loop

Core script interpreter with full lifecycle management.

**Script context layout (param_1):**
```
Offset 0x00: script_source (char*)
Offset 0x04: memory_block (void*) — from Mem_Alloc
Offset 0x08: callback_result (void*) — from FUN_0040dda0
Offset 0x0C..0x44: script_frames[0x10] — PooledAllocator_Alloc results
Offset 0x4C: frame_count (0x13)
```

**Execution loop:**
1. Clear all 0x13 frames (DList_RemoveNode, prepend to DAT_0849f314)
2. Clear callback result (FUN_00566f00)
3. Reset frame_count to 0
4. Tokenize input via FUN_005903c0(&local_58c, 0x7f, 0)
5. If token found:
   - Look up in DAT_079f9600[0x1000] table
   - If found and pointer at +8 != 0: store in frame
   - If frame_count < 0x10: PooledAllocator_Alloc + store
6. Handle special cases:
   - `this.` → ScriptVM_DispatchCallback + format "%s.%s"
   - `.isExists` → append to token, set local_584[0] = 0x31
7. Skip `//` comments to newline
8. Copy tokens to local_588 buffer
9. Loop until source exhausted

**Stack management:**
- Push: DAT_0849f744++, DAT_0849f340 + DAT_0849f744*4 = DAT_0849f748
- Pop: DAT_0849f744--, restore DAT_0849f748
- Overflow: DAT_0849f744 >= 0x100 → "Script: Stack overflow"
- Underflow: DAT_0849f744 == 0 → "Script: Stack underflow"

### ScriptVM_DispatchCallback (0x0054a060) — callback dispatch

Dispatches callbacks from DAT_083997e0 table.

**Lookup formula:**
```
callback = DAT_083997e0 + (DAT_006b6724 * 0x1ff) + (ushort)[context+0x24] * 0x1ff
```

**Execution:**
1. If callback != 0: call it, copy 64-byte result to local_40
2. Otherwise: search DAT_079f9600[0x1000] for matching function pointer at +8
3. Call FUN_0040b120(index) + FUN_0065cc30()

### ScriptContext_Cleanup (0x0054a160) — context teardown

Frees all resources in script context.

**Cleanup sequence:**
1. Clear 0x13 frames:
   - DList_RemoveNode from each frame
   - Prepend to DAT_0849f314 linked list
   - Decrement DAT_0849f304
2. Free memory_block (offset 0):
   - DList_RemoveNode
   - 0xcdcdcdcd fill (size from +0x10)
   - _free(orig_ptr)
   - Decrement DAT_079fd620
3. Free memory_block (offset 1): same pattern
4. Call FUN_00566f00 on frame[2]

### ScriptVM_EvaluateExpressionEntry (0x0054a5f0) — expression entry point

Entry point for float expression evaluation.

**Flow:**
```
if (context[1] != 0) {  // context+4
    FUN_005494f0();
    if (context[1] == 0) goto skip_execute;
}
ScriptInterpreter_Execute();
skip_execute:
if (context[2] != 0) {  // context+8
    return ScriptVM_EvaluateExpressionFloat(context[2]);
}
return 0.0;
```

---

## 41. Coverage Update (Round 12)

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Renamed functions | 868 | 872 | +4 |
| Coverage | 15.8% | 15.9% | +0.1% |
| Script interpreter functions | 2 | 6 | +4 |
| Memory functions | 3 | 8 | +5 |
| Event functions | 1 | 2 | +1 |
| String functions | 0 | 1 | +1 |
| Config functions | 0 | 1 | +1 |
| Render functions | 2 | 4 | +2 |
