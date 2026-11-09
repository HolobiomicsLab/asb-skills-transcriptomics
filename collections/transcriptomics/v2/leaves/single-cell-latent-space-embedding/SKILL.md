---
name: single-cell-latent-space-embedding
description: Use when you have a normalized single-cell transcriptomic dataset (Scanpy
  AnnData) plus prior biological knowledge in GMT format and want each latent dimension
  to correspond to a named gene module rather than an opaque factor.
when_to_use_negative:
- Raw (unnormalized) count data — VEGA was shown handling normalized expression profiles,
  and count support (e.g. via scVI-style likelihoods) was only discussed as a future
  direction
- When no prior gene module annotations are available or the existing annotation (e.g.
  MSigDB) is known to be incomplete or not context-specific, since the hard-coded
  decoder connections cannot correct wrong priors
- When strong generative/predictive performance is the primary goal rather than interpretability
  — VEGA's single-layer sparse decoder sacrifices predictive performance for interpretability
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: VEGA
  role: VAE with sparse linear decoder whose wiring mirrors user-provided gene modules;
    trains the embedding and infers per-cell GMV activities
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Wraps the single-cell dataset as AnnData and performs preprocessing (normalization,
    log-transformation) before VEGA training
- name: scVI
  role: Part of the ecosystem VEGA is built on; discussed as a basis for handling
    count data in place of normalized expression profiles
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:818aad11e9c3b9bbcc36ced62032e7aafb28af2b2b66339f02fdfbe0552426fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-latent-space-embedding

## Summary

Embed a preprocessed single-cell RNA-seq dataset into an interpretable latent space using VEGA, a VAE whose sparse linear decoder is wired one-to-one to user-provided gene module variables (GMVs) such as pathways, gene regulatory networks, or transcription factors, so each latent dimension maps directly to biological activity.

## When to use

Use when you have a normalized single-cell transcriptomic dataset (Scanpy AnnData) plus prior biological knowledge in GMT format and want each latent dimension to correspond to a named gene module rather than an opaque factor.

## When NOT to use

- Raw (unnormalized) count data — VEGA was shown handling normalized expression profiles, and count support (e.g. via scVI-style likelihoods) was only discussed as a future direction
- When no prior gene module annotations are available or the existing annotation (e.g. MSigDB) is known to be incomplete or not context-specific, since the hard-coded decoder connections cannot correct wrong priors
- When strong generative/predictive performance is the primary goal rather than interpretability — VEGA's single-layer sparse decoder sacrifices predictive performance for interpretability

## Inputs

- Single-cell RNA-seq dataset wrapped as a Scanpy AnnData object, preprocessed (normalized, log-transformed)
- GMT file specifying gene module variables (GMVs) and gene membership, e.g. from MSigDB, Gene Ontology, PANTHER, MolSigDB, or Reactome
- Optionally pre-annotated cell type labels for validating latent space structure

## Outputs

- Trained VEGA model with a sparse linear decoder whose dimensions correspond to the input GMVs
- Per-cell latent embedding in the interpretable GMV space (visualizable via UMAP)
- Per-cell gene module activity table at single-cell resolution

## How to apply

Preprocess the single-cell dataset (normalization, log-transformation) in scanpy before passing it to VEGA, since VEGA expects normalized expression profiles rather than raw counts. Convert your prior knowledge into a GMT file listing gene module variables (e.g., MSigDB gene sets, Reactome pathways, or an ARACNe-derived transcription factor-target network defining GMVs like STAT3 and OLIG2), and instantiate the VEGA model from github.com/LucasESBS/vega, which is implemented in PyTorch within the scanpy/scvi-tools ecosystem so the sparse linear decoder is hard-wired one-to-one to the gene modules. Train the VAE on the AnnData object to embed cells into the interpretable latent space, then visualize the embedding (e.g., UMAP) colored by pre-annotated cell types to confirm cell type/cell state disentanglement. Because each latent dimension is a GMV, per-cell module activities can be read out directly at single-cell resolution, and VEGA also supports inference of gene module activities for out-of-sample data.

## Related tools

- **VEGA** (VAE with sparse linear decoder whose wiring mirrors user-provided gene modules; trains the embedding and infers per-cell GMV activities) — https://github.com/LucasESBS/vega
- **scanpy** (Wraps the single-cell dataset as AnnData and performs preprocessing (normalization, log-transformation) before VEGA training)
- **scVI** (Part of the ecosystem VEGA is built on; discussed as a basis for handling count data in place of normalized expression profiles)

## Examples

```
pip install scvega; import vega; vega_model = vega.VEGA(adata, gmts=['gbm_aracne.gmt']); vega_model.train(); latent = vega_model.to_latent(adata)
```

## Evaluation signals

- The latent space dimensions are 1:1 with the GMVs supplied in the GMT file — check decoder sparsity pattern matches gene module membership
- Pre-annotated cell types separate in a UMAP of the latent embedding (e.g. in the Darmanis GBM application, neoplastic vs. immune vs. oligodendrocyte lineage cells disentangle)
- Per-cell GMV activity table reproduces known biology, e.g. anticorrelated STAT3 and OLIG2 activity in neoplastic GBM cells and OLIG2 activation in oligodendrocyte precursor cells
- Out-of-sample data can be projected through the encoder to infer gene module activities, verifying the model generalizes beyond the training cells

## Limitations

- The sparse, single-layer decoder architecture limits generalizability and robustness
- VEGA's generative capacity is limited; it sacrifices predictive performance for biological interpretability of the latent space
- Hard-coded decoder connections leave no room for correcting prior knowledge about gene modules when context requires it
- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific, as unannotated genes can play important roles in gene modules

## Evidence

- [abstract] Decoder wiring mirrors gene modules: "whose decoder wiring mirrors user-provided gene modules, providing direct interpretability to the latent variables"
- [methods] Preprocessing recommendation: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [readme] GMT input requirement: [evidence span withheld — anchor status: external_doc]
- [readme] Implementation stack: [evidence span withheld — anchor status: external_doc]
- [discussion] Count-data caveat: "such that it could handle count data in place of normalized expression profiles (SFig. 8)."
- [discussion] Architecture limitation: "The clear limitations of the current architecture resides in the sparse, single-layer decoder of the model."
