---
name: multinomial-likelihood-cell-type-classification
description: Use when annotating newly sequenced single-cell RNA-seq cells (or clusters)
  against labeled reference data — including cross-species queries mapped to one-to-one
  orthologues — and a fast, portable supervised classifier is needed.
when_to_use_negative:
- Cell types in the query data are not represented in the reference and no null dataset
  is available — SciBet's low false-positive rate relies on setting a null dataset
  as the alternate reference for uncovered types
- The query and reference gene spaces cannot be reconciled (e.g., cross-species work
  without one-to-one orthologues in HomoloGene)
- Labels are unknown in the training data — this is a supervised method requiring
  annotated reference cells
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SciBet
  role: R package providing SelectGene/Entropy (ΔS gene selection), Learn (multinomial
    parameter fitting with Laplace smoothing), Bet/SciBet (MLE annotation), ExportModel/LoadModel
    (portability), and conf_score (false-positive control)
  repo: https://github.com/PaulingLiu/scibet
- name: scikit-learn
  role: Provides model_selection.train_test_split for the stratified 7:3 hold-out
    validation strategy used to benchmark the classifier
- name: HomoloGene
  role: NCBI Build 68 database used to convert mouse (Tabula Muris) training genes
    to one-to-one human orthologues for cross-species classification
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:730ae972b2e76211334efafed9817a660130404ffa3393b79e7a66a4c7f891c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/scibet
  tools:
  - HomoloGene
  - SciBet
  - Tabula Muris
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# multinomial-likelihood-cell-type-classification

## Summary

SciBet-style supervised cell-type annotation that models each training cell type as a multinomial distribution over informative genes and assigns an unknown cell to the type with maximum multinomial likelihood (equivalently, minimal Kullback–Leibler divergence).

## When to use

Use when annotating newly sequenced single-cell RNA-seq cells (or clusters) against labeled reference data — including cross-species queries mapped to one-to-one orthologues — and a fast, portable supervised classifier is needed.

## When NOT to use

- Cell types in the query data are not represented in the reference and no null dataset is available — SciBet's low false-positive rate relies on setting a null dataset as the alternate reference for uncovered types
- The query and reference gene spaces cannot be reconciled (e.g., cross-species work without one-to-one orthologues in HomoloGene)
- Labels are unknown in the training data — this is a supervised method requiring annotated reference cells

## Inputs

- Log-normalized single-cell expression matrix (TPM + log for full-length; Seurat v3 size-factor-10,000, add-one log normalization for UMI) with cell-type labels for the training set
- Query cell expression profiles to annotate (same normalization and gene space as the reference)
- For cross-species use: NCBI HomoloGene Build 68 lookup table of one-to-one human-mouse orthologues (e.g., Tabula Muris mouse training data projected to human gene space)

## Outputs

- Per-cell predicted cell-type labels (e.g., scibet_predictions.csv)
- Serialized portable model (gene set, cell-type parameter vectors, labels) written via ExportModel as scibet_model.rds
- Optional confidence scores C used to withhold low-confidence assignments

## How to apply

Normalize the expression matrix first (TPM with pseudo-value one and log normalization for full-length data, or Seurat v3-style size-factor 10,000, add-one, log normalization for UMI data). Rank genes by the ΔS entropy (E-test) statistic and retain the top k = 500 informative genes by default. With Learn, estimate each cell type's multinomial parameters p_ij from mean expression after across-gene normalization with Laplace (add-one) smoothing; then with Bet/SciBet assign each query cell to the cell type whose mean expression vector over the informative genes maximizes the multinomial likelihood (MLE), which is equivalent to minimizing KL divergence to the query cell. Optionally compute a confidence score C (likelihood ratio of training set vs a null dataset) and withhold assignments below cutoffs in the 0.001–0.5 range to control false positives. Serialize the fitted model (gene set, cell-type parameter vectors, labels) with ExportModel for portable reuse via LoadModel.

## Related tools

- **SciBet** (R package providing SelectGene/Entropy (ΔS gene selection), Learn (multinomial parameter fitting with Laplace smoothing), Bet/SciBet (MLE annotation), ExportModel/LoadModel (portability), and conf_score (false-positive control)) — https://github.com/PaulingLiu/scibet
- **scikit-learn** (Provides model_selection.train_test_split for the stratified 7:3 hold-out validation strategy used to benchmark the classifier)
- **HomoloGene** (NCBI Build 68 database used to convert mouse (Tabula Muris) training genes to one-to-one human orthologues for cross-species classification)

## Examples

```
devtools::install_github("PaulingLiu/scibet"); genes <- SelectGene(train_expr, k=500, r=0.2); model <- Learn(train_expr, genes); pred <- Bet(test_expr, model)
```

## Evaluation signals

- Held-out predictions on a stratified 7:3 train/test split (repeated 50 times) show high concordance with known labels, consistent with SciBet's reported high prediction accuracy
- Cross-species run (Tabula Muris → human pancreas, one-to-one orthologues only) recovers correct cell types for ~92% of human pancreas cells, matching Fig. 3b
- Model file contains the informative gene set (default k=500), per-cell-type parameter vectors, and labels, and LoadModel restores a classifier that reproduces identical predictions
- With null-dataset confidence scoring, cells scoring below chosen cutoffs (0.001–0.5 range) are left unassigned rather than force-labeled
- Classifier completes quickly enough to reflect SciBet's order-of-magnitude speed advantage on the query dataset

## Limitations

- Classification accuracy depends on the ΔS-based informative gene set (default 500 genes); ΔS significance is assessed by random permutation test whose approximation strategy is only described in Supplementary Note 2.4
- No seed or randomization parameters are reported for the 50× repeated splits and permutation tests, complicating exact reproduction
- Software version numbers for dependencies (scikit-learn, Seurat v3, imbalanced-learn, kallisto) are not specified in the article
- False-positive control requires a null dataset (e.g., the integrated mock Human Cell Atlas background from Supplementary Table 4); without it, unrepresented cell types may be misassigned

## Evidence

- [discussion] Multinomial likelihood classification: "We assign cell type j to cell y with maximum likelihood estimation (MLE)"
- [discussion] KL-divergence equivalence: "we annotate the test cell to such the cell type that its mean expression vector of all marker genes has the minimal Kullback–Leibler divergence"
- [discussion] Feature selection parameters: "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] Laplace smoothing: "pij across all gene i could be estimated by Xij after normalization across genes and Laplace Smoothing"
- [discussion] Normalization protocol: "normalizing the UMI count of each cell with size-factor 10,000, adding one and then log normalization"
- [readme] Installation: "devtools::install_github("PaulingLiu/scibet")"
