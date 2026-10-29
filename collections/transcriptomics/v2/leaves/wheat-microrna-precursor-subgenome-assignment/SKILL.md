---
name: wheat-microrna-precursor-subgenome-assignment
description: Use when you have a curated list of wheat (Triticum aestivum) miRNA precursors
  with genomic coordinates and need to quantify their A/B/D subgenome distribution
  and homologous-group (triad) structure.
when_to_use_negative:
- sRNA-Seq data or precursor candidates from species without a reference genome —
  iwa-miRNA 'cannot handle sRNA-Seq data from species without genome sequences'
- Sequences/reads that do not map to the reference genome — the current iwa-miRNA
  version ignores unmapped reads
- Species with a single (non-homeologous) genome, where A/B/D subgenome assignment
  and homologous-group triad analysis are meaningless
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: iwa-miRNA
  role: Source of the curated wheat miRNA precursor list (Manual Curation module output)
    that is assigned to subgenomes and clustered into homologous groups
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Framework hosting the iwa-miRNA modules used to generate and curate the precursor
    list prior to subgenome assignment
  repo: https://github.com/cma2015/iwa-miRNA
- name: Rmarkdown
  role: Used to summarize analysis results into HTML pages for easy visualization,
    interpretation, and sharing of the subgenome/homologous-group summary
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:1251c51850e6e33489479269128226a4d79d07f5d03fecbaa89ff9411ffc251a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - Rmarkdown
  - iwa-miRNA
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# wheat-microrna-precursor-subgenome-assignment

## Summary

Assigns wheat miRNA precursors (exported from the iwa-miRNA Manual Curation module) to the A, B, and D subgenomes of hexaploid bread wheat using GFF3/GTF coordinates, then clusters sequence-similar precursors into homologous groups and flags complete A:B:D triads to reveal homeologous-group structure.

## When to use

Use when you have a curated list of wheat (Triticum aestivum) miRNA precursors with genomic coordinates and need to quantify their A/B/D subgenome distribution and homologous-group (triad) structure.

## When NOT to use

- sRNA-Seq data or precursor candidates from species without a reference genome — iwa-miRNA 'cannot handle sRNA-Seq data from species without genome sequences'
- Sequences/reads that do not map to the reference genome — the current iwa-miRNA version ignores unmapped reads
- Species with a single (non-homeologous) genome, where A/B/D subgenome assignment and homologous-group triad analysis are meaningless

## Inputs

- Selected wheat miRNA precursor list exported from the iwa-miRNA Manual Curation module
- GFF3/GTF genome annotation file with genomic coordinates for the wheat A, B, and D subgenomes
- FASTA file of miRNA precursor sequences
- Tabular (TSV/CSV) metadata table of the selected precursors

## Outputs

- homologous_groups_summary.tsv with total homologous groups, grouped precursor count, complete-triad count and fraction, and per-subgenome (A/B/D) precursor counts
- Precursor-to-subgenome assignment table (A, B, or D per precursor)
- Homologous-group membership table with flags for complete A:B:D triads

## How to apply

Load the selected wheat miRNA precursor list exported from the iwa-miRNA Manual Curation module in three formats: table, GFF3/GTF, and FASTA. Using the genomic coordinates in the GFF3/GTF annotation, assign each precursor to its homeologous subgenome (A, B, or D). Next, cluster precursors with sequence similarity across the three subgenomes into homologous groups, and flag any group that contains a complete A:B:D triad. Compute per-subgenome precursor counts (A, B, and D) and write a group-level summary — total groups, grouped precursors, triad count and fraction, and per-subgenome counts — to homologous_groups_summary.tsv. For the published wheat case study, this procedure grouped 1919 precursors into 1565 homologous groups spanning eight A:B:D configurations, with 7.54% (118/1565) of groups containing complete triads and the D subgenome holding the fewest precursors (571). Summaries and visualizations can be rendered via Rmarkdown HTML pages as done in iwa-miRNA.

## Related tools

- **iwa-miRNA** (Source of the curated wheat miRNA precursor list (Manual Curation module output) that is assigned to subgenomes and clustered into homologous groups) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Framework hosting the iwa-miRNA modules used to generate and curate the precursor list prior to subgenome assignment) — https://github.com/cma2015/iwa-miRNA
- **Rmarkdown** (Used to summarize analysis results into HTML pages for easy visualization, interpretation, and sharing of the subgenome/homologous-group summary)

## Examples

```
Rscript wheat_homolog_groups.R --table selected_precursors.tsv --gff3 wheat_ref.gff3 --fasta precursors.fa --out homologous_groups_summary.tsv
```

## Evaluation signals

- homologous_groups_summary.tsv reports 1565 homologous groups derived from 1919 grouped precursors (reproducing the published wheat case study)
- Complete-triad fraction equals 7.54% (118/1565 groups containing a full A:B:D triad)
- Per-subgenome counts are A: 611, B: 744, D: 571, with D smallest among the three subgenomes
- Homologous groups cover eight distinct A:B:D configuration classes (all subsets of the three subgenomes)
- Every precursor in the table/GFF3/GTF/FASTA inputs is assigned to exactly one of the A, B, or D subgenomes (no unassigned loci)

## Limitations

- Cannot handle sRNA-Seq data from species without genome sequences, so subgenome assignment depends on a reference wheat genome with GFF3/GTF coordinates
- Sequencing reads that do not map to the reference genome are ignored in the current iwa-miRNA version, which may exclude some precursor candidates from subgenome analysis
- Aggregated candidate lists can contain false positives, so curated precursors may need wet-lab validation before functional interpretation of subgenome patterns

## Evidence

- [other] Load the wheat selected miRNA precursor list exported from the iwa-miRNA Manual Curation module in table, GFF3/GTF, and FASTA formats.: [evidence span withheld — anchor status: not_found]
- [other] Assign each selected precursor to its homeologous subgenome (A, B, or D) using the genomic coordinates in the GFF3/GTF annotation.: [evidence span withheld — anchor status: not_found]
- [other] Validation values for the summary: [evidence span withheld — anchor status: not_found]
- [methods] Rmarkdown output rendering: "summarizes the results into HTML pages, using Rmarkdown for easy visualization, interpretation, and sharing."
- [readme] Module structure of iwa-miRNA: "The iwa-miRNA is composed with three functional modules: MiRNA Compilation, MiRNA Selection, and Manual Curation."
