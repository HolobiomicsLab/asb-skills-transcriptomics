---
name: stratified-train-test-splitting
description: Use when a small, imbalanced RNA-modification dataset (e.g., RMBase v2.0
  Ψ sites vs. 10x negatives) must be split into training and test sets while preserving
  class ratios and removing homologous sequences that would inflate evaluation performance.
when_to_use_negative:
- When the dataset is already identity-filtered or when the goal is benchmarking on
  a fixed external test set rather than cross-validation
- When sequence homology between train and test is desired (e.g., evaluating memorization),
  since this pipeline explicitly removes >0.8 / >60% identity pairs
- When class labels are unavailable or highly multi-class with tiny per-class counts
  where stratification cannot preserve ratios
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: Scikit-learn
  role: Performs the 10-fold stratified 80:20 train/test split preserving positive:negative
    class ratios
- name: pandas
  role: Handles sample tables during the stratified splitting step
- name: cd-hit-est-2d
  role: Compares training vs. test sets per fold at 0.8 minimum sequence identity
    (word length 4) to remove cross-set homologs
- name: Needleman-Wunsch
  role: Global alignment of remaining training sequences against the test set; training
    sequences with >60% identity are removed
- name: cd-hit-est
  role: Clusters the test set itself at 0.8 identity threshold and word size 4 to
    remove intra-test redundancy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:132f20d027e323cdf2968ae6f1be0a927c8ade7769570075ae3576856de681b7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MU-PseUDeep
  - Needleman-Wunsch
  - Scikit-learn
  - cd-hit-est
  - cd-hit-est-2d
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# stratified-train-test-splitting

## Summary

Constructs leakage-controlled train/test partitions for imbalanced Ψ-site classification datasets by stratifying class ratios across 10 folds and then re-filtering each fold's train/test pair with sequence-identity tools (cd-hit-est-2d, Needleman-Wunsch, cd-hit-est).

## When to use

Use when a small, imbalanced RNA-modification dataset (e.g., RMBase v2.0 Ψ sites vs. 10x negatives) must be split into training and test sets while preserving class ratios and removing homologous sequences that would inflate evaluation performance.

## When NOT to use

- When the dataset is already identity-filtered or when the goal is benchmarking on a fixed external test set rather than cross-validation
- When sequence homology between train and test is desired (e.g., evaluating memorization), since this pipeline explicitly removes >0.8 / >60% identity pairs
- When class labels are unavailable or highly multi-class with tiny per-class counts where stratification cannot preserve ratios

## Inputs

- Positive Ψ-site 51-nt fragments centered on the modified base (from RMBase v2.0 + BEDTools extraction against hg19, mm10, sacCer3)
- Negative RNA fragments with no experimentally validated Ψ sites, sampled at 10x the positive count
- Sequence records in FASTA format for identity-reduction tools

## Outputs

- 10-fold stratified train/test partitions (80:20) preserving class ratios
- Per-fold filtered training sets (pairs >60% NW identity or >0.8 cd-hit-est-2d identity removed)
- Redundancy-reduced test sets after cd-hit-est clustering (0.8 threshold, word size 4)

## How to apply

First perform a 10-fold stratified split of positive and negative RNA samples into an 80:20 train/test ratio using pandas and Scikit-learn so that each fold preserves the positive:negative class proportions (negatives sampled at 10x the positives for the imbalanced configuration). Then, for each fold, reduce sequence identity between training and testing sets with cd-hit-est-2d at a minimum identity threshold of 0.8 (word length 4 for RNA). Next, globally align the remaining training sequences against the test set with the Needleman-Wunsch algorithm and remove training sequences with >60% identity to any test sequence. Finally, apply cd-hit-est at the same 0.8 threshold and word size to remove high-identity redundancy within the test set itself, ensuring reported metrics reflect generalization rather than near-duplicate fragments.

## Related tools

- **Scikit-learn** (Performs the 10-fold stratified 80:20 train/test split preserving positive:negative class ratios)
- **pandas** (Handles sample tables during the stratified splitting step)
- **cd-hit-est-2d** (Compares training vs. test sets per fold at 0.8 minimum sequence identity (word length 4) to remove cross-set homologs)
- **Needleman-Wunsch** (Global alignment of remaining training sequences against the test set; training sequences with >60% identity are removed)
- **cd-hit-est** (Clusters the test set itself at 0.8 identity threshold and word size 4 to remove intra-test redundancy)

## Examples

```
cd-hit-est-2d -i train.fa -i2 test.fa -o train_filtered.fa -c 0.8 -n 4
```

## Evaluation signals

- Each species yields 51-nt fragments centered on the modified base after extraction
- Imbalanced negative set is 10x the size of the positive class, and this ratio is preserved in each of the 10 stratified folds
- No train/test pair exceeds the 0.8 cd-hit-est-2d identity threshold
- No remaining training sequence has >60% Needleman-Wunsch identity with any test sequence
- Test set contains no redundant sequences above 0.8 identity after cd-hit-est clustering

## Limitations

- Only experimentally validated Ψ sites from RMBase v2.0 for human, mouse, and yeast were used; other species/modifications are out of scope
- Aggressive identity filtering (0.8 cd-hit threshold plus 60% Needleman-Wunsch) reduces training data and may underrepresent sequence diversity
- cd-hit-est-2d and cd-hit-est word length 4 is RNA-specific; parameters may not transfer to other sequence types

## Evidence

- [methods] 10-fold stratified train/test split (80:20): "We did a 10-fold stratified split of positive and negative RNA samples into an 80:20 ratio for training and testing data"
- [methods] Sequence identity reduction between training and testing sets: "We reduced the sequence identity between training and testing sets for each fold using cd-hit-est-2d"
- [methods] Needleman-Wunsch filtering: "removed the sequences from the training set that had >60% sequence identity with the test set"
- [methods] Negative dataset construction: "To create the negative dataset, we collected those regions of RNA that did not contain any experimentally validated Ψ sites"
- [methods] Fragment extraction: "we extracted the Ψ and surrounding 25 bases upstream and downstream nucleotides using BEDTools"
