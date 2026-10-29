---
name: embedding-recommendation-scoring
description: Use when multiple 2D embeddings (from different dimension-reduction,
  feature-selection, or hyperparameter choices) exist for a scRNA-seq dataset and
  you need a data-driven classification of which are suitable for trajectory inference.
when_to_use_negative:
- The trajectory inference method does not use a two-dimensional representation —
  Escort only considers trajectory inference approaches using two-dimensional representations
- Embeddings were derived from RNA velocity-based dimension reduction — support is
  not yet implemented in Escort
- You expect Escort to guarantee at least one recommended embedding among evaluated
  choices — no recommendation is guaranteed
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Escort
  role: R package providing LD_DCClusterscheck, Similaritycheck, GOFeval, UshapeDetector,
    and calcScore for embedding evaluation and scoring
  repo: https://github.com/xiaorudong/Escort
- name: Slingshot
  role: Fits a rough trajectory per embedding via prepTraj to enable pseudotime-based
    ambiguous-cell detection
- name: Shiny
  role: Escort is also available as an integrated R/Shiny application for interactive
    scoring
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:de3979e31e789d78c561508bbebdf89adc8eeac35f2dd836844827649699dafa
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
  - Slingshot
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# embedding-recommendation-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Escort's three-step evaluation pipeline combines cell connectivity (DCcheck), similarity preservation (SimiRetain), goodness-of-fit/density (GOF), and ambiguous-cell (USHAPE) metrics into a single calcScore classification of candidate 2D embeddings as Recommended (score > 0) or Non-recommended (score <= 0) for trajectory inference.

## When to use

Use when multiple 2D embeddings (from different dimension-reduction, feature-selection, or hyperparameter choices) exist for a scRNA-seq dataset and you need a data-driven classification of which are suitable for trajectory inference.

## When NOT to use

- The trajectory inference method does not use a two-dimensional representation — Escort only considers trajectory inference approaches using two-dimensional representations
- Embeddings were derived from RNA velocity-based dimension reduction — support is not yet implemented in Escort
- You expect Escort to guarantee at least one recommended embedding among evaluated choices — no recommendation is guaranteed

## Inputs

- exampleData_linear dataset with normalized expression data and candidate embeddings from the Escort R package
- Normalized scRNA-seq expression matrix
- Set of candidate two-dimensional cell embeddings from varied processing choices (dimension reduction, highly variable gene selection)

## Outputs

- Per-embedding score table reporting coverage-area score, ambiguous-cell score, same-group-level weight, and total score (range negative infinity to two)
- Recommendation status per embedding (Recommended if total score > 0, Non-recommended otherwise)
- Immediate not-recommended flags for embeddings with newly disconnected clusters

## How to apply

First screen embeddings for connectivity with LD_DCClusterscheck (Jaccard-index-based cluster connectivity, 0.3 cutoff, minimum 10 cells per cluster in pairwise comparisons); embeddings with newly disconnected clusters are immediately flagged not recommended and excluded from scoring. For surviving embeddings, compute the 'same group level' similarity-preservation score with Similaritycheck — identify each cell's three closest Euclidean-distance neighbors and count it as high-similarity when at least two neighbors share its cluster — and quantify density with GOFeval as the ratio of the alpha-convex hull area to the minimum enclosing circle of cells. Fit a rough trajectory per embedding with prepTraj (Slingshot) and detect ambiguous cells with UshapeDetector using each cell's pseudotime standard deviation over its closest 10% nearest neighbors, thresholded at the skew-adjusted upper fence. Finally, run calcScore, which scales density and ambiguity components against fixed cutoffs (0.53 for cell coverage area, 0.029 for ambiguous-cell proportion), weights by the same-group-level percentage, and classifies embeddings: total score > 0 is Recommended, score <= 0 is Non-recommended.

## Related tools

- **Escort** (R package providing LD_DCClusterscheck, Similaritycheck, GOFeval, UshapeDetector, and calcScore for embedding evaluation and scoring) — https://github.com/xiaorudong/Escort
- **Slingshot** (Fits a rough trajectory per embedding via prepTraj to enable pseudotime-based ambiguous-cell detection)
- **Shiny** (Escort is also available as an integrated R/Shiny application for interactive scoring)

## Examples

```
if (!requireNamespace("remotes", quietly=TRUE)) install.packages("remotes"); remotes::install_github("xiaorudong/Escort"); library(Escort)
```

## Evaluation signals

- Output table contains each embedding's coverage-area score, ambiguous-cell score, same-group-level weight, and total score
- Total scores fall within the range negative infinity to two, and recommendation status exactly matches the score > 0 criterion
- Embeddings flagged as having newly disconnected clusters by LD_DCClusterscheck are excluded from the scored table
- Density component is the alpha-convex hull area ratio to minimum enclosing circle, and coverage-area scores are scaled against the 0.53 cutoff
- Ambiguous-cell proportions are scaled against the 0.029 cutoff

## Limitations

- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated
- Escort only considers trajectory inference approaches that use two-dimensional representations; methods outside this restriction require further evaluation
- Support for RNA velocity-derived dimension reduction methods is not yet implemented (future work)
- No software version specified for Escort or R; normalization and quality control steps may also significantly impact Escort's embedding evaluations

## Evidence

- [methods] score components: [evidence span withheld — anchor status: not_found]
- [discussion] preprocessing sensitivity: [evidence span withheld — anchor status: not_found]
