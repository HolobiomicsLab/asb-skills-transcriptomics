---
name: count-matrix-reconstruction-error-analysis
description: Use when you have imputed scRNA-Seq count matrices plus ground-truth
  true counts or known cell-group labels and need to quantify reconstruction accuracy
  and clustering recovery.
when_to_use_negative:
- The dataset is real data without ground-truth true counts or known cell labels,
  so MSE and ARI against simulated groups cannot be computed
- The goal is to impute or denoise the data itself rather than to evaluate an already-imputed
  matrix
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: AutoClass
  role: Imputation method under evaluation; provides AutoClassImpute and take_norm
    for generating and normalizing reconstructed count matrices
  repo: https://github.com/datapplab/AutoClass
- name: DCA
  role: Unstable comparison method whose reconstruction quality metrics fluctuate
    across settings
  repo: https://github.com/theislab/dca
- name: Splatter
  role: Generates the simulated Dataset 1 with known true counts and six groups used
    as evaluation ground truth
- name: TensorFlow >=2.0
  role: Runtime backend required to run AutoClassImpute
- name: numpy
  role: Computes MSE and the dropout0/true0 zero-recovery ratio against true counts
- name: scikit-learn
  role: Provides PCA, t-SNE, and KMeans used to derive ASW and ARI clustering metrics
- name: matplotlib
  role: Plots metric-versus-hyperparameter curves comparing AutoClass and DCA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:937732deb390fc515e4b39925ae6356d7aa3e7982cd591c80a6ce2182519833e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - DCA
  - Python
  - Splatter
  - TensorFlow >=2.0
  - matplotlib
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# count-matrix-reconstruction-error-analysis

## Summary

Evaluates how faithfully an imputed/denoised scRNA-Seq count matrix (e.g., from AutoClass or DCA) recovers the true counts and biological structure, using MSE, the dropout0/true0 zero-recovery ratio, and clustering metrics (ASW, ARI) against known simulated cell groups.

## When to use

Use when you have imputed scRNA-Seq count matrices plus ground-truth true counts or known cell-group labels and need to quantify reconstruction accuracy and clustering recovery.

## When NOT to use

- The dataset is real data without ground-truth true counts or known cell labels, so MSE and ARI against simulated groups cannot be computed
- The goal is to impute or denoise the data itself rather than to evaluate an already-imputed matrix

## Inputs

- Simulated dropout count matrix (CSV, e.g., Dataset1_raw.csv, 1000 genes x 500 cells in six groups)
- True count matrix (CSV, e.g., Dataset1_true.csv)
- Imputed/denoised count matrix produced by AutoClassImpute or DCA
- Known cell-group labels for the simulated data (six groups)

## Outputs

- Per-setting metrics table with MSE, dropout0/true0 zero-recovery ratio, ASW, and ARI
- Metric-versus-hyperparameter comparison curves (matplotlib) for AutoClass and DCA
- t-SNE embeddings (PCA to 50 components then t-SNE to 2 dimensions) with KMeans cluster assignments

## How to apply

Load the simulated dropout matrix (Dataset1_raw.csv) and true count matrix (Dataset1_true.csv) with pandas, impute the raw counts (e.g., res = AutoClassImpute(X, dropout_rate=0.3)), then normalize with take_norm. For structure recovery, run PCA to 50 components followed by t-SNE to 2 dimensions, apply KMeans with K equal to the number of simulated groups (six in Dataset 1), and compute ASW (silhouette width) and ARI against the true group labels. For count-level accuracy, compute the MSE between imputed and true counts, plus the dropout0/true0 zero-recovery ratio measuring how well zeros introduced by dropout are restored to the true nonzero values, using numpy. Optionally select the top-200 highly variable genes (find_hv_genes) before clustering evaluation. Assemble all metrics into a single table and plot metric-versus-hyperparameter curves to compare stability across methods (AutoClass vs. DCA).

## Related tools

- **AutoClass** (Imputation method under evaluation; provides AutoClassImpute and take_norm for generating and normalizing reconstructed count matrices) — https://github.com/datapplab/AutoClass
- **DCA** (Unstable comparison method whose reconstruction quality metrics fluctuate across settings) — https://github.com/theislab/dca
- **Splatter** (Generates the simulated Dataset 1 with known true counts and six groups used as evaluation ground truth)
- **TensorFlow >=2.0** (Runtime backend required to run AutoClassImpute)
- **numpy** (Computes MSE and the dropout0/true0 zero-recovery ratio against true counts)
- **scikit-learn** (Provides PCA, t-SNE, and KMeans used to derive ASW and ARI clustering metrics)
- **matplotlib** (Plots metric-versus-hyperparameter curves comparing AutoClass and DCA)

## Examples

```
expr = pd.read_csv('Dataset1_raw.csv',index_col=0); expr_true = pd.read_csv('Dataset1_true.csv',index_col=0); res = AutoClassImpute(expr.values, dropout_rate=0.3)
```

## Evaluation signals

- AutoClass's ASW, ARI, MSE, and dropout0/true0 ratio stay unchanged as bottleneck size sweeps 16-256 and pre-clustering K sweeps 4-8, while DCA metrics vary heavily
- dropout0/true0 ratio approaches 1 for well-recovered dropout zeros relative to the true count matrix
- t-SNE of the imputed matrix recovers the six simulated cell groups as separable clusters
- MSE between imputed and true counts remains low and stable across hyperparameter settings
- Metrics are computed after PCA to 50 components and t-SNE with KMeans(n_clusters=6, random_state=1), matching the paper's protocol

## Limitations

- Requires simulated data with ground-truth true counts and known group labels; metrics like MSE and ARI against true groups are not computable on real datasets
- Evaluation depends on the simulated Dataset 1 structure (1000 genes, 500 cells, six groups); results may not generalize to other simulation parameters (Supplementary Tables 1-3 not provided in the text)
- Cluster quality metrics depend on dimensionality-reduction choices (PCA to 50 components, t-SNE) which can affect reproducibility

## Evidence

- [abstract] AutoClass is robust on key hyperparameter settings including bottleneck layer size, pre-clustering number and classifier weight: "AutoClass is robust on key hyperparameter settings including bottleneck layer size, pre-clustering number and classifier weight"
- [results] PCA and t-SNE pipeline for embeddings: "PCA_raw = PCA(n_components=50).fit_transform(X_norm)
tsne_raw = TSNE(n_components=2).fit_transform(PCA_raw)"
- [results] AutoClassImpute invocation with dropout_rate: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [results] KMeans clustering with fixed random state: "kmeans = KMeans(n_clusters = K,random_state=1).fit(tsne)"
- [intro] Simulated dataset with ground truth: "we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups"
- [results] Data files used for evaluation: "expr = pd.read_csv('Dataset1_raw.csv',index_col=0)"
