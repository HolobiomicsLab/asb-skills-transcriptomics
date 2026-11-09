---
name: cell-barcode-posterior-error-correction
description: Use when raw 10x Genomics droplet scRNA-seq FASTQ reads must have their
  16 bp cell barcodes error-corrected (or discarded) prior to UMI filtering and building
  a gene-barcode count matrix.
when_to_use_negative:
- Input is already a filtered gene-cell-barcode count matrix (e.g. a matrix ready
  for Seurat log-transformation) — barcode correction is a pre-counting step
- Data were not generated on the 10x Genomics GemCode/droplet 3' counting platform,
  so barcode structure and error model do not apply
- Goal is UMI error correction or PCR-duplicate marking rather than cell-barcode error
  correction
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing (including posterior-probability
    barcode correction) and single-cell 3' gene counting
  repo: 10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 (cDNA insert) to hg19 before barcode/UMI filtering; only MAPQ=255
    alignments are retained
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:9ac3b17399b2d94f71e086ce96a938ccb9aa2cbff79ada8290deb6911d08f3a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - SRA SRP073767
  - STAR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cell-barcode-posterior-error-correction

## Summary

Corrects droplet single-cell RNA-seq cell barcodes using a Bayesian posterior-probability rule (threshold ≥0.975) from the 10x Genomics Cell Ranger pipeline, so that sequencing errors in the barcode do not inflate apparent cell counts before UMI collapse and gene-barcode matrix generation.

## When to use

Use when raw 10x Genomics droplet scRNA-seq FASTQ reads must have their 16 bp cell barcodes error-corrected (or discarded) prior to UMI filtering and building a gene-barcode count matrix.

## When NOT to use

- Input is already a filtered gene-cell-barcode count matrix (e.g. a matrix ready for Seurat log-transformation) — barcode correction is a pre-counting step
- Data were not generated on the 10x Genomics GemCode/droplet 3' counting platform, so barcode structure and error model do not apply
- Goal is UMI error correction or PCR-duplicate marking rather than cell-barcode error correction

## Inputs

- Raw single-cell RNA-seq FASTQ files from SRA accession SRP073767 (Read1 cDNA insert, Read2 cell barcode + UMI)
- Cell barcode whitelist / expected recovered cell count for candidate selection
- hg19 human reference genome index for STAR alignment

## Outputs

- Reads with corrected or discarded cell barcode assignments
- Gene-by-cell-barcode UMI count matrix (gene_barcode_matrix.csv) built from non-PCR-duplicate, valid-barcode/UMI, MAPQ=255 reads

## How to apply

After aligning Read1 (the cDNA insert) to the hg19 reference genome with STAR, compute the posterior probability for each candidate cell barcode given the observed barcode and its base qualities; if the posterior probability for any candidate is at least 0.975, correct the read's barcode to the candidate with the highest posterior probability, otherwise discard the read. Restrict candidate whitelisted barcodes first: all top barcodes within the same order of magnitude (>10% of the top nth barcode, where n is 1% of the expected recovered cell count) are considered cell barcodes. Downstream, only reads surviving barcode correction with valid UMIs (quality score >10, not homopolymers, 1-Hamming-corrected to the more abundant neighboring UMI for the same cell barcode and gene) and confident MAPQ=255 alignments are counted. The rationale is that barcode errors from sequencing/GEM processing would otherwise create spurious cell barcodes and inflate duplicate counts in the gene-barcode matrix.

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing (including posterior-probability barcode correction) and single-cell 3' gene counting) — 10XGenomics/cellranger
- **STAR** (Aligns Read1 (cDNA insert) to hg19 before barcode/UMI filtering; only MAPQ=255 alignments are retained)

## Evaluation signals

- Every read retained in the matrix must come from a barcode that was corrected to a whitelisted candidate with posterior probability ≥0.975, or was unambiguous originally
- The gene-barcode matrix must contain counts derived only from confidently mapped (MAPQ=255), non-PCR-duplicate reads with valid barcodes and UMIs
- Retained UMI counts must reflect 1-Hamming-distance collapse toward the UMI with more reads per (cell barcode, gene) pair
- No homopolymer UMIs or UMIs with quality score ≤10 should contribute to final counts
- Number of distinct valid cell barcodes should correspond to the whitelist-selection rule (>10% of the top nth barcode, n = 1% of expected recovered cell count), not raw barcode diversity

## Limitations

- Reads whose candidate barcode posterior probability falls below 0.975 are discarded entirely, which loses data for low-quality barcode reads
- Some earlier libraries used a 5 nt UMI and 5 bp Read2 instead, so the barcode/UMI read structure is not uniform across all deposited libraries
- Cell-count estimates carry a 15–20% error, which can affect the expected-cell-count parameter used in the whitelist-selection rule

## Evidence

- [discussion] If the posterior probability for any candidate barcode is at least 0.975, then the barcode is corrected to the candidate barcode with the highest posterior probability.: "If the posterior probability for any candidate barcode is at least 0.975, then the barcode is corrected to the candidate barcode with the highest posterior probability."
- [discussion] All top barcodes within the same order of magnitude (>10% of the top nth barcode, where n is 1% of the expected recovered cell count) were considered cell barcodes.: "All top barcodes within the same order of magnitude (>10% of the top nth barcode, where n is 1% of the expected recovered cell count) were considered cell barcodes."
- [discussion] Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix.: "Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix."
- [readme] Cell Ranger is a set of analysis pipelines that perform sample demultiplexing, barcode processing, single cell 3' and 5' gene counting: "Cell Ranger is a set of analysis pipelines that perform sample demultiplexing, barcode processing, single cell 3' and 5' gene counting"
