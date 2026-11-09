---
name: translated-read-alignment-to-kegg-ortholog-database
description: Use when raw RNA-seq FASTQ files come from a species without a reference
  transcriptome, and the goal is KO-level read quantification and functional profiling
  instead of de novo assembly plus annotation transfer.
when_to_use_negative:
- When the gene of interest is not in the KO system — KO-based annotation covers only
  76.16% of human and 61.40% of zebrafish protein-coding genes, and vitellogenin is
  absent from the KO system entirely
- When the EcoOmicsDB sub-group database does not match the sample's taxonomy (e.g.,
  reads from a protist sample mapped to the 'fishes' database) — key functional groups
  will be missed
- When a well-annotated reference transcriptome exists and conventional genome-based
  quantification (e.g., Kallisto with minimum quality score 25) is preferable
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seq2Fun 2.0
  role: Performs the translated read-to-KO alignment with reduced memory footprint
    (0.94 GB vs 1.49 GB in v1.0); version 2.0.2 was used in quantification comparisons
    against Kallisto 0.46.1
- name: Seq2Fun 1.0
  role: Original command-line algorithm for mapping RNA-seq reads to KO databases
    with translated search; baseline for the memory/throughput benchmark
- name: EcoOmicsDB
  role: Ortholog database (29 NCBI-taxonomy-based sub-group databases) that reads
    are aligned against; supports species without reference transcriptomes
- name: ExpressAnalyst
  role: Web platform that couples this read-mapping step to downstream statistical
    and functional analysis of the resulting KO count tables
  repo: github.com/xia-lab/ExpressAnalystR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:9ab1782dacdc42e79b7dd900352d3aa4d1222537450e620a849eafc5efd76d38
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EcoOmicsDB
  - ExpressAnalyst
  - Seq2Fun 1.0
  - Seq2Fun 2.0
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Translated read alignment to KEGG ortholog database

## Summary

Maps RNA-seq FASTQ reads from any eukaryotic species directly to a high-resolution KEGG ortholog (KO) protein database via translated search (Seq2Fun), replacing transcript assembly and BLAST-based annotation transfer. Produces KO-level count tables for non-model species without a reference transcriptome.

## When to use

Use when raw RNA-seq FASTQ files come from a species without a reference transcriptome, and the goal is KO-level read quantification and functional profiling instead of de novo assembly plus annotation transfer.

## When NOT to use

- When the gene of interest is not in the KO system — KO-based annotation covers only 76.16% of human and 61.40% of zebrafish protein-coding genes, and vitellogenin is absent from the KO system entirely
- When the EcoOmicsDB sub-group database does not match the sample's taxonomy (e.g., reads from a protist sample mapped to the 'fishes' database) — key functional groups will be missed
- When a well-annotated reference transcriptome exists and conventional genome-based quantification (e.g., Kallisto with minimum quality score 25) is preferable

## Inputs

- Raw paired-end RNA-seq FASTQ files (e.g., twelve zebrafish samples SRR13332314–SRR13332325 from NCBI SRA study SRP299836)
- EcoOmicsDB taxonomic sub-group ortholog database (e.g., zebrafish KO database or a customized 'Fishes' database)
- Run parameters: maximum number of mismatches = 2, minimum matching length = 19, minimum matching BLOSUM62 score = 80

## Outputs

- Per-sample KO-level read count table suitable for downstream DE and pathway enrichment
- Per-run wall-clock runtime and peak memory metrics for benchmarking
- Functional profiles consistent with genome-based pipelines

## How to apply

First run read quality control and join paired-end reads, then translate each cleaned read into dozens of peptide sequences using all possible reading frames. Reduce the search space with two filters — peptide length (retain the longest translated sequence, ties resolved by BLOSUM62 score) and BLOSUM62 score — then run a seed-based search starting from a seven-amino-acid seed with backwards extension from the C terminal to the N terminal, terminating when thresholds are met: 'maximum number of mismatches' = 2, 'minimum matching length' = 19, and 'minimum matching BLOSUM62 score' = 80. Match reads against the appropriate EcoOmicsDB taxonomic sub-group database (built from 12,828,537 protein-coding genes across 687 organisms from KEGG); choose the sub-group to match the sample's taxonomy. Quantify reads per KO to produce a count table that feeds downstream differential expression and pathway enrichment. Seq2Fun 2.0 cuts peak memory from 1.49 GB to 0.94 GB despite a database more than 125 times larger, sustaining ~2 million reads per minute.

## Related tools

- **Seq2Fun 2.0** (Performs the translated read-to-KO alignment with reduced memory footprint (0.94 GB vs 1.49 GB in v1.0); version 2.0.2 was used in quantification comparisons against Kallisto 0.46.1)
- **Seq2Fun 1.0** (Original command-line algorithm for mapping RNA-seq reads to KO databases with translated search; baseline for the memory/throughput benchmark)
- **EcoOmicsDB** (Ortholog database (29 NCBI-taxonomy-based sub-group databases) that reads are aligned against; supports species without reference transcriptomes)
- **ExpressAnalyst** (Web platform that couples this read-mapping step to downstream statistical and functional analysis of the resulting KO count tables) — github.com/xia-lab/ExpressAnalystR

## Evaluation signals

- Benchmark table contains wall-clock runtime and peak memory for all 12 samples under both versions, with v2.0 peak memory near 0.94 GB versus 1.49 GB for v1.0 and throughput near 2 million reads per minute
- Run completes with the paper's settings (mismatches = 2, minimum matching length = 19, minimum BLOSUM62 score = 80) and emits a KO count table for every FASTQ sample
- Functional profiles agree with a genome-based pipeline (zebrafish reads aligned to GRCz11 GCA_000002035.4; Kallisto 0.46.1 with minimum quality score 25 as alternative quantifier)
- Cross-species sanity check: zebrafish samples aligned to the customized 'Fishes' database with zebrafish sequences removed still yield interpretable cross-species functional profiles
- Scaling check: throughput should plateau beyond 16 threads due to the I/O constraint — larger thread counts are not expected to improve performance

## Limitations

- Reads from the same gene may map to similar but different orthologs across species, so care is needed when interpreting taxonomically diverse comparisons
- KO-based annotation has limited transcriptome coverage (e.g., vitellogenin, an egg yolk protein precursor and biomarker of interest, is not found in the KO system)
- Performance does not improve beyond 16 threads due to an I/O constraint; a high-speed drive could relieve it
- Improper EcoOmicsDB sub-group selection (e.g., protist reads mapped to the 'fishes' database) can cause key functional groups to be missed

## Evidence

- [intro] Seq2Fun for mapping RNA-seq reads from eukaryotic species to KEGG ortholog (KO) databases with a translated search: "We previously developed a command-line algorithm, Seq2Fun for mapping RNA-seq reads from eukaryotic species to KEGG ortholog (KO) databases with a translated search"
- [discussion] Two search-space filters: "Seq2Fun employs two filters to greatly reduce the search space: peptide length and BLOSUM62 score"
- [discussion] Search termination thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] Seq2Fun parameters: "“maximum number of mismatches” = 2, “minimum matching length” = 19, and “minimum matching BLOSUM62 score” = 80."
- [intro] Advantage over de novo assembly: "We demonstrated that Seq2Fun outperforms de novo assembly in terms of accuracy, precision, computing time, and memory usage, while producing functional profiles that are consistent with traditional"
