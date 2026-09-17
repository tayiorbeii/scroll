# Plan: bindable space templates for scroll (v2.1 — post-G0 refinement)

> **Planning status (2026-09-17):** deepened through a pi-planning-profile run
> (`20260917162037-gd9n1c`, policy `human-decisions`, baseline `2879127`).
> Independent review by `persona-team.engineering-manager`: **approve-with-changes**
> (attestation passed); readiness: **ready**. Raw artifacts:
> [`pi-plan/spec…md`](pi-plan/spec-space-templates-reviewed-spec.md),
> [`pi-plan/graph…md`](pi-plan/graph-space-templates-execution-graph.md),
> [`pi-plan/review…md`](pi-plan/review-space-templates-engineering-manager-review.md).
>
> **G0 resolved 2026-09-17 (Taylor, human policy owner).** H-01: restore_hide.
> H-02 + **v2.1 pivot (D-06): userland placeholders, atomic apply — no
> compositor placeholder windows, no pending sessions/timeouts/cancel** (see
> §3; supersedes the reviewed spec §7 pending-session machinery and
> simplifies review findings 1 and 4). **G1 is the next frontier.**
> v1 phase proposal: git history `2879127`; v2 reviewed plan: `9cd6a59`.

Grounded in [research.md](research.md). Local anchors refer to this clone
(scroll is a sway fork).

## 0. Design evolution

- **v1 (research-born):** extend `sway_space_container` with slots + `bound`
  pointers; reuse `layout_space_restore`.
- **v2 (reviewed):** template-owned plain data + runtime pending session with
  internal placeholders, bind/commit/cancel, timeouts. Review blocked three
  seams (ownership, persistence lifecycle, transactional rollback) — resolved
  in spec §6–7.
- **v2.1 (owner refinement, current):** the pending session and internal
  placeholders are **gone**. The user/script owns completeness: **apply
  fails unless every slot is already mapped to a live view**. Users who want
  visible placeholders implement them in Lua — e.g. terminal windows running
  a launcher script; selecting an application closes the placeholder and
  swaps the real app into that slot using existing view/move primitives.
  This is exactly the division of labor #384 proposed ("the script would own
  the matching policy... then ask Scroll to apply the template"), and it
  eliminates the null-pointer hazard *by construction*: the compositor never
  materializes a container without a view, before, during, or after apply.

## 1. Verified baseline matrix

| Behavior | Evidence | Status | Gap |
|---|---|---|---|
| Saved Spaces save/load/delete (in-memory, live views) | `include/sway/tree/space.h:39-53`, `sway/tree/space.c:327-414`, `sway/commands/space.c` | verified-complete | preserve; add regression tests |
| Space tree carries children/focus/layout/geometry/fractions | `space.h:19-30` (`sway_space_container`) | verified-complete (prior art) | factor plain-data conversion without live ownership |
| Durable slot identity + stored hints | absence verified (`research.md`) | **missing** | ST-01, G1 |
| Versioned JSON import/export + disk persistence | json-c available (`meson.build:63`); no template parser | **missing** | ST-02/03, G2 |
| Dedicated template IPC | `IPC_GET_SPACES=122`; `swaymsg/main.c:951-988`; `sway/ipc-server.c:984-996` | **missing** | ST-04, G4 |
| Atomic slot-keyed apply (validate → sweep → arrange) | `layout_space_restore` restores live views by saved shape only; no slot keying, no completeness validation | **missing** | ST-06/07/08, G3 |
| Matching + Lua seams | `criteria_matches_view` (`sway/criteria.c:208`); `add_callback("view_map")`, view getters, `exec_process`, JSON round-trip (`sway/lua.c:616-665,1337-1352,1820-1880,1984-2001`) | partial | apply APIs + binder example, G5 |
| Docs/examples | man pages exist; no template section | **missing** | ST-10, G6 |

## 2. Scope

**In scope (ST-01..ST-10, as revised by v2.1):** versioned template object
(name, scroller/layout modifiers, tiling/floating trees, durable slot IDs,
optional `app_id`/`class`/`title` regex hints, focused slot);
`$XDG_CONFIG_HOME/scroll/templates/<name>.json` persistence with safe names +
atomic replacement; round-trip export/import with JSON-path errors,
unknown-key tolerance, version rejection, structural limits; dedicated
`get_space_template` / `apply_space_template` IPC (legacy `get_spaces`
untouched); `space_template save|apply` commands; **atomic apply that fails
unless every slot maps to a mapped live view** (completeness is the caller's
responsibility — `#384`: "loading should require every required slot to be
bound by default"); scratchpad sweep of unrelated target-workspace views at
apply (`restore_hide` semantics, H-01); userland placeholder pattern (Lua
launcher terminals) as the supported way to stage slots; Lua APIs + binder/
launcher example; tests + docs.

**Out of scope v1:** compositor placeholder windows or empty compositor
containers (H-02, revised); pending sessions, timeouts, cancel machinery
(D-06 — the apply is a single atomic command); scratchpad **CLOSE** (never
close unrelated windows); cross-reboot PID/con_id identity; multi-output
targeting; auto-inferred durable policy (generated `slot-N` names are
structural only); Wayland session protocols; compositor-side slot identity
*after* apply (slot replacement post-apply is userland via existing
primitives).

## 3. Decisions (G0 — resolved 2026-09-17)

- **D-01 (unchanged):** dedicated template IPC; `get_spaces` keeps its legacy
  meaning.
- **D-02 (refined):** all-or-nothing means **apply fails otherwise** — the
  user/script ensures each container is mapped before applying, and the
  **Lua callback is where that failure surfaces**: `apply` returns
  `(nil, error)` naming the missing/ambiguous slots, so the user's callback
  can stage more windows and retry. The compositor's own validation is the
  zero-mutation backstop, not the UX. There is no pending state to commit
  out of. Because staging is userland, the user also defines the **fallback
  for unmapped slots**: launch a default program (e.g. a terminal running a
  launcher) or raise an existing window to occupy the slot, then retry —
  the compositor has no built-in fallback policy.
- **D-03 (unchanged):** template-owned plain data only — no view pointers or
  listeners in durable state. (With sessions gone, this is now trivially true:
  apply consumes the template and produces an ordinary workspace tree.)
- **D-04 (unchanged):** lazy named load, cache refresh, atomic writes,
  corrupt-file isolation.
- **D-05 (unchanged):** v1 operates on the current workspace only.
- **D-06 (new, v2.1 pivot):** **no pending sessions / placeholders / timeouts
  / cancel in v1.** Apply = validate → sweep (H-01) → arrange → focus, in one
  command; validation failure aborts before any mutation.
- **H-01 (resolved):** views on the target workspace not bound to any slot are
  moved to the **scratchpad** at apply (`restore_hide` parity,
  `SPACE_RESTORE_HIDE`, `include/sway/tree/space.h:8-11`). CLOSE remains out
  of scope. If arrange fails after the sweep, the sweep is undone
  (single-command rollback window).
- **H-02 (resolved, revised):** **no compositor placeholders — show nothing.**
  Unfilled slots cannot exist in the live tree because apply refuses them.
  Users stage slots with their own placeholder windows (launcher-terminal
  pattern, §5), which also sidesteps the null-pointer risk entirely: every
  container in the tree always holds a real view.

## 4. Canonical JSON contract (v1)

Unchanged from v2 (this is storage, not runtime): see
[`pi-plan/spec…md`](pi-plan/spec-space-templates-reviewed-spec.md) §6.
Summary: `version`, `name`, `scroller` (mode/insert/fit/focus/center/reorder),
`tiling`, `floating`, `focused_slot`; inner nodes carry `layout` + `children`,
leaves carry exactly one non-empty unique `slot` + optional `view_hint` regexes
(`app_id`/`class`/`title`) + finite `width_fraction`/`height_fraction`;
floating nodes carry normalized `x/y/width/height`. No pointers, IDs, PIDs, or
listener state. Atomic validation with JSON-path errors, unknown keys ignored,
PCRE2 compiled at validation, json-c refcounts freed on failure.

## 5. Runtime model (v2.1)

1. **Save/export:** snapshot or author a template; write via atomic replace.
2. **Apply (the only mutating operation):**
   `space_template apply <name> [--mappings <json>|--auto <criteria>]`
   - **Validate first:** every slot resolves to exactly one mapped live view
     (explicit `slot→con_id` mappings, or caller-supplied criteria evaluated
     via `criteria_matches_view`); fractions/geometry sane; workspace present.
     Any failure → error, **zero mutation**. The error names the offending
     slots so a Lua callback can stage more windows and retry
     (D-02 enforcement point).
   - **Sweep (H-01):** target-workspace views not bound to any slot →
     scratchpad (`restore_hide` parity).
   - **Arrange:** build the tree from the bound views per template
     (generalization of the existing `space_load` restore path, keyed by slot
     instead of saved order); apply fractions/scroller state; floating
     geometry; focus `focused_slot`.
   - **Dissolve:** the template object remains a cached file; no template
     state stays attached to the tree. A view that closes later is an
     ordinary view-close.
3. **Userland placeholder + fallback patterns (supported examples, ship in
   G5/G6):** a Lua script stages slots by opening e.g. terminal windows
   running a launcher; applies the template binding those terminals to
   slots; on selection the script closes the placeholder terminal, launches
   the chosen app, and swaps it into the slot with existing move primitives
   (`add_callback("view_map")`, `view_get_app_id/class/title`,
   `exec_process`, container move via `command`). **Per-slot user-defined
   fallback** completes the contract: when apply reports unmapped slots, the
   script launches a default program or raises an existing window for each,
   then re-applies — the fallback permanently occupies the slot until
   swapped. Matching policy, fallbacks, retries, and sequencing stay
   userland — matching #384's intent and the i3-resurrect role split.
4. **Ownership:** templates own strings/lists only; apply reads them and
   produces ordinary tree state. Legacy `sway_space` untouched;
   `space_destroy_all` keeps owning only `root->spaces`.

## 6. Execution graph (revised: G0 → G6)

```text
G0 decisions + contracts        [RESOLVED 2026-09-17 — this document]
  -> G1 plain-data template model
  -> G2 JSON/schema + persistence
  -> G3 atomic apply engine (validate → sweep → arrange → dissolve)
  -> G4 commands + dedicated IPC
  -> G5 Lua adapter + launcher/placeholder example
  -> G6 end-to-end validation + docs/examples
```

G2 may start once G1's data contract is stable; G4's export half after G2, its
apply half after G3; G5 needs G3+G4; G6 needs all.

- **G0 — decisions.** ✅ Resolved: D-01..D-06, H-01, H-02 as recorded in §3.
- **G1 — plain-data template model.** `include/sway/tree/space_template.h`,
  `sway/tree/space_template.c`; narrowly shared geometry helpers with
  `space.{h,c}`; `sway/meson.build`. Partial-tree-safe ctors/dtors; deep copy;
  no view pointers anywhere in durable state. *(Review finding 1 satisfied —
  now simpler with no session object.)*
- **G2 — JSON + persistence.** `space_template_json.c`; config-path seam in
  `sway/config.c`; fixtures + parser/persistence tests. Lazy named load, cache
  refresh, atomic temp-file+rename, corrupt-file isolation, no writes outside
  `<config>/scroll/templates/`. (Fixing legacy `get_spaces` geometry omission
  at `sway/ipc-json.c:1592-1627` = separately reviewed compatibility change.)
- **G3 — atomic apply engine.** Slot-keyed generalization of the restore path
  in `sway/tree/space.c:202-248` / layout helpers: validate-mappings-first
  (fail = zero mutation), scratchpad sweep + undo-on-failure, arrange from
  bound views, fractions/scroller/floating/focus, then dissolve. Acceptance:
  incomplete mapping fails without mutation; out-of-order staging irrelevant
  (apply is one shot); sweep is restore_hide-exact; failure after sweep
  restores membership; no empty leaf containers ever exist.
- **G4 — commands + IPC.** `include/ipc.h`, `include/sway/ipc-json.h`,
  `sway/ipc-server.c`, `swaymsg/main.c` (+`common/ipc-client.c` if needed);
  `sway/commands/space_template.c` (`save <name> [--with-hints]`,
  `apply <name> [mappings…]`); completions (`bash`, `fish`, `zsh`); man pages
  (`scroll-ipc.7.scd`, `scroll.5.scd`). `apply_space_template` IPC accepts
  `{name, workspace:"current", mappings:[{slot, con_id}|{slot, criteria}]}`
  and returns success/error — never arbitrary server paths.
- **G5 — Lua adapter + example.** `space_template_get(name)`,
  `space_template_apply(name, mappings)` with `nil/false, error` convention;
  apply errors enumerate missing/ambiguous slots so callbacks can retry
  (D-02). Example scripts: (a) simple binder — a `view_map` callback
  accumulates slot→view mappings and calls apply **only once complete**;
  (b) launcher placeholders + fallback — apply reports unmapped slots → the
  script launches/raises a user-defined default program per slot, re-applies,
  and swaps fallbacks for real apps on selection (H-02 pattern end to end).
- **G6 — hardening + docs.** TUTORIAL/man updates (template anatomy, criteria,
  apply contract, restore_hide sweep, userland placeholder pattern); two
  example templates (dev: mail+terminal+editor per #384; writing); discussion
  #384 reply (answers both open questions: dedicated API — D-01; all-slots-
  required default — D-02/D-06, now stricter and simpler than proposed).
  Full repo checks; review-ready local commits; **no PR/merge without
  explicit authorization**.

## 7. Traceability

- ST-01/02/03 → G1, G2 · ST-04/05 → G4 · ST-06/07/08 (revised: atomic apply,
  no placeholders/sessions) → G3, G4 · ST-09 → G5 · ST-10 → G2–G6.
- Review findings: 1 (ownership) → G1 (simplified by D-06); 2 (persistence) →
  G2; 3 (JSON contract) → G2/G4; 4 (pending transaction) → **superseded by
  D-06** (residual: single-command sweep rollback in G3); 5 (IPC contract) →
  G4; 6 (Lua/event/test gates) → G5/G6.

## 8. Acceptance gates

Parser/persistence: unchanged from v2 (round-trip, unknown keys, duplicate/
empty slots, bad fractions/regexes, version rejection, limits, precise error
paths, atomic writes, corrupt-file isolation). Apply (new): incomplete/ambiguous
mappings fail with zero mutation; sweep is restore_hide-exact and undone on
post-sweep failure; applied tree matches fractions/scroller/floating/focus;
no empty containers at any point; second apply during apply is serialized.
IPC: exact payloads/replies, unknown names, malformed requests, live
`scrollmsg` round-trip. Lua: API error convention, launcher example cold-start,
no callback leaks. Build/docs: Meson registration, build+tests,
`git diff --check`, completions, man generation.

---

*History: v1 (six phases, `2879127`) → v2 (reviewed spec/graph/review,
`9cd6a59`) → **v2.1 (this document): G0 resolved — restore_hide sweep (H-01),
userland placeholders + atomic apply (H-02/D-06), pending-session machinery
dropped.** Raw reviewed artifacts under [`pi-plan/`](pi-plan/); run state
(untracked) in `.pi/pi-plan/runs/`.*
