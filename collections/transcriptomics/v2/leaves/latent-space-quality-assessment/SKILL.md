---
name: latent-space-quality-assessment
description: Use when a scRNA-seq study produces multiple low-dimensional embeddings
  (e.g., scMGCA graph-autoencoder latents vs PCA/t-SNE/UMAP) and you must quantify
  which embedding preserves cell-type structure.
when_to_use_negative:
- The ground-truth cell-type labels are unavailable for the dataset — silhouette width
  of true labels cannot be computed, so ASW is undefined for this benchmark
- You are comparing clustering accuracy (e.g., ARI/NMI of predicted labels) rather
  than the geometric quality of the low-dimensional latent space itself
- The embedding comes from a method evaluated on different datasets than those used
  in the original benchmark (the paper's claim is specific to 'QS Trachea' and 'QS
  Heart')
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: scMGCA
  role: 'Method under test: graph-embedding autoencoder producing the latent representation
    whose ASW is compared against baselines; installed from PyPI, core model built
    on TensorFlow'
  repo: https://pypi.org/project/scMGCA
- name: SCANPY
  role: Provides PCA, t-SNE, and UMAP baseline embeddings and loads the anndata expression
    matrices and true labels
- name: Seurat
  role: Full Seurat processing pipeline as one of the comparison baselines for the
    ASW benchmark
  repo: https://github.com/satijalab/seurat
- name: scikit-learn
  role: silhouette_score computes per-cell silhouette widths on each embedding using
    the true cell labels
- name: TensorFlow
  role: Backend on which the scMGCA graph convolutional autoencoder is run to obtain
    latent embeddings
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:01c081b5a8e444f01a2f37704e11e8d72926a2c88b2151333e7f69ebd448cc25
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCANPY
  - Seurat
  - TensorFlow
  - anndata
  - numpy
  - pandas
  - scMGCA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# latent-space-quality-assessment

## Summary

Benchmarks the quality of dimensionality-reduced latent representations from single-cell methods by computing the average silhouette width (ASW) of true cell-type labels with scikit-learn, comparing scMGCA embeddings against PCA, t-SNE, UMAP, and full SCANPY/Seurat pipelines on benchmark scRNA-seq datasets.

## When to use

Use when a scRNA-seq study produces multiple low-dimensional embeddings (e.g., scMGCA graph-autoencoder latents vs PCA/t-SNE/UMAP) and you must quantify which embedding best preserves cell-type structure.

## When NOT to use

- The ground-truth cell-type labels are unavailable for the dataset — silhouette width of true labels cannot be computed, so ASW is undefined for this benchmark
- You are comparing clustering accuracy (e.g., ARI/NMI of predicted labels) rather than the geometric quality of the low-dimensional latent space itself
- The embedding comes from a method evaluated on different datasets than those used in the original benchmark (the paper's claim is specific to 'QS Trachea' and 'QS Heart')

## Inputs

- Normalized scRNA-seq gene expression matrix (e.g., anndata .h5ad) with ground-truth cell-type labels for each benchmark dataset ('QS Trachea', 'QS Heart')
- Latent embedded representations from scMGCA (graph convolutional autoencoder) and from PCA, t-SNE, UMAP, SCANPY, and Seurat baseline pipelines

## Outputs

- asw_comparison.csv — comparison table with rows = methods (scMGCA, PCA, t-SNE, UMAP, SCANPY, Seurat), columns = datasets, cells = average silhouette width
- asw_comparison.png — bar-chart figure mirroring the paper's ASW benchmark figure
- Pass/fail validation flag: whether scMGCA attained the highest ASW on each dataset as reported

## How to apply

Load the normalized expression matrices and ground-truth cell-type labels for each benchmark dataset (in the paper, 'QS Trachea' and 'QS Heart') with SCANPY/anndata. Generate each method's latent representation: scMGCA via its graph convolutional autoencoder (TensorFlow backend, GCN encoder plus multinomial-based decoder, with clustering centers initialized from the best of K-means/spectral clustering), and baselines PCA, t-SNE, and UMAP via SCANPY plus full SCANPY and Seurat processing pipelines. For every embedding, compute each cell's silhouette width with scikit-learn's silhouette_score against the TRUE cell labels (not predicted cluster labels), then average per method and per dataset. Assemble the ASW values into a methods-by-datasets table and bar-chart figure mirroring the paper's ASW benchmark. Validation criterion from the paper: scMGCA must attain the highest ASW on 'QS Trachea' and 'QS Heart'; otherwise the reproduction is flagged as failed. Save the table as asw_comparison.csv and the figure as asw_comparison.png.

## Related tools

- **scMGCA** (Method under test: graph-embedding autoencoder producing the latent representation whose ASW is compared against baselines; installed from PyPI, core model built on TensorFlow) — https://pypi.org/project/scMGCA
- **SCANPY** (Provides PCA, t-SNE, and UMAP baseline embeddings and loads the anndata expression matrices and true labels)
- **Seurat** (Full Seurat processing pipeline as one of the comparison baselines for the ASW benchmark) — https://github.com/satijalab/seurat
- **scikit-learn** (silhouette_score computes per-cell silhouette widths on each embedding using the true cell labels)
- **TensorFlow** (Backend on which the scMGCA graph convolutional autoencoder is run to obtain latent embeddings)

## Examples

```
from sklearn.metrics import silhouette_score; asw = silhouette_score(embedding, true_labels)
```

## Evaluation signals

- scMGCA attains the highest average silhouette width on both 'QS Trachea' and 'QS Heart', matching the paper's reported result; failure to reproduce this flags the reproduction as failed
- Silhouette widths are computed with scikit-learn's silhouette_score against TRUE cell labels, not predicted cluster assignments
- The output table asw_comparison.csv has rows = methods (scMGCA, PCA, t-SNE, UMAP, SCANPY, Seurat) and columns = datasets ('QS Trachea', 'QS Heart')
- ASW values fall in [-1, 1] and the best-performing method's bar in asw_comparison.png matches the highest cell in the CSV
- Every method/dataset cell in the table is populated; missing embeddings or NaN silhouettes indicate the baseline was not run correctly

## Limitations

- No software version numbers are reported (Python, TensorFlow, SCANPY, Seurat versions unstated), so exact reproduction of baseline embeddings may vary
- No specific dataset accessions or repository identifiers are provided; benchmark data are only described as coming from public repositories (Source Data file referenced without a DOI or URL in this section)
- Key scMGCA hyperparameters (KNN neighbor count k, restart probability α, loss weights γ1/γ2/γ3, maximum iterations, number of highly variable genes d) are not specified numerically, affecting the latent embeddings being scored

## Evidence

- [results] ASW result on benchmark datasets: [evidence span withheld — anchor status: not_found]
- [readme] Silhouette index used for embedding quality assessment (analogous usage in related tooling): [evidence span withheld — anchor status: external_doc]
- [abstract] Latent representations being assessed come from the graph-embedding autoencoder: "a graph-embedding autoencoder that simultaneously learns cell-cell topology representation and cluster assignments"
