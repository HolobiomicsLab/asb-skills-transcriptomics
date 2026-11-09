---
name: blastx-database-screening
description: Use when translated nucleotide contigs or extracted genomic loci (e.g.,
  HERVd-annotated HERV regions from hg19) must be screened against a protein database
  like NCBI NR to detect retroviral POL/RT, GAG, or ENV protein motifs, or to verify
  viral contig classification.
when_to_use_negative:
- Input is raw RNA-seq reads that still need viral-read identification and assembly
  — use the viRNAtrap deep-learning pipeline first; blastx is a post-processing/verification
  step on assembled contigs
- Comparison target is nucleotide-level matching of assembled contigs against curated
  viral nucleotide databases — the article uses blastn for that post-processing step,
  not blastx
- Sequences already carry validated protein-domain annotations for POL/RT, GAG, or
  ENV, making translated database screening redundant
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: blastx
  role: Translated (nucleotide query vs protein database) similarity search used to
    screen HERV candidates against NCBI NR at E-value 1e-4 and to verify viral read
    classification
- name: twoBitToFa
  role: UCSC utility used to extract the genomic sequences spanning HERVd-annotated
    loci from hg19 to create the blastx query FASTA
- name: MAFFT
  role: Aligns candidate POL (reverse transcriptase) protein sequences to build a
    POL profile used alongside blastx to detect POL homology in HERV candidates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:f6a8b0aca76957c657d408b26e83fadb4d36bb018068468bee611d15ddf5be3a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAFFT
  - NCBI NR protein database
  - blastx
  - twoBitToFa
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# blastx-database-screening

## Summary

Screens translated nucleotide sequences (e.g., HERV loci extracted from hg19) against a protein database such as NCBI NR using blastx, and verifies viral sequence classification, with hits at E-value <= 1e-4 supporting retroviral POL/RT, GAG, or ENV motifs.

## When to use

Use when translated nucleotide contigs or extracted genomic loci (e.g., HERVd-annotated HERV regions from hg19) must be screened against a protein database like NCBI NR to detect retroviral POL/RT, GAG, or ENV protein motifs, or to verify viral contig classification.

## When NOT to use

- Input is raw RNA-seq reads that still need viral-read identification and assembly — use the viRNAtrap deep-learning pipeline first; blastx is a post-processing/verification step on assembled contigs
- Comparison target is nucleotide-level matching of assembled contigs against curated viral nucleotide databases — the article uses blastn for that post-processing step, not blastx
- Sequences already carry validated protein-domain annotations for POL/RT, GAG, or ENV, making translated database screening redundant

## Inputs

- Genomic FASTA sequences of HERVd-annotated endogenous retrovirus loci extracted from hg19 (via twoBitToFa)
- ViRNAtrap-assembled predicted viral contigs (FASTA .txt output files)
- NCBI NR protein database

## Outputs

- blastx hit tables (E-value <= 1e-4) mapping translated HERV candidates to retroviral POL/RT, GAG, or ENV proteins in NR
- herv_database.fasta — curated database of potentially functional HERVs (3044 sequences) each supported by at least one blastx or MAFFT-POL-profile motif hit
- Verified viral read classifications for divergent viruses reported across the 14 TCGA cancer types

## How to apply

First extract the genomic sequences spanning HERVd-annotated endogenous retrovirus loci from the hg19 human reference genome with the UCSC twoBitToFa utility. Translate the extracted HERV sequences and screen them against the NCBI NR protein database with blastx using an E-value threshold of 1e-4. As a complementary homology check, align candidate POL (reverse transcriptase) protein sequences with MAFFT to build a POL profile and search the HERV candidates against it. Retain only those HERV candidate sequences with at least one detectable retroviral protein motif from POL/RT, GAG, or ENV, and write the retained intact-HERV sequences to herv_database.fasta. The same blastx-based targeted analysis is used post-hoc to verify the presence and classification of multiple viral reads for divergent viruses reported by viRNAtrap.

## Related tools

- **blastx** (Translated (nucleotide query vs protein database) similarity search used to screen HERV candidates against NCBI NR at E-value 1e-4 and to verify viral read classification)
- **twoBitToFa** (UCSC utility used to extract the genomic sequences spanning HERVd-annotated loci from hg19 to create the blastx query FASTA)
- **MAFFT** (Aligns candidate POL (reverse transcriptase) protein sequences to build a POL profile used alongside blastx to detect POL homology in HERV candidates)

## Examples

```
blastx -query herv_candidate_sequences.fasta -db nr -evalue 1e-4 -out herv_nr_hits.txt
```

## Evaluation signals

- Every sequence in herv_database.fasta shows at least one blastx hit with E-value <= 1e-4, or a MAFFT-POL-profile hit, supporting a POL/RT, GAG, or ENV retroviral protein motif
- The retained database is consistent with the article's curated set of 3044 potentially functional HERVs filtered by POL/GAG/ENV motifs
- For divergent viruses, multiple viral reads (not a single read) are confirmed by the targeted blastn/blastx analyses in different samples
- Query FASTA sequences trace back to hg19 coordinates of HERVd-annotated endogenous retrovirus loci extracted with twoBitToFa

## Limitations

- Screening retains only HERV candidates with detectable homology to POL/RT, GAG, or ENV motifs in NR or the POL profile — endogenous retroviruses without any of these detectable motifs are excluded, so the database is not exhaustive of all HERV loci
- The article notes it is not possible to model all contaminants of viruses that may have infected samples during laboratory procedures, so blastx verification cannot rule out every contaminant origin for a detected viral sequence

## Evidence

- [discussion] blastx verification of divergent viruses: "the presence and classification of multiple viral reads was verified by targeted blastn- and blastx-based sequence analyses in different samples"
- [discussion] curated HERV database with intact retroviral genes: "We curate a database of HERVs that comprise intact retroviral genes in the human genome and survey the expression of these viruses across different cancer tissues"
- [intro] contig comparison to curated databases: "we used blastn24 to compare the assembled viral contigs to three curated viral databases"
