---
name: nucleotide-read-mapping-benchmarking
description: Use when benchmarking a minimizer-based pre-filter/read mapper against
  a k-mer baseline, or when validating that smaller interleaved Bloom filters (e.g.,
  15.2 GiB vs 16 GiB at 64 bins) do not cost mapping sensitivity.
when_to_use_negative:
- The goal is biological mapping of new reads rather than benchmarking memory/time/sensitivity
  trade-offs of IBF-based pre-filters
- Raptor indices were built with an older version — 'Indices built with an older version
  of raptor cannot be used with version 2.0.0.'
- Bin sizes are highly uneven, since the paper notes 'The main limiting factor of
  our method is its susceptibility to uneven bin sizes as well as a high number of
  bins.'
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Raptor
  role: Provides the minimizer-based IBF pre-filter and the DREAM-Yara re-implementation
    used for index building and querying during the benchmark
  repo: https://github.com/seqan/raptor
- name: DREAM-Yara
  role: Original distributed read mapper providing the 19-mer baseline index against
    which the minimizer re-implementation is compared
  repo: https://github.com/seqan/dream_yara
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:76b14609de39a874c36241e09cd047078ffc9301878024a4b1ec9cb3f235b79b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - DREAM-Yara
  - Raptor
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# nucleotide-read-mapping-benchmarking

## Summary

Benchmark protocol for the Raptor-based DREAM-Yara re-implementation, comparing IBF build time, mapping time, peak RAM, and unmapped-read counts between the original 19-mer index and (23,19)/(31,19)-minimizer indices on clustered NCBI RefSeq bin sets.

## When to use

Use when benchmarking a minimizer-based pre-filter/read mapper against a k-mer baseline, or when validating that smaller interleaved Bloom filters (e.g., 15.2 GiB vs 16 GiB at 64 bins) do not cost mapping sensitivity.

## When NOT to use

- The goal is biological mapping of new reads rather than benchmarking memory/time/sensitivity trade-offs of IBF-based pre-filters
- Raptor indices were built with an older version — 'Indices built with an older version of raptor cannot be used with version 2.0.0.'
- Bin sizes are highly uneven, since the paper notes 'The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins.'

## Inputs

- Clustered NCBI RefSeq reference bin sets (64 bins, Zenodo doi:10.5281/zenodo.4650188; 1,024 bins, Zenodo doi:10.5281/zenodo.4651078)
- Artificial query read sets corresponding to the 64- and 1,024-bin layouts
- DREAM-Yara configuration parameters: 19-mer baseline within a 16 GiB IBF; minimizer shapes (k,w) = (19,19), (23,19), (31,19)

## Outputs

- Per-configuration IBF build time and peak RAM
- Per-configuration mapping wall-clock time and peak RAM (e.g., 17.5 GiB for (23,19) and 15.2 GiB for (31,19) at 64 bins; 10.8 and 9 GiB at 1,024 bins)
- Unmapped-read counts for every configuration
- A consolidated results table reproducing Table 7

## How to apply

Install the minimizer-enabled DREAM-Yara re-implementation via bioconda ('conda install -c bioconda -c conda-forge raptor'). Download the clustered NCBI RefSeq bin sets from Zenodo (doi:10.5281/zenodo.4650188 for 64 bins and doi:10.5281/zenodo.4651078 for 1,024 bins) plus the artificial query read sets. Build the baseline index with original DREAM-Yara using 19-mers within a 16 GiB interleaved Bloom filter, then build minimizer indices with (k,w) = (19,19), (23,19), and (31,19). Map the query reads against each index while profiling wall-clock time and peak RAM for both index construction and mapping, and count unmapped reads per configuration. Compile build time/RAM, mapping time/RAM, and unmapped-read counts into a single results table matching Table 7, judging success by the reported memory reduction (17.5 GiB and 15.2 GiB at 64 bins for (23,19) and (31,19)) with unmapped-read counts comparable to the baseline.

## Related tools

- **Raptor** (Provides the minimizer-based IBF pre-filter and the DREAM-Yara re-implementation used for index building and querying during the benchmark) — https://github.com/seqan/raptor
- **DREAM-Yara** (Original distributed read mapper providing the 19-mer baseline index against which the minimizer re-implementation is compared) — https://github.com/seqan/dream_yara

## Examples

```
conda install -c bioconda -c conda-forge raptor
```

## Evaluation signals

- Mapping memory at 64 bins reproduces the reported drop to 17.5 GiB for (23,19)-minimizers and 15.2 GiB for (31,19)-minimizers
- Mapping memory at 1,024 bins reproduces the reported 10.8 GiB and 9 GiB values
- Unmapped-read counts increase only by 2 (for (23,19)) and 14 (for (31,19)) relative to the original 19-mer DREAM-Yara baseline
- Baseline 19-mer IBF is built within the reported 16 GiB configuration
- Results table columns and configurations match Table 7 (build time/RAM, mapping time/RAM, unmapped reads per (k,w) setting)

## Limitations

- Susceptibility to uneven bin sizes and a high number of bins; processing several ten thousand bins leads to a moderate slowdown
- Raptor indices built with an older version cannot be used with version 2.0.0
- Pre-built DREAM-Yara indices are hosted on an external FTP server (https://ftp.imp.fu-berlin.de/pub/raptor/), not a versioned archive

## Evidence

- [abstract] minimizer-based DREAM-Yara re-implementation rationale: "We show this by re-implementing the distributed read mapping tool DREAM-Yara."
- [abstract] technical basis of Raptor: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for"
- [methods] installation command grounding: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [readme] build-from-source recommendation: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
- [results] key benchmark result: [evidence span withheld — anchor status: not_found]
