---
name: scvi-tools-style-differential-testing-implementation
description: Use when you have trained a VEGA model with gene-module-informed latent
  posteriors and need to call gene modules significantly differentially active between
  two cell groups via a Bayes Factor.
when_to_use_negative:
- The latent variables are not tied to user-provided gene modules (e.g. an unconstrained
  VAE/scVI latent space with no GMT annotation), so there is no GMV posterior to contrast
- You only need dataset-wide module activation scores rather than a two-group statistical
  contrast — VEGA's single-cell GMV activity inference suffices without the Bayes
  Factor test
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: VEGA
  role: Sparse VAE whose gene-module-wired decoder supplies the interpretable latent
    GMV posteriors that the Bayes Factor differential test contrasts; hosts the testing
    module
  repo: https://github.com/LucasESBS/vega
- name: scVI
  role: 'Design reference: VEGA is implemented using the scvi-tools ecosystem and
    follows its differential testing design when packaging the procedure'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:04de50301749e943faae7829d11e3e1cf667ad4835d1bf039d5fe0cb0a806cd0
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# scvi-tools-style differential testing implementation

## Summary

Implements VEGA's Bayesian differential gene module variable (GMV) activity test — a Monte Carlo Bayes Factor procedure contrasting latent GMV activity posteriors between two cell groups — packaged as a reusable module following the scvi-tools-inspired differential testing design.

## When to use

Use when you have trained a VEGA model with gene-module-informed latent posteriors and need to call gene modules significantly differentially active between two cell groups via a Bayes Factor.

## When NOT to use

- The latent variables are not tied to user-provided gene modules (e.g. an unconstrained VAE/scVI latent space with no GMT annotation), so there is no GMV posterior to contrast
- You only need dataset-wide module activation scores rather than a two-group statistical contrast — VEGA's single-cell GMV activity inference suffices without the Bayes Factor test

## Inputs

- Posterior parameters (mean and variance) per GMV per cell group from a trained VEGA model's latent space
- Single-cell dataset wrapped with Scanpy (AnnData)
- GMT file specifying gene module variables (GMVs) and gene membership, e.g. from MSigDB
- Definition of the two cell groups being contrasted
- Synthetic latent posteriors for verification (one strongly differential GMV, one non-differential GMV)

## Outputs

- Per-GMV log_e Bayes Factors comparing the two cell groups
- Per-GMV binary significance calls (|log_e(BF)| > 3)
- Reusable differential-testing function module in the VEGA package layout

## How to apply

Define null and alternative hypotheses for a single GMV, contrasting its latent activity posterior (mean and variance per GMV per cell group) between two cell groups as in VEGA's differential testing framework. Monte Carlo sample GMV activities from each group's posterior to numerically approximate the marginal likelihoods under both hypotheses, then compute the log_e Bayes Factor from the sampled marginal likelihoods. Call a GMV significantly differentially active when |log_e(BF)| > 3, the significance criterion stated in the VEGA paper. Package the procedure as a reusable function module within the VEGA package layout, following the design of differential testing utilities in the scvi-tools ecosystem, on data wrapped with Scanpy (AnnData) with GMV membership from a GMT file. Verify on a controlled synthetic scenario containing one strongly differential and one non-differential GMV before applying to real posteriors.

## Related tools

- **VEGA** (Sparse VAE whose gene-module-wired decoder supplies the interpretable latent GMV posteriors that the Bayes Factor differential test contrasts; hosts the testing module) — https://github.com/LucasESBS/vega
- **scVI** (Design reference: VEGA is implemented using the scvi-tools ecosystem and follows its differential testing design when packaging the procedure)

## Examples

```
pip install scvega
```

## Evaluation signals

- On the synthetic verification scenario, the strongly differential GMV yields |log_e(BF)| > 3 and the non-differential GMV yields |log_e(BF)| <= 3
- Monte Carlo sampling of each group's posterior reproduces stable marginal-likelihood approximations (log BF insensitive to re-sampling seed within tolerance)
- The module exposes a function compatible with the VEGA package layout, mirroring the scvi-tools-inspired differential testing design
- Significance calls follow exactly the paper's stated criterion: significant iff |log_e(BF)| > 3

## Limitations

- The Bayes Factor is a Monte Carlo approximation of posterior probabilities of mutually exclusive hypotheses, so estimates carry sampling noise and depend on the number of samples drawn
- VEGA's sparse, single-layer decoder limits generative capacity and generalizability, which constrains the model whose posteriors feed the test
- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific, affecting the GMVs that can be tested

## Evidence

- [methods] Bayesian differential GMV testing procedure: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [full_text] Significance threshold: [evidence span withheld — anchor status: not_found]
- [readme] scvi-tools-inspired implementation ecosystem: [evidence span withheld — anchor status: external_doc]
- [methods] GMT input format: "GMT file(s) with the gene module membership, such as provided by databases like `MSigDB <https://www.gsea-msigdb.org/gsea/index.jsp>`_"
