---
name: single-cell-variant-calling-with-molecular-barcodes
description: Use when you have eWGS-discovered somatic SNVs and 10x Chromium BAMs
  from cells known to be wild-type (e.g., healthy donor bone marrow), and need to
  estimate per-site false-positive rates of scRNA-seq mutation calling.
when_to_use_negative:
- The cells of interest harbor the somatic mutations being tested — this skill is
  specifically for negative-control cells (healthy donor / wild-type) to measure FPR,
  not for genotyping tumor cells.
- Input BAMs lack Chromium cell-barcode and UM (UB_tag) tags, e.g., plate-based full-length
  cDNA data from Fluidigm C1/SMART-seq, since duplicate collapsing relies on 10x molecular
  barcodes.
- You need de novo variant discovery rather than targeted detection of pre-called
  somatic SNVs — cb_sniffer (like VarTrix) does not perform variant calling.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: cb_sniffer
  role: Mutation barcode caller that parses CB barcodes from 10x single cell BAMs
    and calls mutant and reference barcodes for a supplied somatic SNV list, with
    -f/-mq/-bq filters
  repo: https://github.com/sridnona/cb_sniffer
- name: pysam
  role: Python library used to parse 10x Chromium BAM files and read the UB_tag molecular
    barcode for PCR duplicate collapsing per molecule
- name: python3
  role: Scripting environment for BAM parsing and tabulating variant-supporting observations
    per cell and per site
- name: sridnona/python3:180925.v1
  role: Docker image providing the pinned python3/pysam execution environment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:d5165ced8853b60c4fa20ed6ab35e97c8c4b26d8b4130d2fc4fa3e26621c09e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sridnona/cb_sniffer
  tools:
  - Docker
  - cb_sniffer
  - pysam
  - python3
  - sridnona/cb_sniffer
  - sridnona/python3:180925.v1
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-variant-calling-with-molecular-barcodes

## Summary

A validation skill for estimating the site-specific false-positive rate of scRNA-seq mutation calling by running the cb_sniffer variant-detection workflow on healthy donor bone marrow cells that should not harbor the tested somatic SNVs. It quantifies per-site FPR across ~8057 normal cells using Chromium molecular barcode (UB_tag) duplicate collapsing.

## When to use

Use when you have eWGS-discovered somatic SNVs and 10x Chromium BAMs from cells known to be wild-type (e.g., healthy donor bone marrow), and need to estimate per-site false-positive rates of scRNA-seq mutation calling.

## When NOT to use

- The cells of interest harbor the somatic mutations being tested — this skill is specifically for negative-control cells (healthy donor / wild-type) to measure FPR, not for genotyping tumor cells.
- Input BAMs lack Chromium cell-barcode and UM (UB_tag) tags, e.g., plate-based full-length cDNA data from Fluidigm C1/SMART-seq, since duplicate collapsing relies on 10x molecular barcodes.
- You need de novo variant discovery rather than targeted detection of pre-called somatic SNVs — cb_sniffer (like VarTrix) does not perform variant calling.

## Inputs

- List of eWGS-discovered somatic SNVs from paired tumor/normal sequencing (variant file with chrm, start, stop, ref, var, gene_name, trv_type columns)
- 10x Chromium BAM files for healthy donor bone marrow cells (8057 cells from four healthy donors), aligned with the 10x cellranger pipeline
- List of good cell barcodes file
- Sample name (UPN) used as output file prefix

## Outputs

- Per-barcode read counts supporting each somatic SNV (_counts_UB.tsv, counts for Chromium molecular barcode UB_tag)
- Tabulation of variant-supporting observations per cell and per site across the four donors
- Per-site false-positive rate across the 8057 healthy cells for each eWGS-discovered variant (reported in Supplementary Data 3, max per-site FPR 0.39%)

## How to apply

Start from the list of eWGS-discovered somatic SNVs obtained by paired tumor/normal enhanced whole-genome sequencing. Parse the 10x Chromium BAM files with python3 and pysam, reading the UB_tag (Chromium molecular barcode) to collapse PCR duplicates per molecule per the 10x bam_tags specification. Run the cb_sniffer filter to count reads supporting each somatic SNV in each healthy cell, using the per-barcode read counts reported as _counts_UB.tsv. Tabulate variant-supporting observations per cell and per site across the four healthy donors, then compute the per-site false-positive rate across the 8057 healthy cells for each variant. Correct application is confirmed when no false positives are found and the maximum per-site FPR is 0.39%, matching Supplementary Data 3.

## Related tools

- **cb_sniffer** (Mutation barcode caller that parses CB barcodes from 10x single cell BAMs and calls mutant and reference barcodes for a supplied somatic SNV list, with -f/-mq/-bq filters) — https://github.com/sridnona/cb_sniffer
- **pysam** (Python library used to parse 10x Chromium BAM files and read the UB_tag molecular barcode for PCR duplicate collapsing per molecule)
- **python3** (Scripting environment for BAM parsing and tabulating variant-supporting observations per cell and per site)
- **sridnona/python3:180925.v1** (Docker image providing the pinned python3/pysam execution environment)

## Examples

```
python3 cb_sniffer.py patient_bam.bam eWGS_variants.tsv good_barcodes.tsv upn -f 0 -mq 0 -bq 1
```

## Evaluation signals

- No false-positive variant calls are detected in any of the 8057 healthy donor bone marrow cells across the four donors.
- Maximum per-site FPR equals 0.39%, matching the values reported in Supplementary Data 3.
- Per-barcode read counts (_counts_UB.tsv) are non-degenerate — UM collapsing via UB_tag yields counts consistent with the 10x bam_tags specification.
- Every tested site is an eWGS-discovered somatic SNV from paired tumor/normal sequencing, so the input variant list matches the discovery set exactly.
- cb_sniffer run parameters (mapq, baseq, reads-per-barcode filter) are recorded and reproducible within the sridnona/python3:180925.v1 Docker image.

## Limitations

- Dropout (transcript and allelic dropout) occurs with most scRNA-seq platforms and reduces mutation-detection sensitivity by a factor of two, so absence of calls in healthy cells does not imply sensitivity in tumor cells.
- It is impossible to determine whether a cell is truly wild-type for a given mutation from scRNA-seq data alone.
- Partial transcript coverage on end-biased platforms such as Chromium limits sensitivity, and coverage drops non-linearly across transcript length, so per-site FPR and detectability vary by site.
- cb_sniffer assumes BAMs aligned with the 10x cellranger pipeline and pre-called variants; it does not perform de novo variant calling.

## Evidence

- [results] Applying the same variant search to 8057 bone marrow cells from four healthy donors yielded no false positives: [evidence span withheld — anchor status: not_found]
- [methods] read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv: [evidence span withheld — anchor status: not_found]
- [readme] cb_sniffer parses CB barcodes from single cell RNA-seq data: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data   Bams aligned using 10x cellranger pipeline"
- [discussion] Dropout reduces the sensitivity of mutation detection: "Dropout (including transcript dropout and allelic dropout) occurs with most scRNA-seq platforms."
- [intro] we first use enhanced whole-genome sequencing of paired tumor/normal samples to discover somatic mutations: "we first use “enhanced” whole-genome sequencing (eWGS) of paired tumor/normal samples to discover somatic mutations, and then focus on detecting those mutations in the scRNA-seq data"
