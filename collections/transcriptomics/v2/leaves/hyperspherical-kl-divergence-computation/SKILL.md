---
name: hyperspherical-kl-divergence-computation
description: Decision support for validating or reimplementing VAE training objectives
  where latent variables are vMF-distributed on a unit hypersphere and the KL term
  to a uniform hyperspherical prior must be computed in closed form.
when_to_use_negative:
- Latent space is hyperbolic (Lorentz model / Poincaré disk) — scPhere uses a wrapped
  normal ('wn') latent distribution there, not a vMF posterior with hyperspherical
  KL
- Input is raw UMI counts needing preprocessing or batch correction by an external
  method rather than reimplementing/verifying the vMF-VAE objective itself
- Standard Euclidean VAE (Gaussian latent prior) is sufficient — the hyperspherical
  KL machinery addresses the specific crowding distortion of normal priors on sparse
  droplet scRNA-seq data
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scPhere
  role: Reference implementation of the vMF-VAE whose SCPHERE model (latent_dist='vmf',
    observation_dist='nb') defines the posterior, KL, and decoder being reconstructed
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Backend for implementing the vMF sampler, KL term, and NB decoder (tested
    with tensorflow==1.14)
- name: tensorflow-probability
  role: Probability utilities supporting distribution sampling and log-density computations
    (tested with tensorflow-probability==0.7.0)
- name: numpy
  role: Numerical implementation of Householder rotations and comparison of sampled
    moments and KL values in unit tests
- name: scipy
  role: Special-function evaluation needed for vMF normalization and KL formulas on
    synthetic test inputs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:0b62e0799a890e2c0b2a16fd5e700db54f284562218f877b7d18a9dc465a8417
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# hyperspherical-kl-divergence-computation

## Summary

Reconstructs and numerically verifies the von Mises–Fisher (vMF) latent machinery of scPhere: rejection-sampled vMF posterior, the closed-form KL divergence between the vMF posterior and the uniform prior on the unit hypersphere, and the softmax-parameterized negative-binomial decoder that together replace the Gaussian/Euclidean VAE objective.

## When to use

Decision support for validating or reimplementing VAE training objectives where latent variables are vMF-distributed on a unit hypersphere and the KL term to a uniform hyperspherical prior must be computed in closed form.

## When NOT to use

- Latent space is hyperbolic (Lorentz model / Poincaré disk) — scPhere uses a wrapped normal ('wn') latent distribution there, not a vMF posterior with hyperspherical KL
- Input is raw UMI counts needing preprocessing or batch correction by an external method rather than reimplementing/verifying the vMF-VAE objective itself
- Standard Euclidean VAE (Gaussian latent prior) is sufficient — the hyperspherical KL machinery addresses the specific crowding distortion of normal priors on sparse droplet scRNA-seq data

## Inputs

- scPhere VAE configuration (n_gene, n_batch, z_dim, latent_dist='vmf', observation_dist='nb', seed)
- vMF posterior parameters (mean direction μ on the unit hypersphere, concentration κ) from the encoder
- UMI count matrix for log-likelihood checks (e.g., uc_epi.mtx read via scphere.util.util.read_mtx)

## Outputs

- Closed-form KL divergence values between vMF posterior and uniform hyperspherical prior
- vmf_components_verification.tsv with numeric results and pass/fail status for sampler moments, KL values, and log-likelihoods
- Per-cell log-likelihoods from model.get_log_likelihood(x, batch)

## How to apply

Instantiate the scPhere VAE configuration as in the SCPHERE constructor with latent_dist='vmf' (not 'wn', which is for hyperbolic spaces), observation_dist='nb', z_dim=2, and seed=0. Implement the vMF latent distribution by drawing the concentration-dependent angular component via rejection sampling and rotating base samples to arbitrary mean directions with Householder rotations. Implement the closed-form KL divergence between the vMF posterior and the uniform prior on the unit hypersphere — this term is what prevents the cell crowding caused by a multidimensional normal prior pushing representations to the latent center. Implement the negative-binomial observation model whose per-gene mean expression is computed with a softmax. Verify by unit tests comparing sampled moments, KL values, and log-likelihoods against the Methods formulas on small synthetic inputs, writing results to vmf_components_verification.tsv.

## Related tools

- **scPhere** (Reference implementation of the vMF-VAE whose SCPHERE model (latent_dist='vmf', observation_dist='nb') defines the posterior, KL, and decoder being reconstructed) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Backend for implementing the vMF sampler, KL term, and NB decoder (tested with tensorflow==1.14))
- **tensorflow-probability** (Probability utilities supporting distribution sampling and log-density computations (tested with tensorflow-probability==0.7.0))
- **numpy** (Numerical implementation of Householder rotations and comparison of sampled moments and KL values in unit tests)
- **scipy** (Special-function evaluation needed for vMF normalization and KL formulas on synthetic test inputs)

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)
```

## Evaluation signals

- Sampled vMF moments (mean direction and concentration-dependent angular spread) match the Methods formulas within numerical tolerance on synthetic inputs
- Closed-form KL values agree with independently computed numerical KL between vMF posterior and uniform prior on the unit hypersphere
- Negative-binomial log-likelihoods from the reimplemented decoder match model.get_log_likelihood(x, batch) outputs
- Training trace plots (plot_trace of log_likelihood and kl_divergence per iteration) show converging KL divergence rather than divergence or collapse to the sphere center
- All component checks in vmf_components_verification.tsv report pass

## Limitations

- The hyperbolic variant of scPhere uses a wrapped normal posterior, for which this closed-form hyperspherical KL does not apply
- Further studies are required to validate scPhere for imputing missing counts and removing ambient RNA contamination, so the machinery should not be assumed correct for those tasks
- Few tools exist to process data distributed in hyperbolic space (e.g., efficient k-NN search), limiting downstream reuse beyond the spherical case

## Evidence

- [other] workflow step 3: [evidence span withheld — anchor status: not_found]
- [intro] vMF objective rationale: [evidence span withheld — anchor status: not_found]
- [readme] model configuration: "latent_dist: 'vmf' for spherical latent spaces, and 'wn' for hyperbolic latent spaces"
- [readme] training diagnostics: "Plotting log-likelihood and kl-divergence at each iteration"
- [intro] motivation: "we minimize the distortion by embedding cells to a lower-dimensional hypersphere instead of a low-dimensional Euclidean space"
