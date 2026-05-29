# End-to-End Research Workflow with `data-science-harness`

A walkthrough for a researcher setting up and running a complete analysis **using every plugin** in the collection. Steps are ordered by lifecycle stage (0 → 8) with the cross-cutting *Manage & Comply* lane running throughout.

Two kinds of callout are interleaved with the configuration steps:

- > 🔧 **Do-it-yourself** — the scientific/engineering work the harness does *not* do for you (model selection, scripting, plotting, interpretation). The skills scaffold around this work; they don't replace it.
- > ⚠️ **Scaffolding gap** — a place where the current plugin set leaves you on your own and could plausibly be refined.
- > ✅ **Now planned** — a gap the design discussion has since folded into the intended configuration.

Skills are invoked the way your harness invokes them (e.g. `/new-project` in Claude Code, an auto-triggered rule in Cursor). Names below are the universal skill names.

---

## Phase 0 — Install and configure the harness

Before any research begins, stand up the tooling itself.

1. **Install the CLI**: `pip install ds-harness` (or `uv tool install ds-harness`).
2. **Install the plugins for your harness**: `ds-harness install --harness=claude-code` (use `--scope=project` to track the generated configs inside the study repo).
3. **`claude-config`** (`project-management`) — generate `CLAUDE.md`, settings, and MCP stubs for the project.
4. **`env-check`** (`project-management`) — verify the executable dependencies declared in each plugin's `requires:` are present (git, git-annex, datalad, plus step-specific tools like `bids-validator`, `bagel-cli`, a container runtime). Install the missing `python:` ones; follow the printed guidance for `system:`/`npm:` ones.

> 🔧 **Do-it-yourself:** choose your harness, your language (R / Python / Julia), and a package manager (`conda` / `renv` / `uv`). Decide whether analyses will run in a container (recommended — required for `datalad container-run`), and list the preprocessing pipelines you expect to use (fMRIPrep, QSIPrep, …) so `new-project` can scaffold them into the Nipoppy / container config.

> ✅ **Now planned:** `env-check` verifies a container runtime exists, and `new-project` is slated to scaffold a **basic scientific-Python container** plus the preprocessing pipelines you declared at setup. The remaining refinement is pinning the recipe to your exact language/stack — closing the loop between "env present" and "analysis reproducible."

---

## Stage 0 — Propose & Govern

Set up the administrative and scientific *plan* before touching data.

1. **`literature-search`** (`research-workflow`) — *(scope deliberately open, pending participant feedback)* a thin BibTeX-collection helper (PubMed / Semantic Scholar), **not** an AI summary/aggregation engine — many researchers don't want that. The clearer connection is to **meta-analysis tooling** like **NeuroSynth Compose / NiMARE** for a reproducible meta-analytic view of the topic.
2. **`experiment-design`** (`research-workflow`) — power analysis, effect-size estimation, and a pre-registration template.
3. **`init-ledger`** (`project-governance`) — create `project.yaml`, the administrative source of truth.
4. **`people`** (`project-management`) — add collaborators with ORCID and CRediT roles to the ledger.
5. **`dmp`** (`project-governance`) — author a Data Management Plan (RDA maDMP / funder template); its obligations are written into the ledger.
6. **`ethics-track`** (`project-governance`) — record the IRB/IACUC protocol, approval, and expiry (drives a renewal obligation).
7. **`preregister`** (`project-governance`) — register on OSF / ClinicalTrials.gov / PROSPERO; record the ID in the ledger.
8. **`track-milestone`** (`project-management`) — set the study's key dates and deliverables.

> 🔧 **Do-it-yourself:** this is where the *science* is designed. You decide the hypotheses, the primary and secondary outcomes, the inclusion/exclusion criteria, the model family, and the sample-size justification. `experiment-design` computes power once you supply an expected effect size and design — it does not choose them for you.

> ⚠️ **Open design question (not a defect):** rather than locking a single up-front analysis plan, the harness treats analyses as **modular comparisons** added as the story develops and grouped into **products** (see [Analyses as Modular Products](../README.md#analyses-as-modular-products)). Strict pre-registration is one optional mode; the lighter, common alternative is to propose each comparison, `log-decision` as you go, and add supplemental verifications later. The open question — flagged for participant feedback — is how *minimal* the comparison-tracking artifact should be.

---

## Stage 1 — Initialize

1. **`new-project`** (`project-management`) — scaffold a YODA-structured DataLad dataset, a BIDS skeleton, the environment files, and `CLAUDE.md`. (This is also where `init-ledger` lands if you deferred it.)

> 🔧 **Do-it-yourself:** lay out your `code/` directory, decide naming conventions for derivatives, and pin your environment (`environment.yml` / `renv.lock` / `requirements.txt`). Build/select the analysis container now if you're using one.

---

## Stage 2 — Curate

Get raw data into a standardized, annotated form.

1. **`bids-scaffold`** / **`nipoppy-bidsify`** (`data-standards`) — convert raw acquisitions into BIDS layout. If you've adopted **Nipoppy** as your primary tool, this is one command in a framework whose config files also drive Stages 3 and 5 (pipeline running and processing-status tracking).
2. **`bids-validate`** (`data-standards`) — confirm the dataset is BIDS-compliant.
3. **`merge-data`** / **`merge-agent`** (`data-analysis`) — combine tabular phenotypic/clinical sources.
4. **`gen-data-dict`** (`data-analysis`) — generate a data dictionary for the tabular data.
5. **Annotate variables**: `neurobagel-annotate` + `snomed-lookup` (phenotypic/clinical), `reproschema-annotate` (behavioral assessments), `nidm-annotate` (imaging experiment/results). (`annotation`)

> 🔧 **Do-it-yourself:** the real data wrangling — cleaning, format conversion for non-standard inputs, defining variables and units, deciding how to handle missingness and outliers. The skills *standardize and annotate* what you've defined; they don't define it.

> ⚠️ **Scaffolding gap:** **de-identification has no skill.** `compliance-audit` later *checks* for PHI exposure and `dmp`/`ethics-track` impose the obligation, but nothing helps you actually de-identify (defacing imaging, scrubbing PHI columns, date-shifting). For a clinical/neuro workflow this is a high-value, high-risk gap — a `deidentify` skill in `project-governance` or `data-standards` would be worth adding.

> ✅ **Now planned:** declaring expected preprocessing pipelines (fMRIPrep, QSIPrep, …) at setup and wiring them into the **Nipoppy** config covers pipeline selection and configuration; each runs through `datalad container-run`. (Remaining nicety: guided choice of pipeline *parameters*.)

---

## Stage 3 — Analyze

1. **`plan-analysis`** (`data-analysis`) — guided statistical-test selection with QC checks.
2. **`datalad-run`** / **`datalad-container-run`** (`provenance`) — execute every computation so inputs, command, and outputs are recorded. These auto-trigger on `python …`, `Rscript …`, `apptainer exec …`, etc. Preprocessing pipelines declared via **Nipoppy** also run through this path, keeping their provenance intact.

> 🔧 **Do-it-yourself — this is the core gap between scaffolds.** You write the actual analysis: model specification, feature engineering, estimator/hyperparameter choices, the fitting code, and **all plotting/figure code**. `plan-analysis` recommends *which* test; the provenance skills *wrap* whatever script you run — but the script itself, and the model inside it, are entirely yours.

> ⚠️ **Scaffolding gap:** there is **no analysis-script scaffold** and **no plotting/visualization skill**. The jump from `plan-analysis` (choose a test) to `gen-report` (report results) skips the largest part of the work — fitting the model and making the figures. A `scaffold-analysis` skill (emit a runnable, provenance-wrapped script stub for the chosen test) and a `plot`/`figure` skill (consistent, themed exploratory + publication figures) would be the single highest-impact additions to this collection.

> ⚠️ **Scaffolding gap:** analysis code itself is untested. No skill scaffolds unit tests or smoke tests for analysis scripts, which undercuts the reproducibility promise.

---

## Stage 4 — Checkpoint

1. **`datalad-save`** / **`checkpoint`** (`provenance`) — structured commits of intermediate state; an auto-hook also checkpoints at session end.
2. **`log-decision`** (`project-management`) — record *why* you made each analytic choice, into the decision log.

> 🔧 **Do-it-yourself:** analysis is iterative — loop Stage 3 ↔ 4. Capture the reasoning behind branch points (why this covariate set, why this transform); that log feeds your eventual Methods section and protects against "why did we do that?" six months later.

---

## Stage 5 — QC / Review

1. **`reproducibility`** (`research-workflow`) — audit whether the analysis reproduces from the DataLad log; check input availability via `datalad get`.
2. **`bids-validate`** (`data-standards`) — re-validate after derivatives are added.
3. **`gen-report`** (`data-analysis`) — scaffold an analysis report (results tables, QC metrics).
4. **`compliance-audit`** (`project-governance`) — check ledger obligations, de-identification, and DUA data-scope against what's actually present.

> 🔧 **Do-it-yourself:** interpret the results, run sensitivity/robustness analyses, check statistical assumptions, and decide whether the findings are publication-ready. The audits confirm the work is *reproducible and compliant*; they do not tell you whether it is *correct or meaningful*.

> ⚠️ **Scaffolding gap:** no skill supports sensitivity analyses, assumption checking, or multiple-comparison correction as active steps (QC references exist, but nothing drives them). And nothing checks the executed analysis against the **pre-registered** plan — pairing this with the Stage-0 analysis-plan artifact would let `reproducibility` also report prereg adherence.

---

## Stage 6 — Export

1. **`export-results`** (`research-export`) — bundle `outputs/` into a standalone archive with a provenance summary generated from `datalad log`.
2. **`osf-push`** (`research-export`) — push the dataset version to an OSF node and register it as a DataLad sibling.

> 🔧 **Do-it-yourself:** decide *what* is shareable (which derivatives, which intermediates), the access level (public / embargo), and any storage constraints.

---

## Stage 7 — Publish

1. **`dataset-release`** (`research-export`) — bump `dataset_description.json` version, write a BIDS `CHANGES` entry, create a git tag, and optionally mint a Zenodo DOI.
2. Finalize **`nidm-annotate`** and push the **Neurobagel** graph (`annotation`).

> 🔧 **Do-it-yourself:** choose the dataset license, citation, and authorship; decide semantic-versioning policy for the data product.

---

## Stage 8 — Disseminate & Report

Produce the living research compendium and the classic outputs.

1. **`draft-manuscript`** (`dissemination`) — IMRaD scaffold with Methods / Data-availability / provenance auto-filled from the DataLad log + ledger.
2. **`reporting-checklist`** (`dissemination`) — apply the right EQUATOR guideline (CONSORT / STROBE / PRISMA / ARRIVE) or COBIDAS for neuroimaging.
3. **`executable-article`** (`dissemination`) — scaffold a NeuroLibre reproducible preprint (MyST / Jupyter Book + `binder/` + `repo2data`) whose figures regenerate from the pipeline.
4. **`agent-bundle`** (`dissemination`) — emit a Paper2Agent-style MCP server exposing the methods as callable, tested tools.
5. **`link-outputs`** (`dissemination`) — cross-link dataset / code / paper / preprint / prereg / executable-article / agent-bundle DOIs (DataCite relations) back into the ledger.
6. **`submission-track`** (`dissemination`) — track target journal, submission, and revisions.
7. **`status-report`** (`project-management`) — generate the funder/progress report from the ledger + history.

> 🔧 **Do-it-yourself:** write the science — intro, discussion, related work, the narrative — and prepare **publication-quality figures**. The manuscript scaffold fills in the mechanical/provenance sections; the intellectual content, journal selection, cover letter, and reviewer responses are yours.

> ⚠️ **Scaffolding gap:** publication-figure preparation is again unscaffolded (ties back to the missing plotting skill). The `agent-bundle` also assumes your analysis code is already structured as importable, parameterized functions — if your Stage-3 scripts were one-off, bundling them as tools is real work that nothing helps with.

---

## Ongoing — Manage & Comply (all stages)

Running in parallel from Stage 0 onward:

- **`obligations`** (`project-management`) — on demand, list what's due; a Claude Code `SessionStart` hook surfaces items due soon.
- **`track-milestone`**, **`log-decision`**, **`people`**, **`status-report`** — keep the ledger current.
- **`compliance-audit`** — re-run periodically, not just at QC.

> ⚠️ **Scaffolding gap:** reminders are pull-based (a skill you invoke) plus an opt-in Claude hook. There's no cross-harness push for a deadline you'd miss while *not* in a session — acceptable for v1, but worth noting for users who live in their calendar, not their terminal.

---

## Scaffolding gaps, consolidated

Ranked by likely impact, these are the refinements most worth a hackathon's attention:

1. **`scaffold-analysis`** — emit a runnable, provenance-wrapped script stub for the test chosen by `plan-analysis`. (Bridges the biggest gap, between Stages 3 and 5.)
2. **`plot` / `figure`** — consistent exploratory and publication figures. (The only entirely-unserved core activity.)
3. **`deidentify`** — actually remove PHI, not just audit for it. (High risk in clinical/neuro work.)
4. **Lightweight comparison tracking** — a minimal, standard way to propose and track a *comparison* and group it into a *product* (conceptually DataLad branches), addable at any point. Strict pre-registration adherence is one optional mode. *(Proposed; open for feedback — see README.)*
5. **`build-container`** — *(partly planned)* `new-project` will scaffold a basic scientific-Python container; extend it to pin the exact language/stack and emit the recipe.
6. **Preprocessing pipelines** — *(partly addressed)* declaring expected pipelines and wiring them into Nipoppy covers selection/config; the residual gap is guided parameter choice.
7. **Analysis-code testing** — smoke/unit tests for the scripts the provenance layer wraps.

---

## Recommended process for planning a new analysis

Two things genuinely lock *early* because they shape everything downstream — **governance/compliance** (funding, ethics, data-management obligations) and the **data model** (variables, units, standards). Most *analytic* decisions do **not** need to be fixed up front: analyses are modular comparisons you add as the story develops (see [Analyses as Modular Products](../README.md#analyses-as-modular-products)). Plan in roughly this order:

1. **Frame the question** (`literature-search`, meta-analysis tools) — what's known, what's the gap. *(lightweight)*
2. **Stand up governance** (`init-ledger`, `dmp`, `ethics-track`, `people`) — encode obligations and credit before data exists. These genuinely lock early.
3. **Sketch the first comparison(s)** (`experiment-design`) — outcomes, design, effect size → power/sample size. Use the proposal template per comparison, as needed.
4. **Choose your rigor mode:**
   - *Strict / confirmatory* — write the exact models and decision rules and **pre-register** (`preregister`); later changes become reportable deviations.
   - *Exploratory / flexible (common)* — skip the prereg gate; propose each comparison lightly, `log-decision` as you go, and add comparisons + supplemental verifications to a **product** over time.
5. **Initialize** (`new-project`) — scaffold the dataset, environment/container, and expected preprocessing pipelines.
6. Proceed through Curate → Analyze → …, adding comparisons non-linearly (DataLad branches) and grouping the ones worth publishing into products.

### When each decision must be finalized

| Decision | Finalize by | Recorded in | Why it locks there |
|----------|-------------|-------------|--------------------|
| Hypotheses, primary/secondary outcomes | **Stage 0** *(if pre-registering)* | prereg + ledger `registration` | Defines everything downstream; changing later = deviation |
| Inclusion/exclusion, stopping rules | **Stage 0** *(if pre-registering)* | prereg | Must precede data collection |
| Confirmatory model spec, covariates, corrections | **Stage 0** *(if pre-registering)* | analysis plan | Separates confirmatory from exploratory |
| Sample size / power | **Stage 0** | `experiment-design` output | Determines feasibility & cost |
| Data-management & sharing obligations | **Stage 0** | ledger `dmp` | Funder-mandated; sets later deadlines |
| Ethics scope & de-identification approach | **Stage 0–2** | ledger `ethics` | Gates what data may exist/leave |
| Tech stack, env, container, naming conventions | **Stage 1** | `new-project` / env files | Cheap now, expensive to change after data lands |
| Variable definitions, units, data dictionary | **Stage 2** | `gen-data-dict` + annotations | Must be stable before analysis runs |
| Missingness/outlier handling rules | **Stage 2–3** | decision log | Ideally pre-specified; otherwise log as analytic choice |
| Software/package versions | **Stage 3** | container digest / lockfile | Pinned so results reproduce |
| Final result set & sensitivity analyses | **Stage 5** | `gen-report` + decision log | After interpretation, before publication |
| Dataset version, license, DOI, access level | **Stage 6–7** | `dataset-release` / `dataset_description.json` | At the point of sharing |
| Target journal, reporting guideline, author order | **Stage 8** | ledger `products` + `submission-track` | At write-up; affects format & credit |
| Which living artifacts to produce (article, agent bundle) | **Stage 8** | ledger `products` | Depends on a stable, reproducible pipeline existing first |

**Rule of thumb:** if a decision changes a *compliance obligation* or the *data model* (variables, units, standards), finalize it early — before data lands. Pre-registration additionally locks the confirmatory analysis plan, but that's an *optional* mode. Analytic and interpretive decisions can be made — and added as new comparisons — as the work develops; just `log-decision` when you make them.
