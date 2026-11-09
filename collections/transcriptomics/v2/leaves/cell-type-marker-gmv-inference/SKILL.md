---
name: cell-type-marker-gmv-inference
description: Use when you have an annotated single-cell dataset (e.g., the Field et
  al. week-2 cortical organoid data, GEO GSE106245) plus a cell type marker GMT file
  and want interpretable, module-level per-cell cell type activity rather than opaque
  clustering.
when_to_use_negative:
- You need a strong predictive/generative model rather than an interpretable latent
  space — VEGA's sparse single-layer decoder sacrifices predictive performance for
  interpretability
- 'Your gene modules contain many unannotated genes: hard-coded decoder connections
  cannot correct incomplete or non-context-specific prior knowledge from databases
  like MSigDB'
- Input is raw count data that the standard VEGA workflow does not handle — the discussion
  notes count-data handling would require modifying VEGA following the scVI rationale
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: VEGA
  role: Variational autoencoder with sparse linear decoder wired to the merged Reactome
    + cell type marker gene-module membership matrix; infers per-cell GMV activities
    and runs the Bayesian one-vs-rest differential activity test
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Wraps the single-cell dataset as an AnnData object and performs preprocessing
    (filtering cells/genes, normalization) before VEGA training
- name: scVI
  role: Part of the scvi-tools ecosystem used to build and train the VEGA model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:4a279436c832c19f7518f657ce949570bbc0e2c87df0c86eaebf64716a81395a
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
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cell-type-marker-gmv-inference

## Summary

Trains the VEGA variational autoencoder with cell type marker-set gene module variables (GMVs) added to its latent space so that each cell type's marker-set GMV activity segregates the annotated cell populations, enabling interpretable cell type identification in single-cell RNA-seq.

## When to use

Use when you have an annotated single-cell dataset (e.g., the Field et al. week-2 cortical organoid data, GEO GSE106245) plus a cell type marker GMT file and want interpretable, module-level per-cell cell type activity rather than opaque clustering.

## When NOT to use

- You need a strong predictive/generative model rather than an interpretable latent space — VEGA's sparse single-layer decoder sacrifices predictive performance for interpretability
- Your gene modules contain many unannotated genes: hard-coded decoder connections cannot correct incomplete or non-context-specific prior knowledge from databases like MSigDB
- Input is raw count data that the standard VEGA workflow does not handle — the discussion notes count-data handling would require modifying VEGA following the scVI rationale

## Inputs

- Single-cell RNA-seq dataset as a scanpy AnnData object (e.g., Field et al. week-2 cortical organoid dataset, GEO GSE106245)
- GMT file of cell type marker gene sets
- GMT file of Reactome pathway gene sets
- Cell type annotations for the one-vs-rest differential activity test

## Outputs

- Per-cell gene module variational (GMV) activity matrix covering pathway and cell type marker modules
- Log-Bayes-Factor significance values for one-vs-rest differential activity tests per cell type (e.g., organoid_gmv_results.csv)
- Correlation scatter of GMV activity vs marker gene expression (e.g., mki67_correlation.png)

## How to apply

Load the organoid dataset as an AnnData object and preprocess it with scanpy (filter cells/genes, normalize). Parse the Reactome pathway GMT and the cell type marker GMT and merge them into a combined gene-module membership matrix that defines the sparse linear decoder wiring of VEGA, so both pathway and cell type GMVs are modeled jointly. Train the VEGA VAE (PyTorch, scvi-tools ecosystem) to infer per-cell GMV activities, then run VEGA's one-vs-rest Bayesian differential activity test per annotated cell type, retaining significant modules at |log_e(BF)| > 3. Validate by correlating the CELL_CYCLE_MITOTIC GMV activity against MKI67 expression per cell — the paper reports R2 ≈ 0.64 — and confirm each cell type marker-set GMV activity correctly segregates its corresponding annotated cell type. Save per-cell GMV activities and log-Bayes-Factor significance values (e.g., organoid_gmv_results.csv) and the correlation scatter (mki67_correlation.png).

## Related tools

- **VEGA** (Variational autoencoder with sparse linear decoder wired to the merged Reactome + cell type marker gene-module membership matrix; infers per-cell GMV activities and runs the Bayesian one-vs-rest differential activity test) — https://github.com/LucasESBS/vega
- **scanpy** (Wraps the single-cell dataset as an AnnData object and performs preprocessing (filtering cells/genes, normalization) before VEGA training)
- **scVI** (Part of the scvi-tools ecosystem used to build and train the VEGA model)

## Examples

```
import vega; model = vega.VEGA(my_scanpy_data, gmt_file='reactome_plus_celltype_markers.gmt'); model.train(); model.test_likelihood()
``
```

## Evaluation signals

- Each cell type marker-set GMV activity correctly segregates its corresponding cell type as annotated by the original authors
- One-vs-rest Bayesian tests flag significant modules at |log_e(BF)| > 3 for each annotated cell type
- CELL_CYCLE_MITOTIC GMV activity vs MKI67 expression per cell reproduces the reported R2 ≈ 0.64
- Merged gene-module membership matrix includes both Reactome pathways and cell type marker sets as sparse decoder connections
- Output CSV contains per-cell GMV activities plus log-Bayes-Factor significance values with a matching scatter figure

## Limitations

- VEGA's sparse, single-layer decoder limits generalizability and robustness
- VEGA's limited generative capacity sacrifices predictive performance for biological interpretability of the latent space
- Hard-coded decoder connections leave no room for correcting prior gene module knowledge when the context requires it, since prior knowledge (e.g., MSigDB) can be incomplete or not context-specific

## Evidence

- [abstract] whose decoder wiring mirrors user-provided gene modules, providing direct interpretability to the latent variables: "whose decoder wiring mirrors user-provided gene modules, providing direct interpretability to the latent variables"
- [abstract] We demonstrate the performance of VEGA in diverse biological contexts using pathways, gene regulatory networks and cell type identities as the gene modules that define its latent space: "We demonstrate the performance of VEGA in diverse biological contexts using pathways, gene regulatory networks and cell type identities as the gene modules that define its latent space"
- [methods] GMT file with the gene module membership, such as provided by databases like MSigDB: "GMT file(s) with the gene module membership, such as provided by databases like `MSigDB <https://www.gsea-msigdb.org/gsea/index.jsp>`_"
- [methods] Note: We recommend to preprocess the single-cell dataset before passing it to VEGA.: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [methods] VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [readme] A single-cell dataset wrapped using the Scanpy package; A GMT file specifying the gene module variables (GMVs) and gene membership, eg. from MSigDB: "A single-cell dataset wrapped using the Scanpy package
* A GMT file specifying the gene module variables (GMVs) and gene membership, eg. from MSigDB"
