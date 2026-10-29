---
name: classifier-branch-ablation-comparison
description: Use when you must attribute an imputation/denoising method's performance
  to a specific architecture component and need an ablation control (e.g., AutoClass
  vs. a plain autoencoder) on simulated scRNA-Seq data.
when_to_use_negative:
- The goal is to compare AutoClass against external methods (DCA, SAVER, scImpute,
  MAGIC) rather than test its internal classifier component necessity
- Input data is real experimental scRNA-Seq without a paired dropout-free ground truth
  or known group labels, so no reconstruction-quality reference exists
- You need a distribution-assuming method — this skill's rationale relies on AutoClass
  being distribution agnostic
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: AutoClass
  role: 'Treatment model: autoencoder + classifier whose imputed output (AutoClassImpute,
    dropout_rate=0.3) is evaluated for six-group recovery'
  repo: https://github.com/datapplab/AutoClass
- name: Splatter
  role: Simulates Dataset 1 (1000 genes, 500 cells, six groups) with paired dropout
    and true count matrices
- name: TensorFlow >=2.0
  role: Backend for training the AutoClass model and the classifier-free regular autoencoder
    ablation control
- name: scikit-learn (sklearn)
  role: Provides PCA(n_components=50), TSNE(n_components=2), and silhouette score
    for ASW quantification
- name: matplotlib
  role: Plots the 2-D t-SNE embeddings colored by the six simulated group labels
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:b0da20ac886e6a5fc738a84986a661473e483d06faf2dee5b7c679623a753890
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - Python
  - Splatter
  - TensorFlow >=2.0
  - matplotlib
  - numpy
  - scikit-learn
  - scikit-learn (sklearn)
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# classifier-branch-ablation-comparison

## Summary

An ablation-control skill that tests whether AutoClass's classifier branch (attached to the bottleneck layer) is necessary for scRNA-Seq reconstruction by comparing AutoClass against a regular autoencoder without the classifier on the same Splatter-simulated dropout data.

## When to use

Use when you must attribute an imputation/denoising method's performance to a specific architecture component and need an ablation control (e.g., AutoClass vs. a plain autoencoder) on simulated scRNA-Seq data.

## When NOT to use

- The goal is to compare AutoClass against external methods (DCA, SAVER, scImpute, MAGIC) rather than test its internal classifier component necessity
- Input data is real experimental scRNA-Seq without a paired dropout-free ground truth or known group labels, so no reconstruction-quality reference exists
- You need a distribution-assuming method — this skill's rationale relies on AutoClass being distribution agnostic

## Inputs

- Splatter-simulated Dataset 1: dropout count matrix (Dataset1_raw.csv, 1000 genes x 500 cells in six groups)
- Paired dropout-free true count matrix (Dataset1_true.csv / truecounts.csv)
- Simulated six-group cell labels for coloring t-SNE and computing ASW

## Outputs

- AutoClass-imputed count matrix (dropout_rate=0.3)
- Regular-autoencoder-imputed count matrix (ablation control)
- 2-D t-SNE embeddings (PCA 50 -> t-SNE 2) of raw, AutoClass, regular-autoencoder, and true-count matrices colored by six group labels
- Average silhouette width (ASW) values for the AutoClass and regular-autoencoder t-SNE embeddings

## How to apply

Simulate Dataset 1 with Splatter (1000 genes, 500 cells in six groups) producing a paired dropout count matrix and a dropout-free true count matrix, cross-checked against the repository demo files Dataset1_raw.csv and Dataset1_true.csv. Normalize the dropout matrix with take_norm, reduce to 50 PCA components, and embed with t-SNE (n_components=2) to set the raw-data baseline. Impute with AutoClassImpute(dropout_rate=0.3) built on TensorFlow (>=2.0), then train a regular autoencoder lacking the classifier branch on the same normalized data as the ablation control. Reduce each imputed matrix (plus the true counts) via PCA(50) followed by t-SNE(2), color embeddings by the six simulated group labels with matplotlib, and compute average silhouette width (ASW) on the t-SNE embeddings to quantify recovery.

## Related tools

- **AutoClass** (Treatment model: autoencoder + classifier whose imputed output (AutoClassImpute, dropout_rate=0.3) is evaluated for six-group recovery) — https://github.com/datapplab/AutoClass
- **Splatter** (Simulates Dataset 1 (1000 genes, 500 cells, six groups) with paired dropout and true count matrices)
- **TensorFlow >=2.0** (Backend for training the AutoClass model and the classifier-free regular autoencoder ablation control)
- **scikit-learn (sklearn)** (Provides PCA(n_components=50), TSNE(n_components=2), and silhouette score for ASW quantification)
- **matplotlib** (Plots the 2-D t-SNE embeddings colored by the six simulated group labels)

## Examples

```
expr = pd.read_csv('Dataset1_raw.csv',index_col=0); X = expr.values; res = AutoClassImpute(X, dropout_rate=0.3)
```

## Evaluation signals

- AutoClass t-SNE embedding shows six visually separated groups matching the simulated Splatter group labels
- ASW on the AutoClass t-SNE embedding is higher than ASW on the regular-autoencoder t-SNE embedding
- Raw-data (pre-imputation) t-SNE baseline shows poor group separation relative to both imputed outputs
- Imputed matrices reproduce structure consistent with the dropout-free true count matrix (Dataset1_true.csv)
- Both models trained on the same take_norm-normalized input so the only difference is the classifier branch

## Limitations

- Result established on Splatter-simulated data; transferring the conclusion to real datasets relies on the simulation's fidelity
- No code or data availability statement appears within the discussion text itself; the AutoClass repository is only referenced externally
- No software version numbers beyond TensorFlow (>=2.0) are stated, so exact reproducibility of training runs is not guaranteed
- Supplementary Tables 1–3 (hyperparameter settings and simulation parameters) are not included in the provided text, leaving exact simulation settings partially unspecified

## Evidence

- [intro] Classifier necessity finding: "AutoClass but not the regular autoencoder was able to recover cell type pattern, indicating the classifier component is necessary for reconstructing scRNA-Seq data."
- [intro] Simulation design: "we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups"
- [methods] Software requirements: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed"
- [results] AutoClass imputation call: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [results] Dimensionality reduction baseline: "X_norm = take_norm(X)"
