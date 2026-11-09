---
name: custom-chemistry-geometry-configuration
description: Use when sequencing a single-cell scRNA-seq library whose barcode/UMI
  layout is not a default registered chemistry, and you must express the read structure
  as an explicit geometry string for mapping and quantification.
when_to_use_negative:
- The data is bulk RNA-seq or scATAC-seq, where piscem map-bulk/map-sc-atac and their
  barcode flags apply instead of the map-sc chemistry geometry
- The library is a standard registered chemistry (e.g. 10xv3) and no custom geometry
  encoding is needed — use the registered name directly rather than hand-writing a
  geometry string
- The barcode/UMI read structure is unknown; resolve the library layout first (e.g.
  via scg_lib_structs documentation) before guessing a geometry string
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: simpleaf
  role: Front-end command that accepts the --chemistry flag (registered name or custom
    geometry string) and drives the full quant workflow
  repo: https://github.com/COMBINE-lab/simpleaf
- name: piscem
  role: RNA back-end mapper/indexer; its map-sc --geometry flag consumes the barcode/UMI
    geometry during read mapping
  repo: https://github.com/COMBINE-lab/piscem
- name: alevin-fry
  role: Downstream suite that performs barcode permit-list generation, collate, and
    UMI resolution on the RAD output produced under the configured geometry
  repo: https://github.com/COMBINE-lab/alevin-fry
- name: scg_lib_structs
  role: Reference resource for machine-readable library structure definitions of popular
    single-cell methods used to derive the geometry string
  repo: https://github.com/Teichlab/scg_lib_structs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:c3515becb15e270d9143b2921acaa955884c2e7a3290f0f5599716be01518302
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/simpleaf
  tools:
  - alevin-fry
  - piscem
  - scg_lib_structs
  - simpleaf
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# custom chemistry geometry configuration

## Summary

Configures simpleaf/alevin-fry single-cell quantification for non-default library layouts by supplying a custom barcode/UMI geometry string (e.g. 1{b[16]u[12]x:}2{r:}) or a registered chemistry name such as 10xv3, so piscem map-scrna and alevin-fry parse cell barcodes and UMIs from the correct read positions.

## When to use

Use when sequencing a single-cell scRNA-seq library whose barcode/UMI layout is not a default registered chemistry, and you must express the read structure as an explicit geometry string for mapping and quantification.

## When NOT to use

- The data is bulk RNA-seq or scATAC-seq, where piscem map-bulk/map-sc-atac and their barcode flags apply instead of the map-sc chemistry geometry
- The library is a standard registered chemistry (e.g. 10xv3) and no custom geometry encoding is needed — use the registered name directly rather than hand-writing a geometry string
- The barcode/UMI read structure is unknown; resolve the library layout first (e.g. via scg_lib_structs documentation) before guessing a geometry string

## Inputs

- Paired FASTQ files (comma-separated lists to --reads1/--reads2, order matched)
- piscem splici index directory from simpleaf index
- Chemistry specification: registered chemistry name (e.g. 10xv3) or custom geometry string such as 1{b[16]u[12]x:}2{r:}
- Transcript-to-gene map (t2g) file

## Outputs

- RAD file from piscem mapping with correctly extracted cell barcodes and UMIs
- Permit-list, collate, and quant metadata records under af_quant/alevin
- Matrix Market sparse count matrix (optionally AnnData .h5ad via --anndata-out)

## How to apply

First determine the library's structure (barcode/UMI read lengths and which FASTQ mate carries them) using machine-readable library-structure resources such as Teichlab/scg_lib_structs. Then pass the layout to simpleaf quant via --chemistry, giving either a registered chemistry name (e.g. 10xv3) or a custom geometry string such as "1{b[16]u[12]x:}2{r:}", where b[16] marks a 16 bp cell barcode and u[12] a 12 bp UMI on read 1. If the layout matches a known chemistry, prefer the registered name; if it does not, encode the actual Illumina read structure in the geometry grammar that piscem map-scrna consumes via its --geometry flag. Unless explicitly overridden, set --expected-ori to fw for 10xv2/10xv3 or both for other geometries, and keep the order of files in the --reads1 and --reads2 comma-separated lists matched. Correct geometry configuration upstream determines whether alevin-fry generate-permit-list and downstream UMI resolution (e.g. --resolution cr-like-em) operate on validly extracted barcodes and UMIs.

## Related tools

- **simpleaf** (Front-end command that accepts the --chemistry flag (registered name or custom geometry string) and drives the full quant workflow) — https://github.com/COMBINE-lab/simpleaf
- **piscem** (RNA back-end mapper/indexer; its map-sc --geometry flag consumes the barcode/UMI geometry during read mapping) — https://github.com/COMBINE-lab/piscem
- **alevin-fry** (Downstream suite that performs barcode permit-list generation, collate, and UMI resolution on the RAD output produced under the configured geometry) — https://github.com/COMBINE-lab/alevin-fry
- **scg_lib_structs** (Reference resource for machine-readable library structure definitions of popular single-cell methods used to derive the geometry string) — https://github.com/Teichlab/scg_lib_structs

## Examples

```
simpleaf quant --index piscem_idx --reads1 s1_R1.fastq.gz --reads2 s1_R2.fastq.gz --chemistry "1{b[16]u[12]x:}2{r:}" --resolution cr-like-em --t2g-map t2g.tsv
```

## Evaluation signals

- The geometry string's b[] and u[] lengths match the actual barcode and UMI read lengths of the library (e.g. 16 bp barcode, 12 bp UMI for 10x-style v3)
- The output directory contains the Matrix Market sparse matrix under af_quant/alevin plus permit-list, collate, and quant metadata records, indicating all steps completed
- Mapping produces a valid RAD file from piscem map-scrna without barcode-parsing failures, confirming the --geometry/--chemistry was accepted
- Permit-list generation yields a plausible number of permitted cell barcodes for the chosen permit-list mode (knee, unfiltered-pl with min 10 reads, explicit, expect-cells, or forced-cells)
- Downstream UMI resolution with --t2g-map and e.g. --resolution cr-like-em completes, indicating barcodes/UMIs were extracted at the correct offsets

## Limitations

- A hand-written geometry string that does not match the true read structure will silently mis-extract barcodes/UMIs; verify against documented library structures before running
- Expected-read-orientation defaults must be considered: --expected-ori fw for 10xv2/10xv3, both otherwise, unless explicitly overridden
- Reads1/reads2 lists must be given in the same order; a mismatch pairs R1 with the wrong R2 for multi-sample runs

## Evidence

- [other] Card: custom chemistry string: [evidence span withheld — anchor status: not_found]
- [readme] Piscem geometry flag: [evidence span withheld — anchor status: external_doc]
- [readme] Piscem map-sc output: [evidence span withheld — anchor status: external_doc]
- [readme] scg_lib_structs purpose: [evidence span withheld — anchor status: external_doc]
- [other] Expected-ori rule: [evidence span withheld — anchor status: not_found]
