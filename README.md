# data-science-harness

A community-driven, harness-agnostic collection of AI assistant configurations for academic data science work — skills, agents, commands, hooks, MCP configs, and planning templates that work across Claude Code, Cursor, GitHub Copilot, Windsurf, OpenCode, and Gemini CLI.

---

## What this is

Most AI coding assistant configurations are designed for software products: ship a package, cut a release, deploy a service. Academic data science has a different end goal — **publish a research product**. But a research product is no longer just a static PDF or a frozen dataset. This project treats it as a **living research compendium**:

1. a **provenanced dataset** (versioned, citable, DOI-tagged), plus
2. a **re-executable article** that regenerates its own figures and results (NeuroLibre-style), plus
3. an **agent-callable method bundle** that exposes the work's methods as tools a future researcher's AI assistant can invoke on new data (Paper2Agent-style),

all built from a **single DataLad provenance chain** and cross-linked by DOI. The goal is research that the next person — or the next agent — can *build upon* rapidly, not just read.

This project generalizes the best patterns from software development tooling for academic research workflows, with four priorities:

1. **Provenance by default** — every analysis step *and every administrative change* goes through DataLad so the full chain from raw data to published result is recorded automatically
2. **External standards as first-class citizens** — BIDS, Neurobagel, SNOMED, OSF, Zenodo, NeuroLibre, ORCID, CRediT, and reporting guidelines are integrated into the normal workflow, not bolted on at the end
3. **Research products are living** — the default export re-executes (NeuroLibre) and is agent-callable (Paper2Agent / MCP), not a one-off artifact
4. **Administration is first-class, not an afterthought** — funding, ethics, data-management plans, deadlines, people, and credit are tracked alongside the science, with the same provenance discipline

**Target harnesses**: Claude Code, Cursor, GitHub Copilot, Windsurf, OpenCode, Gemini CLI

**Target workflows**: data analysis, experiment design, literature review, reproducibility, **project governance & compliance**, **administrative tracking & reporting**, **dissemination & living publications**, long-term planning

---

## Research Lifecycle Model

The lifecycle is **a linear scientific pipeline (stages 0–8) running inside a persistent administrative track**. DataLad is the connective tissue — every computation goes through `datalad run` / `datalad container-run`, and every administrative change is `datalad save`-d, so neither the analysis chain nor the administrative record is ever broken.

| Stage | What happens | Primary plugins |
|-------|-------------|-----------------|
| **0. Propose & Govern** | Funding metadata, Data Management Plan, IRB/ethics, pre-registration, project-ledger init | `project-governance` |
| **1. Initialize** | YODA dataset + BIDS layout scaffolded at project creation | `project-management` |
| **2. Curate** | Raw → BIDS conversion; annotate variables with Neurobagel / SNOMED | `data-standards`, `annotation` |
| **3. Analyze** | Run computations via `datalad run` / `datalad container-run` | `provenance`, `data-analysis` |
| **4. Checkpoint** | `datalad save` with structured commit; auto-hook on session end | `provenance` |
| **5. QC / Review** | BIDS validator; data quality checks; reproducibility audit | `data-standards`, `research-workflow` |
| **6. Export** | Bundle outputs; push dataset version to OSF / Zenodo | `research-export` |
| **7. Publish** | Update `dataset_description.json`; mint DOI; push Neurobagel graph | `research-export`, `annotation` |
| **8. Disseminate & Report** | Manuscript **+ living compendium** (executable article + agent bundle); reporting-guideline compliance; DOI cross-linking; progress/final reports | `dissemination`, `project-management` |

```
   ┌─────────────────────────────────────────────────────────────────────────┐
   │  Manage & Comply lane  (cross-cutting, runs across ALL stages)            │
   │  project ledger · obligations & deadlines · decision log · people/credit  │
   │  · status & funder reports · compliance audits                            │
   └─────────────────────────────────────────────────────────────────────────┘
        ▲          ▲          ▲          ▲          ▲          ▲          ▲
   ┌────┴───┐ ┌────┴───┐ ┌────┴───┐ ┌────┴───┐ ┌────┴───┐ ┌────┴───┐ ┌────┴───┐
   │ 0 Gov  │→│ 1 Init │→│2 Curate│→│3 Analyze│→│ 4-5 QC │→│6-7 Pub │→│8 Disse-│
   │        │ │        │ │        │ │ +4 Chk │ │        │ │ +DOI   │ │ minate │
   └────────┘ └────────┘ └────────┘ └────────┘ └────────┘ └────────┘ └────────┘
```

The **Manage & Comply lane** is the key conceptual addition: administration is not a single stage, it is a continuous track the whole pipeline runs inside. It is served by the tracking skills in `project-management` and the compliance skills in `project-governance`, and it is backed by a single versioned [Project Ledger](#the-project-ledger-projectyaml).

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

Nine plugins cover the full research lifecycle and its administrative track, four of them ported from a mature Claude Code reference implementation.

| Plugin | Lifecycle stages | Notes |
|--------|-----------------|-------|
| `project-governance` | 0 + Manage & Comply lane | DMP, ethics, pre-registration, compliance |
| `project-management` | 1, 8 + Manage & Comply lane | Ported from `project-init`; adds tracking skills |
| `data-standards` | 2, 5, 7 — Curate, QC, Publish | Ported from `bids` + `nipoppy-cli` (BIDS skills) |
| `annotation` | 2, 7 — Curate, Publish | Neurobagel, SNOMED CT, NIDM, ReproSchema |
| `provenance` | 3, 4 — Analyze, Checkpoint | Ported from `datalad-cli` (core subset) |
| `data-analysis` | 3, 5 — Analyze, QC | Ported from `stat-analysis` |
| `research-workflow` | 1–3 — lit search, experiment design, reproducibility | — |
| `research-export` | 6, 7 — Export, Publish | OSF, Zenodo, dataset release |
| `dissemination` | 8 — Disseminate & Report | manuscript, reporting guidelines, living artifacts |

### Plugin details

**`project-governance`** — Stand up and maintain the administrative + compliance backbone (lifecycle stage 0 and the Manage & Comply lane). Skills:
- `init-ledger` — scaffold `project.yaml` (called by / extends `project-management/new-project`)
- `dmp` — author/update a Data Management Plan against the RDA DMP Common Standard (machine-actionable DMP) or a funder template; record obligations into the ledger
- `ethics-track` — record IRB/IACUC protocol, approval, expiry, and amendments; flag upcoming renewals
- `preregister` — register the study (OSF Registrations / ClinicalTrials.gov / PROSPERO); record the registration ID + status into the ledger (reuses the pre-registration template from `research-workflow/experiment-design`)
- `compliance-audit` — check ledger obligations, de-identification, and DUA data-scope against the data actually present (reuses the audit pattern from `research-workflow/reproducibility`)
- References (general core + neuro pack): `references/madmp-schema.md`, `references/hipaa-deid.md`, `references/clinicaltrials-fields.md`

**`project-management`** — Scaffold a new research project **and** run the ongoing Manage & Comply lane. Scaffolding skills: `new-project` (YODA-structured DataLad dataset, BIDS layout, environment setup, CLAUDE.md, project ledger), `env-check`, `claude-config`. Tracking skills:
- `log-decision` — append to a decision / lab-notebook log, then `datalad save`
- `track-milestone` — add/update milestones & deadlines in the ledger
- `status-report` — generate a progress / funder-RPPR-style summary from the ledger + `datalad log` + git history
- `obligations` — read the ledger and list what's due (the **harness-agnostic** reminder core)
- `people` — manage collaborators / ORCID / CRediT contributor roles in the ledger
- Hook: `obligations-due.sh` (Claude Code `SessionStart`) surfaces obligations due within N days; degrades to the on-demand `obligations` skill on harnesses without hooks. Mirrors the `provenance` checkpoint-hook mechanism.

**`data-standards`** — BIDS validation and naming throughout the lifecycle. Skills: `bids-validate`, `bids-scaffold`, `nipoppy-bidsify`. References: entity ordering, datatype conventions, sidecar field matrix.

**`annotation`** — Standardize and normalize phenotypic, clinical, and behavioral variables against controlled vocabularies and schemas. Skills: `neurobagel-annotate` (bagel-cli → `.jsonld` annotation files), `snomed-lookup` (SNOMED CT term suggestion + code lookup), `nidm-annotate` (Neuroimaging Data Model — PROV/RDF descriptions of experiments and results via NIDM-Experiment / NIDM-Results), `reproschema-annotate` (ReproSchema — standardize the tracking of behavioral assessment and questionnaire fields). External deps: `bagel-cli`, SNOMED CT API, `pynidm`, `reproschema`.

**`provenance`** — DataLad as the default run path for all analysis. Skills: `datalad-run`, `datalad-container-run`, `datalad-save`, `checkpoint`. Includes auto-checkpoint hook that commits unsaved changes at end of each session. These skills auto-trigger on analysis commands (`python`, `Rscript`, `apptainer exec`, `bash run_*.sh`).

**`data-analysis`** — Statistical analysis workflow: merge tabular data, generate data dictionaries, plan analyses, scaffold reports. Skills: `merge-data`, `gen-data-dict`, `plan-analysis`, `gen-report`. Agent: `merge-agent`. References: R/Python/Julia patterns, statistical decision tree, QC metrics.

**`research-workflow`** — Academic process scaffolding. Skills: `literature-search` (PubMed/Semantic Scholar, BibTeX generation), `experiment-design` (power analysis, pre-registration template), `reproducibility` (audit analysis against DataLad log).

**`research-export`** — Push finished research products. Skills: `osf-push` (osfclient → OSF node, DataLad sibling registration), `dataset-release` (version bump, BIDS CHANGES, git tag, optional Zenodo DOI), `export-results` (bundle `outputs/` with provenance summary). External deps: `osfclient`, `zenodraft`.

**`dissemination`** — Turn the finished, provenanced work into publications — both classic outputs and the living compendium (lifecycle stage 8).

*Classic publication outputs:*
- `draft-manuscript` — scaffold an IMRaD manuscript; auto-fill Methods / Data-availability / provenance sections from `datalad log` + the ledger (reuses the provenance summary from `research-export/export-results`)
- `reporting-checklist` — apply the right EQUATOR guideline (CONSORT / STROBE / PRISMA / ARRIVE) or, with the neuro pack, COBIDAS, as a fill-in checklist
- `submission-track` — track target journal, submission date, revisions, and reviewer responses in the ledger

*Living research compendium:*
- `executable-article` — scaffold a **NeuroLibre-style reproducible preprint**: a MyST `myst.yml` + Jupyter Book content, a `binder/` environment derived from the existing DataLad container digest / lockfile, and a `repo2data` data-requirement file pointing at the OSF/DataLad-published dataset; wire figures to regenerate from the provenanced `datalad run` pipeline; run NeuroLibre's repo-structure pre-submission check. Reuses `provenance` (container digest) + `research-export` (published dataset).
- `agent-bundle` — **Paper2Agent-style**: synthesize an MCP server + parameterized tools from the project's analysis scripts/functions and the data dictionary, emitted as the harness's *own* universal `SKILL.md` + `plugin.yaml` + MCP config so a downstream harness can install and call this work's methods on new data; include tests that reproduce the paper's key results (reuses the `research-workflow/reproducibility` audit). This is the "build upon in future work" artifact, and it dogfoods the project's own content format.

*Shared:*
- `link-outputs` — cross-link dataset / code / paper / preprint / pre-registration / executable-article / agent-bundle DOIs using DataCite `RelatedIdentifier` relation types; write back to the ledger `products:` and `dataset_description.json`.
- References: `references/equator-guidelines.md`, `references/datacite-relations.md`, `references/cobidas.md` (neuro pack), `references/neurolibre-structure.md` (MyST / Jupyter Book / BinderHub / repo2data submission layout), `references/paper2agent-bundle.md` (tool-synthesis + MCP-manifest pattern).

---

## The Project Ledger (`project.yaml`)

The administrative source of truth is a single machine-actionable file at the dataset root, sibling to `dataset_description.json`, and **`datalad save`-d like any other artifact** — so administrative metadata gets *provenance by default* too: every IRB amendment, DMP revision, milestone change, or new DOI is a tracked commit. Every administrative skill reads/writes it; it auto-fills reports, drives the obligations/reminder surface, and powers compliance audits. A human-readable `PROJECT.md` is generated *from* it on demand and never hand-edited.

```yaml
# project.yaml — administrative ledger (validated against schemas/project.schema.json)
study:
  title: "Effect of X on Y in cohort Z"
  short_name: xyz-study
  affiliation_ror: https://ror.org/00xxxx
  start: 2026-01-15
  end: 2028-01-14

funding:
  - funder_id: https://doi.org/10.13039/100000002   # Crossref Funder Registry (NIH)
    award_number: R01-XX000000
    period: { start: 2026-01-15, end: 2028-01-14 }
    reporting:
      - { type: RPPR, due: 2026-12-01, status: pending }

ethics:
  - body: IRB
    protocol_id: "2025-12345"
    approved: 2025-11-01
    expires: 2026-11-01          # → drives a renewal obligation
    status: approved
    amendments: []

agreements:                       # DUAs / MTAs
  - { type: DUA, party: "Site B", signed: 2026-01-10, expires: 2028-01-10, data_scope: "de-identified imaging" }

dmp:
  standard: RDA-maDMP
  location: docs/dmp.md
  version: "1.2"
  last_reviewed: 2026-03-01
  obligations:
    - { req: "Deposit data within 6 months of collection", due: 2026-09-01, status: pending }

registration:
  - { platform: OSF, id: ab12c, url: https://osf.io/ab12c, type: prereg }

people:
  - { name: "B. McPherson", orcid: 0000-0000-0000-0000, roles: [Conceptualization, Software, "Writing – original draft"], affiliation_ror: https://ror.org/00xxxx }

milestones:
  - { name: "Data collection complete", due: 2026-06-30, status: in_progress, deliverable: "raw BIDS dataset" }

products:                         # cross-linked, DOI-bearing outputs
  - { type: dataset,            doi: 10.xxxx/dataset, status: published, relation: IsSourceOf }
  - { type: paper,              doi: 10.xxxx/paper,   status: submitted, relation: IsDocumentedBy }
  - { type: executable-article, url: https://neurolibre.org/..., status: planned, relation: IsSupplementTo }
  - { type: agent-bundle,       doi: 10.xxxx/agent,   status: planned, relation: IsDerivedFrom }

obligations:                      # explicit + derived (from ethics/dmp/funder/milestones)
  - { what: "Renew IRB protocol 2025-12345", due: 2026-11-01, source: ethics, status: pending }
  - { what: "Submit RPPR", due: 2026-12-01, source: funder, status: pending }
```

The ledger ships with a JSON Schema (`schemas/project.schema.json`) so `ds-harness` and editors can validate it. All sections are optional and additive — a project that only needs milestones and people can ignore the rest.

---

## Living Research Products

Stage 8 produces a **living research compendium**: three coupled artifacts, all generated from the *same* DataLad provenance chain and cross-linked by DOI in the ledger.

| Artifact | What it is | How it's built | External tooling |
|----------|-----------|----------------|------------------|
| **Provenanced dataset** | Versioned, citable data + analysis record | DataLad + BIDS, pushed via `research-export` | OSF / Zenodo |
| **Executable article** | A reproducible preprint that re-runs its own figures/results | `dissemination/executable-article` — MyST/Jupyter Book content + `binder/` env (from the DataLad container digest) + `repo2data` config pointing at the published dataset | NeuroLibre (MyST, Jupyter Book, BinderHub, repo2data; GitHub editorial workflow) |
| **Agent bundle** | An MCP server exposing the work's methods as callable, tested tools | `dissemination/agent-bundle` — tools synthesized from the project's scripts + data dictionary, emitted as the harness's own `SKILL.md` + `plugin.yaml` + MCP config, with result-reproduction tests | Paper2Agent pattern + Model Context Protocol |

Why this fits the architecture cleanly:

- **NeuroLibre needs exactly what the harness already produces** — a public code repo (notebooks / MyST), a data config, and a pinned, BinderHub-recognized environment. The `provenance` and `data-standards` plugins already produce all three; `executable-article` just arranges them into NeuroLibre's expected layout.
- **Paper2Agent's output *is* the harness's own format** — an MCP server + a manifest of tools. Because this project already authors universal `SKILL.md` + MCP configs and ships adapters for them, `agent-bundle` emits its output in that same format and reuses the existing adapter layer. The research product becomes installable into the next researcher's harness with zero new tooling.
- **Cross-linking is provenance, not metadata gardening** — `link-outputs` records the relations (dataset `IsSourceOf` article; article `IsSupplementTo` paper; agent bundle `IsDerivedFrom` code) using the DataCite schema, written back into the ledger and `dataset_description.json`.

---

## Repository Structure

```
data-science-harness/
├── pyproject.toml                    # Python package: ds-harness CLI
├── README.md
├── CLAUDE.md                         # Claude Code-specific contributor guidance
├── harness.yaml                      # Root collection manifest
│
├── schemas/
│   └── project.schema.json           # JSON Schema for the project ledger
│
├── examples/
│   └── project.yaml                  # Worked ledger sample (used by skills/hooks/tests)
│
├── templates/
│   ├── skill/SKILL.md                # Universal skill template
│   ├── executable-article/           # MyST myst.yml + binder/ + repo2data skeleton
│   └── agent-bundle/                 # plugin.yaml + SKILL.md + MCP-config skeleton
│
├── plugins/
│   ├── project-governance/           # Stage 0 + compliance lane
│   │   ├── plugin.yaml
│   │   ├── skills/
│   │   │   ├── init-ledger/SKILL.md
│   │   │   ├── dmp/SKILL.md
│   │   │   ├── ethics-track/SKILL.md
│   │   │   ├── preregister/SKILL.md
│   │   │   └── compliance-audit/SKILL.md
│   │   └── references/               # madmp-schema, hipaa-deid, clinicaltrials-fields
│   │
│   ├── project-management/           # Stage 1, 8 + Manage & Comply lane
│   │   ├── plugin.yaml
│   │   ├── skills/
│   │   │   ├── new-project/SKILL.md
│   │   │   ├── env-check/SKILL.md
│   │   │   ├── claude-config/SKILL.md
│   │   │   ├── log-decision/SKILL.md
│   │   │   ├── track-milestone/SKILL.md
│   │   │   ├── status-report/SKILL.md
│   │   │   ├── obligations/SKILL.md
│   │   │   └── people/SKILL.md
│   │   └── hooks/
│   │       └── scripts/obligations-due.sh
│   │
│   ├── data-standards/               # Stages 2, 5, 7: BIDS compliance
│   │   ├── plugin.yaml
│   │   ├── skills/{bids-validate,bids-scaffold,nipoppy-bidsify}/SKILL.md
│   │   └── references/               # entities, datatypes, sidecars
│   │
│   ├── annotation/                   # Stages 2, 7: Variable & assessment standardization
│   │   ├── plugin.yaml
│   │   ├── skills/{neurobagel-annotate,snomed-lookup,nidm-annotate,reproschema-annotate}/SKILL.md
│   │   └── references/               # neurobagel-schema, snomed-hierarchy, nidm-schema, reproschema
│   │
│   ├── provenance/                   # Stages 3, 4: DataLad provenance
│   │   ├── plugin.yaml
│   │   ├── skills/{datalad-run,datalad-container-run,datalad-save,checkpoint}/SKILL.md
│   │   ├── hooks/scripts/datalad-checkpoint.sh
│   │   └── references/               # yoda-layout, annex-content-states
│   │
│   ├── data-analysis/                # Stages 3, 5: Statistical analysis
│   │   ├── plugin.yaml
│   │   ├── skills/{merge-data,gen-data-dict,plan-analysis,gen-report}/SKILL.md
│   │   ├── agents/merge-agent/SKILL.md
│   │   └── references/               # r-patterns, python-patterns, qc-metrics
│   │
│   ├── research-workflow/            # Stages 1–3: Academic process
│   │   ├── plugin.yaml
│   │   └── skills/{literature-search,experiment-design,reproducibility}/SKILL.md
│   │
│   ├── research-export/              # Stages 6–7: Research product publishing
│   │   ├── plugin.yaml
│   │   ├── skills/{osf-push,dataset-release,export-results}/SKILL.md
│   │   └── references/               # osf-workflow, zenodo-workflow, dataset-versioning
│   │
│   └── dissemination/                # Stage 8: Publications + living artifacts
│       ├── plugin.yaml
│       ├── skills/
│       │   ├── draft-manuscript/SKILL.md
│       │   ├── reporting-checklist/SKILL.md
│       │   ├── submission-track/SKILL.md
│       │   ├── executable-article/SKILL.md
│       │   ├── agent-bundle/SKILL.md
│       │   └── link-outputs/SKILL.md
│       └── references/               # equator-guidelines, datacite-relations, cobidas,
│                                     #   neurolibre-structure, paper2agent-bundle
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

### Scientific pipeline

| Standard / Tool | What it does | Plugin | Install requirement |
|-----------------|-------------|--------|---------------------|
| **DataLad** | Provenance backbone — records all analysis commands, inputs, outputs | `provenance` | `pip install datalad` |
| **BIDS** | Brain Imaging Data Structure — canonical neuroimaging dataset format | `data-standards` | `npm install -g bids-validator` |
| **Neurobagel / bagel-cli** | Annotate phenotypic variables with controlled terms; push to graph | `annotation` | `pip install bagel-cli` |
| **SNOMED CT** | Clinical terminology — normalize variable names to standard codes | `annotation` | SNOMED CT API key or local OWL |
| **ReproSchema** | Standardized, versioned representation of behavioral assessments / questionnaires — standardizes the tracking of behavioral fields | `annotation` | `pip install reproschema` |
| **OSF / osfclient** | Open Science Framework — push dataset versions, register DOI | `research-export` | `pip install osfclient` |
| **Zenodo / zenodraft** | Zenodo deposit — mint DOI, archive dataset release | `research-export` | `pip install zenodraft` |

### Administration, compliance & credit

| Standard / Tool | What it does | Plugin | Notes |
|-----------------|-------------|--------|-------|
| **RDA DMP Common Standard (maDMP)** | Machine-actionable Data Management Plan format | `project-governance` | DMPTool / DMPonline export; tracked in ledger `dmp:` |
| **OSF Registrations / ClinicalTrials.gov / PROSPERO** | Study pre-registration & registered reports | `project-governance` | registration ID recorded in ledger `registration:` |
| **ORCID** | Persistent researcher identifiers | `project-management` | ledger `people[].orcid` |
| **CRediT (NISO)** | Contributor Roles Taxonomy | `project-management` | ledger `people[].roles` |
| **ROR** | Research Organization Registry identifiers | `project-management` | ledger `affiliation_ror` |
| **Crossref Funder Registry / NIH RePORTER** | Funder & grant identifiers, reporting deadlines | `project-governance` | ledger `funding[]` |
| **EQUATOR (CONSORT/STROBE/PRISMA/ARRIVE)** | Reporting guidelines / checklists | `dissemination` | `reporting-checklist` |
| **COBIDAS** *(neuro pack)* | Neuroimaging reporting standards | `dissemination` | optional neuro reference pack |
| **NIDM (Neuroimaging Data Model)** *(neuro pack)* | Machine-readable neuroimaging annotation & provenance (NIDM-Experiment / NIDM-Results) | `annotation` | `pip install pynidm` |
| **DataCite Metadata Schema** | DOI cross-linking via `RelatedIdentifier` | `dissemination`, `research-export` | `link-outputs` |

### Living research products

| Standard / Tool | What it does | Plugin | Install requirement |
|-----------------|-------------|--------|---------------------|
| **NeuroLibre** | Reproducible preprint server — re-executes the article | `dissemination` | submission via GitHub editorial workflow |
| **MyST / Jupyter Book** | Executable-article authoring format | `dissemination` | `pip install mystmd jupyter-book` |
| **repo2data / BinderHub / repo2docker** | Data + environment reproducibility for execution | `dissemination`, `provenance` | `pip install repo2data` |
| **Paper2Agent + MCP** | Convert the work's methods into an agent-callable MCP server | `dissemination` | uses the harness's own SKILL.md + MCP format |

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

The `project-management` plugin adds a `sessionstart` hook for the obligations reminder:

```yaml
hooks:
  stop: ../provenance/hooks/scripts/datalad-checkpoint.sh   # illustrative
  sessionstart: ./hooks/scripts/obligations-due.sh          # surfaces due obligations (Claude Code)
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

# Validate a project ledger against the schema
ds-harness validate ./project.yaml
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
  - ./plugins/project-governance
  - ./plugins/project-management
  - ./plugins/data-standards
  - ./plugins/annotation
  - ./plugins/provenance
  - ./plugins/data-analysis
  - ./plugins/research-workflow
  - ./plugins/research-export
  - ./plugins/dissemination
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
9. **Provenance for administration too**: administrative metadata lives in a versioned `project.yaml` ledger and is `datalad save`-d — every ethics amendment, DMP revision, and DOI is a tracked commit.
10. **Obligations are first-class**: deadlines and compliance requirements are explicit, queryable ledger entries, never implicit.
11. **Reminders degrade gracefully**: a harness-agnostic on-demand `obligations` skill works everywhere; an optional Claude Code `SessionStart` hook surfaces due items where hooks exist.
12. **Research products are living**: the default export re-executes (NeuroLibre executable article) and is agent-callable (Paper2Agent / MCP bundle), built from the same provenance chain — never a one-off PDF.

---

## Relationship to `my-skills`

This project generalizes the Claude Code-specific plugins in [`my-skills`](../my-skills):

| `my-skills` plugin | `data-science-harness` plugin | Notes |
|--------------------|-------------------------------|-------|
| `stat-analysis` | `plugins/data-analysis` | Add universal frontmatter |
| `project-init` | `plugins/project-management` | Data-analysis project type; adds tracking skills |
| `bids` | `plugins/data-standards` | Full port including reference files |
| `datalad-cli` | `plugins/provenance` | Core subset (run, container-run, save, checkpoint) |
| `nipoppy-cli` | `plugins/data-standards` (BIDS skills) | BIDS conversion subset; full nipoppy in `my-skills` |
| — | `plugins/project-governance` | DMP, ethics, pre-registration, compliance |
| — | `plugins/dissemination` | manuscript, reporting guidelines, living artifacts |

---

## Roadmap

**Phase 1** — Scaffold + port: `harness.yaml`, `pyproject.toml`, and the four ported plugins (`data-analysis`, `provenance`, `data-standards`, `project-management` scaffolding skills) with universal frontmatter.

**Phase 2** — Science & workflow plugins: `annotation` (Neurobagel, SNOMED, NIDM, ReproSchema), `research-export` (OSF, Zenodo, dataset release), `research-workflow` (lit search, experiment design, reproducibility).

**Phase 3** — Administrative & dissemination layer: the `project.yaml` ledger + `schemas/project.schema.json`; `project-governance` (DMP, ethics, pre-registration, compliance-audit); `project-management` tracking skills + the `obligations-due` hook; `dissemination` (manuscript, reporting-checklist, link-outputs, plus the `executable-article` and `agent-bundle` living artifacts). The living-artifact skills depend on `provenance` and `research-export` existing first.

**Phase 4** — Python CLI: `ds-harness` with Claude Code and Cursor adapters first; ledger validation (`ds-harness validate`).

**Phase 5** — Remaining adapters (Copilot, Windsurf, OpenCode, Gemini CLI), PyPI publish, community contribution guidelines.

---

## Contributing

Contributions are Markdown-first. To add a new skill:

1. Pick the right plugin (or propose a new one in an issue)
2. Copy `templates/skill/SKILL.md`, fill in the universal frontmatter and instruction body
3. Add the path to `plugin.yaml`
4. Open a PR

No Python knowledge required. The adapter layer is maintained by core contributors.
