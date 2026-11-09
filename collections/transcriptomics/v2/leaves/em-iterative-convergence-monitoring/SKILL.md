---
name: em-iterative-convergence-monitoring
description: Use when fitting a Bayesian beta-binomial mixture to maternal read counts
  x_gk from single-cell RNA-Seq (P/B/M states) and you need the state assignments
  and pooled allelic proportions to reach a mutually consistent solution before writing
  per-cell per-gene calls.
when_to_use_negative:
- Bulk RNA-Seq or other non-single-cell allelic data — the partial-pooling rationale
  depends on many cells sharing each allelic state and loses its advantage when cell
  number is small or coverage is high
- When a preliminary clustering of allelic states already exists and per-cell estimates
  are adequate without combining information across cells
- When the goal is burst-kinetic parameter inference (e.g., k_on, k_off, k_syn) rather
  than allelic state classification — use txburst on allelic UMI counts instead
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: scBASE
  role: Implements the iterative classification/partial-pooling loop over the three-component
    beta-binomial mixture; writes P/B/M calls and pooled allelic proportions to CSV
  repo: https://github.com/churchill-lab/scBASE/
- name: Stan
  role: Probabilistic programming backend used by scBASE to fit the Bayesian mixture
    model
- name: EMASE
  role: EM-based weighted allocation of genomic and allelic multi-reads, one of the
    two read-counting results that partial pooling can be applied to
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:a993a33abe24d11f13f80b6615c92080c6b0fd98acf5f5eca90321cff4ae2ada
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/churchill-lab/scBASE
  tools:
  - EMASE
  - Stan
  - scBASE
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# EM-iterative convergence monitoring

## Summary

Run the scBASE classification and partial-pooling steps alternately, each informing the other, until the beta-binomial mixture parameters and per-gene-cell P/B/M state assignments converge. This alternating EM-style loop is what lets the soft zero-and-one-inflated model borrow population information to stabilize sparse single-cell allelic proportions.

## When to use

Use when fitting a Bayesian beta-binomial mixture to maternal read counts x_gk from single-cell RNA-Seq (P/B/M states) and you need the state assignments and pooled allelic proportions to reach a mutually consistent solution before writing per-cell per-gene calls.

## When NOT to use

- Bulk RNA-Seq or other non-single-cell allelic data — the partial-pooling rationale depends on many cells sharing each allelic state and loses its advantage when cell number is small or coverage is high
- When a preliminary clustering of allelic states already exists and per-cell estimates are adequate without combining information across cells
- When the goal is burst-kinetic parameter inference (e.g., k_on, k_off, k_syn) rather than allelic state classification — use txburst on allelic UMI counts instead

## Inputs

- Maternal read counts x_gk per gene per cell (from unique-read counting or EMASE weighted allocation of multi-reads)
- Initial mixture weights and component Beta density parameters for the three beta-binomial mixture components (P, B, M)
- Optionally, allele-specific UMI counts (e.g., Larsson et al. 224 fibroblast cells)

## Outputs

- Per-cell per-gene posterior allelic expression state classifications (P/B/M) written to CSV tables
- Pooled class-specific allelic proportion estimates per gene (weighted averages across cells in the same state)
- Converged beta-binomial mixture weights and component Beta density parameters
- Gene-level ASE profile classes (P, B, M, PB, BM, MP, PBM) via mixture-weight thresholds (π_g⋅s > 0.7)

## How to apply

Initialize the mixture weights and the three component Beta densities, then probabilistically classify each gene-cell observation into paternal monoallelic (P), bi-allelic (B), or maternal monoallelic (M) from posterior component probabilities. Re-estimate class-specific allelic proportions by partial pooling — a weighted average across cells assigned to the same state — and update the mixture weights and Beta densities with those pooled values. Alternate the classification and pooling updates, each informing the other, until the mixture parameters and state assignments converge. Partial pooling can be applied to either of the read-counting results (unique-read counts or EMASE weighted-allocation counts), yielding four method variants; pooling has the biggest impact when read coverage is low and the number of cells is large.

## Related tools

- **scBASE** (Implements the iterative classification/partial-pooling loop over the three-component beta-binomial mixture; writes P/B/M calls and pooled allelic proportions to CSV) — https://github.com/churchill-lab/scBASE/
- **Stan** (Probabilistic programming backend used by scBASE to fit the Bayesian mixture model)
- **EMASE** (EM-based weighted allocation of genomic and allelic multi-reads, one of the two read-counting results that partial pooling can be applied to)

## Evaluation signals

- Mixture weights and component Beta densities stop changing between successive classification/pooling iterations (convergence criterion of the alternated EM loop)
- Posterior P/B/M state assignments are stable across the final iterations — no gene-cell observations flip state after convergence
- Pooled allelic proportions are weighted averages across cells assigned to the same state, and the pooled estimates have lower mean squared error than unpooled estimates on down-sampled data (e.g., 1% of reads in 122 blastocyst cells)
- Mixture-weight thresholds yield gene-level ASE classes only when a majority of cells (π_g⋅s > 0.7) occupy a particular state
- Allelic proportions of pooled runs show reduced U/W-shape sampling variability compared to unique-read-only estimates, without loss of cell-to-cell heterogeneity

## Limitations

- MCMC settings (iterations, burn-in, prior specifics) are deferred to Supplementary Methods, so convergence behavior must be checked per run
- The ASE class boundaries (π_g⋅s > 0.7) are arbitrary — the classification is intended as a simple descriptive summary, not a reproducible hard call
- Partial pooling benefits diminish with high read coverage or few cells; with very sparse cells and many sampling zeros, the mixture must handle U-shaped or W-shaped allelic proportion distributions from random monoallelic bursts

## Evidence

- [intro] Lastly, we can apply partial pooling to improve the individual cell-level estimation by combining information across cells that are in the same allelic expression state.: "Lastly, we can apply partial pooling to improve the individual cell-level estimation by combining information across cells that are in the same allelic expression state."
- [discussion] computing a probabilistic classification based on a mixture model in which the maternal read counts xgk are drawn from one of three beta-binomial distributions: [evidence span withheld — anchor status: not_found]
- [discussion] estimate the allelic proportions as a weighted average: [evidence span withheld — anchor status: not_found]
- [readme] we tackle data sparsity and variability by harnessing information derived from cell's population context.: "we tackle data sparsity and variability by harnessing information derived from cell's population context."
- [discussion] The class boundaries are arbitrary but the aim of this classification is to provide a simple descriptive summary: "The class boundaries are arbitrary but the aim of this classification is to provide a simple descriptive summary"
