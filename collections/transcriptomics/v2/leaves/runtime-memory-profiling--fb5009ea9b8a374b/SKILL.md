---
name: runtime-memory-profiling
description: 'Use when evaluating a binning-directory index (IBF/MIBF) for approximate
  sequence search: you must trade off index size (1-8 GiB), (19,19) vs (23,19) minimizers,
  compression, and partitioning against build time, query time, peak memory, and false-positive
  assignments.'
when_to_use_negative:
- Indices built with an older version of Raptor cannot be used with version 2.0.0
  — do not benchmark a legacy index against the 2.0.0 search code
- Input requires approximate matching with heavy edit-distance errors or the analysis
  goal is exact full-text search — Raptor is a pre-filter, not a mapper like DREAM-Yara
- The target index has highly uneven bin sizes or a very high number of bins, which
  the paper identifies as its main limiting factors
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Raptor
  role: Constructs IBF/MIBF binning-directory indices (raptor build) and queries 2^20
    simulated reads against them (raptor search) while varying IBF size, minimizer,
    compression, and partitioning
  repo: github.com/seqan/raptor
- name: TaxSBP
  role: Companion bin-packing/clustering tool referenced in the same paper's discussion;
    not required for the artificial-dataset benchmark
  repo: github.com/pirovc/taxsbp
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:df13cccb17bd646bf9144da3df1b694e9c5095fa7d931d28c9837d3b4d9c8fc2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Raptor
  - TaxSBP
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# runtime-memory-profiling

## Summary

Benchmark the Raptor binning-directory pre-filter by recording construction time, search time, peak RAM, and false-positive bin assignments across IBF sizes, minimizer settings, compression, and x-PIBF partitioning configurations, reproducing the paper's performance Tables 1 and 2.

## When to use

Use when evaluating a binning-directory index (IBF/MIBF) for approximate sequence search: you must trade off index size (1-8 GiB), (19,19) vs (23,19) minimizers, compression, and partitioning against build time, query time, peak memory, and false-positive assignments.

## When NOT to use

- Indices built with an older version of Raptor cannot be used with version 2.0.0 — do not benchmark a legacy index against the 2.0.0 search code
- Input requires approximate matching with heavy edit-distance errors or the analysis goal is exact full-text search — Raptor is a pre-filter, not a mapper like DREAM-Yara
- The target index has highly uneven bin sizes or a very high number of bins, which the paper identifies as its main limiting factors

## Inputs

- Artificial dataset with 64 bins and 1,024 bins (from github.com/seqan/raptor supplementary materials)
- 2^20 simulated query reads (FASTQ/FASTA) to search against each index
- Raptor CLI (raptor build, raptor search) with --help pages per command
- /usr/bin/time -v (or equivalent) for peak-RAM and wall-clock measurement

## Outputs

- table1.csv: per-index-configuration construction time, search time, peak RAM, and false-positive bin assignments (IBF size x minimizer x compression x bin count)
- TSV reproducing Table 2: timing, memory, and false-positive measurements for 16 x-PIBF configurations (b=64/1024 x minimizer x partition count)

## How to apply

Install Raptor via bioconda or compile from source, since compiling from source may give performance benefits from host-system optimization. Using 'raptor build', construct (19,19)-IBF and (23,19)-MIBF binning-directory indices over the artificial 64-bin and 1,024-bin dataset for each IBF size (1, 2, 4, 8 GiB), in uncompressed and compressed form, and additionally as x-PIBF indices partitioned into x=1, 2, 4, 8 parts. For each index, run 'raptor search' to query 2^20 simulated reads, wrapping each build and search call in '/usr/bin/time -v' to capture wall-clock construction time, search time, and peak RAM. Also record the number of false-positive bin assignments per configuration to quantify specificity. Compile all measurements into a single table (table1.csv / TSV) whose rows and columns reproduce the layout of Tables 1 and 2, e.g. 604,533 FPs at 1 GiB vs 189 at 2 GiB for b=64, and 8 GiB held for a 1-PIBF vs 1.4 GiB for an 8-PIBF.

## Related tools

- **Raptor** (Constructs IBF/MIBF binning-directory indices (raptor build) and queries 2^20 simulated reads against them (raptor search) while varying IBF size, minimizer, compression, and partitioning) — github.com/seqan/raptor
- **TaxSBP** (Companion bin-packing/clustering tool referenced in the same paper's discussion; not required for the artificial-dataset benchmark) — github.com/pirovc/taxsbp

## Examples

```
conda install -c bioconda -c conda-forge raptor
```

## Evaluation signals

- table1.csv exists with one row per index configuration (IBF size x minimizer x compression x bin count) and all timing/memory fields populated
- False-positive bin assignments match the near-zero performance Raptor reports for simulated data once IBF size >= 2 GiB (e.g. 604,533 FPs at 1 GiB vs 189 at 2 GiB for b=64)
- Memory reduction trend for x-PIBF holds: memory to hold a single part decreases with more parts (8 GiB for a 1-PIBF vs 1.4 GiB for an 8-PIBF) while construction and query time increase
- 16 index builds and searches completed for the Table 2 configuration set (2 bin counts x 2 minimizer variants x 4 partition counts)
- Peak-RAM and wall-clock values were captured with /usr/bin/time -v around every raptor build and raptor search call

## Limitations

- Compressed bitvector construction requires both the uncompressed and compressed bitvectors in main memory at some point during construction
- SDSL does not support decompression of the compressed bitvector, so the uncompressed bitvector must also be stored on disk for updates
- Indices built with an older version of raptor cannot be used with version 2.0.0
- Main limiting factors are susceptibility to uneven bin sizes and a high number of bins; processing several ten thousand bins leads to a moderate slowdown
- There may be performance differences depending on whether Raptor is installed via conda or compiled from source optimized for the host system

## Evidence

- [readme] compile benefits from source: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
- [abstract] compression and partitioning: "Our approach allows compression and partitioning of the IBF to enable the effective use of secondary memory."
- [abstract] testing on simulated and real datasets: "We test and show the performance and limitations of the new features using simulated and real datasets."
- [discussion] version incompatibility: "Indices built with an older version of raptor cannot be used with version 2.0.0."
- [methods] install and compile docs: "detailed information on how to download, install, and compile Raptor"
