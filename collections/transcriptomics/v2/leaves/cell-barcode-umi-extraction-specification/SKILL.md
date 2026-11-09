---
name: cell-barcode-umi-extraction-specification
description: Use when sequencing reads embed cell barcodes and UMIs at variable-length
  or non-fixed positions (e.g. 10x Chromium v2/v3, concatenated barcodes) and must
  be normalized to a simple fixed-position geometry before mapping and quantification.
when_to_use_negative:
- Reads already have simple fixed-position barcode/UMI geometry that the underlying
  alevin-fry/piscem mappers support natively, so no normalization is needed
- Barcode sequences are supplied separately from the mappable reads (e.g. piscem map-sc-atac
  reads barcodes via a separate --barcode file) rather than embedded in the read geometry
- Input is bulk RNA-seq or scATAC-seq data with no cell-barcode/UMI fragment structure
  to parse
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: simpleaf
  role: Parses geometry strings and normalizes complex-fragment reads to fixed-position
    format, streaming them to the mapper via programmatically managed FIFOs
  repo: https://github.com/COMBINE-lab/simpleaf
- name: piscem
  role: RNA back-end mapper/indexer that receives the normalized simple-geometry reads;
    its map-sc command accepts a --geometry flag for UMI/cell-barcode geometry
  repo: https://github.com/COMBINE-lab/piscem
- name: alevin-fry
  role: 'Downstream processing of the mapped RAD output: generate-permit-list, collate,
    and quant consume the extracted barcodes/UMIs'
  repo: https://github.com/COMBINE-lab/alevin-fry
- name: scg_lib_structs
  role: Reference collection of machine-readable library structures for popular single-cell
    methods, useful as a source of geometry specifications
  repo: https://github.com/Teichlab/scg_lib_structs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:b28547e01a5ac686f5e5acc5d7f3a23e49994bb4aacba7283b7c1ff478b11408
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/simpleaf
  tools:
  - Rust
  - alevin-fry
  - piscem
  - scg_lib_structs
  - simpleaf
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cell-barcode-umi-extraction-specification

## Summary

Specifies how to parse single-cell library geometry strings (e.g. 10x Chromium v3 '1{b[16]u[12]x:}2{r:}') into fixed-position barcode/UMI extraction records, so mappers designed for simple fragment geometry can handle variable-length or floating-position barcodes without modification.

## When to use

Use when sequencing reads embed cell barcodes and UMIs at variable-length or non-fixed positions (e.g. 10x Chromium v2/v3, concatenated barcodes) and must be normalized to a simple fixed-position geometry before mapping and quantification.

## When NOT to use

- Reads already have simple fixed-position barcode/UMI geometry that the underlying alevin-fry/piscem mappers support natively, so no normalization is needed
- Barcode sequences are supplied separately from the mappable reads (e.g. piscem map-sc-atac reads barcodes via a separate --barcode file) rather than embedded in the read geometry
- Input is bulk RNA-seq or scATAC-seq data with no cell-barcode/UMI fragment structure to parse

## Inputs

- Paired-end FASTQ reads (R1/R2) from a single-cell library with complex fragment geometry
- Geometry string specification (e.g. '1{b[16]u[12]x:}2{r:}', '1{b[16]u[10]x:}2{r:}', '1{b[16]u[12]b[4]x:}')
- piscem index built over an expanded (splici/spliceu) reference

## Outputs

- geometry_spec.json containing structured extraction records (segment type, start offset, length, concatenation group)
- Normalized fixed-position read stream delivered to the mapper via FIFO (no intermediate disk files)
- RAD format mapped output consumable by alevin-fry (generate-permit-list, collate, quant)

## How to apply

Tokenize the geometry string with a parser for the read-pair grammar: segment codes b (barcode), u (UMI), r (mappable read), and x (discard), bracketed fixed lengths such as b[16] or u[12], and the colon marker denoting remainder-of-read. For repeated segment codes (e.g. b[16] followed by b[4]), record each 1-based positional interval and flag the fragment for concatenation into the full barcode — the concatenated-barcode form '1{b[16]u[12]b[4]x:}' resolves to bases 1-16 and 29-32 concatenated. Convert each parsed read specification into a fixed-position extraction record listing segment type, start offset, length, and concatenation group, and write all geometry specifications to geometry_spec.json. simpleaf then normalizes reads with complex fragment geometries on the fly and streams the normalized reads directly to the mapper (piscem map-scrna) through programmatically managed FIFOs, avoiding intermediate disk usage, so the mapper sees only simple-geometry reads. Validation requires the 10xv3 string to yield a 16-base barcode plus 12-base UMI with the rest of read 1 discarded and full-length read 2 as mappable sequence.

## Related tools

- **simpleaf** (Parses geometry strings and normalizes complex-fragment reads to fixed-position format, streaming them to the mapper via programmatically managed FIFOs) — https://github.com/COMBINE-lab/simpleaf
- **piscem** (RNA back-end mapper/indexer that receives the normalized simple-geometry reads; its map-sc command accepts a --geometry flag for UMI/cell-barcode geometry) — https://github.com/COMBINE-lab/piscem
- **alevin-fry** (Downstream processing of the mapped RAD output: generate-permit-list, collate, and quant consume the extracted barcodes/UMIs) — https://github.com/COMBINE-lab/alevin-fry
- **scg_lib_structs** (Reference collection of machine-readable library structures for popular single-cell methods, useful as a source of geometry specifications) — https://github.com/Teichlab/scg_lib_structs

## Examples

```
piscem map-sc -1 reads_R1.fq.gz -2 reads_R2.fq.gz --geometry 1{b[16]u[12]x:}2{r:} -o af_out -t 16
```

## Evaluation signals

- geometry_spec.json must resolve the documented 10xv3 string to a 16-base barcode followed by a 12-base UMI with the remainder of read 1 discarded and full-length read 2 as mappable sequence
- The repeated-barcode form '1{b[16]u[12]b[4]x:}' must resolve to bases 1-16 and 29-32 concatenated into the full barcode
- Every parsed extraction record must list segment type, start offset, length, and concatenation group, with the colon marker correctly denoting remainder-of-read
- Normalized reads must reach the mapper through FIFOs without intermediate disk files, and the resulting RAD file must be consumable by alevin-fry generate-permit-list/collate/quant
- Downstream permit-list generation must find valid 16+4=20-base or protocol-appropriate barcodes, indicating correct concatenation of repeated b segments

## Limitations

- Non-regular inputs such as FIFOs and process substitutions always take the serial decode path in piscem because the parallel decoder requires positional reads, so FIFO streaming forfeits the parallel gzip decoder
- The geometry grammar only covers segment codes b, u, r, x with bracketed fixed lengths and a remainder-of-read marker; protocols outside this grammar cannot be expressed
- The normalization is a pre-mapping on-the-fly step: it does not change the mapper itself, so mappers still expect simple fixed-position geometry in the stream they consume

## Evidence

- [other] simpleaf normalizes reads with complex fragment geometries into a simple fixed-position format on the fly and streams the normalized reads directly to the mapper: [evidence span withheld — anchor status: not_found]
- [intro] piscem used as the RNA back-end mapper and indexer: "`simpleaf` uses [`piscem`](https://github.com/COMBINE-lab/piscem) as its RNA back-end mapper and indexer."
- [readme] piscem map-sc geometry flag specifies UMIs and cell barcodes: [evidence span withheld — anchor status: external_doc]
- [readme] FIFO and process-substitution inputs always use the serial decode path: [evidence span withheld — anchor status: external_doc]
- [abstract] simpleaf introduced to simplify single-cell processing with alevin-fry: "We introduce simpleaf, a program that simplifies the processing of single-cell data using tools from the alevin-fry ecosystem"
