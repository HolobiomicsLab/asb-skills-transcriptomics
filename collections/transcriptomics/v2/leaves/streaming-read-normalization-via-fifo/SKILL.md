---
name: streaming-read-normalization-via-fifo
description: Use when single-cell library read structure cannot be expressed as one
  fixed barcode/UMI offset — e.g. barcodes split across two segments like '1{b[16]u[12]b[4]x:}'
  — yet the underlying mapper only supports simple geometry.
when_to_use_negative:
- The library geometry is already simple and fixed-position (e.g. a single contiguous
  16 bp barcode plus 12 bp UMI expressible directly by the mapper's native --geometry
  flag) — normalize directly in the mapper instead
- You need persistent normalized FASTQ files on disk for QC or reuse — the FIFO design
  streams reads in memory and writes no intermediate files
- Reads are single-end with no read-2 mappable sequence to relay, so the read-pair
  geometry grammar's '2{r:}' component does not apply
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: simpleaf
  role: implements the geometry tokenizer/parser, on-the-fly read normalization, and
    programmatic FIFO management that stream normalized reads to the mapper
  repo: https://github.com/COMBINE-lab/simpleaf
- name: piscem
  role: RNA back-end mapper (piscem map-scrna) that consumes the FIFO-streamed normalized
    reads and emits RAD files; its --geometry flag describes barcode/UMI geometry
  repo: https://github.com/COMBINE-lab/piscem
- name: alevin-fry
  role: downstream processing of the mapper's RAD output (generate-permit-list, collate,
    quant); kept unchanged by normalizing geometry upstream
  repo: https://github.com/COMBINE-lab/alevin-fry
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:a8ffe7178ecb82440b0ed4934a9690f3f3dfe6fdaac3953592dc7edc6719bc8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/simpleaf
  tools:
  - Rust
  - alevin-fry
  - piscem
  - simpleaf
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# streaming-read-normalization-via-fifo

## Summary

A simpleaf technique that normalizes sequencing reads with complex fragment geometries (variable-length or floating-position barcodes, e.g. concatenated barcode segments) into a simple fixed-position format on the fly, and streams the normalized reads directly to the piscem mapper through programmatically managed FIFOs so no intermediate normalized FASTQ is written to disk.

## When to use

Use when single-cell library read structure cannot be expressed as one fixed barcode/UMI offset — e.g. barcodes split across two segments like '1{b[16]u[12]b[4]x:}' — yet the underlying mapper only supports simple geometry.

## When NOT to use

- The library geometry is already simple and fixed-position (e.g. a single contiguous 16 bp barcode plus 12 bp UMI expressible directly by the mapper's native --geometry flag) — normalize directly in the mapper instead
- You need persistent normalized FASTQ files on disk for QC or reuse — the FIFO design streams reads in memory and writes no intermediate files
- Reads are single-end with no read-2 mappable sequence to relay, so the read-pair geometry grammar's '2{r:}' component does not apply

## Inputs

- paired-end FASTQ files (gzip-compressed) from single-cell libraries with complex fragment geometry
- geometry string in the simpleaf/piscem read-pair grammar, e.g. '1{b[16]u[12]x:}2{r:}' or '1{b[16]u[12]b[4]x:}'

## Outputs

- stream of normalized reads with fixed-position barcode/UMI extraction delivered via FIFO to piscem map-scrna
- RAD format output from the downstream mapper suitable for alevin-fry generate-permit-list, collate, and quant

## How to apply

First, express the library's fragment structure in the read-pair geometry grammar, using segment codes b (barcode), u (UMI), r (mappable read) and x (discard), bracketed fixed lengths, and a colon marker for remainder-of-read; documented examples include 10x Chromium v3 '1{b[16]u[12]x:}2{r:}', 10x v2 '1{b[16]u[10]x:}2{r:}', and the concatenated-barcode form '1{b[16]u[12]b[4]x:}'. Build a tokenizer/parser over this grammar that records each 1-based positional interval per segment; for repeated codes (e.g. b[16] followed later by b[4]), flag the fragments for concatenation into the full barcode. Convert each parsed specification into fixed-position extraction records listing segment type, start offset, length, and concatenation group. At run time, apply these extraction records to the raw FASTQ streams and pipe the normalized reads through named FIFOs directly into piscem map-scrna, so the mapper — designed only for simple geometry — sees fixed-position barcode/UMI/read layout unchanged. Rationale: streaming through FIFOs avoids intermediate disk usage and keeps the entire alevin-fry ecosystem (piscem mapping, alevin-fry generate-permit-list, collate, quant) usable without modifying the mappers.

## Related tools

- **simpleaf** (implements the geometry tokenizer/parser, on-the-fly read normalization, and programmatic FIFO management that stream normalized reads to the mapper) — https://github.com/COMBINE-lab/simpleaf
- **piscem** (RNA back-end mapper (piscem map-scrna) that consumes the FIFO-streamed normalized reads and emits RAD files; its --geometry flag describes barcode/UMI geometry) — https://github.com/COMBINE-lab/piscem
- **alevin-fry** (downstream processing of the mapper's RAD output (generate-permit-list, collate, quant); kept unchanged by normalizing geometry upstream) — https://github.com/COMBINE-lab/alevin-fry

## Examples

```
piscem map-sc -1 sample_R1.fastq.gz -2 sample_R2.fastq.gz -i piscem_idx -o sample_af -t 16 --geometry '1{b[16]u[12]x:}2{r:}'
```

## Evaluation signals

- Parsing '1{b[16]u[12]x:}2{r:}' must yield a 16-base barcode followed by a 12-base UMI, with the rest of read 1 discarded and full-length read 2 as mappable sequence
- Parsing the repeated-barcode geometry '1{b[16]u[12]b[4]x:}' must resolve to bases 1-16 and 29-32 concatenated into the full barcode
- All parsed geometry specifications (segment type, start offset, length, concatenation group) round-trip into a fixed-position extraction record without residual variable-length segments
- The mapper receives reads through the FIFO without the pipeline creating intermediate normalized FASTQ files on disk
- Downstream alevin-fry generate-permit-list and quant run on the resulting RAD output and produce cell-by-gene counts consistent with the expected barcode/UMI lengths

## Limitations

- Non-regular inputs such as FIFOs use the serial decompression path, since the parallel decoder requires positional reads — one FIFO among inputs downgrades only that file, but FIFO streaming forfeits parallel gzip decoding
- The geometry grammar requires fixed bracketed lengths per segment; truly variable-length segments are handled only via the colon remainder-of-read marker and concatenation of repeated fixed-length codes
- The approach relies on simpleaf managing the FIFO plumbing; mis-specified geometry strings (wrong offsets or lengths) will silently mis-extract barcodes/UMIs rather than fail loudly

## Evidence

- [methods] simpleaf normalizes reads with complex fragment geometries into a simple fixed-position format on the fly and streams the normalized reads directly to the mapper through FIFOs it manages programmatically: [evidence span withheld — anchor status: not_found]
- [intro] piscem is simpleaf's RNA back-end mapper and indexer: "`simpleaf` uses [`piscem`](https://github.com/COMBINE-lab/piscem) as its RNA back-end mapper and indexer."
- [readme] piscem map-sc --geometry flag specifies UMI and cell barcode geometry: [evidence span withheld — anchor status: external_doc]
- [readme] FIFO inputs take the serial decoder path in piscem: [evidence span withheld — anchor status: external_doc]
- [intro] simpleaf is a rust framework to make using alevin-fry even simpler: "`simpleaf` is a `rust` framework to make using `alevin-fry` _even_ simpler."
