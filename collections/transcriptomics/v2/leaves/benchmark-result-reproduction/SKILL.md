---
name: benchmark-result-reproduction
description: Use when the goal is to verify a published quantitative claim (e.g. that
  scMGCA attains the highest ASW on 'QS Trachea' and 'QS Heart') by re-executing the
  full method-vs-baseline comparison on the original benchmark scRNA-seq datasets.
when_to_use_negative:
- The goal is downstream biological interpretation (e.g. PDAC marker-gene annotation
  or pathway analysis) rather than reproducing the clustering/ASW benchmark ranking.
- Only the method under test is available without the paper's comparison baselines
  (PCA, t-SNE, UMAP, SCANPY, Seurat), so the methods-vs-baselines table cannot be
  assembled.
- The dataset exceeds practical single-run limits and cannot be batched as scMGCA
  requires (the paper splits datasets with >25,000 cells into multiple batches).
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMGCA
  role: Method under test — graph-embedding autoencoder whose latent representations
    and cluster assignments are benchmarked on ASW
  repo: https://pypi.org/project/scMGCA
- name: SCANPY
  role: Data loading (anndata) and source of PCA, t-SNE, and UMAP baseline embeddings
    on the benchmark datasets
- name: Seurat
  role: R baseline pipeline run end-to-end to produce an embedding compared in the
    ASW table
  repo: https://github.com/satijalab/seurat
- name: scikit-learn
  role: silhouette_score used to compute per-cell silhouette widths of every embedding
    against true labels
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:da40cd9d658e688c1ab5227d95c7060fae9326f5f6579577ec89a7d686f8cb68
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCANPY
  - Seurat
  - anndata
  - numpy
  - pandas
  - scMGCA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# benchmark-result-reproduction

## Summary

Reproduce a published benchmark comparison by re-running the method under test (scMGCA graph convolutional autoencoder) alongside its comparison baselines (PCA, t-SNE, UMAP, full SCANPY and Seurat pipelines) and evaluating them with the paper's own metric — average silhouette width (ASW) of true cell-type labels — to confirm the reported ranking.

## When to use

Use when the goal is to verify a published quantitative claim (e.g. that scMGCA attains the highest ASW on 'QS Trachea' and 'QS Heart') by re-executing the full method-vs-baseline comparison on the original benchmark scRNA-seq datasets.

## When NOT to use

- The goal is downstream biological interpretation (e.g. PDAC marker-gene annotation or pathway analysis) rather than reproducing the clustering/ASW benchmark ranking.
- Only the method under test is available without the paper's comparison baselines (PCA, t-SNE, UMAP, SCANPY, Seurat), so the methods-vs-baselines table cannot be assembled.
- The dataset exceeds practical single-run limits and cannot be batched as scMGCA requires (the paper splits datasets with >25,000 cells into multiple batches).

## Inputs

- Normalized 'QS Trachea' and 'QS Heart' scRNA-seq gene expression matrices (anndata/SCANPY objects)
- Ground-truth cell-type label vectors per dataset
- Method-under-test outputs: scMGCA latent embedded representations and cluster assignments (TensorFlow backend)
- Baseline embeddings: PCA, t-SNE, UMAP via SCANPY; full SCANPY and Seurat pipeline outputs (Seurat v5 R toolkit)

## Outputs

- asw_comparison.csv — methods x datasets table of average silhouette width (ASW) values
- asw_comparison.png — bar-chart figure mirroring the paper's ASW benchmark figure
- Pass/fail validation verdict on whether scMGCA attains the highest ASW on 'QS Trachea' and 'QS Heart'

## How to apply

Load the normalized 'QS Trachea' and 'QS Heart' expression matrices plus ground-truth cell-type labels with SCANPY/anndata, first filtering out genes not expressed in cells to mitigate the high dropout rates in scRNA-seq data. Run scMGCA (TensorFlow-backed graph convolutional autoencoder with two GCN layers, multinomial decoder, and KL-divergence self-optimized clustering initialized from the best of K-means/spectral clustering) to obtain latent embeddings and cluster assignments, then compute PCA, t-SNE, and UMAP embeddings via SCANPY and run the full SCANPY and Seurat processing pipelines as baselines. For every embedding, compute per-cell silhouette widths with scikit-learn's silhouette_score against the labels, average per method and per dataset, assemble into a methods-x-datasets table, and mirror the paper's ASW bar-chart figure. Validate against the paper's claim that scMGCA attains the highest ASW on 'QS Trachea' and 'QS Heart'; flag the reproduction as failed if it does not. Save the table as asw_comparison.csv and the figure as asw_comparison.png.

## Related tools

- **scMGCA** (Method under test — graph-embedding autoencoder whose latent representations and cluster assignments are benchmarked on ASW) — https://pypi.org/project/scMGCA
- **SCANPY** (Data loading (anndata) and source of PCA, t-SNE, and UMAP baseline embeddings on the benchmark datasets)
- **Seurat** (R baseline pipeline run end-to-end to produce an embedding compared in the ASW table) — https://github.com/satijalab/seurat
- **scikit-learn** (silhouette_score used to compute per-cell silhouette widths of every embedding against true labels)

## Examples

```
from sklearn.metrics import silhouette_score; asw = silhouette_score(embedding, true_labels)
```

## Evaluation signals

- scMGCA attains the highest ASW on 'QS Trachea' and 'QS Heart'; if it does not, the reproduction is flagged as failed.
- asw_comparison.csv contains one row per method (scMGCA, PCA, t-SNE, UMAP, SCANPY, Seurat) and one column per dataset with valid ASW values.
- asw_comparison.png visually matches the layout of the paper's ASW benchmark bar-chart figure.
- Every embedding is scored with silhouette_score against ground-truth labels (not predicted clusters) before averaging per method and dataset.
- Latent representations come from the full scMGCA pipeline (two GCN layers, multinomial decoder, KL-divergence self-optimization with K-means/spectral initialization), not a partially trained model.

## Limitations

- No software version numbers are reported for Python, TensorFlow, SCANPY, or Seurat, so exact reproducibility of the baselines may vary.
- Key scMGCA hyperparameters are unstated in the paper (KNN neighbor count k, restart probability α, loss weights γ1/γ2/γ3, maximum iterations, number of highly variable genes d), which can affect the reproduced embeddings.
- No specific dataset accessions are provided; benchmark data are only described as obtained from public repositories, so dataset identity must be inferred.

## Evidence

- [results] scMGCA has the highest ASW on 'QS Trachea' and 'QS Heart' datasets: [evidence span withheld — anchor status: not_found]
- [discussion] The first step is to filter out genes that are not expressed in cells, reducing the impact of the high dropout rates in scRNA-seq data.: "The first step is to filter out genes that are not expressed in cells, reducing the impact of the high dropout rates in scRNA-seq data."
- [results] we selected the best strategy from K-means and spectral clustering to initialize the clustering centers: "we selected the best strategy from K-means and spectral clustering to initialize the clustering centers"
- [readme] scMGCA is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder: "scMGCA` is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder that simultaneously learns cell–cell topology representation and"
- [abstract] scMGCA is accurate and effective for cell segregation and batch effect correction, outperforming other state-of-the-art models across multiple platforms: "We show that scMGCA is accurate and effective for cell segregation and batch effect correction, outperforming other state-of-the-art models across multiple platforms"
