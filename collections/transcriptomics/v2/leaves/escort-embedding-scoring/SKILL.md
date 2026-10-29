---
name: escort-embedding-scoring
description: Decision support for choosing among multiple 2-D embeddings (e.g., MDS,
  UMAP or t-SNE computed on 2000, 4000 or 10000 highly variable genes) before trajectory
  inference, when you need a data-driven recommended/not-recommended verdict per embedding.
when_to_use_negative:
- The trajectory inference approach does not rely on a two-dimensional representation
  — Escort 'only considers trajectory inference approaches that use two-dimensional
  representations'
- The embedding is derived from RNA velocity-based dimension reduction — this is explicitly
  not yet implemented and left as future work
- Every candidate embedding fails DCcheck (newly disconnected clusters) — no scoring
  is possible, and Escort is not guaranteed to identify a recommended embedding among
  the choices evaluated
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Escort
  role: R package providing DCcheck connectivity evaluation, SimiRetain, GOF and USHAPE
    metrics, and calcScore for combining component scores into the total embedding
    score
  repo: https://github.com/xiaorudong/Escort
- name: Slingshot
  role: Provides pseudotime estimates whose standard deviations over the closest 10%
    of neighbors feed the USHAPE ambiguous-cell metric
- name: Monocle3
  role: Alternative trajectory/pseudotime estimation tool whose accuracy is assessed
    against Escort scores in the benchmark
- name: R
  role: Implementation environment; Escort is run as an R package or R/Shiny application
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:90e55b9987bb6e7a6c7991fe45b23151df5daa42f2559540999f53e0b4c3be58
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# escort-embedding-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for scoring candidate 2-D embeddings of a scRNA-seq dataset with Escort's trajectory-specific metrics (DCcheck connectivity, SimiRetain, GOF coverage area, USHAPE ambiguity) and verifying that the composite score tracks trajectory accuracy (Kendall correlation, MSE) as shown in Figure 4B of Dong et al. 2024.

## When to use

Decision support for choosing among multiple 2-D embeddings (e.g., MDS, UMAP or t-SNE computed on 2000, 4000 or 10000 highly variable genes) before trajectory inference, when you need a data-driven recommended/not-recommended verdict per embedding.

## When NOT to use

- The trajectory inference approach does not rely on a two-dimensional representation — Escort 'only considers trajectory inference approaches that use two-dimensional representations'
- The embedding is derived from RNA velocity-based dimension reduction — this is explicitly not yet implemented and left as future work
- Every candidate embedding fails DCcheck (newly disconnected clusters) — no scoring is possible, and Escort is not guaranteed to identify a recommended embedding among the choices evaluated

## Inputs

- Two-dimensional cell embeddings (e.g., MDS, UMAP, t-SNE outputs) derived from subsets of highly variable genes
- Preprocessed scRNA-seq count matrix or SingleCellExperiment (e.g., human pancreas reference, 500 cells × 15,894 genes after LogNormalize normalization)
- Cluster/cell-group labels for connectivity and same-group preservation checks
- Ground-truth pseudotime values (for accuracy validation; cells with duplicated ground truth time eliminated)

## Outputs

- Escort total score per embedding on the negative-infinity-to-two scale with recommended (> 0) / not-recommended classification
- Per-embedding metric values: DCcheck connectivity, SimiRetain, GOF coverage-area proportion, USHAPE ambiguous-cell proportion
- escort_score_correlation.csv relating Escort scores to Kendall rank correlation and MSE of inferred vs ground-truth pseudotime

## How to apply

Run Escort's connectivity evaluation (DCcheck) on every 2-D embedding and immediately exclude any embedding with newly disconnected clusters, since such embeddings are classified as not recommended before further scoring. For surviving embeddings, compute SimiRetain from each cell's three closest Euclidean neighbors, the GOF coverage area as the alpha-convex hull area proportion relative to the minimum enclosing circle, and the USHAPE proportion of ambiguous cells from Slingshot pseudotime standard deviations over the closest 10% of neighbors exceeding the skew-adjusted upper fence. Combine components with Escort's calcScore using S=(V_cutoff−x)/V_cutoff with V_cutoff=0.53 for coverage area and 0.029 for ambiguous cells, giving total score = (S_density + S_ambiguous) × W_same-group-level; scores range from negative infinity to two and embeddings with a score greater than zero are recommended. To validate that scores reflect accuracy, compute Kendall rank correlation and mean squared error between inferred and ground-truth pseudotime and correlate them with Escort total scores across embeddings (in the paper, 50 Scaffold-simulated pancreas datasets × 9 embeddings = 450 embeddings), writing escort_score_correlation.csv. The expected pattern is a positive association with Kendall correlation and an inverse association with MSE, with good embeddings (Kendall > 0.85, MSE < 0.01) scoring above the > 0 threshold.

## Related tools

- **Escort** (R package providing DCcheck connectivity evaluation, SimiRetain, GOF and USHAPE metrics, and calcScore for combining component scores into the total embedding score) — https://github.com/xiaorudong/Escort
- **Slingshot** (Provides pseudotime estimates whose standard deviations over the closest 10% of neighbors feed the USHAPE ambiguous-cell metric)
- **Monocle3** (Alternative trajectory/pseudotime estimation tool whose accuracy is assessed against Escort scores in the benchmark)
- **R** (Implementation environment; Escort is run as an R package or R/Shiny application)

## Examples

```
remotes::install_github("xiaorudong/Escort"); library(Escort); Escort::shinyEscort()
```

## Evaluation signals

- Embeddings flagged by DCcheck as having newly disconnected clusters are excluded before composite scoring (score should be NA/not-recommended)
- Embeddings classified as good (Kendall correlation > 0.85 and MSE < 0.01) receive total scores above the > 0 recommendation threshold
- All Escort total scores fall on the standardized scale from negative infinity to two
- Across embeddings, Escort score shows a positive association with Kendall correlation and an inverse association with MSE (reproduces Figure 4B)
- Scored set contains one entry per surviving embedding (e.g., 450 embeddings = 50 simulated datasets × 9 embeddings) written to escort_score_correlation.csv

## Limitations

- Escort only considers trajectory inference approaches that use two-dimensional representations; methods outside this restriction require further evaluation
- Support for RNA velocity-derived dimension reduction methods is not yet implemented (future work)
- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated
- Quality control steps and normalization may also significantly impact Escort's embedding evaluations

## Evidence
