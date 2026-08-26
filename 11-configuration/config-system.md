# Phase 8: Configuration System — IGI 2: Covert Strike

**Status:** ✅ REVERSED + ROUND 5 ENHANCED  
**Config Files:** 8 documented  
**Functions Documented:** 40+ (25+ Round 5 additions)  
**Confidence:** HIGH

---

## Config Files

| File | Path | Purpose |
|------|------|---------|
| SESSION:config.qsc | Main game config script |
| networkconfig.cfg | Network/ multiplayer settings |
| filesys.cfg | File system paths |
| LOCAL:config.qsc | Local configuration |
| editormagicobjconfig.qsc | QED editor config |
| magicobjconfig.qsc | Magic object config |
| MISSION:lod.qsc | LOD configuration |
| SESSION:savegames/config.qsc | Save config |

---

## Configuration API

### Core (0x6A0CF0)
| Function | Purpose |
|----------|---------|
| Config_LoadConfig | Load main config |
| Config_SaveConfig | Save main config |
| Config_GetRGBA | Get RGBA color |
| Config_GetActiveCrossHair | Active crosshair |
| Config_GetCrossHairs | Available crosshairs |
| Config_ResetGraphicOptions | Reset to defaults |
| Config_IsGermany | Region check |
| Config_VerifyContentControlPassword | Content verification |

### Sound (0x6A0E0C-0x6A0FD8)
- Get/Set Music Volume/Enabled
- Get/Set Speech Volume/Enabled
- Get/Set SFX Volume/Enabled
- Get/Set Reverse Stereo

### Graphics (0x6A0FFC-0x6A1248)
- Crosshair: Color (BGR), Alpha, Type
- Gamma, Device, Resolution
- Detail Level: CUSTOM/LOW/MEDIUM/HIGH
- Texture: Filtering (Bilinear/Anisotropic/Trilinear), Detail (UltraLow/Low/High), Compression
- Rendering: Antialiasing (Disabled/X2/X4/X6), Water Quality (Normal/High/Highest)
- Effects: Lens Flares, Bullet Hit, Wind on Forests, High Quality Forests, Stencil Shadows
- Performance: Force Software Vertex Processing

### Input (0x6A126C-0x6A1404)
- Blood Enabled
- Mouse Sensitivity
- Invert Mouse

### Display (0x6A142C-0x6A15D4)
- Languages
- Render Device
- Screen Resolution
- Speaker Setup
- 3D Provider
- Difficulty Level

### Player Profiles (0x6A1638-0x6A1700)
- Count, Create, Delete, Select profiles
- Active mission tracking

---

## Save System

**Format:** `SESSION:savegames/%d-%d-%d-%d.igi2save` (year-month-day-slot)  
**Slots:** 6 maximum  
**Functions:** Game_GetNewestSaveGameSlot, Game_FillSaveGameScrollListBox, Game_GetNumberOfValidSaveGames

**Naming patterns:**
- `comp%02d%02d.qsc` — computer mission
- `co%02d%02d%02d.qsc` — co-op save
- QuickSave: `menusystem/Quicksave`
- AutoSave: `AutoSave`
