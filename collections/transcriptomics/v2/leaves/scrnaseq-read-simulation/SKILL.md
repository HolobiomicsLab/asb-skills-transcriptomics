---
name: scrnaseq-read-simulation
description: Use when a study needs synthetic scRNA-seq reads (BAM/FASTQ) that mimic
  real data at both read-sequence and read-count levels, e.g., benchmarking UMI deduplication
  tools or comparing read-coverage fidelity against simulators like minnow.
when_to_use_negative:
- Benchmarking isoform abundance or alternative splicing tools — scReadSim provides
  gene-level UMI count ground truths only, not isoform-level ground truths
- Count-level-only simulations where reads are unnecessary — scDesign2/scDesign3 alone
  suffice and are faster
- Multi-sample or multi-condition studies requiring simultaneous simulation from pooled
  inputs — one BAM file per sample/condition must be supplied separately
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scReadSim
  role: 'Core read simulator: genome segmentation, synthetic count generation, and
    synthetic read generation producing BAM/FASTQ with UMI-count ground truths'
  repo: https://github.com/JSB-UCLA/scReadSim
- name: scDesign2
  role: Count simulator trained internally by scReadSim to generate synthetic gene-by-cell
    UMI count matrices
  repo: https://github.com/JSB-UCLA/scDesign2
- name: bedtools
  role: getfasta extraction of read sequences from the mm10 reference during BED-to-FASTQ
    conversion
- name: seqtk
  role: FASTA-to-FASTQ conversion in scRNA_BED2FASTQ
- name: bowtie2
  role: Alignment of synthetic FASTQ reads to the mm10 reference genome (AlignSyntheticBam_Pair);
    scReadSim BAM output is already aligned
- name: minnow
  role: Comparator read simulator fed scReadSim's synthetic UMI count matrix to benchmark
    read-coverage fidelity (Spearman 0.065 vs scReadSim's 0.930)
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:c1934ff61b0bbf0bef642c895fb57d40c9a83277dc621e460f93c181b849145c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - IGV
  - bedtools
  - bowtie2
  - minnow
  - samtools
  - scDesign2
  - scReadSim
  - seqtk
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# scrnaseq-read-simulation

## Summary

Simulate realistic single-cell RNA-seq sequencing reads with scReadSim by training scDesign2 on real UMI count matrices and generating synthetic paired-end reads (BAM/FASTQ) with ground-truth gene-by-cell UMI counts, enabling read-level benchmarking that count-only simulators cannot support.

## When to use

Use when a study needs synthetic scRNA-seq reads (BAM/FASTQ) that mimic real data at both read-sequence and read-count levels, e.g., benchmarking UMI deduplication tools or comparing read-coverage fidelity against simulators like minnow.

## When NOT to use

- Benchmarking isoform abundance or alternative splicing tools — scReadSim provides gene-level UMI count ground truths only, not isoform-level ground truths
- Count-level-only simulations where reads are unnecessary — scDesign2/scDesign3 alone suffice and are faster
- Multi-sample or multi-condition studies requiring simultaneous simulation from pooled inputs — one BAM file per sample/condition must be supplied separately

## Inputs

- Real scRNA-seq BAM file containing reads in cells (e.g., mouse 10x Multiome RNA modality)
- Reference genome (mm10) with embedded chromosome size file
- Gene annotation GTF (e.g., gencode.vM10.annotation.gtf)
- User-specified cell number and sequencing depth parameters

## Outputs

- Synthetic paired-end reads in BAM or FASTQ format
- Synthetic gene-by-cell and inter-gene-by-cell UMI count matrices (ground truth)
- Combined BED files of foreground and background synthetic read coordinates
- Per-window pseudo-bulk coverage summaries (e.g., pseudo_bulk_coverage_summary.csv)

## How to apply

Pre-process the real scRNA-seq BAM (e.g., mouse 10x Multiome RNA modality, mm10 reference) with scRNA_CreateFeatureSets and scRNA_bam2countmat to build gene-by-cell and inter-gene-by-cell UMI count matrices. Train scDesign2 via GenerateSyntheticCount.scRNA_GenerateSyntheticCount to produce synthetic count matrices, optionally varying cell numbers (1219–19508) or sequencing depths (1.2M–20.1M reads) for experimental design. Generate synthetic read coordinates with scRNA_GenerateBAM.scRNA_GenerateBAMCoord using the default 90-nt read length and 5-bp jitter size, then combine foreground (gene) and background (inter-gene) BED files with scRNA_CombineBED. Convert BED to paired-end FASTQ via scRNA_BED2FASTQ (bedtools getfasta from mm10 followed by seqtk FASTA-to-FASTQ), optionally aligning with bowtie2 via AlignSyntheticBam_Pair. Validate by binning chromosome 1 into 1000-bp windows and computing Spearman correlation of per-window pseudo-bulk coverage between real and synthetic reads (expected ~0.930 for scReadSim vs ~0.065 for minnow).

## Related tools

- **scReadSim** (Core read simulator: genome segmentation, synthetic count generation, and synthetic read generation producing BAM/FASTQ with UMI-count ground truths) — https://github.com/JSB-UCLA/scReadSim
- **scDesign2** (Count simulator trained internally by scReadSim to generate synthetic gene-by-cell UMI count matrices) — https://github.com/JSB-UCLA/scDesign2
- **bedtools** (getfasta extraction of read sequences from the mm10 reference during BED-to-FASTQ conversion)
- **seqtk** (FASTA-to-FASTQ conversion in scRNA_BED2FASTQ)
- **bowtie2** (Alignment of synthetic FASTQ reads to the mm10 reference genome (AlignSyntheticBam_Pair); scReadSim BAM output is already aligned)
- **minnow** (Comparator read simulator fed scReadSim's synthetic UMI count matrix to benchmark read-coverage fidelity (Spearman 0.065 vs scReadSim's 0.930))

## Examples

```
pip install git+https://github.com/JSB-UCLA/scReadSim.git
```

## Evaluation signals

- Spearman correlation of per-window (1000-bp, chromosome 1) read coverage between real and scReadSim synthetic data should approach the reported 0.930 (minnow comparison yields ~0.065)
- All synthetic conditions yield valid paired-end FASTQ outputs plus a synthetic count matrix with matching cell numbers
- At fixed sequencing depth (5.0M reads), pseudo-bulk coverage of pooled synthetic cells decreases monotonically as synthetic cell number increases from 1219 to 19508
- At fixed cell number (4877), coverage increases monotonically with sequencing depth from 1.2M to 20.1M reads
- Synthetic reads span both annotated genes and intergenic regions, since the background inter-gene feature set is included

## Limitations

- Default implementation may generate intronic synthetic reads when extracting sequences from the reference genome, which can affect benchmarks run in tools like cellranger default mode
- Current version requires one BAM file per sample or condition; multi-sample/condition simulation must be run separately per input BAM
- Lacks isoform-level ground truths (isoform abundance, alternative splicing events); not recommended for benchmarking isoform or splicing tools
- Support for continuous trajectories, spatial locations, and conditions/batches would require scDesign3, which is not yet the internal count simulator

## Evidence

- [methods] Three-step pipeline: "As an overview, scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation."
- [methods] Count simulator choice: "In the second step—synthetic count generation, scReadSim trains the count simulator scDesign2"
- [methods] Read 1 structure: "The read 1 is a concatenated sequence string of a UMI and a cell barcode."
- [abstract] Motivation and scope: "we introduce scReadSim, a single-cell RNA-seq and ATAC-seq read simulator that allows user-specified ground truths and generates synthetic sequencing reads"
- [intro] minnow limitation: "minnow generates scRNA-seq reads only from annotated genes, so it cannot resemble real scRNA-seq data in intergenic regions"
- [methods] Reference genome requirement: "This BAM file uses mm10 as reference genome, the required chromosome size file is also embedded within the package."
- [readme] Install command: [evidence span withheld — anchor status: external_doc]
- [readme] Output formats: [evidence span withheld — anchor status: external_doc]
