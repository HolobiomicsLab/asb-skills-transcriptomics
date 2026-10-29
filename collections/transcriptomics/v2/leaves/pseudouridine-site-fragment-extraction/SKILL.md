---
name: pseudouridine-site-fragment-extraction
description: Use when constructing a training/testing dataset of RNA fragments centered
  on known Ψ sites from RMBase coordinates plus reference genome FASTAs, especially
  to train or benchmark Ψ site classifiers on human, mouse, or yeast data.
when_to_use_negative:
- Input is already a set of extracted fragment sequences with no genomic coordinates
  — re-extraction via BEDTools against hg19/mm10/sacCer3 is unnecessary (only the
  identity-reduction steps would apply).
- The goal is prediction on new fragments rather than benchmark construction — the
  negative sampling at 10x positives and 10-fold stratified splitting are dataset-construction
  steps, not inference steps.
- Targeting RNA modifications other than Ψ, or fragments where the modified base is
  not uridine at the fragment center — the fragment-center filter is Ψ/uridine specific.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0102
tools:
- name: BEDTools
  role: Extracts the Ψ site and surrounding 25 bases upstream and downstream nucleotides
    from reference genomes; also used to obtain genomic regions from the hg19 genome
    and GENCODE gtf file
- name: cd-hit-est-2d
  role: Reduces sequence identity between training and testing sets per fold at a
    minimum identity threshold of 0.8 (word length 4 for RNA)
- name: cd-hit-est
  role: Removes high-identity sequences within the test set at the same 0.8 threshold
    and word size
- name: Needleman-Wunsch
  role: Globally aligns the remaining training set against the test set so training
    sequences with >60% identity to the test set can be removed
- name: pandas
  role: Used with Scikit-learn for the 10-fold stratified 80:20 train/test split of
    positive and negative samples
- name: Scikit-learn
  role: Provides stratified k-fold splitting preserving positive/negative class ratios
    per fold
- name: MU-PseUDeep
  role: Downstream consumer of the constructed fragment dataset; open-source repository
    hosts the pipeline this dataset feeds
  repo: https://github.com/smk5g5/MU-PseUDeep
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:796ad632e965a67aa6bf1115ecc7fee55ad4f74167fb8c2cee77e74a0acf2e1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smk5g5/MU-PseUDeep
  tools:
  - BEDTools
  - MU-PseUDeep
  - Needleman-Wunsch
  - Scikit-learn
  - cd-hit-est
  - cd-hit-est-2d
  - pandas
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# pseudouridine-site-fragment-extraction

## Summary

Builds an experimentally validated pseudouridine (Ψ) site benchmark by extracting 51-nt uridine-centered RNA fragments from RMBase v2.0 annotations against hg19/mm10/sacCer3 reference genomes, then constructs class-imbalanced negatives and removes train/test sequence identity leakage before model training.

## When to use

Use when constructing a training/testing dataset of RNA fragments centered on known Ψ sites from RMBase coordinates plus reference genome FASTAs, especially to train or benchmark Ψ site classifiers on human, mouse, or yeast data.

## When NOT to use

- Input is already a set of extracted fragment sequences with no genomic coordinates — re-extraction via BEDTools against hg19/mm10/sacCer3 is unnecessary (only the identity-reduction steps would apply).
- The goal is prediction on new fragments rather than benchmark construction — the negative sampling at 10x positives and 10-fold stratified splitting are dataset-construction steps, not inference steps.
- Targeting RNA modifications other than Ψ, or fragments where the modified base is not uridine at the fragment center — the fragment-center filter is Ψ/uridine specific.

## Inputs

- RMBase v2.0 experimentally validated Ψ site annotations (BED/GFF coordinates) for human, mouse, or yeast
- Reference genome FASTA files: hg19 (human), mm10 (mouse), sacCer3 (yeast)
- GENCODE GTF file for genomic region/region filtering
- BEDTools for interval extraction; pandas and Scikit-learn for stratified splitting

## Outputs

- Positive 51-nt uridine-centered fragment sets (FASTA) per species, centered on the modified base
- Negative fragment sets at 10x the positive sample count (imbalanced configuration)
- 10-fold stratified 80:20 train/test partitions with class ratios preserved
- De-redundant train/test fold sets with no pair exceeding the 0.8 cd-hit-est-2d or 60% Needleman-Wunsch identity cutoffs

## How to apply

Download experimentally validated Ψ site annotations from RMBase v2.0 and extract each Ψ site plus 25 bases upstream and downstream (51-bp fragments) using BEDTools against hg19 (human), mm10 (mouse), and sacCer3 (yeast) reference genomes. Build the negative dataset from RNA regions containing no experimentally validated Ψ sites, sampling negatives at 10x the positive count for the imbalanced configuration. Perform a 10-fold stratified 80:20 train/test split with pandas and Scikit-learn to preserve class ratios. Reduce train/test identity leakage per fold: run cd-hit-est-2d at a minimum sequence identity threshold of 0.8 (word length 4 for RNA), globally align remaining training sequences against the test set with Needleman-Wunsch and remove training sequences with >60% identity to any test sequence, and cluster the test set internally with cd-hit-est at the same 0.8 threshold. Only fragments with uridine at their center are retained for downstream prediction. The output fragments are later one-of-K encoded as 51x4 binary matrices; this skill only covers dataset construction and de-redundancy, not encoding or model training.

## Related tools

- **BEDTools** (Extracts the Ψ site and surrounding 25 bases upstream and downstream nucleotides from reference genomes; also used to obtain genomic regions from the hg19 genome and GENCODE gtf file)
- **cd-hit-est-2d** (Reduces sequence identity between training and testing sets per fold at a minimum identity threshold of 0.8 (word length 4 for RNA))
- **cd-hit-est** (Removes high-identity sequences within the test set at the same 0.8 threshold and word size)
- **Needleman-Wunsch** (Globally aligns the remaining training set against the test set so training sequences with >60% identity to the test set can be removed)
- **pandas** (Used with Scikit-learn for the 10-fold stratified 80:20 train/test split of positive and negative samples)
- **Scikit-learn** (Provides stratified k-fold splitting preserving positive/negative class ratios per fold)
- **MU-PseUDeep** (Downstream consumer of the constructed fragment dataset; open-source repository hosts the pipeline this dataset feeds) — https://github.com/smk5g5/MU-PseUDeep

## Examples

```
cd-hit-est-2d -i train.fa -j test.fa -o train_cdhit.fa -c 0.8 -n 4
```

## Evaluation signals

- Each species yields 51-nt fragments with uridine at the fragment center (25 bases upstream + 25 downstream of the site).
- Imbalanced negative sets are exactly 10x the positive class count.
- The stratified 10-fold split preserves positive:negative class ratios in every 80:20 partition.
- No train/test pair exceeds the 0.8 cd-hit-est-2d identity threshold or the 60% Needleman-Wunsch identity cutoff.
- Test set internal clustering with cd-hit-est (0.8 threshold, word size 4) leaves no clusters above the identity cutoff.

## Limitations

- Only fragments with uridine at their center are considered for further prediction, so non-uridine-centered windows are discarded.
- The pipeline depends on experimentally validated RMBase v2.0 annotations; negatives are defined as regions without experimentally validated Ψ sites, so unvalidated sites may contaminate the negative class.
- No source code or webserver was available for the comparison method iPseU-CNN, preventing direct comparison on identically constructed benchmarks.

## Evidence

- [methods] Fragment extraction with BEDTools: "we extracted the Ψ and surrounding 25 bases upstream and downstream nucleotides using BEDTools"
- [methods] RMBase source of annotations: "The Ψ site information was downloaded from RMBase v2.0"
- [methods] Stratified split: "We did a 10-fold stratified split of positive and negative RNA samples into an 80:20 ratio for training and testing data"
- [methods] cd-hit-est-2d identity threshold: "using cd-hit-est-2d with a minimum sequence identity threshold (0.8)"
- [methods] Needleman-Wunsch training-set filtering: "removed the sequences from the training set that had >60% sequence identity with the test set"
- [readme] Tool purpose (repository README): "Prediction of Pseudouridine Sites using deep learning"
