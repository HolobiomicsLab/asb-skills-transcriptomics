---
name: allelic-proportion-partial-pooling
description: Use when single-cell allele-specific RNA-Seq read counts are too sparse
  (e.g. <10 unique reads per gene) for reliable per-cell allelic proportion estimates,
  and classification of P/B/M expression states is needed.
when_to_use_negative:
- When reads are bulk (non-single-cell) allele-specific data — the pooling rationale
  depends on information across cells sharing an allelic expression state
- When the goal is bursting-kinetics parameter inference rather than state classification/allelic
  proportion estimation — use txburst instead
- When per-gene read coverage is already high and sampling variability is negligible;
  pooling gains are concentrated at low expression (<10 unique reads per gene)
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: scBASE
  role: Implements the zero-and-one-inflated beta-binomial mixture classification
    and partial pooling of allelic proportions across cells
  repo: https://github.com/kbchoi-jax/scbase
- name: Stan
  role: Probabilistic programming backend used to fit the beta-binomial mixture model
- name: EMASE
  role: EM algorithm for weighted allocation of genomic and allelic multi-reads to
    gene- and allele-specific counts
- name: txburst
  role: Downstream bursting-kinetics workflow that consumed scBASE-estimated counts
    in the article's comparative evaluation
  repo: https://github.com/sandberg-lab/txburst
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:62626e08348a29b731098b358c7785fa7c032668d23c8495391df9ccb6269b48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbchoi-jax/scbase
  tools:
  - EMASE
  - Stan
  - scBASE
  - txburst
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# allelic-proportion-partial-pooling

## Summary

scBASE-style estimation of per-cell allelic proportions (maternal:paternal read fractions) that alternates a three-component beta-binomial classification (paternal monoallelic, bi-allelic, maternal monoallelic) with partial pooling — a weighted average of class-specific proportions across cells in the same state — fitted via Stan.

## When to use

Use when single-cell allele-specific RNA-Seq read counts are too sparse (e.g. <10 unique reads per gene) for reliable per-cell allelic proportion estimates, and classification of P/B/M expression states is needed.

## When NOT to use

- When reads are bulk (non-single-cell) allele-specific data — the pooling rationale depends on information across cells sharing an allelic expression state
- When the goal is bursting-kinetics parameter inference rather than state classification/allelic proportion estimation — use txburst instead
- When per-gene read coverage is already high and sampling variability is negligible; pooling gains are concentrated at low expression (<10 unique reads per gene)

## Inputs

- Gene- and allele-specific read counts per cell (from unique-read counting or EMASE weighted allocation of multi-reads)
- Single-cell allele-specific read count matrix for the cohort of cells (e.g. 122 blastocyst-stage cells)
- Full-data allelic proportion estimates (as reference when validating by down-sampling)

## Outputs

- Per-cell, per-gene posterior classification into paternal monoallelic (P), bi-allelic (B), or maternal monoallelic (M) states
- Pooled per-cell allelic proportion estimates for each counting scheme and pooling option (four scBASE methods)
- Per-gene MSE comparison table (e.g. mse_comparison.csv) of down-sampled vs full-data estimates

## How to apply

First resolve allele-specific counts by either the unique-reads method (discarding all genomic and allelic multi-reads) or EM-based weighted allocation in EMASE, which weights each alignment by its estimated probability. In scBASE (Stan-backed), model the maternal read count x_gk of each gene in each cell as drawn from one of three beta-binomial mixture components representing P, B, and M states, initialized with mixture weights and component Beta densities. Assign each gene-cell observation to its most probable state from posterior component probabilities, then re-estimate allelic proportions by partial pooling — the weighted average of class-specific proportions across cells assigned to the same state — and update the mixture weights and Beta densities. Alternate classification and pooling updates until mixture parameters and state assignments converge; because the two steps inform one another, and pooling can apply to either counting scheme, this yields four scBASE methods. Validate by down-sampling (e.g. 1% of reads in 122 early/mid/late blastocyst cells) and computing per-gene MSE against full-data estimates, which improves for the majority of genes with the greatest gains at low coverage (~<10 unique reads).

## Related tools

- **scBASE** (Implements the zero-and-one-inflated beta-binomial mixture classification and partial pooling of allelic proportions across cells) — https://github.com/kbchoi-jax/scbase
- **Stan** (Probabilistic programming backend used to fit the beta-binomial mixture model)
- **EMASE** (EM algorithm for weighted allocation of genomic and allelic multi-reads to gene- and allele-specific counts)
- **txburst** (Downstream bursting-kinetics workflow that consumed scBASE-estimated counts in the article's comparative evaluation) — https://github.com/sandberg-lab/txburst

## Examples

```
git clone git://github.com/kbchoi-jax/scbase
```

## Evaluation signals

- On 1%-down-sampled blastocyst data, the number of genes improved should approach the reported ranges: 4,392 of 5,759 genes (unique reads, avg MSE diff ~0.018) and 5,078 of 6,751 genes (weighted allocation, avg MSE diff ~0.016)
- MSE gains should be largest for genes with low expression (<10 unique reads per gene), as in the article's Fig 3
- Per-cell P/B/M state assignments and mixture weights should converge over the alternating classification/pooling iterations
- Excluding multi-reads (unique-reads scheme) should produce higher variability in estimates and more sampling zeros than EMASE weighted allocation
- Written output tables should contain all four method variants (two counting schemes x with/without partial pooling)

## Limitations

- Discarding multi-mapping reads leads to higher variability in estimates, increased frequency of sampling zeros, and can lead to spurious findings of dynamic and monoallelic expression
- MCMC settings (iterations, burn-in, prior specifics) are deferred to the Supplementary Methods and are not stated in the main text
- Down-sampling seed/randomization procedure for sampling 1% of reads is not specified, complicating exact reproduction
- Partial pooling has the biggest impact when read coverage is low and the number of cells is large; gains diminish otherwise

## Evidence

- [intro] MSE of down-sampled vs full-data estimates: "Lastly, we can apply partial pooling to improve the individual cell-level estimation by combining information across cells that are in the same allelic expression state."
- [discussion] beta-binomial mixture and weighted-average pooling: [evidence span withheld — anchor status: not_found]
- [discussion] weighting each alignment by its estimated probability: "multi-reads are resolved by computing a weighted allocation based on the estimated probability of each alignment"
- [discussion] biggest impact condition: "Partial pooling has the biggest impact when read coverage is low and the number of cells is large"
- [abstract] reduce sampling variability: "combining information across cells using a hierarchical mixture model reduces sampling variability without sacrificing cell-to-cell heterogeneity"
- [readme] scBASE software framing: "we tackle data sparsity and variability by harnessing information derived from cell's population context."
