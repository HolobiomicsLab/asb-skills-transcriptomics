---
name: scrnaseq-embedding-suitability-evaluation
description: Use when fitting trajectories (e.g., Slingshot, Monocle3) on scRNA-seq
  data and you must decide among candidate embeddings from feature selection (number
  of highly variable genes) and dimension reduction choices, or need to know if a
  trajectory is appropriate at all.
when_to_use_negative:
- The trajectory inference method of interest does not rely on a two-dimensional representation
  — Escort only considers trajectory inference approaches that use two-dimensional
  representations, so methods outside this restriction require further evaluation
- None of the evaluated embeddings is suitable — Escort is not guaranteed to identify
  a recommended embedding among the choices being evaluated
- You need RNA velocity-derived dimension reductions — support for these is not yet
  implemented in Escort (future work)
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Escort
  role: R package / R-Shiny app implementing the three-step framework that scores
    candidate embeddings for trajectory-inference suitability and flags disconnected
    or trajectory-signal-lacking data
  repo: https://github.com/xiaorudong/Escort
- name: Slingshot
  role: Fits trajectories on the Escort-recommended embedding vs. the original/default
    embedding (with mclust hierarchical clusters) to compare lineage recovery
- name: scLANE
  role: Tests genes for trajectory differential expression on both fitted trajectories
    (significant at FDR-adjusted p < 0.01) to quantify differences between embeddings
- name: Shiny
  role: Interactive app interface for running Escort's embedding evaluations locally
    via Escort::shinyEscort()
- name: Monocle3
  role: Source of the original paper's default embedding used as the comparison baseline
    in the case study
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:215e4de626f9a55c963bbe1872a34c59662d38a2aa95f7cadb308e23cda2b70e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaorudong/Escort
  repo_url: https://github.com/xiaorudong/Escort
  tools:
  - Enrichr
  - Escort
  - Monocle3
  - R
  - Shiny
  - Slingshot
  - scLANE
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# scrnaseq-embedding-suitability-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A data-driven skill for evaluating whether a scRNA-seq dataset and its candidate embeddings (feature selection, dimension reduction, hyperparameter choices) are suitable for trajectory inference, using the Escort framework's standardized benchmark score range from negative infinity to two where embeddings scoring >0 are recommended.

## When to use

Use when fitting trajectories (e.g., Slingshot, Monocle3) on scRNA-seq data and you must decide among candidate embeddings from feature selection (number of highly variable genes) and dimension reduction choices, or need to know if a trajectory is appropriate at all.

## When NOT to use

- The trajectory inference method of interest does not rely on a two-dimensional representation — Escort only considers trajectory inference approaches that use two-dimensional representations, so methods outside this restriction require further evaluation
- None of the evaluated embeddings is suitable — Escort is not guaranteed to identify a recommended embedding among the choices being evaluated
- You need RNA velocity-derived dimension reductions — support for these is not yet implemented in Escort (future work)

## Inputs

- Normalized scRNA-seq expression matrix (e.g., preprocessed data from GEO accession GSE190616)
- Set of candidate low-dimensional embeddings generated under different feature selection (highly variable gene counts) and dimension reduction choices (PCA, t-SNE, UMAP, diffusion maps)
- Cell-level quality control metadata (unique gene counts, total molecule counts, percent mitochondrial reads) for upstream filtering

## Outputs

- Per-embedding benchmark scores on the Escort scale (negative infinity to 2), with recommended embeddings defined as total score > 0
- Classification of embeddings as recommended / not-recommended (disconnected embeddings are immediately not recommended)
- Density and ambiguous-cell cutoffs (e.g., 0.53 and 0.029 in the case study)
- Comparison of trajectory fits (lineage count, scLANE dynamic genes at FDR < 0.01) between recommended and default embeddings

## How to apply

Run Escort's three-step framework: first assess whether fitting a trajectory is appropriate for the dataset (disconnected clusters or homogeneous cells lacking trajectory signal are flagged); then evaluate a grid of candidate embeddings — e.g., 13 options spanning PCA with varying top percentages of highly variable genes — using trajectory-specific metrics and the standardized benchmark score (negative infinity to 2; any embedding found disconnected is immediately classified as not recommended). Select the embedding with total score >0 (in the Long et al. 2022 Col10a1 hypertrophic chondrocyte case, the PCA embedding using the top 10% of highly variable genes won, with density cutoff 0.53 and ambiguous-cell cutoff 0.029). Downstream, fit Slingshot on the recommended embedding versus the original/default embedding using hierarchical clusters (mclust), then test genes with scLANE at FDR < 0.01 to compare lineage recovery and dynamic-gene sets. Escort is run via the R package or R/Shiny app (Escort::shinyEscort()), installed from GitHub with remotes::install_github.

## Related tools

- **Escort** (R package / R-Shiny app implementing the three-step framework that scores candidate embeddings for trajectory-inference suitability and flags disconnected or trajectory-signal-lacking data) — https://github.com/xiaorudong/Escort
- **Slingshot** (Fits trajectories on the Escort-recommended embedding vs. the original/default embedding (with mclust hierarchical clusters) to compare lineage recovery)
- **scLANE** (Tests genes for trajectory differential expression on both fitted trajectories (significant at FDR-adjusted p < 0.01) to quantify differences between embeddings)
- **Shiny** (Interactive app interface for running Escort's embedding evaluations locally via Escort::shinyEscort())
- **Monocle3** (Source of the original paper's default embedding used as the comparison baseline in the case study)

## Examples

```
remotes::install_github("xiaorudong/Escort"); library(Escort); shinyEscort()
```

## Evaluation signals

- The selected embedding must receive a total Escort score greater than zero on the standardized benchmark scale (negative infinity to 2)
- Any embedding found to be disconnected must be classified as not recommended, regardless of other metrics
- Downstream comparison should reproduce the paper's Figure 6 contrast: recommended embedding yields two Slingshot lineages (osteoblastic fate and terminal apoptosis) vs. three lineages from the original embedding, with only the osteoblastic lineage characterizable under the original
- scLANE dynamic genes must pass FDR-adjusted p < 0.01, with gene set enrichment (Enrichr BioPlanet 2019 / MSigDB GO biological processes for mouse) supporting the reported biological fates

## Limitations

- Escort only considers trajectory inference approaches that use two-dimensional representations; methods outside this restriction require further evaluation
- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated
- Support for RNA velocity-derived dimension reduction methods is not yet implemented (future work)
- Quality control steps may significantly impact Escort's embedding evaluations; no software version specified for Escort or R

## Evidence
