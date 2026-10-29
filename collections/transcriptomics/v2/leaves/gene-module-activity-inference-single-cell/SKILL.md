---
name: gene-module-activity-inference-single-cell
description: Use when you have a preprocessed single-cell dataset (e.g., Scanpy AnnData)
  and a GMT gene-set file and need pathway-level activity per cell, with downstream
  testing of which modules are significantly activated.
when_to_use_negative:
- You need high generative/predictive performance rather than interpretability — VEGA's
  sparse, single-layer decoder sacrifices predictive performance for biological interpretability
- You need the model to correct or update prior knowledge about gene modules — the
  hard-coded decoder connections leave no room for correcting prior knowledge when
  the context requires it
- You expect accurate reconstruction/generation — VEGA's generative capacity is limited
  by design
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VEGA
  role: 'Core model: sparse VAE whose decoder wiring mirrors user-provided gene modules;
    used to embed single-cell data into an interpretable latent space and infer single-cell
    gene module activities'
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Wraps the single-cell dataset as an AnnData object and performs recommended
    preprocessing (filtering and normalization) before VEGA
- name: scvi-tools
  role: Ecosystem used by the VEGA implementation (PyTorch, scanpy, scvi-tools)
- name: f-scLVM
  role: Prior Bayesian latent variable model with explicit prior biological knowledge
    specification that VEGA improves on in computational cost and out-of-sample inference
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:a0eb761296cb3195609107375e81c94e69459bd412edcc5df208b056e5cc3fb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - f-scLVM
  - scanpy
  - scvi-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gene-module-activity-inference-single-cell

## Summary

Infers per-cell gene module activities (GMVs) for user-defined gene sets (e.g., Reactome pathways from a GMT file) by encoding single-cell data through VEGA, a VAE whose sparse linear decoder is hard-wired to mirror the gene module membership matrix, yielding an interpretable latent space.

## When to use

Use when you have a preprocessed single-cell dataset (e.g., Scanpy AnnData) and a GMT gene-set file and need pathway-level activity per cell, with downstream testing of which modules are significantly activated.

## When NOT to use

- You need high generative/predictive performance rather than interpretability — VEGA's sparse, single-layer decoder sacrifices predictive performance for biological interpretability
- You need the model to correct or update prior knowledge about gene modules — the hard-coded decoder connections leave no room for correcting prior knowledge when the context requires it
- You expect accurate reconstruction/generation — VEGA's generative capacity is limited by design

## Inputs

- Single-cell dataset wrapped as a Scanpy AnnData object (e.g., Kang et al. PBMCs count matrix), preprocessed by filtering and normalization
- GMT file specifying gene module variables (GMVs) and gene membership, e.g., from MSigDB, Reactome, Gene Ontology, PANTHER, or MolSigDB

## Outputs

- Per-cell gene module activity (GMV) table, e.g., vega_gmv_activity.csv, with one activity score per cell per pathway latent dimension
- Trained VEGA model (e.g., saved under ./trained_models/ in the reproducibility repo)
- Bayesian test results identifying significantly activated gene modules in the dataset

## How to apply

Load the single-cell count matrix as a Scanpy AnnData object and apply recommended preprocessing (filtering and normalization) before passing it to VEGA. Parse the GMT file (e.g., Reactome or MSigDB) to build the gene module membership mask that wires the sparse linear decoder of the VAE. Instantiate and train the VEGA model (PyTorch, Scanpy/scvi-tools ecosystem) on the preprocessed data, then use the trained encoder to infer gene module activities (GMVs) at the single-cell level for every cell, including out-of-sample inference. Optionally apply VEGA's Bayesian differential testing procedure to identify significantly activated gene modules. Rationale: the decoder wiring mirrors user-provided gene modules, so each latent variable directly corresponds to a pathway or transcription factor, unlike generic VAEs (scVI, scGen) that lack a biologically meaningful latent representation.

## Related tools

- **VEGA** (Core model: sparse VAE whose decoder wiring mirrors user-provided gene modules; used to embed single-cell data into an interpretable latent space and infer single-cell gene module activities) — https://github.com/LucasESBS/vega
- **scanpy** (Wraps the single-cell dataset as an AnnData object and performs recommended preprocessing (filtering and normalization) before VEGA)
- **scvi-tools** (Ecosystem used by the VEGA implementation (PyTorch, scanpy, scvi-tools))
- **f-scLVM** (Prior Bayesian latent variable model with explicit prior biological knowledge specification that VEGA improves on in computational cost and out-of-sample inference)

## Examples

```
pip install scvega
```

## Evaluation signals

- Qualitative segregation check: interferon-alpha/beta signaling GMV activity segregates stimulated from naive/control cells (Kang et al. PBMCs, Fig. 1b-d pattern)
- Cell-type specificity check: tryptophan catabolism GMV activity separates innate immune cells (dendritic cells, FCGR3A+ and CD14+ monocytes) from adaptive immune cells
- Sanity check that the decoder mask exactly mirrors the GMT-derived gene module membership matrix (one latent dimension per GMV)
- Out-of-sample inference check: encoder must produce GMVs for unseen cells without retraining
- Bayesian differential activity test returns a coherent set of significantly activated gene modules for the dataset

## Limitations

- The sparse, single-layer decoder architecture limits generalizability and robustness
- VEGA's limited generative capacity sacrifices predictive performance for biological interpretability of the latent space
- Hard-coded decoder connections cannot correct prior knowledge about gene modules when the context requires it
- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific; additional unannotated genes may play important roles in certain gene modules

## Evidence

- [abstract] VEGA decoder wiring mirrors gene modules: "we introduce a novel sparse Variational Autoencoder architecture, VEGA (VAE Enhanced by Gene Annotations), whose decoder wiring mirrors user-provided gene modules"
- [methods] Preprocessing recommendation: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [methods] Single-cell inference scope: "Inference of gene module activities at the single-cell level"
- [readme] GMT input requirement from README: [evidence span withheld — anchor status: external_doc]
- [readme] Reproducibility workflow from README: [evidence span withheld — anchor status: external_doc]
