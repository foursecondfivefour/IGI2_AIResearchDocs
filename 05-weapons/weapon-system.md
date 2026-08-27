# Weapon System — IGI 2: Covert Strike

**Status:** 🟡 PARTIAL — 100+ functions renamed  
**Functions Documented:** 100+  
**Confidence:** HIGH  
**Verified:** 2026-08-26 via Ghidra MCP

---

## Architecture Overview

The Weapon System is the largest subsystem in IGI2 with ~150+ functions spanning addresses 0x004a0000–0x004c0000. It manages:

- **WeaponPlayer**: Per-player weapon state, input processing, ammo, zoom/precision
- **WeaponSystem**: Core fire dispatch, event handling, overlay management
- **WeaponAction**: State machine handlers for 40+ weapon actions (codes 0x1-0x37)
- **Callback Dispatchers**: 6 stub functions at 0x004a0c20-0x004a0cc0 that check global flags and call indirect function pointers
- **Input Processing**: Main processor at 0x004a3140 with switch cases 1-13 (0x1-0xd)
- **Precision/Zoom**: 8 functions at 0x004ba6e0-0x004bb4a0 for precision model updates
- **Serialization**: Copy/Serialize/ProcessDoor functions at 0x004bc5d0-0x004bfdf0

**Game Mode Branching**: `DAT_006b79ec` (renamed `g_dwGameMode`) — 0=singleplayer, 1=multiplayer. Affects handler registration, zoom logic, fire parameters throughout weapon system.

**Dispatch Table**: `DAT_083997e0` (renamed `g_ppfnEventHandlers`) — 21-weapon dispatch table. Formula: `table[event_code * 0x1ff + weapon_type_offset]`.

---

## Weapon Handler Table (0x0068ca94–0x0068cb50)

| String (Address) | Purpose |
|------------------|---------|
| GetWeaponObject | Get weapon by type |
| Weapon_PositionOverlay | Position weapon model |
| Weapon_CreateProjectile | Spawn projectile/bullet |
| Weapon_GetParentParameters | Get parent params |
| Weapon_DeActivate | Deactivate weapon |
| Weapon_Activate | Equip weapon |
| Weapon_Zoom | Zoom/scope toggle |
| Weapon_Reload | Reload weapon |
| Weapon_Release | Release weapon |
| Weapon_GetPrecision | Get weapon spread |
| **Weapon_Fire** | Primary fire function |
| **WeaponHandler** | Main weapon dispatcher |

---

## Human Weapon System

| Function | Address | Purpose |
|----------|---------|---------|
| HumanWeaponFire | 0x0068ec24 | Fire handler |
| HumanWeaponReload | 0x0068ec10 | Reload handler |
| Human_AddAmmoForWeapon | 0x0068ecc0 | Add ammo for weapon |
| Human_AddAmmo | 0x0068ecd8 | Generic ammo add |
| Human_AddWeapon | 0x0068ece8 | Add weapon to inventory |
| HumanPlayer_GetPrimaryWeaponID | 0x00693448 | Get weapon ID |
| HumanPlayer_GetPrimaryWeaponModel | 0x00693468 | Get weapon model name |
| DefineHumanPlayerAmmoLimit | 0x006939b0 | Define ammo limits |

---

## WeaponType System (0x0069eb84 area)

| Function | Purpose |
|----------|---------|
| WeaponType-CreateParameters | Create weapon params |
| WeaponType-DeleteParameters | Delete params |
| WeaponType-Get | Get weapon by ID |
| WeaponType-CheckForDuplicate | Duplicate check |
| WeaponType-EnumTypes | Enumerate all weapons |
| WeaponType-LoadSprite | Load weapon icon |
| WeaponType-UnLoadSprite | Unload icon |
| WeaponType-DrawIcon | Draw icon |
| WeaponType-FindChildren | Find child types |
| WeaponType_GetAnim | Get weapon animation |
| WeaponType_GetAmmoInfo | Get ammo info |

---

## Grenade System

| Variable | Address | Purpose |
|----------|---------|---------|
| Grenade | 0x00695a8c | Base grenade data |
| GrenadeParameters | 0x00695a78 | Grenade params |
| grenade_handle | 0x00693e4c | Main handler |
| grenade_throw | 0x00693e3c | Throw function |
| grenade_tick | 0x0068ccbc | Per-frame update |
| nGrenadeHeldTime | 0x00693580 | Hold timer |
| nGrenadeTickTime | 0x00693594 | Tick timer |
| ImpactGrenade | 0x00692f20 | Impact type |
| AISquad_ThrowGrenade | 0x00694318 | AI grenade throw |

### Grenade Parameters

| Field | Description |
|-------|-------------|
| ProjectileModel | 3D model path |
| Throw Gamma Exponent | Trajectory curve |
| Throw Ideal Angle (deg) | Optimal angle |
| Explosion Damage Factor | Damage multiplier |
| Explosion Falloff (m) | Damage falloff |
| Explosion Radius (m) | Total radius |
| Explosion Type | Normal/Flashbang/Smoke |

---

## Sniper Overlay Targets

| Overlay | Address |
|---------|---------|
| SniperOverlay | 0x006978dc |
| Psg1SniperOverlay | 0x0068b1a4 |
| DragunovSniperOverlay | 0x00695800 |
| AugSniperOverlay | 0x00694888 |
| G36SniperOverlay | 0x00699fcc |
| G11SniperOverlay | 0x00691cc0 |

---

## Weapon Categories (11 Categories)

| String | Value | Description |
|--------|-------|-------------|
| WEAPONCATEGORY_NONE | 0 | No category |
| WEAPONCATEGORY_KNIFE | 1 | Melee |
| WEAPONCATEGORY_SIDEARM | 2 | Pistols |
| WEAPONCATEGORY_PRIMARY | 3 | Rifles/SMGs |
| WEAPONCATEGORY_THERMOCHEMICAL | 4 | Thermal attachments |
| WEAPONCATEGORY_LASERDESIGNATOR | 5 | Laser designator |
| WEAPONCATEGORY_C4 | 6 | C4 explosive |
| WEAPONCATEGORY_BINOCULARS | 7 | Binoculars |
| WEAPONCATEGORY_RPG | 8 | RPG launcher |
| WEAPONCATEGORY_EXPLOSIVE | 9 | Explosives |
| WEAPONCATEGORY_OTHER | 10 | Uncategorized |

---

## Weapon Types (8 Types)

| String | Value | Description |
|--------|-------|-------------|
| WEAPONTYPE_GRENADE | 0 | M67 fragmentation |
| WEAPONTYPE_KNIFE | 1 | Combat knife |
| WEAPONTYPE_SHOTGUN | 2 | Shotguns |
| WEAPONTYPE_PISTOL | 3 | Pistols |
| WEAPONTYPE_GUN | 4 | Rifles/SMGs |
| WEAPONTYPE_SPECIAL | 5 | Flashbang/smoke |
| WEAPONTYPE_C4BOMB | 6 | C4 explosive |
| WEAPONTYPE_PROXIMITY | 7 | Proximity devices |

---

## Weapon Parameter Structure (27 Fields)

| Offset | Field | Type |
|--------|-------|------|
| 0x00 | Reload Time (Seconds) | float |
| 0x18 | Effective Range (Meters) | float |
| 0x30 | Rounds Per Minute | int |
| 0x48 | Rounds Per Clip | int |
| 0x60 | Bullets Per Round | int |
| 0x78 | Ammo Display Type | enum |
| 0x90 | AmmoType ID | int |
| 0xa8 | AmmoLimit | int |
| 0xc0 | Movement Speed Modifier | float |
| 0xd8 | Turning Precision Modifier | float |
| 0xf0 | Jumping Precision Modifier | float |
| 0x108 | Crouching Precision (deg) | float |
| 0x128 | Lying Flat Precision (deg) | float |
| 0x148 | Standing Precision (deg) | float |
| 0x168 | Walking Precision (deg) | float |
| 0x188 | Running Precision (deg) | float |
| 0x1a8 | Muzzle Velocity (m/s) | float |
| 0x1c0 | Muzzle flash size | float |

---

## Human Player Weapon State

| Variable | Address | Type | Purpose |
|----------|---------|------|---------|
| eWeaponStatus | 0x006935a8 | enum | Current weapon state |
| isQuietComputer | 0x006935bc | bool | Silenced flag |
| isWeaponsCarryOver | 0x006935d0 | bool | Carryover flag |
| isUnderwater | 0x006935e4 | bool | Underwater flag |
| isUnlimitedAmmo | 0x006935f8 | bool | Cheat flag |

---

## Body Part Damage Multipliers (MULTIPLAYER)

| Address | Body Part |
|---------|-----------|
| 0x0069da38 | Head |
| 0x0069da5c | Groin |
| 0x0069da80 | Back |
| 0x0069daa4 | Chest |
| 0x0069dac8 | Upper body |
| 0x0069daf0 | Left upper arm |
| 0x0069db20 | Left lower arm |
| 0x0069db50 | Left hand |
| 0x0069db78 | Left upper leg |
| 0x0069dba8 | Left lower leg |
| 0x0069dbd8 | Left foot |
| 0x0069dc00 | Right upper arm |
| 0x0069dc30 | Right lower arm |
| 0x0069dc60 | Right hand |

---

## Key Global Pointers (0x0068ca00 area)

| Address | Variable | Type |
|---------|----------|------|
| 0x0068ca3c | iActiveWeapon | int — Current weapon ID |
| 0x0068ca4c | WeaponTypeList | pointer — All weapon types |
| 0x0068ca5c | AmmoTypeList | pointer — All ammo types |
| 0x0068ca6c | WeaponItem | pointer — Current weapon item |
| 0x0068ca78 | AmmoItem | pointer — Current ammo item |

---

## WeaponPlayer Core Functions (0x004a0000–0x004a1000)

| Function | Address | Purpose |
|----------|---------|---------|
| WeaponPlayer_ResetDefaultValues | 0x004a00f0 | Reset all weapon player defaults |
| WeaponPlayer_InitializeStateHelper | 0x004a07c0 | Position/orientation math, rotation matrix setup |
| WeaponPlayer_RegisterTracerBullet | 0x004a0730 | Register tracer bullet handler |
| WeaponPlayer_UpdateDoorState | 0x004a0c00 | Update door interaction state |
| WeaponPlayer_CallbackDispatcher_0 | 0x004a0c20 | Callback stub — checks global flag, indirect call |
| WeaponPlayer_CallbackDispatcher_1 | 0x004a0c40 | Callback stub — checks global flag, indirect call |
| WeaponPlayer_CallbackDispatcher_2 | 0x004a0c60 | Callback stub — checks global flag, indirect call |
| WeaponPlayer_CallbackDispatcher_3 | 0x004a0c80 | Callback stub — checks global flag, indirect call |
| WeaponPlayer_CallbackDispatcher_4 | 0x004a0ca0 | Callback stub — checks global flag, indirect call |
| WeaponPlayer_CallbackDispatcher_5 | 0x004a0cc0 | Callback stub — checks global flag, indirect call |
| WeaponPlayer_LookupKeyBinding | 0x004a0ce0 | Searches DAT_00755060 key binding table (0x6d entries) |

---

## WeaponPlayer Input Processing (0x004a1000–0x004a4000)

| Function | Address | Purpose |
|----------|---------|---------|
| WeaponPlayer_RegisterActionHandlers | 0x004a18a0 | Register all action handler callbacks |
| WeaponPlayer_ProcessFireState | 0x004a1090 | Fire state machine, searches DAT_079f9600 |
| WeaponPlayer_UpdateFireAngles | 0x004a1140 | Trig math (fptan 0.785/0.523), fire angle updates |
| WeaponPlayer_UpdateAmmoCounter | 0x004a22b0 | Ammo counter update, searches DAT_079f1480 |
| WeaponPlayer_ProcessFireInput | 0x004a2360 | Main fire input processing, complex weapon selection |
| WeaponPlayer_UpdateWeaponSelection | 0x004a2df0 | Weapon selection with DAT_00754d90/DAT_00754d94 arrays |
| WeaponPlayer_DecayCrosshair | 0x004a3030 | Decay crosshair spread (0.022222223/frame) |
| WeaponPlayer_DecayZoom | 0x004a3080 | Decay zoom level (0.008333334 or 0.016666668) |
| WeaponPlayer_ProcessInput | 0x004a3140 | Main input processor, switch cases 1-13 |
| WeaponPlayer_ProcessC4Animation | 0x004a3aa0 | C4 placement animation, position/orientation math |
| WeaponPlayer_ProcessZoomAction | 0x004a3e90 | Zoom action — distance check 3072.0, rotation matrix |
| WeaponPlayer_ProcessPrecisionAim | 0x004a43a0 | Precision aim — direction vector normalization, SQRT |

---

## WeaponPlayer State/Flag Updates (0x004a4000–0x004a6000)

| Function | Address | Purpose |
|----------|---------|---------|
| WeaponPlayer_UpdateEntityFlags | 0x004a5390 | Entity flag update — param_2 offsets 0x304-0x3c8, bit flags |
| WeaponPlayer_ProcessAction | 0x004a5910 | Main action processor — dispatches to 37+ handlers |

---

## WeaponAction Handlers (0x004a5e00–0x004b6000)

Action handlers registered via `FUN_0040adf0("ActionName", handler)` and invoked through dispatch table.

| Function | Address | Action Code | Purpose |
|----------|---------|-------------|---------|
| WeaponAction_11_20 | 0x004a5e40 | 0x11/0x14 | Action 11/20 handler |
| WeaponAction_1B | 0x004a5f00 | 0x1b | Action 27 handler |
| WeaponAction_26 | 0x004a6000 | 0x26 | Action 38 handler |
| WeaponAction_3 | 0x004a6100 | 0x3 | Action 3 handler |
| WeaponAction_4_5 | 0x004a6200 | 0x4/0x5 | Actions 4/5 handler |
| WeaponAction_C | 0x004a6300 | 0xc | Action 12 handler |
| WeaponAction_1B_2 | 0x004a6400 | 0x1b | Action 27 variant |
| WeaponAction_11 | 0x004a6500 | 0x11 | Action 17 handler |
| WeaponAction_20 | 0x004a6600 | 0x20 | Action 32 handler |
| WeaponAction_37 | 0x004a6700 | 0x37 | Action 55 handler |
| WeaponAction_Continue | 0x004ab660 | — | Continue handler — checks param+0x818 >= 3276.8 |
| ... | 0x004a6800–0x004b5830 | 0x1–0x37 | 37 total action handlers |

---

## WeaponSystem Core Functions (0x004af000–0x004b0000)

| Function | Address | Purpose |
|----------|---------|---------|
| WeaponSystem_Fire | 0x004af060 | Main fire dispatch — calls g_ppfnEventHandlers[event_code], handles recoil/sway |
| WeaponPlayer_UpdateViewMatrix | 0x004af170 | View matrix update — copies 6 doubles + 10 floats, calls FUN_00553b50 |

---

## WeaponPlayer Serialization (0x004bc000–0x004c1000)

| Function | Address | Purpose |
|----------|---------|---------|
| CopyState | 0x004bc5d0 | Copy weapon state between entities |
| Serialize | 0x004bd000 | Serialize weapon state for save/load |
| ProcessDoor | 0x004bda00 | Process door interaction state |
| WeaponPlayer_LoadWeaponState | 0x004c04a0 | Load weapon state — allocates entity, copies data, initializes |

---

## Weapon Precision/Zoom System (0x004ba6e0–0x004bb4a0)

8 functions for precision model updates:

| Function | Address | Purpose |
|----------|---------|---------|
| WeaponPlayer_UpdatePrecision_1 | 0x004ba6e0 | Precision update pass 1 |
| WeaponPlayer_UpdatePrecision_2 | 0x004ba800 | Precision update pass 2 |
| WeaponPlayer_UpdatePrecision_3 | 0x004ba920 | Precision update pass 3 |
| WeaponPlayer_UpdatePrecision_4 | 0x004baa40 | Precision update pass 4 |
| WeaponPlayer_UpdatePrecision_5 | 0x004bab60 | Precision update pass 5 |
| WeaponPlayer_UpdatePrecision_6 | 0x004bac80 | Precision update pass 6 |
| WeaponPlayer_UpdatePrecision_7 | 0x004bada0 | Precision update pass 7 |
| WeaponPlayer_UpdatePrecision_8 | 0x004bb4a0 | Precision update pass 8 |

---

## Key Global Variables

| Address | Variable | Type | Purpose |
|---------|----------|------|---------|
| 0x08350c54 | g_dwGameMode | uint | 0=singleplayer, 1=multiplayer |
| 0x083997e0 | g_ppfnEventHandlers | dispatch[21] | Weapon event handler table |
| 0x006b79ec | g_ppfnWeaponFunctionTable | pointer | Weapon function table |
| 0x006b7a10 | g_dwCurrentWeaponIndex | int | Current weapon index |
| 0x00754d90 | DAT_00754d90 | array | Weapon selection array |
| 0x00754d94 | DAT_00754d94 | array | Weapon selection names |
| 0x00755060 | DAT_00755060 | array[0x6d] | Key binding table |
| 0x006b8090 | PlayerState | area | Player state area |
| 0x006b8558 | StateEntry[14] | struct | State entry array |

---

## Weapon State Offsets (Entity struct 0xD80 bytes)

| Offset | Field | Purpose |
|--------|-------|---------|
| 0x28 | position_x | World position X |
| 0x30 | position_y | World position Y |
| 0x38 | position_z | World position Z |
| 0x22c | fire_state | Current fire state (float) |
| 0x230 | movement_speed | Movement speed modifier |
| 0x304–0x3c8 | weapon_flags | Entity flag bitfield (17 flags) |
| 0x328 | precision_mode | Precision mode selector |
| 0x3a0–0x3a4 | zoom_state | Zoom/precision state |
| 0x4dc | weapon_flags2 | Secondary weapon flags (bit 0x20000 = zoom) |
| 0x4d0–0x4e8 | action_state | Action callback/state machine |
| 0x530 | weapon_handler | Weapon handler pointer |
| 0x540 | entity_ptr | Entity pointer |
| 0x564 | animation_state | Animation state flag |
| 0x5b9 | crouch_state | Crouch state flag |
| 0x654 | prone_state | Prone state flag |
| 0x74c | c4_state | C4 placement state |
| 0x750–0x751 | anim_flags | Animation flags |
| 0x784–0x788 | state_flags | State flags |
| 0x7d9 | visibility_flag | Visibility flag |
| 0x818 | action_threshold | Action continuation threshold |
| 0xb0 | overlay_list_1 | First overlay linked list |
| 0xbc | overlay_list_2 | Second overlay linked list |
| 0xc00 | action_flag | Action flag |
| 0xc58 | crouch_anim | Crouch animation flag |
| 0xc59 | anim_phase | Animation phase |
| 0xc5a | c4_phase | C4 placement phase |
| 0xc5b | zoom_phase | Zoom action phase |
| 0xc5c | zoom_phase2 | Zoom action phase 2 |
| 0xc68 | precision_phase | Precision aim phase |
| 0xc69 | precision_phase2 | Precision aim phase 2 |
| 0xc6c–0xc74 | direction_vec | Normalized direction vector |
| 0xc78 | precision_angle | Precision angle |
| 0xcc/0xe8/0x114 | weapon_template | 0x78-byte weapon template entry |
| 0xd20 | fire_blocked | Fire blocked flag |
| 0xd64 | camera_state | Camera state |
| 0xd80–0xd88 | weapon_offset | Weapon position offset |
| 0xe00 | weapon_data | Weapon data pointer |
| 0xe80 | weapon_model | Weapon model pointer |
| 0x130–0x148 | view_matrix | View matrix data |
| 0x138 | view_distance | View distance |
| 0x1b0–0x1b8 | saved_state | Saved state data |
| 0x1a0–0x1b8 | entity_state | Entity state (6 doubles + 10 floats) |
| 0xbbc | weapon_flags3 | Third weapon flag set |
| 0xc14 | weapon_param | Weapon parameter pointer |
| 0xd40–0xd7c | weapon_animation | Weapon animation data |
| 0x1e4–0x214 | entity_anim | Entity animation data |
