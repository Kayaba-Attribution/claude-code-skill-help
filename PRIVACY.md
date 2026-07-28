# Privacy and data flow

**skill-help collects nothing and sends nothing anywhere.** No telemetry, no analytics,
no network calls, no API keys, no server operated by this project.

It is a read-only local lookup. It reads skill and command definition files already on
your machine:

- `~/.claude/skills/*/SKILL.md` (your own skills)
- `./.claude/skills/*/SKILL.md` (project skills)
- `~/.claude/plugins/**/skills/*/SKILL.md` and `~/.claude/plugins/**/commands/*.md`
  (plugin-provided entries)

It never writes to, edits, or deletes any of them — including plugin files, which live in
version-pinned directories where edits would be discarded on the next update anyway.

The contents of whichever file you ask about are shown to you in your own Claude Code
session, which is subject to Anthropic's usual data handling for that session. Nothing is
transmitted to the author or to any third party by this plugin.

Optionally it shells out to `claude plugin details` / `claude plugin list` when the name
you give matches a plugin rather than a skill. Those are local Claude Code CLI commands.
