---
name: bayes-factor-differential-module-testing
description: Use when comparing per-cell gene module variational (GMV) activities
  inferred by VEGA between two cell groups (or one-vs-rest across annotated cell types)
  to call significantly activated pathways, TF regulons, or cell type marker sets.
when_to_use_negative:
- The latent space is not constrained to a priori gene modules (e.g., a standard scVI
  or scGen model), since GMV posteriors would not map to annotated biological pathways
- You need to correct or extend the prior gene module annotations at test time — the
  paper notes hard-coded decoder connections 'do not leave any room for correcting
  prior knowledge about gene modules when the context requires it'
- The dataset contains many unannotated genes playing important roles, since prior
  knowledge from databases like MSigDB can be incomplete or not context-specific
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VEGA
  role: Variational autoencoder whose sparse decoder is wired from gene module annotations;
    provides the GMV posteriors and hosts the Bayesian differential activity testing
    procedure
  repo: https://github.com/LucasESBS/vega
- name: scVI
  role: scvi-tools ecosystem design pattern followed when packaging the differential
    testing function module
- name: scanpy
  role: Wraps the single-cell dataset (AnnData) and performs preprocessing before
    differential GMV testing
  repo: https://scanpy.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:c0b650d2089f62dee7cd59a155bffb5b1a7932e4f060c91eadd55d9d20a04e97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bayes-factor-differential-module-testing

## Summary

VEGA's Bayesian testing procedure that identifies gene modules (GMVs) significantly differentially active between two cell groups by Monte Carlo approximating posterior probabilities of mutually exclusive hypotheses and computing a log Bayes Factor, calling modules significant at |log_e(BF)| > 3.

## When to use

Use when comparing per-cell gene module variational (GMV) activities inferred by VEGA between two cell groups (or one-vs-rest across annotated cell types) to call significantly activated pathways, TF regulons, or cell type marker sets.

## When NOT to use

- The latent space is not constrained to a priori gene modules (e.g., a standard scVI or scGen model), since GMV posteriors would not map to annotated biological pathways
- You need to correct or extend the prior gene module annotations at test time — the paper notes hard-coded decoder connections 'do not leave any room for correcting prior knowledge about gene modules when the context requires it'
- The dataset contains many unannotated genes playing important roles, since prior knowledge from databases like MSigDB can be incomplete or not context-specific

## Inputs

- Trained VEGA model with GMV latent posteriors (mean and variance per module per cell group)
- Per-cell GMV activity matrix for the two cell groups being contrasted
- GMT file(s) specifying gene module membership (e.g., MSigDB, Reactome pathways)
- Cell group or cell type annotations on the Scanpy AnnData object

## Outputs

- Per-GMV log_e Bayes Factors
- Binary significance calls per GMV (significant if |log_e(BF)| > 3)
- Table of significant modules per cell type from one-vs-rest tests (e.g., saved to organoid_gmv_results.csv)

## How to apply

Train a VEGA model (PyTorch, scvi-tools ecosystem) whose sparse linear decoder is wired from a GMT file of gene module membership, so each latent GMV has an interpretable posterior mean/variance per group. For each GMV, define null and alternative hypotheses contrasting its latent activity posterior between the two cell groups, then Monte Carlo sample GMV activities from each group's posterior to numerically approximate the marginal likelihoods under both hypotheses and compute the log_e Bayes Factor. Call a module significantly differentially active when |log_e(BF)| > 3, the paper's stated significance criterion; for multi-cell-type datasets run the test one-vs-rest per annotated cell type. Package the procedure as a reusable function module following the scvi-tools-inspired differential testing design; validate on synthetic posteriors containing one strongly differential and one non-differential GMV before applying to real data.

## Related tools

- **VEGA** (Variational autoencoder whose sparse decoder is wired from gene module annotations; provides the GMV posteriors and hosts the Bayesian differential activity testing procedure) — https://github.com/LucasESBS/vega
- **scVI** (scvi-tools ecosystem design pattern followed when packaging the differential testing function module)
- **scanpy** (Wraps the single-cell dataset (AnnData) and performs preprocessing before differential GMV testing) — https://scanpy.readthedocs.io/en/stable/

## Examples

```
pip install scvega; import vega
```

## Evaluation signals

- Synthetic verification: a strongly differential GMV yields |log_e(BF)| > 3 while a non-differential GMV yields |log_e(BF)| <= 3
- One-vs-rest tests recover the correct cell type: each cell type marker-set GMV segregates its annotated population
- Biological sanity check: CELL_CYCLE_MITOTIC GMV activity correlates with MKI67 expression per cell at R2 ≈ 0.64
- Monte Carlo marginal likelihoods are numerically stable across repeated sampling runs (log BF reproducible)

## Limitations

- The Bayes Factor is a Monte Carlo approximation, so sampling variance can affect borderline calls near the |log_e(BF)| = 3 threshold
- Significance depends on the completeness of prior gene module annotations; unannotated genes important to a module will be missed
- The sparse, single-layer decoder limits generative capacity, trading predictive performance for latent interpretability

## Evidence

- [methods] Bayesian testing procedure: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [other] Monte Carlo BF from posterior hypotheses: [evidence span withheld — anchor status: not_found]
- [abstract] Interpretable decoder wiring: "whose decoder wiring mirrors user-provided gene modules, providing direct interpretability to the latent variables"
- [readme] README: VEGA purpose: [evidence span withheld — anchor status: external_doc]
- [readme] README: required inputs: [evidence span withheld — anchor status: external_doc]
- [discussion] MSigDB incompleteness caveat: "prior biological knowledge obtained from existing databases like MSigDB can be incomplete or not context-specific"
