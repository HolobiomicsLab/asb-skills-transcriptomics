---
name: cell-barcode-umi-quantification
description: Use when you have 10x CellRanger output (filtered_feature_bc_matrix,
  possorted_genome_bam.bam) and need per-cell pathogen UMI counts for an infected
  scRNA-seq sample, e.g. HIV EGFP+ cell detection.
when_to_use_negative:
- Input is a single-end or non-10x scRNA-seq library without CellRanger barcodes/UMIs
  — scPathoQuant is built specifically for 10x output folders.
- The host alignment step (cellranger count) has not been run yet — the tool consumes
  cellranger's 10x output and unaligned reads, not raw FASTQ directly.
- You need de novo proviral genome assembly rather than per-cell UMI quantification
  against a fixed reference.
edam_operation: http://edamontology.org/operation_0372
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPathoQuant
  role: Python package that extracts barcodes/UMIs from host-unmapped reads, remaps
    them to the pathogen genome, and integrates per-cell UMI counts into 10x matrices
  repo: https://github.com/galelab/scPathoQuant
- name: cellranger
  role: Upstream host-genome alignment that produces the 10x output folder and possorted_genome_bam.bam
    consumed by scPathoQuant
- name: bbmap
  role: Default aligner used to remap extracted host-unaligned reads to the pathogen
    genome (e.g. with --semiperfectmode)
- name: bowtie2
  role: Alternative aligner (-align bowtie2) used with --very-sensitive --non-deterministic
    parameters
- name: htseq
  role: htseq-count with --intersection-nonempty to quantify reads to the whole pathogen
    and individual pathogen genes
- name: seurat
  role: Downstream consumer of the pathogen-integrated filtered_feature_bc_matrix
    via Read10X
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:4a710077970758f47500805c01332e9d0241f466585877a13709704fdc477421
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - bbmap
  - bowtie2
  - cellranger
  - htseq
  - scPathoQuant
  - seurat
  edam_operation: http://edamontology.org/operation_0372
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cell-barcode-umi-quantification

## Summary

Per-cell pathogen UMI quantification from 10x scRNA-seq data: barcodes and UMIs are recovered from host-unaligned reads (possorted_genome_bam.bam), remapped to a pathogen genome, and total and per-gene pathogen UMI counts are integrated back into the 10x matrix files for downstream pipelines like Seurat.

## When to use

Use when you have 10x CellRanger output (filtered_feature_bc_matrix, possorted_genome_bam.bam) and need per-cell pathogen UMI counts for an infected scRNA-seq sample, e.g. HIV EGFP+ cell detection.

## When NOT to use

- Input is a single-end or non-10x scRNA-seq library without CellRanger barcodes/UMIs — scPathoQuant is built specifically for 10x output folders.
- The host alignment step (cellranger count) has not been run yet — the tool consumes cellranger's 10x output and unaligned reads, not raw FASTQ directly.
- You need de novo proviral genome assembly rather than per-cell UMI quantification against a fixed reference.

## Inputs

- 10x CellRanger count output folder (filtered_feature_bc_matrix, raw_feature_bc_matrix, possorted_genome_bam.bam)
- Pathogen reference genome FASTA (header used to quantify total pathogen reads; keep it simple, e.g. > HIV_virus)
- Optional pathogen GTF annotation file (required for per-gene pathogen counts)
- FastQ reads / raw data for the upstream cellranger count step

## Outputs

- pathogen_al_counts.csv (total pathogen reads mapping per cell)
- pathogen_al_gene_counts.csv (reads mapping to pathogen genes per cell)
- barcodes_umi_read_table.csv (unmapped reads with corresponding barcodes and UMIs)
- pathogen_al_mapped_sort.bam / .bai (sorted pathogen-mapped reads, IGV-ready)
- pathogen_al_mapped.fq.gz (mapped pathogen reads for downstream phylogenetic analysis)
- Overwritten 10x filtered_feature_bc_matrix and raw_feature_bc_matrix with pathogen counts integrated
- pathogen_copy.png and pathogen_genes.png violin plots of per-cell counts

## How to apply

First run cellranger count against the host genome (e.g. GRCh38) and pass the resulting 10x output folder to scPathoQuant via the -10x parameter. scPathoQuant extracts the unmapped reads from possorted_genome_bam.bam into the _tmp/ folder (unmmaped.bam, unmmaped.sam), builds barcodes_umi_read_table.csv pairing each unmapped read with its cell barcode and UMI, and writes unmapped.fq.gz. These reads are then remapped to the pathogen genome (default bbmap, or bowtie2 with --very-sensitive --non-deterministic) and quantified with htseq (using the --intersection-nonempty mode); a pathogen GTF supplied alongside the FASTA in the -p2genome folder additionally yields per-gene UMI counts. Finally, pathogen UMI counts and pathogen UMI gene counts are integrated into features.tsv.gz and matrix.mtx.gz in filtered_feature_bc_matrix and raw_feature_bc_matrix so Seurat's Read10X can ingest them directly.

## Related tools

- **scPathoQuant** (Python package that extracts barcodes/UMIs from host-unmapped reads, remaps them to the pathogen genome, and integrates per-cell UMI counts into 10x matrices) — https://github.com/galelab/scPathoQuant
- **cellranger** (Upstream host-genome alignment that produces the 10x output folder and possorted_genome_bam.bam consumed by scPathoQuant)
- **bbmap** (Default aligner used to remap extracted host-unaligned reads to the pathogen genome (e.g. with --semiperfectmode))
- **bowtie2** (Alternative aligner (-align bowtie2) used with --very-sensitive --non-deterministic parameters)
- **htseq** (htseq-count with --intersection-nonempty to quantify reads to the whole pathogen and individual pathogen genes)
- **seurat** (Downstream consumer of the pathogen-integrated filtered_feature_bc_matrix via Read10X)

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/pathogen/fastafilefolder --tmp_removal
```

## Evaluation signals

- pathogen_al_counts.csv contains a nonzero per-cell pathogen read/UMI count for expected infected cells and zeros for uninfected cells.
- barcodes_umi_read_table.csv in _tmp/ successfully pairs every unmapped read with a cell barcode and UMI (no reads dropped in the sam-to-table step).
- The overwritten filtered_feature_bc_matrix loads into Seurat with Read10X and includes the pathogen UMI gene count feature rows.
- pathogen_genes_al_sort_counts.sam is produced only when a pathogen GTF was supplied alongside the FASTA — its absence confirms whether per-gene counts were requested.
- Cross-check against CellRanger with the pathogen genome computationally integrated (as in the Bradley 2018 HIV donor test set) — per-cell calls should agree, e.g. 100% overlap of EGFP+HIV+ cells across alignment methods.

## Limitations

- Temporary files in _tmp/ can be very large; pass --tmp_removal or delete them manually if space is limited.
- The pathogen FASTA header is used to quantify total pathogen reads, so complicated header names should be simplified (e.g. > HIV_virus) or counts will be misleading.
- A pathogen GTF is not mandatory — without it, only total pathogen counts are produced and no per-gene counts (pathogen_genes_al_sort_counts.sam).
- Only host-unaligned reads are available for pathogen mapping, so reads cross-mapping to the host reference are not counted.

## Evidence

- [abstract] Extract barcodes/UMIs from host-unmapped reads and remap to pathogen: "scPathoQuant extracts sequences that were not aligned to the primary host genome"
- [readme] Integrate pathogen UMI counts into 10x files generated by cellranger count: "This software integrates pathogen UMI counts and pathogen UMI gene counts into 10x files generated by first running [```cellranger count```]"
- [readme] HTS quantification is delegated to samtools, bowtie2, and htseq: "The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads"
- [readme] The intermediate barcode/UMI table is written to the _tmp/ folder: "barcodes_umi_read_table.csv - table of unmapped reads and corresponding barcodes and UMIs"
- [readme] Per-cell pathogen counts are the primary CSV output: "pathogen_al_counts.csv - total number of reads mapping to the pathogen in each cell"
- [readme] htseq uses non-default --intersection-nonempty parameter: "Default parameters are used when aligning reads to the pathogen genome and and non default htseq-count include using the --intersection-nonempty parameter"
