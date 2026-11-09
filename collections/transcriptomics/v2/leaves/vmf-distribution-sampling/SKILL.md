---
name: vmf-distribution-sampling
description: 'Use when reimplementing or verifying scPhere''s hyperspherical VAE:
  you need vMF posterior sampling, the closed-form vMF-vs-uniform-sphere KL, or the
  NB observation model with softmax means.'
when_to_use_negative:
- The latent geometry is hyperbolic rather than spherical — scPhere uses latent_dist='wn'
  (wrapped normal) for hyperbolic (Lorentz-model/Poincaré-disk) embeddings, not the
  vMF machinery.
- You need a Euclidean-latent VAE baseline (standard multidimensional normal prior);
  the vMF sampling and hyperspherical KL do not apply.
- You need scPhere's future extensions (imputation of missing counts, ambient RNA
  removal, perturbation modeling, multimodal integration) — these are explicitly not
  implemented.
tools:
- name: scPhere
  role: Reference implementation whose SCPHERE VAE (latent_dist='vmf', observation_dist='nb')
    defines the vMF sampling, hyperspherical KL, and NB decoder to be reconstructed
    and verified
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Backend used to build and train the scPhere VAE (tensorflow==1.14 per README)
- name: tensorflow-probability
  role: Distribution utilities for the probabilistic latent/observation layers (v0.7.0
    per README)
- name: numpy
  role: Numerical implementation of rejection sampling, Householder rotations, and
    unit tests of sampled moments
- name: scipy
  role: Numerical routines (special functions for vMF normalizing constants / Bessel-type
    terms) supporting KL and likelihood computations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:f11e57ada204f0f0c176e59e729be4f440918ffc7791ab46c3b5856a6bfcd4a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - TensorFlow
  - numpy
  - scPhere
  - scipy
  - tensorflow-probability
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# vmf-distribution-sampling

## Summary

Reconstructs the von Mises–Fisher (vMF) latent machinery of scPhere: rejection-sampled vMF draws mapped to arbitrary mean directions via Householder rotations, the closed-form KL divergence to a uniform prior on the unit hypersphere, and the negative-binomial decoder used with z_dim=2 spherical embeddings.

## When to use

Use when reimplementing or verifying scPhere's hyperspherical VAE: you need vMF posterior sampling, the closed-form vMF-vs-uniform-sphere KL, or the NB observation model with softmax means.

## When NOT to use

- The latent geometry is hyperbolic rather than spherical — scPhere uses latent_dist='wn' (wrapped normal) for hyperbolic (Lorentz-model/Poincaré-disk) embeddings, not the vMF machinery.
- You need a Euclidean-latent VAE baseline (standard multidimensional normal prior); the vMF sampling and hyperspherical KL do not apply.
- You need scPhere's future extensions (imputation of missing counts, ambient RNA removal, perturbation modeling, multimodal integration) — these are explicitly not implemented.

## Inputs

- UMI count matrix (e.g., .mtx read via scphere.util.util.read_mtx, transposed so cells are rows)
- 0-based batch ID vector (pandas TSVs concatenated, e.g., patient/health/location batch vectors)
- Model configuration (n_gene, n_batch list, z_dim=2, latent_dist='vmf', observation_dist='nb', seed=0)
- Small synthetic inputs for numerical unit tests of vMF moments, KL values, and log-likelihoods

## Outputs

- vMF posterior samples on the unit hypersphere (z_mean embeddings, e.g., uc_epi_latent_250epoch.tsv)
- Closed-form KL divergence values between vMF posterior and uniform hyperspherical prior
- Per-cell log-likelihoods (e.g., uc_epi_ll_250epoch.tsv) from the negative-binomial decoder
- Verification table vmf_components_verification.tsv with numeric results and pass/fail status per component

## How to apply

Instantiate the scPhere VAE configuration matching the SCPHERE constructor with latent_dist='vmf', observation_dist='nb', z_dim=2, and seed=0, since 'vmf' selects the spherical latent space ('wn' selects the hyperbolic wrapped-normal alternative). Implement the vMF latent distribution by drawing the concentration-dependent angular component via rejection sampling and rotating base samples to arbitrary mean directions with Householder rotations. Implement the closed-form KL divergence between the vMF posterior and the uniform prior on the unit hypersphere, which replaces the center-pulling multidimensional normal prior of Euclidean VAEs. Implement the negative-binomial observation model whose per-gene mean expression is computed with a softmax. Validate each component by comparing sampled moments, KL values, and log-likelihoods against the Methods formulas on small synthetic inputs, writing numeric results and pass/fail status to vmf_components_verification.tsv.

## Related tools

- **scPhere** (Reference implementation whose SCPHERE VAE (latent_dist='vmf', observation_dist='nb') defines the vMF sampling, hyperspherical KL, and NB decoder to be reconstructed and verified) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Backend used to build and train the scPhere VAE (tensorflow==1.14 per README))
- **tensorflow-probability** (Distribution utilities for the probabilistic latent/observation layers (v0.7.0 per README))
- **numpy** (Numerical implementation of rejection sampling, Householder rotations, and unit tests of sampled moments)
- **scipy** (Numerical routines (special functions for vMF normalizing constants / Bessel-type terms) supporting KL and likelihood computations)

## Examples

```
from scphere.model.vae import SCPHERE; model = SCPHERE(n_gene=x.shape[1], n_batch=0, z_dim=2, latent_dist='vmf', observation_dist='nb', seed=0)
```

## Evaluation signals

- Sampled vMF moments (mean direction and concentration) match theoretical values from the Methods formulas within tolerance on small synthetic inputs.
- Computed KL divergence values agree with the closed-form vMF-to-uniform-hypersphere formula on check cases.
- Negative-binomial log-likelihoods computed by the reimplementation match model.get_log_likelihood(x, batch) outputs on toy data.
- Embeddings have unit norm (points lie on the unit hypersphere), verified as z_mean = model.encode(x, batch) output shape (n_cells, z_dim) with ||z||≈1.
- vmf_components_verification.tsv records numeric results and pass/fail status for each component check.

## Limitations

- Hyperbolic embeddings use the wrapped normal ('wn') posterior, so vMF sampling cannot be reused for Lorentz-model/Poincaré-disk embeddings; moreover, few tools exist for processing hyperbolic data (e.g., efficient k-NN search).
- The paper does not establish imputation of missing counts or ambient RNA removal with scPhere; further studies are required.
- scPhere is tested with specific pinned versions (Python 3.6.7, TensorFlow v1.14.0, tensorflow-probability v0.7.0); deviating versions may break reproducibility of the vMF/NB components.

## Evidence

- [readme] Model construction with vMF latent space: "model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2],
                z_dim=2, latent_dist='vmf', batch_invariant=False, 
                observation_dist='nb', seed=0)"
- [readme] latent_dist semantics: vmf vs wn: "latent_dist: 'vmf' for spherical latent spaces, and 'wn' for hyperbolic latent spaces"
- [intro] vMF posteriors and uniform prior on unit hypersphere: [evidence span withheld — anchor status: not_found]
- [readme] Training trace of KL divergence and log-likelihood: "Plotting log-likelihood and kl-divergence at each iteration"
- [methods] Log-likelihood computation on trained model: "ll = model.get_log_likelihood(x, batch)"
