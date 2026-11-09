---
name: reference-table-value-reproduction
description: Use when a paper's reported numeric table values (e.g., effective text
  ratios r(k) per binning and k-mer size) must be independently recomputed and validated
  from a downloadable dataset with the authors' tool.
when_to_use_negative:
- The goal is approximate sequence search or binning-directory querying (Raptor's
  actual use case), not recomputing publication statistics
- The reference table depends on real data (e.g., Clustered NCBI RefSeq 64/1,024-bin
  partitions on Zenodo) rather than the artificial dataset, so values will not match
  Table 8
- Uneven bin sizes or a very high number of bins are involved, since the paper notes
  the method's main limiting factor is 'its susceptibility to uneven bin sizes as
  well as a high number of bins'
tools:
- name: Raptor
  role: Provides the winnowing-minimizer representative k-mer computation used to
    derive effective text lengths n(k) and ratios r(k)
  repo: https://github.com/seqan/raptor
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:64fddc86beb9a5a39132873d99f427de88c5ae0967da971d90b990d0cb6a8e57
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
yes_rate: 0.5454545454545454
n_claims: 11
---

# reference-table-value-reproduction

## Summary

Reproduces published table values (e.g., Table 8 of the Raptor paper) by recomputing the effective text ratios r(k) of winnowing-minimizer representative k-mers on the artificial 64-bin and 1,024-bin datasets for k=12..20 using Raptor, and validating the recomputed numbers against the reported values.

## When to use

Use when a paper's reported numeric table values (e.g., effective text ratios r(k) per binning and k-mer size) must be independently recomputed and validated from a downloadable dataset with the authors' tool.

## When NOT to use

- The goal is approximate sequence search or binning-directory querying (Raptor's actual use case), not recomputing publication statistics
- The reference table depends on real data (e.g., Clustered NCBI RefSeq 64/1,024-bin partitions on Zenodo) rather than the artificial dataset, so values will not match Table 8
- Uneven bin sizes or a very high number of bins are involved, since the paper notes the method's main limiting factor is 'its susceptibility to uneven bin sizes as well as a high number of bins'

## Inputs

- Artificial dataset from github.com/seqan/raptor partitioned into 64 bins
- Artificial dataset from github.com/seqan/raptor partitioned into 1,024 bins
- k-mer size range k=12..20

## Outputs

- effective_text_ratio.tsv: table of effective text ratios r(k) for k=12..20 at 64 and 1,024 bins
- Per-bin sets of winnowing-minimizer representative k-mers

## How to apply

Install Raptor via bioconda ('conda install -c bioconda -c conda-forge raptor') and confirm availability with 'raptor --help'. Download the artificial dataset in its 64-bin and 1,024-bin partitions from the seqan/raptor repository (the paper's binning directories are hosted there and Zenodo). For each k in 12..20, run Raptor's minimizer computation on each bin to obtain the set of winnowing-minimizer representative k-mers, then count distinct representative k-mers per partitioning to derive the effective text length n(k). Normalize n(k) by the original text length to obtain the effective text ratio r(k) for each binning. Tabulate r(k) for k=12..20 for both binnings and validate against the paper: r(12) = 62.84 (64 bins) and 225.98 (1,024 bins) dropping to r(19) ≈ 1.01 and r(20) = 1.00 for both binnings; save the table as effective_text_ratio.tsv. Check the repository's util subdirectory, which 'contains applications and scripts related to papers', for helper scripts.

## Related tools

- **Raptor** (Provides the winnowing-minimizer representative k-mer computation used to derive effective text lengths n(k) and ratios r(k)) — https://github.com/seqan/raptor

## Examples

```
conda install -c bioconda -c conda-forge raptor && raptor --help
```

## Evaluation signals

- r(12) reproduces 62.84 for the 64-bin partitioning and 225.98 for the 1,024-bin partitioning
- r(19) ≈ 1.01 and r(20) = 1.00 for both the 64-bin and 1,024-bin partitions
- Ratio r(k) decreases monotonically with k across k=12..20 for both binnings
- Output table effective_text_ratio.tsv contains 18 rows (9 k values × 2 binnings) with all values ≥ 1.0
- Raptor commands execute successfully, verified via 'raptor --help' and per-command help pages such as 'raptor build --help'

## Limitations

- Indices built with an older version of Raptor cannot be used with version 2.0.0, so version consistency matters when reproducing results
- Raptor's winnowing approach is susceptible to uneven bin sizes and a high number of bins, which can distort ratio values
- Compiling from source may be needed for host-optimized performance, per the README

## Evidence

- [results] values drop from 62.84 (64 bins) and 225.98 (1,024 bins) at k=12 to 1.01 at k=19 and 1.00 at k=20 for both binnings: [evidence span withheld — anchor status: not_found]
- [abstract] Raptor uses winnowing minimizers to define representative k-mers: [evidence span withheld — anchor status: not_found]
- [readme] conda install -c bioconda -c conda-forge raptor: "bash
conda install -c bioconda -c conda-forge raptor
"
- [readme] raptor --help will show available commands: "`raptor --help` will show available commands. Each command will have a respective help page that can be shown via, e.g.,
`raptor build --help`."
- [readme] The subdirectory util contains applications and scripts related to papers: "The subdirectory `util` contains applications and scripts related to papers."
