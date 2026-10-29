---
name: numerical-threshold-table-generation
description: Use when you need minimizer-sharing thresholds for a pre-filter query
  whose threshold must depend on w, k, the error budget, and the per-read minimizer
  count m (e.g. 15-35), rather than a fixed fraction of shared k-mers.
when_to_use_negative:
- The query threshold must be a simple fixed fraction of shared k-mers and per-read
  minimizer-count adaptation is not required
- You only need to run Raptor searches with precomputed thresholds rather than regenerate
  or re-derive the threshold table itself
- 'The model must account for two errors in close vicinity: the paper developed such
  models but omitted them, so the tabulated thresholds do not cover that regime'
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Raptor
  role: Source of the winnowing-minimizer probabilistic thresholding model (Lemma
    2) being reconstructed; installable via bioconda and used to cross-check thresholds
    during search
  repo: https://github.com/seqan/raptor
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:f31873ac727710b707a75cfeb4543aa3ec63ef7467ac2b4b290f7c3a46c668bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - Raptor
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# numerical-threshold-table-generation

## Summary

Reconstructs Raptor's probabilistic thresholding model (Equations 1-5 and Lemma 2) as a standalone generator of minimizer-sharing thresholds, combining a binomial model of errors affecting (w,k)-minimizers with Monte Carlo estimation of indirectly affected minimizers P(Cn), and tabulates thresholds over the (w, k, errors, m) parameter grid.

## When to use

Use when you need minimizer-sharing thresholds for a pre-filter query whose threshold must depend on w, k, the error budget, and the per-read minimizer count m (e.g. 15-35), rather than a fixed fraction of shared k-mers.

## When NOT to use

- The query threshold must be a simple fixed fraction of shared k-mers and per-read minimizer-count adaptation is not required
- You only need to run Raptor searches with precomputed thresholds rather than regenerate or re-derive the threshold table itself
- The model must account for two errors in close vicinity: the paper developed such models but omitted them, so the tabulated thresholds do not cover that regime

## Inputs

- Parameter grid: window size w, k-mer size k, maximal number of errors e
- Query minimizer count m (range 15-35 per the workflow)
- Raptor Equations 1-5 and Lemma 2 specification (from the methods section of doi:10.1016/j.isci.2021.102782)

## Outputs

- threshold_table.csv with one row per (w, k, errors, m) combination and its numerically solved threshold tau
- Monte Carlo estimates of P(Cn) per parameter combination

## How to apply

Formalize Raptor's Equations 1-5 and Lemma 2 as a binomial model over the probability that a query minimizer is directly affected by errors, parameterized by window size w, k-mer size k, and number of errors. Because the term for indirectly affected minimizers P(Cn) has no tractable closed form (the authors explicitly left this as an open problem), estimate it by Monte Carlo sampling. For each query minimizer count m in the 15-35 range, numerically solve for the threshold tau such that a match sharing fewer minimizers is attributable to random sharing at significance level 0.99. Tabulate thresholds over the (w, k, errors, m) grid and validate against the paper's example: a 100 bp query with 20 minimizers must yield threshold 6, with all tabulated values falling in the reported 4-13 range. Write the result as threshold_table.csv.

## Related tools

- **Raptor** (Source of the winnowing-minimizer probabilistic thresholding model (Lemma 2) being reconstructed; installable via bioconda and used to cross-check thresholds during search) — https://github.com/seqan/raptor

## Examples

```
conda install -c bioconda -c conda-forge raptor  # then use raptor build --help to cross-check thresholds against the tabulated values
```

## Evaluation signals

- Validation case: a 100 bp query with 20 minimizers must yield threshold 6
- All tabulated threshold values fall within the paper's reported 4-13 range
- Monte Carlo estimate of P(Cn) is stable across independent sampling runs before being fed into the tau solve
- Each threshold is the smallest tau such that random sharing is rejected at significance level 0.99 for the given (w, k, errors, m)
- threshold_table.csv covers the full (w, k, errors, m) grid with no missing cells and a consistent schema

## Limitations

- P(Cn) has no tractable formulation; the Raptor authors 'tried to find a tractable formulation of Cn, but did not succeed and leave this as an open problem', so it must be estimated experimentally by sampling
- More involved models accounting for two errors in close vicinity were developed but omitted from the paper, so clustered-error regimes are not covered
- The number and distribution of minimizers is sequence-dependent and varies per read, so a tabulated threshold keyed on m is an approximation for any individual query

## Evidence

- [abstract] Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for"
- [discussion] We tried to find a tractable formulation of Cn, but did not succeed and leave this as an open problem.: "We tried to find a tractable formulation of Cn, but did not succeed and leave this as an open problem."
- [readme] There may be performance benefits when compiling from source, as Raptor can be optimized for the host system.: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
