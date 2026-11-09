---
name: knn-agreement-metric-evaluation
description: Use when judging whether a set of integrated cell labels is consistent
  with cell–cell variation within each individual input batch/modality matrix, e.g.
  comparing scMoMaT Leiden labels vs. original published labels on PBMC.
when_to_use_negative:
- You want to compare labels against cross-batch or integrated embeddings rather than
  per-matrix cell–cell variation — this metric is defined on each individual input
  data matrix
- Labels and cells are from a dataset other than one with matching batch count matrices
  and both label sets available (e.g. the PBMC task cards cover test_pbmc specifically)
tools:
- name: scMoMaT
  role: Source of the derived cell labels being evaluated; its repository provides
    the test_pbmc data, the Leiden clustering (scmomat.leiden_cluster, resolution
    0.9) and post-processing (scmomat.post_process, n_neighbors 30–50) used to construct
    kNN graphs
  repo: https://github.com/PeterZZQ/scMoMaT
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:2505c66d8e66f622f93047159880e5907167f72f9afa3e2b37f6724bb1a523c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PeterZZQ/scMoMaT
  tools:
  - scMoMaT
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# knn-agreement-metric-evaluation

## Summary

Evaluates the quality of cell labels by measuring kNN agreement — the fraction of each cell's k nearest neighbors (in a per-modality reduced embedding) that share its label. Used in scMoMaT work to show scMoMaT-derived Leiden labels agree better with per-matrix cell–cell variation than the original Mimitou et al. PBMC labels.

## When to use

Use when judging whether a set of integrated cell labels is consistent with cell–cell variation within each individual input batch/modality matrix, e.g. comparing scMoMaT Leiden labels vs. original published labels on PBMC.

## When NOT to use

- You want to compare labels against cross-batch or integrated embeddings rather than per-matrix cell–cell variation — this metric is defined on each individual input data matrix
- Labels and cells are from a dataset other than one with matching batch count matrices and both label sets available (e.g. the PBMC task cards cover test_pbmc specifically)

## Inputs

- PBMC batch count matrices from the test_pbmc dataset (scRNA, scATAC, protein modalities)
- scMoMaT-derived Leiden labels (resolution 0.9)
- Original published Mimitou et al. cell labels
- Modality-appropriate low-dimensional embeddings (PCA for scRNA/protein, LSI for scATAC)

## Outputs

- knn_agreement_pbmc.csv — per-modality and averaged kNN agreement scores for both label sets
- knn_agreement_pbmc.png — side-by-side plot of per-modality and averaged agreement

## How to apply

Load the batch count matrices and both label sets (e.g. scMoMaT Leiden labels at resolution 0.9 and the original published labels) from the test_pbmc data. Reduce each modality matrix to low dimension — PCA for scRNA and protein, LSI for scATAC — after preprocessing with utils.preprocess(). Build a kNN graph on each reduced representation with a neighborhood size of 30 (within the 30–50 range recommended for scMoMaT post-processing), then compute per cell the fraction of its k nearest neighbors sharing its label. Average the kNN agreement across cells within each modality, then across modalities and batches, for both label sets. Correctness is judged by a successful run and by the averaged agreement value of scMoMaT labels exceeding that of the original labels, matching the qualitative finding in Fig. 3e (lower plot).

## Related tools

- **scMoMaT** (Source of the derived cell labels being evaluated; its repository provides the test_pbmc data, the Leiden clustering (scmomat.leiden_cluster, resolution 0.9) and post-processing (scmomat.post_process, n_neighbors 30–50) used to construct kNN graphs) — https://github.com/PeterZZQ/scMoMaT

## Examples

```
s_pair_dist, knn_indices, knn_dists = scmomat.post_process(zs, n_neighbors=30, njobs=8, r=None); labels = scmomat.leiden_cluster(X=None, knn_indices=knn_indices, knn_dists=knn_dists, resolution=0.9)
```

## Evaluation signals

- The script runs without error end to end
- knn_agreement_pbmc.csv contains per-modality agreement values plus an averaged column for both label sets
- The averaged kNN agreement of scMoMaT labels exceeds that of the original Mimitou et al. labels (qualitative match to Fig. 3e, lower plot)
- Neighborhood size used is 30, within the 30–50 range recommended for scMoMaT post-processing
- Reductions respect modality: PCA for scRNA and protein, LSI for scATAC

## Limitations

- The finding is dataset-specific (human PBMC / ASAP-PBMC data) and modality-averaged; agreement may vary per modality or batch
- scMoMaT labels must be generated at the specific Leiden resolution (0.9) for the comparison to match the reported result
- The metric only reflects within-matrix neighborhood consistency, not cross-modality or cross-batch integration quality

## Evidence

- [readme] post_process n_neighbors range: "# post-processing
# neighborhood size (30~50)
n_neighbors = 30"
- [readme] post_process call: "s_pair_dist, knn_indices, knn_dists = scmomat.post_process(zs, n_neighbors, njobs = 8, r = r)"
- [readme] leiden clustering call: "labels_leiden = scmomat.leiden_cluster(X = None, knn_indices = knn_indices, knn_dists = knn_dists, resolution = resolution)"
- [other] kNN agreement finding: [evidence span withheld — anchor status: not_found]
- [other] reduction choice: [evidence span withheld — anchor status: not_found]
