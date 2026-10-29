---
name: cell-fate-transition-scoring
description: Use when evaluating RNA velocity or latent-trajectory outputs on a multi-lineage,
  multi-batch scRNA-seq dataset (e.g., mouse gastrulation AnnData with spliced/unspliced
  layers) and you need quantitative transition, batch-mixing, and velocity-consistency
  scores.
when_to_use_negative:
- The dataset has sparse cell clusters with few observed transient cells connecting
  trajectories, where LatentVelo-inferred trajectories do not start from expected
  initial states
- Trajectories must originate from a known, disconnected initial cluster (e.g., dentate
  gyrus microglia), since LatentVelo assumes a single initial latent state reachable
  by continuous trajectories
- Only standard scVI/scANVI latent embeddings without gene expression dynamics are
  needed — the article shows dynamics-aware embeddings outperform these for batch
  correction of velocities
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: LatentVelo
  role: Trains the AnnotVAE dynamics model, outputs latent velocities and trajectories
    (ltv.cell_trajectories) that the CBDir, kBET, and cosine-similarity benchmarks
    evaluate
  repo: github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Builds the velocity graph on the 'spliced_velocity' key for latent-space streamplots
    used in transition scoring
- name: scib
  role: Provides batch-mixing (kBET) benchmark utilities across the 3 sequencing batches
- name: scanpy
  role: Utilities for the nearest-neighbor velocity cosine-similarity computation
    and figure assembly
- name: GSEApy
  role: Runs GO enrichment on genes differentially expressed along the inferred endothelial,
    erythroid, mesenchyme, and neural trajectories
  repo: github.com/zqfang/GSEApy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:6170838ff0511883882bf2a464cd6f70b84f16b44513f1677bfef5f5d2a128b4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Spencerfar/LatentVelo
  tools:
  - GSEApy
  - LatentVelo
  - scVelo
  - scanpy
  - scib
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cell-fate-transition-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Score inferred single-cell fate transitions by computing cross-boundary direction (CBDir) transition scores on RNA-velocity graphs, alongside batch-mixing (kBET) and nearest-neighbor velocity cosine similarity, to validate that LatentVelo-derived trajectories reproduce known lineage transitions in multi-lineage datasets such as mouse gastrulation.

## When to use

Use when evaluating RNA velocity or latent-trajectory outputs on a multi-lineage, multi-batch scRNA-seq dataset (e.g., mouse gastrulation AnnData with spliced/unspliced layers) and you need quantitative transition, batch-mixing, and velocity-consistency scores.

## When NOT to use

- The dataset has sparse cell clusters with few observed transient cells connecting trajectories, where LatentVelo-inferred trajectories do not start from expected initial states
- Trajectories must originate from a known, disconnected initial cluster (e.g., dentate gyrus microglia), since LatentVelo assumes a single initial latent state reachable by continuous trajectories
- Only standard scVI/scANVI latent embeddings without gene expression dynamics are needed — the article shows dynamics-aware embeddings outperform these for batch correction of velocities

## Inputs

- AnnData object with spliced and unspliced count layers, batch key, and celltype key (e.g., 20,000-cell mouse gastrulation subset, .h5ad)
- Trained LatentVelo model parameters saved under the named output folder
- Velocity graph computed on the 'spliced_velocity' key in the latent AnnData

## Outputs

- CBDir cross-boundary directionality transition scores per lineage transition
- kBET batch-mixing scores per sequencing batch
- Nearest-neighbor velocity cosine similarity scores across batches
- Latent trajectories (z_traj, times) from ltv.cell_trajectories
- GO enrichment results (GSEApy) for genes differentially expressed along inferred trajectories
- Figure panels (e.g., figure5_panels.png) combining transition, batch-mixing, cosine-similarity, and GO-enrichment results

## How to apply

Prepare the AnnData with ltv.utils.anvi_clean_recipe (spliced_key='spliced', unspliced_key='unspliced', batch_key='batch', celltype_key='celltype'), train a celltype-annotated ltv.models.AnnotVAE with batch_correction=True (e.g., latent_dim=70, zr_dim=6, batches=3 for the 20,000-cell mouse gastrulation subset), and export results with ltv.output_results(gene_velocity=True, decoded=True, embedding='umap'). Build the velocity graph with scv.tl.velocity_graph on the 'spliced_velocity' key in the latent AnnData, then compute CBDir cross-boundary directionality scores across the lineage transitions, kBET batch-mixing scores, and nearest-neighbor velocity cosine similarity across the sequencing batches using scib and scanpy utilities. Extract latent trajectories with ltv.cell_trajectories, identify differentially expressed genes along inferred lineages (endothelial, erythroid, mesenchyme, neural), and run GO enrichment with GSEApy to confirm the transitions are biologically valid.

## Related tools

- **LatentVelo** (Trains the AnnotVAE dynamics model, outputs latent velocities and trajectories (ltv.cell_trajectories) that the CBDir, kBET, and cosine-similarity benchmarks evaluate) — github.com/Spencerfar/LatentVelo
- **scVelo** (Builds the velocity graph on the 'spliced_velocity' key for latent-space streamplots used in transition scoring)
- **scib** (Provides batch-mixing (kBET) benchmark utilities across the 3 sequencing batches)
- **scanpy** (Utilities for the nearest-neighbor velocity cosine-similarity computation and figure assembly)
- **GSEApy** (Runs GO enrichment on genes differentially expressed along the inferred endothelial, erythroid, mesenchyme, and neural trajectories) — github.com/zqfang/GSEApy

## Examples

```
z_traj, times = ltv.cell_trajectories(model, adata)
```

## Evaluation signals

- kBET batch-mixing score for LatentVelo should beat scVI, scANVI, and the original unintegrated data on the mouse gastrulation dataset
- Nearest-neighbor velocity cosine similarity should be high between sequencing batches, indicating velocities are batch-aligned
- CBDir scores should confirm the expected directionality of gastrulation lineage transitions (endothelial, erythroid, mesenchyme, neural)
- GO enrichment on trajectory-specific DE genes should return biologically coherent terms for each inferred lineage
- Validation autoencoder and trajectory reconstruction losses from ltv.train should converge during training

## Limitations

- LatentVelo assumes a single initial latent state from which all cell trajectories start and that every observed cell is reachable by a continuous trajectory
- In sparse cell clusters with few observed transient cells connecting trajectories, inferred latent trajectories may not start from expected initial states
- On the dentate gyrus dataset, LatentVelo assigned the lowest latent times to the disconnected microglia cluster, so inferred trajectories incorrectly started from it
- Stochastic dynamics and normalization are not addressed; the authors propose replacing latent ODEs with SDEs and modeling normalization factors as latent variables as in scVI

## Evidence
