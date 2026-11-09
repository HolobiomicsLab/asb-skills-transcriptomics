---
name: custom-chemistry-registry-encoding
description: Use when FASTQ reads come from a single-cell protocol whose barcode/UMI
  layout does not fit the mapper's native simple-geometry expectations — e.g. concatenated
  barcodes, variable-length segments, or segments not at fixed offsets.
when_to_use_negative:
- Reads already follow a simple fixed-position geometry natively supported by the
  alevin-fry mapper — no normalization layer is needed.
- The task is building an expanded (splici/spliceu) reference or index, not encoding
  a read chemistry.
- Barcodes are supplied out-of-band as a separate file (e.g. scATAC-style --barcode
  input) rather than embedded in the read geometry.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0080
tools:
- name: simpleaf
  role: Parses the geometry grammar, normalizes complex-geometry reads to fixed-position
    format, and streams them to the mapper via managed FIFOs
  repo: https://github.com/COMBINE-lab/simpleaf
- name: piscem
  role: RNA back-end mapper that consumes the normalized reads; its --geometry flag
    specifies the UMI/cell-barcode layout
  repo: https://github.com/COMBINE-lab/piscem
- name: alevin-fry
  role: Downstream processing (generate-permit-list, collate, quant) of the RAD file
    produced from normalized, mapped reads
  repo: https://github.com/COMBINE-lab/alevin-fry
- name: scg_lib_structs
  role: Reference collection of single-cell library structures and machine-readable
    formats for defining custom chemistries
  repo: https://github.com/Teichlab/scg_lib_structs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:6e84391740e1d38d678700a9383f7222b9bb006f5c397ab55751b2954a2fb6d3
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# custom-chemistry-registry-encoding

## Summary

Encodes custom single-cell library chemistries — including variable-length and floating-position barcodes — as geometry grammar strings (e.g. '1{b[16]u[12]x:}2{r:}', '1{b[16]u[12]b[4]x:}') that simpleaf parses into fixed-position extraction records, normalizing complex fragment geometries on the fly so piscem and alevin-fry can process them unchanged.

## When to use

Use when FASTQ reads come from a single-cell protocol whose barcode/UMI layout does not fit the mapper's native simple-geometry expectations — e.g. concatenated barcodes, variable-length segments, or segments not at fixed offsets.

## When NOT to use

- Reads already follow a simple fixed-position geometry natively supported by the alevin-fry mapper — no normalization layer is needed.
- The task is building an expanded (splici/spliceu) reference or index, not encoding a read chemistry.
- Barcodes are supplied out-of-band as a separate file (e.g. scATAC-style --barcode input) rather than embedded in the read geometry.

## Inputs

- Paired-end FASTQ reads from a custom single-cell sequencing protocol
- Geometry/chemistry specification string in the read-pair grammar (e.g. '1{b[16]u[12]x:}2{r:}')
- Library-structure documentation (e.g. scg_lib_structs machine-readable library structures)

## Outputs

- geometry_spec.json: structured per-geometry extraction records (segment type, start offset, length, concatenation group)
- Normalized fixed-position reads streamed via FIFO to the mapper (no intermediate FASTQ on disk)
- RAD-format mapped output consumable by alevin-fry generate-permit-list / collate / quant

## How to apply

Express the library structure as a geometry string using the read-pair grammar with segment codes b (barcode), u (UMI), r (mappable read), and x (discard), bracketed fixed lengths (e.g. b[16]), and a colon marker denoting remainder-of-read (e.g. 10x Chromium v3 '1{b[16]u[12]x:}2{r:}', 10xv2 '1{b[16]u[10]x:}2{r:}', concatenated-barcode '1{b[16]u[12]b[4]x:}'). Tokenize the string into a structured specification where each repeated segment code (like b[16] followed by b[4]) records its 1-based positional interval and a concatenation group flag so the full barcode is assembled from non-contiguous bases (e.g. bases 1-16 and 29-32). Convert each parsed specification into a fixed-position extraction record listing segment type, start offset, length, and concatenation group. simpleaf then normalizes each read into the simple fixed-position format on the fly and streams the normalized reads to the mapper through program-managed FIFOs, avoiding intermediate disk usage; the mappers therefore handle sophisticated geometries unchanged.

## Related tools

- **simpleaf** (Parses the geometry grammar, normalizes complex-geometry reads to fixed-position format, and streams them to the mapper via managed FIFOs) — https://github.com/COMBINE-lab/simpleaf
- **piscem** (RNA back-end mapper that consumes the normalized reads; its --geometry flag specifies the UMI/cell-barcode layout) — https://github.com/COMBINE-lab/piscem
- **alevin-fry** (Downstream processing (generate-permit-list, collate, quant) of the RAD file produced from normalized, mapped reads) — https://github.com/COMBINE-lab/alevin-fry
- **scg_lib_structs** (Reference collection of single-cell library structures and machine-readable formats for defining custom chemistries) — https://github.com/Teichlab/scg_lib_structs

## Examples

```
piscem map-sc -1 R1.fq.gz -2 R2.fq.gz -i splici_index -o alevin -t 16 --geometry 1{b[16]u[12]x:}2{r:}
```

## Evaluation signals

- The 10xv3 string '1{b[16]u[10]x:}2{r:}' resolves to a 16-base barcode and 10-base UMI with the remainder of read 1 discarded and full read 2 as mappable sequence (v3 variant: 12-base UMI).
- The repeated-barcode string '1{b[16]u[12]b[4]x:}' resolves to bases 1-16 and 29-32 concatenated into the full barcode.
- geometry_spec.json entries each carry segment type, start offset, length, and concatenation group for every segment code.
- No intermediate normalized FASTQ appears on disk — normalized reads pass through program-managed FIFOs.
- The mapper's RAD output is accepted by alevin-fry generate-permit-list without geometry-related errors.

## Limitations

- FIFO/non-regular inputs force the serial decode path in piscem — the parallel decoder requires positional reads, so normalization streaming gives up parallel gzip decoding.
- The colon marker denotes unbounded remainder-of-read, so extraction records for such segments have no fixed length.
- Repeated segment codes require explicit concatenation-group bookkeeping; a geometry grammar without it silently truncates the barcode.
- No simpleaf version is pinned alongside the documented features in the article's discussion, so behavior may drift across releases.

## Evidence

- [methods] simpleaf normalizes reads with complex fragment geometries into a simple fixed-position format on the fly and streams the normalized reads directly to the mapper through FIFOs: [evidence span withheld — anchor status: not_found]
- [abstract] simpleaf simplifies alevin-fry ecosystem processing: "We introduce simpleaf, a program that simplifies the processing of single-cell data using tools from the alevin-fry ecosystem"
- [readme] piscem --geometry flag defines UMI/barcode layout: [evidence span withheld — anchor status: external_doc]
- [readme] FIFO inputs force serial decoding in piscem: [evidence span withheld — anchor status: external_doc]
