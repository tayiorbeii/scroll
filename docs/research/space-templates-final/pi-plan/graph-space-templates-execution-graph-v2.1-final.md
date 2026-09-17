# Execution graph: bindable space templates — v2.1 final vertical slices

## Graph authority and supersession

- **Planning run:** `20260917162037-gd9n1c`
- **Authoritative source:** `docs/research/space-templates-final/plan.md` v2.1 at commit `347ad76b5b8de823ddbef8c0cd4d8dd312280756`.
- **G0:** resolved by Taylor on 2026-09-17; recorded in `spec-space-templates-g0-decisions.md`.
- **Supersedes:** `graph-space-templates-execution-graph.md`, whose G3–G5 still describe pending sessions, placeholders owned by the compositor, bind/commit/cancel, timeout, and built-in retry behavior.
- **Scope:** implementation-ready slices G1–G6 only; no product files are changed by this artifact.

## Final invariants (non-negotiable

1. **One operation:** apply is a single-shot command with the exact lifecycle `validate → sweep → arrange → dissolve`.
2. **Validation is zero-mutation:** before touching workspace or scratchpad state, require the current target workspace and a complete, unambiguous binding of every template slot to a mapped live view. Reject missing, duplicate, stale, unmapped, or ambiguous bindings without mutation.
3. **H-01 sweep:** every live view on the target workspace not selected by a slot moves to the scratchpad (`SPACE_RESTORE_HIDE` parity). `CLOSE` is not implemented.
4. **Post-sweep failure safety:** if arrange fails after the sweep, restore the exact swept membership before returning the error. No partial apply is observable.
5. **No runtime session:** there is no pending session, compositor placeholder window, empty placeholder leaf, bind/commit/cancel lifecycle, timeout, cancellation command, retry loop, fallback policy, or staging state.
6. **Anti-feature guard (D-02):** no compositor, IPC, command, or Lua API field/flag/option may be named or behave as `fallback`, `retry`, `staging`, `pending`, `timeout`, `cancel`, or `commit`. The only incompleteness interface is a structured apply error naming missing/ambiguous slots. Userland may catch that return, fill/launch/raise its own windows, and call the same one-shot apply operation again.
7. **Userland owns matching and placeholders:** template hints are data for scripts. Launcher terminals or other real windows are user-owned staging UX; the compositor only receives final slot-to-live-view bindings and arranges them.
8. **Dissolve:** after arrange, no template/session metadata remains attached to the live tree; later view lifecycle is ordinary view lifecycle.
9. **Compatibility:** legacy Space APIs and `get_spaces` behavior remain unchanged unless a separately identified compatibility fix is reviewed.

## Dependency order

```text
G0 resolved contracts
  └─> G1 model + ownership seam
        └─> G2 JSON/schema + persistence
              └─> G3 atomic apply engine
                    └─> G4 commands + dedicated IPC
                          └─> G5 Lua adapter + userland examples
                                └─> G6 integration, docs, hardening
```

G2 may begin after G1's data contract is stable. G3 consumes G1 and G2. G4 has no apply surface until G3's C contract is stable. G5 consumes G4's public contract but may prototype userland matching against existing Lua view APIs earlier. G6 depends on all slices. Each slice is a vertical seam with executable acceptance tests; do not merge a layer-only implementation without its slice gate.

## G1 — Plain-data template model and ownership seam

**Depends on:** G0.  **Unlocks:** G2 and G3.

### File seams

- `include/sway/tree/space_template.h` — public plain-data types, node/template limits, validation/error structs, and the final binding input type (slot → live view identity; no session type).
- `sway/tree/space_template.c` — constructors, deep copy, partial-tree-safe destruction, slot/hint ownership, structural validation helpers.
- `sway/meson.build` — register the new source.
- `tests/test_space_template_model.py` (new, or the repository's established focused model-test seam) — nested tree, ownership, and invalid-partial-tree coverage.

### Implementation contract

- Store only strings, lists, enums, finite geometry, regex text, slot IDs, and focused-slot data.
- A leaf has exactly one non-empty unique slot; an inner node has children and no slot.
- No `sway_view`, container pointer, listener, PID, con_id, timer, rollback snapshot, or pending/session field is durable or owned by the template model.
- The apply input is a complete final binding collection, not a staging object. It has no fallback/retry/timeout/cancel/commit fields.
- Preserve existing `sway_space`/`sway_space_view` ownership and legacy destruction paths.

### Acceptance contract

- A nested tiling/floating template can be created, copied, validated, and destroyed through every partial-construction failure path.
- Duplicate/empty slots, invalid node shape, invalid geometry, invalid focused slot, and over-limit structures produce stable structured errors.
- Static/source review proves the model contains no live-view or session ownership seam.

### Test gate

- Focused model/ownership tests, including sanitizer-friendly cleanup where available.
- Build the affected target and run `git diff --check`.
- Do not add or test pending-session, placeholder-window, retry, fallback, or staging abstractions.

## G2 — Canonical JSON and XDG persistence

**Depends on:** G1.  **Unlocks:** G3 and G4 export/read surfaces.

### File seams

- `sway/tree/space_template_json.c` and its header — canonical serializer/parser and JSON-path errors.
- `sway/config.c` (or the existing config-path helper seam) — `$XDG_CONFIG_HOME/scroll/templates/<name>.json` path resolution.
- `sway/meson.build`, `tests/meson.build` — source/test registration.
- `tests/test_space_template_json.py` (new) and `tests/fixtures/space-templates/` (new) — valid/invalid fixtures and persistence tests.

### Implementation contract

- Implement v1 `version`, `name`, `scroller`, `tiling`, `floating`, `focused_slot`, node layout/children/size, floating geometry, slot, and optional `view_hint` fields.
- Unknown keys are ignored; unsupported versions, unsafe names, invalid enums, bad fractions, bad regexes, duplicate/empty slots, and structural-limit violations are rejected with JSON paths.
- Named load is lazy and refreshes the cache. Save uses atomic temp-file + rename and never writes outside the template directory. A corrupt replacement cannot displace a valid cached object.
- Persistence has no runtime session record and no staging/fallback/retry metadata on disk.

### Acceptance contract

- Valid fixtures round-trip semantically, including nested geometry, scroller modifiers, hints, slots, and focus.
- Each invalid fixture reports the expected path/category; unknown fields remain forward-compatible.
- Permission/write/rename failure leaves the prior file intact; corrupt/unsupported files do not poison a valid cache; path tests prove no traversal or arbitrary file writes.

### Test gate

- Parser, round-trip, limits, regex compilation, cache-refresh, atomic-write, and corruption-isolation tests.
- Build and `git diff --check`.
- Any legacy `get_spaces` geometry correction is a separately named compatibility change, not hidden in this slice.

## G3 — Atomic apply engine

**Depends on:** G1 and G2.  **Unlocks:** G4 apply command/IPC.

### File seams

- `include/sway/tree/space_template_apply.h` (new) — one atomic apply entry point, complete binding input, and structured missing/ambiguous/stale-slot errors.
- `sway/tree/space_template_apply.c` (new), with narrowly scoped reuse/extensions in `sway/tree/space.c` and existing layout/scroller helpers — validate, sweep, arrange, dissolve.
- Existing scratchpad and workspace helpers in `sway/tree/space.c` / workspace tree code — restore-hide parity and sweep rollback; do not fork `CLOSE` behavior.
- `tests/test_space_template_apply.py` (new) plus existing compositor/client fixtures — live-view arrangement and failure-rollback coverage.

### Implementation contract

1. **Validate:** resolve the current workspace, verify every template slot maps exactly once to a mapped live view, reject stale/duplicate/ambiguous identities, and validate geometry/focus before mutation. A complete final binding list is the only apply input; remove the prior `--auto`/criteria-driven compositor matching proposal. Hints remain available to userland.
2. **Sweep:** record the exact target-workspace views not selected by a slot and move those views to the scratchpad with `SPACE_RESTORE_HIDE` semantics.
3. **Arrange:** construct the ordinary live tree from bound views keyed by slot; apply layout/scroller/fractions/floating geometry and focus. Never create a NULL-view leaf, fake client, placeholder window, pending container, or session record.
4. **Dissolve:** release temporary apply bookkeeping; no template/session state remains attached to the tree.
5. **Failure after sweep:** restore the recorded sweep membership before returning failure. Validation failures happen before the sweep and therefore require no rollback.

### Acceptance contract

- Missing, duplicate, ambiguous, stale, or unmapped slots return a structured error naming the affected slots and leave root tree, focus, geometry, and scratchpad membership byte-for-byte/equivalently unchanged.
- A successful out-of-order userland fill is irrelevant to the compositor: only a complete final binding set is accepted, and all bound views are arranged in the intended slots.
- Unselected target-workspace views move to scratchpad exactly as `SPACE_RESTORE_HIDE`; unrelated views are never closed.
- Any injected arrange failure after sweep restores every swept view and leaves no partial tree or metadata.
- No empty leaves, compositor placeholders, pending sessions, timeout handles, cancellation paths, fallback policy, retry state, or staging state exist in implementation or tests.
- Reentrant/concurrent apply is serialized or rejected as a normal command conflict without introducing a pending session.

### Test gate

- Live compositor tests for complete success, all validation failures, H-01 sweep, focus/geometry, post-sweep rollback, and no-empty-leaf invariant.
- Failure-injection test must force arrange failure after sweep and assert membership restoration.
- Build, sanitizer/leak checks where configured, focused suite, and `git diff --check`.

## G4 — Commands and dedicated IPC

**Depends on:** G2 for persistence and G3 for apply.  **Unlocks:** G5.

### File seams

- `include/ipc.h`, `include/sway/ipc-json.h`, `sway/ipc-server.c`, `swaymsg/main.c`, and `common/ipc-client.c` only if payload plumbing requires it — message enums, dispatch, encode/decode, raw/pretty replies.
- `sway/commands/space_template.c` (new) and command registration/build seams — `save` and one-shot `apply`; no session verbs.
- `completions/bash/`, `completions/fish/`, `completions/zsh/` and `sway/scroll-ipc.7.scd`, `sway/scroll.5.scd` — exact public names and contracts.
- `tests/test_space_template_ipc.py` (new) and command parser tests.

### Implementation contract

- Add dedicated `get_space_template` and `apply_space_template` IPC messages; leave legacy `get_spaces` untouched.
- `get_space_template` accepts a safe template name and returns canonical JSON or structured error.
- `apply_space_template` accepts the template name, current-workspace selector, and a complete final slot-to-live-view binding collection. If transport needs an object field, call it `bindings`; it is not a staging/session parameter and must not permit partial apply. Hints/criteria matching, launching, raising, filling, and retry remain outside IPC.
- Expose `space_template save <name> [--with-hints]` and one-shot `space_template apply <name> ...` with deterministic errors. Do not expose `load`, `bind`, `commit`, `cancel`, `timeout`, `retry`, `fallback`, `staging`, or pending-session commands/fields.
- No arbitrary server file paths. Unknown names, malformed payloads, no current workspace, incomplete bindings, and stale views fail without mutation.

### Acceptance contract

- Raw and pretty IPC round trips return the documented canonical JSON/error shape.
- A complete apply request reaches G3 once; an incomplete/ambiguous request reports named slots and proves zero mutation.
- Command help, completions, and man pages contain only the final one-shot API and no retired lifecycle verbs/options.
- Legacy Space IPC regression tests remain green.

### Test gate

- IPC live-compositor tests, malformed/unknown-name/incomplete-binding cases, command parser tests, completion checks, man-page generation, build, and `git diff --check`.

## G5 — Lua adapter and userland staging patterns

**Depends on:** G3 and G4.  **Unlocks:** G6.

### File seams

- `sway/lua.c` and Lua registration helpers — `space_template_get` and `space_template_apply` only.
- `tests/test_lua_space_templates.py` (new), alongside existing `tests/test_lua_api.py`/`tests/test_lua_safety.py` — return shape, errors, and lifecycle safety.
- `examples/space-templates/fill-then-retry.lua` (new) — userland fill loop.
- `examples/space-templates/launcher-placeholders.lua` (new) — real terminal/window placeholder pattern.
- `examples/space-templates/*.json` (new) and example/doc registration seams.

### Implementation contract

- Lua apply returns success or `(nil, error)`; missing/ambiguous slots are named. No compositor callback, fallback, retry, staging, pending, timeout, cancellation, bind, or commit machinery is added.
- `fill-then-retry.lua` is ordinary user code: call apply once, inspect the structured error, launch/raise user-selected defaults for the named slots, then call the same apply function again. The compositor never stores that loop or its policy.
- `launcher-placeholders.lua` uses real launcher terminals/windows, maps their final live views in the caller's binding collection, applies once, and later swaps real applications using existing view/move primitives. No fake compositor windows are created.
- View matching uses existing Lua callbacks/getters/process primitives only in examples; template hints are not an implicit compositor matching mode.

### Acceptance contract

- Lua can fetch a template and invoke one-shot apply with complete bindings; invalid/incomplete input returns `(nil, error)` naming slots and does not mutate.
- Both examples are plain scripts with no new callback handles owned by the compositor and no hidden retry/fallback parameters.
- Re-running scripts does not leak listeners or retain stale view/container references; ordinary later view close remains ordinary view lifecycle.

### Test gate

- Lua API/error/safety tests, example syntax/smoke tests, a cold-start fixture with out-of-order userland filling, build, and `git diff --check`.

## G6 — Integration, hardening, documentation, and release-ready handoff

**Depends on:** G1–G5.  **Terminal slice:** yes; no publication.

### File seams

- `TUTORIAL.md` or the repository's established tutorial/documentation location; `sway/scroll*.scd`; `swaymsg/scrollmsg.1.scd` — final API and userland workflow.
- `examples/space-templates/` and `tests/fixtures/space-templates/` — dev and writing templates plus validation fixtures.
- `tests/` — cross-slice regression and anti-feature guard tests.
- `docs/research/space-templates-final/` only for planning artifacts; do not publish issue text or modify product docs as part of this planning handoff.

### Implementation contract

- Documentation describes validate → sweep → arrange → dissolve, restore-hide sweep/rollback, current-workspace limitation, complete bindings, userland fill-then-retry, and real-window launcher placeholders.
- Remove every reference in feature docs/examples/tests to pending sessions, compositor placeholders, load/bind/commit/cancel, timeout, built-in fallback/retry, or staging parameters.
- Add an anti-feature test/grep gate that rejects retired API names and payload fields from the implementation and public docs, with explicit allowlisting only for historical planning text.
- Validate both example templates with the parser. Keep generated runtime state and unpublished GitHub drafts out of product files.

### Acceptance contract

- Full docs/API/example contract matches the C/IPC/Lua behavior and has no stale v2 lifecycle claims.
- All new C sources and tests are registered in Meson; normal build, focused suites, full configured tests, man generation, completions, and `git diff --check` pass.
- Final diff audit proves no unrelated legacy Space changes, no generated runtime artifacts, and no remote publication/PR/merge activity.

### Test gate

- Full configured repository checks plus parser, apply, IPC, Lua, rollback, sanitizer/leak, docs/man, completion, and anti-feature guard suites.
- Stop at local, review-ready commits. Publication to `tayiorbeii/scroll` requires separate owner confirmation and is not part of G6.

## Traceability

- ST-01 → G1; ST-02/ST-03 → G2.
- ST-04/ST-05 → G4.
- ST-06/ST-07/ST-08 (revised atomic apply) → G3 and G4.
- ST-09 → G5.
- ST-10 → G2–G6.
- Review finding 1 → G1; finding 2 → G2; finding 3 → G2/G4; finding 4 → superseded by D-06 with only G3 sweep rollback retained; finding 5 → G4; finding 6 → G3/G5/G6.

## Handoff order

1. Implement and gate G1; record the model API and test evidence.
2. Implement and gate G2; freeze canonical JSON/persistence behavior.
3. Implement and gate G3; do not expose command/IPC until atomic apply and sweep rollback pass.
4. Implement and gate G4; run the anti-feature payload/command audit immediately.
5. Implement and gate G5; add the two userland examples and their tests.
6. Implement and gate G6; perform the final stale-spec/anti-feature/diff audit.

No slice may reintroduce a pending session, compositor placeholder, timeout/cancel lifecycle, fallback/retry/staging parameter, or built-in matching/fill policy in the name of convenience.
