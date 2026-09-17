> ⚠️ **HISTORICAL — superseded by [../plan.md](../plan.md) v2.1 (2026-09-17).**
> G0 was resolved by the policy owner: restore_hide sweep (H-01), no
> compositor placeholders, single-shot atomic apply — **the pending-session,
> bind/commit/cancel, and timeout machinery below was dropped** (D-06).
> Kept verbatim as the attested review record; do not implement from this file.

# Reviewed specification: bindable space templates for scroll (#384)

## 1. Planning status and authority

- Planning run: `20260917162037-gd9n1c` (`human-decisions` policy).
- Baseline: `master` at `2879127f663852a272808bc843af6c88a7f4f0d6`.
- Product files were not edited. The only working-tree additions observed are planning/runtime artifacts (`.pi/`, `.pi-persona/`).
- Normative intent: the request, discussion #384 as recorded in `docs/research/space-templates-final/research.md`, and the decisions explicitly retained below.
- Implementation evidence: current source at the baseline revision. `research.md` is evidence, not an implementation contract; `plan.md` is a proposal and is superseded by this reviewed spec where it is more precise.
- Independent review: `persona-team.engineering-manager`, verdict **approve-with-changes**, attested in `review-space-templates-engineering-manager-review.md` with a passed `pi.persona-attestation/v1`.

## 2. Outcome

A user can persist a named, versioned workspace layout whose leaves have durable slot identities and optional matching hints; load it into the current workspace as compositor-owned placeholders; let a Lua/userland binder launch and bind live views in any order; and atomically commit, cancel, or time out the pending restore without corrupting existing Spaces, leaking view listeners, or serializing compositor pointers.

The feature is intentionally compositor-assisted, not a session manager: scroll stores layout/slot state and exposes bind/commit primitives; userland owns launch policy, retries, and application-specific matching.

## 3. Verified baseline matrix

| Requirement/behavior | Intended source | Implementation evidence | Test evidence | Classification | Remaining gap |
|---|---|---|---|---|---|
| Existing in-memory saved Spaces continue to save/load/delete | current behavior; `research.md` | `include/sway/tree/space.h:39-53`, `sway/tree/space.c:327-414`; `cmd_space` in `sway/commands/space.c` | no feature-specific space tests found | **verified-complete** for existing behavior | Preserve while adding templates; add regression coverage |
| Space tree already carries children, focus, layout, geometry, and fractions | `research.md` | `sway_space_container` in `include/sway/tree/space.h:19-30`; recursive restore in `sway/tree/space.c` | existing general test harness | **verified-complete** as reusable prior art | Factor safe plain-data conversion without changing live ownership |
| Durable slot identity and stored hints | `research.md`, #384 | no `slot`/hint fields in `space.h`; views are held through `sway_space_view` | none | **missing** | Add template-owned data and validation |
| Versioned JSON import/export and disk persistence | `plan.md` Phase 2 | `json-c` is available (`meson.build:63`), but no template parser/path | none | **missing** | Define schema, XDG path, atomic writes, reload/corruption behavior |
| Dedicated template IPC | `research.md` design lesson 5 | only `IPC_GET_SPACES=122`, `swaymsg` mapping at `swaymsg/main.c:951-988`, server case at `sway/ipc-server.c:984-996` | none | **missing** | Add exact message/payload/reply contracts |
| Pending placeholders and bind/commit/cancel | `research.md`, #384 | `layout_space_restore` directly mutates workspace and only restores live views; unbound leaves are skipped by current restore helpers | none | **missing** | Add transactional session/projection path; do not overload legacy restore blindly |
| Existing matching and Lua seams | `research.md` | `criteria_matches_view` at `sway/criteria.c:208`, view getters, `add_callback("view_map", ...)`, `workspace_set_focus`, JSON conversion and Lua registry in `sway/lua.c` | Lua tests exist but no binder tests | **partial** | Add template APIs and explicit map-event/lifetime behavior |
| Documentation and examples | `plan.md` Phase 6 | IPC and config man pages exist; no template section/example | none | **missing** | Document schema, precedence, lifecycle, limits, examples |

## 4. Scope and non-goals

### In scope (ST-01..ST-10)

- **ST-01:** v1 template object with name, version, scroller/layout modifiers, tiling/floating template trees, slot IDs, optional `app_id`/`class`/`title` regex hints, and focused slot.
- **ST-02:** durable files under `$XDG_CONFIG_HOME/scroll/templates/<name>.json` (using the repository's existing config-path conventions), with safe name validation and atomic replacement.
- **ST-03:** round-trip export/import with precise JSON-path errors, unknown-key tolerance, unsupported-version rejection, and structural limits.
- **ST-04:** dedicated `get_space_template` and `load_space_template` IPC messages; legacy `get_spaces` remains a legacy Space API.
- **ST-05:** `space_template save|load|bind|commit|cancel` commands, with `--with-hints` controlling capture of hints, not slot existence.
- **ST-06:** one pending session per compositor/current workspace in v1; placeholders are internal compositor containers/scene nodes, never fake Wayland/X11 client windows.
- **ST-07:** userland binding at map time or explicit bind; binding a slot takes precedence over `for_window`/assign placement for that view.
- **ST-08:** explicit commit is all-or-nothing; configured timeout auto-cancels, while the default is manual/no timeout. Commit restores `focused_slot`.
- **ST-09:** Lua APIs and a small binder example using existing view/map/callback/process primitives.
- **ST-10:** unit/parser, IPC, lifecycle, Lua, and end-to-end tests plus man-page/tutorial/example updates.

### Out of scope for v1

- Stable PID/con_id identity across reboot (research proves these are not durable).
- Output-targeted/multi-output templates; v1 targets the current workspace.
- Scratchpad restore modes (`CLOSE`/`HIDE`) inside a pending template session until the decision below is resolved.
- Automatic inference of durable application policy from a live view. Generated `slot-N` names are structural placeholders only; hints are opt-in and user-editable.
- Wayland session-management protocol work or a compositor fake client surface.

## 5. Decisions and gates

The following defaults align with `research.md` and are implementation defaults unless the human policy owner changes them:

- **D-01, API:** dedicated template IPC; do not change the meaning or shape of legacy `get_spaces` merely to carry templates.
- **D-02, binding:** pending → explicit commit/cancel; default timeout is manual. A positive configured timeout auto-cancels; timeout never auto-commits a partial template.
- **D-03, ownership:** use a template-owned plain-data tree plus a runtime session. Do not add `bound`/`live` pointers to durable data. If geometry helpers are shared with `sway_space_container`, ownership and destruction must remain separate from `sway_space_view` listeners.
- **D-04, persistence:** named load/get lazily parse the named file and refresh the in-memory cache; no startup scan is required. Config reload must not destroy an active pending session. A corrupt file is rejected and does not replace a previously cached valid object.
- **D-05, current workspace:** v1 starts only on `config->handler_context.workspace`; reject no-workspace/no-output and second-pending-session requests. Do not silently retarget another output.
- **Human decision gate H-01:** choose behavior for unrelated live views at commit (proposed default: preserve them and reject destructive CLOSE/HIDE modes in v1). Do not implement destructive scratchpad semantics until confirmed.
- **Human decision gate H-02:** placeholder presentation (proposed default: internal empty containers skipped by Overview/Jump and no fake titled view). The implementation must make the choice explicit in tests and docs.

## 6. Canonical data and JSON contract

A template is an immutable parsed object; a pending session owns a private working copy plus runtime bindings. A leaf has exactly one non-empty unique `slot`; an inner node has `children` and no slot. `view_hint` contains only present string regexes (`app_id`, `class`, `title`); it is not inferred unless capture requested it. No JSON field contains a pointer, node ID, PID, fd, or listener state.

Canonical v1 shape:

```json
{
  "version": 1,
  "name": "work",
  "scroller": {
    "mode": "horizontal|vertical|none",
    "insert": "before|after|beginning|end",
    "fit": "nofit|fitsplit|fitfraction",
    "focus": true,
    "center_horizontal": false,
    "center_vertical": false,
    "reorder": false
  },
  "tiling": [/* template nodes */],
  "floating": [/* template nodes */],
  "focused_slot": "editor"
}
```

A node carries `layout` when it is an inner split, `children` for inner nodes, and a `size` object with finite non-negative `width_fraction`/`height_fraction` values. Floating nodes additionally carry normalized `x`, `y`, `width`, and `height`; tiling geometry uses the existing fraction representation. A leaf may carry `slot` and `view_hint`. The serializer uses the repository's existing layout/scroller string mappings; the parser rejects unknown enum values rather than silently changing layout.

Validation is atomic: required root keys/types, version, name/path-safe identity, non-empty unique slots, leaf/inner exclusivity, finite numeric ranges, split sums ≤ 1 (with a documented epsilon), focused-slot membership, maximum depth/node/regex lengths, and PCRE2 compilation all fail with a JSON path such as `$.tiling[0].children[1].view_hint.app_id`. Unknown keys are ignored for forward compatibility. `json-c` reference counts and all partially built trees are freed on failure.

## 7. Runtime lifecycle and safety invariants

1. **Idle:** no active session; loading a template parses/validates it and creates the current-workspace session.
2. **Pending:** the session projects internal empty placeholders and retains an owned rollback snapshot. A live view can bind once; duplicate slot or duplicate view binding is rejected. Existing live views are not closed or moved to scratchpad before H-01 is resolved.
3. **Bind:** explicit `space_template bind <slot> <container/con_id>` or Lua equivalent. Map-event helpers may select candidates using stored regex hints, but userland owns launch/retry policy. Slot binding wins over ordinary assign/`for_window` placement.
4. **Commit:** reject unless every slot is bound and all bindings still refer to mapped, live views. Apply the template tree through a new placeholder-aware layout path, restore focus to `focused_slot`, arrange/dirty the workspace, publish success, then release rollback state/listeners.
5. **Cancel/timeout:** remove all placeholder state, restore the pre-session tree, floating geometry, focus, and scratchpad membership exactly, clear callback/session state, and leave the persisted template untouched. Cancellation is idempotent; failed commit must take the same rollback path.
6. **Lifetime:** template files and parsed template objects own strings/lists only. Runtime sessions own transient view references/listeners and detach them before freeing. Legacy `space_destroy_all` remains responsible only for `root->spaces`.

## 8. Public surfaces

- Commands: `space_template save <name> [--with-hints]`, `load <name>`, `bind <slot> <container|con_id>`, `commit <name>`, `cancel <name>`. Return command errors for invalid names, no current workspace, duplicate session, missing slots, malformed files, incomplete commit, or stale containers.
- IPC: allocate unused scroll-specific enum values. `get_space_template` takes a UTF-8 template name and replies with the canonical object or `{ "success": false, "error": "...", "path": "..." }`. `load_space_template` takes `{ "name": "...", "workspace": "current" }` and replies with a success/error object; it does not accept arbitrary server file paths. Add mapping in `swaymsg/main.c`, server switch cases, public declarations, raw/pretty behavior, and man-page entries.
- Lua: `space_template_get(name)`, `space_template_load(name)`, `space_template_bind(slot, container)`, `space_template_commit(name)`, `space_template_cancel(name)`. Return `nil/false,error` consistently on failure. The example uses `add_callback("view_map", ...)`, `view_get_app_id/class/title`, `view_get_container`, `exec_process`, and existing JSON conversions; callback handles are removed on commit/cancel.

## 9. Acceptance and validation

- Parser tests cover valid nested templates, every layout/scroller enum, geometry/focus round-trip, unknown keys, duplicate/empty slots, invalid fractions, bad regexes, unsupported versions, depth/size limits, and precise error paths.
- Persistence tests cover directory creation, safe names, atomic replacement, reload/cache invalidation, permission/write failures, corrupt-file isolation, and no file outside the template directory.
- Runtime tests cover two slots launched out of order, explicit binds, duplicate/stale binds, commit refusal, focus restoration, timeout/cancel rollback, unmap during pending, existing-view preservation/H-01, and Overview/Jump/H-02 behavior.
- IPC tests exercise exact request payloads, reply/error JSON, unknown names, malformed load requests, and `scrollmsg -t get_space_template`/load against a live test compositor.
- Lua tests exercise callback ordering/lifetime, binder retries, JSON API shape, and cold-start restoration of the three-slot example.
- Build/docs gates: add all new C sources and tests to Meson; run the repository's configured build/test targets, `git diff --check`, and man-page generation; verify Bash/Fish/Zsh completions and tutorial/example snippets.
