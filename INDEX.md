# Reverse Engineering Index — IGI 2: Covert Strike

Complete index of all documented systems and subsystems.

**Coverage:** 872 / 5,507 functions renamed (15.9%)  
**Last Verified:** 2026-08-27 via Ghidra MCP

### Round 6 Updates (2026-08-27)
- WinMain_CRTStartup (0x00404280) — renamed, documented
- System_InitializeAll (0x005D6040) — renamed, documented
- 6 new global data areas discovered
- 90+ subsystem init pipeline documented
- vftable, config callbacks, string references catalogued

### Round 7 Updates (2026-08-27)
- 14 new function renames across QTask, FileSys, GameEvent systems
- QTask architecture documented: 0x34-byte types, 0xd slots, 0x200-byte state arrays
- File system architecture documented: 4 device types, 0x90-byte handler table
- Game event system documented: 0xc-byte entries, 0x20 max events
- Pooled allocator FUN_0055b820 identified as game code

### Round 8 Updates (2026-08-27)
- 14 new function renames across DList, FatalError, Network, Render, Event systems
- Doubly linked list documented: 852 xrefs = most used data structure
- Fatal error system: callback dispatch with 256-byte buffer
- Server logging: timestamped, traffic suppression after 0x1e messages
- Resource manager: GLOB:/ and LOCA:/ paths, linked list at DAT_0070b304
- Network system: 8180-byte packet buffer, "Guaranteed" delivery
- D3D error converter: 15+ D3DERR_* error strings
- Performance counter: QueryPerformanceCounter wrapper with ms conversion
- Event dispatcher: indirect callback dispatch via DAT_083997e0 table

### Round 9 Updates (2026-08-27)
- 13 new function renames across Memory, Event, String, Config, Render systems
- Memory validation: power-of-2 alignment check, bounds validation
- Memory pool allocator: 0xc-byte header, callback table filling
- Memory deallocation: 0xcdcdcdcd MSVC debug fill pattern, DList tracking
- Event dispatcher: linked list traversal, callback lookup via type*0x1ff
- String parser: 2052-byte buffer, parse/convert pipeline
- Config system: case-insensitive __stricmp search, 0x44-byte entries
- Render object creation: 13-field initialization, DList tracking
- Render task cleanup: callback dispatch, DList reinsertion
- QTaskList array-based variants: Add/Remove with overflow checks

### Round 10 Updates (2026-08-27)
- 7 new function renames: Script VM, Symbol Table, Memory Pool systems
- **Script VM** discovered: custom bytecode interpreter with 30+ opcodes
- ScriptVM_AddParameter (1754 xrefs): 0x14-byte parameter entries
- ScriptVM_ReadParameter (518 xrefs): 3 read modes (string ptr/raw/strncpy)
- ScriptVM_EvaluateExpression (303 xrefs): int expression evaluator
  - Arithmetic: add/sub/mul/div
  - Bitwise: OR/XOR/AND/shl/shr/not
  - Comparison: ==,!=,<,<=,>,>=,!=0
  - Logic: short-circuit OR/AND, ternary ?:
  - Assignment: with const/syntax error checks
- ScriptVM_EvaluateExpressionFloat (229 xrefs): float10 (extended precision) variant
- **Symbol Table** discovered: runtime symbol registry with callback dispatch
- SymbolTable_Add (1101 xrefs): string name + 6 params + DAT_07ab6c10 callbacks
- SymbolTable_Remove (666 xrefs): array remove + memmove + DList + 0xcdcdcdcd fill
- **Memory Pool** discovered: reference-counted pool allocator
- MemoryPool_Free (214 xrefs): ref count decrement or full free with debug fill

### Round 11 Updates (2026-08-27)
- 4 new function renames: Memory allocator, DList context, Script VM string, Pattern matching
- **Mem_Alloc** (667 xrefs): custom allocator with alignment, 0xabababab debug fill, DList tracking, allocation failure handler
- **DList_RemoveFromContext** (89 xrefs): context-specific DList removal with prev/next pointer updates
- **ScriptVM_EvaluateToString** (97 xrefs): string conversion for script expressions, concat support, 2048-byte buffer
- **PatternMatcher_Match** (65 xrefs): wildcard pattern matching with $, ., ?, *, [], ^, {}, () syntax

### Round 12 Updates (2026-08-27)
- 4 new function renames: Script interpreter core
- **ScriptInterpreter_Execute** (main loop): tokenization, .isExists, "this." calls, // comments, 0x100 depth stack, 0x10 frames
- **ScriptVM_DispatchCallback**: callback lookup via DAT_083997e0 + DAT_006b6724 * 0x1ff, 64-byte result
- **ScriptContext_Cleanup**: frees 3 memory blocks + 0x13 frames, DList + 0xcdcdcdcd + _free
- **ScriptVM_EvaluateExpressionEntry**: expression evaluator entry, calls FUN_005494f0 + ScriptInterpreter_Execute + ScriptVM_EvaluateExpressionFloat


---

> **Coverage Notice:** This project represents partial reverse engineering. Of 5,507 total functions, 830 have been renamed and documented (15.0%). The remaining 4,677 functions are still labeled as `FUN_*` in Ghidra. All documented content is accurate and verified — only the coverage percentage is partial.

---

## 01. Executable & PE Structure

| Document | Description | Key Addresses |
|----------|-------------|---------------|
| [functions.md](01-executable/functions.md) | Entry point, CRT, WinMain, module init | 0x0065d36d, 0x00404280 |
| [imports.md](01-executable/imports.md) | All 80+ imports across 11 DLLs | KERNEL32, MSS32, WININET |
| [exports.md](01-executable/exports.md) | 38 exported C++ class methods | Ordinals 1-18 |
| [pe-structure.md](01-executable/pe-structure.md) | Memory layout, sections, resources | 7 segments, 136MB |

---

## 02. Classes & Data Structures

| Document | Description | Key Addresses |
|----------|-------------|---------------|
| [classes.md](02-classes/classes.md) | HumanPlayer, HumanSoldier state machines | 0x006b8090+ |
| [structs.md](02-classes/structs.md) | Estimated struct layouts | TBD |

---

## 03. Rendering System

| Document | Description | Key Addresses |
|----------|-------------|---------------|
| [rendering-system.md](03-rendering/rendering-system.md) | DirectX pipeline, textures, models | 0x00446360, 0x6887f0 |

### Subsections:
- DirectX initialization (DirectDraw 7 + D3D8)
- Texture system (allocate, download, refresh)
- MEF model loading with 5 LOD levels
- Camera system with 4 filter types
- Render state error tracking

---

## 04. AI System

| Document | Description | Key Addresses |
|----------|-------------|---------------|
| [ai-system.md](04-ai/ai-system.md) | Full AI architecture, states, events, actions | 0x00462c80, 0x0069424c |

### Subsections:
- AI entity types (HumanSoldier, Human, HumanShadow, Bodyguard)
- Squad system (18 squad functions)
- 10 AI states (Patrol → Combat → Idle)
- 26+ AI events (detection, alert, combat)
- 7+ AI actions (patrol, cover, combat)
- 30+ AI config API functions
- 13 patrol path commands
- AI tuning tables

---

## 05. Weapon System

| Document | Description | Key Addresses |
|----------|-------------|---------------|
| [weapon-system.md](05-weapons/weapon-system.md) | Full weapon architecture — Player, System, Actions, Input, Precision, Serialization | 0x004a0000–0x004c0000, 0x0068ca94, 0x0069eb84 |

### Subsections:
- **WeaponPlayer Core** (11 functions) — Reset, Init, CallbackDispatchers, KeyBinding
- **WeaponPlayer Input** (12 functions) — FireState, FireAngles, AmmoCounter, FireInput, WeaponSelection, DecayCrosshair, DecayZoom, ProcessInput, C4Animation, ZoomAction, PrecisionAim
- **WeaponPlayer State/Flags** (2 functions) — EntityFlags, ProcessAction
- **WeaponAction Handlers** (37+ handlers) — Codes 0x1–0x37, Continue handler
- **WeaponSystem Core** (2 functions) — Fire dispatch, ViewMatrix update
- **WeaponPlayer Serialization** (4 functions) — CopyState, Serialize, ProcessDoor, LoadWeaponState
- **Precision/Zoom System** (8 functions) — Precision update passes 1–8
- **WeaponType System** (11 functions) — Create/Delete/Get/Enum/LoadSprite/DrawIcon
- **Grenade System** (9 entries) — Throw, tick, impact, AI throw
- **11 weapon categories, 8 weapon types**
- **27 weapon parameter fields**
- **14 body-part damage multipliers**
- **Key global variables** (g_dwGameMode, g_ppfnEventHandlers, etc.)
- **Entity state offsets** (0xD80-byte entity struct, 50+ fields)

---

## 06. Level & Mission System

| Document | Description | Key Addresses |
|----------|-------------|---------------|
| [level-system.md](06-level-system/level-system.md) | Level loading, objectives, spawns, graphs | 0x00407420, 0x005b1ad0 |

### Subsections:
- 28-step level initialization pipeline
- 6 resource path prefixes
- QTask/QVM scripting (4096 func nodes)
- Objective system (8 objectives, 672 bytes)
- Trigger/volume system (12+ triggers)
- Spawn areas with respawn timers
- Graph navigation (1024 nodes, 12 properties)
- Level flow system (5 event types)

---

## 07. Scripting Engine

- TBD — QVM bytecode interpreter, opcodes, standard library

## 08. Resource Formats

- TBD — ILFF, LOOP, BIT, CTR, MEF, TEX, LMP, HMP formats

## 09. Audio System

- TBD — Miles Sound System (MSS32), 37 AIL functions, 3D audio

## 10. Menu System

- TBD — Windows GUI registration, in-game menus, HUD

## 11. Configuration

- TBD — 13 config files, save system, registry usage

## 12. Networking

- TBD — WinInet HTTP, WinSock, DirectPlay multiplayer

## 13. Global Data

- 19 data areas documented (13 original + 6 new Round 6)
- vftable, config callbacks, QTask globals, string references

## 14. Reverse Engineering Notes

- [reverse-engineering-notes.md](14-reverse-engineering-notes/reverse-engineering-notes.md)
- Master initialization pipeline (95 steps)
- Code gaps analysis (2171 gaps, largest 1267 bytes)
- switchD catalog (40+ functions)
- AI type strings
- Telnet protocol support
- Coverage metrics

---

## Cross-References

- **AI ↔ Rendering:** AICameraInfo links to camera system
- **Weapons ↔ AI:** Grenade throw probability in AI config
- **Levels ↔ AI:** Graph navigation used by all AI types
- **Resources ↔ Rendering:** TEX→texture upload pipeline
- **Scripts ↔ AI:** QSC scripts define AI behavior trees
- **Weapons ↔ Levels:** Carry-over system between missions
