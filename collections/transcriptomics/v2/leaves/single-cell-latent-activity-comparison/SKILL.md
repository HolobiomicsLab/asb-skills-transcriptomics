---
name: single-cell-latent-activity-comparison
description: Use when comparing gene module (pathway/TF/cell-type) latent activity
  between two cell groups in scRNA-seq data, e.g. stimulated vs control PBMCs, and
  you need per-module significance rather than eyeballing latent means.
when_to_use_negative:
- You need corrected gene module membership or new modules discovered from data —
  VEGA's hard-coded sparse decoder leaves no room for correcting prior knowledge
- You need raw-count modeling rather than normalized expression — VEGA was shown handling
  counts only as an extension (SFig. 8), not in the differential test design
- The gene modules lack prior annotation (e.g. MSigDB/Reactome membership), since
  VEGA's latent space is defined entirely by user-provided GMT gene modules
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: VEGA
  role: Sparse VAE whose decoder wiring mirrors GMT gene modules; its latent posteriors
    feed the Monte Carlo Bayes Factor differential activity test
  repo: https://github.com/LucasESBS/vega
- name: scVI
  role: scvi-tools model ecosystem whose differential testing design VEGA's reusable
    testing module follows
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:05d732f415a442774376bc21b8044771ae3fa7abb1c1fda44b48358702a4ecb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - scVI
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# single-cell-latent-activity-comparison

## Summary

Statistically compares gene module variable (GMV) activity between two single-cell cell groups using VEGA's Bayesian differential testing procedure, which Monte Carlo samples the VAE latent posteriors to compute a log Bayes Factor per GMV.

## When to use

Use when comparing gene module (pathway/TF/cell-type) latent activity between two cell groups in scRNA-seq data, e.g. stimulated vs control PBMCs, and you need per-module significance rather than eyeballing latent means.

## When NOT to use

- You need corrected gene module membership or new modules discovered from data — VEGA's hard-coded sparse decoder leaves no room for correcting prior knowledge
- You need raw-count modeling rather than normalized expression — VEGA was shown handling counts only as an extension (SFig. 8), not in the differential test design
- The gene modules lack prior annotation (e.g. MSigDB/Reactome membership), since VEGA's latent space is defined entirely by user-provided GMT gene modules

## Inputs

- Single-cell dataset wrapped as a Scanpy AnnData object
- GMT file specifying gene module variables (GMVs) and gene membership, e.g. from MSigDB
- Trained VEGA model posteriors (per-GMV latent mean and variance) for each of two cell groups

## Outputs

- Per-GMV log_e Bayes Factors comparing latent activity between the two groups
- Binary significance calls per GMV (significant when |log_e(BF)| > 3)

## How to apply

For each GMV, define null and alternative hypotheses contrasting its latent activity posterior (per-GMV mean and variance from VEGA's encoder) between the two cell groups. Monte Carlo sample GMV activities from each group's posterior to numerically approximate the marginal likelihoods under both hypotheses, then compute the natural-log Bayes Factor from the sampled marginal likelihoods. Call a GMV significantly differentially active when |log_e(BF)| > 3, the paper's stated significance criterion. Package the procedure as a reusable function module in the VEGA package, following the scvi-tools-inspired differential testing design (VEGA is built on scanpy and the scvi-tools ecosystem). Validate on controlled synthetic posteriors containing one strongly differential and one non-differential GMV before running on real data.

## Related tools

- **VEGA** (Sparse VAE whose decoder wiring mirrors GMT gene modules; its latent posteriors feed the Monte Carlo Bayes Factor differential activity test) — https://github.com/LucasESBS/vega
- **scVI** (scvi-tools model ecosystem whose differential testing design VEGA's reusable testing module follows)

## Examples

```
log_bf = np.log(mc_marginal_h1) - np.log(mc_marginal_h0); sig = np.abs(log_bf) > 3
```

## Evaluation signals

- On synthetic posteriors with a strongly differential GMV and a non-differential GMV, the differential GMV yields |log_e(BF)| > 3 and the non-differential GMV yields |log_e(BF)| <= 3
- Log Bayes Factors are finite and signed in the direction of the group with higher posterior GMV activity
- Per-GMV outputs are reported for every GMV in the GMT file (no missing modules)
- The testing function is packaged as a reusable module consistent with VEGA's documented API/tutorials (vega-documentation.readthedocs.io)

## Limitations

- VEGA's sparse, single-layer decoder limits generalizability and robustness, and its limited generative capacity sacrifices predictive performance for interpretability
- Hard-coded decoder connections prevent correcting prior gene module knowledge when context requires it
- Prior knowledge from databases like MSigDB can be incomplete or not context-specific, so module activity tests only cover annotated genes

## Evidence

- [methods] Bayesian testing threshold and purpose: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [full_text] Differential GMV testing mechanism: [evidence span withheld — anchor status: not_found]
- [readme] Ecosystem the testing module follows: [evidence span withheld — anchor status: external_doc]
- [readme] Required inputs (AnnData + GMT): [evidence span withheld — anchor status: external_doc]
- [intro] Latent variables are a priori gene modules: "Here we propose VEGA (VAE enhanced by gene annotations), a VAE with a sparse linear decoder informed by biological networks"
