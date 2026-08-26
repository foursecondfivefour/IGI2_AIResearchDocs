# Reverse Engineering Index — IGI 2: Covert Strike

Complete index of all documented systems and subsystems.

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
| [weapon-system.md](05-weapons/weapon-system.md) | Weapons, ammo, grenades, overlays | 0x0068ca94, 0x0069eb84 |

### Subsections:
- 11 weapon categories, 8 weapon types
- 27 weapon parameter fields
- Weapon handler table (12 functions)
- Human weapon system (6 functions)
- WeaponType system (11 functions)
- Grenade system (9 entries)
- 6 sniper overlay targets
- 14 body-part damage multipliers
- Human player weapon state (5 flags)

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

---

## Cross-References

- **AI ↔ Rendering:** AICameraInfo links to camera system
- **Weapons ↔ AI:** Grenade throw probability in AI config
- **Levels ↔ AI:** Graph navigation used by all AI types
- **Resources ↔ Rendering:** TEX→texture upload pipeline
- **Scripts ↔ AI:** QSC scripts define AI behavior trees
- **Weapons ↔ Levels:** Carry-over system between missions
