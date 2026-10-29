---
name: gene-regulatory-network-informed-decoder-wiring
description: Use when you have single-cell transcriptomics plus a prior gene regulatory
  network (e.g. ARACNe-derived TF-target modules) and want an interpretable latent
  space whose dimensions correspond to transcription factor / pathway activity rather
  than abstract factors.
when_to_use_negative:
- Input is raw count data — VEGA was demonstrated on normalized expression profiles;
  the article notes handling counts in place of normalized profiles (e.g. within scVI)
  is a discussed extension, not the wired-decoder use case
- No prior gene module or network annotation exists — VEGA's decoder requires user-provided
  gene modules (e.g. a GMT from MSigDB, Reactome, or an inferred GRN); without one
  the architecture cannot be defined
- The goal is maximum predictive/generative performance rather than interpretable
  module activity — the article states VEGA's limited generative capacity sacrifices
  predictive performance for interpretability
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: VEGA
  role: VAE with a sparse linear decoder whose wiring mirrors user-provided gene modules
    (GMVs from the GMT file); trained to embed cells in the interpretable latent space
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Used to wrap the single-cell dataset as an AnnData object and preprocess (normalization,
    log-transformation) before passing it to VEGA
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scVI
  role: Ecosystem in which VEGA is implemented; also a comparison model discussed
    for handling count data in place of normalized expression profiles
  repo: https://scvi-tools.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:7d47396a0efc20ac3f773de4e5480cee85f97611807f2d5b4505fbb0b711f94b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# gene-regulatory-network-informed-decoder-wiring

## Summary

Wire the sparse linear decoder of a VAE (VEGA) one-to-one to gene module variables (GMVs) parsed from a gene regulatory network — e.g. the Carro et al. GBM ARACNe transcription factor-target network supplied as a GMT file — so each latent dimension directly reports the activity of one transcription factor master regulator.

## When to use

Use when you have single-cell transcriptomics plus a prior gene regulatory network (e.g. ARACNe-derived TF-target modules) and want an interpretable latent space whose dimensions correspond to transcription factor / pathway activity rather than abstract factors.

## When NOT to use

- Input is raw count data — VEGA was demonstrated on normalized expression profiles; the article notes handling counts in place of normalized profiles (e.g. within scVI) is a discussed extension, not the wired-decoder use case
- No prior gene module or network annotation exists — VEGA's decoder requires user-provided gene modules (e.g. a GMT from MSigDB, Reactome, or an inferred GRN); without one the architecture cannot be defined
- The goal is maximum predictive/generative performance rather than interpretable module activity — the article states VEGA's limited generative capacity sacrifices predictive performance for interpretability

## Inputs

- Single-cell expression dataset preprocessed (normalized, log-transformed) and wrapped as a Scanpy AnnData object (e.g. Darmanis et al. GBM dataset, GEO GSE84465)
- GMT file specifying gene module variables (GMVs) and gene membership, e.g. ARACNe-inferred TF-target modules such as STAT3 and OLIG2 from the Carro et al. GBM network

## Outputs

- Trained VEGA model whose sparse linear decoder is wired one-to-one to the user-provided GMVs
- Interpretable latent embedding (one dimension per GMV) suitable for UMAP visualization colored by cell type
- Per-cell gene module activity table (e.g. STAT3 and OLIG2 GMV activity per neoplastic cell)

## How to apply

Parse the gene regulatory network (e.g. the Carro et al. GBM ARACNe TF-target network) into a GMT file that defines each gene module variable (GMV), such as STAT3 and OLIG2, with its target-gene membership. Preprocess the single-cell dataset (normalization, log-transformation) in scanpy before passing it to VEGA, since VEGA expects normalized expression profiles rather than raw counts. Build an AnnData object and instantiate VEGA in PyTorch within the scanpy/scvi-tools ecosystem, hard-coding the sparse linear decoder so each latent node connects one-to-one to the genes of exactly one GMV from the GMT file. Train the model so the encoder embeds cells into the interpretable latent space, then compute per-cell GMV activities and optionally run VEGA's Bayesian testing procedure to find significantly activated modules. Because decoder connections are hard-coded from prior knowledge, no re-wiring or pruning of the network is performed during training.

## Related tools

- **VEGA** (VAE with a sparse linear decoder whose wiring mirrors user-provided gene modules (GMVs from the GMT file); trained to embed cells in the interpretable latent space) — https://github.com/LucasESBS/vega
- **scanpy** (Used to wrap the single-cell dataset as an AnnData object and preprocess (normalization, log-transformation) before passing it to VEGA) — https://scanpy.readthedocs.io/en/stable/
- **scVI** (Ecosystem in which VEGA is implemented; also a comparison model discussed for handling count data in place of normalized expression profiles) — https://scvi-tools.org/

## Examples

```
pip install scvega; import vega  # then train VEGA on the scanpy AnnData with the ARACNe-derived GMT file defining STAT3/OLIG2 GMVs
```

## Evaluation signals

- The GMT file correctly parses the GRN so each GMV (e.g. STAT3, OLIG2) maps one-to-one to a latent dimension with connections only to its target genes
- Pre-annotated cell types separate in the UMAP visualization of the VEGA latent embedding (acceptance criterion from the GBM task)
- The per-cell GMV activity table reproduces the reported pattern: anticorrelated STAT3 and OLIG2 activity in neoplastic cells and OLIG2 activity activated in oligodendrocyte precursor cells (Fig. 2d-e)
- VEGA's Bayesian differential module activity testing flags the expected master regulators as significantly activated in the relevant cell populations
- Reproducibility check against the LucasESBS/vega-reproducibility pipeline: train_all_models.sh completes and the notebooks regenerate the paper figures

## Limitations

- The sparse, single-layer decoder is a clear limitation that prevents further improvement of generalizability and robustness
- VEGA's generative capacity is limited; predictive performance is sacrificed for biological interpretability of the latent space
- Hard-coded decoder connections leave no room for correcting prior knowledge about gene modules when the context requires it
- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific, as additional unannotated genes can play an important role in certain gene modules

## Evidence

- [abstract] sparse decoder wired to GMVs: "whose decoder wiring mirrors user-provided gene modules, providing direct interpretability to the latent variables"
- [methods] GMV definition via GMT: "GMT file(s) with the gene module membership, such as provided by databases like `MSigDB"
- [methods] preprocessing recommendation: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [methods] Bayesian module testing: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [discussion] single-layer decoder limitation: "The clear limitations of the current architecture resides in the sparse, single-layer decoder of the model."
- [readme] scanpy/scvi-tools ecosystem: [evidence span withheld — anchor status: external_doc]
