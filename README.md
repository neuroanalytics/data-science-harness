# data-science-harness

A community-driven, harness-agnostic collection of AI assistant configurations for academic data science work — skills, agents, commands, MCP configs, and planning templates that work across Claude Code, Cursor, GitHub Copilot, Windsurf, OpenCode, and Gemini CLI.

---

## What this is

Most AI coding assistant configurations are tightly coupled to one harness. This project generalizes the best patterns from software development tooling (skills, subagents, slash commands, MCP servers, planning specs) for academic research workflows, then distributes them in a format any harness can consume.

**Target harnesses**: Claude Code, Cursor, GitHub Copilot, Windsurf, OpenCode, Gemini CLI

**Target workflows**: data analysis, experiment design, literature review, reproducibility, project management, long-term planning

---

## Architecture

Three layers, designed so each is independently useful:

| Layer | Format | Who needs it |
|-------|--------|-------------|
| **1. Content** (source of truth) | Universal SKILL.md + `plugin.yaml` | Everyone — contributors only write Markdown |
| **2. Installer** (convenience) | Python CLI (`ds-harness`) | Users who want automated multi-harness install |
| **3. Manual fallback** | `bin/install.sh` + per-harness docs | Users in locked-down environments |

The content layer is plain Markdown + YAML. The Python package is just an installer/translator — you can clone the repo and manually copy files to any harness without it.

### Why Python over Node

The target community (academic data scientists) already uses `pip`/`uv`. The format itself has zero Python dependency — Python is only needed to run the CLI installer.

---

## Universal Skill Format

All skills are authored once in a **universal SKILL.md** with a superset YAML frontmatter. The installer translates this into harness-specific output at install time — no duplication per harness.

```yaml
---
name: plan-analysis
description: >
  Guide statistical test selection for research datasets with QC checks.
  Triggers when user asks to "plan analysis", "choose a statistical test",
  or "what test should I use for this dataset".
when:
  always: false
  globs: ["*.R", "*.py", "*.ipynb"]
category: data-analysis
tools: [Read, Grep, Bash]
version: "0.1.0"
harnesses: [all]
---
```

**Harness translation map:**

| Universal field | Claude Code | Cursor (.mdc) | Copilot (.instructions.md) | Windsurf |
|-----------------|-------------|---------------|---------------------------|----------|
| `name` | `name:` | frontmatter `description:` | frontmatter | filename |
| `description` | `description:` | `description:` | filename | filename |
| `when.globs` | (auto-load in project) | `globs:` | `applyTo:` | `triggers:` |
| `when.always` | (user-invocable) | `alwaysApply:` | always-loaded | global rules |
| `tools` | `allowed-tools:` | (ignored) | (ignored) | (ignored) |

---

## Repository Structure

```
data-science-harness/
├── pyproject.toml                    # Python package: ds-harness CLI
├── README.md
├── CLAUDE.md                         # Claude Code-specific contributor guidance
├── harness.yaml                      # Root collection manifest
│
├── plugins/                          # Thematic plugin bundles (the content)
│   ├── research-workflow/            # Academic research lifecycle
│   │   ├── plugin.yaml
│   │   ├── skills/
│   │   │   ├── literature-search/SKILL.md
│   │   │   ├── experiment-design/SKILL.md
│   │   │   └── reproducibility/SKILL.md
│   │   └── agents/
│   │       └── data-shepherd/SKILL.md
│   │
│   ├── data-analysis/                # Statistical analysis workflows
│   │   ├── plugin.yaml
│   │   ├── skills/
│   │   │   ├── merge-data/SKILL.md
│   │   │   ├── plan-analysis/SKILL.md
│   │   │   ├── gen-report/SKILL.md
│   │   │   └── gen-data-dict/SKILL.md
│   │   ├── agents/merge-agent/SKILL.md
│   │   └── references/               # Shared: python-patterns, r-patterns, qc-metrics
│   │
│   ├── project-management/           # Research project lifecycle
│   │   ├── plugin.yaml
│   │   └── skills/
│   │       ├── new-project/SKILL.md
│   │       ├── checkpoint/SKILL.md
│   │       └── env-check/SKILL.md
│   │
│   └── planning/                     # Long-term spec and tracking
│       ├── plugin.yaml
│       └── skills/
│           ├── write-spec/SKILL.md
│           └── review-spec/SKILL.md
│
├── src/ds_harness/                   # Python CLI (installer only)
│   ├── cli.py
│   ├── manifest.py                   # Parses plugin.yaml and SKILL.md frontmatter
│   ├── installer.py
│   └── adapters/                     # One per harness
│       ├── base.py                   # Abstract: translate() + install_path()
│       ├── claude_code.py            # → ~/.claude/skills/ + plugin.json
│       ├── cursor.py                 # → .cursor/rules/*.mdc
│       ├── copilot.py                # → .github/instructions/*.instructions.md
│       ├── windsurf.py               # → .windsurf/rules/*.md
│       └── opencode.py               # → TBD
│
├── config/                           # Harness-specific global config templates
│   ├── claude/
│   ├── cursor/
│   └── copilot/
│
└── bin/
    └── install.sh                    # Zero-dependency shell fallback
```

---

## Plugin Manifest (`plugin.yaml`)

Human-readable, no tooling required to understand or contribute:

```yaml
name: data-analysis
description: Language-agnostic statistical analysis workflows for academic research
version: "0.1.0"
author:
  name: bcmcpher
  email: bcmcpher@gmail.com
license: MIT
keywords: [statistics, data-analysis, R, python, julia, jupyter]
skills:
  - ./skills/merge-data
  - ./skills/plan-analysis
  - ./skills/gen-report
  - ./skills/gen-data-dict
agents:
  - ./agents/data-shepherd
harnesses: [all]
```

---

## CLI Usage (`ds-harness`)

```bash
# Install all plugins for a specific harness
ds-harness install --harness=claude-code
ds-harness install --harness=cursor --scope=project

# Install a specific plugin
ds-harness install data-analysis --harness=copilot

# Dry run
ds-harness install --dry-run --harness=windsurf

# List, update, remove
ds-harness list
ds-harness update
ds-harness remove data-analysis --harness=cursor
```

Install the CLI:

```bash
pip install ds-harness
# or
uv tool install ds-harness
```

---

## Root Manifest (`harness.yaml`)

```yaml
name: data-science-harness
description: Community-driven AI assistant configuration for academic data science
version: "0.1.0"
plugins:
  - ./plugins/research-workflow
  - ./plugins/data-analysis
  - ./plugins/project-management
  - ./plugins/planning
harnesses:
  supported: [claude-code, cursor, copilot, windsurf, opencode, gemini-cli]
```

---

## Design Rules

1. **Spec-first**: all content is Markdown + YAML. No Python required to read or contribute.
2. **Installer is optional**: `bin/install.sh` and per-harness docs let users install without the CLI.
3. **Claude Code-native but not Claude-only**: Claude Code plugin format is the reference; adapters translate outward to other harnesses.
4. **One SKILL.md per skill**: no duplication per harness — adapters generate harness-specific output at install time.
5. **Community contribution = write Markdown**: contributors don't touch Python code.
6. **References stay in `references/`**: large domain knowledge lives in `references/` dirs, not in SKILL.md bodies.

---

## Relationship to `my-skills`

This project generalizes the Claude Code-specific plugins in [`my-skills`](../my-skills):

| `my-skills` plugin | `data-science-harness` plugin | Notes |
|--------------------|-------------------------------|-------|
| `stat-analysis` | `plugins/data-analysis` | Add universal frontmatter |
| `project-init` | `plugins/project-management` | Generalize from Claude-specific config |
| `datalad-cli` | `plugins/research-workflow` (partial) | DataLad-specific; port the concept |

---

## Roadmap

**Phase 1** — Format + scaffold: define universal SKILL.md frontmatter, scaffold the directory structure, port `data-analysis` from `my-skills` as the first concrete plugin.

**Phase 2** — Python CLI: build `ds-harness` starting with Claude Code and Cursor adapters.

**Phase 3** — Remaining adapters: Copilot, Windsurf, OpenCode, Gemini CLI as their formats stabilize.

**Phase 4** — Publish to PyPI, add `harness.yaml` marketplace discovery, community contribution guidelines.

---

## Contributing

Contributions are Markdown-first. To add a new skill:

1. Pick the right plugin (or propose a new one in an issue)
2. Copy `templates/skill/SKILL.md`, fill in the universal frontmatter and instruction body
3. Add the path to `plugin.yaml`
4. Open a PR

No Python knowledge required. The adapter layer is maintained by core contributors.
