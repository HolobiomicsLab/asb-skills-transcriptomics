---
name: false-discovery-rate-control
description: Use when calling differential (XDE) genes along pseudotime from multi-sample
  single-cell RNA-seq with two or more conditions per contrast, and you must demonstrate
  FDR control at FDR.overall < 0.05 on a null dataset or spike-in simulation.
when_to_use_negative:
- Datasets with only one biological sample per condition — Lamian's FDR guarantee
  relies on estimating sample-level variance, and condiments' one-sample-per-condition
  assumption is exactly the limitation this skill is designed to avoid
- Trajectories inferred by Monocle2, Monocle3, or Slingshot for Lamian modules 1 and
  2 — the paper states these are not supported and tailoring them is deferred to future
  work
- Differential expression testing unrelated to pseudotime (e.g. plain bulk RNA-seq
  contrasts), where limma alone is already calibrated and the pseudotime permutation
  machinery is unnecessary
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Lamian
  role: Performs the permutation-based likelihood ratio test for XDE that accounts
    for cross-sample variability, yielding conservative FDR control; installed from
    GitHub
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Default method Lamian uses to construct pseudotime, which must be updated
    from GitHub before installing Lamian
  repo: https://github.com/zji90/TSCAN
- name: tradeSeq
  role: Competing method whose patternTest/earlyDETest/diffEndTest null false positives
    (5822/7259/8783) are counted to contrast FDR behavior
- name: condiments
  role: Competing method benchmarked for false positives (8753 on the null HCA-BM
    dataset); assumes one sample per condition and ignores sample-to-sample variation
- name: Phenopath
  role: Competing method benchmarked for false positives (7400 on the null HCA-BM
    dataset); assumes linear expression changes along pseudotime
- name: Monocle
  role: Competing trajectory test whose null false positives (7846 via monocle2TrajTestCorr)
    are tallied in the benchmark
- name: limma
  role: Competing method that reported 0 null false positives, used as a reference
    point in the null-dataset benchmark
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:994c5a585ed0d6ad1573e2f4a9b84857b8f5eff97b3b4e5b2fd4f44779961865
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - DAseq
  - Lamian
  - Monocle
  - Phenopath
  - Slingshot
  - TSCAN
  - condiments
  - limma
  - milo
  - tradeSeq
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# false-discovery-rate-control

## Summary

A benchmarking-and-inference procedure for verifying that differential pseudotime gene calls control FDR in multi-sample scRNA-seq: Lamian's permutation-based likelihood ratio test accounts for cross-sample variability, unlike tradeSeq, PhenoPath, condiments, Monocle, limma, and related methods that report thousands of null false positives.

## When to use

Use when calling differential (XDE) genes along pseudotime from multi-sample single-cell RNA-seq with two or more conditions per contrast, and you must demonstrate FDR control at FDR.overall < 0.05 on a null dataset or spike-in simulation.

## When NOT to use

- Datasets with only one biological sample per condition — Lamian's FDR guarantee relies on estimating sample-level variance, and condiments' one-sample-per-condition assumption is exactly the limitation this skill is designed to avoid
- Trajectories inferred by Monocle2, Monocle3, or Slingshot for Lamian modules 1 and 2 — the paper states these are not supported and tailoring them is deferred to future work
- Differential expression testing unrelated to pseudotime (e.g. plain bulk RNA-seq contrasts), where limma alone is already calibrated and the pseudotime permutation machinery is unnecessary

## Inputs

- Gene-by-cell scRNA-seq expression matrix (e.g. hca_bm_saver, SAVER-imputed HCA bone marrow counts from 8 healthy donors)
- Cell-to-sample annotation table (e.g. hca_bm_cellanno)
- Pseudotime vector for each cell (TSCAN-based trajectory)
- Design matrix with the intercept as its first column (condition/covariate indicators)
- Null dataset (HCA-BM samples randomly partitioned into two pseudo-conditions with group differences removed) or spike-in simulation with 1814 gold-standard genes at signal strengths 0.5-4

## Outputs

- Per-gene test statistics with reported FDR (FDR.overall) from lamian_test
- Counts of significant genes at FDR.overall < 0.05 per method on the null dataset (null_false_positives.csv)
- Real (empirical) FDR vs. reported FDR difference at each spike-in signal strength
- Sensitivity as a function of real FDR, summarized by AUC, for Lamian and competing methods (lamian_fdr_sensitivity_benchmark.csv)

## How to apply

Fit Lamian's XDE test with lamian_test using test.type='variable', testvar=2, and the permutation-based likelihood ratio test (default permuiter=100), providing the four Lamian inputs: gene-by-cell expression matrix, cell-to-sample annotation, pseudotime vector, and a design matrix whose first column is the intercept. The rationale is that in each permutation the cells are bootstrapped to account for pseudotime variability and pseudotime is permuted within each sample, so inference is drawn after accounting for cross-sample variability rather than treating cells as independent. Call significant genes at FDR.overall < 0.05 and validate calibration two ways: (1) on a null dataset built from the 8 HCA-BM healthy donors by randomly partitioning samples into two pseudo-conditions and removing group differences, count false positives per method — Lamian.pm should report 0 (Lamian.chisq 62) while tradeSeq, PhenoPath, condiments, and monocle2TrajTestCorr report thousands (5822/7259/8783, 7400, 8753, 7846); (2) in spike-in simulations with signals of strength 0.5–4 in 1814 gold-standard genes, verify the real (empirical) FDR does not exceed the FDR reported by Lamian.pm, i.e. conservative calibration, and that sensitivity-realFDR AUC is highest among all compared methods.

## Related tools

- **Lamian** (Performs the permutation-based likelihood ratio test for XDE that accounts for cross-sample variability, yielding conservative FDR control; installed from GitHub) — https://github.com/Winnie09/Lamian
- **TSCAN** (Default method Lamian uses to construct pseudotime, which must be updated from GitHub before installing Lamian) — https://github.com/zji90/TSCAN
- **tradeSeq** (Competing method whose patternTest/earlyDETest/diffEndTest null false positives (5822/7259/8783) are counted to contrast FDR behavior)
- **condiments** (Competing method benchmarked for false positives (8753 on the null HCA-BM dataset); assumes one sample per condition and ignores sample-to-sample variation)
- **Phenopath** (Competing method benchmarked for false positives (7400 on the null HCA-BM dataset); assumes linear expression changes along pseudotime)
- **Monocle** (Competing trajectory test whose null false positives (7846 via monocle2TrajTestCorr) are tallied in the benchmark)
- **limma** (Competing method that reported 0 null false positives, used as a reference point in the null-dataset benchmark)

## Examples

```
res <- lamian_test(dat = expr, cellanno = cellanno, pseudotime = pt, design = design, test.type = 'variable', testvar = 2, test.method = 'permutation', permuiter = 100); sig <- res$FDR.overall < 0.05
```

## Evaluation signals

- On the null HCA-BM dataset (8 donors randomly partitioned into two pseudo-conditions, group differences removed), Lamian.pm must report 0 XDE genes at FDR.overall < 0.05, while monocle2TrajTestCorr reports 7846 and condiments 8753
- In spike-in simulations at every signal strength (0.5-4), the real FDR must be smaller than the FDR reported by Lamian.pm (conservative calibration); competing methods underestimate the real FDR
- Lamian's sensitivity-realFDR AUC must exceed that of Monocle, TSCAN, Slingshot, tradeSeq, Phenopath, condiments, milo, and DAseq in the spike-in benchmark
- Permutation test must use the default permuiter=100 with pseudotime permuted within each sample after cell bootstrapping — a different permutation scheme invalidates the calibration claim
- Significant-gene counts must be compiled per method into null_false_positives.csv and the FDR/power comparison into lamian_fdr_sensitivity_benchmark.csv with the expected ordering: Lamian.pm 0, limma 0, Lamian.chisq 62, then thousands-scale counts for the remaining methods

## Limitations

- FDR control relies on multiple biological samples per condition; with single samples per condition, sample-level variance cannot be separated from cell-level variance
- A systematic comparison of harmonization (sample-embedding) methods was beyond the scope of the study, so FDR calibration was demonstrated only with the Seurat-based harmonization and TSCAN pseudotime used by the authors
- The chi-square variant (Lamian.chisq) is less conservative — it reported 62 null false positives versus 0 for the permutation-based Lamian.pm

## Evidence

- [abstract] Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false: "Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false"
- [discussion] Permutation scheme: "In each permutation, we first bootstrap the cells to account for pseudotime variability, and we then permute the pseudotime of the cells within each sample."
- [discussion] Test framework: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [readme] FDR control claim: "Lamian is able to more appropriately control the false discovery rate (FDR) when analyzing multi-sample data, a property not offered by any other methods."
- [readme] Sample variability accounting: "Lamian accounts for variability across biological samples which other existing pseudotime methods do not consider."
