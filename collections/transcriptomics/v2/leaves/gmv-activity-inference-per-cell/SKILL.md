---
name: gmv-activity-inference-per-cell
description: Use when single-cell transcriptomics data must be projected onto a prior
  gene module set (e.g., an ARACNe TF-target network in GMT format) to obtain per-cell
  module or master regulator activity scores with an interpretable latent space.
when_to_use_negative:
- When raw count data must be modeled directly — VEGA operates on normalized/log-transformed
  profiles; scVI-style count handling was only explored as a discussion extension,
  not the default
- When the goal is predictive/generative performance rather than biological interpretability
  — VEGA's sparse, single-layer decoder sacrifices generative capacity for interpretability
- When prior gene module annotations are unreliable or need correction for the context
  — VEGA's hard-coded decoder connections cannot correct incomplete or non-context-specific
  prior knowledge
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: VEGA
  role: Core VAE with sparse linear decoder wired to GMVs; embeds cells in an interpretable
    latent space and infers per-cell gene module activities, with Bayesian testing
    for module activation
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Preprocessing (normalization, log-transformation) and AnnData wrapping of
    the single-cell dataset before training VEGA; latent embedding visualization
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scvi-tools
  role: Ecosystem in which VEGA's PyTorch implementation is built, supporting model
    instantiation and training
  repo: https://scvi-tools.org/
- name: vega-reproducibility
  role: Code, preprocessed data archive, and train_all_models.sh plus jupyter notebooks
    to reproduce VEGA paper results including per-cell GMV activity figures
  repo: https://github.com/LucasESBS/vega-reproducibility
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:a9ce84c77053ad957740d048d2319a04352b20ba3fc69e5fec2791fef6528bd6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - scanpy
  - scvi-tools
  - vega-reproducibility
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# gmv-activity-inference-per-cell

## Summary

Infers gene module variable (GMV) activities at the single-cell level using VEGA, a variational autoencoder whose sparse linear decoder is wired one-to-one to user-provided gene modules (pathways, GRN targets, or cell type markers). Each latent dimension directly reports the activity of one biological module per cell, enabling interpretable readouts such as master regulator activity in glioblastoma.

## When to use

Use when single-cell transcriptomics data must be projected onto a prior gene module set (e.g., an ARACNe TF-target network in GMT format) to obtain per-cell module or master regulator activity scores with an interpretable latent space.

## When NOT to use

- When raw count data must be modeled directly — VEGA operates on normalized/log-transformed profiles; scVI-style count handling was only explored as a discussion extension, not the default
- When the goal is predictive/generative performance rather than biological interpretability — VEGA's sparse, single-layer decoder sacrifices generative capacity for interpretability
- When prior gene module annotations are unreliable or need correction for the context — VEGA's hard-coded decoder connections cannot correct incomplete or non-context-specific prior knowledge

## Inputs

- Single-cell RNA-seq dataset (e.g., Darmanis et al. GBM, GEO GSE84465) wrapped as a scanpy AnnData object, normalized and log-transformed
- GMT file specifying gene module variables (GMVs) and gene membership, e.g., from MSigDB, Reactome, or an inferred GRN such as the Carro et al. GBM ARACNe TF-target network

## Outputs

- Per-cell GMV activity table (one activity score per cell per gene module)
- Interpretable latent embedding of the single-cell data (e.g., visualized as UMAP colored by cell type)
- Significantly activated gene modules from VEGA's Bayesian testing procedure

## How to apply

Preprocess the single-cell dataset (normalization, log-transformation) in scanpy before passing it to VEGA, since VEGA expects normalized expression profiles rather than raw counts. Parse the gene modules into a GMT file (e.g., STAT3 and OLIG2 TF-target modules from the Carro et al. GBM ARACNe network, or MSigDB gene sets) and build an AnnData object. Instantiate the VEGA model so its sparse linear decoder connects each gene module variable (GMV) one-to-one to one latent dimension, then train the VAE in the PyTorch/scanpy/scvi-tools ecosystem. After training, infer GMV activities for each cell from the latent space, and optionally apply VEGA's Bayesian testing procedure to call significantly activated gene modules. Validate by checking that pre-annotated cell types separate in the latent UMAP and that known activity patterns are recovered (e.g., anticorrelated STAT3/OLIG2 GMV activity in neoplastic GBM cells, OLIG2 activation in oligodendrocyte precursor cells).

## Related tools

- **VEGA** (Core VAE with sparse linear decoder wired to GMVs; embeds cells in an interpretable latent space and infers per-cell gene module activities, with Bayesian testing for module activation) — https://github.com/LucasESBS/vega
- **scanpy** (Preprocessing (normalization, log-transformation) and AnnData wrapping of the single-cell dataset before training VEGA; latent embedding visualization) — https://scanpy.readthedocs.io/en/stable/
- **scvi-tools** (Ecosystem in which VEGA's PyTorch implementation is built, supporting model instantiation and training) — https://scvi-tools.org/
- **vega-reproducibility** (Code, preprocessed data archive, and train_all_models.sh plus jupyter notebooks to reproduce VEGA paper results including per-cell GMV activity figures) — https://github.com/LucasESBS/vega-reproducibility

## Examples

```
pip install scvega; import vega
```

## Evaluation signals

- Pre-annotated cell types separate in the UMAP of the VEGA latent embedding
- Per-cell GMV activity table shows anticorrelated STAT3 and OLIG2 activity in neoplastic GBM cells
- OLIG2 GMV activity is inferred as activated in oligodendrocyte precursor cells (OPCs), matching Fig. 2d-e
- The trained model's latent dimensions correspond one-to-one to the GMT-defined GMVs (decoder sparsity check)
- Bayesian module activation test returns biologically expected significantly activated modules

## Limitations

- The sparse, single-layer decoder architecture limits generalizability and robustness
- The generative capacity of VEGA is limited; predictive performance is sacrificed for biological interpretability of the latent space
- Hard-coded decoder connections leave no room for correcting prior knowledge about gene modules when the context requires it
- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific, since unannotated genes may play important roles in certain modules

## Evidence

- [abstract] Decoder wiring mirrors gene modules: "whose decoder wiring mirrors user-provided gene modules, providing direct interpretability to the latent variables"
- [methods] Per-cell GMV inference workflow: "Inference of gene module activities at the single-cell level"
- [methods] Bayesian module activation testing: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [methods] Preprocessing requirement: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [readme] GMT input format: [evidence span withheld — anchor status: external_doc]
