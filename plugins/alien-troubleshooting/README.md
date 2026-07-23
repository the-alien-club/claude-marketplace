# alien-troubleshooting
A symptom → cause → fix guide for the most common failures an external builder
hits on the Alien platform, and how to diagnose each using only the `alien_*`
MCP tools. One diagnostic skill plus a reference catalog.

## Contents
- **alien-troubleshooting** (skill) — the decision guide: match a symptom, find
  the likely cause, apply the fix.
- **reference/failure-catalog.md** — the full table: every known failure mode,
  its signature, the MCP tool that confirms it, and the remedy.

## Requirements
An MCP client connected to the Alien MCP server. Most diagnosis needs only read
abilities; some fixes (rebuilding an agent, re-ingesting) need write.
