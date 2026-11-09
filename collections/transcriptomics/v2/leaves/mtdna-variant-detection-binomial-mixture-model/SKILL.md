---
name: mtdna-variant-detection-binomial-mixture-model
description: Use when single-cell RNA-seq, DNA-seq, or ATAC-seq BAM files must be
  genotyped across the mitochondrial genome to call heteroplasmic variants and select
  the subset informative for clonal substructure inference.
when_to_use_negative:
- Calling nuclear SNVs, where diploid assumptions hold — use nuclear SNV callers such
  as Monovar or Conbase, since the mitochondrial genome violates diploidy
- scATAC-seq data processed without NUMT masking — the article masks NUMT regions
  from the standard hg19 reference before CellRanger-ATAC, and mtDNA-specific methods
  like mgatk/EMBLEM were designed primarily for scATAC-seq quality
- scDNA-seq data below ~3,000 mitochondrial reads per cell, where downsampling shows
  a linear drop in assignable cells, informative variants detected, and clonal assignment
  concordance
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: MQuad
  role: Fits a binomial mixture model per mtDNA variant to assess heteroplasmy vs
    background noise and selects informative variants via a ΔBIC Kneedle cutoff; version
    v0.1.6 in the paper
  repo: github.com/single-cell-genetics/MQuad
- name: cellsnp-lite
  role: Piles up raw reads from BAM files to generate the SNP-by-cell AD/DP count
    matrices that MQuad consumes; version v1.2.1 in the paper
  repo: github.com/single-cell-genetics/cellsnp-lite
- name: cellSNP
  role: Python predecessor of cellsnp-lite used for the same mtDNA pile-up step in
    the recommended pipeline (preprocessing_cmds.sh example in the MQuad repo)
  repo: github.com/single-cell-genetics/cellSNP
- name: vireoSNP
  role: 'Downstream clonal reconstruction: uses variational inference to assign cells
    to clones based on the MQuad-selected mtDNA SNPs; version v0.5.3 in the paper'
  repo: github.com/single-cell-genetics/vireo
- name: Picard MarkDuplicates
  role: Marks and removes PCR duplicates from raw BAM files before variant detection
    in the paper's preprocessing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:043e0b6721f4664da9fb0d3b9c5281c3684350876391a697dad6d0a567fab5aa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MQuad
  - Picard MarkDuplicates
  - cellSNP
  - cellsnp-lite
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mtdna-variant-detection-binomial-mixture-model

## Summary

Detect clonally informative mitochondrial DNA variants in single-cell omics data by first piling up mtDNA reads into SNP-by-cell AD/DP matrices with cellsnp-lite/cellSNP, then separating informative heteroplasmic variants from noise with MQuad's binomial mixture model and a ΔBIC knee-point cutoff. This mtDNA-specific approach avoids the diploid assumption that breaks nuclear SNV callers on the mitochondrial genome.

## When to use

Use when single-cell RNA-seq, DNA-seq, or ATAC-seq BAM files must be genotyped across the mitochondrial genome to call heteroplasmic variants and select the subset informative for clonal substructure inference.

## When NOT to use

- Calling nuclear SNVs, where diploid assumptions hold — use nuclear SNV callers such as Monovar or Conbase, since the mitochondrial genome violates diploidy
- scATAC-seq data processed without NUMT masking — the article masks NUMT regions from the standard hg19 reference before CellRanger-ATAC, and mtDNA-specific methods like mgatk/EMBLEM were designed primarily for scATAC-seq quality
- scDNA-seq data below ~3,000 mitochondrial reads per cell, where downsampling shows a linear drop in assignable cells, informative variants detected, and clonal assignment concordance

## Inputs

- Analysis-ready single-cell BAM file(s) with cell barcodes (10x scRNA-seq, scDNA-seq such as MKN-45 aligned to hg38, or Smart-seq2 scRNA-seq)
- Cell barcode list (-b) matching the BAM
- Mitochondrial chromosome target ('chrMT') or candidate SNP VCF/region list (-R)
- Cellranger/CellRanger-ATAC processed BAMs (NUMT-masked reference for mtscATAC-seq)

## Outputs

- SNP-by-cell pile-up matrices of allele AD/DP read counts (cellsnp-lite/cellSNP output)
- List of clonally informative mtDNA variants with binomial mixture model fit statistics and ΔBIC scores
- Cell-by-variant genotype matrices for downstream vireoSNP clonal assignment
- Per-clone cell assignments from vireoSNP using the MQuad-selected SNPs

## How to apply

Pile up raw reads from BAM files with cellsnp-lite (v1.2.1; scDNA/ATAC) or cellSNP with '--chrom chrMT' (scRNA-seq) to produce a SNP-by-cell matrix of AD and DP counts per allele, targeting the mitochondrial chromosome. Feed the pile-up to MQuad (v0.1.6), which fits a binomial mixture model to the per-cell allele frequencies of each variant to assess heteroplasmy against background noise, and select informative variants using a ΔBIC cutoff determined with the Kneedle algorithm (kneed v0.7.0). Explicitly remove known error-prone rCRS regions 302–315, 513–525, and 3105–3109 before interpretation. For comparison baselines with mgatk apply its suggested cutoffs log10(VMR) ≥ −2 and strand correlation coefficient ≥ 0.65. Pass the selected SNPs to vireoSNP (v0.5.3) variational inference for clonal reconstruction, and validate by checking clonal concordance against full-depth labels; on 10x scDNA-seq data the MKN-45 downsampling analysis supports a minimum of 3,000 mitochondrial reads per cell.

## Related tools

- **MQuad** (Fits a binomial mixture model per mtDNA variant to assess heteroplasmy vs background noise and selects informative variants via a ΔBIC Kneedle cutoff; version v0.1.6 in the paper) — github.com/single-cell-genetics/MQuad
- **cellsnp-lite** (Piles up raw reads from BAM files to generate the SNP-by-cell AD/DP count matrices that MQuad consumes; version v1.2.1 in the paper) — github.com/single-cell-genetics/cellsnp-lite
- **cellSNP** (Python predecessor of cellsnp-lite used for the same mtDNA pile-up step in the recommended pipeline (preprocessing_cmds.sh example in the MQuad repo)) — github.com/single-cell-genetics/cellSNP
- **vireoSNP** (Downstream clonal reconstruction: uses variational inference to assign cells to clones based on the MQuad-selected mtDNA SNPs; version v0.5.3 in the paper) — github.com/single-cell-genetics/vireo
- **Picard MarkDuplicates** (Marks and removes PCR duplicates from raw BAM files before variant detection in the paper's preprocessing)

## Examples

```
cellSNP -s $BAM -b $BARCODE -O $OUT_DIR -p 22 --minMAF 0.1 --minCOUNT 100 --chrom chrMT
```

## Evaluation signals

- The ΔBIC distribution for each variant shows a clear knee point identified by the Kneedle algorithm, and only variants above that cutoff are retained
- No selected variants fall in the error-prone rCRS regions 302–315, 513–525, or 3105–3109
- The SNP-by-cell pile-up contains the expected AD/DP count structure per allele and cell barcode (cellsnp-lite output format)
- Downstream vireoSNP clonal assignment concordance relative to full-depth clone labels stays consistent when depth ≥ 3,000 mitochondrial reads per cell on 10x scDNA-seq data
- MQuad sensitivity/specificity outperforms comparator callers (mgatk with log10(VMR) ≥ −2 and strand correlation ≥ 0.65) on the same data

## Limitations

- cellSNP mode 2 uses pysam pileup() which filters duplicates and orphan reads by default and may cause unexpected coverage reduction; PCR duplicate filtering is off by default (--maxFLAG) because duplicates are not well flagged in CellRanger, so flag settings must be tuned per data type
- Nuclear SNV callers assume diploid context which is violated in the mitochondrial genome; conversely, mtDNA-specific methods (EMBLEM, mgatk) were designed primarily for scATAC-seq and their quality in other data types was unevaluated, motivating MQuad's cross-assay design
- MQuad was tested with Python 3.8.8 on Windows 10 and CentOS Linux 7; cellsnp-lite was still described as 'at testing stage so might be unstable' in the MQuad README

## Evidence

- [discussion] MQuad uses a binomial mixture model with ΔBIC selection: "MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model"
- [discussion] ΔBIC cutoff determined with Kneedle: "a ΔBIC cutoff for selection of SNPs is determined using the Kneedle algorithm36"
- [discussion] cellsnp-lite pile-up of BAM reads generates the SNP-by-cell matrix: "raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23. This generates a SNP-by-cell matrix"
- [discussion] Error-prone rCRS regions removed explicitly: "We also explicitly removed frequent false positive variants occurring in error prone regions as described by Xu et al.22, including rCRS 302–315, 513–525, 3105–3109"
- [readme] MQuad's purpose per README: binomial mixture model against background noise: "It uses a binomial mixture model to assess the heteroplasmy of mtDNA variants among background noise."
- [abstract] MQuad accuracy claim: "Here we present an open source computational tool MQuad that accurately calls clonally informative mtDNA variants in a population of single cells"
