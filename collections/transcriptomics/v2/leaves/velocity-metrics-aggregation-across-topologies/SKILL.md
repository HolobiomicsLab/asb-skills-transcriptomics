---
name: velocity-metrics-aggregation-across-topologies
description: Use when comparing RNA velocity methods on synthetic dyngen AnnData with
  ground-truth velocities across multiple developmental topologies and a combined
  benchmark table is required.
when_to_use_negative:
- The AnnData lacks spliced and unspliced count layers, since LatentVelo requires
  both ('This object must have two layers containing spliced and unspliced counts.')
- The goal is batch-corrected velocity benchmarking on real multiplexed data rather
  than ground-truth-based scoring on synthetic dyngen topologies
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LatentVelo
  role: VAE-based velocity model whose latent velocities (spliced_velocity) are the
    method under test, applied via ltv.utils.standard_clean_recipe, ltv.models.VAE,
    ltv.train, and ltv.output_results
  repo: github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Baseline linear RNA velocity method run in dynamical and stochastic modes
    on the same four dyngen datasets; also computes the velocity graph with scv.tl.velocity_graph(vkey='spliced_velocity')
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:d19163ed917b5965d001229e242c5bf8e7c3edfde6c825425783ef09496c4e2b
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# velocity-metrics-aggregation-across-topologies

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark LatentVelo against scVelo's dynamical and stochastic modes on dyngen synthetic datasets (linear, bifurcation, trifurcation, binary tree; 5000 cells each), scoring velocities with cosine similarity to ground truth plus CBDir and ICCoh on 50 PCs, and aggregate all scores into a single metrics table (synthetic_benchmark_metrics.csv).

## When to use

Use when comparing RNA velocity methods on synthetic dyngen AnnData with ground-truth velocities across multiple developmental topologies and a combined benchmark table is required.

## When NOT to use

- The AnnData lacks spliced and unspliced count layers, since LatentVelo requires both ('This object must have two layers containing spliced and unspliced counts.')
- The goal is batch-corrected velocity benchmarking on real multiplexed data rather than ground-truth-based scoring on synthetic dyngen topologies

## Inputs

- dyngen synthetic AnnData .h5ad datasets with spliced and unspliced layers (linear, bifurcation, trifurcation, binary tree; 5000 cells each)
- Ground-truth velocities from dyngen simulations

## Outputs

- synthetic_benchmark_metrics.csv aggregating cosine similarity, CBDir, and ICCoh scores for LatentVelo, scVelo dynamical, and scVelo stochastic across all four topologies
- LatentVelo latent AnnData with spliced_velocity and computed velocity graph per dataset

## How to apply

Load each dyngen synthetic AnnData (spliced/unspliced layers, 5000 cells) and prepare it with ltv.utils.standard_clean_recipe using spliced_key='spliced' and unspliced_key='unspliced'. Initialize ltv.models.VAE with configured observed-gene, latent_dim, zr_dim, and h_dim settings, train via ltv.train (monitoring validation autoencoder and trajectory reconstruction losses), export with ltv.output_results (gene_velocity=True), and build the velocity graph with scv.tl.velocity_graph on the latent AnnData using vkey='spliced_velocity'. Run scVelo in both dynamical and stochastic modes on the same four datasets as baselines. Compute cosine similarity between ground-truth and estimated velocities plus CBDir and ICCoh on 50 PCs for all three methods on each topology, then aggregate into one table saved as synthetic_benchmark_metrics.csv. Correct application is judged by the metrics table existing and LatentVelo scoring above both scVelo modes, matching the ordering in Figures 6A-6C.

## Related tools

- **LatentVelo** (VAE-based velocity model whose latent velocities (spliced_velocity) are the method under test, applied via ltv.utils.standard_clean_recipe, ltv.models.VAE, ltv.train, and ltv.output_results) — github.com/Spencerfar/LatentVelo
- **scVelo** (Baseline linear RNA velocity method run in dynamical and stochastic modes on the same four dyngen datasets; also computes the velocity graph with scv.tl.velocity_graph(vkey='spliced_velocity'))

## Examples

```
latent_adata, adata = ltv.output_results(model, adata, gene_velocity=True); scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity')
```

## Evaluation signals

- synthetic_benchmark_metrics.csv exists and contains rows for all four topologies (linear, bifurcation, trifurcation, binary tree) and all three methods
- LatentVelo cosine similarity between ground-truth and estimated velocities exceeds both scVelo dynamical and stochastic scores on each dataset
- LatentVelo CBDir and ICCoh scores computed on 50 PCs rank above both scVelo modes, matching the qualitative ordering of Figures 6A-6C
- Training converges with decreasing validation autoencoder and trajectory reconstruction losses reported by ltv.train
- The velocity graph computes successfully on the latent AnnData with vkey='spliced_velocity'

## Limitations

- Conclusions rest on dyngen synthetic topologies with a single initial latent state and continuous trajectories; LatentVelo assumes 'a single initial latent state from which the trajectory of all cells start from'
- LatentVelo may encounter issues with sparse cell clusters with few observed transient cells connecting trajectories, which can affect velocity-informed trajectory starts even when velocities remain accurate
- Hyperparameter choices (latent dimension, cell-type annotated model, gene subsetting) rely on heuristics; 'more work needs to be done in this area'

## Evidence
