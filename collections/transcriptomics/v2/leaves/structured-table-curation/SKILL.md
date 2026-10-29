---
name: structured-table-curation
description: Use when a published article tabulates per-entity configuration values
  (e.g., per-technology cell barcode length, UMI length, and special-case flags) that
  must be reproduced as a structured, row-per-item table for downstream automation.
when_to_use_negative:
- The input is narrative text describing parameters without a per-entity tabulation,
  so no row-per-technology table can be reliably built
- The target tool cannot detect input chemistry from pre-set parameters — UniverSC
  determines the appropriate 3' scRNA-seq chemistry automatically, so manual parameter
  lookup is unnecessary for supported chemistries
- The technology is a 5' scRNA-seq platform other than Chromium, which is outside
  UniverSC's supported scope
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: UniverSC
  role: The tool whose Table 1 pre-set per-technology parameters (barcode length,
    UMI length, special-case flags) are curated into the structured table
  repo: https://github.com/minoda-lab/universc
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34681-z
    title: A flexible cross-platform single-cell data processing pipeline
  source_content_hash: sha256:4fe5c7753d0ddbb7d862107efd6e9c44455a1de146eba9b45d014e71ffa1723e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/minoda-lab/universc
  tools:
  - UniverSC
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# structured-table-curation

## Summary

Curates a published parameter catalogue—here, UniverSC's Table 1 of 40 pre-set single-cell technology processing settings—into a flat, machine-readable CSV with one row per item and validated columns.

## When to use

Use when a published article tabulates per-entity configuration values (e.g., per-technology cell barcode length, UMI length, and special-case flags) that must be reproduced as a structured, row-per-item table for downstream automation.

## When NOT to use

- The input is narrative text describing parameters without a per-entity tabulation, so no row-per-technology table can be reliably built
- The target tool cannot detect input chemistry from pre-set parameters — UniverSC determines the appropriate 3' scRNA-seq chemistry automatically, so manual parameter lookup is unnecessary for supported chemistries
- The technology is a 5' scRNA-seq platform other than Chromium, which is outside UniverSC's supported scope

## Inputs

- Table 1 of the UniverSC article (per-technology pre-set parameter catalogue)
- Per-technology entries: name, platform/vendor, barcode length, UMI length, special-case flags

## Outputs

- technology_parameters.csv — a flat table with one row per technology (40 rows) and eight columns (technology name, platform/vendor, barcode length, UMI length, UMI-precedes-barcode flag, read-inversion flag, segmented-barcodes flag, dual-index flag)

## How to apply

Locate the source table (Table 1 of the UniverSC Nature Communications article, DOI 10.1038/s41467-022-34681-z), which tabulates the pre-set settings per technology that the tool uses to detect the input chemistry. For each of the 40 supported technologies, extract the technology name, platform/vendor, cell barcode length, and UMI length (e.g., 10x-v1: 14 bp barcode/10 bp UMI; 10x-v2: 16 bp barcode/10 bp UMI). Additionally extract per-technology configuration flags as boolean columns: whether the UMI precedes the barcode, whether Read 1/Read 2 are inverted relative to the Chromium default, whether barcodes are segmented, and whether the technology uses dual indexes. Write all eight fields into a single flat table with one row per technology, saved as technology_parameters.csv, then validate that the table contains exactly 40 rows and spot-check entries against the published values.

## Related tools

- **UniverSC** (The tool whose Table 1 pre-set per-technology parameters (barcode length, UMI length, special-case flags) are curated into the structured table) — https://github.com/minoda-lab/universc

## Evaluation signals

- Row-count invariant: technology_parameters.csv contains exactly 40 rows, one per supported technology
- Schema check: all eight parameter columns (technology name, platform/vendor, barcode length, UMI length, and four boolean flags) are present and populated for every row
- Spot-check example entries against published Table 1 values, e.g., 10x-v1: 14 bp barcode/10 bp UMI and 10x-v2: 16 bp barcode/10 bp UMI
- Boolean flags are encoded as explicit boolean values, not free text, for all four special-case flag columns

## Limitations

- UniverSC determines the chemistry appropriate for each single-cell technology for 3' scRNA-seq automatically, so the curated parameters reflect the tool's detection settings rather than user-supplied choices
- Technologies for 5' scRNA-seq other than that of Chromium are not supported at the time of writing, so the 40-technology catalogue does not extend to those platforms
- UniverSC operates as a wrapper for Cell Ranger version 3.0.2 due to licensing, so the parameter table is tied to that pipeline version

## Evidence

- [other] Table 1 enumerates the pre-set settings per technology, giving barcode and UMI lengths: [evidence span withheld — anchor status: not_found]
- [methods] automatic chemistry determination: [evidence span withheld — anchor status: not_found]
- [methods] 5' technologies unsupported: [evidence span withheld — anchor status: not_found]
- [readme] UniverSC scope from README: [evidence span withheld — anchor status: external_doc]
