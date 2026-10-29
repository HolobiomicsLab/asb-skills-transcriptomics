---
name: differential-expression-method-benchmarking
description: Use when evaluating which multi-sample trajectory differential expression
  method controls false discoveries on single-cell data, especially by counting called
  genes on a null dataset where no gene is truly differential.
when_to_use_negative:
- The dataset contains genuine condition effects — counting detections is only interpretable
  as a false-positive benchmark when the null dataset is constructed to have no true
  differential genes.
- Only one sample per condition is available — Lamian's cross-sample variability accounting,
  which drives the benchmark's expected result, cannot be applied.
- The comparison targets topology or cell-density (XCD) changes rather than gene expression
  dynamics along pseudotime (XDE).
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Lamian
  role: Reference XDE method with permutation-based likelihood ratio test and chi-square
    variant (lamian_test, test.type='variable', test.method='chisq'); expected to
    report near-zero false positives on the null dataset
  repo: https://github.com/Winnie09/Lamian
- name: limma
  role: Competing differential expression method benchmarked as a baseline; reported
    0 false positives in the validation pattern
- name: tradeSeq
  role: Competing trajectory method run via patternTest, earlyDETest, and diffEndTest;
    reported thousands of false positives (5822/7259/8783)
- name: Phenopath
  role: Competing pseudotime method assuming linear changes along pseudotime; reported
    7400 false positives on the null dataset
- name: condiments
  role: Competing condition-comparison method that assumes one sample per condition;
    reported 8753 false positives
- name: Monocle
  role: Competing trajectory method (monocle2TrajTestCorr); reported 7846 false positives,
    the baseline figure the benchmark pattern starts from
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:3d5fe0d41fbaaac6d656747c4943f0dc3832bb76891ddacb2eae965294c60220
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Monocle
  - Normalisr
  - Phenopath
  - Sanity
  - Splatter
  - SymSim
  - bayNorm
  - condiments
  - edgeR
  - limma
  - sctransform
  - tradeSeq
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# differential-expression-method-benchmarking

## Summary

Benchmark protocol for measuring false-positive rates of cross-sample differential expression (XDE) methods on a null single-cell RNA-seq dataset built from the 8 HCA bone marrow donors, comparing Lamian.pm/Lamian.chisq against limma, tradeSeq, PhenoPath, condiments, and Monocle2 trajectory tests at FDR 5%.

## When to use

Use when evaluating which multi-sample trajectory differential expression method controls false discoveries on single-cell data, especially by counting called genes on a null dataset where no gene is truly differential.

## When NOT to use

- The dataset contains genuine condition effects — counting detections is only interpretable as a false-positive benchmark when the null dataset is constructed to have no true differential genes.
- Only one sample per condition is available — Lamian's cross-sample variability accounting, which drives the benchmark's expected result, cannot be applied.
- The comparison targets topology or cell-density (XCD) changes rather than gene expression dynamics along pseudotime (XDE).

## Inputs

- Gene-by-cell raw count matrices from 8 HCA-BM healthy donors (10x Genomics scRNA-seq from the Human Cell Atlas data portal)
- Cell-to-sample annotation table (hca_bm_cellanno)
- Pseudotime per cell (TSCAN-based trajectory within Lamian)
- Sample-level design matrix assigning samples to the two pseudo-condition groups

## Outputs

- Per-method count of genes called significant at FDR.overall < 0.05 on the null dataset (null_false_positives.csv)
- Per-method significance results tables (e.g., FDR.overall column from lamian_test output)

## How to apply

Build a null dataset by loading gene-by-cell expression matrices from the 8 HCA-BM healthy donors (10x Genomics, pre-filtered for cells with ≥5000 reads, ≥1000 expressed genes, ≤10% mitochondrial reads, and genes expressed in ≥0.1% of cells), randomly partition the 8 samples into two pseudo-condition groups, and remove group differences in expression dynamics so no true XDE genes exist. Run Lamian's XDE test via lamian_test (test.type='variable', permutation-based likelihood ratio test with permuiter=100) and the chi-square variant (test.method='chisq'), then run limma, tradeSeq (patternTest, earlyDETest, diffEndTest), PhenoPath, Monocle2 trajectory test, and condiments on the identical null data. Call significant genes per method at FDR.overall < 0.05 and count them as false positives; the key rationale is that Lamian accounts for cross-sample variability, so a correct benchmark shows Lamian.pm near zero false positives while methods that ignore sample variability report thousands.

## Related tools

- **Lamian** (Reference XDE method with permutation-based likelihood ratio test and chi-square variant (lamian_test, test.type='variable', test.method='chisq'); expected to report near-zero false positives on the null dataset) — https://github.com/Winnie09/Lamian
- **limma** (Competing differential expression method benchmarked as a baseline; reported 0 false positives in the validation pattern)
- **tradeSeq** (Competing trajectory method run via patternTest, earlyDETest, and diffEndTest; reported thousands of false positives (5822/7259/8783))
- **Phenopath** (Competing pseudotime method assuming linear changes along pseudotime; reported 7400 false positives on the null dataset)
- **condiments** (Competing condition-comparison method that assumes one sample per condition; reported 8753 false positives)
- **Monocle** (Competing trajectory method (monocle2TrajTestCorr); reported 7846 false positives, the baseline figure the benchmark pattern starts from)

## Examples

```
devtools::install_github("Winnie09/Lamian")
```

## Evaluation signals

- Lamian.pm reports 0 XDE genes at the 5% FDR cutoff and limma reports 0 on the null HCA-BM dataset.
- Lamian.chisq reports 62 false positives — low thousands-scale methods are a red flag.
- tradeSeq tests report 5822 (patternTest) / 7259 (earlyDETest) / 8783 (diffEndTest), PhenoPath 7400, monocle2TrajTestCorr 7846, condiments 8753.
- All methods are run on the identical null partitioning of the 8 samples, so counts are directly comparable across methods.
- The null construction is verified: random partitioning plus removal of group differences leaves no truly differential genes, so any call at FDR.overall < 0.05 is a false positive by definition.

## Limitations

- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or slingshot trajectories, so tailoring competing methods is deferred to future work.
- Condiments assumes each condition has one sample and does not consider sample-to-sample variation when there are multiple replicates, which inflates its false positives in multi-sample benchmarks.
- Phenopath assumes gene expression changes linearly along pseudotime and cannot deal with arbitrary non-linear differences between conditions, limiting its behavior on real trajectories.
- Pseudotime inference uncertainty, gene expression/cell abundance uncertainty, and cross-sample variability are not all simultaneously accounted for — a stated open research topic.

## Evidence

- [abstract] Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false: "Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false"
- [discussion] The raw count matrix of bone marrow scRNA-seq data sequenced in 10x Genomics platform from 8 healthy donors were downloaded from the Human Cell Atlas (HCA) data portal: "The raw count matrix of bone marrow scRNA-seq data sequenced in 10x Genomics platform from 8 healthy donors were downloaded from the Human Cell Atlas (HCA) data portal37,38"
- [discussion] To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test.: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [intro] Condiments assumes that each condition has one sample and therefore does not consider sample-to-sample variation within each condition: "Condiments assumes that each condition has one sample and therefore does not consider sample-to-sample variation within each condition when each condition has multiple replicates"
- [readme] Lamian is able to more appropriately control the false discovery rate (FDR) when analyzing multi-sample data, a property not offered by any other methods.: "Lamian is able to more appropriately control the false discovery rate (FDR) when analyzing multi-sample data, a property not offered by any other methods."
