---
name: read-quality-control-and-paired-end-joining
description: Use when raw paired-end RNA-seq FASTQ reads from a non-model eukaryotic
  species must be converted into per-sample ortholog (KO) count tables, replacing
  transcript assembly and annotation.
when_to_use_negative:
- The input is already a gene expression/ortholog count matrix — use the ExpressAnalyst/ExpressAnalystR
  statistical modules (annotation, normalization, DE) instead of re-running read processing
- The analysis requires gene-level identifiers beyond KOs (e.g. biomarkers like vitellogenin
  that are absent from the KO system) or a reference transcriptome is available and
  gene-level resolution is essential
- A sub-group database matching the organism is unavailable (e.g. protist reads mapped
  to a 'fishes' database) — key functional groups may be missed
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seq2Fun 2.0
  role: Performs reads quality control, paired-end joining, six-frame translation,
    peptide length/BLOSUM62 filtering, and KO mapping of the FASTQ reads
- name: ExpressAnalyst
  role: Web-based platform wrapping Seq2Fun 2.0 and integrating the EcoOmicsDB ortholog
    databases used for read mapping
  repo: github.com/xia-lab/ExpressAnalystR
- name: EcoOmicsDB
  role: Ortholog database of 29 taxonomic sub-groups (based on NCBI taxonomy) against
    which translated reads are searched
- name: ExpressAnalystR
  role: R package for downstream statistical analysis of the resulting count tables;
    run all functions in sequence as in the downloaded R command history
  repo: https://github.com/xia-lab/ExpressAnalystR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:fb84b7a6e54e25231e46d23abcb04f3117e379b731b5b36e61b170dc9bb962bb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EcoOmicsDB
  - ExpressAnalyst
  - ExpressAnalystR
  - NCBI SRA SRR7499348–SRR7499365
  - Seq2Fun 2.0
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Read quality control and paired-end joining

## Summary

Internal pre-processing stage of the Seq2Fun 2.0 algorithm (run via ExpressAnalyst against an EcoOmicsDB sub-group database) that performs read quality control, joins paired-end RNA-seq FASTQ reads, and six-frame translates the cleaned reads into peptide sequences for ortholog mapping.

## When to use

Use when raw paired-end RNA-seq FASTQ reads from a non-model eukaryotic species must be converted into per-sample ortholog (KO) count tables, replacing transcript assembly and annotation.

## When NOT to use

- The input is already a gene expression/ortholog count matrix — use the ExpressAnalyst/ExpressAnalystR statistical modules (annotation, normalization, DE) instead of re-running read processing
- The analysis requires gene-level identifiers beyond KOs (e.g. biomarkers like vitellogenin that are absent from the KO system) or a reference transcriptome is available and gene-level resolution is essential
- A sub-group database matching the organism is unavailable (e.g. protist reads mapped to a 'fishes' database) — key functional groups may be missed

## Inputs

- Raw paired-end RNA-seq FASTQ files (e.g. NCBI SRA accessions SRR7499348–SRR7499365, with SRR7499350 excluded)
- EcoOmicsDB ortholog sub-group database appropriate to the organism (e.g. 'Vertebrates')

## Outputs

- Per-sample mapping results summarized as a samples x KO-identifier matrix of integer read counts (e.g. salamander_ortholog_counts.tsv)
- KEGG ortholog (KO) identifiers as features for downstream differential expression and enrichment analysis

## How to apply

Run each paired-end FASTQ pair through Seq2Fun 2.0 (within ExpressAnalyst, e.g. the EcoOmicsDB 'Vertebrates' taxonomic sub-group database for the salamander samples SRR7499348–SRR7499365, skipping the outlier SRR7499350). Internally, Seq2Fun first performs read quality control and joins the paired-end reads; each cleaned read is then translated into dozens of peptide sequences using all possible reading frames. Translated frames are filtered on peptide length (the 'true' translation produces the longest amino acid sequence, ties resolved by BLOSUM62 score) and on BLOSUM62 score to reduce the search space. Mapping uses a seed-based search of seven amino acids with backwards extension from the C terminal to the N terminal, governed by calibrated parameters: maximum number of mismatches = 2, minimum matching length = 19, and minimum matching BLOSUM62 score = 80. The per-sample mapping results are then summarized into a matrix of samples x Seq2Fun ortholog (KO) identifiers with integer read counts (e.g. salamander_ortholog_counts.tsv).

## Related tools

- **Seq2Fun 2.0** (Performs reads quality control, paired-end joining, six-frame translation, peptide length/BLOSUM62 filtering, and KO mapping of the FASTQ reads)
- **ExpressAnalyst** (Web-based platform wrapping Seq2Fun 2.0 and integrating the EcoOmicsDB ortholog databases used for read mapping) — github.com/xia-lab/ExpressAnalystR
- **EcoOmicsDB** (Ortholog database of 29 taxonomic sub-groups (based on NCBI taxonomy) against which translated reads are searched)
- **ExpressAnalystR** (R package for downstream statistical analysis of the resulting count tables; run all functions in sequence as in the downloaded R command history) — https://github.com/xia-lab/ExpressAnalystR

## Evaluation signals

- Confirm the output count table contains 17 sample entries (accessions SRR7499348–SRR7499365 minus the excluded outlier SRR7499350)
- Verify all counts are integers and non-negative, with KEGG ortholog (KO) identifiers as the feature column
- Check that the calibrated Seq2Fun parameters were applied: maximum number of mismatches = 2, minimum matching length = 19, minimum matching BLOSUM62 score = 80
- Confirm the correct EcoOmicsDB taxonomic sub-group database ('Vertebrates' for salamander) was used, since improper sub-group selection can miss key functional groups
- Spot-check per-sample read counts against Seq2Fun's per-sample mapping summaries for consistency before building the matrix

## Limitations

- Reads from the same gene may map to similar but different orthologs in different species, so care must be taken when interpreting taxonomically diverse comparisons
- Improper sub-group database selection may miss key functional groups
- Performance does not improve beyond 16 threads due to an I/O constraint; a high-speed drive could relieve it
- KO-based annotation has limited transcriptome coverage (e.g. only 76.16% of human protein-coding genes carry KOs), so some genes are unobservable in the output

## Evidence

- [discussion] Seq2Fun first step: "The first step in the Seq2Fun algorithm is to conduct reads quality control and join paired-end reads."
- [discussion] six-frame translation: "each cleaned read is translated into dozens of peptide sequences using all possible reading frames."
- [discussion] search-space filters: "Seq2Fun employs two filters to greatly reduce the search space: peptide length and BLOSUM62 score"
- [readme] execution order requirement: "Every function must be executed in sequence as it is shown on the R Command history, please do not skip any commands as this can result in errors downstream."
