---
name: molecular-barcode-duplicate-collapsing
description: Use when calling somatic SNVs from 10x Chromium scRNA-seq BAMs (e.g.,
  with cb_sniffer) and per-cell variant read counts must be deduplicated at the molecule
  level using the UB_tag before computing per-site false-positive rates.
when_to_use_negative:
- The scRNA-seq data come from full-length plate-based platforms (e.g., Fluidigm C1/SMART-seq)
  that lack 10x Chromium UB_tag molecular barcodes, so molecule-level deduplication
  is not possible.
- The BAM was not produced by the Cell Ranger pipeline and does not carry the UB_tag/CB
  tags expected by cb_sniffer and the 10x bam_tags specification.
- The goal is de novo variant discovery from the RNA data itself — this skill only
  genotypes pre-called somatic variants; discovery is done upstream with eWGS of paired
  tumor/normal samples.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: python3
  role: Scripting language used to parse the 10x Chromium BAM files and read UB_tag
    molecular barcodes
- name: pysam
  role: Python library for reading BAM alignments and extracting the UB_tag per read
    during duplicate collapsing
- name: sridnona/python3:180925.v1
  role: Docker image providing the python3/pysam environment for BAM parsing
- name: cb_sniffer
  role: Mutation barcode caller that uses UB read counts (_counts_UB.tsv) to call
    mutant and reference barcodes per cell from 10x data
  repo: https://github.com/sridnona/cb_sniffer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:2bae552e222c9e71e37cfc4193adde728fe88dbf646a9b5c9244dd5adb9dcc7b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# molecular-barcode-duplicate-collapsing

## Summary

Collapses PCR duplicates in 10x Chromium single-cell data by counting reads per Chromium molecular barcode (UB_tag), so that variant-supporting evidence in each cell reflects unique molecules rather than PCR amplification artifacts.

## When to use

Use when calling somatic SNVs from 10x Chromium scRNA-seq BAMs (e.g., with cb_sniffer) and per-cell variant read counts must be deduplicated at the molecule level using the UB_tag before computing per-site false-positive rates.

## When NOT to use

- The scRNA-seq data come from full-length plate-based platforms (e.g., Fluidigm C1/SMART-seq) that lack 10x Chromium UB_tag molecular barcodes, so molecule-level deduplication is not possible.
- The BAM was not produced by the Cell Ranger pipeline and does not carry the UB_tag/CB tags expected by cb_sniffer and the 10x bam_tags specification.
- The goal is de novo variant discovery from the RNA data itself — this skill only genotypes pre-called somatic variants; discovery is done upstream with eWGS of paired tumor/normal samples.

## Inputs

- 10x Chromium BAM file aligned by the Cell Ranger pipeline with UB_tag and CB barcodes
- Variant file of eWGS-discovered somatic SNVs with header (chrm, start, stop, ref, var, gene_name, trv_type)
- File of good cell barcodes (one per line)
- Sample/prefix name (upn)

## Outputs

- _counts_UB.tsv: per-cell-barcode read counts for Chromium molecular barcodes (UB_tag)
- Per-cell, per-site variant-supporting molecule counts
- Per-site false-positive rate across healthy donor cells (maximum reported: 0.39%)

## How to apply

Parse the 10x Chromium BAM files (aligned with the Cell Ranger pipeline) using python3 and pysam inside the sridnona/python3:180925.v1 Docker image, reading the UB_tag (Chromium molecular barcode) to collapse PCR duplicates per molecule, per the 10x bam_tags specification. Run cb_sniffer over the list of eWGS-discovered somatic SNVs, using the per-barcode UB read counts written to _counts_UB.tsv so each cell barcode contributes at most one observation per molecule at a variant site. In the healthy-donor control workflow, apply the identical variant search to 8057 bone marrow cells from four healthy donors, tabulating variant-supporting observations per cell and per site. Decision point: any variant-supporting molecule in wild-type cells is a false positive; in this article's validation, none were found and the maximum per-site FPR across healthy cells was 0.39%, matching Supplementary Data 3.

## Related tools

- **python3** (Scripting language used to parse the 10x Chromium BAM files and read UB_tag molecular barcodes)
- **pysam** (Python library for reading BAM alignments and extracting the UB_tag per read during duplicate collapsing)
- **sridnona/python3:180925.v1** (Docker image providing the python3/pysam environment for BAM parsing)
- **cb_sniffer** (Mutation barcode caller that uses UB read counts (_counts_UB.tsv) to call mutant and reference barcodes per cell from 10x data) — https://github.com/sridnona/cb_sniffer

## Examples

```
python3 cb_sniffer.py possorted_genome_bam.bam variants.tsv barcodes.tsv UPN1
```

## Evaluation signals

- Applying the same variant search to 8057 bone marrow cells from four healthy donors yields no false positives.
- The maximum per-site false-positive rate across healthy cells is 0.39%, matching the values reported in Supplementary Data 3.
- _counts_UB.tsv is produced with per-barcode counts for the Chromium molecular barcode (UB_tag) as specified by the 10x bam_tags specification.
- Collapsed per-barcode UB counts are never larger than the raw per-barcode read counts at a given variant site.
- cb_sniffer runs successfully on Cell Ranger BAMs with the variant file and good-barcodes file, using the -f/-mq/-bq quality filters.

## Limitations

- It is impossible to determine whether a cell is truly wild-type for a given mutation, so absence of variant calls cannot be distinguished from dropout-driven sensitivity loss.
- Dropout (including transcript dropout and allelic dropout) occurs with most scRNA-seq platforms and reduces the sensitivity of mutation detection by a factor of two.
- Partial transcript coverage is specific to end-biased platforms such as Chromium and limits the sensitivity of variant detection, with coverage dropping non-linearly across transcript length so some variants are more easily detectable than others.

## Evidence

- [methods] read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv: [evidence span withheld — anchor status: not_found]
- [methods] Parse the 10x Chromium BAM files ... reading the UB_tag (Chromium molecular barcode) to collapse PCR duplicates per molecule, per the 10x bam_tags specification.: [evidence span withheld — anchor status: not_found]
- [readme] Mutation barcode caller, calls mutant and ref barcodes from 10x single cell data Bams aligned using 10x cellranger pipeline: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data   Bams aligned using 10x cellranger pipeline"
- [results] Applying the same variant search to 8057 bone marrow cells from four healthy donors yielded no false positives: [evidence span withheld — anchor status: not_found]
- [discussion] Dropout (including transcript dropout and allelic dropout) occurs with most scRNA-seq platforms.: "Dropout (including transcript dropout and allelic dropout) occurs with most scRNA-seq platforms."
