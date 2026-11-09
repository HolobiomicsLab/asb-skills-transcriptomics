---
name: laplace-smoothing-parameter-estimation
description: Use when training a multinomial cell-type classifier on single-cell expression
  data and zero mean expression for some genes would otherwise yield zero likelihood;
  i.e., building a SciBet Learn model from log-normalized counts.
when_to_use_negative:
- The query data contains cell types absent from the training reference — SciBet's
  confidence score C (likelihood ratio vs. a null dataset, cutoffs 0.001–0.5) exists
  precisely to flag such cells, and a plain smoothed multinomial will still force
  a label.
- The expression matrix has not been log-normalized consistently with the training
  data (TPM-log for full-length, Seurat v3 size-factor-10,000 for UMI), since p_ij
  estimation and likelihood comparison assume matched scales.
- Feature selection has not been performed — the article fits parameters on the top-500
  ΔS informative genes, not the full transcriptome.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SciBet
  role: Learn() estimates Laplace-smoothed multinomial p_ij per cell type; Bet() applies
    maximum-likelihood classification; ExportModel/LoadModel provide model portability
  repo: github.com/PaulingLiu/scibet
- name: scikit-learn (sklearn)
  role: model_selection.train_test_split provides the stratified 7:3 training split
    over which the smoothed parameters are fit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:e1509b72726271d64cf58fa1bab6952bf010703b2b85ef0e4029958c9894d76c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciBet
  - scikit-learn
  - scikit-learn (sklearn)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# laplace-smoothing-parameter-estimation

## Summary

Estimate per-cell-type multinomial parameters (p_ij) in SciBet by normalizing the mean expression vector across genes and applying Laplace smoothing (adding one), so that maximum-likelihood classification over the top-500 informative genes never encounters zero probabilities.

## When to use

Use when training a multinomial cell-type classifier on single-cell expression data and zero mean expression for some genes would otherwise yield zero likelihood; i.e., building a SciBet Learn model from log-normalized counts.

## When NOT to use

- The query data contains cell types absent from the training reference — SciBet's confidence score C (likelihood ratio vs. a null dataset, cutoffs 0.001–0.5) exists precisely to flag such cells, and a plain smoothed multinomial will still force a label.
- The expression matrix has not been log-normalized consistently with the training data (TPM-log for full-length, Seurat v3 size-factor-10,000 for UMI), since p_ij estimation and likelihood comparison assume matched scales.
- Feature selection has not been performed — the article fits parameters on the top-500 ΔS informative genes, not the full transcriptome.

## Inputs

- Log-normalized single-cell expression matrix (TPM + log-normalization with pseudocount one for full-length data; Seurat v3 size-factor-10,000 log-normalized UMI counts)
- Cell-type labels for each training cell
- Selected informative-gene set (top 500 genes by ΔS from SelectGene/Entropy E-test)

## Outputs

- Per-cell-type multinomial parameter vectors (Laplace-smoothed p_ij over the 500 informative genes)
- Fitted SciBet model object (gene set + parameters + labels) serializable via ExportModel as scibet_model.rds
- Per-cell maximum-likelihood predicted labels (e.g., scibet_predictions.csv)

## How to apply

After SelectGene/Entropy ranks genes by the ΔS statistic and retains the top 500 informative genes, compute each cell type's mean expression vector over those genes from the training set (e.g., the 70% side of a stratified 7:3 train/test split). Normalize the mean expression vector across genes and apply Laplace Smoothing — adding one — to estimate p_ij for all genes i, which prevents zero-probability genes from zeroing out the multinomial likelihood. The R function Learn(expr, geneset, k, a) performs this estimation, with k=500 genes by default. The resulting per-cell-type parameter vectors are what Bet/SciBet later compares to an unknown cell's profile via maximum likelihood estimation (MLE), equivalent to minimizing Kullback–Leibler divergence between the test cell y and each cell type's mean expression vector. Serialize the fitted parameters, gene set, and labels with ExportModel so the model can be reloaded with LoadModel for portable prediction.

## Related tools

- **SciBet** (Learn() estimates Laplace-smoothed multinomial p_ij per cell type; Bet() applies maximum-likelihood classification; ExportModel/LoadModel provide model portability) — github.com/PaulingLiu/scibet
- **scikit-learn (sklearn)** (model_selection.train_test_split provides the stratified 7:3 training split over which the smoothed parameters are fit)

## Examples

```
model <- Learn(expr_train, geneset, k = 500, a = 1); ExportModel(model)
```

## Evaluation signals

- No p_ij equals zero after estimation — Laplace smoothing must eliminate all zero-probability entries across the 500 informative genes.
- Held-out test cells (30% split, repeated 50 times) are assigned to existing reference cell types with high concordance to known labels, consistent with SciBet's reported high prediction accuracy.
- Prediction on a test cell matches the cell type whose mean expression vector has the minimal KL divergence to that cell — re-deriving labels via KL divergence should reproduce Bet() output.
- Exported model (ExportModel) reloads via LoadModel with identical gene set, parameter vectors, and labels, and yields identical predictions on the same query matrix.

## Limitations

- No version numbers were reported for scikit-learn, Seurat v3, or imbalanced-learn, and no seeds are given for the 50× repeated splits, hampering exact reproduction.
- Significance of the ΔS feature-selection statistic is assessed by a permutation test whose approximation strategy is only described in Supplementary Note 2.4, outside the main text.
- Cells whose types are absent from the reference set will still receive a forced label unless the separate confidence-score (C) filtering step with null-dataset likelihood ratios is applied.

## Evidence

- [methods] Laplace smoothing of multinomial parameters: "pij across all gene i could be estimated by Xij after normalization across genes and Laplace Smoothing"
- [discussion] MLE classification as KL minimization: "we annotate the test cell to such the cell type that its mean expression vector of all marker genes has the minimal Kullback–Leibler divergence to the unknown cell y."
- [discussion] Informative gene selection feeding parameter estimation: "we selected genes with top ΔS as informative genes (500 by default)"
- [readme] Installation of the tool providing Learn/ExportModel: "devtools::install_github("PaulingLiu/scibet")"
