---
name: velocity-directional-coherence-scoring-cbdir-iccoh
description: Use when benchmarking RNA velocity models on multi-batch data and you
  need to quantify whether inferred velocities remain directionally coherent after
  batch correction, not just whether cell states are integrated.
when_to_use_negative:
- Single-batch datasets where batch-correction/integration metrics (kBET, iLISI, cLISI)
  are undefined or trivial and velocity metrics alone suffice
- Datasets lacking spliced/unspliced layers (e.g. only a raw count matrix), since
  RNA velocity scoring requires spliced_key='spliced' and unspliced_key='unspliced'
- Questions about trajectory shape or latent time accuracy rather than velocity directional
  coherence under batch correction
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: LatentVelo
  role: Velocity model with dynamics-based latent-space batch correction (batch_correction=True)
    whose velocities are scored; provides ltv.utils.standard_clean_recipe, ltv.train,
    ltv.output_results
  repo: github.com/Spencerfar/LatentVelo
- name: UniTVelo
  role: Baseline velocity model run on unintegrated, ComBat-corrected, and scGen-corrected
    data for comparison
- name: scGen
  role: Gene-space batch correction of counts applied before rerunning UniTVelo as
    a baseline
- name: scib
  role: Computes cell-state batch-integration metrics kBET, iLISI, and cLISI for all
    methods
- name: scVelo
  role: Builds velocity graphs (scv.tl.velocity_graph with vkey='spliced_velocity')
    used for nearest-neighbor velocity cosine similarity
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:38df8e6ed8130bcea61848952a8b94b0bd586a5e8bc7779039551aa9afedce68
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Spencerfar/LatentVelo
  tools:
  - LatentVelo
  - UniTVelo
  - scGen
  - scVelo
  - scib
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# Velocity directional coherence scoring (CBDir, ICCoh, NN velocity cosine)

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores RNA velocity quality on batch-corrected cell states using directional-coherence metrics (CBDir, ICCoh, and nearest-neighbor velocity cosine similarity from scVelo velocity graphs), alongside batch-integration metrics (kBET, iLISI, cLISI from scib), to test whether a model such as LatentVelo batch-corrects velocities better than gene-space corrections like ComBat or scGen applied to UniTVelo.

## When to use

Use when benchmarking RNA velocity models on multi-batch data and you need to quantify whether inferred velocities remain directionally coherent after batch correction, not just whether cell states are integrated.

## When NOT to use

- Single-batch datasets where batch-correction/integration metrics (kBET, iLISI, cLISI) are undefined or trivial and velocity metrics alone suffice
- Datasets lacking spliced/unspliced layers (e.g. only a raw count matrix), since RNA velocity scoring requires spliced_key='spliced' and unspliced_key='unspliced'
- Questions about trajectory shape or latent time accuracy rather than velocity directional coherence under batch correction

## Inputs

- Two-batch dyngen bifurcation synthetic dataset (example_bifurcation.h5ad) with 'spliced' and 'unspliced' layers and a 'batch' key
- Velocity outputs from LatentVelo (spliced_velocity on latent space) and from UniTVelo run on unintegrated, ComBat-corrected, and scGen-corrected count matrices

## Outputs

- Per-method table of batch-correction metrics (kBET, iLISI, cLISI) and velocity metrics (CBDir, ICCoh, nearest-neighbor velocity cosine similarity), e.g. dyngen_bifurcation_batch_metrics.csv
- scVelo velocity graphs (scv.tl.velocity_graph with vkey='spliced_velocity') used to compute NN velocity cosine similarity

## How to apply

Run each velocity method (LatentVelo with batch_correction=True, UniTVelo on unintegrated data, and UniTVelo on ComBat- or scGen gene-space-corrected counts) on the two-batch dyngen bifurcation dataset (example_bifurcation.h5ad). For LatentVelo, prepare data with ltv.utils.standard_clean_recipe (spliced_key='spliced', unspliced_key='unspliced', batch_key='batch'), train with ltv.train, export results with ltv.output_results, then build the velocity graph with scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity'). Compute cell-state integration metrics with scib (kBET, iLISI, cLISI) and velocity metrics CBDir, ICCoh, and nearest-neighbor velocity cosine similarity for all methods. Compile everything into a single table (e.g. dyngen_bifurcation_batch_metrics.csv) and check that LatentVelo ranks best on every metric, reflecting that the dynamics-based embedding corrects both cell states and velocities while gene-space corrections (ComBat, scGen) fail to properly integrate batches for velocity inference.

## Related tools

- **LatentVelo** (Velocity model with dynamics-based latent-space batch correction (batch_correction=True) whose velocities are scored; provides ltv.utils.standard_clean_recipe, ltv.train, ltv.output_results) — github.com/Spencerfar/LatentVelo
- **UniTVelo** (Baseline velocity model run on unintegrated, ComBat-corrected, and scGen-corrected data for comparison)
- **scGen** (Gene-space batch correction of counts applied before rerunning UniTVelo as a baseline)
- **scib** (Computes cell-state batch-integration metrics kBET, iLISI, and cLISI for all methods)
- **scVelo** (Builds velocity graphs (scv.tl.velocity_graph with vkey='spliced_velocity') used for nearest-neighbor velocity cosine similarity)

## Examples

```
ltv.utils.standard_clean_recipe(adata, spliced_key='spliced', unspliced_key='unspliced', batch_key='batch'); model = ltv.models.VAE(observed=adata.n_vars, latent_dim=20, zr_dim=2, h_dim=2, batch_correction=True, batches=2); epochs, val_ae, val_traj = ltv.train(model, adata, batch_size=100); latent_adata, adata = ltv.output_results(model, adata); scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity')
```

## Evaluation signals

- LatentVelo scores best on every batch-correction and velocity metric in the compiled table, matching the reported finding that it is 'the only model that scores highly on all batch-correction and velocity metrics'
- ComBat- and scGen-corrected UniTVelo runs fail to properly integrate the batches, giving poor kBET/iLISI/cLISI values
- scVelo velocity graphs build successfully on the latent_adata output with vkey='spliced_velocity' without errors
- The output CSV (dyngen_bifurcation_batch_metrics.csv) contains rows for all three methods and all six metrics (kBET, iLISI, cLISI, CBDir, ICCoh, NN velocity cosine)

## Limitations

- LatentVelo assumes a single initial latent state from which all cell trajectories start, with each cell reachable by a continuous trajectory; disconnected or sparse clusters can distort inferred trajectories (e.g. microglia in the dentate gyrus dataset)
- Sparse cell clusters with few transient connecting cells can cause latent trajectories to not start from expected initial states
- Some datasets require hyperparameter adjustment (larger latent dimension, AnnotVAE cell-type-annotated model, or gene subsetting); heuristics for choosing hyperparameters remain underdeveloped

## Evidence

- [results] LatentVelo is the only model that scores highly on all batch-correction and velocity metrics: [evidence span withheld — anchor status: not_found]
