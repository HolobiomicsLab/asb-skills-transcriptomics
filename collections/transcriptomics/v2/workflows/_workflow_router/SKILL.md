---
name: transcriptomics-workflow-router
description: Use when a user has a whole transcriptomics analysis goal rather than a single atomic step and needs an end-to-end composite workflow selected and grounded against source papers.
license: CC-BY-4.0
metadata:
  kind: workflow-router
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/transcriptomics/v2
  indexes:
  - workflows_index.json
  bound_by: perspicacite-semantic
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# ASB Transcriptomics composite workflow router

This is the goal-level entry point of ASB Transcriptomics. The leaf router (`transcriptomics-router`)
finds one atomic skill; this router selects an end-to-end workflow with ordered
stages, candidate leaf skills, grounding and a declarative `workflow.yaml`.

## 1. Select

Run the commands as written, from any working directory; do not `cd`.
If a command still shows a `${...}` variable instead of an absolute path, the
host did not fill it in: replace the variable with the absolute directory of
this file, `<collection root>/workflows/_workflow_router`.

```bash
python3 "${CLAUDE_SKILL_DIR}/../../bin/search_skills.py" --query "<the user's goal>" --target workflows -k 3
```

Python 3.9 or later, standard library only, no network; where `python3` is missing, use `python` or `py -3`.
For a manual lookup, match the goal against the `description` of each row of
`${CLAUDE_SKILL_DIR}/../workflows_index.json`.

Available workflows (published as outlines):

| workflow | what it does |
|---|---|
| `bulk-rnaseq-differential-expression` | Trim reads, quantify and aggregate gene counts with Kallisto, derive consensus differential-expression calls and rankings, and run pathway enrichment. |
| `single-cell-clustering-annotation` | Filter raw counts, construct a scaled log-CPM matrix and PCA embedding, cluster cells with Leiden, identify cluster-specific markers, and assign cell types. |

If no workflow fits the goal, fall back to the leaf router (`transcriptomics-router`) and
assemble steps from atomic skills.

## 2. Run

Read the chosen `workflows/<slug>/SKILL.md` (the `read:` path of the hit) and
follow its **Stages** in order. Each stage carries a goal, candidate leaf skills
(primary first), the tools to install or invoke, and its typed inputs and
outputs. The machine-readable `workflows/<slug>/workflow.yaml` is the DAG
(`after`, `inputs_from`). Automatic grading of that DAG (`asb solve-workflow`)
is **not part of this release**: no released ASB version loads these files, so
run the stages yourself. Honor the I/O contract: each stage consumes the prior
stage's declared outputs. Optional stages are marked.

For a stage's leaf skills, read each `${CLAUDE_SKILL_DIR}/../../leaves/<leaf-slug>/SKILL.md` for
the procedure, or use the leaf router to pick among the candidates for the
exact data.

## 3. Ground

Before trusting a parameter or default, ground the stage's leaves against the
papers they were distilled from. Each stage's `grounding.kb_slugs` and `dois` in
`workflow.yaml` name the source papers and their `asb-paper-<doi>` knowledge
bases.

> These workflows are published as **outlines**: the stage structure is
> validated, the execution is not. Bindings were chosen by semantic retrieval
> plus deterministic selection. `derived_from_workflows` is a provenance record;
> no ablation experiment consuming it is released.
