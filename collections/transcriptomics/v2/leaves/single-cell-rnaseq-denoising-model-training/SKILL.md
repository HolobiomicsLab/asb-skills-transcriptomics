---
name: single-cell-rnaseq-denoising-model-training
description: Use when you need to demonstrate or verify that an scRNA-Seq denoising/imputation
  model's results (t-SNE patterns, ASW, ARI, MSE, zero-recovery) are insensitive to
  hyperparameter choices, benchmarked against DCA.
when_to_use_negative:
- Input is real scRNA-Seq data without ground-truth group labels — the ARI/ASW evaluation
  relies on the six simulated Splatter groups of Dataset 1
- The matrix is already denoised/imputed — retraining imputation models on imputed
  output double-denoises and invalidates the zero-recovery comparison
- The comparison method is not distribution-assumption-based (e.g., DCA) — the paper's
  robustness contrast specifically pits AutoClass against DCA, which fluctuates under
  the same protocol
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: AutoClass
  role: Model whose hyperparameters (bottleneck size, pre-clustering K, classifier
    weight, loss exponent p) are swept via AutoClassImpute retraining
  repo: https://github.com/datapplab/AutoClass
- name: DCA
  role: Unstable comparison method run on the same Dataset 1 input under the same
    evaluation protocol
  repo: https://github.com/theislab/dca
- name: Splatter
  role: Simulation tool that generated Dataset 1 (1000 genes, 500 cells, six groups)
    used for the sweep
- name: TensorFlow >=2.0
  role: Backend for retraining the AutoClass autoencoder-plus-classifier network across
    the hyperparameter grid
- name: scikit-learn
  role: PCA (50 components), t-SNE, and KMeans used to derive ASW and ARI clustering
    metrics per setting
- name: numpy
  role: Computes MSE and the dropout0/true0 zero-recovery ratio against the true count
    matrix
- name: matplotlib
  role: Plots metric-versus-hyperparameter curves for AutoClass and DCA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:228f2a721bc7dec085d0b7c74e57a333e4a6b14cc56bbbf95d54d9bbe43b38f0
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# single-cell-rnaseq-denoising-model-training

## Summary

Sweeps AutoClass hyperparameters (bottleneck size 16-256, pre-clustering K=4-8, classifier weight w=0.1-0.9, loss exponent p=1-3) on simulated Splatter data (Dataset 1) and evaluates stability of ASW, ARI, MSE, and dropout0/true0 zero-recovery ratio against the unstable comparison method DCA (Fig. 6, Supplementary Figs. 6-8, 10).

## When to use

Use when you need to demonstrate or verify that an scRNA-Seq denoising/imputation model's results (t-SNE patterns, ASW, ARI, MSE, zero-recovery) are insensitive to hyperparameter choices, benchmarked against DCA.

## When NOT to use

- Input is real scRNA-Seq data without ground-truth group labels — the ARI/ASW evaluation relies on the six simulated Splatter groups of Dataset 1
- The matrix is already denoised/imputed — retraining imputation models on imputed output double-denoises and invalidates the zero-recovery comparison
- The comparison method is not distribution-assumption-based (e.g., DCA) — the paper's robustness contrast specifically pits AutoClass against DCA, which fluctuates under the same protocol

## Inputs

- Dataset1_raw.csv (simulated dropout count matrix, Splatter-generated, 1000 genes x 500 cells in six groups)
- Dataset1_true.csv / truecounts.csv (true underlying count matrix for MSE and zero-recovery ratio)
- Six simulated cell-group labels for clustering evaluation (ARI/purity)
- Hyperparameter grid: bottleneck size 16-256, pre-clustering K = 4-8, classifier weight w = 0.1-0.9, loss exponent p = 1, 2, 3

## Outputs

- Table of ASW, ARI, MSE, and dropout0/true0 ratio for each AutoClass and DCA hyperparameter setting
- Matplotlib metric-versus-hyperparameter curves comparing AutoClass and DCA (reproducing Fig. 6, Supplementary Figs. 6-8, 10)
- t-SNE embeddings from PCA (50 components) for each setting

## How to apply

Load the simulated dropout matrix (Dataset1_raw.csv) and true counts (Dataset1_true.csv) with pandas, then retrain AutoClassImpute (TensorFlow >=2.0) across a grid of bottleneck sizes 16-256, pre-clustering K = 4-8, classifier weight w = 0.1-0.9, and loss exponent p = 1, 2, 3. Run DCA on the same Dataset 1 input under an identical evaluation protocol. For each setting, normalize with take_norm, project via PCA to 50 components then t-SNE to 2 dimensions, cluster with KMeans against the six simulated Splatter groups, and compute ASW/ARI plus MSE and the dropout0/true0 zero-recovery ratio against the true counts with numpy. Assemble all metrics into a single table and plot metric-versus-hyperparameter curves for both methods with matplotlib. Validation succeeds when AutoClass metrics stay flat across the grid while DCA metrics fluctuate, consistent with the paper's reported robustness.

## Related tools

- **AutoClass** (Model whose hyperparameters (bottleneck size, pre-clustering K, classifier weight, loss exponent p) are swept via AutoClassImpute retraining) — https://github.com/datapplab/AutoClass
- **DCA** (Unstable comparison method run on the same Dataset 1 input under the same evaluation protocol) — https://github.com/theislab/dca
- **Splatter** (Simulation tool that generated Dataset 1 (1000 genes, 500 cells, six groups) used for the sweep)
- **TensorFlow >=2.0** (Backend for retraining the AutoClass autoencoder-plus-classifier network across the hyperparameter grid)
- **scikit-learn** (PCA (50 components), t-SNE, and KMeans used to derive ASW and ARI clustering metrics per setting)
- **numpy** (Computes MSE and the dropout0/true0 zero-recovery ratio against the true count matrix)
- **matplotlib** (Plots metric-versus-hyperparameter curves for AutoClass and DCA)

## Examples

```
expr = pd.read_csv('Dataset1_raw.csv',index_col=0); res = AutoClassImpute(expr.values, dropout_rate=0.3)
```

## Evaluation signals

- AutoClass's ASW, ARI, MSE, and dropout0/true0 ratio remain essentially unchanged as bottleneck size increases from 16 to 256 and over K = 4-8
- DCA metrics vary heavily across the same hyperparameter grid, providing the expected contrast
- t-SNE patterns from PCA(50)+t-SNE preserve the six simulated cell groups for every AutoClass setting
- MSE and dropout0/true0 ratio are computed against Dataset1_true.csv, so zero-recovery ratio near the true-count expectation indicates correct recovery
- All metrics assemble into a single table whose metric-versus-hyperparameter curves match Fig. 6 and Supplementary Figs. 6-8

## Limitations

- Supplementary Tables 1-3 (hyperparameter settings and simulation parameters) and Supplementary Fig. 10 (loss-function comparison, p = 1, 2, 3) are referenced but their full contents are not provided in the article text
- The paper's discussion reports no limitations of AutoClass itself, so failure modes must be probed empirically rather than taken from the text
- No software version numbers are stated in the article, so exact reproducibility of metric curves may depend on library versions

## Evidence

- [abstract] AutoClass is robust on key hyperparameter settings including bottleneck layer size, pre-clustering number and classifier weight: "AutoClass is robust on key hyperparameter settings including bottleneck layer size, pre-clustering number and classifier weight"
- [methods] Python/TensorFlow requirement for retraining: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed"
- [results] Loading the simulated dropout matrix: "expr = pd.read_csv('Dataset1_raw.csv',index_col=0)"
- [results] PCA+t-SNE projection used before clustering metrics: "PCA_raw = PCA(n_components=50).fit_transform(X_norm)"
- [results] KMeans clustering on t-SNE embedding: "kmeans = KMeans(n_clusters = K,random_state=1).fit(tsne)"
- [readme] AutoClass architecture rationale from README: "AutoClass integrates two deep neural network components, an autoencoder and a classifier, as to maximize both noise removal and signal retention."
