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

### Required `make/` scripts

Verify each script exists. Create any that are missing using the templates in `create.md` Step 5:

- [ ] `make/check-pio.sh` — verifies PlatformIO is installed
- [ ] `make/install-pio.sh` — installs PlatformIO via pip
- [ ] `make/run-pio.sh` — wraps `pio` with PATH fallback
- [ ] `make/clean.sh` — delegates clean to PlatformIO

For Standard tier also check:
- [ ] `make/detect_board.sh` — auto-detects USB port

### Required Makefile targets

Verify each target exists. Add any that are missing using the template in `create.md` Step 6. Do not remove project-specific targets.

| Target | Must exist in |
|--------|--------------|
| `build` | All tiers |
| `upload` | All tiers |
| `flash` | All tiers |
| `monitor` | All tiers |
| `clean` | All tiers |
| `deps` | All tiers |
| `check` | All tiers |
| `test` | All tiers |
| `erase` | All tiers |
| `check-pio` | All tiers |
| `install-pio` | All tiers |
| `help` | All tiers |

Also verify:
- `MAKEFLAGS += --no-print-directory` is the first line
- `.DEFAULT_GOAL := help` is the second line
- All targets are declared in a single `.PHONY` line (alphabetical)
- `check-pio` has `## Verify PlatformIO is installed` so it appears in `make help`

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
