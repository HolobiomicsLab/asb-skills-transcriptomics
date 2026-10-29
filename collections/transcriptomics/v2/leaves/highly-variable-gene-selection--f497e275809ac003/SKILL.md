---
name: highly-variable-gene-selection
description: Use when a raw scRNA-Seq count matrix (thousands of genes) must be feature-reduced
  before AutoClass/control imputation or K-means clustering, e.g., top-200 genes for
  clustering evaluation or 1000/5000-gene subsets for the Tian et al. noise benchmark.
when_to_use_negative:
- The analysis targets differential expression or marker discovery over the full transcriptome,
  where discarding genes would remove candidates of interest
- The input matrix is already a small targeted gene panel (feature reduction would
  discard most of the measured features)
- The downstream task needs all genes preserved, e.g., recovering a full denoised
  expression matrix for every gene
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: AutoClass
  role: Provides the find_hv_genes function used to rank and select the top-K highly
    variable genes prior to imputation and clustering
  repo: https://github.com/datapplab/AutoClass
- name: pandas
  role: Loads the raw count CSV matrices (index_col=0) that are subset to the selected
    genes
- name: scikit-learn (sklearn)
  role: Used downstream on the gene-subset matrix for KMeans clustering and PCA before
    t-SNE embedding
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:166dd788f84187965b9ee601243b4d1720910ed40382717b333d1b8dfb3d9e2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - DCA
  - MAGIC
  - POT
  - Python
  - SAVER
  - Scanpy
  - SpaOTsc
  - numpy
  - pandas
  - scImpute
  - scikit-learn
  - scikit-learn (sklearn)
  - scipy
  - sklearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# highly-variable-gene-selection

## Summary

Reduces scRNA-Seq count matrices to their top-K most variable genes (e.g., find_hv_genes(X, top=200), or 1000/5000 in the benchmarking tasks) before imputation or clustering, focusing analysis on genes carrying cell-type signal while cutting noise and dimensionality.

## When to use

Use when a raw scRNA-Seq count matrix (thousands of genes) must be feature-reduced before AutoClass/control imputation or K-means clustering, e.g., top-200 genes for clustering evaluation or 1000/5000-gene subsets for the Tian et al. noise benchmark.

## When NOT to use

- The analysis targets differential expression or marker discovery over the full transcriptome, where discarding genes would remove candidates of interest
- The input matrix is already a small targeted gene panel (feature reduction would discard most of the measured features)
- The downstream task needs all genes preserved, e.g., recovering a full denoised expression matrix for every gene

## Inputs

- Raw scRNA-Seq count matrix as CSV with gene rows/cell columns (e.g., Dataset1_raw.csv, Baron.csv, Buettner.csv, Usoskin, Lake, Zeisel, or the 305-cell x 13,426-gene Tian et al. matrix)
- Integer feature count K for the top highly variable genes (200, 1000, or 5000 depending on the analysis)
- Known cell-type label vector for the same cells (used downstream, defines what 'informative' variation means)

## Outputs

- List/array of selected highly variable gene indices or identifiers (highvar_genes)
- Count matrix restricted to the top-K highly variable genes, used for imputation and KMeans/t-SNE/ASW analysis

## How to apply

Load the gene-by-cell count matrix with pandas, then call the AutoClass module's find_hv_genes function with the top parameter set to the desired feature count — 200 for the paper's clustering-evaluation scheme, or 1000/5000 when subsetting the Tian et al. sc_mixology matrix for noise benchmarks. Restrict the raw matrix and every downstream imputed matrix (AutoClassImpute with dropout_rate=0.3, DCA, SAVER, scImpute, MAGIC) to the identical gene subset so comparisons across methods remain fair. The rationale is that low-variance genes contribute mostly profiling noise, and the deep-learning imputers (TF >=2.0) train faster and denoise better on the informative feature space. Keep the subset consistent with the known cell-type labels so ARI, JI, NMI, purity, or ASW metrics computed afterwards reflect the selected feature set.

## Related tools

- **AutoClass** (Provides the find_hv_genes function used to rank and select the top-K highly variable genes prior to imputation and clustering) — https://github.com/datapplab/AutoClass
- **pandas** (Loads the raw count CSV matrices (index_col=0) that are subset to the selected genes)
- **scikit-learn (sklearn)** (Used downstream on the gene-subset matrix for KMeans clustering and PCA before t-SNE embedding)

## Examples

```
highvar_genes = find_hv_genes(X, top=200); X_hvg = X[:, highvar_genes]
```

## Evaluation signals

- The selected gene set has exactly the requested size K (200, 1000, or 5000) and no duplicate genes
- The same highvar_genes subset is applied to the raw matrix and to every imputed matrix (AutoClass, DCA, SAVER, scImpute, MAGIC) before metric computation
- KMeans with the true cluster number K (random_state=1) on the subset reproduces reported patterns, e.g., Usoskin ARI = 0.869 for AutoClass vs 0.221 on raw data
- For the Tian et al. benchmark, ASW on the 1000-gene and 5000-gene subsets is highest for AutoClass under both dropout and negative-binomial noise
- t-SNE/PCA embeddings from the subset matrix retain cell-type separation consistent with the known labels

## Limitations

- The exact variance criterion inside find_hv_genes is not detailed in the provided text; only the function signature find_hv_genes(X, top=200) and the top-K choices (200/1000/5000) are documented
- Supplementary Tables 1-3 with hyperparameter and simulation settings are referenced but not included, so some parameter justification cannot be verified from the provided material
- Results depend on the chosen K: the paper evaluates different feature sizes (1000 vs 5000 genes) and performance varies across datasets, so a single fixed K may not generalize

## Evidence

- [results] Top-200 HVG selection precedes clustering evaluation: [evidence span withheld — anchor status: not_found]
- [results] Concrete function call for HVG selection: "highvar_genes = find_hv_genes(X,top=200)"
- [other] Alternative feature sizes used in the Tian et al. benchmark: [evidence span withheld — anchor status: not_found]
- [results] Downstream pipeline applied to the subset matrix: "X_norm = take_norm(X)"
- [methods] Software environment for the AutoClass module used in selection and imputation: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed"
