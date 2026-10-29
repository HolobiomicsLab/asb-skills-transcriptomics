---
name: kendall-correlation-and-mse-computation
description: Use when evaluating candidate 2D embeddings (e.g. MDS/UMAP/t-SNE outputs)
  against ground-truth pseudotime, or when validating that an embedding-quality score
  such as Escort's correlates with actual trajectory accuracy.
when_to_use_negative:
- No ground-truth pseudotime or cell timing is available (Kendall correlation and
  MSE require a true cell ordering; the paper preprocessed by eliminating cells with
  duplicated ground truth time)
- The trajectory method does not operate on a two-dimensional representation — Escort
  only considers trajectory inference approaches using 2D representations
- The embedding is already flagged as disconnected by Escort's DCcheck — it is immediately
  classified as not recommended and should not proceed to accuracy scoring
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Escort
  role: Provides DCcheck connectivity exclusion and the calcScore components (SimiRetain,
    GOF, USHAPE) whose correlation with Kendall/MSE is being validated
  repo: https://github.com/xiaorudong/Escort
- name: Slingshot
  role: Infers pseudotime on each embedding for comparison against ground truth (USHAPE
    ambiguity score also derives from Slingshot pseudotime standard deviations)
- name: Monocle3
  role: Alternative trajectory/pseudotime estimation method whose embeddings are scored
    by Kendall correlation and MSE
- name: R
  role: Implementation language for Escort and the Kendall/MSE computation workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:4bafbb58ccf43f3597ab846576a0787f1b1cbfc2da44e7b92f1b19c0bcfc872a
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# kendall-correlation-and-mse-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark-level accuracy assessment for 2D embeddings by computing Kendall rank correlation and mean squared error (MSE) between inferred and ground-truth pseudotime, used in the Escort paper to validate that the composite Escort score tracks embedding trajectory accuracy.

## When to use

Use when evaluating candidate 2D embeddings (e.g. MDS/UMAP/t-SNE outputs) against ground-truth pseudotime, or when validating that an embedding-quality score such as Escort's correlates with actual trajectory accuracy.

## When NOT to use

- No ground-truth pseudotime or cell timing is available (Kendall correlation and MSE require a true cell ordering; the paper preprocessed by eliminating cells with duplicated ground truth time)
- The trajectory method does not operate on a two-dimensional representation — Escort only considers trajectory inference approaches using 2D representations
- The embedding is already flagged as disconnected by Escort's DCcheck — it is immediately classified as not recommended and should not proceed to accuracy scoring

## Inputs

- 2D cell embeddings (e.g. MDS, UMAP, t-SNE projections of scRNA-seq data on 2000/4000/10000 HVG subsets)
- Ground-truth pseudotime or cell ordering per simulated dataset (e.g. from Scaffold simulations)
- Inferred pseudotime per embedding from Slingshot or Monocle3
- Escort DCcheck connectivity output for each embedding (to exclude disconnected embeddings before scoring)

## Outputs

- Per-embedding Kendall rank correlation between inferred and ground-truth pseudotime
- Per-embedding mean squared error (MSE) between inferred and ground-truth pseudotime
- Escort total scores via calcScore (range -infinity to 2; score > 0 recommended)
- Cross-embedding correlation of Escort scores vs. Kendall/MSE (Figure 4B equivalent), written as escort_score_correlation.csv

## How to apply

Generate the embedding set — e.g. the paper's simulation benchmark uses 50 Scaffold-simulated scRNA-seq datasets from a human pancreas reference (500 cells, 15,894 genes, 20% dynamic genes), each yielding nine embeddings via MDS, UMAP and t-SNE on subsets of 2000, 4000 or 10,000 highly variable genes (450 embeddings total). First run Escort's connectivity evaluation (DCcheck) and immediately exclude any embedding with newly disconnected clusters, since these are classified as not recommended before accuracy scoring. For each surviving embedding, infer pseudotime (Slingshot or Monocle3) and compute (1) the Kendall rank correlation and (2) the mean squared error between inferred and ground-truth pseudotime, after removing cells with duplicated ground truth time. Then compute the Escort score components (SimiRetain from each cell's three closest Euclidean neighbors, GOF as alpha-convex hull area over minimum enclosing circle, USHAPE ambiguous-cell proportion) and combine them via calcScore with S=(V_cutoff−x)/V_cutoff using V_cutoff=0.53 for coverage area and 0.029 for ambiguous cells, giving total score = (S_density + S_ambiguous) × W_same-group-level. Finally correlate Escort total scores with the Kendall and MSE values across all embeddings to confirm that higher Escort scores reflect higher trajectory accuracy.

## Related tools

- **Escort** (Provides DCcheck connectivity exclusion and the calcScore components (SimiRetain, GOF, USHAPE) whose correlation with Kendall/MSE is being validated) — https://github.com/xiaorudong/Escort
- **Slingshot** (Infers pseudotime on each embedding for comparison against ground truth (USHAPE ambiguity score also derives from Slingshot pseudotime standard deviations))
- **Monocle3** (Alternative trajectory/pseudotime estimation method whose embeddings are scored by Kendall correlation and MSE)
- **R** (Implementation language for Escort and the Kendall/MSE computation workflow)

## Examples

```
kendall_r <- cor(inferred_pseudotime, ground_truth_pseudotime, method='kendall'); mse <- mean((inferred_pseudotime - ground_truth_pseudotime)^2)
```

## Evaluation signals

- Embeddings classified as good (Kendall correlation > 0.85 and MSE < 0.01) should receive Escort total scores above the score > 0 recommendation threshold
- Across all 450 benchmark embeddings, Escort total scores should show a positive association with Kendall rank correlation and an inverse association with MSE (Figure 4B reproduction)
- Embeddings flagged disconnected by DCcheck must be excluded from the Kendall/MSE scoring set before correlating
- Escort scores must fall within the standardized benchmark range of negative infinity to two
- Output CSV (escort_score_correlation.csv) must contain paired Escort score vs. Kendall and Escort score vs. MSE values for every evaluated embedding

## Limitations

- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated
- Escort only considers trajectory inference approaches that use two-dimensional representations; methods outside this restriction require further evaluation
- Support for RNA velocity-derived dimension reduction methods is not yet implemented (future work)
- Upstream choices like normalization and quality control steps may significantly impact Escort's embedding evaluations

## Evidence
