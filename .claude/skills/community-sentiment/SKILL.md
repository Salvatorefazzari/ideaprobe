---
name: community-sentiment
description: "Find real people experiencing a problem and gauge pain intensity. Use when the user asks about community sentiment, pain signals, or says 'scan sentiment' for an idea."
---

<!-- Generated from skills/community-sentiment/SKILL.md by scripts/build-project-local.py. Edit the source, then rebuild. -->

# Community Sentiment Analysis

Find real people experiencing a problem and gauge the intensity and frequency of their pain. This is a standalone skill — use it when you need qualitative pain signals without running the full validation workflow.

## Context Loading

Read these files before proceeding:
1. `.claude/ideaprobe/refs/data-sources.md` — where to search and how to cite
2. `.claude/ideaprobe/refs/output-templates.md` — the Research Report template

## Step 1: Clarify the Problem Space

If the user hasn't provided enough context, ask:
- What problem should I scan communities for?
- Any specific communities or platforms I should prioritize?

## Step 2: Launch Research Agent

Launch the **`sentiment-scanner`** agent with:
- The problem description
- Any specific communities to prioritize
- Instruction to cover Reddit and Hacker News at minimum

Use a **standard model** for this agent.

If the agent returns shallow or incomplete results, re-dispatch with a more capable model.

## Step 3: Present Findings

Use the Research Report template from `.claude/ideaprobe/refs/output-templates.md`. Present:

- **Signal strength rating:** Strong (50+ posts, recurring pain) / Moderate (10-50 posts) / Weak (<10 posts) / None (can't find anyone discussing this).
- **Top 5 pain points:** In users' own words — direct quotes with source URLs.
- **Existing solutions mentioned:** What tools people currently use, and what they complain about with each.
- **Notable quotes:** The most compelling quotes that capture frustration — these are valuable for marketing copy and landing pages.
- **Platform breakdown:** How many posts found on Reddit vs HN vs elsewhere.

Every quote and finding must include a source URL.

## Step 4: Suggest Next Steps

After presenting findings:
- If signal is strong: "Strong pain signals. Want to run a full validation? Use `idea-validation` for a complete GO/STOP verdict."
- If signal is moderate: "Moderate signals — the problem exists but may not be urgent. Consider `competitor-analysis` to see if existing solutions are failing these users."
- If signal is weak or none: "Weak community signals suggest few people are actively experiencing this pain publicly. This could mean no market, or that the audience doesn't use Reddit/HN. Consider alternative research channels."


## Project-Local Dispatch (cloud-compatible)

This skill runs from the project-local IdeaProbe bundle, not the plugin. Agents
are project subagents, so dispatch them by `subagent_type` (no `ideaprobe:` prefix):

| Agent | Definition | Suggested `model` |
|-------|-----------|-------------------|
| `sentiment-scanner` | `.claude/agents/sentiment-scanner.md` | `haiku` |

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
