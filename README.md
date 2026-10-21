# Obenseuer Agent Knowledge Base

Notes on how Obenseuer works inside, written for AI agents that write mods. Short notes for each game subsystem: where the logic lives, the key classes and methods, safe patch points, and pitfalls. Every fact is tagged by how it was checked. Give your agent `INDEX.md` and the one note it needs, instead of decompiling the game.

This is knowledge, not a library. There is nothing to install and no runtime dependency; the notes describe the game as it is, and everything in them works with plain BepInEx and Harmony.

## What is here

The notes were written against Obenseuer 0.5.08 (Unity 6, Mono), for mods built on BepInEx 6 (Unity Mono) with HarmonyX. Game code is `Assembly-CSharp.dll` in `Obenseuer_Data\Managed`. Targets must be `netstandard2.1`; assemblies built for net6 or newer do not load in Unity Mono. Later game builds may have changed details, so a note that matters for your mod is worth a quick check against your own decompiled copy.

Each note covers one subsystem (inventory and storage, crafting, power, water, growing, saves, police and crime, and so on). Code is referenced as `Class.Member`, never by line number, because line numbers move between builds. No decompiled game code is copied here, only names and descriptions of behaviour.

## Setup

The repository is an agent skill: `SKILL.md` tells the agent how to keep the notes up to date, how to read them and how to record new findings.

Claude Code: clone it into your mod project's skills folder, or into `~/.claude/skills/` to have it in every project. The skill loads by itself when a task touches Obenseuer, and once a day it checks GitHub for a new version. If there is one, it shows you what changed and updates only with your yes.

```bash
git clone https://github.com/absurdmane/obenseuer-agent-knowledge-base .claude/skills/obenseuer-agent-knowledge-base
```

Other agents (Codex, Cursor and others): clone it anywhere in your project and add one line to your `AGENTS.md` or rules file, with your path: `For anything about Obenseuer game code or mechanics, read docs/obenseuer-agent-knowledge-base/SKILL.md first and follow it.`

## How to use it with an AI agent

1. Put `INDEX.md` into the agent's context. It lists every note with a one-line description.
2. Let the agent open one note at a time, the one for the subsystem you are working on. Do not load all notes at once; most tasks need one or two.
3. Ask the agent to follow the patch points and pitfalls in the note and to say when something is not covered, so you know what to look up in the decompiled code.

## How to read the tags

Every note starts with four lines: a title, `Scope` (what is and is not covered), `Status` and `Related` (links to other notes). `Status` says how the note was checked:

- `code-only`: read from decompiled code, not tested in the game. Treat as likely correct but unverified.
- `partly verified`: some items were seen in the running game.
- `unverified source`: not from the game code (for example the Steam page, or text digested from game files without playing).

Inline tags mark single facts: `[code]` was read from decompiled code, `[verified]` was seen in the game, `[data]` was read from game data (JSON or asset files). In a `code-only` note an untagged fact is `[code]`.

## Spoilers

The `lore/` folder digests the game's dialogue and texts and tells the whole story. Do not open it if you want to play the game unspoiled. The other notes are about mechanics.

## Regenerating the sources yourself

The notes do not ship the decompiled code. To get your own copy, install `ilspycmd` (the command-line ILSpy, `dotnet tool install -g ilspycmd`) and decompile `Obenseuer_Data\Managed\Assembly-CSharp.dll` into one file per type (the `-p` project mode with `-o <folder>` does this). Then search it with a normal text search for a class or field name and read only the lines you need; the files are large. `asset-bundles.md` describes how to read prefab and scene data (values that are not visible in code) offline with UnityPy.

## Contributing

Pull requests with findings are welcome. Put a fact into the note of the subsystem it belongs to, tag it as above, say which game build you checked it against, and keep to `Class.Member` references without copied code. A new note needs the four header lines and a row in `INDEX.md`.

## License

This work is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). You may share and adapt it, including commercially, as long as you give credit. The full text is in `LICENSE`. Obenseuer and its content belong to their owners; these notes only describe how the game works.
