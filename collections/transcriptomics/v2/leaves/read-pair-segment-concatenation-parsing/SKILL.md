---
name: read-pair-segment-concatenation-parsing
description: Use when input FASTQ libraries follow sequencing protocols whose cell-barcode/UMI
  layout is variable-length, floating-position, or split across multiple segments
  — i.e., not expressible as a single fixed-position barcode+UMI geometry a mapper
  like piscem natively supports.
when_to_use_negative:
- Reads already follow a simple fixed-position 3'-tag geometry that piscem/alevin-fry
  support directly — no tokenizer or normalization is needed
- The task is bulk RNA-seq or scATAC-seq mapping (piscem map-bulk / map-sc-atac),
  which does not use the barcode/UMI read-pair geometry grammar
- The barcode segments are non-contiguous in a way not expressible as repeated segment
  codes with bracketed lengths plus colon remainders in the b/u/r/x grammar
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: simpleaf
  role: Implements the geometry tokenizer/normalizer and streams normalized reads
    to the mapper through programmatically managed FIFOs
  repo: https://github.com/COMBINE-lab/simpleaf
- name: piscem
  role: RNA back-end mapper whose map-sc command receives the normalized reads and
    produces RAD output
  repo: https://github.com/COMBINE-lab/piscem
- name: alevin-fry
  role: Downstream single-cell processing suite (permit lists, collate, quant) that
    consumes the RAD files produced after geometry normalization
  repo: https://github.com/COMBINE-lab/alevin-fry
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:eb39145130469ec1944a4a9b5cbebc5a61f1239a07b5fa8bbb4d700bd5b5ae65
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# read-pair-segment-concatenation-parsing

## Summary

A tokenizer/parser for read-pair geometry strings (e.g., 10x Chromium v3 '1{b[16]u[12]x:}2{r:}', 10xv2 '1{b[16]u[10]x:}2{r:}', concatenated-barcode '1{b[16]u[12]b[4]x:}') that converts complex fragment layouts into fixed-position extraction records so simple-geometry mappers can process them unchanged.

## When to use

Use when input FASTQ libraries follow sequencing protocols whose cell-barcode/UMI layout is variable-length, floating-position, or split across multiple segments — i.e., not expressible as a single fixed-position barcode+UMI geometry a mapper like piscem natively supports.

## When NOT to use

- Reads already follow a simple fixed-position 3'-tag geometry that piscem/alevin-fry support directly — no tokenizer or normalization is needed
- The task is bulk RNA-seq or scATAC-seq mapping (piscem map-bulk / map-sc-atac), which does not use the barcode/UMI read-pair geometry grammar
- The barcode segments are non-contiguous in a way not expressible as repeated segment codes with bracketed lengths plus colon remainders in the b/u/r/x grammar

## Inputs

- Paired-end FASTQ reads from protocols with complex fragment geometry (e.g., 10x Chromium v3/v2, concatenated-barcode libraries)
- Geometry strings in the simpleaf/piscem grammar (e.g., '1{b[16]u[12]x:}2{r:}', '1{b[16]u[12]b[4]x:}')
- piscem index built over a splici/spliceu expanded reference

## Outputs

- geometry_spec.json containing fixed-position extraction records (segment type, start offset, length, concatenation group) for each parsed geometry
- Normalized fixed-position read pairs streamed to the mapper via FIFOs
- RAD-format mapping output consumable by alevin-fry

## How to apply

Parse the geometry string with a tokenizer over the segment codes b (barcode), u (UMI), r (mappable read), and x (discard), where bracketed numbers (e.g., b[16]) denote fixed lengths and the colon marker denotes remainder-of-read. For read 1 vs read 2, the grammar prefix '1{...}' and '2{...}' assigns segments to each mate; '2{r:}' means the full-length read 2 is mappable sequence. When the same segment code recurs (e.g., b[16] followed later by b[4]), record each 1-based positional interval and flag the fragment for concatenation into the full barcode — the concatenated-barcode example resolves to bases 1-16 and 29-32 joined into a 20-base barcode. Emit one structured extraction record per segment listing segment type, start offset, length, and concatenation group (e.g., into geometry_spec.json), then use those records to normalize reads on the fly and stream them directly to the mapper (piscem) through programmatically managed FIFOs, avoiding intermediate disk usage. This preserves the existing mappers unchanged: piscem map-sc consumes the normalized reads with its --geometry flag and emits RAD files that alevin-fry processes downstream.

## Related tools

- **simpleaf** (Implements the geometry tokenizer/normalizer and streams normalized reads to the mapper through programmatically managed FIFOs) — https://github.com/COMBINE-lab/simpleaf
- **piscem** (RNA back-end mapper whose map-sc command receives the normalized reads and produces RAD output) — https://github.com/COMBINE-lab/piscem
- **alevin-fry** (Downstream single-cell processing suite (permit lists, collate, quant) that consumes the RAD files produced after geometry normalization) — https://github.com/COMBINE-lab/alevin-fry

## Examples

```
piscem map-sc -i index -1 R1.fq.gz -2 R2.fq.gz -o map_out -t 16 --geometry '1{b[16]u[12]b[4]x:}2{r:}'
```

## Evaluation signals

- The 10xv3 geometry string '1{b[16]u[12]x:}2{r:}' must resolve to a 16-base barcode followed by a 12-base UMI, with the rest of read 1 discarded and full-length read 2 as mappable sequence
- The repeated-barcode string '1{b[16]u[12]b[4]x:}' must resolve to bases 1-16 and 29-32 concatenated into the full barcode
- Each parsed record in geometry_spec.json must carry segment type, start offset, length, and concatenation group fields
- The colon marker must be interpreted as remainder-of-read (e.g., 'x:' discards the rest of read 1)
- The mapper must receive reads in fixed-position format via FIFO without intermediate on-disk normalized FASTQ files

## Limitations

- Normalization-on-the-fly relies on FIFO streaming; piscem notes that FIFO and process-substitution inputs always use the serial decoder path, forfeiting the parallel gzip decoder
- The grammar covers only the b/u/r/x segment codes with bracketed fixed lengths and colon remainders; more exotic fragment layouts fall outside it
- The pinned simpleaf version for the documented geometry features is not stated in the article, complicating exact reproduction

## Evidence

- [readme] geometry flag documents the barcode/UMI layout the mapper expects: [evidence span withheld — anchor status: external_doc]
- [other] task-card workflow specifies the exact geometry strings to parse: [evidence span withheld — anchor status: not_found]
- [other] task-card finding on FIFO streaming avoiding disk: [evidence span withheld — anchor status: not_found]
- [readme] piscem README on FIFO input decoding behavior: [evidence span withheld — anchor status: external_doc]
- [readme] alevin-fry README on RAD input consumption: [evidence span withheld — anchor status: external_doc]
