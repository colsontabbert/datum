# Workflow

How work runs in this project. `/card`, `/side`, `/patch`, `/interview`, `/wrap-up`, and `/catch-up` read this file. Change it with a `/side` task.

- **Size:** medium
- **Plan:** `docs/BUILD_PLAN.md` (cards) and `docs/ROADMAP.md` (my guide)
- **Notes:** `docs/progress/<card-id>.md`
- **Work reaches main:** a branch and pull request per card
- **CI:** runs once a pull request is marked ready; drafts run nothing
- **Main protected:** <set by /new-project>

## Commands

| What | Command |
|---|---|
| Install | `uv sync` |
| Quick checks (lint, types, unit tests for what changed) | `uv run ruff check && uv run ruff format --check && uv run pytest` |
| Full tests | the quick checks, plus the probe build from `probe/`: `JAVA_HOME="$APPDATA/PrismLauncher/java/java-runtime-epsilon" ./gradlew.bat build` |
| End-to-end or real-run checks | a real session on the laptop with me there: `uv run datum check`, then `uv run datum run plans/<plan>.toml` (from P1-07) |
| Screens (UI cards: screenshots for my sign-off) | the GUI from its `.pyw` (from P1-19), and `report.html` in the browser preview (from P1-20) |
| Run it | `uv run datum` (terminal); the GUI from P1-19 on |

## Risky areas

Cards that touch these are effort X: they wait for my OK before code and before merging. Patches and side tasks that touch them wait for "merge it".

- **Prism's instances folder:** anything that writes inside `%APPDATA%\PrismLauncher\instances` (creating or deleting clones, editing their files, renaming mod jars, copying worlds in). Datum's clone, reset, world-copy, and mod-toggle code in `src/datum/`.
- **Restoring backed-up settings:** the backup and restore of config files around runs.
- **Deleting anything:** clones, plans (even to the Recycle Bin), results.

## Checks only I can do

- **Before merge:** sign-off on screenshots of the look, the report, and the GUI (P1-19 to P1-23); the 16 px icon; my OK on every X card before code and before merge.
- **On my machine, with me there:** every card that launches the game (P0-01, P1-03 to P1-07, and the cards after that run sessions). Plugged in, nothing else running, me not using the laptop during runs.
- **After merge, on the live thing:** none. Datum isn't deployed; it only runs on this laptop.

## Docs to read

- **UI work:** `docs/design-language.md` (written by P1-19). Until it exists, the spec's "The look" section. Tactile Web is archived in `docs/archive/tactile-web/` and not used.
- **Security and data:** the risky areas above, and the spec's "Benchmark clones".
- **Spec:** `docs/SPEC.md`
- **Before any work:** `docs/research.md` for verified facts about Prism, PresentMon, and the game.

## Generated files

none
