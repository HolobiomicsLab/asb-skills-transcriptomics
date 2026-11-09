---
name: profile-based-homology-search
description: Use when building a reference database of endogenous retroviral sequences
  that must each be backed by detectable retroviral protein motifs (POL/RT, GAG, ENV)
  before downstream alignment of assembled contigs.
when_to_use_negative:
- When the goal is de novo viral discovery from raw RNAseq reads — use the viRNAtrap
  deep-learning pipeline instead; this skill only builds the reference HERV database
- When candidate sequences are already curated as intact retroviruses and only need
  classification against known viral genomes via blastn
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: twoBitToFa
  role: Extracts genomic sequences spanning HERVd-annotated loci from the hg19 2bit
    reference
- name: blastx
  role: Screens translated HERV candidate sequences against the NCBI NR protein database
    at E-value 1e-4 to identify retroviral protein motifs
- name: MAFFT
  role: Aligns candidate POL protein sequences to build a POL profile used to detect
    POL homology in HERV candidates
- name: viRNAtrap
  role: Upstream contig assembler whose unmapped-read-derived contigs are subsequently
    aligned against the curated HERV database
  repo: https://github.com/AuslanderLab/virnatrap.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:7c128a12944530bc774334a53412843b1ac351ce80bf7be3dd68facd477fede4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - MAFFT
  - blastx
  - twoBitToFa
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# profile-based-homology-search

## Summary

Constructs a curated database of potentially functional human endogenous retroviruses (HERVs) by extracting HERVd-annotated loci from hg19, screening translated sequences against NCBI NR with blastx (E-value 1e-4), and building a MAFFT POL profile to retain only sequences carrying POL/RT, GAG, or ENV retroviral motifs.

## When to use

Use when building a reference database of endogenous retroviral sequences that must each be backed by detectable retroviral protein motifs (POL/RT, GAG, ENV) before downstream alignment of assembled contigs.

## When NOT to use

- When the goal is de novo viral discovery from raw RNAseq reads — use the viRNAtrap deep-learning pipeline instead; this skill only builds the reference HERV database
- When candidate sequences are already curated as intact retroviruses and only need classification against known viral genomes via blastn

## Inputs

- hg19 human reference genome (2bit format, queried via UCSC twoBitToFa)
- HERVd-annotated endogenous retrovirus locus coordinates
- NCBI NR protein database for blastx screening
- Candidate POL (reverse transcriptase) protein sequences for MAFFT profile construction

## Outputs

- herv_database.fasta: curated set of intact HERV sequences, each supported by at least one POL/RT, GAG, or ENV motif hit
- MAFFT-built POL protein profile used to score POL homology in candidate sequences

## How to apply

Extract genomic sequences spanning HERVd-annotated endogenous retrovirus loci from the hg19 human reference genome using the UCSC twoBitToFa utility. Translate the extracted HERV sequences and screen them against the NCBI NR protein database with blastx using an E-value threshold of 1e-4. Align candidate POL (reverse transcriptase) protein sequences with MAFFT to build a POL profile, then search the HERV candidate sequences against this profile to detect POL homology beyond what blastx alone recovers. Retain only candidate HERV sequences with at least one detectable retroviral protein motif from POL/RT, GAG, or ENV, and write the retained intact-HERV sequences to herv_database.fasta. This curated database then serves as the alignment target for viRNAtrap-assembled contigs to characterize HERV expression across the 14 TCGA cancer types.

## Related tools

- **twoBitToFa** (Extracts genomic sequences spanning HERVd-annotated loci from the hg19 2bit reference)
- **blastx** (Screens translated HERV candidate sequences against the NCBI NR protein database at E-value 1e-4 to identify retroviral protein motifs)
- **MAFFT** (Aligns candidate POL protein sequences to build a POL profile used to detect POL homology in HERV candidates)
- **viRNAtrap** (Upstream contig assembler whose unmapped-read-derived contigs are subsequently aligned against the curated HERV database) — https://github.com/AuslanderLab/virnatrap.git

## Examples

```
twoBitToFa hg19.2bit herv_loci.fa -seqList=hervd_loci.txt; blastx -query herv_loci.fa -db nr -evalue 1e-4 -outfmt 5; mafft pol_candidates.fasta > pol_profile.aln
```

## Evaluation signals

- Every sequence written to herv_database.fasta has at least one blastx hit with E-value <= 1e-4 OR a MAFFT POL-profile hit supporting a POL/RT, GAG, or ENV retroviral protein motif
- No candidate HERV lacking all three motif types (POL/RT, GAG, ENV) is retained in the final FASTA
- Extracted locus sequences exactly match the hg19 coordinates of the HERVd-annotated endogenous retrovirus entries
- The final database size is consistent with the paper's count of 3044 potentially functional HERVs
- Downstream: viRNAtrap contigs align successfully against herv_database.fasta, reproducing HERV expression patterns across the 14 cancer types

## Limitations

- Sensitivity depends on the completeness of the HERVd annotation and the NCBI NR database version used at build time
- The dual evidence requirement (blastx OR MAFFT POL profile) may miss HERVs whose retroviral motifs are too diverged for either method
- The database is genome-build specific (hg19); locus coordinates must be re-extracted for other builds

## Evidence

- [methods] curated HERV database as alignment target: [evidence span withheld — anchor status: not_found]
- [discussion] curated intact HERV genes: "We curate a database of HERVs that comprise intact retroviral genes in the human genome and survey the expression of these viruses across different cancer tissues"
- [intro] curate potentially functional HERVs: "we curate a database of potentially functional human endogenous retroviruses (HERVs) and analyze expression patterns of different HERVs across human cancers"
- [supplementary] HERV database construction workflow: [evidence span withheld — anchor status: not_found]
