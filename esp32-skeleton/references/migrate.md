# Reorganizing an Existing PlatformIO Project

Flow for projects that already have `platformio.ini`. Always read `layout.md` first.

---

## Step 1 — Audit Current State

Read these files if they exist:
- `platformio.ini`
- `Makefile`
- `src/` directory structure
- `make/` or `scripts/` directory

Identify what is missing compared to the target layout in `layout.md`.

---

## Step 2 — Determine Target Tier

Look at `platformio.ini` to determine current complexity:

- Single `[env:*]` section → Simple or Standard
- Multiple `[env:*]` sections → Advanced
- `platform = native` test env → Advanced with host testing

---

## Step 3 — Common Gaps to Fix

### Missing `make/` scripts

If the project has no helper scripts, create:
- `make/check-pio.sh`
- `make/run-pio.sh`

See `create.md` Step 5 for content.

### Makefile missing standard targets

Compare against the standard Makefile in `create.md` Step 6. Add missing targets without removing project-specific ones.

### Secrets committed to git

If `secret.h` or `.env` is tracked:
1. Add to `.gitignore`
2. Create `secret.h.template` from `secret.h` (replace values with placeholders)
3. Inform the user they need to rotate any exposed credentials.

### Missing `.env.example`

Create from the pattern in `layout.md`.

### Missing `.vscode/extensions.json`

Create from the pattern in `layout.md`.

---

## Step 4 — Do Not Break

- Never change `platformio.ini` environment names without updating the Makefile.
- Never move files out of `src/` — PlatformIO requires it.
- Never remove `lib_deps` entries without checking if the code imports them.
- If `data/` exists with filesystem assets, preserve it.
- If `test/` exists, preserve all test files.
