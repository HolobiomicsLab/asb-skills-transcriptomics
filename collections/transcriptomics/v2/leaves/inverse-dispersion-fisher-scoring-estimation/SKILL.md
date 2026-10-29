---
name: inverse-dispersion-fisher-scoring-estimation
description: Use when training scDBM (BoltzmannMachines.jl) on scRNA-seq count matrices
  where the negative-binomial likelihood's inverse dispersion must be re-estimated
  during training rather than fixed, e.g. to support Gibbs sampling of synthetic cells.
when_to_use_negative:
- Input is not raw count data from a negative-binomial-generating process (e.g. already-normalized
  or batch-corrected expression matrices), where dispersion estimation assumptions
  break down
- Data contain strong batch effects, since scDBMs currently cannot learn batch effects
  and all models were examined without batch correction capabilities
- A fixed dispersion suffices for the analysis and re-estimation every five epochs
  adds unnecessary complexity
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: scDBM
  role: Deep Boltzmann machine model whose negative-binomial inverse dispersion parameter
    is estimated by regularized Fisher-scoring every five epochs during training
  repo: https://github.com/MTreppner/scDBM.jl
- name: BoltzmannMachines.jl
  role: Julia package providing the underlying DBM training machinery on which the
    Fisher-scoring dispersion update is implemented
  repo: https://github.com/MTreppner/scDBM.jl
- name: Julia
  role: Runtime (version 1.5.0) in which scDBM.jl executes the dispersion refinement
    and Gibbs sampling
  repo: https://github.com/MTreppner/scDBM.jl
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:d3eed0c22855a5918916a6e6565cff69146b4ee724ea0bba7f21ca0746adc43c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MTreppner/scDBM.jl
  tools:
  - BoltzmannMachines.jl
  - Julia
  - scDBM
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# inverse-dispersion-fisher-scoring-estimation

## Summary

Within scDBM training, the inverse dispersion parameter of the negative-binomial observation model is estimated via a regularized Fisher-scoring update, refined every five epochs to keep the dispersion estimate aligned with the evolving hidden representation. This stabilizes synthetic count generation for scRNA-seq data with gene-specific overdispersion.

## When to use

Use when training scDBM (BoltzmannMachines.jl) on scRNA-seq count matrices where the negative-binomial likelihood's inverse dispersion must be re-estimated during training rather than fixed, e.g. to support Gibbs sampling of synthetic cells.

## When NOT to use

- Input is not raw count data from a negative-binomial-generating process (e.g. already-normalized or batch-corrected expression matrices), where dispersion estimation assumptions break down
- Data contain strong batch effects, since scDBMs currently cannot learn batch effects and all models were examined without batch correction capabilities
- A fixed dispersion suffices for the analysis and re-estimation every five epochs adds unnecessary complexity

## Inputs

- Preprocessed scRNA-seq count matrix (e.g. PBMC4k: 4182 cells × 1000 highly variable genes; Segerstolpe: 2090 cells × 2000 highly variable genes)
- Pilot sub-sample count matrices of 384–2304 cells
- scDBM hyperparameters: two hidden layers, latent dimensionality 2–4, small learning rate, dispersion update interval of 5 epochs

## Outputs

- Fitted regularized inverse dispersion parameter for the scDBM negative-binomial observation model
- Trained scDBM capable of generating synthetic count matrices via Gibbs sampling

## How to apply

Configure the scDBM in Julia (scDBM.jl, built on BoltzmannMachines.jl 1.5.0) with two hidden layers and a latent dimensionality between two and four, plus a small learning rate. During training on the preprocessed count matrix (e.g. PBMC4k with 4182 cells × 1000 HVGs or Segerstolpe with 2090 cells × 2000 HVGs), invoke the regularized Fisher-scoring estimate of the inverse dispersion parameter every five epochs instead of holding it fixed. Regularization prevents unstable dispersion estimates on sparse pilot sub-samples (384–2304 cells). The resulting fitted dispersion is carried into Gibbs sampling, which generates synthetic cells up to the original study size.

## Related tools

- **scDBM** (Deep Boltzmann machine model whose negative-binomial inverse dispersion parameter is estimated by regularized Fisher-scoring every five epochs during training) — https://github.com/MTreppner/scDBM.jl
- **BoltzmannMachines.jl** (Julia package providing the underlying DBM training machinery on which the Fisher-scoring dispersion update is implemented) — https://github.com/MTreppner/scDBM.jl
- **Julia** (Runtime (version 1.5.0) in which scDBM.jl executes the dispersion refinement and Gibbs sampling) — https://github.com/MTreppner/scDBM.jl

## Examples

```
julia> using BoltzmannMachines; dbm = fitdbm(counts, epochs=100, learningrate=small_lr, hiddenlayers=[2,4], cbfunction=every_5_epochs_fisher_scoring_dispersion_update)
```

## Evaluation signals

- Dispersion parameter is re-estimated at the prescribed interval (every five epochs) throughout scDBM training rather than only once at initialization
- Trained scDBM generates a synthetic count matrix with the original study's cell count (4182 for PBMC4k, 2090 for Segerstolpe) via Gibbs sampling, as part of the 1080 generated artifacts (2 datasets × 3 models × 6 pilot sizes × 30 repetitions)
- Per-gene summaries of synthetic data (medians, interquartile ranges, coefficients of variation, proportion of zero counts) show low mean absolute deviation from the original data for most genes
- Training converges with a small learning rate and latent dimensionality between two and four without divergent dispersion updates on 384-cell pilot sub-samples

## Limitations

- scDBMs currently cannot learn batch effects, and all models were examined without using potentially available batch correction capabilities
- All approaches overrepresent highly abundant cell types and miss less abundant cell types, and problems occurred with bimodal gene distributions
- For 10x Genomics datasets, which have higher sparsity, it is more challenging to make an inference from small to larger datasets, which may stress the dispersion estimates

## Evidence

- [results] refining the regularized Fisher-scoring estimate of the inverse dispersion parameter every five epochs: [evidence span withheld — anchor status: not_found]
- [results] we sampled from the scDBM using Gibbs sampling: "we sampled from the scDBM using Gibbs sampling and from scVI using the prior and posterior distribution"
- [readme] Second, we propose single-cell deep Boltzmann machines (scDBM).: "Second, we propose single-cell deep Boltzmann machines (scDBM)."
- [abstract] for 10x datasets, which have higher sparsity, it is more challenging to make an inference from small to larger datasets: "Across all analyses, in comparing 10x Genomics and Smart-seq2 technologies, we could show that for 10x datasets, which have higher sparsity, it is more challenging to make an inference from small to"
