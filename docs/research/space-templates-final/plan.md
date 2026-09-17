# Plan: bindable space templates for scroll (v2 — reviewed)

> **Planning status (2026-09-17):** deepened through a pi-planning-profile run
> (`20260917162037-gd9n1c`, policy `human-decisions`, baseline
> `2879127`). Independent review by `persona-team.engineering-manager`:
> **approve-with-changes** (attestation passed); readiness: **ready**.
> Raw artifacts: [`pi-plan/spec…md`](pi-plan/spec-space-templates-reviewed-spec.md),
> [`pi-plan/graph…md`](pi-plan/graph-space-templates-execution-graph.md),
> [`pi-plan/review…md`](pi-plan/review-space-templates-engineering-manager-review.md).
> This document supersedes the v1 phase proposal (git history: `2879127`).
> **Implementation is gated on decision frontier G0** — see below.

Grounded in [research.md](research.md). Local anchors refer to this clone
(scroll is a sway fork).

## 0. What the review changed vs. v1

The independent review blocked three v1 assumptions; the deepened plan adopts
the corrections:

1. **Ownership (was: add `bound`/`live` pointers to `sway_space_container`).**
   Rejected. Durable template data must own strings/lists/geometry only —
   never view pointers or listeners (`include/sway/tree/space.h:12-53` shows
   the legacy `sway_space_view` listener ownership that must not leak into
   durable data). Bindings live in a **runtime session** object instead.
2. **Persistence lifecycle (was: "write files under XDG path").** Now fully
   specified: lazy named load + in-memory cache, cache invalidation on
   reload, atomic temp-file+rename, corrupt/unsupported files never replace
   a valid cached object, no writes outside `<config>/scroll/templates/`.
3. **Transactional restore (was: implicitly reuse `layout_space_restore`).**
   `layout_space_restore` mutates the workspace directly and has
   CLOSE/HIDE side effects (`sway/tree/space.c:202-248`); v2 adds an explicit
   **placeholder-aware path** with a rollback snapshot: pending → commit /
   cancel / timeout, with idempotent cancellation and identical rollback on
   failed commit.

## 1. Verified baseline matrix

| Behavior | Evidence | Status | Gap |
|---|---|---|---|
| Saved Spaces save/load/delete (in-memory, live views) | `include/sway/tree/space.h:39-53`, `sway/tree/space.c:327-414`, `sway/commands/space.c` | verified-complete | preserve; add regression tests |
| Space tree carries children/focus/layout/geometry/fractions | `space.h:19-30` (`sway_space_container`) | verified-complete (prior art) | factor plain-data conversion without live ownership |
| Durable slot identity + stored hints | absence verified (`research.md`) | **missing** | ST-01, G1 |
| Versioned JSON import/export + disk persistence | json-c available (`meson.build:63`); no template parser | **missing** | ST-02/03, G2 |
| Dedicated template IPC | `IPC_GET_SPACES=122`; `swaymsg/main.c:951-988`; `sway/ipc-server.c:984-996` | **missing** | ST-04, G4 |
| Pending placeholders + bind/commit/cancel | `layout_space_restore` only restores live views; NULL-view leaves silently dropped | **missing** | ST-06/07/08, G3 |
| Matching + Lua seams | `criteria_matches_view` (`sway/criteria.c:208`); `add_callback("view_map")`, view getters, JSON round-trip (`sway/lua.c:616-665,1337-1352,1820-1880,1984-2001`) | partial | template APIs + callback lifetime, G5 |
| Docs/examples | man pages exist; no template section | **missing** | ST-10, G6 |

## 2. Scope

**In scope (ST-01..ST-10):** versioned template object (name, scroller/layout
modifiers, tiling/floating trees, durable slot IDs, optional `app_id`/`class`/
`title` regex hints, focused slot); `$XDG_CONFIG_HOME/scroll/templates/<name>.json`
persistence with safe names + atomic replacement; round-trip export/import with
JSON-path errors, unknown-key tolerance, version rejection, structural limits;
dedicated `get_space_template`/`load_space_template` IPC (legacy `get_spaces`
untouched); `space_template save|load|bind|commit|cancel` commands; one pending
session per current workspace with internal-compositor placeholders (never fake
client windows); slot binding wins over `for_window`/assign; explicit
all-or-nothing commit, manual default timeout (timeout cancels, never
auto-commits); Lua APIs + binder example; tests + docs.

**Out of scope v1:** cross-reboot PID/con_id identity (proven non-durable —
`[Nama/swayrst/README.md:20-24]`); multi-output targeting; scratchpad
CLOSE/HIDE inside pending sessions (until H-01); auto-inferred durable policy
(generated `slot-N` names are structural only); Wayland session protocols.

## 3. Decisions and the G0 human gate

Implementation defaults (from research + review), changeable by the policy
owner:

- **D-01:** dedicated template IPC; `get_spaces` keeps its legacy meaning.
- **D-02:** pending → explicit commit/cancel; default no timeout; configured
  timeout only ever cancels.
- **D-03:** template-owned plain-data tree + runtime session; no `bound`/`live`
  pointers in durable data; legacy `sway_space` untouched.
- **D-04:** lazy named load, cache refresh, atomic writes, corrupt-file
  isolation; config reload never destroys an active pending session.
- **D-05:** v1 operates on the current workspace only; no silent retargeting.

**G0 — human decisions required before G1 starts:**
- **H-01:** behavior of unrelated live views at commit. Proposed default:
  preserve them; reject destructive CLOSE/HIDE modes in v1.
- **H-02:** placeholder presentation. Proposed default: internal empty
  containers, skipped by Overview/Jump/animations, no fake titled views.

## 4. Canonical JSON contract (v1)

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
  "tiling": [ /* template nodes */ ],
  "floating": [ /* template nodes */ ],
  "focused_slot": "editor"
}
```

- Inner node: `layout` + `children`, no `slot`. Leaf: exactly one non-empty
  unique `slot`, optional `view_hint` with only-present string regexes
  (`app_id`, `class`, `title`). `size` = finite non-negative
  `width_fraction`/`height_fraction`; floating nodes carry normalized
  `x/y/width/height`. No pointer, node ID, PID, fd, or listener state anywhere.
- Validation is atomic with JSON-path errors
  (`$.tiling[0].children[1].view_hint.app_id` style): root keys/types, version,
  path-safe name, unique non-empty slots, leaf/inner exclusivity, finite
  ranges, split sums ≤ 1 (documented epsilon), focused-slot membership,
  depth/node/regex limits, PCRE2 compilation. Unknown keys ignored (forward
  compatibility). json-c refcounts + partial trees freed on failure.

## 5. Runtime lifecycle + safety invariants

1. **Idle → load:** parse/validate, create current-workspace session
   (reject: no workspace/output, second pending session).
2. **Pending:** project internal placeholders; retain owned rollback snapshot;
   a view binds once; duplicate slot/view binding rejected; unrelated live
   views untouched until H-01 resolved.
3. **Bind:** `space_template bind <slot> <container|con_id>` or Lua; map-event
   helpers may *suggest* candidates via stored hints, but userland owns
   launch/retry policy; slot binding wins over assign/`for_window`.
4. **Commit:** only when every slot is bound and all bindings still reference
   mapped live views; apply tree via the new placeholder-aware layout path;
   restore `focused_slot`; arrange/publish; release rollback state.
5. **Cancel/timeout/failure:** one rollback path — restore tree, floating
   geometry, focus, scratchpad membership exactly; detach listeners; clear
   session; idempotent; persisted template untouched.
6. **Ownership:** templates own strings/lists only; sessions own transient
   view refs/listeners and detach before free; legacy `space_destroy_all`
   keeps owning only `root->spaces`.

## 6. Execution graph (G0 → G6)

```text
G0 decisions + contracts
  -> G1 plain-data model + session ownership
  -> G2 JSON/schema + persistence
  -> G3 placeholder projection + transactional binding
  -> G4 commands + IPC
  -> G5 Lua adapter + binder example
  -> G6 end-to-end validation + docs/examples
```

G2 may start once G1's data contract is stable; G4's export half after G2, its
load/bind half after G3; G5 needs G3+G4; G6 needs all. No node may silently
resolve H-01/H-02.

- **G0 — decisions.** Record H-01/H-02 choices; confirm D-01..D-05. *No
  implementation before this gate.*
- **G1 — model + ownership.** New `include/sway/tree/space_template.h`,
  `sway/tree/space_template.c`; narrowly shared helpers in `space.{h,c}`;
  root lifecycle; `sway/meson.build`. Constructors/destructors safe on
  partial-tree failure; deep copy; session cleanup on commit/cancel/unmap/
  reload/teardown; no double `wl_listener` removal; one-active-session policy.
- **G2 — JSON + persistence.** `space_template_json.c`; config-path seam in
  `sway/config.c`; fixtures + parser/persistence tests; Meson wiring
  (`sway/meson.build`, `tests/meson.build`). Semantic round-trip; stable error
  paths; save failure leaves prior file intact; reload sees replacement.
  (Fixing the legacy `get_spaces` geometry omission at
  `sway/ipc-json.c:1592-1627` becomes a **separately reviewed compatibility
  change**, not part of the template serializer.)
- **G3 — placeholders + transactions.** `space_template_session.c`; explicit
  placeholder-aware layout path (does **not** depend on
  `layout_space_container_restore_tiling` dropping NULL-view leaves);
  map/unmap + transaction seams; event-loop timeout. Acceptance: out-of-order
  binds land correctly; duplicate/stale bind fails; incomplete commit mutates
  nothing; cancel/timeout restore exactly; unmap during pending leaves no
  dangling binding.
- **G4 — commands + IPC.** `include/ipc.h` + `include/sway/ipc-json.h` +
  `sway/ipc-server.c` + `swaymsg/main.c` (+ `common/ipc-client.c` if payload
  requires); `sway/commands/space_template.c`; `sway/commands.c`; completions
  (`completions/bash/scrollmsg`, `completions/fish/scrollmsg.fish`,
  `completions/zsh/_scrollmsg`); man pages (`scroll-ipc.7.scd`,
  `scroll.5.scd`). `load_space_template` accepts `{name, workspace:"current"}`
  JSON — never arbitrary server paths. Deterministic error objects
  (`success/error/path`).
- **G5 — Lua + example.** Registry entries in `sway/lua.c`:
  `space_template_get/load/bind/commit/cancel` with `nil/false, error`
  convention. Example binder uses `add_callback("view_map", ...)`,
  view getters, `exec_process`, bounded retry, commit-when-complete; callback
  handles removed on commit/cancel; no stale references.
- **G6 — hardening + docs.** TUTORIAL/man updates; two example templates
  (dev: mail+terminal+editor per #384; writing); discussion #384 reply draft
  (answers both of its open questions: dedicated API — yes, D-01;
  all-slots-bound default — pending+commit, D-02). Full repo checks; stop at
  review-ready local commits; **no PR/merge without explicit authorization**.

## 7. Traceability

- ST-01/02/03 → G1, G2 · ST-04/05 → G4 · ST-06/07/08 → G3 (+G0 gates) ·
  ST-09 → G5 · ST-10 → G2–G6.
- Review corrections → ownership (G1), registry/reload (G2), schema error
  paths (G2), rollback (G3), IPC/session contracts (G4), Lua/map/test gates
  (G3/G5/G6).

## 8. Acceptance gates (per node)

Parser: valid nested templates, every enum, round-trip, unknown keys,
duplicate/empty slots, bad fractions/regexes, unsupported version, limits,
precise paths. Persistence: directory creation, safe names, atomic replace,
reload/cache invalidation, write-failure, corrupt-file isolation. Runtime:
out-of-order binds, duplicate/stale rejection, commit refusal, focus
restoration, timeout/cancel rollback, unmap-during-pending, H-01/H-02
behavior. IPC: exact payloads/replies, unknown names, malformed requests,
live `scrollmsg` round-trip. Lua: callback ordering/lifetime, retries,
cold-start three-slot example. Build/docs: Meson registration, build+tests,
`git diff --check`, completions, man generation.

---

*History: v1 of this plan (six phases, commit `2879127`) was the research-born
proposal; v2 incorporates the pi-planning-profile reviewed spec, execution
graph, and independent engineering-manager review. Raw artifacts exported
under [`pi-plan/`](pi-plan/); run state (untracked) in `.pi/pi-plan/runs/`.*
