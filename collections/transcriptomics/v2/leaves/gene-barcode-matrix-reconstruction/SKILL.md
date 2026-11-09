---
name: gene-barcode-matrix-reconstruction
description: Use when raw scRNA-seq FASTQ/BAM reads (e.g. from SRA SRP073767 or 10x
  datasets site) must be converted into gene-cell-barcode matrices, or when matrices
  from different preparations must be regenerated under identical counting settings
  before comparison.
when_to_use_negative:
- Input is already a gene-barcode matrix from the 10x datasets site and no read-level
  reprocessing or downsampling is required
- Data are not 10x GemCode/Chromium 3' droplet reads with the expected Read1 cDNA
  insert / barcode / UMI structure (e.g. full-length or plate-based scRNA-seq)
- The goal is variant-level analysis requiring Freebayes SNV calling on the genome
  BAM rather than gene counting
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing and single-cell 3' gene
    counting to produce the gene-barcode matrix
  repo: https://www.10xgenomics.com/support/software/cell-ranger/downloads
- name: STAR
  role: Aligns Read1 (cDNA insert) to the reference genome (hg19/mm10) inside the
    counting pipeline
- name: Seurat
  role: Consumes the log-transformed gene-cell-barcode matrix for downstream secondary
    analysis (e.g. PBMC clustering)
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
- name: samtools/sambamba
  role: Randomly downsample reads from each library so both datasets share the same
    mean reads per cell before re-counting
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:ef30ea4a74f11348aaf3d3c6612c56b24aad69d34f53d3963cc540b7dc26b819
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell Ranger Single-Cell Software Suite
  - STAR
  - Seurat
  - samtools/sambamba
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gene-barcode-matrix-reconstruction

## Summary

Regenerates 10x Genomics single-cell 3' gene-barcode count matrices from raw sequencing reads by demultiplexing, barcode processing and gene counting with the Cell Ranger pipeline (STAR alignment, barcode posterior-probability correction, 1-Hamming UMI correction, MAPQ=255 non-duplicate read filtering), enabling matched-matrix comparisons across libraries.

## When to use

Use when raw scRNA-seq FASTQ/BAM reads (e.g. from SRA SRP073767 or 10x datasets site) must be converted into gene-cell-barcode matrices, or when matrices from different preparations must be regenerated under identical counting settings before comparison.

## When NOT to use

- Input is already a gene-barcode matrix from the 10x datasets site and no read-level reprocessing or downsampling is required
- Data are not 10x GemCode/Chromium 3' droplet reads with the expected Read1 cDNA insert / barcode / UMI structure (e.g. full-length or plate-based scRNA-seq)
- The goal is variant-level analysis requiring Freebayes SNV calling on the genome BAM rather than gene counting

## Inputs

- Raw scRNA-seq reads (FASTQ/BAM) from droplet-based 3' libraries, e.g. SRA SRP073767 or support.10xgenomics.com/single-cell/datasets downloads
- Reference genome FASTQ/index, e.g. hg19 (human), mm10 (mouse), or ERCC reference (tools.thermofisher.com/content/sfs/manuals/cms_095047.txt)
- Expected recovered cell count (for cell-barcode calling, n = 1% of expected recovered cells)

## Outputs

- Gene-barcode count matrix (genes × cells) per sample
- BAM with aligned reads annotated with cell barcodes and UMIs
- Per-cell QC metrics (detected genes per cell, UMI counts per cell) usable for downstream t-tests and correlation analyses

## How to apply

Align Read1, which contains the cDNA insert, to an appropriate reference genome (hg19 for human cells, mm10 for mouse) with STAR via Cell Ranger. Correct cell barcodes only when the posterior probability for a candidate barcode is at least 0.975, correcting to the highest-posterior candidate. Correct UMIs that are 1-Hamming-distance away from another UMI with more reads for the same cell barcode and gene to the UMI with more reads, considering UMIs valid if quality score >10 and not homopolymers. Build the gene-barcode matrix from only confidently mapped (MAPQ=255), non-PCR-duplicate reads with valid barcodes and UMIs; consider top barcodes cell barcodes when they are within the same order of magnitude (>10% of the top nth barcode, n = 1% of the expected recovered cell count). Re-running this counting pipeline on downsampled reads (e.g. equalized mean reads per cell with samtools/sambamba) yields matched matrices for downstream statistics such as Pearson correlation of average expression and two-sided t-tests on genes/UMIs per cell.

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing and single-cell 3' gene counting to produce the gene-barcode matrix) — https://www.10xgenomics.com/support/software/cell-ranger/downloads
- **STAR** (Aligns Read1 (cDNA insert) to the reference genome (hg19/mm10) inside the counting pipeline)
- **Seurat** (Consumes the log-transformed gene-cell-barcode matrix for downstream secondary analysis (e.g. PBMC clustering)) — https://github.com/10XGenomics/single-cell-3prime-paper
- **samtools/sambamba** (Randomly downsample reads from each library so both datasets share the same mean reads per cell before re-counting)

## Examples

```
samtools view -s 0.25 reads.bam > downsampled.bam; cellranger count --transcriptome=hg19 --fastqs=downsampled --sample=pbmc
```

## Evaluation signals

- Downsampled fresh vs cryopreserved Donor A matrices reproduce Pearson r=0.96 for average gene expression
- Two-sided t-tests on genes per cell (P=0.8) and UMI counts per cell (P=0.1) are non-significant, indicating equal sensitivity after downsampling
- All reads contributing to the gene-barcode matrix are confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs
- Every corrected barcode trace had posterior probability >=0.975 and corrected UMIs are 1-Hamming distance from a higher-read UMI for the same cell barcode and gene
- Cell barcodes called match the order-of-magnitude rule (>10% of the top nth barcode, n = 1% of expected recovered cell count)

## Limitations

- Some earlier libraries used 5 nt UMI and 5 bp Read2, introducing protocol heterogeneity across libraries
- Cell Ranger source code is informational only; 10x does not support interpreting, modifying, building, or running it — use the officially supported release binaries
- Cell capture efficiency is ~50% with 15–20% error in cell counts, so recovered cell counts carry uncertainty when interpreting per-cell metrics
- SNV calling on the genome BAM requires Freebayes 1.0.2 and is a separate analysis from gene counting

## Evidence

- [discussion] Cell Ranger demultiplexing and counting: "The Cell Ranger Single-Cell Software Suite was used to perform sample demultiplexing, barcode processing and single-cell 3′ gene counting"
- [discussion] STAR alignment of Read1: "Read1, which contains the cDNA insert, was aligned to an appropriate reference genome using STAR35."
- [discussion] Barcode correction threshold: "If the posterior probability for any candidate barcode is at least 0.975, then the barcode is corrected to the candidate barcode with the highest posterior probability."
- [discussion] UMI correction rule: "A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads."
- [discussion] Read filters for matrix generation: "Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix."
- [readme] Data availability: [evidence span withheld — anchor status: not_found]
