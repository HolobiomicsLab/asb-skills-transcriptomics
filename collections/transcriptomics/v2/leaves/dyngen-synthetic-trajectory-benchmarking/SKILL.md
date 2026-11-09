---
name: dyngen-synthetic-trajectory-benchmarking
description: Use when comparing velocity estimation methods on synthetic data with
  known ground-truth velocities and diverse developmental topologies (linear, bifurcation,
  trifurcation, binary tree), e.g. validating a deep-learning velocity method against
  linear baselines.
when_to_use_negative:
- The dataset has no ground-truth velocities, so cosine similarity to truth cannot
  be computed (use real-data transition metrics instead)
- The goal is benchmarking batch correction rather than synthetic-trajectory velocity
  accuracy; the paper uses a separate batch-correction benchmark for that
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: LatentVelo
  role: Deep-learning method under test; VAE embedding of cells with neural-ODE latent
    dynamics, producing spliced_velocity estimates in latent space
  repo: github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Baseline linear velocity method run in dynamical and stochastic modes on the
    same synthetic datasets
- name: scanpy
  role: Loads the dyngen .h5ad AnnData objects for preparation and analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:9d1ae8521a757205e38b7c4d95f3b0d94e27cc3e0a9c5f69a22c8725ea72df95
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
  - scVelo
  - scanpy
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dyngen-synthetic-trajectory-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks LatentVelo against scVelo's dynamical and stochastic modes on dyngen synthetic AnnData datasets (linear, bifurcation, trifurcation, binary tree; 5000 cells) using cosine similarity between ground-truth and estimated velocities plus CBDir and ICCoh scores computed on 50 PCs.

## When to use

Use when comparing velocity estimation methods on synthetic data with known ground-truth velocities and diverse developmental topologies (linear, bifurcation, trifurcation, binary tree), e.g. validating a deep-learning velocity method against linear baselines.

## When NOT to use

- The dataset has no ground-truth velocities, so cosine similarity to truth cannot be computed (use real-data transition metrics instead)
- The goal is benchmarking batch correction rather than synthetic-trajectory velocity accuracy; the paper uses a separate batch-correction benchmark for that

## Inputs

- dyngen synthetic AnnData .h5ad datasets (e.g. example_bifurcation.h5ad) with spliced and unspliced layers and ground-truth velocities, 5000 cells per topology
- Four developmental topologies: linear, bifurcation, trifurcation, binary tree

## Outputs

- synthetic_benchmark_metrics.csv containing cosine similarity, CBDir, and ICCoh for LatentVelo, scVelo dynamical, and scVelo stochastic on each of the four topologies
- LatentVelo latent AnnData with spliced_velocity velocities and a computed velocity graph

## How to apply

Load each dyngen synthetic AnnData (5000 cells per topology) and prepare it with ltv.utils.standard_clean_recipe using spliced_key='spliced' and unspliced_key='unspliced'. Initialize a LatentVelo VAE via ltv.models.VAE with configured observed-gene, latent_dim, zr_dim, and h_dim settings, train it with ltv.train (monitoring validation autoencoder and trajectory reconstruction losses), export results with ltv.output_results (gene_velocity enabled), and compute the velocity graph with scv.tl.velocity_graph on the latent AnnData using vkey='spliced_velocity'. Run scVelo in both dynamical and stochastic modes on the same four datasets to obtain baseline velocity estimates. Compute cosine similarity between ground-truth and estimated velocities, plus CBDir and ICCoh scores on 50 PCs, for all three methods on each topology, and aggregate into a single metrics table saved as synthetic_benchmark_metrics.csv.

## Related tools

- **LatentVelo** (Deep-learning method under test; VAE embedding of cells with neural-ODE latent dynamics, producing spliced_velocity estimates in latent space) — github.com/Spencerfar/LatentVelo
- **scVelo** (Baseline linear velocity method run in dynamical and stochastic modes on the same synthetic datasets)
- **scanpy** (Loads the dyngen .h5ad AnnData objects for preparation and analysis)

## Examples

```
adata = sc.read('../synthetic_datasets/example_bifurcation.h5ad'); ltv.utils.standard_clean_recipe(adata, spliced_key='spliced', unspliced_key='unspliced'); latent_adata, adata = ltv.output_results(model, adata, gene_velocity=True); scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity')
```

## Evaluation signals

- synthetic_benchmark_metrics.csv exists and contains rows for all three methods across all four topologies
- LatentVelo scores above both scVelo dynamical and stochastic modes on cosine similarity, CBDir, and ICCoh, matching the ordering in Figures 6A-6C
- LatentVelo training shows decreasing validation autoencoder and trajectory reconstruction losses via ltv.train output
- Velocity graph computed successfully on the latent AnnData with vkey='spliced_velocity' without errors

## Limitations

- LatentVelo assumes a single initial latent state from which all cell trajectories start, with each cell reachable by a continuous trajectory
- Sparse cell clusters with few observed transient cells connecting trajectories can cause latent trajectories not to start from expected initial states
- Some datasets require additional hyperparameter adjustment (larger latent dimension, cell-type annotated model, or gene subsetting), and only heuristics exist for choosing hyperparameters

## Evidence

- [results] LatentVelo substantially outperforms scVelo's dynamical and stochastic modes on the synthetic dyngen datasets, as measured by cosine similarity between ground-truth and estimated velocities and by CBDir and ICCoh scores on 50 PCs: [evidence span withheld — anchor status: not_found]
