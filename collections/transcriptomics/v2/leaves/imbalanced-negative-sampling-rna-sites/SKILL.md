---
name: imbalanced-negative-sampling-rna-sites
description: Use when building Ψ-site classification datasets from RMBase v2.0 annotations
  and hg19/mm10/sacCer3 genomes where Ψ-free negative RNA regions must be sampled
  (e.g., 10:1 imbalanced configuration) before model training.
when_to_use_negative:
- The target task requires a balanced benchmark rather than the 10:1 imbalanced configuration
  (e.g., balanced-dataset comparisons where negatives equal positives in count).
- The RNA modification of interest is not pseudouridine (Ψ); RMBase Ψ annotations
  and Ψ-free region selection are specific to Ψ site prediction.
- Input fragments are already encoded feature matrices (e.g., 51x4 one-of-K tensors);
  this skill operates on genome-anchored regions before encoding.
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3474
tools:
- name: BEDTools
  role: Extracts 51-bp fragments centered on RMBase Ψ sites (±25 nt) against hg19/mm10/sacCer3
    reference genomes and builds region sets for negative sampling.
- name: pandas
  role: Holds labeled positive/negative fragment tables and drives the 10-fold stratified
    80:20 train/test partitioning.
- name: Scikit-learn
  role: Performs the stratified train/test split that preserves positive:negative
    class ratios across folds.
- name: cd-hit-est-2d
  role: Reduces sequence identity between training and testing sets for each fold
    at a minimum identity threshold of 0.8 (word length 4 for RNA).
- name: cd-hit-est
  role: Removes high-identity sequences within the test set at the same 0.8 identity
    threshold and word size.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:a3fa5ed858ddd25ba1202d43805828e642272095d634cbce5dc54ab6a40a38ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BEDTools
  - MU-PseUDeep
  - Needleman-Wunsch
  - Scikit-learn
  - cd-hit-est
  - cd-hit-est-2d
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# imbalanced-negative-sampling-rna-sites

## Summary

Constructs pseudouridine (Ψ) classification benchmarks by pairing 51-nt Ψ-centered positive RNA fragments with Ψ-free negative regions sampled at 10 times the positive count, then applying stratified 80:20 train/test splits and sequence-identity filtering to prevent train/test leakage.

## When to use

Use when building Ψ-site classification datasets from RMBase v2.0 annotations and hg19/mm10/sacCer3 genomes where Ψ-free negative RNA regions must be sampled (e.g., 10:1 imbalanced configuration) before model training.

## When NOT to use

- The target task requires a balanced benchmark rather than the 10:1 imbalanced configuration (e.g., balanced-dataset comparisons where negatives equal positives in count).
- The RNA modification of interest is not pseudouridine (Ψ); RMBase Ψ annotations and Ψ-free region selection are specific to Ψ site prediction.
- Input fragments are already encoded feature matrices (e.g., 51x4 one-of-K tensors); this skill operates on genome-anchored regions before encoding.

## Inputs

- RMBase v2.0 experimentally validated Ψ site annotations for human, mouse, and yeast
- Reference genome FASTA files: hg19 (human), mm10 (mouse), sacCer3 (yeast)
- GENCODE gtf annotation file (for defining transcribed regions, e.g., from hg19 via BEDTools)
- BED-format coordinates of Ψ sites (positive) and candidate Ψ-free regions (negative pool)

## Outputs

- 51-nt positive RNA fragments centered on the Ψ-modified base
- Negative RNA fragments containing no experimentally validated Ψ sites, sampled at 10x the positive count (imbalanced configuration)
- Per-fold stratified 80:20 train/test partitions preserving positive:negative class ratios
- Identity-reduced train/test fold sets (cd-hit-est-2d ≤0.8, Needleman-Wunsch ≤60% train-vs-test, cd-hit-est ≤0.8 within test)

## How to apply

Download experimentally validated Ψ site annotations from RMBase v2.0 for human, mouse, and yeast, then extract each Ψ site plus 25 bases upstream and downstream (51-bp fragments) with BEDTools against hg19 (human), mm10 (mouse), and sacCer3 (yeast) reference genomes. Collect negative RNA regions that contain no experimentally validated Ψ sites, sampling negatives at 10 times the number of positive samples for the imbalanced configuration. Perform a 10-fold stratified split of positives and negatives into 80:20 train/test partitions with pandas and Scikit-learn so class ratios are preserved per fold. Reduce train/test sequence redundancy per fold with cd-hit-est-2d at a minimum identity threshold of 0.8, globally align remaining training sequences against the test set with Needleman-Wunsch and drop training sequences with >60% identity, then cluster the test set with cd-hit-est at the same 0.8 threshold. The rationale is that without the 10x negative oversampling the imbalanced-config evaluation is not comparable, and without the identity filters CNN performance is inflated by near-duplicate sequences leaking between folds.

## Related tools

- **BEDTools** (Extracts 51-bp fragments centered on RMBase Ψ sites (±25 nt) against hg19/mm10/sacCer3 reference genomes and builds region sets for negative sampling.)
- **pandas** (Holds labeled positive/negative fragment tables and drives the 10-fold stratified 80:20 train/test partitioning.)
- **Scikit-learn** (Performs the stratified train/test split that preserves positive:negative class ratios across folds.)
- **cd-hit-est-2d** (Reduces sequence identity between training and testing sets for each fold at a minimum identity threshold of 0.8 (word length 4 for RNA).)
- **cd-hit-est** (Removes high-identity sequences within the test set at the same 0.8 identity threshold and word size.)

## Examples

```
from sklearn.model_selection import train_test_split; X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, stratify=y)
```

## Evaluation signals

- Imbalanced negative sets contain exactly 10 times the number of positive samples per species.
- Every extracted fragment is 51 nt long with the modified (uridine/Ψ) base at the center; only fragments with uridine at center are retained for prediction.
- No negative fragment overlaps an experimentally validated RMBase v2.0 Ψ site.
- Per-fold class ratios in the 80:20 train/test partitions match the full dataset (stratification check).
- After filtering, no train/test pair exceeds the cd-hit-est-2d 0.8 identity threshold or 60% Needleman-Wunsch identity, and the test set is clustered at 0.8 by cd-hit-est.

## Limitations

- Negatives are defined only as regions lacking experimentally validated Ψ sites, so unvalidated or undetected Ψ sites may remain in the negative set.
- The 10:1 negative:positive ratio and 0.8 / 60% identity cutoffs are fixed benchmark parameters; changing them produces datasets not comparable to the published MU-PseUDeep results.
- Identity filtering depends on cd-hit-est-2d (word length 4) and Needleman-Wunsch global alignment; very short or low-complexity fragments may be over- or under-clustered.

## Evidence

- [methods] Negative dataset construction: "To create the negative dataset, we collected those regions of RNA that did not contain any experimentally validated Ψ sites"
- [methods] Fragment extraction with BEDTools: "we extracted the Ψ and surrounding 25 bases upstream and downstream nucleotides using BEDTools"
- [methods] Stratified 80:20 split: "We did a 10-fold stratified split of positive and negative RNA samples into an 80:20 ratio for training and testing data"
- [methods] Train/test identity reduction: "We reduced the sequence identity between training and testing sets for each fold using cd-hit-est-2d"
