---
name: scrnaseq-count-matrix-preprocessing
description: Use when input is a raw scRNA-Seq count matrix (gene x cell CSV) that
  must be quality-filtered and reduced to highly variable genes before imputation
  or K-means clustering evaluation.
when_to_use_negative:
- Input is already an imputed, normalized, or feature-engineered expression matrix
  rather than a raw count matrix — take_norm/AutoClassImpute expect the raw count
  structure
- The analysis is bulk RNA-Seq or another data type without single-cell dropout structure;
  AutoClass preprocessing targets scRNA-Seq noise (dropouts, profiling noise, batch
  effects)
- The matrix has already been restricted to the desired highly variable gene set and
  quality-filtered; re-filtering would discard signal
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutoClass
  role: Provides take_norm and find_hv_genes preprocessing helpers; its AutoClassImpute
    consumes the preprocessed raw count matrix
  repo: https://github.com/datapplab/AutoClass
- name: Python 3
  role: Runtime environment for the preprocessing workflow (pandas loading, numpy
    arrays, sklearn metrics)
- name: TensorFlow >=2.0
  role: Dependency required to run the AutoClass module used for imputation after
    preprocessing
- name: numpy
  role: Array backend for count matrices, e.g. X = expr.values
- name: pandas
  role: Loads count matrices and true label CSVs with read_csv(..., index_col=0)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:986bf5edff662e1d6ed2ae9d220583e7cf93ec77e28e7d50d1c22870d5959623
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - DCA
  - DESC
  - MAGIC
  - Python
  - Python 3
  - SAVER
  - SCANPY
  - Seurat
  - TensorFlow >=2.0
  - anndata
  - numpy
  - pandas
  - scCAEs
  - scDeepCluster
  - scImpute
  - scMGCA
  - scVI
  - scikit-learn
  - scipy
  - sklearn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# scrnaseq-count-matrix-preprocessing

## Summary

Load raw scRNA-Seq count matrices (CSV) with pandas, filter low-quality genes/cells, and retain the top highly variable genes (e.g., top 1000 via find_hv_genes) so downstream imputation (AutoClass) and clustering are computed on a cleaned feature space.

## When to use

Use when input is a raw scRNA-Seq count matrix (gene x cell CSV) that must be quality-filtered and reduced to highly variable genes before imputation or K-means clustering evaluation.

## When NOT to use

- Input is already an imputed, normalized, or feature-engineered expression matrix rather than a raw count matrix — take_norm/AutoClassImpute expect the raw count structure
- The analysis is bulk RNA-Seq or another data type without single-cell dropout structure; AutoClass preprocessing targets scRNA-Seq noise (dropouts, profiling noise, batch effects)
- The matrix has already been restricted to the desired highly variable gene set and quality-filtered; re-filtering would discard signal

## Inputs

- Raw scRNA-Seq count matrix as CSV (gene x cell, e.g. Buettner.csv, Usoskin, Lake, Zeisel, Dataset1_raw.csv)
- Known cell-type label file per dataset (e.g. Dataset1_true.csv)
- Number of highly variable genes to retain (top=200 in the paper's evaluation; top=1000 in the reproduction workflow)

## Outputs

- Filtered count matrix restricted to top highly variable genes (numpy array X suitable for AutoClassImpute)
- Normalized matrix X_norm = take_norm(X) for PCA/t-SNE visualization and clustering
- Cleaned gene and cell sets passed to KMeans clustering with true K (random_state=1)

## How to apply

Load each count matrix and its known cell-type labels with pandas read_csv (index_col=0), e.g. Buettner.csv, Usoskin, Lake, Zeisel, or Dataset1_raw.csv. Preprocess per the Methods section: filter low-quality genes/cells, then select the top highly variable genes with the find_hv_genes helper (the paper's clustering evaluation uses find_hv_genes(X, top=200); the reproduction workflow retains the top 1000 HVGs per dataset). For visualization or clustering pipelines, normalize counts with take_norm (X_norm = take_norm(X)) before dimensionality reduction (PCA to 50 components, then t-SNE to 2 dimensions). Keep raw integer counts as the direct input to AutoClassImpute(X, dropout_rate=0.3), since the model itself is distribution agnostic and expects the raw count structure. The rationale is that removing low-quality units and focusing on variable genes amplifies cell-type signal while leaving dropout handling to the imputation step.

## Related tools

- **AutoClass** (Provides take_norm and find_hv_genes preprocessing helpers; its AutoClassImpute consumes the preprocessed raw count matrix) — https://github.com/datapplab/AutoClass
- **Python 3** (Runtime environment for the preprocessing workflow (pandas loading, numpy arrays, sklearn metrics))
- **TensorFlow >=2.0** (Dependency required to run the AutoClass module used for imputation after preprocessing)
- **numpy** (Array backend for count matrices, e.g. X = expr.values)
- **pandas** (Loads count matrices and true label CSVs with read_csv(..., index_col=0))

## Examples

```
expr = pd.read_csv('Buettner.csv', index_col=0); X = expr.values; highvar_genes = find_hv_genes(X, top=200); X_norm = take_norm(X)
```

## Evaluation signals

- Preprocessed matrix contains exactly the requested number of highly variable genes (find_hv_genes(X, top=200) yields 200; workflow variant yields 1000)
- Loaded matrices align with cell-type labels: same cells/order as Dataset1_true.csv (or equivalent label file) so clustering metrics are computable
- Downstream sanity check: KMeans (n_clusters=K, random_state=1) on the preprocessed/imputed data reproduces reported patterns, e.g. Usoskin ARI = 0.869 for AutoClass vs 0.221 raw
- Raw counts passed to AutoClassImpute retain their count structure (no prior log/normalization applied to the imputation input); normalization (take_norm) applied only for PCA/t-SNE branches
- Preprocessing pipeline requires TensorFlow >=2.0 and installs numpy, pandas, sklearn, matplotlib per the Methods instructions

## Limitations

- The paper states no explicit limitations section; supplementary tables with hyperparameter settings (Supplementary Tables 1-3) are referenced but not included in the provided text
- No software version numbers are stated in the article beyond the TensorFlow >=2.0 requirement
- The highly variable gene count differs between contexts (top=200 in the paper's clustering-evaluation snippet vs top=1000 in the reproduction workflow), so the cutoff must be chosen deliberately per analysis

## Evidence

- [methods] AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed"
- [results] find_hv_genes top-200: "highvar_genes = find_hv_genes(X,top=200)"
- [results] take_norm normalization: "X_norm = take_norm(X)"
- [results] PCA/t-SNE reduction: "PCA_raw = PCA(n_components=50).fit_transform(X_norm)
tsne_raw = TSNE(n_components=2).fit_transform(PCA_raw)"
- [results] pandas loading of raw data: "expr = pd.read_csv('Dataset1_raw.csv',index_col=0)"
- [results] imputation input with dropout rate: "res = AutoClassImpute(X,dropout_rate=0.3)"
