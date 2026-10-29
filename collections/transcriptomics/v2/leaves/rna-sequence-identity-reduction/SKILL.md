---
name: rna-sequence-identity-reduction
description: Use when building train/test splits of RNA fragments (e.g., 51-nt pseudouridine-site
  windows) where homologous sequences across folds would inflate prediction accuracy
  through leakage.
when_to_use_negative:
- Input is already a single de-duplicated benchmark where train/test homology filtering
  was applied previously
- The comparison targets methods evaluated without identity reduction (e.g., iPseU-CNN,
  for which no source code or webserver was available for direct comparison)
- Working with non-RNA sequences where cd-hit-est word-length 4 defaults are inappropriate
  (use the protein or DNA cd-hit variants instead)
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: cd-hit-est-2d
  role: Compares training vs. test RNA sequences per fold and removes pairs above
    the 0.8 identity threshold (word length 4)
- name: Needleman-Wunsch
  role: Global alignment of remaining training sequences against the test set to remove
    training sequences with >60% identity
- name: cd-hit-est
  role: Clusters the test set internally at threshold 0.8 to remove high-identity
    sequences within the held-out data
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:da2a4d6cce36dcd61b798489f0a3ace3ef2b9765c75fe2bd49b344c658e3a433
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MU-PseUDeep
  - Needleman-Wunsch
  - PseUI
  - Scikit-learn
  - XG-PseU
  - cd-hit-est
  - cd-hit-est-2d
  - iRNA-PseU
  - pandas
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# rna-sequence-identity-reduction

## Summary

De-replicates RNA sequence benchmark sets so that training and test fragments share no high-identity homologs, using cd-hit-est-2d (threshold 0.8, word length 4), Needleman-Wunsch global alignment (>60% identity removal), and cd-hit-est within-set clustering.

## When to use

Use when building train/test splits of RNA fragments (e.g., 51-nt pseudouridine-site windows) where homologous sequences across folds would inflate prediction accuracy through leakage.

## When NOT to use

- Input is already a single de-duplicated benchmark where train/test homology filtering was applied previously
- The comparison targets methods evaluated without identity reduction (e.g., iPseU-CNN, for which no source code or webserver was available for direct comparison)
- Working with non-RNA sequences where cd-hit-est word-length 4 defaults are inappropriate (use the protein or DNA cd-hit variants instead)

## Inputs

- 51-bp RNA fragment FASTA files (positive and negative samples, e.g. uridine-centered Ψ site windows from hg19/mm10/sacCer3)
- Per-fold training and test sequence sets from the 10-fold stratified split

## Outputs

- Redundancy-filtered training FASTA (sequences with >0.8 cd-hit-est-2d or >60% Needleman-Wunsch identity to test removed)
- De-duplicated test FASTA clustered with cd-hit-est at threshold 0.8

## How to apply

After the 10-fold stratified 80:20 split, run cd-hit-est-2d pairwise between the training and test sets with a minimum sequence identity threshold of 0.8 and word length 4 to remove cross-set redundancy. Then globally align the remaining training set against the test set with the Needleman-Wunsch algorithm and discard training sequences with >60% identity to any test sequence, catching homologs missed by the word-based method. Finally, cluster the test set internally with cd-hit-est at the same 0.8 threshold and word size to remove near-duplicates within the held-out data. Rationale: cd-hit-est word-based filtering is fast but can miss shorter or gapped alignments, so the alignment-based NW step provides a stricter identity guarantee before model evaluation.

## Related tools

- **cd-hit-est-2d** (Compares training vs. test RNA sequences per fold and removes pairs above the 0.8 identity threshold (word length 4))
- **Needleman-Wunsch** (Global alignment of remaining training sequences against the test set to remove training sequences with >60% identity)
- **cd-hit-est** (Clusters the test set internally at threshold 0.8 to remove high-identity sequences within the held-out data)

## Examples

```
cd-hit-est-2d -i train.fasta -i2 test.fasta -o train_filtered.fasta -c 0.8 -n 4
```

## Evaluation signals

- No training/test sequence pair exceeds the 0.8 cd-hit-est-2d identity threshold for any fold
- No remaining training sequence has >60% Needleman-Wunsch global-alignment identity to any test sequence
- The filtered test set contains no cd-hit-est clusters at threshold 0.8 with more than one member
- Downstream MU-PseUDeep accuracy (human balanced: 0.726 ± 0.020) is consistent with leakage-free evaluation rather than suspiciously inflated scores

## Limitations

- cd-hit-est word-based identity estimation can miss gapped or short local homologies, which is why the Needleman-Wunsch >60% step is applied in addition
- Aggressive de-replication shrinks the training set and may lower absolute performance metrics relative to unfiltered baselines
- The 0.8 threshold and word length 4 are RNA-specific; they do not transfer directly to protein or long-genomic sequence contexts

## Evidence

- [methods] cd-hit-est-2d cross-fold filtering: "We reduced the sequence identity between training and testing sets for each fold using cd-hit-est-2d"
- [methods] identity threshold parameter: "using cd-hit-est-2d with a minimum sequence identity threshold (0.8)"
- [methods] Needleman-Wunsch training filtering: "we globally aligned the remaining training set against the test set using the Needleman-Wunsch algorithm"
- [methods] 60% training identity removal: "removed the sequences from the training set that had >60% sequence identity with the test set"
- [other] test-set internal clustering validation: [evidence span withheld — anchor status: not_found]
