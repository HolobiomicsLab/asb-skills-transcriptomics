---
name: subgenome-homeologous-group-clustering
description: Use when annotating miRNA precursors in a polyploid genome with distinct
  A/B/D (or other) subgenomes and you need homeologous-group structure, subgenome
  counts, and complete-triad fractions from curated precursor sets.
when_to_use_negative:
- The species lacks reference genome sequences or distinct homeologous subgenomes,
  so GFF3/GTF coordinates cannot assign precursors to A/B/D subgenomes
- The input is raw sRNA-Seq reads rather than a curated, genome-anchored precursor
  list (iwa-miRNA ignores reads that do not map to the reference genome)
- The analysis goal is downstream enrichment (GO annotation, miRNA set enrichment)
  or cross-species comparative analysis, which the iwa-miRNA workflow does not support
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0610
tools:
- name: iwa-miRNA
  role: Provides the curated selected miRNA precursor list (table, GFF3/GTF, FASTA)
    via its Manual Curation module that is then clustered into subgenome homologous
    groups
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Web framework in which the iwa-miRNA Manual Curation outputs consumed by this
    clustering workflow are produced and exported
  repo: https://github.com/cma2015/iwa-miRNA
- name: Rmarkdown
  role: Used to summarize results into HTML pages for visualization and sharing of
    the homologous-group summary
  repo: https://github.com/cma2015/iwa-miRNA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:0b41b210aeda7d96fbd518f2c583f70d7dc0bd6d3dbe3cd4fe8feeac66a4e6a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - Rmarkdown
  - iwa-miRNA
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# subgenome-homeologous-group-clustering

## Summary

Clusters miRNA precursors from a polyploid plant (e.g., hexaploid bread wheat) into homologous groups across the A, B, and D subgenomes using genomic coordinates and sequence similarity, then quantifies per-subgenome distribution and complete A:B:D triad content.

## When to use

Use when annotating miRNA precursors in a polyploid genome with distinct A/B/D (or other) subgenomes and you need homeologous-group structure, subgenome counts, and complete-triad fractions from curated precursor sets.

## When NOT to use

- The species lacks reference genome sequences or distinct homeologous subgenomes, so GFF3/GTF coordinates cannot assign precursors to A/B/D subgenomes
- The input is raw sRNA-Seq reads rather than a curated, genome-anchored precursor list (iwa-miRNA ignores reads that do not map to the reference genome)
- The analysis goal is downstream enrichment (GO annotation, miRNA set enrichment) or cross-species comparative analysis, which the iwa-miRNA workflow does not support

## Inputs

- Selected wheat miRNA precursor table exported from the iwa-miRNA Manual Curation module
- Selected miRNA precursor coordinates in GFF3/GTF format (with chromosome/subgenome identifiers)
- Selected miRNA precursor sequences in FASTA format

## Outputs

- homologous_groups_summary.tsv: total homologous groups, grouped precursor count, complete-triad count and fraction (e.g., 1565 groups, 1919 grouped precursors, 118/1565 = 7.54% triads)
- Per-subgenome precursor counts (e.g., A: 611; B: 744; D: 571)
- Homologous group membership table with A:B:D configuration per group (eight configurations)

## How to apply

Load the selected miRNA precursor list exported from the iwa-miRNA Manual Curation module in three formats: table, GFF3/GTF, and FASTA. Assign each precursor to its homeologous subgenome (A, B, or D) using the genomic coordinates in the GFF3/GTF annotation, keeping only precursors clearly located on one of the three subgenomes. Cluster precursors into homologous groups by sequence similarity across the three subgenomes, then flag groups containing a complete A/B/D triad and compute per-subgenome precursor counts. In the published wheat case study this yielded 1919 grouped precursors (A: 611; B: 744; D: 571, with D the fewest) forming 1565 homologous groups across eight A:B:D configurations, of which 7.54% (118/1565) were complete triads. Write the group-level summary (total groups, grouped precursors, triad count and fraction, per-subgenome counts) to homologous_groups_summary.tsv and validate against the published values.

## Related tools

- **iwa-miRNA** (Provides the curated selected miRNA precursor list (table, GFF3/GTF, FASTA) via its Manual Curation module that is then clustered into subgenome homologous groups) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Web framework in which the iwa-miRNA Manual Curation outputs consumed by this clustering workflow are produced and exported) — https://github.com/cma2015/iwa-miRNA
- **Rmarkdown** (Used to summarize results into HTML pages for visualization and sharing of the homologous-group summary) — https://github.com/cma2015/iwa-miRNA

## Examples

```
write.table(homologous_groups_summary, "homologous_groups_summary.tsv", sep="\t", quote=FALSE, row.names=FALSE)
```

## Evaluation signals

- Summary reports 1565 homologous groups formed from 1919 grouped precursors (out of 2030 selected candidates)
- Complete-triad fraction equals 118/1565 = 7.54%
- Per-subgenome counts are A: 611, B: 744, D: 571, with D the fewest
- Every grouped precursor is assigned to exactly one of the A, B, or D subgenomes via GFF3/GTF coordinates; unassignable precursors are excluded from grouping
- Every homologous group maps to one of eight A:B:D configurations, and triad-flagged groups contain all three subgenome members

## Limitations

- iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences, so no genome-anchored coordinates exist for subgenome assignment
- Sequencing reads that do not map to the reference genome are ignored in the current version, potentially missing subgenome-specific precursors
- Only precursors 'clearly located on the three subgenomes' can be grouped; ambiguous or unanchored precursors are excluded from the triad analysis

## Evidence

- [intro] We provide examples of the application of iwa-miRNA for miRNA annotation of A. thaliana, maize (Zea mays L.), and hexaploid bread wheat (Triticum aestivum L.).: "We provide examples of the application of iwa-miRNA for miRNA annotation of A. thaliana, maize (Zea mays L.), and hexaploid bread wheat (Triticum aestivum L.)."
- [methods] The iwa-miRNA comprises three modules: MiRNA Compilation, MiRNA Selection, and Manual Curation (Figure 1; Table 1).: "The iwa-miRNA comprises three modules: MiRNA Compilation, MiRNA Selection, and Manual Curation (Figure 1; Table 1)."
- [results] This resulted in the identification of 667 miRNA precursor candidates, corresponding to 1190 mature miRNA candidates (Table S2): "This resulted in the identification of 667 miRNA precursor candidates, corresponding to 1190 mature miRNA candidates (Table S2)"
- [readme] Test datasets referred in the tutorials for iwa-miRNA: https://github.com/cma2015/iwa-miRNA/tree/master/Test_data: "Test datasets referred in the tutorials for iwa-miRNA: https://github.com/cma2015/iwa-miRNA/tree/master/Test_data"
