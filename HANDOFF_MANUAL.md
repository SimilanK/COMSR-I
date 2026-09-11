# CURSR-V Main Board — Handoff Manual

Read this first in any new chat. Points to the existing files — nothing here
duplicates them.

## Files in this folder (already correct, do not regenerate)

- `MODULE_AND_PART_LIST.md` — 110-part group/tier breakdown, net classes, open items.
- `PCB_LAYOUT_STEPS.md` — the runbook, Step 0–5, current step marked.
- `references/DATASHEETS_README.md` — datasheet links, corrected part list.
- `references/primary-reference.md` — text extract of the tutorial's method
  (10-step procedure, partition/grounding rules). **Use this instead of
  re-reading `PCBDesignTutorialRevA.pdf` as images.** Only open the actual PDF
  if this text extract doesn't answer the question.
- `MODULE_LAYOUT_NOTES.md` — pre-existing per-module electrical rules (§1–§11),
  cited throughout the other docs. Not modified this session.

## Where things stand right now

- Step 0 (board setup): done except the final stackup correction (JLC standard
  0.2104/1.065/0.2104mm, εr 4.6) — being applied by whichever session has
  working file access to `Main board.kicad_pcb`.
- Step 1 (import 110 footprints): done.
- Step 2 (park by group outside the outline): current step, in progress.
- Step 3–5 (place by tier, route, finish): not started.

## Known constraint — read before touching the PCB file

**This chat's file-bridge access to `Main board.kicad_pcb` / `CURSR-Vboard.kicad_pcb`
is unreliable/blocked** (uncertain root cause — never confirmed, don't repeat
the old "hardlink" claim as fact). Working arrangement: whichever
session/chat actually has working write access to the `.kicad_pcb` does the
PCB edits; this chat only produces paste-ready values and instructions.
Check this at the start of a new chat rather than assuming either way.

## Safety item — unresolved, blocking

`PYRO+` has no arming switch and no fuse anywhere in its path
(J6 → D3 → FETs → CN1/CN2). Must be added to the schematic before build.
Not a layout task.

## Open technical items

- C25/C26 group ambiguous (buck vs U7 +5V vs USB VBUS) — check the net directly.
- Naming collision: capacitor ref "C2" vs group code "C2 (Comms)" — use group
  names, not codes, when giving instructions.
- Leftover junk files in the folder from an old diagnostic test:
  `test.kicad_pcb`, `test.kicad_zzz` — safe to delete, not part of the design.

## Working style for this project

- Manual/tutorial is the primary source before any internet search.
- No coordinate-obsessed instructions — give thickness/values/groupings, not
  X/Y placement math, unless explicitly asked.
- Verify against the real schematic netlist, not memory or old BOMs, before
  updating any of the MD files.
- Plain, short, non-decorated language. No unnecessary tool calls.
