---
name: spliced-unspliced-adata-preparation
description: Use when an AnnData object must be structured with spliced_key='spliced'
  and unspliced_key='unspliced' layers (plus optional batch_key and celltype_key)
  before running LatentVelo RNA velocity estimation or benchmarking against scVelo.
when_to_use_negative:
- The input lacks unspliced (or spliced) count layers — LatentVelo requires 'This
  object must have two layers containing spliced and unspliced counts', so fall back
  to methods that operate on spliced counts alone
- The goal is a pure scVelo-only pipeline with no LatentVelo model, since this recipe
  is LatentVelo-specific (ltv.utils.*) preparation
- Using anvi_clean_recipe when no celltype_key is available — the celltype-annotated
  preparation 'must include a celltype key'; use standard_clean_recipe instead
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LatentVelo
  role: Provides ltv.utils.standard_clean_recipe and ltv.utils.anvi_clean_recipe used
    to prepare AnnData with spliced/unspliced layers for model training
  repo: https://github.com/Spencerfar/LatentVelo
- name: anndata
  role: Provides the AnnData object format (sc.read of .h5ad) with spliced/unspliced
    layers that the recipes clean and standardize
- name: scanpy
  role: Used (as sc) to read the .h5ad AnnData files that are subsequently cleaned
    with the LatentVelo recipes
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:98ec6762fa91aab1a79d0333fe048e27e21ef7e43f3d9e5f571a55ca94e68671
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Spencerfar/LatentVelo
  repo_url: https://github.com/Spencerfar/LatentVelo
  tools:
  - GSEApy
  - LatentVelo
  - anndata
  - numpy
  - pandas
  - scanpy
  - scib
  - scvi-tools
  - sklearn
  - torchdiffeq
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# spliced-unspliced-adata-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare AnnData (.h5ad) objects with spliced and unspliced count layers for LatentVelo using ltv.utils.standard_clean_recipe or, for cell-type-annotated models, ltv.utils.anvi_clean_recipe. This cleaning/structuring step is the required entry point before initializing and training any LatentVelo VAE or AnnotVAE model.

## When to use

Use when an AnnData object must be structured with spliced_key='spliced' and unspliced_key='unspliced' layers (plus optional batch_key and celltype_key) before running LatentVelo RNA velocity estimation or benchmarking against scVelo.

## When NOT to use

- The input lacks unspliced (or spliced) count layers — LatentVelo requires 'This object must have two layers containing spliced and unspliced counts', so fall back to methods that operate on spliced counts alone
- The goal is a pure scVelo-only pipeline with no LatentVelo model, since this recipe is LatentVelo-specific (ltv.utils.*) preparation
- Using anvi_clean_recipe when no celltype_key is available — the celltype-annotated preparation 'must include a celltype key'; use standard_clean_recipe instead

## Inputs

- AnnData .h5ad object with 'spliced' and 'unspliced' count layers (e.g., scEU_organoid.h5ad, example_bifurcation.h5ad, 10 real developmental datasets)
- Optional obs columns referenced by batch_key (sequencing batch) and celltype_key (cell-type annotations) for the AnnotVAE route

## Outputs

- Cleaned AnnData with standardized spliced/unspliced layers and celltype/batch fields, ready for ltv.models.VAE or ltv.models.AnnotVAE initialization and ltv.train
- Number of retained genes (observed) usable as the VAE observed-dimension parameter

## How to apply

Load each h5ad AnnData (e.g., scEU_organoid.h5ad, dyngen synthetic bifurcation, mouse gastrulation subset) and call ltv.utils.standard_clean_recipe(adata, spliced_key='spliced', unspliced_key='unspliced', batch_key=..., celltype_key=...); batch and celltype keys are optional for the standard model. For the cell-type-annotated AnnotVAE model, instead use ltv.utils.anvi_clean_recipe, which must include a celltype_key. The recipe validates that the two required layers exist, applies gene filtering/cleaning consistent with the LatentVelo pipeline, and leaves the AnnData ready for model initialization with ltv.models.VAE(observed=number_of_genes, latent_dim=..., zr_dim=..., h_dim=...) — batch_correction=True and batches=number_of_batches when integration across sequencing batches is needed. Choose standard_clean_recipe for the standard VAE and anvi_clean_recipe whenever downstream training uses AnnotVAE with cell-type labels (e.g., mouse gastrulation with batch_key='batch', celltype_key='celltype').

## Related tools

- **LatentVelo** (Provides ltv.utils.standard_clean_recipe and ltv.utils.anvi_clean_recipe used to prepare AnnData with spliced/unspliced layers for model training) — https://github.com/Spencerfar/LatentVelo
- **anndata** (Provides the AnnData object format (sc.read of .h5ad) with spliced/unspliced layers that the recipes clean and standardize)
- **scanpy** (Used (as sc) to read the .h5ad AnnData files that are subsequently cleaned with the LatentVelo recipes)

## Examples

```
import scvelo as scv, latentvelo as ltv; adata = scv.read('scEU_organoid.h5ad'); ltv.utils.standard_clean_recipe(adata, spliced_key='spliced', unspliced_key='unspliced', batch_key='batch', celltype_key='celltype')
```

## Evaluation signals

- The cleaned AnnData retains 'spliced' and 'unspliced' layers and the specified batch_key/celltype_key columns in obs
- Downstream ltv.models.VAE(observed=number_of_genes, ...) initializes without dimension errors, confirming gene count matches the cleaned matrix
- ltv.train outputs validation autoencoder (val_ae) and trajectory reconstruction (val_traj) losses, indicating the recipe produced a trainable object
- For the AnnotVAE route, anvi_clean_recipe was called with a celltype_key present; for standard VAE route, batch/celltype keys were optional and correctly named
- Downstream ltv.output_results(model, adata, gene_velocity=True, embedding='umap') returns latent_adata on which scv.tl.velocity_graph runs with vkey='spliced_velocity'

## Limitations

- LatentVelo assumes 'a single initial latent state from which the trajectory of all cells start from and that each observed cell can be reached by a continuous trajectory', which constrains what the prepared data can support
- Some datasets 'require additional hyperparameter adjustment by increasing the dimension of the latent states, using the cell-type annotated model, or restricting to only a subset of genes' after preparation
- Preparation alone cannot fix weak unspliced signals or disconnected clusters (e.g., microglia in dentate gyrus receiving lowest latent times)

## Evidence
