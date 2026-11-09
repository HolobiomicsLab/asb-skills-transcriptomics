---
name: non-model-species-rnaseq-functional-profiling
description: Use when starting from raw paired-end FASTQ reads of a non-model eukaryotic
  species (no reference transcriptome needed) and the goal is a KO-level ortholog
  count matrix for downstream statistical analysis.
when_to_use_negative:
- Input is already a count table or gene expression matrix — use the ExpressAnalyst/ExpressAnalystR
  statistical, normalization, and enrichment modules instead of re-quantifying reads.
- The analysis targets specific protein-coding genes absent from the KO system (e.g.,
  vitellogenin is not found in the KO system) or requires full transcriptome-level
  coverage; KO-based annotation covers only ~76% of human and ~61% of zebrafish protein-coding
  genes.
- The sample's taxonomy is distant from the selected database sub-group (e.g., mapping
  protist reads to the 'fishes' database) — key functional groups may be missed.
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0080
tools:
- name: Seq2Fun 2.0
  role: Ultra-fast read mapping of translated RNA-seq reads to KEGG ortholog (KO)
    entries with quality control, paired-end joining, six-frame translation, and peptide/BLOSUM62
    filtering
- name: ExpressAnalyst
  role: Web platform implementing the Seq2Fun module and integrated downstream statistical
    and functional analysis of the resulting KO count tables
  repo: https://github.com/xia-lab/ExpressAnalystR
- name: EcoOmicsDB
  role: High-resolution protein ortholog database (29 taxonomic sub-groups built from
    KEGG protein-coding genes of 687 organisms) used as the mapping target
- name: ExpressAnalystR
  role: R package synchronized with the ExpressAnalyst web server for local statistical,
    enrichment, and meta-analysis of the KO count tables
  repo: https://github.com/xia-lab/ExpressAnalystR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:342fbc55cf2a7d6d16e00dcc3c8ee7a42ccd82b4e5a1d181ee842a677f369e23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/ExpressAnalystR
  tools:
  - EcoOmicsDB
  - ExpressAnalyst
  - ExpressAnalystR
  - NCBI SRA SRR7499348–SRR7499365
  - Seq2Fun 2.0
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# non-model-species-rnaseq-functional-profiling

## Summary

Quantify raw RNA-seq reads from non-model species directly into KEGG ortholog (KO) count tables by mapping translated reads against an EcoOmicsDB taxonomic sub-group database with Seq2Fun 2.0 via ExpressAnalyst, bypassing transcript assembly and annotation. This yields functionally coherent, reproducible gene-level features for downstream differential expression and pathway analysis.

## When to use

Use when starting from raw paired-end FASTQ reads of a non-model eukaryotic species (no reference transcriptome needed) and the goal is a KO-level ortholog count matrix for downstream statistical analysis.

## When NOT to use

- Input is already a count table or gene expression matrix — use the ExpressAnalyst/ExpressAnalystR statistical, normalization, and enrichment modules instead of re-quantifying reads.
- The analysis targets specific protein-coding genes absent from the KO system (e.g., vitellogenin is not found in the KO system) or requires full transcriptome-level coverage; KO-based annotation covers only ~76% of human and ~61% of zebrafish protein-coding genes.
- The sample's taxonomy is distant from the selected database sub-group (e.g., mapping protist reads to the 'fishes' database) — key functional groups may be missed.

## Inputs

- Raw paired-end FASTQ files from NCBI SRA (e.g., accessions SRR7499348–SRR7499365)
- EcoOmicsDB taxonomic sub-group protein ortholog database (e.g., the 'Vertebrates' sub-group)
- Seq2Fun 2.0 (command-line or ExpressAnalyst web implementation)

## Outputs

- Per-sample mapping/quantification summaries from Seq2Fun
- A samples x KO (KEGG ortholog) identifier matrix of integer non-negative read counts (e.g., salamander_ortholog_counts.tsv)

## How to apply

Download the raw paired-end FASTQ files from NCBI SRA (e.g., accessions SRR7499348–SRR7499365 using the SRA Toolkit fasterq-dump/prefetch), excluding any sample flagged as an outlier by prior QA/QC (in the salamander case, SRR7499350). Run Seq2Fun 2.0 against the appropriate EcoOmicsDB taxonomic sub-group database (e.g., 'Vertebrates', one of 29 sub-groups based on NCBI taxonomy) using the calibrated parameters: maximum number of mismatches = 2, minimum matching length = 19, and minimum matching BLOSUM62 score = 80. Seq2Fun internally performs read quality control, joins paired-end reads, translates each cleaned read into peptides across all six reading frames, and filters translated sequences by peptide length and BLOSUM62 score before the seed-based ortholog search. Summarize the per-sample mapping results into a matrix of samples (rows) x Seq2Fun ortholog (KO) identifiers (columns) with integer read counts. Rationale: aligning individual translated reads to a high-resolution protein ortholog database replaces computationally intensive transcript assembly and BLAST-based annotation transfer, and Seq2Fun was shown to outperform de novo assembly in accuracy, precision, computing time, and memory usage.

## Related tools

- **Seq2Fun 2.0** (Ultra-fast read mapping of translated RNA-seq reads to KEGG ortholog (KO) entries with quality control, paired-end joining, six-frame translation, and peptide/BLOSUM62 filtering)
- **ExpressAnalyst** (Web platform implementing the Seq2Fun module and integrated downstream statistical and functional analysis of the resulting KO count tables) — https://github.com/xia-lab/ExpressAnalystR
- **EcoOmicsDB** (High-resolution protein ortholog database (29 taxonomic sub-groups built from KEGG protein-coding genes of 687 organisms) used as the mapping target)
- **ExpressAnalystR** (R package synchronized with the ExpressAnalyst web server for local statistical, enrichment, and meta-analysis of the KO count tables) — https://github.com/xia-lab/ExpressAnalystR

## Evaluation signals

- Validate the output table contains exactly 17 sample entries for the salamander case (all accessions SRR7499348–SRR7499365 minus excluded outlier SRR7499350).
- Confirm all counts are integer, non-negative values and the row/column features are KEGG ortholog (KO) identifiers.
- Confirm the Seq2Fun parameters actually used match the calibrated values: maximum number of mismatches = 2, minimum matching length = 19, minimum matching BLOSUM62 score = 80.
- Confirm the database used is the correct EcoOmicsDB taxonomic sub-group ('Vertebrates' for the salamander case study) matching the sample's taxonomy.
- Per-sample mapping results in the final matrix should be reproducible from Seq2Fun's individual mapping/quantification output files.

## Limitations

- KO-based annotation has limited transcriptome coverage: 14,964 of 19,648 human protein-coding genes (76.16%) and 16,322 of 26,584 zebrafish genes (61.40%) are annotated with KOs, and some biomarkers such as vitellogenin are not in the KO system.
- Improper sub-group database selection may miss key functional groups (e.g., reads from a protist sample mapped to the 'fishes' database).
- Reads from the same gene may map to similar but different orthologs for different species, requiring care when interpreting taxonomically diverse comparisons.
- Seq2Fun performance does not improve beyond 16 threads due to an I/O constraint; a high-speed drive could relieve it.

## Evidence

- [discussion] Calibrated Seq2Fun parameters: "“maximum number of mismatches” = 2, “minimum matching length” = 19, and “minimum matching BLOSUM62 score” = 80."
- [discussion] Outlier sample exclusion: "excluding sample SRR7499350 which was identified as an outlier by QA/QC performed by the original publication"
- [discussion] Seq2Fun internal first step: "The first step in the Seq2Fun algorithm is to conduct reads quality control and join paired-end reads."
- [intro] Superiority over de novo assembly: "Seq2Fun outperforms de novo assembly in terms of accuracy, precision, computing time, and memory usage, while producing functional profiles that are consistent with traditional"
- [discussion] EcoOmicsDB structure: "EcoOmicsDB currently consists of 29 taxonomic sub-group databases based on the NCBI taxonomy system"
- [readme] ExpressAnalystR scope: "ExpressAnalystR is the underlying R package synchronized with ExpressAnalyst web server. It is designed for statistical analysis, enrichment analysis and visual analytics"
