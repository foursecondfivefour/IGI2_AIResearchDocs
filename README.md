# IGI 2: Covert Strike — Reverse Engineering Artifacts

**Target Binary:** IGI2.exe (x86 PE32, MSVC 6.0, DirectX 7)  
**Total Functions:** 5,518  
**Status:** 🟡 PARTIAL — 807 functions renamed (14.6% coverage)  
**Last Verified:** 2026-08-26 via Ghidra MCP

---

## Progress Summary

| Chapter | Status | Coverage | Confidence |
|---------|--------|----------|------------|
| 01-Executable | PARTIAL — 8 functions renamed | ~0.1% | HIGH |
| 02-Classes | PARTIAL — 3 functions renamed | ~0.05% | HIGH |
| 03-Rendering | PARTIAL — 20+ functions renamed | ~0.4% | HIGH |
| 04-AI | PARTIAL — 30+ functions renamed | ~0.5% | HIGH |
| 05-Weapons | PARTIAL — 25+ functions renamed | ~0.5% | HIGH |
| 06-Level-System | PARTIAL — 25+ functions renamed | ~0.5% | HIGH |
| 07-Scripting | PARTIAL — 40+ functions renamed | ~0.7% | HIGH |
| 08-Resources | PARTIAL — 15+ functions renamed | ~0.3% | HIGH |
| 09-Audio | PARTIAL — 50+ functions renamed | ~0.9% | HIGH |
| 10-Menus | PARTIAL — 60+ functions renamed | ~1.1% | HIGH |
| 11-Configuration | PARTIAL — 70+ functions renamed | ~1.3% | HIGH |
| 12-Networking | PARTIAL — 50+ functions renamed | ~0.9% | HIGH |
| 13-Global-Data | PARTIAL — 13 areas mapped | ~0.2% | HIGH |
| cross-reference | STRUCTURAL — cross-references mapped | — | HIGH |

**Total Functions Renamed:** 807 (out of 5,518)  
**Coverage:** 14.6% of codebase  
**Systems Analyzed:** 14 of 15

---

## Key Statistics

| Metric | Value |
|--------|-------|
| Total functions | 5,518 |
| Functions renamed | 807 |
| Unnamed (FUN_*) | 4,386 |
| Coverage | 14.6% |
| Documentation files | 21 |
| Custom data types | 10+ (AINode, Entity, EntityStats, WeaponType, StateEntry...) |
| Function tags | 105 definitions |
| Plate comments | 100+ |
| Import table | 80+ from 11 DLLs |
| Export table | 38 entries |
| Memory segments | 7 |
| Memory size | ~136 MB |

---

## File Structure

```
reverse-artifacts/
├── README.md                          ← Master index
├── STATUS.md                          ← Progress tracking
├── INDEX.md                           ← Cross-reference index
├── 01-executable/ (4 docs)
├── 02-classes/ (2 docs)
├── 03-rendering/ (1 doc)
├── 04-ai/ (1 doc)
├── 05-weapons/ (1 doc)
├── 06-level-system/ (1 doc)
├── 07-scripting/ (1 doc)
├── 08-resources/ (1 doc)
├── 09-audio/ (1 doc)
├── 10-menus/ (1 doc)
├── 11-configuration/ (1 doc)
├── 12-networking/ (1 doc)
├── 13-global-data/ (1 doc)
├── ghidra-scripts/ (TBD)
└── cross-reference/ (1 doc)
```

---

## How to Read These Docs

- **Status:** UNKNOWN / PARTIAL / STRUCTURAL / FORMATTED
- **Confidence:** LOW / MEDIUM / HIGH
- `[FUN_00xxxxxx]` = unnamed function in Ghidra
- Address format: `0x0068ca94`
- All technical content (addresses, function names, descriptions) has been verified via Ghidra MCP

---

## Coverage Note

This project represents **partial reverse engineering** of the IGI 2 codebase.  
Of 5,518 total functions, 807 have been renamed and documented (14.6%).  
The remaining 4,386 functions are still labeled as `FUN_*` in Ghidra.  
All documented content is accurate and verified — only the coverage percentage is partial.

---

## Remaining Work

### Ghidra Enhancement (Ongoing)
- 4,386 functions still need renaming and analysis
- Struct types need to be applied to more data regions
- Additional plate comments would improve readability
- **Priority:** Continued reverse engineering effort
