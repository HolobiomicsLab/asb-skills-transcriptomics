---
name: variational-autoencoder-pytorch-implementation
description: 'Use when reconstructing or extending VEGA: you have a Scanpy-wrapped
  single-cell dataset plus a GMT gene-module file and need a PyTorch VAE whose decoder
  mirrors user-provided gene modules.'
when_to_use_negative:
- The analysis goal is maximum generative/predictive fidelity rather than interpretable
  module activity — the sparse, single-layer decoder sacrifices predictive performance
  for biological interpretability
- Gene-module prior knowledge needs correction or adaptation to the context at hand
  — the decoder's hard-coded connections leave no room for revising prior knowledge
  about gene modules
- Input is raw count data — VEGA operates on normalized expression profiles (count-model
  variants like scVI-style handling were only discussed as future work)
tools:
- name: VEGA
  role: The masked sparse-decoder VAE architecture being implemented; installed via
    `pip install scvega`
  repo: https://github.com/LucasESBS/vega
- name: PyTorch
  role: Framework for implementing the encoder (mu/Sigma, latent dropout) and the
    single-layer masked sparse linear decoder with nonnegativity clamping
- name: scanpy
  role: Wraps the single-cell dataset (AnnData) that defines the gene order the binary
    mask must match
  repo: https://github.com/LucasESBS/vega
- name: scvi-tools
  role: Ecosystem component used alongside scanpy in the VEGA implementation
  repo: https://github.com/LucasESBS/vega
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:f121be3d81d9053fec46d98235215608112db94509c16f61b48ec26cc560cf28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - PyTorch
  - VEGA
  - scanpy
  - scvi-tools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# variational-autoencoder-pytorch-implementation

## Summary

Implements VEGA, a sparse Variational Autoencoder in PyTorch whose single-layer decoder wiring is hard-coded by a binary gene-membership mask so each latent gene-module variable (GMV) connects only to its annotated member genes, yielding an interpretable latent space for single-cell RNA-seq.

## When to use

Use when reconstructing or extending VEGA: you have a Scanpy-wrapped single-cell dataset plus a GMT gene-module file and need a PyTorch VAE whose decoder mirrors user-provided gene modules.

## When NOT to use

- The analysis goal is maximum generative/predictive fidelity rather than interpretable module activity — the sparse, single-layer decoder sacrifices predictive performance for biological interpretability
- Gene-module prior knowledge needs correction or adaptation to the context at hand — the decoder's hard-coded connections leave no room for revising prior knowledge about gene modules
- Input is raw count data — VEGA operates on normalized expression profiles (count-model variants like scVI-style handling were only discussed as future work)

## Inputs

- Single-cell expression dataset wrapped as a Scanpy AnnData object (preprocessed per VEGA recommendations)
- GMT file(s) specifying gene module variables (GMVs) and gene membership, e.g. from MSigDB, Gene Ontology, PANTHER, MolSigDB, or Reactome

## Outputs

- vega.py module implementing the masked encoder/decoder in PyTorch
- vega_unit_test_report.md with pass/fail results of mask-sparsity and weight-constraint checks
- Trained VEGA model exposing per-cell gene module activities in the interpretable latent space

## How to apply

Parse the GMT gene-module membership files to assemble a binary mask M linking each latent GMV to its member genes, ordering mask columns to match the AnnData expression matrix gene order. Implement the PyTorch encoder producing mu and Sigma with latent dropout in the probabilistic latent layer, then build the single-layer masked sparse linear decoder with nonnegative weights, appending fully connected hidden nodes for unannotated genes and an optional batch-key covariate input path. Clamp decoder weights to remain nonnegative after initialization and throughout optimization so module-to-gene links stay interpretable. Write unit tests asserting that each output gene receives input only from its annotated module or an FC node (mask sparsity) and that all decoder weights remain nonnegative. After training, GMV activities can be read directly per cell and tested for differential module activity with VEGA's Bayesian testing procedure.

## Related tools

- **VEGA** (The masked sparse-decoder VAE architecture being implemented; installed via `pip install scvega`) — https://github.com/LucasESBS/vega
- **PyTorch** (Framework for implementing the encoder (mu/Sigma, latent dropout) and the single-layer masked sparse linear decoder with nonnegativity clamping)
- **scanpy** (Wraps the single-cell dataset (AnnData) that defines the gene order the binary mask must match) — https://github.com/LucasESBS/vega
- **scvi-tools** (Ecosystem component used alongside scanpy in the VEGA implementation) — https://github.com/LucasESBS/vega

## Examples

```
pip install -r requirements.txt && bash train_all_models.sh
```

## Evaluation signals

- Mask-sparsity unit test: each decoder output gene receives input only from its annotated gene module or an appended fully connected (FC) hidden node
- Nonnegativity check: all decoder weights remain nonnegative after initialization and after parameter clamping during training
- Mask column order matches the gene order of the AnnData expression matrix used for training
- Trained GMV activations reproduce expected biological signals, e.g. retained biological signal in the Kang et al. PBMCs dataset UMAP embedding
- Reproducibility: training via the LucasESBS/vega-reproducibility pipeline (revision1 branch) regenerates paper figures from ./trained_models/

## Limitations

- The sparse, single-layer decoder architecture limits generalizability and robustness, and VEGA's generative capacity is limited
- VEGA's limited generative capacity sacrifices predictive performance for biological interpretability of the latent space
- Hard-coded decoder connections cannot correct prior knowledge about gene modules when context requires it; MSigDB-style priors can be incomplete or not context-specific

## Evidence

- [abstract] VEGA decoder wiring mirrors gene modules: "we introduce a novel sparse Variational Autoencoder architecture, VEGA (VAE Enhanced by Gene Annotations), whose decoder wiring mirrors user-provided gene modules"
- [methods] GMT input for gene module membership: "GMT file(s) with the gene module membership, such as provided by databases like `MSigDB <https://www.gsea-msigdb.org/gsea/index.jsp>`_"
- [readme] VEGA purpose and implementation stack: [evidence span withheld — anchor status: external_doc]
- [discussion] Single-layer decoder limitation: "The clear limitations of the current architecture resides in the sparse, single-layer decoder of the model."
