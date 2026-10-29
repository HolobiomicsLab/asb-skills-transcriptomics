---
name: nucleotide-sequence-index-construction
description: Use when a very large collection of nucleotide sequences (e.
when_to_use_negative:
- Bin sizes are highly uneven or the number of bins is in the tens of thousands —
  the article names uneven bin sizes and a high number of bins as the main limiting
  factors, and several ten thousand bins lead to a moderate slowdown
- Indices must be shared across incompatible tool versions — indices built with an
  older version of Raptor cannot be used with version 2.0.0
- Only a small, fixed reference is queried and a full-text (FM-index) solution suffices
  — this skill targets ultra-large NGS databases where full-text indices cannot scale
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: Raptor
  role: 'Primary tool: builds the binning-directory index (x-partitioned IBF with
    winnowing minimizers and probabilistic thresholding) and performs approximate
    read searches'
  repo: https://github.com/seqan/raptor
- name: COBS
  role: 'State-of-the-art competitor baseline: index built at 64 and 1,024 bins with
    k=19 and 53% k-mer-lemma threshold for construction/search/memory/FP comparison'
  repo: https://github.com/bingmann/cobs
- name: Mantis
  role: 'State-of-the-art competitor baseline: index built with Squeaker preprocessing
    adapted to FASTQ quality scores; expected to segfault at 64 bins'
  repo: https://github.com/splatlab/mantis
- name: TaxSBP
  role: Hierarchical taxonomy-aware bin packing used to partition sequence collections
    into bins before indexing
  repo: https://github.com/pirovc/taxsbp
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:60fe36c7f62c707875eb8d164f1ab665d1d3fe1a34381ad7ec044df50587a519
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - COBS
  - Mantis
  - Raptor
  - TaxSBP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# nucleotide-sequence-index-construction

## Summary

Builds a searchable index over large collections of nucleotide sequences (binning directories based on extended Interleaved Bloom Filters, e.g. with Raptor) to enable fast approximate search of NGS reads or transcripts, and benchmarks index construction against state-of-the-art competitors like COBS and Mantis.

## When to use

Use when a very large collection of nucleotide sequences (e.g. 64 or 1,024 bins of RefSeq or artificial data, k=19) must be indexed so that many queries such as next-generation sequencing reads can be searched approximately, or when benchmarking index construction time, RAM, and disk usage against COBS/Mantis.

## When NOT to use

- Bin sizes are highly uneven or the number of bins is in the tens of thousands — the article names uneven bin sizes and a high number of bins as the main limiting factors, and several ten thousand bins lead to a moderate slowdown
- Indices must be shared across incompatible tool versions — indices built with an older version of Raptor cannot be used with version 2.0.0
- Only a small, fixed reference is queried and a full-text (FM-index) solution suffices — this skill targets ultra-large NGS databases where full-text indices cannot scale

## Inputs

- Collection of nucleotide sequences divided into bins (e.g. FASTA/FASTQ; artificial dataset with 64 and 1,024 bins, or clustered NCBI RefSeq bins)
- Bin layout / binning assignment (e.g. TaxSBP output with sequence id, start, end, taxonomic id, bin id)
- Query read set (e.g. FASTQ, 2^20 simulated reads)

## Outputs

- Constructed sequence-search index (binning directory / compressed, partitioned IBF)
- Construction time, peak RAM, and index disk-space measurements per tool and bin count
- Search times per tool and bin count for the query read set
- False-positive counts against known bin memberships
- Comparison table matching Table 4's layout

## How to apply

Split the sequence collection into bins (using, e.g., TaxSBP for taxonomy-aware bin packing, or fixed artificial bin layouts with 64 and 1,024 bins) and build an index with Raptor's `raptor build` command; Raptor uses winnowing minimizers to define representative k-mers, an x-partitioned extension of interleaved Bloom filters (IBFs), and probabilistic thresholding for minimizers, with k=19 and a 53% k-mer-lemma threshold in the paper's artificial-dataset setup. Compile Raptor from source for host-specific optimization. Record construction time, peak RAM, and resulting index disk space; the approach allows compression and partitioning of the IBF for effective use of secondary memory. For comparison, build COBS (with the same k and 53% k-mer-lemma threshold) and Mantis (with Squeaker preprocessing adapted to FASTQ quality scores) indices over the same bins, then query all indices with an identical set of 2^20 simulated reads and count false positives against known bin memberships. Assemble construction time, search time, RAM, disk space, and FP counts into a single comparison table (e.g. matching the layout of Table 4); Raptor's search takes about 0.9 s at 64 bins and 1.6 s at 1,024 bins, roughly 144x faster than COBS and about 30x faster than Mantis at 1,024 bins.

## Related tools

- **Raptor** (Primary tool: builds the binning-directory index (x-partitioned IBF with winnowing minimizers and probabilistic thresholding) and performs approximate read searches) — https://github.com/seqan/raptor
- **COBS** (State-of-the-art competitor baseline: index built at 64 and 1,024 bins with k=19 and 53% k-mer-lemma threshold for construction/search/memory/FP comparison) — https://github.com/bingmann/cobs
- **Mantis** (State-of-the-art competitor baseline: index built with Squeaker preprocessing adapted to FASTQ quality scores; expected to segfault at 64 bins) — https://github.com/splatlab/mantis
- **TaxSBP** (Hierarchical taxonomy-aware bin packing used to partition sequence collections into bins before indexing) — https://github.com/pirovc/taxsbp

## Examples

```
conda install -c bioconda -c conda-forge raptor; raptor build --help
```

## Evaluation signals

- The comparison table contains a measurement row for each tool (Raptor, COBS, Mantis) at both 64 and 1,024 bins, with Mantis recorded as failing (segmentation fault) at 64 bins
- At k=19 on the artificial dataset, Raptor search times fall between 0.9 s (64 bins) and 1.6 s (1,024 bins), about 144x faster than COBS and about 30x faster than Mantis at 1,024 bins
- COBS/Mantis construction times are at least three times longer than Raptor's, and recorded RAM/disk/FP values are comparable in magnitude to Table 4
- Index construction runs via `raptor build` and its help page is reachable (`raptor build --help`), confirming correct installation and command availability
- False-positive counts computed against the known bin memberships of the 2^20 simulated query reads are consistent across tool/bin-count configurations

## Limitations

- Raptor's main limiting factor is susceptibility to uneven bin sizes as well as a high number of bins; processing several ten thousand bins leads to a moderate slowdown
- An adaptive IBF that automatically adapts to inputs with uneven bin sizes is under development, not yet available; a hierarchical multi-level IBF is proposed but not shown or implemented
- For compressed bitvector construction, both the uncompressed and compressed bitvectors must be in main memory at some point, and since SDSL does not support decompression, the uncompressed bitvector must also be stored on disk for updates
- Indices built with an older version of Raptor cannot be used with version 2.0.0
- Mantis (Squeaker) segfaulted at 64 bins on the artificial dataset, and COBS/Mantis require at least three times longer construction

## Evidence

- [abstract] Raptor system scope: "We present Raptor, a system for approximately searching many queries such as next-generation sequencing reads or transcripts in large collections of nucleotide sequences."
- [intro] Indexing resource comparison baseline: "Kraken (Wood and Salzberg, 2014) needs ≈147 GiB RAM for indexing 380 GBases."
- [methods] Build command and help page: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [discussion] Main limiting factors: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
- [readme] Compile from source for optimization: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
