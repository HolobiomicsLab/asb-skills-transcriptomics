---
name: synthetic-posterior-verification-design
description: Use when implementing or validating a Bayesian differential activity
  test (e.g., VEGA's Bayes Factor test on VAE latent GMVs) and you need a ground-truth
  scenario with one known-differential and one known-non-differential module.
when_to_use_negative:
- When the goal is differential expression of individual genes rather than differential
  activity of a priori specified gene modules — use standard single-gene DE tests
  instead
- When no trained VEGA model or latent posteriors exist yet — the test operates on
  VAE latent GMV activities, not raw count matrices
- When gene modules are not available in GMT form (e.g., MSigDB) — VEGA requires a
  priori GMV membership to define its latent space
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VEGA
  role: Sparse VAE whose latent variables are a priori gene module variables (GMVs);
    hosts the Bayesian differential GMV activity testing procedure being verified,
    in a scvi-tools-inspired package layout
  repo: https://github.com/LucasESBS/vega
- name: scVI
  role: Reference ecosystem whose differential testing design the VEGA test module
    follows; scvi-tools components wrap the Scanpy dataset used with VEGA
  repo: https://github.com/LucasESBS/vega
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:0d23c1857d57ebcc22245bf66cb054bbe38cab456067c98d07e981b3b8258de6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - scVI
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# synthetic-posterior-verification-design

## Summary

Designs controlled synthetic latent posteriors (mean and variance per gene module variable, per cell group) to numerically verify VEGA's Bayesian differential GMV activity test, which approximates marginal likelihoods by Monte Carlo sampling and reports a log_e Bayes Factor with significance at |log_e(BF)| > 3.

## When to use

Use when implementing or validating a Bayesian differential activity test (e.g., VEGA's Bayes Factor test on VAE latent GMVs) and you need a ground-truth scenario with one known-differential and one known-non-differential module.

## When NOT to use

- When the goal is differential expression of individual genes rather than differential activity of a priori specified gene modules — use standard single-gene DE tests instead
- When no trained VEGA model or latent posteriors exist yet — the test operates on VAE latent GMV activities, not raw count matrices
- When gene modules are not available in GMT form (e.g., MSigDB) — VEGA requires a priori GMV membership to define its latent space

## Inputs

- Synthetic latent posterior parameters (mean and variance per GMV, per cell group) for a controlled scenario
- VEGA latent space definition: GMT file specifying gene module variables (GMVs) and gene membership, e.g., from MSigDB
- Monte Carlo sample size for marginal likelihood approximation

## Outputs

- Per-GMV log_e Bayes Factor for the two-group contrast
- Per-GMV significance calls (differentially active if |log_e(BF)| > 3)
- Reusable differential-testing function module in the VEGA package layout

## How to apply

Define mutually exclusive null and alternative hypotheses contrasting a single GMV's latent activity posterior between two cell groups, following VEGA's differential testing framework. Generate synthetic latent posteriors — a mean and variance per GMV per cell group — constructing a controlled scenario with one strongly differential GMV and one non-differential GMV. Monte Carlo sample GMV activities from each group's posterior to numerically approximate the marginal likelihoods under both hypotheses, then compute the log_e Bayes Factor and call a GMV significantly differentially active when |log_e(BF)| > 3. Package the procedure as a reusable function module inside the VEGA package layout, following the scvi-tools-inspired differential testing design. Rationale: synthetic posteriors with known ground truth isolate the testing procedure from encoder training noise, so correctness of the BF approximation can be judged directly against the planted signal.

## Related tools

- **VEGA** (Sparse VAE whose latent variables are a priori gene module variables (GMVs); hosts the Bayesian differential GMV activity testing procedure being verified, in a scvi-tools-inspired package layout) — https://github.com/LucasESBS/vega
- **scVI** (Reference ecosystem whose differential testing design the VEGA test module follows; scvi-tools components wrap the Scanpy dataset used with VEGA) — https://github.com/LucasESBS/vega

## Examples

```
log_bf = np.log(mc_marginal(z1, mu1, var1)) - np.log(mc_marginal(z2, mu2, var2)); sig = np.abs(log_bf) > 3
```

## Evaluation signals

- Strongly differential synthetic GMV yields |log_e(BF)| > 3 (significance criterion met)
- Non-differential synthetic GMV yields |log_e(BF)| <= 3 (no false positive on planted null)
- Monte Carlo marginal likelihood estimates are stable across repeated sampling runs (log_e BF does not cross the ±3 threshold on re-run)
- Module integrates cleanly into the VEGA package layout and runs end-to-end on the synthetic posteriors, recording per-GMV log Bayes Factors

## Limitations

- The test is a Monte Carlo approximation, so estimates carry sampling noise; small sample counts can misclassify borderline GMVs near the |log_e(BF)| = 3 cutoff
- Grounded in VEGA's sparse, single-layer decoder with hard-coded gene module connections, so it only applies to latent posteriors of a priori specified GMVs, not free latents
- VEGA's limited generative capacity means synthetic posteriors verified in isolation may not reflect posterior behavior of a fully trained encoder on real single-cell data

## Evidence

- [results] VEGA's differential GMV test approximates the posterior probability of mutually exclusive hypotheses through Monte Carlo sampling of the VAE's latent variable distribution, yielding a Bayes Factor: [evidence span withheld — anchor status: not_found]
- [methods] Bayesian testing procedure for significantly activated gene modules: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [abstract] VEGA decoder wiring mirrors user-provided gene modules: "we introduce a novel sparse Variational Autoencoder architecture, VEGA (VAE Enhanced by Gene Annotations), whose decoder wiring mirrors user-provided gene modules"
- [readme] VEGA analyzes a priori specified latent variables such as pathways: [evidence span withheld — anchor status: external_doc]
