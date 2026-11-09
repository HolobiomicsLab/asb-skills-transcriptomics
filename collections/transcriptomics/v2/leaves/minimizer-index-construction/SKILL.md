---
name: minimizer-index-construction
description: Use when indexing very large nucleotide reference collections (e.g. NCBI
  RefSeq bins) for approximate read/sequence queries and memory, not CPU, is the binding
  constraint.
when_to_use_negative:
- The bin sizes are highly uneven or the number of bins is in the tens of thousands
  — the article names uneven bin sizes and a high number of bins as the main limiting
  factors, with a moderate slowdown for many bins
- The task is exact or edit-distance-sensitive full-text search where a word/membership
  pre-filter is inappropriate — the paper states word-based indices are not appropriate
  for edit-distance-based biological sequence searches and Raptor is a pre-filter,
  not a mapper
- You need to update or decompress an existing compressed IBF — SDSL does not support
  decompression of the compressed bitvector, so updates require the uncompressed bitvector
  to be kept on disk
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Raptor
  role: Builds the minimizer-reduced interleaved Bloom filter (MIBF) index via 'raptor
    build' and queries it via 'raptor search'
  repo: https://github.com/seqan/raptor
- name: DREAM-Yara
  role: Distributed read mapper re-implemented on top of Raptor's IBF/minimizer index
    to evaluate mapping time, memory, and unmapped-read counts
  repo: https://github.com/seqan/dream_yara
- name: TaxSBP
  role: Hierarchically clusters reference sequences (e.g. NCBI Taxonomy) into bins
    before index construction
  repo: https://github.com/pirovc/taxsbp
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:1e44cbfc389965572ac0e5d1e56a3da26c7e4dcb0c8131a433339ead2e7fd62d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - DREAM-Yara
  - Raptor
  - TaxSBP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# minimizer-index-construction

## Summary

Constructs compressed, minimizer-reduced binning indices (MIBFs) with Raptor, replacing full k-mer content of an interleaved Bloom filter (IBF) with winnowing minimizers so that very large nucleotide collections can be indexed and searched with far less RAM.

## When to use

Use when indexing very large nucleotide reference collections (e.g. NCBI RefSeq bins) for approximate read/sequence queries and memory, not CPU, is the binding constraint.

## When NOT to use

- The bin sizes are highly uneven or the number of bins is in the tens of thousands — the article names uneven bin sizes and a high number of bins as the main limiting factors, with a moderate slowdown for many bins
- The task is exact or edit-distance-sensitive full-text search where a word/membership pre-filter is inappropriate — the paper states word-based indices are not appropriate for edit-distance-based biological sequence searches and Raptor is a pre-filter, not a mapper
- You need to update or decompress an existing compressed IBF — SDSL does not support decompression of the compressed bitvector, so updates require the uncompressed bitvector to be kept on disk

## Inputs

- Partitioned reference nucleotide sequences in a Raptor binning directory (e.g. 64-bin or 1024-bin clustered NCBI RefSeq sets from Zenodo 10.5281/zenodo.4650188 and 10.5281/zenodo.4651078)
- Chosen minimizer parameters (k,w), e.g. (19,19), (23,19), (31,19)
- IBF size budget (e.g. 1–8 GiB in the artificial-dataset experiments)

## Outputs

- Minimizer-based interleaved Bloom filter index (MIBF) usable by 'raptor search'
- Optionally compressed IBF for secondary-memory use
- Measured build time and peak RAM per configuration

## How to apply

Install Raptor via bioconda ('conda install -c bioconda -c conda-forge raptor') or compile from source for host-system optimization. Use 'raptor build' to construct a binning-directory index over the partitioned reference bins, choosing minimizer (k,w) parameters — the paper compares (19,19), (23,19), and (31,19) — so that winnowing minimizers represent a reduced set of k-mers inside an extension of the interleaved Bloom filter, with probabilistic thresholding compensating for the missing k-mers. Larger window/mer ratios (e.g. (23,19), (31,19)) shrink the index: in the DREAM-Yara case study mapping memory fell from a 16 GiB IBF baseline to 17.5 GiB for (23,19) and 15.2 GiB for (31,19) with 64 bins (10.8 and 9 GiB with 1024 bins). Decide the k/w trade-off by checking that sensitivity is preserved: count unmapped reads after 'raptor search' and confirm they remain comparable to the non-minimizer baseline (only +2 and +14 unmapped reads in the reported case). Note that compression and partitioning of the IBF are supported for secondary-memory use, and indices built with an older Raptor version cannot be read by version 2.0.0.

## Related tools

- **Raptor** (Builds the minimizer-reduced interleaved Bloom filter (MIBF) index via 'raptor build' and queries it via 'raptor search') — https://github.com/seqan/raptor
- **DREAM-Yara** (Distributed read mapper re-implemented on top of Raptor's IBF/minimizer index to evaluate mapping time, memory, and unmapped-read counts) — https://github.com/seqan/dream_yara
- **TaxSBP** (Hierarchically clusters reference sequences (e.g. NCBI Taxonomy) into bins before index construction) — https://github.com/pirovc/taxsbp

## Examples

```
raptor build --help
```

## Evaluation signals

- False-positive bin assignments collapse once the IBF is large enough: on the artificial 64-bin dataset, FPs drop from 604,533 at 1 GiB to 189 at 2 GiB with (19,19)-minimizers — an index at 1 GiB with high FPs signals under-sizing
- Unmapped-read counts with minimizer indices stay within a few reads of the non-minimizer baseline (reported: +2 for (23,19) and +14 for (31,19) in the DREAM-Yara case study)
- Mapping memory at 64 bins reproduces the reported reduction: 17.5 GiB for (23,19)-minimizers and 15.2 GiB for (31,19)-minimizers (10.8 and 9 GiB at 1024 bins)
- Index is loadable by the same Raptor version that built it — indices built with an older version cannot be used with version 2.0.0
- 'raptor build' and 'raptor search' exit cleanly and report construction time and peak RAM consistent with the chosen IBF size (1–8 GiB range tested)

## Limitations

- Susceptible to uneven bin sizes; an IBF that automatically adapts to uneven bin sizes is under development and not yet available
- A high number of bins is a main limiting factor; processing several ten thousand bins leads to a moderate slowdown
- Compressed bitvector construction requires both uncompressed and compressed bitvectors in main memory at some point
- Indices built with an older version of Raptor cannot be used with version 2.0.0

## Evidence

- [abstract] Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs): "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for"
- [results] 604,533 FPs at 1 GiB vs 189 at 2 GiB: [evidence span withheld — anchor status: not_found]
- [results] mapping memory drops to 17.5 GiB: [evidence span withheld — anchor status: not_found]
- [readme] install via bioconda: "conda install -c bioconda -c conda-forge raptor"
- [readme] per-command help pages incl. raptor build: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [readme] compile from source for host optimization: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
