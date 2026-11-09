---
name: single-cell-neighbor-preservation-evaluation
description: Use when you have one or more 2D embeddings (e.g. UMAP, t-SNE, MDS on
  subsets of highly variable genes) of scRNA-seq data and must decide which embedding
  is suitable for pseudotime/trajectory inference before fitting Slingshot or Monocle3.
when_to_use_negative:
- Your trajectory inference approach does not use two-dimensional representations
  — Escort only considers trajectory inference approaches using two-dimensional representations,
  and RNA velocity-derived dimension reduction is not yet implemented
- The embedding has already been judged directly by ground-truth pseudotime (Kendall
  rank correlation / MSE against known labels); Escort's scores are surrogate data-driven
  assessments, not a substitute when ground truth is available
- The dataset is not scRNA-seq or lacks a plausible trajectory signal (disconnected
  or homogeneous cells), in which case Escort's DCcheck will simply classify embeddings
  as not recommended rather than providing guidance
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Escort
  role: R package implementing the connectivity evaluation (DCcheck), SimiRetain,
    GOF and USHAPE metrics and the calcScore composite scoring
  repo: https://github.com/xiaorudong/Escort
- name: R
  role: Implementation language of the Escort R package and R/Shiny application used
    to run the evaluations
- name: Shiny
  role: Interactive interface for running Escort's embedding evaluations locally via
    shinyEscort()
- name: Slingshot
  role: Provides pseudotime estimates whose standard deviations over the closest 10%
    of neighbors feed the USHAPE ambiguous-cell metric
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:21904b8150c7c159c386e714e1b8f5bb46a6049b1bef7e652a0059aaad8a05fc
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
  - Shiny
  - Slingshot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-neighbor-preservation-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An Escort-based evaluation that scores scRNA-seq 2D embeddings for trajectory-inference suitability using neighbor-preservation and trajectory-specific metrics (DCcheck connectivity, SimiRetain same-group preservation, GOF coverage, USHAPE ambiguity), combined into a standardized score from negative infinity to two where scores above zero are recommended.

## When to use

Use when you have one or more 2D embeddings (e.g. UMAP, t-SNE, MDS on subsets of highly variable genes) of scRNA-seq data and must decide which embedding is suitable for pseudotime/trajectory inference before fitting Slingshot or Monocle3.

## When NOT to use

- Your trajectory inference approach does not use two-dimensional representations — Escort only considers trajectory inference approaches using two-dimensional representations, and RNA velocity-derived dimension reduction is not yet implemented
- The embedding has already been judged directly by ground-truth pseudotime (Kendall rank correlation / MSE against known labels); Escort's scores are surrogate data-driven assessments, not a substitute when ground truth is available
- The dataset is not scRNA-seq or lacks a plausible trajectory signal (disconnected or homogeneous cells), in which case Escort's DCcheck will simply classify embeddings as not recommended rather than providing guidance

## Inputs

- 2D low-dimensional embeddings of an scRNA-seq dataset (e.g. nine embeddings per dataset from MDS, UMAP and t-SNE on 2000, 4000 or 10,000 highly variable genes)
- Preprocessed single-cell dataset (e.g. SingleCellExperiment, LogNormalize-normalized via Seurat NormalizeData, with low-quality cells and genes with <3 total counts removed)
- Slingshot pseudotime estimates for computing USHAPE ambiguity

## Outputs

- Per-embedding Escort total score (range negative infinity to two) with recommended/not-recommended classification (score > 0)
- Per-embedding component scores: S_density (coverage area), S_ambiguous (USHAPE), SimiRetain same-group preservation, DCcheck connectivity result
- Score summary table (e.g. escort_score_correlation.csv when scores are correlated with external accuracy measures)

## How to apply

For each embedding, first run Escort's connectivity evaluation (DCcheck) and immediately exclude any embedding showing newly disconnected clusters, since disconnected embeddings are classified as not recommended. For remaining embeddings, compute the same-group-level preservation score (SimiRetain) from each cell's three closest Euclidean neighbors, the cell coverage area as the alpha-convex hull area proportion relative to the minimum enclosing circle (GOF), and the proportion of ambiguous cells from Slingshot pseudotime standard deviations over the closest 10% of neighbors exceeding the skew-adjusted upper fence (USHAPE). Combine components with Escort's calcScore using S=(V_cutoff−x)/V_cutoff with V_cutoff=0.53 for coverage area and 0.029 for ambiguous cells, giving total score = (S_density + S_ambiguous) × W_same-group-level. Rationale: even universal processing steps like feature selection and dimension reduction make trajectory method performance highly dataset-specific, so data-driven scoring replaces arbitrary embedding choice; embeddings scoring above zero (range negative infinity to two) are recommended for trajectory construction.

## Related tools

- **Escort** (R package implementing the connectivity evaluation (DCcheck), SimiRetain, GOF and USHAPE metrics and the calcScore composite scoring) — https://github.com/xiaorudong/Escort
- **R** (Implementation language of the Escort R package and R/Shiny application used to run the evaluations)
- **Shiny** (Interactive interface for running Escort's embedding evaluations locally via shinyEscort())
- **Slingshot** (Provides pseudotime estimates whose standard deviations over the closest 10% of neighbors feed the USHAPE ambiguous-cell metric)

## Examples

```
if (!requireNamespace("remotes", quietly=TRUE)) install.packages("remotes"); remotes::install_github("xiaorudong/Escort"); library(Escort); shinyEscort()
```

## Evaluation signals

- Any embedding found disconnected by DCcheck must be classified immediately as not recommended and excluded before scoring
- Embeddings independently judged good (Kendall correlation > 0.85 and MSE < 0.01 against ground-truth pseudotime) should receive total scores above the recommended threshold of 0
- Total scores must fall in Escort's standardized range from negative infinity to two
- Across a benchmark of embeddings (e.g. 450 in the simulation study), Escort scores should correlate positively with Kendall rank correlation and inversely with pseudotime MSE
- S=(V_cutoff−x)/V_cutoff must use V_cutoff=0.53 for coverage area and 0.029 for ambiguous cells, with total score = (S_density + S_ambiguous) × W_same-group-level

## Limitations

- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated
- Presently Escort only considers trajectory inference approaches that use two-dimensional representations; methods outside this restriction require further evaluation
- Support for RNA velocity-derived dimension reduction methods is not yet implemented (noted as future work)
- Quality control steps and normalization may also significantly impact Escort's embedding evaluations, so upstream preprocessing choices should be considered

## Evidence
