# Research: tiling managers ecosystems saved real-world window (INCONCLUSIVE)

> ⚠ **This run is INCONCLUSIVE.** No fully evidence-backed conclusion was reached.
> The sections below describe what was searched and why proof remained incomplete — they are
> diagnostics, not verified findings. Do not present this as completed research.

## Goal

Find real-world examples of how tiling window managers and their ecosystems implement saved layouts and partial session restoration with named placeholder windows/slots, to inform implementing 'bindable space templates' in dawsers/scroll (a sway fork with scrollable PaperWM-style tiling and a Lua scripting API; see its GitHub discussion #384). Cover: (1) i3-resurrect / i3 layout save-restore with placeholder swallow criteria; (2) sway ecosystem window-matching tools (swayr, sworkstyle, etc.); (3) niri and hyprscroller/Hyprland scrollable-tiling persistence; (4) PaperWM/GNOME, KDE, Qtile, awesome session-restore approaches; (5) the xdg-session-management Wayland protocol draft. Extract concrete patterns: template/schema formats, window-matching criteria, placeholder-to-window binding flows, IPC/API design, and how to add a versioned template export/import API plus a Lua slot-binding API to a sway-based compositor.

## Status

- **Status:** `inconclusive`
- **Reasons:** no_selected_repositories, no_evidence_anchors
- Candidates evaluated: 12
- Repositories selected: 0
- Evidence anchors: 6

## Summary

Run `run_tiling_managers_ecosystems_real_world_1789617779706` evaluated 12 candidate(s), selected
0 repository/repositories, and captured 6
evidence anchor(s), but did not satisfy the complete proof contract. This is
**not** proof that the answer is negative. Treat these artifacts as diagnostics,
not completed findings.

## What was searched

- [high_precision] "tiling" "managers" — Find files combining the two most distinctive feature concepts.
- [repo_search] tiling managers ecosystems saved real-world window — Discover repos by name/topic/readme when code search misses.

## Open questions (unresolved)

- Were any files actually read for the candidates? Are likelyFiles/proof signals correct?

## Recovery steps

- Inspect research.md "What was searched" to confirm probes targeted the right entities.
- If discovery missed the project, add the official repo/docs URL as an explicit hint or candidate.
- Provide structured hints (requiredConcepts, likelyFiles, proofRequirements) for novel domains.
- Re-run into a new output directory and compare; do not overwrite inconclusive artifacts.

## Diagnostics (warnings)

- missingProof: eval-exec/neomacs was not proof-read (maxReposToProve reached).
- missingProof: FancyWM/fancywm was not proof-read (maxReposToProve reached).
- missingProof: grandyang/leetcode was not proof-read (maxReposToProve reached).
- missingProof: halfrost/LeetCode-Go was not proof-read (maxReposToProve reached).
- missingProof: jdpedersen1/Void-Herbstluftwm was not proof-read (maxReposToProve reached).
- missingProof: JohnnyFoulds/macos-window-grid was not proof-read (maxReposToProve reached).

## Skeptic notes

- An inconclusive run must never be presented as completed research.
- Re-run after fixing the highest-priority reason above before drawing any conclusion.
