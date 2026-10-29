---
name: differential-pathway-activity-testing
description: Use when single-cell transcriptomic data has been embedded in VEGA's
  pathway-informed latent space and you need to test which gene modules differ in
  activity between defined cell groups or conditions.
when_to_use_negative:
- You need a predictive or strongly generative model rather than an interpretable
  latent representation — VEGA's sparse single-layer decoder sacrifices predictive/generative
  performance for interpretability
- Your gene sets are known to be incomplete or not context-specific and you need to
  correct prior knowledge during training — VEGA's hard-coded decoder connections
  leave no room for correcting prior module membership
- Input data lacks a GMT gene module definition, so no pathway-wired latent space
  can be constructed
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: VEGA
  role: Trains a VAE with a sparse linear decoder hard-wired to gene modules from
    a GMT file and infers per-cell gene module activities (GMVs); provides the Bayesian
    testing procedure to find significantly activated gene modules
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Wraps and preprocesses the single-cell dataset (AnnData object) before passing
    it to VEGA
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scvi-tools
  role: Ecosystem used by VEGA's implementation (PyTorch-based) for single-cell model
    infrastructure
  repo: https://scvi-tools.org/
- name: f-scLVM
  role: Alternative Bayesian latent variable model with explicit prior biological
    knowledge; lacks out-of-sample inference and is computationally costlier, motivating
    VEGA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:5e015949be77593396e416eba6f792045356cf9d4f75d589402ad33c66de4d10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - f-scLVM
  - scanpy
  - scvi-tools
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# differential-pathway-activity-testing

## Summary

Identify significantly activated gene module variables (GMVs) by comparing VEGA-inferred pathway activity distributions across cell groups (e.g., interferon-stimulated vs. control PBMCs), leveraging VEGA's interpretable latent space where each latent dimension maps to a biological pathway or gene set.

## When to use

Use when single-cell transcriptomic data has been embedded in VEGA's pathway-informed latent space and you need to test which gene modules differ in activity between defined cell groups or conditions.

## When NOT to use

- You need a predictive or strongly generative model rather than an interpretable latent representation — VEGA's sparse single-layer decoder sacrifices predictive/generative performance for interpretability
- Your gene sets are known to be incomplete or not context-specific and you need to correct prior knowledge during training — VEGA's hard-coded decoder connections leave no room for correcting prior module membership
- Input data lacks a GMT gene module definition, so no pathway-wired latent space can be constructed

## Inputs

- Preprocessed single-cell count matrix as a Scanpy AnnData object (e.g., Kang et al. PBMCs dataset with stimulated/control labels)
- GMT file specifying gene module variables (GMVs) and gene membership, e.g., Reactome pathways or MSigDB gene sets
- Trained VEGA model with decoder wiring matching the GMT gene modules
- Cell group/condition labels per cell (e.g., stimulated vs. control, innate vs. adaptive cell types)

## Outputs

- Per-cell gene module activity (GMV) table, e.g., vega_gmv_activity.csv
- List of significantly activated gene modules per group from VEGA's Bayesian testing procedure
- Pathway activity plots in the style of Fig. 1b-d (GMV distributions across conditions/cell types)

## How to apply

First train a VEGA model whose sparse linear decoder is hard-wired to gene modules from a GMT file (e.g., Reactome pathways or MSigDB gene sets) on preprocessed single-cell data such as the Kang et al. PBMC count matrix wrapped as an AnnData/Scanpy object. Run single-cell inference of gene module activities (GMVs) for every cell using the trained encoder. Then apply VEGA's Bayesian testing procedure to compare GMV distributions across groups — e.g., interferon-alpha/beta signaling activity between stimulated and control cells, or tryptophan catabolism activity between innate (dendritic cells, FCGR3A+ and CD14+ monocytes) and adaptive immune cell types. Validate by confirming the expected qualitative segregation: interferon-alpha/beta GMV separates stimulated from naive cells and tryptophan catabolism GMV separates innate from adaptive cells, matching Fig. 1b-d patterns. Export per-cell GMV values (e.g., as vega_gmv_activity.csv) for downstream visualization.

## Related tools

- **VEGA** (Trains a VAE with a sparse linear decoder hard-wired to gene modules from a GMT file and infers per-cell gene module activities (GMVs); provides the Bayesian testing procedure to find significantly activated gene modules) — https://github.com/LucasESBS/vega
- **scanpy** (Wraps and preprocesses the single-cell dataset (AnnData object) before passing it to VEGA) — https://scanpy.readthedocs.io/en/stable/
- **scvi-tools** (Ecosystem used by VEGA's implementation (PyTorch-based) for single-cell model infrastructure) — https://scvi-tools.org/
- **f-scLVM** (Alternative Bayesian latent variable model with explicit prior biological knowledge; lacks out-of-sample inference and is computationally costlier, motivating VEGA)

## Examples

```
import scanpy as sc, vega; adata = sc.read_h5ad('kang_pbmc.h5ad'); vega_model = vega.VEGA(adata, 'reactome.gmt'); vega_model.train(); gmvs = vega_model.to_dataframe(); active = vega_model.test_gmvs(group_labels=adata.obs['condition'])
```

## Evaluation signals

- Interferon-alpha/beta signaling GMV activity visibly segregates stimulated from control/naive cells in the Kang et al. PBMC dataset
- Tryptophan catabolism GMV activity separates innate immune cells (dendritic cells, FCGR3A+ and CD14+ monocytes) from adaptive immune cells
- GMV distributions match the qualitative pattern reported in Fig. 1b-d of the VEGA paper
- Per-cell GMV table exports with one row per cell and one column per gene module from the input GMT (e.g., vega_gmv_activity.csv)
- Bayesian test output identifies the expected modules (interferon-alpha/beta signaling, tryptophan catabolism) as significantly activated in the appropriate groups

## Limitations

- VEGA's sparse, single-layer decoder limits generalizability and robustness, and its generative capacity is limited — predictive performance is sacrificed for biological interpretability
- Hard-coded decoder connections cannot correct prior knowledge about gene modules when the biological context requires it
- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific, since unannotated genes may play important roles in certain gene modules

## Evidence

- [methods] VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [methods] Preprocessing recommendation: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [methods] GMT input format: "GMT file(s) with the gene module membership, such as provided by databases like `MSigDB <https://www.gsea-msigdb.org/gsea/index.jsp>`_"
- [readme] README on VEGA inputs: [evidence span withheld — anchor status: external_doc]
- [intro] Scope of VEGA: "the scope of VEGA is twofold, (1) encoding data over an interpretable latent space and (2) inferring gene module activities for out-of-sample data"
- [discussion] Limitation of decoder architecture: "The clear limitations of the current architecture resides in the sparse, single-layer decoder of the model."
