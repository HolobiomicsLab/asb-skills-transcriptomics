---
name: somatic-snv-validation-across-donor-cohorts
description: Use when you have a panel of eWGS-discovered somatic SNVs from paired
  tumor/normal sequencing and need to show the variant-detection approach does not
  falsely call them in cells known not to harbor them (healthy donor scRNA-seq).
when_to_use_negative:
- The goal is to genotype AML/tumor cells to find cells harboring the mutation — this
  skill estimates the false-positive rate in cells that do not harbor the tested variants
  (healthy donor controls).
- The scRNA-seq data lacks Chromium UB/CB tags (i.e., is not 10x 3'/5' Chromium output),
  since cb_sniffer and the UB_tag duplicate-collapsing strategy are specific to 10x
  Cellranger BAMs.
- You need to identify arbitrary de novo mutations from the RNA data itself — this
  skill only tests previously discovered somatic SNVs, as noise from reverse transcription
  gives a high false-positive rate for de novo calling.
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: python3
  role: Used with pysam to parse 10x Chromium BAM files and read the UB_tag to collapse
    PCR duplicates per molecule
- name: pysam
  role: Parses the BAM files to extract the UB_tag (Chromium molecular barcode) for
    duplicate collapsing before variant counting
- name: cb_sniffer
  role: Mutation barcode caller run on the healthy donor BAMs to count reads supporting
    each eWGS-discovered somatic SNV in each cell, producing per-barcode read counts
    (_counts_UB.tsv)
  repo: https://github.com/sridnona/cb_sniffer
- name: Docker (sridnona/python3:180925.v1)
  role: Container image providing the python3/pysam environment used for BAM parsing
    and variant counting
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:27a090c85c6b102195cb6386de955fe222ffe0ae52bedb4124509d7fb4aa073b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sridnona/cb_sniffer
  tools:
  - Docker
  - Docker (sridnona/python3:180925.v1)
  - cb_sniffer
  - pysam
  - python3
  - sridnona/cb_sniffer
  - sridnona/python3:180925.v1
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# somatic-snv-validation-across-donor-cohorts

## Summary

A negative-control validation skill that applies an scRNA-seq somatic SNV detection workflow (eWGS-discovered variants + cb_sniffer on 10x Chromium BAMs) to healthy donor cells to estimate the per-site false-positive rate. The article found zero false positives across 8057 healthy bone marrow cells from four donors, with a maximum per-site FPR of 0.39%.

## When to use

Use when you have a panel of eWGS-discovered somatic SNVs from paired tumor/normal sequencing and need to show the variant-detection approach does not falsely call them in cells known not to harbor them (healthy donor scRNA-seq).

## When NOT to use

- The goal is to genotype AML/tumor cells to find cells harboring the mutation — this skill estimates the false-positive rate in cells that do not harbor the tested variants (healthy donor controls).
- The scRNA-seq data lacks Chromium UB/CB tags (i.e., is not 10x 3'/5' Chromium output), since cb_sniffer and the UB_tag duplicate-collapsing strategy are specific to 10x Cellranger BAMs.
- You need to identify arbitrary de novo mutations from the RNA data itself — this skill only tests previously discovered somatic SNVs, as noise from reverse transcription gives a high false-positive rate for de novo calling.

## Inputs

- List of eWGS-discovered somatic SNVs from paired tumor/normal sequencing (variant file with header: chrm, start, stop, ref, var, gene_name, trv_type)
- 10x Chromium BAM files (Cellranger-aligned) for healthy donor bone marrow cells (8057 cells from four donors)
- File of good cell barcodes
- _counts_UB.tsv read counts for the Chromium molecular barcode (UB_tag)

## Outputs

- Per-barcode mutant and reference read counts per variant (_counts_UB.tsv)
- Per-cell and per-site tabulation of false-positive variant observations across the four donors
- Per-site false-positive rate across 8057 healthy cells for each eWGS-discovered variant (maximum observed: 0.39%)

## How to apply

Load the list of enhanced whole-genome sequencing (eWGS)-discovered somatic SNVs discovered from paired tumor/normal samples as the variant input to be tested. Parse the 10x Chromium BAM files (Cellranger-aligned) with python3 and pysam inside the sridnona/python3:180925.v1 Docker image, reading the UB_tag (Chromium molecular barcode) to collapse PCR duplicates per molecule. Run cb_sniffer with a variant file (chrm, start, stop, ref, var, gene_name, trv_type), a good-barcodes list, and sample name, reporting per-barcode read counts as _counts_UB.tsv. Tabulate variant-supporting observations per cell and per site across the four healthy donors, then compute the per-site false-positive rate across all 8057 healthy cells for each eWGS-discovered variant. The rationale is that dropout (transcript and allelic) and RT noise could create spurious variant calls, so a zero-call result in wild-type cells bounds the assay's error rate before interpreting AML subclone genotypes.

## Related tools

- **python3** (Used with pysam to parse 10x Chromium BAM files and read the UB_tag to collapse PCR duplicates per molecule)
- **pysam** (Parses the BAM files to extract the UB_tag (Chromium molecular barcode) for duplicate collapsing before variant counting)
- **cb_sniffer** (Mutation barcode caller run on the healthy donor BAMs to count reads supporting each eWGS-discovered somatic SNV in each cell, producing per-barcode read counts (_counts_UB.tsv)) — https://github.com/sridnona/cb_sniffer
- **Docker (sridnona/python3:180925.v1)** (Container image providing the python3/pysam environment used for BAM parsing and variant counting)

## Examples

```
python3 cb_sniffer.py bam_file variant_file barcodes upn -f 0 -mq 0 -bq 1
```

## Evaluation signals

- Zero false-positive variant calls across all 8057 healthy donor bone marrow cells from the four donors
- Maximum per-site false-positive rate equals 0.39%, matching the values reported in Supplementary Data 3
- Per-cell/per-site tallies show which (if any) sites produce false-positive calls in healthy cells — expected: none
- UB_tag-based PCR duplicate collapsing is applied, so counted observations are per molecule rather than per read
- cb_sniffer output _counts_UB.tsv contains per-barcode read counts consistent with the Chromium CB/UB tag specification

## Limitations

- As a result, it is impossible to determine whether a cell is truly wild-type for a given mutation; the FPR estimate is a lower bound on spurious calls in a genuinely wild-type population.
- Dropout (transcript and allelic) occurs with most scRNA-seq platforms, reducing sensitivity of mutation detection by a factor of two — a zero-FPR result does not guarantee high sensitivity.
- Noise inherent in reverse transcription leads to a high false-positive rate for de novo variant discovery; this skill only validates a pre-called, eWGS-discovered variant set.

## Evidence

- [methods] read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [readme] Mutation barcode caller, calls mutant and ref barcodes from 10x single cell data: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data"
- [discussion] It is impossible to determine whether a cell is truly wild-type for a given mutation: "As a result, it is impossible to determine whether a cell is truly wild-type for a given mutation."
- [intro] Detecting genetic variants in scRNA-seq reads is difficult due to low transcript abundance, allelic dropout, and incomplete transcript coverage: "Detecting genetic variants in scRNA-seq reads is difficult due to the low transcript abundance, allelic dropout, and incomplete transcript coverage inherent to this platform"
- [readme] Noise inherent in reverse transcription leads to a high false positive rate: [evidence span withheld — anchor status: external_doc]
