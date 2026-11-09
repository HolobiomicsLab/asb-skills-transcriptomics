---
name: spliced-read-alignment-hisat
description: Use when reconstructing mRNA 3' end cleavage sites from Microwell-seq
  (MCA/HCL) FASTQ data at single-nucleotide resolution, after PEAR merging and poly(A)
  removal, before peak calling.
when_to_use_negative:
- Input is standard 10x Genomics data, where most reads do not span mRNA 3' end cleavage
  sites and exact de novo CS mapping is prevented — use scUTRquant isoform quantification
  instead
- Goal is gene/3'UTR isoform expression quantification from existing libraries — the
  scUTRquant kallisto-bustools pipeline, not de novo HISAT alignment, is the right
  tool
- Reads still carry poly(A) tails or are unmerged single-end reads, since aligning
  non-genomic sequence would produce false cleavage site coordinates
edam_operation: http://edamontology.org/operation_3350
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: HISAT
  role: Spliced aligner used to map poly(A)-trimmed Microwell-seq reads to hg38 and
    mm10 genomes for cleavage site identification
- name: PEAR
  role: Merges overlapping paired-end Microwell-seq reads to reconstruct full insert
    sequences carrying 3'-end information prior to alignment
- name: mca-utrome
  role: Snakemake pipeline that calls 3' UTR cleavage sites de novo from the Mouse
    Cell Atlas, producing the Mouse UTRome used downstream by scUTRquant
  repo: https://github.com/Mayrlab/mca-utrome
- name: hcl-utrome
  role: Reprocesses Human Cell Landscape data for cleavage site identification, producing
    the Human UTRome
  repo: https://github.com/Mayrlab/hcl-utrome
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:1fc158f7899da1bfc368b3d1b5ef45a811b3462af542f64093b4fcbe0e606e1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/mca-utrome
  tools:
  - HISAT
  - PEAR
  - hcl-utrome
  - mca-utrome
  - scUTRquant
  edam_operation: http://edamontology.org/operation_3350
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spliced-read-alignment-hisat

## Summary

Aligns poly(A)-trimmed Microwell-seq read inserts to the hg38 and mm10 reference genomes with HISAT so that read 5' ends can be aggregated per cell type and cleavage site (CS) candidates can be called de novo for the mouse/human UTRome atlases.

## When to use

Use when reconstructing mRNA 3' end cleavage sites from Microwell-seq (MCA/HCL) FASTQ data at single-nucleotide resolution, after PEAR merging and poly(A) removal, before peak calling.

## When NOT to use

- Input is standard 10x Genomics data, where most reads do not span mRNA 3' end cleavage sites and exact de novo CS mapping is prevented — use scUTRquant isoform quantification instead
- Goal is gene/3'UTR isoform expression quantification from existing libraries — the scUTRquant kallisto-bustools pipeline, not de novo HISAT alignment, is the right tool
- Reads still carry poly(A) tails or are unmerged single-end reads, since aligning non-genomic sequence would produce false cleavage site coordinates

## Inputs

- Merged paired-end Microwell-seq insert sequences (PEAR output) with poly(A) tails removed
- Mouse Cell Atlas FASTQ files (GEO:GSE108097)
- Human Cell Landscape FASTQ files (GEO:GSE134355)
- hg38 and mm10 reference genomes
- MCA/HCL cell barcode annotations (barcodes/UMIs)

## Outputs

- HISAT alignments (BAM) of trimmed reads to hg38/mm10
- Per-cell-type aggregation of read 5' ends used for coverage peak calling
- Coordinate-annotated CS candidate GTF records (mouse N=170,617; human N=150,191)

## How to apply

Download the Microwell-seq FASTQ files for the Mouse Cell Atlas (GEO:GSE108097) and Human Cell Landscape (GEO:GSE134355), merge overlapping paired-end reads with PEAR to reconstruct full inserts carrying 3'-end information, and remove poly(A) tails so only genomic-derived sequence is aligned. Run HISAT in spliced mode against the hg38 and mm10 reference genomes, which permits reads spanning exon junctions to be placed while preserving the coordinate of the read 5' end (the cleavage site proxy). Extract cell barcodes/UMIs from merged reads and assign reads to cell types via the MCA/HCL annotations so that coverage peaks (CS candidates) can later be aggregated per cell type. The alignment output feeds read 5'-end aggregation and peak calling, followed by merging candidate peaks within 30 nt and restricting to GENCODE v39/vM21 protein-coding transcripts with verified 3' ends, excluding mRNA_end_NF-tagged transcripts. Correct execution is validated by the expected atlas sizes: mouse N=170,617 and human N=150,191 CS candidates as coordinate-annotated GTF records.

## Related tools

- **HISAT** (Spliced aligner used to map poly(A)-trimmed Microwell-seq reads to hg38 and mm10 genomes for cleavage site identification)
- **PEAR** (Merges overlapping paired-end Microwell-seq reads to reconstruct full insert sequences carrying 3'-end information prior to alignment)
- **mca-utrome** (Snakemake pipeline that calls 3' UTR cleavage sites de novo from the Mouse Cell Atlas, producing the Mouse UTRome used downstream by scUTRquant) — https://github.com/Mayrlab/mca-utrome
- **hcl-utrome** (Reprocesses Human Cell Landscape data for cleavage site identification, producing the Human UTRome) — https://github.com/Mayrlab/hcl-utrome

## Examples

```
pear -f R1.fastq.gz -r R2.fastq.gz -o merged && hisat -x hg38 -U merged.assembled.fastq -S aligned.sam
```

## Evaluation signals

- Mouse candidate set contains exactly N=170,617 CS candidates as coordinate-annotated GTF records
- Human candidate set contains exactly N=150,191 CS candidates as coordinate-annotated GTF records
- All candidate peaks are restricted to protein-coding transcripts with verified 3' ends in GENCODE v39/vM21 and no mRNA_end_NF-tagged transcripts remain
- Adjacent candidate peaks have been merged within 30 nt so no two candidates overlap within that window
- Rigorous QC and internal-priming filtering has been applied, excluding peaks arising from priming at genomic adenosine stretches

## Limitations

- This HISAT-based de novo mapping depends on MWS reads spanning the 3' end; 10x Genomics data mostly do not span CS and cannot be used for exact de novo mapping
- Genomic adenosine stretches cause internal priming artifacts that must be filtered from candidate peaks
- Low-abundance reads must be filtered out before peak calling, so rare cleavage sites may be missed

## Evidence

- [methods] Align to hg38/mm10 with HISAT as step 4 of the CS atlas workflow: [evidence span withheld — anchor status: not_found]
- [readme] mca-utrome README describes de novo CS calling from the Mouse Cell Atlas: [evidence span withheld — anchor status: external_doc]
- [other] CS candidates produced by removing poly(A) tails and filtering low-abundance reads and internal priming: [evidence span withheld — anchor status: not_found]
- [other] Validation of expected candidate set sizes: [evidence span withheld — anchor status: not_found]
- [discussion] QC and internal priming filtering for MWS CS: "We performed rigorous quality control and filtering for internal priming and show that MWS CS are of high quality"
