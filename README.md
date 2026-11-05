# IdeaProbe

Structured, data-backed business idea validation for coding agents.

IdeaProbe is a Claude Code plugin that gives your agent business idea validation superpowers — a 5-dimension scoring framework, parallel research agents, critical review, and composable skills that take you from "I have an idea" to "here's the buildable MVP spec."

## Install

Add the GitHub repository as a plugin marketplace, then install:

```bash
# Add the marketplace source (one-time setup)
claude plugin marketplace add https://github.com/j7an/ideaprobe.git

# Install the plugin
claude plugin install ideaprobe@ideaprobe
```

Or from within a Claude Code session:

```
/plugin marketplace add https://github.com/j7an/ideaprobe.git
/plugin install ideaprobe@ideaprobe
```

### Project-local install (Claude Code on the web / cloud sessions)

Where `/plugin` is unavailable (and plugin hooks never run), install IdeaProbe as
plain project files that Claude Code discovers on its own:

```bash
# Inside a clone of this repo, it's already installed (.claude/ + CLAUDE.md are committed).
# To add IdeaProbe to another project:
python3 scripts/build-project-local.py --target /path/to/your/project
```

This generates:

| Path | Purpose |
|------|---------|
| `.claude/skills/*/SKILL.md` | All 9 skills (`/idea-validation`, `/idea-comparison`, …) |
| `.claude/agents/*.md` | The 4 research/review subagents |
| `.claude/ideaprobe/refs/` | Scoring rubric, output templates, data sources, methodology |
| `.claude/ideaprobe/manifest.json` | Version + list of generated files |
| `CLAUDE.md` (marked block) | Bootstrap router, which replaces the session-start hook |

`skills/`, `agents/` and `refs/` stay the source of truth. After editing them, re-run
`python3 scripts/build-project-local.py` (the test suite fails if the bundle is stale).
Re-running is idempotent and keeps any of your own content in `CLAUDE.md`.

**Cloud caveats:**
- Containers are ephemeral, so commit and push `.ideaprobe/user-profile.md` and any
  saved reports (`.ideaprobe/reports/`) to keep them. Think twice if the repo is public.
- Project subagents load at session start. If `.claude/agents/` was created mid-session,
  skills fall back to `general-purpose` agents that read the agent file.
- Research quality depends on the environment's web access. `WebSearch` may need loading
  via ToolSearch, and some sites (notably reddit.com) may not be directly fetchable.

### Update

```bash
claude plugin update ideaprobe
```

### Uninstall

```bash
claude plugin uninstall ideaprobe
```

## Skills

| Skill | What it does | When to use |
|-------|-------------|-------------|
| `using-ideaprobe` | Bootstrap router | Automatic at session start |
| `founder-profile` | View/update/reset your profile | When your skills, constraints, or goals change |
| `idea-generation` | Generate 3-5 idea candidates | "What should I build in [domain]?" |
| `idea-validation` | Full validation with GO/STOP verdict | "Is this idea worth building?" |
| `market-research` | Demand signal deep dive | "Is there demand for [problem]?" |
| `competitor-analysis` | Competitor mapping + gap analysis | "Who else is doing this?" |
| `community-sentiment` | Reddit/HN pain scanning | "Are people complaining about this?" |
| `idea-comparison` | Side-by-side ranked scoring | "Which of these ideas should I pursue?" |
| `mvp-scoping` | Validated idea → buildable spec | "I got a GO — now what do I build?" |

## How It Works

```
No idea       → idea-generation → idea-comparison → idea-validation → mvp-scoping
Has one idea  → idea-validation → mvp-scoping
Has many ideas → idea-comparison → idea-validation → mvp-scoping
Deep dive     → market-research / competitor-analysis / community-sentiment
```

### Scoring Framework

Ideas are scored across 5 dimensions (0-20 each, 0-100 total):

| Dimension | What it measures |
|-----------|-----------------|
| Demand Signals | Search trends, community complaints, "how do I..." queries |
| Competitor Landscape | Existing solutions, gaps, saturation level |
| Willingness to Pay | Competitor pricing, frustration with free tools |
| Build Feasibility | MVP timeline, API availability, regulatory hurdles |
| Founder Fit | Your domain knowledge, distribution, first-user potential |

### Verdicts

| Score | Verdict | Meaning |
|-------|---------|---------|
| 80-100 | **GO** | Strong signals. Start building. |
| 60-79 | **EXPLORE** | Promising. Validate weak areas deeper. |
| 40-59 | **PIVOT** | Problem exists, approach needs rethinking. |
| 0-39 | **STOP** | Weak demand. Move to next idea. |

## Founder Profile

On first use, IdeaProbe offers to set up a founder profile — your skills, distribution channels, constraints, and goals. This personalizes Founder Fit scoring across all validations.

The profile is stored at `.ideaprobe/user-profile.md` in your project directory (not in the plugin or the generated bundle, so it survives updates). Manage it anytime with the `founder-profile` skill.

## Architecture

IdeaProbe follows the [Superpowers](https://github.com/obra/superpowers) plugin architecture:

- **Skills** — markdown files that teach the agent structured workflows
- **Agents** — focused subagents launched by skills for parallel research
- **Refs** — shared context (scoring rubric, output templates, data sources)
- **Hook** — session-start bootstrap that makes the agent aware of IdeaProbe

No API keys required. No external services. Works with any model available in Claude Code.

## License

MIT
