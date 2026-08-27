# Phase 6: Audio System — IGI 2: Covert Strike

**Status:** 🟡 PARTIAL — 65+ functions renamed  
**Functions Documented:** 65+ (50+ original + 15 Round 5)  
**Confidence:** HIGH  
**Verified:** 2026-08-27 via Ghidra MCP

### Round 6 Update (2026-08-27)
- [x] System_InitializeAll (0x005D6040) — master init for audio subsystems
- [x] SoundDef_Group_Register (FUN_006111a0) — already renamed
- [x] SoundDef_Graph_Register (FUN_00610240) — already renamed
- [x] SoundDef_Sound_Register (FUN_0060d1c0) — already renamed
- [x] SoundDef_TriggerOnce_Register (FUN_006099e0) — already renamed
- [x] SoundManager_Register (FUN_00606dd0) — already renamed
- [x] AnimSound_Register (FUN_00605cd0) — already renamed

---

## Audio Engine Architecture

**Engine:** Miles Sound System (MSS)  
**DLL:** MSS32.dll  
**API:** 37 AIL_* functions (Audio I/O Library)  
**Music:** DirectMusic (COM interface)  
**Max Sounds:** 1200 simultaneous  
**Sound Slots:** 3072 allocated

---

## 3D Audio Initialization (Sound_InitializeMiles @ 0x00542570)

**Address:** 0x00542570  
**Plate Comment:** Miles Sound System core init — allocates sound memory, sets AIL redistrib dir, calls _AIL_startup, _AIL_open_digital_driver, _AIL_enumerate_3D_providers. Selects "Miles Fast 2D Positional Audio" provider. Allocates 3D sample pool.

### Initialization Flow:
1. Check init flag (0x006c07f0)
2. Allocate provider struct (0x28 bytes)
3. Set distance boundaries (near/mid/far/max)
4. Call `AIL_open_3D_provider`
5. Allocate 3072 sound slot descriptors (0x60 bytes each)
6. Enumerate providers — matches **"Miles Fast 2D Positional Audio"**
7. Set 3D speaker type
8. Open 3D context (max 1200 sounds)
9. Init sound pool (32 pool entries)

### Shutdown:
- `AIL_close_3D_provider`
- `AIL_shutdown`
- Cleanup routine at 0x00445dc0

---

## Sound System Functions (Round 5 — 15 new renames)

### Sound Core
| Address | Name | Purpose |
|---------|------|---------|
| 0x00542570 | Sound_InitializeMiles | Miles Sound System core init: _AIL_startup, 3D provider selection |
| 0x00606dd0 | SoundManager_Register | SoundManager task init with parameter list update callback |
| 0x00407420 | Game_InitSoundSystem | Main game sound init: loads common/location/mission sounds, sets up music |
| 0x00420a40 | GOSound_Output | Game object sound output: Music/Speech/FX calls to buffer |

### Sound Definitions
| Address | Name | Purpose |
|---------|------|---------|
| 0x006099e0 | SoundDef_TriggerOnce_Register | Trigger-once sound with Falloff Begin/End params |
| 0x0060d1c0 | SoundDef_Sound_Register | Sound definition with Falloff Begin/End params |
| 0x00610240 | SoundDef_Graph_Register | Graph sound with Falloff Begin/End params |
| 0x006111a0 | SoundDef_Group_Register | Sound group container registration |

### Sound Generator
| Address | Name | Purpose |
|---------|------|---------|
| 0x00489950 | SoundGenerator_Register | Play/stop callbacks, sound generator pools |
| 0x00489b80 | SoundGenerator_Shutdown | Frees SoundGenerator pools |
| 0x004898f0 | SoundGenerator_PlaySound | Play sound handler with sound lookup and handle storage |
| 0x00489820 | SoundGenerator_StopSound | Stop sound handler with handle clearing |
| 0x004895f0 | SoundGenerator_Stop3DSound | Stop 3D sound via 3D sample table iteration |

### Music & Special
| Address | Name | Purpose |
|---------|------|---------|
| 0x00431160 | MenuMusic_Register | Menu music init with "menu_music" parameter |
| 0x00605cd0 | AnimSound_Register | Animation sound with parameter callbacks |
| 0x00472290 | RadioSound_Register | Radio sound system with start/stop/volume callbacks |

---

## AIL Functions (37 Imported)

### Stream Management
| Function | Address | Purpose |
|----------|---------|---------|
| AIL_open_stream@12 | 0x6723cc | Open audio stream |
| AIL_close_stream@4 | 0x6723d8 | Close stream |
| AIL_start_stream@4 | 0x6723ec | Start/resume |
| AIL_stream_status@4 | 0x6723f0 | Get state |
| AIL_set_stream_volume_pan@12 | 0x67240c | Volume + pan |
| AIL_set_stream_loop_count@8 | 0x67242c | Loop count |
| AIL_pause_stream@8 | 0x6723b4 | Pause |

### Sample Management
| Function | Address | Purpose |
|----------|---------|---------|
| AIL_allocate_sample_handle@4 | 0x67244a | Allocate handle |
| AIL_init_sample@4 | 0x67243a | Initialize |
| AIL_stop_sample@4 | 0x67241e | Stop |
| AIL_resume_sample@4 | 0x672434 | Resume |
| AIL_release_sample_handle@4 | 0x67242e | Release |
| AIL_sample_status@4 | 0x672440 | Get state |
| AIL_set_sample_volume_pan@12 | 0x672458 | Volume + pan |
| AIL_set_sample_loop_count@8 | 0x67243e | Loop count |
| AIL_set_sample_playback_rate@8 | 0x67241c | Change pitch |
| AIL_set_sample_ms_position@8 | 0x67240c | Seek |
| AIL_set_sample_address@12 | 0x672396 | Set memory buffer |
| AIL_set_sample_type@12 | 0x6723ee | Set format |
| AIL_set_named_sample_file@20 | 0x6723ac | Load from file |

### 3D Audio
| Function | Address | Purpose |
|----------|---------|---------|
| AIL_set_3D_sample_info@8 | 0x67237a | 3D metadata |
| AIL_allocate_3D_sample_handle@4 | 0x6723ec | Allocate handle |
| AIL_stop_3D_sample@4 | 0x6723f8 | Stop 3D |
| AIL_resume_3D_sample@4 | 0x6723e4 | Resume 3D |
| AIL_end_3D_sample@4 | 0x6723ee | End 3D |
| AIL_release_3D_sample_handle@4 | 0x6723fe | Release |
| AIL_set_3D_sample_volume@8 | 0x6723d0 | 3D volume |
| AIL_set_3D_position@16 | 0x6723ee | 3D position (x,y,z) |
| AIL_set_3D_velocity@20 | 0x6723c4 | Velocity (Doppler) |
| AIL_set_3D_sample_distances@12 | 0x6723e0 | Distance boundaries |
| AIL_set_3D_sample_occlusion@8 | 0x67237a | Occlusion value |
| AIL_3D_sample_status@4 | 0x6723ea | Query status |
| AIL_3D_sample_offset@4 | 0x6723cc | Get offset |
| AIL_sample_ms_position@12 | 0x6723ce | Get MS position |

### Speaker & Provider
| Function | Address | Purpose |
|----------|---------|---------|
| AIL_3D_speaker_type@4 | 0x6723fe | Query speaker config |
| AIL_set_3D_speaker_type@8 | 0x672330 | Set 8/12/16 channel |
| AIL_enumerate_3D_providers@12 | — | List providers |
| AIL_set_redist_directory@4 | — | Redist directory |

### Core
| Function | Address | Purpose |
|----------|---------|---------|
| AIL_startup@0 | — | Start MSS |
| AIL_shutdown@0 | 0x6723b6 | Stop all audio |

---

## 3D Audio Positioning

**Provider:** "Miles Fast 2D Positional Audio" at 0x0068a7b8

### Sound Slot Structure (0x60 bytes each, 3072 slots)
| Offset | Field | Description |
|--------|-------|-------------|
| +0x00 | next free slot pointer | |
| +0x04 | prev slot pointer | |
| +0x08 | handle pointer | |
| +0x0c | buffer pointer | |
| +0x10 | buffer size | |
| +0x14 | sample handle | |

**Active sound counter:** 0x083507d4

### Distance Boundaries
- 0x4456a0, 0x445720, 0x445770, 0x445860 (near/mid/far/max)

---

## Sound File Paths

| Path | Address | Usage |
|------|---------|-------|
| MISSION:sounds/ | 0x6a5fdc | Mission sounds |
| LOCAL:common/sounds/ | 0x6a607c | Shared sounds |
| LOCAL:%s%s | 0x684004 | Path construction |

**Format:** .wav files loaded via AIL

---

## Sound Definition Classes

| Class | Address | Purpose |
|-------|---------|---------|
| SoundDefBase | 0x68a640 | Base class |
| SoundDefSound | 0x687b9c | Regular SFX |
| SoundDefGraph | 0x687c08 | Graph-based sound |
| SoundDefTriggerOnce | 0x687c78 | One-shot trigger |
| SoundDefGroup | 0x687cf0 | Sound group |
| SoundDefGroupEdit | 0x683b90 | Editable group |
| SoundDefSoundEdit | 0x683fd4 | Editable SFX |
| SoundDefGraphEdit | 0x683cc4 | Editable graph |
| SoundChannel | 0x684110 | Channel management |
| VolumeChannel | 0x684130 | Volume control |
| Volume Envelope | 0x683ca0 | ADSR envelope |

**Manager:** "SoundManager" at 0x684374

---

## Music System (DirectMusic)

**Technology:** Microsoft DirectMusic COM

### Error
`"Couldn't create CLSID_DirectMusic"` at 0x69b068

### Volume Controls

| Function | Address | Parameter |
|----------|---------|-----------|
| Config_SoundOptionsGetMusicVolume | 0x6a0e74 | Get music vol |
| Config_SoundOptionsSetMusicVolume | 0x6a0f6c | Set music vol |
| Config_SoundOptionsGetMusic | 0x6a0e98 | Get music on/off |
| Config_SoundOptionsSetMusic | 0x6a0f90 | Set music on/off |
| Config_SoundOptionsGetSpeechVolume | 0x6a0e30 | Get speech vol |
| Config_SoundOptionsSetSpeechVolume | 0x6a0f28 | Set speech vol |
| Config_SoundOptionsGetSoundsEffectsVolume | 0x6a0eb4 | Get SFX vol |
| Config_SoundOptionsSetSoundsEffectsVolume | 0x6a0fac | Set SFX vol |
| Config_SoundOptionsGetReverseStereo | 0x6a0e0c | Get reverse stereo |
| Config_SoundOptionsSetReverseStereo | 0x6a0e0c | Set reverse stereo |

### Runtime Controls
| Function | Address | Purpose |
|----------|---------|---------|
| Game_DisableMusic | 0x6a5930 | Disable music |
| Game_EnableMusic | 0x6a5944 | Enable music |
| Game_SetMusicVolume | 0x6a596c | Set volume runtime |
| Game_SetSFXVolume | 0x6a5958 | Set SFX runtime |
| Game_UpdateVolume | 0x6a59b0 | Apply changes |

### Messages
- `"Setting up music..."` at 0x6a5f30
- `GOSoundMusic` at 0x6a1a4c (GO-level music)

---

## Key Addresses

| Address | Significance |
|---------|-------------|
| 0x00542570 | Sound_InitializeMiles — Miles Sound System core init |
| 0x68a7b8 | Provider name string ("Miles Fast 2D Positional Audio") |
| 0x68a640 | SoundDefBase |
| 0x684374 | SoundManager |
| 0x6a0c80 | Config region |
| 0x672330-672458 | AIL function pointers |
| 0x00606dd0 | SoundManager_Register |
| 0x00407420 | Game_InitSoundSystem |
| 0x00489950 | SoundGenerator_Register |
| 0x004898f0 | SoundGenerator_PlaySound |
| 0x00489820 | SoundGenerator_StopSound |
