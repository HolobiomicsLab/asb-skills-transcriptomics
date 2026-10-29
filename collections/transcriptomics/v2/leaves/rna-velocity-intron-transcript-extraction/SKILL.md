---
name: rna-velocity-intron-transcript-extraction
description: Use when preparing a kallisto/bustools index for RNA velocity or splicing-aware
  scRNA-seq quantification, so that pseudoalignment can separate spliced (cDNA) reads
  from unspliced (intronic) reads.
when_to_use_negative:
- If the goal is only gene-level quantification without splicing dynamics or RNA velocity
  — a plain cDNA transcriptome index suffices and intron extraction is unnecessary
  overhead.
- If the genome is not available as a BSgenome object and cannot be converted; the
  protocol notes the required genome may not exist as a BSgenome object and users
  need to convert a customized genome.
- If alignment to a whole genome with splice-aware alignment (e.g. STAR-based) is
  required instead of transcriptome pseudoalignment.
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: BUSpaRse
  role: Runs get_velocity_files to extract intronic intervals from the GTF against
    the genome and emit cDNA+intron FASTA and transcript-to-gene mappings
- name: kallisto
  role: Builds the pseudoalignment index on the combined cDNA+intron FASTA and pseudoaligns
    reads (via scKB)
- name: scKB
  role: Wrapper script of kallisto and bustools that consumes the index to generate
    spliced and unspliced transcript count matrices
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: BSgenome
  role: Provides the genome sequence object (BSgenome.Athaliana.TAIR.TAIR9) required
    by get_velocity_files
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:7c0ece7e6c01548ceb8639b3ae5a3e2d64bcade462b5a59072593f339da84c11
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  repo_url: https://github.com/Hsu-Che-Wei/COPILOT
  tools:
  - BSgenome
  - BUSpaRse
  - bustools
  - kallisto
  - scKB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# rna-velocity-intron-transcript-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a kallisto pseudoalignment index that distinguishes spliced from unspliced reads by using BUSpaRse::get_velocity_files to extract intronic intervals from a GTF annotation (e.g. TAIR10) against a genome (BSgenome.Athaliana.TAIR.TAIR9), concatenating cDNA and intron sequences, and indexing the combined FASTA for scKB.

## When to use

Use when preparing a kallisto/bustools index for RNA velocity or splicing-aware scRNA-seq quantification, so that pseudoalignment can separate spliced (cDNA) reads from unspliced (intronic) reads.

## When NOT to use

- If the goal is only gene-level quantification without splicing dynamics or RNA velocity — a plain cDNA transcriptome index suffices and intron extraction is unnecessary overhead.
- If the genome is not available as a BSgenome object and cannot be converted; the protocol notes the required genome may not exist as a BSgenome object and users need to convert a customized genome.
- If alignment to a whole genome with splice-aware alignment (e.g. STAR-based) is required instead of transcriptome pseudoalignment.

## Inputs

- Transcriptome annotation GTF/GFF3 file (e.g. TAIR10 for Arabidopsis thaliana)
- Genome sequence as a BSgenome object or FASTA (e.g. BSgenome.Athaliana.TAIR.TAIR9)
- Flank length parameter L (91 nt per protocol)

## Outputs

- cDNA_introns.fa (combined spliced + unspliced transcript FASTA)
- cDNA_tx_to_capture.txt and introns_tx_to_capture.txt (transcript-to-capture mappings)
- tr2g.tsv (transcript-to-gene mapping)
- kallisto index file (e.g. cDNA_introns_10xv3.idx / sc_kb_kallisto_index.idx)
- Downstream: spliced and unspliced gene-by-cell UMI count matrices from scKB

## How to apply

Load the genome sequence (e.g. BSgenome.Athaliana.TAIR.TAIR9) and the matching GTF/GFF3 annotation (e.g. TAIR10) into R, then run BUSpaRse::get_velocity_files with L=91, isoform_action='separate', chrs_only=FALSE, and style='Ensembl' to emit cDNA_introns.fa, cDNA_tx_to_capture.txt, introns_tx_to_capture.txt, and tr2g.tsv. The rationale for isoform_action='separate' is that isoform-intronic sequences are written out separately rather than collapsed, preserving spliced/unspliced distinction. Concatenate the cDNA and intron transcript sequences into a single transcriptome FASTA (cDNA_introns.fa) covering both features, then run 'kallisto index' on it to produce the index (e.g. cDNA_introns_10xv3.idx / sc_kb_kallisto_index.idx). scKB, a wrapper of kallisto and bustools, then consumes this index together with the transcript-to-gene mappings to pseudoalign reads and emit spliced and unspliced count matrices. Verify that 'kallisto index' completes without error and that the index and t2c files are non-empty and loadable by kallisto/scKB.

## Related tools

- **BUSpaRse** (Runs get_velocity_files to extract intronic intervals from the GTF against the genome and emit cDNA+intron FASTA and transcript-to-gene mappings)
- **kallisto** (Builds the pseudoalignment index on the combined cDNA+intron FASTA and pseudoaligns reads (via scKB))
- **scKB** (Wrapper script of kallisto and bustools that consumes the index to generate spliced and unspliced transcript count matrices) — https://github.com/Hsu-Che-Wei/COPILOT
- **BSgenome** (Provides the genome sequence object (BSgenome.Athaliana.TAIR.TAIR9) required by get_velocity_files)

## Examples

```
R> get_velocity_files(X="TAIR10.gtf", L=91, Genome=BSgenome.Athaliana.TAIR.TAIR9, isoform_action="separate", chrs_only=FALSE, style="Ensembl"); system("kallisto index -i cDNA_introns_10xv3.idx cDNA_introns.fa")
```

## Evaluation signals

- 'kallisto index' completes without error on the combined cDNA_introns.fa
- cDNA_introns.fa, cDNA_tx_to_capture.txt, introns_tx_to_capture.txt, and tr2g.tsv are created in the working directory, non-empty, and correctly named
- The index and transcript-to-gene mappings are loadable by kallisto and scKB
- scKB subsequently produces both spliced and unspliced count matrices from the index
- Output index name matches scKB expectations (e.g. cDNA_introns_10xv3.idx for 10x v3 chemistry)

## Limitations

- COPILOT/scKB annotation support is limited: 'The current version of COPILOT only supports cell type and developmental stage annotation for the root of Arabidopsis thaliana.'
- The required genome may not be available as a BSgenome object; users must convert customized genomes before running get_velocity_files.
- scKB can fail to run properly during the 'align raw reads to genome via scKB' step (protocol troubleshooting Problem 3).

## Evidence
