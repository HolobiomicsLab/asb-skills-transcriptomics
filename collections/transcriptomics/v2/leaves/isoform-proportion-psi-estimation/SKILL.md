---
name: isoform-proportion-psi-estimation
description: Use when you have per-sample, per-allele inclusion/skipping junction
  counts (IJC_REF/SJC_REF/IJC_ALT/SJC_ALT) with effective isoform lengths from a merged
  PAIRADISE count table, and need psi estimation plus an allelic-difference p-value
  aggregated across individuals.
when_to_use_negative:
- You do not yet have a merged count table — the model requires Step 1-4 output (personalization,
  mapping, assignment, annotation, counting, merging), not raw FASTQ or BAM files
- The heterozygous SNP lies within the alternative exon or too far from it to co-occur
  on a short RNA-seq read — PAIRADISE cannot analyze such events
- You need standard two-condition differential splicing without a paired allele/subject
  structure — the rMATS paired model is the appropriate analogue instead
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: PAIRADISE
  role: Fits the paired binomial + logit-normal hierarchical model via nloptr optimization
    (doParallel/foreach over events), runs the LRT on delta_i, and performs Benjamini-Hochberg
    FDR adjustment
  repo: https://github.com/Xinglab/PAIRADISE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:83b99e1e759cc3ebf2b2bc4391b1e070959232e4ec292e92223ff51c6daf4dfc
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# isoform-proportion-psi-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate exon inclusion proportions (psi) from paired per-allele RNA-seq counts and test for allele-specific alternative splicing with the PAIRADISE hierarchical model: binomial likelihoods on length-normalized inclusion/skipping counts, logit(psi) normal distributions sharing a subject effect, and an LRT on the allelic difference delta_i.

## When to use

Use when you have per-sample, per-allele inclusion/skipping junction counts (IJC_REF/SJC_REF/IJC_ALT/SJC_ALT) with effective isoform lengths from a merged PAIRADISE count table, and need psi estimation plus an allelic-difference p-value aggregated across individuals.

## When NOT to use

- You do not yet have a merged count table — the model requires Step 1-4 output (personalization, mapping, assignment, annotation, counting, merging), not raw FASTQ or BAM files
- The heterozygous SNP lies within the alternative exon or too far from it to co-occur on a short RNA-seq read — PAIRADISE cannot analyze such events
- You need standard two-condition differential splicing without a paired allele/subject structure — the rMATS paired model is the appropriate analogue instead

## Inputs

- Merged PAIRADISE Step 4 count table with per-event IJC_REF, SJC_REF, IJC_ALT, SJC_ALT counts
- Effective isoform lengths incLen/skpLen per event
- sample_name.txt (sample IDs, replicate grouping, and RNA-seq read lengths)

## Outputs

- pairadise_significant_asas_events.tsv with columns ExonID, IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen, pval, qval, IncLevel1/2, AvgIncLevel1/2, IncLevelDifference, AvgTotalCount1/2, SampleName, RefAltAllele, AF
- Plots of significant ASAS events (Step 5 visualization)

## How to apply

Load the merged count table from PAIRADISE Step 4 (IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen/skpLen, sample names from sample_name.txt) and fit the paired model with nloptr-based numerical optimization, parallelized over events with doParallel/foreach. Both allele groups share the subject effect alpha_ik; integrating it out yields bivariate normal logit(psi) levels with mean [mu_i, mu_i + delta_i] and covariance sigma_i^2 + diag([sigma_i1^2, sigma_i2^2]). Run a likelihood-ratio test of the null delta_i = 0 against chi-squared with 1 degree of freedom to get the raw pval per event. Adjust raw p-values with Benjamini-Hochberg to qval and retain significant ASAS events at FDR <= 10%. Optionally set sigma_i1 = sigma_i2 to increase power (e.g., 116 to 133 events in GM12878; 91 to 153 in Geuvadis CEU), recognizing this is a less conservative choice.

## Related tools

- **PAIRADISE** (Fits the paired binomial + logit-normal hierarchical model via nloptr optimization (doParallel/foreach over events), runs the LRT on delta_i, and performs Benjamini-Hochberg FDR adjustment) — https://github.com/Xinglab/PAIRADISE

## Examples

```
qsub -cwd -l h_vmem=10G qsub_files/pairadise_step5_stat.qsub
```

## Evaluation signals

- Output file exists and contains the required PAIRADISE columns (ExonID, pval, qval, IncLevelDifference, IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen)
- Every retained event has qval <= 0.10 (default FDR threshold)
- Each raw pval is consistent with an LRT statistic evaluated against chi-squared with 1 degree of freedom (pval in [0,1], no NAs for successfully optimized events)
- IncLevelDifference is computed from length-normalized inclusion levels (IncLevel1/2 built from IJC/incLen and SJC/skpLen)
- Count columns in the output match the Step 4 merged table row-for-row per ExonID

## Limitations

- Requires a heterozygous SNP outside the alternative exon but close enough to the AS event for allele-specific read assignment on the same RNA-seq read; SNPs within or too distant from the exon cannot be analyzed with short reads
- Causal SNPs affecting splicing may be missed — only tagging SNPs outside the exon are tested
- Results are intentionally conservative; allowing sigma_i1 = sigma_i2 yields more significant events (e.g., 91 to 153 in Geuvadis CEU) but is left as a user option
- Individual-specific allelic differences modulated by cis SNPs or trans-acting regulators are not modeled; covariate layers are deferred to future work

## Evidence

- [full_text] binomial likelihoods on length-normalized inclusion/skipping counts per allele, with logit(psi) normal distributions for reference and alternative alleles sharing the subject effect alpha_ik: [evidence span withheld — anchor status: not_found]
- [full_text] Run a likelihood-ratio test of the null allelic-difference hypothesis delta_i = 0 against a chi-squared distribution with 1 degree of freedom to obtain a raw p-value (pval) for each event: [evidence span withheld — anchor status: not_found]
