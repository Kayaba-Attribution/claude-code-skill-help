---
name: usage
description: >-
  Prints the usage, flags and examples for any installed skill, command or plugin —
  user, project or plugin-provided — by reading its source file, because the slash
  menu only shows a truncated one-line description. Use when the user asks what a
  skill does, what flags or options it takes, how to invoke it, or wants the full
  untruncated list of what is available.
argument-hint: '[skill-name | plugin-name]   (no args = list everything)'
disable-model-invocation: true
---

# skill-help

Read-only. Never edit the target skill, and never invoke it — the user asked what it
does, not for it to run.

## Resolve the name

Strip a leading `/`. A name like `codex:rescue` means plugin `codex`, entry `rescue`.

Two file layouts back a slash command, and you must search both — a plugin's entries are
often mostly commands. `claude plugin details codex` lists 11 "skills" while only 3 exist
as `skills/*/SKILL.md`; the other 8 are `commands/*.md`.

```bash
NAME=rescue      # bare name, no plugin prefix
find ~/.claude/skills ./.claude/skills ~/.claude/plugins \
     -type f \( -path "*/skills/$NAME/SKILL.md" -o -path "*/commands/$NAME.md" \) 2>/dev/null
```

Use `find`, not shell globs: in zsh a glob that matches nothing aborts the whole command
instead of expanding to nothing, so a `ls a/* b/*` lookup fails outright.

Precedence: `~/.claude/skills` (user) → `./.claude/skills` (project) → plugin. Plugin
copies live in two places — `plugins/cache/<marketplace>/<plugin>/<version>/…` (the
installed, version-pinned copy, prefer this and report the version) and
`plugins/marketplaces/<marketplace>/{plugins,external_plugins}/<plugin>/…` (the checkout).

If nothing matches, the argument may name a *plugin* rather than one of its entries — run
`claude plugin details <name>` for its component inventory and projected token cost, and
`claude plugin list` when the name is unfamiliar. Those are the built-ins for plugin-level
questions; there is no built-in equivalent for a single skill.

## Print

Read the whole file, then emit:

1. **Name and source** — `user`, `project`, or `plugin <name>@<version>`, with the path.
2. **Purpose** — one line, from the frontmatter description.
3. **Usage** — in this order: the frontmatter `argument-hint` (the authoritative,
   author-declared argument list); then a `## Usage` section or fenced help block, printed
   verbatim. If neither exists, build one: scan the body for `--flag` tokens and for
   `Flags`/`Options`/`Arguments` tables. Say explicitly when nothing documents any flags
   rather than inventing them.
4. **What it produces** — output files, docs, or side effects, if the skill names them.
5. **Extra files** — anything alongside SKILL.md (`scripts/`, `templates.md`, configs);
   `ls` the directory. Flag a `config.local.json` as required setup if the body says so.
6. **Examples** — the skill's own examples if present, otherwise one realistic invocation.

Keep it under ~25 lines unless the user asks for the full text. Do not paraphrase flag
semantics loosely: a wrong flag description is worse than pointing at the file.

## No arguments

List everything grouped by source with its **full, untruncated** purpose — that is the
whole point, since the menu cuts descriptions off at terminal width:

```bash
find ~/.claude/skills ./.claude/skills ~/.claude/plugins/cache \
     -type f \( -name SKILL.md -o -path "*/commands/*.md" \) 2>/dev/null
```

User and project entries first, plugin ones after as `plugin:name`. Mark which take
arguments (they have an `argument-hint`) and point at `/skill-help <name>` for detail.

## Making your own skills self-documenting

- **`argument-hint` in frontmatter is the built-in mechanism** — Anthropic's own plugin-dev
  skill documents it as "document expected arguments for autocomplete", and it works in
  `SKILL.md` as well as `commands/*.md`:

  ```yaml
  argument-hint: '[--fast] [--help] <PR refs> [doc-dir]'
  ```

  Prefer this over stuffing flags into the description. `codex:rescue` is a good example of
  a rich one.
- **Handle `--help` in the body** for anything a hint can't express: a `## Usage` section
  plus the rule *"if the arguments contain `--help`, `-h`, or `?`, print this block verbatim
  and stop"*. Arguments pass straight through, so `/my-skill --help` then costs nothing and
  does nothing else.
- Keep the description for *when to use it*, since that is what model-invocation matches on.

For plugin-provided skills you don't own: read them, never patch them. The installed copy
sits under a **version-pinned** path (`…/codex/1.0.5/…`), so any edit is discarded by the
next plugin update, and the marketplace checkout is a git working copy that will conflict.
If you need a plugin's behaviour changed, wrap it in your own skill that calls it, or fork
the plugin. To ship your own skills with these conventions baked in,
`claude plugin init <name>` scaffolds into `~/.claude/skills/<name>/` and auto-loads as
`<name>@skills-dir`.
