# Datum build plan

## How to read this

The work is split into phases, phases into milestones, milestones into cards. A new phase starts only where something has to be true first (its gate). Each milestone is an interview group (`docs/ROADMAP.md`, "Interview groups"). Each card has an ID, a goal, what to build, and a "Done when" list, and one session builds one card with `/card`. Cards added later get IDs `C-01`, `C-02`, and so on, in the milestone they belong to.

Cards point at `docs/SPEC.md` for detail. If a card and the spec disagree, stop and ask.

Milestones marked `pending` in the roadmap were planned from `PROJECT_BRIEF.md` and the first interview only. Their cards are a sketch: their own interview rewrites them before they're built.

## Project setup

`/new-project` reads this section instead of asking again.

- **Name:** datum
- **Description:** Benchmarks and tunes Minecraft Java Edition settings, mods, and packs, with honest statistics.
- **Size:** medium
- **Stack:** Python 3.14 runner and analysis, managed with uv (ruff, pytest), tkinter for the desktop GUI. Fabric probe mod in Java 25 with Gradle and Fabric Loom. Already set up on 2026-10-05.
- **Frontend:** yes: a desktop GUI (tkinter, Minecraft game-menu look) and a generated static `report.html` in the same look. Tactile Web is not used on this project for now; the look is defined in Datum's own `docs/design-language.md` (written by P1-19).
- **CI:** yes: ruff and pytest on Windows, the probe build for 26.2 and 26.3. Already running; becomes draft-skipping with a "CI passed" check.
- **colsontabbert-python-lib:** no. It's private and Datum is public, and its config module reads env vars, not TOML. Datum uses the built-in `tomllib` and `logging`.
- **License:** MIT
- **Repo:** public, `colsontabbert/datum`
- **Platform:** Windows only (v1)

## Principles

1. Measure from outside the game. Only the probe is tied to a Minecraft version, so updates mostly break only the probe.
2. Never touch my real Prism instances or saves. Datum works on clones it made itself, and only deletes what carries its marker.
3. A run counts only once the game itself proves the GPU and graphics API it used. Anything that can't be proved is thrown out, never guessed.
4. Honest over impressive: no composite score, an A/A noise floor every session, and "no measurable difference" when that's the truth.
5. Plain English everywhere I read: the terminal, the GUI, `findings.md`, the report.
6. It recommends, I decide. Datum never applies settings to my real instances.
7. Every session tells me how long it will take and waits for my yes before starting.

## Phase overview

| Phase | Goal | Gate before it starts | Rough time |
|---|---|---|---|
| 0 | Prove by hand that frame times can be captured on both graphics APIs | none | one sitting |
| 1 | v1 for me: unattended sessions that produce findings and a report I trust, driven from a desktop GUI | Phase 0's proof passed | months |
| 2 | Later: older versions and other loaders, sharing with other people | v1 works and I've used it for real tuning decisions | open |

## Definition of done (every card)

- The card's "Done when" list is complete, each item with its evidence in the pull request.
- The checks in `docs/WORKFLOW.md` pass.
- UI work (GUI and `report.html`) follows `docs/design-language.md` once it exists, with screenshots in the pull request for my sign-off, keyboard focus visible, and every action reachable by a click.
- No secrets or personal data in code, logs, fixtures, or screenshots.
- Never touches my real Prism instances or saves. Tests use fake instance folders.
- Docs updated when behavior or setup changed. The card's notes in `docs/progress/<card-id>.md`.
- Rebased on the latest `main` and checked against everything that merged meanwhile.

---

## Phase 0: Manual proof

Capture frame times by hand from a Fabric instance on OpenGL and on Vulkan, and confirm the facts the runner skeleton relies on.

**Gate to Phase 1:** PresentMon captures valid frame times on both OpenGL and Vulkan, and the game's graphics backend and graphics device log lines are recorded for both. If Vulkan can't be captured, building stops and the plan is rethought with me.

### Milestone 0A: Manual proof

#### P0-01 Capture frame times by hand on OpenGL and Vulkan
**Goal:** proof that the outside-the-game measurement works on this laptop, plus the facts M1 needs, written down.
**Build:** a session walks me through it at the laptop and reads the files afterward. No Datum code. Uses copies of my FO instances made with Prism's own Copy button (`docs/SETUP_CHECKLIST.md`). Steps: find where Prism is installed; launch the FO 26.2 copy by hand with `preferredGraphicsBackend:opengl`, capture 60 s with PresentMon 2.6.0 from `tools/` targeted by process ID; the same on `vulkan`; launch once with `prismlauncher.exe --launch <copy> --world <save>` to confirm Quick Play from the command line; try `jcmd <pid> JFR.dump` on the running game with Flight Recorder on; one OpenGL launch of the FO 26.3 copy for Prism issue #6073. Add `tools/` to `.gitignore`. Spec: "Launching and recording a run". Research: `docs/research.md`, "Verify before building".
**Done when:**
- `docs/research.md` records, with the date: Prism's install path; the game process (`javaw.exe`, child of Prism) and how it was found; both PresentMon CSVs' row counts, and that `MsBetweenPresents` and `MsGPUBusy` hold real values for both APIs; `PresentMode` for both.
- The exact "Using graphics backend" and "Using graphics device" log lines for OpenGL and for Vulkan are quoted, and the device line names the RTX card.
- The result of the `--launch ... --world` test, the `jcmd JFR.dump` test (a readable `.jfr` file or the error), and the 26.3 OpenGL launch (crash or not) are recorded.
- The gate result is stated plainly: passed, or what failed.
- `tools/` is git-ignored, and `docs/SETUP_CHECKLIST.md` ticks what was done.

---

## Phase 1: v1 for me

Unattended sessions on 26.2 and 26.3 that produce `findings.md` and `report.html` I trust, started from a desktop GUI.

### Milestone 1A: Runner skeleton

One plan, one config, repeated runs: clone, launch, record, collect, clean up. No probe yet, so the game sits in the loaded world while it records.

#### P1-01 Datum's settings and `datum check`
**Goal:** `datum check` tells me in plain English whether this laptop is ready for a session, and how to fix what isn't.
**Build:** `datum.local.toml` (git-ignored) and a committed `datum.example.toml`; finding Prism's data folder and executable on its own, with the file only needed when that fails; the PresentMon path; logging setup (`datum.log` per session later, console now) with the built-in `logging`. `datum check` checks: Prism found, Prism not running, PresentMon present and the pinned version, `nvidia-smi` works and sees the RTX card, on AC power, the user in "Performance Log Users", Prism's Java 25 has `jcmd` and `jfr`. Spec: "Datum's settings and `datum check`". Builds on P0-01's recorded paths.
**Done when:**
- `uv run datum check` prints one line per check with OK or a plain fix ("PresentMon not found in tools\. Download PresentMon 2.6.0 ...").
- Exit code is 0 only when every check passes.
- Tests cover settings loading (missing file, bad value with a plain message) and Prism detection against fake folders.
- `datum.local.toml` is git-ignored and the example file documents every key.

#### P1-02 Plans, the session folder, and the time estimate
**Goal:** `datum run <plan>` reads a plan, shows what will happen and how long it takes, and waits for my yes.
**Build:** plan file parsing and validation (spec "Plans"); session ID and `results/<session-id>/` with `session.json` (plan, Datum version, start time); the rough-guess estimate and the confirm prompt (`Start? [y/N]`, `--yes` skips waiting); `results/` and `plans/` git-ignored; `plans/example.toml` committed. Spec: "Plans", "Sessions and the time estimate". Builds on P1-01.
**Done when:**
- A valid plan prints runs, configs, estimated length, and finish time, then waits; "n" or Enter exits without creating anything in Prism.
- Invalid plans are refused with a plain message naming the field (missing instance, fewer than 4 runs, unknown `options.txt` value type).
- `--yes` prints the estimate and continues without waiting.
- Tests cover plan validation and the estimate arithmetic.

#### P1-03 Benchmark clones
**Goal:** Datum makes a throwaway copy of my instance for each session and can put it back to a known state before every run, without ever touching the original.
**Build:** clone into Prism's `instances` folder (temp copy on the same drive, `uuid` line removed, renamed into place), a Datum marker file, `instance.cfg` overrides (`OverrideConsole=true`, `ShowConsoleOnError=false`), a snapshot of the clone's config files after setup, reset from that snapshot before every run, the first-run `options.txt` keys and the plan's config values with the graphics API always explicit, a fresh copy of the plan's save into the clone before every run, and deleting the clone at session end only when its marker is present (`--keep` keeps it). Spec: "Benchmark clones". Risky area. Builds on P1-02.
**Done when:**
- Tests on a fake `instances` folder show the source instance's files are byte-identical before and after a full clone, reset, and delete cycle.
- Deleting refuses, with a plain message, any folder without Datum's marker.
- The clone's `options.txt` holds the first-run keys and the plan's values after a reset; `preferredGraphicsBackend` is never `default`.
- A real clone of my FO 26.2 instance shows up in Prism under its `datum-bench-...` name and launches by hand (checked with me there).

#### P1-04 Launching the game and knowing when the world is ready
**Goal:** Datum can start the game through Prism, find the real game process, and tell when the world has loaded.
**Build:** refuse to start when Prism is already open; `prismlauncher.exe --launch <clone> --world <save>`; find the game as the child `javaw.exe` of the Prism process Datum started (never by name alone); tail the clone's `latest.log` for world-loaded and for the graphics backend and device lines; 3-minute timeout; close the game and then Prism at the end of a run. Spec: "Launching and recording a run". Builds on P1-03 and P0-01's findings.
**Done when:**
- With me there: three launches in a row each reach "world loaded" and close cleanly, and Prism isn't left running.
- A run that never loads (tested by naming a save that doesn't exist) fails after the timeout with a plain reason.
- With Prism already open, `datum run` refuses before cloning anything.
- Tests cover log parsing (world loaded, backend line, device line) against saved log samples.

#### P1-05 Recorders
**Goal:** every run comes back with frame times, GPU telemetry, and Flight Recorder data for exactly the capture window.
**Build:** PresentMon by process ID with `--qpc_time`, started once the world is loaded and stopped cleanly with Ctrl+Break; `nvidia-smi` logging every 500 ms; Flight Recorder through the clone's JVM arguments, saved with `jcmd <pid> JFR.dump` before closing the game; warmup (default 15 s) then the capture window (default 60 s), marked by timestamps; files collected into `runs/<run-id>/`. Spec: "Launching and recording a run". Builds on P1-04.
**Done when:**
- With me there: one run produces `presentmon.csv`, `nvidia-smi.csv`, `recording.jfr`, and `latest.log` in its run folder, each non-empty.
- `run.json` records the capture window's start and end timestamps, and the PresentMon rows inside it are about 60 s of frames.
- Tests cover reading PresentMon columns by header name and trimming to the capture window.

#### P1-06 Run snapshot and validity
**Goal:** every run says what it actually ran on, and runs that can't be trusted are marked with a reason.
**Build:** the per-run system snapshot (spec "Run validity and the system snapshot"): GPU and API from the game's log lines, GPU processes from `nvidia-smi -q -x`, driver version, AC power, Java version, JVM arguments, Minecraft and loader versions from `mmc-pack.json`, the mod list with file hashes, hashes of every config file Datum touched, and hardware-accelerated GPU scheduling on or off. Invalid reasons: wrong GPU, wrong API, `options.txt` changed by the game, on battery, crash. Builds on P1-05.
**Done when:**
- `run.json` holds the snapshot and `valid: true` or `valid: false` with each reason in plain words.
- Tests show each invalid reason triggers from saved samples (an integrated-GPU device line, an OpenGL line on a Vulkan run, a rewritten `options.txt`, battery power).
- Invalid runs keep all their files.

#### P1-07 The session loop
**Goal:** `datum run plan.toml` runs a whole session unattended and leaves everything as it found it.
**Build:** repeat runs per the plan; two failed runs in a row stop the session; Ctrl+C stops cleanly (game closed, recorders stopped, clone cleaned up, finished runs kept, session marked incomplete); keep Windows awake and the screen on, restored after; one plain line per step and the updated finish time after each run; `datum.log` in the session folder; a warning at the start not to use the laptop; the estimate uses real run times from past sessions once any exist, and says which it used. Spec: "The session loop". Builds on P1-02 to P1-06. Writes `docs/walkthroughs/1A.md`.
**Done when:**
- With me there: a 5-run session finishes, the clone is gone afterward, and my source instance is unchanged (hashes before and after).
- Ctrl+C mid-run leaves no game or Prism running, no clone left, and `session.json` says incomplete.
- A second session's estimate says "based on your last session".
- Tests cover the stop rules and the estimate from past run times.

### Milestone 1B: Probe and frozen worlds

Sketch, rewritten by the 1B interview. A smooth scripted camera path, capture markers, a clean quit, and repeatable worlds.

#### P1-08 Probe: camera path, markers, clean quit
**Goal:** the probe flies a smooth, repeatable camera path, marks capture start and end, records tick time, entity counts, and loaded chunks, and quits the game cleanly.
**Build:** in `probe/`, on Fabric API `ClientTickEvents`; markers to its own file; `Minecraft.stop()` for the quit. One jar for 26.2 and 26.3. Spec: "Probe".
**Done when:** the same jar runs the path and quits cleanly on both 26.2 and 26.3 in game (checked with me there); the markers file and metrics file are written; CI builds it for both versions.

#### P1-09 Frozen golden worlds
**Goal:** one pregenerated test world per version and worldgen mod set, copied fresh into every run.
**Build:** creating and storing golden worlds in Datum's data folder; pregeneration; Distant Horizons pregen then distant generation off; the scenes (Overlook, Flight, Crowd, and an optional New Terrain). Spec: "Frozen worlds". Risky area (writes into clones).
**Done when:** decided in the 1B interview.

#### P1-10 Runner drives the probe
**Goal:** sessions use the probe's markers and clean quit instead of a timer and `jcmd`.
**Build:** install the same probe jar into every clone; capture trimmed by the probe's markers; missing markers make a run invalid; Flight Recorder data from the clean exit. Spec: "Probe", "Launching and recording a run". Risky area (writes into clones).
**Done when:** decided in the 1B interview.

### Milestone 1C: Honest numbers

Sketch, rewritten by the 1C interview.

#### P1-11 Per-run metrics
**Goal:** each run's frame times and Flight Recorder data become the metrics in the spec.
**Build:** average FPS, median frame time, 1% and 0.1% lows (defined and labeled), p99, worst frame, stutter count, GPU busy ratio and the CPU-bound or GPU-bound label, GC pause total and max, peak heap (via the JDK's `jfr print --json`), and a first-half against second-half drift check. Spec: "Metrics".
**Done when:** decided in the 1C interview.

#### P1-12 Session design and rerun rules
**Goal:** sessions are arranged so laptop heat and order can't fake a result.
**Build:** the A/A check at the start, counterbalanced ABBA order, discarding the first launch, waiting for a target temperature between runs, throwing out and rerunning bad runs (including heavy throttling and focus loss), identical `PresentMode` across compared configs. Spec: "Methodology".
**Done when:** decided in the 1C interview.

#### P1-13 Statistics and findings
**Goal:** `summary.csv` and a plain-English `findings.md` I trust.
**Build:** Hodges-Lehmann shift on log metrics, exact Mann-Whitney interval, the A/A noise band, "no measurable difference" when the interval sits inside it, no composite score. Spec: "Statistics and findings".
**Done when:** decided in the 1C interview.

### Milestone 1D: Settings sweep, packs, and versions

Sketch, rewritten by the 1D interview.

#### P1-14 Several configs and safe settings changes
**Goal:** a plan can compare several configs, and every file Datum changes is backed up and restored, even after a crash.
**Build:** configs covering `options.txt`, Sodium, Iris, Distant Horizons, and heap (through `instance.cfg`); backup and restore around every run and after crashes. Spec: "Settings sweep". Risky area.
**Done when:** decided in the 1D interview.

#### P1-15 Graphics API sweep
**Goal:** OpenGL against Vulkan, only where every mod in the set is confirmed to work on Vulkan.
**Build:** the Vulkan compatibility gate per mod set with the reason in the report when it's skipped; Iris shader tests OpenGL only; Distant Horizons' renderer pinned per run. Spec: "Settings sweep".
**Done when:** decided in the 1D interview.

#### P1-16 Pack and version comparison
**Goal:** vanilla against FO against FO plus my mods against a stress pack, and the same pack on 26.2 against 26.3.
**Build:** configs that point at different instances; results flagged when the worlds differ. Spec: "Pack and version comparison".
**Done when:** decided in the 1D interview.

### Milestone 1E: Mod testing

Sketch, rewritten by the 1E interview.

#### P1-17 Mod inventory and dependencies
**Goal:** Datum knows every mod in an instance, what each depends on, and which are safe to switch off.
**Build:** read `fabric.mod.json` in every jar; the dependency graph; client, performance, and visual mods separated from content and worldgen mods; disabling by renaming to `.jar.disabled`, never deleting. Spec: "Mod testing". Risky area.
**Done when:** decided in the 1E interview.

#### P1-18 Leave-one-out and group search
**Goal:** find which mods actually cost or gain frames.
**Build:** group toggles first (all visual off, all performance off), then narrowing down, with the estimate shown before each session. Spec: "Mod testing".
**Done when:** decided in the 1E interview.

### Milestone 1F: The look and the report

Sketch, rewritten by the 1F interview. The look's main decisions are already settled (spec "The look").

#### P1-19 The look: design doc, icon, and window shell
**Goal:** the Minecraft game-menu look exists for real, as a design doc, icon files, and an empty GUI window I sign off on.
**Build:** `docs/design-language.md` (colors including the icon's diamond blue, ruler cream, and handle brown; the dirt-style pattern; stone buttons with hover and grayed-out states; Monocraft for headings, buttons, and the log, Segoe UI for longer text; always dark); `assets/icon/datum.aseprite` copied from `C:\Coding\Icons\ruleroverdiamondpick.aseprite`; a `.ico` with 16, 32, 48, and 256; Monocraft bundled with its license; the empty GUI window with tabs and the full look, opened from a `.pyw` file. If tkinter can't do the look well, stop and come back with options. Spec: "The look".
**Done when:**
- Screenshots of the empty window and the icon at every size are in the pull request, and I've signed off.
- The 16 px icon is shown next to the 32; if I redraw it, the redraw is swapped in.
- No Mojang textures or fonts anywhere in the repo.

#### P1-20 `report.html`
**Goal:** a static report in the same look, with results that stay easy to read.
**Build:** generated per session from `summary.csv` and the runs; tables and charts in a plain readable font; metric definitions, the noise floor, and every flag (worlds differ, Vulkan skipped and why). Spec: "Report".
**Done when:** decided in the 1F interview; includes my sign-off on screenshots.

### Milestone 1G: Desktop GUI

Sketch, rewritten by the 1G interview. Most of the GUI is already settled (spec "Desktop GUI").

#### P1-21 Run tab
**Goal:** start and watch a session from the window instead of a terminal.
**Build:** the plan dropdown, "Check setup" as a checklist with fixes, "Start session" with the estimate in a confirm box, the log panel, minimizing during a session and coming back after, the "Stop the session?" prompt on close. Spec: "Desktop GUI".
**Done when:** decided in the 1G interview; includes my sign-off on screenshots.

#### P1-22 New session form
**Goal:** I make plans by picking from lists, never by editing text.
**Build:** instance and world dropdowns, what to compare, runs and length, the live estimate, refusals explained next to the field, saving to `plans/`, editing, "Delete plan..." to the Recycle Bin, "Open plans folder". Spec: "Desktop GUI". Risky area (deletes files).
**Done when:** decided in the 1G interview; includes my sign-off on screenshots.

#### P1-23 Results and Settings tabs
**Goal:** find past sessions and change Datum's settings from the window.
**Build:** the sessions list with buttons to open `findings.md`, `report.html`, and the folder; Settings with paths and "Browse...", hover descriptions, "Save settings", "Create Start menu shortcut". Spec: "Desktop GUI". Writes `docs/walkthroughs/1G.md`.
**Done when:** decided in the 1G interview; includes my sign-off on screenshots.

---

## Phase 2: Later

Sketch only; its interview plans it once the gate is met.

**Gate:** v1 works and I've used it for real tuning decisions.

### Milestone 2A: Older versions and other loaders

#### P2-01 Tier 0 bench datapack
**Goal:** benchmark without a mod, so older versions and other loaders work.
**Build:** a datapack that moves the camera smoothly (spectating an `item_display` moved with `teleport_duration`) and prints markers to `latest.log`; ending a run by closing the game. Spec: "Probe" (Tier 0 notes).
**Done when:** decided in the 2A interview.

#### P2-02 Older packs on other loaders
**Goal:** benchmark big older packs (1.21.1 NeoForge and similar) through Tier 0.
**Build:** decided in the 2A interview.
**Done when:** decided in the 2A interview.

### Milestone 2B: Sharing

#### P2-03 Docs and defaults for other people
**Goal:** someone else can install Datum and run a sensible first session.
**Build:** decided in the 2B interview (docs, defaults, a default test world).
**Done when:** decided in the 2B interview.

#### P2-04 Shareable reports
**Goal:** a report I can send to someone.
**Build:** decided in the 2B interview.
**Done when:** decided in the 2B interview.

---

## Back pocket

Kept for later; each needs my go-ahead.

- Resuming a stopped session.
- A Windows notification when a session finishes.
- Building the stress pack automatically through the Modrinth API.
