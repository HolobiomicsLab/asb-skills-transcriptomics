---
name: rare-cluster-threshold-classification
description: Use when after cluster decomposition and merging in scCAD, you need to
  score each M-cluster's rarity from isolation-forest anomaly scores and decide which
  sub-clusters count as rare cell types.
when_to_use_negative:
- You need anomaly scores on the full global expression profile rather than cluster-specific
  top-20 differential gene sets — scCAD's IS is defined relative to each M-cluster's
  candidate gene set
- Clusters have not yet been decomposed and merged (rare_h default 0.01, merge_h default
  0.3); the independence score presupposes M-cluster assignments
- You want a quantitative rarity ranking rather than a binary rare/common classification
  at a fixed threshold
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCAD
  role: End-to-end pipeline whose final step computes sub-cluster independence scores
    and flags rare sub-clusters at overlap_h = 0.7
  repo: https://github.com/xuyp-csu/scCAD
- name: scikit-learn
  role: Provides the isolation forest model used to compute per-cell anomaly scores
    on each cluster's top-20 differential genes (tested version 1.0.2)
- name: Scanpy
  role: Preprocessing (log-normalization, PCA to top 40 PCs, KNN k=15, Louvain) that
    produces the clusters scored in this step (tested version 1.9.1)
- name: numpy
  role: Numeric computations of median differences, anomaly score formula, and the
    IS overlap |T_Ni ∩ C_i| / N_i (tested version 1.21.5)
- name: pandas
  role: Writes per-cluster independence scores and rare-cluster DEG lists (e.g., independence_scores.csv)
    (tested version 1.3.5)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:ebe898207c175d615cfca94c4436f2fbbe85f19974c2769f061898f7a360f1bb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  repo_url: https://github.com/xuyp-csu/scCAD
  tools:
  - Scanpy
  - numpy
  - pandas
  - scCAD
  - scikit-learn
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# rare-cluster-threshold-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies scCAD's M-clusters as rare or common by feeding each cluster's top-20 differential genes into an isolation forest, converting anomaly scores into an independence score (IS), and flagging clusters with IS above the default threshold I = 0.7.

## When to use

Use when, after cluster decomposition and merging in scCAD, you need to score each M-cluster's rarity from isolation-forest anomaly scores and decide which sub-clusters count as rare cell types.

## When NOT to use

- You need anomaly scores on the full global expression profile rather than cluster-specific top-20 differential gene sets — scCAD's IS is defined relative to each M-cluster's candidate gene set
- Clusters have not yet been decomposed and merged (rare_h default 0.01, merge_h default 0.3); the independence score presupposes M-cluster assignments
- You want a quantitative rarity ranking rather than a binary rare/common classification at a fixed threshold

## Inputs

- Preprocessed cells x genes expression matrix (e.g., loaded from 1%Jurkat.h5 via h5py, with cell/gene names)
- M-cluster assignments for each cell (from scCAD's iterative decomposition and merge, merge_h default 0.3)
- Per-cluster candidate DE gene lists (top 20 genes by median difference) restricted expression subsets

## Outputs

- Per-sub-cluster independence scores (np.array of n sub-clusters, each in [0, 1])
- Rare sub-cluster cell assignments (list of cell names per rare sub-cluster)
- Candidate differentially expressed gene lists used for each rare sub-cluster
- Results table/file such as independence_scores.csv with scores, rare-cluster flags, and DEG lists

## How to apply

For each M-cluster i, compute per-gene median differences d_k = |median(X_Ci^k) - median(X_Ci'^k)| between cells inside and outside the cluster, and take the top 20 genes with the largest differences as the candidate gene set S_i. Feed the expression matrix restricted to S_i into a scikit-learn isolation forest and compute anomaly scores AS_j^Si = 2^(-E(h(j))/c(n)), where c(n) = 2H(n-1) - 2(n-1)/n with H(n-1) ≈ ln(n-1) + 0.5772156649. Rank all cells by anomaly score, take the top N_i cells as T_Ni, and compute the independence score IS_i = |T_Ni ∩ C_i| / N_i, i.e., the overlap between the most abnormal cells and cells actually inside the cluster. Flag a cluster as rare when IS_i exceeds the threshold I, defaulting to I = 0.7 across all datasets; this maps to the README's overlap_h parameter (default 0.7). Record each rare cluster's independence score and candidate DEG list (e.g., to independence_scores.csv or via the score and degs_list return values).

## Related tools

- **scCAD** (End-to-end pipeline whose final step computes sub-cluster independence scores and flags rare sub-clusters at overlap_h = 0.7) — https://github.com/xuyp-csu/scCAD
- **scikit-learn** (Provides the isolation forest model used to compute per-cell anomaly scores on each cluster's top-20 differential genes (tested version 1.0.2))
- **Scanpy** (Preprocessing (log-normalization, PCA to top 40 PCs, KNN k=15, Louvain) that produces the clusters scored in this step (tested version 1.9.1))
- **numpy** (Numeric computations of median differences, anomaly score formula, and the IS overlap |T_Ni ∩ C_i| / N_i (tested version 1.21.5))
- **pandas** (Writes per-cluster independence scores and rare-cluster DEG lists (e.g., independence_scores.csv) (tested version 1.3.5))

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- All computed independence scores fall within [0, 1]
- Flagged rare clusters are exactly those with IS_i > 0.7 (the paper's default threshold I)
- Each rare cluster has a candidate DEG list of at most 20 genes (top 20 by median difference)
- The number of scores returned equals the number of sub-clusters (score is np.array[n sub-clusters])
- For the packaged 1%Jurkat.h5 demo (1540 293T cells + 16 Jurkat cells), the rare sub-cluster cells are overwhelmingly Jurkat-labeled when labels are cross-tabulated

## Limitations

- The default I = 0.7 was applied across all benchmark datasets without dataset-specific tuning, so it may misclassify borderline clusters
- IS depends on the quality of the candidate gene list (top 20 median-difference genes); noisy DEG selection propagates into anomaly scores
- Threshold classification is binary — clusters just below I are not reported as rare even if biologically relevant

## Evidence

- [discussion] top-20 differential genes feed isolation forest: [evidence span withheld — anchor status: not_found]
- [discussion] independence score threshold flags rare clusters: [evidence span withheld — anchor status: not_found]
- [discussion] threshold applied uniformly: [evidence span withheld — anchor status: not_found]
