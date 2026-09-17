# G0 human decisions — bindable space templates

- **Planning run:** `20260917162037-gd9n1c`
- **Decision owner:** Taylor (human policy owner)
- **Authority:** `docs/research/space-templates-final/plan.md` v2.1, commit `55f27e4`
- **Gate:** **G0 RESOLVED**
- **Status:** These decisions supersede the reviewed specification wherever it describes pending-session, placeholder, timeout, cancel, or partial-restore machinery.

## H-01 — unrelated live views

`restore_hide` semantics apply on template application: live views in the target workspace that are not bound to any template slot move to the scratchpad, matching `SPACE_RESTORE_HIDE` parity. `CLOSE` remains out of scope.

The operation must undo the unrelated-view sweep if arrangement fails after the sweep, so a failed apply leaves the workspace unchanged.

## H-02 + D-06 — atomic, complete apply

- Do **not** create compositor placeholder windows.
- Do **not** create pending sessions.
- Do **not** add timeout or cancel machinery.
- Apply is one atomic command.
- Every slot must already be mapped to a live view or apply fails; completeness is the responsibility of the user/script.
- The Lua apply call surfaces failure as `(nil, error)` with the missing slot names to the user's callback so it can retry.
- Compositor-side validation is the zero-mutation backstop: validate completeness and all inputs before mutating workspace state.
- Users stage slots with their own placeholder windows (for example, launcher terminals swapped for real applications on selection).

The remaining rollback residual is limited to the single-command `restore_hide` sweep: if arrange fails after that sweep, restore the swept views and do not leave a partial apply.

## Required spec interpretation

Replace the reviewed spec's §7 pending-session lifecycle with a direct validate-then-apply lifecycle. Remove pending-session ownership, internal placeholder presentation, bind/commit/cancel session state, timeout handling, and listener cleanup from the v1 design. Update acceptance/tests and public Lua/API descriptions to cover complete atomic apply, missing-slot errors, `restore_hide` sweep rollback, and zero mutation on validation failure.

> **Addendum (post-recording):** D-02 was tightened after this artifact was
> written — the retry loop is plain userland scripting (apply → fill the named
> slots → apply again); callbacks are optional, and the apply API carries **no
> fallback/retry/staging parameters** (anti-feature guard, plan.md D-02).
