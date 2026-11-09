---
name: kullback-leibler-divergence-annotation
description: Use when annotating log-normalized single-cell RNA-seq profiles against
  a labeled reference and you need a fast, portable multinomial MLE classifier (top-k
  entropy-selected genes, KL-divergence decision rule).
when_to_use_negative:
- The query contains cell types not represented in the reference without a null dataset
  available — SciBet's low false positive rate depends on the null-dataset alternate
  reference.
- The expression matrix is raw unnormalized counts — the multinomial parameters assume
  TPM/log or Seurat v3 (size-factor 10,000) log-normalized values.
- You need unsupervised cluster annotation from scratch — SciBet is a supervised classifier
  requiring annotated labels.
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SciBet
  role: Implements SelectGene/Entropy gene selection, Learn (Laplace-smoothed multinomial
    fitting), and Bet classification by minimum KL divergence; ExportModel/LoadModel
    for portability
  repo: https://github.com/PaulingLiu/scibet
- name: scikit-learn (sklearn)
  role: model_selection.train_test_split for the stratified 7:3 hold-out split used
    to validate the classifier
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:641be4aa4bc583163033ef823500e6ca53860e3413b1ce537a7ed7aafbea58f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/scibet
  tools:
  - SciBet
  - scikit-learn
  - scikit-learn (sklearn)
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# kullback-leibler-divergence-annotation

## Summary

SciBet-style supervised cell type annotation: each cell type is a multinomial model over informative genes, and an unknown cell is assigned the label whose mean expression vector minimizes Kullback–Leibler divergence to the cell's RNA profile via maximum likelihood estimation.

## When to use

Use when annotating log-normalized single-cell RNA-seq profiles against a labeled reference and you need a fast, portable multinomial MLE classifier (top-k entropy-selected genes, KL-divergence decision rule).

## When NOT to use

- The query contains cell types not represented in the reference without a null dataset available — SciBet's low false positive rate depends on the null-dataset alternate reference.
- The expression matrix is raw unnormalized counts — the multinomial parameters assume TPM/log or Seurat v3 (size-factor 10,000) log-normalized values.
- You need unsupervised cluster annotation from scratch — SciBet is a supervised classifier requiring annotated labels.

## Inputs

- Log-normalized single-cell expression matrix (TPM + log for full-length reads; Seurat v3 size-factor 10,000 + log for UMI counts)
- Cell-type labels for the training/reference cells
- Informative gene set selected by ΔS ranking (top 500 by default)
- Serialized SciBet model (gene set, cell-type parameter vectors, labels) from ExportModel

## Outputs

- Per-cell predicted cell type labels (e.g. scibet_predictions.csv)
- Optional confidence scores C (training-set vs null-dataset likelihood ratio) per cell
- Portable model file (e.g. scibet_model.rds) loadable via LoadModel

## How to apply

Normalize the expression matrix first — TPM plus pseudo-value 1 and log normalization for full-length data, or size-factor 10,000 with add-one log normalization for UMI data. Rank genes by the ΔS entropy statistic (E-test/SelectGene) and retain the top 500 informative genes (default k=500). Fit cell-type parameters with Learn: estimate p_ij from the mean expression vector of each cell type after across-gene normalization with Laplace smoothing (adding one). Classify each query cell by maximum likelihood estimation — equivalently, assign the cell type whose mean expression vector over the marker genes has the minimal Kullback–Leibler divergence to the unknown cell y. Optionally compute the confidence score C (likelihood ratio of training set vs null dataset) with cutoffs from 0.001 to 0.5 to control false positives for cell types not covered by the reference, and serialize the fitted classifier with ExportModel.

## Related tools

- **SciBet** (Implements SelectGene/Entropy gene selection, Learn (Laplace-smoothed multinomial fitting), and Bet classification by minimum KL divergence; ExportModel/LoadModel for portability) — https://github.com/PaulingLiu/scibet
- **scikit-learn (sklearn)** (model_selection.train_test_split for the stratified 7:3 hold-out split used to validate the classifier)

## Examples

```
devtools::install_github("PaulingLiu/scibet"); model <- ExportModel(Bet(expr, result))
```

## Evaluation signals

- Held-out test predictions (stratified 7:3 split, repeated 50 times) show high concordance with known cell type labels, consistent with SciBet's reported high prediction accuracy.
- Each predicted label is one of the reference cell types represented in the fitted model (no out-of-vocabulary labels).
- Gene set passed to Learn/Bet contains exactly k=500 (default) informative genes ranked by ΔS.
- Fitted model serializes and reloads cleanly via ExportModel/LoadModel with matching gene set, parameter vectors, and labels.
- False positives for out-of-reference cell types drop when applying confidence-score cutoffs in the 0.001–0.5 range.

## Limitations

- Relative similarity alone (as used by scmap and Seurat v3) can misclassify previously unseen types; SciBet's advantage requires modeling absolute similarity against a null dataset built from datasets not covered by the reference (e.g. Supplementary Table 4 integration).
- Across-species use requires HomoloGene (Build 68) one-to-one human-mouse gene conversion before classification.
- Classification accuracy depends on the informative-gene selection (top ΔS, 500 by default) and the log-normalization scheme matching the assay (TPM vs UMI).
- No seed or randomization parameters are reported for the repeated splits and permutation tests, complicating exact reproduction.

## Evidence

- [discussion] MLE as KL minimization: "we annotate the test cell to such the cell type that its mean expression vector of all marker genes has the minimal Kullback–Leibler divergence to the unknown cell y."
- [discussion] Laplace-smoothed multinomial parameters: "pij across all gene i could be estimated by Xij after normalization across genes and Laplace Smoothing"
- [discussion] informative gene selection: "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] hold-out validation: "we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3"
- [readme] SciBet purpose from README: "A portable and fast single cell type identifier"
