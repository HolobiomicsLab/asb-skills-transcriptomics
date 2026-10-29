---
name: cell-type-separation-visualization
description: Use when a VEGA (or similar interpretable VAE) model has been trained
  on single-cell RNA-seq with gene-module-wired latents and you must check that latent
  dimensions separate annotated cell populations before interpreting module activities.
when_to_use_negative:
- The model is a generic black-box VAE (e.g. plain scVI) whose latent dimensions are
  not tied to gene modules — separation would not be interpretable per module
- The input data lacks pre-annotated cell type labels, so separation cannot be assessed
  against a ground truth
- Prior gene module knowledge (e.g. MSigDB) is incomplete or not context-specific
  for the tissue, so GMV wiring may misrepresent the biology
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: VEGA
  role: Interpretable VAE with sparse linear decoder wired to GMVs; its latent space
    provides the embedding to visualize
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Wraps the single-cell dataset as AnnData and provides UMAP embedding/plotting
    of the VEGA latent space colored by cell type
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scVI
  role: Ecosystem reference within which VEGA is implemented; alternative encoder
    backbone for count data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:2c9e2b35e3a870159ab97b5cfba696bcc843bbe76685250cc662a4c7a9c718e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cell-type-separation-visualization

## Summary

Visualize the VEGA-trained interpretable latent embedding (e.g. via UMAP) colored by pre-annotated cell types to verify that the GMV-structured latent space disentangles cell type and cell state identity, as demonstrated for the Darmanis GBM single-cell dataset.

## When to use

Use when a VEGA (or similar interpretable VAE) model has been trained on single-cell RNA-seq with gene-module-wired latents and you must check that latent dimensions separate annotated cell populations before interpreting module activities.

## When NOT to use

- The model is a generic black-box VAE (e.g. plain scVI) whose latent dimensions are not tied to gene modules — separation would not be interpretable per module
- The input data lacks pre-annotated cell type labels, so separation cannot be assessed against a ground truth
- Prior gene module knowledge (e.g. MSigDB) is incomplete or not context-specific for the tissue, so GMV wiring may misrepresent the biology

## Inputs

- Trained VEGA model with GMV-wired sparse linear decoder
- AnnData object with log-normalized single-cell expression and pre-annotated cell type labels (e.g. from GEO GSE84465 Darmanis GBM dataset)
- GMT file defining gene module variables (e.g. Carro et al. ARACNe TF-target network with STAT3, OLIG2 modules)

## Outputs

- UMAP embedding of the VEGA latent space colored by pre-annotated cell types
- Per-cell GMV activity table from the interpretable latent layer
- Qualitative assessment of cell type / cell state separation in the embedding

## How to apply

Train the VEGA model (pytorch, scanpy/scvi-tools ecosystem) on a preprocessed, log-normalized AnnData object whose decoder is wired one-to-one to gene module variables (GMVs) parsed from a GMT file, e.g. the Carro et al. GBM ARACNe network with STAT3 and OLIG2 modules. Compute a UMAP embedding of the trained VEGA latent space for all single cells and color it by the pre-annotated cell types (e.g. neoplastic cells, oligodendrocyte precursor cells). Judge success qualitatively: annotated cell types should form distinguishable clusters in the latent embedding, confirming the biologically informed latent space captures cell type and cell state identity. For the GBM case, follow up by extracting per-cell GMV activities from the latent layer and confirming the expected pattern (anticorrelated STAT3/OLIG2 activity in neoplastic cells; OLIG2 activity in OPCs).

## Related tools

- **VEGA** (Interpretable VAE with sparse linear decoder wired to GMVs; its latent space provides the embedding to visualize) — https://github.com/LucasESBS/vega
- **scanpy** (Wraps the single-cell dataset as AnnData and provides UMAP embedding/plotting of the VEGA latent space colored by cell type) — https://scanpy.readthedocs.io/en/stable/
- **scVI** (Ecosystem reference within which VEGA is implemented; alternative encoder backbone for count data)

## Examples

```
bash train_all_models.sh; jupyter notebook  # then plot UMAP of VEGA latent colored by cell type as in vega-reproducibility notebooks
```

## Evaluation signals

- Pre-annotated cell types form visually distinguishable, non-overlapping clusters in the UMAP of the VEGA latent space
- Per-cell GMV activity table shows anticorrelated STAT3 and OLIG2 activity within neoplastic cells (qualitative criterion from the GBM task)
- OLIG2 GMV activity is inferred as activated in oligodendrocyte precursor cells (OPCs), matching Fig. 2d-e of the paper
- Latent embedding retains biological signal comparable to reference reproductions in the vega-reproducibility notebooks

## Limitations

- The sparse, single-layer decoder prevents further improvement of generalizability and robustness
- The generative capacity of VEGA is limited, sacrificing predictive performance for biological interpretability
- Hard-coded decoder connections leave no room for correcting prior knowledge about gene modules when context requires it

## Evidence

- [abstract] decoder wiring mirrors user-provided gene modules: "we introduce a novel sparse Variational Autoencoder architecture, VEGA (VAE Enhanced by Gene Annotations), whose decoder wiring mirrors user-provided gene modules"
- [intro] interpretable latent space objective: "the scope of VEGA is twofold, (1) encoding data over an interpretable latent space and (2) inferring gene module activities for out-of-sample data"
- [methods] embedding/visualization workflow: "* Embed single-cell data into an interpretable latent space"
- [methods] preprocessing recommendation: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [readme] ecosystem and inputs: [evidence span withheld — anchor status: external_doc]
