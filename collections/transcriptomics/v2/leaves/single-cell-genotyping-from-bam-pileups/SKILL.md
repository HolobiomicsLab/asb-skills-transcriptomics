---
name: single-cell-genotyping-from-bam-pileups
description: Use when a list of somatic SNVs from paired tumor/normal eWGS must be
  detected per-cell in 10x 5' scRNA-seq BAMs (CB/UB_tag barcodes), producing per-sample
  mutant-cell counts.
when_to_use_negative:
- No pre-called somatic variant list exists — cb_sniffer/VarTrix assign previously
  defined variants, they do not call variants de novo (VarTrix 'does not perform variant
  calling')
- The goal is determining which cells are truly wild-type for a mutation — dropout
  makes this impossible; only mutant/unknown labels are supportable
- Multi-allelic sites are the target — VarTrix ignores all multi-allelic sites, leaving
  empty matrix values
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: cb_sniffer
  role: Parses CB barcodes from 10x Cell Ranger BAMs and calls mutant vs ref barcodes
    at eWGS-discovered variant positions, honoring MAPQ/base-quality/read-count filters
  repo: https://github.com/sridnona/cb_sniffer
- name: VarTrix
  role: Alternative/complementary tool that assigns single cells to pre-called variants
    using Smith-Waterman alignment, outputting a Cell Ranger-format genome matrix
  repo: https://github.com/10XGenomics/vartrix
- name: pysam
  role: Python library used inside the Docker image to perform barcode/UMI-aware pileups
    over the BAM at each variant position
- name: Docker
  role: Provides the reproducible python3/pysam runtime via image sridnona/python3:180925.v1
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:caffcc2984f4dd2c4398f4800680db7fb2accd9f07e36d7cde0e6410215d9358
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sridnona/cb_sniffer
  tools:
  - Docker
  - VarTrix
  - cb_sniffer
  - pysam
  - python3
  - sridnona/python3:180925.v1
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-genotyping-from-bam-pileups

## Summary

Genotype individual cells in 10x Chromium scRNA-seq BAM data at somatic SNVs pre-discovered by enhanced whole-genome sequencing (eWGS), using barcode-aware pileups to label each cell mutant or unknown and count mutant cells per sample.

## When to use

Use when a list of somatic SNVs from paired tumor/normal eWGS must be detected per-cell in 10x 5' scRNA-seq BAMs (CB/UB_tag barcodes), producing per-sample mutant-cell counts.

## When NOT to use

- No pre-called somatic variant list exists — cb_sniffer/VarTrix assign previously defined variants, they do not call variants de novo (VarTrix 'does not perform variant calling')
- The goal is determining which cells are truly wild-type for a mutation — dropout makes this impossible; only mutant/unknown labels are supportable
- Multi-allelic sites are the target — VarTrix ignores all multi-allelic sites, leaving empty matrix values

## Inputs

- 10x Cell Ranger BAM files with CB (cell barcode) and UB_tag (Chromium molecular barcode/UMI) tags, plus BAM index
- Somatic SNV variant file with header: chrm, start, stop, ref, var, gene_name, trv_type
- List of good cell barcodes file
- Sample/UPN name used as output file prefix

## Outputs

- Per-cell per-variant mutant-UMI read counts table (_counts_UB.tsv)
- Per-cell genotype labels: mutant (≥1 variant-containing read/UMI) or unknown
- Per-sample and per-variant mutant-cell counts

## How to apply

Load the eWGS-discovered somatic SNV list (chrm, start, stop, ref, var, gene_name, trv_type format) together with the Cell Ranger BAM files and a list of good cell barcodes. Run cb_sniffer in a containerized python3/pysam environment (Docker image sridnona/python3:180925.v1): for each variant position, perform a barcode-aware pileup over reads with valid CB and UB_tag (Chromium molecular barcode), honoring MAPQ and base-quality filters (defaults -mq 0, -bq 1, optional -f read-count filter per barcode). Collapse reads by UMI within each cell and classify each cell/variant pair as mutant if at least one mutant-supporting read/UMI is present, otherwise 'unknown' — never wild-type, because transcript and allelic dropout make a true wild-type call impossible. Write per-cell per-variant mutant-UMI read counts (e.g. _counts_UB.tsv) and aggregate mutant cells per sample and per variant; the paper's validation range is hundreds to thousands of mutant cells per AML case (~22% of cells on average, 396–8200 across five AML samples).

## Related tools

- **cb_sniffer** (Parses CB barcodes from 10x Cell Ranger BAMs and calls mutant vs ref barcodes at eWGS-discovered variant positions, honoring MAPQ/base-quality/read-count filters) — https://github.com/sridnona/cb_sniffer
- **VarTrix** (Alternative/complementary tool that assigns single cells to pre-called variants using Smith-Waterman alignment, outputting a Cell Ranger-format genome matrix) — https://github.com/10XGenomics/vartrix
- **pysam** (Python library used inside the Docker image to perform barcode/UMI-aware pileups over the BAM at each variant position)
- **Docker** (Provides the reproducible python3/pysam runtime via image sridnona/python3:180925.v1)

## Examples

```
python3 cb_sniffer.py sample_5prime.bam somatic_snvs.tsv good_barcodes.tsv UPN1
```

## Evaluation signals

- Per-sample mutant-cell counts fall in the reported range of hundreds to thousands (396 to 8200 across five AML cases; e.g. 669, 604, 8200, 2003, 354), ~22% of total cells on average
- Average mutant cells per variant ≈ 49
- Every cell/variant pair resolves to 'mutant' or 'unknown' — no cell should be labeled wild-type, since dropout prevents confident wild-type calls
- Output _counts_UB.tsv contains per-cell per-variant read counts for the Chromium molecular barcode (UB_tag)
- Only reads with valid CB and UB_tag 10x BAM tag conventions contribute to pileups

## Limitations

- Dropout (transcript and allelic) occurs on most scRNA-seq platforms, halves mutation-detection sensitivity, and makes it impossible to determine whether a cell is truly wild-type for a given mutation
- Partial transcript coverage on end-biased platforms like Chromium limits variant-detection sensitivity, and coverage drops non-linearly across transcript length, so some variants are much more easily detectable than others
- Low transcript abundance, allelic dropout, and incomplete transcript coverage make variant detection in scRNA-seq reads inherently difficult; utility depends on the sample's mutational burden and will likely perform better in higher-mutation-burden tumors than AML

## Evidence

- [readme] cb_sniffer barcode caller usage and parameters: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data Bams aligned using 10x cellranger pipeline"
- [readme] cb_sniffer MAPQ/base-quality filter options: "Skip read with mapq smaller than default : 0 -bq BASEQ, --baseq BASEQ Skip bases with base quality less than default : 1"
- [readme] Variant file format: "chrm	start	stop	ref	var	gene_name	trv_type 19	4xxx	4xxx	G	A	gene_name	silent/misense/frame_shift"
- [methods] cb_sniffer output counts: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [abstract] Mutant cell counts per AML sample: "Applying this approach to five cryopreserved AML samples, we identify hundreds to thousands of cells containing tumor-specific mutations in each case"
- [discussion] Dropout limits sensitivity and prevents wild-type calls: "Dropout (including transcript dropout and allelic dropout) occurs with most scRNA-seq platforms."
- [readme] VarTrix does not call variants: [evidence span withheld — anchor status: external_doc]
