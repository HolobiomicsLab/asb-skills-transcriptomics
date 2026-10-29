---
name: simulation-based-benchmark-evaluation-scrnaseq
description: Use when benchmarking an scRNA-Seq imputation/denoising method (e.g.
  AutoClass vs DCA) against a known ground truth, or when testing robustness to hyperparameters
  such as bottleneck size, pre-clustering number K, or classifier weight.
when_to_use_negative:
- Input is real scRNA-Seq data without simulated ground truth — ASW/ARI against known
  groups and MSE/dropout0/true0 against true counts require true labels and true expression
  values
- The goal is biological discovery (e.g. differential expression on a real dataset
  like Baron) rather than method benchmarking or robustness testing
- The candidate method cannot be retrained programmatically across a hyperparameter
  grid (no Python/TensorFlow >=2.0 interface)
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3577
tools:
- name: AutoClass
  role: Method under test; imputes simulated dropout data via AutoClassImpute and
    is swept over bottleneck size, pre-clustering number, and classifier weight
  repo: https://github.com/datapplab/AutoClass
- name: DCA
  role: Unstable comparison method run on the same Dataset 1 input under the same
    evaluation protocol
  repo: https://github.com/theislab/dca
- name: Splatter
  role: Simulation tool that generated Dataset 1 with 1000 genes and 500 cells in
    six groups, providing the ground truth
- name: TensorFlow >=2.0
  role: Backend for training AutoClassImpute at each hyperparameter setting
- name: scikit-learn
  role: PCA (50 components), t-SNE (2 components), and KMeans clustering for computing
    ASW/ARI
- name: matplotlib
  role: Plots metric-versus-hyperparameter curves for AutoClass and DCA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:e1eb6a2c178b0265e8ccecebed1a3c443e97e2f42cedb97965b2a1698d248c40
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
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# simulation-based-benchmark-evaluation-scrnaseq

## Summary

Evaluate scRNA-Seq denoising/imputation methods on simulated ground-truth data (Splatter Dataset 1: 1000 genes, 500 cells, six groups) by sweeping method hyperparameters and scoring clustering (ASW, ARI), reconstruction (MSE), and zero-recovery (dropout0/true0 ratio) metrics against true counts.

## When to use

Use when benchmarking an scRNA-Seq imputation/denoising method (e.g. AutoClass vs DCA) against a known ground truth, or when testing robustness to hyperparameters such as bottleneck size, pre-clustering number K, or classifier weight.

## When NOT to use

- Input is real scRNA-Seq data without simulated ground truth — ASW/ARI against known groups and MSE/dropout0/true0 against true counts require true labels and true expression values
- The goal is biological discovery (e.g. differential expression on a real dataset like Baron) rather than method benchmarking or robustness testing
- The candidate method cannot be retrained programmatically across a hyperparameter grid (no Python/TensorFlow >=2.0 interface)

## Inputs

- Simulated dropout count matrix Dataset1_raw.csv (Splatter, 1000 genes x 500 cells, six groups)
- Ground-truth count matrix Dataset1_true.csv
- Simulated group labels for the six cell groups
- Hyperparameter grid specification (bottleneck size 16-256, K = 4-8, classifier weight 0.1-0.9, loss exponent p = 1, 2, 3)

## Outputs

- Table of ASW, ARI, MSE, and dropout0/true0 ratio per method per hyperparameter setting
- PCA-to-50-components + t-SNE 2D embeddings per imputed matrix
- KMeans cluster assignments (n_clusters = K, random_state=1) per setting
- Metric-versus-hyperparameter curves comparing AutoClass and DCA (matplotlib)

## How to apply

Load the simulated dropout matrix (Dataset1_raw.csv) and the ground-truth matrix (Dataset1_true.csv) with pandas; these were generated with Splatter using 1000 genes and 500 cells in six groups. Impute the raw counts with the candidate method (AutoClassImpute with dropout_rate=0.3, TensorFlow >=2.0) and run the comparison method (DCA) under the same protocol. For each imputed output, normalize with take_norm, reduce with PCA to 50 components followed by t-SNE to 2 dimensions, and run KMeans (random_state=1) to compute ASW and ARI against the six simulated groups; additionally compute MSE against true counts and the dropout0/true0 zero-recovery ratio with numpy. Sweep the hyperparameter grid — bottleneck sizes 16-256, pre-clustering K = 4-8, classifier weight 0.1-0.9, loss exponent p = 1, 2, 3 — evaluating every setting identically. Rationale: a robust method keeps t-SNE patterns, ASW, ARI, MSE, and dropout0/true0 ratio stable across settings, while a distribution-assuming method like DCA fluctuates heavily; assemble metrics into a table and plot metric-versus-hyperparameter curves with matplotlib.

## Related tools

- **AutoClass** (Method under test; imputes simulated dropout data via AutoClassImpute and is swept over bottleneck size, pre-clustering number, and classifier weight) — https://github.com/datapplab/AutoClass
- **DCA** (Unstable comparison method run on the same Dataset 1 input under the same evaluation protocol) — https://github.com/theislab/dca
- **Splatter** (Simulation tool that generated Dataset 1 with 1000 genes and 500 cells in six groups, providing the ground truth)
- **TensorFlow >=2.0** (Backend for training AutoClassImpute at each hyperparameter setting)
- **scikit-learn** (PCA (50 components), t-SNE (2 components), and KMeans clustering for computing ASW/ARI)
- **matplotlib** (Plots metric-versus-hyperparameter curves for AutoClass and DCA)

## Examples

```
expr = pd.read_csv('Dataset1_raw.csv',index_col=0); res = AutoClassImpute(expr.values, dropout_rate=0.3)
```

## Evaluation signals

- AutoClass's ASW, ARI, MSE, and dropout0/true0 ratio stay essentially unchanged across bottleneck sizes 16-256 and K = 4-8, matching the paper's robustness claim
- The same metrics vary heavily for DCA across the identical grid, confirming the comparison detects sensitivity
- KMeans on the t-SNE embedding recovers the six simulated groups (set n_clusters to the six-group structure, random_state=1)
- dropout0/true0 ratio uses zero-recovery against Dataset1_true.csv zeros; values near 1 indicate faithful zero recovery
- All settings are evaluated with the identical pipeline (take_norm -> PCA(50) -> t-SNE(2) -> KMeans) so metric differences are attributable to the method/hyperparameter, not the protocol

## Limitations

- Supplementary Tables 1-3 (hyperparameter settings and simulation parameters) are referenced in the discussion but not included in the provided text, so exact grid values rely on the source card
- Supplementary Fig. 10 (loss-function comparison, p = 1, 2, 3) is referenced but the underlying per-setting numbers are not in the provided text
- Findings are grounded in one simulated dataset (Splatter Dataset 1, six groups); robustness on other simulation structures must be re-established
- The paper's discussion contains no limitations section; sensitivity of the metrics to t-SNE stochasticity is not discussed

## Evidence

- [intro] Splatter Dataset 1 simulation provides the ground truth for the sweep: "we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups"
- [abstract] Robustness claim being verified by the evaluation: "AutoClass is robust on key hyperparameter settings including bottleneck layer size, pre-clustering number and classifier weight"
- [results] Standard evaluation embedding pipeline applied to each imputed matrix: "PCA_raw = PCA(n_components=50).fit_transform(X_norm)
tsne_raw = TSNE(n_components=2).fit_transform(PCA_raw)"
- [results] KMeans clustering used to derive cluster-quality metrics per setting: "kmeans = KMeans(n_clusters = K,random_state=1).fit(tsne)"
- [results] Dropout and true count matrices are the named inputs: "expr = pd.read_csv('Dataset1_raw.csv',index_col=0)"
- [methods] Runtime environment required to retrain AutoClass across the grid: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed"
