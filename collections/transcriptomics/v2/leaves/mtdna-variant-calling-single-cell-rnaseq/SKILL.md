---
name: mtdna-variant-calling-single-cell-rnaseq
description: Use when scRNA-seq (e.g., Smart-seq2), scDNA-seq, or mtscATAC-seq BAM
  files from a population of single cells should be partitioned into mitochondrial
  clones, or when mtDNA variants must complement nuclear SNV/CNV clone calls for finer
  clonal resolution.
when_to_use_negative:
- The goal is nuclear-genome SNV clonal reconstruction — Monovar/Conbase assume a
  diploid context that is violated in the mitochondrial genome, and this skill targets
  the mtDNA chromosome (chrMT) instead
- Only a pre-computed SNP×cell count table exists — this skill's upstream step (cellsnp-lite
  piling from BAM) is unnecessary; go directly to MQuad
- Calling variants from read-coverage CNV profiles rather than true point-mutation
  propagation — the article notes CNV-inferred subclones may be less dynamic than
  mtDNA point-mutation subclones
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3168
tools:
- name: cellsnp-lite
  role: Piles up mitochondrial reads from BAM files to generate the SNP-by-cell AD/DP
    matrices consumed by MQuad (v1.2.1 in the paper)
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: MQuad
  role: Selects high-quality clonally informative mtDNA variants with a binomial mixture
    model and ΔBIC/Kneedle cutoff (v0.1.6 in the paper)
  repo: https://github.com/single-cell-genetics/MQuad
- name: vireoSNP
  role: Assigns cells to mitochondrial clones via variational inference using the
    MQuad-selected SNPs (v0.5.3 in the paper)
  repo: https://github.com/single-cell-genetics/vireo
- name: mgatk
  role: Alternative mtDNA variant caller for benchmarking; run with log10(VMR) ≥ −2
    and strand correlation coefficient ≥ 0.65 cutoffs
- name: STAR
  role: Aligns Smart-seq2 FASTQ files to the UCSC hg19 reference with default parameters
- name: Picard MarkDuplicates
  role: Marks and removes PCR duplicates from raw BAMs before variant detection (v2.18.9
    in the paper)
- name: kneed
  role: Determines the ΔBIC cutoff via the Kneedle knee point of the cumulative ΔBIC
    distribution
- name: Cardelino
  role: Provides nuclear SNV clone labels against which mtDNA clone assignments are
    cross-tabulated
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:28fe8f943fbd00d891753917e7570228c589e3fde7c76b7083054ce517c055b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/cellsnp-lite
  tools:
  - Cardelino
  - MQuad
  - Picard
  - Picard MarkDuplicates
  - STAR
  - cellsnp-lite
  - kneed
  - mgatk
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# mtdna-variant-calling-single-cell-rnaseq

## Summary

Calls clonally informative mitochondrial DNA variants from single-cell sequencing data by piling up mtDNA alleles with cellsnp-lite, selecting informative variants with MQuad's binomial mixture model (ΔBIC with a Kneedle knee-point cutoff), and reconstructing clones with vireoSNP.

## When to use

Use when scRNA-seq (e.g., Smart-seq2), scDNA-seq, or mtscATAC-seq BAM files from a population of single cells should be partitioned into mitochondrial clones, or when mtDNA variants must complement nuclear SNV/CNV clone calls for finer clonal resolution.

## When NOT to use

- The goal is nuclear-genome SNV clonal reconstruction — Monovar/Conbase assume a diploid context that is violated in the mitochondrial genome, and this skill targets the mtDNA chromosome (chrMT) instead
- Only a pre-computed SNP×cell count table exists — this skill's upstream step (cellsnp-lite piling from BAM) is unnecessary; go directly to MQuad
- Calling variants from read-coverage CNV profiles rather than true point-mutation propagation — the article notes CNV-inferred subclones may be less dynamic than mtDNA point-mutation subclones

## Inputs

- Analysis-ready single-cell BAM files aligned to hg19/mm10 (STAR + Picard MarkDuplicates for Smart-seq2 scRNA-seq)
- Raw FASTQ files for Smart-seq2 datasets requiring alignment
- SNP-by-cell matrices of alternate-allele counts (AD) and depths (DP) from cellsnp-lite (VCF/sparse-matrix form)
- Optional nuclear clone labels (e.g., Cardelino clone assignments) for cross-validation of mtDNA clones

## Outputs

- List of clonally informative mtDNA SNVs with ΔBIC values and Kneedle knee-point cutoff
- Cell-to-mitochondrial-clone assignments from vireoSNP
- Table of mean allelic frequency per mtDNA SNV per clone
- Confusion matrix of mtDNA clones vs nuclear (Cardelino) clone labels
- Integrated clonal tree placing mtDNA subclones (e.g., MT0/MT1) within nuclear clones

## How to apply

Align FASTQ files to the UCSC hg19 (or mm10) reference with STAR, then remove PCR duplicates with Picard MarkDuplicates (v2.18.9); for 10x/ATAC data use analysis-ready BAMs directly (mask NUMT regions from hg19 before CellRanger-ATAC). Pile up mitochondrial reads with cellsnp-lite v1.2.1 to produce SNP-by-cell AD and DP count matrices. For each variant, fit MQuad's two-component binomial mixture model against a one-component null, compute ΔBIC = BIC(M0) − BIC(M1), and set the selection cutoff at the Kneedle knee point of the cumulative ΔBIC distribution (kneed package); explicitly remove known false-positive variants in error-prone rCRS regions 302–315, 513–525, and 3105–3109 (Xu et al.). Assign cells to mitochondrial clones with vireoSNP (v0.5.3) variational inference using the MQuad-selected SNPs, then cross-tabulate assignments against nuclear clone labels (e.g., Cardelino) to integrate mtDNA subclones into the nuclear clonal tree. The ΔBIC model-selection approach is used because it is more robust than raw allele frequencies alone and reduces false positives in deeply sequenced Smart-seq2 data.

## Related tools

- **cellsnp-lite** (Piles up mitochondrial reads from BAM files to generate the SNP-by-cell AD/DP matrices consumed by MQuad (v1.2.1 in the paper)) — https://github.com/single-cell-genetics/cellsnp-lite
- **MQuad** (Selects high-quality clonally informative mtDNA variants with a binomial mixture model and ΔBIC/Kneedle cutoff (v0.1.6 in the paper)) — https://github.com/single-cell-genetics/MQuad
- **vireoSNP** (Assigns cells to mitochondrial clones via variational inference using the MQuad-selected SNPs (v0.5.3 in the paper)) — https://github.com/single-cell-genetics/vireo
- **mgatk** (Alternative mtDNA variant caller for benchmarking; run with log10(VMR) ≥ −2 and strand correlation coefficient ≥ 0.65 cutoffs)
- **STAR** (Aligns Smart-seq2 FASTQ files to the UCSC hg19 reference with default parameters)
- **Picard MarkDuplicates** (Marks and removes PCR duplicates from raw BAMs before variant detection (v2.18.9 in the paper))
- **kneed** (Determines the ΔBIC cutoff via the Kneedle knee point of the cumulative ΔBIC distribution)
- **Cardelino** (Provides nuclear SNV clone labels against which mtDNA clone assignments are cross-tabulated)

## Examples

```
cellSNP -s $BAM -b $BARCODE -O $OUT_DIR -p 22 --minMAF 0.1 --minCOUNT 100 --chrom chrMT
```

## Evaluation signals

- On the joxm fibroblast Smart-seq2 dataset, the run should recover ~24 informative SNVs assigning 77 cells into 3 clones, with nuclear clone 3 completely identical to mitochondrial clone 2
- The cumulative ΔBIC distribution should show a sharp knee, and the cutoff should be placed at that Kneedle knee point
- On the Kim ccRCC dataset, MQuad should find ~146 informative mtDNA variants with ~93% vireoSNP assignment concordance to source labels (PDX pRCC, PDX mRCC, Pt mRCC), vs mgatk's 312 mtSNVs at ~64%
- Clone-specific variants such as 11196G>A and the heteroplasmy-drifted 2691-region 2619A>T should appear in the per-clone mean allelic frequency table for the fibroblast data
- No selected variants should fall in the error-prone rCRS regions 302–315, 513–525, 3105–3109

## Limitations

- Very few computational methods exist for analyzing mitochondrial SNVs in conventional scRNA-seq data; mtDNA-specific callers like EMBLEM and mgatk were designed primarily for scATAC-seq and their quality on other data types was unevaluated before this work
- cellSNP mode 2 uses pysam's pileup() which filters duplicates and orphan reads by default, possibly causing unexpected coverage reduction; cellsnp-lite offers more flexible reads filtering
- PCR duplicates are not well flagged in CellRanger, so duplicate filtering defaults must be relaxed (--maxFLAG set to a large value) to avoid losing a substantial fraction of SNPs
- Uninformative and noisy mtDNA SNVs, combined with sequencing errors, confound clonal inference if not filtered by ΔBIC model selection

## Evidence

- [discussion] ΔBIC model selection: "it adopts a model selection approach in evaluating the informativeness of each variant, which is more robust than considering the raw allele frequencies alone"
- [discussion] Kneedle cutoff: "a ΔBIC cutoff for selection of SNPs is determined using the Kneedle algorithm36"
- [discussion] error-prone region masking: "We also explicitly removed frequent false positive variants occurring in error prone regions as described by Xu et al.22, including rCRS 302–315, 513–525, 3105–3109"
- [discussion] pipeline steps: "raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23"
- [discussion] mgatk comparison thresholds: "we used the suggested cutoffs to call variants: log10(VMR) ≥ −2 and strand correlation coefficient ≥0.65"
- [readme] recommended pipeline order: "use MQuad to differentiate informative mtDNA variants from noisy backbground"
