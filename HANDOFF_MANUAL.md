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

**Root cause now confirmed (2026-09-11): not OneDrive, not this folder
specifically.** The connected-folder bridge (`device_bash`) fails at the VM
mount layer itself — error `sandbox-helper: no Plan9 drive shares mounted` —
for every connected path simultaneously (tested against the OneDrive-labeled
path, the plain `Desktop` connection, and the shell's own home scratch
directory; all fail identically, before any command runs). This is a known
platform bug tracked by Anthropic (Windows update from ~Sept 8 broke the
Plan9/virtiofs mount on Windows Cowork sessions — see GitHub issues
anthropics/claude-code#92958, #92984, #93221). It is NOT caused by OneDrive
sync, folder permissions, or anything about this specific project — do not
re-investigate OneDrive as a cause.

**Separately (cosmetic, not the bug above):** this task's Context panel also
carries a stale `C:\Users\simil\OneDrive\Desktop\Main board` folder entry
that doesn't exist on disk — leftover from an earlier misconfigured
connection. There is currently no UI way to remove a folder from a task's
context (open feature request: anthropics/claude-code#40043) — ignore that
pill, it's harmless clutter, not the cause of the mount failure.

**Korya is migrating the project to a new folder path** to work around/retest
this — check with Korya what the new path is before assuming the old
`C:\Users\simil\Desktop\Main board` still applies.

**Working arrangement until the mount bug is fixed:** `device_bash` (in-place
shell edits) does not work. Use `device_stage_files` → edit in the cloud
workspace → `device_commit_files` instead — this path is unaffected and
works fine. Whichever session is doing PCB edits should use that stage/edit/
commit flow rather than waiting for `device_bash` to start working.

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
