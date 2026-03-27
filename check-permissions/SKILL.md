---
name: check-permissions
description: "Search through this conversation and the conversations with spawned subagents to find all the instances where you asked for permission to execute a command. Review and suggest improvements to allow/deny lists in settings files to reduce friction."
allowed-tools: ["Bash", "Read", "Edit", "Grep", "Glob"]
---

# Check Permissions Skill

Review permission settings and suggest improvements to reduce friction.

## Steps

1. Search through this conversation and the conversations with spawned subagents to find all the instances where you asked for permission to execute a command.
2. Review the allow/deny lists in User and Project settings:
   - `~/.claude/settings.json`
   - `~/.claude/settings.local.json`
   - `~/<project>/settings.json`
   - `~/<project>/settings.local.json`
3. Make suggestions to update the settings so that we can remove friction.
4. Ask the user questions then update the relevant settings to improve the allow/deny lists.
