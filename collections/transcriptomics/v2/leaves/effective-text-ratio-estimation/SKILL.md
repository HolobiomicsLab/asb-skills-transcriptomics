---
name: effective-text-ratio-estimation
description: Use when computing or validating effective text ratios r(k) for a binned
  nucleotide sequence collection (e.g., Raptor's artificial dataset with 64 or 1,024
  bins) across k-mer sizes k=12..20 using Raptor's minimizer approach.
when_to_use_negative:
- The dataset consists of real genomes with highly uneven bin sizes — Raptor's method
  is susceptible to uneven bin sizes, so r(k) from the balanced artificial dataset
  will not transfer.
- The goal is query/search performance benchmarking (e.g., against Mantis or COBS)
  rather than characterizing the minimizer-compressed text length.
- Only a single k-mer size is of interest and no cross-k comparison (k=12..20) is
  needed to observe the ratio dropping toward 1.
tools:
- name: Raptor
  role: Runs the winnowing-minimizer computation on each bin to obtain the representative
    k-mer sets from which effective text lengths n(k) are counted
  repo: https://github.com/seqan/raptor
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:4315f0d80285c4e8d8b206859946257a6451486e7eee20ffab6659d9c7cd23a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - Raptor
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# effective-text-ratio-estimation

## Summary

Reproduces the effective text ratio r(k) computation from Raptor's Table 8: for each k-mer size k=12..20, the ratio of the winnowing-minimizer representative k-mer text length to the original text length is derived for the artificial dataset under 64-bin and 1,024-bin partitionings.

## When to use

Use when computing or validating effective text ratios r(k) for a binned nucleotide sequence collection (e.g., Raptor's artificial dataset with 64 or 1,024 bins) across k-mer sizes k=12..20 using Raptor's minimizer approach.

## When NOT to use

- The dataset consists of real genomes with highly uneven bin sizes — Raptor's method is susceptible to uneven bin sizes, so r(k) from the balanced artificial dataset will not transfer.
- The goal is query/search performance benchmarking (e.g., against Mantis or COBS) rather than characterizing the minimizer-compressed text length.
- Only a single k-mer size is of interest and no cross-k comparison (k=12..20) is needed to observe the ratio dropping toward 1.

## Inputs

- Artificial dataset with 64-bin partitioning (from github.com/seqan/raptor)
- Artificial dataset with 1,024-bin partitioning (from github.com/seqan/raptor)
- K-mer sizes k = 12..20

## Outputs

- effective_text_ratio.tsv: table of r(k) for k=12..20 at 64 and 1,024 bins
- Per-bin sets of distinct winnowing-minimizer representative k-mers
- Effective text length n(k) per partitioning

## How to apply

Download the artificial dataset (64-bin and 1,024-bin partitions) from the seqan/raptor repository and install Raptor via bioconda ('conda install -c bioconda -c conda-forge raptor'), verifying availability with 'raptor --help'. For each k in 12..20, run Raptor's minimizer computation on every bin to obtain the set of winnowing-minimizer representative k-mers; Raptor uses winnowing minimizers to define this representative set. Count the distinct representative k-mers per partitioning to derive the effective text length n(k), then normalize n(k) by the original text length to obtain r(k) for each binning. Tabulate r(k) for k=12..20 at both binnings and validate against Table 8: r(12)=62.84 (64 bins) and 225.98 (1,024 bins), dropping to r(19)≈1.01 and r(20)=1.00 for both binnings; save as effective_text_ratio.tsv.

## Related tools

- **Raptor** (Runs the winnowing-minimizer computation on each bin to obtain the representative k-mer sets from which effective text lengths n(k) are counted) — https://github.com/seqan/raptor

## Examples

```
conda install -c bioconda -c conda-forge raptor && raptor --help
```

## Evaluation signals

- r(12) matches Table 8: 62.84 for 64 bins and 225.98 for 1,024 bins
- r(19) ≈ 1.01 for both the 64-bin and 1,024-bin partitionings
- r(20) = 1.00 for both binnings, confirming the ratio saturates at 1 for large k
- r(k) decreases monotonically with k from k=12 to k=20 within each binning
- Output file effective_text_ratio.tsv contains 9 rows (k=12..20) with two ratio columns

## Limitations

- Raptor's method is susceptible to uneven bin sizes as well as a high number of bins, which affects how representative the artificial-dataset ratios are for other collections
- P(Cn) has no tractable formulation; it was left as an open problem and estimated experimentally by sampling, so the minimizer model relies on empirical estimation
- Indices built with an older version of Raptor cannot be used with version 2.0.0, so version consistency matters when reproducing results

## Evidence

- [abstract] winnowing minimizers define representative k-mers: [evidence span withheld — anchor status: not_found]
- [readme] Raptor install via bioconda: "conda install -c bioconda -c conda-forge raptor"
- [readme] command help availability: "`raptor --help` will show available commands. Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [discussion] limitation on bin sizes: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
