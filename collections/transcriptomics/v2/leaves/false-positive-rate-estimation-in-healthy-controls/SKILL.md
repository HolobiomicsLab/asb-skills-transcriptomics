---
name: false-positive-rate-estimation-in-healthy-controls
description: 'Use when validating a scRNA-seq mutation-calling pipeline: apply it
  to cells known to lack the somatic variants (e.g., 8057 bone marrow cells from four
  healthy donors) to measure per-site false-positive rates and confirm no spurious
  calls in wild-type cells.'
when_to_use_negative:
- You are genotyping cells that may genuinely harbor the somatic mutations (e.g.,
  AML tumor samples) — this skill estimates the null FPR in mutation-negative cells
  only.
- The BAM data are not from a 10x Chromium pipeline with CB/UB molecular barcodes,
  since cb_sniffer and UB-tag duplicate collapse depend on cellranger tag conventions.
- No independently discovered variant set (e.g., eWGS from paired tumor/normal) is
  available — scRNA-seq-derived variant calls have a high noise-driven false-positive
  rate and cannot serve as the trusted query set.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: cb_sniffer
  role: Mutation barcode caller that counts mutant and reference read support per
    cell barcode for each candidate SNV in the healthy donor BAMs
  repo: https://github.com/sridnona/cb_sniffer
- name: pysam
  role: Python library used to parse the 10x Chromium BAM files and read the UB_tag
    molecular barcodes for PCR duplicate collapse
- name: python3
  role: Scripting environment (in Docker image sridnona/python3:180925.v1) for parsing
    BAMs and computing per-site FPRs
- name: Docker
  role: Containerization of the analysis environment (image sridnona/python3:180925.v1)
- name: sridnona/python3:180925.v1
  role: Docker image providing python3 and pysam for BAM parsing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:70cd9a929bb26d3ce5964a85f7bd35c7921cfbc189840bd041a486f7ec6e5646
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
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# false-positive-rate-estimation-in-healthy-controls

## Summary

Estimates the site-specific false-positive rate of scRNA-seq somatic SNV calling by applying the same variant-detection approach (cb_sniffer on 10x Chromium BAMs with UB-tag duplicate collapse) to healthy donor cells that should not carry the tested variants, providing a null/control benchmark for the mutation-calling method.

## When to use

Use when validating a scRNA-seq mutation-calling pipeline: apply it to cells known to lack the somatic variants (e.g., 8057 bone marrow cells from four healthy donors) to measure per-site false-positive rates and confirm no spurious calls in wild-type cells.

## When NOT to use

- You are genotyping cells that may genuinely harbor the somatic mutations (e.g., AML tumor samples) — this skill estimates the null FPR in mutation-negative cells only.
- The BAM data are not from a 10x Chromium pipeline with CB/UB molecular barcodes, since cb_sniffer and UB-tag duplicate collapse depend on cellranger tag conventions.
- No independently discovered variant set (e.g., eWGS from paired tumor/normal) is available — scRNA-seq-derived variant calls have a high noise-driven false-positive rate and cannot serve as the trusted query set.

## Inputs

- List of eWGS-discovered somatic SNVs (chrm/start/stop/ref/var/gene_name/trv_type variant file with header)
- 10x Chromium single-cell BAM files for healthy donor cells (aligned with the 10x cellranger pipeline)
- Cell barcodes file (list of good barcodes)
- UB_tag (Chromium molecular barcode) per-barcode read counts (_counts_UB.tsv)

## Outputs

- Per-site false-positive rate across the 8057 healthy cells for each eWGS-discovered variant
- Tabulation of variant-supporting observations per cell and per site across the four donors
- Variant-supporting barcode read counts from cb_sniffer output files

## How to apply

Load the list of eWGS-discovered somatic SNVs found from paired tumor/normal sequencing as the query variant set. Parse the 10x Chromium BAM files (here, 8057 bone marrow cells from four healthy donors) with python3/pysam in the sridnona/python3:180925.v1 Docker image, reading the UB_tag Chromium molecular barcode to collapse PCR duplicates per molecule. Run cb_sniffer to count reads supporting each somatic SNV in each healthy cell, using per-barcode read counts reported in _counts_UB.tsv. Tabulate variant-supporting observations per cell and per site, then compute the per-site false-positive rate across all healthy cells for each eWGS-discovered variant. The result is treated as a negative-control FPR; in the source study this produced no false positives and a maximum per-site FPR of 0.39% (Supplementary Data 3).

## Related tools

- **cb_sniffer** (Mutation barcode caller that counts mutant and reference read support per cell barcode for each candidate SNV in the healthy donor BAMs) — https://github.com/sridnona/cb_sniffer
- **pysam** (Python library used to parse the 10x Chromium BAM files and read the UB_tag molecular barcodes for PCR duplicate collapse)
- **python3** (Scripting environment (in Docker image sridnona/python3:180925.v1) for parsing BAMs and computing per-site FPRs)
- **Docker** (Containerization of the analysis environment (image sridnona/python3:180925.v1))
- **sridnona/python3:180925.v1** (Docker image providing python3 and pysam for BAM parsing)

## Examples

```
python3 cb_sniffer.py healthy_donor_bamfile.bam eWGS_variants.tsv good_barcodes.tsv upn
```

## Evaluation signals

- Zero (or near-zero) variant-supporting observations across all 8057 healthy donor cells for the tested eWGS-discovered variants
- Maximum per-site FPR equals 0.39%, matching the values reported in Supplementary Data 3
- No false-positive calls of the tested somatic variants are recorded in any healthy cell after UB-tag duplicate collapse
- cb_sniffer outputs (_counts_UB.tsv-derived per-barcode counts) reconcile with pysam UB_tag read totals per molecule
- Variant-supporting counts are consistent across the four healthy donors rather than concentrated in a single sample

## Limitations

- A zero false-positive result in healthy cells does not guarantee sensitivity; dropout (transcript and allelic) reduces the sensitivity of mutation detection, making it impossible to determine whether a cell is truly wild-type for a given mutation.
- The per-site FPR depends on transcript coverage: coverage drops non-linearly across transcript length on end-biased Chromium platforms, so some sites may be more prone to noise than others.
- Noise inherent in reverse transcription makes calling variants de novo from RNA-seq unreliable — the FPR estimate only applies to a trusted, independently discovered (eWGS) variant set.

## Evidence

- [results] Applying the same variant search to 8057 bone marrow cells from four healthy donors yielded no false positives: [evidence span withheld — anchor status: not_found]
- [intro] we first use “enhanced” whole-genome sequencing (eWGS) of paired tumor/normal samples to discover somatic mutations: "we first use “enhanced” whole-genome sequencing (eWGS) of paired tumor/normal samples to discover somatic mutations, and then focus on detecting those mutations in the scRNA-seq data"
- [readme] Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data   Bams aligned using 10x cellranger pipeline: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data   Bams aligned using 10x cellranger pipeline"
- [readme] Noise inherent in reverse transcription leads to a high false positive rate.: [evidence span withheld — anchor status: not_found]
