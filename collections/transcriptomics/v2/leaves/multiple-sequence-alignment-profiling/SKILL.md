---
name: multiple-sequence-alignment-profiling
description: 'Use when constructing a curated database of potentially functional HERVs:
  genomic loci must be translated, screened against retroviral protein motifs (POL/RT,
  GAG, ENV), and filtered to retain only sequences with detectable homology.'
when_to_use_negative:
- When the target is exogenous (non-human) viral identification from RNAseq reads
  — that is handled by the viRNAtrap deep learning pipeline, not HERV database curation.
- When HERV sequences have already been annotated as functional/intact (e.g., via
  an existing curated database) and only expression quantification is needed.
- When loci lack any detectable POL/RT, GAG, or ENV motif after blastx/MAFFT-profiling
  — these fail the filter and should not be retained in the database.
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: MAFFT
  role: Aligns candidate POL (reverse transcriptase) protein sequences to build the
    POL profile used to detect POL homology in HERV candidate sequences
- name: blastx
  role: Screens translated HERV genomic sequences against the NCBI NR protein database
    (E-value <= 1e-4) to detect GAG, ENV, or POL/RT retroviral protein motifs
- name: twoBitToFa
  role: Extracts genomic FASTA sequences spanning HERVd-annotated endogenous retrovirus
    loci from the hg19 human reference genome
- name: viRNAtrap
  role: 'Downstream consumer: its assembled contigs are aligned against the curated
    HERV database to identify HERV expression patterns in tumors'
  repo: https://github.com/AuslanderLab/virnatrap.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:8b17c19f26944707ac8161076414eec2149b1bf9b35afb8e9bf3ce90fbe7c856
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - MAFFT
  - blastx
  - twoBitToFa
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# multiple-sequence-alignment-profiling

## Summary

Builds a POL protein profile by aligning candidate reverse transcriptase sequences with MAFFT, then screening HERV candidate sequences against it (alongside blastx hits) to retain HERVs with retroviral POL/RT, GAG, or ENV motifs for a curated, potentially-functional HERV database.

## When to use

Use when constructing a curated database of potentially functional HERVs: genomic loci must be translated, screened against retroviral protein motifs (POL/RT, GAG, ENV), and filtered to retain only sequences with detectable homology.

## When NOT to use

- When the target is exogenous (non-human) viral identification from RNAseq reads — that is handled by the viRNAtrap deep learning pipeline, not HERV database curation.
- When HERV sequences have already been annotated as functional/intact (e.g., via an existing curated database) and only expression quantification is needed.
- When loci lack any detectable POL/RT, GAG, or ENV motif after blastx/MAFFT-profiling — these fail the filter and should not be retained in the database.

## Inputs

- hg19 human reference genome (2bit) with HERVd-annotated endogenous retrovirus loci
- Candidate HERV genomic sequences spanning HERVd-annotated loci
- Candidate POL (reverse transcriptase) protein sequences
- NCBI NR protein database for blastx screening

## Outputs

- herv_database.fasta — curated database of 3044 potentially functional HERVs, each with ≥1 supporting POL/RT, GAG, or ENV retroviral protein motif hit
- MAFFT multiple sequence alignment of candidate POL protein sequences (POL profile)

## How to apply

Extract genomic sequences spanning HERVd-annotated endogenous retrovirus loci from the hg19 reference genome with UCSC twoBitToFa. Translate the extracted sequences and screen them against the NCBI NR protein database with blastx at an E-value threshold of 1e-4 to flag candidate retroviral protein homology. Align the candidate POL (reverse transcriptase) protein sequences with MAFFT to build a POL profile, and search the HERV candidate sequences against this profile to detect POL homology. Retain only candidates showing at least one detectable retroviral motif from POL/RT, GAG, or ENV, and write the retained intact-HERV sequences to herv_database.fasta; this curated database (3044 HERVs) then serves as the alignment target for viRNAtrap-assembled contigs to characterize HERV expression across the 14 TCGA cancer types.

## Related tools

- **MAFFT** (Aligns candidate POL (reverse transcriptase) protein sequences to build the POL profile used to detect POL homology in HERV candidate sequences)
- **blastx** (Screens translated HERV genomic sequences against the NCBI NR protein database (E-value <= 1e-4) to detect GAG, ENV, or POL/RT retroviral protein motifs)
- **twoBitToFa** (Extracts genomic FASTA sequences spanning HERVd-annotated endogenous retrovirus loci from the hg19 human reference genome)
- **viRNAtrap** (Downstream consumer: its assembled contigs are aligned against the curated HERV database to identify HERV expression patterns in tumors) — https://github.com/AuslanderLab/virnatrap.git

## Examples

```
blastx -query herv_candidates_translated.fa -db nr -evalue 1e-4 -out herv_vs_nr_blastx.xml
```

## Evaluation signals

- Every sequence in herv_database.fasta has at least one blastx hit (E-value <= 1e-4) or MAFFT-POL-profile hit supporting a POL/RT, GAG, or ENV retroviral protein motif.
- The curated database contains 3044 HERVs filtered by POL/GAG/ENV motifs, matching the count reported for the article's HERV database.
- blastx screening runs against the NCBI NR protein database at exactly the E-value threshold 1e-4.
- All database source sequences originate from HERVd-annotated endogenous retrovirus loci extracted from the hg19 reference genome.
- Downstream alignment of viRNAtrap contigs against herv_database.fasta yields HERV expression calls consistent with the 14 cancer types analyzed.

## Limitations

- The filter requires at least one retroviral motif (POL/RT, GAG, or ENV); HERVs lacking all three motifs are excluded even if biologically relevant.
- The blastx E-value cutoff of 1e-4 may miss highly divergent HERV loci with weak homology to known retroviral proteins.
- Detection of POL homology depends on the quality of the MAFFT-built POL profile; false positives or negatives propagate into the curated database.

## Evidence

- [discussion] We curate a database of HERVs that comprise intact retroviral genes in the human genome and survey the expression of these viruses across different cancer tissues: "We curate a database of HERVs that comprise intact retroviral genes in the human genome and survey the expression of these viruses across different cancer tissues"
- [intro] we curate a database of potentially functional human endogenous retroviruses (HERVs) and analyze expression patterns of different HERVs across human cancers: "we curate a database of potentially functional human endogenous retroviruses (HERVs) and analyze expression patterns of different HERVs across human cancers to find that HERV expression is associated"
- [other] Translate the extracted HERV sequences and screen them against the NCBI NR protein database with blastx using an E-value threshold of 1e-4.: [evidence span withheld — anchor status: not_found]
- [discussion] the presence and classification of multiple viral reads was verified by targeted blastn- and blastx-based sequence analyses in different samples: "the presence and classification of multiple viral reads was verified by targeted blastn- and blastx-based sequence analyses in different samples"
