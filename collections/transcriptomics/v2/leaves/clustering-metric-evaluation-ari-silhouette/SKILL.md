---
name: clustering-metric-evaluation-ari-silhouette
description: Use when imputed or denoised scRNA-Seq count matrices must be scored
  for cell-type recovery against known simulated groups (e.g., the six Splatter-simulated
  groups of Dataset 1) using ASW, ARI, MSE, and zero-recovery metrics.
when_to_use_negative:
- The dataset has no ground-truth group labels or true count matrix — ASW/ARI against
  'true' clusters and the dropout0/true0 zero-recovery ratio cannot be computed; use
  qualitative metrics only.
- Raw (unimputed) counts are the only input and the goal is a publication t-SNE alone
  — this skill is designed to score imputation/denoising results across settings,
  not just produce one embedding.
- Cells come from real, unlabeled tissue where cell identity is unknown — the six
  simulated groups of Dataset 1 define the cluster count and ARI target, which do
  not exist here.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3170
tools:
- name: AutoClass
  role: Imputation/denoising method whose outputs (e.g., AutoClassImpute(X, dropout_rate=0.3)
    on Dataset 1) are scored with ASW/ARI across hyperparameter sweeps
  repo: https://github.com/datapplab/AutoClass
- name: DCA
  role: Unstable comparison method run on Dataset 1 under the same evaluation protocol
    to contrast its fluctuating metrics against AutoClass stability
  repo: https://github.com/theislab/dca
- name: scikit-learn
  role: Provides PCA (n_components=50), TSNE (n_components=2), and KMeans (random_state=1)
    used to derive the clustering evaluated by ASW and ARI
- name: numpy
  role: Computes MSE and the dropout0/true0 zero-recovery ratio against the true counts
- name: Splatter
  role: Generated the simulated Dataset 1 (1000 genes, 500 cells, six groups) whose
    known group labels serve as ARI ground truth
  repo: https://github.com/LuyiTian/sc_mixology
- name: matplotlib
  role: Plots metric-versus-hyperparameter curves assembling the robustness figure
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:6ead792fecc443012d0d6f9bb6a9003a4f7d5f69117ff47c02adf42d40eb533c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - DCA
  - Python
  - Splatter
  - matplotlib
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# clustering-metric-evaluation-ari-silhouette

## Summary

Evaluates clustering quality of scRNA-Seq cell embeddings by running KMeans on PCA-to-50-components + t-SNE embeddings and computing ASW (average silhouette width) and ARI (adjusted Rand index) against known simulated cell groups, alongside MSE and dropout0/true0 zero-recovery ratios against true counts. It underpins the paper's demonstrations that AutoClass, unlike DCA, stays stable under hyperparameter sweeps.

## When to use

Use when imputed or denoised scRNA-Seq count matrices must be scored for cell-type recovery against known simulated groups (e.g., the six Splatter-simulated groups of Dataset 1) using ASW, ARI, MSE, and zero-recovery metrics.

## When NOT to use

- The dataset has no ground-truth group labels or true count matrix — ASW/ARI against 'true' clusters and the dropout0/true0 zero-recovery ratio cannot be computed; use qualitative metrics only.
- Raw (unimputed) counts are the only input and the goal is a publication t-SNE alone — this skill is designed to score imputation/denoising results across settings, not just produce one embedding.
- Cells come from real, unlabeled tissue where cell identity is unknown — the six simulated groups of Dataset 1 define the cluster count and ARI target, which do not exist here.

## Inputs

- Dataset1_raw.csv — simulated dropout count matrix (pandas read_csv, index_col=0)
- Dataset1_true.csv — true count matrix used as ground truth for MSE and dropout0/true0 ratio
- Known cell group labels for the six simulated groups (from Splatter simulation of Dataset 1)

## Outputs

- Per-setting table of ASW, ARI, MSE, and dropout0/true0 ratio (numpy/sklearn)
- KMeans cluster assignments on the t-SNE embedding (KMeans(n_clusters=K, random_state=1))
- Matplotlib metric-versus-hyperparameter curves comparing AutoClass and DCA (t-SNE patterns, Supplementary Figs. 6-8, 10)

## How to apply

Load the simulated dropout matrix (Dataset1_raw.csv) and the ground-truth counts (Dataset1_true.csv) with pandas. Normalize the raw counts with take_norm, reduce with PCA to 50 components, embed to 2 dimensions with t-SNE, and run KMeans with random_state=1 on the embedding. Compute ASW and ARI from the KMeans assignments against the six simulated groups (K=6 for Dataset 1, or K=4-8 when testing pre-clustering-number robustness). To assess expression recovery, compute MSE between imputed and true counts and the dropout0/true0 ratio (fraction of zero-dropout entries that remain zero after imputation). Assemble all metrics into a single table and plot metric-versus-hyperparameter curves to compare methods such as AutoClass and DCA; a correct application shows AutoClass's ASW, ARI, MSE, and dropout0/true0 ratio stable across bottleneck sizes 16-256, K=4-8, and classifier weights, while the same metrics fluctuate for DCA.

## Related tools

- **AutoClass** (Imputation/denoising method whose outputs (e.g., AutoClassImpute(X, dropout_rate=0.3) on Dataset 1) are scored with ASW/ARI across hyperparameter sweeps) — https://github.com/datapplab/AutoClass
- **DCA** (Unstable comparison method run on Dataset 1 under the same evaluation protocol to contrast its fluctuating metrics against AutoClass stability) — https://github.com/theislab/dca
- **scikit-learn** (Provides PCA (n_components=50), TSNE (n_components=2), and KMeans (random_state=1) used to derive the clustering evaluated by ASW and ARI)
- **numpy** (Computes MSE and the dropout0/true0 zero-recovery ratio against the true counts)
- **Splatter** (Generated the simulated Dataset 1 (1000 genes, 500 cells, six groups) whose known group labels serve as ARI ground truth) — https://github.com/LuyiTian/sc_mixology
- **matplotlib** (Plots metric-versus-hyperparameter curves assembling the robustness figure)

## Examples

```
expr = pd.read_csv('Dataset1_raw.csv', index_col=0); res = AutoClassImpute(expr.values, dropout_rate=0.3); X_norm = take_norm(res); tsne = TSNE(n_components=2).fit_transform(PCA(n_components=50).fit_transform(X_norm)); kmeans = KMeans(n_clusters=6, random_state=1).fit(tsne)
```

## Evaluation signals

- AutoClass's ASW, ARI, MSE, and dropout0/true0 ratio remain unchanged as bottleneck size varies 16→256 and pre-clustering K varies 4→8, matching the paper's robustness finding.
- The same metrics for DCA fluctuate heavily across the same settings, serving as the expected negative control pattern.
- KMeans is fit with n_clusters equal to the number of simulated groups (K=6 for Dataset 1) and random_state=1 for reproducibility.
- The dropout0/true0 ratio quantifies how many original zero-dropout entries remain zero after imputation — values drifting from the true-count structure indicate over-imputation artifacts.
- The dimensionality-reduction chain matches the paper exactly: take_norm normalization → PCA(n_components=50) → TSNE(n_components=2) before clustering.

## Limitations

- Supplementary Tables 1–3 with the exact hyperparameter and simulation settings are referenced but not reproduced in the discussion text, so exact grid values must be taken from the supplementary material.
- The evaluation depends on Splatter-simulated ground truth (Dataset 1); ASW/ARI and zero-recovery conclusions do not directly transfer to real datasets without known groups.
- KMeans on a 2-D t-SNE embedding is used for the metric computation; t-SNE is stochastic and results depend on the random state and embedding choice.

## Evidence

- [results] AutoClass results (t-SNE patterns, ASW, ARI, MSE, and dropout0/true0 ratio) stayed unchanged as bottleneck size increased from 16 to 256 and over K = 4-8, whereas the same metrics varied heavily for DCA.: [evidence span withheld — anchor status: not_found]
- [results] PCA to 50 components followed by t-SNE to 2 dimensions: "PCA_raw = PCA(n_components=50).fit_transform(X_norm)
tsne_raw = TSNE(n_components=2).fit_transform(PCA_raw)"
- [results] KMeans clustering on t-SNE embedding for cluster quality metrics: "kmeans = KMeans(n_clusters = K,random_state=1).fit(tsne)"
- [intro] Splatter simulation of Dataset 1 with 1000 genes, 500 cells, six groups provides the ARI ground-truth labels: "we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups"
- [abstract] Robustness finding on hyperparameters stated in the abstract: "AutoClass is robust on key hyperparameter settings including bottleneck layer size, pre-clustering number and classifier weight"
