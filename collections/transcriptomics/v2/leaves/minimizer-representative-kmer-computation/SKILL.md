---
name: minimizer-representative-kmer-computation
description: 'Use when reproducing or evaluating Raptor-style minimizer pre-filter
  sizing: you have a binned partitioning of nucleotide sequences (e.g., 64 or 1,024
  bins) and need effective text lengths/ratios r(k) for k=12..20 as reported in Table
  8.'
when_to_use_negative:
- Input is already a built Raptor index or binning directory — the skill operates
  on raw binned sequence data before index construction
- The goal is approximate read search performance benchmarking against Mantis or COBS
  rather than index sizing via effective text ratios
- Sequence collections with highly uneven bin sizes, where Raptor's main limiting
  factor is susceptibility to uneven bin sizes and the effective ratio is not the
  relevant quantity
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: Raptor
  role: Computes winnowing-minimizer representative k-mer sets per bin that define
    the index content; installed via bioconda for the minimizer computation on the
    64- and 1,024-bin artificial dataset
  repo: https://github.com/seqan/raptor
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:4b40320eb6a3e487cffd4e2033d7a9704e392d8458ea413f573881ce2398b965
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - Raptor
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# minimizer-representative-kmer-computation

## Summary

Computes the set of winnowing-minimizer representative k-mers for binned nucleotide sequence collections (e.g., the Raptor artificial dataset with 64 and 1,024 bins) and derives effective text ratios r(k) for k-mer sizes k=12..20, the quantity underlying Raptor's index size analysis.

## When to use

Use when reproducing or evaluating Raptor-style minimizer pre-filter sizing: you have a binned partitioning of nucleotide sequences (e.g., 64 or 1,024 bins) and need effective text lengths/ratios r(k) for k=12..20 as reported in Table 8.

## When NOT to use

- Input is already a built Raptor index or binning directory — the skill operates on raw binned sequence data before index construction
- The goal is approximate read search performance benchmarking against Mantis or COBS rather than index sizing via effective text ratios
- Sequence collections with highly uneven bin sizes, where Raptor's main limiting factor is susceptibility to uneven bin sizes and the effective ratio is not the relevant quantity

## Inputs

- Artificial dataset partitioned into 64 bins (from github.com/seqan/raptor)
- Artificial dataset partitioned into 1,024 bins (from github.com/seqan/raptor)
- k-mer sizes k = 12..20
- Original (unpartitioned) text length for normalization

## Outputs

- Set of distinct winnowing-minimizer representative k-mers per bin per partitioning
- Effective text length n(k) per partitioning per k
- Effective text ratio table effective_text_ratio.tsv (r(k) for k=12..20, 64 and 1,024 bins)

## How to apply

Download the artificial dataset (64-bin and 1,024-bin partitions) from the Raptor repository (github.com/seqan/raptor) and install Raptor via bioconda (`conda install -c bioconda -c conda-forge raptor`), confirming availability with `raptor --help`. For each k in 12..20, run Raptor's minimizer computation on each bin to obtain the winnowing-minimizer representative k-mer set that Raptor uses to define its index content. Count the distinct representative k-mers per partitioning to derive the effective text length n(k), then normalize n(k) by the original text length to obtain the effective text ratio r(k) for each binning. Tabulate r(k) for k=12..20 at both 64 and 1,024 bins and validate against the reported values (r(12) = 62.84 for 64 bins and 225.98 for 1,024 bins; convergence to ~1.01 at k=19 and 1.00 at k=20 for both binnings), saving the table as effective_text_ratio.tsv.

## Related tools

- **Raptor** (Computes winnowing-minimizer representative k-mer sets per bin that define the index content; installed via bioconda for the minimizer computation on the 64- and 1,024-bin artificial dataset) — https://github.com/seqan/raptor

## Examples

```
conda install -c bioconda -c conda-forge raptor && raptor --help
```

## Evaluation signals

- r(12) should be approximately 62.84 for the 64-bin partitioning and 225.98 for the 1,024-bin partitioning
- r(19) should be approximately 1.01 and r(20) approximately 1.00 for BOTH the 64-bin and 1,024-bin partitionings (ratios converge as k grows)
- The ratio r(k) must decrease monotonically with increasing k across k=12..20 for each binning
- Each computed r(k) equals the distinct representative k-mer count n(k) divided by the original text length — verify the normalization denominator is the unpartitioned text length
- Output table effective_text_ratio.tsv must contain one row per (k, binning) pair, i.e., 2 × 9 = 18 ratio values for k=12..20

## Limitations

- Raptor's main limiting factor is susceptibility to uneven bin sizes as well as a high number of bins, which distorts the effective text ratio for unbalanced partitionings
- The probability P(Cn) of minimizer collisions has no tractable formulation — it was left as an open problem and estimated experimentally, so computed representative k-mer counts are approximate
- Indices built with an older version of raptor cannot be used with version 2.0.0 — ensure the same version is used when reproducing published ratios

## Evidence

- [abstract] Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs)"
- [results] Table 8 effective text ratio values for the artificial dataset: [evidence span withheld — anchor status: not_found]
- [methods] Raptor CLI help pages document the per-command interface used for the minimizer computation: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [readme] Installing Raptor from source can change performance of the minimizer computation: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
