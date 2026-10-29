---
name: inter-gene-feature-segmentation
description: Use when building scReadSim's feature space from a scRNA-seq BAM file
  and a GTF annotation (e.g. gencode.vM10.annotation.gtf), and you need non-overlapping
  gene and inter-gene regions to construct gene-by-cell and inter-gene-by-cell count
  matrices.
when_to_use_negative:
- The goal is scATAC-seq feature generation — use Utility.scATAC_CreateFeatureSets
  instead
- Input is already a feature BED / count matrix from a prior segmentation run — skip
  directly to count-matrix construction
- You need isoform-level ground truths (isoform abundance, alternative splicing events),
  which the current scReadSim version does not generate
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: scReadSim
  role: Provides Utility.scRNA_CreateFeatureSets and Utility.ComplementFeature to
    segment the genome into gene and inter-gene feature sets from the BAM and GTF
    inputs
  repo: https://github.com/JSB-UCLA/scReadSim
- name: MACS3
  role: Used by scReadSim through Utility.CallPeak (relevant in the ATAC-seq branch
    of feature generation; not core to the scRNA-seq inter-gene segmentation)
  repo: https://github.com/macs3-project/MACS
provenance:
  source_task_ids:
  - task_008
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:1bbf8e3d6f767eb7d27a12e71627cd6e126670afba7e795d4acae6cbe5f45c2d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - MACS3
  - Picard
  - Seurat
  - bedtools
  - bowtie2
  - fgbio
  - samtools
  - scDesign3
  - scReadSim
  - seqtk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# Inter-gene feature segmentation

## Summary

A genome-segmentation skill from scReadSim that partitions the reference genome into gene-region foreground features and complementary inter-gene background features, so that synthetic scRNA-seq reads can be simulated for both annotated genes and the genomic background.

## When to use

Use when building scReadSim's feature space from a scRNA-seq BAM file and a GTF annotation (e.g. gencode.vM10.annotation.gtf), and you need non-overlapping gene and inter-gene regions to construct gene-by-cell and inter-gene-by-cell count matrices.

## When NOT to use

- The goal is scATAC-seq feature generation — use Utility.scATAC_CreateFeatureSets instead
- Input is already a feature BED / count matrix from a prior segmentation run — skip directly to count-matrix construction
- You need isoform-level ground truths (isoform abundance, alternative splicing events), which the current scReadSim version does not generate

## Inputs

- scRNA-seq BAM file containing reads in cells (e.g. 10x_RNA_chr1_3073253_4526737.bam, mm10 reference)
- Gene annotation GTF file (gencode.vM10.annotation.gtf)
- Chromosome size file (mm10.chrom.sizes)

## Outputs

- Foreground gene-region BED file (overlapping genes aggregated into non-overlapping supergenes)
- Complementary inter-gene background BED file
- Downstream gene-by-cell UMI count matrix (<filename>.countmatrix.txt / <filename>.UMI.countmatrix.txt) and inter-gene-by-cell count matrix with identical cells in identical column order

## How to apply

Run Utility.scRNA_CreateFeatureSets on the input BAM (e.g. 10x_RNA_chr1_3073253_4526737.bam) together with the gencode.vM10.annotation.gtf annotation and the mm10 chromosome size file to produce a foreground BED of gene regions and a background BED of the genomic complement, aggregating overlapping genes into non-overlapping supergenes. Utility.ComplementFeature can be used to generate the background (inter-gene) feature set from the input foreground features. After segmentation, the reads in the BAM are summarized per cell barcode into two UMI count matrices (gene-by-cell and inter-gene-by-cell) via Utility.scRNA_bam2countmat, keeping the same cells in the same column order in both matrices. This segmentation is the first of scReadSim's three major steps (genome segmentation, synthetic count generation, synthetic read generation) and feeds the downstream scDesign2-based synthetic count simulation.

## Related tools

- **scReadSim** (Provides Utility.scRNA_CreateFeatureSets and Utility.ComplementFeature to segment the genome into gene and inter-gene feature sets from the BAM and GTF inputs) — https://github.com/JSB-UCLA/scReadSim
- **MACS3** (Used by scReadSim through Utility.CallPeak (relevant in the ATAC-seq branch of feature generation; not core to the scRNA-seq inter-gene segmentation)) — https://github.com/macs3-project/MACS

## Examples

```
Utility.scRNA_CreateFeatureSets(BAM=10x_RNA_chr1_3073253_4526737.bam, gtf=gencode.vM10.annotation.gtf, chrom_sizes=mm10.chrom.sizes)
```

## Evaluation signals

- The gene-region foreground BED and the complementary inter-gene BED together tile the genome with no overlap (supergenes are non-overlapping)
- Utility.scRNA_bam2countmat on both BED files writes '<filename>.countmatrix.txt' and '<filename>.UMI.countmatrix.txt' to the output directory
- The gene-by-cell and inter-gene-by-cell matrices contain exactly the same cells in the same column order
- The synthetic matrix trained downstream (via GenerateSyntheticCount.scRNA_GenerateSyntheticCount) retains the same feature rows as the input matrix
- The resulting synthetic FASTQs (*.read1.bed2fa.fq, *.read2.bed2fa.fq) and synthetic_cell_barcode.txt exist after the full pipeline, confirming valid feature space

## Limitations

- The default implementation may generate intronic synthetic reads when extracting sequences from the reference genome, which can affect benchmarks such as cellranger default mode
- Segmentation provides gene-level ground truths only — no isoform-level ground truths (isoform abundance or alternative splicing events)
- Multi-sample or multi-condition simulation requires one BAM file per sample/condition as input

## Evidence

- [methods] Genome segmentation is the first of three pipeline steps: "As an overview, scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation."
- [methods] The function to generate the feature sets: "Use function `scRNA_CreateFeatureSets` to generate features."
- [methods] Background features are generated as a complement of the input features: "To generate background features for input features, use function `Utility.ComplementFeature` with following arguments"
- [methods] Segmentation feeds count-matrix construction for foreground and background features: "scReasSim constructs the count matrices for both foreground feautures and background features through function `Utility.scRNA_bam2countmat`"
- [readme] README positioning of scReadSim as a trained read simulator: [evidence span withheld — anchor status: external_doc]
