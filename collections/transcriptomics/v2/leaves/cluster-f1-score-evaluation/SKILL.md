---
name: cluster-f1-score-evaluation
description: Use when clusters (possibly from multi-stage scCAN re-clustering) must
  be quantitatively validated against annotated ground-truth labels for one target
  cell type, especially rare populations.
when_to_use_negative:
- No ground-truth cell annotations are available for the dataset, so an F1 against
  a reference type cannot be computed (scCAN itself is unsupervised and can still
  cluster, only the scoring step needs labels).
- The evaluation goal is overall clustering quality across all types (ARI, estimated
  number of clusters via the eigengap method) rather than recovery of one specific
  annotated rare cell type.
- Ambiguous cells labeled 'zothers', 'dropped', 'not applicable', or 'Not available'
  have not been removed first, which the article recommends for label-bearing datasets
  before comparing to annotations.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCAN
  role: Produces the cluster assignments being scored; also supplies adjustedRandIndex
    for cross-checking cluster-vs-label agreement.
  repo: https://cran.r-project.org/package=scCAN
- name: SHARP
  role: Comparison method whose cluster assignments can be scored with the same F1
    protocol.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:9adc23cc77fbdd30766a3ce85ce7a6e5fc0513f5f8cef01a61f8abfdd1de8c25
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SHARP
  - scCAN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cluster-f1-score-evaluation

## Summary

Score how completely a clustering recovers a specific annotated cell type (e.g., the rare tRBC population in the Zilionis lung dataset) by computing a one-vs-rest F1 score of cluster assignments against known cell labels.

## When to use

Use when clusters (possibly from multi-stage scCAN re-clustering) must be quantitatively validated against annotated ground-truth labels for one target cell type, especially rare populations.

## When NOT to use

- No ground-truth cell annotations are available for the dataset, so an F1 against a reference type cannot be computed (scCAN itself is unsupervised and can still cluster, only the scoring step needs labels).
- The evaluation goal is overall clustering quality across all types (ARI, estimated number of clusters via the eigengap method) rather than recovery of one specific annotated rare cell type.
- Ambiguous cells labeled 'zothers', 'dropped', 'not applicable', or 'Not available' have not been removed first, which the article recommends for label-bearing datasets before comparing to annotations.

## Inputs

- Cluster assignment vector for all cells (e.g., stage-two sub-cluster labels like 2_1, 2_2) from scCAN output (result$cluster)
- Ground-truth annotated cell-type labels aligned to the same 34,558 cells (e.g., tRBC vs. all other types in the Zilionis GSE127465 dataset)
- Target cell-type name to treat as the positive class (e.g., tRBC)

## Outputs

- Per-target-type F1 score of cluster separation (expected value 1.0 within numerical tolerance for the two-stage tRBC task)
- Optional auxiliary agreement metrics (e.g., ARI via scCAN::adjustedRandIndex)
- Pass/fail validation verdict against the reported reference score

## How to apply

Take the final cluster assignments (e.g., stage-two sub-clusters such as 2_1 and 2_2 from re-clustering a stage-one scCAN cluster) and the vector of annotated cell-type labels for the same 34,558 Zilionis cells. Compute an F1 score treating the target type (tRBC) as the positive class and everything else as negative, judging whether clusters completely separate the rare cells. In the source task, the two-stage strategy with default samp.size=5,000 produced stage-two sub-clusters 2_1 and 2_2 that fully separated tRBC cells, giving F1 = 1; a single-stage run at the same setting is the natural baseline comparison. Because scCAN's big-data path randomly selects a 5,000-cell training set and k-NN classifies the rest, control the random seed (e.g., r.seed = 1 as in the README example) before scoring. Optionally cross-check global agreement with scCAN::adjustedRandIndex, which the package ships for exactly this purpose.

## Related tools

- **scCAN** (Produces the cluster assignments being scored; also supplies adjustedRandIndex for cross-checking cluster-vs-label agreement.) — https://cran.r-project.org/package=scCAN
- **SHARP** (Comparison method whose cluster assignments can be scored with the same F1 protocol.)

## Examples

```
library(scCAN); result <- scCAN(data, r.seed = 1); cluster <- result$cluster; ari <- round(scCAN::adjustedRandIndex(cluster, label), 2)
```

## Evaluation signals

- The computed tRBC-separation F1 equals the reported value of 1 for the two-stage strategy (F1 = 1 within numerical tolerance).
- Stage-two clustering splits the target stage-one cluster (cluster 2) into sub-clusters 2_1 and 2_2, and tRBC cells appear exclusively in one sub-cluster (no mixing).
- F1 is computed one-vs-rest against the annotated tRBC labels only, using the same 34,558-cell alignment between clusters and labels.
- Re-running with a fixed random seed (e.g., r.seed = 1) reproduces the score, since the 5,000-cell training split is random.
- Cross-check that global agreement is sane via scCAN::adjustedRandIndex(cluster, label) before trusting the single-type F1.

## Limitations

- F1 requires ground-truth annotations; datasets with ambiguous labels ('zothers', 'dropped', 'not applicable', 'Not available') need those cells removed before scoring, as done for Chen, Wang, Segerstolpe, Guo, Kanton, and Miller datasets.
- The score is specific to one target cell type and does not certify global clustering accuracy or that scCAN estimated the correct number of cell types.
- Under the default samp.size=5,000 big-data path (Module 3), results depend on the random training-set selection and seed, so the F1 is not deterministic unless r.seed is fixed.

## Evidence

- [other] In the two-stage clustering with the default samp.size=5,000, stage two split cluster 2 into sub-clusters 2_1 and 2_2, completely separating the rare tRBC cells from the rest with an F1 score of 1.: [evidence span withheld — anchor status: not_found]
- [results] we removed cells with the label “zothers” from Chen,“dropped” from Wang, “not applicable” from Segerstolpe, and “Not available” from Guo, Kanton, and Miller.: "we removed cells with the label “zothers” from Chen,“dropped” from Wang, “not applicable” from Segerstolpe, and “Not available” from Guo, Kanton, and Miller."
- [methods] we split the cells into two sets: a training set of 5000 randomly selected cells and a testing set that consists of the remaining cells: "we split the cells into two sets: a training set of 5000 randomly selected cells and a testing set that consists of the remaining cells"
- [readme] ari <- round(scCAN::adjustedRandIndex(cluster,label), 2): "ari <- round(scCAN::adjustedRandIndex(cluster,label), 2)"
