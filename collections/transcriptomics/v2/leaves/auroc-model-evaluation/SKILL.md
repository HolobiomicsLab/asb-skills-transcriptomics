---
name: auroc-model-evaluation
description: Use when a trained sequence classifier (e.g., a CNN scoring candidate
  polyA sites) must be benchmarked on held-out genomic test sets from independent
  databases, with AUROC as the accuracy metric.
when_to_use_negative:
- The candidate sites overlap the training data — evaluation would be optimistic;
  only fully non-overlapping held-out sets support a valid AUROC estimate
- The task requires sample-wise IPA quantification from RNA-seq coverage rather than
  sequence-only polyA site classification — the InPACT read module or downstream tools
  apply instead
- Test positives/negatives come from the same source database used for training (e.g.,
  only RefSeq sites) — cross-database generalization cannot be assessed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: InPACT
  role: Source of the CNN sequence module being evaluated; trained on RefSeq-annotated
    polyA sites and scored on held-out test sets
  repo: https://github.com/YY-TMU/InPACT
- name: DeepPASS
  role: Baseline polyA site predictor benchmarked on the same constructed test sets
    for AUROC comparison
- name: APARENT
  role: Baseline polyA site predictor benchmarked on the same constructed test sets
    for AUROC comparison
- name: DeepPASTA
  role: Baseline polyA site predictor benchmarked on the same constructed test sets
    for AUROC comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:e86fdcf4db991acf5240df314d5eed9bd2b84413f39f22fe9245b1d64ff2e7d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - APARENT
  - DeepPASS
  - DeepPASTA
  - InPACT
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# auroc-model-evaluation

## Summary

Evaluate a sequence-based classifier (here, the InPACT CNN sequence module for polyA site prediction) on held-out, non-overlapping test sets by computing AUROC against independently curated positive/negative sites, enabling unbiased comparison against external baselines.

## When to use

Use when a trained sequence classifier (e.g., a CNN scoring candidate polyA sites) must be benchmarked on held-out genomic test sets from independent databases, with AUROC as the accuracy metric.

## When NOT to use

- The candidate sites overlap the training data — evaluation would be optimistic; only fully non-overlapping held-out sets support a valid AUROC estimate
- The task requires sample-wise IPA quantification from RNA-seq coverage rather than sequence-only polyA site classification — the InPACT read module or downstream tools apply instead
- Test positives/negatives come from the same source database used for training (e.g., only RefSeq sites) — cross-database generalization cannot be assessed

## Inputs

- Held-out 201-nt genomic windows centered on polyA sites from GENCODE, PolyA_DB 3, and PolyASite 2.0 (positives)
- Equal number of randomly sampled 201-nt intergenic windows (negatives)
- One-hot encoded nucleotide matrices (A/T/G/C channels)
- Trained CNN sequence module (trained on RefSeq-annotated polyA sites)

## Outputs

- Per-database AUROC values for the sequence module (e.g., 0.954 GENCODE, 0.920 PolyA_DB 3, 0.794 PolyASite 2.0)
- sequence_module_auroc_results.csv summarizing AUROCs across test sets
- Comparable AUROC values for baselines (DeepPASS, APARENT, DeepPASTA) on the same test sets

## How to apply

First construct the held-out test sets so they share no overlap with the training data: in InPACT, positives are 201-nt genomic windows centered on annotated polyA sites drawn from independent databases (GENCODE, PolyA_DB 3, PolyASite 2.0), and an equal number of 201-nt windows randomly sampled from intergenic regions serve as negatives; sequences are one-hot encoded (A=[1,0,0,0], T=[0,1,0,0], G=[0,0,1,0], C=[0,0,0,1]). Train the CNN (two convolution-ReLU-max-pooling pairs plus one fully connected layer with dropout) on RefSeq-annotated polyA sites, tuning hyper-parameters on a validation set by random sampling. Score each test set and compute the AUROC per database. For fair comparison, construct identical test sets and benchmark competing tools (DeepPASS, APARENT, DeepPASTA) under the same evaluation protocol. Record per-database AUROCs (InPACT achieved 0.954 for GENCODE, 0.920 for PolyA_DB 3, 0.794 for PolyASite 2.0) in a results table (e.g., sequence_module_auroc_results.csv).

## Related tools

- **InPACT** (Source of the CNN sequence module being evaluated; trained on RefSeq-annotated polyA sites and scored on held-out test sets) — https://github.com/YY-TMU/InPACT
- **DeepPASS** (Baseline polyA site predictor benchmarked on the same constructed test sets for AUROC comparison)
- **APARENT** (Baseline polyA site predictor benchmarked on the same constructed test sets for AUROC comparison)
- **DeepPASTA** (Baseline polyA site predictor benchmarked on the same constructed test sets for AUROC comparison)

## Examples

```
python evaluate_sequence_module.py --model inpact_cnn.h5 --test_sets gencode.fa,polyadb3.fa,polyasite2.fa --negatives intergenic_201nt.fa --onehot A=1000,T=0100,G=0010,C=0001 --out sequence_module_auroc_results.csv
```

## Evaluation signals

- AUROC per held-out database falls within expected reported ranges (≈0.954 GENCODE, 0.920 PolyA_DB 3, 0.794 PolyASite 2.0 for the InPACT sequence module)
- No test windows overlap training-site intervals — verify via coordinate intersection against the RefSeq training set
- Positive and negative test sets are balanced (equal counts of 201-nt polyA-centered and intergenic windows)
- Baseline tools (DeepPASS, APARENT, DeepPASTA) are scored on byte-identical test sets so AUROC differences reflect model, not data, differences
- Results table (sequence_module_auroc_results.csv) contains exactly one AUROC per evaluated database with no missing values

## Limitations

- PolyASite 2.0 AUROC is notably lower (0.794), indicating performance depends on how well the test database's site definition matches the RefSeq training annotation
- No version numbers are given for InPACT or the compared tools, complicating exact reproducibility of the benchmark
- No parameters or thresholds are reported for the comparative benchmarks in some sections, limiting reproduction detail
- Negatives are randomly sampled intergenic windows, so AUROC reflects discrimination against generic sequence background, not against functional near-candidates

## Evidence

- [methods] CNN trained on RefSeq-annotated polyA sites evaluated on independent databases: "evaluated on the polyA sites of GENCODE, PolyA_DB 3 and PolyASite 2.0"
- [results] AUROC values achieved by the sequence module on the three held-out test sets: "achieving AUROC values of 0.954 for GENCODE, 0.920 for PolyA_DB 3, and 0.794 for PolyASite 2.0"
- [methods] Sequence module architecture described in methods: "The sequence module incorporates a convolutional neural network (CNN) with multiple convolutional and pooling layers"
- [methods] One-hot encoding scheme for nucleotide input windows: [evidence span withheld — anchor status: not_found]
- [readme] RefSeq annotation source used for training data: "hg38.109.20211119.ncbiRefSeq.gtf.gz"
