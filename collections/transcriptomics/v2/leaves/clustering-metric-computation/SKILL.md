---
name: clustering-metric-computation
description: Use when comparing cell-cluster assignments produced by two or more scRNA-seq
  clustering methods against ground-truth cell-type labels and need ARI, NMI and V-measure
  scores to rank methods.
when_to_use_negative:
- When no reference cell-type labels exist — ARI, NMI and V-measure are external metrics
  requiring ground truth
- When the goal is marker gene identification rather than clustering performance benchmarking
  (Miscell's attribution method is a separate skill)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Miscell
  role: Self-supervised model producing latent features whose DBSCAN/Leiden cluster
    assignments are one input to the metric computation
  repo: https://github.com/lixiangchun/Miscell
- name: scikit-learn
  role: Provides sklearn.cluster.dbscan and the adjusted rand index, normalized mutual
    information, and V-measure implementations used to score assignments against reference
    labels
- name: Scanpy
  role: 'Baseline method: neighborhood graph (sc.pp.neighbors, n_neighbors=10) plus
    Leiden clustering whose assignments are scored with the same metrics'
- name: Seurat
  role: Baseline R method whose cluster assignments on the PBMC dataset are scored
    with the same metrics
- name: scVI
  role: Baseline deep-generative method whose latent-space cluster assignments are
    scored with the same metrics
- name: FIt-SNE
  role: Embeds Miscell latent features (seed=123, nthreads=12, perplexity_list=[30,
    36, 42, 48]) before DBSCAN, generating the assignments being evaluated
  repo: https://github.com/KlugerLab/FIt-SNE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:055e6d1a2002d343ab6f82cc9f33ba1a6bd3033a9395b320dc3a9bbfe6b8b735
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  repo_url: https://github.com/lixiangchun/Miscell
  tools:
  - FIt-SNE
  - Miscell
  - Scanpy
  - Seurat
  - scVI
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# clustering-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute external clustering validation metrics — adjusted rand index (ARI), normalized mutual information (NMI), and V-measure — comparing each method's cell cluster assignments against reference cell-type labels on scRNA-seq datasets, as done to benchmark Miscell against Scanpy, Seurat and scVI on the 43,095-cell 10x PBMC dataset.

## When to use

Use when comparing cell-cluster assignments produced by two or more scRNA-seq clustering methods against ground-truth cell-type labels and need ARI, NMI and V-measure scores to rank methods.

## When NOT to use

- When no reference cell-type labels exist — ARI, NMI and V-measure are external metrics requiring ground truth
- When the goal is marker gene identification rather than clustering performance benchmarking (Miscell's attribution method is a separate skill)

## Inputs

- Cluster label vector per method (e.g. DBSCAN labels on FIt-SNE-embedded Miscell features, Leiden cluster assignments from scanpy, Seurat or scVI cluster assignments)
- Reference cell-type label vector for the same 43,095 PBMC cells

## Outputs

- Per-method ARI, NMI and V-measure scores
- clustering_metrics.csv with per-method clustering metrics

## How to apply

For each method's cluster label vector (e.g. DBSCAN labels on FIt-SNE-embedded Miscell features with eps=1, min_samples=5, or Leiden clusters from scanpy with n_neighbors=10), compute adjusted rand index, normalized mutual information, and V-measure against the reference cell-type labels (scikit-learn metrics). Apply the identical metric computation to every competing method (Scanpy, Seurat, scVI) on the same dataset so scores are directly comparable. On the 10x PBMC dataset (43,095 cells), Miscell was reported to exceed the second-best method by +33% ARI, +6% NMI and +7% V-measure, so any correct re-implementation should reproduce that ordering. Assemble the per-method scores into a table such as clustering_metrics.csv for downstream comparison.

## Related tools

- **Miscell** (Self-supervised model producing latent features whose DBSCAN/Leiden cluster assignments are one input to the metric computation) — https://github.com/lixiangchun/Miscell
- **scikit-learn** (Provides sklearn.cluster.dbscan and the adjusted rand index, normalized mutual information, and V-measure implementations used to score assignments against reference labels)
- **Scanpy** (Baseline method: neighborhood graph (sc.pp.neighbors, n_neighbors=10) plus Leiden clustering whose assignments are scored with the same metrics)
- **Seurat** (Baseline R method whose cluster assignments on the PBMC dataset are scored with the same metrics)
- **scVI** (Baseline deep-generative method whose latent-space cluster assignments are scored with the same metrics)
- **FIt-SNE** (Embeds Miscell latent features (seed=123, nthreads=12, perplexity_list=[30, 36, 42, 48]) before DBSCAN, generating the assignments being evaluated) — https://github.com/KlugerLab/FIt-SNE

## Examples

```
from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score, v_measure_score; ari = adjusted_rand_score(y_true, y_pred); nmi = normalized_mutual_info_score(y_true, y_pred); vm = v_measure_score(y_true, y_pred)
```

## Evaluation signals

- On the 10x PBMC dataset (43,095 cells), Miscell should show a gain of +33% ARI, +6% NMI and +7% V-measure over the second-best method (Scanpy, Seurat or scVI)
- All three metrics must be computed against the same reference cell-type label vector for every method, so per-method score rows in clustering_metrics.csv are comparable
- ARI of 1.0 indicates perfect agreement with reference labels; scores far below the reported ordering (Miscell > Scanpy/Seurat/scVI) suggest a labeling or index-alignment error
- NMI and V-measure should be bounded in [0, 1] and mutually consistent (V-measure equals the harmonic mean of homogeneity and completeness); negative or >1 values indicate miscomputed inputs

## Limitations

- Metric values depend on upstream clustering hyperparameters (DBSCAN eps=1/min_samples=5, Leiden resolution, n_neighbors=10); changing them changes scores
- Miscell demands a large sample size to obtain good performance, limiting metric-based benchmarking on small datasets
- External metrics require trustworthy reference labels; the paper's PBMC benchmark had 43,095 cells with defined cell types, which may not exist for new datasets

## Evidence

- [abstract] Miscell achieved at least comparable or better performance than the other methods by significant margin on clustering metrics including adjusted rand index, normalized mutual information, and V-measure score.: [evidence span withheld — anchor status: not_found]
- [intro] Evaluation datasets: "We systematically evaluated Miscell with three different datasets that were subjected to different sequencing protocols. The PMBC dataset consisted of 43,095 peripheral blood mononuclear cells"
- [methods] DBSCAN clustering parameters: "_, y = sklearn.cluster.dbscan(tsne, eps=1, min_samples=5, algorithm='auto')"
- [discussion] Small-sample limitation: "Analogous to methods built upon deep neural networks, Miscell demands a large sample size to obtain good performance."
