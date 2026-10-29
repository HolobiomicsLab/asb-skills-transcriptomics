---
name: integration-metric-evaluation-cilisi-asw
description: Use when you have integrated (or uncorrected) scRNA-seq embeddings and
  true cell type annotations and need to quantify batch mixing (CiLISI) and preservation
  of biological variance (celltype ASW) across methods such as ssSTACAS, scANVI, scGen,
  or Harmony.
when_to_use_negative:
- Cell type annotations are unavailable for the cells being scored — CiLISI and similar
  cell type-aware metrics require cell type annotations
- A cell type of interest is present in only one of the datasets being integrated,
  or datasets have highly unequal numbers of cells — CiLISI cannot be calculated or
  is suboptimal in these cases
- You only need to integrate technical replicates — cell type-agnostic batch mixing
  suffices there, but its utility is arguably very limited beyond that setting
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scIntegrationMetrics
  role: R package providing LISI, per-cell type LISI (CiLISI), and silhouette average
    width (ASW) metrics, with Seurat compatibility and per-cell-type/subset computation,
    used to score integration quality
  repo: https://github.com/carmonalab/scIntegrationMetrics
- name: scib-pipeline
  role: Snakemake benchmarking pipeline (built on scib) that orchestrates integration
    method runs and computes the CiLISI metric on each method's latent embedding
  repo: https://github.com/carmonalab/scib-pipeline
- name: STACAS
  role: Integration method whose semi-supervised variant is the reference top performer
    whose celltype ASW and CiLISI scores anchor the evaluation expectations
  repo: https://github.com/carmonalab/STACAS
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:48799384e8b0336ef8bb2192b846116f18a31b296c23bbe03e4547f58b854b7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/scIntegrationMetrics
  tools:
  - FastMNN
  - Harmony
  - STACAS
  - Scanorama
  - Seurat v4
  - scANVI
  - scGen
  - scIntegrationMetrics
  - scVI
  - scib-pipeline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# integration-metric-evaluation-cilisi-asw

## Summary

Evaluate the quality of single-cell RNA-seq data integration by computing the cell-type-aware batch-mixing metric CiLISI and the biological-variance-preservation metric celltype ASW, using the carmonalab/scIntegrationMetrics package within the carmonalab/scib-pipeline benchmarking framework.

## When to use

Use when you have integrated (or uncorrected) scRNA-seq embeddings and true cell type annotations and need to quantify batch mixing (CiLISI) and preservation of biological variance (celltype ASW) across methods such as ssSTACAS, scANVI, scGen, or Harmony.

## When NOT to use

- Cell type annotations are unavailable for the cells being scored — CiLISI and similar cell type-aware metrics require cell type annotations
- A cell type of interest is present in only one of the datasets being integrated, or datasets have highly unequal numbers of cells — CiLISI cannot be calculated or is suboptimal in these cases
- You only need to integrate technical replicates — cell type-agnostic batch mixing suffices there, but its utility is arguably very limited beyond that setting

## Inputs

- Seurat objects or h5ad/latent embeddings from integration methods (50-dimensional latent space per method)
- True (uncorrupted) cell type annotations for every cell in the integrated collection
- Benchmark scRNA-seq collections (Pancreas, Immune, Lung, mouse CD8 T cell Utility collection)

## Outputs

- Per-method, per-task CiLISI and celltype_ASW scores (e.g. label_corruption_benchmark_scores.csv)
- Aggregated ranking table across tasks (e.g. integration_benchmark_ranking.csv)
- Metric-versus-corruption-level or ranking comparison plots

## How to apply

Load each Seurat-integrated or scib-pipeline-processed collection with its TRUE (uncorrupted) cell type labels — never the noisy inputs used to run supervised methods (e.g. 15% unknown, 20% shuffled labels). Extract a 50-dimensional latent embedding per method (the benchmark standard in this study) and compute the cell-type-aware LISI (CiLISI) and silhouette average width by cell type (celltype_ASW) via carmonalab/scIntegrationMetrics, orchestrated through the carmonalab/scib-pipeline Snakemake pipeline. Aggregate metric pairs per task (Pancreas, Immune, Lung, mouse T cells) and compute per-method global ranks to compare integration tools. Judge success against the paper's expected patterns: Harmony attains high batch mixing (CiLISI) but low celltype ASW, while semi-supervised STACAS attains higher celltype ASW than unsupervised STACAS and scANVI without sacrificing CiLISI.

## Related tools

- **scIntegrationMetrics** (R package providing LISI, per-cell type LISI (CiLISI), and silhouette average width (ASW) metrics, with Seurat compatibility and per-cell-type/subset computation, used to score integration quality) — https://github.com/carmonalab/scIntegrationMetrics
- **scib-pipeline** (Snakemake benchmarking pipeline (built on scib) that orchestrates integration method runs and computes the CiLISI metric on each method's latent embedding) — https://github.com/carmonalab/scib-pipeline
- **STACAS** (Integration method whose semi-supervised variant is the reference top performer whose celltype ASW and CiLISI scores anchor the evaluation expectations) — https://github.com/carmonalab/STACAS

## Evaluation signals

- Metrics are computed against the TRUE (uncorrupted) labels, not the perturbed inputs (e.g. not the 15% unknown / 20% shuffled noisy labels used to run supervised methods)
- Each method's embedding is reduced to a 50-dimensional latent space before scoring, matching the benchmark configuration
- Harmony, if included as a comparator, should show high CiLISI (batch mixing) but low celltype ASW — a useful sanity check that the metrics are being computed correctly
- Semi-supervised STACAS should attain higher celltype ASW than unsupervised STACAS with no negative impact on CiLISI; across four tasks ssSTACAS should rank first globally
- All per-method/per-task metric pairs are written to a structured CSV (e.g. integration_benchmark_ranking.csv or label_corruption_benchmark_scores.csv)

## Limitations

- CiLISI and similar metrics require cell type annotations, which may not exist for exploratory integrations
- CiLISI cannot be calculated for cell types that are only present in one dataset
- As with LISI, CiLISI remains suboptimal when datasets have highly unequal numbers of cells; batch mixing metrics robust to lopsided cell proportions are still lacking
- Benchmarks that include supervised integration methods should account for noisy and partial cell-type information, as supervised method performance depends strongly on label quality

## Evidence

- [discussion] CiLISI requires cell type annotations: "One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations."
- [discussion] CiLISI single-dataset cell type limitation: "Another limitation is that CiLISI cannot be calculated for cell types that are only present in one dataset."
- [discussion] CiLISI unequal cell numbers limitation: "as LISI, CiLISI remains suboptimal when datasets have highly unequal numbers of cells"
- [readme] scIntegrationMetrics scope: [evidence span withheld — anchor status: external_doc]
- [readme] scib-pipeline benchmark scope: [evidence span withheld — anchor status: external_doc]
