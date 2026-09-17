# Research: save a window-manager layout as a template with placeholder slots and restore it by binding freshly launched windows to those slots

## Goal

Find real-world implementations of tiling window manager layout save and restore with placeholder windows: i3 and i3-resurrect layout files with swallow criteria, sway swaymsg save/append_layout, swayr window matching, niri scrollable-tiling, PaperWM GNOME session restore, Qtile and awesome-wm layout persistence, and the xdg-session-management Wayland protocol. Extract how each stores a layout template, matches running windows to placeholders, and restores them over IPC, to design bindable space templates (named slots, versioned JSON, Lua binding API) for dawsers/scroll.

## Summary

Run `run_wm_space_template_restore_1789618096231` evaluated 15 candidate repo(s) against 6 proof requirement(s), selected 3 repo(s), and captured 121 evidence anchor(s) (119 proved).

## Selected repositories

| Repo | Class | Score | Why selected |
|---|---:|---:|---|
| [flatpak/flatpak](https://github.com/flatpak/flatpak) | reference | 0.50 | multi_file_implementation, runtime_flow_proof, production_shaped |
| [rcaelers/workrave](https://github.com/rcaelers/workrave) | reference | 0.40 | multi_file_implementation, runtime_flow_proof |
| [kitnil/notes](https://github.com/kitnil/notes) | reference | 0.35 | multi_file_implementation, production_shaped |

## Candidate analysis

### flatpak/flatpak
- Class: `reference` — Score: **0.50** — Next action: `read_more`
- Positive signals: multi_file_implementation, runtime_flow_proof, production_shaped
- Negative signals: none
- Evidence anchors: 111
- Missing proof: none

### rcaelers/workrave
- Class: `reference` — Score: **0.40** — Next action: `read_more`
- Positive signals: multi_file_implementation, runtime_flow_proof
- Negative signals: none
- Evidence anchors: 5
- Missing proof:
  - layout_serialization: Serializes the container/window tree (hierarchy, orientation, sizes, geometry) into a versioned JSON layout file via a save command.
  - placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.

### kitnil/notes
- Class: `reference` — Score: **0.35** — Next action: `read_more`
- Positive signals: multi_file_implementation, production_shaped
- Negative signals: none
- Evidence anchors: 3
- Missing proof:
  - placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
  - restore_over_ipc: Restore path: an IPC message/command (e.g. i3 APPEND_LAYOUT, swaymsg -t append_layout, or a CLI restore command) applies the saved layout to the live tree.

### colemickens/nixcfg
- Class: `docs` — Score: **0.00** — Next action: `reject`
- Positive signals: env_deployment_docs
- Negative signals: docs_only, no_exact_evidence
- Evidence anchors: 1
- Missing proof:
  - layout_serialization: Serializes the container/window tree (hierarchy, orientation, sizes, geometry) into a versioned JSON layout file via a save command.
  - placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
  - window_matching: Runtime window matcher: inspects live views (app_id / class / instance / title / pid) to decide which placeholder adopts the new window.
  - restore_over_ipc: Restore path: an IPC message/command (e.g. i3 APPEND_LAYOUT, swaymsg -t append_layout, or a CLI restore command) applies the saved layout to the live tree.
- Rejection reasons: docs_only, no_implementation_files

### OpenGamingCollective/ScopeBuddy
- Class: `docs` — Score: **0.00** — Next action: `reject`
- Positive signals: env_deployment_docs
- Negative signals: docs_only, no_exact_evidence
- Evidence anchors: 1
- Missing proof:
  - layout_serialization: Serializes the container/window tree (hierarchy, orientation, sizes, geometry) into a versioned JSON layout file via a save command.
  - placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
  - window_matching: Runtime window matcher: inspects live views (app_id / class / instance / title / pid) to decide which placeholder adopts the new window.
  - restore_over_ipc: Restore path: an IPC message/command (e.g. i3 APPEND_LAYOUT, swaymsg -t append_layout, or a CLI restore command) applies the saved layout to the live tree.
- Rejection reasons: docs_only, no_implementation_files

### sickcodes/Docker-OSX
- Class: `false_positive` — Score: **0.00** — Next action: `reject`
- Positive signals: none
- Negative signals: docs_only, no_exact_evidence
- Evidence anchors: 0
- Missing proof:
  - layout_serialization: Serializes the container/window tree (hierarchy, orientation, sizes, geometry) into a versioned JSON layout file via a save command.
  - placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
  - window_matching: Runtime window matcher: inspects live views (app_id / class / instance / title / pid) to decide which placeholder adopts the new window.
  - restore_over_ipc: Restore path: an IPC message/command (e.g. i3 APPEND_LAYOUT, swaymsg -t append_layout, or a CLI restore command) applies the saved layout to the live tree.
- Rejection reasons: docs_only, no_implementation_files, ambiguous_feature

### UbuntuBudgie/budgie-extras
- Class: `unknown` — Score: **0.00** — Next action: `reject`
- Positive signals: none
- Negative signals: no_exact_evidence
- Evidence anchors: 0
- Missing proof:
  - layout_serialization: Serializes the container/window tree (hierarchy, orientation, sizes, geometry) into a versioned JSON layout file via a save command.
  - placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
  - window_matching: Runtime window matcher: inspects live views (app_id / class / instance / title / pid) to decide which placeholder adopts the new window.
  - restore_over_ipc: Restore path: an IPC message/command (e.g. i3 APPEND_LAYOUT, swaymsg -t append_layout, or a CLI restore command) applies the saved layout to the live tree.
- Rejection reasons: no_implementation_files

### vslavik/poedit
- Class: `unknown` — Score: **0.00** — Next action: `reject`
- Positive signals: none
- Negative signals: no_exact_evidence
- Evidence anchors: 0
- Missing proof:
  - layout_serialization: Serializes the container/window tree (hierarchy, orientation, sizes, geometry) into a versioned JSON layout file via a save command.
  - placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
  - window_matching: Runtime window matcher: inspects live views (app_id / class / instance / title / pid) to decide which placeholder adopts the new window.
  - restore_over_ipc: Restore path: an IPC message/command (e.g. i3 APPEND_LAYOUT, swaymsg -t append_layout, or a CLI restore command) applies the saved layout to the live tree.
- Rejection reasons: no_implementation_files

### 8harath/Car-Parking-Detection
- Class: `unknown` — Score: **0.00** — Next action: `reject`
- Positive signals: none
- Negative signals: no_exact_evidence
- Evidence anchors: 0
- Missing proof:
  - layout_serialization: Serializes the container/window tree (hierarchy, orientation, sizes, geometry) into a versioned JSON layout file via a save command.
  - placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
  - window_matching: Runtime window matcher: inspects live views (app_id / class / instance / title / pid) to decide which placeholder adopts the new window.
  - restore_over_ipc: Restore path: an IPC message/command (e.g. i3 APPEND_LAYOUT, swaymsg -t append_layout, or a CLI restore command) applies the saved layout to the live tree.
- Rejection reasons: no_implementation_files

### AcceleratedIndustries/DropSync
- Class: `false_positive` — Score: **0.00** — Next action: `reject`
- Positive signals: none
- Negative signals: docs_only, no_exact_evidence
- Evidence anchors: 0
- Missing proof:
  - layout_serialization: Serializes the container/window tree (hierarchy, orientation, sizes, geometry) into a versioned JSON layout file via a save command.
  - placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
  - window_matching: Runtime window matcher: inspects live views (app_id / class / instance / title / pid) to decide which placeholder adopts the new window.
  - restore_over_ipc: Restore path: an IPC message/command (e.g. i3 APPEND_LAYOUT, swaymsg -t append_layout, or a CLI restore command) applies the saved layout to the live tree.
- Rejection reasons: docs_only, no_implementation_files, ambiguous_feature

### adamrpostjr/notif
- Class: `false_positive` — Score: **0.00** — Next action: `reject`
- Positive signals: none
- Negative signals: docs_only, no_exact_evidence
- Evidence anchors: 0
- Missing proof:
  - layout_serialization: Serializes the container/window tree (hierarchy, orientation, sizes, geometry) into a versioned JSON layout file via a save command.
  - placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
  - window_matching: Runtime window matcher: inspects live views (app_id / class / instance / title / pid) to decide which placeholder adopts the new window.
  - restore_over_ipc: Restore path: an IPC message/command (e.g. i3 APPEND_LAYOUT, swaymsg -t append_layout, or a CLI restore command) applies the saved layout to the live tree.
- Rejection reasons: docs_only, no_implementation_files, ambiguous_feature

### AdguardTeam/CodeGuidelines
- Class: `false_positive` — Score: **0.00** — Next action: `reject`
- Positive signals: none
- Negative signals: docs_only, no_exact_evidence
- Evidence anchors: 0
- Missing proof:
  - layout_serialization: Serializes the container/window tree (hierarchy, orientation, sizes, geometry) into a versioned JSON layout file via a save command.
  - placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
  - window_matching: Runtime window matcher: inspects live views (app_id / class / instance / title / pid) to decide which placeholder adopts the new window.
  - restore_over_ipc: Restore path: an IPC message/command (e.g. i3 APPEND_LAYOUT, swaymsg -t append_layout, or a CLI restore command) applies the saved layout to the live tree.
- Rejection reasons: docs_only, no_implementation_files, ambiguous_feature

### adlocode/xfwm4
- Class: `unknown` — Score: **0.00** — Next action: `reject`
- Positive signals: none
- Negative signals: no_exact_evidence
- Evidence anchors: 0
- Missing proof:
  - layout_serialization: Serializes the container/window tree (hierarchy, orientation, sizes, geometry) into a versioned JSON layout file via a save command.
  - placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
  - window_matching: Runtime window matcher: inspects live views (app_id / class / instance / title / pid) to decide which placeholder adopts the new window.
  - restore_over_ipc: Restore path: an IPC message/command (e.g. i3 APPEND_LAYOUT, swaymsg -t append_layout, or a CLI restore command) applies the saved layout to the live tree.
- Rejection reasons: no_implementation_files

### Aethersea/clipboard-helper
- Class: `false_positive` — Score: **0.00** — Next action: `reject`
- Positive signals: none
- Negative signals: docs_only, no_exact_evidence
- Evidence anchors: 0
- Missing proof:
  - layout_serialization: Serializes the container/window tree (hierarchy, orientation, sizes, geometry) into a versioned JSON layout file via a save command.
  - placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
  - window_matching: Runtime window matcher: inspects live views (app_id / class / instance / title / pid) to decide which placeholder adopts the new window.
  - restore_over_ipc: Restore path: an IPC message/command (e.g. i3 APPEND_LAYOUT, swaymsg -t append_layout, or a CLI restore command) applies the saved layout to the live tree.
- Rejection reasons: docs_only, no_implementation_files, ambiguous_feature

### agonopol/go-stem
- Class: `unknown` — Score: **0.00** — Next action: `reject`
- Positive signals: none
- Negative signals: no_exact_evidence
- Evidence anchors: 0
- Missing proof:
  - layout_serialization: Serializes the container/window tree (hierarchy, orientation, sizes, geometry) into a versioned JSON layout file via a save command.
  - placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
  - window_matching: Runtime window matcher: inspects live views (app_id / class / instance / title / pid) to decide which placeholder adopts the new window.
  - restore_over_ipc: Restore path: an IPC message/command (e.g. i3 APPEND_LAYOUT, swaymsg -t append_layout, or a CLI restore command) applies the saved layout to the live tree.
- Rejection reasons: no_implementation_files

## Implementation pattern

**Server flow**

**Client flow**

## Evidence map

### flatpak/flatpak

- `NEWS:80` [layout_serialization, proved] — `* Fix crashes in the portal update monitor (#6692) and OCI JSON handling` ([link](https://github.com/flatpak/flatpak/blob/main/NEWS#L80))
- `NEWS:639` [placeholder_slots, proved] — `* 'flatpak run -vv $app_id' shows all applicable sandboxing parameters` ([link](https://github.com/flatpak/flatpak/blob/main/NEWS#L639))
- `NEWS:639` [window_matching, proved] — `* 'flatpak run -vv $app_id' shows all applicable sandboxing parameters` ([link](https://github.com/flatpak/flatpak/blob/main/NEWS#L639))
- `NEWS:1453` [restore_over_ipc, proved] — `* Restore compatibility with older appstream-glib versions, fixing a` ([link](https://github.com/flatpak/flatpak/blob/main/NEWS#L1453))
- `NEWS:1571` [named_slots_binding, proved] — `- Better diagnostics when a --bind or other bind-mount fails` ([link](https://github.com/flatpak/flatpak/blob/main/NEWS#L1571))
- `NEWS:54` [session_management_protocol, proved] — `* Lock the session helper's runtime directory to prevent systemd-tmpfiles from` ([link](https://github.com/flatpak/flatpak/blob/main/NEWS#L54))
- `po/cs.po:1387` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/cs.po#L1387))
- `po/cs.po:4928` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/cs.po#L4928))
- `po/cs.po:2257` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/cs.po#L2257))
- `po/cs.po:204` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/cs.po#L204))
- `po/cs.po:237` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/cs.po#L237))
- `po/da.po:1431` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/da.po#L1431))
- `po/da.po:4971` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/da.po#L4971))
- `po/da.po:2301` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/da.po#L2301))
- `po/da.po:237` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/da.po#L237))
- `po/da.po:270` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/da.po#L270))
- `po/de.po:1389` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/de.po#L1389))
- `po/de.po:4943` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/de.po#L4943))
- `po/de.po:2270` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/de.po#L2270))
- `po/de.po:210` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/de.po#L210))
- `po/de.po:243` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/de.po#L243))
- `po/eo.po:1381` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/eo.po#L1381))
- `po/eo.po:4914` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/eo.po#L4914))
- `po/eo.po:2249` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/eo.po#L2249))
- `po/eo.po:201` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/eo.po#L201))
- `po/eo.po:234` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/eo.po#L234))
- `po/es.po:1430` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/es.po#L1430))
- `po/es.po:5028` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/es.po#L5028))
- `po/es.po:2313` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/es.po#L2313))
- `po/es.po:208` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/es.po#L208))
- `po/es.po:241` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/es.po#L241))
- `po/fr.po:1358` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/fr.po#L1358))
- `po/fr.po:4819` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/fr.po#L4819))
- `po/fr.po:2220` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/fr.po#L2220))
- `po/fr.po:205` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/fr.po#L205))
- `po/fr.po:238` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/fr.po#L238))
- `po/gl.po:1422` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/gl.po#L1422))
- `po/gl.po:5074` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/gl.po#L5074))
- `po/gl.po:2338` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/gl.po#L2338))
- `po/gl.po:205` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/gl.po#L205))
- `po/gl.po:238` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/gl.po#L238))
- `po/hr.po:1413` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/hr.po#L1413))
- `po/hr.po:4959` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/hr.po#L4959))
- `po/hr.po:2287` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/hr.po#L2287))
- `po/hr.po:205` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/hr.po#L205))
- `po/hr.po:238` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/hr.po#L238))
- `po/hu.po:1424` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/hu.po#L1424))
- `po/hu.po:5117` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/hu.po#L5117))
- `po/hu.po:2356` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/hu.po#L2356))
- `po/hu.po:204` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/hu.po#L204))
- `po/hu.po:237` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/hu.po#L237))
- `po/id.po:1399` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/id.po#L1399))
- `po/id.po:4954` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/id.po#L4954))
- `po/id.po:2273` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/id.po#L2273))
- `po/id.po:203` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/id.po#L203))
- `po/id.po:236` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/id.po#L236))
- `po/nl.po:1418` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/nl.po#L1418))
- `po/nl.po:4981` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/nl.po#L4981))
- `po/nl.po:2293` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/nl.po#L2293))
- `po/nl.po:206` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/nl.po#L206))
- `po/nl.po:239` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/nl.po#L239))
- `po/oc.po:1349` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/oc.po#L1349))
- `po/oc.po:4816` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/oc.po#L4816))
- `po/oc.po:2212` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/oc.po#L2212))
- `po/oc.po:195` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/oc.po#L195))
- `po/oc.po:228` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/oc.po#L228))
- `po/pl.po:1434` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/pl.po#L1434))
- `po/pl.po:5026` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/pl.po#L5026))
- `po/pl.po:2317` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/pl.po#L2317))
- `po/pl.po:207` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/pl.po#L207))
- `po/pl.po:242` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/pl.po#L242))
- `po/pt.po:1402` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/pt.po#L1402))
- `po/pt.po:4947` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/pt.po#L4947))
- `po/pt.po:2275` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/pt.po#L2275))
- `po/pt.po:206` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/pt.po#L206))
- `po/pt.po:239` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/pt.po#L239))
- `po/ro.po:1415` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/ro.po#L1415))
- `po/ro.po:5001` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/ro.po#L5001))
- `po/ro.po:2292` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/ro.po#L2292))
- `po/ro.po:206` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/ro.po#L206))
- `po/ro.po:240` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/ro.po#L240))
- `po/ru.po:1414` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/ru.po#L1414))
- `po/ru.po:4990` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/ru.po#L4990))
- `po/ru.po:2292` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/ru.po#L2292))
- `po/ru.po:209` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/ru.po#L209))
- `po/ru.po:242` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/ru.po#L242))
- `po/sk.po:1384` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sk.po#L1384))
- `po/sk.po:5039` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/sk.po#L5039))
- `po/sk.po:2315` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sk.po#L2315))
- `po/sk.po:204` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sk.po#L204))
- `po/sk.po:237` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sk.po#L237))
- `po/sl.po:1392` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sl.po#L1392))
- `po/sl.po:4944` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/sl.po#L4944))
- `po/sl.po:2261` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sl.po#L2261))
- `po/sl.po:204` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sl.po#L204))
- `po/sl.po:237` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sl.po#L237))
- `po/sr.po:1403` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sr.po#L1403))
- `po/sr.po:4959` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/sr.po#L4959))
- `po/sr.po:2276` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sr.po#L2276))
- `po/sr.po:203` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sr.po#L203))
- `po/sr.po:236` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sr.po#L236))
- `po/sv.po:1400` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sv.po#L1400))
- `po/sv.po:4971` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/sv.po#L4971))
- `po/sv.po:2277` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sv.po#L2277))
- `po/sv.po:204` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sv.po#L204))
- `po/sv.po:237` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/sv.po#L237))
- `po/tr.po:1395` [layout_serialization, proved] — `msgid "Show output in JSON format"` ([link](https://github.com/flatpak/flatpak/blob/main/po/tr.po#L1395))
- `po/tr.po:4965` [placeholder_slots, proved] — `#. Translators: The placeholder is for an app ref.` ([link](https://github.com/flatpak/flatpak/blob/main/po/tr.po#L4965))
- `po/tr.po:2270` [window_matching, proved] — `msgid "TABLE ID [APP_ID] - Remove item from permission store"` ([link](https://github.com/flatpak/flatpak/blob/main/po/tr.po#L2270))
- `po/tr.po:206` [named_slots_binding, proved] — `msgid "Add bind mount"` ([link](https://github.com/flatpak/flatpak/blob/main/po/tr.po#L206))
- `po/tr.po:239` [session_management_protocol, proved] — `msgid "Log session bus calls"` ([link](https://github.com/flatpak/flatpak/blob/main/po/tr.po#L239))

### rcaelers/workrave

- `NEWS:918` [window_matching, proved] — `modes are "None" (NEW: no input is blocked and break windows have a title bar` ([link](https://github.com/rcaelers/workrave/blob/main/NEWS#L918))
- `NEWS:363` [restore_over_ipc, proved] — `** Restore support for Windows Vista and up (#367)` ([link](https://github.com/rcaelers/workrave/blob/main/NEWS#L363))
- `NEWS:981` [session_management_protocol, proved] — `** Beter support for Gnome session management. Workrave is now properly restarted` ([link](https://github.com/rcaelers/workrave/blob/main/NEWS#L981))
- `po/nl.po:312` [window_matching, proved] — `msgid "Use the Sanctuary design for break windows and the status window."` ([link](https://github.com/rcaelers/workrave/blob/main/po/nl.po#L312))
- `po/nl.po:2006` [named_slots_binding, proved] — `msgid "Where each timer appears. Pair timers into the same slot to save space — Workrave will alternate between them."` ([link](https://github.com/rcaelers/workrave/blob/main/po/nl.po#L2006))

### kitnil/notes

- `poe.org:53` [layout_serialization, proved] — `- [[https://github.com/Vilsol/timeless-jewels][Vilsol/timeless-jewels: A timeless jewel calculator and skill tree for Path of Exile]]` ([link](https://github.com/kitnil/notes/blob/main/poe.org#L53))
- `wayland.org:4` [window_matching, proved] — `#+title: Wayland` ([link](https://github.com/kitnil/notes/blob/main/wayland.org#L4))
- `wm.org:4` [window_matching, proved] — `- [[https://github.com/aesophor/wmderland][aesophor/wmderland: 🌳 X11 tiling window manager using space partitioning trees]]` ([link](https://github.com/kitnil/notes/blob/main/wm.org#L4))

## Libraries and dependencies

- `wlroots`
- `wayland`

## Files likely needed in target project

- `package.json` — dependency manifest
- `README.md` — save a window-manager layout as a template with placeholder slots and restore it by binding freshly launched windows to those slots implementation file

## Pitfalls and edge cases

- Verify current library versions before copying any pattern.
- Confirm evidence anchors against the live repositories before implementation.

## Open questions

- rcaelers/workrave: layout_serialization: Serializes the container/window tree (hierarchy, orientation, sizes, geometry) into a versioned JSON layout file via a save command.
- rcaelers/workrave: placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
- kitnil/notes: placeholder_slots: Placeholder container model: saved layout nodes carry match criteria (swallow / app_id / class / instance / title) that identify which future window may occupy the placeholder.
- kitnil/notes: restore_over_ipc: Restore path: an IPC message/command (e.g. i3 APPEND_LAYOUT, swaymsg -t append_layout, or a CLI restore command) applies the saved layout to the live tree.

## Skeptic notes

- Verify every accepted repo actually implements the feature end to end.
- Confirm no docs-only repo was misclassified as implementation proof.
- Re-check anchors against live repository contents before relying on them.
- Warning: missingProof: UbuntuBudgie/budgie-extras was not proof-read (maxReposToProve reached).
- Warning: missingProof: vslavik/poedit was not proof-read (maxReposToProve reached).
- Warning: missingProof: 8harath/Car-Parking-Detection was not proof-read (maxReposToProve reached).
- Warning: missingProof: AcceleratedIndustries/DropSync was not proof-read (maxReposToProve reached).
- Warning: missingProof: adamrpostjr/notif was not proof-read (maxReposToProve reached).
- Warning: missingProof: AdguardTeam/CodeGuidelines was not proof-read (maxReposToProve reached).
- Warning: missingProof: adlocode/xfwm4 was not proof-read (maxReposToProve reached).
- Warning: missingProof: Aethersea/clipboard-helper was not proof-read (maxReposToProve reached).
- Warning: missingProof: agonopol/go-stem was not proof-read (maxReposToProve reached).
