---
name: spliced-unspliced-read-classification-setup
description: Use when preparing the reference index for scKB/kallisto so single-cell
  RNA-seq reads can be classified as spliced or unspliced (e.g., for RNA velocity
  or trajectory inference from splicing dynamics).
when_to_use_negative:
- The analysis goal is standard quantification without splicing dynamics — a spliced-only
  transcriptome index suffices and the intron-containing index is unnecessary
- Using a pipeline other than kallisto/scKB (e.g., Cell Ranger), which does not consume
  the cDNA_introns kallisto index or tr2g.tsv format
- The genome is not available as a BSgenome object and no customized conversion is
  possible
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BUSpaRse
  role: R function get_velocity_files extracts intronic intervals from the GTF and
    generates the cDNA+intron FASTA and tr2g.tsv transcript-to-gene mappings
- name: kallisto
  role: Builds the pseudoalignment index ('kallisto index') on the combined cDNA_introns.fa
    and performs pseudoalignment of reads via scKB
- name: scKB
  role: Wrapper script of kallisto and bustools that consumes the index to align raw
    reads and generate spliced and unspliced transcript count matrices
- name: bustools
  role: Underlying busTools engine used by scKB to correct barcodes/UMIs and produce
    count matrices from the bus file using tr2g.tsv
- name: BSgenome
  role: Supplies the genome sequence object required by BUSpaRse::get_velocity_files
    (e.g., BSgenome.Athaliana.TAIR.TAIR9)
- name: COPILOT
  role: Downstream consumer of the raw spliced and unspliced matrices for quality
    filtering; motivates producing separate spliced/unspliced counts
  repo: https://github.com/Hsu-Che-Wei/COPILOT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:813c9aa33fbe0b42e38779e85cf07e0b083b6e3762f6188a58b311740a8e715f
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
  - COPILOT
  - bustools
  - kallisto
  - scKB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# spliced-unspliced-read-classification-setup

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a kallisto transcriptome index that contains both cDNA (spliced) and intronic (unspliced) sequences, plus transcript-to-gene mappings, so that scKB/kallisto can pseudoalign reads and emit separate spliced and unspliced UMI count matrices for RNA velocity analysis.

## When to use

Use when preparing the reference index for scKB/kallisto so single-cell RNA-seq reads can be classified as spliced or unspliced (e.g., for RNA velocity or trajectory inference from splicing dynamics).

## When NOT to use

- The analysis goal is standard quantification without splicing dynamics — a spliced-only transcriptome index suffices and the intron-containing index is unnecessary
- Using a pipeline other than kallisto/scKB (e.g., Cell Ranger), which does not consume the cDNA_introns kallisto index or tr2g.tsv format
- The genome is not available as a BSgenome object and no customized conversion is possible

## Inputs

- Genome sequence as a BSgenome object or FASTA (e.g., BSgenome.Athaliana.TAIR.TAIR9)
- Matching transcriptome annotation GTF/GFF3 file (e.g., TAIR10 for Arabidopsis)
- kallite/busTools-compatible parameter choices: L=91 barcode/UMI-aware fragment length, style='Ensembl'

## Outputs

- cDNA_introns.fa — combined spliced + unspliced transcript FASTA
- cDNA_introns_10xv3.idx — kallisto index built on the combined FASTA
- cDNA_tx_to_capture.txt and introns_tx_to_capture.txt — capture lists
- tr2g.tsv — transcript-to-gene mapping used by bustools/scKB
- Downstream: raw spliced and unspliced gene-by-cell UMI count matrices from scKB

## How to apply

Load the genome sequence as a BSgenome object (e.g., BSgenome.Athaliana.TAIR.TAIR9 for the Arabidopsis TAIR9 genome) and the matching annotation (TAIR10 GTF/GFF3) into R, then run BUSpaRse::get_velocity_files with L=91, isoform_action="separate", chrs_only=FALSE, and style="Ensembl" to extract intronic intervals and emit cDNA/introns FASTA plus transcript-to-gene mapping files (cDNA_introns.fa, cDNA_tx_to_capture.txt, introns_tx_to_capture.txt, tr2g.tsv). The rationale for isoform_action="separate" is that scKB must distinguish spliced from unspliced features rather than collapsing isoforms. Concatenate the cDNA and intron transcript sequences into one transcriptome FASTA (cDNA_introns.fa) and build the pseudoalignment index with 'kallisto index', saving it (e.g., cDNA_introns_10xv3.idx) in the working directory scKB expects. Verify the index and t2c files are non-empty and loadable, because scKB consumes the index during the 'align raw reads to genome via scKB' step to produce raw spliced and unspliced count matrices that COPILOT then quality-filters.

## Related tools

- **BUSpaRse** (R function get_velocity_files extracts intronic intervals from the GTF and generates the cDNA+intron FASTA and tr2g.tsv transcript-to-gene mappings)
- **kallisto** (Builds the pseudoalignment index ('kallisto index') on the combined cDNA_introns.fa and performs pseudoalignment of reads via scKB)
- **scKB** (Wrapper script of kallisto and bustools that consumes the index to align raw reads and generate spliced and unspliced transcript count matrices)
- **bustools** (Underlying busTools engine used by scKB to correct barcodes/UMIs and produce count matrices from the bus file using tr2g.tsv)
- **BSgenome** (Supplies the genome sequence object required by BUSpaRse::get_velocity_files (e.g., BSgenome.Athaliana.TAIR.TAIR9))
- **COPILOT** (Downstream consumer of the raw spliced and unspliced matrices for quality filtering; motivates producing separate spliced/unspliced counts) — https://github.com/Hsu-Che-Wei/COPILOT

## Examples

```
BUSpaRse::get_velocity_files(X="TAIR10.gtf", L=91, Genome=BSgenome.Athaliana.TAIR.TAIR9, isoform_action="separate", chrs_only=FALSE, style="Ensembl"); system("kallisto index -i cDNA_introns_10xv3.idx cDNA_introns.fa")
```

## Evaluation signals

- 'kallisto index' completes without error on the combined cDNA_introns FASTA and the resulting .idx file is non-empty and loadable by kallisto/scKB
- All four expected files (cDNA_introns.fa, cDNA_tx_to_capture.txt, introns_tx_to_capture.txt, tr2g.tsv) exist in the working directory and are non-empty
- tr2g.tsv maps every transcript in cDNA_introns.fa, including intronic (unspliced) entries tagged distinctly from cDNA entries under isoform_action='separate'
- scKB run on the index yields both a spliced and an unspliced gene-by-cell UMI count matrix rather than a single matrix
- The raw spliced and unspliced matrices are accepted as COPILOT input (copilot() spliced.mtx/unspliced.mtx arguments) for downstream quality filtering

## Limitations

- The required genome may not be available as a BSgenome object; users may need to convert a customized genome into a BSgenome object before running BUSpaRse
- Index construction parameters here are tuned for the Arabidopsis TAIR9 genome/TAIR10 annotation pairing; other species require their own matching genome and annotation versions
- scKB may fail to run properly during the 'align raw reads to genome via scKB' step if the index or t2c files are missing or misnamed in the working directory

## Evidence
