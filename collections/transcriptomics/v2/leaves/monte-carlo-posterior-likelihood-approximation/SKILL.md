---
name: monte-carlo-posterior-likelihood-approximation
description: Use when you have latent posterior parameters (mean, variance per GMV)
  for two cell groups from a trained VEGA model and need Bayesian evidence (|log_e(BF)|
  > 3) for differential GMV activity.
when_to_use_negative:
- You only have raw single-cell counts and no trained VEGA model — the posteriors
  must first be obtained by encoding data over the annotated latent space
- You need gene-level (not gene module-level) differential expression; this skill
  tests GMV activity, not individual gene markers
- Your model handles unannotated genes — VEGA's hard-coded, sparse single-layer decoder
  leaves no room for correcting prior knowledge about gene modules
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VEGA
  role: Provides the VAE whose latent GMV posteriors are Monte Carlo sampled, and
    hosts the Bayesian differential testing procedure in its package layout
  repo: github.com/LucasESBS/vega
- name: scVI
  role: Ecosystem whose differential testing design inspired the reusable function
    module structure
  repo: https://scvi-tools.org/
- name: scanpy
  role: Wraps the single-cell dataset that is passed to VEGA for encoding and posterior
    estimation
  repo: https://scanpy.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:9aab2334d3fad8f17149a7ed35e91cf153d7ec9de355f936f415ce5db0f9d72f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - VEGA
  - scVI
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Monte Carlo posterior likelihood approximation

## Summary

Monte Carlo sampling from the Gaussian latent posteriors of a trained VEGA model to numerically approximate marginal likelihoods and compute a log_e Bayes Factor for differential gene module variable (GMV) activity between two cell groups.

## When to use

Use when you have latent posterior parameters (mean, variance per GMV) for two cell groups from a trained VEGA model and need Bayesian evidence (|log_e(BF)| > 3) for differential GMV activity.

## When NOT to use

- You only have raw single-cell counts and no trained VEGA model — the posteriors must first be obtained by encoding data over the annotated latent space
- You need gene-level (not gene module-level) differential expression; this skill tests GMV activity, not individual gene markers
- Your model handles unannotated genes — VEGA's hard-coded, sparse single-layer decoder leaves no room for correcting prior knowledge about gene modules

## Inputs

- Latent posterior parameters (mean and variance per GMV) for two cell groups, from a trained VEGA encoder
- Gene module membership in GMT format (e.g. from MSigDB) defining the GMVs
- Single-cell dataset wrapped with Scanpy

## Outputs

- Per-GMV log_e Bayes Factors comparing the two cell groups
- Binary significance calls per GMV at the |log_e(BF)| > 3 threshold

## How to apply

Define mutually exclusive null and alternative hypotheses contrasting a single GMV's latent activity posterior between the two cell groups, following VEGA's differential testing framework. Draw Monte Carlo samples from each group's posterior using the per-GMV mean and variance produced by the VAE encoder, then numerically approximate the marginal likelihood under each hypothesis. Compute the log_e Bayes Factor from the sampled marginal likelihoods and call a GMV significantly differentially active when |log_e(BF)| > 3. Validate the implementation on synthetic posteriors containing one strongly differential and one non-differential GMV: the differential GMV must exceed the threshold while the non-differential one must not. Package the procedure as a reusable function module following the scvi-tools-inspired differential testing design of the VEGA package.

## Related tools

- **VEGA** (Provides the VAE whose latent GMV posteriors are Monte Carlo sampled, and hosts the Bayesian differential testing procedure in its package layout) — github.com/LucasESBS/vega
- **scVI** (Ecosystem whose differential testing design inspired the reusable function module structure) — https://scvi-tools.org/
- **scanpy** (Wraps the single-cell dataset that is passed to VEGA for encoding and posterior estimation) — https://scanpy.readthedocs.io/en/stable/

## Examples

```
import numpy as np; z_a = np.random.normal(mu_a, sd_a, size=(10000, n_gmvs)); z_b = np.random.normal(mu_b, sd_b, size=(10000, n_gmvs))
```

## Evaluation signals

- Strongly differential synthetic GMV yields |log_e(BF)| > 3 in the verification scenario
- Non-differential synthetic GMV yields |log_e(BF)| <= 3, matching the paper's significance criterion
- Monte Carlo estimate of the log_e Bayes Factor is stable (within tolerance) across repeated samplings with large sample counts
- Per-GMV posterior means and variances match those encoded by the trained VEGA model for each cell group
- Module integrates cleanly into the VEGA package layout following the scvi-tools-inspired differential testing design

## Limitations

- VEGA's sparse, single-layer decoder limits generalizability and generative capacity, which constrains how trustworthy the encoded posteriors are for downstream likelihood approximation
- Significance depends on a fixed |log_e(BF)| > 3 cutoff; Monte Carlo sampling noise can shift estimates near this threshold
- Hard-coded decoder connections cannot correct incomplete or non-context-specific prior knowledge from databases like MSigDB

## Evidence

- [methods] Bayesian differential testing procedure: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [readme] VEGA as annotated latent-variable VAE: [evidence span withheld — anchor status: external_doc]
- [readme] Ecosystem inspiration for the module: [evidence span withheld — anchor status: external_doc]
- [discussion] Architecture limits on generative capacity: "VEGA’s limited generative capacity sacrifices predictive performance for biological interpretability of the latent space"
