# hyperbutler skills

Agent skills for [hyperbutler](https://github.com/Higangssh/hyperbutler), the Docker engine
for Windows 11 that runs in its own Hyper-V VM without WSL. With the skill installed, a
coding agent knows hyperbutler's commands, how its ports, bind mounts and host access
differ from Docker Desktop, how to point tools that ignore docker contexts at it, and how
to troubleshoot it, and it asks before anything destructive.

The skill is a plain [Agent Skills](https://agentskills.io) folder
(`skills/hyperbutler/SKILL.md`), so the same files work in Claude Code, Codex and other
agents that read `SKILL.md`.

## Install

### Claude Code

```
/plugin marketplace add Higangssh/hyperbutler-skills
/plugin install hyperbutler@hyperbutler-skills
```

The plugin also registers hyperbutler's MCP server, read-only (`hyperbutler.exe mcp`), so
`hyperbutler.exe` must be on your PATH. To allow starting, stopping and running
containers, add the server yourself with `--allow-write` instead
(see [references/mcp.md](skills/hyperbutler/references/mcp.md)).

### Codex

Copy the skill folder into your personal skills folder:

```powershell
git clone https://github.com/Higangssh/hyperbutler-skills $env:TEMP\hyperbutler-skills
New-Item -ItemType Directory -Force "$HOME\.agents\skills" | Out-Null
Copy-Item -Recurse "$env:TEMP\hyperbutler-skills\skills\hyperbutler" "$HOME\.agents\skills\"
```

Or, for one repository only, into its `.agents/skills/` folder. Restart Codex if the skill
does not show up. The MCP server is configured separately in `~/.codex/config.toml`
(see [references/mcp.md](skills/hyperbutler/references/mcp.md)).

### Other agents

Copy `skills/hyperbutler` to wherever your agent reads skills from.

## What is in it

| File | What it covers |
|---|---|
| [`skills/hyperbutler/SKILL.md`](skills/hyperbutler/SKILL.md) | state checks, everyday commands, settings, the differences from Docker Desktop, what to ask the user before doing |
| [`references/troubleshooting.md`](skills/hyperbutler/references/troubleshooting.md) | the common errors and their fixes |
| [`references/mcp.md`](skills/hyperbutler/references/mcp.md) | the MCP server: client configuration and what each flag allows |

## License

Apache-2.0, as hyperbutler itself.
