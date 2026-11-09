---
name: fisher-combined-probability-testing
description: Use when per-individual (or per-replicate) ASAS p values must be merged
  into one event-level p value, or when a non-paired baseline is needed to benchmark
  a paired differential AS model such as PAIRADISE.
when_to_use_negative:
- When the two alleles of an individual can be treated as paired replicates and a
  paired model (e.g., PAIRADISE or the rMATS paired model) can be fit — Fisher's method
  forgoes the paired structure and loses power
- When per-individual p values or allele-specific count data (IJC/SJC) are unavailable
  for the event being tested
- When the goal is de novo ASAS detection from raw RNA-seq reads rather than combining
  already-computed per-individual test results
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PAIRADISE
  role: Provides the event-level count table (IJC/SJC, incLen/skpLen, pval) and simulation
    framework against which the Fisher's combined baseline is benchmarked
  repo: github.com/Xinglab/PAIRADISE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:1fde4d5761a10caaeb54990031585a749d42424797f7ca0fa529da6c9f8c99de
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Xinglab/PAIRADISE
  tools:
  - PAIRADISE
  - rMATS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# fisher-combined-probability-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fisher's combined probability test aggregates per-individual allele-specific splicing p values for the same event into a single event-level p value. In the PAIRADISE paper it serves as one of four baseline ASAS detection methods benchmarked in the exon-skipping simulation study.

## When to use

Use when per-individual (or per-replicate) ASAS p values must be merged into one event-level p value, or when a non-paired baseline is needed to benchmark a paired differential AS model such as PAIRADISE.

## When NOT to use

- When the two alleles of an individual can be treated as paired replicates and a paired model (e.g., PAIRADISE or the rMATS paired model) can be fit — Fisher's method forgoes the paired structure and loses power
- When per-individual p values or allele-specific count data (IJC/SJC) are unavailable for the event being tested
- When the goal is de novo ASAS detection from raw RNA-seq reads rather than combining already-computed per-individual test results

## Inputs

- Per-individual allele-specific inclusion/skipping counts (IJC_REF, SJC_REF, IJC_ALT, SJC_ALT) for each ASAS event
- Effective isoform lengths (incLen, skpLen) per event, in the PAIRADISE output-table schema
- Per-individual p values for the same splicing event (input to the combination step)

## Outputs

- Event-level Fisher combined p value (and BH-corrected qval) per ASAS event
- ROC curves with AUC and TPR at 5% FPR for comparison against PAIRADISE and other baselines (e.g., simulation_roc_results.csv)

## How to apply

For each exon-skipping ASAS event, first compute an independent per-individual p value from the allele-specific counts (IJC_REF, SJC_REF, IJC_ALT, SJC_ALT), then combine them with Fisher's method: form the statistic X2 = -2 * sum(ln p_i), which under the null follows a chi-square distribution with 2k degrees of freedom for k individuals/replicates. In the PAIRADISE benchmark, run this on the same simulated events used for PAIRADISE — 5,000 simulated exon-skipping events with M=3 replicates under the low-variance setting, with psi and effective-length parameters sampled from the Geuvadis CEU population distribution, formatted in the PAIRADISE output schema (IJC/SJC counts plus incLen/skpLen effective lengths). Apply Benjamini-Hochberg FDR correction to obtain event-level q values, then compute ROC curves over p-value thresholds and extract AUC and TPR at 5% FPR alongside the paired t test, paired Wilcoxon test, and rMATS paired model. Expected result: Fisher's combined method attains the lowest performance of the five methods (AUC 60%, TPR 4% at 5% FPR), because it analyzes individuals separately and does not exploit the paired structure that PAIRADISE models.

## Related tools

- **PAIRADISE** (Provides the event-level count table (IJC/SJC, incLen/skpLen, pval) and simulation framework against which the Fisher's combined baseline is benchmarked) — github.com/Xinglab/PAIRADISE

## Examples

```
p_fisher <- pchisq(-2 * sum(log(p_ind)), df = 2 * length(p_ind), lower.tail = FALSE)  # Fisher's combined probability test
```

## Evaluation signals

- In the 3-replicate low-variance simulation, Fisher's combined method should yield the lowest ROC AUC of the five methods (60% vs. 84% PAIRADISE, 74% paired t test, 68% paired Wilcoxon, 63% rMATS paired)
- TPR at 5% FPR should be approximately 4% for Fisher's method, the lowest reported (61% PAIRADISE, 26% t test, 12% Wilcoxon, 23% rMATS)
- The combined statistic -2 * sum(ln p_i) must follow a chi-square distribution with 2k degrees of freedom for k combined p values
- The output table must reproduce the PAIRADISE schema columns (ExonID, IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen, pval, qval)
- Benjamini-Hochberg corrected q values must be monotonically non-decreasing with p values within the result table

## Limitations

- Lowest detection power in the reported simulations (TPR 4% at 5% FPR), reflecting loss of information from analyzing each individual separately
- Does not model the paired structure of the two alleles within an individual or isoform-proportion (psi) measurement uncertainty
- Requires a heterozygous SNP outside but near the alternative exon to obtain allele-specific read assignment, per PAIRADISE's ASAS detection setup

## Evidence
