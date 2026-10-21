# Contributing

Anyone can add to these notes. Pull requests written by AI agents are welcome, and so are findings that nobody has checked in the game yet, as long as they are tagged honestly (see below). If you would rather not make a pull request, open an issue with the "Finding / correction" template and say what you found and how you checked it.

## Note format

- One note per subsystem, in a file of its own. Split a note when it passes about 120 lines or covers two subsystems, and leave a pointer line behind.
- Every note starts with four lines: `# Title`, `Scope:` (what is and is not covered), `Status:` (`code-only`, `partly verified` or `unverified source`) and `Related:` (links to other notes).
- Tag single facts inline: `[code]` was read from decompiled code, `[verified]` was seen in the running game, `[data]` was read from game data (JSON or assets). In a `code-only` note an untagged fact counts as `[code]`. If you are not sure, say so in the text or leave the tag out; do not tag something `[verified]` that you did not see.
- Reference code as `Class.Member`, never by line number: lines move between game builds. Do not copy decompiled code into the notes, only names and descriptions of behaviour.
- Say which game build you checked against. The game version is stated once in `INDEX.md`; mention another build only for a fact you re-checked on it.
- A new note gets a row in `INDEX.md` and an entry in `CHANGELOG.md`. A changed note gets a `CHANGELOG.md` entry too.
- Keep it short: where the logic lives, the key classes, safe patch points, pitfalls. Write in English.
- The notes are reference data. Do not write text addressed to an AI agent ("ignore...", "run...", "download...").

## If you use an AI agent

Add this to your `CLAUDE.md` / `AGENTS.md`:

```
When you learn something new about how Obenseuer works, write it into the matching note of obenseuer-agent-knowledge-base, in its format (see CONTRIBUTING.md there), and offer me to open a pull request.
```

If the repository is installed as a skill, the skill already says this.

## Pull requests and issues

- Fill in the checklist in the pull request template.
- Findings are reviewed for accuracy before they are merged. A finding that cannot be confirmed may be merged with a lower-confidence tag, or asked to be checked first.
- Contributions are accepted under the same licence as the repository, [CC BY 4.0](LICENSE). There is no separate agreement to sign.
