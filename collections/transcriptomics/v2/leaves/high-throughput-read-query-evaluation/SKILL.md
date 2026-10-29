---
name: high-throughput-read-query-evaluation
description: Use when you need to characterize how partitioning a large IBF index
  (e.g., 8 GiB) into x parts affects wall-clock build time, query time, peak memory,
  and specificity, before choosing a partition count for large-scale read search.
when_to_use_negative:
- When the goal is to compare Raptor against competitor tools (Mantis, COBS) on real
  RefSeq data rather than to characterize the internal x-PIBF partitioning trade-off.
- When the index is small enough to fit comfortably in main memory unpartitioned —
  partitioning only pays off when secondary-memory use is needed.
- When bins have highly uneven sizes; the paper notes susceptibility to uneven bin
  sizes is a main limiting factor.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: Raptor
  role: Builds the x-partitioned IBF indices ('raptor build') and searches the simulated
    reads against them ('raptor search') for each of the 16 configurations.
  repo: https://github.com/seqan/raptor
- name: TaxSBP
  role: Not used in this benchmark; only relevant if real-data bins must be re-clustered
    (the artificial dataset already comes pre-partitioned into 64/1,024 bins).
  repo: https://github.com/pirovc/taxsbp
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:841357c6e162cbb3097269c67ea1f7ca4f5fe5ec9a49c409e5ecdfbc7d292dc3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - Raptor
  - TaxSBP
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# high-throughput-read-query-evaluation

## Summary

Benchmark methodology for quantifying the x-partitioned Interleaved Bloom Filter (x-PIBF) run-time vs. memory trade-off: building an 8 GiB Raptor index as x=1, 2, 4, or 8 parts and measuring construction time, search time, peak RAM, and false-positive assignments across bin counts and minimizer shapes.

## When to use

Use when you need to characterize how partitioning a large IBF index (e.g., 8 GiB) into x parts affects wall-clock build time, query time, peak memory, and specificity, before choosing a partition count for large-scale read search.

## When NOT to use

- When the goal is to compare Raptor against competitor tools (Mantis, COBS) on real RefSeq data rather than to characterize the internal x-PIBF partitioning trade-off.
- When the index is small enough to fit comfortably in main memory unpartitioned — partitioning only pays off when secondary-memory use is needed.
- When bins have highly uneven sizes; the paper notes susceptibility to uneven bin sizes is a main limiting factor.

## Inputs

- Artificial dataset with 64 bins (binning file/bin lists from Raptor repository materials)
- Artificial dataset with 1,024 bins
- Simulated query reads (2^20 reads) FASTA/FASTQ
- Raptor binary (installed via bioconda or compiled from source)

## Outputs

- 16 built indices (2 bin counts x 2 minimizer variants x 4 partition levels x=1,2,4,8)
- Per-configuration wall-clock construction time, wall-clock search time, and peak RAM measurements
- Per-configuration false-positive read assignment counts
- Single TSV table reproducing the layout of Table 2

## How to apply

Obtain the artificial dataset (64 and 1,024 bins) with its simulated query reads (2^20 reads) from the Raptor repository materials, then for each bin count and each minimizer variant ((19,19) and (23,19)) build an IBF of fixed 8 GiB size with 'raptor build' in the unpartitioned (x=1), 2-, 4-, and 8-partitioned configurations — 16 index builds total. Wrap each build and search call in '/usr/bin/time -v' to capture wall-clock construction time, wall-clock search time, and peak RAM. Query the simulated reads against each index using 'raptor search' and record the number of false-positive read assignments per configuration to quantify specificity. Assemble all 16 configurations' timing, memory, and false-positive measurements into a single TSV table reproducing the layout of Table 2. The rationale is the paper's claim that 'we only use a fraction of the memory and can further trade run time for main memory consumption': more parts (e.g., 8-PIBF at ~1.4 GiB per part) reduce memory needed to hold a single part in RAM at the cost of increased build and query run time.

## Related tools

- **Raptor** (Builds the x-partitioned IBF indices ('raptor build') and searches the simulated reads against them ('raptor search') for each of the 16 configurations.) — https://github.com/seqan/raptor
- **TaxSBP** (Not used in this benchmark; only relevant if real-data bins must be re-clustered (the artificial dataset already comes pre-partitioned into 64/1,024 bins).) — https://github.com/pirovc/taxsbp

## Examples

```
/usr/bin/time -v raptor build --input bins.list --parts 8 --size 8gib --output ibf_8parts.index
```

## Evaluation signals

- Exactly 16 index builds are completed: 2 bin counts (64, 1,024) x 2 minimizer shapes ((19,19), (23,19)) x 4 partition levels (x=1, 2, 4, 8).
- Total IBF size is held fixed at 8 GiB across all configurations; per-part memory decreases with x (e.g., ~8 GiB for 1-PIBF vs ~1.4 GiB for 8-PIBF).
- Construction and search wall-clock times should be monotonically non-decreasing as x increases, consistent with the reported trade-off.
- Peak RAM measurements from '/usr/bin/time -v' are present for every build and search call.
- False-positive read assignment counts are recorded per configuration and show no systematic degradation with partition count (partitioning should not affect specificity).

## Limitations

- Indices built with an older version of raptor cannot be used with version 2.0.0.
- The method is susceptible to uneven bin sizes and to a high number of bins; processing several ten thousand bins leads to a moderate slowdown.
- There may be performance benefits when compiling from source, as Raptor can be optimized for the host system — timing results are host-dependent.
- Compiling/installing requires following the SeqAn documentation; each command's options must be checked via its help page.

## Evidence

- [intro] partitioning trade-off rationale: "In addition, we only use a fraction of the memory and can further trade run time for main memory consumption."
- [abstract] IBF partitioning feature: "Our approach allows compression and partitioning of the IBF to enable the effective use of secondary memory."
- [methods] command-line usage: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [discussion] version compatibility caveat: "Indices built with an older version of raptor cannot be used with version 2.0.0."
- [discussion] artificial dataset availability: "Artificial data set (64 and 1024 bins)This paperhttps://github.com/seqan/raptor"
