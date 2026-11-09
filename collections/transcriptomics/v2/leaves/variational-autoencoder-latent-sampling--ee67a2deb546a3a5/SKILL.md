---
name: variational-autoencoder-latent-sampling
description: Use when generating synthetic scRNA-seq count matrices from a trained
  scVI model while avoiding the posterior-sampling bias that amplifies artifacts of
  small pilot datasets.
when_to_use_negative:
- When evaluating faithfulness of synthetic data to a specific pilot's cell-type composition
  via label mapping, since prior sampling decouples generated cells from the pilot's
  posterior representations
- When batch effects are present and correction is needed — the study examined all
  models without using potentially available batch correction capabilities, and batch
  effects cannot be learned with scDBMs
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scVI
  role: VAE model trained on pilot scRNA-seq counts; adapted so latent z is sampled
    from Normal(0, 1) and library sizes from Normal(l_mu, 1) instead of the posterior,
    decoded via gamma-Poisson
  repo: https://github.com/scverse/scvi-tools
- name: Julia
  role: Runtime (Julia 1.5.0) for the Pluto notebooks that orchestrate scVI installation
    (Conda.add("scvi=0.6")) and comparison against scDBM
  repo: https://github.com/MTreppner/scDBM-paper
- name: BoltzmannMachines.jl
  role: Underlying Julia package used by scDBM.jl for the scDBM baseline that prior-sampling
    scVI is compared against
  repo: https://github.com/MTreppner/scDBM.jl
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:1ceec496b1421d61ac53f7a44dfb6fa5e9caf9e6440249880f6344b347e941c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/scvi-tools
  tools:
  - BoltzmannMachines.jl
  - Julia
  - scVI
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# variational-autoencoder-latent-sampling

## Summary

A scVI generative variant that draws latent variables directly from the prior Normal(0, 1) instead of the posterior p(z|x), and library sizes from Normal(l_mu, 1), then decodes them through scVI's gamma-Poisson (negative binomial) decoder to synthesize UMI counts without relying on copies of original observations.

## When to use

Use when generating synthetic scRNA-seq count matrices from a trained scVI model while avoiding the posterior-sampling bias that amplifies artifacts of small pilot datasets.

## When NOT to use

- When evaluating faithfulness of synthetic data to a specific pilot's cell-type composition via label mapping, since prior sampling decouples generated cells from the pilot's posterior representations
- When batch effects are present and correction is needed — the study examined all models without using potentially available batch correction capabilities, and batch effects cannot be learned with scDBMs

## Inputs

- Preprocessed scRNA-seq count matrix (e.g., PBMC4k: 4182 cells × 1000 highly variable genes; Segerstolpe: 2090 cells × 2000 highly variable genes)
- A trained scVI model with learned library-size mean l_mu
- Target number of synthetic cells to generate (original study size)

## Outputs

- prior_sampling.py — script that demonstrably draws z from Normal(0, 1) and library sizes from Normal(l_mu, 1)
- synthetic_counts_prior.csv — cell-by-gene synthetic UMI count matrix of the expected dimensionality

## How to apply

Load the preprocessed count matrix (e.g., PBMC4k, 4182 cells × 1000 highly variable genes) and train the scVI model (default architecture: one hidden layer for encoder and decoder, latent dimensionality 10). Replace the standard posterior sampling step p(z|x) with sampling latent z from the standard normal prior Normal(0, 1). Replace the inferred library-size posterior with sampling library sizes from Normal(l_mu, 1), where l_mu is the learned library-size mean. Feed the prior-sampled z and library sizes through scVI's gamma-Poisson (negative binomial) decoder to obtain synthetic UMI counts, stacking samples (e.g., replicating pilot inputs seven times) until the original study size is reached. Save the sampling script (prior_sampling.py) and the synthetic matrix (synthetic_counts_prior.csv). Rationale: the paper found this prior variant shows much lower variability than posterior sampling when extrapolating from small pilots, because posterior sampling amplifies the sampling bias of small datasets.

## Related tools

- **scVI** (VAE model trained on pilot scRNA-seq counts; adapted so latent z is sampled from Normal(0, 1) and library sizes from Normal(l_mu, 1) instead of the posterior, decoded via gamma-Poisson) — https://github.com/scverse/scvi-tools
- **Julia** (Runtime (Julia 1.5.0) for the Pluto notebooks that orchestrate scVI installation (Conda.add("scvi=0.6")) and comparison against scDBM) — https://github.com/MTreppner/scDBM-paper
- **BoltzmannMachines.jl** (Underlying Julia package used by scDBM.jl for the scDBM baseline that prior-sampling scVI is compared against) — https://github.com/MTreppner/scDBM.jl

## Examples

```
z = np.random.normal(0, 1, (n_cells, 10)); lib = np.random.normal(l_mu, 1); counts = scvi_model.decoder(z, lib)
```

## Evaluation signals

- prior_sampling.py runs end-to-end without error and demonstrably draws z from Normal(0, 1) and library sizes from Normal(l_mu, 1) rather than from the posterior
- synthetic_counts_prior.csv is a cell-by-gene count matrix of the expected dimensionality (genes matching the 1000 or 2000 highly variable gene set)
- Generated synthetic cell count equals the original study's cell count for every dataset × model × pilot size × repetition combination (1080 generated artifacts across 2 datasets × 3 models × 6 pilot sizes × 30 repetitions)
- Variability of downstream clustering similarity (ARI, DBI) for the prior variant is substantially lower than the scVI posterior variant, which showed high variability due to amplifying artifacts of small datasets

## Limitations

- Posterior sampling in scVI amplifies the sampling bias of small pilot datasets; prior sampling trades this off for reduced fidelity to the pilot's specific composition
- All approaches overrepresented highly abundant cell types and missed less abundant cell types
- All approaches learn the univariate distribution of most genes but have problems with bimodality
- For sparser 10x Genomics datasets it is more challenging to make an inference from small to larger datasets than for Smart-seq2 data
- Conclusions from small samples to larger datasets require that the pilot sample is representative, which depends on keeping experimental settings unchanged

## Evidence

- [abstract] we considered single-cell variational inference (scVI) in two variants, generating samples from the posterior distribution, the standard approach, or the prior distribution: "First, we considered single-cell variational inference (scVI) in two variants, generating samples from the posterior distribution, the standard approach, or the prior distribution."
- [results] the scVI (posterior) variant resulted in high variability, most likely due to amplifying artifacts of small datasets: "we find that the scVI (posterior) variant resulted in high variability, most likely due to amplifying artifacts of small datasets."
- [full_text] replace the posterior sampling step p(z|x) with sampling latent z from Normal(0, 1), exactly as the paper's adaptation specifies: [evidence span withheld — anchor status: not_found]
- [full_text] sampling library sizes from Normal(l_mu, 1), where l_mu is the learned library-size mean: [evidence span withheld — anchor status: not_found]
- [readme] Conda.add("scvi=0.6", channel="bioconda"): [evidence span withheld — anchor status: external_doc]
