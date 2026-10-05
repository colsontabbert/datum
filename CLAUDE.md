# Standing preferences for Datum

Keep this file and `AGENTS.md` identical. Edit both together.

- Before recommending a library or tool, check it's actively maintained and current, don't default to whichever is most well-known.
- Don't assume facts about the project or about me that haven't been stated directly, ask instead of guessing.
- Verify claims and cross-check sources before stating something as settled fact.
- No em dashes in anything written for this project.
- Give direct, unsoftened feedback rather than a hedged version.
- If `docs/design-language.md` and `docs/design-tokens.md` exist in this repo, read both before doing any UI/frontend work and follow them, don't default to generic modern SaaS/Tailwind conventions. Here that means the generated `report.html`.
- On Python projects that depend on colsontabbert-python-lib: use it instead of rewriting retry/backoff, rate limiting, an API client wrapper, env/config loading, or logging setup boilerplate. Only add something new to colsontabbert-python-lib itself after the same code has been written the same way in three separate projects, not before.
- Datum does not depend on colsontabbert-python-lib yet. Ask me whether to add it when work reaches config loading and logging.

## Project context

- Plan: `PROJECT_BRIEF.md`. Verified facts, decisions that supersede the brief, and sources: `docs/research.md`. Read both before planning work.
- Python runner in `src/datum/` (uv, Python 3.14). Fabric probe in `probe/` (Gradle, Loom, Java 25), one jar for Minecraft 26.2 and 26.3.
- Only ever touch cloned benchmark instances. Never modify the user's real Prism instances or saves.

## Working notes

- Build the probe locally from `probe/`: `JAVA_HOME="$APPDATA/PrismLauncher/java/java-runtime-epsilon" ./gradlew.bat build`. System Java is 21; Prism's bundled Java 25 is a full JDK (has javac, jfr, jcmd).
- When release notes or wikis disagree with expected game behavior, read the client jar: 26.x is unobfuscated, so `javap -c -p` on its classes shows the real logic.
- Prism's wiki CLI page is out of date. For launcher behavior, read Prism's source at the release tag in use.
- When the first pytest test lands, remove the `|| [ $? -eq 5 ]` allowance from the pytest step in `.github/workflows/ci.yml`.
- Never put the user's email or other personal info in HTTP headers (e.g. Modrinth's User-Agent). Ask what contact info, if any, to use.
