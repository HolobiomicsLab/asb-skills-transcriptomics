---
name: kmer-uniqueness-analysis-across-binnings
description: Use when evaluating or reproducing Raptor's minimizer-based representative
  k-mer statistics (Table 8), e.g. to check how quickly r(k) collapses toward 1.00
  as k grows for a given number of bins.
when_to_use_negative:
- The goal is running actual sequence queries or read mapping with Raptor/DREAM-Yara
  rather than characterizing index k-mer statistics
- Bins have highly uneven sizes — the paper notes Raptor's main limiting factor is
  'its susceptibility to uneven bin sizes as well as a high number of bins', which
  distorts minimizer behavior
- Comparing against Raptor indices built with an older version, since indices built
  before version 2.0.0 are incompatible
tools:
- name: Raptor
  role: Provides the winnowing-minimizer computation used to derive representative
    k-mer sets and effective text ratios per binning
  repo: https://github.com/seqan/raptor
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:fdb108b5b01761714417a6c25a52a7b2d696ff134cbc7cfa6ec13952079f2e66
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
yes_rate: 0.42857142857142855
n_claims: 7
---

# kmer-uniqueness-analysis-across-binnings

## Summary

Computes Raptor's effective text ratios r(k), the size of the winnowing-minimizer representative k-mer set relative to the original text length, for a dataset partitioned into 64 and 1,024 bins across k-mer sizes k=12..20. This quantifies how many distinct k-mers actually index a binning, explaining Raptor's space/accuracy trade-off.

## When to use

Use when evaluating or reproducing Raptor's minimizer-based representative k-mer statistics (Table 8), e.g. to check how quickly r(k) collapses toward 1.00 as k grows for a given number of bins.

## When NOT to use

- The goal is running actual sequence queries or read mapping with Raptor/DREAM-Yara rather than characterizing index k-mer statistics
- Bins have highly uneven sizes — the paper notes Raptor's main limiting factor is 'its susceptibility to uneven bin sizes as well as a high number of bins', which distorts minimizer behavior
- Comparing against Raptor indices built with an older version, since indices built before version 2.0.0 are incompatible

## Inputs

- Artificial dataset partitioned into 64 bins (from github.com/seqan/raptor)
- Artificial dataset partitioned into 1,024 bins (from github.com/seqan/raptor)
- Raptor CLI with minimizer/representative k-mer computation commands

## Outputs

- effective_text_ratio.tsv: table of r(k) for k=12..20 at 64 and 1,024 bins
- Distinct representative k-mer counts per partitioning (effective text lengths n(k))

## How to apply

Download the artificial dataset partitions (64-bin and 1,024-bin) from the seqan/raptor repository. Install Raptor via bioconda ('conda install -c bioconda -c conda-forge raptor') or compile from source for host-specific optimization, confirming availability with 'raptor --help'. For each k in 12..20, run Raptor's minimizer computation on each bin to obtain the winnowing-minimizer representative k-mer set, then count distinct representative k-mers per partitioning to derive the effective text length n(k). Normalize n(k) by the original text length to obtain r(k), and tabulate both binnings side by side. Validate against Table 8 expectations: r(12) = 62.84 for 64 bins and 225.98 for 1,024 bins, dropping to about 1.01 at k=19 and 1.00 at k=20 for both binnings; save the table as effective_text_ratio.tsv.

## Related tools

- **Raptor** (Provides the winnowing-minimizer computation used to derive representative k-mer sets and effective text ratios per binning) — https://github.com/seqan/raptor

## Examples

```
conda install -c bioconda -c conda-forge raptor && raptor --help
```

## Evaluation signals

- r(19) ≈ 1.01 and r(20) = 1.00 for both the 64-bin and 1,024-bin partitionings
- r(12) at 64 bins equals 62.84 and at 1,024 bins equals 225.98, matching Table 8
- r(k) decreases monotonically in k within each binning
- Output file effective_text_ratio.tsv contains exactly 9 rows (k=12..20) × 2 binning columns
- Distinct representative k-mer count per binning is computed from the union over all bins, not summed per bin (avoid double-counting shared k-mers)

## Limitations

- Raptor's method is susceptible to uneven bin sizes and a high number of bins, so ratios for other binnings may deviate from the reported values
- No tractable formulation of P(Cn) exists; the paper 'left this as an open problem' and estimates it experimentally by sampling, so ratios are empirical rather than analytic
- Raptor indices built with an older version cannot be used with version 2.0.0, so recomputation must pin a compatible version
- Raptor is Linux-targeted via bioconda; installation guidance is explicitly for Linux

## Evidence

- [abstract] Raptor uses winnowing minimizers to define a set of representative k-mers: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs)"
- [discussion] The main limiting factor of our method is its susceptibility to uneven bin sizes: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
- [readme] conda install -c bioconda -c conda-forge raptor: "conda install -c bioconda -c conda-forge raptor"
- [readme] The subdirectory util contains applications and scripts related to papers: "The subdirectory `util` contains applications and scripts related to papers."
- [discussion] Indices built with an older version of raptor cannot be used with version 2.0.0: "Indices built with an older version of raptor cannot be used with version 2.0.0."
