---
name: check-permissions
description: "Optimize permission settings to reduce approval friction. Use when the user wants fewer prompts, to fix allow/deny lists, or says 'stop asking me'. Analyzes session permission requests and updates settings.json."
allowed-tools: ["Bash", "Read", "Edit", "Grep", "Glob"]
---

# Check Permissions Skill

Review permission settings and suggest improvements to reduce friction.

## Steps

1. Search through this session and catalogue every tool call to identify which ones would have triggered permission prompts
2. Include in the catalogue all tool calls made by spawned subagents that triggered permission prompts
3. Review the allow/deny lists in User and Project settings:
   - `~/.claude/settings.json` (user-level)
   - `~/.claude/settings.local.json` (user-level, local)
   - `<project>/.claude/settings.json` (project-level, shared)
   - `<project>/.claude/settings.local.json` (project-level, local/gitignored)
4. Do a gap analysis and make suggestions to update the settings so that we can remove friction
5. Ask the user questions for guidance
6. Update the relevant settings to improve the allow/deny lists
