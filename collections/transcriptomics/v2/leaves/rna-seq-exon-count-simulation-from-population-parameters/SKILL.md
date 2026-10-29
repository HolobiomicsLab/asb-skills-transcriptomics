---
name: rna-seq-exon-count-simulation-from-population-parameters
description: Use when benchmarking an alternative-splicing or ASAS detection method
  against competitors and you need simulated paired allelic count data with controlled
  variance and replicate number instead of real RNA-seq reads.
when_to_use_negative:
- You need ASAS detection on real RNA-seq data (e.g. GM12878 or Geuvadis samples)
  rather than a controlled power comparison — use the full PAIRADISE pipeline with
  fastq inputs instead
- Your differential-splicing comparison uses unpaired (single-condition) RNA-seq replicates,
  where the paired-model framing of PAIRADISE vs the rMATS paired model does not apply
- You only have raw fastq/vcf data and no population psi/effective-length distribution
  to sample simulation parameters from
tools:
- name: PAIRADISE
  role: Method under test and provider of the output-table schema (IJC/SJC counts,
    incLen/skpLen, pval, qval) that simulated data must mirror; FDR <= 10% default
    significance threshold for significant events
  repo: github.com/Xinglab/PAIRADISE
- name: rMATS
  role: 'Competing benchmark method: the rMATS paired model run on the same simulated
    events to obtain comparison p-values'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:8da91c4485b46d5795895c58d5ff5c4dec2421ceb5a118cf2859970fb056fd41
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# rna-seq-exon-count-simulation-from-population-parameters

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates exon-skipping inclusion/skipping count data (IJC/SJC with incLen/skpLen effective lengths, mirroring the PAIRADISE output table schema) by sampling psi and effective-length parameters from the Geuvadis CEU population distribution, to benchmark ASAS detection methods by ROC AUC and TPR at 5% FPR.

## When to use

Use when benchmarking an alternative-splicing or ASAS detection method against competitors and you need simulated paired allelic count data with controlled variance and replicate number instead of real RNA-seq reads.

## When NOT to use

- You need ASAS detection on real RNA-seq data (e.g. GM12878 or Geuvadis samples) rather than a controlled power comparison — use the full PAIRADISE pipeline with fastq inputs instead
- Your differential-splicing comparison uses unpaired (single-condition) RNA-seq replicates, where the paired-model framing of PAIRADISE vs the rMATS paired model does not apply
- You only have raw fastq/vcf data and no population psi/effective-length distribution to sample simulation parameters from

## Inputs

- Population parameter distribution (isoform proportion psi and effective lengths) sampled from Geuvadis CEU population
- Simulation design parameters: number of events (e.g. 5,000), number of replicates M (e.g. 3), variance setting (low-variance)
- Event-level count table schema: IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen

## Outputs

- Simulated exon-skipping count table in PAIRADISE output-table format (IJC/SJC for reference and alternative alleles, incLen, skpLen)
- Per-method p-values with Benjamini-Hochberg FDR-corrected qval
- simulation_roc_results.csv with AUC and TPR at 5% FPR per method
- ROC curves over p-value thresholds for each compared method

## How to apply

Generate simulated exon-skipping count data (5,000 events, M=3 replicates in the low-variance setting) with numpy/R, sampling isoform proportion (psi) and effective-length parameters from the Geuvadis CEU population distribution. Format the simulated data exactly as the PAIRADISE output table: IJC_REF, SJC_REF, IJC_ALT, SJC_ALT inclusion/skipping counts plus incLen and skpLen effective lengths. Run the method under test (e.g. PAIRADISE with Benjamini-Hochberg qval correction) and the competing tests (paired t test, paired Wilcoxon, rMATS paired, Fisher's combined method) on the identical simulated events. Compute ROC curves over p-value thresholds for each method and extract AUC and TPR at 5% FPR; in the article's simulation PAIRADISE attained AUC 84% versus 74% (paired t), 68% (Wilcoxon), 63% (rMATS paired), 60% (Fisher), with TPR 61%/26%/12%/23%/4%. The rationale is that few replicates and low variability are the hardest regime for ASAS detection, so the simulation isolates statistical power differences.

## Related tools

- **PAIRADISE** (Method under test and provider of the output-table schema (IJC/SJC counts, incLen/skpLen, pval, qval) that simulated data must mirror; FDR <= 10% default significance threshold for significant events) — github.com/Xinglab/PAIRADISE
- **rMATS** (Competing benchmark method: the rMATS paired model run on the same simulated events to obtain comparison p-values)

## Examples

```
import numpy as np; IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen = simulate_events(n_events=5000, M=3, setting='low_variance', psi_dist='Geuvadis_CEU')
```

## Evaluation signals

- Simulated table must match the PAIRADISE schema: columns IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen (plus pval/qval after testing)
- Reproduced ranking must show PAIRADISE at the top with AUC 84% and TPR at 5% FPR of 61%, against 74%/68%/63%/60% AUC and 26%/12%/23%/4% TPR for paired t, Wilcoxon, rMATS paired, and Fisher's combined method
- simulation_roc_results.csv must contain both AUC and TPR-at-5%-FPR values for all five methods over the same simulated events
- qval values must reflect Benjamini-Hochberg correction applied to raw pval across all 5,000 simulated events
- Parameter draws should be traceable to the Geuvadis CEU population distribution for psi and effective-length parameters

## Limitations

- The rMATS paired model's correlation parameter can be unreliably estimated, which can inflate its p values and distort the comparison
- Simulation results are tied to the low-variance, M=3 replicate regime; power rankings may differ with more replicates or higher variability
- PAIRADISE's own default significance is FDR <= 10%; different significance thresholds will change TPR at fixed FPR

## Evidence
