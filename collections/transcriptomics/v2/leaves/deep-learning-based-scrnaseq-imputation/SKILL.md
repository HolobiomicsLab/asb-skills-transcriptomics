---
name: deep-learning-based-scrnaseq-imputation
description: Use when a raw scRNA-Seq count matrix (CSV, genes x cells) exhibits dropouts/noise
  and the goal is imputation or improved downstream clustering, with a benchmark against
  DCA, SAVER, scImpute, or MAGIC.
when_to_use_negative:
- The data are already imputed/denoised or are bulk RNA-Seq rather than single-cell
  count matrices with dropouts
- The analysis goal is differential expression or batch-effect-specific correction
  rather than imputation-benchmarked-by-clustering — AutoClass covers these, but the
  K-means ARI/JI/NMI/purity evaluation scheme here is clustering-specific
- TensorFlow >=2.0 and Python 3 cannot be installed in the environment, since AutoClass
  requires them
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: AutoClass
  role: 'Primary imputation model: AutoClassImpute(X, dropout_rate=0.3) integrates
    an autoencoder and a classifier to clean the scRNA-Seq matrix'
  repo: https://github.com/datapplab/AutoClass
- name: DCA
  role: Control imputation method (deep count autoencoder with zero-inflated negative
    binomial loss) used for benchmarking against AutoClass
  repo: https://github.com/theislab/dca
- name: SAVER
  role: Control imputation method (negative-binomial regularized regression with empirical
    Bayes) used for benchmarking
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Control imputation method (Gaussian-Gamma mixture model) used for benchmarking
  repo: https://github.com/Vivianstats/scImpute
- name: MAGIC
  role: Control imputation/diffusion method used for benchmarking
  repo: https://github.com/KrishnaswamyLab/MAGIC
- name: TensorFlow >=2.0
  role: Deep learning backend required to run the AutoClass module
- name: sklearn
  role: Provides KMeans (n_clusters=K, random_state=1) and clustering metrics for
    the ARI/JI/NMI/purity evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:2f23b68b3764e9e83827010b21a565c5675c875e355d9c844bea78893b8a51e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - DCA
  - MAGIC
  - Python
  - SAVER
  - TensorFlow >=2.0
  - numpy
  - pandas
  - scImpute
  - sklearn
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# deep-learning-based-scrnaseq-imputation

## Summary

Impute and denoise single-cell RNA-Seq count matrices with AutoClass, a TensorFlow >=2.0 deep neural network that integrates an autoencoder and a classifier and is free of distribution assumptions, and benchmark the imputation against controls (DCA, SAVER, scImpute, MAGIC) using downstream K-means clustering metrics (ARI, JI, NMI, purity).

## When to use

Use when a raw scRNA-Seq count matrix (CSV, genes x cells) exhibits dropouts/noise and the goal is imputation or improved downstream clustering, with a benchmark against DCA, SAVER, scImpute, or MAGIC.

## When NOT to use

- The data are already imputed/denoised or are bulk RNA-Seq rather than single-cell count matrices with dropouts
- The analysis goal is differential expression or batch-effect-specific correction rather than imputation-benchmarked-by-clustering — AutoClass covers these, but the K-means ARI/JI/NMI/purity evaluation scheme here is clustering-specific
- TensorFlow >=2.0 and Python 3 cannot be installed in the environment, since AutoClass requires them

## Inputs

- Raw scRNA-Seq count matrix as CSV (genes x cells), e.g., Dataset1_raw.csv, Buettner.csv, loaded with pd.read_csv(..., index_col=0)
- Known true cell-type labels (e.g., Dataset1_true.csv) for clustering evaluation
- True cluster number K for KMeans

## Outputs

- Imputed expression matrix returned by AutoClassImpute (res)
- Clustering evaluation metrics table (datasets x ARI/JI/NMI/purity, e.g., clustering_metrics_table.csv with 16 rows for 4 datasets x 4 metrics)
- PCA(50) + t-SNE(2) embeddings of raw and imputed data for cluster visualization

## How to apply

Clone AutoClass from github.com/datapplab/AutoClass and run under Python 3 with TensorFlow >=2.0 plus numpy, pandas, sklearn, and matplotlib. Load the count matrix with pandas (e.g., pd.read_csv with index_col=0), optionally normalize with take_norm, and impute with AutoClassImpute(X, dropout_rate=0.3). For fair benchmarking, impute the same matrix with control methods DCA, SAVER, scImpute, and MAGIC, then run KMeans with the true cluster number K (random_state=1) on raw versus imputed data. Rationale: AutoClass's classifier branch (not just the autoencoder) is what recovers cell-type pattern, and it is distribution agnostic, avoiding artifacts from wrong distribution assumptions in DCA (zero-inflated negative binomial), SAVER (negative binomial), and scImpute (Gaussian-Gamma mixture). Evaluate clustering with ARI, JI, NMI, and purity score computed via a contingency-matrix function, on highly variable gene subsets (e.g., top-200 via find_hv_genes; the Table 1 task uses top-1000 HVGs).

## Related tools

- **AutoClass** (Primary imputation model: AutoClassImpute(X, dropout_rate=0.3) integrates an autoencoder and a classifier to clean the scRNA-Seq matrix) — https://github.com/datapplab/AutoClass
- **DCA** (Control imputation method (deep count autoencoder with zero-inflated negative binomial loss) used for benchmarking against AutoClass) — https://github.com/theislab/dca
- **SAVER** (Control imputation method (negative-binomial regularized regression with empirical Bayes) used for benchmarking) — https://github.com/mohuangx/SAVER
- **scImpute** (Control imputation method (Gaussian-Gamma mixture model) used for benchmarking) — https://github.com/Vivianstats/scImpute
- **MAGIC** (Control imputation/diffusion method used for benchmarking) — https://github.com/KrishnaswamyLab/MAGIC
- **TensorFlow >=2.0** (Deep learning backend required to run the AutoClass module)
- **sklearn** (Provides KMeans (n_clusters=K, random_state=1) and clustering metrics for the ARI/JI/NMI/purity evaluation)

## Examples

```
git clone https://github.com/datapplab/AutoClass; res = AutoClassImpute(X, dropout_rate=0.3)
```

## Evaluation signals

- AutoClass is the only method improving all four clustering metrics (ARI, JI, NMI, purity) over the raw data, and it achieves the best results for 3 of 4 real datasets (Buettner, Usoskin, Lake, Zeisel)
- Spot-check numeric targets from Table 1, e.g., Usoskin ARI = 0.869 for AutoClass vs 0.221 on raw data
- The KMeans run must use the true cluster number K with random_state=1, and metrics must compare KMeans labels to the true cell-type labels
- Purity must be computed from the contingency matrix between y_true and y_pred, not from a different formula
- The final table has 4 datasets x 4 metrics (16 rows) covering raw plus all control imputation methods (DCA, SAVER, scImpute, MAGIC)

## Limitations

- Supplementary Tables 1–3 (hyperparameter settings and simulation parameters) and Supplementary Fig. 10 (loss-function comparison) are referenced but not included in the provided text, so exact hyperparameter sensitivity details are not reproducible from this context alone
- Existing imputation methods with inaccurate distribution assumptions may not denoise properly but rather introduce new complexities and artifacts — a caveat when interpreting DCA/SAVER/scImpute benchmark results
- No explicit limitations section appears in the discussion; reported improvements are entirely affirmative, so independent validation on new datasets is advisable
- Performance varies by dataset: AutoClass was best on 3 of 4 real datasets, not all 4

## Evidence

- [abstract] AutoClass is distribution agnostic as it makes no assumption on specific data distributions: "AutoClass is distribution agnostic as it makes no assumption on specific data distributions, hence can effectively clean a wide range of noise and artifacts"
- [abstract] AutoClass integrates two deep neural network components, an autoencoder and a classifier: "Here we design and develop the AutoClass model, integrating two deep neural network components, an autoencoder, and a classifier"
- [methods] AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed first"
- [methods] git clone https://github.com/datapplab/AutoClass: "git clone https://github.com/datapplab/AutoClass"
- [results] Impute data with AutoClassImpute using dropout_rate=0.3: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [results] KMeans clustering with random_state=1 on t-SNE embedding: "kmeans = KMeans(n_clusters = K,random_state=1).fit(tsne)"
- [intro] The classifier component is necessary for reconstructing scRNA-Seq data: "AutoClass but not the regular autoencoder was able to recover cell type pattern, indicating the classifier component is necessary for reconstructing scRNA-Seq data."
- [intro] DCA assumes zero-inflated negative binomial; SAVER negative binomial; scImpute Gaussian-Gamma mixture — distribution-assuming controls: "deep count autoencoder (DCA)9 assumes negative binomial distribution with or without zero inflation"
