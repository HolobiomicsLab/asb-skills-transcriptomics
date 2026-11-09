---
name: multi-method-quantification-agreement-analysis
description: Use when a 10x scRNA-seq dataset has been quantified for pathogen reads
  by more than one aligner or reference strategy and you must verify that positive-cell
  calls and per-cell UMI counts agree across methods.
when_to_use_negative:
- Only one quantification method is available or planned — there is no second method
  to compare against, so use a single-pipeline quantification skill instead
- Input is already a per-cell pathogen count matrix from a single tool and the goal
  is downstream Seurat analysis, not cross-method agreement assessment
- Data are not 10x single-cell with cell barcodes/UMIs (e.g. bulk viral sequencing)
  — per-cell positive-cell comparison and UMI correlation are undefined
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scPathoQuant
  role: Generates the alternative pathogen quantifications via bbmap and bowtie2 alignment
    of host-unmapped reads; outputs pathogen_al_counts.csv and pathogen-integrated
    10x matrices
  repo: https://github.com/galelab/scPathoQuant
- name: cellranger
  role: Produces the reference 10x outputs consumed via -10x and, with a combined
    host+pathogen reference, the direct pathogen UMI counts used as the comparison
    baseline
- name: bbmap
  role: Default scPathoQuant aligner used to map host-unmapped reads to the pathogen
    genome in the first comparison arm
- name: bowtie2
  role: Alternative scPathoQuant aligner run with --very-sensitive --non-deterministic
    to produce the third quantification method
- name: htseq
  role: Used by scPathoQuant (with --intersection-nonempty) to quantify reads mapping
    to pathogen genes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:7d97a2a9c0b4dc5005e4208efa4ac38901807eb1f48c5b600b1c71819b421c17
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# multi-method-quantification-agreement-analysis

## Summary

Benchmark agreement between independent pathogen-quantification pipelines (CellRanger combined-reference counts vs scPathoQuant bbmap and bowtie2 runs) by comparing per-cell positive-cell calls and correlating UMI counts, as demonstrated on SARS-CoV-2 in the Ravindra et al. 2021 HBEC 1-3 dpi time-course.

## When to use

Use when a 10x scRNA-seq dataset has been quantified for pathogen reads by more than one aligner or reference strategy and you must verify that positive-cell calls and per-cell UMI counts agree across methods.

## When NOT to use

- Only one quantification method is available or planned — there is no second method to compare against, so use a single-pipeline quantification skill instead
- Input is already a per-cell pathogen count matrix from a single tool and the goal is downstream Seurat analysis, not cross-method agreement assessment
- Data are not 10x single-cell with cell barcodes/UMIs (e.g. bulk viral sequencing) — per-cell positive-cell comparison and UMI correlation are undefined

## Inputs

- CellRanger count output folders (filtered_feature_bc_matrix, raw_feature_bc_matrix, possorted_genome_bam.bam) for each 10x sample
- Pathogen reference genome FASTA and GTF (SARS-CoV-2 GCF_009858895.2) with simplified fasta header for total-read quantification
- Host-only and combined host+pathogen CellRanger reference genomes
- scPathoQuant output CSVs (pathogen_al_counts.csv, pathogen_al_gene_counts.csv) from bbmap and bowtie2 runs

## Outputs

- Per-time-point pathogen-positive cell counts for each of the three methods (e.g. sars2_cell_counts.csv reporting 4,279 / 13,998 / 26,935 cells at 1/2/3 dpi)
- Kendall rank correlation values between scPathoQuant (bbmap, bowtie2) and CellRanger UMI counts in positive cells
- Comparison summary of positive-cell identity agreement across alignment methods

## How to apply

For each sample, generate three quantifications: (1) a CellRanger count run against a combined host (GRCh38) + pathogen (SARS-CoV-2, NCBI GCF_009858895.2) reference with the fasta header simplified (e.g. > SARS-CoV-2) so total pathogen reads are called directly; (2) scPathoQuant on the host-only CellRanger output with default bbmap (-align bbmap, -p 8, -p2genome to the SARS-CoV-2 fasta/gtf folder); (3) scPathoQuant repeated with -align bowtie2 --bowtie2_params='--very-sensitive --non-deterministic'. Extract per-cell pathogen counts from each method's pathogen_al_counts.csv and pathogen-integrated filtered_feature_bc_matrix, define pathogen-positive cells per time point, then compare positive-cell identity/counts across methods and compute Kendall rank correlations between scPathoQuant and CellRanger UMI counts in the shared positive cells. In the HBEC SARS-CoV-2 validation, all three methods must identify identical positive-cell sets (4,279 at 1 dpi, 13,998 at 2 dpi, 26,935 at 3 dpi) with high Kendall correlations, slightly higher for SPQ bbmap vs CellRanger.

## Related tools

- **scPathoQuant** (Generates the alternative pathogen quantifications via bbmap and bowtie2 alignment of host-unmapped reads; outputs pathogen_al_counts.csv and pathogen-integrated 10x matrices) — https://github.com/galelab/scPathoQuant
- **cellranger** (Produces the reference 10x outputs consumed via -10x and, with a combined host+pathogen reference, the direct pathogen UMI counts used as the comparison baseline)
- **bbmap** (Default scPathoQuant aligner used to map host-unmapped reads to the pathogen genome in the first comparison arm)
- **bowtie2** (Alternative scPathoQuant aligner run with --very-sensitive --non-deterministic to produce the third quantification method)
- **htseq** (Used by scPathoQuant (with --intersection-nonempty) to quantify reads mapping to pathogen genes)

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/viral/fastafilefolder -align bowtie2 --bowtie2_params="--very-sensitive --non-deterministic"
```

## Evaluation signals

- Positive-cell counts must match 4,279 / 13,998 / 26,935 at 1, 2, and 3 dpi and be identical across CellRanger, SPQ bbmap, and SPQ bowtie2
- Kendall correlations between SPQ and CellRanger UMI counts must be reported in sars2_cell_counts.csv and be high, with bbmap-vs-CellRanger slightly higher than bowtie2-vs-CellRanger
- pathogen_al_counts.csv must exist for each scPathoQuant run and contain total pathogen reads per cell keyed by cell barcode
- The bowtie2 run must use --very-sensitive --non-deterministic; the bbmap run must use default parameters
- The combined-reference CellRanger fasta header must be simplified (e.g. > SARS-CoV-2) since it is used for total pathogen read quantification

## Limitations

- Agreement was validated on SARS-CoV-2 in the HBEC time-course; other pathogens or lower viral loads were not assessed by this comparison card
- Temporary files in the _tmp/ directory (e.g. unmmaped.bam, barcodes_umi_read_table.csv) can be large; use --tmp_removal if space is an issue
- htseq-count uses non-default --intersection-nonempty parameter inside scPathoQuant; other tools may apply different gene-assignment rules, affecting gene-level (not total) count comparability

## Evidence

- [abstract] we have built a python package, called scPathoQuant: "we have built a python package, called scPathoQuant"
- [readme] The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads.: "The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads."
- [readme] pathogen_al_counts.csv - total number of reads mapping to the pathogen in each cell: "pathogen_al_counts.csv - total number of reads mapping to the pathogen in each cell"
- [methods] -10x = Path/to/10x/sample/ (this the path to the output folder generated by first running cellranger count ): "-10x = Path/to/10x/sample/ (this the path to the output folder generated by first running cellranger count )"
- [readme] -align = alignment tool bbmap (default) or bowtie2: "-align = alignment tool bbmap (default) or bowtie2"
