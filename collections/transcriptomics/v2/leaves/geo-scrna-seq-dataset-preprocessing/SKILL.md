---
name: geo-scrna-seq-dataset-preprocessing
description: Use when a public GEO single-cell dataset (e.g., GSE84465) must be acquired,
  normalized, and log-transformed into a scanpy AnnData object to serve as input for
  an interpretable VAE such as VEGA.
when_to_use_negative:
- Input is already a normalized feature table or processed embedding, not raw GEO
  single-cell expression data
- No prior biological knowledge (GMT gene modules) is available to define the VEGA
  latent space — the decoder requires user-provided gene modules
- Data are raw counts that must stay in count form for a scVI-style negative-binomial
  model; VEGA as published expects normalized expression profiles
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3307
tools:
- name: VEGA
  role: Target model that consumes the preprocessed AnnData and GMT file; its decoder
    is wired one-to-one to the GMT gene module variables
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Wraps the single-cell dataset (AnnData) and performs normalization and log-transformation
    before the model is trained
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scVI
  role: Ecosystem/alternative framework mentioned for handling count data in place
    of normalized expression profiles
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:92eed72dabc2f42034649c7d54b6b8061b51f4fe10a2fe066a370c36ad5ea72c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# geo-scrna-seq-dataset-preprocessing

## Summary

Download and preprocess a GEO-deposited single-cell RNA-seq dataset (e.g., the Darmanis et al. GBM dataset GSE84465 with pre-annotated cell types) in scanpy so it can be passed to the VEGA model for network-guided latent inference.

## When to use

Use when a public GEO single-cell dataset (e.g., GSE84465) must be acquired, normalized, and log-transformed into a scanpy AnnData object to serve as input for an interpretable VAE such as VEGA.

## When NOT to use

- Input is already a normalized feature table or processed embedding, not raw GEO single-cell expression data
- No prior biological knowledge (GMT gene modules) is available to define the VEGA latent space — the decoder requires user-provided gene modules
- Data are raw counts that must stay in count form for a scVI-style negative-binomial model; VEGA as published expects normalized expression profiles

## Inputs

- GEO-deposited single-cell RNA-seq dataset accession (e.g., GSE84465, Darmanis et al. GBM) with pre-annotated cell types
- GMT file specifying gene module variables (GMVs) and gene membership, e.g. from MSigDB or a parsed ARACNe transcription factor-target network
- Raw count or expression matrix convertible to scanpy AnnData (.h5ad)

## Outputs

- Normalized, log-transformed scanpy AnnData object with cell type annotations
- Preprocessed single-cell matrix ready for VEGA training
- Downstream: interpretable latent embedding (e.g. UMAP of GMV space) and per-cell gene module activity table

## How to apply

Retrieve the raw preprocessed archive from GEO (e.g., GSE84465 for the Darmanis et al. GBM dataset), keeping the pre-annotated cell type labels since they are needed later for embedding validation. Load the data in scanpy and perform normalization and log-transformation, because VEGA operates on normalized expression profiles rather than raw counts (count-data handling is only proposed as a future scVI-style extension in the paper's discussion). Wrap the matrix in an AnnData object carrying cell metadata, and prepare a GMT file specifying gene module variables (GMVs) with their gene membership, e.g. parsed from the Carro et al. GBM ARACNe transcription factor-target network or from MSigDB. The VEGA pipeline expects exactly these two inputs: a single-cell dataset wrapped in scanpy/AnnData and a GMT file defining the decoder wiring. Only then instantiate and train the VEGA model, embedding cells in a latent space where each dimension corresponds to one GMV.

## Related tools

- **VEGA** (Target model that consumes the preprocessed AnnData and GMT file; its decoder is wired one-to-one to the GMT gene module variables) — https://github.com/LucasESBS/vega
- **scanpy** (Wraps the single-cell dataset (AnnData) and performs normalization and log-transformation before the model is trained) — https://scanpy.readthedocs.io/en/stable/
- **scVI** (Ecosystem/alternative framework mentioned for handling count data in place of normalized expression profiles)

## Examples

```
pip install scvega; import vega
```

## Evaluation signals

- AnnData object contains log-normalized expression values (not raw counts), matching VEGA's expectation of normalized expression profiles
- Cell type annotations from the GEO deposit (e.g., Darmanis et al. pre-annotated types) survive preprocessing and are available for coloring the UMAP
- Gene identifiers in the GMT modules (e.g., STAT3, OLIG2 modules from the ARACNe network) intersect with genes present in the preprocessed dataset
- After training, pre-annotated cell types separate in the latent embedding and GMV activity shows the expected pattern (anticorrelated STAT3/OLIG2 in neoplastic cells; OLIG2 active in OPCs)
- Reproducibility pipeline checks out against the LucasESBS/vega-reproducibility repository (train_all_models.sh then notebooks regenerate figures)

## Limitations

- Prior biological knowledge (e.g., MSigDB modules) can be incomplete or not context-specific; hard-coded decoder connections leave no room for correcting prior knowledge
- VEGA's sparse, single-layer decoder limits generative capacity and sacrifices predictive performance for interpretability
- As published, VEGA handles normalized expression profiles; direct count-data handling requires scVI-style extensions

## Evidence

- [methods] Preprocessing recommendation: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [readme] README required inputs: [evidence span withheld — anchor status: external_doc]
- [readme] README GMT requirement: [evidence span withheld — anchor status: external_doc]
- [discussion] Normalized expression vs counts: "such that it could handle count data in place of normalized expression profiles (SFig. 8)."
- [intro] Gene annotation sources for modules: "guided by gene module membership as recorded in gene annotation databases (e.g., Gene Ontology, PANTHER, MolSigDB, or Reactome)"
