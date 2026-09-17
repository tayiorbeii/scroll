# Independent planning review — bindable space templates

- Reviewer: `persona-team.engineering-manager`
- Role/authority: planning-read-only
- Attestation: `.pi-persona/attestations/36e0b3ea-d5c1-456c-8c79-a3701f2f730f-16067ea20de0d0fe-0.json`
- Attested status: `passed`; all four required review methods applied; no files edited.
- Verdict: **approve-with-changes**. Execution must remain blocked until persistence/reload and rollback semantics are explicit.

## Findings recorded by the reviewer

1. **Separate aggregate ownership (block):** define template-vs-legacy `sway_space` ownership and destruction. The current `sway_space_container` owns live-view listeners and `sway_space` exposes the legacy `space_save`/`space_load` lifecycle (`include/sway/tree/space.h:12-53`, `sway/tree/space.c`). Do not let durable templates serialize or retain view pointers.
2. **Durable registry lifecycle (block):** specify XDG startup/lazy reload, cache invalidation, atomic save, and corrupt/unsupported-file behavior. Current spaces are in-memory only; the plan's path alone is insufficient (`research.md` gaps; `plan.md` Phase 2).
3. **Versioned JSON contract (approve-with-changes):** enumerate required/optional keys, unique non-empty slots, finite fraction/range and split invariants, regex compilation errors with JSON paths, unknown-key policy, version rejection, and nested error propagation.
4. **Pending transaction and rollback (block):** define the source-of-truth and lifecycle for pending → commit/cancel/timeout. Existing `layout_space_restore` mutates the workspace directly and has CLOSE/HIDE side effects (`sway/tree/space.c:202-248`); a cancel must restore views, tree, focus, floating state, and scratchpad membership without dangling listeners.
5. **IPC/session contract (approve-with-changes):** specify exact request payloads, reply/error objects, name/path validation, enum allocation, and whether one pending session exists per compositor/current workspace. Wire both `include/ipc.h` and `swaymsg/main.c` in addition to `sway/ipc-server.c`.
6. **Lua/event/test gates (approve-with-changes):** use the existing `add_callback("view_map", ...)`, view getters, `workspace_set_focus`, and JSON conversion APIs (`sway/lua.c:616-665,1337-1352,1820-1880,1984-2001`), but define callback lifetime and map-event ordering. Add Meson-registered parser/unit tests, IPC integration tests, pending rollback tests, Lua binder tests, and docs/build acceptance.

## Reviewer evidence

- `docs/research/space-templates-final/research.md:75-87` establishes the proven placeholder/swallow, userland matching, and existing Space gaps.
- `docs/research/space-templates-final/plan.md:32-116` contains the current six-phase proposal and its under-specified persistence/transaction seams.
- `include/sway/tree/space.h:12-53` and `sway/tree/space.c` show the existing live-view ownership and direct restore behavior.
- `sway/meson.build` explicitly registers source files, so new template modules/tests require explicit build wiring.
