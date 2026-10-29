---
name: same-group-level-similarity-scoring
description: Use when scoring candidate 2D embeddings (e.g., PCA, t-SNE, UMAP) for
  trajectory-inference suitability and you need to measure whether local neighbor
  structure in the embedding respects pre-computed cell cluster/group assignments.
when_to_use_negative:
- The embedding has already been flagged as newly disconnected by the Jaccard-based
  connectivity check (LD_DCClusterscheck, 0.3 cutoff, min 10 cells per cluster) —
  it is classified not-recommended and excluded from scoring
- The analysis uses a representation that is not two-dimensional — Escort only considers
  trajectory inference approaches using two-dimensional representations
- The goal is method benchmarking rather than evaluating a specific dataset's embeddings
  — dynguidelines-style method selection is a different task
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Escort
  role: R package providing Similaritycheck and calcScore that implement the same-group-level
    similarity scoring and its use as a score weight
  repo: https://github.com/xiaorudong/Escort
- name: R
  role: Runtime environment for the Escort R package used to run the similarity evaluation
- name: Slingshot
  role: Used in the companion prepTraj step to fit rough trajectories per embedding
    that the overall Escort scoring pipeline (which Similaritycheck feeds into) evaluates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:a9b02b232e6eb33131082bfffa9e2b3109f863ba47cfe6ec99d0804a61f1875d
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
  - Slingshot
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# same-group-level-similarity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Escort's similarity-preservation evaluation (Similaritycheck) that quantifies how well a 2D embedding preserves each cell's cluster-level relationships by checking whether a cell's nearest neighbors in the embedding share its cluster label; the resulting 'same group level' percentage weights the total Escort score.

## When to use

Use when scoring candidate 2D embeddings (e.g., PCA, t-SNE, UMAP) for trajectory-inference suitability and you need to measure whether local neighbor structure in the embedding respects pre-computed cell cluster/group assignments.

## When NOT to use

- The embedding has already been flagged as newly disconnected by the Jaccard-based connectivity check (LD_DCClusterscheck, 0.3 cutoff, min 10 cells per cluster) — it is classified not-recommended and excluded from scoring
- The analysis uses a representation that is not two-dimensional — Escort only considers trajectory inference approaches using two-dimensional representations
- The goal is method benchmarking rather than evaluating a specific dataset's embeddings — dynguidelines-style method selection is a different task

## Inputs

- A candidate two-dimensional cell embedding (coordinate matrix) from Escort's exampleData_linear or user-supplied dimension-reduction output
- Per-cell cluster/group labels for the same cells
- Normalized scRNA-seq expression data loaded via the Escort R package

## Outputs

- A 'same group level' similarity-preservation percentage for the embedding
- A column in the Escort score table reporting the same-group-level weight used by calcScore toward the total score (range negative infinity to two)

## How to apply

For each candidate embedding, run Escort's Similaritycheck: identify each cell's three closest Euclidean-distance neighbors in the 2D embedding, and count a cell as 'high-similarity' when at least two of those three neighbors share its cluster label. Aggregate to the percentage of high-similarity cells, which becomes the 'same group level' similarity-preservation score. This percentage is then used by calcScore as the weight applied to the density (coverage-area vs 0.53 cutoff) and ambiguous-cell (vs 0.029 cutoff) components, so embeddings where cluster relationships are poorly preserved are penalized before classifying total score > 0 as Recommended. Evaluate embeddings only after the connectivity screen (LD_DCClusterscheck, Jaccard 0.3 cutoff, minimum 10 cells per cluster) has excluded newly disconnected embeddings, since those are flagged not-recommended without scoring.

## Related tools

- **Escort** (R package providing Similaritycheck and calcScore that implement the same-group-level similarity scoring and its use as a score weight) — https://github.com/xiaorudong/Escort
- **R** (Runtime environment for the Escort R package used to run the similarity evaluation)
- **Slingshot** (Used in the companion prepTraj step to fit rough trajectories per embedding that the overall Escort scoring pipeline (which Similaritycheck feeds into) evaluates)

## Examples

```
library(Escort); same_group_pct <- Escort::Similaritycheck(embedding_coords, cluster_labels)
```

## Evaluation signals

- The reported same-group-level percentage equals the fraction of cells with at least two of their three nearest Euclidean neighbors in the same cluster
- The score table contains the same-group-level weight column alongside coverage-area score and ambiguous-cell score columns
- Total score stays within the documented range of negative infinity to two, and recommendation status matches total score > 0
- Embeddings previously flagged disconnected by LD_DCClusterscheck are absent from the scored output

## Limitations

- Quality control steps may also significantly impact Escort's embedding evaluations, altering the similarity score
- Escort only considers trajectory inference approaches that use two-dimensional representations
- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated

## Evidence

- [methods] Similaritycheck three-neighbor rule: [evidence span withheld — anchor status: not_found]
