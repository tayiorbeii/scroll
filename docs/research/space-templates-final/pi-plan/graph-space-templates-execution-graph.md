# Execution graph: bindable space templates (#384)

## Graph metadata

- Baseline: `2879127f663852a272808bc843af6c88a7f4f0d6` on `master`.
- Scope: all feature work is remaining; legacy Space behavior is a preserved historical seam, not a rewrite target.
- Owner rule: one writer per node/worktree; integrate only after node acceptance and normal repository checks.
- Human-decision gates: H-01 (unrelated live views / destructive restore semantics) and H-02 (placeholder presentation). Defaults are recorded in the spec but must be confirmed before the nodes that depend on them.
- Review gate: engineering-manager independent review is `approve-with-changes`; its six corrections are incorporated below.

## Dependencies

```text
G0 decisions + contracts
  -> G1 plain-data model + session ownership
  -> G2 JSON/schema + persistence
  -> G3 placeholder projection + transactional binding
  -> G4 commands + IPC
  -> G5 Lua adapter + binder example
  -> G6 end-to-end validation + docs/examples
```

G2 can begin after G1's data contract is stable. G4's read/export half can begin after G2; its load/bind half requires G3. G5 requires G3 and uses the public API from G4. G6 requires all prior nodes. No node may silently resolve H-01/H-02 by implementation convenience.

## Nodes

### G0 — Confirm product and compatibility gates

- **Readiness:** blocked until human decisions are confirmed; no product mutation.
- **Inputs:** reviewed spec sections 4–7; `research.md` design lessons 3–6; existing `SPACE_RESTORE_LOAD/CLOSE/HIDE` behavior in `sway/tree/space.c:202-248`.
- **Deliverable:** record H-01 and H-02 decisions in the implementation issue/plan handoff; confirm one pending session per current workspace, dedicated IPC, manual default timeout, and legacy `get_spaces` compatibility.
- **Acceptance:** explicit choice for unrelated live views at commit and placeholder visibility in Overview/Jump/animations; explicit v1 exclusion of scratchpad CLOSE/HIDE if not approved.
- **Gate check:** no implementation node starts if a decision changes the JSON shape, destructive behavior, or ownership model.

### G1 — Introduce template-owned plain data and runtime session boundaries

- **Depends on:** G0.
- **Likely files:** `include/sway/tree/space_template.h`, `sway/tree/space_template.c`, narrowly shared helpers in `include/sway/tree/space.h`/`sway/tree/space.c`, `include/sway/tree/root.h` and root lifecycle code, `sway/meson.build`.
- **Outcome:** a template tree owns strings/lists/geometry only; a runtime session owns the pending workspace, immutable template reference/private copy, slot→live-container bindings, placeholder containers, rollback snapshot, timer/listeners, and focus state. Legacy `sway_space`, `sway_space_view`, `space_save`, `space_load`, `space_delete`, and `space_destroy_all` remain valid and do not gain durable view pointers.
- **Required contracts:** constructors/destructors for every partial-tree failure; deep copy; slot/hint ownership; session cleanup on commit/cancel/unmap/config reload/root teardown; no double-removal of `wl_listener` links; one active session policy.
- **Acceptance:** unit tests can create/destroy a template with nested tiling/floating nodes, no live view pointers, and a session snapshot without changing `root->spaces`; ASan/LSan-oriented cleanup path is covered where the repository supports it.
- **Validation:** compile the target, run the new model/lifetime tests, `git diff --check`, and the normal Meson test subset.

### G2 — Implement version-1 JSON and XDG persistence

- **Depends on:** G1.
- **Likely files:** `sway/tree/space_template_json.c` and header (or the G1 module if smaller), `include/sway/ipc-json.h`, `sway/ipc-json.c`, config-path helper seam in `sway/config.c`, parser/persistence tests and fixtures, `sway/meson.build`, `tests/meson.build`.
- **Outcome:** canonical export/import for the spec's `version`, `name`, `scroller`, `tiling`, `floating`, `focused_slot`, node `layout`/`children`/`size`/floating geometry/`slot`/`view_hint` fields. Add a template serializer rather than changing the legacy `get_spaces` contract; if geometry is also fixed in legacy output, make that a separately reviewed compatibility change.
- **Required contracts:** name/path-safe validation; existing config-home conventions; lazy named load and cache refresh; atomic temp-file + rename with correct permissions; corrupt/unsupported files do not replace a valid cached object; unknown keys tolerated; all parser errors carry JSON paths; PCRE2 patterns compile during validation; depth/node/regex limits prevent pathological input; no arbitrary IPC file path.
- **Acceptance:** fixture JSON round-trips semantically (including fractions, scroller modifiers, slots, hints, and focused slot); every invalid class is rejected with stable error path; save failure leaves prior file intact; reload sees replacement; no writes outside `<config>/scroll/templates/`.
- **Validation:** parser/persistence test target, malformed fixtures, permissions/write-failure tests where portable, build, and `git diff --check`.

### G3 — Build placeholder projection and transaction semantics

- **Depends on:** G1, G0; consumes G2's validated template object.
- **Likely files:** `sway/tree/space_template_session.c`/header, focused extensions to `sway/tree/space.c` or layout helpers, map/unmap and transaction seams in `sway/ipc-server.c`/view code, runtime tests.
- **Outcome:** loading the current workspace enters Pending without serializing pointers; internal placeholder leaves represent every slot; a view can bind once by slot; all slots are required for commit; commit applies the bound tree, focus, fractions, floating geometry, and scroller state; cancel/timeout/failure restores the exact pre-session workspace.
- **Required contracts:** placeholder path must not depend on current `layout_space_container_restore_tiling` silently dropping NULL-view leaves; add an explicit placeholder-aware path. Preserve H-01 behavior for unrelated views. Define H-02 behavior in Overview/Jump/animation traversal. Slot binding wins over ordinary assign/`for_window`. Map/unmap events must reject stale or duplicate bindings and remove callback/listener state. Timeout uses the event loop; default manual; timeout only cancels.
- **Acceptance:** two-slot apps mapped out of order bind to intended slots; duplicate/stale bind fails; incomplete commit fails without mutation; successful commit restores focused slot; cancel and timeout restore tree/fractions/floating positions/focus/scratchpad membership and remove listeners; unmap during Pending cannot leave a dangling binding; second session is rejected.
- **Validation:** focused runtime/integration tests under the existing compositor fixture, leak/crash tests, and normal build/test gates.

### G4 — Add commands and dedicated IPC contracts

- **Depends on:** G2 for named persistence/export; G3 for session operations.
- **Likely files:** `include/ipc.h`, `include/sway/ipc-json.h`, `sway/ipc-server.c`, `swaymsg/main.c`, `common/ipc-client.c` only if required by payload handling, `sway/commands/space_template.c` or `sway/commands/space.c`, `sway/commands.c`, `sway/meson.build`, `completions/bash/scrollmsg`, `completions/fish/scrollmsg.fish`, `completions/zsh/_scrollmsg`, `sway/scroll-ipc.7.scd`, `sway/scroll.5.scd`.
- **Outcome:** allocate unused scroll-specific enum values and wire message-name dispatch. `get_space_template` accepts a name and returns canonical template JSON or a structured error. `load_space_template` accepts a JSON object containing the name/current-workspace selector and returns success/error. Commands expose `save [--with-hints]`, `load`, `bind`, `commit`, `cancel` with exact argument/error semantics and no arbitrary server path.
- **Required contracts:** IPC malformed payloads, unknown names, no workspace/output, duplicate sessions, incomplete commits, and stale containers return deterministic errors; existing `get_spaces` remains unchanged; raw/pretty `scrollmsg` behavior is deliberate; completions and man pages use the exact names.
- **Acceptance:** live test compositor can save/get/load a fixture; shell command errors are observable; IPC round-trip works in raw mode; completion entries and man-page tables contain all new types/commands.
- **Validation:** IPC integration tests, `scrollmsg -t get_space_template <name>`, command parser tests, completion checks, man-page generation, build.

### G5 — Expose Lua adapter and userland binder example

- **Depends on:** G3 and G4 public/session contracts.
- **Likely files:** `sway/lua.c`, Lua API tests (`tests/test_lua_api.py` and a new template test), `examples/` or `docs/`, README/tutorial references, Meson test registration.
- **Outcome:** register `space_template_get/load/bind/commit/cancel` with consistent return/error convention. The example listens to `add_callback("view_map", ...)`, reads view app_id/class/title, launches missing apps with `exec_process`, retries with bounded backoff, binds matching slots, and commits only when complete.
- **Required contracts:** callback ordering relative to view map and IPC events; closure removal on commit/cancel/error; no retained stale container/view references; `workspace_set_focus`/existing JSON conversion reuse; matching policy remains userland-owned.
- **Acceptance:** Lua can inspect a template, load it, bind a container, cancel/commit, and receive useful errors; the example cold-starts a three-slot fixture and succeeds with out-of-order maps; repeated reload/cancel does not accumulate callbacks.
- **Validation:** Lua API/integration tests, example smoke test through `scrollmsg --lua_repl`, build and leak checks.

### G6 — Cross-cutting hardening, documentation, and release handoff

- **Depends on:** G1–G5 and G0 decisions.
- **Likely files:** `TUTORIAL.md` or repository docs location, `sway/scroll*.scd`, `swaymsg/scrollmsg.1.scd`, two example JSON templates, tests/fixtures, preserved discussion reply draft outside product source if requested.
- **Outcome:** teach template anatomy, criteria regexes, generated-vs-authored slots, precedence, pending/commit/cancel/timeout, current-workspace limitation, unsupported scratchpad/output behavior, and safe persistence. Ship dev (mail+terminal+editor) and writing examples without pretending app launch is compositor responsibility.
- **Acceptance:** docs match the actual JSON/API/error contracts; examples validate with the parser; all new source/test files are in Meson; no generated runtime state is committed; `git diff --check`, configured build/tests, parser/IPC/Lua/runtime suites, and man generation pass.
- **Validation:** full normal repository checks and a final baseline/diff audit. Stop at local commits/review-ready state; no PR or merge without explicit authorization.

## Traceability

- ST-01/ST-02/ST-03 → G1, G2.
- ST-04/ST-05 → G4.
- ST-06/ST-07/ST-08 → G3, with G0 H-01/H-02 gates.
- ST-09 → G5.
- ST-10 → G2–G6.
- Legacy compatibility and no-pointer persistence invariant → G1/G2/G3/G4.
- Independent review corrections → explicit ownership (G1), registry/reload (G2), schema/error paths (G2), rollback (G3), IPC payload/session contracts (G4), Lua/map/test gates (G3/G5/G6).

## Current frontier

The plan is ready for implementation planning but not for blind execution: G0 is the human-decision frontier, followed by G1. Once H-01/H-02 are confirmed, hand off G1 as the first bounded vertical slice; no implementation should begin by editing the existing live Space serializer or `layout_space_restore` without the ownership and rollback contracts above.
