---
name: scdesign2-synthetic-count-scaling
description: Use when you need synthetic single-cell RNA-seq data whose throughput
  (cell number) or library size (sequencing depth) is systematically scaled around
  a real reference dataset to study pseudo-bulk coverage or guide experimental design.
when_to_use_negative:
- Input is single-cell chromatin accessibility data — use the scATAC-seq module (Utility.scATAC_CreateFeatureSets
  / GenerateSyntheticCount.scATAC_GenerateSyntheticCount) instead
- Cells follow continuous trajectories or have spatial locations — scReadSim currently
  implements scDesign2, not scDesign3, internally
- Goal is isoform-level simulation (isoform abundance or alternative splicing) — current
  scReadSim provides gene-level UMI count ground truths only
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: scReadSim
  role: Simulator framework that pre-processes the BAM, invokes scDesign2 for synthetic
    counts, and converts count matrices to synthetic read coordinates, BED, FASTQ,
    and BAM
  repo: https://github.com/JSB-UCLA/scReadSim
- name: scDesign2
  role: Internal count simulator trained on real UMI count matrices to generate synthetic
    count matrices under user-specified cell numbers and sequencing depths
  repo: https://github.com/JSB-UCLA/scDesign2
- name: bedtools
  role: getfasta extraction of reference sequences from the combined BED coordinates
    during BED-to-FASTQ conversion
- name: seqtk
  role: FASTA-to-FASTQ conversion in scRNA_BED2FASTQ
- name: bowtie2
  role: Optional alignment of synthetic paired-end FASTQ reads to the mm10 reference
    genome via AlignSyntheticBam_Pair
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:c8eeedaf32e524c26b4049555c5c1cc9e539064cdc88d44773053ab642463ac6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - bedtools
  - bowtie2
  - samtools
  - scDesign2
  - scReadSim
  - seqtk
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# scdesign2-synthetic-count-scaling

## Summary

Generates synthetic scRNA-seq count matrices at varying cell numbers (e.g., 1219–19508 cells) and sequencing depths (e.g., 1.2M–20.1M reads) by training scDesign2 through scReadSim's GenerateSyntheticCount.scRNA_GenerateSyntheticCount, enabling coverage trade-off analyses for experimental design.

## When to use

Use when you need synthetic single-cell RNA-seq data whose throughput (cell number) or library size (sequencing depth) is systematically scaled around a real reference dataset to study pseudo-bulk coverage or guide experimental design.

## When NOT to use

- Input is single-cell chromatin accessibility data — use the scATAC-seq module (Utility.scATAC_CreateFeatureSets / GenerateSyntheticCount.scATAC_GenerateSyntheticCount) instead
- Cells follow continuous trajectories or have spatial locations — scReadSim currently implements scDesign2, not scDesign3, internally
- Goal is isoform-level simulation (isoform abundance or alternative splicing) — current scReadSim provides gene-level UMI count ground truths only

## Inputs

- Pre-processed RNA-modality BAM file from a real single-cell dataset (e.g., mouse 10x single-cell Multiome, mm10 reference)
- Gene-by-cell and inter-gene-by-cell UMI count matrices built by scRNA_CreateFeatureSets and scRNA_bam2countmat
- Reference genome FASTA (mm10) and chromosome size/GTF annotation (gencode.vM10.annotation.gtf)
- Target cell numbers and sequencing depths (e.g., 0.25x–4x scaling factors)

## Outputs

- Synthetic gene- and inter-gene-by-cell UMI count matrices per condition (from scDesign2)
- Foreground (gene) and background (inter-gene) BED files of synthetic read coordinates (90-nt reads, 5-bp jitter)
- Combined BED files converted to paired-end FASTQ files per condition
- Optionally, synthetic BAM files via bowtie2 alignment to mm10 (AlignSyntheticBam_Pair)
- Pseudo-bulk per-window (1000-bp) coverage summary table, e.g., pseudo_bulk_coverage_summary.csv

## How to apply

First pre-process the real data: run scReadSim's scRNA_CreateFeatureSets and scRNA_bam2countmat on the RNA-modality BAM file (e.g., the mouse 10x Multiome dataset with mm10 and gencode.vM10.annotation.gtf) to obtain gene-by-cell and inter-gene-by-cell UMI count matrices. Then train scDesign2 via GenerateSyntheticCount.scRNA_GenerateSyntheticCount, setting the number of synthetic cells (e.g., 1219, 2438, 4877, 9754, 19508) while fixing sequencing depth at the original 5.0M reads, and separately varying depth (1.2M, 2.5M, 5.0M, 10.1M, 20.1M) while holding cells at 4877. Convert each synthetic count matrix into reads with scRNA_GenerateBAM.scRNA_GenerateBAMCoord (default 90-nt read length, 5-bp jitter size), combine foreground/background BEDs with scRNA_CombineBED, and produce paired-end FASTQ via scRNA_BED2FASTQ (bedtools getfasta + seqtk) with optional bowtie2 alignment. The rationale is that scReadSim is flexible for 'varying throughputs and library sizes as input parameters to guide experimental design', so scaling these parameters reveals the expected trade-off between cell number and per-cell depth.

## Related tools

- **scReadSim** (Simulator framework that pre-processes the BAM, invokes scDesign2 for synthetic counts, and converts count matrices to synthetic read coordinates, BED, FASTQ, and BAM) — https://github.com/JSB-UCLA/scReadSim
- **scDesign2** (Internal count simulator trained on real UMI count matrices to generate synthetic count matrices under user-specified cell numbers and sequencing depths) — https://github.com/JSB-UCLA/scDesign2
- **bedtools** (getfasta extraction of reference sequences from the combined BED coordinates during BED-to-FASTQ conversion)
- **seqtk** (FASTA-to-FASTQ conversion in scRNA_BED2FASTQ)
- **bowtie2** (Optional alignment of synthetic paired-end FASTQ reads to the mm10 reference genome via AlignSyntheticBam_Pair)

## Examples

```
pip install git+https://github.com/JSB-UCLA/scReadSim.git; then in R/Python: GenerateSyntheticCount.scRNA_GenerateSyntheticCount(count_mat, n_cell=19508, depth=5.0e6)
```

## Evaluation signals

- All ten synthetic conditions (5 cell-number scalings x fixed 5.0M depth, 5 depth scalings x fixed 4877 cells) yield paired-end FASTQ and synthetic count matrix outputs
- At fixed sequencing depth, summarized pseudo-bulk coverage of pooled synthetic cells decreases monotonically as synthetic cell number increases (1219 → 19508)
- At fixed cell number, pseudo-bulk coverage increases monotonically as sequencing depth increases (1.2M → 20.1M reads)
- Synthetic read coordinates use the default 90-nt read length and 5-bp jitter size, and align to mm10 with bowtie2 without unexpected failure
- Per-window coverage on chromosome 1 (1000-bp bins) for synthetic conditions resembles the real-data coverage profile at the matched condition

## Limitations

- At fixed sequencing depth, increasing synthetic cell number necessarily decreases pseudo-bulk per-cell coverage — an expected trade-off, not a simulator failure
- The default implementation may generate intronic synthetic reads when extracting sequences from the reference genome, which can affect benchmarks like cellranger default mode
- Multi-sample or multi-condition generation requires supplying one BAM file per sample or condition
- scDesign2-based scaling only supports clustered/discrete cell populations; continuous trajectories or spatial layouts require scDesign3, which is not yet the internal simulator

## Evidence

- [methods] scReadSim trains the count simulator scDesign2: "In the second step—synthetic count generation, scReadSim trains the count simulator scDesign2"
- [methods] workflow functions chain: "scReasSim constructs the count matrices for both foreground feautures and background features through function `Utility.scRNA_bam2countmat`"
- [methods] Use function `GenerateSyntheticCount.scRNA_GenerateSyntheticCount` to generate synthetic count matrix: "Use function `GenerateSyntheticCount.scRNA_GenerateSyntheticCount` to generate synthetic count matrix"
- [readme] flexible throughput parameters: [evidence span withheld — anchor status: external_doc]
- [readme] scDesign2 repo and installation: [evidence span withheld — anchor status: external_doc]
