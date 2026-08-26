# Phase 3.3: Rendering System — IGI 2: Covert Strike

**Status:** ✅ REVERSED  
**Functions Documented:** 20+  
**Confidence:** HIGH

---

## DirectX Initialization (FUN_00446360)

**Address:** 0x00446360  
**Purpose:** Full Direct3D 7/8 initialization pipeline

### Flow:
1. Load DDRAW.DLL → DirectDrawCreate
2. Create primary/secondary surfaces
3. QueryInterface → DDraw2, DDS3, DDS4
4. CoInitialize() → COM
5. CoCreateInstance(CLSID_DirectMusic) → audio
6. DirectDrawCreateEx → D3D device
7. Load D3D8.DLL → Direct3D 8
8. Load dpnhpast.dll → DirectPlay

### Error Codes:
| Code | Meaning |
|------|---------|
| 0x100 | DDraw2 QI failure |
| 0x200 | DInput failure |
| 0x300 | DDS3 QI failure |
| 0x500 | DDS4 QI failure |
| 0x700 | D3D8.DLL load failure |
| 0x800 | dpnhpast.dll load failure |
| 0x801 | SUCCESS |

---

## Rendering Pipeline

### Core Pipeline:
```
1. DirectDraw surface creation
2. D3D device enumeration
3. Texture allocation
4. Render state configuration
5. Terrain mesh rendering
6. BSP tree rendering
7. Model rendering (rigid, bone, sprite)
8. Forest renderer
9. Water renderer
10. Lightmap compositing
11. Environmental maps
12. Sky rendering (dome/flat)
13. Lens flare effects
14. Fog application
15. Sprite rendering
16. Present/swap
```

---

## Texture System

### Texture Functions (pc_direct3dtexture.c)

| Function | Address | Purpose |
|----------|---------|---------|
| AllocTexture | 0x6887f0 | Allocate texture |
| DeAllocTexture | 0x6887e0 | Free texture |
| DownloadTexture | 0x6887d0 | Upload to VRAM |
| RefreshTexture | 0x688774 | Update texture |
| ReloadTexture | 0x688764 | Reload from disk |
| BeginTextureList | 0x6887bc | Start enumeration |
| EndTextureList | 0x6887ac | End enumeration |

### Texture Paths:
- `LOCAL:textures/%s.tex` — Base textures
- `%stextures/%s.res` — Resource-pack
- `LOCAL:common/textures/%s` — Common textures
- `MISSION:textures/` — Mission-specific

### Texture Formats/Filtering:
| Mode | Constant |
|------|----------|
| Disabled | — |
| Bilinear | 0x692b30 |
| Trilinear | 0x692b84 |
| Anisotropic | 0x692b58 |

---

## Model System (MEF Format)

### Model Functions

| Function | Address | Purpose |
|----------|---------|---------|
| LoadModel | 0x68fdd4 | Load model file |
| UnloadModel | 0x68fdc8 | Unload model |
| InitModels | 0x68fdf0 | Init model system |
| DeinitModels | 0x68fde0 | Cleanup models |

### Model Types:
- Character Models (0x68bd38)
- WeaponModel (0x68615c)
- 1st Person Model (0x693a88)
- Camera Model (0x694d6c)
- ProjectileModel (0x695a94)
- Casing Model (0x69d5ac)

### LOD System:
| Setting | Address | Purpose |
|---------|---------|---------|
| LOD bias (Trilinear) | 0x684548 | Mipmapping bias |
| LOD bias (Standard) | 0x684568 | Mipmapping bias |
| LOD blend | 0x684594 | LOD blending |
| View cutoff (m) | 0x686464 | Distance cutoff |
| LODs affected by wind | 0x686424 | Wind LOD count |

### LOD Settings (0x69d368 area):
- LocalModelLODSettingsContainer
- ModelLODSettingsContainer
- ModelLODSettings
- LODSettings

---

## Camera System

### Camera Functions

| Function | Address | Purpose |
|----------|---------|---------|
| FreeCamera | 0x6824bc | Free resources |
| GetCameraTarget | 0x685c8c | Get target |
| EnterCameraCube | 0x6863ec | Cube transition in |
| LeaveCameraCube | 0x6863dc | Cube transition out |
| DrawCameraCone | 0x690190 | Debug visualization |
| GetHumanCameraInfo | 0x6910a8 | Player camera data |

### Camera Properties:
| Property | Address | Purpose |
|----------|---------|---------|
| Camera position | 0x687004 | World position |
| Camera orientation | 0x686ff0 | Rotation |
| Camera shake | 0x68781c | Shake intensity |
| Maximum FOV | 0x68c6cc | FOV upper limit |
| Minimum FOV | 0x68c6d8 | FOV lower limit |
| Trace FOV | 0x6912ec | Trace FOV |

### Camera Filters:
| Type | Constant |
|------|----------|
| None | 0x68a750 |
| Line | 0x68a738 |
| Band | 0x68a720 |
| Handicam | 0x68a6e4 |

---

## Render State Errors (Debug Strings)

| Error | Address |
|-------|---------|
| "Couldn't set renderstate" | 0x69af0c |
| "D3DERR_CONFLICTINGRENDERSTATE" | 0x69ac3c |
| "Couldn't set texturestagestate" | 0x6845c0 |
| "D3DERR_WRONGTEXTUREFORMAT" | 0x69aa40 |
| "D3DERR_UNSUPPORTEDTEXTUREFILTER" | 0x69aa5c |

---

## Key Globals (Graphics)

| Address | Variable | Purpose |
|---------|----------|---------|
| 0x0073fb4c | DAT_0073fb4c | FPU control word |
| 0x0073fb50-5c | CPUID flags | MMX/SSE detection |
| 0x00688d4c | ModelObj | Model container |
| 0x0068fc3c | zModel | Depth-sorted model |
| 0x0068bfb0 | ModelViewBox | Bounding box |
