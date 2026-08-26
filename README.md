# IGI 2: Covert Strike — Reverse Engineering Artifacts

**Target Binary:** IGI2.exe (x86 PE32, MSVC 6.0, DirectX 7)  
**Total Functions:** ~1,460  
**Status:** ✅ CORE SYSTEMS COMPLETE

---

## Progress Summary

| Chapter | Status | Functions | Confidence |
|---------|--------|-----------|------------|
| 01-Executable | ✅ REVERSED | 8 | HIGH |
| 02-Classes | ✅ REVERSED | 3 | HIGH |
| 03-Rendering | ✅ REVERSED | 20+ | HIGH |
| 04-AI | ✅ REVERSED | 30+ | HIGH |
| 05-Weapons | ✅ REVERSED | 25+ | HIGH |
| 06-Level-System | ✅ REVERSED | 25+ | HIGH |
| 07-Scripting | ✅ REVERSED | 40+ | HIGH |
| 08-Resources | ✅ REVERSED | 15+ formats | HIGH |
| 09-Audio | ✅ REVERSED | 50+ | HIGH |
| 10-Menus | ✅ REVERSED | 60+ | HIGH |
| 11-Configuration | ✅ REVERSED | 70+ options | HIGH |
| 12-Networking | ✅ REVERSED | 50+ | HIGH |
| 13-Global-Data | ✅ REVERSED | 13 areas | HIGH |
| cross-reference | ✅ REVERSED | — | HIGH |

**Total Functions Documented:** ~400+ (out of ~1,460)  
**Coverage:** ~27% of codebase  
**Systems Documented:** 14 of 15

---

## Key Statistics

| Metric | Value |
|--------|-------|
| Total functions | ~1,460 |
| Functions analyzed | ~400+ |
| Documentation files | 24 |
| Subagents used | 11 |
| File formats decoded | 15+ |
| Network packets | 27 GM types |
| AI events | 26+ |
| Audio functions | 37/37 mapped |
| Config options | 70+ |
| QVM opcodes | 30+ |
| Global data areas | 13 |
| Network modes | 7 |
| Menu screens | 14+ |

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

- **Status:** UNKNOWN / PARTIAL / REVERSED / VERIFIED
- **Confidence:** LOW / MEDIUM / HIGH
- `[FUN_00xxxxxx]` = unnamed function in Ghidra
- Address format: `0x0068ca94`

---

## Remaining Work (1/15)

### 14. Ghidra Enhancement ⬜
- Struct types not yet applied in Ghidra
- Function renaming not bulk-applied
- Comments not added to functions
- **Priority:** Implementation phase (requires Ghidra tool calls)
