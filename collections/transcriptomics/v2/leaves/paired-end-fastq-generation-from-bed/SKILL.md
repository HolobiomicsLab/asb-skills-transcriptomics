---
name: paired-end-fastq-generation-from-bed
description: Use when scReadSim has produced synthetic read-coordinate BED files (e.g.,
  combined gene/inter-gene BED from scRNA_CombineBED) and you need actual paired-end
  FASTQ reads for downstream aligners or UMI-deduplication benchmarks.
when_to_use_negative:
- You only need synthetic count matrices (gene-by-cell UMI counts) and not sequencing
  reads — stop after GenerateSyntheticCount.scRNA_GenerateSyntheticCount
- Input BED coordinates are not from scReadSim's foreground/background feature scheme
  or lack the 10x-style CB/UMI structure, since read 1 assumes a concatenated UMI
  + cell barcode layout
- Benchmarking isoform-level or alternative-splicing tools — scReadSim generates gene-level
  UMI-count ground truths only and may emit intronic synthetic reads
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: scReadSim
  role: Provides scRNA_BED2FASTQ, scRNA_CombineBED, AlignSyntheticBam_Pair, and scRNA_ErrorBase
    for BED-to-FASTQ conversion and optional alignment/erroring
  repo: https://github.com/JSB-UCLA/scReadSim
- name: bedtools
  role: getfasta extracts reference sequence at each synthetic BED interval from the
    mm10 genome
- name: seqtk
  role: Converts the extracted FASTA sequences into FASTQ format
- name: bowtie2
  role: Aligns the synthetic paired-end FASTQs to the mm10 reference genome when BAM
    output is desired
- name: samtools
  role: Used in the alignment/sorting step producing synthetic BAM output
- name: fgbio
  role: Introduces Illumina-style base substitution errors into the synthetic reads
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:2cd90fba5d2d241bb79531c5b3f2309d25ca849807b9fb1115562b57179581d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - Picard
  - bedtools
  - bowtie2
  - fgbio
  - samtools
  - scReadSim
  - seqtk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# paired-end-fastq-generation-from-bed

## Summary

Converts scReadSim's combined synthetic BED coordinates (foreground gene regions plus background inter-gene regions) into paired-end FASTQ reads by extracting sequences from the mm10 reference genome with bedtools getfasta and converting FASTA to FASTQ with seqtk. This produces the final synthetic scRNA-seq read files that mimic 10x-style read 1 (cell barcode + UMI) and read 2 (RNA sequence) structure.

## When to use

Use when scReadSim has produced synthetic read-coordinate BED files (e.g., combined gene/inter-gene BED from scRNA_CombineBED) and you need actual paired-end FASTQ reads for downstream aligners or UMI-deduplication benchmarks.

## When NOT to use

- You only need synthetic count matrices (gene-by-cell UMI counts) and not sequencing reads — stop after GenerateSyntheticCount.scRNA_GenerateSyntheticCount
- Input BED coordinates are not from scReadSim's foreground/background feature scheme or lack the 10x-style CB/UMI structure, since read 1 assumes a concatenated UMI + cell barcode layout
- Benchmarking isoform-level or alternative-splicing tools — scReadSim generates gene-level UMI-count ground truths only and may emit intronic synthetic reads

## Inputs

- Combined synthetic-read BED file (foreground + background coordinates, from scRNA_CombineBED)
- mm10 reference genome FASTA (chromosome size file embedded in the package)
- Synthetic cell barcode file (synthetic_cell_barcode.txt) and synthetic count matrix (*.scDesign2Simulated.txt) for consistency checks
- Optional bowtie2 index of mm10 for downstream alignment

## Outputs

- *.read1.bed2fa.fq — paired-end read 1 FASTQ (cell barcode + UMI sequence string)
- *.read2.bed2fa.fq — paired-end read 2 FASTQ (90-nt RNA sequence reads)
- Optionally, sorted BAM/alignment output from AlignSyntheticBam_Pair

## How to apply

After generating synthetic read coordinates with scRNA_GenerateBAM.scRNA_GenerateBAMCoord (read_len=90, jitter_size=5, CB_len=16, UB_len=10) and merging foreground (gene) and background (inter-gene) BED files with scRNA_CombineBED, run scRNA_BED2FASTQ on the combined BED. The function extracts the reference sequence at each BED interval from the mm10 chromosome FASTA using bedtools getfasta, then converts the resulting FASTA to FASTQ with seqtk; set sort_FASTQ=True to keep reads consistently ordered. Read 1 encodes a concatenated UMI and cell barcode string while read 2 carries the RNA sequence, mirroring the 10x chemistry of the source mouse 10x Multiome data. Optionally align the FASTQs to a bowtie2-indexed mm10 reference with AlignSyntheticBam_Pair and introduce Illumina-style substitution errors with scRNA_ErrorBase (fgbio with a Picard reference dictionary).

## Related tools

- **scReadSim** (Provides scRNA_BED2FASTQ, scRNA_CombineBED, AlignSyntheticBam_Pair, and scRNA_ErrorBase for BED-to-FASTQ conversion and optional alignment/erroring) — https://github.com/JSB-UCLA/scReadSim
- **bedtools** (getfasta extracts reference sequence at each synthetic BED interval from the mm10 genome)
- **seqtk** (Converts the extracted FASTA sequences into FASTQ format)
- **bowtie2** (Aligns the synthetic paired-end FASTQs to the mm10 reference genome when BAM output is desired)
- **samtools** (Used in the alignment/sorting step producing synthetic BAM output)
- **fgbio** (Introduces Illumina-style base substitution errors into the synthetic reads)

## Examples

```
from scReadSim.scRNA_GenerateBAM import scRNA_BED2FASTQ; scRNA_BED2FASTQ(bed_file='combined.bed', genome_fa_file='chr1.fa', output_dir='out', sort_FASTQ=True)
```

## Evaluation signals

- Both paired-end output files (*.read1.bed2fa.fq and *.read2.bed2fa.fq) exist in the output directory
- Read 1 length equals CB_len + UB_len (16 + 10 = 26 bp); read 2 sequences are 90 nt, matching the default read_len
- Read 2 sequences extracted at each BED coordinate match the mm10 reference (verifiable by realigning to chr1 with bowtie2)
- Number of read pairs equals the number of records in the combined BED, and cell barcodes in read 1 match synthetic_cell_barcode.txt entries
- Binned per-window read coverage on chromosome 1 (1000-bp bins) of aligned synthetic reads resembles the real data's coverage profile

## Limitations

- The default implementation may generate intronic synthetic reads when extracting sequences from the reference genome, which can confound benchmarks such as cellranger default mode
- Multi-sample or multi-condition simulation requires one input BAM per sample/condition; the FASTQ conversion inherits this one-BAM-per-condition structure
- Ground truths are gene-level UMI counts only — no isoform abundance or alternative splicing ground truth accompanies the FASTQ output

## Evidence

- [methods] scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation.: "As an overview, scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation."
- [methods] Read 1 encodes concatenated UMI and cell barcode: "The read 1 is a concatenated sequence string of a UMI and a cell barcode."
- [methods] mm10 reference genome requirement: "This BAM file uses mm10 as reference genome, the required chromosome size file is also embedded within the package."
- [readme] FASTQ output capability: [evidence span withheld — anchor status: external_doc]
- [other] Validation artifact for paired-end FASTQ outputs: [evidence span withheld — anchor status: not_found]
