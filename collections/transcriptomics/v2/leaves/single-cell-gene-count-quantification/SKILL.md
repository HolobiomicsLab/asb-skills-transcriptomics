---
name: single-cell-gene-count-quantification
description: Use when raw paired FASTQ reads from a single-cell scRNA-seq library
  (e.g. 10xv2/10xv3 chemistry) must be converted into a gene-by-cell sparse count
  matrix (Matrix Market or AnnData .h5ad) from an existing splici/piscem index.
when_to_use_negative:
- Input is already a count matrix or feature-barcode table — this skill consumes raw
  FASTQ reads, not processed matrices.
- No expanded reference/index exists yet — run the simpleaf index step first to build
  the splici/piscem index before quantifying.
- Data are bulk RNA-seq or scATAC-seq — use the appropriate piscem/alevin-fry modes
  instead of the single-cell RNA quant path.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: simpleaf
  role: single-command wrapper that orchestrates mapping, permit-list generation,
    collate, and UMI resolution for one sample
  repo: https://github.com/COMBINE-lab/simpleaf
- name: alevin-fry
  role: underlying engine performing permit-list generation (generate-permit-list),
    collate, and UMI resolution/quant
  repo: https://github.com/COMBINE-lab/alevin-fry
- name: piscem
  role: RNA back-end mapper and indexer; maps single-cell reads against the index
    producing a RAD file for alevin-fry
  repo: https://github.com/COMBINE-lab/piscem
- name: roers
  role: library used by simpleaf to build the augmented (splici) transcriptome reference
    that the index step prepares
  repo: https://github.com/COMBINE-lab/roers
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:92580db0fcf1c8ad06bfd3d093d6170ea2083112a5b16d6a830c8caf03b2c9a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/simpleaf
  tools:
  - alevin-fry
  - piscem
  - roers
  - simpleaf
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42105263157894735
n_claims: 19
---

# single-cell-gene-count-quantification

## Summary

Produces a per-sample single-cell gene count matrix by running the simpleaf `quant` command, which wraps the alevin-fry pipeline stages of read mapping with piscem, cell-barcode detection/correction, and UMI resolution.

## When to use

Use when raw paired FASTQ reads from a single-cell scRNA-seq library (e.g. 10xv2/10xv3 chemistry) must be converted into a gene-by-cell sparse count matrix (Matrix Market or AnnData .h5ad) from an existing splici/piscem index.

## When NOT to use

- Input is already a count matrix or feature-barcode table — this skill consumes raw FASTQ reads, not processed matrices.
- No expanded reference/index exists yet — run the simpleaf index step first to build the splici/piscem index before quantifying.
- Data are bulk RNA-seq or scATAC-seq — use the appropriate piscem/alevin-fry modes instead of the single-cell RNA quant path.

## Inputs

- paired single-cell FASTQ read files (comma-separated lists via --reads1/--reads2)
- splici or spliceu piscem index directory produced by simpleaf index
- transcript-to-gene mapping file (t2g) supplied via --t2g-map
- chemistry specification: registered name (e.g. 10xv3) or custom geometry string such as "1{b[16]u[12]x:}2{r:}"

## Outputs

- Matrix Market sparse gene-by-cell count matrix under af_quant/alevin
- permit-list, collate, and quant metadata records
- optional AnnData .h5ad matrix when --anndata-out is passed

## How to apply

First configure the runtime by exporting ALEVIN_FRY_HOME and registering the alevin-fry and piscem executables via `simpleaf set-paths`. Then invoke `simpleaf quant` with `--index` pointing at the splici/piscem index from the upstream simpleaf index step, passing paired FASTQ files as comma-separated lists to `--reads1`/`--reads2` in matching order. Under the hood, simpleaf runs piscem mapping (producing a RAD file), alevin-fry permit-list generation, collate, and quant — i.e. mapping, barcode detection and correction, and UMI resolution. Supply `--chemistry` as a registered name (e.g. 10xv3) or a custom geometry string such as "1{b[16]u[12]x:}2{r:}"; choose a permit-list mode (--knee, --unfiltered-pl, --explicit-pl, --expect-cells, or --forced-cells), using the 10-reads-per-cell minimum threshold for --unfiltered-pl, and set --expected-ori to fw for 10xv2/10xv3 or both otherwise. Select UMI resolution (e.g. cr-like-em) with --resolution, pass the transcript-to-gene map with --t2g-map, and optionally add --anndata-out to also emit a .h5ad matrix.

## Related tools

- **simpleaf** (single-command wrapper that orchestrates mapping, permit-list generation, collate, and UMI resolution for one sample) — https://github.com/COMBINE-lab/simpleaf
- **alevin-fry** (underlying engine performing permit-list generation (generate-permit-list), collate, and UMI resolution/quant) — https://github.com/COMBINE-lab/alevin-fry
- **piscem** (RNA back-end mapper and indexer; maps single-cell reads against the index producing a RAD file for alevin-fry) — https://github.com/COMBINE-lab/piscem
- **roers** (library used by simpleaf to build the augmented (splici) transcriptome reference that the index step prepares) — https://github.com/COMBINE-lab/roers

## Examples

```
simpleaf quant --index splici_index --reads1 R1.fastq.gz --reads2 R2.fastq.gz --chemistry 10xv3 --unfiltered-pl --expected-ori fw --resolution cr-like-em --t2g-map probe_t2g.tsv --anndata-out
```

## Evaluation signals

- The output directory contains a Matrix Market sparse count matrix under af_quant/alevin plus permit-list, collate, and quant metadata records, confirming all pipeline steps completed.
- Permit-list mode matches the intended chemistry/strategy (e.g. --unfiltered-pl applies the default minimum read count threshold of 10 reads per cell).
- --expected-ori was set consistently with chemistry (fw for 10xv2/10xv3, both otherwise) so barcode orientation is not misassigned.
- With --anndata-out, a valid .h5ad file is generated whose values match the Matrix Market matrix derived from it.
- The --reads1 and --reads2 comma-separated lists were supplied in matching order, as required by the mapping stage.

## Limitations

- simpleaf uses piscem as its RNA back-end mapper and indexer; if piscem index construction fails, increase the file handle limit before filing an issue on GitHub.
- The quant command assumes an upstream index step (splici/spliceu expanded reference and piscem build) has already been completed.
- Multiple FASTQ pairs must be passed in the same order to --reads1 and --reads2; mismatches silently pair the wrong files.

## Evidence

- [intro] the quantification of a sample into a single command (`quant`): "`simpleaf` encapsulates the process of creating an expanded reference for quantification into a single command (`index`) and the quantification of a sample into a single command (`quant`)"
- [methods] alevin-fry stages wrapped by quant: "multi-barcode permit-list generation with ``alevin-fry generate-permit-list``"
- [readme] mapping produces RAD for alevin-fry: [evidence span withheld — anchor status: external_doc]
- [abstract] simpleaf purpose statement: "We introduce simpleaf, a program that simplifies the processing of single-cell data using tools from the alevin-fry ecosystem"
- [methods] unfiltered permit list threshold: "--min-reads <MIN_READS>  Minimum read count threshold for unfiltered permit list [default: 10]"
