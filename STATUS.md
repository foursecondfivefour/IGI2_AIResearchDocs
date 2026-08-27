# IGI 2: Covert Strike — Reverse Engineering Status

**Last Updated:** 2026-08-27  
**Status:** 🟡 PARTIAL — 880 functions renamed total (16.0% of 5,507), 100+ plate comments, 10+ custom data types, 105 function tag definitions, 72/95 System_InitializeAll subsystems catalogued (100% of batch analysis)  
**Last Verified:** 2026-08-27 via Ghidra MCP

### Round 6 — NEW (2026-08-27)
- [x] WinMain_CRTStartup (0x00404280) — renamed from FUN_00404280, plate comment added
- [x] System_InitializeAll (0x005D6040) — renamed from FUN_005d6040, plate comment added
- [x] Master init pipeline documented (90+ subsystems)
- [x] Code gaps analysis: 2171 gaps found
- [x] vftable discovered @ 0x0067c7a8 (9 xrefs)
- [x] New AI types: HUMANAI_TYPE_C1_NORMAL_SOLDIER
- [x] switchD catalog expanded: 40+ total
- [x] Config callback table: 48 entries (Window, WinInput, Track, Editor, Server, Player, Password, Zbias, Config, SessionDir, FPSLock)

### Round 7 — NEW (2026-08-27)
- [x] QTaskType_Register (0x0040adf0) — 372 xrefs, type registration with 0x34-byte entries
- [x] QTaskType_Dispatch (0x0040af30) — 375 xrefs, cleanup/destructor dispatch
- [x] GameEvent_Alloc (0x0040aca0) — 330 xrefs, 0xc-byte event slots, 0x20 max
- [x] QTaskList_FindByType (0x0040b2f0) — 313 xrefs, linked list search
- [x] QTaskType_IsDerivedFrom (0x0040b3f0) — 206 xrefs, inheritance chain traversal
- [x] QTaskType_ClearActive (0x0040adc0) — 234 xrefs, active flag clearing
- [x] QTaskList_FindByTypeID (0x0040b350) — 183 xrefs, type ID lookup
- [x] QTaskList_DestroyAll (0x0040bb00) — 114 xrefs, full list cleanup
- [x] Game_GetDebugFlag (0x0040ade0) — 176 xrefs, global flag access
- [x] FileSys_Read (0x0040c6d0) — 41 xrefs, device handler dispatch, 0x90-byte entries
- [x] FileSys_Close (0x0040cf20) — 37 xrefs, handle cleanup
- [x] CommandLine_Process (0x0040d020) — 32 xrefs, 1024-byte buffer, callback dispatch
- [x] FileSys_ParsePath (0x0040c330) — 35 xrefs, Windows path parsing (C:\...)
- [x] QTaskList_FindByTypeOffset (0x0040b380) — 28 xrefs, offset-based search
- [x] Pooled allocator FUN_0055b820 identified (game code, below CRT boundary)
- [x] QTask system architecture documented: 0x34-byte types, 0xd slots, 0x200-byte state arrays
- [x] File system architecture documented: 4 device types, 0x90-byte handler table
- [x] Game event system documented: 0xc-byte entries, 0x20 max events

### Round 16 — NEW (2026-08-27) — System_InitializeAll Deep Analysis (100% Coverage)
- [x] System_InitializeAll (0x005d6040) — 95 init calls fully catalogued: 72 unique subsystems identified
- [x] Batch 1 (1-20): Activate/Deactivate/GetState, Particle2DDynCubeObjTask, InputPort, DirtyRectangle, MenuItem, MenuItemTask, PushButton, ParameterSystem, Level/Static/Dynamic/CameraBase, Serialization, EnumTypes, ScriptVM frames, MagicObjConfigContainer, MagicObjConfig, LevelScript, SkyTask, PhysicsObjType, PhysicsObj, PhysicsMagicObj
- [x] Batch 2 (21-40): EditorRigidObj, DiscardTerrain, DiscardTerrainArea, CubeModifier, AnimTask/AnimData, EditRigidObj, EditBoneObj, DiscardTask, TextureModifier, Terrain/TerrainMap/TerrainMaterial, Water/WaterLayer, EditHazinglayer, LODSettings/ModelLODSettings, EditRigidObjAttachedToBone, CutScene, AreaActivate, Vehicle, GlobalLight/GlobalLightKeyframe, Dirlight/DirlightKeyframe
- [x] Batch 3 (41-60): LightmapInfo, Lensflare/LensflareItem, Container, Origo, NoiseQTask, MoveRigidObj, SmoothQTask, MipMapControl, ScreenGrab, FlatSky/FlatSkyLayer, FlatSkyKeyframe, FlatWater/FlatWaterLayer, Forest, RunDelayed, SoundSysEdit, SoundDefGroupEdit, SoundDefGraphEdit, SoundDefSoundEdit, SoundDefTriggerOnceEdit, SoundManager
- [x] Batch 4 (61-72): EditCamera, LevelTimer, EditorMagicObj, AnimSound, TerrainLightMap, SimpointObjContainer/SimpointObj, SplinePathDynCubeObj, SplinePathGuideQTask, HeightMap, GFXCapsViewer, OLELoader, DebugConsoleTask
- [x] Key patterns: QTask (0x34 bytes/type, 0xd slots, 0x200-byte state arrays), ParameterSystem (0x13-byte entries, 0x4c0 max), PatternMatcher wildcards ($, ., ?, *, [], ^, {}, ()), Radiosity colors (0xc8fb1a07, 0x39ea7756, 0x7ff13ccd), ScriptVM param types (7=Real32, 6=Int32, 3=String, 1=bool8)
- [x] Complete subsystem catalog documented in plan.md with 72 entries organized by 4 analysis batches

### Round 15 — NEW (2026-08-27) — 02-classes Extended + System_InitializeAll Partial Analysis
- [x] HumanAnimController struct created — 604 bytes, 147 fields, offsets: 0x2c(flags), 0x48-0x5c, 0x568-0x574, 0x590
- [x] HumanAnimController_Validate (0x005a4190) — validation with "Human animcontroller is invalid" error
- [x] HumanAnimController_ProcessAnimation (0x005a4b20) — 85+ xrefs, core animation processing
- [x] HumanAnimController_Cleanup (0x005a4660) — animation cleanup
- [x] Model_CreateVertexObject (0x00589d70) — 0x28-byte model/vertex object creator
- [x] Model_ClearVertexBuffers (0x00589c90) — vertex buffer clearing
- [x] Model_AllocateVertices (0x005f7c50) — model vertex allocation
- [x] Model_ProcessString (0x005f7ed0) — model string processing (2048-byte buffer)
- [x] AnimationStateMachine_Update (0x005fd2a0) — state machine with callback table @ 0x083997e0
- [x] System_InitializeAll deep analysis — 30/95 subsystems identified (31 still unknown)
- [x] Subsystems: Activate/Deactivate, Particle2D, InputPort, DirtyRect, MenuItem, MenuItemTask, PushButton
- [x] Subsystems: ParameterSystem, Level/Static/Dynamic/CameraBase, Serialization, EnumTypes
- [x] Subsystems: ScriptVM, MagicObjConfigContainer, MagicObjConfig, LevelScript, SkyTask
- [x] Subsystems: PhysicsObjType, PhysicsObj, PhysicsMagicObj, EditorRigidObj, DiscardTerrain, DiscardTerrainArea
- [x] Subsystems: CubeModifier, AnimTask, BspTreeEdit, TagItem(+variants), SplineObj, EditBoneObj, EditRigidObj
- [x] PatternMatcher patterns: TASKTYPE_MAGICOBJ, BONE, SHADOWVOLUME, HUMANSHADOW, PHYSICSMAGICOBJ
- [x] TagItem variants: Int32, Real32, String16, String256, Bool8, PushButton (6 types)
- [x] ScriptVM parameters documented: "Time factor", "Transition time" with float values

### Round 14 — NEW (2026-08-27) — 02-classes Extended Analysis
- [x] HumanAnimController discovered (FUN_005a4190) — "Human animcontroller is invalid" error
- [x] Model system: LoadModel, UnloadModel, InitModels, DeinitModels, zModel, VirModel
- [x] Animation system: Stand/Death/Idle/Driving/1stPerson/UpperBody/Main animations
- [x] FUN_00589d70 — Creates 0x28-byte model/vertex object with float params (0x43000000=4.0, 0x3f800000=1.0)
- [x] FUN_00589c90 — Clears model vertex buffer (short[4]=width/height, clears buffers at +0xc, +0x10)
- [x] FUN_005fd2a0 — Animation controller state machine (offsets 0x2c, 0x48, 0x4c, 0x50, 0x54, 0x58, 0x5c)
- [x] FUN_005a4660 — Animation cleanup (checks offset 0x56c, calls FUN_005fd2a0)
- [x] FUN_005f7c50 — Model vertex allocation (param_1 * unaff_EBX * 2 + 0x12 bytes)
- [x] FUN_005f7ed0 — __thiscall model string processing (2048-byte buffer)
- [x] AI Animation functions: AIFunction_GetAnimationToPlay, AIAction_PlayAnimation
- [x] Model file formats: %smodels/%s.res, LOCAL:models/%s.mef

### Round 13 — NEW (2026-08-27) — 02-classes Analysis
- [x] HumanPlayer vtable (0x006aa4a0) — 16 entries analyzed, addresses from unloaded DLL (0x2axxxx/0x2bxxxx)
- [x] Generic vftable (0x0067c7a8) — 16 entries: 6 functions + 3 NULL + 3 placeholders (0xFFFFFFFF) + 1 data
- [x] FUN_00660386 — __thiscall destructor/cleanup: frees memory if param_2 & 1
- [x] FUN_006609a3, FUN_00660a05 — __ArrayUnwind exception handling
- [x] FUN_00660371 — Sets vftable to &type_info::vftable, cleanup function
- [x] RTTI type_info structures — 8 C++ exception types decoded
- [x] Bad_cast, bad_typeid, __non_rtti_object, type_info, out_of_range, logic_error, length_error, exception

### Round 12 — NEW (2026-08-27)
- [x] ScriptVM_DispatchCallback (0x0054a060) — script callback dispatch via DAT_083997e0 table, 64-byte result buffer
- [x] ScriptContext_Cleanup (0x0054a160) — frees 3 memory blocks + 0x13 script frames, DList + 0xcdcdcdcd + _free
- [x] ScriptInterpreter_Execute (0x0054a2a0) — main script loop, tokenization, .isExists, "this." calls, // comments, 0x100 depth stack
- [x] ScriptVM_EvaluateExpressionEntry (0x0054a5f0) — expression evaluator entry point, calls FUN_005494f0 + ScriptVM_EvaluateExpressionFloat

### Round 11 — NEW (2026-08-27)
- [x] Mem_Alloc (0x0054c240) — 667 xrefs, custom allocator with alignment, 0xabababab debug fill, DList tracking, allocation failure handler
- [x] DList_RemoveFromContext (0x005520e0) — 89 xrefs, remove node from context DList, update prev/next pointers
- [x] ScriptVM_EvaluateToString (0x00565e40) — 97 xrefs, string conversion for script expressions, concat support
- [x] PatternMatcher_Match (0x0056ec80) — 65 xrefs, wildcard pattern matching ($, ., ?, *, [], ^, {}, [])

### Round 10 — NEW (2026-08-27)
- [x] ScriptVM_AddParameter (0x005772d0) — 1754 xrefs, add parameter to script VM context, 0x14-byte entry with type/value/name
- [x] SymbolTable_Add (0x0056a3c0) — 1101 xrefs, symbol registry with string name + 6 params + callback via DAT_07ab6c10
- [x] SymbolTable_Remove (0x0056a4c0) — 666 xrefs, symbol removal with "Undefined symbol" error, DList + 0xcdcdcdcd fill + free
- [x] ScriptVM_ReadParameter (0x005697f0) — 518 xrefs, read from script VM context (string ptr/raw value/strncpy modes)
- [x] ScriptVM_EvaluateExpression (0x005661c0) — 303 xrefs, int expression evaluator: add/sub/mul/div/bitwise/cmp/assign/ternary
- [x] ScriptVM_EvaluateExpressionFloat (0x00566730) — 229 xrefs, float10 expression evaluator, same opcodes as int variant
- [x] MemoryPool_Free (0x0055b880) — 214 xrefs, pool free with ref counting, DList removal, 0xcdcdcdcd debug fill

### Round 9 — NEW (2026-08-27)
- [x] EventDispatcher_DispatchCallback (0x0040ac10) — 46 xrefs, callback lookup via DAT_083997e0 + type*0x1ff
- [x] String_ParseInput (0x0040dea0) — 46 xrefs, 2052-byte stack buffer, parse/convert pipeline
- [x] Memory_ValidateBlock (0x004451b0) — 47 xrefs, alignment check, bounds validation
- [x] MemoryPool_Allocate (0x00445270) — 33 xrefs, 0xc-byte header + block pool allocator
- [x] Memory_Allocate (0x0041f3f0) — 27 xrefs, wrapper around FUN_0054c240
- [x] Memory_Deallocate (0x0041f410) — 27 xrefs, 0xcdcdcdcd debug fill, DList_RemoveNode, _free
- [x] RenderObject_Create (0x00463680) — 27 xrefs, alloc + init + DList_InsertNode
- [x] TaskCallback_Dispatch (0x0040e300) — 120 xrefs, indexed callback lookup
- [x] PooledAllocator_Alloc (0x0040baa0) — 161 xrefs, wrapper around FUN_0055b820
- [x] Config_LookupString (0x004335d0) — 49 xrefs, case-insensitive __stricmp search
- [x] ResourceManager_LoadAll (0x00425a10) — 64 xrefs, calls ResourceManager_Load(1)
- [x] RenderTask_Cleanup (0x004637c0) — 30 xrefs, callback + DList_RemoveNode + reinsert

### Round 8 — NEW (2026-08-27)
- [x] DList_RemoveNode (0x00445540) — 852 xrefs, doubly linked list node removal
- [x] FatalError_Handler (0x00446000) — 553 xrefs, fatal error with callback dispatch
- [x] Server_LogMessage (0x0041d470) — 347 xrefs, timestamped server logging
- [x] DList_InsertNode (0x00445460) — 217 xrefs, doubly linked list node insertion
- [x] ResourceManager_Load (0x00425740) — 143 xrefs, GLOB:/ and LOCA:/ resource loading
- [x] D3D_ErrorString (0x00448530) — 129 xrefs, D3DERR_* HRESULT to string converter
- [x] SoundDef_Find (0x00430730) — 100 xrefs, sound definition with GameEvent_Alloc
- [x] Performance_GetCounter (0x00435820) — performance counter with millisecond conversion
- [x] Render_DrawParticles (0x00427730) — 162 xrefs, D3D particle/effects rendering
- [x] Network_SendPacket (0x00419300) — 75 xrefs, network packet with 8180-byte buffer
- [x] QTaskList_FindByTypeAndCallback (0x004194c0) — 183 xrefs, find + callback at offset 0x28
- [x] QTaskList_Add (0x0041f310) — 60 xrefs, array-based task list append with overflow check
- [x] QTaskList_Remove (0x0041f340) — 56 xrefs, array-based task list removal by value
- [x] EventDispatcher_Dispatch (0x004378e0) — 61 xrefs, event handler table dispatch

---

## ANALYZED SYSTEMS (14 of 15) — All PARTIAL coverage

### 01. PE Structure 🟡 PARTIAL
- Entry point, CRT, WinMain (0x0065d36d, 0x00404280)
- 80+ imports across 11 DLLs
- 38 exported C++ methods
- 7 memory segments, 136MB total

### 02. C++ Classes ✅
- HumanPlayer: 10 states
- HumanSoldier: 1 state
- 18 vtable entries

### 03. Rendering ✅
- DirectX 7/8 pipeline
- Texture system (7 functions)
- MEF models + 5 LOD levels
- Camera + 4 filters

### 04. AI ✅
- 10 squad states
- 26+ events
- 30+ config API functions
- Graph nav: 1024 nodes, 12 properties

### 05. Weapons 🟡 PARTIAL
- 11 categories, 8 types
- 27 parameter fields
- Grenade system
- 6 sniper overlays
- 14 body-part damage multipliers

### 06. Level System ✅
- 28-step init pipeline
- QTask/QVM scripting
- Objectives (8, 672 bytes)
- Spawn areas + respawn timers
- Graph navigation

### 07. QVM Scripting ✅
- 30+ opcodes
- Frame structure (0xD20 bytes)
- 88+ QTask types
- 25 AIFunctions
- 23 known QSC files

### 08. Resource Formats ✅
- ILFF archive format
- 15+ file formats decoded
- MEF, TEX, LMP, FNT, SPR, IFF, RES, DAT, QSC
- zlib compression

### 09. Audio ✅
- Miles Sound System (37/37 AIL functions)
- 3D positioning system
- 3072 sound slots
- DirectMusic integration
- 12 sound definition classes

### 10. Menus ✅
- 14+ menu screen types
- Widget system (buttons, text, sliders)
- 90+ menu items
- HUD sprites (15+)
- DirectInput setup (keyboard, mouse, joypad)

### 11. Configuration ✅
- 8 config files documented
- 70+ config options
- Player profiles system
- Save system (6 slots)

### 12. Networking 🟡 PARTIAL
- UDP-based client/server
- 27 GM message types
- 7 game modes
- 50+ server config options
- Guaranteed/non-guaranteed delivery

### 13. Global Data ✅
- 13 critical data areas
- 14-entry AI state machine jump table
- Input action dispatch table
- Entity struct (0xd80 bytes ≈ 3456 bytes)
- Texture format table (DXT formats)
- D3D API dispatch table (96 entries)

### Cross-References ✅
- 5 call graphs documented
- 100+ string references mapped
- Global data dependencies traced

---

## GHIDRA ENHANCEMENT — IN PROGRESS (807 total renamed across all rounds)

### COMPLETED RENAMES — ROUND 1 (55+ functions)

#### QVM System (12 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x005699C0 | QVM_ExecuteBytecode | Main bytecode interpreter |
| 0x00569650 | QVM_ExecuteLoop | Opcode dispatch loop |
| 0x005696A0 | QVM_PrintErrorLog | Error logging |
| 0x0040DC60 | QVM_LoadScriptFile | Script file loading |
| 0x0056D450 | QVM_AllocateFrame | Frame memory allocation |
| 0x0056D160 | QVM_InitializeFrame | Frame initialization |
| 0x0056D1C0 | QVM_DeallocateFrame | Frame deallocation |
| 0x0056E2D0 | QVM_ParseToken | Token parsing |
| 0x0056D930 | QVM_ParseBlock | Block parsing |
| 0x0056DE40 | QVM_FreeBlock | Block deallocation |
| 0x0056DDB0 | QVM_ReadToken | Token reading |
| 0x005670C0 | QVM_ParseExpression | Expression parsing |

#### Game Core (10 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00401210 | Game_ApplicationInit | WinMain entry point |
| 0x00401040 | Game_ParseSaveGamePath | Save game path parsing |
| 0x00401440 | Game_MainLoop | Main loop with setjmp |
| 0x00401490 | Game_ProcessGameState | State dispatch |
| 0x00401600 | Game_LoadLevel | Level loading |
| 0x00401970 | Game_UpdateInput | Input processing |
| 0x00401A30 | Game_ProcessEvents | Event processing |
| 0x00401B20 | Game_RunFrame | Frame processing |
| 0x00402C00 | Game_ResetTiming | Timing reset |
| 0x00402CF0 | Game_CalculateFrameTime | Frame time calculation |

#### AI System (8 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x005D3D80 | AI_SendHeartbeat | UDP heartbeat to servers |
| 0x005B2600 | AI_EntityUpdate | Entity behavior loop |
| 0x005B3AA0 | AI_EntityCleanup | Memory cleanup |
| 0x005B3B60 | AIGraph_CreateCoverNode | Cover node creation |
| 0x005B44A0 | AIGraph_LinkNode | Node linking with position |
| 0x005AF1A0 | AI_AssignNodeID | Node ID assignment |
| 0x0054A030 | AI_EnumerateEntities | Entity list iteration |
| 0x005AF4C0 | AIGraph_ManageNode | Node slot management |

#### Networking (2 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x005D4A10 | Network_SendHeartbeat | UDP heartbeat packet |

#### Level System (3 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00549B50 | Level_AllocateQTaskID | QTask ID allocation |
| 0x0054AD40 | Level_RegisterTypes | Type system init |

#### Config/String (3 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x004281C0 | Config_SetDefaults | Default config values |
| 0x0065CC30 | String_FormatPrint | Formatted string output |
| 0x005903C0 | String_ParseToken | String token parsing |

#### Rendering/Graphics (7 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x005D6040 | System_InitializeAll | All subsystems init |
| 0x00534C80 | MissionTimer_Initialize | Mission timer system |
| 0x00527CB0 | ActionMenuScreen_Initialize | Action menu/HUD |
| 0x005815C0 | Terrain_Initialize | Terrain system |
| 0x005A7870 | Water_Initialize | Water rendering |
| 0x005BA990 | Vehicle_Initialize | Vehicle/hitzone system |
| 0x00599CF0 | MagicObjConfig_Initialize | Shadow/magic objects |

#### Serialization (4 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x005597B0 | Serialization_RegisterHandlers | Serialize/DeSerialize |
| 0x00577700 | ParameterSystem_Initialize | Parameter type system |
| 0x00577140 | ParameterType_Register | Type registration |
| 0x005772D0 | ParameterType_Assign | Type assignment |

#### QTask System (2 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x0056A3C0 | QTask_RegisterType | QTask type registration |
| 0x00577450 | ParameterType_Copy | Type cloning |

#### Other (2 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00401000 | ParseString | Quoted string parsing |
| 0x005FDE60 | AIGraph_RegisterNodeCriteria | Node criteria registration |

### ✅ PLATE COMMENTS ADDED (55+ functions)
All renamed functions have detailed plate comments explaining:
- Function purpose
- Key algorithms
- Data structures used
- Network protocols (UDP heartbeat format)
- File paths (MISSION:graphs/graphcover%d.dat)

### ✅ NEW RENAMES — ROUND 3 (110+ functions across 4 subsystems)

#### Weapon System (27 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00522550 | WeaponSystem_Initialize | Function table registration (Weapon_Fire, Weapon_Reload, etc.) |
| 0x0051ece0 | WeaponSystem_DispatchEvent | Event routing by magic constant (FRWS, LRWS, ATWS, SRWS, MZWS) |
| 0x0051f0d0 | WeaponSystem_Update | Weapon state updates, fire timers, cooldowns |
| 0x0051f5d0 | WeaponSystem_PositionOverlay | Weapon overlay position on character model |
| 0x00520ae0 | WeaponSystem_Fire | Fire handler: validates state, triggers animations/sounds, ammo consumption |
| 0x00520d70 | WeaponSystem_ProcessEvent | Event dispatcher |
| 0x00520fc0 | WeaponSystem_FrameUpdate | Frame-by-frame weapon system update |
| 0x005201b0 | WeaponSystem_Zoom | Zoom in/out handler |
| 0x00520250 | WeaponSystem_Activate | Single weapon activation handler |
| 0x005209c0 | WeaponSystem_SetAim | Weapon position and aim setter |
| 0x00520e70 | WeaponSystem_TransitionState | State transition handler |
| 0x005211b0 | WeaponSystem_LookupType | Weapon type lookup by ID/name |
| 0x005212e0 | WeaponSystem_CheckAmmo | Ammo status check |
| 0x00521410 | WeaponSystem_GetFireParams | Fire parameters retrieval |
| 0x00521e10 | WeaponSystem_Reset | System state reset |
| 0x00521cb0 | WeaponSystem_DropEquip | Drop/equip handler |
| 0x00522290 | WeaponSystem_QueueSound | Sound queue management |
| 0x00522330 | WeaponSystem_PlaySound | Sound playback |
| 0x005224b0 | WeaponSystem_ApplyDamage | Damage application |
| 0x0051ebf0 | WeaponSystem_InitParamTypes | Parameter type initialization |
| 0x004a4960 | WeaponPlayer_Update | Main player weapon update loop |
| 0x004b6450 | WeaponFire_Handler | Fire handler with animation state |
| 0x004b7380 | WeaponFire_StateMachine | Fire state machine (states 0-7) |
| 0x004b7b90 | WeaponReload_Handler | Reload state machine (states 0,1,4) |
| 0x004b8a20 | WeaponFireSequence_Handler | Fire sequence and animation management |
| 0x004fd7e0 | WeaponPrecision_Calculate | Spread, recoil, aim correction |
| 0x00490420 | ObjectSystem_Initialize | Object system (WeaponHit, ExplosionHit registration) |

#### Rendering/Graphics (11 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00421500 | Rendering_DetectD3DDrivers | D3D hardware acceleration scan, best resolution selection |
| 0x00564D20 | Rendering_SetTextureFiltering | Bilinear/anisotropic/trilinear texture filtering |
| 0x005827C0 | Rendering_InitTextureSystem | Alloc/DeAlloc/DownloadTexture, GetTMUs, SetActiveTMU, RefreshTexture, SetLOD |
| 0x00585510 | Rendering_DrawWater | D3D water renderer with vertex/pixel processing |
| 0x00586FF0 | Rendering_InitForestBSP | Forest BSP renderer (max 19 instances) |
| 0x00596770 | Rendering_InitRenderPipeline | All render methods: DrawRigidMesh, DrawBoneMesh, DrawSplineMesh, DrawLightmapMesh, DrawForest |
| 0x005DC080 | Rendering_InitPaletteSystem | AllocPalette, DeAllocPalette, DownloadPalette |
| 0x005F6840 | Rendering_InitDiscardTask | D3D resource discard management |
| 0x004F7AD0 | Model_LoadMEF | MEF model loader: SHEF/HIER/NAME/TEXF/VTXF/PRTF/TCVF/TCXF/CMSH/CVTF/CFCE/CMAT/CSHP chunks, v0.16 |
| 0x004F8230 | Model_LoadMEFResource | MEF model resource loader: FACE/VTXF/EDGE chunks |
| 0x004F89B0 | Model_LoadMTPTextures | MTP model texture and resource loading |
| 0x004F9030 | Rendering_InitModelMethods | LoadModel/UnloadModel/InitModels/DeinitModels registration |
| 0x00611E70 | Rendering_WireTextureCallbacks | Texture function pointer wiring |
| 0x00612E30 | Rendering_WireAllTextureOps | All 12 texture operations wiring |
| 0x0061B0C0 | Rendering_InitializeAll | Master graphics initialization |

#### Network (17 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00442670 | Network_ReceivePacket | UDP receive: select() timeout + recvfrom(), 200-entry ring buffer |
| 0x00440db0 | Network_SendPacket | UDP send: select() + sendto() |
| 0x0043f7c0 | Network_BindUDPSocket | Bind client UDP socket, retry 10 ports |
| 0x0043f690 | Network_CloseUDPSocket | Close global UDP socket |
| 0x0061b300 | Network_Shutdown | Full shutdown: WSACleanup(), free queues, destroy NID pool |
| 0x004421c0 | Network_StartServer | UDP server: FIONBIO, 35-entry NID pool, sends/receive queues |
| 0x00442cb0 | Network_StartClient | UDP client: FIONBIO, NID pool, queue allocation |
| 0x00443b50 | Network_FindServer | Server enumeration: UDP broadcast to master servers |
| 0x00443260 | Network_SendConnectRequest | Connect request with password, 500ms timeout (25x20ms) |
| 0x00441c30 | Network_ProcessServerPacket | Server packet dispatcher: CLIENTCONNECT, QUIT, SERVERINFO, DISCONNECT |
| 0x0051ba80 | Network_ProcessSpawnMessage | Spawn approval/denial, weapon assignment (knife, glock/makarov) |
| 0x0051b160 | Network_ProcessServerAnnounce | SERVER ANNOUNCE handling, weapon/round state matching |
| 0x0041d8a0 | NetworkManager_RunInterrupt | Network interrupt processing, pending tasks |
| 0x004352d0 | NetworkManager_RebalanceTeams | Server-side team rebalancing |
| 0x0044e120 | Network_CheckConnectionQuality | PING, PLOSS, IDLE, GOUT validation |
| 0x0044e6d0 | NetworkServer_InitHandlers | 22 message handler registration, encryption seed |
| 0x005c7eb0 | NetworkManager_ParseServerList | Server list response parsing |
| 0x005c7bb0 | NetworkManager_AddDiscoveredServer | Add discovered server to list |

#### AI System (29 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x005B28A0 | AI_GraphInitialize | AI navigation graph creation |
| 0x0054A030 | AI_EntityEnumerate | AI entity list iteration |
| 0x005AF1A0 | AI_NodeAssignID | Node ID assignment |
| 0x005D3D80 | AI_HeartbeatSend | UDP heartbeat to servers |
| 0x005B3B60 | AI_CoverNodeCreate | Cover node creation |
| 0x005B44A0 | AI_GraphLinkNode | Node linking with position math |
| 0x005AF4C0 | AI_NodeManage | Node slot management |
| 0x005FDE60 | AI_NodeCriteriaRegister | Node criteria registration |
| 0x005B3040 | AI_PathfindAStar | A* pathfinding: f=g+h, terrain penalty +81920, 2000-node lists |
| 0x005B2E80 | AI_NodeSlotFind | Free node slot finder |
| 0x005B2F20 | AI_PathInsertNode | Insert node into path (A*) |
| 0x005B2F60 | AI_PathExtendNode | Extend path through nodes (A*) |
| 0x005B3700 | AI_PathClearLists | Clear A* open/closed lists |
| 0x005B41A0 | AI_GraphConnectivityAnalyze | Graph connectivity analysis |
| 0x005AF2A0 | AI_EntityGetByID | Get AI entity by ID |
| 0x005AF330 | AI_PositionTransform | 3D coordinate transformation |
| 0x005AF520 | AI_NodeDataSizeGet | Node data size index |
| 0x005AF6F0 | AI_NodePositionUpdate | Node position update |
| 0x005AF750 | AI_NodePositionGet | Node position pointer |
| 0x005AF7A0 | AI_MatrixTransform | Matrix-vector multiplication |
| 0x005B1AD0 | AI_GraphDataInit | Graph data type initialization |
| 0x005B2BC0 | AI_GraphShutdown | Graph system shutdown |
| 0x005B37A0 | AI_NodeAdjacencyCompute | Adjacency matrix: 4x8 grid, trigonometric sampling |
| 0x005AE7E0 | AI_NodeInit | Node initialization handler |
| 0x005AE4D0 | AI_NodeCleanup | Node cleanup handler |
| 0x005AE8C0 | AI_NodeTransformUpdate | Node position/rotation with 3x3 rotation matrices |
| 0x005AEFB0 | AI_NodeLoad | Node load handler |
| 0x005AE620 | AI_NodeSave | Node save handler |
| 0x005B0160 | AI_NodeCriteriaApply | Node criteria handler |
| 0x005B0200 | AI_NodeLinkTypeApply | Node link type handler |

### ✅ PLATE COMMENTS ADDED (20+ detailed comments)
Key functions with detailed plate comments:
- **WeaponSystem_Initialize** — function table mapping script names to C handlers
- **WeaponSystem_DispatchEvent** — magic constant routing (FRWS, LRWS, ATWS, SRWS, MZWS)
- **WeaponSystem_Fire** — validates state, triggers animations/sounds, ammo consumption
- **WeaponReload_Handler** — reload state machine (states 0,1,4)
- **WeaponPrecision_Calculate** — spread, recoil, aim correction
- **AI_PathfindAStar** — f=g+h, Euclidean SQRT, terrain penalty +81920
- **AI_NodeAdjacencyCompute** — 4x8 grid, pi/32 trigonometric sampling
- **Model_LoadMEF** — 13 chunk types, version 0.16 validation
- **Network_ReceivePacket** — select() + recvfrom(), 200-entry ring buffer
- **Network_StartServer** — FIONBIO, 35-entry NID pool
- **Network_ProcessSpawnMessage** — weapon assignment (knife, glock/makarov)
- **Rendering_InitTextureSystem** — 12 texture operations
- **Rendering_InitRenderPipeline** — 10+ render methods
- **Rendering_DetectD3DDrivers** — D3D hardware scan
- **System_InitializeAll** — camera, physics, serialization, bone dynamics, terrain, water, vehicles

### ✅ NEW RENAMES — ROUND 4 (38+ functions across 8 subsystems)

#### Object System / Collision (6 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x005502e0 | CollisionEval_Initialize | Registers CollisionEval, LineOfSight, TraceLines, GetNextZ, GetMaterial, ModifyGroundMaterial |
| 0x00576360 | DynCubeObj_CollisionHandler | BoneDynCubeObj type init with CollisionEval message binding |
| 0x0053b4e0 | DoorObject_Action | Door system init with StopOnCollision, physics params (0xc0240000=-2.0f, 0x40240000=2.0f) |
| 0x00453da0 | HumanRemote_Action | HumanRemote type init with WeaponHit/ExplosionHit handlers |
| 0x00507310 | Human_Action | Full Human subsystem: 15+ msg IDs, 40+ handlers, HumanShadow subtype, 80+ params, collision flags |
| 0x005bd400 | Particle2DCollision_Action | Particle2DDynCubeObjTask init with Particle2DCollisionEvaluate binding |

#### Serialization (8 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00558ef0 | Serialization_Handler_Serialize | Iterates task ref linked list, dispatches via DAT_083997E0 table |
| 0x00559080 | Serialization_Handler_DeSerialize | Entry point, checks GetSerializeParameterList, calls dispatcher |
| 0x00559280 | Serialization_Dispatcher_DeSerialize | Iterates 0x14-byte task entries, vtable dispatch at DAT_07AC30C8 |
| 0x005595c0 | Serialization_Handler_SerializeInfo | SerializeInfo handler, calls DefineTaskState, CommaList, FinalizeString |
| 0x005590f0 | Serialization_BuildDefineTaskState | Builds "DefineTaskState" string with serialized params |
| 0x00559150 | Serialization_BuildCommaList | Comma-separated parameter value list builder |
| 0x00559230 | Serialization_FinalizeString | Null terminator, newline, handler dispatch from DAT_0070488C |
| 0x00559630 | Serialization_FormatTaskChain | Formats task chain info: "%d, \"%s\", %d" with parent traversal |

#### Level / QTask (2 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00549d70 | Level_ProcessQTaskTree | Recursive QTask tree walker, dispatches via DAT_083997E0 |
| 0x00558c50 | Level_CreateAndSpawnQTask | Creates QTask entity, calls UpdateInternalData, allocates IDs |

#### Rendering (3 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x005f4f40 | Rendering_DrawTerrain | Registers DrawTerrain in render method table |
| 0x005ee470 | Rendering_DrawLightmap | Registers DrawLightmap with EnumerateLightmap, 16 rotation matrices |
| 0x0061a4f0 | Rendering_MasterInitialize | Master render init: envmap, model methods, discard, lightmap, water, terrain |

#### Entity/Object Initialization (12 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00545f00 | Filter_InitVideoFilters | Registers LineFilter, WideScreenFilter, BandFilter, HandiCamFilter, ComputerFilter |
| 0x004935b0 | Camera_Initialize | SCamera/SCameraLens types, DrawCameraCone/DrawComputerObject handlers |
| 0x004f2480 | ComputerObject_Initialize | ComputerObject type with DrawComputerObject/DrawCameraCone |
| 0x0046fbc0 | Enemy_Initialize | Enemy type with DrawComputerObject handler |
| 0x0049dbf0 | AISquad_Initialize | AISquad type, AIType_Offensive/Defensive/OnVehicle |
| 0x004f1670 | C4Bomb_Initialize | C4Bomb/C4BombArea/RemoteBomb types |
| 0x00518880 | AIStationaryGunHolder_Initialize | AIStationaryGunHolder with DrawComputerObject |
| 0x004dcc30 | Fence_Initialize | Fence type with DrawComputerObject |
| 0x00480fc0 | PatrolEnemy_Initialize | Patrol enemy type with DrawComputerObject |
| 0x00524190 | ConditionalContainer_Initialize | ConditionalContainer with DrawComputerObject |
| 0x00510510 | ComputerHighlight_Initialize | ComputerHilight with DrawComputerObject |
| 0x004c0730 | HumanPlayer_Initialize | HumanPlayer type with HumanUpdate/DrawComputerObject |

#### HUD/Draw (1 function)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00510400 | ComputerHighlight_Draw | Draw callback for ComputerHilight, iterates highlight list |

#### Debug Console (1 function)
| Address | Name | Purpose |
|---------|------|---------|
| 0x0054f970 | DebugConsole_DrawConsoleWindow | Debug console with background alpha, command history |

### PLATE COMMENTS ADDED ROUND 4 (6 detailed comments)
- **CollisionEval_Initialize** — Registers 6 message IDs, zeros 80 bytes global state
- **Human_Action** — Full Human subsystem: 15+ msg IDs, 40+ handlers, 80+ params, collision flags
- **Serialization_Handler_Serialize** — Linked list iteration, dispatch table lookup
- **Serialization_Dispatcher_DeSerialize** — 0x14-byte task entries, vtable dispatch
- **Level_ProcessQTaskTree** — Recursive QTask tree walker, nested children processing
- **Rendering_DrawTerrain** — Terrain rendering pipeline registration
- **Serialization_FormatTaskChain** — Formats task chain info: "%d, \"%s\", %d" with parent traversal
- **MissionTimer_Tick** — State machine: expression match → countdown → cleanup
- **NetMission_EventHandler** — Main mission event handler: timer countdown, expressions, QTask children, isMissionComplete/Failed flags
- **NetObjective_EventHandler** — Evaluates objective expressions (Complete/Failed), broadcasts via FUN_00457a10

### ✅ NEW RENAMES — ROUND 4 PART 2 (60+ additional functions)

#### Mission/Objective System (9 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00534ae0 | MissionTimer_Tick | State machine: expression match → countdown → cleanup |
| 0x004cffb0 | MissionSystem_Initialize | Registers Network_Mission/Objective QTasks, 0x48/0x84-byte params |
| 0x00414030 | NetMission_StartMission | Validates init, sets camera, weapon transfer, mission state machine |
| 0x004cf540 | NetMission_EventHandler | Main mission tick: timer countdown (0x34 field), expressions, QTask children |
| 0x004cfb70 | NetMission_Destroy | Finds mission QTask by ID, frees resources |
| 0x004cf110 | NetObjective_EventHandler | Evaluates objective expressions (Complete/Failed), broadcasts objective ID |
| 0x004cfc00 | NetObjective_OnComplete | Sets current objective ID, marks complete, updates mission state |
| 0x0049d230 | AISquad_RegisterTaskTypes | Registers 45+ AISquad QTask types with event codes 0-17 |
| 0x00494c80 | SequenceCommand_Initialize | SequenceCommand QTask for mission sequence/flow control |

#### AISquad (9 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x0049d980 | AISquad_StartCombat | Sets combat flag (param_1+0x2212), iterates squad members |
| 0x0049da20 | AISquad_EndCombat | Clears combat flag, cleanup handlers |
| 0x0049b480 | AISquad_Cleanup | Frees all squad entities, stops state machines, clears task lists |
| 0x0049e920 | AISquad_GetDistanceToEvent | Returns distance from squad to event location (param_1+0x21a0) |
| 0x0049ea00 | AISquad_SetAlertFlags | Sets alert flags on all squad members (entity+0xf8c) |
| 0x0049ea50 | AISquad_ReceiverHasTarget | Checks if squad member has target (in_EAX[0x12]) |
| 0x0049ebc0 | AISquad_ThrowGrenade | Sets grenade params at param_1+0x21d4/0x21d8 |
| 0x0049ed20 | AISquad_MoveToNode | Copies position data, calls movement function |
| 0x0049f690 | AISquad_TriggerEvent | Triggers event on target entity via dispatch table |

#### Parameter Type Handlers (11 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00547390 | ParameterTypeHandler_Quat32_Copy | Copy handler for Quat32 (5 floats: x,y,z,w,scalar) |
| 0x00546CC0 | ParameterTypeHandler_Real32x3_Copy | Copy handler for Real32x3 (3 floats: x,y,z) |
| 0x00546E60 | ParameterTypeHandler_Real32x9_Copy | Copy handler for Real32x9 (9 floats, 3x3 matrix) |
| 0x00546F90 | ParameterTypeHandler_Real32x9_CopyCallback | Formats 3 floats as "%.20G, %.20G, %.20G" |
| 0x005476F0 | ParameterTypeHandler_Real64x3_Copy | Copy handler for Real64x3 (3 doubles) |
| 0x00547C60 | ParameterTypeHandler_VarString_MemoryManage | Free old string, malloc new, copy content |
| 0x005468E0 | ParameterTypeHandler_ObjectPos_Format | Format 3 doubles with offset from DAT_08663dc0 |
| 0x00546A00 | ParameterTypeHandler_GraphEnvelope_Parse | Parse handler for GraphEnvelope (11 floats) |
| 0x00546AD0 | ParameterTypeHandler_GraphEnvelope_CopyCallback | Formats 11 floats as comma-separated values |
| 0x00547030 | ParameterTypeHandler_Real32x9Exact_Parse | Parse handler for Real32x9Exact (10 floats) |
| 0x00548030 | ParameterTypeHandler_LevelExpression_Parse | Parse handler for LevelExpression (string expression) |
| 0x00592AB0 | EnumTypes_Register | Registers EnumInt32, EnumReal32, EnumString16/32/256 |

#### WeaponSystem (10 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00522000 | WeaponSystem_FindWeapon | Find weapon by ID in linked list at offset 0xb0 |
| 0x00522050 | WeaponSystem_FindWeaponByIndex | Find by index, checks enabled flag at offset 0x2a0 |
| 0x00522120 | WeaponSystem_Cleanup | Clean up weapon arrays (2x3 and 10 entries) |
| 0x005221d0 | WeaponSystem_GetTotalDamage | Sum weapon damage from linked list, offset 0x36c |
| 0x00522240 | WeaponSystem_AddWeaponName | Add weapon name to queue (max 10, max 0x20 chars) |
| 0x00522300 | WeaponSystem_ProcessSoundQueue | Process queued weapon sounds |
| 0x00522520 | WeaponSystem_DropAllEquipment | Drop all equipped weapons from linked list |
| 0x005229e0 | WeaponSystem_Shutdown | Full subsystem cleanup — frees DAT_079f0f20 array |
| 0x00522cc0 | WeaponSystem_UpdateAimPosition | Update aim/crosshair with screen bounds (320x240) |
| 0x00522e10 | WeaponSystem_RegisterCursor | Register cursor parameter/callbacks |

#### AI Pathfinding/Cover (5 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x005b3520 | AI_CoverGraphLineSegmentIntersect | Line segment intersection with cover graph bounds |
| 0x005b3cb0 | AI_LoadCoverGraph | Load cover graph from MISSION:graphs/graphcover%d.dat, IFFI/ICCI format |
| 0x005b3ea0 | AI_CoverGraphAngleGet | Compute angle in cover graph coordinate space (0→5.092958 rad) |
| 0x005b3f40 | AI_CoverGraphNodeExists | Check if cover graph node exists and is active |
| 0x005b3fd0 | AI_CoverGraphNodeRemoveFlag | Remove flag from cover graph node |

#### Network (3 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00442980 | Network_ProcessServerPackets | Process incoming packets with guaranteed delivery, IP logging |
| 0x00442ec0 | Network_StartSockLib | Initialize Winsock: WSAStartup(0x101), version/description |
| 0x00442fc0 | Network_ProcessClientPackets | Process client packets, validate guaranteed messages g_in linked list |

#### Texture System (9 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00582140 | Texture_BilinearSample | Bilinear texture sampling with coordinate lookup, 4-pixel interpolation |
| 0x005824b0 | Texture_CountValidTextures | Count valid textures: TEX/BOB signatures, alignment check |
| 0x005825a0 | Texture_LoadFromResource | Load textures from resource block, parse TEX/BOB magic numbers |
| 0x00582b30 | Texture_ClearState | Clear texture state array entries (12 entries) |
| 0x00582c10 | Texture_Create | Create new texture with 14 local entries |
| 0x00582d60 | Texture_SetState | Set texture state with 7 parameters, callback at DAT_084a2144 |
| 0x00582e00 | Texture_CreateFull | Full texture creation with 10 params, 2 callbacks |
| 0x00582ea0 | Texture_Destroy | Destroy texture and free resources |
| 0x00582f30 | Texture_LoadFromFile | Load texture from file (LOOP magic), parse header 0x10-0x20 |

#### Serialization/Lensflare (5 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x005580d0 | Lensflare_UpdateParameter | Update lensflare effect params, reads offsets 0x74/0x78/0x7c |
| 0x00558300 | Serialization_InitLensflare | Initialize lensflare: creates Lensflare/LensflareItem types, 8 ParameterType_Assign |
| 0x005585d0 | Serialization_CleanupLensflare | Cleanup lensflare: frees DAT_079fd6c0 and DAT_079fd7a0 |
| 0x005587e0 | Serialization_FindEffectType | Find effect type by name via __stricmp against DAT_079fd7d4+0x30 |
| 0x00558890 | Serialization_AddEffectEntry | Add new effect type entry (max 0x28 entries) |

#### Other (3 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x0049fac0 | ExplodeObject_Initialize | Registers ExplodeObject QTask with 0x110-byte explosion params |
| 0x00494600 | MenuFrame_Initialize | Registers MenuFrame QTask for menu/frame rendering |

### ✅ PLATE COMMENTS ADDED ROUND 4 PART 2 (4 detailed comments)
- **MissionTimer_Tick** — State machine: expression match → countdown → cleanup
- **NetMission_EventHandler** — Main mission event handler: timer countdown, expressions, QTask children, isMissionComplete/Failed flags
- **NetObjective_EventHandler** — Evaluates objective expressions (Complete/Failed), broadcasts via FUN_00457a10
- **MissionSystem_Initialize** — Registers Network_Mission/Objective QTasks, 0x48/0x84-byte params

### ✅ NEW RENAMES — ROUND 5 (150+ functions across 8 subsystems)

#### Physics System (12 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x005c3ed0 | PhysicsObj_Action | Core PhysicsObj handler — Move, UpdateOrientation, Hit callbacks |
| 0x004838d0 | GenericPhysicsObj_Action | GenericPhysicsObj with full lifecycle (create=0, update=1, delete=2) |
| 0x004d4380 | GenericPhysicsMagicObj_Action | Physics object with magic/enchantment properties |
| 0x005d5f40 | PhysicsMagicObj_Action | Direct physics-magic hybrid object |
| 0x006095b0 | MoveRigidObj_Action | Position/rotation control with Alpha/Beta/Gamma and velocity params |
| 0x005ca4a0 | EditRigidObj_Action | Editor/interactive rigid object manipulation |
| 0x005a31f0 | RigidDynCubeObj_Action | Dynamic cube-based rigid body with physics collision |
| 0x00606a60 | AnimRigidDynCubeObj_Action | Animated dynamic cube rigid body with 8 callback slots |
| 0x006114c0 | EditRigidObjAttachedToBone_Action | Rigid object attached to skeleton bone |
| 0x0043d420 | PhysicsObjType_ParseConfigFile | Physics object type config/script loader |
| 0x0053ccc0 | Smoke_Action | Smoke particle system with Gravity factor parameter |
| 0x00594980 | EditorRigidObj_Action | Editor-only rigid object visualization/debugging |

#### Audio/Miles Sound System (14 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00542570 | Sound_InitializeMiles | Miles Sound System core init: _AIL_startup, _AIL_open_digital_driver, 3D provider selection |
| 0x00606dd0 | SoundManager_Register | SoundManager task init with parameter list update callback |
| 0x006099e0 | SoundDef_TriggerOnce_Register | Trigger-once sound with Falloff Begin/End params |
| 0x0060d1c0 | SoundDef_Sound_Register | Sound definition with Falloff Begin/End params |
| 0x00610240 | SoundDef_Graph_Register | Graph sound with Falloff Begin/End params |
| 0x006111a0 | SoundDef_Group_Register | Sound group container registration |
| 0x00489950 | SoundGenerator_Register | Play/stop callbacks, sound generator pools |
| 0x00489b80 | SoundGenerator_Shutdown | Frees SoundGenerator pools |
| 0x004898f0 | SoundGenerator_PlaySound | Play sound handler with sound lookup and handle storage |
| 0x00489820 | SoundGenerator_StopSound | Stop sound handler with handle clearing |
| 0x004895f0 | SoundGenerator_Stop3DSound | Stop 3D sound via 3D sample table iteration |
| 0x00407420 | Game_InitSoundSystem | Main game sound init: loads common/location/mission sounds, sets up music |
| 0x00431160 | MenuMusic_Register | Menu music init with "menu_music" parameter |
| 0x00420a40 | GOSound_Output | Game object sound output: Music/Speech/FX calls to buffer |
| 0x00605cd0 | AnimSound_Register | Animation sound with parameter callbacks |
| 0x00472290 | RadioSound_Register | Radio sound system with start/stop/volume callbacks |

#### Input System (5 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x004262c0 | InputSubsystem_RegisterTypes | ~100+ input QTask types, movement bindings, keyboard devices |
| 0x00420ca0 | Input_LogConfig | Logs mouse sensitivity, invert, key remappings |
| 0x00435f10 | InputPort_Initialize | InputPort subsystem with global device state zeroing |
| 0x0052d3e0 | HumanPlayerInput_Initialize | HumanPlayerInput with record/playback handlers |

#### Menu/UI System (10 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x005c6c30 | MenuItemTask_Initialize | MenuItemTask with input/hitbox/navigation handlers |
| 0x005c5e40 | MenuItem_Initialize | MenuItem with Get/Activate/Update handlers |
| 0x0060aee0 | PushButton_Initialize | PushButton with input/hitbox registration |
| 0x0059ae10 | TagItem_Initialize | TagItem base + derived types (PushButton, Int32, Real32, String16/256, Bool8) |
| 0x00433c50 | MenuManager_Initialize | Screen management, input handling, button callbacks |
| 0x00433e90 | MenuManager_Deinitialize | Frees parameter structure, clears active menu screen |
| 0x00468ce0 | MenuButton_Initialize | MenuButton with input/activation/click handlers |
| 0x004cb030 | ComputerButtonItem_Initialize | Computer/console menu interaction with hitbox registration |

#### Terrain System (6 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x005512e0 | Terrain_MaterialLoad | Loads .tim/.phm/.dmc/.csq terrain resources, allocates tile/material maps |
| 0x00607890 | Terrain_DiscardArea | DiscardTerrainArea action type registration |
| 0x00604b20 | Terrain_Discard | DiscardTerrain with 15+ callbacks |
| 0x00604430 | Terrain_OLELoader | Terrain resource loading subsystem |
| 0x0055b360 | TerrainLightMap_Init | Terrain lightmap system with magic constants |
| 0x00580810 | Terrain_HeightmapLoad | Loads .thm/.tmm/.tlm height/material/lightmaps |

#### Water System (4 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x0060ccd0 | Water_SubsystemInit | FlatWater/FlatWaterLayer action types, 10+ parameter types |
| 0x00586ea0 | Rendering_DrawWaterRegister | DrawWater render method registration (128 slots) |
| 0x004aee70 | Water_LandTransition | Player water/land state transitions (swim/tread/underwater) |
| 0x005a55b0 | Water_MergeScript | Generates water_w_01 through water_w_08 resource merge script |

#### HitZone System (1 function)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00467800 | HitZone_Init | TASKTYPE_HITZONE with 14+ callbacks, 0x480c-byte structure |

#### Scripting / QVM / QTask System (20 functions)
| Address | Name | Purpose |
|---------|------|---------|
| 0x00565870 | Opcode_GetParameterCount | Opcode parameter count lookup — maps opcodes (1-0x1d) to expected arg counts (1-10) |
| 0x00569060 | Opcode_Execute | Opcode executor — dispatches opcode execution, checks \r/\x11 byte, aborts on unhandled |
| 0x0056c2b0 | Script_ExecuteBytecode | Main bytecode interpreter loop — allocates VM frame, loads script, dispatches commands |
| 0x0040b840 | QTask_Initialize | QTask system init — clears arrays, registers "QTask" type, allocates 0x1f80-entry list |
| 0x0041f310 | QTaskList_Add | QTaskList add — appends QTask to list, checks overflow, aborts "QTaskList is full" |
| 0x0043da60 | Task_New | Task_New symbol context setter — resolves symbol, loads script, sets execution context |
| 0x00549640 | Task_New_Execute | Executes Task_New script — resolves symbol, loads file, executes via QVM |
| 0x005cc910 | Console_ExecuteCommand | Console command dispatcher — parses command, looks up handler, "Unknown command: '%s'" |
| 0x0040e680 | Level_Initialize | Level/Static/Dynamic/CameraBase type init, calls QTask_RegisterType for each |
| 0x00608950 | NoiseQTask_Register | NoiseQTask type with 15 parameters and callback handlers |
| 0x006054c0 | SmoothQTask_Register | SmoothQTask type with 10 parameters and callbacks |
| 0x00605ed0 | LevelScript_Register | LevelScript type with "RunScript" command and 1 parameter |
| 0x0060a630 | SplinePathGuideQTask_Register | SplinePathGuideQTask with 5 params including Startposition |
| 0x00529b70 | ProjectileLauncher_Register | ProjectileLauncher with 15 parameters and callbacks |
| 0x00512f30 | StatusMessage_Register | StatusMessage with 10+ parameters and display callbacks |
| 0x004f7300 | Train_Register | Train type with 10+ params including Acceleration, Displacement X/Z |
| 0x0054ce20 | SplinePathNodeQTask_Register | SplinePathNodeQTask with 7 parameters |
| 0x0054c930 | ViewportQTask_Register | ViewportQTask with parameter handlers and debug array |
| 0x00548700 | Task_SerializeOutput | Serializes task data to output buffer with \r\n line wrapping |
| 0x00548c00 | Task_GetParameter | Hash-based parameter lookup — (param % 0xfb) * 3 index, walks linked list |
| 0x00548d00 | Task_FreeParameters | Frees all parameters from task's parameter list |
| 0x0054a760 | SaveTask | Writes task data to backup file, iterates QTask list, calls serialization |
| 0x00549c90 | Level_GetQTaskRange | Walks QTask tree, calls GetLevelQTaskRange callback, checks address range |
| 0x0054c3b0 | BreakOnAllocID_Initialize | BreakOnAllocID debug feature — linked list, registers debug command |
| 0x0060acb0 | CubeModifier_Register | CubeModifier type with 3 parameters |

### ✅ DATA TYPES CREATED (Round 5)

| Type | Size | Address | Description |
|------|------|---------|-------------|
| WeaponType (enum) | 4 bytes | 0x0069e368 | 21 weapon types: SPECIAL, C4BOMB, PROXIMITYMINE, MEDIPACK, BINOCULAR, GRENADE, KNIFE, GLOCK, MAKAROV, UZI, M16, AK47, MP5, AWG, RPG, SNIPER, SHOTGUN, FLAMETHROWER, LASER, M203, MINIGUN |
| WeaponData (struct) | 64 bytes | 0x0069eb84 | 16 fields: next, count, name, sprite, icon, type, category, damage, ammo, fire rate, accuracy, range, recoil, weight |
| AINode (struct) | 512 bytes | 0x007b5500 | 23 fields: position, direction, up vectors, neighbor data, A* scores (g/h/f), cover type, graph ID |
| StateEntry (struct) | 32 bytes | 0x006b8558 | 8 fields: primary/secondary function pointers, param/action/mode/reserved flags |
| EntityStats (struct) | 60 bytes | N/A | 15 fields: string pointers for nHealth, nTime, nHits, nKills, etc. |

### ✅ GLOBAL DATA TYPES APPLIED

| Address | Name | Type | Plate Comment |
|---------|------|------|---------------|
| 0x0068e320 | g_pszEntityStatNames | char * | Entity/player stat string pointer table (0xD80 bytes, ~864 entries) |
| 0x0069eb84 | g_pWeaponDataArray | WeaponData * | Global weapon data array: 21 weapons, 8 categories, 27 parameter fields |
| 0x007b5500 | g_pAINodePool | AINode * | AI navigation graph node pool: 1024 nodes × 512 bytes each |
| 0x006b8558 | (StateEntry array) | StateEntry[14] | HumanPlayer state machine: 14 entries × 28 bytes = 392 bytes |

### ✅ FUNCTION TAGS ADDED

| Function | Tags |
|----------|------|
| PhysicsObj_Action (0x005c3ed0) | physics, collision |
| Sound_InitializeMiles (0x00542570) | audio, miles |
| InputSubsystem_RegisterTypes (0x004262c0) | input, directinput |
| MenuManager_Initialize (0x00433c50) | menu, ui |
| Terrain_MaterialLoad (0x005512e0) | terrain, level |

### PLATE COMMENTS ADDED ROUND 5 (6 detailed comments)
- **PhysicsObj_Action** — Core PhysicsObj handler with Move, UpdateOrientation, Hit callbacks
- **Sound_InitializeMiles** — Miles Sound System init: _AIL_startup, 3D provider selection, sample pool
- **InputSubsystem_RegisterTypes** — ~100+ input QTask types, movement bindings, keyboard devices
- **Terrain_MaterialLoad** — Loads .tim/.phm/.dmc/.csq terrain resources, allocates tile/material maps
- **MenuManager_Initialize** — Screen management, input handling, button callbacks, 0x98-byte params
- **HitZone_Init** — TASKTYPE_HITZONE with 14+ callbacks, 0x480c-byte structure

---

## Metrics

| Metric | Value |
|--------|-------|
| Total functions | 5,507 |
| Functions renamed | 880 |
| Unnamed (FUN_*) | 4,627 |
| Coverage | 16.0% |
| Plate comments | 100+ |
| Documentation files | 21 |
| Custom data types | 10+ (AINode 512B, Entity 3464B, EntityStats 60B, WeaponType enum, StateEntry, FuncInfo, HandlerType...) |
| Function tags | 105 definitions (weapon=79, player=29, state=10, scripting=11, qtask=9, ...) |
| Import table | 80+ from 11 DLLs |
| Export table | 38 entries |
| Memory segments | 7 |
| Memory size | ~136 MB |
| Formats decoded | 15+ |
| Audio functions | 52+ renamed |
| Network modes | 7 |
| QVM opcodes | 30+ |
| A* pathfinding | Documented (2000-node capacity) |
| MEF chunks | 13 types decoded |
| D3D API dispatch | 96 entries |
| Network packet types | 20+ identified |
| Global data typed | 4 globals with plate comments |
| Scripting functions | 25+ renamed (Opcode, QTask, LevelScript, Train, etc.) |

### DOCUMENTATION UPDATES (Priority 3)

All 5 subsystem documentation files updated with Round 5 renames:

| File | Updates |
|------|---------|
| `07-scripting/qvm-system.md` | +25 functions (Opcode, QTask, Console, Task_New, etc.), expanded Key Addresses (38 entries) |
| `08-resources/resource-formats.md` | New names (Model_LoadMEF, Terrain_MaterialLoad, Terrain_HeightmapLoad), updated pipeline |
| `09-audio/audio-system.md` | +15 functions (Sound_InitializeMiles, SoundGenerator, SoundDef, Game_InitSoundSystem) |
| `10-menus/menu-system.md` | +8 functions (MenuManager, MenuItem, PushButton, TagItem), +4 Input functions |
| `11-configuration/config-system.md` | Updated function names, cross-references to menu-system.md |

---

## File Structure

```
reverse-artifacts/
├── README.md
├── STATUS.md
├── INDEX.md
├── 01-executable/ (4 docs)
├── 02-classes/ (2 docs)
├── 03-rendering/ (1 doc)
├── 04-ai/ (1 doc)
├── 05-weapons/ (1 doc)
├── 06-level-system/ (1 doc)
├── 07-scripting/ (1 doc)
├── 08-resources/ (1 doc)
├── 09-audio/ (1 doc)
├── 10-menus/ (1 doc)
├── 11-configuration/ (1 doc)
├── 12-networking/ (1 doc)
├── 13-global-data/ (1 doc)
├── ghidra-scripts/ (TBD)
└── cross-reference/ (1 doc)
```
