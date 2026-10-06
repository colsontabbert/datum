# Interview notes: feature, target and optimal setup

Started 2026-10-06. Written after every round, so a new session can pick this up.

Branch `side/interview-target-optimal-setup`, draft pull request colsontabbert/datum#4.

Scope (chosen 2026-10-06, "Both"): (1) revisit the spec's "Target setup (v1)" section and the benchmark conditions in "Methodology"; (2) a new goal-seeking tuning feature: I give Datum a goal (for example "1% lows above 60 FPS at the highest render distance possible"), it searches settings for the best setup that meets it and recommends it.

## Where we are

- **Status:** in progress
- **Mode and size:** feature, medium
- **Now on:** Part 1, target setup and benchmark conditions, round 1
- **Parts left:** Part 1 (target setup), Part 2 (goal-seeking tuning)
- **Waiting on:** my answers to Part 1 round 1

## Firm from the brief

None new. Existing decisions that bind this interview: no composite score; it recommends, I decide (never applies settings to real instances); A/A noise floor every session; 4 runs minimum, 5 default; rank-based statistics; estimate shown before every session.

## Answers and decisions

### Part 1: target setup and benchmark conditions

(round 1 asked 2026-10-06, waiting)

### Part 2: goal-seeking tuning

(not started)

## Research findings

- Laptop, read 2026-10-06 (read only): model Vector A16 HX A8WHG; internal screen 2560x1600, running at 60 Hz (max 240 Hz); one display active; the RTX 5070 Ti drives the screen (the AMD 610M shows no active resolution, so the MUX looks set to discrete GPU: **unconfirmed** how Datum detects that); NVIDIA driver 617.14; GPU max power limit 140 W; Windows power plan "Balanced". (written into `docs/research.md`: not yet)
- Prism instances, read 2026-10-06 (read only): only `26.2 Vanilla` (no loader) and `Fabulously Optimized` (26.2, Fabric Loader 0.19.3). No FO 26.3 instance exists yet. FO plays fullscreen, vsync on, `maxFps:60`, render distance 12, simulation 10, GUI scale 4, backend `opengl`. Neither instance overrides memory, window, or JVM args. (written into `docs/research.md`: not yet)
- Prism global settings, read 2026-10-06: memory 512 to 4096 MB; JVM args `-XX:+UseZGC -XX:+AlwaysPreTouch -XX:+DisableExplicitGC -XX:+UseStringDeduplication`; window 854x480 (unused, since options.txt has `fullscreen:true`). Whether a clone's `OverrideJavaArgs=true` replaces these global args or adds to them is **unconfirmed** (check Prism 11.1.1 source). (written into `docs/research.md`: not yet)
- Versions, checked 2026-10-06 on Modrinth and GitHub: FO for 26.3 is still alpha (15.0.0-alpha.5, 2026-10-04); FO 14.1.0 is still the latest for 26.2; Prism 11.1.1 is still the latest release. (written into `docs/research.md`: not yet)

## Affects other parts

- Vanilla 26.2 has no Fabric loader, so the Fabric probe can't run there. The vanilla leg of the pack comparison (P1-16) needs a decision (see Part 1, question 12).
- If clone JVM args replace Prism's global ones, Flight Recorder setup (P1-05) must copy the global args into the clone, or runs lose ZGC and differ from real play.

## Parked for later

## Plan check
