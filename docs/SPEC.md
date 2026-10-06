# Datum spec

What the product does, area by area. Cards in `docs/BUILD_PLAN.md` point here. When this file passes roughly 600 lines, split it into `docs/specs/<area>.md` and keep this file as the index.

Areas marked **(sketch)** come from `PROJECT_BRIEF.md` and the first interview only; their milestone's interview fills them in before their cards are built.

## How it fits together

```
GUI (tkinter)  or  terminal (datum run / datum check)
        |                     |
        +------ runner (Python, outside the game) ------+
                |                                       |
   clone instance, reset settings, copy fresh world     |
                |                                       |
   launch through Prism CLI  -->  game (javaw.exe)      |
                |                  probe mod (from 1B)  |
   PresentMon + nvidia-smi + Flight Recorder record     |
                |                                       |
   close game, collect files, check the run is valid ---+
                |
   repeat per plan  -->  analysis  -->  summary.csv, findings.md, report.html
```

- **Runner** (`src/datum/`): everything outside the game. The GUI and the terminal commands call the same runner code; anything the GUI does can also be done from a terminal.
- **Probe** (`probe/`): a small Fabric mod, one jar for 26.2 and 26.3. From milestone 1B. Until then the game sits in the loaded world while Datum records on a timer.
- **Recorders:** PresentMon (frame times), `nvidia-smi` (GPU clocks, temperature, power, throttling), Java Flight Recorder (garbage collection, heap). All outside the game.
- **Results:** one folder per session under `results/`, git-ignored.

## Target setup (v1)

- **Machine:** MSI Vector A16 HX laptop, Ryzen 9 8940HX, RTX 5070 Ti laptop GPU, 16 GB DDR5, NVMe SSD, Windows. The CPU also has an integrated GPU, so every run must prove it used the RTX card.
- **Launcher:** Prism Launcher (11.1.1 when researched).
- **Minecraft:** 26.2 and 26.3, Fabric, Java 25.
- **Packs:** Fabulously Optimized 14.1.0 on 26.2, 15.0.0-alpha on 26.3 (alpha, with alpha Sodium, so 26.3 results are less trustworthy for now). My usual instance is FO plus extra mods; Datum reads the actual mod list from the instance.
- **Graphics API:** OpenGL and Vulkan are both options on 26.2 and 26.3, and the API is one of the things Datum compares.

## Datum's settings and `datum check`

- Datum's own settings live in `datum.local.toml` at the repo root, git-ignored. `datum.example.toml` is committed and documents every key.
- Datum finds Prism's data folder (`%APPDATA%\PrismLauncher`) and executable on its own. The settings file is only needed when that fails, or to point at PresentMon if it isn't in `tools/`.
- Outside tools live in `tools/` in the repo, git-ignored. PresentMon is pinned to 2.6.0; Datum checks the version.
- `datum check` launches nothing and prints one line per check, each OK or a plain fix:
  - Prism found (data folder and executable).
  - Prism not running.
  - PresentMon present, right version.
  - `nvidia-smi` works and lists the RTX card.
  - On AC power.
  - The user is in "Performance Log Users" (or Datum runs as admin).
  - Prism's Java 25 runtime has `jcmd` and `jfr`.
- Exit code 0 only when every check passes.

**Done when:** `datum check` on this laptop shows all OK, and each failing check names its fix.

## Plans

A plan is a small TOML file describing one session. I make them in the GUI's "New session" form (from 1G); the terminal takes a plan file path.

- **M1 contents:** source instance name, a save to load (from the source instance until frozen worlds exist), warmup seconds (default 15), run length seconds (default 60), runs per config, and one config: a name and a list of `options.txt` values.
- **From M4:** several configs per plan, each able to change `options.txt`, Sodium, Iris, Distant Horizons, heap, the instance itself (packs, versions), and the mod set (from 1E).
- **Refused, with a plain message naming the field:** a missing or unknown instance, a missing save, fewer than 4 runs per config ("too few runs to ever show a real difference"), a graphics API other than `opengl` or `vulkan`, Vulkan with Iris installed ("Iris shaders only work on OpenGL").
- Plans live in `plans/` in the repo, git-ignored, because they name my own instances and worlds. `plans/example.toml` is committed.

## Sessions and the time estimate

- A session ID looks like `2026-10-06_1432_<plan name>`.
- Before every session Datum shows what will happen and how long it takes, then waits for a yes: "10 runs (2 configs x 5), about 1 h 50 min, finishing around 1:40 AM. Start?"
  - Terminal: `Start? [y/N]`. `datum run plan.toml --yes` prints the estimate and starts without waiting.
  - GUI: a confirm box with Start and Cancel. The GUI always asks.
- The estimate is a rough guess at first (launch + warmup + run length + cooldown, per run). Once sessions have run on this laptop, it uses their real run times. It always says which: "rough guess" or "based on your last 3 sessions".
- After each run, the log (and the GUI) shows the updated finish time.

**Done when:** no session can start without the estimate shown, and the estimate says where its number came from.

## Benchmark clones

- One clone per session, named `datum-bench-<session-id>`, made in Prism's `instances` folder: copied to a temp folder on the same drive, the `uuid` line removed from `instance.cfg`, then renamed into place.
- Datum writes a marker file into every clone it makes. It deletes only folders with that marker, and refuses anything else with a plain message.
- The clone's `instance.cfg` gets `OverrideConsole=true` and `ShowConsoleOnError=false`, so closing the game doesn't leave Prism's console window open.
- `options.txt` first-run keys on every clone: `inactivityFpsLimit:minimized`, `onboardAccessibility:false`, `tutorialStep:none`, `skipMultiplayerWarning:true`, `narrator:0`. Then the plan's config values. `preferredGraphicsBackend` is always written as `opengl` or `vulkan`, never `default`.
- After setup, Datum snapshots the clone's config files and resets them from the snapshot before every run.
- Before every run, a fresh copy of the plan's save goes into the clone's `saves` folder. The source save is only ever read.
- At session end the clone is deleted. `--keep` keeps it.
- Datum never writes to the source instance or its saves.

**Done when:** a full session leaves the source instance byte-identical, and no clone is left behind unless `--keep` was given.

## Launching and recording a run

- If Prism is already open, Datum refuses to start and asks me to close it, because a second launch would quietly hand off to the open copy. Datum opens and closes Prism itself for each run.
- Launch: `prismlauncher.exe --launch <clone folder> --world <save folder>`. Prism turns `--world` into Quick Play.
- The game process is the `javaw.exe` child of the Prism process Datum started. Never matched by name alone.
- Datum tails the clone's `latest.log` for "world loaded" and for the game's own lines:
  - `Using graphics backend <Vulkan or OpenGL>, ...`
  - `Using graphics device: <GPU name> (<vendor>)`
- If the world hasn't loaded within 3 minutes (for example, a Microsoft login popup is stuck), the run fails.
- Recording starts once the world has loaded:
  - PresentMon 2.6.0, targeted by process ID, with `--qpc_time`, running continuously and stopped with Ctrl+Break. Columns read by header name.
  - `nvidia-smi --query-gpu=timestamp,pstate,clocks.gr,clocks.sm,power.draw,temperature.gpu,utilization.gpu,clocks_event_reasons.active,... --format=csv,nounits -lms 500`.
  - Flight Recorder through the clone's JVM arguments, the same on every run.
- Warmup (default 15 s), then the capture window (the run length, default 60 s). The window's start and end timestamps go in `run.json`, and frames are trimmed to it.
- Until the probe exists (1B): Flight Recorder data is saved with `jcmd <pid> JFR.dump`, then the game is closed. From 1B the probe quits the game cleanly and Flight Recorder writes on exit.
- Files per run: `presentmon.csv`, `nvidia-smi.csv`, `recording.jfr`, `latest.log`, `run.json`.

## Run validity and the system snapshot

- Every `run.json` holds: the GPU and API from the game's log lines, GPU processes from `nvidia-smi -q -x`, driver version, AC or battery, Java version, JVM arguments, Minecraft and loader versions (from `mmc-pack.json`), the mod list with file hashes, hashes of every config file Datum touched, hardware-accelerated GPU scheduling on or off.
- A run is invalid, with each reason in plain words, when: the GPU isn't the RTX card; the API isn't the one planned; the game rewrote `options.txt` (26.2 and 26.3 do this after a startup crash); the laptop was on battery; the game crashed. From 1B: missing markers. From 1C: heavy throttling, focus loss, `PresentMode` differs.
- Invalid runs keep all their files. Automatic reruns come in 1C; until then they're only marked.

## The session loop

- Runs repeat per the plan.
- Two failed runs in a row stop the session.
- Ctrl+C (or "Stop" in the GUI) closes the game, stops recording, cleans up the clone, keeps finished runs, and marks the session incomplete in `session.json`. No resuming a stopped session in v1.
- During a session Datum keeps Windows awake and the screen on, and switches that back afterward.
- At the start, Datum warns: don't use the laptop during the session. Detecting focus loss comes in 1C.
- Progress: one plain line per step ("Run 3/10: launching... recording 60 s... done") and the updated finish time. Everything also goes to `datum.log` in the session folder.

**Done when:** an unattended 5-run session finishes and leaves nothing behind but its results folder.

## Results folder

```
results/<session-id>/
  session.json        plan, system info, versions, mod lists + hashes, complete or incomplete
  datum.log
  runs/<run-id>/
    presentmon.csv
    nvidia-smi.csv
    recording.jfr
    latest.log
    run.json          snapshot, capture window, valid or invalid with reasons
  summary.csv         (from 1C)
  findings.md         (from 1C)
  report.html         (from 1F)
```

## Probe (sketch)

- Tier 1, the Fabric probe: a smooth interpolated camera path, markers to its own file, client tick time, entity counts, loaded chunks, then a clean quit with `Minecraft.stop()`. Chunk rebuild counts are optional (Sodium hides the vanilla counters).
- Installed identically in every run being compared.
- One jar compiled against 26.2; whether it loads on 26.3 too is unconfirmed until tested in game.
- Tier 0 (Phase 2): a bench datapack spectating an `item_display` moved with `teleport_duration`, markers in `latest.log` (unconfirmed on 26.x), the run ended by closing the game. Datapack formats: 26.2 = 107.1, 26.3 = 121.0.

## Frozen worlds (sketch)

- One golden world per Minecraft version and per worldgen mod set, pregenerated, stored in Datum's data folder, copied fresh into every run.
- With Distant Horizons: `/dh pregen`, then distant generation off for benchmark runs.
- Scenes: Overlook (slow pan at max view distance, GPU heavy), Flight (straight line through pregenerated terrain, chunk streaming), Crowd (lots of entities, CPU heavy), New Terrain (optional, into ungenerated chunks, noisier).
- When compared configs have different worldgen mods, the worlds differ and the report says so.

## Metrics (sketch)

Per run, from the capture window only: average FPS, median frame time, 1% low and 0.1% low (average FPS of the slowest 1% and 0.1% of frames, labeled with that definition, with p99 frame time beside them), p99 frame time, worst frame, stutter count (frames over 2x the median), GPU busy ratio and a CPU-bound or GPU-bound label, GC pause total and max, peak heap. A first-half against second-half check catches drift inside a run. Metrics are per run; frames are never pooled across runs.

## Methodology (sketch)

- Every session starts with an A/A check (the same config against itself) to measure the noise floor.
- Counterbalanced ABBA order. The first launch of a session is discarded.
- Between runs, wait for a target temperature instead of a fixed delay.
- 5 runs per config by default, 4 minimum.
- Conditions: plugged in, same MSI Center mode, same resolution, vsync off, FPS cap off. Datum checks what it can and warns about the rest.
- Throw out and rerun: wrong GPU or API, battery, crash, missing markers, heavy throttling (threshold set in 1C), focus loss.

## Statistics and findings (sketch)

- Effect: Hodges-Lehmann shift (median of all pairwise differences) on log metrics, so results read as percentages. Interval: exact Mann-Whitney inversion (5 vs 5: 3rd smallest to 3rd largest pairwise difference, about 96.8% coverage).
- "No measurable difference" when the interval sits inside the A/A noise band.
- No composite score. Every metric is read on its own.
- `summary.csv` and `findings.md`, in this tone (format only, made-up numbers): "Vulkan vs OpenGL (26.2, FO, overlook scene): Vulkan +18% avg FPS, 1% lows +9% (95% CI +5% to +13%). GPU-bound both ways. Worth switching."

## Settings sweep (sketch)

- Settings: render distance, simulation distance, graphics API, shaders on or off and shader profile, Distant Horizons LOD distance and quality, heap size, key Sodium options.
- Every changed file is backed up and restored after every run, including after crashes.
- Vulkan is tested for a mod set only once every mod in it is confirmed to work on Vulkan; otherwise the report says why it was skipped. Iris shaders are OpenGL only. Distant Horizons' renderer setting is pinned per run.

## Pack and version comparison (sketch)

- Vanilla against FO against FO plus my mods against a big stress pack (FO plus popular 26.2 Fabric mods, built by me).
- The same pack on 26.2 against 26.3. The worlds differ, and the report says so.

## Mod testing (sketch)

- Read `fabric.mod.json` in every jar; never disable a library something else needs, and disable its dependents with it.
- Disable by renaming to `<name>.jar.disabled`; never delete a jar.
- Leave-one-out for client, performance, and visual mods only. Content and worldgen mods are excluded (removing them changes the world).
- Big mod lists start with group toggles, then narrow down.

## The look

Settled in the first interview; P1-19 writes it up as `docs/design-language.md`, which then becomes the reference.

- Minecraft game-menu look: a dark dirt-style background pattern, chunky gray beveled stone buttons, white text with a shadow.
- Always dark. It doesn't follow Windows' light or dark setting.
- Fonts: Monocraft (SIL OFL 1.1, bundled with its license) for headings, buttons, and the log; Segoe UI for descriptions and longer text.
- Accents from the icon: diamond blue, ruler cream, handle brown.
- Buttons have a lighter hover state and a grayed-out state when they can't be pressed. Text stays high-contrast. Anything shown in color also says it in words ("INVALID: wrong GPU", not only red).
- Patterns and textures are original. No Mojang textures or fonts.
- The icon is my own drawing, `assets/icon/datum.aseprite` (32x32: a cream wooden ruler over a diamond pickaxe). The `.ico` holds 16, 32, 48, and 256; 48 and 256 are clean pixel enlargements; the 16 is a half-size version I approve or redraw.
- Tactile Web is not used on this project for now; its docs are kept in `docs/archive/tactile-web/`.

## Report (sketch)

- `report.html`: static, generated per session, in the same Minecraft look. Results tables and charts in a plain readable font.
- Shows metric definitions, the session's noise floor, and every flag (worlds differ, Vulkan skipped and why, invalid runs and their reasons).

## Desktop GUI

Most of this was settled in the first interview; the 1G interview settles the form's controls.

- tkinter, the look above. Opened from a `.pyw` at the repo root (no console window) or a Start menu shortcut.
- Every action is a button; keyboard shortcuts are only extras. No long text fields, so no dictation button; one is added if a long text field ever appears.
- **Run tab** (main): a plan dropdown; "Check setup" showing `datum check` as a checklist with how to fix each problem; a big "Start session" button at the bottom that shows the estimate in a confirm box; a log panel as the main element, with the same lines as the terminal.
- **During a session:** the window minimizes when the session starts, so it can't pull focus from the game, and comes back when it ends. Closing the window asks "Stop the session?": Yes stops cleanly like Ctrl+C, No keeps it running.
- **New session form:** instance dropdown (my Prism instances, read only; `datum-bench-...` clones hidden); world dropdown (saves in that instance until frozen worlds exist, then the frozen worlds); what to compare (whatever a plan can hold when 1G is built); runs per config and run length; a live estimate. Plans that can't run are refused with the reason next to the field. Saved to `plans/`. Existing plans open in the same form. "Delete plan..." asks first and moves the file to the Recycle Bin. "Open plans folder".
- **Results tab:** past sessions (date, plan, runs finished and valid, complete or stopped), with buttons to open `findings.md`, `report.html`, and the session folder.
- **Settings tab:** Prism and PresentMon paths with "Browse...", a description on hover for every setting, "Save settings", "Create Start menu shortcut".

## Out of scope (v1)

- An in-game GUI (the game closes between runs, updates would break it, the datapack path has no mod, and anything added to the game can change frame times).
- Auto-applying recommended settings to my real instances.
- A crowdsourced results database.
- Mac or Linux.
- Server TPS benchmarking (spark covers servers).
- Resuming a stopped session; a Windows notification when a session ends (both in Back pocket).
