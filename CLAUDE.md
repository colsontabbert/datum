# Standing preferences for Datum

Keep this file and `AGENTS.md` identical. Edit both together.

- Before recommending a library or tool, check it's actively maintained and current, don't default to whichever is most well-known.
- Don't assume facts about the project or about me that haven't been stated directly, ask instead of guessing.
- Verify claims and cross-check sources before stating something as settled fact.
- No em dashes in anything written for this project.
- Give direct, unsoftened feedback rather than a hedged version.
- How work runs here: `docs/WORKFLOW.md`. The plan: `docs/BUILD_PLAN.md` and `docs/ROADMAP.md`. Build with `/card`, repo work with `/side`, fixes with `/patch`, planning with `/interview` (Codex: `$card`, `$side`, `$patch`, `$interview`).
- Read `docs/design-language.md` before any UI work (the desktop GUI and the generated `report.html`) and follow it; don't default to generic modern SaaS or Tailwind conventions. Until P1-19 writes it, the "The look" section of `docs/SPEC.md` is the reference. Tactile Web (`docs/archive/tactile-web/`) is not used on this project.
- No feature may need a multi-key shortcut: every shortcut also has a clickable way to do it, since I mostly use a touchpad, touchscreens, dictation, and one-finger typing.
- Long text fields get a dictation button.
- On Python projects that depend on colsontabbert-python-lib: use it instead of rewriting retry/backoff, rate limiting, an API client wrapper, env/config loading, or logging setup boilerplate. Only add something new to colsontabbert-python-lib itself after the same code has been written the same way in three separate projects, not before.
- Datum does not use colsontabbert-python-lib (decided 2026-10-06, see `docs/DECISIONS.md`): it's private while Datum is public, and Datum's config is TOML. Use the built-in `tomllib` and `logging`.

## Project context

- Plan: `docs/BUILD_PLAN.md`, `docs/SPEC.md`, and `docs/DECISIONS.md` (`PROJECT_BRIEF.md` is the original brief). Verified facts and sources: `docs/research.md`. Read them before planning work.
- Python runner in `src/datum/` (uv, Python 3.14). Fabric probe in `probe/` (Gradle, Loom, Java 25), one jar for Minecraft 26.2 and 26.3.
- Only ever touch cloned benchmark instances. Never modify the user's real Prism instances or saves.

## Working notes

- Build the probe locally from `probe/`: `JAVA_HOME="$APPDATA/PrismLauncher/java/java-runtime-epsilon" ./gradlew.bat build`. System Java is 21; Prism's bundled Java 25 is a full JDK (has javac, jfr, jcmd).
- When release notes or wikis disagree with expected game behavior, read the client jar: 26.x is unobfuscated, so `javap -c -p` on its classes shows the real logic.
- Prism's wiki CLI page is out of date. For launcher behavior, read Prism's source at the release tag in use.
- When the first pytest test lands, remove the `|| [ $? -eq 5 ]` allowance from the pytest step in `.github/workflows/ci.yml`.
- Never put the user's email or other personal info in HTTP headers (e.g. Modrinth's User-Agent). Ask what contact info, if any, to use.
