# lead-eng

A Claude Code skill that translates product intuition into engineer-ready GitHub issues via the `gh` CLI. Produces one parent epic + 3–12 child issues, each grounded in real file paths and line numbers, with explicit out-of-scope guardrails so agents picking up the work don't scope-creep.

## What it does

Given a concrete trigger (a file, a feature spec, a bug list, an audit report) and a real user persona, `lead-eng`:

1. Drafts every issue to `/tmp/` first using an 8-section template.
2. Pauses for review before touching GitHub.
3. Creates issues one at a time (no array loops — those have caused off-by-one title/body mismatches).
4. Verifies every created issue title matches its body before moving on.
5. Refreshes the epic body to link to the children.

The full template + prior-art reference live in `skills/lead-eng/references/`.

## Refuses to draft without

- An **explicit target** — a file, feature, audit report. Not "we should improve onboarding."
- **Real file paths + line numbers** for the "What exists today" section. Runs `grep -rn` / `find` first if needed.
- A **concrete persona**. "Dr Patel between patients at 14:30 with a chipped tooth on hold" beats "the user."

If any are missing, the next thing it says is a clarification question — not a draft.

## Install

### Option A — Plugin marketplace (recommended)

Inside Claude Code, run:

```
/plugin marketplace add farmhutsoftwareteam/lead-eng
/plugin install lead-eng@farmhut-skills
```

Then invoke with `/lead-eng:lead-eng`.

**Why this is the recommended path:** Claude Code tracks the version, you get update notifications when new releases ship, and you can uninstall cleanly with `/plugin uninstall lead-eng@farmhut-skills`.

> Tip: to keep the shorter `/lead-eng` form, add a slash-command alias in your `~/.claude/settings.json`:
> ```json
> {
>   "aliases": { "/lead-eng": "/lead-eng:lead-eng" }
> }
> ```

### Option B — Direct git clone

Skip the plugin layer and drop the skill straight into your personal skills directory:

```bash
git clone https://github.com/farmhutsoftwareteam/lead-eng /tmp/lead-eng \
  && cp -R /tmp/lead-eng/skills/lead-eng ~/.claude/skills/lead-eng \
  && rm -rf /tmp/lead-eng
```

Invoke with `/lead-eng` (no namespacing). Update later with the same command — it overwrites cleanly. No version tracking or update notifications with this path.

## Verify install

Open Claude Code and run:

```
/help
```

You should see `lead-eng` listed. Then run the skill — it will refuse to draft until you give it the three required inputs (target, file paths + line numbers, persona) before doing anything.

## Requirements

- [Claude Code](https://www.anthropic.com/claude-code) 2.0+
- [`gh` CLI](https://cli.github.com/) authenticated to the repo you want to file issues in
- A repo with labels configured (the skill creates `area:<thing>` labels on demand, but priority and type labels you'll want to set up once: `type:epic`, `type:task`, `priority:urgent|high|medium|low`)

## Why this exists

Most "scope this for me" prompts produce vague, untriaged issue lists that the next agent has to re-research from scratch. `lead-eng` enforces the discipline that makes issues actually pick-up-able: file paths, line numbers, a concrete persona, and explicit out-of-scope guardrails.

The output is opinionated and somewhat strict — that's the point. If you need a looser brainstorm-mode skill, this isn't it.

## Updating

**Plugin install:** `/plugin update lead-eng@farmhut-skills`

**Direct clone install:** re-run the install snippet above.

## License

MIT — fork it, modify it, ship variants.

## Author

Built by [@munyamakosa](https://github.com/munyamakosa). Issues + PRs welcome.
