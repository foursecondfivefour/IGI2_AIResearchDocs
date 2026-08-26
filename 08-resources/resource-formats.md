# Phase 5: Resource File Formats — IGI 2: Covert Strike

**Status:** ✅ REVERSED + ROUND 5 ENHANCED  
**Formats Documented:** 15+  
**Functions Documented:** 50+ (25+ Round 5 additions)  
**Confidence:** HIGH

---

## Archive Format: ILFF (Innerloop Library File Format)

**Signature:** `"ILFF|P"` (0x494C46467C50) at 0x006B15E8

### Directory Structure:
```
Bytes 0-7:   Signature "ILFF|P" (8 bytes)
Bytes 8-11:  Entry count (DWORD)
Per entry:
  0-3:       Chunk type ID (4 ASCII chars)
  4-7:       Chunk size (DWORD)
  8-11:      Offset from data section start (DWORD)
  12-15:     Data size (DWORD)
  16-19:     File size (DWORD)
  20-23:     Flags (DWORD)
```

### Found Chunk Types:
- `FONT` / `FNT` — Font resources
- `ANMF` — Animation frames

---

## File Path Prefixes (5 VIRTUAL DIRECTORIES)

| Prefix | Purpose | Example |
|--------|---------|---------|
| LOCAL: | Base game files | LOCAL:common/textures/%s.tex |
| MISSION: | Mission-specific | MISSION:graphs/graph%d.dat |
| LANGUAGE: | Localization | LANGUAGE:objectives.res |
| COMPUTER: | In-game UI | COMPUTER:hud_1.spr |
| SESSION: | Save games | SESSION:savegames/%d-%d-%d-%d.igi2save |

---

## File Format Handlers

### MEF — 3D Model Files
**Path:** `LOCAL:models/%s.mef`  
**Loader:** FUN_004f7ad0 (full), FUN_004f8230 (simplified)

**MEF Chunk Format (FOURCC):**
| Chunk ID | Name | Description |
|----------|------|-------------|
| SMES | Mesh | Core mesh data (0x1C bytes) |
| SAFC | Faces | Face index data |
| SVTX | Vertices | Vertex positions (0x0C each) |
| EDGE | Edges | Edge connectivity |
| HSEM | Mesh | Mesh sub-structure |
| IERH | Hierarchy | Bone hierarchy/animation |
| MANB | Bone | Bone transform data |
| HPRM | — | Animation data |
| WLOG | Glow | Glow/lighting effects |
| ATTA | Attachments | Object attachments |
| XVTR/XVTP/XVTC | Texture | Texture vertex data |

**Version:** First float must be 0.16 (version 1.16)

---

### TEX — Texture Files
**Path:** `LOCAL:textures/%s.tex`  
**Loader:** FUN_004f8790

**Chunks:** TEXF (texture data), PALF (palette)  
**Filtering:** None, Point, Bilinear, Trilinear, Anisotropic  
**Format:** Direct3D textures, mipmaps (power-of-2 enforced)  
**TGA support:** TGA_Load(), TGA_Write()  
**Alpha blending:** Supported (isLockAlpha, DeltaAlpha)

---

### BSP — Forest/BSP Map Files
**Paths:**
- `MISSION:forest%d.dat`
- `MISSION:forest_%d.dat`

**Compiler:** `"Forest BSP Compiler (c)2002 Innerloop Studios, Version: 2.1, Author: Alexej Kryazhev"`  
**Renderer:** DrawForest at 0x687784  
**Storage:** BspTree, BspGenerationList at 0x6848E8  
**Density:** `"Density of trees [1/m^2]"`

---

### LMP — Lightmap Files
**Path:** `LOCAL:missions/location%d/level%d/lightmaps/%s`  
**Loader:** FUN_0053EB70

**Binary Format:**
```
Header (0x3C bytes):
  [0x00] Magic: 0x3DF5C28F or 0x3E23D70A
  [0x04] Version (float)
  [0x08] Lightmap count (DWORD)
  [0x0C] Width
  [0x10] Height
Per lightmap (0x34 bytes):
  [0x00] Width (DWORD)
  [0x04] Height (DWORD)
  [0x08] Pixel data pointer
  [0x0C] UV offset X, Y
  [0x14] UV scale, transform
  [0x20] Data size
  [0x24-0x30] Transform matrix
```

**Offline lightmaps:** `%s/%s_%05d.olm` format

---

### FNT — Font Files
**Paths:**
- `LOCAL:common/fonts/font1.fnt` through `font4.fnt`
- `LOCAL:common/fonts/fontmp.fnt` (multiplayer)
- `LOCAL:debug.fnt`

**Resources:** fonts_hi.res, fonts_lo.res, fonts_med.res

---

### SPR — Sprite Files
**Paths:**
- `LOCAL:common/sprites/`
- `LOCAL:common/sprites/battery.spr`
- `COMPUTER:compass.spr`, `COMPUTER:hud_1.spr`, etc. (60+ HUD sprites)

**Types:** Sprite3D, particle system sprites, UI sprites

---

### IFF — Animation Files
**Path:** `LOCAL:anims/%s.iff`  
**Loader:** FUN_004F8930

**System:**
- AnimBlender at 0x6848C8
- AnimController at 0x89B5C
- Predefined states: Idle, Walk, Run, Fire, Reload, Death, Crawl

**Animation types:** ANIMTYPE_SOLDIER, ANIMTYPE_PRIBOY  
**Body parts:** 1st Person, Upper body, Main animation

---

### RES — Resource Files

| File | Path | Purpose |
|------|------|---------|
| objectives.res | LANGUAGE:objectives.res | Mission objectives |
| computer.res | LANGUAGE:computer.res | In-game computer |
| menusystem.res | LANGUAGE:menusystem.res | Menu system |
| messages.res | LANGUAGE:messages.res | System messages |
| weapons.res | LOCAL:weapons/weapons.res | Weapon definitions |
| missionsprites.res | LOCAL:menusystem/missionsprites.res | Mission sprites |
| loadingscreen.res | LOCAL:menusystem/loadingscreen.res | Loading screen |
| sounds.res | LOCAL:menusystem/sound/sounds.res | Sound definitions |
| heightmaps.res | MISSION:heightmaps/heightmaps.res | Heightmaps |

---

### DAT — Data Files

| File | Purpose |
|------|---------|
| MISSION:forest%d.dat | Forest BSP data |
| MISSION:graphs/graph%d.dat | AI navigation graph |
| MISSION:graphs/graphcover%d.dat | AI graph cover data |

---

### QSC — Quest Script Files

| File | Purpose |
|------|---------|
| LOCAL:config.qsc | Global config |
| LOCAL:common/ai/*.qsc | AI behavior scripts |
| LOCAL:humanplayer/humanplayer.qsc | Player config |
| LOCAL:weapons/ammo.qsc | Ammo definitions |
| LOCAL:menusystem/*.qsc | Menu scripts |
| MISSION:AI/*.qsc | Per-mission AI |
| MISSION:lod.qsc | LOD config |
| SESSION:savegames/*.qsc | Saved state |

---

### Save Game Format
**Pattern:** `SESSION:savegames/%d-%d-%d-%d.igi2save`  
Format: `year-month-day-slot.igi2save`  
Slots: 6 maximum (0-5)

---

### Other Formats

| Format | Description |
|--------|-------------|
| .jpg | JPEG splash screens (loading%02d.jpg, innerloop.jpg) |
| .tga | Targa line filters, drop effect, laser dot |
| .MP3 | MP3 audio (*.MP3) |
| .bin | Binary data |
| .olm | Offline lightmap cache |

---

## External Libraries

| Library | Purpose |
|---------|---------|
| libpng 1.0.5 (1999) | PNG image loading |
| libjpeg | JPEG loading |
| zlib | Compression/decompression |
| Direct3D | Textures, vertex buffers |
| OpenAL (AIL_*) | 3D audio |

---

## Compression & Encryption
- **zlib** compression (error messages at 0x67A53C)
- ILFF directory entries have flags field (may indicate compression)
- No explicit encryption found

---

## Resource Loading Pipeline

```
Resource_Load (FUN_0040c6d0) — Core resource dispatcher
  └── Resource_Open (FUN_0040c330) — File open via CreateFileA/FindFirstFileA
      └── Dispatch table @ 0x00704884 (0x90-byte entries)
          ├── Load handler → FUN_0040bb30
          ├── Parse handler → FUN_0040c6d0
          ├── Read handler → FUN_0040c6d0
          ├── Process handler → FUN_0040c480
          └── Cleanup handler → FUN_0040bb90/FUN_0054c310

QTask_Save (SaveTask @ 0x0054a760) — Writes task data to backup file
QTask_SerializeOutput (Task_SerializeOutput @ 0x00548700) — Serializes task data with \r\n wrapping
```

---

## Key Addresses

| Address | Significance |
|---------|-------------|
| 0x0040C6D0 | Resource_Load — Core resource dispatcher |
| 0x0040C330 | Resource_Open — File open handler |
| 0x004F7AD0 | Model_LoadMEF — Full MEF model loader (13 chunk types, v0.16) |
| 0x004F8230 | Model_LoadMEFResource — Simplified MEF loader |
| 0x004F89B0 | Model_LoadMTPTextures — MTP model texture loading |
| 0x004F8790 | TEX texture loader |
| 0x004F8930 | Animation IFF loader |
| 0x0053EB70 | Lightmap loader |
| 0x005512E0 | Terrain_MaterialLoad — Terrain resource loader (.tim/.phm/.dmc/.csq) |
| 0x00580810 | Terrain_HeightmapLoad — Height/material/lightmap loader (.thm/.tmm/.tlm) |
| 0x0055B360 | TerrainLightMap_Init — Terrain lightmap system |
| 0x00704884 | File type dispatch table |
| 0x00548700 | Task_SerializeOutput — Task serialization |
| 0x0054a760 | SaveTask — Task save to backup |
