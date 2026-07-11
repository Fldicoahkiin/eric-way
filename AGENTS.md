# Agent Notes

- Keep the skill sources in `skills/` as the canonical repo copy.
- The same skills are also installed globally under `/Users/akrc/.codex/skills/`.
- When updating any `skills/eric-*` file in this repo, sync the matching global skill directory too.
- When a skill needs a file that already lives elsewhere in this repo (e.g. `docs/`), add a relative symlink under the skill's `references/` instead of copying the file. `install.sh` dereferences symlinks (`cp -RL`) at install time, so installed skills stay self-contained.
