---
name: polya-site-benchmark-dataset-curation
description: Use when a sequence-based polyA/IPA site predictor must be independently
  validated on external polyA databases (GENCODE, PolyA_DB 3, PolyASite 2.0) rather
  than on random splits of its own training annotations.
when_to_use_negative:
- When the goal is read-based IPA detection from RNA-seq alignments — that is the
  InPACT read module's role, not sequence-module benchmarking
- When the predictor was trained on or exposed to sites overlapping the external test
  databases, since the exclusion of training-overlapping sites is essential to this
  skill's validity
- When no independent external polyA annotation is available and only a random split
  of the training data is possible — leakage-free cross-database evaluation cannot
  be performed
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: InPACT
  role: Provides the CNN sequence module that is trained on RefSeq-annotated polyA
    sites and evaluated by AUROC on the curated benchmark sets; repository hosts the
    code and the precomputed GRCh38 polyA site annotation (InPACT_polyAsites.hg38.saf)
  repo: https://github.com/YY-TMU/InPACT
- name: DeepPASS
  role: Comparison method benchmarked on the same constructed GENCODE/PolyA_DB 3/PolyASite
    2.0 test sets
- name: APARENT
  role: Comparison method benchmarked on the same constructed GENCODE/PolyA_DB 3/PolyASite
    2.0 test sets
- name: DeepPASTA
  role: Comparison method benchmarked on the same constructed GENCODE/PolyA_DB 3/PolyASite
    2.0 test sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:967bb3743af742873c33433b91e4d8dadf3b1d50308ea8fd9ec92e3c72eb0d21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - APARENT
  - DeepPASS
  - DeepPASTA
  - InPACT
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# polya-site-benchmark-dataset-curation

## Summary

Constructs held-out polyA site benchmark test sets from GENCODE, PolyA_DB 3, and PolyASite 2.0 annotations (excluding sites overlapping the training data) and uses them to score a trained InPACT CNN sequence module by AUROC. This establishes an independent, non-overlapping evaluation of sequence-based polyA site prediction against external ground-truth databases.

## When to use

Use when a sequence-based polyA/IPA site predictor must be independently validated on external polyA databases (GENCODE, PolyA_DB 3, PolyASite 2.0) rather than on random splits of its own training annotations.

## When NOT to use

- When the goal is read-based IPA detection from RNA-seq alignments — that is the InPACT read module's role, not sequence-module benchmarking
- When the predictor was trained on or exposed to sites overlapping the external test databases, since the exclusion of training-overlapping sites is essential to this skill's validity
- When no independent external polyA annotation is available and only a random split of the training data is possible — leakage-free cross-database evaluation cannot be performed

## Inputs

- RefSeq polyA site annotations (hg38.109.20211119.ncbiRefSeq.gtf from UCSC goldenPath) for building the training positives
- 201-nt genomic windows centered on polyA sites and randomly sampled intergenic negatives, one-hot encoded (4x201 matrices)
- PolyA site sets from GENCODE, PolyA_DB 3, and PolyASite 2.0 for constructing held-out test sets

## Outputs

- Three held-out benchmark test sets (GENCODE, PolyA_DB 3, PolyASite 2.0) with training-overlapping sites removed
- AUROC values per benchmark database (expected: 0.954 GENCODE, 0.920 PolyA_DB 3, 0.794 PolyASite 2.0), recorded in sequence_module_auroc_results.csv
- Comparable AUROC scores for DeepPASS, APARENT, and DeepPASTA on the same test sets

## How to apply

First, train the InPACT CNN sequence module on 201-nt genomic windows centered at RefSeq-annotated polyA sites (positives) with an equal number of 201-nt intergenic windows as negatives, one-hot encoding nucleotides (A=[1,0,0,0], T=[0,1,0,0], G=[0,0,1,0], C=[0,0,0,1]); the network uses two convolution-ReLU-max-pooling pairs plus a fully connected layer with dropout, tuned by random sampling on a validation set. Then, for each of the three external sources — GENCODE, PolyA_DB 3, and PolyASite 2.0 — construct a held-out test set of polyA sites, explicitly excluding any sites that overlap the RefSeq training data to prevent leakage. Evaluate the trained CNN on each test set by computing the area under the ROC curve, and, to contextualize the numbers, benchmark competing tools (DeepPASS, APARENT, DeepPASTA) on the same constructed test sets. Correct application is confirmed when the three AUROC values land in the reported range (0.954 GENCODE, 0.920 PolyA_DB 3, 0.794 PolyASite 2.0) and are recorded in sequence_module_auroc_results.csv.

## Related tools

- **InPACT** (Provides the CNN sequence module that is trained on RefSeq-annotated polyA sites and evaluated by AUROC on the curated benchmark sets; repository hosts the code and the precomputed GRCh38 polyA site annotation (InPACT_polyAsites.hg38.saf)) — https://github.com/YY-TMU/InPACT
- **DeepPASS** (Comparison method benchmarked on the same constructed GENCODE/PolyA_DB 3/PolyASite 2.0 test sets)
- **APARENT** (Comparison method benchmarked on the same constructed GENCODE/PolyA_DB 3/PolyASite 2.0 test sets)
- **DeepPASTA** (Comparison method benchmarked on the same constructed GENCODE/PolyA_DB 3/PolyASite 2.0 test sets)

## Examples

```
git clone https://github.com/YY-TMU/InPACT.git; conda env create -f environment.yml; conda activate InPACT; InPACT -h
```

## Evaluation signals

- AUROC for GENCODE test set is approximately 0.954
- AUROC for PolyA_DB 3 test set is approximately 0.920
- AUROC for PolyASite 2.0 test set is approximately 0.794 (note the lower value; PolyASite 2.0 sites are experimentally derived and hence harder for sequence-only models)
- A check confirms zero overlap between each constructed test set and the RefSeq-derived training windows
- Results are serialized as three rows in sequence_module_auroc_results.csv, one AUROC per database

## Limitations

- AUROC degrades substantially on PolyASite 2.0 (0.794) relative to annotation-derived sets (GENCODE 0.954, PolyA_DB 3 0.920), reflecting distributional differences in experimentally determined polyA sites
- No version numbers are given for InPACT or the compared tools, complicating exact reproduction of benchmark numbers
- Hyperparameters are tuned by random sampling on a validation set, so the reported AUROCs depend on the random search outcome

## Evidence

- [methods] The sequence module incorporates a convolutional neural network (CNN) with multiple convolutional and pooling layers: "The sequence module incorporates a convolutional neural network (CNN) with multiple convolutional and pooling layers"
- [methods] evaluated on the polyA sites of GENCODE, PolyA_DB 3 and PolyASite 2.0: "evaluated on the polyA sites of GENCODE, PolyA_DB 3 and PolyASite 2.0"
- [readme] InPACT is a computational method designed to identify and quantify IPA sites via the examination of contextual sequence patterns: "InPACT is a computational method designed to identify and quantify IPA sites via the examination of contextual sequence patterns"
