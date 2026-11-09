---
name: sequence-search-benchmarking
description: Use when evaluating or comparing approximate sequence search tools (Raptor,
  COBS, Mantis) over large binned nucleotide collections and you need construction
  time, search speed, RAM/disk, and specificity measurements.
when_to_use_negative:
- When your goal is a single-tool deployment rather than a head-to-head speed/memory/FP
  comparison of Raptor, COBS, and Mantis
- When the input is a real, unevenly sized collection with a very high number of bins
  — Raptor's own discussion flags uneven bin sizes and high bin counts as its main
  limiting factors, so a benchmark on the artificial dataset would not represent that
  regime
- When edit-distance-tolerant search against un-binned, small references is needed
  — full-text/FM-index approaches may be more appropriate than pre-filter benchmarks
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: Raptor
  role: Tool under test; builds the binning-directory index (x-partitioned IBF with
    winnowing minimizers) whose search times and false positives are benchmarked
  repo: https://github.com/seqan/raptor
- name: COBS
  role: State-of-the-art competitor indexed at k=19 with a 53% k-mer-lemma threshold;
    construction time, RAM, disk, search time, and FPs recorded for comparison
  repo: https://github.com/bingmann/cobs
- name: Mantis
  role: State-of-the-art competitor built with Squeaker preprocessing adapted to FASTQ
    quality scores; expected to fail (segmentation fault) at 64 bins
  repo: https://github.com/splatlab/mantis
- name: TaxSBP
  role: Hierarchical bin-packing utility for binning/organizing reference sequences
    (e.g., NCBI RefSeq) into bins before index construction
  repo: https://github.com/pirovc/taxsbp
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:ee2794f590a2a29107ec7e80ef1d48df5d5dfb98076f99eeec5767fb8c5b5041
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - COBS
  - Mantis
  - Raptor
  - TaxSBP
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# sequence-search-benchmarking

## Summary

Benchmarking of approximate nucleotide sequence search tools (Raptor, COBS, Mantis) on binned reference collections, recording index construction time, search time, memory/disk usage, and false positives so pre-filter performance can be compared fairly.

## When to use

Use when evaluating or comparing approximate sequence search tools (Raptor, COBS, Mantis) over large binned nucleotide collections and you need construction time, search speed, RAM/disk, and specificity measurements.

## When NOT to use

- When your goal is a single-tool deployment rather than a head-to-head speed/memory/FP comparison of Raptor, COBS, and Mantis
- When the input is a real, unevenly sized collection with a very high number of bins — Raptor's own discussion flags uneven bin sizes and high bin counts as its main limiting factors, so a benchmark on the artificial dataset would not represent that regime
- When edit-distance-tolerant search against un-binned, small references is needed — full-text/FM-index approaches may be more appropriate than pre-filter benchmarks

## Inputs

- Artificial dataset with 64 and 1,024 bins (downloadable from the Raptor repository)
- Set of 2^20 simulated query reads with known bin memberships
- Bin definitions for reference sequences (optionally produced with TaxSBP)
- Installed copies of Raptor, COBS (k=19, 53% k-mer-lemma threshold), and Mantis (Squeaker preprocessing)

## Outputs

- Comparison table (Table 4 layout) with one measurement row per tool and bin count (64 and 1,024 bins)
- Per-configuration index construction time, peak RAM, and index disk size
- Per-configuration search times over the identical 2^20 query reads
- False-positive counts per tool/bin-count configuration

## How to apply

Obtain the benchmark collections (e.g., the artificial dataset with 64 and 1,024 bins from github.com/seqan/raptor) and bin the reference sequences consistently — TaxSBP can be used for taxonomy-aware bin packing. Build each index with matched parameters: for the Raptor paper's comparison, k=19 with a 53% k-mer lemma threshold for COBS, and Mantis built with Squeaker preprocessing adapted to FASTQ quality scores. Query every index with the identical set of simulated reads (2^20 reads in the paper) and record, per tool and bin count, construction time, search time, peak RAM, index disk space, and the number of false positives counted against known query-to-bin memberships. Assemble all measurements into a single comparison table matching the layout of the paper's Table 4, with expected failures (e.g., Mantis segmentation fault at 64 bins) explicitly recorded.

## Related tools

- **Raptor** (Tool under test; builds the binning-directory index (x-partitioned IBF with winnowing minimizers) whose search times and false positives are benchmarked) — https://github.com/seqan/raptor
- **COBS** (State-of-the-art competitor indexed at k=19 with a 53% k-mer-lemma threshold; construction time, RAM, disk, search time, and FPs recorded for comparison) — https://github.com/bingmann/cobs
- **Mantis** (State-of-the-art competitor built with Squeaker preprocessing adapted to FASTQ quality scores; expected to fail (segmentation fault) at 64 bins) — https://github.com/splatlab/mantis
- **TaxSBP** (Hierarchical bin-packing utility for binning/organizing reference sequences (e.g., NCBI RefSeq) into bins before index construction) — https://github.com/pirovc/taxsbp

## Examples

```
conda install -c bioconda -c conda-forge raptor && raptor build --help
```

## Evaluation signals

- The comparison table contains a measurement row for each tool (Raptor, COBS, Mantis) at both 64 and 1,024 bins
- Mantis is explicitly recorded as failing (segmentation fault) at 64 bins, matching the paper's Table 4
- Recorded Raptor search times fall in the reported range of ~0.9 s (64 bins) to ~1.6 s (1,024 bins) at k=19, i.e., roughly 144x faster than COBS and ~30x faster than Mantis at 1,024 bins
- COBS/Mantis construction times are at least three times longer than Raptor's, and recorded RAM/disk/FP values are comparable in magnitude to Table 4
- All configurations were queried with the identical set of 2^20 simulated reads and false positives were counted against known bin memberships

## Limitations

- Raptor's main limiting factor is susceptibility to uneven bin sizes as well as a high number of bins
- Processing several ten thousand bins leads to a moderate slowdown
- Indices built with an older version of raptor cannot be used with version 2.0.0, so benchmark results may not be reproducible across major versions
- Mantis (Squeaker) segmentation fault at 64 bins means the 64-bin comparison cell cannot be filled for that tool

## Evidence

- [intro] We present our implementation, called Raptor, discuss its capabilities and limitations, and compare it with the state-of-the-art methods Mantis and COBS.: "We present our implementation, called Raptor, discuss its capabilities and limitations, and compare it with the state-of-the-art methods Mantis and COBS."
- [discussion] Raptor limiting factors: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
- [discussion] Version incompatibility caveat: "Indices built with an older version of raptor cannot be used with version 2.0.0."
- [readme] Raptor installation from README: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
