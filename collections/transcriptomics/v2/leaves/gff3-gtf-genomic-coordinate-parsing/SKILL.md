---
name: gff3-gtf-genomic-coordinate-parsing
description: Use when a feature list (e.g., miRNA precursors in table, GFF3/GTF, and
  FASTA formats) must be mapped to chromosome coordinates or assigned to subgenomes
  of a polyploid genome such as hexaploid bread wheat.
when_to_use_negative:
- Species without a reference genome or without GFF3/GTF annotations, since iwa-miRNA
  cannot handle sRNA-Seq data from species without genome sequences
- Precursors whose genomic coordinates are unplaced or ambiguous, which would corrupt
  subgenome assignment
- Analyses needing downstream enrichment (GO, miRNA set enrichment) or cross-species
  comparison, which the coordinate-parsing step does not provide
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0780
tools:
- name: iwa-miRNA
  role: Produces the curated precursor list and formats genome sequences and annotation
    (genomePrepare) that the coordinate parsing consumes
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Hosts the iwa-miRNA modules so GFF3/GTF parsing and subgenome assignment run
    as reproducible web-based steps
  repo: https://github.com/cma2015/iwa-miRNA
- name: Rmarkdown
  role: Summarizes coordinate-based subgenome and homologous-group results into HTML
    pages for visualization and sharing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:5162eb792e180d8525a83903fa477ed55f39a8828d6c66bcfc52ef8b55abb4fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - Rmarkdown
  - iwa-miRNA
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# GFF3/GTF genomic coordinate parsing

## Summary

Parses GFF3/GTF genome annotations to assign genomic features (e.g., wheat miRNA precursors from iwa-miRNA) to chromosomes and subgenomes using their coordinate fields, enabling polyploid-specific analyses such as A/B/D homeologous grouping.

## When to use

Use when a feature list (e.g., miRNA precursors in table, GFF3/GTF, and FASTA formats) must be mapped to chromosome coordinates or assigned to subgenomes of a polyploid genome such as hexaploid bread wheat.

## When NOT to use

- Species without a reference genome or without GFF3/GTF annotations, since iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences
- Precursors whose genomic coordinates are unplaced or ambiguous, which would corrupt subgenome assignment
- Analyses needing downstream enrichment (GO, miRNA set enrichment) or cross-species comparison, which the coordinate-parsing step does not provide

## Inputs

- GFF3/GTF genome annotation file with chromosome coordinates
- Table of selected miRNA precursor candidates (from iwa-miRNA Manual Curation module)
- FASTA sequences of the selected precursors

## Outputs

- Precursor-to-subgenome assignment table (A, B, or D per precursor)
- Per-subgenome precursor counts (e.g., homologous_groups_summary.tsv fields)
- Coordinate-annotated precursor list feeding homologous-group clustering

## How to apply

Load the genome annotation in GFF3/GTF format via genomePrepare-style formatting, then join each precursor identifier from the curated selection (e.g., the 2030 wheat candidates exported from the iwa-miRNA Manual Curation module) to its seqid and start/end coordinates. Infer subgenome membership from the chromosome naming convention of hexaploid bread wheat (Triticum aestivum) chromosomes carrying A, B, or D homeologs, assigning each precursor to exactly one subgenome. Filter out features whose coordinates are ambiguous or unplaced, then pass the coordinate-assigned table onward for sequence-similarity clustering into homologous groups. Validate by reconciling per-subgenome totals against expected counts (e.g., A: 611; B: 744; D: 571 for the wheat case study, totaling 1919 clearly located precursors).

## Related tools

- **iwa-miRNA** (Produces the curated precursor list and formats genome sequences and annotation (genomePrepare) that the coordinate parsing consumes) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Hosts the iwa-miRNA modules so GFF3/GTF parsing and subgenome assignment run as reproducible web-based steps) — https://github.com/cma2015/iwa-miRNA
- **Rmarkdown** (Summarizes coordinate-based subgenome and homologous-group results into HTML pages for visualization and sharing)

## Examples

```
Rscript assign_subgenomes.R --gff3 wheat_annotation.gff3 --precursors selected_precursors.tsv --out subgenome_assignments.tsv
```

## Evaluation signals

- Per-subgenome counts match published values: A: 611, B: 744, D: 571, totaling 1919 clearly located precursors
- Grouped precursors (1919) plus unplaced precursors reconcile with the total of 2030 selected wheat candidates
- Homologous-group summary reports 1565 groups from 1919 grouped precursors, as in the wheat case study
- Each precursor is assigned to exactly one subgenome; no precursor appears in A, B, and D simultaneously unless coordinates say so
- Written summary (homologous_groups_summary.tsv) contains total groups, grouped precursors, triad count 118 and triad fraction 7.54%

## Limitations

- Reads or features that do not map to the reference genome are ignored; in the current version, iwa-miRNA ignores sequencing reads that do not map to the reference genome
- Precursors without clear genomic coordinates cannot be assigned to a subgenome (91 of the 2030 wheat candidates were excluded from the 1919 clearly located set)
- Coordinate parsing provides no downstream enrichment analysis (e.g., GO annotation or miRNA set enrichment) or cross-species comparative analysis

## Evidence

- [other] Assign each selected precursor to its homeologous subgenome (A, B, or D) using the genomic coordinates in the GFF3/GTF annotation.: [evidence span withheld — anchor status: not_found]
- [other] Of the wheat miRNA precursors clearly located on the three subgenomes, 1919 (A: 611; B: 744; D: 571) formed 1565 homologous groups: [evidence span withheld — anchor status: not_found]
- [methods] genomePrepare Name of species or genome sequences and annotation Path of formatted genome sequences and annotation: [evidence span withheld — anchor status: not_found]
- [intro] We provide examples of the application of iwa-miRNA for miRNA annotation of A. thaliana, maize (Zea mays L.), and hexaploid bread wheat (Triticum aestivum L.).: "We provide examples of the application of iwa-miRNA for miRNA annotation of A. thaliana, maize (Zea mays L.), and hexaploid bread wheat (Triticum aestivum L.)."
- [discussion] It cannot handle sRNA-Seq data from species without genome sequences.: "It cannot handle sRNA-Seq data from species without genome sequences."
