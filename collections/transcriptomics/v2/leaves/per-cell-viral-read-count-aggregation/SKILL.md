---
name: per-cell-viral-read-count-aggregation
description: Use when you have per-sample pathogen_al_counts.csv files from scPathoQuant
  runs over a multi-virus reference (e.g. SARS-CoV-2 + metapneumovirus) and need to
  aggregate per-cell counts into a per-virus infected-cell and read-abundance table.
when_to_use_negative:
- The input is not 10x scRNAseq data already processed with cellranger count (scPathoQuant
  expects the cellranger output folder as -10x input).
- You need pathogen gene-level quantification rather than total pathogen reads per
  cell — that requires supplying a pathogen gtf alongside the fasta.
- The reference folder would contain more than 2 fasta entries, which exceeds the
  documented scPathoQuant multi-virus configuration.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0781
tools:
- name: scPathoQuant
  role: Extracts unaligned reads from the cellranger output, maps them to the multi-virus
    reference, and reports total reads mapping to each pathogen in each cell (pathogen_al_counts.csv)
    and per-cell infected-cell/UMI violin plots (pathogen_copy.png)
  repo: https://github.com/galelab/scPathoQuant
- name: cellranger
  role: Runs cellranger count to align reads to the host genome first; its output
    folder is the -10x input for scPathoQuant
- name: Python
  role: Language of the scPathoQuant package and the scripting used to aggregate per-patient
    per-cell counts into the detection table
  repo: https://github.com/galelab/scPathoQuant
- name: samtools
  role: External dependency used by scPathoQuant to process pathogen-aligned reads
    during quantification
- name: htseq
  role: External dependency (htseq-count with --intersection-nonempty) used by scPathoQuant
    to quantify pathogen reads
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:f07f5ab09132ad257bdcdbeedb84e2c857551066159dfd5d25dcb02b22436136
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - Python
  - bowtie2
  - cellranger
  - htseq
  - samtools
  - scPathoQuant
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# per-cell-viral-read-count-aggregation

## Summary

Aggregates scPathoQuant pathogen_al_counts.csv outputs (total pathogen reads per cell per sample) into a per-patient, per-virus detection table listing infected-cell counts and read abundance for each pathogen, exploiting scPathoQuant's ability to differentiate multiple viruses in a single 10x scRNAseq sample.

## When to use

Use when you have per-sample pathogen_al_counts.csv files from scPathoQuant runs over a multi-virus reference (e.g. SARS-CoV-2 + metapneumovirus) and need to aggregate per-cell counts into a per-virus infected-cell and read-abundance table.

## When NOT to use

- The input is not 10x scRNAseq data already processed with cellranger count (scPathoQuant expects the cellranger output folder as -10x input).
- You need pathogen gene-level quantification rather than total pathogen reads per cell — that requires supplying a pathogen gtf alongside the fasta.
- The reference folder would contain more than 2 fasta entries, which exceeds the documented scPathoQuant multi-virus configuration.

## Inputs

- cellranger count output folders for each 10x scRNAseq sample (filtered_feature_bc_matrix / raw_feature_bc_matrix)
- Pathogen reference folder with at most 2 fasta files (simplified headers, e.g. SARS-CoV-2 GCF_009858895.2 and metapneumovirus genome; optional gtf)
- Per-sample pathogen_al_counts.csv files produced by scPathoQuant
- pathogen_copy.png violin plots for per-cell infected-cell and UMI counts

## Outputs

- patient_virus_detection_table.csv: per-patient per-virus table of infected-cell counts and read abundance
- Per-cell read counts for each pathogen (pathogen_al_counts.csv columns)
- pathogen_copy.png violin plots of infected-cell and UMI counts per virus

## How to apply

Run scPathoQuant per 10x sample (cellranger count output folder via -10x, results via -op, -p 8, -p2genome pointing to a folder with at most 2 simplified-header fasta entries such as the SARS-CoV-2 reference GCF_009858895.2 and a metapneumovirus genome), keeping the default bbmap aligner. Collect each sample's pathogen_al_counts.csv, which reports the total number of reads mapping to each pathogen in each cell — the simplified fasta header (e.g. >HIV_virus style) is what scPathoQuant uses to quantify total pathogen reads, so headers must be distinct per virus. Aggregate the per-patient per-cell counts into a per-patient per-virus table listing infected-cell counts and read abundances for each virus separately (e.g. patient_virus_detection_table.csv), counting a cell as infected for a given virus when it has nonzero mapped reads in pathogen_al_counts.csv. Optionally consult pathogen_copy.png violin plots for per-cell infected-cell and UMI count distributions. Validate the table against the expected biological pattern: both SARS-CoV-2 and metapneumovirus detected in the co-infected patient, and SARS-CoV-2 only in the other patients (Fig. 4A and B).

## Related tools

- **scPathoQuant** (Extracts unaligned reads from the cellranger output, maps them to the multi-virus reference, and reports total reads mapping to each pathogen in each cell (pathogen_al_counts.csv) and per-cell infected-cell/UMI violin plots (pathogen_copy.png)) — https://github.com/galelab/scPathoQuant
- **cellranger** (Runs cellranger count to align reads to the host genome first; its output folder is the -10x input for scPathoQuant)
- **Python** (Language of the scPathoQuant package and the scripting used to aggregate per-patient per-cell counts into the detection table) — https://github.com/galelab/scPathoQuant
- **samtools** (External dependency used by scPathoQuant to process pathogen-aligned reads during quantification)
- **htseq** (External dependency (htseq-count with --intersection-nonempty) used by scPathoQuant to quantify pathogen reads)

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/viral/fastafilefolder
```

## Evaluation signals

- patient_virus_detection_table.csv reproduces the Fig. 4 pattern: both SARS-CoV-2 and metapneumovirus detected in the co-infected severe COVID patient 1, with respective infected-cell counts and read abundances.
- SARS-CoV-2-only detection in patients 2 and 3 (no metapneumovirus cells or reads reported).
- pathogen_al_counts.csv in each results folder reports nonzero total reads mapping to each expected pathogen, keyed by the simplified fasta header.
- pathogen_copy.png violin plots show plausible per-cell UMI count distributions for each detected virus.
- Infected-cell counts per virus match the number of cells with nonzero entries in the corresponding pathogen_al_counts.csv column.

## Limitations

- scPathoQuant supports at most 2 fasta files in the -p2genome reference folder, so this aggregation is limited to distinguishing at most two pathogens per run.
- The fasta header is used to quantify total pathogen reads; complicated headers must be simplified (e.g. >HIV_virus) or per-virus attribution breaks.
- Temporary files in _tmp/ can be large; use --tmp_removal if disk space is an issue.

## Evidence

- [abstract] can differentiate and define multiple viruses in a single sample scRNAseq output: "can differentiate and define multiple viruses in a single sample scRNAseq output"
- [results] SPQ detected both SARS-CoV-2 and metapneumovirus in severe COVID patient 1: [evidence span withheld — anchor status: not_found]
- [readme] pathogen_al_counts.csv - total number of reads mapping to the pathogen in each cell: "pathogen_al_counts.csv - total number of reads mapping to the pathogen in each cell"
- [readme] In the fasta file the header will be used to quantify the number of pathogen total reads, it is recommended that if the fasta header is a complicated name it be simplified: "In the fasta file the header will be used to quantify the number of pathogen total reads, it is recommended that if the fasta header is a complicated name it be simplified (i.e. > HIV_virus)."
- [methods] -10x = Path/to/10x/sample/ (this the path to the output folder generated by first running cellranger count ): "-10x = Path/to/10x/sample/ (this the path to the output folder generated by first running cellranger count )"
