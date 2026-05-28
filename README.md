# data-science-harness

A community-driven, harness-agnostic collection of AI assistant configurations for academic data science work — skills, agents, commands, MCP configs, and planning templates that work across Claude Code, Cursor, GitHub Copilot, Windsurf, OpenCode, and Gemini CLI.

---

## What this is

Most AI coding assistant configurations are designed for software products: ship a package, cut a release, deploy a service. Academic data science has a different end goal — **publish a research product**: a versioned, citable dataset or reproducible analysis record.

This project generalizes the best patterns from software development tooling for academic research workflows, with three priorities:

1. **Provenance by default** — every analysis step goes through DataLad so the full chain from raw data to published result is recorded automatically
2. **External standards as first-class citizens** — BIDS, Neurobagel, SNOMED, OSF, and Zenodo are integrated into the normal workflow, not bolted on at the end
3. **Research products as the default export** — the output is a versioned, DOI-tagged dataset pushed to OSF or Zenodo, not a pip package

**Target harnesses**: Claude Code, Cursor, GitHub Copilot, Windsurf, OpenCode, Gemini CLI

**Target workflows**: data analysis, experiment design, literature review, reproducibility, project management, long-term planning

---

## Research Lifecycle Model

The seven stages below define the lifecycle that plugins collectively support. DataLad is the connective tissue — every computation goes through `datalad run` or `datalad container-run` so the provenance chain is never broken.

| Stage | What happens | Primary plugins |
|-------|-------------|-----------------|
| **1. Initialize** | YODA dataset + BIDS layout scaffolded at project creation | `project-management` |
| **2. Curate** | Raw → BIDS conversion; annotate variables with Neurobagel / SNOMED | `data-standards`, `annotation` |
| **3. Analyze** | Run computations via `datalad run` / `datalad container-run` | `provenance`, `data-analysis` |
| **4. Checkpoint** | `datalad save` with structured commit; auto-hook on session end | `provenance` |
| **5. QC / Review** | BIDS validator; data quality checks; reproducibility audit | `data-standards`, `research-workflow` |
| **6. Export** | Bundle outputs; push dataset version to OSF / Zenodo | `research-export` |
| **7. Publish** | Update `dataset_description.json`; mint DOI; push Neurobagel graph | `research-export`, `annotation` |

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

## Plugins

Seven plugins cover the full research lifecycle. Four are ported from a mature Claude Code reference implementation; three are new.

| Plugin | Lifecycle stages | Source |
|--------|-----------------|--------|
| `project-management` | 1 — Initialize | Ported from `project-init` (data-analysis type) |
| `data-standards` | 2, 5, 7 — Curate, QC, Publish | Ported from `bids` + `nipoppy-cli` (BIDS skills) |
| `annotation` | 2, 7 — Curate, Publish | **New** — Neurobagel + SNOMED CT |
| `provenance` | 3, 4 — Analyze, Checkpoint | Ported from `datalad-cli` (core subset) |
| `data-analysis` | 3, 5 — Analyze, QC | Ported from `stat-analysis` |
| `research-workflow` | 1–3 — lit search, experiment design, reproducibility | **New** |
| `research-export` | 6, 7 — Export, Publish | **New** — OSF, Zenodo, dataset release |

### Plugin details

**`project-management`** — Scaffold a new research project: YODA-structured DataLad dataset, BIDS layout, environment setup, CLAUDE.md. Skills: `new-project`, `env-check`, `claude-config`.

**`data-standards`** — BIDS validation and naming throughout the lifecycle. Skills: `bids-validate`, `bids-scaffold`, `nipoppy-bidsify`. References: entity ordering, datatype conventions, sidecar field matrix.

**`annotation`** — Normalize phenotypic and clinical variable names against controlled vocabularies. Skills: `neurobagel-annotate` (bagel-cli → `.jsonld` annotation files), `snomed-lookup` (SNOMED CT term suggestion + code lookup). External deps: `bagel-cli`, SNOMED CT API.

**`provenance`** — DataLad as the default run path for all analysis. Skills: `datalad-run`, `datalad-container-run`, `datalad-save`, `checkpoint`. Includes auto-checkpoint hook that commits unsaved changes at end of each session. These skills auto-trigger on analysis commands (`python`, `Rscript`, `apptainer exec`, `bash run_*.sh`).

**`data-analysis`** — Statistical analysis workflow: merge tabular data, generate data dictionaries, plan analyses, scaffold reports. Skills: `merge-data`, `gen-data-dict`, `plan-analysis`, `gen-report`. Agent: `merge-agent`. References: R/Python/Julia patterns, statistical decision tree, QC metrics.

**`research-workflow`** — Academic process scaffolding. Skills: `literature-search` (PubMed/Semantic Scholar, BibTeX generation), `experiment-design` (power analysis, pre-registration), `reproducibility` (audit analysis against DataLad log).

**`research-export`** — Push finished research products. Skills: `osf-push` (osfclient → OSF node, DataLad sibling registration), `dataset-release` (version bump, BIDS CHANGES, git tag, optional Zenodo DOI), `export-results` (bundle `outputs/` with provenance summary). External deps: `osfclient`, `zenodraft`.

---

## Repository Structure

```
data-science-harness/
├── pyproject.toml                    # Python package: ds-harness CLI
├── README.md
├── CLAUDE.md                         # Claude Code-specific contributor guidance
├── harness.yaml                      # Root collection manifest
│
├── plugins/
│   ├── project-management/           # Stage 1: Initialize
│   │   ├── plugin.yaml
│   │   └── skills/
│   │       ├── new-project/SKILL.md
│   │       ├── env-check/SKILL.md
│   │       └── claude-config/SKILL.md
│   │
│   ├── data-standards/               # Stages 2, 5, 7: BIDS compliance
│   │   ├── plugin.yaml
│   │   ├── skills/
│   │   │   ├── bids-validate/SKILL.md
│   │   │   ├── bids-scaffold/SKILL.md
│   │   │   └── nipoppy-bidsify/SKILL.md
│   │   └── references/               # entities, datatypes, sidecars
│   │
│   ├── annotation/                   # Stages 2, 7: Variable normalization (NEW)
│   │   ├── plugin.yaml
│   │   ├── skills/
│   │   │   ├── neurobagel-annotate/SKILL.md
│   │   │   └── snomed-lookup/SKILL.md
│   │   └── references/               # neurobagel-schema, snomed-hierarchy
│   │
│   ├── provenance/                   # Stages 3, 4: DataLad provenance
│   │   ├── plugin.yaml
│   │   ├── skills/
│   │   │   ├── datalad-run/SKILL.md
│   │   │   ├── datalad-container-run/SKILL.md
│   │   │   ├── datalad-save/SKILL.md
│   │   │   └── checkpoint/SKILL.md
│   │   ├── hooks/
│   │   │   └── scripts/datalad-checkpoint.sh
│   │   └── references/               # yoda-layout, annex-content-states
│   │
│   ├── data-analysis/                # Stages 3, 5: Statistical analysis
│   │   ├── plugin.yaml
│   │   ├── skills/
│   │   │   ├── merge-data/SKILL.md
│   │   │   ├── gen-data-dict/SKILL.md
│   │   │   ├── plan-analysis/SKILL.md
│   │   │   └── gen-report/SKILL.md
│   │   ├── agents/merge-agent/SKILL.md
│   │   └── references/               # r-patterns, python-patterns, qc-metrics
│   │
│   ├── research-workflow/            # Stages 1–3: Academic process (NEW)
│   │   ├── plugin.yaml
│   │   └── skills/
│   │       ├── literature-search/SKILL.md
│   │       ├── experiment-design/SKILL.md
│   │       └── reproducibility/SKILL.md
│   │
│   └── research-export/              # Stages 6–7: Research product publishing (NEW)
│       ├── plugin.yaml
│       ├── skills/
│       │   ├── osf-push/SKILL.md
│       │   ├── dataset-release/SKILL.md
│       │   └── export-results/SKILL.md
│       └── references/               # osf-workflow, zenodo-workflow, dataset-versioning
│
├── src/ds_harness/                   # Python CLI (installer only)
│   ├── cli.py
│   ├── manifest.py
│   ├── installer.py
│   └── adapters/
│       ├── base.py
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

## External Standards & Tool Integrations

| Standard / Tool | What it does | Plugin | Install requirement |
|-----------------|-------------|--------|---------------------|
| **DataLad** | Provenance backbone — records all analysis commands, inputs, outputs | `provenance` | `pip install datalad` |
| **BIDS** | Brain Imaging Data Structure — canonical neuroimaging dataset format | `data-standards` | `npm install -g bids-validator` |
| **Neurobagel / bagel-cli** | Annotate phenotypic variables with controlled terms; push to graph | `annotation` | `pip install bagel-cli` |
| **SNOMED CT** | Clinical terminology — normalize variable names to standard codes | `annotation` | SNOMED CT API key or local OWL |
| **OSF / osfclient** | Open Science Framework — push dataset versions, register DOI | `research-export` | `pip install osfclient` |
| **Zenodo / zenodraft** | Zenodo deposit — mint DOI, archive dataset release | `research-export` | `pip install zenodraft` |

---

## Plugin Manifest (`plugin.yaml`)

Human-readable, no tooling required to understand or contribute:

```yaml
name: provenance
description: DataLad-based provenance tracking for all analysis steps
version: "0.1.0"
author:
  name: bcmcpher
  email: bcmcpher@gmail.com
license: MIT
keywords: [datalad, provenance, reproducibility, YODA]
skills:
  - ./skills/datalad-run
  - ./skills/datalad-container-run
  - ./skills/datalad-save
  - ./skills/checkpoint
hooks:
  stop: ./hooks/scripts/datalad-checkpoint.sh
harnesses: [all]
```

---

## CLI Usage (`ds-harness`)

```bash
# Install all plugins for a specific harness
ds-harness install --harness=claude-code
ds-harness install --harness=cursor --scope=project

# Install a specific plugin
ds-harness install provenance --harness=copilot

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
  - ./plugins/project-management
  - ./plugins/data-standards
  - ./plugins/annotation
  - ./plugins/provenance
  - ./plugins/data-analysis
  - ./plugins/research-workflow
  - ./plugins/research-export
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
7. **DataLad is the default run path**: the `provenance` plugin's skills auto-trigger on analysis commands so the provenance chain is never accidentally broken.
8. **Research products first**: the default project export is a versioned, citable dataset — not a software package.

---

## Relationship to `my-skills`

This project generalizes the Claude Code-specific plugins in [`my-skills`](../my-skills):

| `my-skills` plugin | `data-science-harness` plugin | Notes |
|--------------------|-------------------------------|-------|
| `stat-analysis` | `plugins/data-analysis` | Add universal frontmatter |
| `project-init` | `plugins/project-management` | Data-analysis project type only |
| `bids` | `plugins/data-standards` | Full port including reference files |
| `datalad-cli` | `plugins/provenance` | Core subset (run, container-run, save, checkpoint) |
| `nipoppy-cli` | `plugins/data-standards` (BIDS skills) | BIDS conversion subset; full nipoppy in `my-skills` |

---

## Roadmap

**Phase 1** — Scaffold + port: `harness.yaml`, `pyproject.toml`, and the four ported plugins (`data-analysis`, `provenance`, `data-standards`, `project-management`) with universal frontmatter.

**Phase 2** — New plugins: `annotation` (Neurobagel, SNOMED), `research-export` (OSF, Zenodo, dataset release), `research-workflow` (lit search, experiment design, reproducibility).

**Phase 3** — Python CLI: `ds-harness` with Claude Code and Cursor adapters first.

**Phase 4** — Remaining adapters (Copilot, Windsurf, OpenCode, Gemini CLI), PyPI publish, community contribution guidelines.

---

## Contributing

Contributions are Markdown-first. To add a new skill:

1. Pick the right plugin (or propose a new one in an issue)
2. Copy `templates/skill/SKILL.md`, fill in the universal frontmatter and instruction body
3. Add the path to `plugin.yaml`
4. Open a PR

No Python knowledge required. The adapter layer is maintained by core contributors.
