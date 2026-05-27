# lead-eng

A Claude Code skill that translates product intuition into engineer-ready GitHub issues via the `gh` CLI. Produces one parent epic + 3–12 child issues, each grounded in real file paths and line numbers, with explicit out-of-scope guardrails so agents picking up the work don't scope-creep.

Built to be invoked as `/lead-eng` inside Claude Code (or matched on phrases like "scope this", "break this down for an agent", "write this up as issues").

## What it does

Given a concrete trigger (a file, a feature spec, a bug list, an audit report) and a real user persona, `lead-eng`:

1. Drafts every issue to `/tmp/` first using an 8-section template.
2. Pauses for review before touching GitHub.
3. Creates issues one at a time (no array loops — those have caused off-by-one title/body mismatches).
4. Verifies every created issue title matches its body before moving on.
5. Refreshes the epic body to link to the children.

The full template + prior-art reference live in `references/`.

## Refuses to draft without

- An **explicit target** — a file, feature, audit report. Not "we should improve onboarding."
- **Real file paths + line numbers** for the "What exists today" section. Runs `grep -rn` / `find` first if needed.
- A **concrete persona**. "Dr Patel between patients at 14:30 with a chipped tooth on hold" beats "the user."

If any are missing, the next thing it says is a clarification question — not a draft.

## Install

Clone into your personal Claude skills directory:

```bash
git clone https://github.com/farmhutsoftwareteam/lead-eng ~/.claude/skills/lead-eng
```

Verify Claude Code picks it up:

```bash
claude
> /lead-eng
```

You should see the skill greet you and ask for the three required inputs (target, file paths + line numbers, persona) before doing anything.

### Updating

```bash
cd ~/.claude/skills/lead-eng && git pull
```

## Requirements

- [Claude Code](https://www.anthropic.com/claude-code) 2.0+
- [`gh` CLI](https://cli.github.com/) authenticated to the repo you want to file issues in
- A repo with labels configured (the skill creates `area:<thing>` labels on demand, but priority and type labels you'll want to set up once: `type:epic`, `type:task`, `priority:urgent|high|medium|low`)

## Why this exists

Most "scope this for me" prompts produce vague, untriaged issue lists that the next agent has to re-research from scratch. `lead-eng` enforces the discipline that makes issues actually pick-up-able: file paths, line numbers, a concrete persona, and explicit out-of-scope guardrails.

The output is opinionated and somewhat strict — that's the point. If you need a looser brainstorm-mode skill, this isn't it.

## License

MIT — fork it, modify it, share variants.

## Author

Built by [@munyamakosa](https://github.com/munyamakosa). Issues + PRs welcome.
