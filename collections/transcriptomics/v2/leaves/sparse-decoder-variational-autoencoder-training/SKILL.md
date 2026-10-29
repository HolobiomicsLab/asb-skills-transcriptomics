---
name: sparse-decoder-variational-autoencoder-training
description: Use when you need an interpretable latent space for scRNA-seq where each
  latent dimension corresponds to a pathway, TF regulon, or cell-type gene set, and
  you must infer gene module activities per cell (in-sample and out-of-sample).
when_to_use_negative:
- You need a model that can correct or refine incomplete or non-context-specific prior
  knowledge (e.g., MSigDB gene sets) at training time — VEGA's hard-coded decoder
  connections leave no room for this
- You need raw count modeling or strong generative/predictive performance — VEGA's
  sparse single-layer decoder limits generative capacity and sacrifices predictive
  performance for interpretability
- Input is not single-cell expression data with available gene module annotations
  (a GMT file is required to wire the decoder)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VEGA
  role: The VAE with sparse linear decoder informed by biological networks; trained
    on preprocessed AnnData with GMT-defined gene modules to infer GMV activities
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Wraps the single-cell dataset as AnnData and performs recommended preprocessing/filtering/normalization
    before VEGA training
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scvi-tools
  role: Ecosystem in which VEGA is implemented alongside scanpy for model training
    and inference
  repo: https://scvi-tools.org/
- name: vega-reproducibility
  role: Repository with preprocessed data, training scripts (train_all_models.sh),
    and notebooks to reproduce the VEGA paper figures
  repo: https://github.com/LucasESBS/vega-reproducibility
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:d7ef6f5ce9d6a14878abbb3f83a9e27732903423e327a24c94e7ace86b28f663
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - f-scLVM
  - scanpy
  - scvi-tools
  - vega-reproducibility
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# sparse-decoder-variational-autoencoder-training

## Summary

Train VEGA, a variational autoencoder whose decoder is a sparse linear layer hard-wired to user-provided gene modules (e.g., Reactome pathways from a GMT file), so each latent dimension directly encodes a biological gene module activity (GMV) on single-cell RNA-seq data.

## When to use

Use when you need an interpretable latent space for scRNA-seq where each latent dimension corresponds to a pathway, TF regulon, or cell-type gene set, and you must infer gene module activities per cell (in-sample and out-of-sample).

## When NOT to use

- You need a model that can correct or refine incomplete or non-context-specific prior knowledge (e.g., MSigDB gene sets) at training time — VEGA's hard-coded decoder connections leave no room for this
- You need raw count modeling or strong generative/predictive performance — VEGA's sparse single-layer decoder limits generative capacity and sacrifices predictive performance for interpretability
- Input is not single-cell expression data with available gene module annotations (a GMT file is required to wire the decoder)

## Inputs

- Single-cell RNA-seq count matrix wrapped as a Scanpy AnnData object (e.g., Kang et al. PBMC dataset)
- GMT file(s) with gene module membership, e.g., from MSigDB, Reactome, Gene Ontology, PANTHER, or MolSigDB

## Outputs

- Trained VEGA model with gene-module-wired sparse linear decoder
- Per-cell gene module variable (GMV) activity table for interpretable latent dimensions
- Significantly activated gene modules via Bayesian differential pathway activity testing
- Latent-space embeddings/UMAP plots retaining biological signal (e.g., Fig. 1b-d style pathway activity plots)

## How to apply

Wrap the single-cell count data in a Scanpy AnnData object and apply recommended preprocessing (filtering and normalization) before passing it to VEGA, since VEGA works on normalized expression profiles rather than raw counts. Parse a GMT file specifying the gene module variables (GMVs) and gene membership to build the gene-to-module membership matrix that fixes the sparse wiring of the single-layer linear decoder. Instantiate and train the VEGA model (PyTorch implementation built on the scanpy and scvi-tools ecosystem), then use the trained encoder to infer gene module activities at the single-cell level for every cell. Optionally run VEGA's Bayesian testing procedure to identify significantly activated gene modules, and compare GMV distributions across conditions or cell types (e.g., interferon-alpha/beta signaling between stimulated vs. control PBMCs, tryptophan catabolism between innate and adaptive immune cells).

## Related tools

- **VEGA** (The VAE with sparse linear decoder informed by biological networks; trained on preprocessed AnnData with GMT-defined gene modules to infer GMV activities) — https://github.com/LucasESBS/vega
- **scanpy** (Wraps the single-cell dataset as AnnData and performs recommended preprocessing/filtering/normalization before VEGA training) — https://scanpy.readthedocs.io/en/stable/
- **scvi-tools** (Ecosystem in which VEGA is implemented alongside scanpy for model training and inference) — https://scvi-tools.org/
- **vega-reproducibility** (Repository with preprocessed data, training scripts (train_all_models.sh), and notebooks to reproduce the VEGA paper figures) — https://github.com/LucasESBS/vega-reproducibility

## Examples

```
pip install scvega  # then, to reproduce paper results: git clone https://github.com/LucasESBS/vega-reproducibility && bash train_all_models.sh
```

## Evaluation signals

- Interferon-alpha/beta signaling GMV activity visibly segregates stimulated from naive/control cells in the Kang et al. PBMC dataset
- Tryptophan catabolism GMV activity separates innate immune cells (dendritic cells, FCGR3A+ and CD14+ monocytes) from adaptive immune cells
- UMAP embedding of the VEGA latent space retains the biological signal of the Kang et al. PBMCs dataset
- Bayesian differential pathway activity testing returns a coherent set of significantly activated gene modules
- Reproduction matches the qualitative pattern of Fig. 1b-d and per-cell GMVs can be exported (e.g., vega_gmv_activity.csv)

## Limitations

- The sparse, single-layer decoder architecture limits generalizability and robustness and constrains the model's generative capacity
- VEGA sacrifices predictive performance for biological interpretability of the latent space
- Hard-coded decoder connections cannot correct prior knowledge about gene modules when the context requires it
- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific; unannotated genes may matter in some modules

## Evidence

- [abstract] abstract architecture claim: "we introduce a novel sparse Variational Autoencoder architecture, VEGA (VAE Enhanced by Gene Annotations), whose decoder wiring mirrors user-provided gene modules"
- [intro] intro model definition: "Here we propose VEGA (VAE enhanced by gene annotations), a VAE with a sparse linear decoder informed by biological networks"
- [methods] preprocessing recommendation: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [methods] Bayesian module testing: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [readme] GMT input requirement: [evidence span withheld — anchor status: external_doc]
- [methods] count data handling: "such that it could handle count data in place of normalized expression profiles (SFig. 8)."
