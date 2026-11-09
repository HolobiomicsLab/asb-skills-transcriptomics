---
name: scibet-multinomial-classification
description: Use when a labeled scRNA-seq reference exists and cell types must be
  predicted for newly sequenced cells or clusters, including cross-platform transfer
  (full-length vs UMI, e.g., 6 ordered human pancreas training-test pairs), with benchmarking
  against Seurat v3 and scmapCluster.
when_to_use_negative:
- The query contains cell types absent from the labeled reference — SciBet needs a
  null dataset configured (with confidence-score cutoffs 0.001–0.5) to control false
  positives, otherwise unrepresented cells will be force-assigned
- Input is raw unnormalized counts for a mixed-technology comparison — the full-length
  (TPM) vs UMI (size-factor 10,000) normalization must be applied per dataset first
- The goal is unsupervised discovery of novel cell types rather than supervised annotation
  against existing labels
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SciBet
  role: 'Core classifier: SelectGene (ΔS ranking), Learn (multinomial with Laplace
    smoothing), and Bet (MLE/KL-divergence assignment)'
  repo: https://github.com/PaulingLiu/scibet
- name: Seurat v3
  role: Benchmark comparator using FindTransferAnchors and TransferData for cross-platform
    label transfer
- name: scmap
  role: Benchmark comparator using scmapCluster, ignoring unassigned cells
- name: scikit-learn (sklearn)
  role: Hold-out validation strategy via model_selection.train_test_split for stratified
    7:3 splits
- name: imbalanced-learn
  role: Down-sampling of training data via under_sampling.RandomUnderSampler
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:3ebfaa739ccc6eba5abe4fabf1262df9fb31e57566b817b1f28d110394566ccd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/scibet
  tools:
  - Rcpp
  - SciBet
  - Seurat v3
  - imbalanced-learn
  - scikit-learn (sklearn)
  - scmap
  - tidyverse
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# scibet-multinomial-classification

## Summary

Applies SciBet, a supervised single-cell classifier built on a multinomial-distribution model with maximum-likelihood estimation and ΔS-based gene selection (top 500 by default), to annotate newly sequenced scRNA-seq cells against labeled reference data. It is designed to be fast, portable, technology-independent, and comparable to Seurat v3 and scmap on cross-platform benchmarks.

## When to use

Use when a labeled scRNA-seq reference exists and cell types must be predicted for newly sequenced cells or clusters, including cross-platform transfer (full-length vs UMI, e.g., 6 ordered human pancreas training-test pairs), with benchmarking against Seurat v3 and scmapCluster.

## When NOT to use

- The query contains cell types absent from the labeled reference — SciBet needs a null dataset configured (with confidence-score cutoffs 0.001–0.5) to control false positives, otherwise unrepresented cells will be force-assigned
- Input is raw unnormalized counts for a mixed-technology comparison — the full-length (TPM) vs UMI (size-factor 10,000) normalization must be applied per dataset first
- The goal is unsupervised discovery of novel cell types rather than supervised annotation against existing labels

## Inputs

- Normalized scRNA-seq expression matrix for training (cell × gene) with annotated cell type labels
- Held-out test scRNA-seq expression matrix from the same or a distinct sequencing platform (full-length or UMI)
- Technology-appropriate normalization: TPM+1 log-normalized full-length data or Seurat v3 size-factor-10,000 log-normalized UMI data

## Outputs

- Per-cell predicted cell type labels with multinomial maximum-likelihood assignment
- Mean classification accuracy per training-test pair per method (e.g., 6 pairs × SciBet/Seurat v3/scmap, as in fig3a_benchmark_accuracy.csv)
- Ranked informative gene list (top 500 by ΔS) and an exportable trained model via ExportModel

## How to apply

Normalize data per technology: TPM plus pseudo-value one with log-normalization for full-length read data, or size-factor 10,000, add one, then log normalization for UMI data. Form stratified 7:3 train/test splits without replacement (sklearn model_selection.train_test_split, with imbalanced-learn RandomUnderSampler down-sampling) and repeat the whole procedure 50 times. On each training set, run SciBet's SelectGene to rank genes by the ΔS statistic (significance assessable by random permutation test) and keep the top 500 informative genes, then Learn the multinomial model with Laplace smoothing. Classify held-out cells with Bet, which assigns the type whose mean expression vector has minimal Kullback–Leibler divergence to the test cell; optionally score confidence C as the likelihood ratio against a null dataset with cutoffs from 0.001 to 0.5. For benchmarking, compare against Seurat v3 (FindTransferAnchors + TransferData) and scmapCluster (ignoring unassigned cells) and average per-repeat accuracy over the 50 repeats.

## Related tools

- **SciBet** (Core classifier: SelectGene (ΔS ranking), Learn (multinomial with Laplace smoothing), and Bet (MLE/KL-divergence assignment)) — https://github.com/PaulingLiu/scibet
- **Seurat v3** (Benchmark comparator using FindTransferAnchors and TransferData for cross-platform label transfer)
- **scmap** (Benchmark comparator using scmapCluster, ignoring unassigned cells)
- **scikit-learn (sklearn)** (Hold-out validation strategy via model_selection.train_test_split for stratified 7:3 splits)
- **imbalanced-learn** (Down-sampling of training data via under_sampling.RandomUnderSampler)

## Examples

```
devtools::install_github("PaulingLiu/scibet"); genes <- SelectGene(train, k=3); model <- Learn(train, geneset=genes, k=3, a=1); pred <- Bet(test, model)
```

## Evaluation signals

- Output accuracy table contains all expected dataset pairs (e.g., 6 cross-platform pancreas pairs) and 3 method columns with mean accuracies in [0,1]
- SciBet's mean accuracy ranks at or above Seurat v3's, which ranks above scmap's, reproducing the reported Fig. 3a ordering
- Exactly 500 top-ΔS informative genes are retained by SelectGene before Learn
- Each method's accuracy is averaged over the 50 repeated stratified 7:3 splits, not a single split
- A learned model can be exported (ExportModel) and reloaded (LoadModel) with matching gene and label lists

## Limitations

- No seed or randomization parameters are reported for the 50× repeated splits and permutation tests, complicating exact reproduction
- Exact software versions (scikit-learn, Seurat v3 minor version, imbalanced-learn) are unspecified
- The permutation-test approximation for ΔS significance is only described in Supplementary Note 2.4, not in the main text
- Cells whose types are absent from the reference require an explicitly integrated null dataset (Supplementary Table 4) for false-positive control

## Evidence

- [discussion] We assign cell type j to cell y with maximum likelihood estimation (MLE): "We assign cell type j to cell y with maximum likelihood estimation (MLE)"
- [discussion] we selected genes with top ΔS as informative genes (500 by default): "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] stratified 7:3 split repeated 50 times: "we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times"
- [discussion] Laplace smoothing of multinomial parameters: "pij across all gene i could be estimated by Xij after normalization across genes and Laplace Smoothing"
- [discussion] SciBet classifies by minimal KL divergence to cell-type mean expression vector: "we annotate the test cell to such the cell type that its mean expression vector of all marker genes has the minimal Kullback–Leibler divergence"
- [readme] SciBet is a portable cross-platform single-cell classifier: "This user-friendly and cross-platform tool can be widely useful for single cell type identification."
- [discussion] UMI normalization protocol: "normalizing the UMI count of each cell with size-factor 10,000, adding one and then log normalization"
