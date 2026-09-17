# Implementation checklist — bindable space templates (v2.1)

Tracks [plan.md](plan.md) v2.1 (G0 resolved: H-01 restore_hide sweep; H-02/D-06
no compositor placeholders, single-shot atomic apply; fill-then-retry is
userland scripting). "Verified" = built + check run in this clone.

## G0 — decisions ✅ RESOLVED 2026-09-17
- [x] H-01: restore_hide sweep at apply; CLOSE stays out; sweep undone on post-sweep failure
- [x] H-02/D-06: no compositor placeholders, no pending/timeout/cancel machinery
- [x] D-02: apply error names missing slots; fill-then-retry is userland scripting
- [x] Recorded in run `20260917162037-gd9n1c` (`pi-plan/spec-space-templates-g0-decisions.md`)

## G1 — plain-data template model
- [ ] `include/sway/tree/space_template.h` + `sway/tree/space_template.c`: versioned template object (name, scroller modifiers, tiling/floating trees, slot IDs, optional hints, focused slot)
- [ ] No view pointers/listeners anywhere in durable state; partial-tree-safe ctors/dtors; deep copy
- [ ] `sway/meson.build` registers new sources; legacy `sway_space` untouched
- [ ] Verified: fixture tree round-trips; ASan/LSan-clean create/destroy

## G2 — JSON + persistence
- [ ] `space_template_json.c` exporter/importer per plan §4 (enums, fractions, hints, focused_slot)
- [ ] JSON-path errors; unknown keys tolerated; version rejected; depth/node/regex limits; PCRE2 compiled at validation
- [ ] `$XDG_CONFIG_HOME/scroll/templates/<name>.json`: safe names, atomic temp+rename, lazy load + cache refresh, corrupt-file isolation
- [ ] Verified: semantic round-trip identity; malformed fixtures rejected with stable paths; save failure leaves prior file intact

## G3 — atomic apply engine
- [ ] Validate-mappings-first: every slot → exactly one mapped live view, else fail with **zero mutation**
- [ ] Error enumerates missing/ambiguous slots (the only incompleteness interface — D-02)
- [ ] Scratchpad sweep of unrelated target-workspace views (`restore_hide` parity); sweep undone if arrange fails
- [ ] Arrange bound views per template (fractions/scroller/floating/focus `focused_slot`); dissolve all template state after
- [ ] No empty leaf containers at any point; second apply serialized
- [ ] Verified: incomplete mapping mutates nothing; out-of-order staging irrelevant; sweep/rollback exact

## G4 — commands + IPC
- [ ] `space_template save|apply` commands; `apply_space_template` IPC (`{name, workspace:"current", mappings:[…]}`) — **no fallback/retry/staging fields (anti-feature guard, D-02)**
- [ ] Wired through `include/ipc.h`, `sway/ipc-server.c`, `swaymsg/main.c`; legacy `get_spaces` unchanged
- [ ] Completions (bash/fish/zsh) + man pages (`scroll-ipc.7.scd`, `scroll.5.scd`)
- [ ] Verified: live `scrollmsg -t get_space_template` round-trip; deterministic error objects

## G5 — Lua adapter + examples
- [ ] `space_template_get(name)`, `space_template_apply(name, mappings)` → `(nil, error)` naming missing slots; **no fallback/retry helpers in the binding (anti-feature guard)**
- [ ] Example (a) canonical fill-then-retry script (plain loop, no callbacks required)
- [ ] Example (b) launcher placeholders: stage launcher terminals → apply → swap for real apps on selection
- [ ] Verified: cold-start three-slot restore via `scrollmsg --lua_repl`; no callback/state leaks

## G6 — hardening + docs
- [ ] TUTORIAL/man updates: template anatomy, apply contract, restore_hide sweep, userland fallback/placeholder patterns, current-workspace limitation
- [ ] Two example templates (dev mail+terminal+editor per #384; writing)
- [ ] Discussion #384 reply (dedicated API — D-01; all-slots-required with userland fallback — D-02/D-06)
- [ ] Full repo checks (Meson build+tests, `git diff --check`, completions, man generation); review-ready local commits; **no PR/merge without explicit authorization**
- [ ] PR branches cut fresh from upstream `master`, cherry-picked implementation commits only; `docs/research/**`, `.pi/`, `.pi-persona/` never enter an upstream branch

## Cross-cutting invariants
- [ ] Anti-feature: no fallback/retry/staging parameters anywhere in compositor, IPC, or binding (D-02)
- [ ] No state files outside `<config>/scroll/templates/`; no generated runtime state committed
- [ ] Crash safety: templates are plain data; apply is the only mutating operation
