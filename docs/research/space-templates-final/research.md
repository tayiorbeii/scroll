# Research: bindable space templates for scroll (dawsers/scroll discussion #384)

**Goal:** Find how other tiling window managers / Wayland ecosystems implement
layout save & restore with placeholder windows, and extract implementable
patterns for scroll's proposed "bindable space templates" feature.

**Request under study:** [discussion #384 — "Partial persistent sessions through bindable space templates"](https://github.com/dawsers/scroll/discussions/384)
(A durable Space template = saved layout whose views are replaced by *named
slots*; a user script re-launches apps and binds live windows to slots.)

## Method

- `research-orchestrator` planning kit (automated candidate discovery + proof):
  3 octocode-mode runs (`run_tiling_managers_ecosystems_real_world_1789617779706`,
  `run_tiling_manager_save_window_1789617857250`,
  hints-based `wm-space-template-restore`). Automated discovery was **noisy for
  this niche domain** (generic keyword matches selected `flatpak/flatpak`,
  `kitnil/notes`, etc. — see the kept `space-templates-run*/` dirs). The kit's
  own guidance for this case: inject known entities and drive
  `explainRepoPattern`/`scoreRepos` directly, which was done for 6 target repos.
- Exact-evidence reads via octocode CLI (`ghSearch` code search +
  line-anchored `ghGetFileContent`).
- Local exact-evidence reads in the scroll clone itself (it is a sway fork —
  the most direct prior art is the code scroll already contains).

Evidence anchors: `[repo/path:line]` (remote) or `[scroll/path:line]` (local clone).

## Landscape

| Project | Model | Placeholders? | Matching criteria | Restore trigger | Reusable lesson |
|---|---|---|---|---|---|
| i3 (X11) | Compositor-native layout files | **Yes** — real X11 placeholder windows with visible swallow criteria | `class`, `instance`, `title`, `id`, `dock`, `restart_mode` (PCRE2 regexes) | `append_layout <file>` IPC command | The canonical design; placeholder+swallow works, but criteria are *commented out* in saved output — human or script must choose them |
| sway (Wayland) | — | — | — | — | **Never implemented** i3's layout save/restore; scroll inherits nothing for free |
| i3-resurrect | Userland Python tool on top of i3 IPC | Reuses i3's swallows | criteria from live `window_properties` (class/instance/title), regex-escaped; per-class overrides in config | `append_layout` + app relaunch by the tool | Automates exactly the "bind" step the discussion wants user scripts to own |
| swayrst | Userland save/load of sway tree JSON | No | workspace→output mapping + window lists; **cannot identify windows across reboots** (explicit README limitation) | external tool (kanshi/shikane hook) | Documents the exact pain named slots solve: *node IDs reset after a reboot* |
| niri (Wayland, scrollable tiling) | — | — | window rules match on `app-id`/`title` at spawn | — | No session restore either; its window-rule matcher is the same criteria model scroll already has |
| PaperWM / GNOME | Delegated to GNOME Shell session management | — | — | gnome-session restart | Session persistence can live outside the WM — supports the discussion's "keep app logic in scripts" stance |
| xdg-session-management (Wayland) | Protocol draft | — | — | — | **Not merged** — absent from `wayland-protocols/staging` today; a compositor-side partial solution like #384 is the realistic near-term path |

## Key findings (with evidence)

### 1. i3's placeholder + swallow design is the proven pattern

- Layout files are JSON trees; leaf nodes carry a `swallows` array of match
  criteria; `append_layout <file>` creates placeholder containers/windows and
  the tree is filled as matching windows appear.
  `[i3/docs/layout-saving:9-26]` — "When an application opens a window that
  matches the specified swallow criteria, it will be placed in the
  corresponding placeholder window."
- Parsing: `load_layout.c` reads `swallows` entries into `Match` structs with
  `regex_new`; an **empty swallow definition is rejected as invalid**
  `[i3/src/load_layout.c:24,57,219-223,303]`.
- Adoption happens at window-map time:
  `[i3/src/manage.c:279]` — `con_for_window(search_at, cwindow, &match)`
  "See if any container swallows this new window"; swallowing takes
  precedence over assignment rules `[i3/docs/layout-saving:41-44]`.
- Saved output includes criteria but **commented out** — i3 deliberately
  avoids auto-generating matching policy `[i3/docs/layout-saving:24-30]`.
- i3 placeholders are actual X11 windows (a trick unavailable on Wayland —
  on Wayland a placeholder must be a compositor-internal container/visual,
  which scroll's saved-space containers already are).

### 2. The userland-tool generation (i3-resurrect, swayrst) shows both halves of the problem

- i3-resurrect derives swallow criteria from live window properties and
  regex-escapes them, with per-window-class config overrides:
  `[JonnyHaystack/i3-resurrect/i3_resurrect/treeutils.py:54-68]`.
- swayrst's README states the core limitation that named slots eliminate:
  `[Nama/swayrst/README.md:20-24]` — "There is no known way (to me) to
  identify the exact same windows after a reboot... node IDs reset after a
  reboot." A durable `slot` name is the fix; `con_id`/PID are not stable.

### 3. scroll's existing Spaces are 80% of the template data model already

Local anchors (this clone):

- The saved-space model already stores the full geometry a template needs:
  `[scroll/include/sway/tree/space.h:13-32]` —
  `struct sway_space_container { children; view; focused_inactive;
  width_fraction; height_fraction; layout; x, y, width, height; }` and
  `struct sway_space { name; tiling; floating; focused; }`.
- `space_save()` snapshots a workspace's tiling+floating into that model;
  `space_load()` restores via `layout_space_restore()` with
  LOAD/CLOSE/HIDE policies `[scroll/sway/tree/space.c:366-406]`.
- **Gaps vs. #384:** no *slot* identity on leaf containers; no stored
  *view hints* (app_id/class/title live only in the referenced view, which is
  what makes spaces transient); no disk persistence; no versioned
  export/import.
- IPC `get_spaces` exists (`IPC_GET_SPACES`, `[scroll/swaymsg/main.c:983-984]`)
  but its JSON **omits the geometry**: `ipc_json_describe_space_container()`
  emits only `children` + live `view` metadata
  `[scroll/sway/ipc-json.c:1592-1627]`, and `ipc_json_describe_space()` only
  `name`/`tiling`/`floating` `[scroll/sway/ipc-json.c:1629-1648]`. This is
  exactly the "not enough state to faithfully export its layout" complaint in
  #384.
- The matcher needed for slots already exists: pcre2-based
  `criteria_matches_view()` with `app_id`, `class`, `instance`, `title`,
  `shell`, `tag` patterns `[scroll/include/sway/criteria.h:39-77]`.
- The Lua API already exposes everything a user-side binder needs except slot
  identity: `view_get_app_id/class/title/pid`, `container_get_children`,
  `container_get_width_fraction`, `workspace_get_tiling`, `add_callback`,
  `ipc_send`, `json_to_lua`/`lua_to_json`
  `[scroll/sway/lua.c:1984-2066]`.

### 4. Nobody upstream will hand this to Wayland compositors soon

`wayland-protocols` staging contains no session-management protocol
(freedesktop GitLab tree query, 2026-09: `staging/` lists
alpha-modifier…ext-data-control etc., no session-management). niri has no
session restore (only D-Bus app-id introspection,
`[niri-wm/niri/src/dbus/gnome_shell_introspect.rs]`). PaperWM delegates to
GNOME Shell session modes. Discussion #384's premise — a compositor-local,
user-scripted partial restore — is the only realistic path today.

## Design lessons for scroll

1. **Templates = saved Spaces + slot names + view hints + disk format.**
   Extend `sway_space_container` with `char *slot` and stored view-hint
   patterns rather than inventing a parallel tree type; i3's layout file and
   scroll's space model are nearly isomorphic (children/fractions/layout ≈
   nodes/percent/layout).
2. **Version the file format from day one** (`"version": 1`), as #384's
   sketch already does. i3's format evolved by tolerating unknown keys.
3. **Default to all-or-nothing binding.** #384 proposes requiring every slot
   bound before apply; i3 instead allows indefinite placeholders. For scroll,
   a *timed or explicit-commit* model avoids both silent partial restores and
   forever-placeholders: keep placeholders visible until `apply`/`cancel`.
4. **Keep matching policy in user scripts** (i3-resurrect role): compositor
   stores hints; the Lua binder owns launching, retries, and match
   precedence. The compositor should expose `bind(slot, container)` and
   `view_mapped` events, not heuristics.
5. **Dedicated template IPC beats overloading `get_spaces`** (agreeing with
   #384's own conclusion): `get_spaces` describes live saved spaces; add
   `get_space_template` / `load_space_template` (or `space_template_save` /
   `space_template_load` commands) with the versioned JSON.
6. **Precedence rule needed:** when a new view matches both a slot and a
   `for_window`/assign rule, define which wins (i3: placeholder wins,
   `[i3/docs/layout-saving:41-44]`).

Full implementation plan: see [plan.md](plan.md). Checklist:
[implementation-checklist.md](implementation-checklist.md). Raw evidence:
[evidence.json](evidence.json). Automated kit run attempts are preserved in
`../space-templates/`, `../space-templates-run2/`, `../space-templates-run3/`.

## Addendum (2026-09-17): design-lesson outcomes

G0 was resolved by the policy owner after this research was written; where the
lessons above proposed options, the outcomes are:

- Lesson 3 (all-or-nothing binding): adopted, and strengthened — **apply fails
  unless every slot is mapped; there is no pending state, timeout, or commit
  step at all** (plan.md D-02/D-06). The "timed or explicit-commit" option was
  rejected.
- Lesson 4 (matching policy in user scripts): adopted; per-slot fallbacks
  (launch a default program / raise an existing window) are also userland.
- Lesson 5 (dedicated template IPC): adopted (D-01); the API is single-shot —
  no fallback/retry/staging parameters (anti-feature guard).
- Lesson 6 (precedence): subsumed — since apply only accepts explicit
  mappings for *all* slots and sweeps the rest, there is no concurrent-match
  ambiguity to arbitrate.

Authoritative decisions: [plan.md §3](plan.md). Historical reviewed artifacts
under [pi-plan/](pi-plan/) carry superseded banners where machinery changed.
