---
name: false-positive-rate-assessment
description: Use when evaluating how the size of a Raptor interleaved Bloom filter
  (e.g. 1, 2, 4, 8 GiB), minimizer choice, or compression affects the number of false-positive
  bin assignments and run time/memory when searching large read collections.
when_to_use_negative:
- The input is real, unevenly sized bins rather than the artificial equal-size dataset
  — the paper flags susceptibility to uneven bin sizes as a main limiting factor,
  and the adaptive IBF was not yet available
- The index was built with an older Raptor version — such indices cannot be used with
  version 2.0.0, so FP measurements would not be comparable
- The analysis goal is edit-distance-sensitive search rather than approximate pre-filtering;
  Raptor is a pre-filter, not a full-text search index
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: Raptor
  role: Used to build (raptor build) and search (raptor search) IBF/MIBF binning-directory
    indices whose false-positive bin assignments are counted across IBF sizes, minimizers,
    and compression settings
  repo: https://github.com/seqan/raptor
- name: TaxSBP
  role: Related binning tool referenced by the article's repository ecosystem; not
    used directly in this FP benchmark
  repo: https://github.com/pirovc/taxsbp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:27d589331a3c1dc8e830238cb18b44017a1f40c400b3c85cfcc22eb01a9ab954
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - Raptor
  - TaxSBP
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# false-positive-rate-assessment

## Summary

Benchmarks the false-positive bin-assignment behavior of Raptor binning-directory indices (IBF/MIBF) by sweeping IBF size (1–8 GiB), minimizer shape ((19,19) vs (23,19)), and compression, then searching a fixed query set (e.g. 2^20 simulated reads) and counting spurious bin hits.

## When to use

Use when evaluating how the size of a Raptor interleaved Bloom filter (e.g. 1, 2, 4, 8 GiB), minimizer choice, or compression affects the number of false-positive bin assignments and run time/memory when searching large read collections.

## When NOT to use

- The input is real, unevenly sized bins rather than the artificial equal-size dataset — the paper flags susceptibility to uneven bin sizes as a main limiting factor, and the adaptive IBF was not yet available
- The index was built with an older Raptor version — such indices cannot be used with version 2.0.0, so FP measurements would not be comparable
- The analysis goal is edit-distance-sensitive search rather than approximate pre-filtering; Raptor is a pre-filter, not a full-text search index

## Inputs

- Artificial dataset binned into 64 or 1,024 bins (github.com/seqan/raptor)
- Query set of 2^20 simulated reads (FASTA/FASTQ)
- Raptor binning-directory indices built with (19,19) and (23,19) minimizers at 1, 2, 4, and 8 GiB, uncompressed and compressed

## Outputs

- Performance table (e.g. table1.csv) with construction time, search time, peak RAM, and false-positive bin assignments per index configuration
- False-positive counts per (bin count, IBF size, minimizer, compression) configuration

## How to apply

Install Raptor via 'conda install -c bioconda -c conda-forge raptor' or compile from source for host-system optimization. With 'raptor build', construct (19,19)-IBF and (23,19)-MIBF binning directories over the artificial 64-bin and 1,024-bin datasets for each IBF size (1, 2, 4, 8 GiB) in uncompressed and compressed form. For each index, run 'raptor search' against the 2^20 simulated reads, recording construction time, search time, peak RAM, and false-positive bin assignments. Compile the measurements into a single table (e.g. table1.csv) with one row per configuration. Key decision point: with (19,19)-minimizers, a 1 GiB IBF yields excessive false positives (604,533 FPs for b=64), so 2 GiB is the minimum size that reduces FPs to near zero (189 for b=64).

## Related tools

- **Raptor** (Used to build (raptor build) and search (raptor search) IBF/MIBF binning-directory indices whose false-positive bin assignments are counted across IBF sizes, minimizers, and compression settings) — https://github.com/seqan/raptor
- **TaxSBP** (Related binning tool referenced by the article's repository ecosystem; not used directly in this FP benchmark) — https://github.com/pirovc/taxsbp

## Examples

```
conda install -c bioconda -c conda-forge raptor; raptor build --help; raptor search --help
```

## Evaluation signals

- table1.csv exists and contains one row per index configuration (2 bin counts × 4 IBF sizes × 2 minimizers × 2 compression states)
- FP counts at IBF ≥ 2 GiB drop to near zero (e.g. 189 FPs for b=64 at 2 GiB) matching the reported near-zero false-positive performance on simulated data
- 1 GiB (19,19)-IBF configurations reproduce the anomalously high FP counts (e.g. 604,533 FPs for b=64)
- Recorded construction/search times and peak RAM values fall in plausible ranges and are internally consistent across configurations
- raptor search reports zero unexpected hits for reads simulated from the 64/1,024 artificial bins

## Limitations

- Main limiting factor is susceptibility to uneven bin sizes and a high number of bins; processing several ten thousand bins leads to a moderate slowdown
- Compressed bitvector construction requires both uncompressed and compressed bitvectors in main memory, inflating peak RAM during build
- Indices built with an older version of Raptor cannot be used with version 2.0.0
- 1 GiB IBF with (19,19)-minimizers produces very high false-positive bin assignments; small filters are unreliable

## Evidence

- [abstract] We test and show the performance and limitations of the new features using simulated and real datasets.: "We test and show the performance and limitations of the new features using simulated and real datasets."
- [methods] raptor build help: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [readme] compile from source: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
- [discussion] limiting factors: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
