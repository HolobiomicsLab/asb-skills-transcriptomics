---
name: roc-auc-and-tpr-method-comparison
description: Use when ranking competing statistical detection methods on simulated
  count data with known positives, by comparing ROC AUC and TPR at a fixed 5% FPR
  across methods.
when_to_use_negative:
- Real RNA-seq data without simulated ground truth — ROC AUC and TPR at 5% FPR require
  known positive ASAS events, which real datasets lack
- Replicate numbers or variance settings far from the benchmark conditions — the reported
  ranking (AUC 84% vs 74%/68%/63%/60%) was established specifically for M=3 replicates
  and low variance
- Competing methods that do not emit per-event p values or scores that can be thresholded
  into an ROC curve
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: PAIRADISE
  role: 'Primary method under benchmark: run on simulated IJC/SJC counts to produce
    event-level p values (with Benjamini-Hochberg qval) that anchor the ROC comparison'
  repo: https://github.com/Xinglab/PAIRADISE
- name: rMATS
  role: Supplies the rMATS paired model, one of the four competing differential-AS
    tests evaluated on the same simulated events
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:1ef60c7428a8bf7ccbd0f671b3f88fc41992e0d4a9878ada2049e48d871f8564
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Xinglab/PAIRADISE
  repo_url: https://github.com/Xinglab/PAIRADISE
  tools:
  - PAIRADISE
  - rMATS
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# roc-auc-and-tpr-method-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking method that ranks ASAS/differential-splicing detection approaches by simulating exon-skipping count data (5,000 events, M=3 replicates) and comparing ROC AUC and true positive rate at a fixed 5% FPR across PAIRADISE and four competing statistical tests.

## When to use

Use when ranking competing statistical detection methods on simulated count data with known positives, by comparing ROC AUC and TPR at a fixed 5% FPR across methods.

## When NOT to use

- Real RNA-seq data without simulated ground truth — ROC AUC and TPR at 5% FPR require known positive ASAS events, which real datasets lack
- Replicate numbers or variance settings far from the benchmark conditions — the reported ranking (AUC 84% vs 74%/68%/63%/60%) was established specifically for M=3 replicates and low variance
- Competing methods that do not emit per-event p values or scores that can be thresholded into an ROC curve

## Inputs

- Simulated exon-skipping count table in the PAIRADISE output schema (ExonID, IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen) for 5,000 events x M=3 replicates under the low-variance setting
- Event-level p values from each compared method: PAIRADISE pval with Benjamini-Hochberg qval, paired t test, paired Wilcoxon test, rMATS paired model, and Fisher's combined method
- Ground-truth labels of which simulated ASAS events carry true allelic splicing differences

## Outputs

- ROC curves computed over p-value thresholds for each of the five methods
- Per-method AUC and TPR at 5% FPR assembled into simulation_roc_results.csv

## How to apply

Simulate exon-skipping count data with numpy/R for 5,000 events and M=3 replicates under the low-variance setting, sampling isoform proportion (psi) and effective-length parameters from the Geuvadis CEU population distribution, and format IJC/SJC inclusion and skipping counts for reference and alternative alleles plus incLen/skpLen effective lengths to mirror the PAIRADISE output table schema (ExonID, IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen, pval). Run PAIRADISE on the simulated counts and apply Benjamini-Hochberg FDR correction (qval) to obtain event-level p values, then run the four competing tests on the same events: paired t test, paired Wilcoxon test, the rMATS paired model, and Fisher's method for combining p values. Compute ROC curves over p-value thresholds for each of the five methods and extract AUC and TPR at 5% FPR, assembling per-method values into simulation_roc_results.csv. Validate by checking the table reproduces the reported ranking: AUC 84%/74%/68%/63%/60% and TPR 61%/26%/12%/23%/4% (in the method order PAIRADISE, paired t, paired Wilcoxon, rMATS paired, Fisher's combined), with PAIRADISE attaining the top AUC (84%) and top TPR (61%).

## Related tools

- **PAIRADISE** (Primary method under benchmark: run on simulated IJC/SJC counts to produce event-level p values (with Benjamini-Hochberg qval) that anchor the ROC comparison) — https://github.com/Xinglab/PAIRADISE
- **rMATS** (Supplies the rMATS paired model, one of the four competing differential-AS tests evaluated on the same simulated events)

## Examples

```
qsub -cwd -l h_vmem=10G qsub_files/pairadise_step5_stat.qsub
```

## Evaluation signals

- simulation_roc_results.csv contains both AUC and TPR-at-5%-FPR values for all five methods (PAIRADISE, paired t test, paired Wilcoxon, rMATS paired, Fisher's combined)
- Reproduced AUC ordering matches the report: PAIRADISE 84% > paired t test 74% > paired Wilcoxon 68% > rMATS paired 63% > Fisher's combined 60%
- Reproduced TPR at 5% FPR matches the per-method values: PAIRADISE 61%, paired t 26%, paired Wilcoxon 12%, rMATS paired 23%, Fisher's combined 4% (note rMATS beats Wilcoxon on TPR despite lower AUC)
- Simulated count table columns match the PAIRADISE output schema: IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen, pval
- PAIRADISE p values carry Benjamini-Hochberg FDR correction (qval) before ROC thresholding over p-value cutoffs

## Limitations

- The AUC/TPR ranking was established for the low-variance, M=3-replicate simulation with psi and effective-length parameters drawn from the Geuvadis CEU population distribution; transfer to other settings is unverified
- The rMATS paired model's correlation parameter can be unreliably estimated, leading to inflated p values, which can depress its measured ROC performance in the comparison
- PAIRADISE's results are intentionally conservative by default (sigma_i1 not restricted to equal sigma_i2); enforcing sigma_i1=sigma_i2 increases significant ASAS events (e.g., 91 to 153 in Geuvadis CEU), which would shift benchmark numbers
- Simulated events must include a heterozygous SNP outside the alternative exon but close enough to be captured on the same RNA-seq read; SNPs within or too distant from the exon cannot be analyzed

## Evidence

- [methods] ROC extraction procedure over p-value thresholds: [evidence span withheld — anchor status: not_found]
- [results] Reported per-method AUC in the three-replicate, low-variance simulation: [evidence span withheld — anchor status: not_found]
