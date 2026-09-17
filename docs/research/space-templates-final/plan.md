# Plan: bindable space templates for scroll

Grounded in [research.md](research.md). Phase order is dependency-ordered;
each phase is independently testable and small. Local anchors refer to this
clone (sway tree layout applies — scroll is a sway fork).

## Phase 1 — Extend the data model (slots + hints)

1. `include/sway/tree/space.h`:
   - add to `struct sway_space_container`:
     `char *slot;` (durable leaf identity, NULL for inner nodes) and
     `struct { char *app_id; char *class; char *title; } hint;`
     (optional regex-able view hints, i3 `Match`-style
     `[i3/src/load_layout.c:303-311]`).
   - add `bool bound` / `struct sway_container *live;` so a template leaf can
     point at a bound live container during a restore session.
2. `sway/tree/space.c`:
   - `space_container_create()` copies slot/hint from a live container
     (initially NULL — slots are authored, not auto-derived, per i3's stance
     of not guessing criteria `[i3/docs/layout-saving:24-30]`).
   - add `space_container_set_slot()`, `space_container_set_hint()`.
3. **Templates become a first-class object, not just a saved Space:**
   add `struct space_template` (versioned wrapper around `sway_space`) or a
   `template` flag on `sway_space`; templates are loaded from JSON and hold
   *unbound* leaves, saved Spaces keep holding live views.
   - new file `sway/tree/space_template.c` + header; keep `space.c` untouched
     except for accessors it already exposes.

**Test:** unit-ish via existing harness (`tests/`, pytest.ini) — build a
template from a fixture JSON, assert tree/fractions/slots survive round-trip.

## Phase 2 — Versioned JSON serialization

1. Exporter: `ipc_json_describe_space_template()` alongside the existing
   `[scroll/sway/ipc-json.c:1592-1648]` descriptors — emits the #384 shape:
   `version`, `name`, workspace scroller mode, tiling/floating trees with
   `slot`, `size.width_fraction/height_fraction`, `view_hint`, `focused_slot`.
   Reuse `ipc_json_describe_space_container()` and add the missing geometry
   fields (the current live-space JSON omits them — a bug worth fixing
   regardless, `[scroll/sway/ipc-json.c:1592-1627]`).
2. Importer: `space_template_load_json()` using json-c (already a dependency,
   `[scroll/sway/ipc-json.c]` includes) — mirror i3's parser structure
   (`[i3/src/load_layout.c:24,57,219]`): track current node, reject empty
   slot definitions, tolerate unknown keys for forward compatibility,
   validate fractions sum ≤ 1 per split.
3. Persist location: `$XDG_CONFIG_HOME/scroll/templates/<name>.json`
   (no state written until this phase lands).

**Test:** round-trip JSON → template → JSON is identity; malformed files
rejected with the offending path in the error string.

## Phase 3 — IPC + commands

1. New IPC messages (mirroring `IPC_GET_SPACES` plumbing):
   - `scrollmsg -t get_space_template <name>` → JSON export;
   - `scrollmsg -t load_space_template` ← JSON body (or command-style
     `space_template load <file>`), keeping `get_spaces` unchanged —
     matches #384's preference for a dedicated API over overloading
     `get_spaces`.
   - touch points: `[scroll/swaymsg/main.c:983-997]` (message-type dispatch),
     `sway/ipc-server.c` (handler + reply), `include/sway/ipc-server.h`
     (new `IPC_GET_SPACE_TEMPLATE`/`IPC_LOAD_SPACE_TEMPLATE` codes — pick
     unused ints), `completions/` for the CLI.
2. Config commands in `sway/commands/` (near the existing `space` command):
   - `space_template save <name> [--with-hints]` — snapshots current workspace
     into a template (leaves get slot names only if `--with-hints` given,
     else `slot-<n>`);
   - `space_template load <name> [restore]` — same LOAD/CLOSE/HIDE semantics
     as `space_load` `[scroll/sway/tree/space.c:399-406]`, but binds
     placeholders instead of reattaching views.
3. Document in `scroll-ipc.7.scd` / `scroll.5.scd` (man pages in-repo).

**Test:** `scrollmsg -t get_space_template work` on a live session; load
against a fresh config; CLI completion.

## Phase 4 — Binding engine (placeholders at map time)

1. Placeholder behavior: a template leaf with `slot` but no bound view
   renders as an empty container (optionally a titled empty view visual —
   i3 uses real X11 windows `[i3/docs/layout-saving:33-38]`; on Wayland a
   compositor-internal container/scene node is the equivalent and avoids
   fake-map).
2. `bind_slot` primitive (command + Lua):
   - `space_template bind <slot> [criteria|con_id]` — attaches the container
     under the slot, applying stored fractions via the same path
     `layout_space_restore()` uses (`sway/tree/layout.c`).
   - matching helper: reuse `criteria_matches_view()`
     `[scroll/include/sway/criteria.h:75-76]` — hints compile to the existing
     pcre2 patterns; no new matcher.
3. All-or-nothing apply (per #384): `space_template load` enters
   *pending* state; a `--commit`/timeout auto-cancels unbound slots
   (configurable `space_template_bind_timeout`, default: manual commit).
   Document the precedence decision: **slot binding wins over for_window
   assign** — mirror i3 `[i3/docs/layout-saving:41-44]` and note it in the
   man page.
4. Focus restoration: on commit, focus `focused_slot`
   (`workspace_set_focus` Lua binding already exists
   `[scroll/sway/lua.c:2041]`).

**Test:** two-slot template; launch apps out of order; scripted binds;
verify fractions/focus after commit; cancel path restores prior state.

## Phase 5 — Lua API + user-side binder example

1. New Lua bindings in `[scroll/sway/lua.c:1984-2066]` registry:
   - `space_template_get(name)`, `space_template_load(name)`,
     `space_template_bind(slot, container)`,
     `space_template_commit(name)`, `space_template_cancel(name)`.
2. Ship an example script (`examples/` or `docs/`) that reproduces the
   i3-resurrect workflow in ~40 lines of Lua: listen via
   `add_callback`/`view_mapped`, match `view_get_app_id()`/`view_get_title()`
   against the template's `view_hint`s, launch missing apps with
   `exec_process`, retry with backoff, commit when all slots bound —
   demonstrating #384's "application/session logic outside the compositor"
   `[scroll/sway/lua.c:1993-2001]` for the needed primitives.
3. `scrollmsg --lua_repl` is the interactive test harness
   (README feature list).

## Phase 6 — Docs, example templates, upstreaming

1. `TUTORIAL.md` section + man pages: template anatomy, criteria reference,
   precedence, timeout/commit semantics.
2. Ship 2 example templates (dev "mail+terminal+editor" from #384; writing).
3. Prepare the discussion #384 reply: link artifacts, note the two open
   questions #384 asks (dedicated API vs `get_spaces` — resolved: dedicated;
   all-slots-bound default — resolved: pending+commit).
4. Local commits per phase; **no PR without Taylor's explicit request**.

## Risks / open questions

- Placeholder visuals: empty-container vs fake-window choice affects
  animations/overview (`Overview`/`Jump` modes must skip or render
  placeholders deliberately).
- Fraction math when a template is loaded on a differently-sized output —
  fractions are relative so this should be free; verify with mixed scales
  (content_scale is per-view `[scroll/include/sway/tree/space.h:16]`).
- Multi-output: template binds to *current* workspace in v1 (matches
  `space_save/load` semantics); output-targeted restore is future work
  (swayrst's workspace→display mapping is prior art).
- Scratchpad/HIDE restore mode interplay with pending slots needs a decision
  before Phase 4.
