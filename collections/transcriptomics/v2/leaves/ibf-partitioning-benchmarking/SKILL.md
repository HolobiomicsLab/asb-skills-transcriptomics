---
name: ibf-partitioning-benchmarking
description: Use when you must quantify how partitioning a large fixed-size IBF index
  (e.g., 8 GiB) into x parts trades wall-clock construction/search time against peak
  RAM, as in Table 2 of the Raptor paper.
when_to_use_negative:
- You need the real-data comparison versus Mantis or COBS rather than the artificial
  partitioning trade-off benchmark (use the NCBI RefSeq / NIH tissue datasets instead)
- The index fits comfortably in main memory unpartitioned — partitioning only buys
  reduced per-part memory at the cost of run time
- Input bins are highly uneven in size — the paper notes susceptibility to uneven
  bin sizes as a main limiting factor
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Raptor
  role: Builds the x-partitioned IBF indices ('raptor build') and queries the simulated
    reads against them ('raptor search') across x=1,2,4,8 partitions
  repo: https://github.com/seqan/raptor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:80997ac05067d48457ff676e5546e99859d64e457bd7271e46a7a5c4ef9c2ed8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - Raptor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ibf-partitioning-benchmarking

## Summary

Benchmark the x-partitioned Interleaved Bloom Filter (x-PIBF) trade-off from Raptor: building and searching an 8 GiB IBF split into x=1, 2, 4, and 8 parts, measuring how increasing partitions raises run time while shrinking the per-part memory footprint.

## When to use

Use when you must quantify how partitioning a large fixed-size IBF index (e.g., 8 GiB) into x parts trades wall-clock construction/search time against peak RAM, as in Table 2 of the Raptor paper.

## When NOT to use

- You need the real-data comparison versus Mantis or COBS rather than the artificial partitioning trade-off benchmark (use the NCBI RefSeq / NIH tissue datasets instead)
- The index fits comfortably in main memory unpartitioned — partitioning only buys reduced per-part memory at the cost of run time
- Input bins are highly uneven in size — the paper notes susceptibility to uneven bin sizes as a main limiting factor

## Inputs

- Artificial dataset with 64 bins and 1,024 bins (from github.com/seqan/raptor supplementary materials)
- 2^20 simulated query reads (FASTA/FASTQ)
- Raptor build configuration: fixed 8 GiB index size, minimizer shapes (19,19) and (23,19), partition counts x=1,2,4,8

## Outputs

- 16 built IBF/x-PIBF indices (8 GiB each)
- Wall-clock construction time, wall-clock search time, and peak RAM per configuration (e.g., from /usr/bin/time -v)
- False-positive read assignment counts per configuration
- TSV table reproducing the layout of Table 2 (rows/columns = bin count, minimizer variant, x partitioning)

## How to apply

Obtain the artificial dataset (64 and 1,024 bins) with its simulated query reads (2^20 reads) from the Raptor repository materials. For each bin count (b=64, b=1,024) and each minimizer setting ((19,19) and (23,19)), build an 8 GiB IBF with 'raptor build' in the unpartitioned (x=1), 2-partitioned, 4-partitioned, and 8-partitioned configurations, giving 16 index builds total. Query each of the 16 indices against the simulated reads with 'raptor search'. Wrap every build and search call in '/usr/bin/time -v' to capture wall-clock time and peak RAM, and record false-positive read assignments per configuration to quantify specificity. Assemble the 16 configurations' timing, memory, and false-positive measurements into a TSV laid out like Table 2; the expected pattern is more parts -> longer construction and query time but less memory per part (8 GiB for a 1-PIBF vs. ~1.4 GiB for an 8-PIBF).

## Related tools

- **Raptor** (Builds the x-partitioned IBF indices ('raptor build') and queries the simulated reads against them ('raptor search') across x=1,2,4,8 partitions) — https://github.com/seqan/raptor

## Examples

```
raptor build --help
```

## Evaluation signals

- Exactly 16 index builds are produced: 2 bin counts (64, 1,024) x 2 minimizer variants ((19,19), (23,19)) x 4 partition levels (x=1,2,4,8)
- All indices have the same fixed 8 GiB size regardless of x, so memory comparisons reflect per-part footprint only
- Peak memory for holding a single part decreases monotonically with x (e.g., ~8 GiB at x=1 vs. ~1.4 GiB at x=8)
- Construction and search times increase (or at least do not decrease) as x grows
- All measurements are captured consistently via /usr/bin/time -v and assembled into a TSV matching Table 2's row/column layout

## Limitations

- Raptor's main limiting factor is susceptibility to uneven bin sizes as well as a high number of bins
- Compressed bitvector construction requires both the uncompressed and compressed bitvectors in main memory at some point during construction, capping achievable memory savings at build time
- Indices built with an older version of Raptor cannot be used with version 2.0.0 — use a consistent version across all 16 builds
- Processing several ten thousand bins leads to a moderate slowdown

## Evidence

- [abstract] Partitioning reduces memory at run-time cost: "Our approach allows compression and partitioning of the IBF to enable the effective use of secondary memory."
- [intro] Run time / memory trade-off motivation: "In addition, we only use a fraction of the memory and can further trade run time for main memory consumption."
- [readme] CLI help grounding for raptor build: "`raptor --help` will show available commands. Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [readme] Compile-from-source performance note: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
- [discussion] Uneven bin sizes limitation: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
