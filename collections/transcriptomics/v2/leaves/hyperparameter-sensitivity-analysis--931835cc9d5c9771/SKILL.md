---
name: hyperparameter-sensitivity-analysis
description: Use when reproducing hyperparameter robustness analyses (Fig. 6, Supplementary
  Figs. 6-8, 10) or when verifying that a deep imputation method's clustering and
  imputation metrics are stable across bottleneck size, pre-clustering number, or
  classifier-weight settings.
when_to_use_negative:
- Input is real scRNA-Seq data without a simulated ground-truth count matrix — MSE
  and dropout0/true0 zero-recovery ratios cannot be computed and the robustness claim
  cannot be validated
- Goal is to evaluate a method other than AutoClass or DCA (e.g., SAVER, scImpute,
  MAGIC) whose hyperparameter grids and loss structures differ and are not covered
  by this paper's grid
- You only need a single default AutoClass run — sweeping the grid is unnecessary
  when hyperparameter stability is not the question
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: AutoClass
  role: Method whose hyperparameters (bottleneck layer size, pre-clustering number,
    classifier weight, loss exponent) are swept via AutoClassImpute to test robustness
  repo: https://github.com/datapplab/AutoClass
- name: DCA
  role: Unstable comparison method run on the same Dataset 1 input to contrast its
    fluctuating metrics with AutoClass's stability
  repo: https://github.com/theislab/dca
- name: TensorFlow >=2.0
  role: Backend for retraining AutoClassImpute across the hyperparameter grid
- name: numpy
  role: Computes MSE and dropout0/true0 zero-recovery ratio against true counts
- name: scikit-learn
  role: Provides PCA (50 components), t-SNE, and KMeans used to derive ASW and ARI
    per setting
- name: matplotlib
  role: Plots metric-versus-hyperparameter curves from the assembled results table
- name: Splatter
  role: Simulated Dataset 1 (1000 genes, 500 cells, six groups) used as the benchmark
    input
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:9cfd24f8f43f328d324a5210ac926bc2992d8912fb53efd72797e5c85b30b982
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# hyperparameter-sensitivity-analysis

## Summary

Systematic robustness test that retrains AutoClass (TensorFlow >=2.0) across a hyperparameter grid — bottleneck layer size, pre-clustering number K, classifier weight, and loss exponent p — on simulated Splatter dropout data, and compares metric stability against DCA.

## When to use

Use when reproducing hyperparameter robustness analyses (Fig. 6, Supplementary Figs. 6-8, 10) or when verifying that a deep imputation method's clustering and imputation metrics are stable across bottleneck size, pre-clustering number, or classifier-weight settings.

## When NOT to use

- Input is real scRNA-Seq data without a simulated ground-truth count matrix — MSE and dropout0/true0 zero-recovery ratios cannot be computed and the robustness claim cannot be validated
- Goal is to evaluate a method other than AutoClass or DCA (e.g., SAVER, scImpute, MAGIC) whose hyperparameter grids and loss structures differ and are not covered by this paper's grid
- You only need a single default AutoClass run — sweeping the grid is unnecessary when hyperparameter stability is not the question

## Inputs

- Dataset1_raw.csv — simulated dropout count matrix from Splatter (1000 genes, 500 cells, six groups)
- Dataset1_true.csv — matching true count matrix for zero-recovery and MSE evaluation
- AutoClass datasets directory files (e.g., Dataset1/truecounts.csv)

## Outputs

- Table of ASW, ARI, MSE, and dropout0/true0 ratio for every hyperparameter setting and both methods
- Matplotlib metric-versus-hyperparameter curves for AutoClass and DCA (reproducing Fig. 6 and Supplementary Figs. 6-8, 10)
- t-SNE embeddings (PCA to 50 components then t-SNE) showing clustering patterns per setting

## How to apply

Load the simulated dropout matrix (Dataset1_raw.csv) and true count matrix (Dataset1_true.csv) with pandas, then retrain AutoClassImpute while sweeping bottleneck size from 16 to 256, pre-clustering number K from 4 to 8, classifier weight w from 0.1 to 0.9, and loss exponent p = 1, 2, 3. Run DCA on the same Dataset 1 input under the same protocol as the unstable comparison method. For each setting, compute ASW and ARI from KMeans clustering (with PCA to 50 components followed by t-SNE via sklearn), plus MSE and the dropout0/true0 zero-recovery ratio against the true counts with numpy. Assemble all metrics into one table and plot metric-versus-hyperparameter curves for AutoClass and DCA with matplotlib. The validation criterion is that AutoClass's t-SNE patterns, ASW, ARI, MSE, and dropout0/true0 ratio stay unchanged across the grid while the same metrics fluctuate heavily for DCA, consistent with the paper's reported robustness.

## Related tools

- **AutoClass** (Method whose hyperparameters (bottleneck layer size, pre-clustering number, classifier weight, loss exponent) are swept via AutoClassImpute to test robustness) — https://github.com/datapplab/AutoClass
- **DCA** (Unstable comparison method run on the same Dataset 1 input to contrast its fluctuating metrics with AutoClass's stability) — https://github.com/theislab/dca
- **TensorFlow >=2.0** (Backend for retraining AutoClassImpute across the hyperparameter grid)
- **numpy** (Computes MSE and dropout0/true0 zero-recovery ratio against true counts)
- **scikit-learn** (Provides PCA (50 components), t-SNE, and KMeans used to derive ASW and ARI per setting)
- **matplotlib** (Plots metric-versus-hyperparameter curves from the assembled results table)
- **Splatter** (Simulated Dataset 1 (1000 genes, 500 cells, six groups) used as the benchmark input)

## Examples

```
expr = pd.read_csv('Dataset1_raw.csv',index_col=0); res = AutoClassImpute(X,dropout_rate=0.3)
```

## Evaluation signals

- AutoClass's ASW, ARI, MSE, and dropout0/true0 ratio remain unchanged as bottleneck size increases from 16 to 256
- AutoClass metrics stay stable over pre-clustering number K = 4-8 and classifier weight w = 0.1-0.9
- DCA's same metrics fluctuate heavily across the identical settings, providing the contrast expected by the paper
- KMeans clustering (on PCA-50 + t-SNE embeddings) recovers the six simulated groups for every AutoClass setting
- Loss exponent p = 1, 2, 3 comparison (Supplementary Fig. 10) shows no material change in AutoClass performance

## Limitations

- Supplementary Fig. 10 (loss-function comparison) and Supplementary Tables 1-3 (hyperparameter settings and simulation parameters) are referenced but were not included in the provided text, so exact grid values may need checking against the SI
- Robustness was demonstrated only on the simulated Splatter Dataset 1 with six groups; stability on other datasets is not established by this analysis
- The comparison's unstable baseline is limited to DCA; other imputation methods were not part of this hyperparameter sweep

## Evidence

- [abstract] AutoClass is robust on key hyperparameter settings including bottleneck layer size, pre-clustering number and classifier weight: "AutoClass is robust on key hyperparameter settings including bottleneck layer size, pre-clustering number and classifier weight"
- [results] Sweep grid and metrics workflow: "expr = pd.read_csv('Dataset1_raw.csv',index_col=0)"
- [discussion] Supplementary Fig. 10 reference: "(p = 1, 2, 3, etc, Supplementary Fig. 10)"
- [methods] Installation dependencies: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed"
- [intro] Simulation setup for Dataset 1: "we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups"
