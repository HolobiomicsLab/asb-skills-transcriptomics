---
name: clustering-metric-computation-ari-nmi-vmeasure
description: Use when comparing clustering assignments of scRNA-seq cells against
  ground-truth labels (e.g., FACS-sorted CD4+/CD8+ T cells or Tabula Muris Senis cell
  types) and reporting ARI, NMI, and V-measure.
when_to_use_negative:
- No ground-truth cell-type labels exist for the dataset — ARI/NMI/V-measure require
  reference labels and cannot be computed
- 'Small datasets: Miscell (and thus this benchmarking pattern) demands a large sample
  size to obtain good performance'
- Data contain batch effects that have not been corrected — Miscell does not internally
  include any batch-correction methods, so labels would be compared against biased
  clusters
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn
  role: Computes adjusted Rand index, normalized mutual information, V-measure, and
    performs DBSCAN (eps=1, min_samples=5) on the t-SNE embedding
- name: Miscell
  role: Self-supervised deep neural network encoder whose latent features and cluster
    assignments are scored against reference labels
  repo: github.com/lixiangchun/Miscell
- name: FIt-SNE
  role: Produces the 2D embedding (seed=123, nthreads=12, perplexity_list=[30,36,42,48])
    on which DBSCAN clustering is run before metric computation
  repo: github.com/KlugerLab/FIt-SNE
- name: Scanpy
  role: Baseline clustering via neighborhood graph (n_neighbors=10, use_rep='X') and
    Leiden clustering; its sc.pl.tsne output is saved as '_scanpy.pdf'
- name: Seurat
  role: Baseline method whose cluster assignments are evaluated with the same ARI/NMI/V-measure
    metrics
- name: scVI
  role: Deep generative baseline method whose cluster assignments are evaluated with
    the same metrics
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:b5eeec2766647e89b07f5d574d71fcf4b3d9b82f4245deda2d4b1efe8396a9f1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  tools:
  - FIt-SNE
  - Miscell
  - Scanpy
  - Seurat
  - anndata
  - scVI
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# clustering-metric-computation-ari-nmi-vmeasure

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes adjusted Rand index (ARI), normalized mutual information (NMI), and V-measure to quantify how well single-cell cluster assignments (from Miscell, Seurat, Scanpy, or scVI pipelines) agree with ground-truth cell-type labels, enabling the head-to-head benchmark in the Miscell paper.

## When to use

Use when comparing clustering assignments of scRNA-seq cells against ground-truth labels (e.g., FACS-sorted CD4+/CD8+ T cells or Tabula Muris Senis cell types) and reporting ARI, NMI, and V-measure.

## When NOT to use

- No ground-truth cell-type labels exist for the dataset — ARI/NMI/V-measure require reference labels and cannot be computed
- Small datasets: Miscell (and thus this benchmarking pattern) demands a large sample size to obtain good performance
- Data contain batch effects that have not been corrected — Miscell does not internally include any batch-correction methods, so labels would be compared against biased clusters

## Inputs

- Latent feature matrix produced by a method's encoder (e.g., Miscell densenet encoder, scVI latent space)
- 2D FIt-SNE embedding of features (seed=123, nthreads=12, perplexity_list=[30,36,42,48])
- Predicted cluster labels (DBSCAN with eps=1, min_samples=5, or scanpy Leiden clusters)
- Ground-truth cell-type labels (FACS-sorted CD4+/CD8+ T cell labels; Tabula Muris Senis reference labels)
- AnnData object holding features, t-SNE coordinates, and cluster assignments

## Outputs

- Per-method ARI, NMI, and V-measure values
- Comparison table saved as CSV (e.g., clustering_metrics.csv, smartseq_clustering_metrics.csv)
- t-SNE plot colored by Leiden clusters (e.g., tsne_scanpy.pdf)

## How to apply

After each method (Miscell, Seurat, Scanpy, scVI) produces latent features and cluster assignments, embed the features in 2D with FIt-SNE (seed=123, nthreads=12, perplexity_list=[30, 36, 42, 48]) and obtain clusters either via DBSCAN (eps=1, min_samples=5, algorithm='auto') or via scanpy graph-based Leiden clustering on a neighborhood graph built with sc.pp.neighbors(n_neighbors=10, use_rep='X'). Then, using scikit-learn, compute ARI, NMI, and V-measure of each method's predicted cluster labels against the reference labels (e.g., the FACS-sorted T cell labels for Smart-seq or the cell-type labels for the 55,656-cell Tabula Muris Senis matrix from Figshare project 64982). Assemble all three metrics per method into a comparison table saved as CSV (e.g., clustering_metrics.csv or smartseq_clustering_metrics.csv). Judgment of correctness mirrors the paper's validation: Miscell should show a gain of at least ~10% ARI on Muris (and 120% ARI / 66% NMI / 67% V-measure on Smart-seq) over the second-best method, with comparable or better NMI and V-measure.

## Related tools

- **scikit-learn** (Computes adjusted Rand index, normalized mutual information, V-measure, and performs DBSCAN (eps=1, min_samples=5) on the t-SNE embedding)
- **Miscell** (Self-supervised deep neural network encoder whose latent features and cluster assignments are scored against reference labels) — github.com/lixiangchun/Miscell
- **FIt-SNE** (Produces the 2D embedding (seed=123, nthreads=12, perplexity_list=[30,36,42,48]) on which DBSCAN clustering is run before metric computation) — github.com/KlugerLab/FIt-SNE
- **Scanpy** (Baseline clustering via neighborhood graph (n_neighbors=10, use_rep='X') and Leiden clustering; its sc.pl.tsne output is saved as '_scanpy.pdf')
- **Seurat** (Baseline method whose cluster assignments are evaluated with the same ARI/NMI/V-measure metrics)
- **scVI** (Deep generative baseline method whose cluster assignments are evaluated with the same metrics)

## Examples

```
from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score, v_measure_score; ari = adjusted_rand_score(y_true, y)
```

## Evaluation signals

- clustering_metrics.csv (or smartseq_clustering_metrics.csv) contains ARI, NMI, and V-measure for all four methods (Miscell, Seurat, Scanpy, scVI)
- Miscell shows at least a +10% ARI gain over the second-best method on the 55,656-cell Tabula Muris dataset with comparable NMI and V-measure
- On the 71,494-cell Smart-seq dataset, Miscell metrics exceed the second-best method by ~120% ARI, ~66% NMI, and ~67% V-measure
- t-SNE plot colored by Leiden clusters was generated (e.g., tsne_scanpy.pdf / '_scanpy.pdf') from an embedding computed with seed=123 and perplexity_list=[30,36,42,48]
- Predicted cluster labels and reference labels have the same number of cells before metric computation (row alignment check)

## Limitations

- Miscell demands a large sample size to obtain good performance, limiting this benchmark pattern on small datasets
- Miscell does not internally include batch-correction methods; batch effects must be corrected as a preprocessing step before clustering and scoring
- Cluster-specific marker identification downstream of clustering is limited to highly variable genes
- Latent features lack explainability; biological mechanisms underlying them remain unknown

## Evidence

- [abstract] Miscell achieved at least comparable or better performance than the other methods by significant margin on clustering metrics including adjusted rand index, normalized mutual information, and V-measure score.: [evidence span withheld — anchor status: not_found]
