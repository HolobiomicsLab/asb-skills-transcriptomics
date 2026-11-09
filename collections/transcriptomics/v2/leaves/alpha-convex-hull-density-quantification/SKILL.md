---
name: alpha-convex-hull-density-quantification
description: Use when evaluating 2D scRNA-seq embeddings (e.g., MDS/UMAP/t-SNE of
  2000/4000/10000 highly variable genes) for trajectory inference suitability by quantifying
  cell coverage area as the alpha-convex hull proportion of the minimum enclosing
  circle (GOF).
when_to_use_negative:
- The embedding shows newly disconnected clusters — DCcheck classifies it immediately
  as not recommended, so GOF/density quantification is not the appropriate next step.
- The trajectory inference approach does not use a two-dimensional representation
  — Escort only considers two-dimensional embeddings.
- The embedding derives from RNA velocity-based dimension reduction — not yet implemented
  in Escort's framework.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: Escort
  role: R package providing calcScore and the GOF/coverage-area evaluation that converts
    the alpha-convex hull proportion into a benchmark score for embedding suitability
  repo: https://github.com/xiaorudong/Escort
- name: R
  role: Implementation environment; Escort is installed from GitHub and run as a package
    or local Shiny app
  repo: https://github.com/xiaorudong/Escort
- name: alphahull
  role: CRAN dependency supplying the alpha-convex hull computation underlying the
    coverage-area (GOF) metric
- name: Slingshot
  role: Used to infer pseudotime (with Monocle3) for computing ground-truth comparisons
    (Kendall correlation, MSE) against Escort scores
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:800b6fadb534d157f3832d65f83d06e62ee4c3d5bce33d8e1c1399fc6b38d8c5
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
  - Escort
  - Monocle3
  - R
  - Slingshot
  - alphahull
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# alpha-convex-hull-density-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Escort's GOF component quantifies how densely a 2D single-cell embedding covers its space, computed as the alpha-convex hull area of cells relative to the minimum enclosing circle area. This coverage-area proportion feeds Escort's benchmark score (range −∞ to 2; >0 recommended) for judging embedding suitability for trajectory inference.

## When to use

Use when evaluating 2D scRNA-seq embeddings (e.g., MDS/UMAP/t-SNE of 2000/4000/10000 highly variable genes) for trajectory inference suitability by quantifying cell coverage area as the alpha-convex hull proportion of the minimum enclosing circle (GOF).

## When NOT to use

- The embedding shows newly disconnected clusters — DCcheck classifies it immediately as not recommended, so GOF/density quantification is not the appropriate next step.
- The trajectory inference approach does not use a two-dimensional representation — Escort only considers two-dimensional embeddings.
- The embedding derives from RNA velocity-based dimension reduction — not yet implemented in Escort's framework.

## Inputs

- 2D cell embeddings of scRNA-seq data (e.g., UMAP, t-SNE, MDS on subsets of highly variable genes)
- Single-cell expression matrix / SingleCellExperiment (e.g., human pancreas reference, 500 cells, 15,894 genes)
- Ground-truth pseudotime for validation of embedding accuracy (Kendall correlation, MSE)

## Outputs

- Per-embedding GOF coverage-area proportion (alpha-convex hull area / minimum enclosing circle area)
- Density subscore S_density = (0.53 − x)/0.53 and combined Escort total score (range −∞ to 2)
- Recommended / not-recommended classification per embedding (score > 0 recommended)
- escort_score_correlation.csv correlating Escort scores with Kendall correlations and MSE across embeddings

## How to apply

First run Escort's connectivity evaluation (DCcheck) on each embedding and immediately exclude any embedding with newly disconnected clusters, since disconnected embeddings are classified as not recommended before GOF is computed. For surviving embeddings, compute the alpha-convex hull area of the embedded cells (Escort's alphahull CRAN dependency) and divide by the minimum enclosing circle area to obtain the coverage-area proportion (GOF). Convert this into a subscore via S=(V_cutoff−x)/V_cutoff with V_cutoff=0.53 for coverage area, then combine with the ambiguous-cell subscore (USHAPE, V_cutoff=0.029) multiplied by the same-group-level preservation weight (SimiRetain) through Escort's calcScore. Compare total Escort scores against accuracy anchors — Kendall rank correlation > 0.85 and MSE < 0.01 versus ground-truth pseudotime — to confirm good embeddings score above the >0 recommendation threshold. In the simulation benchmark (50 Scaffold-simulated pancreas datasets, 500 cells, 15,894 genes, 20% dynamic genes, 450 embeddings), this produced the positive association between Escort score and Kendall correlation (and inverse with MSE) shown in Figure 4B.

## Related tools

- **Escort** (R package providing calcScore and the GOF/coverage-area evaluation that converts the alpha-convex hull proportion into a benchmark score for embedding suitability) — https://github.com/xiaorudong/Escort
- **R** (Implementation environment; Escort is installed from GitHub and run as a package or local Shiny app) — https://github.com/xiaorudong/Escort
- **alphahull** (CRAN dependency supplying the alpha-convex hull computation underlying the coverage-area (GOF) metric)
- **Slingshot** (Used to infer pseudotime (with Monocle3) for computing ground-truth comparisons (Kendall correlation, MSE) against Escort scores)

## Examples

```
remotes::install_github("xiaorudong/Escort"); library(Escort); Escort::shinyEscort()
```

## Evaluation signals

- Coverage-area proportions are converted with V_cutoff=0.53, so embeddings with coverage proportion below 0.53 receive a positive S_density contribution and the total score stays within Escort's standardized range of −∞ to 2.
- Embeddings classified as good (Kendall rank correlation > 0.85 and MSE < 0.01 versus ground-truth pseudotime) receive total Escort scores above the >0 recommendation threshold.
- Across all 450 benchmark embeddings, Escort total scores correlate positively with Kendall correlation and inversely with MSE, reproducing Figure 4B.
- escort_score_correlation.csv is written and every embedding passed through DCcheck without newly disconnected clusters before GOF was computed.
- Output scores are reproducible via Escort's calcScore with W_same-group-level weighting from SimiRetain and V_cutoff=0.029 for the ambiguous-cell (USHAPE) component.

## Limitations

- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated.
- Escort only considers trajectory inference approaches that use two-dimensional representations; methods outside this restriction require further evaluation.
- Support for RNA velocity-derived dimension reduction methods is not yet implemented (future work).
- Quality control and normalization are key upstream steps that may significantly impact Escort's embedding evaluations, including the coverage-area metric.

## Evidence

- [readme] Escort is an R package designed to guide users through the trajectory inference process by offering goodness-of-fit evaluations for embeddings: "Escort is an R package designed to guide users through the trajectory inference process by offering goodness-of-fit evaluations for embeddings"
- [abstract] Escort uses a standardized benchmark scoring system ranging from negative infinity to two. Higher scores indicate better performance. Embeddings with a score greater than zero are recommended: "Escort uses a standardized benchmark scoring system ranging from negative infinity to two. Higher scores indicate better performance. Embeddings with a score greater than zero are recommended"
- [methods] any embeddings found to be disconnected are classified immediately as not recommended for trajectory inference: "any embeddings found to be disconnected are classified immediately as not recommended for trajectory inference"
- [other] compute the cell coverage area as the alpha-convex hull area proportion relative to the minimum enclosing circle (GOF): [evidence span withheld — anchor status: not_found]
