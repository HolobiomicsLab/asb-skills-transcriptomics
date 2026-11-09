---
name: paired-replicate-differential-splicing-testing
description: Use when RNA-seq count data provide per-allele exon inclusion and skipping
  counts (IJC/SJC) with incLen/skpLen effective lengths for individuals sharing a
  heterozygous SNP, and the goal is to call significant ASAS events at FDR <= 10%.
when_to_use_negative:
- No heterozygous SNP exists outside the alternative exon but close enough to the
  AS event to be detected on the same RNA-seq read — PAIRADISE cannot assign reads
  allele-specifically
- SNPs are located within or too distant from alternative exons in short-read RNA-seq
  data — they cannot be analyzed
- The task is standard group-level differential splicing (e.g., case-control across
  conditions) rather than allele-specific comparison within the identical cellular
  environment of individuals
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: PAIRADISE
  role: Statistical framework and software performing the paired differential splicing
    test; installed via ./configure; make; make install and run as pipeline Step 5
    (pairadise_step5_stat.qsub)
  repo: github.com/Xinglab/PAIRADISE
- name: STAR
  role: Spliced aligner used in Step 1 to align reads to haplotype-modified personal
    genomes before allele-specific counting (dependency >= 2.6.0a)
  repo: https://github.com/alexdobin/STAR
- name: rMATS
  role: Paired-model competitor whose correlation parameter can be unreliably estimated,
    used as a baseline for ROC AUC comparison (63% vs PAIRADISE 84%)
- name: rPGA pipeline
  role: Provides the personalization, mapping, and read-assignment procedures (haplotype-modified
    personal genomes) that precede PAIRADISE testing
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:5723b6083a6f1ef23d88fb0f4916e5b882e3fb6073c43fa369eb44b95867ebe5
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
  - STAR
  - rMATS
  - rPGA pipeline
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# paired-replicate-differential-splicing-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply the PAIRADISE paired statistical model to inclusion/skipping junction counts (IJC/SJC) from reference and alternative alleles to detect allele-specific alternative splicing (ASAS), aggregating signals across paired replicates or individuals instead of testing one sample at a time.

## When to use

Use when RNA-seq count data provide per-allele exon inclusion and skipping counts (IJC/SJC) with incLen/skpLen effective lengths for individuals sharing a heterozygous SNP, and the goal is to call significant ASAS events at FDR <= 10%.

## When NOT to use

- No heterozygous SNP exists outside the alternative exon but close enough to the AS event to be detected on the same RNA-seq read — PAIRADISE cannot assign reads allele-specifically
- SNPs are located within or too distant from alternative exons in short-read RNA-seq data — they cannot be analyzed
- The task is standard group-level differential splicing (e.g., case-control across conditions) rather than allele-specific comparison within the identical cellular environment of individuals

## Inputs

- Merged count table with IJC_REF, SJC_REF, IJC_ALT, SJC_ALT inclusion/skipping junction counts per allele
- incLen and skpLen effective lengths of the inclusion and skipping isoforms
- sample_name.txt manifest (sample IDs, sample names, RNA-seq read lengths)
- VCF genotype files per sample for identifying heterozygous SNPs

## Outputs

- Table of significant ASAS events at FDR <= 10% with 19 documented columns (ExonID, IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen, pval, qval, psi levels, allele frequencies)
- Raw PAIRADISE p values and BH-adjusted q values per event
- Plots of significant ASAS events

## How to apply

Format the count table with columns ExonID, IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen mirroring the PAIRADISE output schema, treating the two alleles of each individual as paired replicates and individuals sharing a heterozygous SNP as replicates. Run the PAIRADISE statistical model (Step 5 of the pipeline: qsub -cwd -l h_vmem=10G qsub_files/pairadise_step5_stat.qsub) on the merged counts to obtain event-level p values. Apply Benjamini-Hochberg FDR correction and call events significant at qval <= 0.10 (the default). Optionally enable the sigma_i1 = sigma_i2 user option to increase power, recognizing the paper's conservative default left the variances unrestricted. In benchmark settings, compare against paired t test, paired Wilcoxon test, rMATS paired model, and Fisher's combined method using ROC AUC and TPR at 5% FPR, where PAIRADISE achieved the highest AUC (84%) and TPR (61%).

## Related tools

- **PAIRADISE** (Statistical framework and software performing the paired differential splicing test; installed via ./configure; make; make install and run as pipeline Step 5 (pairadise_step5_stat.qsub)) — github.com/Xinglab/PAIRADISE
- **STAR** (Spliced aligner used in Step 1 to align reads to haplotype-modified personal genomes before allele-specific counting (dependency >= 2.6.0a)) — https://github.com/alexdobin/STAR
- **rMATS** (Paired-model competitor whose correlation parameter can be unreliably estimated, used as a baseline for ROC AUC comparison (63% vs PAIRADISE 84%))
- **rPGA pipeline** (Provides the personalization, mapping, and read-assignment procedures (haplotype-modified personal genomes) that precede PAIRADISE testing)

## Examples

```
qsub -cwd -l h_vmem=10G qsub_files/pairadise_step5_stat.qsub
```

## Evaluation signals

- Output table exists and contains the 19 documented columns with qval <= 0.10 for all listed significant events
- Per-population significant ASAS event counts reproduce reported values under the conservative model: CEU 91, FIN 82, GBR 75, TSI 71, YRI 143
- In GM12878 single-individual analysis, the conservative model yields 116 significant ASAS events (rising to 133 with sigma_i1 = sigma_i2)
- In simulations (M=3 replicates, low variance, 5,000 events), the method attains ROC AUC of 84% and TPR at 5% FPR of 61%, above paired t test (74%/26%), paired Wilcoxon (68%/12%), rMATS paired (63%/23%), and Fisher's combined method (60%/4%)
- All reads assigned to a haplotype were uniquely aligned to both haplotype-modified personal genomes at the same location (QC filtering invariant)

## Limitations

- Requires a heterozygous SNP outside the alternative exon but close enough to the AS event for allele-specific read assignment on the same RNA-seq read; SNPs within or too distant from alternative exons cannot be analyzed with short reads
- Causal SNPs located within the alternative exons or nearby intronic regions are not tested — only tagging SNPs are identified
- The rMATS paired model shares similarities but its correlation parameter can be unreliably estimated, leading to inflated p values
- Approach and results were intentionally conservative; allowing sigma_i1 = sigma_i2 produces more events (e.g., 91 to 153 in Geuvadis CEU) but is left as a user option
- Individual-specific allelic differences depending on cis SNPs or trans-acting splicing regulators are not modeled via covariates; deferred to future work

## Evidence

- [results] In simulations with three replicates and low variance, PAIRADISE achieved the highest ROC AUC (84%) compared to the paired t test (74%), paired Wilcoxon test (68%), rMATS paired test (63%), and Fisher's combined method (60%).: [evidence span withheld — anchor status: not_found]
