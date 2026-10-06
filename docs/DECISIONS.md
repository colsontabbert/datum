# Decisions

Each entry: the choice, why, and what would make us revisit it. New entries go at the bottom of their section, starting with the date and ending with where they came from (a card ID, `(<group> interview)`, or `(patch: <name>)`). Only real choices between options go here; details go in the spec.

## Product

| Decision | Why | Revisit when |
|---|---|---|
| 2026-10-05 Measure from outside the game (PresentMon, nvidia-smi, Flight Recorder); only the probe is tied to a game version. | Game updates, and the OpenGL to Vulkan switch, mostly break only the probe. (brief) | Never, without a strong reason. |
| 2026-10-05 Only ever touch cloned benchmark instances, never my real instances or saves. | Losing a real world or instance setup is the worst thing this tool could do. (brief) | Never. |
| 2026-10-05 No composite score; every metric read on its own. | A single number hides tradeoffs, such as more FPS but worse stutter. (brief) | Never. |
| 2026-10-05 Rank-based statistics: Hodges-Lehmann shift plus exact Mann-Whitney interval, instead of a bootstrap. | A percentile bootstrap with 5 runs per side covered only about 87% in simulation, not 95%. (research) | Sessions with many more runs per config become normal. |
| 2026-10-05 5 runs per config by default, 4 minimum. | With 3 per side no difference can ever be significant. (research) | Statistics method changes. |
| 2026-10-05 Every session starts with an A/A check; differences inside its noise band are "no measurable difference". | Laptop noise is large; anything smaller than it is not a finding. (brief) | Never. |
| 2026-10-05 Counterbalanced ABBA order, first launch discarded, temperature wait between runs. | Laptop heat drifts over a session. (research) | Never. |
| 2026-10-05 A run counts only once the game's own log lines prove the GPU and graphics API. | Both "Prefer" settings fall back silently, and the laptop has an integrated GPU. (research) | Never. |
| 2026-10-05 Always write `preferredGraphicsBackend` as `opengl` or `vulkan`, never `default`. | "Default" means OpenGL on 26.2 and 26.3 and Vulkan from 26.4. (research) | Never. |
| 2026-10-05 Experiment order: A/A, settings sweep, pack comparison, mod testing, version comparison. | Most useful findings first. (brief) | After v1, if priorities change. |
| 2026-10-05 Not in v1: in-game GUI, auto-applying settings, crowdsourced database, Mac or Linux, server benchmarking. | Scope. (brief) | Phase 2 interview. |
| 2026-10-06 The brief's "no in-game GUI" stays. | The game closes between runs, updates would break it, the datapack path has no mod, and anything in the game can change frame times. (new interview) | Never. |
| 2026-10-06 Phases: 0 manual proof, 1 v1 for me, 2 later work. | M0 is a real gate: without Vulkan capture the plan changes. Phase 2 waits until v1 has been used for real. (new interview) | A gate fails. |
| 2026-10-06 M1 records on a timer in the loaded world and saves Flight Recorder data with `jcmd JFR.dump` before closing the game. | The probe (1B) isn't there yet, and a killed game loses Flight Recorder data. (new interview) | 1B replaces it with the probe's clean quit. |
| 2026-10-06 One clone per session, reset before every run, deleted at the end only when it carries Datum's marker. | Cheaper than a clone per run; the marker makes deleting a real instance impossible. (new interview) | Resetting proves unreliable in 1A. |
| 2026-10-06 Datum refuses to start while Prism is open, and opens and closes Prism itself each run. | A second launch hands off to the open Prism and exits, so Datum can't find its game. (new interview) | Never. |
| 2026-10-06 Every session shows its estimate and waits for a yes; `--yes` skips waiting in the terminal only. | I always want to know how long before it starts. (new interview) | Never. |
| 2026-10-06 A desktop GUI in v1 (milestone 1G, after the report), sharing the runner code with the terminal commands. | I want a small window like my Delta-RetroArch Synchronizer's; building it last avoids reworking it every milestone. (new interview) | Never. |
| 2026-10-06 Plans are built in the GUI's "New session" form; I never edit plan text. | I don't want to edit settings files by hand. (new interview) | Never. |
| 2026-10-06 The GUI and report use a Minecraft game-menu look, always dark, Monocraft plus Segoe UI. Tactile Web is not used here for now. | I want it to feel like Minecraft. (new interview) | I want Tactile Web back. |
| 2026-10-06 The icon is my own drawing, `ruleroverdiamondpick.aseprite`: a wooden ruler over a diamond pickaxe. | I drew it after reviewing drafts. (new interview) | I redraw it. |
| 2026-10-06 Pack and version comparison live in 1D with the settings sweep. | Same comparison machinery. (new interview) | 1D interview. |

## Stack

| Decision | Why | Revisit when |
|---|---|---|
| 2026-10-05 Python 3.14 with uv for the runner and analysis; ruff and pytest. | Latest stable Python all dependencies support. (brief) | A dependency needs an older Python. |
| 2026-10-05 Probe: one Fabric jar for 26.2 and 26.3, compiled against 26.2, no Stonecutter. CI also compiles against 26.3. | Simplest setup that covers both. (research) | The jar fails to load on 26.3 in game (P1-08). |
| 2026-10-05 Fabric probe before the datapack; the datapack moves to Phase 2. | `/tp` judders at high FPS, datapack log output is unconfirmed on 26.x, and killing the game loses Flight Recorder data. (research) | Never for 26.x. |
| 2026-10-05 PresentMon 2.6.0 by process ID, columns by header name, run continuously and trimmed by timestamps. | Process names are ambiguous under Prism; column names changed between versions. (research) | A new PresentMon version. |
| 2026-10-06 colsontabbert-python-lib is not added. Built-in `tomllib` and `logging` instead. | It's private and Datum is public (CI would need a token), and its config module reads env vars, not TOML. (new interview) | The lib goes public and has something Datum needs. |
| 2026-10-06 Datum's settings in a git-ignored `datum.local.toml`, with a committed `datum.example.toml`; outside tools in a git-ignored `tools/`. | Personal paths stay out of a public repo. (new interview) | Phase 2 sharing. |
| 2026-10-06 tkinter for the GUI. | Matches my Delta-RetroArch Synchronizer project; ships with Python, no extra dependency. (new interview) | P1-19 finds tkinter can't do the look well. |
| 2026-10-06 Monocraft font (SIL OFL 1.1), bundled with its license. | Minecraft-style, actively maintained, and legal in a public MIT repo; Mojang's font isn't. (new interview) | It stops being maintained. |

## Open (decide when the card comes up)

- Exact warmup and cooldown numbers, and the target temperature between runs: P1-12.
- The "heavy throttling" threshold: P1-12.
- Heap sizes to sweep: P1-14.
- Stress pack size (the brief says 100 to 200 mods): P1-16.
- Which scenes to build first, and whether New Terrain is in v1: P1-09.
- How mods are classified as client, performance, visual, content, or worldgen: P1-17.
- The chart library for `report.html` (check maintenance first): P1-20.
- What the "New session" form offers for each setting: P1-22.
