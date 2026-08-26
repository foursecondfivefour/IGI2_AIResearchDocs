# Phase 3.2: Weapon System — IGI 2: Covert Strike

**Status:** ✅ REVERSED  
**Functions Documented:** 25+  
**Confidence:** HIGH

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
