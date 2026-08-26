# IGI 2: Covert Strike — PE Structure

**Status:** ✅ REVERSED  
**Confidence:** HIGH

---

## Basic PE Information

| Property | Value |
|----------|-------|
| File Format | PE32 (32-bit) |
| Architecture | x86 |
| Endianness | Little Endian |
| Compiler | Microsoft Visual C++ 6.0 |
| Image Base | 0x00400000 |
| Entry Point | 0x0065d36d |
| Subsystem | Windows GUI (likely) |
| Memory Size | ~136 MB (0x00400000 - 0x08679fff) |

---

## Section Headers

| Name | Virtual Address | Virtual Size | Raw Size | Entropy | Purpose |
|------|-----------------|--------------|----------|---------|---------|
| .text | 0x00401000 | ~101 MB | — | HIGH | Executable code |
| .rdata | 0x06720000 | ~3.6 MB | — | MEDIUM | Read-only data, strings, import table |
| .data | 0x06ab0000 | ~27 MB | — | LOW | Initialized data, globals |
| .data1 | 0x08676000 | 4 KB | — | — | Additional data |
| .rsrc | 0x08677000 | 12 KB | — | — | Resources (icons, version info) |
| tdb | 0xffdff000 | 64 KB | — | — | Thread Storage (NT TEB area) |

---

## Resources (.rsrc)

| Type | ID | Address | Description |
|------|----|---------|-------------|
| Icon | 1 | 0x086771b0 | Primary application icon |
| Icon | 2 | 0x08677498 | Secondary icon |
| Icon | 3 | 0x08677d40 | Tertiary icon |
| Icon | 4 | 0x08677e70 | Alternate icon |
| Icon | 5 | 0x08678d18 | Small/icon variant |
| GroupIcon | 65 | 0x08678e40 | Grouped icon resource |
| Version | 1 | 0x08678e90 | VS_VERSION_INFO |

---

## Section Characteristics

### .text (Code)
- **Contains:** All executable code
- **Permissions:** Read, Execute
- **Notable regions:**
  - 0x00401000 - 0x0040ffff: Startup/PE header code
  - 0x00410000 - 0x00600000: Core engine functions
  - 0x00600000 - 0x0671ffff: Game systems (rendering, AI, etc.)
  - 0x65d000-660000: CRT/standard library
  - 0x660000-0x672000: Utility functions

### .rdata (Read-Only)
- **Contains:** String table, import address table, constant data
- **Notable:**
  - 0x06a92d0-0x06a9c00: DLL import names
  - 0x06a92d0+: AIL_* function names (Miles Sound)
  - 0x06a9990+: WININET function names

### .data (Global Data)
- **Contains:** Global variables, initialized data, BSS
- **Key regions:**
  - 0x006bf6c4 - 0x006bf898: Runtime configuration
  - 0x006aee38: Early init callback pointer
  - 0x006ab000: Module init array
  - 0x08351cec: CRT file handle table
  - 0x08352060: Process heap pointer

---

## Import Address Table (IAT)

The IAT is located in .rdata. Imports are loaded via:
- **KERNEL32.DLL** — 48 functions (core OS)
- **USER32.DLL** — 1 function (MessageBoxA)
- **GDI32.DLL** — (names in strings, TBD)
- **WINMM.DLL** — 2 functions (multimedia)
- **WININET.DLL** — 5 functions (HTTP/networking)
- **WSOCK32.DLL** — (names TBD)
- **MSS32.DLL** — 37 functions (audio)
- **VERSION.DLL** — 3 functions (version info)

---

## Export Table

- **Ordinal range:** 1-18
- **Exported functions:** 16 state machine inits + 2 entry points + version queries
- **Export pattern:** CDAPFN0506_* (C++ class methods)

---

## Debug Information

- **No .pdb or debug symbols embedded** — stripped binary
- **Security cookies:** `__security_check_cookie` present at 0x00664514
- **Exception handling:** SEH-based (RtlUnwind, SetUnhandledExceptionFilter)

---

## Notable Characteristics

1. **Large .text section (101 MB)** — unusual for a game executable; likely includes embedded shaders, bytecode, or JIT code generation
2. **Miles Sound System** — full AIL API integration (37 functions)
3. **HTTP capability** — WinInet suggests update/download system
4. **MSVC 6.0 compilation** — no C++ exceptions (SEH only), no RTTI visible
5. **No packer detected** — clean PE structure, no UPX/Themida
6. **TLS block at 0xffdff000** — standard Windows TEB location

---

## Comparison: IGI 1 vs IGI 2

| Aspect | IGI 1 | IGI 2: Covert Strike |
|--------|-------|----------------------|
| Engine | Same (Quake-derived) | Same engine, enhanced |
| Rendering | DirectX 7 | DirectX 7 |
| Scripting | QVM bytecode | QVM bytecode |
| Audio | Miles Sound System | Miles Sound System |
| File Format | ILFF archives | ILFF archives |
| Level Count | 14 | 14+ missions |
| AI Types | 9 | 10+ types |

---

## Next Steps

- [ ] Map .text section regions to game systems
- [ ] Identify shader/effect data in .text
- [ ] Check for embedded resources in .rsrc
- [ ] Analyze section entropy for packed/encrypted regions
