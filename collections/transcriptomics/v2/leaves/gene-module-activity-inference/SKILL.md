---
name: gene-module-activity-inference
description: Use when you have a single-cell count matrix (AnnData/scanpy) plus a
  GMT gene-module file and need per-cell, interpretable activity scores for pathways
  or marker sets, including out-of-sample inference on unseen cells or studies.
when_to_use_negative:
- You need an uninterpretable latent space optimized for reconstruction accuracy —
  VEGA's sparse single-layer decoder sacrifices predictive/generative performance
  for biological interpretability
- The prior biological knowledge (e.g., MSigDB modules) is known to be incomplete
  or not context-specific and needs correcting during training — the hard-coded decoder
  connections leave no room for this
- Input is bulk RNA-seq or already-aggregated module scores rather than single-cell
  count data
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: VEGA
  role: 'Core model: sparse VAE whose decoder wiring mirrors user-provided gene modules
    (vega.VEGA / vega.VegaSCVI), providing interpretable GMV activities and Bayesian
    differential module testing'
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Loading, filtering, and normalization of the single-cell count matrices and
    AnnData wrapping before passing data to VEGA
- name: scvi-tools
  role: Backbone ecosystem for VEGA (VegaSCVI) and the scvi-tools-inspired Bayesian
    differential GMV activity testing procedure
- name: PyTorch
  role: Deep learning framework in which VEGA's variational autoencoder is implemented
    and trained
provenance:
  source_task_ids:
  - task_006
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:f136e51210ff07716648f65a8bb01b01e3389475fdae2d433410059cee5c4ac0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - PyTorch
  - VEGA
  - gseapy
  - scanpy
  - scvi-tools
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gene-module-activity-inference

## Summary

Infer per-cell gene module variable (GMV) activities using VEGA, a variational autoencoder whose sparse linear decoder is hard-wired to user-provided gene modules from annotation files (e.g., MSigDB GMT), yielding an interpretable latent space for pathways, transcription factors, or cell type markers in single-cell RNA-seq data.

## When to use

Use when you have a single-cell count matrix (AnnData/scanpy) plus a GMT gene-module file and need per-cell, interpretable activity scores for pathways or marker sets, including out-of-sample inference on unseen cells or studies.

## When NOT to use

- You need an uninterpretable latent space optimized for reconstruction accuracy — VEGA's sparse single-layer decoder sacrifices predictive/generative performance for biological interpretability
- The prior biological knowledge (e.g., MSigDB modules) is known to be incomplete or not context-specific and needs correcting during training — the hard-coded decoder connections leave no room for this
- Input is bulk RNA-seq or already-aggregated module scores rather than single-cell count data

## Inputs

- Single-cell count matrix wrapped as a scanpy AnnData object (e.g., Kang et al. PBMCs GSE96583, Zheng et al. PBMCs, Field et al. cortical organoids GSE106245)
- GMT file specifying gene module variables (GMVs) and gene membership, e.g., MSigDB pathways, Reactome pathways, GRNs, or cell type marker sets

## Outputs

- Per-cell gene module activity scores (GMV latent embeddings)
- Differential GMV results table with gene module names and log_e(Bayes Factor) values
- Trained VEGA model usable for out-of-sample GMV inference

## How to apply

Preprocess the single-cell dataset (filter cells/genes, normalize) with scanpy as recommended before passing it to VEGA, wrapping the data in an AnnData object. Parse the GMT file (e.g., from MSigDB, Reactome, Gene Ontology, or a cell-type marker GMT) to build the mask that wires the VEGA sparse linear decoder (vega.VEGA / vega.VegaSCVI on the scvi-tools/PyTorch backbone), so each latent unit corresponds to one biological module. Train the VAE on the dataset, then infer GMV activities at single-cell level; VEGA's key advantage is that these activities can also be inferred for out-of-sample data (held-out cell type/condition pairs or an entirely unseen study). For differential activity, use VEGA's scvi-tools-inspired Bayesian testing procedure and call modules significant at |log_e(BF)| > 3 (e.g., interferon signaling and tryptophan catabolism in stimulated vs control innate immune cells of Kang et al. PBMCs).

## Related tools

- **VEGA** (Core model: sparse VAE whose decoder wiring mirrors user-provided gene modules (vega.VEGA / vega.VegaSCVI), providing interpretable GMV activities and Bayesian differential module testing) — https://github.com/LucasESBS/vega
- **scanpy** (Loading, filtering, and normalization of the single-cell count matrices and AnnData wrapping before passing data to VEGA)
- **scvi-tools** (Backbone ecosystem for VEGA (VegaSCVI) and the scvi-tools-inspired Bayesian differential GMV activity testing procedure)
- **PyTorch** (Deep learning framework in which VEGA's variational autoencoder is implemented and trained)

## Examples

```
pip install scvega; import vega; model = vega.VEGA(adata, gmt_path='msigdb.gmt'); model.train(); gmv = model.to_dataframe()
```

## Evaluation signals

- Differential GMV calls at |log_e(BF)| > 3 reproduce reported findings, e.g., interferon signaling and tryptophan catabolism activated in stimulated vs control innate immune cells
- Out-of-sample cross-study inference: ~67% average overlap of top-50 differential GMVs between study A (Kang et al.) and study B (Zheng et al.), with only marginal R2 decrease
- Biological generalization: ~72% average top-50 GMV overlap for held-out cell type/condition pairs with marginal R2 decrease
- Marker-set GMV activities correctly segregate annotated cell types (e.g., CELL_CYCLE_MITOTIC GMV correlates with MKI67 expression, R2 ≈ 0.64)
- Results are reproducible via the LucasESBS/vega-reproducibility repository (preprocessed data archive, train_all_models.sh, figure notebooks)

## Limitations

- The sparse, single-layer decoder architecture prevents further improvement of generalizability and robustness
- The generative capacity of VEGA is limited; predictive performance is sacrificed for biological interpretability of the latent space
- Hard-coded decoder connections leave no room for correcting prior knowledge about gene modules when context requires it
- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific, as unannotated genes can play important roles in certain gene modules

## Evidence

- [abstract] we introduce a novel sparse Variational Autoencoder architecture, VEGA (VAE Enhanced by Gene Annotations), whose decoder wiring mirrors user-provided gene modules: "we introduce a novel sparse Variational Autoencoder architecture, VEGA (VAE Enhanced by Gene Annotations), whose decoder wiring mirrors user-provided gene modules"
- [methods] GMT file with gene module membership: "GMT file(s) with the gene module membership, such as provided by databases like `MSigDB <https://www.gsea-msigdb.org/gsea/index.jsp>`_"
- [methods] Preprocessing recommendation: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [methods] Bayesian testing procedure: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [intro] Out-of-sample scope: "the scope of VEGA is twofold, (1) encoding data over an interpretable latent space and (2) inferring gene module activities for out-of-sample data"
- [readme] README installation/usage: [evidence span withheld — anchor status: not_found]
