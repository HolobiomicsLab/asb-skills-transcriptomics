---
name: translated-read-to-ortholog-mapping
description: Use when raw RNA-seq FASTQ reads from any eukaryotic (especially non-model)
  species must be quantified and functionally annotated without a reference transcriptome,
  by mapping translated reads directly to KO orthologs.
when_to_use_negative:
- 'The organism has a well-annotated reference transcriptome and the analysis requires
  full gene/transcript-level IDs (KO coverage is limited: only 76.16% of human and
  61.40% of zebrafish protein-coding genes carry KOs; e.g., vitellogenin is absent
  from the KO system)'
- A count table or gene list already exists — downstream analysis should use ExpressAnalystR's
  matrix/gene-list modules instead of re-quantifying reads
- The protein sub-group database is mismatched to the sample (e.g., mapping protist
  reads against the 'Fishes' database), which can miss key functional groups
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0128
tools:
- name: Seq2Fun 2.0.2
  role: 'Performs the translated-read-to-KO mapping: read QC, paired-end joining,
    six-frame translation, peptide filtering, and seed-and-extend search against the
    KO database'
- name: ExpressAnalyst
  role: Web-based platform hosting Seq2Fun 2.0 and downstream normalization, PCA QC,
    differential expression, and functional analysis of the resulting count table
  repo: github.com/xia-lab/ExpressAnalystR
- name: ExpressAnalystR
  role: R package synchronized with the web server for annotation, normalization,
    and differential/meta-analysis of the KO count table produced by mapping
  repo: github.com/xia-lab/ExpressAnalystR
- name: EcoOmicsDB
  role: Ortholog database of protein-coding genes from 687 organisms (29 taxonomic
    sub-groups) against which translated reads are searched
- name: Kallisto 0.46.1
  role: Reference-transcriptome-based quantifier used as the benchmark comparison
    (minimum quality score parameter = 25) to validate Seq2Fun equivalence
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:6963f37b21024babddc833cdbe4cc5e032bda953fcff807c5e436bada388d999
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EcoOmicsDB
  - ExpressAnalyst
  - ExpressAnalystR
  - Kallisto 0.46.1
  - Seq2Fun
  - Seq2Fun 2.0
  - Seq2Fun 2.0.2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Translated-read-to-ortholog-mapping

## Summary

Directly maps raw RNA-seq FASTQ reads to protein ortholog (KEGG KO) databases by translating each cleaned read in all reading frames and performing a BLOSUM62-scored peptide search, replacing transcript assembly and annotation in non-model eukaryotes (implemented as Seq2Fun/ExpressAnalyst).

## When to use

Use when raw RNA-seq FASTQ reads from any eukaryotic (especially non-model) species must be quantified and functionally annotated without a reference transcriptome, by mapping translated reads directly to KO orthologs.

## When NOT to use

- The organism has a well-annotated reference transcriptome and the analysis requires full gene/transcript-level IDs (KO coverage is limited: only 76.16% of human and 61.40% of zebrafish protein-coding genes carry KOs; e.g., vitellogenin is absent from the KO system)
- A count table or gene list already exists — downstream analysis should use ExpressAnalystR's matrix/gene-list modules instead of re-quantifying reads
- The protein sub-group database is mismatched to the sample (e.g., mapping protist reads against the 'Fishes' database), which can miss key functional groups

## Inputs

- Raw RNA-seq FASTQ files (single-end or paired-end) from any eukaryotic species
- Selected taxonomic sub-group ortholog database from EcoOmicsDB (KEGG KO-based, e.g., 'Fishes')
- Seq2Fun parameters: maximum number of mismatches (2), minimum matching length (19), minimum matching BLOSUM62 score (80)

## Outputs

- KO (KEGG ortholog)-level read count table quantified from raw reads
- Normalized expression matrix suitable for PCA QC, limma differential expression, and pathway enrichment in ExpressAnalyst

## How to apply

Run Seq2Fun (v2.0.2 in ExpressAnalyst) on the raw FASTQ files: first perform read quality control and join paired-end reads; then translate each cleaned read into dozens of peptide sequences using all possible reading frames. Filter the translated sequences by peptide length (the 'true' translation yields the longest amino acid sequence) and BLOSUM62 score to reduce the search space, then seed the search with seven amino acids and extend potential matches backwards from the C terminal to the N terminal. Set 'maximum number of mismatches' = 2, 'minimum matching length' = 19, and 'minimum matching BLOSUM62 score' = 80; the search stops when a potential match meets the minimum matching score and maximum substitution thresholds. Output is a KO-level count table that can be normalized and fed into differential expression (limma) and pathway analysis in ExpressAnalyst/ExpressAnalystR, using the appropriate taxonomic sub-group of EcoOmicsDB (e.g., 'Fishes' for zebrafish/lobster).

## Related tools

- **Seq2Fun 2.0.2** (Performs the translated-read-to-KO mapping: read QC, paired-end joining, six-frame translation, peptide filtering, and seed-and-extend search against the KO database)
- **ExpressAnalyst** (Web-based platform hosting Seq2Fun 2.0 and downstream normalization, PCA QC, differential expression, and functional analysis of the resulting count table) — github.com/xia-lab/ExpressAnalystR
- **ExpressAnalystR** (R package synchronized with the web server for annotation, normalization, and differential/meta-analysis of the KO count table produced by mapping) — github.com/xia-lab/ExpressAnalystR
- **EcoOmicsDB** (Ortholog database of protein-coding genes from 687 organisms (29 taxonomic sub-groups) against which translated reads are searched)
- **Kallisto 0.46.1** (Reference-transcriptome-based quantifier used as the benchmark comparison (minimum quality score parameter = 25) to validate Seq2Fun equivalence)

## Evaluation signals

- For organisms with reference transcriptomes (e.g., zebrafish GRCz11/GCA_000002035.4, American lobster GMGI_Hamer_2.0/GCA_018991925.1), Seq2Fun should reproduce Kallisto results with almost identical PCA variance structures and relative numbers of DEGs (FDR < 0.05, |log2FC| > 1.5) per treatment contrast
- Running Seq2Fun 2.0.2 with max mismatches = 2, min matching length = 19, min BLOSUM62 score = 80 matches the equivalence benchmark of Supplementary Figs. 2-3 and Tables 1-2
- Normalization and QC should apply variance filter 15 and abundance filter 4 before PCA and DE analysis
- Memory footprint of Seq2Fun 2.0 should be ~0.94 GB (down from 1.49 GB in v1.0), and reads should map to KO-level profiles consistent with traditional workflows
- Removing the query species from the sub-group database (e.g., zebrafish sequences removed from 'Fishes') should still yield consistent functional profiles for the same samples, demonstrating robust cross-species mapping

## Limitations

- Reads from the same gene may map to similar but different orthologs across species, so care must be taken when interpreting results from taxonomically diverse comparisons
- Improper sub-group database selection (e.g., protist reads mapped to the 'Fishes' database) may miss key functional groups
- KO-based coverage is incomplete (76.16% of human and 61.40% of zebrafish protein-coding genes have KOs; vitellogenin is absent), so gene-level biomarkers outside the KO system cannot be captured
- Performance does not improve beyond 16 threads due to an I/O constraint; a high-speed drive can relieve this

## Evidence

- [discussion] each cleaned read is translated into dozens of peptide sequences using all possible reading frames.: "each cleaned read is translated into dozens of peptide sequences using all possible reading frames."
- [discussion] Seq2Fun employs two filters to greatly reduce the search space: peptide length and BLOSUM62 score: "Seq2Fun employs two filters to greatly reduce the search space: peptide length and BLOSUM62 score"
- [discussion] "maximum number of mismatches" = 2, "minimum matching length" = 19, and "minimum matching BLOSUM62 score" = 80.: [evidence span withheld — anchor status: not_found]
- [intro] the typical steps of transcript assembly and annotation are replaced with the alignment of individual translated reads directly to a comprehensive, high-resolution protein ortholog database: "the typical steps of transcript assembly and annotation are replaced with the alignment of individual translated reads directly to a comprehensive, high-resolution protein ortholog database"
- [discussion] quantification based on Kallisto (version 0.46.1) or Seq2Fun (version 2.0.2): "quantification based on Kallisto (version 0.46.1) or Seq2Fun (version 2.0.2)"
