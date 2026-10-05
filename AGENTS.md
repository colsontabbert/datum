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
