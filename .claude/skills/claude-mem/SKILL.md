---
name: claude-mem
description: Install, configure, and use claude-mem (github.com/thedotmack/claude-mem), a persistent memory compression plugin for Claude Code. Use when the user asks to set up cross-session memory, search past session work, or check whether claude-mem is installed.
---

# claude-mem

claude-mem is a Claude Code plugin by Alex Newman (thedotmack, Apache-2.0). It records tool use from each session, compresses it into observations, and injects relevant past context into new sessions. It ships its own skills (`mem-search`, `smart-explore`, `make-plan`, `handoff`, `do`, `babysit`, `standup`, and others) under `plugin/skills/` in its repo.

## Check whether it is installed

- Run `/plugin list` in Claude Code, or look for `claude-mem` under `~/.claude/plugins/`.
- Check for the `mem-search` skill. If it is present, the plugin is loaded.

## Install

Preferred, from inside Claude Code:

```
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
```

Or from a shell:

```
npx claude-mem install
```

Do not use `npm install -g claude-mem`. That installs only the SDK and does not register the hooks or the worker service.

## Privacy and provider choice

The default installer asks the user to sign in to claude-mem's hosted observer (cmem.ai). Signing in provisions a memory key and sends session tool-use content to that service. Before installing on work that touches confidential material, choose one of:

- Pass an explicit `--provider` flag (use a local or BYO-key provider).
- Set `CLAUDE_MEM_ONLINE_OPTIN=false`.
- Run non-interactively, which skips the account step.

Confirm the choice with the user before running the installer.

## Use

- Search past work with the `mem-search` skill. It follows index, then context, then details, so it does not pull full records first.
- Save durable notes only through claude-mem's own flows, not by editing its database.

## Conflicts

Before adding or updating any skill with a name listed above, check for a same-named skill in `.claude/skills/` and in the plugin. Installed skills and this file can both match a request, so keep descriptions distinct.

## Source

- Repo: https://github.com/thedotmack/claude-mem
- Docs: https://docs.claude-mem.ai/installation
