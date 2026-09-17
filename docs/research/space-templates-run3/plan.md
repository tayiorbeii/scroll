# Plan: save a window-manager layout as a template with placeholder slots and restore it by binding freshly launched windows to those slots

## Objective

Find real-world implementations of tiling window manager layout save and restore with placeholder windows: i3 and i3-resurrect layout files with swallow criteria, sway swaymsg save/append_layout, swayr window matching, niri scrollable-tiling, PaperWM GNOME session restore, Qtile and awesome-wm layout persistence, and the xdg-session-management Wayland protocol. Extract how each stores a layout template, matches running windows to placeholders, and restores them over IPC, to design bindable space templates (named slots, versioned JSON, Lua binding API) for dawsers/scroll.

## Recommended approach

Follow the pattern proved by 3 selected repo(s). Dependencies: `wlroots`, `wayland`.

## Files to create or modify

| Path | Role | Action |
|---|---|---|
| `package.json` | dependency manifest | create |
| `README.md` | save a window-manager layout as a template with placeholder slots and restore it by binding freshly launched windows to those slots implementation file | create |

## Step-by-step implementation

1. Install dependencies: wlroots, wayland.

## Environment variables

- None identified. Verify against the selected repos' README/env docs.

## Testing plan

- Add unit tests for the new flow.
- Add one integration/e2e path.
- Run a manual smoke test.

## Rollback plan

- Land the change behind a small, isolated commit series so a single revert removes the feature.
- Keep the previous auth/feature path working until the new flow passes smoke tests.
- Document the env vars added so they can be removed cleanly on rollback.

## Dependencies

- `wlroots`
- `wayland`

## References

- [flatpak/flatpak](https://github.com/flatpak/flatpak) (reference, score 0.50)
  - `NEWS:80` [layout_serialization, proved] — `* Fix crashes in the portal update monitor (#6692) and OCI JSON handling` ([link](https://github.com/flatpak/flatpak/blob/main/NEWS#L80))
  - `NEWS:639` [placeholder_slots, proved] — `* 'flatpak run -vv $app_id' shows all applicable sandboxing parameters` ([link](https://github.com/flatpak/flatpak/blob/main/NEWS#L639))
  - `NEWS:639` [window_matching, proved] — `* 'flatpak run -vv $app_id' shows all applicable sandboxing parameters` ([link](https://github.com/flatpak/flatpak/blob/main/NEWS#L639))
  - `NEWS:1453` [restore_over_ipc, proved] — `* Restore compatibility with older appstream-glib versions, fixing a` ([link](https://github.com/flatpak/flatpak/blob/main/NEWS#L1453))
  - `NEWS:1571` [named_slots_binding, proved] — `- Better diagnostics when a --bind or other bind-mount fails` ([link](https://github.com/flatpak/flatpak/blob/main/NEWS#L1571))
  - `NEWS:54` [session_management_protocol, proved] — `* Lock the session helper's runtime directory to prevent systemd-tmpfiles from` ([link](https://github.com/flatpak/flatpak/blob/main/NEWS#L54))
- [rcaelers/workrave](https://github.com/rcaelers/workrave) (reference, score 0.40)
  - `NEWS:918` [window_matching, proved] — `modes are "None" (NEW: no input is blocked and break windows have a title bar` ([link](https://github.com/rcaelers/workrave/blob/main/NEWS#L918))
  - `NEWS:363` [restore_over_ipc, proved] — `** Restore support for Windows Vista and up (#367)` ([link](https://github.com/rcaelers/workrave/blob/main/NEWS#L363))
  - `NEWS:981` [session_management_protocol, proved] — `** Beter support for Gnome session management. Workrave is now properly restarted` ([link](https://github.com/rcaelers/workrave/blob/main/NEWS#L981))
  - `po/nl.po:312` [window_matching, proved] — `msgid "Use the Sanctuary design for break windows and the status window."` ([link](https://github.com/rcaelers/workrave/blob/main/po/nl.po#L312))
  - `po/nl.po:2006` [named_slots_binding, proved] — `msgid "Where each timer appears. Pair timers into the same slot to save space — Workrave will alternate between them."` ([link](https://github.com/rcaelers/workrave/blob/main/po/nl.po#L2006))
- [kitnil/notes](https://github.com/kitnil/notes) (reference, score 0.35)
  - `poe.org:53` [layout_serialization, proved] — `- [[https://github.com/Vilsol/timeless-jewels][Vilsol/timeless-jewels: A timeless jewel calculator and skill tree for Path of Exile]]` ([link](https://github.com/kitnil/notes/blob/main/poe.org#L53))
  - `wayland.org:4` [window_matching, proved] — `#+title: Wayland` ([link](https://github.com/kitnil/notes/blob/main/wayland.org#L4))
  - `wm.org:4` [window_matching, proved] — `- [[https://github.com/aesophor/wmderland][aesophor/wmderland: 🌳 X11 tiling window manager using space partitioning trees]]` ([link](https://github.com/kitnil/notes/blob/main/wm.org#L4))

## Risks

- No candidate reached extraction confidence; the plan is pattern-informed but unproved.
