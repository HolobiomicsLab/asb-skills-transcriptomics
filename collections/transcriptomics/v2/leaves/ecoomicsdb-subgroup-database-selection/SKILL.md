---
name: ecoomicsdb-subgroup-database-selection
description: Use when quantifying raw RNA-seq reads from a non-model eukaryotic species
  into KO-level counts and you must pick which of EcoOmicsDB's 29 NCBI-taxonomy-based
  taxonomic sub-group databases to align against.
when_to_use_negative:
- Input is already a gene/KO count table or reference-transcriptome-backed counts
  — annotation and database selection steps are unnecessary
- The species has a well-annotated reference genome/transcriptome and genome-level
  IDs (e.g., vitellogenin) outside the KO system are required — KO-based databases
  lack such features
- The sample's taxonomy is far outside all available sub-groups (e.g., protist reads
  vs. the 'fishes' database) — an improper sub-group will miss key functional groups
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: Seq2Fun 2.0
  role: Aligns quality-controlled, translated reads to the selected EcoOmicsDB sub-group
    ortholog database to produce KO-level counts
- name: ExpressAnalyst
  role: Web platform hosting the Seq2Fun 2.0 module and EcoOmicsDB for processing
    and downstream analysis of non-model RNA-seq
  repo: github.com/xia-lab/ExpressAnalystR
- name: EcoOmicsDB
  role: Ortholog database organized into 29 NCBI-taxonomy-based sub-groups; the sub-group
    choice determines mapping targets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:1e69d082c027baca59300b7f7414991515b59c03d72e4abb0d44501f7c8a7610
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EcoOmicsDB
  - ExpressAnalyst
  - NCBI SRA SRR7499348–SRR7499365
  - Seq2Fun 2.0
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# ecoomicsdb-subgroup-database-selection

## Summary

Selects the appropriate EcoOmicsDB taxonomic sub-group protein ortholog database (e.g., 'Vertebrates' for salamander) so Seq2Fun 2.0 can map translated RNA-seq reads to KEGG ortholog (KO) identifiers for species without a reference transcriptome.

## When to use

Use when quantifying raw RNA-seq reads from a non-model eukaryotic species into KO-level counts and you must pick which of EcoOmicsDB's 29 NCBI-taxonomy-based taxonomic sub-group databases to align against.

## When NOT to use

- Input is already a gene/KO count table or reference-transcriptome-backed counts — annotation and database selection steps are unnecessary
- The species has a well-annotated reference genome/transcriptome and genome-level IDs (e.g., vitellogenin) outside the KO system are required — KO-based databases lack such features
- The sample's taxonomy is far outside all available sub-groups (e.g., protist reads vs. the 'fishes' database) — an improper sub-group will miss key functional groups

## Inputs

- Paired-end FASTQ files from NCBI SRA (e.g., SRR7499348–SRR7499365 salamander accessions)
- EcoOmicsDB taxonomic sub-group database selection (e.g., 'Vertebrates')
- Seq2Fun 2.0 parameter settings (max mismatches = 2, min matching length = 19, min BLOSUM62 score = 80)

## Outputs

- Per-sample KO-level read count table (samples x KEGG ortholog identifiers with integer counts, e.g., salamander_ortholog_counts.tsv)
- Seq2Fun per-sample mapping/alignment statistics

## How to apply

Match the organism's NCBI taxonomy to one of EcoOmicsDB's 29 taxonomic sub-group databases before running Seq2Fun 2.0; e.g., salamander reads (SRR7499348–SRR7499365) are mapped to the 'Vertebrates' sub-group. Because Seq2Fun aligns individual six-frame-translated reads directly to a high-resolution protein ortholog database, the sub-group choice replaces both transcript assembly and annotation transfer. Calibrate the translated-search filters to the database: maximum number of mismatches = 2, minimum matching length = 19, minimum matching BLOSUM62 score = 80. If cross-species mapping behavior is a concern, a customized database can be built with target sequences removed (as done for the 'Fishes' database with zebrafish removed). Validate the choice by confirming the output features are KEGG ortholog (KO) identifiers rather than in-house transcript IDs.

## Related tools

- **Seq2Fun 2.0** (Aligns quality-controlled, translated reads to the selected EcoOmicsDB sub-group ortholog database to produce KO-level counts)
- **ExpressAnalyst** (Web platform hosting the Seq2Fun 2.0 module and EcoOmicsDB for processing and downstream analysis of non-model RNA-seq) — github.com/xia-lab/ExpressAnalystR
- **EcoOmicsDB** (Ortholog database organized into 29 NCBI-taxonomy-based sub-groups; the sub-group choice determines mapping targets)

## Examples

```
fasterq-dump SRR7499348 --split-files && seq2fun --reads1 SRR7499348_1.fastq --reads2 SRR7499348_2.fastq --database Vertebrates
```

## Evaluation signals

- Output feature identifiers are KEGG ortholog (KO) IDs, not in-house transcript or genome IDs
- All 17 retained salamander samples (SRR7499348–SRR7499365 minus outlier SRR7499350) are present with integer, non-negative counts
- Seq2Fun parameters match the calibrated values: maximum number of mismatches = 2, minimum matching length = 19, minimum matching BLOSUM62 score = 80
- The chosen sub-group database is consistent with the organism's NCBI taxonomy (salamander → 'Vertebrates')

## Limitations

- Reads from the same gene may map to similar but different orthologs across species, requiring care in interpretation for taxonomically diverse comparisons
- If the protein database sub-group is not selected properly (e.g., protist reads mapped to the 'fishes' database), key functional groups may be missed
- KO-based annotation has limited transcriptome coverage — only ~76% of human and ~61% of zebrafish protein-coding genes carry KOs, and some biomarkers (e.g., vitellogenin) are absent from the KO system

## Evidence

- [abstract] All modules are integrated with EcoOmicsDB, an ortholog database that enables comprehensive analysis for species without a reference transcriptome.: "All modules are integrated with EcoOmicsDB, an ortholog database that enables comprehensive analysis for species without a reference transcriptome."
- [discussion] Improper sub-group database selection may miss key functional groups: "if the protein database is not selected properly (e.g., reads from a protist sample are mapped to the “fishes” database), key functional groups may be missed"
- [discussion] Seq2Fun parameters: mismatches 2, length 19, BLOSUM62 80: "“maximum number of mismatches” = 2, “minimum matching length” = 19, and “minimum matching BLOSUM62 score” = 80."
- [discussion] EcoOmicsDB has 29 taxonomic sub-group databases based on NCBI taxonomy: "EcoOmicsDB currently consists of 29 taxonomic sub-group databases based on the NCBI taxonomy system"
- [intro] ExpressAnalyst replaces assembly and annotation with direct translated-read alignment to the ortholog database: "the typical steps of transcript assembly and annotation are replaced with the alignment of individual translated reads directly to a comprehensive, high-resolution protein ortholog database"
