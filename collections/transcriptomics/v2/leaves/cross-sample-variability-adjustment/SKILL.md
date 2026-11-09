---
name: cross-sample-variability-adjustment
description: Use when comparing pseudotemporal gene expression dynamics across two
  or more conditions where each condition has multiple biological sample replicates,
  and cell-level methods would inflate false discoveries.
when_to_use_negative:
- Data are single-sample trajectories with no biological replicates per condition
  — condiments-style assumptions or single-trajectory methods apply instead
- The goal is cell-level abundance or topology change detection rather than cross-condition
  gene expression dynamics (use Lamian's cell density / topology modules instead)
- Data have already been collapsed to a per-sample feature table where cell-level
  variability is no longer distinguishable
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Lamian
  role: 'Primary framework: permutation-based likelihood-ratio XDE test (lamian_test,
    test.type=''variable'') that draws inference after accounting for cross-sample
    variability'
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Default trajectory method in Lamian used to construct pseudotime via cluster-based
    minimum spanning tree before XDE testing
  repo: https://github.com/zji90/TSCAN
- name: tradeSeq
  role: Competing XDE-detection method benchmarked for false positives via patternTest,
    earlyDETest, and diffEndTest on the same null data
- name: condiments
  role: Competing pseudotime condition-comparison method benchmarked (8753 false positives
    on null HCA-BM)
- name: Monocle
  role: Competing trajectory method benchmarked via monocle2TrajTestCorr (7846 false
    positives on null HCA-BM)
- name: Phenopath
  role: Competing pseudotime differential method benchmarked (7400 false positives
    on null HCA-BM)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:b6a9ff7a84e37ad0940f4ba6cc1bd105fab9706122519572da149f9704ae5808
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Monocle
  - Phenopath
  - TSCAN
  - condiments
  - tradeSeq
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# cross-sample-variability-adjustment

## Summary

A skill for testing differential gene expression along pseudotime in multi-sample scRNA-seq data while explicitly modeling biological sample-to-sample variability, so that significance calls generalize to new samples and FDR is appropriately controlled. The paper's null-dataset benchmark on 8 HCA bone-marrow donors shows this eliminates the thousands of false positives produced by methods that ignore sample variability.

## When to use

Use when comparing pseudotemporal gene expression dynamics across two or more conditions where each condition has multiple biological sample replicates, and cell-level methods would inflate false discoveries.

## When NOT to use

- Data are single-sample trajectories with no biological replicates per condition — condiments-style assumptions or single-trajectory methods apply instead
- The goal is cell-level abundance or topology change detection rather than cross-condition gene expression dynamics (use Lamian's cell density / topology modules instead)
- Data have already been collapsed to a per-sample feature table where cell-level variability is no longer distinguishable

## Inputs

- Gene-by-cell scRNA-seq expression matrices from multiple biological samples (e.g., 8 HCA-BM healthy donors, 10x Genomics raw counts)
- Cell-to-sample annotation data (hca_bm_cellanno style) mapping each cell to its donor/sample
- Pseudotime values per cell (e.g., hca_bm_pca low-dimensional embedding + TSCAN trajectory)
- Sample-level design matrix with condition covariates (e.g., pseudo-condition group assignments for the null dataset)

## Outputs

- Per-gene XDE test statistics and FDR.overall values with significant-gene calls at FDR.overall < 0.05
- Per-method false-positive counts on the null dataset (null_false_positives.csv) for benchmarking against limma, tradeSeq, PhenoPath, condiments, and Monocle2

## How to apply

Filter the 8 HCA-BM donor scRNA-seq matrices (drop cells with <5000 reads, <1000 expressed genes, or >10% mitochondrial reads; drop genes expressed in <0.1% of cells), embed cells in a common low-dimensional space, and infer pseudotime with TSCAN's cluster-based minimum spanning tree as implemented in Lamian. For cross-condition expression-dynamics (XDE) testing, run lamian_test with test.type='variable' using the default permutation-based likelihood ratio test (permuiter=100) that compares nested models after accounting for cross-sample variability; optionally run the chi-square variant (test.method='chisq') for comparison. Call significant genes at FDR.overall < 0.05. Validate calibration on a null dataset: randomly partition the 8 donors into two pseudo-condition groups and remove group differences so no gene is truly differential, then count false positives per method and expect Lamian.pm to report ~0, whereas limma reports 0, Lamian.chisq 62, and competing methods thousands (tradeSeq 5822/7259/8783, PhenoPath 7400, monocle2TrajTestCorr 7846, condiments 8753).

## Related tools

- **Lamian** (Primary framework: permutation-based likelihood-ratio XDE test (lamian_test, test.type='variable') that draws inference after accounting for cross-sample variability) — https://github.com/Winnie09/Lamian
- **TSCAN** (Default trajectory method in Lamian used to construct pseudotime via cluster-based minimum spanning tree before XDE testing) — https://github.com/zji90/TSCAN
- **tradeSeq** (Competing XDE-detection method benchmarked for false positives via patternTest, earlyDETest, and diffEndTest on the same null data)
- **condiments** (Competing pseudotime condition-comparison method benchmarked (8753 false positives on null HCA-BM))
- **Monocle** (Competing trajectory method benchmarked via monocle2TrajTestCorr (7846 false positives on null HCA-BM))
- **Phenopath** (Competing pseudotime differential method benchmarked (7400 false positives on null HCA-BM))

## Examples

```
R: res <- lamian_test(dat = testres, pseudotime = pt, cellanno = cellanno, design = design, ncores = 1, test.type = 'variable', permuiter = 100)  # XDE test, then call sig genes at FDR.overall < 0.05
```

## Evaluation signals

- On the null HCA-BM dataset, Lamian.pm must report 0 XDE genes at the 5% FDR cutoff
- Expected per-method false-positive pattern: limma 0, Lamian.chisq 62, tradeSeq 5822/7259/8783, PhenoPath 7400, monocle2TrajTestCorr 7846, condiments 8753
- All counts are compiled into null_false_positives.csv with one row per method
- The permutation test uses the default permuiter=100 permutations with pseudotime permuted appropriately within samples
- Significance calls use FDR.overall < 0.05 consistently across all benchmarked methods

## Limitations

- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories; interoperability tailoring is deferred to future work
- Lamian.pm and Lamian.chisq differ in calibration (0 vs 62 false positives on null HCA-BM), so the chi-square variant is not as conservative
- The XDE testing procedure requires multiple biological samples per condition; with single samples per condition, cross-sample variability cannot be estimated

## Evidence

- [abstract] Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability: "Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability"
- [discussion] To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test.: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [discussion] Cells with fewer than 5000 reads, fewer than 1,000 expressed genes (i.e. genes with nonzero read count), or more than 10% of reads mapped to the mitochondrial genome were deemed as low quality: "Cells with fewer than 5000 reads, fewer than 1,000 expressed genes (i.e. genes with nonzero read count), or more than 10% of reads mapped to the mitochondrial genome were deemed as low quality"
- [methods] Please follow the details here https://github.com/Winnie09/Lamian to download **Lamian** package: "Please follow the details here https://github.com/Winnie09/Lamian to download **Lamian** package"
- [discussion] In Lamian, TSCAN is used as the default method to construct pseudotime due to its flexibility and scalability.: "In Lamian, TSCAN is used as the default method to construct pseudotime due to its flexibility and scalability."
