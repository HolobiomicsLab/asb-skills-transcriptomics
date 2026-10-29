---
name: scrna-seq-technology-parameter-extraction
description: Use when building a reference table of supported single-cell RNA-seq
  chemistries and their barcode/UMI read-layout settings, or when verifying which
  technology a FASTQ input corresponds to.
when_to_use_negative:
- The goal is to actually process scRNA-seq FASTQ data with UniverSC rather than catalogue
  its per-technology settings
- The technology is a 5' scRNA-seq chemistry other than 10x Chromium, which UniverSC
  does not support at the time of writing
- The platform is not UMI-based, since UniverSC only supports UMI-based platforms
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: UniverSC
  role: Source of the pre-set per-technology parameters (barcode/UMI lengths and configuration
    flags) catalogued in Table 1; the tool auto-detects the input chemistry using
    these settings
  repo: https://github.com/minoda-lab/universc
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34681-z
    title: A flexible cross-platform single-cell data processing pipeline
  source_content_hash: sha256:0191dc8e40611b9126598d5d228b6ee6969ecfbe3d10891a655da79960eaa926
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/minoda-lab/universc
  tools:
  - UniverSC
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# scrna-seq-technology-parameter-extraction

## Summary

Extracts the pre-set per-technology processing parameters (cell barcode length, UMI length, and special-case configuration flags) that UniverSC uses for chemistry detection, reproducing Table 1 of the UniverSC paper as a flat per-technology table.

## When to use

Use when building a reference table of supported single-cell RNA-seq chemistries and their barcode/UMI read-layout settings, or when verifying which technology a FASTQ input corresponds to.

## When NOT to use

- The goal is to actually process scRNA-seq FASTQ data with UniverSC rather than catalogue its per-technology settings
- The technology is a 5' scRNA-seq chemistry other than 10x Chromium, which UniverSC does not support at the time of writing
- The platform is not UMI-based, since UniverSC only supports UMI-based platforms

## Inputs

- UniverSC article Table 1 (pre-set parameters per technology, with footnotes for special cases)
- UniverSC GitHub repository documentation (https://github.com/minoda-lab/universc) listing supported technologies

## Outputs

- technology_parameters.csv — one row per supported technology with eight columns: technology name, platform/vendor, cell barcode length, UMI length, UMI-before-barcode flag, Read1/Read2-inverted flag, segmented-barcodes flag, dual-index flag

## How to apply

Locate Table 1 of the UniverSC article (Battenberg et al., Nature Communications 2022, DOI 10.1038/s41467-022-34681-z), which tabulates pre-set settings per technology used to detect the input chemistry. For each of the 40 supported UMI-based technologies, extract the technology name, platform/vendor, cell barcode length, and UMI length (e.g., 10x-v1: 14 bp barcode/10 bp UMI; 10x-v2: 16 bp barcode/10 bp UMI). Also extract the footnoted per-technology configuration flags as booleans: whether the UMI precedes the barcode, whether Read 1/Read 2 are inverted relative to the Chromium default, whether barcodes are segmented, and whether the technology uses dual indexes. Write everything to a single flat table with one row per technology and these eight fields as columns, saved as technology_parameters.csv. Validate that the table has exactly 40 rows and that entries conform to the published Table 1 values.

## Related tools

- **UniverSC** (Source of the pre-set per-technology parameters (barcode/UMI lengths and configuration flags) catalogued in Table 1; the tool auto-detects the input chemistry using these settings) — https://github.com/minoda-lab/universc

## Examples

```
python scripts/extract_table1.py --article-table Table1 --fields technology,platform,barcode_length,umi_length,umi_first,read_inverted,segmented_barcodes,dual_index --out technology_parameters.csv
```

## Evaluation signals

- The output table contains exactly 40 rows, one per supported technology
- All eight parameter columns (name, platform, barcode length, UMI length, and the four boolean flags) are present and populated for every row
- Spot-checked entries match published Table 1 values (e.g., 10x-v1: 14 bp barcode/10 bp UMI; 10x-v2: 16 bp barcode/10 bp UMI)
- Boolean flag columns encode footnoted special cases (UMI-before-barcode, inverted Read 1/Read 2, segmented barcodes, dual indexes) consistently as booleans

## Limitations

- Only 3' scRNA-seq UMI-based technologies are covered; technologies for 5' scRNA-seq other than 10x Chromium are not supported by UniverSC at the time of writing
- UniverSC wraps Cell Ranger 3.0.2 (due to licensing), so parameter semantics are tied to that Cell Ranger version
- The extracted flags reflect footnoted special cases in Table 1; misreading footnotes can silently invert a flag column

## Evidence

- [methods] Table 1 enumerates the pre-set settings per technology, giving barcode and UMI lengths (e.g., 10x-v1: 14 bp barcode/10 bp UMI; 10x-v2: 16 bp barcode/10 bp UMI): [evidence span withheld — anchor status: not_found]
- [methods] The chemistry appropriate for each single-cell technology for 3' scRNA-seq is determined automatically: [evidence span withheld — anchor status: not_found]
- [abstract] UniverSC is a universal single-cell RNA-seq data processing tool that supports any unique molecular identifier-based platform: "a universal single-cell RNA-seq data processing tool that supports any unique molecular identifier-based platform"
- [readme] Single-cell processing across technologies — UniverSC supports any UMI-based platform (README): [evidence span withheld — anchor status: external_doc]
