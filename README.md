# Datum

Named after a surveyor's datum: the fixed reference point everything else gets measured from.

A personal benchmarking and tuning tool for Minecraft Java Edition on Windows. It launches Minecraft over and over with different settings, mod sets, and modpacks, measures frame times from outside the game, and reports in plain English what actually helps, what hurts, and how confident it is.

Targets Minecraft 26.2 and 26.3 on Fabric, launched through Prism Launcher.

## How it works

- **Runner** (`src/datum/`, Python): clones a benchmark copy of a Prism instance, applies one run's settings and mod set, copies in a fresh test world, launches the game, records, restores everything, and repeats in a randomized, counterbalanced order.
- **Measurement** (outside the game): PresentMon for frame times, Java Flight Recorder for garbage collection and heap, nvidia-smi for GPU clocks, temperature, and throttling. The game's own log confirms which graphics API and which GPU each run actually used; runs that don't match are thrown out.
- **Probe** (`probe/`, Fabric mod): drives a repeatable camera path through the test world, marks when capture starts and ends, and quits the game cleanly.
- **Analysis:** an A/A run measures the noise floor first. Each comparison reports a rank-based effect size with an exact confidence interval, and anything inside the noise floor is reported as "no measurable difference". There is no composite score. Output is `summary.csv`, `findings.md`, and a static `report.html`.

See [PROJECT_BRIEF.md](PROJECT_BRIEF.md) for the full plan and [docs/research.md](docs/research.md) for verified facts about the tools and game versions involved.

## Setup

Requirements: Windows, [uv](https://docs.astral.sh/uv/), Prism Launcher, and a JDK 25 for building the probe (Prism's bundled Java 25 at `%APPDATA%\PrismLauncher\java\java-runtime-epsilon` works).

```bash
git clone https://github.com/colsontabbert/datum.git
cd datum
git config core.hooksPath .githooks
uv sync
```

Run the runner (a placeholder for now):

```bash
uv run datum
```

Lint and test:

```bash
uv run ruff check
uv run ruff format --check
uv run pytest
```

Build the probe (from `probe/`, with `JAVA_HOME` pointing at a JDK 25):

```bash
./gradlew build
```

The jar lands in `probe/build/libs/`. It compiles against 26.2 by default; add `-Pminecraft_version=26.3 -Pfabric_api_version=0.161.0+26.3` to compile against 26.3.

## Status

Project scaffolding only. No benchmarking code yet.

Milestones (order updated after research):

- **M0, manual proof:** capture PresentMon by hand from a Fabric instance on OpenGL and Vulkan; record the game's backend and GPU log lines.
- **M1, runner skeleton:** session plan file, clone instance, launch through Prism, record, collect files.
- **M2, Fabric probe** for 26.2 and 26.3: camera path, capture markers, clean quit.
- **M3, honest numbers:** repeats, counterbalanced order, A/A check, statistics, `summary.csv` and `findings.md`.
- **M4, settings sweep** with backup and restore.
- **M5, mod testing** with dependency graph and group search.
- **M6, HTML report.**
- **Later:** datapack path for older versions and other loaders, docs and defaults for other people.

## Auto-push

Every commit pushes itself to `origin` through a git hook in `.githooks/`. A fresh clone needs this once:

```bash
git config core.hooksPath .githooks
```

## License

MIT. See [LICENSE](LICENSE).
