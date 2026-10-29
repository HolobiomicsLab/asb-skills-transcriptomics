---
name: lisi-integration-quality-metric-computation
description: Use when two or more scRNA-seq datasets (e.g., Muraro CEL-Seq2 and Segerstolpe
  Smart-Seq2) have been integrated by batch-correction tools and you need numeric
  iLISI/cLISI scores, plus runtime, to rank the methods.
when_to_use_negative:
- Only one dataset is present with no batch structure to mix — iLISI is uninformative
  (both iLISI and cLISI stay near 1.0, as with no correction at iLISI 1.07 / cLISI
  1.05)
- BATMAN integration is intended for more than two datasets — the current implementation
  works only with two datasets, so LISI comparison of a multi-dataset integration
  cannot use it directly
- The analysis goal is cell-type annotation accuracy rather than batch-mixing quality
  — cLISI alone does not validate label transfer
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BATMAN
  role: Integrator being scored; batch correction via minimum-weight matching, evaluated
    at iLISI 1.55 / cLISI 1.08 in 9 s on the pancreas benchmark
  repo: https://github.com/mandricigor/batman
- name: Seurat V3.0
  role: Comparison integrator (iLISI 1.35) and source of the pre-processing steps
    used to obtain log-counts for all tools
- name: MNNcorrect
  role: Comparison MNN-based integrator scored at iLISI 1.18 / cLISI 1.08 in 228.6
    s
- name: Scanorama
  role: Comparison integrator scored at iLISI 1.22 / cLISI 1.04 in 14 s
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:20986fd423cc3a55c6cc985c67f2cf75c6d6015dd5487a93efac437e3a176320
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mandricigor/batman
  repo_url: https://github.com/mandricigor/batman
  tools:
  - BATMAN
  - MNNcorrect
  - Scanorama
  - Seurat V3.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lisi-integration-quality-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computation of the LISI family of integration quality metrics (iLISI for batch mixing, cLISI for cell-type/local-structure preservation) to quantitatively compare scRNA-seq batch-correction methods such as BATMAN, Seurat V3.0, MNNcorrect, and Scanorama on integrated representations.

## When to use

Use when two or more scRNA-seq datasets (e.g., Muraro CEL-Seq2 and Segerstolpe Smart-Seq2) have been integrated by batch-correction tools and you need numeric iLISI/cLISI scores, plus runtime, to rank the methods.

## When NOT to use

- Only one dataset is present with no batch structure to mix — iLISI is uninformative (both iLISI and cLISI stay near 1.0, as with no correction at iLISI 1.07 / cLISI 1.05)
- BATMAN integration is intended for more than two datasets — the current implementation works only with two datasets, so LISI comparison of a multi-dataset integration cannot use it directly
- The analysis goal is cell-type annotation accuracy rather than batch-mixing quality — cLISI alone does not validate label transfer

## Inputs

- Integrated (batch-corrected) expression representation from each tool, computed on identical Seurat V3.0 log-count inputs per dataset
- Cell batch/dataset labels used as the mixing covariate for iLISI
- Cell-type annotations used as the covariate for cLISI
- Per-tool wall-clock runtime measurements

## Outputs

- Mean iLISI and mean cLISI per method on each integrated representation
- Comparison table (e.g., integration_benchmark.csv) with iLISI, cLISI, and runtime per tool
- Optional mean 50-RNN values as a local-structure retention check

## How to apply

After running each integrator (BATMAN via minimum-weight matching on a bipartite graph with Euclidean-distance-weighted edges, or competitors like Seurat V3.0, MNNcorrect, Scanorama) on the same Seurat V3.0 log-count inputs, compute the mean iLISI and cLISI over cells in each integrated representation. In the BATMAN pancreas all-cell-types benchmark, BATMAN scored iLISI 1.55 / cLISI 1.08 (9 s), versus Seurat V3.0 (1.35/1.03, 22.3 s), MNNcorrect (1.18/1.08, 228.6 s), and Scanorama (1.22/1.04, 14 s); uncorrected data gave iLISI 1.07 / cLISI 1.05. Interpret higher iLISI as better batch mixing and lower cLISI near 1 as better retention of cell-type/local structure — a good method must improve iLISI without inflating cLISI. Also record wall-clock runtime per tool, and pair LISI with a neighborhood-retention metric such as mean 50-RNN to detect methods that mix batches while destroying local structure (e.g., Scanorama at 0.14 vs BATMAN at 0.92 in the LB scenario).

## Related tools

- **BATMAN** (Integrator being scored; batch correction via minimum-weight matching, evaluated at iLISI 1.55 / cLISI 1.08 in 9 s on the pancreas benchmark) — https://github.com/mandricigor/batman
- **Seurat V3.0** (Comparison integrator (iLISI 1.35) and source of the pre-processing steps used to obtain log-counts for all tools)
- **MNNcorrect** (Comparison MNN-based integrator scored at iLISI 1.18 / cLISI 1.08 in 228.6 s)
- **Scanorama** (Comparison integrator scored at iLISI 1.22 / cLISI 1.04 in 14 s)

## Evaluation signals

- Mean iLISI for a genuinely mixing method should exceed the no-correction baseline of 1.07 while mean cLISI stays near 1.05
- cLISI remaining near 1.0 (e.g., 1.03–1.08 for all four methods) indicates cell-type/local structure was not collapsed during correction
- All tools must be evaluated on the same Seurat V3.0 log-count representation of the raw datasets (Muraro 2,285 cells; Segerstolpe 2,394 cells) for scores to be comparable
- In the LB scenario, a structure-retention check via mean 50-RNN should show high values for good methods (BATMAN 0.92) versus mixing-at-any-cost methods (Scanorama 0.14)
- Benchmark table rows should include runtime per method (BATMAN 9 s, Seurat 22.3 s, MNNcorrect 228.6 s, Scanorama 14 s) and save cleanly as integration_benchmark.csv

## Limitations

- LISI scores alone can be maximized by over-mixing; pair with local-structure metrics such as mean 50-RNN to detect loss of cell-to-cell relationships
- The comparison is only meaningful when all integrators consume identical pre-processed inputs (same log-counts); different pipelines make iLISI/cLISI incomparable
- MNN-based methods may fail to both correct batch effects and maintain local structure, so iLISI and cLISI must be reported together, not individually

## Evidence
