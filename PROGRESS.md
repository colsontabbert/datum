# Progress Log

## 2026-10-05
**Done:**
- Project set up with /new-project: public repo https://github.com/colsontabbert/datum, MIT, auto-push hook on, CI green (Python on Windows; probe built for 26.2 and 26.3).
- Python runner scaffold (uv, Python 3.14, ruff, pytest) in `src/datum/`; empty Fabric probe in `probe/` (Loom 1.18.2, Gradle 9.7.1, one jar for 26.2 and 26.3), built locally with Prism's bundled Java 25.
- Research across five areas verified the brief's "Verify before building" list; findings, sources, and open items in `docs/research.md`.
- Decisions: rank-based statistics (Hodges-Lehmann plus exact Mann-Whitney interval) instead of bootstrap; probe before datapack; single probe jar, no Stonecutter. Brief has a pointer note.
- Confirmed from the game code that the 26.2/26.3 Friends List opt-in dialog only opens on a click or the O key, so it can't block Quick Play runs.

**Next:**
- M0 manual proof: find where Prism is installed, capture 60 s of PresentMon from the FO instance on OpenGL and on Vulkan, and record the game's "Using graphics backend" / "Using graphics device" log lines for both.
- Also in M0: check Prism issue #6073 (OpenGL startup crash on 26.3) against this setup.
