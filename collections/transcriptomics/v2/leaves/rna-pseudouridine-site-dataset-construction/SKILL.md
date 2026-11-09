---
name: rna-pseudouridine-site-dataset-construction
description: Use when building positive/negative Ψ-site training and test sets from
  RMBase v2.0 annotations that require sequence-identity reduction across 10-fold
  splits, e.g. for evaluating RNA modification predictors under balanced and imbalanced
  class ratios.
when_to_use_negative:
- The task is annotating Ψ sites de novo in a genome rather than benchmarking predictors
  on curated RMBase-validated sites
- Input is already a sequence-identity-reduced, split-ready feature matrix — re-clustering
  with cd-hit-est-2d at 0.8 would discard data unnecessarily
- A modification type other than pseudouridine is targeted, since negative sets are
  defined specifically by absence of experimentally validated Ψ sites
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: BEDTools
  role: Extracts each Ψ site with 25 bases of upstream/downstream flanking sequence
    against species reference files
- name: cd-hit-est-2d
  role: Reduces sequence identity between training and test sets per fold at threshold
    0.8 (word length 4)
- name: RNAshapes
  role: Generates abstract dot-bracket secondary-structure notation for each RNA fragment
- name: EDeN
  role: Converts dot-bracket structures into six sub-shape contexts (S, M, H, I, F,
    T) for encoding
  repo: https://github.com/fabriziocosta/EDeN
- name: MU-PseUDeep
  role: Consumer of the constructed one-of-K encoded datasets; reference implementation
    of the training/evaluation pipeline
  repo: https://github.com/smk5g5/MU-PseUDeep
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:9baf03d03e8a7da074c6e027d4486b99315722a03b497504605ca02d0580afad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabriziocosta/EDeN
  tools:
  - BEDTools
  - EDeN
  - MU-PseUDeep
  - PseUI
  - RNAshapes
  - XG-PseU
  - cd-hit-est-2d
  - iRNA-PseU
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# rna-pseudouridine-site-dataset-construction

## Summary

Constructs benchmark datasets of pseudouridine (Ψ) sites for human, mouse, and yeast from RMBase v2.0 annotations, with sequence-identity-reduced 10-fold splits that enable fair comparison of Ψ-site predictors (MU-PseUDeep vs. PseUI, iRNA-PseU, XG-PseU) on both balanced and imbalanced regimes.

## When to use

Use when building positive/negative Ψ-site training and test sets from RMBase v2.0 annotations that require sequence-identity reduction across 10-fold splits, e.g. for evaluating RNA modification predictors under balanced and imbalanced class ratios.

## When NOT to use

- The task is annotating Ψ sites de novo in a genome rather than benchmarking predictors on curated RMBase-validated sites
- Input is already a sequence-identity-reduced, split-ready feature matrix — re-clustering with cd-hit-est-2d at 0.8 would discard data unnecessarily
- A modification type other than pseudouridine is targeted, since negative sets are defined specifically by absence of experimentally validated Ψ sites

## Inputs

- RMBase v2.0 pseudouridine site annotations for human, mouse, and yeast
- Reference genome FASTA files: hg19 (human), mm10 (mouse), sacCer3 (yeast)
- GENCODE GTF annotation file for transcript region extraction
- Positive and negative RNA fragment sequences (51 nt, uridine-centered)
- RNAshapes secondary-structure dot-bracket notation per fragment

## Outputs

- 10-fold stratified 80:20 train/test splits with sequence-identity-reduced partitions
- Balanced (bootstrap-binned) and imbalanced (10:1) positive/negative dataset variants
- One-of-K encoded sequence matrices (51×4) and structure matrices (51×6) per fold
- Per-fold accuracy, F1-score, and MCC averaged over bootstrap iterations

## How to apply

Extract each validated Ψ site plus 25 bases of upstream and downstream flanking sequence from RMBase v2.0 using BEDTools against hg19, mm10, or sacCer3 reference files, keeping only fragments with uridine at their center. Build the negative set from RNA regions lacking experimentally validated Ψ sites at a 10:1 negative-to-positive ratio. Perform a 10-fold stratified 80:20 train/test split, then reduce sequence identity between folds with cd-hit-est-2d (identity threshold 0.8, word length 4), remove remaining training sequences with >60% Needleman-Wunsch global-alignment identity to the test set, and cluster the test set internally with cd-hit-est at the same threshold. For model input, encode each 51-base sequence and its RNAshapes-predicted abstract dot-bracket secondary structure (converted to six sub-shape contexts S, M, H, I, F, T via EDeN) as one-of-K binary matrices of size 51×4 and 51×6 respectively.

## Related tools

- **BEDTools** (Extracts each Ψ site with 25 bases of upstream/downstream flanking sequence against species reference files)
- **cd-hit-est-2d** (Reduces sequence identity between training and test sets per fold at threshold 0.8 (word length 4))
- **RNAshapes** (Generates abstract dot-bracket secondary-structure notation for each RNA fragment)
- **EDeN** (Converts dot-bracket structures into six sub-shape contexts (S, M, H, I, F, T) for encoding) — https://github.com/fabriziocosta/EDeN
- **MU-PseUDeep** (Consumer of the constructed one-of-K encoded datasets; reference implementation of the training/evaluation pipeline) — https://github.com/smk5g5/MU-PseUDeep

## Examples

```
cd-hit-est-2d -i train.fa -i2 test.fa -o train_reduced.fa -c 0.8 -n 4
```

## Evaluation signals

- The reproduced MU-PseUDeep human balanced accuracy should match 0.726 ± 0.020 (F1 0.745 ± 0.041, MCC 0.524 ± 0.043) and imbalanced accuracy 0.894 ± 0.011 (Table 1)
- Every retained fragment must be exactly 51 nt with uridine at its center after BEDTools extraction
- No training-set sequence in any fold should share >60% Needleman-Wunsch global-alignment identity with any test-set sequence
- The imbalanced dataset should exhibit approximately a 10:1 negative-to-positive ratio before bootstrap binning
- cd-hit-est-2d should run with identity threshold 0.8 and word length 4, and cd-hit-est should cluster the test set at the same threshold

## Limitations

- Direct comparison with iPseU-CNN is impossible because no source code or webserver is available for that method
- Secondary-structure features depend on RNAshapes abstract shapes rather than experimentally determined structures, so encoding quality is prediction-limited
- Sequence-identity reduction via cd-hit-est-2d plus Needleman-Wunsch filtering can substantially shrink the training set, affecting bootstrap-binned negative sampling

## Evidence

- [methods] Extraction of Ψ sites with flanks: "we extracted the Ψ and surrounding 25 bases upstream and downstream nucleotides using BEDTools"
- [methods] Negative set definition: "To create the negative dataset, we collected those regions of RNA that did not contain any experimentally validated Ψ sites"
- [methods] Stratified splitting: "We did a 10-fold stratified split of positive and negative RNA samples into an 80:20 ratio for training and testing data"
- [methods] Identity reduction threshold: "using cd-hit-est-2d with a minimum sequence identity threshold (0.8)"
- [methods] Uridine-centered filter: "Only those fragments with uridine at their center were considered for further prediction"
- [readme] Skill scope from README: "Prediction of Pseudouridine Sites using deep learning"
