---
name: market-research
description: "Deep dive into demand signals for a problem or solution space. Use when the user asks about market demand, search trends, or says 'check demand' for an idea."
---

<!-- Generated from skills/market-research/SKILL.md by scripts/build-project-local.py. Edit the source, then rebuild. -->

# Market Research

Deep dive into demand signals for a specific problem or solution space. This is a standalone skill — use it when you need detailed demand analysis without running the full validation workflow.

## Context Loading

Read these files before proceeding:
1. `.claude/ideaprobe/refs/data-sources.md` — where to search and how to cite
2. `.claude/ideaprobe/refs/output-templates.md` — the Research Report template

## Step 1: Clarify the Problem Space

If the user hasn't provided enough context, ask:
- What problem or solution space should I research?
- Any specific keywords or competitors I should include in the search?

## Step 2: Launch Research Agent

Launch the **`market-researcher`** agent with:
- The problem/solution space description
- Any specific keywords the user provided
- Instruction to cover all sources from data-sources.md

Use a **standard model** for this agent.

If the agent returns shallow or incomplete results, re-dispatch with a more capable model.

## Step 3: Present Findings

Use the Research Report template from `.claude/ideaprobe/refs/output-templates.md`. Present:

- **Search volume trends:** Rising, stable, or declining? Seasonal patterns? Include Google Trends data with trajectory direction.
- **Community signal volume:** How many relevant posts on Reddit and HN? Which subreddits are most active?
- **Lightweight market sizing:** Directional estimate from industry reports, competitor customer counts, job posting volume. Don't over-engineer TAM/SAM/SOM.
- **Trend trajectory:** Is this problem growing (expanding market) or being solved (shrinking)? What forces are driving the trend?
- **Related rising queries:** Adjacent search terms that reveal opportunities the user may not have considered.

Every finding must include a source URL.

## Step 4: Suggest Next Steps

After presenting findings:
- If demand looks strong: "Want to run a full validation? Use the `idea-validation` skill for a complete GO/STOP verdict."
- If demand is unclear: "Consider checking community sentiment directly with `community-sentiment` for more qualitative signals."
- If demand is weak: "Low demand signals suggest this market may be too small. Consider a different angle or problem space."


## Project-Local Dispatch (cloud-compatible)

This skill runs from the project-local IdeaProbe bundle, not the plugin. Agents
are project subagents, so dispatch them by `subagent_type` (no `ideaprobe:` prefix):

| Agent | Definition | Suggested `model` |
|-------|-----------|-------------------|
| `market-researcher` | `.claude/agents/market-researcher.md` | `sonnet` |

Model tiers map to the Agent tool's `model` parameter: fast → `haiku`,
standard → `sonnet`, most capable → `opus` (or omit `model` to use the session's
model). Escalate one tier if results come back thin.

If a `subagent_type` is rejected as unknown (e.g. the session started before
`.claude/agents/` existed), fall back to the `general-purpose` agent and begin
its prompt with: "Read `.claude/agents/<agent>.md` and follow it exactly as your
instructions." followed by the full idea context.

Research agents need web access. If `WebSearch` is listed only as a deferred
tool, load it with ToolSearch first. If no web tool works, report that in
**Data Gaps** and score conservatively — never fill gaps with invented data.
