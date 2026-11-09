---
name: per-cell-umi-count-statistical-testing
description: Use when two 10x scRNA-seq libraries must be compared for equal per-cell
  capture depth (genes/UMIs per cell), e.g. fresh vs cryopreserved PBMCs, after matching
  mean reads per cell by downsampling.
when_to_use_negative:
- The gene-barcode matrices were generated under unequal mean reads per cell — downsample
  first, otherwise the count comparison is confounded by depth
- The question is about expression levels rather than detection depth/sensitivity
  — per-cell UMI testing does not address differential expression
- Input is already an aggregated per-sample summary (e.g. median counts) — the t-test
  requires per-cell values
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Re-runs single-cell 3' gene counting on downsampled reads to regenerate matched
    gene-barcode matrices whose per-cell counts are then tested
  repo: 10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 (cDNA insert) to hg19 during the Cell Ranger counting step
- name: Seurat
  role: R environment used to analyse the 68k PBMC gene-cell-barcode matrix (log-transformed
    input) for the secondary analyses
  repo: github.com/10XGenomics/single-cell-3prime-paper
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:a47afba10c4c77e5585a2d5ad605cdcd5f508ca188a61a5ccfdfb47e78db1b27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - STAR
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Per-cell UMI/gene count statistical testing

## Summary

Statistically compares per-cell detection depth (number of detected genes and UMI counts per cell) between two scRNA-seq preparations, e.g. fresh versus cryopreserved PBMCs, using two-sided t-tests after read downsampling, to test whether sample handling compromises counting sensitivity.

## When to use

Use when two 10x scRNA-seq libraries must be compared for equal per-cell capture depth (genes/UMIs per cell), e.g. fresh vs cryopreserved PBMCs, after matching mean reads per cell by downsampling.

## When NOT to use

- The gene-barcode matrices were generated under unequal mean reads per cell — downsample first, otherwise the count comparison is confounded by depth
- The question is about expression levels rather than detection depth/sensitivity — per-cell UMI testing does not address differential expression
- Input is already an aggregated per-sample summary (e.g. median counts) — the t-test requires per-cell values

## Inputs

- Gene-barcode matrices (fresh and cryopreserved PBMCs) regenerated from downsampled reads by Cell Ranger
- BAM/FASTQ files of raw sequencing data (e.g. SRA SRP073767, support.10xgenomics.com/single-cell/datasets)
- Downsampled read files with matched mean reads per cell

## Outputs

- downsampling_comparison.csv with Pearson correlation of average gene expression (expected ~r=0.96) and genes-per-cell / UMI-per-cell t-test statistics with P-values
- Per-cell gene-count and UMI-count distributions for each preparation

## How to apply

First equalize sequencing depth: randomly downsample reads from each library with sambamba or samtools view -s so both have the same mean reads per cell, then re-run Cell Ranger counting (STAR alignment to hg19, barcode correction at posterior probability >= 0.975, 1-Hamming UMI correction, MAPQ=255 non-duplicate filter) to regenerate matched gene-barcode matrices. From each matrix extract two per-cell vectors: the number of detected genes and the total UMI count per cell. For each metric, run a two-sided t-test between the fresh and cryopreserved Donor A cells. Optionally compute the Pearson correlation of average gene expression across the downsampled matrices to contextualize the depth comparison. Write the correlation coefficient and genes-per-cell and UMI-per-cell statistics with P-values to downsampling_comparison.csv.

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Re-runs single-cell 3' gene counting on downsampled reads to regenerate matched gene-barcode matrices whose per-cell counts are then tested) — 10XGenomics/cellranger
- **STAR** (Aligns Read1 (cDNA insert) to hg19 during the Cell Ranger counting step)
- **Seurat** (R environment used to analyse the 68k PBMC gene-cell-barcode matrix (log-transformed input) for the secondary analyses) — github.com/10XGenomics/single-cell-3prime-paper

## Examples

```
from scipy.stats import ttest_ind; t_genes, p_genes = ttest_ind(fresh_genes_per_cell, frozen_genes_per_cell); t_umi, p_umi = ttest_ind(fresh_umi_per_cell, frozen_umi_per_cell)
```

## Evaluation signals

- Recomputed Pearson correlation of average gene expression between fresh and cryopreserved cells reproduces the reported r=0.96
- Two-sided t-test on genes per cell yields the reported non-significant result (P=0.8)
- Two-sided t-test on UMI counts per cell yields the reported non-significant result (P=0.1)
- Both downsampled libraries have the same mean reads per cell before counting
- Per-cell counts are computed only from confidently mapped (MAPQ=255), non-PCR-duplicate reads with valid barcodes and UMIs

## Limitations

- Earlier 10x libraries used a 5 nt UMI and 5 bp Read2 instead of the later configuration, introducing protocol heterogeneity that can affect count comparability
- A 15–20% error in cell counts was noted, which could contribute to variability in efficiency estimates
- Non-significant t-tests indicate no detectable depth difference at the tested depth; they do not prove absence of any freeze-thaw effect at other depths or on rare populations

## Evidence

- [discussion] Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix.: "Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix."
- [discussion] A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads.: "A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads."
- [discussion] The Cell Ranger Single-Cell Software Suite was used to perform sample demultiplexing, barcode processing and single-cell 3′ gene counting: "The Cell Ranger Single-Cell Software Suite was used to perform sample demultiplexing, barcode processing and single-cell 3′ gene counting"
- [readme] All the data described in the manuscript can be downloaded here, http://support.10xgenomics.com/single-cell/datasets: [evidence span withheld — anchor status: not_found]
