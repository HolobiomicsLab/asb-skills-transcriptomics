---
name: retroviral-protein-motif-annotation
description: Use when building or validating a curated database of potentially functional
  HERVs by screening genomic endogenous-retrovirus loci for intact retroviral protein
  motifs (POL/RT, GAG, ENV).
when_to_use_negative:
- Input is assembled contigs from exogenous (non-endogenous) viruses — use blastn/blastx
  against curated viral databases instead of the HERV locus-extraction workflow
- The reference genome is not hg19 — HERVd-annotated locus coordinates must match
  the genome build from which sequences are extracted
- The goal is de novo discovery of proviral insertions rather than annotating previously
  cataloged HERVd loci for functional potential
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: twoBitToFa
  role: Extracts genomic sequences spanning HERVd-annotated endogenous retrovirus
    loci from the hg19 twoBit reference genome
- name: blastx
  role: Screens translated HERV candidate sequences against the NCBI NR protein database
    at E-value <= 1e-4 to detect retroviral protein motif homology
- name: MAFFT
  role: Aligns candidate POL (reverse transcriptase) protein sequences to build a
    POL profile used to detect POL homology in HERV candidates
- name: viRNAtrap
  role: Assembles viral contigs from unmapped TCGA RNAseq reads that are subsequently
    aligned against the curated HERV database to characterize HERV expression
  repo: https://github.com/AuslanderLab/virnatrap.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:5a51d1526cfa749bd7f24f08d42ed2f916c0545d72b53a05bdf8464846905384
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - MAFFT
  - NCBI NR protein database
  - blastx
  - twoBitToFa
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# retroviral-protein-motif-annotation

## Summary

Constructs a curated database of potentially functional human endogenous retroviruses (HERVs) by extracting HERVd-annotated loci from hg19 and retaining only sequences with detectable retroviral protein motifs (POL/RT, GAG, or ENV). The resulting ~3044-HERV database (herv_database.fasta) serves as the alignment target for characterizing HERV expression in tumors.

## When to use

Use when building or validating a curated database of potentially functional HERVs by screening genomic endogenous-retrovirus loci for intact retroviral protein motifs (POL/RT, GAG, ENV).

## When NOT to use

- Input is assembled contigs from exogenous (non-endogenous) viruses — use blastn/blastx against curated viral databases instead of the HERV locus-extraction workflow
- The reference genome is not hg19 — HERVd-annotated locus coordinates must match the genome build from which sequences are extracted
- The goal is de novo discovery of proviral insertions rather than annotating previously cataloged HERVd loci for functional potential

## Inputs

- hg19 human reference genome (twoBit format) with HERVd-annotated endogenous retrovirus locus coordinates
- NCBI NR protein database for blastx screening
- Candidate POL reverse transcriptase protein sequences for MAFFT profile construction

## Outputs

- herv_database.fasta — curated set of 3044 potentially functional HERV sequences each supported by a POL/RT, GAG, or ENV retroviral protein motif hit
- blastx (E-value <= 1e-4) annotation of translated HERV sequences against NR
- MAFFT-built POL protein profile used for homology detection

## How to apply

Extract genomic sequences spanning HERVd-annotated endogenous retrovirus loci from the hg19 human reference genome using the UCSC twoBitToFa utility. Translate the extracted HERV sequences and screen them against the NCBI NR protein database with blastx using an E-value threshold of 1e-4. Align candidate POL (reverse transcriptase) protein sequences with MAFFT to build a POL profile, then search the HERV candidate sequences against this profile to detect POL homology. Retain from the candidate set only those HERV sequences with at least one detectable retroviral protein motif from POL/RT, GAG, or ENV, and write the retained intact-HERV sequences to herv_database.fasta. Downstream, viRNAtrap-assembled contigs from TCGA RNAseq samples are aligned against this database to identify patterns of HERV expression across the 14 cancer types studied.

## Related tools

- **twoBitToFa** (Extracts genomic sequences spanning HERVd-annotated endogenous retrovirus loci from the hg19 twoBit reference genome)
- **blastx** (Screens translated HERV candidate sequences against the NCBI NR protein database at E-value <= 1e-4 to detect retroviral protein motif homology)
- **MAFFT** (Aligns candidate POL (reverse transcriptase) protein sequences to build a POL profile used to detect POL homology in HERV candidates)
- **viRNAtrap** (Assembles viral contigs from unmapped TCGA RNAseq reads that are subsequently aligned against the curated HERV database to characterize HERV expression) — https://github.com/AuslanderLab/virnatrap.git

## Examples

```
blastx -query herv_candidates.fasta -db nr -evalue 1e-4
```

## Evaluation signals

- Every sequence in herv_database.fasta shows at least one blastx hit (E-value <= 1e-4) or MAFFT POL-profile hit supporting a POL/RT, GAG, or ENV retroviral protein motif
- The database contains 3044 HERV sequences filtered from the full set of HERVd-annotated hg19 loci
- Each retained sequence's coordinates map back to a HERVd-annotated endogenous retrovirus locus in hg19
- Retained sequences each carry at least one of the three retroviral gene motifs (POL/RT, GAG, ENV), matching the 'intact retroviral genes' curation criterion

## Limitations

- HERV loci lacking detectable POL/RT, GAG, or ENV motifs (e.g., highly degraded or divergent proviruses) are excluded, so the database captures only potentially functional elements
- The database is built from hg19 HERVd annotations; loci annotated in other builds or newly described HERVd entries are not represented
- Motif detection depends on blastx (E-value 1e-4) and the MAFFT POL profile; thresholds too strict or too permissive directly change the curated set

## Evidence

- [intro] we curate a database of potentially functional human endogenous retroviruses (HERVs) and analyze expression patterns of different HERVs across human cancers: "we curate a database of potentially functional human endogenous retroviruses (HERVs) and analyze expression patterns of different HERVs across human cancers"
- [discussion] We curate a database of HERVs that comprise intact retroviral genes in the human genome and survey the expression of these viruses across different cancer tissues: "We curate a database of HERVs that comprise intact retroviral genes in the human genome and survey the expression of these viruses across different cancer tissues"
- [discussion] the presence and classification of multiple viral reads was verified by targeted blastn- and blastx-based sequence analyses in different samples: "the presence and classification of multiple viral reads was verified by targeted blastn- and blastx-based sequence analyses in different samples"
