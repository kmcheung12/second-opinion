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
- [opencode](https://opencode.ai) — if targeting opencode models
- [Claude Code](https://claude.ai/code) — for the Claude target and for running this plugin

## Install

### Option 1 — Via Claude Code CLI (once published to a marketplace)

```bash
claude plugin marketplace add <your-username>/second-opinion-marketplace
claude plugin install second-opinion
```

> Note: this requires the repo to include a marketplace manifest. Not yet set up — use Option 2 for now.

### Option 2 — Manual local install

1. Clone this repo:
   ```bash
   git clone https://github.com/<your-username>/second-opinion ~/.claude/plugins/local/second-opinion
   ```

2. Add to `~/.claude/plugins/installed_plugins.json` under `"plugins"`:
   ```json
   "second-opinion@local": [{
     "scope": "user",
     "installPath": "/Users/<you>/.claude/plugins/local/second-opinion",
     "version": "1.0.0",
     "installedAt": "2026-01-01T00:00:00.000Z",
     "lastUpdated": "2026-01-01T00:00:00.000Z"
   }]
   ```

3. Enable it:
   ```bash
   claude plugin enable second-opinion@local
   ```

4. Restart Claude Code.

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

- Port 4097 is used for the opencode server. If it's taken, kill the occupying process or edit the port in SKILL.md before installing.
- Two concurrent second-opinion calls against the same session will interleave — invoke one at a time.
- `plugin.json` targets Claude Code's plugin system. The underlying pattern (persistent server + session ID tracking) can be adapted to any harness that supports shell tool calls.
