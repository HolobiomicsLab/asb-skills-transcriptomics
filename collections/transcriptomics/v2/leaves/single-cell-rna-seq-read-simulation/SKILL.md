---
name: single-cell-rna-seq-read-simulation
description: Use when benchmarking read-level scRNA-seq tools (UMI deduplication,
  aligners) requires synthetic reads (BAM/FASTQ) with ground-truth UMI count matrices,
  learned from a real BAM + GTF rather than untrained simulator defaults.
when_to_use_negative:
- Benchmarking tools that study isoforms or alternative splicing events — scReadSim
  provides gene-level ground truths (UMI counts) only, not isoform abundance or alternative
  splicing events
- Simulating long-read scRNA-seq data with ground-truth isoform abundance — unavailable
  in existing simulators including scReadSim
- The analysis target is a count matrix already (e.g. comparing count simulators directly)
  rather than synthetic sequencing reads
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scReadSim
  role: 'Primary three-step simulator: genome segmentation, synthetic count generation,
    synthetic read generation; outputs synthetic BAM/FASTQ reads'
  repo: https://github.com/JSB-UCLA/scReadSim
- name: scDesign2
  role: Count simulator trained on real gene/inter-gene count matrices to generate
    synthetic count matrices when cells belong to distinct clusters
  repo: https://github.com/JSB-UCLA/scDesign2
- name: scDesign3
  role: Alternative count simulator used when cells follow continuous trajectories
    instead of distinct clusters
- name: bedtools
  role: Extracts genomic sequences from combined BED coordinates in scRNA_BED2FASTQ
    (getfasta)
- name: seqtk
  role: Converts BED-derived FASTA to paired-end FASTQ in scRNA_BED2FASTQ
- name: bowtie2
  role: Aligns synthetic paired-end FASTQs to the indexed reference in AlignSyntheticBam_Pair
- name: fgbio
  role: Introduces Illumina-style substitution base errors in scRNA_ErrorBase
- name: Picard
  role: Creates the reference dictionary required by the fgbio base-error step
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:1262a5582c779edfc7b3593940e0c48b97c8d439e34248f231da5df62042f9bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - Picard
  - SCAN-ATAC-Sim
  - bedtools
  - bowtie2
  - fgbio
  - samtools
  - scDesign2
  - scDesign3
  - scReadSim
  - seqtk
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# single-cell-rna-seq-read-simulation

## Summary

Simulate realistic single-cell RNA-seq sequencing reads with scReadSim via its three-step architecture (genome segmentation, synthetic count generation, synthetic read generation), training scDesign2 on real gene and inter-gene count matrices to output synthetic BAM/FASTQ reads with ground-truth UMI counts.

## When to use

Use when benchmarking read-level scRNA-seq tools (UMI deduplication, aligners) requires synthetic reads (BAM/FASTQ) with ground-truth UMI count matrices, learned from a real BAM + GTF rather than untrained simulator defaults.

## When NOT to use

- Benchmarking tools that study isoforms or alternative splicing events — scReadSim provides gene-level ground truths (UMI counts) only, not isoform abundance or alternative splicing events
- Simulating long-read scRNA-seq data with ground-truth isoform abundance — unavailable in existing simulators including scReadSim
- The analysis target is a count matrix already (e.g. comparing count simulators directly) rather than synthetic sequencing reads

## Inputs

- BAM file of scRNA-seq reads in cells (e.g. 10X_RNA_chr1_3073253_4526737.bam, mm10 reference genome)
- Gene annotation GTF (gencode.vM10.annotation.gtf)
- Reference genome FASTA (mm10 chr1) and bowtie2-indexed reference
- Optional cell-type labels (otherwise Louvain clustering is applied)

## Outputs

- Paired-end FASTQ files (*.read1.bed2fa.fq and *.read2.bed2fa.fq) and/or synthetic BAM
- Synthetic gene and inter-gene count matrices (*.scDesign2Simulated.txt)
- Synthetic cell barcodes (synthetic_cell_barcode.txt)
- Foreground/background BED files and ground-truth UMI count matrix for benchmarking

## How to apply

First segment the genome: build the feature space with Utility.scRNA_CreateFeatureSets from the input BAM (e.g. 10X_RNA_chr1_3073253_4526737.bam) and gencode.vM10.annotation.gtf, using gene regions as foreground features and their genomic complement as inter-gene background features (Utility.ComplementFeature). Second, construct read and UMI count matrices for foreground/background features with Utility.scRNA_bam2countmat (UMI_modeling=True) and train scDesign2 via GenerateSyntheticCount.scRNA_GenerateSyntheticCount on the gene and inter-gene count matrices; use scDesign2 when cells belong to distinct clusters (Louvain clustering is applied if no cell labels are given) and fall back to scDesign3 when cells follow continuous trajectories. Third, generate synthetic read coordinates with scRNA_GenerateBAM.scRNA_GenerateBAMCoord (read_len=90, jitter_size=5, CB_len=16, UB_len=10) sampling real reads from the BAM, combine foreground/background BED files with scRNA_CombineBED, and convert to paired-end FASTQ with scRNA_BED2FASTQ (bedtools getfasta + seqtk, sort_FASTQ=True) against the mm10 chr1 reference. Optionally align with AlignSyntheticBam_Pair (bowtie2) and add Illumina-style substitution errors with scRNA_ErrorBase (fgbio + Picard reference dictionary). Defaults keep the same cell number and sequencing depth as the real data.

## Related tools

- **scReadSim** (Primary three-step simulator: genome segmentation, synthetic count generation, synthetic read generation; outputs synthetic BAM/FASTQ reads) — https://github.com/JSB-UCLA/scReadSim
- **scDesign2** (Count simulator trained on real gene/inter-gene count matrices to generate synthetic count matrices when cells belong to distinct clusters) — https://github.com/JSB-UCLA/scDesign2
- **scDesign3** (Alternative count simulator used when cells follow continuous trajectories instead of distinct clusters)
- **bedtools** (Extracts genomic sequences from combined BED coordinates in scRNA_BED2FASTQ (getfasta))
- **seqtk** (Converts BED-derived FASTA to paired-end FASTQ in scRNA_BED2FASTQ)
- **bowtie2** (Aligns synthetic paired-end FASTQs to the indexed reference in AlignSyntheticBam_Pair)
- **fgbio** (Introduces Illumina-style substitution base errors in scRNA_ErrorBase)
- **Picard** (Creates the reference dictionary required by the fgbio base-error step)

## Examples

```
pip install scReadSim; python -c "from scReadSim import Utility, GenerateSyntheticCount, scRNA_GenerateBAM; Utility.scRNA_CreateFeatureSets(BAM='10X_RNA_chr1_3073253_4526737.bam', gtf='gencode.vM10.annotation.gtf')"
```

## Evaluation signals

- Validation files exist in the output directory: *.read1.bed2fa.fq and *.read2.bed2fa.fq paired-end FASTQs, the *.scDesign2Simulated.txt synthetic count matrix, and synthetic_cell_barcode.txt
- Synthetic count matrix matches the real data's cell number and sequencing depth (the scDesign2 default)
- Read 1 sequences are concatenated UMI + cell barcode strings with the configured lengths (UB_len=10, CB_len=16)
- Synthetic reads mimic the real data at the read-count level: summary statistics (peak/feature-wise mean, variance, coefficient of variance, zero proportion; cell-wise library size, zero proportion) show small Kolmogorov-Smirnov statistics versus the real matrix
- Synthetic cells mix with real cells in UMAP embeddings (miLISI close to the perfect-mixing value of 2) when projected via the real-matrix PCA/UMAP spaces

## Limitations

- The default implementation may generate intronic synthetic reads when extracting sequences from the reference genome, which can affect benchmarks such as cellranger default mode
- Multi-sample or multi-condition generation requires users to supply one BAM file per sample or condition
- Only gene-level ground truths (UMI counts) are provided; no isoform abundance or alternative splicing events, so it should not be used to benchmark isoform/splicing tools
- Support for continuous trajectories, spatial locations, conditions/batches, and multi-omics depends on scDesign3, which is not yet the internal default simulator

## Evidence

- [methods] three-step architecture: "As an overview, scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation."
- [intro] scDesign2 vs scDesign3 selection: "scReadSim trains the count simulator scDesign26 (if the cells belong to distinct clusters; otherwise, scDesign322 can be used"
- [intro] ground truth purpose: "the synthetic UMI count matrix—an intermediate output of scReadSim—serves as the ground truth for benchmarking scRNA-seq UMI deduplication tools"
- [methods] paired-end FASTQ output: [evidence span withheld — anchor status: not_found]
- [abstract] abstract capability claim: "At both read-sequence and read-count levels, scReadSim mimics real scRNA-seq and scATAC-seq data."
- [readme] install from README: [evidence span withheld — anchor status: external_doc]
