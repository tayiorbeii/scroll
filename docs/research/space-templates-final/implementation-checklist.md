# Implementation checklist — bindable space templates (scroll #384)

Phase order per [plan.md](plan.md). "Verified" = built + manual/automated check
run in this clone.

## Phase 1 — Data model
- [ ] `sway_space_container.slot` + `hint{app_id,class,title}` fields added (`include/sway/tree/space.h:13-32`)
- [ ] accessors + destroy paths updated (`sway/tree/space.c`)
- [ ] `space_template` object (or flagged `sway_space`) created; unbound leaves supported
- [ ] meson.build lists new `sway/tree/space_template.c`
- [ ] Verified: fixture JSON → template tree round-trips slots/fractions

## Phase 2 — JSON serialization
- [ ] `ipc_json_describe_space_template()` emits `version`/`name`/tiling/floating/slot/size fractions/view_hint/focused_slot
- [ ] geometry fields added to live `get_spaces` JSON too (fixes current omission at `sway/ipc-json.c:1592-1627`)
- [ ] `space_template_load_json()` parser (json-c): rejects empty slots, tolerates unknown keys, validates fractions
- [ ] Verified: round-trip identity; malformed-input rejection with precise error

## Phase 3 — IPC + commands
- [ ] `IPC_GET_SPACE_TEMPLATE` / `IPC_LOAD_SPACE_TEMPLATE` wired: `swaymsg/main.c` dispatch (near :983), `sway/ipc-server.c` handler, `include/sway/ipc-server.h` codes
- [ ] `space_template save|load|bind|commit|cancel` commands in `sway/commands/`
- [ ] completions updated (`completions/`)
- [ ] man pages: `scroll-ipc.7.scd`, `scroll.5.scd`
- [ ] Verified: `scrollmsg -t get_space_template work` on live session

## Phase 4 — Binding engine
- [ ] placeholder rendering decided (empty container vs titled visual); Overview/Jump interplay handled
- [ ] `space_template bind <slot>` reuses `criteria_matches_view()` for hints (`include/sway/criteria.h:75-76`)
- [ ] pending→commit/cancel lifecycle; unbound-slot timeout configurable
- [ ] documented precedence: slot binding > for_window assign (i3 parity, `i3/docs/layout-saving:41-44`)
- [ ] Verified: out-of-order app launch + binds; fractions/focus correct after commit; cancel restores prior state

## Phase 5 — Lua API + example
- [ ] `space_template_get/load/bind/commit/cancel` registered in `sway/lua.c:1984` registry
- [ ] example binder script (view_mapped + hints + exec_process + retry + commit) shipped and tested via `scrollmsg --lua_repl`
- [ ] Verified: example restores a 3-slot template from cold start

## Phase 6 — Docs + upstream
- [ ] TUTORIAL.md section; 2 example templates
- [ ] discussion #384 reply drafted (answers its two open questions with evidence from research.md)
- [ ] phased local commits; NO PR without explicit approval

## Cross-cutting
- [ ] `git diff --check` clean; meson build green each phase
- [ ] no state files written outside `$XDG_CONFIG_HOME/scroll/templates/`
- [ ] crash safety: templates are plain data; no view pointers serialized
