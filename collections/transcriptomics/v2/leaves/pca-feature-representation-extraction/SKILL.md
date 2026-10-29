---
name: pca-feature-representation-extraction
description: Use when preparing a cell-typed scRNA-seq .h5ad (AnnData) as input to
  scValue or another OOB random-forest sketching step that expects a dense PCA feature
  matrix via use_rep='X_pca'.
when_to_use_negative:
- The dataset is not cell-typed or lacks a cell_type annotation key, since downstream
  DV computation requires labels Y
- The dataset is already a dense feature/representation table (e.g. an existing embedding);
  recomputing PCA on top would be redundant
- The downstream tool expects raw expression (adata.X) rather than a PCA representation
  — use use_rep='X' instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Scanpy
  role: Reads the .h5ad dataset and computes HVG selection (sc.pp.highly_variable_genes,
    n_top_genes=3000) and PCA (sc.pp.pca, n_comps=50) storing adata.obsm['X_pca']
- name: scValue
  role: Consumes the PCA representation via use_rep='X_pca' to fit the random forest
    and compute per-cell OOB data values
  repo: https://github.com/LHBCB/scvalue
- name: scikit-learn 1.5.2
  role: RandomForestClassifier (n_estimators=100, seed=42) fit on the PCA feature
    matrix X to derive OOB data values
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:51fcd90b12e248ae01d780f4a021b57388a636045e1811258ade99df4d54c431
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/LHBCB/scvalue
  repo_url: https://github.com/LHBCB/scvalue
  tools:
  - Scanpy
  - scValue
  - scikit-learn
  - scikit-learn 1.5.2
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# pca-feature-representation-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts a PCA representation from a log-normalised scRNA-seq .h5ad dataset (Scanpy, top 3,000 highly variable genes, 50 PCs) and stores it in adata.obsm['X_pca'] as the feature matrix X consumed by scValue's random-forest data value computation.

## When to use

Use when preparing a cell-typed scRNA-seq .h5ad (AnnData) as input to scValue or another OOB random-forest sketching step that expects a dense PCA feature matrix via use_rep='X_pca'.

## When NOT to use

- The dataset is not cell-typed or lacks a cell_type annotation key, since downstream DV computation requires labels Y
- The dataset is already a dense feature/representation table (e.g. an existing embedding); recomputing PCA on top would be redundant
- The downstream tool expects raw expression (adata.X) rather than a PCA representation — use use_rep='X' instead

## Inputs

- Cell-typed scRNA-seq dataset in .h5ad (AnnData) format, log-normalised, with cell type labels in adata.obs['cell_type']
- Highly variable gene annotation (sc.pp.highly_variable_genes, n_top_genes=3000)
- PCA embedding stored in adata.obsm['X_pca'] (sc.pp.pca, n_comps=50)

## Outputs

- adata.obsm['X_pca'] — dense N-cells × 50 PCs feature matrix ready for random-forest fitting
- Preprocessed AnnData object passed to SCValue(adata=..., use_rep='X_pca')

## How to apply

Load the .h5ad dataset with Scanpy (sc.read_h5ad); the mTC example is already log-normalised. Run sc.pp.highly_variable_genes with n_top_genes=3000 to restrict to the top highly variable genes, then sc.pp.pca with n_comps=50, which stores the principal components in adata.obsm['X_pca']. Pass use_rep='X_pca' (the scValue default) so the RandomForestClassifier is fit on the PCA matrix rather than raw counts; alternatively use_rep='X' uses adata.X directly, but sparse matrices may be densified internally. The rationale is that PCA on HVGs gives a compact, low-noise feature representation suited to fitting the B=100-tree random forest that scores per-cell OOB data values.

## Related tools

- **Scanpy** (Reads the .h5ad dataset and computes HVG selection (sc.pp.highly_variable_genes, n_top_genes=3000) and PCA (sc.pp.pca, n_comps=50) storing adata.obsm['X_pca'])
- **scValue** (Consumes the PCA representation via use_rep='X_pca' to fit the random forest and compute per-cell OOB data values) — https://github.com/LHBCB/scvalue
- **scikit-learn 1.5.2** (RandomForestClassifier (n_estimators=100, seed=42) fit on the PCA feature matrix X to derive OOB data values)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('mTC.h5ad'); sc.pp.highly_variable_genes(adata, n_top_genes=3000); sc.pp.pca(adata, n_comps=50)
```

## Evaluation signals

- adata.obsm['X_pca'] exists and has shape (n_cells, 50) matching sc.pp.pca(n_comps=50)
- Highly variable gene selection restricted features to the top 3,000 HVGs before PCA
- SCValue accepts the AnnData without densification warnings, confirming a dense X_pca matrix
- Subsequent scValue run produces per-cell DV scores in [0,1] stored in adata.obs['dv'], indicating the feature matrix was fit-usable
- Fixed seed=42 yields reproducible PCA/RF results across runs

## Limitations

- Using use_rep='X' with sparse matrices may be densified internally, increasing memory usage; PCA on HVGs avoids this
- PCA is computed on the top 3,000 highly variable genes only, so information from non-HVG genes is discarded
- Experiment versions (Scanpy 1.10.2, AnnData 0.10.8) differ from the package's core dependency pins (Python 3.10, scikit-learn 1.5.2, SciPy 1.14.0, NumPy 1.26.4, pandas 1.5.3, joblib 1.4.2)

## Evidence

- [methods] task_006: [evidence span withheld — anchor status: not_found]
