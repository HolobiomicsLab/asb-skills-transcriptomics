---
name: fisher-exact-pooled-read-testing
description: Use when simulating or benchmarking ASAS/differential splicing methods
  and a simple read-pooled baseline is needed to quantify the power gained by modeling
  replicates, particularly with injected outlier samples.
when_to_use_negative:
- When analyzing actual replicate RNA-seq data for ASAS detection rather than establishing
  a benchmark baseline — pooling discards replicate structure and loses robustness
  to outlier samples
- When reads cannot be uniquely assigned to alleles, e.g. SNPs located within or too
  distant from alternative exons, since allele-specific counting is a prerequisite
  for both pooling and PAIRADISE
- When the goal is differential AS between conditions rather than allele-specific
  comparisons within individuals
edam_topics:
- http://edamontology.org/topic_3523
tools:
- name: PAIRADISE
  role: Source of the allelic count table format (IJC/SJC/incLen/skpLen) that pooling
    operates on, and the replicate-aware method that pooled-Fisher is benchmarked
    against
  repo: github.com/Xinglab/PAIRADISE
- name: R
  role: Implements fisher.test on pooled 2x2 allelic count tables; PAIRADISE R dependencies
    (nloptr, doParallel, foreach) share this runtime
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:c888febdfd4713f6d89d04be8349461f6551d4aaef4961f843877aa2d9e65c34
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Xinglab/PAIRADISE
  tools:
  - Fisher's exact test (pooled reads)
  - PAIRADISE
  - Python
  - R
  - nloptr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# fisher-exact-pooled-read-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A baseline strategy for allele-specific alternative splicing (ASAS) detection in which allelic exon inclusion/skipping counts are pooled across replicates and tested with Fisher's exact test, ignoring per-replicate structure. It serves as the comparison benchmark that demonstrates why replicate-aware models like PAIRADISE are more robust, especially under outlier contamination.

## When to use

Use when simulating or benchmarking ASAS/differential splicing methods and a simple read-pooled baseline is needed to quantify the power gained by modeling replicates, particularly with injected outlier samples.

## When NOT to use

- When analyzing actual replicate RNA-seq data for ASAS detection rather than establishing a benchmark baseline — pooling discards replicate structure and loses robustness to outlier samples
- When reads cannot be uniquely assigned to alleles, e.g. SNPs located within or too distant from alternative exons, since allele-specific counting is a prerequisite for both pooling and PAIRADISE
- When the goal is differential AS between conditions rather than allele-specific comparisons within individuals

## Inputs

- Per-replicate allelic count table with columns IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen (PAIRADISE output format)
- Simulated allele-specific alternative splicing count datasets with an injected outlier replicate (PSI ~ Uniform[0,1])
- FDR threshold specification (default 10% with Benjamini-Hochberg adjustment)

## Outputs

- Pooled 2x2 contingency table per ASAS event (reference vs. alternative allele inclusion/skipping counts)
- Fisher's exact test p-values per event with Benjamini-Hochberg-adjusted significance calls at FDR <= 10%
- Power-versus-sample-size comparison table, e.g. simulation_outlier_power_comparison.csv, contrasting pooled-Fisher against PAIRADISE

## How to apply

For each simulated or real RNA-seq event, take the per-allele exon inclusion counts (IJC_REF, IJC_ALT) and skipping counts (SJC_REF, SJC_ALT) plus effective isoform lengths (incLen, skpLen) as produced by the PAIRADISE counting pipeline. Sum these 2x2 allelic count tables across all replicates, collapsing replicate structure into a single pooled contingency table, then run Fisher's exact test (R fisher.test) on the pooled counts. Call significant events using Benjamini-Hochberg adjustment at FDR <= 10%, matching PAIRADISE's default significance threshold so the comparison is fair. Repeat across increasing replicate numbers, with one outlier replicate whose PSI is drawn from Uniform[0,1], to trace the power-versus-sample-size curve and tabulate detection power for both methods (e.g., to simulation_outlier_power_comparison.csv).

## Related tools

- **PAIRADISE** (Source of the allelic count table format (IJC/SJC/incLen/skpLen) that pooling operates on, and the replicate-aware method that pooled-Fisher is benchmarked against) — github.com/Xinglab/PAIRADISE
- **R** (Implements fisher.test on pooled 2x2 allelic count tables; PAIRADISE R dependencies (nloptr, doParallel, foreach) share this runtime)

## Examples

```
fisher.test(matrix(c(sum(IJC_REF), sum(SJC_REF), sum(IJC_ALT), sum(SJC_ALT)), nrow=2))
```

## Evaluation signals

- Sanity check that pooled counts equal the column-wise sums of per-replicate IJC/SJC counts across all replicates for each event
- Significance calls must use the same Benjamini-Hochberg FDR <= 10% threshold as PAIRADISE for a fair power comparison
- Expected result under the outlier scenario: PAIRADISE substantially outperforms pooled Fisher's exact test, and both recover power as replicate count increases, with the pooled method lagging
- The simulation must inject exactly one outlier replicate with PSI drawn from Uniform[0,1]; verify the power curve is traced over multiple replicate counts
- Output table must contain paired power values (PAIRADISE vs. Fisher pooled) per sample size, e.g. in simulation_outlier_power_comparison.csv

## Limitations

- Pooling reads across replicates is vulnerable to outlier samples: with one replicate PSI drawn from Uniform[0,1], Fisher's pooled strategy performed substantially worse than PAIRADISE in simulations
- The pooled test cannot model per-individual allelic differences or measurement uncertainty of isoform ratios, motivating replicate-aware frameworks
- Like PAIRADISE, it requires a heterozygous SNP close enough to the alternative exon for allele-specific read assignment; SNPs within or too distant from the exon cannot be analyzed with short-read RNA-seq

## Evidence
