---
name: library-size-factor-modeling
description: Use when generating synthetic scRNA-seq counts with scVI (e.g. from a
  PBMC4k pilot with 4182 cells and 1000 HVGs) and you must decide whether library-size
  factors come from the per-cell posterior or from the Normal(l_mu, 1) prior variant.
when_to_use_negative:
- You need synthetic data faithful to individual pilot observations — the posterior-sampling
  variant (the standard approach) is designed for that, and the prior variant explicitly
  avoids copies of the original observations.
- Your dataset requires batch correction — the paper examined all models without using
  potentially available batch correction capabilities, and scDBMs currently cannot
  learn batch effects.
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scVI
  role: Generative VAE whose gamma-Poisson decoder consumes the sampled latent z and
    Normal(l_mu, 1) library-size factors to produce synthetic UMI counts; used in
    prior and posterior variants
  repo: https://github.com/scverse/scvi-tools
- name: Julia
  role: Runtime (Julia 1.5.0) for the scDBM-paper analysis environment in which scVI
    (version 0.6.0) is installed via Conda.jl
  repo: https://github.com/MTreppner/scDBM-paper
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:e4d21da04241abe267e8159f8cc13be50dbdc054f7a4906aaf19ee2f4d8c0509
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/scvi-tools
  tools:
  - Julia
  - scVI
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# library-size-factor-modeling

## Summary

In the scVI generative pipeline from Treppner et al. (2021), library-size factors are modeled probabilistically so that synthetic UMI counts have realistic per-cell sequencing depth. In the prior-sampling variant, library sizes are drawn from Normal(l_mu, 1) — where l_mu is the learned library-size mean — instead of from the inferred per-cell library-size posterior.

## When to use

Use when generating synthetic scRNA-seq counts with scVI (e.g. from a PBMC4k pilot with 4182 cells and 1000 HVGs) and you must decide whether library-size factors come from the per-cell posterior or from the Normal(l_mu, 1) prior variant.

## When NOT to use

- You need synthetic data faithful to individual pilot observations — the posterior-sampling variant (the standard approach) is designed for that, and the prior variant explicitly avoids copies of the original observations.
- Your dataset requires batch correction — the paper examined all models without using potentially available batch correction capabilities, and scDBMs currently cannot learn batch effects.

## Inputs

- Preprocessed PBMC4k count matrix (4182 cells, 1000 highly variable genes) from the trained scVI model
- Trained scVI model with gamma-Poisson (negative binomial) decoder
- Learned library-size means (l_mu) per cell

## Outputs

- synthetic_counts_prior.csv — synthetic cell-by-gene UMI count matrix of expected dimensionality
- prior_sampling.py — script demonstrating z ~ Normal(0,1) and library sizes ~ Normal(l_mu, 1)

## How to apply

Train the scVI model on the preprocessed PBMC4k count matrix (4182 cells, 1000 highly variable genes) with its gamma-Poisson (negative binomial) decoder. For generation, decide between two variants: the standard posterior approach samples z and library sizes from p(z|x) and the inferred library-size posterior, while the paper's prior-sampling adaptation draws z from Normal(0, 1) and library sizes from Normal(l_mu, 1), where l_mu is the learned library-size mean. Feed the prior-sampled z and library sizes through the gamma-Poisson decoder to obtain synthetic UMI counts, then write the cell-by-gene matrix to synthetic_counts_prior.csv and the code to prior_sampling.py. Rationale: decoupling generation from copies of the original observations lets the prior variant generate data without relying on per-cell posterior encodings, complementing the posterior variant that showed high variability on small pilot datasets.

## Related tools

- **scVI** (Generative VAE whose gamma-Poisson decoder consumes the sampled latent z and Normal(l_mu, 1) library-size factors to produce synthetic UMI counts; used in prior and posterior variants) — https://github.com/scverse/scvi-tools
- **Julia** (Runtime (Julia 1.5.0) for the scDBM-paper analysis environment in which scVI (version 0.6.0) is installed via Conda.jl) — https://github.com/MTreppner/scDBM-paper

## Examples

```
python prior_sampling.py  # draws z ~ Normal(0,1), library sizes ~ Normal(l_mu,1), saves synthetic_counts_prior.csv
```

## Evaluation signals

- prior_sampling.py runs end-to-end without error
- The script demonstrably draws z from Normal(0, 1) and library sizes from Normal(l_mu, 1) rather than from the posterior
- synthetic_counts_prior.csv is a cell-by-gene count matrix of the expected dimensionality (e.g. matching the 1000 highly variable genes of the PBMC4k pilot)
- Generated counts are non-negative integers consistent with a gamma-Poisson (negative binomial) decoder
- Per-gene statistics (e.g. median, proportion of zero counts) of synthetic data can be compared to the pilot data via the mean absolute deviation per gene as in the paper

## Limitations

- The scVI (posterior) variant showed high variability in clustering similarity, most likely due to amplifying artifacts of small datasets — the prior variant does not fix this bias.
- All approaches overrepresented highly abundant cell types and missed less abundant cell types.
- For 10x Genomics datasets (higher sparsity), it is more challenging to make an inference from small to larger datasets.

## Evidence

- [methods] Replace the inferred library-size posterior with sampling library sizes from Normal(l_mu, 1), where l_mu is the learned library-size mean.: [evidence span withheld — anchor status: not_found]
- [readme] scVI in two variants, generating samples from the posterior distribution, the standard approach, or the prior distribution.: "we considered single-cell variational inference (scVI) in two variants, generating samples from the posterior distribution, the standard approach, or the prior distribution."
- [results] we sampled from the scDBM using Gibbs sampling and from scVI using the prior and posterior distribution: "we sampled from the scDBM using Gibbs sampling and from scVI using the prior and posterior distribution"
