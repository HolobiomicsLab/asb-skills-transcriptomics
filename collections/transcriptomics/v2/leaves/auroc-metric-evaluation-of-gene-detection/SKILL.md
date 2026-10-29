---
name: auroc-metric-evaluation-of-gene-detection
description: Use when benchmarking cluster-free DE methods on single-cell data with
  known ground-truth DE labels (e.g., the simulated COVID-19 cohort 1 dataset) and
  you need per-method AUROC of up- and down-regulated gene detection alongside Pearson
  r.
when_to_use_negative:
- No ground-truth DE labels exist (e.g., real single-cell data without simulated DE
  truth) — AUROC of gene detection is undefined
- No method LFC estimates yet — this is the evaluation/benchmark stage, not the differential
  expression step itself
- Goal is manifold fitting rather than benchmarking (GEDI setup with mode='Bl2'/'Bsphere'
  is the upstream step, evaluated here only via its detection AUROC)
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GEDI
  role: Method under evaluation; its cluster-free DE log-fold-changes (mode='Bl2'
    hyperplane and mode='Bsphere' hyperellipsoid fits) are scored by AUROC against
    simulated ground truth
  repo: https://github.com/csglab/GEDI/
- name: LEMUR
  role: Competing latent-embedding regression DE method whose gene-detection AUROC
    is compared with GEDI's
- name: miloDE
  role: Competing cluster-free DE method included in the same AUROC benchmark
- name: DESeq2
  role: Pseudobulk DE baseline included in the same AUROC benchmark
- name: R
  role: Implementation language for the benchmark notebook and metrics table
  repo: https://github.com/csglab/GEDI_manuscript/tree/main/C_matrices
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:524b6d17b039049653bdc62b4afd86a36dadbb4b27a149f4ae58821b213c752e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - DESeq2
  - GEDI
  - LEMUR
  - R
  - miloDE
  - scran
  - scuttle
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# auroc-metric-evaluation-of-gene-detection

## Summary

Benchmark metric that quantifies how well a cluster-free differential expression method detects up-regulated and down-regulated genes, by computing separate AUROCs of each method's estimated log-fold-changes against simulated ground-truth DE labels (here: GEDI Bl2/Bsphere vs LEMUR, miloDE, and DESeq2 pseudobulk).

## When to use

Use when benchmarking cluster-free DE methods on single-cell data with known ground-truth DE labels (e.g., the simulated COVID-19 cohort 1 dataset) and you need per-method AUROC of up- and down-regulated gene detection alongside Pearson r.

## When NOT to use

- No ground-truth DE labels exist (e.g., real single-cell data without simulated DE truth) — AUROC of gene detection is undefined
- No method LFC estimates yet — this is the evaluation/benchmark stage, not the differential expression step itself
- Goal is manifold fitting rather than benchmarking (GEDI setup with mode='Bl2'/'Bsphere' is the upstream step, evaluated here only via its detection AUROC)

## Inputs

- Simulated cohort-level single-cell count matrix (dgCMatrix, COVID-19 cohort 1 parameters)
- Ground-truth differential expression labels per gene (up-regulated / down-regulated)
- Per-method estimated log-fold-changes (GEDI Bl2/Bsphere, LEMUR, miloDE, DESeq2 pseudobulk)

## Outputs

- fig4_de_benchmark_metrics.csv with per-method median Pearson r, AUROC up-regulated, AUROC down-regulated

## How to apply

On the simulated cohort-level count matrix (sparse dgCMatrix, generated from COVID-19 cohort 1 parameters), first obtain each method's estimated log-fold-changes: GEDI fit with model$setup mode='Bl2' (hyperplane, L2 norm of B fixed) and mode='Bsphere' (hyperellipsoid, randomSeed=1), plus LEMUR, miloDE, and DESeq2 pseudobulk. For each method, correlate estimated log-fold-changes with the simulated ground-truth DE vector and record the median Pearson r. Then dichotomize gene detection using the ground-truth up/down-regulated labels and compute AUROC separately for up-regulated and down-regulated genes, since methods can detect one direction better than the other. Write all per-method metrics (Pearson r, AUROC up, AUROC down) to fig4_de_benchmark_metrics.csv and check against the reference values: median AUROC ≈ 0.79 (GEDI Bl2) vs 0.72 (LEMUR) for up-regulated and ≈ 0.78 vs 0.55 for down-regulated genes.

## Related tools

- **GEDI** (Method under evaluation; its cluster-free DE log-fold-changes (mode='Bl2' hyperplane and mode='Bsphere' hyperellipsoid fits) are scored by AUROC against simulated ground truth) — https://github.com/csglab/GEDI/
- **LEMUR** (Competing latent-embedding regression DE method whose gene-detection AUROC is compared with GEDI's)
- **miloDE** (Competing cluster-free DE method included in the same AUROC benchmark)
- **DESeq2** (Pseudobulk DE baseline included in the same AUROC benchmark)
- **R** (Implementation language for the benchmark notebook and metrics table) — https://github.com/csglab/GEDI_manuscript/tree/main/C_matrices

## Examples

```
metrics <- data.frame(method, pearson_r, auroc_up, auroc_down); write.csv(metrics, "fig4_de_benchmark_metrics.csv", row.names = FALSE)
```

## Evaluation signals

- fig4_de_benchmark_metrics.csv exists with a row per method containing Pearson r, AUROC up, and AUROC down columns
- GEDI hyperplane (Bl2) reaches median AUROC ≈ 0.79 for up-regulated and ≈ 0.78 for down-regulated genes
- LEMUR reaches median AUROC ≈ 0.72 (up) and ≈ 0.55 (down), i.e., notably weaker down-regulation detection
- All AUROC values lie in [0, 1] and GEDI's up/down AUROCs are close (≈0.79/≈0.78), indicating directionally balanced detection
- The companion median Pearson r values reproduce ≈ 0.4 (GEDI Bl2) vs ≈ 0.14 (LEMUR) on the same estimates

## Limitations

- The article notes more extensive analyses are needed to understand how the number of DE genes per cell, DE magnitude, cell-cell/gene-gene correlations, variances, and sample/cell counts affect DE analysis performance
- AUROC is computed on one simulated dataset (COVID-19 cohort 1 parameters); generalization to other simulation regimes or real data with unknown truth is not established by this skill
- Evaluated GEDI fits rely on linear manifold learning (optionally restricted to a hyperellipsoid), which bounds the DE estimates being scored

## Evidence

- [other] Compute AUROC of up-regulated and down-regulated gene detection per method.: [evidence span withheld — anchor status: not_found]
- [other] Expected AUROC values for the benchmark: [evidence span withheld — anchor status: not_found]
- [other] Median Pearson correlation companion metric: [evidence span withheld — anchor status: not_found]
- [readme] Benchmark notebook in manuscript repo: [evidence span withheld — anchor status: external_doc]
- [discussion] Rationale for proper DE evaluation: "Previous work has shown that properly modeling sample-to-sample variability is key for unbiased differential expression analysis in single-cell data"
