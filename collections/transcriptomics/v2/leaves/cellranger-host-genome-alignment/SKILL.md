---
name: cellranger-host-genome-alignment
description: Use when raw 10x scRNA-seq FASTQs (e.g. HBEC SARS-CoV-2 time-course samples
  at 1-3 dpi) must first be aligned to the host genome (GRCh38) with cellranger count
  to produce the 10x output folders and unmapped host reads required by scPathoQuant.
when_to_use_negative:
- Input is already a completed cellranger count output folder — at that point scPathoQuant
  should be run directly with -10x rather than re-aligning
- Data are not 10x single-cell scRNA-seq FASTQs (e.g. bulk RNA-seq or non-10x platforms),
  since cellranger count's output structure is specific to 10x Genomics libraries
- Only downstream Seurat analysis of already pathogen-integrated matrices is needed
  — use Read10X on the scPathoQuant filtered_feature_bc_matrix instead of re-running
  alignment
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: cellranger
  role: 'Primary aligner: runs cellranger count against the host reference (e.g. GRCh38)
    to generate the 10x output folders and the host-unmapped reads consumed by scPathoQuant'
- name: scPathoQuant
  role: 'Downstream consumer: takes the -10x path to the cellranger output folder,
    extracts unaligned reads, and maps them to the pathogen genome'
  repo: github.com/galelab/scPathoQuant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:e4b50a8762d3ff37192d2f82abfd4c1a05b5b502fc7c1e7c847f78ce938990b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bbmap
  - bowtie2
  - cellranger
  - scPathoQuant
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cellranger-host-genome-alignment

## Summary

Run 10x Genomics cellranger count to align scRNA-seq reads to a host reference genome (e.g. GRCh38), generating the filtered_feature_bc_matrix/raw_feature_bc_matrix outputs and the host-unmapped reads (possorted_genome_bam.bam) that scPathoQuant consumes to quantify pathogen UMI counts.

## When to use

Use when raw 10x scRNA-seq FASTQs (e.g. HBEC SARS-CoV-2 time-course samples at 1-3 dpi) must first be aligned to the host genome (GRCh38) with cellranger count to produce the 10x output folders and unmapped host reads required by scPathoQuant.

## When NOT to use

- Input is already a completed cellranger count output folder — at that point scPathoQuant should be run directly with -10x rather than re-aligning
- Data are not 10x single-cell scRNA-seq FASTQs (e.g. bulk RNA-seq or non-10x platforms), since cellranger count's output structure is specific to 10x Genomics libraries
- Only downstream Seurat analysis of already pathogen-integrated matrices is needed — use Read10X on the scPathoQuant filtered_feature_bc_matrix instead of re-running alignment

## Inputs

- 10x scRNA-seq FASTQ files per sample (e.g. 1, 2, 3 dpi HBEC time points)
- Host transcriptome reference genome (e.g. GRCh38) prebuilt for cellranger
- Optional combined host + pathogen reference (GRCh38 + SARS-CoV-2, GCF_009858895.2, with simplified fasta header)

## Outputs

- cellranger count output folder per sample (consumed by scPathoQuant via -10x)
- filtered_feature_bc_matrix and raw_feature_bc_matrix with features.tsv.gz and matrix.mtx.gz
- possorted_genome_bam.bam containing the host-unmapped reads pulled by scPathoQuant into _tmp/unmmaped.bam
- Optionally, pathogen-integrated matrices when the combined host+pathogen reference was used

## How to apply

Run `cellranger count` on each sample's FASTQs against the host transcriptome reference (e.g. GRCh38 for the Ravindra et al. 2021 HBEC SARS-CoV-2 1, 2, and 3 dpi time points); this aligns reads to the host genome of interest and generates the standard 10x outputs including features.tsv.gz and matrix.mtx.gz in the filtered_feature_bc_matrix and raw_feature_bc_matrix folders. Optionally run a second cellranger count against a combined GRCh38 + pathogen (e.g. SARS-CoV-2 from NCBI GCF_009858895.2) reference so viral reads are called directly; the pathogen fasta header should be simplified (e.g. > SARS-CoV-2) since it is used for total pathogen read quantification. The unaligned host reads are retained in possorted_genome_bam.bam, and the path to the cellranger output folder is passed to scPathoQuant via its -10x parameter, where scPathoQuant extracts the unmapped reads and maps them to the pathogen genome before integrating pathogen UMI counts back into the cellranger files.

## Related tools

- **cellranger** (Primary aligner: runs cellranger count against the host reference (e.g. GRCh38) to generate the 10x output folders and the host-unmapped reads consumed by scPathoQuant)
- **scPathoQuant** (Downstream consumer: takes the -10x path to the cellranger output folder, extracts unaligned reads, and maps them to the pathogen genome) — github.com/galelab/scPathoQuant

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/pathogen/fastafilefolder
```

## Evaluation signals

- cellranger count completes per sample and produces filtered_feature_bc_matrix and raw_feature_bc_matrix folders containing features.tsv.gz and matrix.mtx.gz
- possorted_genome_bam.bam exists per sample so scPathoQuant can extract unmapped reads into _tmp/unmmaped.bam
- Downstream cross-check: all three methods (CellRanger combined-reference, SPQ bbmap, SPQ bowtie2) identify identical SARS-CoV-2+ cells — 4,279 at 1 dpi, 13,998 at 2 dpi, 26,935 at 3 dpi — in the HBEC dataset
- Kendall rank correlations between scPathoQuant UMI counts and CellRanger UMI counts are computed and reported in sars2_cell_counts.csv, with SPQ bbmap slightly higher than bowtie2
- Reproducibility: replicate the numbered testSet scripts (e.g. 2b.CellRanger_sb.sh followed by 3.scPathoQuant.sh) from galelab/Whitmore_scPathoQuant_testSets

## Limitations

- Two separate cellranger count runs may be needed (host-only for scPathoQuant input, host+pathogen combined for direct viral calls), increasing compute and disk usage
- Temporary files can be large: scPathoQuant's --tmp_removal flag exists because _tmp/ files 'can be large so if space is an issue these should be deleted'
- raw_feature_bc_matrix integration into Seurat 'is not always needed' per the README, so avoid unnecessary downstream loading
- scPathoQuant only sees reads unaligned to the host, so pathogen reads that cellranger aligns to homologous host sequence are never quantified as pathogen

## Evidence

- [intro] pathogen UMI counts and pathogen UMI gene counts into 10x files generated by first running cellranger count: "pathogen UMI counts and pathogen UMI gene counts into 10x files generated by first running cellranger count"
- [intro] generated by first running [cellranger count] which aligns reads to your host genome of interest: [evidence span withheld — anchor status: not_found]
- [readme] This generates output files for each sample such as features.tsv.gz and matrix.mtx.gz in the filtered_feature_bc_matrix and raw_feature_bc_matrix folders.: "This generates output files for each sample such as features.tsv.gz and matrix.mtx.gz in the filtered_feature_bc_matrix and  raw_feature_bc_matrix folders."
- [readme] scPathoQuant then takes the unaligned reads from the host genome and maps them to your pathogen genome of interest and then integrates them back into the cellranger files.: "scPathoQuant then takes the unaligned reads from the host genome and maps them to your pathogen genome of interest and then integrates them back into the cellranger files."
- [readme] unmmaped.bam - all unmapped reads from CellRanger (pulled from possorted_genome_bam.bam): "unmmaped.bam - all unmapped reads from CellRanger (pulled from possorted_genome_bam.bam)"
