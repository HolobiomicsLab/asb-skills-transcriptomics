---
name: latent-regulatory-state-analysis
description: Use when a LatentVelo model with zr_dim > 0 has been trained on spliced/unspliced
  AnnData and you need to test whether the latent regulatory state predicts divergent
  cell fates or lineage outcomes.
when_to_use_negative:
- The dataset has a single continuous lineage with no divergent fates — zr separation
  analysis is uninformative without distinct outcome groups
- A celltype-annotated model is required (use ltv.models.AnnotVAE with ltv.utils.anvi_clean_recipe
  and a celltype_key instead of the standard VAE/zr workflow)
- Cells form sparse clusters with few observed transient cells connecting trajectories,
  where LatentVelo's latent trajectories may not start from expected initial states
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: LatentVelo
  role: Trains the VAE with zr_dim latent regulatory state, outputs latent AnnData,
    and computes diverging cell trajectories via ltv.cell_trajectories
  repo: https://github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Computes the velocity graph on the latent space (scv.tl.velocity_graph with
    vkey='spliced_velocity') and provides the scVelo-velocity control for CellRank
    terminal-state comparison
- name: scanpy
  role: Provides the UMAP embedding used to visualize zr separation and latent trajectories
- name: CellRank
  role: Used as a control to confirm scVelo-inferred velocities recover only a single
    (dead-end) terminal state, contrasting with zr's fate separation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:e954fd2c8d0aba097bfbfad2a9403c45adf291f1bc007ceaab2ba8cfc04100ed
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
  - CellRank
  - LatentVelo
  - scVelo
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# latent-regulatory-state-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Analyzes the latent regulatory state (zr) inferred by LatentVelo's VAE to determine whether it separates cells into distinct dynamics regimes (e.g., reprogrammed vs. dead-end fates in the Biddy et al. fibroblast reprogramming dataset), using visualization on the latent UMAP and a logistic regression classifier with ROC evaluation.

## When to use

Use when a LatentVelo model with zr_dim > 0 has been trained on spliced/unspliced AnnData and you need to test whether the latent regulatory state predicts divergent cell fates or lineage outcomes.

## When NOT to use

- The dataset has a single continuous lineage with no divergent fates — zr separation analysis is uninformative without distinct outcome groups
- A celltype-annotated model is required (use ltv.models.AnnotVAE with ltv.utils.anvi_clean_recipe and a celltype_key instead of the standard VAE/zr workflow)
- Cells form sparse clusters with few observed transient cells connecting trajectories, where LatentVelo's latent trajectories may not start from expected initial states

## Inputs

- Trained LatentVelo VAE model (with zr_dim latent regulatory dimensions)
- Prepared AnnData object with spliced and unspliced layers (via ltv.utils.standard_clean_recipe, spliced_key='spliced', unspliced_key='unspliced', optional batch_key='batch')
- Latent AnnData from ltv.output_results containing the latent space and UMAP embedding
- Cell fate/outcome labels (e.g., reprogrammed vs. dead-end) for classifier training

## Outputs

- Latent UMAP visualization of zr showing separation of reprogrammed and dead-end cells
- Logistic regression ROC curve quantifying how predictive zr is of reprogramming outcome
- Diverging latent trajectories (z_traj, times from ltv.cell_trajectories) plotted on the latent UMAP
- CellRank terminal-state comparison showing scVelo velocities recover only one (dead-end) terminal state

## How to apply

After training a LatentVelo VAE (ltv.models.VAE with observed genes, latent_dim, zr_dim, h_dim) and running ltv.output_results(model, adata, gene_velocity=True, decoded=True, embedding='umap'), visualize the zr dimensions colored by known fate labels (reprogrammed vs. dead-end) on the latent UMAP. Fit a logistic regression classifier on zr to predict the reprogramming outcome and report the ROC curve (as in Figure 3K of the Biddy et al. reprogramming analysis). Complement this with ltv.cell_trajectories(model, adata) to show latent trajectories diverging toward the distinct fates, and run a CellRank control using scVelo-inferred velocities to confirm the baseline recovers only a single (dead-end) terminal state, demonstrating that zr adds fate-resolving signal beyond linear RNA velocity.

## Related tools

- **LatentVelo** (Trains the VAE with zr_dim latent regulatory state, outputs latent AnnData, and computes diverging cell trajectories via ltv.cell_trajectories) — https://github.com/Spencerfar/LatentVelo
- **scVelo** (Computes the velocity graph on the latent space (scv.tl.velocity_graph with vkey='spliced_velocity') and provides the scVelo-velocity control for CellRank terminal-state comparison)
- **scanpy** (Provides the UMAP embedding used to visualize zr separation and latent trajectories)
- **CellRank** (Used as a control to confirm scVelo-inferred velocities recover only a single (dead-end) terminal state, contrasting with zr's fate separation)

## Examples

```
model = ltv.models.VAE(observed=n_genes, latent_dim=20, zr_dim=2, h_dim=2); epochs, val_ae, val_traj = ltv.train(model, adata, batch_size=512, epochs=100); latent_adata, adata = ltv.output_results(model, adata, gene_velocity=True, decoded=True, embedding='umap')
```

## Evaluation signals

- zr visualization on the latent UMAP qualitatively shows reprogrammed and dead-end cells occupying distinct regions, reproducing Figure 3
- The logistic regression ROC curve demonstrates zr is predictive of reprogramming outcome (Figure 3K)
- ltv.cell_trajectories trajectories visibly diverge toward reprogrammed and dead-end fates on the latent UMAP
- CellRank run on scVelo-inferred velocities recovers only one (dead-end) terminal state, validating that fate resolution comes from the zr state
- Validation autoencoder and trajectory reconstruction losses from ltv.train decrease and converge during training

## Limitations

- LatentVelo assumes a single initial latent state from which all cell trajectories start, with each cell reachable by a continuous trajectory
- In datasets with sparse cell clusters and few transient connecting cells, latent trajectories may not start from the expected initial states (e.g., the disconnected microglia cluster in the dentate gyrus dataset)
- Some datasets require hyperparameter adjustment (larger latent_dim, cell-type annotated AnnotVAE model, or gene subsetting) for the zr state to be informative
- Two challenges not addressed are stochastic dynamics and normalization; potential fixes include replacing latent ODEs with SDEs and modeling normalization factors as latent variables as in scVI

## Evidence
