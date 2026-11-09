---
name: cds-versus-utr-read-density-enrichment
description: Use when validating RiboSeq alignment output (length-selected BAM/BED
  files) to confirm that ribosome-protected fragments accumulate in coding sequences
  rather than untranslated regions before downstream statistical analysis.
when_to_use_negative:
- Input is not aligned, length-filtered RPF data (e.g. raw FastQ or contaminant-depleted-but-unaligned
  reads) — count enrichment is meaningless before mapping
- The goal is quantifying per-gene expression for differential analysis rather than
  a qualitative ribosome-activity control
- The annotation lacks UTR features, making the CDS-versus-UTR comparison undefined
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: RiboDoc
  role: Pipeline that runs the CDS-overrepresentation QC panel and assembles it into
    the QC report
  repo: https://github.com/equipeGST/RiboDoc
- name: HTSeq
  role: Feature counting of reads overlapping CDS versus UTR annotations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:7707f20d8a1c893afa4e3f61e53f1ed54f5d39d197bc3a093fa6efbcb29ed113
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  repo_url: https://github.com/equipeGST/RiboDoc
  tools:
  - BEDTools
  - HTSeq
  - RiboDoc
  - riboWaltz
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# CDS versus UTR read-density enrichment QC

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A ribosome-profiling quality control that counts footprint reads overlapping annotated CDS versus UTR features (e.g. with HTSeq feature counting) and checks that footprints are overrepresented within the CDS, verifying the data correspond to active ribosome footprints.

## When to use

Use when validating RiboSeq alignment output (length-selected BAM/BED files) to confirm that ribosome-protected fragments accumulate in coding sequences rather than untranslated regions before downstream statistical analysis.

## When NOT to use

- Input is not aligned, length-filtered RPF data (e.g. raw FastQ or contaminant-depleted-but-unaligned reads) — count enrichment is meaningless before mapping
- The goal is quantifying per-gene expression for differential analysis rather than a qualitative ribosome-activity control
- The annotation lacks UTR features, making the CDS-versus-UTR comparison undefined

## Inputs

- Length-split BAM/BED alignment files of RPF-length reads (e.g. *.25-35.bam)
- GFF3 genome annotation containing CDS and UTR features

## Outputs

- CDS versus UTR read counts and CDS overrepresentation enrichment metric/plot
- QC panel incorporated into ribodoc_qc_report.pdf

## How to apply

After alignment and selection of RPF-length reads (default 25-35 nt), count reads overlapping CDS versus UTR features with HTSeq-style feature counting against the GFF3 genome annotation, then compute the CDS overrepresentation enrichment of footprints. This control is one of the qualitative QC panels RiboDoc assembles (together with read-length distribution and metagene periodicity) into the single QC report (ribodoc_qc_report.pdf). Accept the data as active ribosome footprints when footprints are overrepresented within the CDS, the modal read length is ~28 nt, and 3-nt periodicity with CDS-anchored phasing is visible in the metagene. Use it alongside, not instead of, the metagene-periodicity and read-length checks.

## Related tools

- **RiboDoc** (Pipeline that runs the CDS-overrepresentation QC panel and assembles it into the QC report) — https://github.com/equipeGST/RiboDoc
- **HTSeq** (Feature counting of reads overlapping CDS versus UTR annotations)

## Examples

```
docker pull equipegst/ribodoc
```

## Evaluation signals

- CDS overrepresentation enrichment shows footprints accumulated within the CDS relative to UTRs
- Counts are consistent across the length-selected BAM/BED files (modal read length near 28 nt)
- Enrichment panel coheres with the metagene periodicity plot (3-nt phasing anchored at start/stop codons)
- QC figures are assembled in ribodoc_qc_report.pdf alongside read-length distribution and periodicity panels

## Limitations

- The GEO GSE173856 example dataset does not align enough material for the riboWaltz pipeline (TRiP should be selected instead), which can affect upstream QC panels
- This is a qualitative control — RiboDoc complements it with statistical validations (PCA and pairwise Spearman correlation) to assess replicate suitability
- Requires accurate GFF3 annotation; mis-annotated CDS/UTR boundaries will distort the enrichment metric

## Evidence

- [intro] Provide quality controls (CDS enrichment and metagene-periodicity) and statistical validations (PCA and Spearman's correlation) to check that RiboSeq data correspond to active ribosome footprints: "which provides controls (CDS enrichment and metagene-periodicity) for checking that the data correspond to active ribosome footprints"
- [results] Checking overrepresentation of footprints within the CDS: "it is also possible to check the overrepresentation of footprints within the CDS (Fig. 2B)"
- [readme] Validation criteria include CDS overrepresentation: "RiboDoc is designed to perform all classical steps of **ribosome profiling** (RiboSeq) data analysis from the FastQ files to the differential expression analysis with necessary quality controls."
- [methods] Read counting with htseq-count: "Reads Count with [`htseq-count`](https://htseq.readthedocs.io/en/release_0.11.1/count.html#)"
