# GitHub issue drafts — bindable space templates v2.1

**Target repository:** `tayiorbeii/scroll`  
**Publication status:** PUBLISHED 2026-09-17 to `tayiorbeii/scroll` (fork created and issues enabled this date):

- Issue 1: https://github.com/tayiorbeii/scroll/issues/1
- Issue 2: https://github.com/tayiorbeii/scroll/issues/2
- Issue 3: https://github.com/tayiorbeii/scroll/issues/3
- Issue 4: https://github.com/tayiorbeii/scroll/issues/4
- Issue 5: https://github.com/tayiorbeii/scroll/issues/5
- Issue 6: https://github.com/tayiorbeii/scroll/issues/6

These drafts correspond to `space-templates-execution-graph-v2.1-final.md`. They intentionally encode the final anti-features: no pending sessions, compositor placeholders, timeout/cancel/commit lifecycle, fallback/retry/staging parameters, or built-in fill/matching policy. Userland may fill slots and call the same one-shot apply again after an error.

---

## Issue 1 — Add the plain-data space-template model

**Title:** `feat(space-templates): add the plain-data template model and ownership boundary`

**Proposed labels:** `enhancement`, `space-templates`, `c`

**Body:**

### Goal

Introduce the durable v1 template model for named tiling/floating layouts without adding live-view or runtime-session ownership to saved data.

### Scope

- Add `include/sway/tree/space_template.h` and `sway/tree/space_template.c`.
- Model version/name, scroller settings, nested tiling/floating nodes, slot IDs, optional `app_id`/`class`/`title` hints, finite geometry, and focused slot.
- Add constructors, deep copy, structural validation, and partial-tree-safe destruction.
- Register sources in `sway/meson.build` and add focused model tests.

### Acceptance criteria

- Leaves contain exactly one non-empty unique slot; inner nodes contain children and no slot.
- Invalid node shapes, duplicate/empty slots, bad geometry, invalid focus, and structural-limit violations return stable structured errors.
- The model owns strings/lists/enums/geometry only; it contains no view/container pointers, listeners, PID/con_id, timer, rollback, pending, or session fields.
- Existing `sway_space` and `sway_space_view` ownership/destruction behavior remains unchanged.
- Partial construction and destruction paths pass focused tests and sanitizer-friendly checks where configured.

### Dependencies

G0 resolved. This issue unlocks JSON/persistence and atomic apply work.

---

## Issue 2 — Add canonical JSON import/export and template persistence

**Title:** `feat(space-templates): implement v1 JSON schema and atomic XDG persistence`

**Proposed labels:** `enhancement`, `space-templates`, `persistence`, `c`

**Body:**

### Goal

Persist validated templates under the existing config conventions and provide canonical JSON round-tripping without changing legacy Space serialization.

### Scope

- Add `sway/tree/space_template_json.c` and its header.
- Resolve `$XDG_CONFIG_HOME/scroll/templates/<name>.json` through the existing config-path seam.
- Add fixtures and parser/persistence tests under `tests/`.
- Implement lazy named load, cache refresh, atomic temp-file + rename, and corrupt-file isolation.

### Acceptance criteria

- Canonical v1 JSON covers `version`, `name`, `scroller`, `tiling`, `floating`, `focused_slot`, node geometry/layout, slots, and optional view hints.
- Valid nested fixtures round-trip semantically.
- Unknown keys are ignored; unsupported versions, unsafe names, bad enums/fractions/regexes, duplicate/empty slots, and limits fail with JSON-path errors.
- PCRE2 patterns compile during validation and all partially built JSON trees are freed on failure.
- Save failures preserve the prior file; corrupt/unsupported replacements do not displace a valid cache; no path traversal or writes outside the template directory are possible.
- No runtime session, staging, fallback, or retry metadata is persisted.

### Dependencies

Depends on the plain-data model (G1). Unlocks G3 and the read/export half of G4.

---

## Issue 3 — Implement single-shot atomic template apply

**Title:** `feat(space-templates): apply complete slot bindings atomically with restore-hide sweep`

**Proposed labels:** `enhancement`, `space-templates`, `compositor`, `c`, `tests`

**Body:**

### Goal

Apply a complete template in one compositor command using `validate → sweep → arrange → dissolve`.

### Scope

- Add `include/sway/tree/space_template_apply.h` and `sway/tree/space_template_apply.c`, reusing existing layout/scroller/scratchpad helpers narrowly.
- Validate current-workspace availability and a complete final slot-to-live-view binding collection before mutation.
- Sweep unrelated target-workspace views to the scratchpad with `SPACE_RESTORE_HIDE` parity.
- Arrange bound views by slot, apply geometry/scroller/focus, and release temporary bookkeeping.
- Roll back the sweep if arrange fails.

### Acceptance criteria

- Missing, duplicate, ambiguous, stale, or unmapped slots return an error naming the affected slots and leave tree, focus, geometry, and scratchpad membership unchanged.
- A successful apply never creates an empty leaf, fake client, compositor placeholder, pending session, timeout handle, cancellation path, fallback state, retry state, or staging state.
- Unselected target-workspace views move to the scratchpad; no unrelated view is closed.
- Injected arrange failure after the sweep restores exact prior membership and exposes no partial apply.
- Template/session metadata is not retained by the live tree after apply.
- The compositor does not perform automatic matching, launching, filling, retrying, or fallback selection; hints are userland data.

### Dependencies

Depends on G1 and G2. Blocks command/IPC and Lua public surfaces.

---

## Issue 4 — Expose dedicated command and IPC surfaces

**Title:** `feat(space-templates): add get/apply IPC and one-shot space_template commands`

**Proposed labels:** `enhancement`, `space-templates`, `ipc`, `c`

**Body:**

### Goal

Expose named template retrieval and one-shot apply while preserving legacy `get_spaces` behavior.

### Scope

- Update `include/ipc.h`, `include/sway/ipc-json.h`, `sway/ipc-server.c`, `swaymsg/main.c`, and `common/ipc-client.c` only if payload plumbing requires it.
- Add `sway/commands/space_template.c` and command registration.
- Update completions and `sway/scroll-ipc.7.scd` / `sway/scroll.5.scd`.
- Add IPC and command parser tests.

### Acceptance criteria

- `get_space_template` accepts a safe name and returns canonical JSON or a structured error.
- `apply_space_template` accepts the current-workspace selector and a complete final slot-to-live-view binding collection.
- Incomplete/ambiguous/stale input reports named slots and reaches the zero-mutation G3 path.
- The payload has no `fallback`, `retry`, `staging`, `pending`, `timeout`, `cancel`, or `commit` fields/options, and there are no session lifecycle commands.
- Hints/criteria matching, launching, raising, filling, and user retries remain outside IPC.
- Unknown names, malformed payloads, missing workspace, and legacy IPC regressions have deterministic tests.
- Completions and man pages document only the final one-shot API.

### Dependencies

Depends on G2 for named persistence and G3 for apply. Unlocks Lua integration.

---

## Issue 5 — Add Lua one-shot adapter and userland examples

**Title:** `feat(space-templates): expose Lua apply errors and ship fill/retry examples`

**Proposed labels:** `enhancement`, `space-templates`, `lua`, `documentation`, `tests`

**Body:**

### Goal

Expose the minimal Lua API and demonstrate that matching, placeholders, filling, and retries belong to userland scripts rather than compositor state.

### Scope

- Update `sway/lua.c` with `space_template_get` and `space_template_apply`.
- Add Lua API/error/safety tests.
- Add `examples/space-templates/fill-then-retry.lua` and `examples/space-templates/launcher-placeholders.lua` plus sample JSON templates.

### Acceptance criteria

- Successful apply returns the documented success value; incomplete/ambiguous input returns `(nil, error)` naming missing/ambiguous slots and performs no mutation.
- The Lua API adds no bind/commit/cancel/pending/timeout/fallback/retry/staging machinery and owns no compositor callback lifecycle for apply.
- The fill-then-retry example catches the structured error, launches/raises user-selected defaults for the named slots, and invokes the same one-shot apply again as ordinary Lua code.
- The launcher example uses real terminal/windows as user-owned placeholders and existing view/move primitives; no compositor fake window is created.
- Matching uses existing Lua getters/callback/process primitives; template hints are not an implicit compositor matching mode.
- Example reloads do not leak listeners or retain stale views/containers.

### Dependencies

Depends on G3 and G4. The examples may be authored as plain userland scripts, but the public API is not considered complete until the G3/G4 error contract is stable.

---

## Issue 6 — Harden, document, and prepare the local release handoff

**Title:** `docs(space-templates): document atomic apply and finalize v2.1 validation`

**Proposed labels:** `documentation`, `space-templates`, `tests`, `release`

**Body:**

### Goal

Make the implementation, documentation, examples, and tests agree on the final v2.1 contract and stop at a review-ready local state.

### Scope

- Update tutorial/man pages and example documentation.
- Add dev and writing fixtures and cross-slice regression coverage.
- Add anti-feature checks for retired lifecycle/API/payload terms.
- Register all new C sources/tests in Meson and run the normal repository gates.

### Acceptance criteria

- Documentation describes only `validate → sweep → arrange → dissolve`, complete bindings, restore-hide sweep/rollback, current-workspace scope, and userland fill-then-retry/real-window launcher patterns.
- No product docs/examples/tests claim support for pending sessions, compositor placeholders, load/bind/commit/cancel, timeouts, built-in fallback/retry, or staging parameters; historical planning text is explicitly allowlisted if needed.
- Example JSON validates; parser, apply, rollback, IPC, Lua, completion, man-generation, sanitizer/leak, focused, and full configured test gates pass.
- Legacy Space behavior remains regression-tested and no unrelated compatibility change is bundled.
- `git diff --check` is clean, generated runtime state is absent, and the final audit confirms no publication, PR, merge, or remote mutation occurred.

### Dependencies

Depends on G1–G5. This issue is the final local handoff gate; publishing to `tayiorbeii/scroll` requires separate owner confirmation.
