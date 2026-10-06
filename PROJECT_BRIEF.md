# Datum

> **Superseded by `docs/BUILD_PLAN.md`, `docs/SPEC.md`, and `docs/DECISIONS.md` on 2026-10-06; kept as the original brief.**

Named after a surveyor's datum: the fixed reference point everything else gets measured from.

A personal benchmarking and tuning tool for Minecraft Java Edition. It launches Minecraft over and over with different settings, mod sets, and modpacks, measures frame times from outside the game, and tells me in plain English what actually helps, what hurts, and how confident it is.

Built for me first (my laptop, my packs). Shareable with other people later. Not a business.

> **Updated after setup research (2026-10-05):** three decisions below are superseded. Comparisons use a rank-based method (Hodges-Lehmann shift + exact Mann-Whitney interval), not a bootstrap. The Fabric probe comes right after the runner skeleton, before the datapack. The probe is one jar for 26.2 and 26.3, no Stonecutter. Details, verified answers to the "Verify before building" list, and sources are in [docs/research.md](docs/research.md).

---

## Answers for /new-project

- **Tech stack:**
  - Runner + analysis: Python (latest stable version that all dependencies support), managed with uv.
  - In-game probe: small Fabric mod, Java 25, Gradle + Fabric Loom.
  - Portable fallback: a vanilla datapack (no mod needed).
  - Before adding any dependency, check that it's actively maintained and pick the best-maintained option, not just the most famous one.
- **Frontend:** yes, but only a generated static HTML report (no server, no web app). Apply the Tactile Web design language to it.
- **CI:** yes. Python: lint (ruff) + tests (pytest) on push. Probe: Gradle build for each Minecraft target.
- **colsontabbert-python-lib:** ask me when we get to config loading and logging. Probably useful there.
- **License:** MIT.
- **Repo:** public GitHub repo named `datum` under colsontabbert, auto push on.
- **Platform:** Windows only for v1.

---

## Why this exists (research summary, Oct 2026)

Pieces of this already exist, but nobody ties them together:

- **FPS Benchmark** (Modrinth, MIT): one-click scripted cinematic run in a temp world, 41 stress tests, A/B session compare. Fabric 1.21 to 1.21.1 only, about 740 downloads, not updated in months. Good reference for scenes and metrics. https://modrinth.com/mod/fps-benchmark
- **PotatoBench:** `/bench` command, 30 second run with a 360° camera spin for fairness. https://github.com/Hybridash/PotatoBench
- **vanilla-bench:** Python runner + Fabric probe, isolated instances, randomized multi-round runs, refuses to make a composite score. Closest to the architecture I want. 1.20.1 only. https://github.com/Trcmoe/vanilla-bench
- **brucethemoose's benchmarks:** used Prism Launcher + Intel PresentMon for client benchmarks. Proves the external measurement approach works. https://github.com/brucethemoose/Minecraft-Performance-Flags-Benchmarks
- **spark:** CPU profiler (client and server) that attributes time to mods. Blind to GPU-bound problems. https://spark.lucko.me
- **mod-bisect-tool:** bisects mod lists to find conflicts. https://github.com/Qendolin/mod-bisect-tool

The gap: automated A/B runs + settings sweeps + per-mod testing + honest statistics + plain-English verdicts, on current versions (26.x).

---

## Target environment (v1)

- **Machine:** MSI Vector A16 HX laptop, Ryzen 9 8940HX, RTX 5070 Ti laptop GPU, 16GB DDR5 (2x8GB), NVMe SSD, Windows. The CPU also has an integrated GPU, so every run must confirm Minecraft actually ran on the RTX card.
- **Launcher:** Prism Launcher.
- **Minecraft:** 26.2 and 26.3, Fabric. Both need Java 25.
  - 26.x is unobfuscated. Use the `net.fabricmc.fabric-loom` plugin, Mojang's official names, `implementation`/`compileOnly` instead of `modImplementation`, and `jar` instead of `remapJar`. Get current Loom/Loader/Fabric API versions from Fabric's develop site, don't guess.
- **Packs:**
  - Fabulously Optimized 14.1.0 for 26.2 (stable).
  - Fabulously Optimized 15.0.0-alpha.x for 26.3 (alpha; Sodium for 26.3 is also alpha, so 26.3 results are less trustworthy for now).
  - My usual instance = FO + extra mods. The runner should read the actual mod list from the instance instead of me typing it.
  - Distant Horizons lists 26.2 as its newest version as of early Oct 2026. Check 26.3 support before planning around it.
- **Graphics API:** 26.2+ has an OpenGL/Vulkan setting, and the 26.4 snapshots made Vulkan the default. Treat the API as a sweep axis.

---

## How it works

### Big picture

```
runner (Python, outside the game)
  -> clone/prepare a benchmark instance
  -> apply this run's settings + mod set
  -> copy a fresh frozen test world in
  -> launch via Prism CLI
  -> game auto-loads the world, scripted camera path runs
  -> PresentMon + JFR + nvidia-smi record
  -> runner stops capture, kills the game, restores everything
  -> repeat for every run in the plan
  -> analyze -> summary.csv + findings.md + report.html
```

**Design rule:** keep as much as possible outside Minecraft. Only the probe mod is tied to a game version, so updates (and the OpenGL to Vulkan switch) mostly only break the probe.

### Measurement (outside the game)

- **PresentMon console app** for frame timing. Works with OpenGL and Vulkan. Flags to start from (verify against the current README): `--process_name <java exe> --output_file <path> --timed <seconds> --terminate_after_timed --v2_metrics`.
  - Needs to run as admin or as a member of the "Performance Log Users" group. On Windows Home the Local Users and Groups screen may not exist; `net localgroup "Performance Log Users" <username> /add` from an admin terminal should work (verify).
  - Use the GPU Busy metric to label each scene as CPU-bound or GPU-bound. Check which PresentMon columns are actually valid for OpenGL/Vulkan apps, since the README notes some differ for apps that don't present through DXGI.
- **JFR (Java Flight Recorder)** via JVM args for GC pauses, heap, and allocation. Same flags on every run so overhead is equal.
- **nvidia-smi** logging during each run (GPU name, clocks, temp, power) to catch thermal throttling and wrong-GPU runs.
- **Per-run system snapshot:** GPU actually used, graphics API actually used, driver version, AC vs battery, Java version, JVM args, Minecraft version, loader version, full mod list with file hashes, hashes of every config file touched.
- **Graphics API check (same idea as the wrong-GPU check):** the API a run is labeled with means nothing until the game confirms it. Read the active backend from what the game itself reports (startup log line, or the probe in Tier 1), not from what the runner wrote into options.txt. A run planned as Vulkan that actually ran on OpenGL (or the other way around) is thrown out.
  - Re-read the API setting in options.txt before every run too. Before 26.4, the game could switch the API setting on its own after a startup crash, so one bad Vulkan launch could quietly turn every later run into OpenGL.

### Driving the game: two tiers

**Tier 0: no mod, works on any version and loader (including vanilla)**
- Auto-join the world with Minecraft's Quick Play (`--quickPlaySingleplayer`) or Prism's Quick Play instance setting (added in Prism 9.0). Figure out how the runner can set this per instance (instance.cfg key or CLI flag).
- A bench datapack with a tick function: puts the player in spectator, moves along a waypoint path with `/tp`, and prints marker lines (`BENCH_WARMUP_START`, `BENCH_CAPTURE_START`, `BENCH_END`). The runner tails `latest.log` for the markers to start and stop PresentMon. Verify these messages actually show up in `latest.log` on 26.x.
- Limits: no tick-time breakdown, no clean quit (runner kills the process; the world is a throwaway copy so that's fine).

**Tier 1: Fabric probe for 26.2 and 26.3**
- Same job, cleaner: smooth interpolated camera path, writes markers to its own file, records client tick time, chunk section rebuilds, loaded chunks, entity counts, and the active graphics backend, then quits the game cleanly.
- Multi-version setup: either two Gradle subprojects or Stonecutter (check it's maintained and supports 26.x first).
- The probe must be installed identically in every run being compared.

### Frozen benchmark worlds

- One golden world per Minecraft version (and per worldgen mod set, since worldgen mods change the terrain).
- Pregenerate the test area. If Distant Horizons is installed, run `/dh pregen` and wait for it to finish, then turn DH's distant generation off for benchmark runs so I'm not measuring background LOD building.
- Store golden worlds in the project's data folder. Copy a fresh one into the instance before every run, delete it after.
- Scenes:
  1. **Overlook:** slow pan at max view distance. GPU heavy.
  2. **Flight:** straight line at speed through pregenerated terrain. Chunk and LOD streaming.
  3. **Crowd:** village or mob farm with lots of entities. CPU heavy.
  4. **New terrain (optional, separate):** fly into ungenerated chunks. Only way to see worldgen mods like C2ME. Expect more noise.
- If two compared configs have different worldgen mods, the worlds differ. The report has to say so.

### Changing things between runs

- **Settings:** edit options.txt, Sodium options, Iris shader settings, DH config, and JVM heap (Prism instance config). Back up originals and restore after every run, including after crashes.
- **Mods:** disable by renaming the same way Prism does when you untick a mod (verify the suffix). Never delete a jar.
- **Dependencies:** parse `fabric.mod.json` in every jar. Never disable a library something else needs; disable its dependents with it.
- **Safety:** only work on cloned instances made for benchmarking. Never touch my real instances or saves.

---

## Methodology rules (non-negotiable)

- **A/A check first:** every session starts by running the same config against itself. That measures the noise floor. Any difference smaller than the noise floor gets reported as noise.
- **Warmup:** skip the first chunk of each run (JIT, chunk loading), plus a short thermal warmup at the start of a session.
- **Repeats:** at least 5 runs per config by default.
- **Order:** randomized and interleaved (A B B A A B...), never all A then all B. Laptop heat drifts over a session.
- **Conditions:** plugged in, same MSI Center mode, same resolution, vsync off, FPS cap off. Runner checks what it can and warns about the rest.
- **Per-run metrics:** average FPS, median frame time, 1% low and 0.1% low (defined as average FPS of the slowest 1% / 0.1% of frames; document this in the report), p99 frame time, worst frame, stutter count (frames over 2x the median), GPU busy ratio, GC pause total and max, peak heap.
- **Comparisons:** median of per-run metrics per config, bootstrap confidence interval on the difference. If the interval crosses zero or the difference is under the noise floor, say "no measurable difference."
- **No composite score.** Every metric gets read on its own.
- **Throw out and rerun:** wrong GPU, wrong graphics API, on battery, crash, missing markers, heavy throttling.
- **Show an ETA** before starting any session.

---

## Experiments (priority order)

1. **A/A noise check.**
2. **Settings sweep:** render distance, simulation distance, graphics API (OpenGL vs Vulkan), shaders on/off and shader profile, DH LOD distance and quality (if installed), heap size (e.g. 4 / 6 / 8 GB on a 16GB machine), key Sodium options.
   - Only include Vulkan for a mod set after confirming every mod in it actually works on the Vulkan backend (see verify list). If a mod forces OpenGL or breaks on Vulkan, the report should say that instead of showing a fake comparison.
3. **Pack comparison** on the same version: vanilla vs FO vs FO + my mods vs a big stress pack.
4. **Mod testing (leave-one-out)** for client, performance, and visual mods only. Content and worldgen mods are excluded since removing them changes the world; use spark profiles for those instead. For big mod lists, start with group toggles (all visual mods off, all perf mods off) and narrow down.
5. **Version comparison:** same pack on 26.2 vs 26.3 (different worlds, flag that).

---

## Bigger modpacks

Most big packs (All the Mods, Better MC, Prominence, etc.) live on 1.20.1 or 1.21.1, mostly on (Neo)Forge. 26.x doesn't have many large packs yet.

- **v1:** build my own stress pack on 26.2: FO + 100 to 200 popular Fabric mods that support 26.2 (content plus visuals). Later the runner could assemble it from a list through the Modrinth API.
- **Later:** Tier 0 mode means older big packs (1.21.1 NeoForge, etc.) can be benchmarked without porting the probe.

---

## Outputs

```
results/<session-id>/
  session.json        plan, system info, versions, mod lists + hashes
  runs/<run-id>/
    presentmon.csv
    recording.jfr
    latest.log
    nvidia-smi.csv
    run.json
  summary.csv
  findings.md         plain English
  report.html         static, Tactile Web
```

Example of the findings tone (format only, these numbers are made up):

> Vulkan vs OpenGL (26.2, FO, overlook scene): Vulkan +18% avg FPS, 1% lows +9% (95% CI +5% to +13%). GPU-bound both ways. Worth switching.
>
> Entity Culling on vs off (crowd scene): no measurable difference. Noise floor this session was ±3%.

---

## Milestones

- **M0, manual proof:** install PresentMon, launch my FO instance by hand, capture 60 seconds with the CLI, look at the CSV. Confirm the java process name and that both OpenGL and Vulkan get captured.
- **M1, runner skeleton:** session plan file (TOML), clone instance, launch through Prism CLI, start/stop PresentMon on a timer, kill the game, collect files.
- **M2, repeatable runs:** frozen world, Tier 0 datapack path, log markers, fresh world copy per run.
- **M3, honest numbers:** repeats, interleaving, A/A check, stats, summary.csv + findings.md.
- **M4, settings sweep** with backup and restore.
- **M5, Fabric probe** for 26.2, then 26.3.
- **M6, mod testing** with dependency graph, group search, ETA.
- **M7, HTML report.**
- **M8 (later, for other people):** docs, sane defaults, a default test world, more versions and loaders, shareable reports, maybe a GUI.

---

## Not doing in v1

- No in-game GUI.
- No auto-applying "recommended" settings. It recommends, I decide.
- No crowdsourced results database.
- No Mac or Linux.
- No server TPS benchmarking (spark already covers servers).

---

## Verify before building (don't guess these)

- Prism CLI: `prismlauncher.exe --launch <instance ID>` (ID is the instance folder name). Confirm it works on Windows and whether the launcher window can stay out of the way.
- How to set Quick Play singleplayer per Prism instance from a script.
- Prism's disabled-mod file naming.
- Java process name under Prism on Windows (java.exe vs javaw.exe) for PresentMon.
- Current PresentMon console version, flags, and which columns are valid for OpenGL/Vulkan.
- options.txt key for the graphics API on 26.2/26.3.
- How to tell from outside the game which API a run actually used (a startup line in `latest.log`? something else?).
- Which mods in my packs (Sodium, Iris, DH, and the rest) work on the Vulkan backend on 26.2/26.3, and whether any force OpenGL, fall back silently, or crash.
- Whether 26.2/26.3 switch the API setting by themselves after a startup crash, and how to detect when that happened.
- Whether datapack `tellraw`/`say` output lands in `latest.log` on 26.x.
- Distant Horizons, Iris, and Sodium versions for 26.2 and 26.3.
- Current Fabric Loom, Loader, and Fabric API versions for 26.2/26.3.
- Stonecutter status, if used.

---

## References

- Fabric 26.1 changes (unobfuscated, Loom, Java 25): https://fabricmc.net/2026/03/14/261.html
- Fabric porting docs: https://docs.fabricmc.net/develop/porting/
- PresentMon console README: https://github.com/GameTechDev/PresentMon/blob/main/README-ConsoleApplication.md
- Prism Quick Play PR: https://github.com/PrismLauncher/PrismLauncher/pull/2716
- Mojang Vulkan announcement: https://www.minecraft.net/en-us/article/another-step-towards-vibrant-visuals-for-java-edition
- 26.4 Snapshot 1 (Vulkan default): https://www.minecraft.net/article/minecraft-26-4-snapshot-1
- Fabulously Optimized releases: https://github.com/Fabulously-Optimized/fabulously-optimized/releases
- spark finding lag spikes guide: https://spark.lucko.me/docs/guides/Finding-lag-spikes
