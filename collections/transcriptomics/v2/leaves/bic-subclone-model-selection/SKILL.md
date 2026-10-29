---
name: bic-subclone-model-selection
description: Use when inferring a tumor phylogeny from bulk DNA WES and scRNA-seq
  SNV data and the number of subclones K is unknown and must be chosen by a model-selection
  criterion rather than fixed a priori.
when_to_use_negative:
- The number of subclones is already known or externally constrained and only the
  tree structure needs inference
- Input data are raw reads lacking the pre-processed SNV callset QC (stringent QC
  via the Zenodo preprocessing pipeline should be done first)
- Mutations have non-estimable bursting kinetics (subset to mutations with nonzero
  alternative read counts and estimable alpha/beta before selecting K)
edam_operation: http://edamontology.org/operation_3677
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3173
tools:
- name: Canopy2
  role: R package providing get_trees(), get_diagnostics(), and get_best_tree() for
    MCMC sampling over Klist and modified-BIC subclone selection
  repo: https://github.com/annweideman/canopy2
- name: R
  role: Runtime environment for the canopy2 package and its MCMC/BIC workflow
  repo: https://github.com/annweideman/canopy2
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:eb78012322ae9e337a16d94288a9c9f70a2ce732b7869039a43c7a8ffb0d60f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - Canopy
  - Canopy2
  - Cardelino
  - R
  edam_operation: http://edamontology.org/operation_3677
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# bic-subclone-model-selection

## Summary

Selects the optimal number of tumor subclones (K) in Canopy2 by running get_trees() over a Klist range and minimizing a modified Bayesian information criterion (BIC) across chains and iterations.

## When to use

Use when inferring a tumor phylogeny from bulk DNA WES and scRNA-seq SNV data and the number of subclones K is unknown and must be chosen by a model-selection criterion rather than fixed a priori.

## When NOT to use

- The number of subclones is already known or externally constrained and only the tree structure needs inference
- Input data are raw reads lacking the pre-processed SNV callset QC (stringent QC via the Zenodo preprocessing pipeline should be done first)
- Mutations have non-estimable bursting kinetics (subset to mutations with nonzero alternative read counts and estimable alpha/beta before selecting K)

## Inputs

- Rs: M x N mutation-by-single-cell alternative read count matrix from scRNA-seq
- Xs: M x N mutation-by-single-cell total read count matrix
- Rb: M x T mutation-by-bulk-sample alternative read count matrix from bulk DNA WES
- Xb: M x T bulk total read count matrix
- alpha and beta: 1 x M vectors of mutation-specific gene activation/deactivation rates (from get_burstiness_bpsc() or get_burstiness_scale())
- Klist: range of candidate subclone counts (e.g., 3:10)

## Outputs

- Selected tree with minimum modified BIC: Z (M x K clonal configuration matrix), Ps (K x N cell-to-clone assignment matrix), Pb (K x T sample-to-clone assignment matrix)
- MCMC diagnostics (lag ACF, posterior density and trace plots) per chain and K
- Graph of BIC versus number of subclones

## How to apply

Pass the post-processed Rs, Xs (single-cell alternative/total read counts), Rb, Xb (bulk WES counts), and mutation-specific bursting-kinetics estimates alpha and beta to get_trees() across Klist=3:10 with multiple chains (>=10 recommended), thin=20, pburn=0.2, and 50k-100k iterations. Set sequencing-error hyperparameters kappa=1 and tau=999 so the beta-distributed error rate has mean 0.001. Check convergence per chain and K with get_diagnostics(), requiring lag ACF below 0.3, spatially aligned posterior densities, and noise-like trace plots. Then call get_best_tree(), which for each k in Klist selects the tree with highest mean posterior density across all post-burn-in iterations and chains, and minimizes the modified BIC across the K range; examine the graph of BIC versus number of subclones to confirm the optimum (e.g., K=6 for GBM10).

## Related tools

- **Canopy2** (R package providing get_trees(), get_diagnostics(), and get_best_tree() for MCMC sampling over Klist and modified-BIC subclone selection) — https://github.com/annweideman/canopy2
- **R** (Runtime environment for the canopy2 package and its MCMC/BIC workflow) — https://github.com/annweideman/canopy2

## Examples

```
trees <- get_trees(Rs, Xs, Rb, Xb, alpha, beta, kappa=1, tau=999, Klist=3:10, niter=100000, nchains=10, thin=20, pburn=0.2); best <- get_best_tree(trees)
```

## Evaluation signals

- get_best_tree() converges on a minimum-BIC configuration across the Klist range
- Per-chain lag ACF is below the 0.3 threshold for the selected K
- Posterior densities are spatially aligned across chains and trace plots resemble random noise
- For each k in Klist, the selected tree has the highest mean posterior density across all post-burn-in iterations and all chains
- BIC-versus-K plot shows a clear minimum at the selected subclone count (e.g., K=6 for GBM10)

## Limitations

- Reliance on Metropolis-within-Gibbs sampling creates a convergence bottleneck influenced by proposal distributions; poor proposals can delay convergence or cause settling on a local rather than global optimum
- Canopy2 is the slowest of the four benchmarked methods (microbenchmarked on a 12th Gen Intel Core i9-12900HK, ncores=20)
- Not published on CRAN as of 6/24/23; the developmental GitHub version is recommended to pick up corrected bugs

## Evidence

- [readme] Klist: A range of possible numbers of subclones to use for Bayesian information criterion (BIC) calculations. For each k in Klist, the tree with highest mean posterior density across all post-burn-in iterations and all chains is selected, and then BIC is used to select the optimal configuration.: [evidence span withheld — anchor status: not_found]
- [readme] We suggest examining the graph of BIC versus number of subclones to determine the optimal number of subclones.: [evidence span withheld — anchor status: external_doc]
- [full_text] Select the optimal number of subclones per patient with get_best_tree(), which minimizes a modified Bayesian information criterion (BIC) across the K range.: [evidence span withheld — anchor status: not_found]
- [full_text] Assess MCMC convergence for each chain and subclone count using get_diagnostics(), checking lag ACF (below 0.3), posterior density alignment, and trace plots resembling random noise.: [evidence span withheld — anchor status: not_found]
- [discussion] Proposals that do not allow for sufficient mixing can lead to dependencies on the initial state, delaying convergence or causing it to settle on a local rather than global optimum.: "delaying convergence or causing it to settle on a local rather than global optimum."
