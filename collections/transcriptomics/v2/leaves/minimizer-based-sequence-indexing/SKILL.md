---
name: minimizer-based-sequence-indexing
description: Use when many short reads/transcripts must be approximately searched
  against very large collections of nucleotide sequences (e.g., hundreds of bins/genomes)
  where FM-indexes are infeasible and word-based indexes cannot handle edit-distance
  searches.
when_to_use_negative:
- Bin sizes are highly uneven or the number of bins is extremely large — the paper
  states 'The main limiting factor of our method is its susceptibility to uneven bin
  sizes as well as a high number of bins.'
- Exact, full-text search results are required — Raptor is a pre-filter with probabilistic
  thresholding, so assignments can contain false positives and false negatives.
- Only a small reference collection is searched, where a conventional aligner or FM-index
  would be simpler and exhaustive.
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Raptor
  role: Builds the minimizer interleaved Bloom filter index ('raptor build') and performs
    thresholded search of reads against bins ('raptor search')
  repo: https://github.com/seqan/raptor
- name: TaxSBP
  role: Hierarchically clusters reference sequences into bins before index construction
  repo: https://github.com/pirovc/taxsbp
- name: DREAM-Yara
  role: Read mapper re-implemented on top of Raptor to demonstrate downstream use
    of the pre-filter
  repo: https://github.com/seqan/dream_yara
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:e1695648b3bb3e5fa5a3992409973f209c55c5248f27eab959917c37898b3aa6
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# minimizer-based-sequence-indexing

## Summary

Build and query a minimizer interleaved Bloom filter (MIBF) pre-filter, as implemented in Raptor, which uses winnowing minimizers (e.g., a (23,19)-scheme) to represent large nucleotide sequence collections with probabilistic thresholding, enabling fast approximate search of NGS reads in binning directories.

## When to use

Use when many short reads/transcripts must be approximately searched against very large collections of nucleotide sequences (e.g., hundreds of bins/genomes) where FM-indexes are infeasible and word-based indexes cannot handle edit-distance searches.

## When NOT to use

- Bin sizes are highly uneven or the number of bins is extremely large — the paper states 'The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins.'
- Exact, full-text search results are required — Raptor is a pre-filter with probabilistic thresholding, so assignments can contain false positives and false negatives.
- Only a small reference collection is searched, where a conventional aligner or FM-index would be simpler and exhaustive.

## Inputs

- Nucleotide reference sequences partitioned into bins (e.g., FASTA, clustered NCBI RefSeq in 64 or 1,024 bins)
- Query nucleotide sequences (e.g., simulated/real NGS reads)
- Binning layout (bin count, optionally TaxSBP-derived clustering)

## Outputs

- Raptor MIBF index (x-partitioned interleaved Bloom filter built via 'raptor build')
- Per-read bin assignments from 'raptor search'
- FP/FN counts versus ground-truth bin of every read (e.g., fp_fn_table.tsv)

## How to apply

Install Raptor (e.g., `conda install -c bioconda -c conda-forge raptor`) or compile from source for host-optimized performance. Partition reference sequences into bins (e.g., 64 or 1,024 bins; the clustered NCBI RefSeq datasets at 10.5281/zenodo.4650188 and 10.5281/zenodo.4651078, or the artificial dataset distributed with the repository), then run `raptor build` to construct an x-partitioned interleaved Bloom filter over winnowing minimizers, choosing k-mer/minimizer parameters such as (23,19) and an index size (1–8 GiB) that trades space for sensitivity. Query reads with `raptor search`, using the probabilistic (adapted k-mer lemma) minimizer threshold — for the (23,19)-MIBF configurations in the paper this results in a threshold of ≈41% — and compare reported bin assignments against ground truth to assess false positives (reads reported in bins that do not contain them) and false negatives (reads missing from their true bin). The validation anchor for correctness is reproducing the reported FN counts of 96/37/22/24 for the 64-bin 1/2/4/8 GiB indices.

## Related tools

- **Raptor** (Builds the minimizer interleaved Bloom filter index ('raptor build') and performs thresholded search of reads against bins ('raptor search')) — https://github.com/seqan/raptor
- **TaxSBP** (Hierarchically clusters reference sequences into bins before index construction) — https://github.com/pirovc/taxsbp
- **DREAM-Yara** (Read mapper re-implemented on top of Raptor to demonstrate downstream use of the pre-filter) — https://github.com/seqan/dream_yara

## Examples

```
conda install -c bioconda -c conda-forge raptor && raptor build --help
```

## Evaluation signals

- 64-bin false-negative counts reproduce the reported values 96/37/22/24 for the 1/2/4/8 GiB (23,19)-MIBF indices
- Resulting probabilistic threshold for the (23,19)-minimizer scheme matches the reported ≈41%
- FP/FN table (fp_fn_table.tsv) assembled across 64 and 1,024 bins at all four index sizes matches Table 3 structure
- Index was built with the same Raptor version as used for search (indices built with an older version cannot be used with version 2.0.0)

## Limitations

- Susceptible to uneven bin sizes and a high number of bins; several ten thousand bins cause a moderate slowdown
- P(Cn) for probabilistic thresholding has no tractable formulation and was estimated experimentally by sampling
- Indices built with an older version of Raptor cannot be used with version 2.0.0
- Adaptive IBF for uneven bin sizes and hierarchical multi-level IBF structure were not yet available at publication time

## Evidence

- [abstract] Raptor uses winnowing minimizers, an extension of interleaved Bloom filters (IBFs), and probabilistic thresholding for minimizers: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs)"
- [methods] Raptor commands provide help pages; build/search are invoked per command: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [readme] Compiling from source may improve performance for the host system: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
- [discussion] Main limitations: uneven bin sizes and high bin count: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
