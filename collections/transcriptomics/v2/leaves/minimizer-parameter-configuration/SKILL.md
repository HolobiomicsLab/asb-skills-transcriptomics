---
name: minimizer-parameter-configuration
description: Use when building a Raptor binning-directory index and you must choose
  a winnowing minimizer variant (e.g., (19,19) vs. (23,19)) and an x-partitioned IBF
  (x=1, 2, 4, 8) to balance run time, memory, and specificity.
when_to_use_negative:
- The goal is a cross-tool benchmark against Mantis or COBS on real RefSeq data —
  use the published Zenodo datasets (64/1,024-bin clustered RefSeq) rather than the
  artificial dataset designed for the x-PIBF trade-off.
- The input has highly uneven bin sizes — the paper notes uneven bin sizes are a main
  limiting factor and an adaptive IBF was not yet available.
- Only a single unpartitioned index is needed with no memory constraint — configuring
  partitions adds run time without benefit.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Raptor
  role: Builds the x-partitioned IBF index ('raptor build') and queries the simulated
    reads against it ('raptor search'); supports partitioning to trade run time for
    main memory.
  repo: https://github.com/seqan/raptor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:61a449326515564a296ea177c0b708ba85069552da5aebcb49d0e214ab389642
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - Raptor
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# minimizer-parameter-configuration

## Summary

Configures winnowing minimizer parameters (k, w) and x-partitioning of the Interleaved Bloom Filter (IBF) for Raptor indices, trading construction/search run time against peak memory. This reproduces the parameter sensitivity behavior reported for the 8 GiB x-PIBF benchmark.

## When to use

Use when building a Raptor binning-directory index and you must choose a winnowing minimizer variant (e.g., (19,19) vs. (23,19)) and an x-partitioned IBF (x=1, 2, 4, 8) to balance run time, memory, and specificity.

## When NOT to use

- The goal is a cross-tool benchmark against Mantis or COBS on real RefSeq data — use the published Zenodo datasets (64/1,024-bin clustered RefSeq) rather than the artificial dataset designed for the x-PIBF trade-off.
- The input has highly uneven bin sizes — the paper notes uneven bin sizes are a main limiting factor and an adaptive IBF was not yet available.
- Only a single unpartitioned index is needed with no memory constraint — configuring partitions adds run time without benefit.

## Inputs

- Artificial dataset of reference sequences split into 64 and 1,024 bins (from the Raptor repository materials)
- 2^20 simulated query reads matching the binned references
- Minimizer parameter choices: (k=19, w=19) and (k=23, w=19)
- Partitioning factor x in {1, 2, 4, 8} with a fixed total IBF size of 8 GiB

## Outputs

- 16 built Raptor indices (x-partitioned IBFs) covering both bin counts and both minimizer variants
- TSV table reproducing Table 2 with construction time, search time, peak RAM, and false-positive counts per configuration
- Per-configuration false-positive read assignment counts (specificity measurements)

## How to apply

Obtain the artificial dataset (64 and 1,024 bins) with its simulated query reads from the Raptor repository materials. For each bin count (b=64, b=1,024) and each minimizer variant ((19,19) and (23,19)), build an IBF of fixed 8 GiB size with 'raptor build' in the unpartitioned (x=1), 2-, 4-, and 8-partitioned (x-PIBF) configurations — 16 index builds total. Query the 2^20 simulated reads against each index with 'raptor search', wrapping each call in /usr/bin/time -v to capture wall-clock construction time, search time, and peak RAM. Record false-positive read assignments per configuration to quantify specificity, and assemble all measurements into a single TSV laid out like Table 2. The rationale: partitioning increases construction and query run time but reduces the memory needed to hold a single part in memory (e.g., 8 GiB for a 1-PIBF vs. 1.4 GiB for an 8-PIBF).

## Related tools

- **Raptor** (Builds the x-partitioned IBF index ('raptor build') and queries the simulated reads against it ('raptor search'); supports partitioning to trade run time for main memory.) — https://github.com/seqan/raptor

## Examples

```
raptor build --help
```

## Evaluation signals

- Exactly 16 index builds are produced: 2 bin counts (64, 1,024) x 2 minimizer variants ((19,19), (23,19)) x 4 partitioning levels (x=1, 2, 4, 8), all at a fixed 8 GiB index size.
- Peak memory of the loaded index part decreases monotonically with x (e.g., ~8 GiB for a 1-PIBF vs. ~1.4 GiB for an 8-PIBF), while construction and search times increase with x.
- Search wall-clock times and false-positive counts are recorded for every configuration and fit the row/column layout of Table 2 in a single TSV.
- Measurements come from a reproducible resource monitor (/usr/bin/time -v) around each build and search call, not ad hoc timing.
- Specificity is quantified: false-positive read assignments per configuration are nonzero-tracked and compared across configurations.

## Limitations

- Raptor is susceptible to uneven bin sizes and a high number of bins; processing several ten thousand bins leads to a moderate slowdown.
- Compressed bitvector construction requires both uncompressed and compressed bitvectors in main memory at some point; SDSL does not support decompression, so the uncompressed bitvector must also be stored on disk.
- Indices built with an older version of raptor cannot be used with version 2.0.0.

## Evidence

- [abstract] Raptor uses winnowing minimizers: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for"
- [abstract] Partitioning enables secondary-memory use: "Our approach allows compression and partitioning of the IBF to enable the effective use of secondary memory."
- [intro] Run time vs. memory trade-off: "In addition, we only use a fraction of the memory and can further trade run time for main memory consumption."
- [readme] CLI help for build parameters: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
