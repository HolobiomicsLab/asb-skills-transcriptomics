---
name: allelic-differential-splicing-statistical-modeling
description: Use when merged per-allele inclusion and skipping counts (IJC/SJC for
  reference and alternative alleles across samples) are available and the goal is
  to call significant ASAS events at FDR ≤ 10% from replicate or population-scale
  RNA-seq data.
when_to_use_negative:
- The AS event has no heterozygous SNP outside the alternative exon close enough to
  be detected on the same RNA-seq read — SNPs located within or too distant from alternative
  exons cannot be analyzed by PAIRADISE with short-read data
- You need to identify the causal splicing-regulatory SNP — PAIRADISE only tests tagging
  SNPs, not causal SNPs located within the alternative exon or nearby intronic regions
- The goal is sQTL association testing from unpaired genotype-phenotype data (e.g.,
  GLiMMPS-style analysis) rather than allele-specific within-individual splicing comparison
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PAIRADISE
  role: Paired statistical model fitted in R to merged per-allele counts; reports
    FDR-adjusted significant ASAS events
  repo: github.com/Xinglab/PAIRADISE
- name: nloptr
  role: R dependency used by the PAIRADISE model for numerical optimization of the
    likelihood
  repo: github.com/Xinglab/PAIRADISE
- name: doParallel
  role: R dependency enabling parallel fitting of the model across AS event-SNP pairs
  repo: github.com/Xinglab/PAIRADISE
- name: ggplot2
  role: R dependency used to create plots of the significant ASAS events
  repo: github.com/Xinglab/PAIRADISE
- name: STAR
  role: Upstream aligner (>= 2.6.0a) that produced the haplotype-specific alignments
    feeding the counts the model consumes
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:a39cb5188001345b03af33d6ef2dbea58872bee6bb1021ceb2235f7893db9bd7
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
  - binom
  - doParallel
  - foreach
  - ggplot2
  - nloptr
  - numpy
  - pybedtools
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# allelic-differential-splicing-statistical-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs the PAIRADISE paired statistical model (R) on allele-stratified exon inclusion/skipping counts to detect allele-specific alternative splicing (ASAS), aggregating signal across replicates/individuals sharing a heterozygous SNP and reporting Benjamini-Hochberg FDR-significant AS event-SNP pairs.

## When to use

Use when merged per-allele inclusion and skipping counts (IJC/SJC for reference and alternative alleles across samples) are available and the goal is to call significant ASAS events at FDR ≤ 10% from replicate or population-scale RNA-seq data.

## When NOT to use

- The AS event has no heterozygous SNP outside the alternative exon close enough to be detected on the same RNA-seq read — SNPs located within or too distant from alternative exons cannot be analyzed by PAIRADISE with short-read data
- You need to identify the causal splicing-regulatory SNP — PAIRADISE only tests tagging SNPs, not causal SNPs located within the alternative exon or nearby intronic regions
- The goal is sQTL association testing from unpaired genotype-phenotype data (e.g., GLiMMPS-style analysis) rather than allele-specific within-individual splicing comparison

## Inputs

- Merged count matrix from Step 4 containing exon inclusion counts (IJC) and skipping counts (SJC) for reference and alternative alleles per sample
- sample_name.txt (sample IDs and RNA-seq read lengths) from the data directory
- Effective isoform lengths (incLen, skpLen)
- PAIRADISE R installation (nloptr >= 1.2.1, doParallel >= 1.0.14, foreach >= 1.4.4, binom >= 1.1.1, ggplot2 >= 3.1.1)

## Outputs

- Table of significant ASAS events at FDR <= 10% with columns ExonID, IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen, pval (raw PAIRADISE p-value)
- Plots of the significant ASAS events
- FDR-adjusted (Benjamini-Hochberg) output table

## How to apply

After Step 4 has merged per-sample counts, submit the Step 5 job (`qsub -cwd -l h_vmem=10G qsub_files/pairadise_step5_stat.qsub`), which fits the PAIRADISE paired model for differential AS using R dependencies nloptr, doParallel, and foreach, testing one SNP at a time for evidence of ASAS for a given exon. Use the default conservative setting that does NOT restrict sigma_i1 = sigma_i2; restricting them is a user option that raises power (GM12878: 116 → 133 events) but the paper's reported results are conservative. The model treats the two alleles of an individual as paired and multiple individuals sharing a heterozygous SNP as replicates, aggregating allelic differences across samples. Output is filtered with Benjamini-Hochberg FDR ≤ 10%, and ggplot2 plots of significant events are generated alongside the output table.

## Related tools

- **PAIRADISE** (Paired statistical model fitted in R to merged per-allele counts; reports FDR-adjusted significant ASAS events) — github.com/Xinglab/PAIRADISE
- **nloptr** (R dependency used by the PAIRADISE model for numerical optimization of the likelihood) — github.com/Xinglab/PAIRADISE
- **doParallel** (R dependency enabling parallel fitting of the model across AS event-SNP pairs) — github.com/Xinglab/PAIRADISE
- **ggplot2** (R dependency used to create plots of the significant ASAS events) — github.com/Xinglab/PAIRADISE
- **STAR** (Upstream aligner (>= 2.6.0a) that produced the haplotype-specific alignments feeding the counts the model consumes) — https://github.com/alexdobin/STAR

## Examples

```
qsub -cwd -l h_vmem=10G qsub_files/pairadise_step5_stat.qsub
```

## Evaluation signals

- For the GM12878 six-replicate run with default settings (sigma_i1 and sigma_i2 not restricted), the Benjamini-Hochberg-adjusted table must contain exactly 116 significant ASAS event-SNP pairs at FDR ≤ 10%, matching the paper
- Every output row must carry the schema columns ExonID, IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen, pval
- All reported events must pass Benjamini-Hochberg FDR <= 10%; the default significance threshold is pre-set in the software
- Running with the sigma_i1 = sigma_i2 option should increase the event count (116 → 133 for GM12878); if the unrestricted default yields more events than the restricted option, something is misconfigured
- Downstream of Step 4, the count matrix must be complete for every sample before Step 5 is submitted

## Limitations

- Requires a heterozygous SNP outside the alternative exon but close enough to the AS event to be detected on the same RNA-seq read; SNPs located within or too distant from alternative exons cannot be analyzed with short-read RNA-seq data
- Only tagging SNPs are identified — causal SNPs located within alternative exons or nearby intronic regions are not tested by PAIRADISE
- Results are intentionally conservative: not restricting sigma_i1 = sigma_i2 yields fewer events (116 vs 133 in GM12878); the rMATS-like paired model's correlation parameter can be unreliably estimated, leading to inflated p values
- Individual-specific allelic differences depending on covariates (e.g., cis SNPs or trans-acting splicing regulators) are not modeled; deferred to future work

## Evidence

- [methods] Step 5 definition: "Finally, we run the PAIRADISE statistical model and create plots of the significant events."
- [readme] Default FDR threshold and output schema: "PAIRADISE outputs a table of significant ASAS events (default significance is set to FDR <= 10%). Each row of the output table corresponds to a significant ASAS event"
- [discussion] Sigma equality user option changes power: "By running PAIRADISE with σi1=σi2, the number of significant ASAS events increased from 116 to 133 in GM12878"
- [discussion] SNP placement limitation: "it requires a heterozygous SNP outside of the alternative exon to enable allele-specific read assignment, but this SNP also needs to be close enough to the AS event"
- [readme] Paired replicate framing: "By treating the two alleles of an individual as paired, and multiple individuals sharing a heterozygous SNP as replicates, PAIRADISE formulates ASAS detection as a statistical problem"
