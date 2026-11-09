---
name: lisi-integration-metric-evaluation
description: Use when benchmarking whether mapped query cells (e.g., Symphony mapQuery
  output) mix with reference cells across technology or donor labels, by comparing
  LISI against a de novo integration baseline.
when_to_use_negative:
- When the goal is novel cell-type discovery in the query alone — the article notes
  Symphony/mapping metrics do not supplant de novo analyses and metrics may be less
  informative when a novel population is very similar to an existing reference population
- When the reference lacks comprehensive cell-type coverage (Condition I), making
  LISI-based equivalence with de novo integration less flexible/meaningful
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: LISI
  role: Computes Local Inverse Simpson's Index on the joint embedding to quantify
    cell mixing across technology or donor labels
  repo: https://github.com/immunogenomics/LISI
- name: Harmony
  role: Provides the de novo integration embedding whose LISI serves as the gold-standard
    baseline for mapping fidelity
  repo: https://github.com/immunogenomics/singlecellmethods
- name: Symphony
  role: Produces the query-mapped embedding whose LISI is evaluated against the de
    novo Harmony baseline
  repo: https://github.com/immunogenomics/symphony
- name: scArches
  role: Alternative mapping method whose LISI is compared against Symphony in the
    pancreas benchmark
  repo: https://github.com/theislab/scarches
- name: Seurat
  role: Anchor-based integration comparator benchmarked by LISI in the pancreatic
    islet experiment
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:d88448f0a1236f75353648be3a2f49c165c8b2efb6170100472bb018bdfd044b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/LISI
  tools:
  - Harmony
  - LISI
  - Seurat
  - Symphony
  - dplyr
  - irlba
  - scArches
  - uwot
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# lisi-integration-metric-evaluation

## Summary

Quantifies how well single-cell embeddings mix cells across nuisance covariates (technology, donor, batch) using the Local Inverse Simpson's Index (LISI), applied in the Symphony paper to benchmark query mapping against de novo Harmony integration.

## When to use

Use when benchmarking whether mapped query cells (e.g., Symphony mapQuery output) mix with reference cells across technology or donor labels, by comparing LISI against a de novo integration baseline.

## When NOT to use

- When the goal is novel cell-type discovery in the query alone — the article notes Symphony/mapping metrics do not supplant de novo analyses and metrics may be less informative when a novel population is very similar to an existing reference population
- When the reference lacks comprehensive cell-type coverage (Condition I), making LISI-based equivalence with de novo integration less flexible/meaningful

## Inputs

- Joint low-dimensional cell embedding matrix (e.g., 20-dimensional Harmony or Symphony-mapped PCs) covering reference and query cells
- Per-cell categorical metadata vector (technology or donor labels) from pbmcs_meta_small or pancreas islet metadata
- Optional comparator embeddings: de novo Harmony integration of all datasets, and PCA-only baseline for k-NN correlation contrast

## Outputs

- Per-cell LISI scores and their mean (e.g., technology LISI 2.12 Symphony vs 2.15 de novo Harmony; donor LISI ~2.72)
- Per-cell k-NN correlations and fraction exceeding 0.4 at k = 500
- Compiled benchmark table (e.g., pbmc_mapping_benchmark_metrics.csv) with LISI, k-NN-correlation fractions, and classification accuracy/F1 per fold

## How to apply

Run LISI (immunogenomics/LISI implementation, as used in the Harmony paper) on the joint low-dimensional embedding (e.g., 20-PC Harmony or Symphony-mapped space) using a categorical covariate such as technology or donor. For leave-one-technology-out PBMC mapping with pbmcs_exprs_small/pbmcs_meta_small, compute technology LISI on the Symphony-mapped embedding and on the de novo Harmony integration of all three datasets, accepting the mapping if the two values closely match (reported ~2.12 vs ~2.15 on a scale where 1 = no mixing and 2 = near-complete mixing of two technologies). For the multi-donor, multi-species pancreas experiment (four technologies, 10,455-cell inDrop human+mouse query), compute donor LISI on the joint embedding and expect ~2.72 for Symphony. Pair LISI with per-cell k-NN correlation (calcknncorr at k = 500, fraction of cells with correlation > 0.4, reported 87.7% vs 50.2% PCA-only baseline) and label-transfer F1 (evaluate function) so mixing is not rewarded at the cost of cell-type resolution.

## Related tools

- **LISI** (Computes Local Inverse Simpson's Index on the joint embedding to quantify cell mixing across technology or donor labels) — https://github.com/immunogenomics/LISI
- **Harmony** (Provides the de novo integration embedding whose LISI serves as the gold-standard baseline for mapping fidelity) — https://github.com/immunogenomics/singlecellmethods
- **Symphony** (Produces the query-mapped embedding whose LISI is evaluated against the de novo Harmony baseline) — https://github.com/immunogenomics/symphony
- **scArches** (Alternative mapping method whose LISI is compared against Symphony in the pancreas benchmark) — https://github.com/theislab/scarches
- **Seurat** (Anchor-based integration comparator benchmarked by LISI in the pancreatic islet experiment) — https://github.com/satijalab/seurat

## Examples

```
reference = symphony::buildReference(..., vars = 'donor'); query = mapQuery(query_exp, query_meta, reference, ...); lisi::compute_lisi(query$Z, meta, c('technology'))
```

## Evaluation signals

- Symphony-mapped technology LISI closely matches de novo Harmony LISI (reported ~2.12 vs ~2.15 across three leave-one-technology-out PBMC folds)
- Majority of query cells exceed k-NN correlation 0.4 at k = 500 versus the de novo reference embedding (reported 87.7% vs 50.2% for the PCA-only baseline)
- Pancreas donor LISI for Symphony mapping approximates 2.72
- LISI gains are accompanied by preserved label-transfer accuracy: ~97.5% 5-NN classification accuracy, median F1 ~0.97 (PBMCs); human F1 ~0.96, mouse F1 ~0.95 (pancreas)
- Symphony LISI outperforms Seurat anchor-based integration and scArches on the same query as reported in Fig. 4 and Supplementary Tables 8-10

## Limitations

- Mapping metrics may be less informative when a novel query population is very similar to an existing reference population
- Identification of novel cell types that have failed to map is an unsolved future direction for mapping algorithms
- Condition I (comprehensive cell type coverage in the reference) is not flexible and limits equivalence with de novo Harmony integration

## Evidence

- [readme] To assess whether clusters of cells in a single-cell RNA-seq dataset are well-mixed across some categorical variable (e.g. batch, technology, donor), we provide an algorithm for computing a Local Inverse Simpson’s Index (LISI).: [evidence span withheld — anchor status: external_doc]
- [results] Symphony embeddings achieved a mean technology LISI of 2.12 versus 2.15 for de novo Harmony integration, nearly identical mixing of query and reference cells.: [evidence span withheld — anchor status: not_found]
- [supplementary] donor LISI for the Symphony mapping should approximate 2.72, human cell-type F1 approximately 0.96, and mouse F1 approximately 0.95: [evidence span withheld — anchor status: not_found]
- [methods] the majority of query cells exceed k-NN correlation 0.4 at k = 500 (reported 87.7% vs 50.2% PCA-only): [evidence span withheld — anchor status: not_found]
- [discussion] Condition I, which requires comprehensive cell type coverage in the reference, is less flexible.: "Condition I, which requires comprehensive cell type coverage in the reference, is less flexible."
