# Research notes (verified 2026-10-05)

What setup research confirmed or changed relative to `PROJECT_BRIEF.md`. Items marked **unconfirmed** still need a real test before code depends on them. Recheck versions before relying on them; this is a snapshot.

## Decisions taken from this research

- **Statistics:** rank-based comparison replaces the bootstrap. Effect = Hodges-Lehmann shift (median of all pairwise A-minus-B differences, on log metrics so results read as percentages). Interval = exact Mann-Whitney inversion (5 vs 5 runs: 3rd smallest to 3rd largest pairwise difference, about 96.8% coverage). "No measurable difference" only when the interval sits inside the A/A noise band. Reason: a percentile bootstrap with 5 runs per side covered only about 87% in simulation instead of 95%. With 3 runs per side no difference can ever be significant; 4 is the floor, 5 the default.
- **Milestones:** the Fabric probe comes right after the runner skeleton. The datapack (Tier 0) stays in the plan for older versions and other loaders later. Reasons are under "Driving the game".
- **Probe:** one jar for 26.2 and 26.3, compiled against 26.2, no Stonecutter. CI also compiles against 26.3. Whether one jar actually loads on both is **unconfirmed** until tested in game.

## Versions (as of 2026-10-05)

| Thing | Version |
|---|---|
| Minecraft | 26.2 (2026-06-16), 26.3 (2026-09-15), 26.4 in snapshots. All need Java 25. |
| Fabric Loader / Loom / Gradle | 0.19.5 / 1.18.2 / 9.7.1 wrapper |
| Fabric API | 0.161.0+26.2, 0.161.0+26.3 |
| Sodium | mc26.2-0.9.2, mc26.3-0.9.2 |
| Iris | 1.11.4+26.2, 1.11.7+26.3 |
| Distant Horizons | 3.3.3-26.2, 3.3.4-26.3 (26.3 is supported) |
| Fabulously Optimized | 14.1.0 (26.2), 15.0.0-alpha.5 (26.3) |
| Prism Launcher | 11.1.1 (2026-09-28) |
| PresentMon | 2.6.0 (2026-09-21) |
| Python / uv / ruff / pytest | 3.14 / 0.12.x / 0.16.10 / 9.1.1 |

Always filter Modrinth queries with `loaders=["fabric"]`; an unfiltered 26.3 query returned the NeoForge Sodium build first.

## Graphics API

- options.txt key: `preferredGraphicsBackend`, values `default`, `opengl`, `vulkan`. Launch argument `--graphicsBackend` also exists.
- "Default" means OpenGL in 26.2 and 26.3, and Vulkan from 26.4. **Always write `opengl` or `vulkan` explicitly.**
- Both "Prefer" values fall back silently to the other API at runtime without changing options.txt.
- 26.2 and 26.3 rewrite the setting after a startup crash (Vulkan to Default, Default to OpenGL). 26.4 stops doing this. Compare options.txt before and after every run.
- The game's own log names the backend and the GPU (seen in a real 26.2 Vulkan log):
  ```
  [Render thread/INFO]: Using graphics backend Vulkan, using drivers: ...
  [Render thread/INFO]: Using graphics device: <GPU name> (<vendor>)
  ```
  This one check catches wrong-API and wrong-GPU runs. The OpenGL wording of these lines is **unconfirmed**; capture one in M0.
- Iris cannot render shaders on Vulkan, so shader tests are OpenGL only. With Iris installed, `--graphicsBackend vulkan` crashes (Iris #3357); set the API through options.txt only.
- Distant Horizons has its own renderer setting (Auto, Blaze3D, OpenGL). Pin it per run.
- Sodium supports Vulkan experimentally since 0.9.0. ImmediatelyFast says it is Vulkan compatible. Lithium, Entity Culling, More Culling, Continuity: **unconfirmed**.
- An early 26.2 snapshot test found Vulkan 21 to 31% faster on NVIDIA. Probably stale.

## Prism Launcher (read from 11.1.1 source)

- Launch: `prismlauncher.exe --launch <instance folder name> --world <save folder name>`. `--world` exists since Prism 9.0 even though the wiki's CLI page omits it. Prism turns it into `--quickPlaySingleplayer`, and its metadata marks 26.2 and 26.3 as supporting that.
- If Prism is already running, a second `--launch` forwards to it and exits. Prism does not wait for the game, so the runner must find the game process itself (child `javaw.exe` of Prism). Do not match on the name alone.
- Default Java executable on Windows is `javaw.exe`.
- instance.cfg is a Qt INI file (`[General]`). Keys: `OverrideMemory` + `MinMemAlloc`/`MaxMemAlloc` (MB), `OverrideJavaArgs` + `JvmArgs`, `OverrideJavaLocation` + `JavaPath`, `OverrideConsole` + `ShowConsoleOnError`. Prism reloads it right before each launch; edit only between runs with the settings window closed.
- Killing the game counts as a crash: Prism opens a console window and stays alive. Set `OverrideConsole=true`, `ShowConsoleOnError=false` in benchmark clones.
- Disabled mods are renamed to `<name>.jar.disabled`.
- Clone by copying to a temp folder on the same drive, removing the `uuid` line, then renaming into `instances/`.
- Each new Prism process refreshes the Microsoft login once. If that fails, a modal login dialog blocks unattended runs.
- Game folder is `minecraft` unless only `.minecraft` exists. Versions are in `mmc-pack.json` (`net.minecraft`, `net.fabricmc.fabric-loader`).
- Open Prism issue #6073: OpenGL startup crash on 26.3 test builds under Prism. Check in M0.

## Measurement

- **PresentMon 2.6.0:** target by `--process_id`, not `--process_name`. Default columns are the newest set; `--v2_metrics` now switches to the older names. Parse columns by header name and pin the version.
- For OpenGL/Vulkan (runtime "Other"), `MsInPresentAPI` is meaningless. `MsBetweenPresents`, `MsBetweenDisplayChange` and `MsGPUBusy` work. GPU busy reads about 0.5 ms high with hardware-accelerated GPU scheduling on; record that setting per run.
- Run PresentMon continuously with `--qpc_time` and trim by timestamps, rather than starting and stopping it. Stop it cleanly with Ctrl+Break.
- Permissions: admin or membership in "Performance Log Users". This machine's user is already a member.
- **nvidia-smi:** `--query-gpu=timestamp,pstate,clocks.gr,clocks.sm,power.draw,temperature.gpu,utilization.gpu,clocks_event_reasons.active,... --format=csv,nounits -lms 500 -f out.csv`. `nvidia-smi -q -x` lists graphics processes by PID on Windows, which confirms the game ran on the RTX card.
- **JFR:** `-XX:StartFlightRecording=filename=run.jfr,settings=default,dumponexit=true`. Data is only written on a clean JVM exit; a killed game loses it. Parse with the JDK's `jfr print --json` (no maintained Python parser exists). Prism's bundled Java 25 (`java-runtime-epsilon`) includes `jfr.exe` and `jcmd.exe`.
- Set `inactivityFpsLimit:minimized` in options.txt. The default `afk` lowers FPS when there is no input and would ruin unattended runs.
- Other first-run keys: `onboardAccessibility:false`, `tutorialStep:none`, `skipMultiplayerWarning:true`, `narrator:0`. `graphicsPreset` replaced `graphicsMode`.
- The 26.2 Friends List opt-in dialog only opens when the Friends button is clicked or the Friends key (O) is pressed (checked in the 26.2 and 26.3 game code). Quick Play runs never trigger it.

## Driving the game

Why the probe comes before the datapack on 26.x:
- `/tp` every tick moves the camera 20 times a second, which judders at high FPS. The smoother datapack trick is spectating an `item_display` moved with `teleport_duration` (see Cinemalya).
- Datapack chat output reaching `latest.log` is **unconfirmed** on 26.x, and 26.3 removed some log lines.
- The datapack path can only end a run by killing the game, which loses JFR data.

Probe notes: `ClientTickEvents` (Fabric API) for ticks, `ClientLevel.getEntityCount()`, `Minecraft.stop()` for a clean quit. The backend name moved packages between 26.2 and 26.3, but the log line above makes that unnecessary. With Sodium installed, vanilla chunk counters are meaningless and rebuild counts need Sodium internals, so treat that metric as optional. 26.3 replaced GLFW with SDL3 for windowing and input.

Datapack formats if Tier 0 is built later: 26.2 = 107.1, 26.3 = 121.0. Use a `min_format`/`max_format` range.

## Methodology additions

- Counterbalanced ABBA order, not just random. Discard the first launch of each session.
- Wait for a target temperature between runs instead of a fixed delay. Log AC power, clocks and throttle reasons per run.
- Compare the first and second half of each run to catch drift inside a run (JIT may never settle: Barrett et al. 2017).
- Compute metrics per run and take medians across runs. Never pool frames across runs.
- "1% low" as the average of the slowest 1% of frames favors configs that render more frames. Label the definition and show p99 frame time beside it.
- Require identical `PresentMode` across compared configs.

## Prior art (beyond the brief)

- **agentpixelated/minecraft-benchmark** (MIT, Aug to Sep 2026): Python, 26.2 only, OpenGL vs Vulkan with Sodium stacks, ABBA order, rejects runs whose log does not prove the backend. Closest modern match.
- **vanilla-bench** (updated 2026-09-24, still 1.20.1): fresh instance and world per launch, settings checked by the probe, validity gates (frame coverage, focus loss), fingerprints of mods and configs, failed runs kept with a reason. Worth copying.
- Neither does settings sweeps, per-mod testing, 26.3, or small-sample statistics.

## Sources

- Prism source at tag 11.1.1: https://github.com/PrismLauncher/PrismLauncher/tree/11.1.1/launcher
- Prism metadata: https://meta.prismlauncher.org/v1/net.minecraft/26.2.json
- PresentMon: https://github.com/GameTechDev/PresentMon (README-ConsoleApplication.md, CsvOutput.cpp)
- Minecraft options.txt: https://minecraft.wiki/w/Options.txt
- Quick Play: https://minecraft.wiki/w/Quick_Play
- Pack formats: https://minecraft.wiki/w/Pack_format
- 26.2 release notes: https://minecraft.wiki/w/Java_Edition_26.2
- Fabric 26.1 changes: https://fabricmc.net/2026/03/14/261.html
- Fabric example mod: https://github.com/FabricMC/fabric-example-mod (branches 26.2, 26.3)
- Iris Vulkan crash: https://github.com/IrisShaders/Iris/issues/3357
- Prism OpenGL issue: https://github.com/PrismLauncher/PrismLauncher/issues/6073
- Early Vulkan test: https://nemez.net/posts/20260410-minecraft-snapshot-opengl-vs-vulkan-nvidia-amd-intel/
- Small-sample bootstrap: Hesterberg 2015, https://arxiv.org/abs/1411.5279
- JIT warmup: Barrett et al. 2017, https://arxiv.org/abs/1602.00602
- CapFrameX metric definitions: https://www.capframex.com/blog/post/Explanation%20of%20different%20performance%20metrics
- Cinemalya camera approach: https://github.com/Stoupy51/Cinemalya
- agentpixelated/minecraft-benchmark: https://github.com/agentpixelated/minecraft-benchmark
- vanilla-bench: https://github.com/Trcmoe/vanilla-bench
