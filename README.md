# TalentSourcer AI Skill

Agent skill for using TalentSourcer AI from AI coding agents such as Cursor, Codex, OpenCode, and similar tools.

The skill teaches agents to use the TalentSourcer AI CLI safely for recruiting workflows including projects, searches, candidate imports, candidate checks, shortlists, campaigns, analytics, and outreach tasks.

## Install

Install the TalentSourcer AI CLI first:

```bash
npm install -g @talentsourcer/cli
```

Authenticate locally with a TalentSourcer AI personal access token:

```bash
talentsourcer auth token set
talentsourcer auth status --json
```

Then install this skill using your agent's skill manager.

## Usage Notes

- Prefer `--json` for all CLI commands when an agent is acting on your behalf.
- Do not paste personal access tokens into AI chat windows.
- Generated payload files should go under `.talentsourcer-agent/`.
- Agents should not automate LinkedIn sending or browser-side message sending.

## Source

This public repository is automatically synced from the TalentSourcer AI monorepo. Source changes are made upstream.

## License

MIT
