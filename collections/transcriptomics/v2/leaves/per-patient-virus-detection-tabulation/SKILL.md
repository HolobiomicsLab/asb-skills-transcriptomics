---
name: per-patient-virus-detection-tabulation
description: Use when scPathoQuant has been run with a multi-virus reference fasta
  (up to 2 entries) on 10x cellranger count outputs and you must tabulate which viruses
  were detected per patient, with per-virus infected-cell counts and read abundances.
when_to_use_negative:
- Input has no upstream scPathoQuant run — no pathogen_al_counts.csv per-cell per-pathogen
  read counts exist to aggregate.
- The pathogen reference folder contains more than 2 fasta entries, exceeding scPathoQuant's
  supported maximum of 2 files.
- The goal is pathogen gene-level quantification rather than total per-virus detection/abundance
  tabulation (pathogen_al_gene_counts.csv and a gtf would be required instead).
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0781
tools:
- name: scPathoQuant
  role: Aligns unmapped 10x reads to the multi-virus reference and produces pathogen_al_counts.csv
    per patient, which is the input aggregated into the detection table
  repo: https://github.com/galelab/scPathoQuant
- name: cellranger
  role: Produces the -10x output folders (filtered_feature_bc_matrix, raw_feature_bc_matrix)
    per patient sample that scPathoQuant consumes
- name: Python
  role: Language in which scPathoQuant is implemented and used to aggregate per-patient
    per-cell counts into patient_virus_detection_table.csv
  repo: https://github.com/galelab/scPathoQuant
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:491d6e0483dd0ed72ef645c4c71a70b7089b6b12bdce306a835164a18bf17760
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - Python
  - bowtie2
  - cellranger
  - scPathoQuant
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# per-patient-virus-detection-tabulation

## Summary

Aggregate per-patient scPathoQuant pathogen_al_counts.csv outputs into a per-patient per-virus detection table that separately reports infected-cell counts and viral read abundance for each pathogen (e.g. SARS-CoV-2 and metapneumovirus) in co-infected scRNAseq samples.

## When to use

Use when scPathoQuant has been run with a multi-virus reference fasta (up to 2 entries) on 10x cellranger count outputs and you must tabulate which viruses were detected per patient, with per-virus infected-cell counts and read abundances.

## When NOT to use

- Input has no upstream scPathoQuant run — no pathogen_al_counts.csv per-cell per-pathogen read counts exist to aggregate.
- The pathogen reference folder contains more than 2 fasta entries, exceeding scPathoQuant's supported maximum of 2 files.
- The goal is pathogen gene-level quantification rather than total per-virus detection/abundance tabulation (pathogen_al_gene_counts.csv and a gtf would be required instead).

## Inputs

- pathogen_al_counts.csv per patient (total reads mapping to each pathogen per cell, from scPathoQuant)
- 10x cellranger count output folders for each patient sample
- pathogen reference folder with at most 2 fasta entries (dual-virus reference, simplified fasta headers)
- pathogen_copy.png violin plots (per-cell infected-cell and UMI counts)

## Outputs

- patient_virus_detection_table.csv — per-patient per-virus table of infected-cell counts and viral read abundances
- Aggregated detection pattern distinguishing co-infected samples (e.g. both SARS-CoV-2 and metapneumovirus in patient 1) from single-virus samples

## How to apply

Run scPathoQuant per 10x sample with -10x pointing at the cellranger count output folder, -op to a per-patient results folder, -p 8, and -p2genome pointing to a reference folder containing at most 2 fasta entries (e.g. SARS-CoV-2 GCF_009858895.2 and a metapneumovirus genome) with simplified headers such as >HIV_virus, since scPathoQuant uses the fasta header to quantify total pathogen reads. Collect each patient's pathogen_al_counts.csv, which reports total reads mapping to each pathogen in each cell, and pathogen_copy.png violin plots for per-cell infected-cell and UMI counts. Aggregate these per-cell, per-virus counts into a single per-patient per-virus detection table (patient_virus_detection_table.csv) listing infected-cell counts and read abundances for each virus separately, exploiting scPathoQuant's ability to differentiate multiple viruses in one sample.

## Related tools

- **scPathoQuant** (Aligns unmapped 10x reads to the multi-virus reference and produces pathogen_al_counts.csv per patient, which is the input aggregated into the detection table) — https://github.com/galelab/scPathoQuant
- **cellranger** (Produces the -10x output folders (filtered_feature_bc_matrix, raw_feature_bc_matrix) per patient sample that scPathoQuant consumes)
- **Python** (Language in which scPathoQuant is implemented and used to aggregate per-patient per-cell counts into patient_virus_detection_table.csv) — https://github.com/galelab/scPathoQuant

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/pathogen/fastafilefolder
```

## Evaluation signals

- patient_virus_detection_table.csv reproduces the expected co-infection pattern: both SARS-CoV-2 and metapneumovirus detected in severe COVID patient 1 (Fig. 4A and B), SARS-CoV-2 only in patients 2 and 3.
- Each patient row reports infected-cell counts and viral read abundance separately for each virus, matching the per-virus split achievable because the fasta header defines the quantified pathogen.
- Per-virus infected-cell counts are consistent with the pathogen_copy.png violin plots (number of cells with pathogen and UMI counts).
- Each per-patient pathogen_al_counts.csv contains per-cell read totals for both entries in the dual-virus reference, confirming both viruses were quantified before aggregation.

## Limitations

- The -p2genome folder may contain at most 2 files (one fasta, optionally one gtf), so co-infection tabulation is limited to two pathogen entries per run.
- Fasta headers with complicated names hinder quantification — headers must be simplified (e.g. >HIV_virus) since they are used to count total pathogen reads.
- Pathogen gene-level counts (pathogen_genes_al_sort_counts.sam) are not produced if no pathogen/viral gtf is provided, limiting the table to whole-genome read totals.

## Evidence

- [readme] The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads.: "The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads."
- [readme] pathogen_al_counts.csv definition: "pathogen_al_counts.csv - total number of reads mapping to the pathogen in each cell"
- [readme] fasta header quantification and simplification advice: "In the fasta file the header will be used to quantify the number of pathogen total reads, it is recommended that if the fasta header is a complicated name it be simplified (i.e. > HIV_virus)."
- [abstract] multi-virus differentiation finding: "can differentiate and define multiple viruses in a single sample scRNAseq output"
- [other] test set description of co-infection quantification: [evidence span withheld — anchor status: external_doc]
