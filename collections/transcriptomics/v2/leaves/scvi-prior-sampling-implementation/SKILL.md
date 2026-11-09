---
name: scvi-prior-sampling-implementation
description: Use when generating synthetic scRNA-seq observations with a trained VAE
  (scVI) but you want generation independent of the original observations, i.e., sampling
  directly from the prior of the latent space rather than the posterior.
when_to_use_negative:
- You need the standard scVI generative variant that conditions generation on copies
  of original observations via the posterior p(z|x) rather than the prior.
- Your trained model lacks a learned library-size mean (l_mu) or does not use a gamma-Poisson
  (negative binomial) decoder, so prior sampling with Normal(l_mu, 1) library sizes
  is not well-defined.
- 'Your goal is batch correction: the paper examined all models without using potentially
  available batch correction capabilities and explicitly omitted batch-correction
  methods from its scope.'
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scVI
  role: Trained VAE whose posterior sampling p(z|x) and inferred library-size posterior
    are replaced by prior draws z ~ Normal(0,1) and library sizes ~ Normal(l_mu,1),
    decoded with the gamma-Poisson decoder to generate synthetic counts
  repo: https://github.com/scverse/scvi-tools
- name: scDBM-paper
  role: Reference implementation repository for the paper's scVI prior/posterior sampling
    analyses and Pluto notebooks that call scVI
  repo: https://github.com/MTreppner/scDBM-paper
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:812184bda476dc793eab9bf148fc8fa31baab7c56cb876e1e69db673c15364dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/scvi-tools
  tools:
  - scDBM-paper
  - scVI
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# scvi-prior-sampling-implementation

## Summary

Reconstructs the scVI prior-sampling generative variant from Treppner et al. (2021), in which latent z is drawn from Normal(0, 1) and library sizes from Normal(l_mu, 1) instead of the standard posterior sampling p(z|x), and synthetic UMI counts are produced via scVI's gamma-Poisson (negative binomial) decoder.

## When to use

Use when generating synthetic scRNA-seq observations with a trained VAE (scVI) but you want generation independent of the original observations, i.e., sampling directly from the prior of the latent space rather than the posterior.

## When NOT to use

- You need the standard scVI generative variant that conditions generation on copies of original observations via the posterior p(z|x) rather than the prior.
- Your trained model lacks a learned library-size mean (l_mu) or does not use a gamma-Poisson (negative binomial) decoder, so prior sampling with Normal(l_mu, 1) library sizes is not well-defined.
- Your goal is batch correction: the paper examined all models without using potentially available batch correction capabilities and explicitly omitted batch-correction methods from its scope.

## Inputs

- Preprocessed single-cell UMI count matrix as the pilot dataset (e.g., PBMC4k, 4182 cells x 1000 highly variable genes)
- Trained scVI model (scvi 0.6.0) with learned encoder/decoder parameters and learned library-size mean l_mu

## Outputs

- prior_sampling.py — script that demonstrably draws z from Normal(0, 1) and library sizes from Normal(l_mu, 1)
- synthetic_counts_prior.csv — cell-by-gene synthetic UMI count matrix of expected dimensionality

## How to apply

Load the preprocessed count matrix (e.g., PBMC4k: 4182 cells x 1000 highly variable genes) as the pilot dataset for the trained scVI model (paper uses scvi 0.6, installable via Conda channel bioconda). In the Python scVI implementation, replace the posterior sampling step p(z|x) with sampling the latent z from a standard Normal(0, 1), exactly as the paper's adaptation specifies. Replace the inferred library-size posterior with sampling library sizes from Normal(l_mu, 1), where l_mu is the learned library-size mean. Feed the prior-sampled z and library sizes through scVI's gamma-Poisson (negative binomial) decoder to obtain synthetic UMI counts, save the sampling code to prior_sampling.py, and write the synthetic expression matrix to synthetic_counts_prior.csv. The rationale is to compare a fully generative prior variant against the standard posterior variant, which the paper found to show high variability on small pilot datasets, most likely due to amplifying artifacts.

## Related tools

- **scVI** (Trained VAE whose posterior sampling p(z|x) and inferred library-size posterior are replaced by prior draws z ~ Normal(0,1) and library sizes ~ Normal(l_mu,1), decoded with the gamma-Poisson decoder to generate synthetic counts) — https://github.com/scverse/scvi-tools
- **scDBM-paper** (Reference implementation repository for the paper's scVI prior/posterior sampling analyses and Pluto notebooks that call scVI) — https://github.com/MTreppner/scDBM-paper

## Examples

```
julia> using Conda; Conda.add("scvi=0.6", channel="bioconda")
```

## Evaluation signals

- prior_sampling.py runs end-to-end without error
- The script demonstrably draws latent z from Normal(0, 1) rather than from the posterior p(z|x)
- The script demonstrably draws library sizes from Normal(l_mu, 1), where l_mu is the learned library-size mean, rather than from the inferred library-size posterior
- synthetic_counts_prior.csv is a cell-by-gene count matrix of the expected dimensionality (e.g., 4182 cells x 1000 highly variable genes for PBMC4k)
- Synthetic UMI counts are produced by feeding prior-sampled z and library sizes through scVI's gamma-Poisson (negative binomial) decoder

## Limitations

- The scVI posterior variant (standard approach) showed high variability in clustering similarity, most likely due to amplifying artifacts of small datasets, so variant choice affects downstream comparability.
- All generative approaches overrepresented highly abundant cell types and missed less abundant cell types, and problems occurred with bimodal gene distributions.
- Conclusions from small pilot samples to larger datasets require that the pilot sample be representative, and quality evaluation of synthetic data can be ambiguous across multiple measures.
- Models were examined without using potentially available batch correction capabilities, and batch effects cannot currently be learned with scDBMs.

## Evidence

- [methods] Replace the posterior sampling step p(z|x) with sampling latent z from Normal(0, 1), exactly as the paper's adaptation specifies.: [evidence span withheld — anchor status: not_found]
- [methods] Sample library sizes from Normal(l_mu, 1) and decode via gamma-Poisson.: [evidence span withheld — anchor status: not_found]
- [readme] The paper considered scVI in two generative variants: posterior (standard) or prior distribution sampling.: "we considered single-cell variational inference (scVI) in two variants, generating samples from the posterior distribution, the standard approach, or the prior distribution"
- [abstract] The posterior variant showed high variability due to amplifying artifacts of small datasets.: "the scVI (posterior) variant resulted in high variability, most likely due to amplifying artifacts of small datasets"
- [readme] scVI 0.6 is installed via Conda channel bioconda for the paper's analyses.: [evidence span withheld — anchor status: external_doc]
