---
name: interleaved-bloom-filter-indexing
description: Use when many NGS reads or transcripts must be searched against a large
  binned collection (e.g., 64/1,024-bin NCBI RefSeq sets) and an IBF index with (k,w)-minimizers
  must be built and searched.
when_to_use_negative:
- The reference collection has severely uneven bin sizes or a very high number of
  bins, where Raptor's authors note susceptibility to uneven bin sizes and a moderate
  slowdown when processing several ten thousand bins
- Exact full-text search or edit-distance-aware mapping results are required directly
  — the IBF is a pre-filter whose assignments must be verified by a downstream mapper
- An existing index was built with a Raptor version older than 2.0.0 and cannot be
  reused; it must be rebuilt
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Raptor
  role: Builds the interleaved Bloom filter / minimizer IBF binning directory (`raptor
    build`) and queries reads against it (`raptor search`), reporting bin assignments,
    run time, and peak RAM.
  repo: https://github.com/seqan/raptor
- name: DREAM-Yara
  role: Downstream distributed read mapper re-implemented on top of Raptor's IBF and
    minimizers to measure mapping time, memory (e.g., 17.5 GiB with (23,19)-minimizers,
    64 bins), and unmapped-read counts.
  repo: https://github.com/seqan/dream_yara
- name: TaxSBP
  role: Hierarchically clusters reference sequences (e.g., NCBI RefSeq by taxonomy)
    into bins that define the binning directory structure indexed by the IBF.
  repo: https://github.com/pirovc/taxsbp
provenance:
  source_task_ids:
  - task_006
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:b931617c36b8b00b0bd1fe43d64f162fd5ca3bb6e9fdc0ccd888debad13294b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - DREAM-Yara
  - Raptor
  - TaxSBP
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# interleaved-bloom-filter-indexing

## Summary

Build and query compressed, partitioned Interleaved Bloom Filter (IBF) binning directories over large nucleotide sequence collections using Raptor's winnowing minimizers and probabilistic thresholding, enabling fast approximate sequence search as a pre-filter for tools like DREAM-Yara.

## When to use

Use when many NGS reads or transcripts must be searched against a large binned collection (e.g., 64/1,024-bin NCBI RefSeq sets) and an IBF index with (k,w)-minimizers must be built and searched.

## When NOT to use

- The reference collection has severely uneven bin sizes or a very high number of bins, where Raptor's authors note susceptibility to uneven bin sizes and a moderate slowdown when processing several ten thousand bins
- Exact full-text search or edit-distance-aware mapping results are required directly — the IBF is a pre-filter whose assignments must be verified by a downstream mapper
- An existing index was built with a Raptor version older than 2.0.0 and cannot be reused; it must be rebuilt

## Inputs

- Binning directory files (reference sequences split into bins, e.g., clustered NCBI RefSeq 64-bin and 1,024-bin sets from Zenodo doi:10.5281/zenodo.4650188 and doi:10.5281/zenodo.4651078)
- Query read sets (e.g., 2^20 simulated reads or real Illumina HiSeq 4000 reads)
- Minimizer scheme parameters (k,w) such as (19,19), (23,19), (31,19)
- IBF size budget (1–8 GiB), compression and partitioning flags

## Outputs

- IBF / MIBF index files built by `raptor build` (uncompressed or compressed)
- Per-read bin assignments from `raptor search`
- Performance tables (build time, search time, peak RAM) and FP/FN counts per configuration, e.g., table1.csv or fp_fn_table.tsv

## How to apply

Install Raptor via bioconda (`conda install -c bioconda -c conda-forge raptor`) or compile from source for host-system optimization. Run `raptor build` over a binning directory to construct either a plain (19,19)-IBF or a minimizer-based (MIBF) index with (k,w) schemes such as (19,19), (23,19), or (31,19), choosing IBF sizes among 1, 2, 4, and 8 GiB, optionally compressed or partitioned (x-partitioned) to trade run time for main memory. Run `raptor search` to query 2^20 simulated (or real) reads against each index, using Raptor's adapted k-mer lemma probabilistic thresholding (threshold ≈41% for (23,19)-minimizers) to decide bin assignments. Record construction time, search time, peak RAM, and false-positive bin assignments per configuration; evaluate against ground-truth bins of every read. For sensitivity checks, count false negatives (reads missing from their true bin) and unmapped reads when the IBF feeds a downstream mapper such as the DREAM-Yara re-implementation. The rationale is that doubling the IBF size heavily reduces false positives while minimizers shrink the index, so size and (k,w) choice are the key decision points.

## Related tools

- **Raptor** (Builds the interleaved Bloom filter / minimizer IBF binning directory (`raptor build`) and queries reads against it (`raptor search`), reporting bin assignments, run time, and peak RAM.) — https://github.com/seqan/raptor
- **DREAM-Yara** (Downstream distributed read mapper re-implemented on top of Raptor's IBF and minimizers to measure mapping time, memory (e.g., 17.5 GiB with (23,19)-minimizers, 64 bins), and unmapped-read counts.) — https://github.com/seqan/dream_yara
- **TaxSBP** (Hierarchically clusters reference sequences (e.g., NCBI RefSeq by taxonomy) into bins that define the binning directory structure indexed by the IBF.) — https://github.com/pirovc/taxsbp

## Examples

```
conda install -c bioconda -c conda-forge raptor && raptor build --help
```

## Evaluation signals

- False-positive bin assignments drop sharply with IBF size: 604,533 FPs at 1 GiB vs 189 FPs at 2 GiB for (19,19)-minimizers with b=64, so FP counts must follow this pattern for simulated data
- 64-bin FN counts with probabilistic thresholding reproduce the reported 96/37/22/24 for 1/2/4/8 GiB (23,19)-MIBF indices
- Mapping memory with minimizers matches the reported 17.5 GiB for (23,19) and 15.2 GiB for (31,19) with 64 bins (10.8 and 9 GiB with 1,024 bins), with unmapped reads increasing only by 2 and 14 over the 19-mer baseline
- Output table (e.g., table1.csv) contains one row per index configuration with consistent build/search time and peak-RAM values
- Search results against simulated reads show near-zero false-positive bin assignments relative to each read's ground-truth bin

## Limitations

- Susceptible to uneven bin sizes; an adaptive IBF for uneven inputs is under development and not yet available
- Processing several ten thousand bins leads to a moderate slowdown
- Compressed bitvector construction requires both uncompressed and compressed bitvectors in main memory, and SDSL does not support decompression of the compressed bitvector, so an uncompressed copy must also be kept on disk
- P(Cn) has no tractable formulation and is estimated experimentally; a tractable formulation is left as an open problem
- Indices built with an older version of Raptor cannot be used with version 2.0.0

## Evidence

- [abstract] Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure"
- [abstract] Our approach allows compression and partitioning of the IBF to enable the effective use of secondary memory.: "Our approach allows compression and partitioning of the IBF to enable the effective use of secondary memory."
- [methods] Each command will have a respective help page that can be shown via, e.g., `raptor build --help`.: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [discussion] Indices built with an older version of raptor cannot be used with version 2.0.0.: "Indices built with an older version of raptor cannot be used with version 2.0.0."
- [readme] There may be performance benefits when compiling from source, as Raptor can be optimized for the host system.: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
