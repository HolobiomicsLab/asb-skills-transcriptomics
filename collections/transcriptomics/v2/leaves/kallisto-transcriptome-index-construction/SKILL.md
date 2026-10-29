---
name: kallisto-transcriptome-index-construction
description: Use when building a kallisto index for scRNA-seq that must distinguish
  spliced (cDNA) from intronic (unspliced) reads — e.g., before running scKB to emit
  spliced and unspliced count matrices for COPILOT or RNA velocity analysis.
when_to_use_negative:
- The reference genome is not available as a BSgenome object and a customized genome
  cannot be converted — the protocol's troubleshooting explicitly flags this as a
  blocking problem.
- Only whole-transcriptome UMI counts are needed (no splicing dynamics or RNA velocity
  analysis); a cDNA-only kallisto index without intronic entries would be the simpler
  choice.
edam_operation: http://edamontology.org/operation_3580
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BUSpaRse
  role: get_velocity_files extracts intronic intervals from the TAIR10 GTF against
    the BSgenome and emits the combined cDNA+intron FASTA plus transcript capture
    and tr2g files
- name: kallisto
  role: builds the pseudoalignment index ('kallisto index') on cDNA_introns.fa and
    performs the underlying pseudoalignment invoked through scKB
- name: scKB
  role: wrapper of kallisto and bustools that loads cDNA_introns_10xv3.idx and tr2g.tsv
    to accelerate alignment and generate spliced/unpliced transcript count matrices
- name: BSgenome
  role: supplies the TAIR9 genome sequence object (BSgenome.Athaliana.TAIR.TAIR9)
    required as the Genome argument of get_velocity_files
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:4878febcc4604957a932fa74bf710f0223612289e47eb9a5177b0f49f8b0d8d4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  tools:
  - BSgenome
  - BUSpaRse
  - bustools
  - kallisto
  - scKB
  edam_operation: http://edamontology.org/operation_3580
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# kallisto-transcriptome-index-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a combined cDNA+intron kallisto pseudoalignment index (cDNA_introns_10xv3.idx) from the TAIR10 GTF and the TAIR9 BSgenome sequence via BUSpaRse::get_velocity_files, so scKB/kallisto can split scRNA-seq reads into spliced and unspliced count matrices.

## When to use

Use when building a kallisto index for scRNA-seq that must distinguish spliced (cDNA) from intronic (unspliced) reads — e.g., before running scKB to emit spliced and unspliced count matrices for COPILOT or RNA velocity analysis.

## When NOT to use

- The reference genome is not available as a BSgenome object and a customized genome cannot be converted — the protocol's troubleshooting explicitly flags this as a blocking problem.
- Only whole-transcriptome UMI counts are needed (no splicing dynamics or RNA velocity analysis); a cDNA-only kallisto index without intronic entries would be the simpler choice.

## Inputs

- Genome annotation file (GTF/GFF3, e.g., the TAIR10 gtf for Arabidopsis thaliana)
- Genome sequence as a BSgenome object (BSgenome.Athaliana.TAIR.TAIR9) or matching FASTA

## Outputs

- cDNA_introns.fa — combined spliced + intronic transcript FASTA
- cDNA_introns_10xv3.idx — kallisto pseudoalignment index built by 'kallisto index'
- cDNA_tx_to_capture.txt and introns_tx_to_capture.txt — transcript capture lists for cDNA and introns
- tr2g.tsv — transcript-to-gene mapping used by scKB/bustools

## How to apply

Load the Arabidopsis thaliana TAIR10 GTF/GFF3 annotation together with the genome sequence as a BSgenome object (BSgenome.Athaliana.TAIR.TAIR9) into R. Run BUSpaRse::get_velocity_files with X pointing to the TAIR10 gtf, L = 91, isoform_action = 'separate', chrs_only = FALSE, and style = 'Ensembl'; this extracts intronic intervals and writes cDNA_introns.fa, cDNA_tx_to_capture.txt, introns_tx_to_capture.txt, and tr2g.tsv into the working directory. Then run 'kallisto index' on cDNA_introns.fa to produce cDNA_introns_10xv3.idx. The key decision point is isoform_action='separate', which keeps intronic sequences as distinct transcript entries so pseudoalignment can attribute reads to unspliced rather than spliced features. Validate that 'kallisto index' completes without error and that the index and transcript-to-gene mapping files are non-empty and correctly named, since scKB later loads the index and tr2g.tsv to produce the raw spliced and unpliced count matrices that COPILOT consumes.

## Related tools

- **BUSpaRse** (get_velocity_files extracts intronic intervals from the TAIR10 GTF against the BSgenome and emits the combined cDNA+intron FASTA plus transcript capture and tr2g files)
- **kallisto** (builds the pseudoalignment index ('kallisto index') on cDNA_introns.fa and performs the underlying pseudoalignment invoked through scKB)
- **scKB** (wrapper of kallisto and bustools that loads cDNA_introns_10xv3.idx and tr2g.tsv to accelerate alignment and generate spliced/unpliced transcript count matrices)
- **BSgenome** (supplies the TAIR9 genome sequence object (BSgenome.Athaliana.TAIR.TAIR9) required as the Genome argument of get_velocity_files)

## Examples

```
BUSpaRse::get_velocity_files(X='TAIR10.gtf', L=91, Genome=BSgenome.Athaliana.TAIR.TAIR9, isoform_action='separate', chrs_only=FALSE, style='Ensembl'); kallisto index -i cDNA_introns_10xv3.idx cDNA_introns.fa
```

## Evaluation signals

- 'kallisto index' completes without error on the combined cDNA_introns.fa (the protocol's stated validation criterion).
- All four expected files — cDNA_introns.fa, cDNA_tx_to_capture.txt, introns_tx_to_capture.txt, and tr2g.tsv — exist in the working directory and are non-empty.
- cDNA_introns_10xv3.idx is loadable by kallisto/scKB, i.e., a subsequent scKB alignment run starts without index errors.
- isoform_action='separate' yields distinct intronic transcript entries, so a downstream scKB run emits both spliced and unpliced count matrices rather than a single total matrix.
- The scKB output matrices are accepted as COPILOT input (copilot() runs with spliced.mtx/unspliced.mtx arguments populated).

## Limitations

- The required genome may not be available as a BSgenome object; users need to convert a customized genome into a BSgenome object (protocol troubleshooting).
- The demonstrated parameters (TAIR10 GTF against BSgenome.Athaliana.TAIR.TAIR9, L = 91, chrs_only = FALSE, style = 'Ensembl') are specific to the Arabidopsis thaliana setup; other organisms require re-deriving these values.
- The protocol's troubleshooting notes that scKB can fail to run properly in the related 'align raw reads to genome via scKB' section, which depends on a correctly built index.

## Evidence

- [methods] BUSpaRse::get_velocity_files with X pointing to the TAIR10 gtf, L = 91, Genome = BSgenome.Athaliana.TAIR.TAIR9, isoform_action = "separate", chrs_only = FALSE, and style = "Ensembl": [evidence span withheld — anchor status: not_found]
- [methods] the working directory then contains cDNA_introns.fa, cDNA_tx_to_capture.txt, introns_tx_to_capture.txt, and tr2g.tsv: [evidence span withheld — anchor status: not_found]
- [methods] cDNA_introns.fa is indexed with "kallisto index" to produce cDNA_introns_10xv3.idx for use by scKB: [evidence span withheld — anchor status: not_found]
