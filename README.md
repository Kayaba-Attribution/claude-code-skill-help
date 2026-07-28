# claude-code-skill-help

**See the usage, flags and arguments of any Claude Code skill, slash command or plugin.**

The slash-command menu shows one truncated line of a skill's description, and Claude Code has
no built-in way to inspect a single skill — there is no `claude skill details`. So you end up
`cat`-ing `SKILL.md` files to remember whether a command takes arguments.

```
/skill-help pr-review     -> purpose, flags, what it produces, examples
/skill-help codex         -> falls back to `claude plugin details` for a plugin
/skill-help               -> every skill and command, full untruncated descriptions
```

## Install

```bash
claude plugin marketplace add Kayaba-Attribution/claude-code-skill-help
claude plugin install skill-help
```

Then `/skill-help:usage <name>`.

## What it handles that a naive lookup doesn't

- **Two file layouts.** A slash command is backed by either `skills/<name>/SKILL.md` or
  `commands/<name>.md`, and a plugin's entries are often mostly the latter:
  `claude plugin details codex` reports 11 skills while only 3 exist as `SKILL.md` — the other
  8 are commands. Search one layout and you miss most plugin commands.
- **Two plugin locations.** The installed copy lives under
  `plugins/cache/<marketplace>/<plugin>/<version>/`, the checkout under
  `plugins/marketplaces/<marketplace>/{plugins,external_plugins}/<plugin>/`. The version-pinned
  one is authoritative and is the version it reports.
- **zsh glob semantics.** In zsh a glob matching nothing **aborts the whole command** rather
  than expanding to nothing, so a multi-path `ls a/* b/*` lookup fails outright the moment one
  path is empty. It uses `find`.
- **Authoritative usage first.** It reports the frontmatter `argument-hint` before falling back
  to a `## Usage` block, and only then to scanning the body for `--flag` tokens. If nothing
  documents any flags it says so instead of inventing them.

## Make your own skills self-describing

`argument-hint` is a real frontmatter field that Claude Code renders inline in the
slash-command menu — Anthropic's own `plugin-dev` skill documents it as "document expected
arguments for autocomplete", and it works in `SKILL.md`, not just `commands/*.md`:

```yaml
---
name: pr-review
description: What it does and when to use it (this is what auto-invocation matches on).
argument-hint: '[--fast] [--help] <PR refs> [doc-dir]'
---
```

For anything a single line can't express, add a `## Usage` section plus the rule *"if the
arguments contain `--help`, `-h`, or `?`, print this block verbatim and stop"*. Arguments are
passed straight through, so `/your-skill --help` then costs nothing and does nothing else.

Don't add hints to plugins you don't own: the installed copy sits under a version-pinned path,
so edits are discarded on the next plugin update.

## License

MIT — see [LICENSE](LICENSE).
