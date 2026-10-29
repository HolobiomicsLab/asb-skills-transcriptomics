---
name: null-dataset-simulation-via-sample-partitioning
description: 'Use when benchmarking false discovery control of XDE/pseudotime methods
  on multi-sample scRNA-seq: you hold samples with no true condition differences (built
  via random partitioning) and must count FDR-cutoff false positives.'
when_to_use_negative:
- The dataset contains genuine between-condition differences you want to detect —
  random partitioning and removal of group effects would erase the signal under study.
- Each condition has only one biological sample — the article notes condiments assumes
  one sample per condition, while this null requires multiple samples per pseudo-condition
  to expose sample-variability-driven false discoveries.
- Benchmarking topology or cell-density modules (differential branching, cell abundance
  changes) rather than gene-expression XDE false-positive control.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Lamian
  role: 'Method under test: runs the XDE test via lamian_test with test.type=''variable''
    (permutation-based, permuiter=100) and the chi-square variant test.method=''chisq''
    on the null data'
  repo: https://github.com/Winnie09/Lamian
- name: limma
  role: Competing method benchmarked on the null dataset for comparison of false-positive
    counts
- name: tradeSeq
  role: Competing method benchmarked via patternTest, earlyDETest, and diffEndTest
    on the same null data
- name: Phenopath
  role: Competing pseudotime method benchmarked on the null dataset
- name: condiments
  role: Competing pseudotime method benchmarked on the null dataset
- name: Monocle
  role: Competing method benchmarked via the monocle2 trajectory test (monocle2TrajTestCorr)
    on the null dataset
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:411b393e2edc5e8c64c458fa6792b0e3fe86e9084d76ad0fa6e8144085e94dde
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Monocle
  - Phenopath
  - condiments
  - limma
  - tradeSeq
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# null-dataset-simulation-via-sample-partitioning

## Summary

Constructs a false-positive benchmark for cross-sample differential expression (XDE) methods by randomly partitioning the 8 HCA-BM healthy donors into two pseudo-condition groups and removing group differences so no gene is truly differential, then counts significant calls per method at the 5% FDR cutoff.

## When to use

Use when benchmarking false discovery control of XDE/pseudotime methods on multi-sample scRNA-seq: you hold samples with no true condition differences (built via random partitioning) and must count FDR-cutoff false positives.

## When NOT to use

- The dataset contains genuine between-condition differences you want to detect — random partitioning and removal of group effects would erase the signal under study.
- Each condition has only one biological sample — the article notes condiments assumes one sample per condition, while this null requires multiple samples per pseudo-condition to expose sample-variability-driven false discoveries.
- Benchmarking topology or cell-density modules (differential branching, cell abundance changes) rather than gene-expression XDE false-positive control.

## Inputs

- Gene-by-cell raw count matrix from 8 HCA-BM healthy donors (10x Genomics, downloaded from the Human Cell Atlas data portal)
- Cell-to-sample annotation mapping each cell to its donor
- Pseudotime values per cell (TSCAN-constructed trajectory)
- Random 4/4 partition of the 8 samples into two pseudo-condition groups with group expression differences removed

## Outputs

- Per-method significant-gene calls on the null dataset at FDR.overall < 0.05 (all are false positives by construction)
- Per-method false-positive counts compiled into null_false_positives.csv

## How to apply

Load the 10x Genomics scRNA-seq data from the 8 HCA-BM healthy donors (HCA data portal), preparing gene-by-cell expression matrices, cell-to-sample annotations, and pseudotime. Build the null by randomly partitioning the 8 samples into two pseudo-condition groups and removing any group differences in expression dynamics so that no gene is truly differential. Run Lamian's XDE test with lamian_test (test.type='variable', permutation-based, default permuiter=100) and the chi-square variant (test.method='chisq'), which uses a permutation-based likelihood ratio test comparing nested models. Run the competing methods on the same null data: limma, tradeSeq (patternTest, earlyDETest, diffEndTest), PhenoPath, monocle2TrajTestCorr, and condiments. Call significant genes per method at FDR.overall < 0.05 and count them as false positives, compiling per-method counts into null_false_positives.csv. Validate against the reported pattern: Lamian.pm 0, limma 0, Lamian.chisq 62, versus thousands for tradeSeq (5822/7259/8783), PhenoPath (7400), monocle2 (7846), and condiments (8753).

## Related tools

- **Lamian** (Method under test: runs the XDE test via lamian_test with test.type='variable' (permutation-based, permuiter=100) and the chi-square variant test.method='chisq' on the null data) — https://github.com/Winnie09/Lamian
- **limma** (Competing method benchmarked on the null dataset for comparison of false-positive counts)
- **tradeSeq** (Competing method benchmarked via patternTest, earlyDETest, and diffEndTest on the same null data)
- **Phenopath** (Competing pseudotime method benchmarked on the null dataset)
- **condiments** (Competing pseudotime method benchmarked on the null dataset)
- **Monocle** (Competing method benchmarked via the monocle2 trajectory test (monocle2TrajTestCorr) on the null dataset)

## Examples

```
res <- lamian_test(test.type = 'variable', permuiter = 100)  # chi-square variant: test.method = 'chisq'
```

## Evaluation signals

- Lamian.pm reports 0 significant XDE genes at the 5% FDR cutoff on the null dataset, and limma also reports 0.
- Lamian.chisq reports 62 false positives — roughly two orders of magnitude below the thousands-scale counts of competitors.
- Competitor counts match the reported pattern: tradeSeq 5822/7259/8783 (patternTest/earlyDETest/diffEndTest), PhenoPath 7400, monocle2TrajTestCorr 7846, condiments 8753.
- Counts are compiled into null_false_positives.csv with one entry per method.
- Every gene called significant under the null is by construction a false positive, so counts directly measure each method's FDR control.

## Limitations

- Only 8 HCA-BM donors are available, so the null is built from a single random 4/4 partition; results may vary with the partition drawn and leave limited room for replicate resampling.
- The thousands of competitor false positives reflect sample-specific discoveries not generalizable to new samples, since methods like condiments do not consider sample-to-sample variation within conditions.
- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or slingshot trajectories, restricting how competing trajectory methods can be coupled to the framework.
- Phenopath assumes gene expression changes linearly along pseudotime, constraining the condition differences it can model even on null data.

## Evidence

- [other] Build the null dataset by randomly partitioning the 8 samples into two pseudo-condition groups and removing any group differences in expression dynamics so that no gene is truly differential.: [evidence span withheld — anchor status: not_found]
- [other] Call significant genes per method at FDR.overall < 0.05 and count them as false positives.: [evidence span withheld — anchor status: not_found]
- [other] Run Lamian's XDE test with lamian_test (test.type='variable', permutation-based, default permuiter=100) and the chi-square variant (test.method='chisq').: [evidence span withheld — anchor status: not_found]
- [discussion] The raw count matrix of bone marrow scRNA-seq data sequenced in 10x Genomics platform from 8 healthy donors were downloaded from the Human Cell Atlas (HCA) data portal37,38: "The raw count matrix of bone marrow scRNA-seq data sequenced in 10x Genomics platform from 8 healthy donors were downloaded from the Human Cell Atlas (HCA) data portal37,38"
- [discussion] To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test.: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [readme] As a result, Lamian is able to more appropriately control the false discovery rate (FDR) when analyzing multi-sample data, a property not offered by any other methods.: "As a result, Lamian is able to more appropriately control the false discovery rate (FDR) when analyzing multi-sample data, a property not offered by any other methods."
