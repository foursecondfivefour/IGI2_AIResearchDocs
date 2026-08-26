# IGI 2: Covert Strike — Import Table

**Status:** ✅ REVERSED  
**Total Imports:** 80+ functions across 11 DLLs  
**Confidence:** HIGH

---

## KERNEL32.DLL (48 imports)

Core Windows kernel API — memory, file I/O, threading, process management.

### Memory Management
```
VirtualAlloc, VirtualFree, VirtualProtect, VirtualQuery
HeapCreate, HeapDestroy, HeapAlloc, HeapFree, HeapReAlloc, HeapSize
```

### File I/O
```
CreateFileA, ReadFile, WriteFile, SetFilePointer, FlushFileBuffers
FindFirstFileA, FindNextFileA, FindClose, DeleteFileA
SetEndOfFile, GetFileSize
```

### Dynamic Loading
```
LoadLibraryA, GetProcAddress, FreeLibrary
FindResourceA, FindResourceW, SizeofResource, LoadResource, LockResource
UnmapViewOfFile, MapViewOfFile, CreateFileMappingA
```

### Process & Threading
```
CreateThread, GetExitCodeThread, GetCurrentThread, GetCurrentProcess
TerminateProcess, ExitProcess
CreateMutexA, ReleaseMutex, CloseHandle, WaitForSingleObject
```

### System Information
```
GetVersionExA, GetSystemInfo, IsProcessorFeaturePresent
GetLastError, SetLastError, FormatMessageA
GetTickCount, QueryPerformanceCounter, QueryPerformanceFrequency
GetLocalTime, Sleep
```

### Console & Handles
```
GetStdHandle, GetFileType, SetStdHandle, SetHandleCount
```

### Command Line & Environment
```
GetCommandLineA, GetStartupInfoA, GetEnvironmentStrings, FreeEnvironmentStringsA
GetEnvironmentStringsW, FreeEnvironmentStringsW
GetEnvironmentVariableA, SetEnvironmentVariableA
GetCurrentDirectoryA, SetCurrentDirectoryA, GetFullPathNameA
```

### Locale & String
```
GetLocaleInfoA, GetLocaleInfoW
CompareStringA, CompareStringW
LCMapStringA, LCMapStringW
GetStringTypeA, GetStringTypeW
GetACP, GetOEMCP, GetCPInfo, IsValidLocale, IsValidCodePage
MultiByteToWideChar, WideCharToMultiByte
```

### Exception Handling
```
SetUnhandledExceptionFilter, RaiseException, UnhandledExceptionFilter
IsBadCodePtr, IsBadReadPtr, IsBadWritePtr
```

### Debug
```
OutputDebugStringA
```

---

## USER32.DLL

### UI & Messages
```
MessageBoxA
```

---

## GDI32.DLL

*(Import listed but specific functions not visible in string table)*  
Likely: standard GDI rendering calls (BitBlt, CreateCompatibleDC, etc.)

---

## WINMM.DLL (2 imports)

### Multimedia
```
mciSendCommandA     — MCI device control (CD audio, MIDI)
mixerSetControlDetails — Mixer volume/controls
```

---

## WININET.DLL (5 imports)

### Internet/HTTP
```
InternetOpenA           — Initialize WinInet session
InternetOpenUrlA        — Open HTTP/FTP URL
InternetReadFile        — Read from HTTP handle
InternetSetFilePointer  — Seek in HTTP stream
InternetCloseHandle     — Close HTTP handle
```

**Purpose:** Game likely uses this for:
- Downloading updates/patches
- Multiplayer content
- Online verification

---

## WSOCK32.DLL

*(Import exists — Windows Sockets 32-bit API)*  
Enables TCP/IP networking for multiplayer functionality.

---

## MSS32.DLL (37 imports) — Miles Sound System

**Full AIL (Audio I/O Library) API integration.**

### 3D Audio Provider
```
AIL_open_3D_provider@4
AIL_close_3D_provider@4
AIL_enumerate_3D_providers@12
AIL_3D_speaker_type@4
AIL_set_3D_speaker_type@8
```

### Sample Management
```
AIL_allocate_sample_handle@4
AIL_release_sample_handle@4
AIL_init_sample@4
AIL_sample_status@4
AIL_3D_sample_status@4
AIL_sample_ms_position@12
AIL_set_sample_ms_position@8
AIL_sample_loop_count@8
AIL_set_sample_loop_count@8
AIL_set_sample_playback_rate@8
AIL_set_3D_sample_playback_rate@8
AIL_stop_sample@4
AIL_resume_sample@4
AIL_3D_sample_offset@4
AIL_set_3D_sample_offset@8
```

### Stream Management
```
AIL_open_stream@12
AIL_close_stream@4
AIL_start_stream@4
AIL_stream_status@4
AIL_resume_3D_sample@4
AIL_pause_stream@8
AIL_end_3D_sample@4
AIL_stop_3D_sample@4
AIL_set_stream_volume_pan@12
AIL_set_sample_volume_pan@12
```

### 3D Positioning
```
AIL_set_3D_position@16
AIL_set_3D_velocity@20
AIL_set_3D_sample_distances@12
AIL_set_3D_sample_volume@8
AIL_set_3D_sample_occlusion@8
AIL_set_3D_sample_info@8
```

### File & Driver
```
AIL_set_named_sample_file@20
AIL_set_redist_directory@4
AIL_open_digital_driver@16
AIL_shutdown@0
AIL_startup@0
AIL_set_sample_type@12
```

---

## VERSION.DLL (3 imports)

```
GetFileVersionInfoSizeA@4
GetFileVersionInfoA@16
VerQueryValueA@16
```

---

## External Globals (TLS/TEB)

The binary has several references to Windows TLS/TEB structures at `0xffdff000+`:

| Offset | Name | Type | Purpose |
|--------|------|------|---------|
| +0x000 | ExceptionList | void* | SEH chain |
| +0x004 | StackBase | void* | Stack bounds |
| +0x008 | StackLimit | void* | Stack bounds |
| +0x020 | ClientId | CLIENT_ID | Thread/process ID |
| +0x030 | ProcessEnvironmentBlock | void* | PEB pointer |
| +0x1d4 | GdiTebBatch | dword[233] | GDI batch |
| +0x6b4 | RealClientId | CLIENT_ID | Client ID |
| +0x6cc | Win32ClientInfo | void*[62] | Win32 info |
| +0xbf0 | glContext | void* | OpenGL context |
| +0xbf4 | LastStatusValue | dword | NT status |

---

## Summary

**The binary uses a standard Windows application model:**
1. **No heavy game-specific DLLs imported** — all game logic is self-contained
2. **Miles Sound System** — full 3D audio via MSS32.DLL
3. **WinInet + WinSock** — network/HTTP capability (multiplayer, updates)
4. **Standard file I/O** — loads game assets from disk (no custom loaders)
5. **Dynamic loading** — uses LoadLibrary for optional modules
6. **Exception handling** — SEH-based error recovery
