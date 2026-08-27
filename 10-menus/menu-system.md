# Phase 7: Menu, Configuration & HUD System — IGI 2: Covert Strike

**Status:** 🟡 PARTIAL — 80+ functions renamed  
**Functions Documented:** 80+ (60+ original + 20 Round 5)  
**Confidence:** HIGH  
**Verified:** 2026-08-26 via Ghidra MCP

---

## Menu System Architecture

### Core Components

| Component | Address | Purpose |
|-----------|---------|---------|
| MenuManager | 0x69F39C | Central manager, stack handling |
| MenuScreen | 0x6963A8 | Base screen class |
| MenuWindow | 0x68E01C | Window container |
| MenuFrame | 0x694B40 | Frame/screen wrapper |
| MenuItem | 0x685FBC | Menu item data |
| MenuButton | 0x6978B8 | Button widget |
| MenuText | 0x69A34C | Text widget |
| MenuTextSelection | 0x69A338 | Text selection |
| DialogWindow | 0x68B16C | Modal dialog |
| SlideBar | 0x690880 | Slider widget |

### MenuManager States (0x69F404-0x69F494)
| State | Description |
|-------|-------------|
| MENUMANAGER_IDLE | Idle |
| MENUMANAGER_STARTGAME | Starting game |
| MENUMANAGER_QUITTOMAINMENU | Return to main menu |
| MENUMANAGER_QUIT | Quit app |
| MENUMANAGER_JOINGAME | Join multiplayer |
| MENUMANAGER_RESTARTLEVEL | Restart level |

### Menu Screens
| Screen | Address | Resource |
|--------|---------|----------|
| ComputerMenu | 0x68B6C4 | missions.res |
| ComputerPlanMenu | 0x695FDC | — |
| ComputerSaveMenu | 0x690A80 | — |
| ActionMenuScreen | 0x68C734 | — |
| ControlsMenu | 0x690DF8 | — |
| MenuStatsScreen | 0x694F00 | — |
| LoadingScreen | 0x6A0C5C | loadingscreen.res |

### Main Menu
- Script: `LOCAL:menusystem/mainmenu.qsc`
- Event: FLOW_EVENT_MAINMENU at 0x6A6AD0

### Menu Loading Pipeline (Round 5 Enhanced)
1. FUN_004324C0 (MenuResourceLoader):
   - loads menusystem.res
   - missionsprites.res
   - missions.res
2. FUN_00433C50 (MenuManager_Initialize):
   - Creates "MenuManager" task
   - Registers 7 menu items
   - Event handlers at offsets 0x2714, 0x16C, 0x28, 0x170, 0x271E, 0x2719, 0x168
   - **Plate Comment:** Screen management, input handling, button callbacks, 0x98-byte params
3. FUN_00433E90 (MenuManager_Deinitialize):
   - Frees parameter structure, clears active menu screen

### Menu System Initialization (Round 5 — 8 new renames)

| Address | Name | Purpose |
|---------|------|---------|
| 0x00433c50 | MenuManager_Initialize | Screen management, input handling, button callbacks, 0x98-byte params |
| 0x00433e90 | MenuManager_Deinitialize | Frees parameter structure, clears active menu screen |
| 0x005c6c30 | MenuItemTask_Initialize | MenuItemTask with input/hitbox/navigation handlers |
| 0x005c5e40 | MenuItem_Initialize | MenuItem with Get/Activate/Update handlers |
| 0x0060aee0 | PushButton_Initialize | PushButton with input/hitbox registration |
| 0x0059ae10 | TagItem_Initialize | TagItem base + derived types (PushButton, Int32, Real32, String16/256, Bool8) |
| 0x00468ce0 | MenuButton_Initialize | MenuButton with input/activation/click handlers |
| 0x004cb030 | ComputerButtonItem_Initialize | Computer/console menu interaction with hitbox registration |

### Input Handling (InputSubsystem_RegisterTypes @ 0x004262C0)

**Address:** 0x004262C0  
**Plate Comment:** Registers ~100+ input QTask types including movement bindings, keyboard device types, key angle mappings, and language configs. Initializes input state arrays for 10 players (0x2B0 bytes each).

#### Input System Functions (Round 5)

| Address | Name | Purpose |
|---------|------|---------|
| 0x004262C0 | InputSubsystem_RegisterTypes | ~100+ input QTask types, movement bindings, keyboard devices |
| 0x00435F10 | InputPort_Initialize | InputPort subsystem with global device state zeroing |
| 0x00420CA0 | Input_LogConfig | Logs mouse sensitivity, invert, key remappings |
| 0x0052D3E0 | HumanPlayerInput_Initialize | HumanPlayerInput with record/playback handlers |

---

## Configuration System

### Core Config I/O (0x6A0CF0 region)
| Function | Address | Purpose |
|----------|---------|---------|
| Config_LoadConfig | 0x6A0CF0 | Load main config |
| Config_SaveConfig | 0x6A0CDC | Save main config |
| Config_GetRGBA | 0x6A0D04 | Get RGBA color |
| Config_GetActiveCrossHair | 0x6A0D14 | Active crosshair |
| Config_GetCrossHairs | 0x6A0D30 | Available crosshairs |
| Config_ResetGraphicOptions | 0x6A0D48 | Reset to defaults |
| Config_IsGermany | 0x6A0D64 | Region check |
| Config_VerifyContentControlPassword | 0x6A0DA0 | Content verification |

### Sound Options (0x6A0E0C - 0x6A0FD8)
| Option | Get | Set |
|--------|-----|-----|
| Reverse Stereo | GetReverseStereo | SetReverseStereo |
| Speech Volume | GetSpeechVolume | SetSpeechVolume |
| Speech | GetSpeech | SetSpeech |
| Music Volume | GetMusicVolume | SetMusicVolume |
| Music | GetMusic | SetMusic |
| SFX Volume | GetSoundsEffectsVolume | SetSoundsEffectsVolume |
| SFX | GetSoundsEffects | SetSoundsEffects |

### Graphic Options (0x6A0FFC - 0x6A1248)
| Option | Global Variable |
|--------|----------------|
| CrossHair Color B | GoGfxCrossHairColour |
| CrossHair Color G | GOGfxCrossHairAlpha |
| CrossHair Color R | — |
| CrossHair Alpha | GOGfxCrossHairAlpha |
| Gamma | GOGfxGamma |
| Device | GOGfxDevice |
| Transparency | — |
| Resolution | — |

### GfxOptions Detail Settings (0x692290-0x6927D8)
| Setting | Options |
|---------|---------|
| Texture Filtering | BILINEAR, ANISOTROPIC, TRILINEAR |
| Antialiasing | DISABLED, X2, X4, X6 |
| Terrain Detail | float |
| Water Quality | NORMAL, HIGH, HIGHEST |
| Model Detail | float (LOD bias) |
| LOD Bias | float |
| Texture Detail | ULTRALOW, LOW, HIGH |
| Detail Level | CUSTOM, LOW, MEDIUM, HIGH |
| Bullet Hit Effects | bool |
| Lens Flares | bool |
| Wind on Forests | bool |
| High Quality Forests | bool |
| Texture Compression | bool |
| Stencil Shadows | bool |
| Force Software Vertex Processing | bool |
| Thick Crosshair | bool |

### Input Options (0x6A126C - 0x6A1404)
| Option | Get | Set |
|--------|-----|-----|
| Blood Enabled | GetBloodEnabled | SetBloodEnabled |
| Mouse Sensitivity | GetMouseSensitivity | SetMouseSensitivity |
| Invert Mouse | GetInvertMouse | SetInvertMouse |

### Display Options (0x6A142C - 0x6A15D4)
| Option | Function |
|--------|----------|
| Number of Languages | Config_GetNumberOfLanguages |
| Language Name | Config_GetLanguageName |
| Render Device | Config_FillRenderDeviceListBox |
| Screen Resolution | Config_FillScreenResolutionListBox |
| Speaker Setup | Get/SetActiveSpeakerSetup, FillSpeakerSetupListBox |
| 3D Provider | Get/SetActive3DProvider, Fill3DProviderScrollListBox |
| Difficulty | Get/SetActiveDifficultyLevel, FillDifficultyLevelListBox |

### Player Profile System (0x6A1638-0x6A1700)
| Function | Purpose |
|----------|---------|
| Config_GetNumberOfPlayerProfiles | Count profiles |
| Config_PlayerGetActiveMission | Active mission per profile |
| Config_DeletePlayerProfile | Remove profile |
| Config_CreateNewPlayerProfile | Add profile |
| Config_SetActivePlayerProfileIndex | Select active |
| Config_GetActivePlayerProfileIndex | Get current |
| Config_FillPlayerProfileListBox | Populate UI |

### Config Files
| File | Path | Purpose |
|------|------|---------|
| SESSION:config.qsc | Main game config |
| networkconfig.cfg | Network settings |
| filesys.cfg | File system paths |
| LOCAL:config.qsc | Local config |

---

## Save Game System (0x6A5860)

| Function | Address | Purpose |
|----------|---------|---------|
| Game_GetNewestSaveGameSlot | 0x6A5860 | Get latest save |
| Game_FillSaveGameScrollListBox | 0x6A5898 | Populate list |
| Game_GetNumberOfValidSaveGames | 0x6A58B8 | Count saves |
| Game_SetMenuLoadFromSlot | 0x6A587C | Set load target |

**Save format:** `SESSION:savegames/%d-%d-%d-%d.igi2save`  
Format: `year-month-day-slot.igi2save`  
Slots: 6 maximum (0-5)  
QuickSave: `menusystem/Quicksave`  
AutoSave: `AutoSave` (task type)

---

## HUD Elements

### HUD Sprites
| Sprite | Path | Purpose |
|--------|------|---------|
| hud_1.spr | COMPUTER:hud_1.spr | HUD element 1 |
| hud_2.spr | COMPUTER:hud_2.spr | HUD element 2 |
| weapon.spr | STATUSSCREEN:weapon.spr | Weapon display |
| visibilitybar.spr | STATUSSCREEN:visibilitybar.spr | Visibility bar |
| powerbar.spr | STATUSSCREEN:powerbar.spr | Stamina bar |
| powerbarbackground.spr | STATUSSCREEN:powerbarbackground.spr | Bar background |
| a_c4.spr | STATUSSCREEN:a_c4.spr | C4 timer |
| crosshair.spr | crosshair.spr | Default crosshair |
| crosshair-t.spr | crosshair-t.spr | Thick crosshair |

### Objective System (0x68BFCC)
**Manager:** ComputerObjectives  
**Resource:** LANGUAGE:objectives.res  
**Sprites:** COMPUTER:objective_1.spr through objective_8.spr

**Per-Objective Data:**
1. Text Resource (e.g., "Objective 1")
2. Link To Position (waypoint)
3. Complete Expression (success trigger)
4. Failed Expression (failure trigger)
5. Objectives Valid flag

**Multiplayer:**
- "IGI WON OBJECTIVE #%d: YOUR ATTACK/DEFENCE SUCCESSFUL"
- "CONSPIRACY WON OBJECTIVE #%d: YOUR ATTACK/DEFENCE SUCCESSFUL"

### Status Screens
| Screen | Address | Purpose |
|--------|---------|---------|
| MissionTimer | 0x68B90C | Mission elapsed time |
| MissionTimerDrawTask | 0x68B8F4 | Timer rendering |
| MissionScoreScreen | 0x69779C | End-of-mission stats |
| MenuStatsScreen | 0x694F00 | Stats display |

---

## Input Handling

### DirectInput Setup
- **DirectInput8Create** at 0x69B47C (dinput8.dll)
- **Keyboard:** INPUTPORT_DEVICE_KEYBOARD (0x6Bafc8)
- **Mouse:** INPUTPORT_DEVICE_MOUSE (0x6BB010)
- **Mouse Buttons:** MOUSE_BUTTON_1 through MOUSE_BUTTON_8 (0x6BBEB3)
- **Mouse Wheel:** MOUSE_WHEEL_UP

**Error messages:**
- `Could not create DirectInput device keyboard!`
- `Could not set data format for keyboard!`
- `Could not set cooperative level for keyboard!`

---

## Key Addresses

| Address | Significance |
|---------|-------------|
| 0x69F39C | MenuManager |
| 0x6963A8 | MenuScreen |
| 0x6A0CF0 | Config_LoadConfig |
| 0x6A5860 | Save game system |
| 0x68BFCC | ComputerObjectives |
| 0x6A0C80 | Config options region |
| 0x6A0E0C | Sound options region |
| 0x6A0FFC | Graphic options region |
| 0x690DF8 | Controls menu |
| 0x6A0C5C | Loading screen |
| 0x00433c50 | MenuManager_Initialize |
| 0x00433e90 | MenuManager_Deinitialize |
| 0x005c6c30 | MenuItemTask_Initialize |
| 0x005c5e40 | MenuItem_Initialize |
| 0x004262c0 | InputSubsystem_RegisterTypes |
| 0x00435f10 | InputPort_Initialize |
