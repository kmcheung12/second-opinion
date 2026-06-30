# second-opinion

A skill that lets you ask another AI harness or model for a second opinion mid-session, without leaving your current session. Back-and-forth follow-ups are supported — sessions are preserved across exchanges.

**Invoke with phrases like:**
- "ask opencode's kimi for a second opinion on this"
- "get a second opinion from claude opus"
- "ask them to dig deeper into X"
- "follow up with kimi on point 2"

## How it works

The skill compiles relevant context from the current conversation, dispatches it to the target harness, and streams the response back — wrapped in `<second-opinion>` delimiters so it's treated as a peer perspective to weigh, not an authority to defer to.

- **opencode targets**: starts `opencode serve` once (port 4097 by default), uses `opencode run --attach` for dispatch and `opencode run -s <id>` for follow-ups. Sessions persist as long as the server is alive. **Quitting your primary session kills the server and ends the follow-up chain.**
- **Claude targets**: uses `claude -p --output-format json` for dispatch (no streaming) and `claude --resume <id> -p` for follow-ups. Sessions persist on disk independently.
- **Model resolution**: Claude handles aliases (`opus`, `sonnet`) natively. opencode resolves via `opencode models | grep -i <hint>` with disambiguation if multiple matches.

## Requirements

- `jq` (`brew install jq`) — used for JSON parsing throughout
- `curl`
- [Claude Code](https://claude.ai/code) — if using as Claude Code host or targeting Claude
- [opencode](https://opencode.ai) — if using as opencode host or targeting opencode models

## Install

### Claude Code

**Personal (all projects):** Copy the skill into your personal skills directory:

```bash
mkdir -p ~/.claude/skills/second-opinion
cp skills/second-opinion/SKILL.md ~/.claude/skills/second-opinion/SKILL.md
```

**Per-project:** Copy the skill directory into your project's `.claude/skills/`:

```bash
cp -r skills/second-opinion .claude/skills/
```

The skill is available immediately — no restart needed. Invoke it with `/second-opinion` or let Claude load it automatically when relevant.

### opencode

Copy `agents/second-opinion.md` to your global or project agents directory:

```bash
# Global (available in all projects)
cp agents/second-opinion.md ~/.config/opencode/agents/

# Per-project
cp agents/second-opinion.md .opencode/agents/
```

Then invoke with `@second-opinion ask claude opus about X`.

## State files

The skill stores session state in your home directory:

| File | Purpose |
|---|---|
| `~/.second-opinion-session` | Active session ID |
| `~/.second-opinion-harness` | Active harness (`claude` or `opencode`) |
| `~/.second-opinion-server.pid` | opencode server PID |

To reset state: `rm ~/.second-opinion-{session,harness,server.pid}`

To stop the opencode server: `kill $(cat ~/.second-opinion-server.pid)`

## Notes

- Port 4097 is used for the opencode server. If it's taken, kill the occupying process or edit the port in `skills/second-opinion/SKILL.md` and `agents/second-opinion.md`.
- Two concurrent second-opinion calls against the same session will interleave — invoke one at a time.
- `plugin.json` targets Claude Code's plugin system. The underlying pattern (persistent server + session ID tracking) can be adapted to any harness that supports shell tool calls.
