---
name: hiv-gene-level-quantification-comparison
description: Use when you have scPathoQuant outputs from two aligner runs (bbmap and
  bowtie2) on the same 10x scRNAseq samples and need to assess agreement of per-gene
  pathogen UMI counts across aligners.
when_to_use_negative:
- Input is already an assembled correlation/summary table rather than raw per-cell
  pathogen_al_gene_counts.csv files from paired aligner runs.
- The scPathoQuant run was performed without a pathogen gtf file, so per-gene outputs
  (pathogen_genes_al_sort_counts.sam and per-gene UMI columns) were never generated
  and only total-pathogen counts exist.
- Data are not from a 10x single-cell pipeline processed by cellranger count; the
  skill depends on the per-cell × per-gene UMI matrix format that scPathoQuant integrates
  into cellranger output.
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scPathoQuant
  role: Generates the per-cell, per-gene pathogen UMI counts (pathogen_al_gene_counts.csv)
    being compared; run once per aligner with -align bbmap (default) or bowtie2.
  repo: https://github.com/galelab/scPathoQuant
- name: Python
  role: Language for loading the pathogen_al_gene_counts.csv files into pandas data
    frames and computing per-gene Pearson correlations between aligner runs.
- name: bbmap
  role: Default alignment tool option in scPathoQuant; one arm of the per-gene UMI
    correlation comparison.
- name: bowtie2
  role: Alternative aligner option in scPathoQuant (-align bowtie2); the other arm
    of the comparison, with default parameters used when aligning reads to the pathogen
    genome.
- name: htseq
  role: htseq-count quantifies reads mapping to individual pathogen genes using --intersection-nonempty,
    producing the gene-level UMI counts underlying the correlation.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:201c9e44f1a4c843bb39c0ecf5151190ae1bba14bb3565e92d3eb9a2dc5a04cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - Python
  - bbmap
  - bowtie2
  - htseq
  - scPathoQuant
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# hiv-gene-level-quantification-comparison

## Summary

Validates that scPathoQuant's two aligner options (bbmap and bowtie2) yield consistent per-gene HIV UMI quantification by computing Pearson correlations of per-cell UMI counts for the HIV genes gag, nef, pol, vif, and vpr between paired runs on the same donor samples.

## When to use

Use when you have scPathoQuant outputs from two aligner runs (bbmap and bowtie2) on the same 10x scRNAseq samples and need to assess agreement of per-gene pathogen UMI counts across aligners.

## When NOT to use

- Input is already an assembled correlation/summary table rather than raw per-cell pathogen_al_gene_counts.csv files from paired aligner runs.
- The scPathoQuant run was performed without a pathogen gtf file, so per-gene outputs (pathogen_genes_al_sort_counts.sam and per-gene UMI columns) were never generated and only total-pathogen counts exist.
- Data are not from a 10x single-cell pipeline processed by cellranger count; the skill depends on the per-cell × per-gene UMI matrix format that scPathoQuant integrates into cellranger output.

## Inputs

- pathogen_al_gene_counts.csv from a bbmap-based scPathoQuant run per donor sample (per-cell UMI counts for pathogen genes)
- pathogen_al_gene_counts.csv from a bowtie2-based scPathoQuant run per donor sample on the same 10x sample
- HIV reference genome fasta and gtf file supplied via -p2genome so gene-level (per-gag/nef/pol/vif/vpr) counts are generated
- 10x cellranger count output folders for the EGFP-HIV+ donor samples

## Outputs

- hiv_gene_umi_correlation_table.csv — a per-donor, per-gene table of Pearson correlation coefficients for gag, nef, pol, vif, and vpr

## How to apply

Run scPathoQuant twice per donor sample on the same cellranger count output, once with the default bbmap aligner and once with -align bowtie2, providing the HIV reference fasta and gtf so per-gene counts are produced. Load the resulting pathogen_al_gene_counts.csv files from each run into pandas, subset the per-cell UMI count columns to the five HIV genes gag, nef, pol, vif, and vpr, and compute the Pearson correlation between the bbmap-based and bowtie2-based counts for each gene within each donor. Assemble the per-donor, per-gene correlation values into a single table (hiv_gene_umi_correlation_table.csv). The rationale is that aligner choice should not materially change gene-level UMI quantification; in the paper this check confirmed the significant high correlations reported in Fig. 2G, supporting interchangeability of the two aligner options in scPathoQuant.

## Related tools

- **scPathoQuant** (Generates the per-cell, per-gene pathogen UMI counts (pathogen_al_gene_counts.csv) being compared; run once per aligner with -align bbmap (default) or bowtie2.) — https://github.com/galelab/scPathoQuant
- **Python** (Language for loading the pathogen_al_gene_counts.csv files into pandas data frames and computing per-gene Pearson correlations between aligner runs.)
- **bbmap** (Default alignment tool option in scPathoQuant; one arm of the per-gene UMI correlation comparison.)
- **bowtie2** (Alternative aligner option in scPathoQuant (-align bowtie2); the other arm of the comparison, with default parameters used when aligning reads to the pathogen genome.)
- **htseq** (htseq-count quantifies reads mapping to individual pathogen genes using --intersection-nonempty, producing the gene-level UMI counts underlying the correlation.)

## Examples

```
import pandas as pd; bb=pd.read_csv('donor1_bbmap/pathogen_al_gene_counts.csv',index_col=0); bt=pd.read_csv('donor1_bowtie2/pathogen_al_gene_counts.csv',index_col=0); genes=['gag','nef','pol','vif','vpr']; r={g: bb[g].corr(bt[g]) for g in genes}
```

## Evaluation signals

- Each of the five HIV genes (gag, nef, pol, vif, vpr) shows a significant and high Pearson correlation between bbmap and bowtie2 counts in each donor sample, matching the Fig. 2G result of the article.
- The output table hiv_gene_umi_correlation_table.csv contains one correlation value per donor × gene combination (e.g. 2 donors × 5 genes = 10 rows).
- Both pathogen_al_gene_counts.csv files (bbmap-based and bowtie2-based) exist for each donor and share the same per-cell barcode index, enabling a row-aligned comparison.
- Correlation columns were computed only on the five HIV gene UMI count columns, not on total pathogen copy-number columns from pathogen_al_counts.csv.

## Limitations

- Non-default htseq-count settings are used (--intersection-nonempty); per-gene quantification depends on providing the pathogen gtf — without it, pathogen_genes_al_sort_counts.sam and per-gene counts are not produced.
- Total pathogen read quantification uses the fasta header name, so complicated reference headers should be simplified (e.g. > HIV_virus) or counts can be mislabeled.
- Comparison is validated for the five HIV genes in EGFP-HIV+ cells from two donor samples; high correlations should not be assumed for other pathogens or genes without repeating the check.
- Temporary files in the _tmp/ directory can be large; use --tmp_removal if space is an issue.

## Evidence

- [results] Per-gene UMI count correlations between bowtie2 and bbmap for the five HIV genes were significant and high in each donor sample, indicating consistent quantification of HIV gene counts across the two aligner options in SPQ.: [evidence span withheld — anchor status: not_found]
- [readme] pathogen_al_gene_counts.csv - number of reads mapping to pathoen genes in each cell: "pathogen_al_gene_counts.csv - number of reads mapping to pathoen genes in each cell"
- [readme] -align = alignment tool bbmap (default) or bowtie2: "-align = alignment tool bbmap (default) or bowtie2"
- [readme] Default parameters are used when aligning reads to the pathogen genome and and non default htseq-count include using the --intersection-nonempty parameter.: "Default parameters are used when aligning reads to the pathogen genome and and non default htseq-count include using the --intersection-nonempty parameter."
- [other] confirm that the computed correlations for the five genes in each donor are significant and high, consistent with the significant high correlations reported in Fig. 2G of the article: [evidence span withheld — anchor status: not_found]
