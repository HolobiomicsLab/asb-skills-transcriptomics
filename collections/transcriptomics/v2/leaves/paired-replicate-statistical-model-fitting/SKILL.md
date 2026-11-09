---
name: paired-replicate-statistical-model-fitting
description: Use when you have per-allele exon inclusion (IJC) and skipping (SJC)
  counts plus effective isoform lengths (incLen, skpLen) from replicate RNA-seq samples
  sharing a heterozygous SNP, and need to test for differential splicing between alleles
  across replicates.
when_to_use_negative:
- SNPs are located within the alternative exon or too distant from it to be co-detected
  on the same short RNA-seq read — PAIRADISE requires a heterozygous SNP outside the
  alternative exon but close enough for allele-specific read assignment
- Data are single-sample with no replicates or heterozygous-SNP-sharing individuals
  to aggregate — the paired-replicate model needs multiple paired observations for
  power
- Reads fail the uniqueness filter (not uniquely aligned to both haplotype-modified
  genomes at the same location) or multi-SNP reads with a majority-voting draw — these
  must be discarded before fitting
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0203
tools:
- name: PAIRADISE
  role: Provides the paired statistical model and the Step 5 fitting/visualization
    scripts; requires nloptr, doParallel, foreach, binom, ggplot2 R packages
  repo: https://github.com/Xinglab/PAIRADISE
- name: nloptr
  role: R optimization backend used by the PAIRADISE likelihood fit (version >= 1.2.1)
  repo: https://github.com/Xinglab/PAIRADISE
- name: STAR
  role: Spliced aligner (>= 2.6.0a) used upstream to map reads to haplotype-modified
    personal genomes before counting
  repo: https://github.com/alexdobin/STAR
- name: rMATS
  role: Conceptual comparison point — the PAIRADISE model shares similarities with
    the rMATS paired model for differential AS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:51ef8cf5fdc2eff064d70e941de8ca627e46f4670e83192d1e7b2158784ff8ed
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
  - Python
  - R
  - STAR
  - nloptr
  - rMATS
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# paired-replicate-statistical-model-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fit the PAIRADISE paired statistical model (R with nloptr optimization, parallelized via doParallel/foreach) to per-allele exon inclusion/skipping counts to detect allele-specific alternative splicing (ASAS) by aggregating signals across replicates/individuals, calling significant events at FDR <= 10% (Benjamini-Hochberg).

## When to use

Use when you have per-allele exon inclusion (IJC) and skipping (SJC) counts plus effective isoform lengths (incLen, skpLen) from replicate RNA-seq samples sharing a heterozygous SNP, and need to test for differential splicing between alleles across replicates.

## When NOT to use

- SNPs are located within the alternative exon or too distant from it to be co-detected on the same short RNA-seq read — PAIRADISE requires a heterozygous SNP outside the alternative exon but close enough for allele-specific read assignment
- Data are single-sample with no replicates or heterozygous-SNP-sharing individuals to aggregate — the paired-replicate model needs multiple paired observations for power
- Reads fail the uniqueness filter (not uniquely aligned to both haplotype-modified genomes at the same location) or multi-SNP reads with a majority-voting draw — these must be discarded before fitting

## Inputs

- Merged PAIRADISE counts table with ExonID, IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen per event and replicate
- sample_name.txt (sample IDs and RNA-seq read lengths; replicates mapped to common sample in columns 1–2)
- Allele-specific read assignments from haplotype-modified personal genome alignment (rPGA-style, unique-alignment filtered)

## Outputs

- Table of significant ASAS events at FDR <= 10% with ExonID, per-allele IJC/SJC counts, incLen, skpLen, raw pval, and adjusted FDR
- Plots of significant events (visualization step)

## How to apply

Run Step 5 of the PAIRADISE pipeline (statistical modeling and visualization) on the merged counts table produced by Steps 1–4, which contain IJC/SJC counts for reference and alternative alleles and incLen/skpLen effective lengths. The paired model treats the two alleles of an individual as paired observations and individuals sharing a heterozygous SNP as replicates, optimizing the likelihood with nloptr in parallel (doParallel/foreach). Optionally set sigma_i1 = sigma_i2 as a user option to gain power (e.g., GM12878 significant events rose from 116 to 133; Geuvadis CEU from 91 to 153), at the cost of the paper's intentionally conservative default. Call events significant at the default FDR <= 10% using Benjamini-Hochberg adjustment; the model shares similarities with the rMATS paired model but avoids the unreliable correlation-parameter estimation that inflates p values.

## Related tools

- **PAIRADISE** (Provides the paired statistical model and the Step 5 fitting/visualization scripts; requires nloptr, doParallel, foreach, binom, ggplot2 R packages) — https://github.com/Xinglab/PAIRADISE
- **nloptr** (R optimization backend used by the PAIRADISE likelihood fit (version >= 1.2.1)) — https://github.com/Xinglab/PAIRADISE
- **STAR** (Spliced aligner (>= 2.6.0a) used upstream to map reads to haplotype-modified personal genomes before counting) — https://github.com/alexdobin/STAR
- **rMATS** (Conceptual comparison point — the PAIRADISE model shares similarities with the rMATS paired model for differential AS)

## Examples

```
qsub -cwd -l h_vmem=10G qsub_files/pairadise_step5_stat.qsub
```

## Evaluation signals

- Output table contains all required columns (ExonID, IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen, pval) with adjusted FDR <= 0.10 for retained rows
- Every called event corresponds to a heterozygous SNP outside the alternative exon; events with intra-exonic or too-distant SNPs should be absent
- Running with sigma_i1 = sigma_i2 yields more significant events than the default (e.g., 116→133 GM12878, 91→153 Geuvadis CEU), consistent with the conservative default
- Fitted events show consistent per-allele PSI differences across replicates after pairing, rather than single-sample signals
- P values are not inflated relative to expectation (the model should avoid the rMATS-like correlation-parameter inflation problem)

## Limitations

- Requires a heterozygous SNP outside the alternative exon but close enough to the AS event for allele-specific assignment on the same RNA-seq read; SNPs within or too distant from alternative exons cannot be analyzed with short reads
- Causal SNPs affecting splicing may lie within the alternative exon or nearby intronic regions and are not tested — only tagging SNPs are identified
- Unreliable estimation of the correlation parameter can lead to inflated p values (the rMATS issue PAIRADISE is designed to avoid); default results are intentionally conservative
- ASAS signals may vary across individuals (cis SNPs, trans-acting regulators); modeling individual-specific allelic differences via covariates is deferred to future work and would need a much larger population sample size

## Evidence
