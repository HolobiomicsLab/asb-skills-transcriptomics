---
name: minimizer-binning-directory-construction
description: Use when a large nucleotide sequence collection (e.g., thousands of SRA
  human RNA-Seq experiments) must be indexed into bins so many queries (reads or transcripts)
  can be approximately searched for set membership faster and with less memory than
  full-text indices like Mantis or COBS.
when_to_use_negative:
- The collection has highly uneven bin sizes or a very high number of bins — the paper
  states uneven bin sizes and high bin counts are the main limiting factor, causing
  slowdown
- Exact, edit-distance-complete search over the full index is required — Raptor is
  a probabilistic pre-filter with a nonzero FP rate (~0.017 on the RNA-Seq benchmark),
  not a full-text index like Mantis
- The dataset is small enough that Mantis/COBS query times are already acceptable
  — the pre-filter's speed and memory advantages target very large collections (petabase-scale
  NGS databases)
edam_operation: http://edamontology.org/operation_2406
tools:
- name: Raptor
  role: Builds the minimizer-based binning directory ('raptor build') and searches
    it ('raptor search'); the index and binning directories are available in SeqAn
  repo: https://github.com/seqan/raptor
- name: TaxSBP
  role: Computes taxonomy-aware bin packing layouts that define which sequences go
    into which bins before 'raptor build'
  repo: https://github.com/pirovc/taxsbp
- name: Mantis
  role: State-of-the-art comparator index used as ground truth to compute Raptor's
    false-positive rate and benchmark construction/query resources
  repo: https://github.com/splatlab/mantis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:22881f15e83d5e653a14500c756a5d55063ea5baafec45a31cdc705d10eada2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - Mantis
  - Raptor
  - TaxSBP
  edam_operation: http://edamontology.org/operation_2406
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# minimizer-binning-directory-construction

## Summary

Construction of a Raptor binning directory: an x-partitioned Interleaved Bloom Filter (IBF) over winnowing minimizers with probabilistic thresholding, used to index very large collections of nucleotide sequences (e.g., thousands of RNA-Seq experiments) as a fast, space-efficient pre-filter for approximate sequence queries.

## When to use

Use when a large nucleotide sequence collection (e.g., thousands of SRA human RNA-Seq experiments) must be indexed into bins so many queries (reads or transcripts) can be approximately searched for set membership faster and with less memory than full-text indices like Mantis or COBS.

## When NOT to use

- The collection has highly uneven bin sizes or a very high number of bins — the paper states uneven bin sizes and high bin counts are the main limiting factor, causing slowdown
- Exact, edit-distance-complete search over the full index is required — Raptor is a probabilistic pre-filter with a nonzero FP rate (~0.017 on the RNA-Seq benchmark), not a full-text index like Mantis
- The dataset is small enough that Mantis/COBS query times are already acceptable — the pre-filter's speed and memory advantages target very large collections (petabase-scale NGS databases)

## Inputs

- Collection of nucleotide sequences (e.g., 1,742 human RNA-Seq experiments from Zenodo DOI 10.5281/zenodo.1186393)
- Binning layout / partitioning of sequences into bins (e.g., 64 or 1024 bins, optionally computed with TaxSBP)
- Minimizer scheme parameters (k-mer size k=20; shape/weight settings such as (20,20) or (40,20) minimizers)

## Outputs

- Raptor binning directory (IBF-based index) on disk, built with the chosen minimizer scheme
- Construction resource metrics: wall-clock time, peak RAM, and on-disk index size

## How to apply

Install Raptor (bioconda or compile from source, which can be optimized for the host system), then run 'raptor build' over the binning layout of the collection to produce a binning directory based on an extension of the Interleaved Bloom Filters combined with winnowing minimizers and probabilistic thresholding. Choose the minimizer scheme deliberately: the paper's real RNA-Seq benchmark uses k=20 with (20,20)- and (40,20)-minimizer settings, where (40,20)-minimizers trade a little sensitivity for speed (12-58x faster than Mantis) while keeping a low false-positive rate (~0.017) versus Mantis ground truth. Record wall-clock time, peak RAM, and on-disk index size at construction, since Raptor's selling point is using only a fraction of the competitors' main memory and it can further trade run time for main memory consumption. After building, validate by searching sampled transcript query sets (e.g., 100/1,000/10,000 human transcripts) with 'raptor search' and computing the false-positive rate against Mantis results as ground truth. Note the compression/partitioning options allow effective use of secondary memory for very large collections.

## Related tools

- **Raptor** (Builds the minimizer-based binning directory ('raptor build') and searches it ('raptor search'); the index and binning directories are available in SeqAn) — https://github.com/seqan/raptor
- **TaxSBP** (Computes taxonomy-aware bin packing layouts that define which sequences go into which bins before 'raptor build') — https://github.com/pirovc/taxsbp
- **Mantis** (State-of-the-art comparator index used as ground truth to compute Raptor's false-positive rate and benchmark construction/query resources) — https://github.com/splatlab/mantis

## Examples

```
conda install -c bioconda -c conda-forge raptor; raptor build --help
```

## Evaluation signals

- Construction resources (wall-clock time, peak RAM, on-disk size) fall within the ranges of the paper's benchmark tables for the same minimizer settings (k=20; (20,20) and (40,20))
- Query FPR against Mantis ground truth is low (~0.017) for the (40,20)-minimizer index across query scales of 100, 1,000, and 10,000 transcripts
- Index is usable with the installed Raptor version — indices built with an older version cannot be used with version 2.0.0
- Raptor search is substantially faster (12-58x vs Mantis) and uses only a fraction of Mantis's main memory on the same 1,742-experiment RNA-Seq collection
- Each CLI step is exercised via its help page (e.g., 'raptor build --help') to confirm correct parameterization before the real build

## Limitations

- Susceptible to uneven bin sizes and to a high number of bins; processing several ten thousand bins leads to a moderate slowdown
- Compressed bitvector construction requires both the uncompressed and compressed bitvectors in main memory at some point, and SDSL does not support decompression, so the uncompressed bitvector must also be stored on disk for updates
- Indices built with an older version of raptor cannot be used with version 2.0.0
- P(Cn) has no tractable formulation and was left as an open problem, estimated experimentally by sampling

## Evidence

- [abstract] Raptor uses winnowing minimizers and probabilistic thresholding: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for"
- [abstract] Compression and partitioning of the IBF: "Our approach allows compression and partitioning of the IBF to enable the effective use of secondary memory."
- [methods] raptor build help page: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [discussion] Uneven bins and many bins limit performance: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
- [discussion] Index version incompatibility: "Indices built with an older version of raptor cannot be used with version 2.0.0."
- [readme] Compile from source for host optimization: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
