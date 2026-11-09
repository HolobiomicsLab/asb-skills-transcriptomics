---
name: binomial-mixture-variant-scoring
description: Use when you have a SNP-by-cell AD/DP count matrix (e.g. from cellsnp-lite/cellSNP
  over mtDNA) and need to separate clone-discriminative heteroplasmic mtDNA variants
  from noisy background variants before clonal assignment.
when_to_use_negative:
- Variants come from nuclear genome calling where a diploid context holds (e.g. Monovar/Conbase
  assumptions), not the mitochondrial genome
- Data are 10x scRNA-seq needing per-UMI genotyping with strict duplicate handling
  — cellsnp-lite in mode 2 may need tuned read filtering since duplicates are often
  poorly flagged in CellRanger output
- The goal is donor demultiplexing from common germline SNPs (vireoSNP donor deconvolution)
  rather than somatic clonal mtDNA variant discovery
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: MQuad
  role: Fits the 1- vs 2-component binomial mixture model per variant, computes ΔBIC,
    and selects informative variants at the Kneedle knee-point cutoff
  repo: https://github.com/single-cell-genetics/MQuad
- name: cellsnp-lite
  role: Piles up mtDNA alleles from BAM files into the SNP-by-cell AD/DP matrices
    that MQuad scores
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: cellSNP
  role: Python predecessor of cellsnp-lite used for pileup; preprocessing_cmds.sh
    in MQuad's examples folder shows the shell commands
  repo: https://github.com/single-cell-genetics/cellSNP
- name: vireoSNP
  role: Downstream consumer of MQuad-selected variants; assigns cells to mitochondrial
    clones via variational inference (v0.5.3)
  repo: https://github.com/single-cell-genetics/vireo
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:8a589a4f82891094d587f9fbf3e096208ad15b3678a744129e794ccaf1fb0f20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/MQuad
  tools:
  - MQuad
  - Monovar
  - NEAT-genReads
  - Picard
  - STAR
  - SomatoSim
  - cellSNP
  - cellsnp-lite
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# binomial-mixture-variant-scoring

## Summary

Score mtDNA variants for clonal informativeness using MQuad's binomial mixture model, which fits one-component vs two-component models per variant and ranks them by ΔBIC, then selects informative variants at a Kneedle knee-point cutoff for downstream clonal reconstruction.

## When to use

Use when you have a SNP-by-cell AD/DP count matrix (e.g. from cellsnp-lite/cellSNP over mtDNA) and need to separate clone-discriminative heteroplasmic mtDNA variants from noisy background variants before clonal assignment.

## When NOT to use

- Variants come from nuclear genome calling where a diploid context holds (e.g. Monovar/Conbase assumptions), not the mitochondrial genome
- Data are 10x scRNA-seq needing per-UMI genotyping with strict duplicate handling — cellsnp-lite in mode 2 may need tuned read filtering since duplicates are often poorly flagged in CellRanger output
- The goal is donor demultiplexing from common germline SNPs (vireoSNP donor deconvolution) rather than somatic clonal mtDNA variant discovery

## Inputs

- SNP-by-cell AD and DP count matrices (VCF plus sparse matrix format) from cellsnp-lite/cellSNP pileup of mtDNA
- Analysis-ready single-cell BAM files (e.g. STAR-aligned, Picard MarkDuplicates-deduplicated Smart-seq2 BAMs)
- Optionally, a list of mtDNA positions/regions to pile up (chrom 'chrMT')

## Outputs

- Per-variant ΔBIC informativeness scores and 1- vs 2-component model fits
- ΔBIC cumulative distribution function plot with Kneedle knee-point cutoff
- Set of selected clonally informative mtDNA variants for vireoSNP clone assignment

## How to apply

First pile up mtDNA alleles per cell from analysis-ready BAMs with cellsnp-lite (v1.2.1) to produce the SNP-by-cell AD/DP matrices MQuad consumes. For each variant, fit a binomial mixture model comparing a 1-component null (M0, background noise only) against a 2-component alternative (M1, an informative subpopulation of cells), and compute ΔBIC = BIC(M0) − BIC(M1) as the informativeness score. Plot the cumulative distribution function of ΔBIC across variants and set the selection cutoff at the knee point detected with the Kneedle algorithm (kneed package), rather than thresholding on raw allele frequencies, which the paper reports is less robust. Remove known false-positive variants in error-prone rCRS regions 302–315, 513–525, and 3105–3109 before final selection. Feed the selected informative variants into vireoSNP for clonal assignment; benchmark validation showed this approach reaches AUPRC ≈ 0.976 (default simulation, M=10 variants per clone) and outperforms mgatk (log10(VMR) ≥ −2, strand correlation ≥ 0.65 cutoffs) and Monovar.

## Related tools

- **MQuad** (Fits the 1- vs 2-component binomial mixture model per variant, computes ΔBIC, and selects informative variants at the Kneedle knee-point cutoff) — https://github.com/single-cell-genetics/MQuad
- **cellsnp-lite** (Piles up mtDNA alleles from BAM files into the SNP-by-cell AD/DP matrices that MQuad scores) — https://github.com/single-cell-genetics/cellsnp-lite
- **cellSNP** (Python predecessor of cellsnp-lite used for pileup; preprocessing_cmds.sh in MQuad's examples folder shows the shell commands) — https://github.com/single-cell-genetics/cellSNP
- **vireoSNP** (Downstream consumer of MQuad-selected variants; assigns cells to mitochondrial clones via variational inference (v0.5.3)) — https://github.com/single-cell-genetics/vireo

## Examples

```
mquad run --cellSNP_out cellsnp_out/ -o mquad_out/ --null_reads 10 --p_val 0.1 --runID MQuad_run
```

## Evaluation signals

- ΔBIC cumulative distribution function shows a sharp knee used as the cutoff (as reported for the Kim ccRCC dataset)
- In simulation benchmarks (M=10 variants per clone, μ=0.1, branched lineage tree), MQuad should reach AUPRC ≈ 0.976 and AUROC ≈ 1.00, exceeding mgatk (AUPRC 0.800) and Monovar (AUPRC 0.147)
- On the ccRCC dataset, ~146 informative mtDNA variants should be selected and yield ~93% clonal assignment concordance with source labels via vireoSNP, vs mgatk's 312 mtSNVs at 64% concordance
- No selected variants fall in error-prone rCRS regions 302–315, 513–525, 3105–3109
- Input matrices conform to MQuad's expected cellsnp-lite AD/DP VCF/sparse-matrix format

## Limitations

- Existing mtDNA-specific alternatives (EMBLEM, mgatk) were designed primarily for scATAC-seq, and nuclear SNV callers (Monovar, Conbase) assume a diploid context violated in mtDNA — comparisons should account for this mismatch
- cellSNP mode 2 uses pysam pileup() which filters duplicates and orphan reads by default and may cause unexpected coverage reduction; cellsnp-lite offers more flexible read filtering
- PCR duplicates may not be well flagged in CellRanger BAMs, so default duplicate filtering can lose a substantial fraction of SNPs
- MQuad v0.1.6 tested on Python 3.8.8 (Windows 10, CentOS 7); cellsnp-lite described as 'still at testing stage so might be unstable' in the MQuad README

## Evidence

- [discussion] MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model: "MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model"
- [discussion] a ΔBIC cutoff for selection of SNPs is determined using the Kneedle algorithm: "a ΔBIC cutoff for selection of SNPs is determined using the Kneedle algorithm36"
- [discussion] it adopts a model selection approach in evaluating the informativeness of each variant, which is more robust than considering the raw allele frequencies alone: "it adopts a model selection approach in evaluating the informativeness of each variant, which is more robust than considering the raw allele frequencies alone"
- [readme] use MQuad to differentiate informative mtDNA variants from noisy backbground: "use MQuad to differentiate informative mtDNA variants from noisy backbground"
- [discussion] We also explicitly removed frequent false positive variants occurring in error prone regions as described by Xu et al.: "We also explicitly removed frequent false positive variants occurring in error prone regions as described by Xu et al.22, including rCRS 302–315, 513–525, 3105–3109"
