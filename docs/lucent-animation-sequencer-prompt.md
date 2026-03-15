# Implementation Prompt — Lucent Animation & Sequencer

Use this prompt to start a fresh implementation session.

---

You are implementing full Sequencer animation support, Blueprint runtime control, and smooth
preset transitions for the **Lucent** plugin in an Unreal Engine 5.7 project.

**Before writing a single line of code:**
1. Read `docs/lucent-animation-sequencer.md` in full — it is the authoritative spec.
2. Read the key source files listed in that document's Key Files table to understand the
   existing code before modifying it.
3. Reference `Plugins/Tonal/` as the completed reference implementation for all design patterns
   and naming conventions. Mirror its structure exactly, substituting `Lucent`/`FLucentSettings`
   where Tonal uses `Tonal`/`FTonalSettings`.

**Non-negotiable rules:**
- Follow Lucent's existing code style and naming conventions exactly (F-prefix structs, U-prefix
  UObjects, `r.Lucent.*` CVar prefix, `LUCENT_API` / `LUCENTEDITOR_API` export macros).
- Do not leave dead code, orphaned declarations, or unimplemented stubs. Every function
  declared in a header must have a corresponding implementation. Every `#include` added must
  be used.
- Do not add features beyond what the spec describes. No extra refactoring, no speculative
  helpers, no comment cleanup on untouched code.
- Do not modify shader files (.usf/.ush). All changes are C++ only.
- Implement phases in the order specified: 1 → 4 → 2 → 3 → 5.1–5.7. Complete and verify each
  phase before starting the next.
- After Phase 1, confirm the three new CVarLucent* variables (`CVarLucentDistortionCustomCoefficients`,
  `CVarLucentDistortionSharpening`, `CVarLucentCAFeather`) are declared in `LucentCVars.h`
  before using them in `LucentSettings.cpp`.
