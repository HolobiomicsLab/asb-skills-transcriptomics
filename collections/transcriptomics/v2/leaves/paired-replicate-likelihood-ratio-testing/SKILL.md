---
name: paired-replicate-likelihood-ratio-testing
description: Use when per-event allele-specific inclusion/skipping counts (IJC/SJC
  for REF and ALT alleles) are available for multiple individuals sharing a heterozygous
  SNP and you need a combined ASAS p-value via a paired-replicate LRT on δi = 0.
when_to_use_negative:
- The heterozygous SNP is inside the alternative exon or too distant to co-occur on
  the same short RNA-seq read — the event cannot be allele-assigned for this test
- You need to test the causal SNP itself within the exon or nearby intronic splicing
  regions — PAIRADISE only tests tagging SNPs outside the alternative exon
- Input is genotype–phenotype sQTL association data rather than paired REF/ALT allele
  counts per individual — the within-individual paired allelic model does not apply
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0625
tools:
- name: PAIRADISE
  role: Implements the paired hierarchical model (binomial + logit-normal with shared
    subject effect alpha_ik) and the nloptr-based LRT in Step 5 (statistical modeling
    and visualization)
  repo: github.com/Xinglab/PAIRADISE
- name: rMATS
  role: Related paired differential-AS model that PAIRADISE shares similarities with;
    PAIRADISE avoids its unreliable correlation-parameter estimation that inflates
    p values
- name: nloptr
  role: R package (>= 1.2.1) used for numerical optimization of the PAIRADISE hierarchical
    likelihood per event
- name: doParallel/foreach
  role: R packages (>= 1.0.14 / >= 1.4.4) used to parallelize model fitting over ASAS
    events
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:75ce76783c2ac50748c763228726585db44a658840e061f62ffe5f159909ab3e
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
  - doParallel/foreach
  - nloptr
  - rMATS
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# paired-replicate-likelihood-ratio-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

The PAIRADISE hierarchical paired-replicate model: binomial read-count likelihoods plus logit-normal isoform ratios where both alleles of an individual share the subject effect alpha_ik; after integrating it out, the two alleles' logit exon inclusion levels follow a bivariate normal with mean [μi, μi+δi], and significance comes from a likelihood-ratio test of δi = 0 (chi-squared, 1 df).

## When to use

Use when per-event allele-specific inclusion/skipping counts (IJC/SJC for REF and ALT alleles) are available for multiple individuals sharing a heterozygous SNP and you need a combined ASAS p-value via a paired-replicate LRT on δi = 0.

## When NOT to use

- The heterozygous SNP is inside the alternative exon or too distant to co-occur on the same short RNA-seq read — the event cannot be allele-assigned for this test
- You need to test the causal SNP itself within the exon or nearby intronic splicing regions — PAIRADISE only tests tagging SNPs outside the alternative exon
- Input is genotype–phenotype sQTL association data rather than paired REF/ALT allele counts per individual — the within-individual paired allelic model does not apply

## Inputs

- Merged count table from PAIRADISE Step 4 (merging the counts) with per-event IJC_REF, SJC_REF, IJC_ALT, SJC_ALT inclusion/skipping counts
- incLen and skpLen effective isoform lengths per event
- sample_name.txt (sample IDs, biological replicate groupings, RNA-seq read lengths)

## Outputs

- pairadise_significant_asas_events.tsv with columns ExonID, IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen, pval, qval, IncLevel1/2, AvgIncLevel1/2, IncLevelDifference, AvgTotalCount1/2, SampleName, RefAltAllele, AF
- Plots of the significant ASAS events (Step 5 visualization)

## How to apply

Load the merged count table from PAIRADISE Step 4 (per-event IJC_REF, SJC_REF, IJC_ALT, SJC_ALT counts with effective isoform lengths incLen/skpLen) plus sample_name.txt, which maps biological replicate IDs to samples for the paired design. Fit the hierarchical model per event: binomial likelihoods on length-normalized inclusion/skipping counts for each allele, with logit(psi) normal distributions for reference and alternative alleles that share the subject effect alpha_ik; after integrating alpha_ik out, the two alleles' logit inclusion levels follow a bivariate normal with mean [μi, μi+δi] and covariance σi²+diag([σi1², σi2²]). Maximize the likelihood numerically with nloptr (>= 1.2.1), parallelizing over events with doParallel (>= 1.0.14) and foreach (>= 1.4.4). Compute the raw pval by a likelihood-ratio test of the null allelic-difference hypothesis δi = 0 against a chi-squared distribution with 1 degree of freedom, then Benjamini-Hochberg-adjust to qval and retain events at the default FDR <= 10%. Unlike the rMATS paired model it resembles, PAIRADISE omits a correlation parameter whose unreliable estimation inflates p values; restricting σi1 = σi2 is a user option that increases detections (e.g., 116 to 133 in GM12878).

## Related tools

- **PAIRADISE** (Implements the paired hierarchical model (binomial + logit-normal with shared subject effect alpha_ik) and the nloptr-based LRT in Step 5 (statistical modeling and visualization)) — github.com/Xinglab/PAIRADISE
- **rMATS** (Related paired differential-AS model that PAIRADISE shares similarities with; PAIRADISE avoids its unreliable correlation-parameter estimation that inflates p values)
- **nloptr** (R package (>= 1.2.1) used for numerical optimization of the PAIRADISE hierarchical likelihood per event)
- **doParallel/foreach** (R packages (>= 1.0.14 / >= 1.4.4) used to parallelize model fitting over ASAS events)

## Examples

```
qsub -cwd -l h_vmem=10G qsub_files/pairadise_step5_stat.qsub
```

## Evaluation signals

- Output file pairadise_significant_asas_events.tsv exists and contains the required PAIRADISE columns (e.g., ExonID, pval, qval, IncLevelDifference)
- Every reported event has qval <= 0.10 (default FDR threshold)
- Each pval is a valid LRT p-value against chi-squared with 1 df (0 <= pval <= 1), and BH-adjusted qval is monotone non-decreasing with pval
- IncLevel1/IncLevel2 and IncLevelDifference are consistent with the count and length columns (IJC, SJC, incLen, skpLen) for the REF and ALT alleles
- Sensitivity check: rerunning with the sigma_i1 = sigma_i2 user option should increase the significant-event count (e.g., 116 to 133 in GM12878)

## Limitations

- Requires a heterozygous SNP outside the alternative exon but close enough to be detected on the same RNA-seq read; SNPs located within or too distant from alternative exons cannot be analyzed with short-read RNA-seq data
- Causal SNPs affecting splicing regulation may not be tested — only tagging SNPs are identified
- Results are intentionally conservative; allowing sigma_i1 = sigma_i2 (left as a user option) would produce more significant events
- Individual-specific allelic differences depending on covariates (cis SNPs or trans-acting splicing regulators) are not modeled; this extension is deferred to future work

## Evidence

- [methods] Both allele groups share the subject effect αik; after integrating it out, the two alleles' logit exon inclusion levels follow a bivariate normal distribution with mean [μi, μi+δi]: [evidence span withheld — anchor status: not_found]
- [methods] Run a likelihood-ratio test of the null allelic-difference hypothesis delta_i = 0 against a chi-squared distribution with 1 degree of freedom to obtain a raw p-value (pval) for each event.: [evidence span withheld — anchor status: not_found]
