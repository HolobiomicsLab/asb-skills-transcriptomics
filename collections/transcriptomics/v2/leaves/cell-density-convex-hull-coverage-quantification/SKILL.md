---
name: cell-density-convex-hull-coverage-quantification
description: 'Use when evaluating scRNA-seq embeddings for trajectory inference suitability
  and you need a quantitative cell-density metric: cells should be well-distributed
  (neither over-clumped nor over-dispersed) in the 2D embedding before fitting pseudotime.'
when_to_use_negative:
- The analysis does not rely on a two-dimensional embedding — Escort only considers
  trajectory inference approaches that use two-dimensional representations
- The embedding has newly disconnected cell clusters — such embeddings are classified
  immediately as not recommended and excluded from scoring before density evaluation
- The goal is to evaluate RNA velocity-derived dimension reduction methods — this
  support is not yet implemented in Escort
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Escort
  role: R package providing GOFeval (alpha-convex hull coverage-area computation)
    and calcScore (scaling against the 0.53 cutoff and classification)
  repo: https://github.com/xiaorudong/Escort
- name: R
  role: Implementation environment for the Escort package and its Shiny app
- name: Shiny
  role: Web application interface for running Escort embedding evaluations interactively
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:f57d6ae75c8458e6fb9efcea6215c7a415a970321bdf42daca6e4934e74bbe50
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
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Cell density convex hull coverage quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify cell density in a two-dimensional embedding as the ratio of the alpha-convex hull area (cell coverage area) to the minimum enclosing circle of the cells, using Escort's GOFeval. This coverage-area metric feeds Escort's calcScore, with 0.53 as the scaling cutoff.

## When to use

Use when evaluating scRNA-seq embeddings for trajectory inference suitability and you need a quantitative cell-density metric: cells should be well-distributed (neither over-clumped nor over-dispersed) in the 2D embedding before fitting pseudotime.

## When NOT to use

- The analysis does not rely on a two-dimensional embedding — Escort only considers trajectory inference approaches that use two-dimensional representations
- The embedding has newly disconnected cell clusters — such embeddings are classified immediately as not recommended and excluded from scoring before density evaluation
- The goal is to evaluate RNA velocity-derived dimension reduction methods — this support is not yet implemented in Escort

## Inputs

- Candidate two-dimensional cell embeddings (e.g., PCA, t-SNE, UMAP, diffusion maps) from scRNA-seq data, with cluster labels
- Normalized expression data and example datasets such as exampleData_linear from the Escort R package

## Outputs

- Per-embedding cell coverage area (alpha-convex hull area / minimum enclosing circle ratio)
- Scaled density (coverage-area) score component in the Escort score table
- Escort total score (range negative infinity to two) with Recommended (> 0) or Non-recommended (<= 0) status

## How to apply

Run Escort's GOFeval on each candidate two-dimensional embedding to compute the alpha-convex hull area — termed the cell coverage area — and divide it by the area of the minimum enclosing circle of the embedded cells; this ratio captures how densely cells fill the embedding space. Escort scales this density component against the cutoff of 0.53 for cell coverage area within calcScore. The scaled density score is then weighted by the 'same group level' similarity-preservation percentage (from Similaritycheck, where a cell counts as high-similarity when at least two of its three closest Euclidean-distance neighbors share its cluster) and combined with the ambiguous-cell score and connectivity evaluation. Embeddings with a total score greater than zero are classified as Recommended for trajectory inference; embeddings newly found to have disconnected clusters (Jaccard-index connectivity with 0.3 cutoff, minimum 10 cells per cluster) are excluded from scoring entirely.

## Related tools

- **Escort** (R package providing GOFeval (alpha-convex hull coverage-area computation) and calcScore (scaling against the 0.53 cutoff and classification)) — https://github.com/xiaorudong/Escort
- **R** (Implementation environment for the Escort package and its Shiny app)
- **Shiny** (Web application interface for running Escort embedding evaluations interactively)

## Examples

```
library(Escort); gof <- GOFeval(embedding = exampleData_linear$embeddings[[1]], clusters = exampleData_linear$clusters); score <- calcScore(gof = gof)
```

## Evaluation signals

- The coverage-area ratio (alpha-convex hull area / minimum enclosing circle) is compared against the 0.53 cutoff during calcScore scaling
- The output table reports each embedding's coverage-area score, ambiguous-cell score, same-group-level weight, and total score
- Total scores fall in the range negative infinity to two, with scores > 0 classified as Recommended
- Embeddings flagged as having newly disconnected clusters (Jaccard-based connectivity, 0.3 cutoff, ≥ 10 cells per cluster) are excluded from the score table
- Recommendation status in the output matches the score > 0 criterion for every evaluated embedding

## Limitations

- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated
- Escort only considers trajectory inference approaches that use two-dimensional representations
- Support for RNA velocity-derived dimension reduction methods is not yet implemented (future work)
- Quality control steps and normalization may significantly impact Escort's embedding evaluations, and no software version is specified

## Evidence

- [methods] GOF density metric: [evidence span withheld — anchor status: not_found]
- [methods] Scoring cutoff and weighting: [evidence span withheld — anchor status: not_found]
- [abstract] Score classification benchmark: "Escort uses a standardized benchmark scoring system ranging from negative infinity to two. Higher scores indicate better performance. Embeddings with a score greater than zero are recommended"
- [methods] Disconnection exclusion: "any embeddings found to be disconnected are classified immediately as not recommended for trajectory inference"
- [discussion] Two-dimensional limitation: "Presently, Escort only considers trajectory inference approaches that use two-dimensional representations"
