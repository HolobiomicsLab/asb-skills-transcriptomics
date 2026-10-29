---
name: umi-count-matrix-construction-from-bam
description: Use when you have an scRNA-seq BAM file aligned to mm10, a gene annotation
  GTF (e.g., gencode.vM10.annotation.gtf), and a chromosome size file, and you need
  gene- and inter-gene-by-cell UMI count matrices as ground truth for simulation or
  benchmarking.
when_to_use_negative:
- Your goal is scATAC-seq feature/count construction — use Utility.scATAC_CreateFeatureSets
  and Utility.scATAC_bam2countmat instead, which work on peak/foreground regions rather
  than gene annotations.
- You only need synthetic reads without ground-truth counts — but note count matrices
  are the required intermediate for scDesign2 training, so they must be built anyway
  when simulating.
- The data are isoform-level or alternative-splicing studies — scReadSim provides
  gene-level UMI ground truths only.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scReadSim
  role: Provides Utility.scRNA_CreateFeatureSets to segment the genome into gene/inter-gene
    BED features and Utility.scRNA_bam2countmat to build the gene-by-cell and inter-gene-by-cell
    UMI count matrices from the BAM file.
  repo: https://github.com/JSB-UCLA/scReadSim
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:e7599799f7f1f76278f5d2765c87b53356d9cf15e658f368a322fb88400d944a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - bedtools
  - bowtie2
  - samtools
  - scReadSim
  - seqtk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# umi-count-matrix-construction-from-bam

## Summary

Segment an scRNA-seq reference genome into genes (foreground) and inter-genes (background) from a BAM file plus GTF annotation, then summarize reads into gene-by-cell and inter-gene-by-cell UMI count matrices with scReadSim. These matrices are the ground-truth inputs for scDesign2-based synthetic count generation.

## When to use

Use when you have an scRNA-seq BAM file aligned to mm10, a gene annotation GTF (e.g., gencode.vM10.annotation.gtf), and a chromosome size file, and you need gene- and inter-gene-by-cell UMI count matrices as ground truth for simulation or benchmarking.

## When NOT to use

- Your goal is scATAC-seq feature/count construction — use Utility.scATAC_CreateFeatureSets and Utility.scATAC_bam2countmat instead, which work on peak/foreground regions rather than gene annotations.
- You only need synthetic reads without ground-truth counts — but note count matrices are the required intermediate for scDesign2 training, so they must be built anyway when simulating.
- The data are isoform-level or alternative-splicing studies — scReadSim provides gene-level UMI ground truths only.

## Inputs

- scRNA-seq BAM file (reads in cells, mm10 reference)
- Gene annotation GTF file (gencode.vM10.annotation.gtf)
- Reference chromosome size file (mm10.chrom.sizes)
- Output directory path

## Outputs

- Foreground gene-region BED file
- Complementary inter-gene background BED file
- Gene-by-cell raw and UMI count matrices ('<filename>.countmatrix.txt' and '<filename>.UMI.countmatrix.txt')
- Inter-gene-by-cell raw and UMI count matrices in matching column order

## How to apply

First run Utility.scRNA_CreateFeatureSets on the input BAM with gencode.vM10.annotation.gtf and the mm10 chromosome size file to produce a gene-region foreground BED and the complementary inter-gene background BED, aggregating overlapping genes into non-overlapping supergenes. Then run Utility.scRNA_bam2countmat on the foreground gene BED with UMI_modeling=True to count, per cell barcode, the UMIs whose reads overlap each gene, yielding the gene-by-cell UMI count matrix. Run scRNA_bam2countmat again on the background inter-gene BED, keeping the same cells in the same column order as the gene matrix. Both raw and UMI count matrices are written to the output directory as '<filename>.countmatrix.txt' and '<filename>.UMI.countmatrix.txt'. The gene-by-cell UMI count matrix then serves as ground truth for training scDesign2 via GenerateSyntheticCount.scRNA_GenerateSyntheticCount.

## Related tools

- **scReadSim** (Provides Utility.scRNA_CreateFeatureSets to segment the genome into gene/inter-gene BED features and Utility.scRNA_bam2countmat to build the gene-by-cell and inter-gene-by-cell UMI count matrices from the BAM file.) — https://github.com/JSB-UCLA/scReadSim

## Examples

```
from scReadSim.Utility import scRNA_CreateFeatureSets, scRNA_bam2countmat; scRNA_CreateFeatureSets(input_bam, out_dir, refgenome="mm10", gtf="gencode.vM10.annotation.gtf", chrom_size="mm10.chrom.sizes"); scRNA_bam2countmat(fg_bed, out_dir, UMI_modeling=True)
```

## Evaluation signals

- Both output files '<filename>.countmatrix.txt' and '<filename>.UMI.countmatrix.txt' exist in the output directory for each BED input.
- The gene-by-cell and inter-gene-by-cell matrices contain the same cells in the same column order.
- Overlapping genes are aggregated into non-overlapping supergenes, so foreground features do not overlap.
- With UMI_modeling=True, per-cell entries equal deduplicated UMI counts of reads overlapping each feature, not raw read counts.
- Downstream, GenerateSyntheticCount.scRNA_GenerateSyntheticCount successfully trains scDesign2 on the gene-by-cell UMI matrix (e.g., producing synthetic cells such as 1219/2438/4877/9754/19508 in Supplementary Fig. 11-style experiments).

## Limitations

- scReadSim currently generates scRNA-seq reads with gene-level ground truths (UMI counts) only, not isoform-level ground truths including isoform abundance and alternative splicing events.
- Default implementation may generate intronic synthetic reads when extracting sequences from the reference genome, which can affect downstream benchmarks such as cellranger default mode.
- Multi-sample/condition generation requires users to supply one BAM file per sample or condition.

## Evidence

- [methods] Workflow overview: "As an overview, scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation."
- [methods] Required inputs: "To pre-process real scRNA-seq data for training, scReadSim requires a BAM file (containing scRNA-seq reads in cells)"
- [methods] Feature generation function: "Use function `scRNA_CreateFeatureSets` to generate features."
- [readme] Read-level simulation context: [evidence span withheld — anchor status: external_doc]
- [discussion] Isoform ground-truth limitation: "generates scRNA-seq reads with gene-level ground truths (UMI counts) only, not isoform-level ground truths including isoform abundance and alternative splicing events"
