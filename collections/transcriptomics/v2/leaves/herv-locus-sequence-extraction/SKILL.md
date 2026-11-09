---
name: herv-locus-sequence-extraction
description: Use when a study needs a reference set of intact/potentially functional
  HERV sequences from the human genome to serve as an alignment target for endogenous
  retroviral expression analysis in transcriptomic data.
when_to_use_negative:
- The goal is detecting exogenous/divergent viral reads in RNAseq — that requires
  the viRNAtrap deep-learning pipeline, not a human-genome-derived HERV database
- Only annotated-but-unverified HERV coordinates are needed without protein-motif
  evidence — plain twoBitToFa extraction alone suffices and the blastx/MAFFT filtering
  should be skipped
- A non-human genome is the target; this database is built exclusively from hg19 HERVd
  annotations
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0102
tools:
- name: twoBitToFa
  role: Extracts genomic FASTA sequences spanning HERVd-annotated loci from the hg19
    2bit reference genome
- name: blastx
  role: Translated search of extracted HERV candidate sequences against the NCBI NR
    protein database (E-value threshold 1e-4) to detect POL/RT, GAG, or ENV retroviral
    protein motifs
- name: MAFFT
  role: Aligns candidate POL/RT protein sequences to build a POL profile used to detect
    POL homology in HERV candidates
- name: viRNAtrap
  role: 'Downstream consumer: viRNAtrap-assembled tumor RNAseq contigs are aligned
    against the resulting herv_database.fasta to characterize HERV expression across
    14 cancer types'
  repo: https://github.com/AuslanderLab/virnatrap.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:ba2ee0d2a0941d9ca55a3417c49aadfb702e21ebff95bb068294ac1c3deffd11
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
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# herv-locus-sequence-extraction

## Summary

Construct a curated database of potentially functional human endogenous retroviruses (HERVs) by extracting HERVd-annotated loci from hg19 with UCSC twoBitToFa and retaining only sequences showing retroviral POL/RT, GAG, or ENV protein motifs via blastx (E-value ≤ 1e-4) and a MAFFT POL profile. This database serves as the alignment target for characterizing HERV expression in 14 TCGA cancer types.

## When to use

Use when a study needs a reference set of intact/potentially functional HERV sequences from the human genome to serve as an alignment target for endogenous retroviral expression analysis in transcriptomic data.

## When NOT to use

- The goal is detecting exogenous/divergent viral reads in RNAseq — that requires the viRNAtrap deep-learning pipeline, not a human-genome-derived HERV database
- Only annotated-but-unverified HERV coordinates are needed without protein-motif evidence — plain twoBitToFa extraction alone suffices and the blastx/MAFFT filtering should be skipped
- A non-human genome is the target; this database is built exclusively from hg19 HERVd annotations

## Inputs

- hg19 human reference genome (2bit format for twoBitToFa)
- HERVd-annotated endogenous retrovirus locus coordinates
- NCBI NR protein database (for blastx screening)

## Outputs

- herv_database.fasta — curated FASTA of 3044 potentially functional HERV sequences, each supported by a POL/RT, GAG, or ENV protein-motif hit

## How to apply

First, extract genomic sequences spanning HERVd-annotated endogenous retrovirus loci from the hg19 human reference genome using the UCSC twoBitToFa utility. Translate the extracted sequences and screen them against the NCBI NR protein database with blastx using an E-value threshold of 1e-4. Build a POL profile by aligning candidate reverse transcriptase (RT) protein sequences with MAFFT, then search the HERV candidates against this profile to detect POL homology. Retain only HERV sequences carrying at least one detectable retroviral protein motif from POL/RT, GAG, or ENV, and write them to herv_database.fasta. The rationale is that only sequences with intact retroviral gene evidence count as 'potentially functional' HERVs suitable as an expression-alignment target for tumor RNAseq contigs.

## Related tools

- **twoBitToFa** (Extracts genomic FASTA sequences spanning HERVd-annotated loci from the hg19 2bit reference genome)
- **blastx** (Translated search of extracted HERV candidate sequences against the NCBI NR protein database (E-value threshold 1e-4) to detect POL/RT, GAG, or ENV retroviral protein motifs)
- **MAFFT** (Aligns candidate POL/RT protein sequences to build a POL profile used to detect POL homology in HERV candidates)
- **viRNAtrap** (Downstream consumer: viRNAtrap-assembled tumor RNAseq contigs are aligned against the resulting herv_database.fasta to characterize HERV expression across 14 cancer types) — https://github.com/AuslanderLab/virnatrap.git

## Examples

```
twoBitToFa hg19.2bit -bed=hervd_loci.bed herv_candidates.fa && blastx -query herv_candidates.fa -db nr -evalue 1e-4 -out herv_motifs.tsv
```

## Evaluation signals

- Every sequence in herv_database.fasta has at least one supporting hit: a blastx hit (E-value ≤ 1e-4) against NR or a MAFFT POL-profile hit evidencing a POL/RT, GAG, or ENV motif
- The curated count matches the published ~3044 filtered HERVs
- All output sequences originate from hg19 coordinates annotated in HERVd
- Sequences lacking any retroviral protein motif evidence are excluded from the FASTA
- The database functions as an alignment target: viRNAtrap contigs can be aligned against it to reproduce HERV expression patterns across the 14 TCGA cancer types

## Limitations

- The database captures only HERVs with detectable retroviral protein motifs (POL/RT, GAG, ENV); degraded/fragmented HERVs lacking these motifs are excluded
- Detection depends on blastx/MAFFT profile sensitivity (E-value 1e-4); highly divergent HERV loci may fail motif detection and be missed
- The database is hg19-specific; locus coordinates do not directly transfer to other genome builds

## Evidence

- [methods] Extract genomic sequences spanning the HERVd-annotated endogenous retrovirus loci from the hg19 human reference genome using the UCSC twoBitToFa utility: [evidence span withheld — anchor status: not_found]
- [discussion] Curate a database of HERVs with intact retroviral genes: "We curate a database of HERVs that comprise intact retroviral genes in the human genome and survey the expression of these viruses across different cancer tissues"
- [methods] Retain HERV sequences with retroviral motifs: [evidence span withheld — anchor status: not_found]
