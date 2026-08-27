# Phase 4: QVM Scripting System — IGI 2: Covert Strike

**Status:** 🟡 PARTIAL — 65+ functions renamed  
**Functions Documented:** 65+ (40+ original + 25 Round 5)  
**Confidence:** HIGH  
**Verified:** 2026-08-27 via Ghidra MCP

### Round 6 Update (2026-08-27)
- [x] RunDelayed QTask — new global (DAT_07b54d4c)
- [x] CamTask — new global (DAT_006b022c)
- [x] GetInput — new global (DAT_07b524a8)
- [x] LevelInfo — new global (DAT_006aff04)
- [x] SimBoneDynCubeObj — new global (DAT_006b0268)
- [x] SerializeList, SerializeArray, SerializeStruct, SerializeQTaskRef — new serialization types

---

## QVM Interpreter (QVM_ExecuteBytecode @ 0x005699C0)

**Address:** 0x005699C0

### Bytecode Execution Loop (0x00569650)
- Reads opcode: `byte [EBX + ESI]`
- Dispatches via table: `(&DAT_084d8488)[opcode * 6]()`
- Increments ESI past instruction

### Opcode Handler (0x00565870)
- Switch statement returning byte-width for each opcode

### Jump Table
- Handler dispatch table at 0x00568B74
- 30 entries for opcodes 0x00-0x1D+
- Each entry = 4-byte pointer to handler

---

## Opcode Executor (Opcode_Execute @ 0x00569060)

**Address:** 0x00569060  
**Plate Comment:** Dispatches opcode execution, checks for \r or \x11 byte, calls handler with opcode type and string argument. Aborts on unhandled opcode with "Unhandled opcode."

## Opcode Parameter Count (Opcode_GetParameterCount @ 0x00565870)

**Address:** 0x00565870  
**Plate Comment:** Returns parameter count for each opcode type via switch on opcode value. Maps opcodes (1-0x1d) to expected arg counts (1-10). Aborts on unknown opcode with "Script: Internal Error: Unknown opcode: %02X."

## Script Bytecode Executor (Script_ExecuteBytecode @ 0x0056c2b0)

**Address:** 0x0056c2b0  
**Plate Comment:** Main bytecode interpreter loop — allocates VM frame, loads script, executes bytecode via dispatch table, handles commands (break, eval), manages memory pools. References "Unknown opcode: %s\n" at 0x006890a0.

---

## QTask System (QTask_Initialize @ 0x0040b840)

**Address:** 0x0040b840  
**Plate Comment:** Initializes QTask system — clears arrays, registers "QTask" type, creates "Create"/"ParentChanged" events, allocates QTask list with 0x1f80 entries, calls QTask_RegisterType.

## QTaskList Management (QTaskList_Add @ 0x0041f310)

**Address:** 0x0041f310  
**Plate Comment:** Adds a QTask to the QTaskList array — checks if list is full, appends to end, increments count. Aborts with "QTaskList is full" at 0x006a1ee8 if overflow.

## Task_New (Task_New @ 0x0043da60)

**Address:** 0x0043da60  
**Plate Comment:** Script symbol context setter — resolves "Task_New" symbol, loads script file, sets up execution context. Error: "Script_SetSymbolContext(): Symbol not found: %s" at 0x006a55b0.

## Task_New Execute (Task_New_Execute @ 0x00549640)

**Address:** 0x00549640  
**Purpose:** Executes Task_New script — resolves symbol, loads file, executes via QVM, returns result.

## Console Command Dispatcher (Console_ExecuteCommand @ 0x005cc910)

**Address:** 0x005cc910  
**Plate Comment:** Main console command dispatcher — parses command string, looks up handler in command table, executes or prints "Unknown command: '%s'" at 0x00685bb4. Lists "Available commands:" at 0x00685c08 for help.

## Level Init (Level_Initialize @ 0x0040e680)

**Address:** 0x0040e680  
**Purpose:** Initializes Level/Static/Dynamic/CameraBase types, calls QTask_RegisterType for each. References "Level" at 0x00684428.

## Task Serialization (Task_SerializeOutput @ 0x00548700)

**Address:** 0x00548700  
**Purpose:** Serializes task data to output buffer — iterates nodes, formats strings, handles line wrapping with \r\n.

## Task Parameter Access (Task_GetParameter @ 0x00548c00)

**Address:** 0x00548c00  
**Purpose:** Hash-based parameter lookup — uses (param % 0xfb) * 3 index, walks linked list, finds matching task ID.

## Task Free Parameters (Task_FreeParameters @ 0x00548d00)

**Address:** 0x00548d00  
**Purpose:** Frees all parameters from a task's parameter list — walks linked list, frees each entry.

## Save Task (SaveTask @ 0x0054a760)

**Address:** 0x0054a760  
**Purpose:** Writes task data to backup file, iterates QTask list, calls serialization, writes to original file. Formats "Task_New(%d, \"%s\", " at 0x0068a49c.

## Level Get QTask Range (Level_GetQTaskRange @ 0x00549c90)

**Address:** 0x00549c90  
**Purpose:** Walks QTask tree, calls GetLevelQTaskRange callback, checks if address falls within any task's range.

## BreakOnAllocID (BreakOnAllocID_Initialize @ 0x0054c3b0)

**Address:** 0x0054c3b0  
**Purpose:** Initializes BreakOnAllocID debug feature — sets up linked list, registers debug command. References "BreakOnAllocID:" at 0x0068a0a8.

---

## QTask Type Registrars (Round 5 — 12 new functions)

| Address | Name | Purpose |
|---------|------|---------|
| 0x00608950 | NoiseQTask_Register | Registers NoiseQTask type with 15 parameters and callback handlers |
| 0x006054c0 | SmoothQTask_Register | Registers SmoothQTask type with 10 parameters and callbacks |
| 0x00605ed0 | LevelScript_Register | Registers "LevelScript" type with "RunScript" command and 1 parameter |
| 0x0060a630 | SplinePathGuideQTask_Register | Registers SplinePathGuideQTask with 5 parameters including Startposition |
| 0x00529b70 | ProjectileLauncher_Register | Registers ProjectileLauncher type with 15 parameters and callbacks |
| 0x00512f30 | StatusMessage_Register | Registers StatusMessage type with 10+ parameters and display callbacks |
| 0x004f7300 | Train_Register | Registers Train type with 10+ parameters including Acceleration, Displacement X/Z |
| 0x0054ce20 | SplinePathNodeQTask_Register | Registers SplinePathNodeQTask with 7 parameters |
| 0x0054c930 | ViewportQTask_Register | Registers ViewportQTask with parameter handlers and debug array |
| 0x0060acb0 | CubeModifier_Register | Registers CubeModifier type with 3 parameters |
| 0x005c08e0 | EditBoneObj_Register | Registers EditBoneObj type with 11 parameters including Time factor |
| 0x00599f20 | MagicObjConfig_Initialize | Registers MagicObjConfig with TASKTYPE_MAGICOBJ, BONEMAGICOBJ, SHADOWVOLUME, HUMANSHADOW |

---

## QVM Opcodes (30+ Opcodes)

### Push/Pop (0x00-0x15)

| ID | Name | Bytes | Description |
|----|------|-------|-------------|
| 0x00 | NOP | 1 | No operation |
| 0x01 | PUSH | 5 | Push 32-bit integer |
| 0x02 | PUSHB | 2 | Push byte |
| 0x03 | PUSHW | 2 | Push word |
| 0x04 | PUSHF | 5 | Push float |
| 0x05 | PUSHA | 5 | Push address |
| 0x06 | PUSHS | 5 | Push string |
| 0x07 | PUSHA | 5 | Push attribute address |
| 0x08 | PUSHSI | 5 | Push string attribute |
| 0x09 | PUSHSIB | 2 | Push string attribute (byte) |
| 0x0A | PUSHSIW | 2 | Push string attribute (word) |
| 0x0B | PUSHII | 5 | Push qualifier |
| 0x0C | PUSHIIB | 2 | Push qualifier (byte) |
| 0x0D | PUSHIIW | 2 | Push qualifier (word) |
| 0x0E | PUSH0 | 1 | Push 0 |
| 0x0F | PUSH1 | 1 | Push 1 |
| 0x10 | PUSHM | 1 | Push -1 |
| 0x11 | PUSH | 5 | Push integer (alt) |
| 0x15 | POP | 1 | Pop value |

### Control Flow (0x16-0x1A)

| ID | Name | Bytes | Description |
|----|------|-------|-------------|
| 0x16 | RET | 1 | Return from function |
| 0x17 | BRA | 5 | Unconditional branch |
| 0x18 | BF | 5 | Branch if false |
| 0x19 | BT | 5 | Branch if true |
| 0x1A | JSR | 5 | Jump to subroutine |
| 0x1B | CALL | 5 | Call function |

### Arithmetic (0x1C-0x21)

| ID | Name | Description |
|----|------|-------------|
| 0x1C | ADD | Integer add |
| 0x1D | SUB | Integer subtract |
| (expr 5) | MUL | Multiply |
| (expr 6) | DIV | Divide |

### Bitwise

| ID | Name | Description |
|----|------|-------------|
| (expr 0xB) | SHL | Shift left |
| (expr 0xC) | SHR | Shift right |
| (expr 0xA) | AND | Bitwise AND |
| (expr 8) | OR | Bitwise OR |
| (expr 9) | XOR | Bitwise XOR |
| (expr 0xD) | NOT | Bitwise NOT |

### Logical

| ID | Name | Description |
|----|------|-------------|
| (expr 0x1B) | LAND | Logical AND |
| (expr 0x1C) | LOR | Logical OR |

### Comparison

| ID | Name | Description |
|----|------|-------------|
| (expr 0xE) | EQ | Equal |
| (expr 0xF) | NE | Not equal |
| (expr 0x10) | LT | Less than |
| (expr 0x12) | LE | Less or equal |
| (expr 0x11) | GT | Greater than |
| (expr 0x13) | GE | Greater or equal |
| (expr 0x14) | NOT0 | Not zero |

### Assignment/Unary

| ID | Name | Description |
|----|------|-------------|
| (expr 0x19) | ASSIGN | Assign |
| (expr 3) | PLUS | Unary plus |
| (expr 4) | MINUS | Unary minus |
| 0x1E | BLK | Block start |

---

## QVM Data Structures

### Frame/Execution Context (pointer in ESI)

| Offset | Field | Size | Description |
|--------|-------|------|-------------|
| 0x00 | PC | 4 | Program counter |
| 0x04 | stack_top | 4 | Stack pointer |
| 0x08 | stack_base | 4 | Stack base address |
| 0x0C | stack_bottom | 4 | Stack bottom |
| 0x10 | return_address | 4 | Saved return address |
| 0x14 | instruction_pointer | 4 | Current instruction offset |
| 0x18 | local_base | 4 | Local variable base |
| 0x1C | local_count | 4 | Local variable count |
| 0x20 | error_code | 4 | Runtime error (0=ok) |
| 0x24 | data_stack | 4 | Data stack pointer |
| 0x28 | local_variables | 4 | Local variables pointer |
| 0x2C | temp_int | 4 | Temporary integer |
| 0x34 | flags | 4 | Execution flags |
| 0x38 | frame_flags | 4 | Frame control flags |
| 0xD20 | execution_state | 4 | State tracker |

---

## QVM Compilation & Loading

### Compilation
**QscCompile** at 0x004B8410
- Compiles `.qsc` → `.qvm` bytecode
- Function node limit: 4096
- Argument node limit: 65536

### Loading
**QvmLoad** at 0x004B80B0
- Loads compiled `.qvm` files
- File paths include `*.qvm` filter at 0x006A556C

### QTask Registry
**QTaskHashTableSet** at 0x004BAAC0
- Registers task types by string name → integer ID
- 88+ known task types

---

## QTask Type System (88+ Types)

### Entity Types
| Type | Purpose |
|------|---------|
| TASKTYPE_HUMAN | Player AI |
| TASKTYPE_HUMANSOLDIER | Enemy AI |
| TASKTYPE_HELI | Helicopter |
| TASKTYPE_PLANE | Aircraft |
| TASKTYPE_MOVERIGIDOBJ | Physics |
| TASKTYPE_NOISEQTASK | AI noise behavior |
| TASKTYPE_SMOOTHQTASK | AI smoothing |

### Level Types
| Type | Purpose |
|------|---------|
| TASKTYPE_LEVELTIMER | Level timing |
| TASKTYPE_LEVELFLOW | Flow control |
| TASKTYPE_SPAWNAREA | Spawn area |
| TASKTYPE_SPAWNPOINT | Spawn point |
| TASKTYPE_C4BOMBAREA | C4 bomb task |
| TASKTYPE_SHADOWVOLUME | Shadow volume |
| TASKTYPE_SMOKE | Smoke effects |
| TASKTYPE_LADDER | Climbing |
| TASKTYPE_PROXIMITYMINE | Proximity mine |
| TASKTYPE_MISSILE | Weapon projectile |
| TASKTYPE_MEDIPACK | Powerup |

### Editor Types
| Type | Purpose |
|------|---------|
| TASKTYPE_EDITORMAGICOBJ | Editor tools |

---

## Standard Library Functions (AIFunctions)

| Function | Purpose |
|----------|---------|
| AIFunction_PassEventOnToSquad | Squad event propagation |
| AIFunction_SendResponse | AI response |
| AIFunction_GetAnimationToPlay | Animation selection |
| AIFunction_SetAnimationInterval | Animation timing |
| AIFunction_GetEventDistance | Event distance query |
| AIFunction_GetRandomValue | Random value |
| AIFunction_GetScriptRealValue | Read script float |
| AIFunction_GetScriptIntegerValue | Read script int |
| AIFunction_SetScriptRealValue | Write script float |
| AIFunction_SetScriptIntegerValue | Write script int |
| AIFunction_SetGunnerID | Set gunner reference |
| AIFunction_SetDeathAnimation | Death animation |
| AIFunction_SetInstantDeath | Kill flag |
| AIFunction_SetInvulnerability | Immortality flag |
| AIFunction_SetEventPriority | Event priority |
| AIFunction_LimitViewRange | Vision range |
| AIFunction_HasTarget | Target query |
| AIFunction_SetStandActionAnimation | Idle animation |
| AIFunction_UseIdleView | Idle view |
| AIFunction_UseCombatView | Combat view |
| AIFunction_SetSideKick | Companion AI |
| AIFunction_SetHearingRangeFactor | Hearing range |
| AIFunction_SetAnimSpeedFactor | Animation speed |
| AIFunction_SetInvestigateShoutFrequency | Shout frequency |
| AIFunction_DefaultHandler | Default event handler |

---

## QSC Script Files (23 Known)

| Path | Purpose |
|------|---------|
| LOCAL:common/ai/squaddefault.qsc | Default squad behavior |
| LOCAL:common/ai/default.qsc | Default AI settings |
| LOCAL:common/ai/settings.qsc | AI settings |
| LOCAL:humanplayer/humanplayer.qsc | Player config |
| LOCAL:weapons/ammo.qsc | Ammo definitions |
| LOCAL:material/material.qsc | Material definitions |
| LOCAL:menusystem/ingamemenu.qsc | Game menu script |
| LOCAL:menusystem/mainmenu.qsc | Main menu script |
| LOCAL:config.qsc | Global configuration |
| MISSION:AI/Squad_%d.qsc | Per-mission squad AI |
| MISSION:AI/%d.qsc | Per-mission AI |
| MISSION:lod.qsc | LOD configuration |
| SESSION:savegames/missions/comp%02d%02d.qsc | Saved mission state |
| SESSION:savegames/missions/co%02d%02d%02d.qsc | Co-op save |
| SESSION:config.qsc | Session config |

---

## Interpreter Error Codes

| Code | Error Message |
|------|--------------|
| 1 | Stack underflow |
| 2 | Stack overflow |
| 3 | Internal error (read: BUG!!!) |
| 5 | Illegal instruction |
| 6 | Illegal operand(s) for instruction |
| 7 | Unknown identifier |
| 8 | Identifier not a variable |
| 9 | Identifier not a function |
| 10 | No function on stack |
| 0xB | Syntax error |
| 0xD | Function argument overflow |
| 0xE | Datablock not present |
| 0xF | Index out of range |

---

## Key Addresses

| Address | Significance |
|---------|-------------|
| 0x005699C0 | QVM_ExecuteBytecode — Main interpreter loop |
| 0x00569650 | QVM_ExecuteLoop — Bytecode execution dispatch |
| 0x00565870 | Opcode_GetParameterCount — Opcode→arg count mapping |
| 0x00569060 | Opcode_Execute — Opcode dispatcher |
| 0x0056c2b0 | Script_ExecuteBytecode — Main bytecode interpreter |
| 0x00568B74 | Opcode Dispatch Jump Table |
| 0x00569360 | Opcode Handler Dispatch |
| 0x00689540 | QVM String Table (opcode names) |
| 0x005661C0 | Expression Evaluator (int) |
| 0x00566730 | Expression Evaluator (float) |
| 0x00565E40 | Expression Evaluator (str) |
| 0x00565D70 | Expression Evaluator (assign) |
| 0x0040b840 | QTask_Initialize — QTask system init |
| 0x0041f310 | QTaskList_Add — QTask list management |
| 0x0043da60 | Task_New — Symbol context setter |
| 0x00549640 | Task_New_Execute — Script execution |
| 0x005cc910 | Console_ExecuteCommand — Console command dispatcher |
| 0x0040e680 | Level_Initialize — Level type registration |
| 0x00548700 | Task_SerializeOutput — Task serialization |
| 0x00548c00 | Task_GetParameter — Hash-based param lookup |
| 0x00548d00 | Task_FreeParameters — Parameter cleanup |
| 0x0054a760 | SaveTask — Task save to backup |
| 0x00549c90 | Level_GetQTaskRange — QTask range query |
| 0x0054c3b0 | BreakOnAllocID_Initialize — Debug feature |
| 0x00608950 | NoiseQTask_Register |
| 0x006054c0 | SmoothQTask_Register |
| 0x00605ed0 | LevelScript_Register |
| 0x0060a630 | SplinePathGuideQTask_Register |
| 0x00529b70 | ProjectileLauncher_Register |
| 0x00512f30 | StatusMessage_Register |
| 0x004f7300 | Train_Register |
| 0x0054ce20 | SplinePathNodeQTask_Register |
| 0x0054c930 | ViewportQTask_Register |
| 0x0060acb0 | CubeModifier_Register |
| 0x005c08e0 | EditBoneObj_Register |
| 0x00599f20 | MagicObjConfig_Initialize |
| 0x004B8410 | QscCompile — QSC→QVM compiler |
| 0x004B80B0 | QvmLoad — QVM file loader |
| 0x004BAAC0 | QTaskHashTableSet — Task type registry |
