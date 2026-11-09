---
name: fragment-geometry-string-parsing
description: Use when single-cell FASTQ reads come from protocols whose barcode/UMI
  layout is not simple fixed-position (e.g. repeated barcode segments like b[16]...b[4],
  or 10x-style 1{b[16]u[12]x:}2{r:} geometry strings) and must be normalized before
  mapping.
when_to_use_negative:
- The protocol already has a simple fixed-position barcode/UMI geometry that the underlying
  mapper (e.g. piscem map-sc --geometry or alevin-fry) supports directly, so normalization
  would add overhead without benefit
- The input is not paired-end reads with an explicit barcode/UMI layout string — e.g.
  single-end reads with barcodes supplied through a separate file such as piscem map-sc-atac
  --barcode
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: simpleaf
  role: Implements the geometry normalization and streams normalized reads to the
    mapper through FIFOs it manages programmatically
  repo: https://github.com/COMBINE-lab/simpleaf
- name: piscem
  role: RNA back-end mapper/indexer that consumes the geometry-normalized reads (map-sc/map-scrna
    with --geometry flag)
  repo: https://github.com/COMBINE-lab/piscem
- name: alevin-fry
  role: Downstream processing (generate-permit-list, collate, quant) of reads normalized
    by the geometry parser
  repo: https://github.com/COMBINE-lab/alevin-fry
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:d36cff63298376566082c998a5175dc637b54f5ac88c29f02edd00f3617b2b91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/simpleaf
  tools:
  - Rust
  - alevin-fry
  - piscem
  - simpleaf
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# fragment-geometry-string-parsing

## Summary

Parses read-pair fragment-geometry specification strings (segment codes b/u/r/x with bracketed lengths and a colon remainder marker) into fixed-position extraction records, so alevin-fry/piscem mappers designed for simple barcodes can handle variable-length or floating-position barcodes without change.

## When to use

Use when single-cell FASTQ reads come from protocols whose barcode/UMI layout is not simple fixed-position (e.g. repeated barcode segments like b[16]...b[4], or 10x-style 1{b[16]u[12]x:}2{r:} geometry strings) and must be normalized before mapping.

## When NOT to use

- The protocol already has a simple fixed-position barcode/UMI geometry that the underlying mapper (e.g. piscem map-sc --geometry or alevin-fry) supports directly, so normalization would add overhead without benefit
- The input is not paired-end reads with an explicit barcode/UMI layout string — e.g. single-end reads with barcodes supplied through a separate file such as piscem map-sc-atac --barcode

## Inputs

- Geometry specification strings such as '1{b[16]u[12]x:}2{r:}', '1{b[16]u[10]x:}2{r:}', '1{b[16]u[12]b[4]x:}'
- Paired FASTQ read files from single-cell sequencing libraries
- Read-pair grammar segment codes b (barcode), u (UMI), r (mappable read), x (discard)

## Outputs

- geometry_spec.json holding structured extraction records (segment type, start offset, length, concatenation group) per parsed geometry
- Normalized read pairs in simple fixed-position barcode/UMI format streamed to the mapper via FIFOs

## How to apply

Tokenize the geometry string grammar where b denotes barcode, u denotes UMI, r denotes the mappable read, x denotes discarded sequence, bracketed integers give fixed lengths, and the colon denotes remainder-of-read (e.g. '1{b[16]u[12]x:}2{r:}' for 10x Chromium v3, '1{b[16]u[10]x:}2{r:}' for 10xv2, and '1{b[16]u[12]b[4]x:}' for concatenated barcodes). For repeated segment codes, record each 1-based positional interval and flag the fragment for concatenation into the full barcode. Convert the parsed specification into fixed-position extraction records listing segment type, start offset, length, and concatenation group, then normalize each read pair on the fly and stream the result to the mapper (piscem map-scrna) through programmatically managed FIFOs so no intermediate disk files are written. The downstream mapper and alevin-fry (generate-permit-list, collate, quant) consume the normalized reads as if the geometry were simple.

## Related tools

- **simpleaf** (Implements the geometry normalization and streams normalized reads to the mapper through FIFOs it manages programmatically) — https://github.com/COMBINE-lab/simpleaf
- **piscem** (RNA back-end mapper/indexer that consumes the geometry-normalized reads (map-sc/map-scrna with --geometry flag)) — https://github.com/COMBINE-lab/piscem
- **alevin-fry** (Downstream processing (generate-permit-list, collate, quant) of reads normalized by the geometry parser) — https://github.com/COMBINE-lab/alevin-fry

## Examples

```
piscem map-sc -1 R1.fastq.gz -2 R2.fastq.gz -o map_out --geometry '1{b[16]u[12]x:}2{r:}'
```

## Evaluation signals

- Parsing '1{b[16]u[12]x:}2{r:}' must yield a 16-base barcode followed by a 12-base UMI, with the rest of read 1 discarded and full-length read 2 as mappable sequence
- Parsing the repeated-barcode string '1{b[16]u[12]b[4]x:}' must resolve to bases 1-16 and 29-32 concatenated into the full barcode
- geometry_spec.json records must carry segment type, start offset, length, and concatenation group for every parsed segment
- Downstream alevin-fry generate-permit-list on the normalized reads produces a valid permit list without geometry-related errors
- No intermediate normalized FASTQ files should be written to disk when streaming through FIFOs

## Limitations

- Non-regular inputs such as FIFOs use the serial decode path, since the parallel decoder requires positional reads — one FIFO among inputs downgrades that file's decoding
- Repeated segment codes require the parser to track multiple 1-based positional intervals per code and flag concatenation; only documented example geometries have been validated
- piscem index builds need a file handle limit of at least 2048 (ulimit -n 2048) and a suitable -k (odd, <= 31) and -m (< k) for the underlying index

## Evidence

- [other] simpleaf normalizes reads with complex fragment geometries into a simple fixed-position format on the fly and streams the normalized reads directly to the mapper through FIFOs it manages programmatically: [evidence span withheld — anchor status: not_found]
- [readme] The --geometry flag specifies the geometry of the UMIs and cell barcodes for the reads: [evidence span withheld — anchor status: external_doc]
- [intro] simpleaf uses piscem as its RNA back-end mapper and indexer.: "`simpleaf` uses [`piscem`](https://github.com/COMBINE-lab/piscem) as its RNA back-end mapper and indexer."
- [methods] multi-barcode permit-list generation with alevin-fry generate-permit-list: "multi-barcode permit-list generation with ``alevin-fry generate-permit-list``"
- [readme] Non-regular inputs — FIFOs, process substitutions such as -1 <(zcat …) — always use the serial path, because the parallel decoder requires positional reads.: [evidence span withheld — anchor status: external_doc]
