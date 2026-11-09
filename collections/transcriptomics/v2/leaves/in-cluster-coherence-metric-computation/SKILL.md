---
name: in-cluster-coherence-metric-computation
description: Use when benchmarking RNA velocity methods on h5ad developmental datasets
  and you need to score the directional coherence of inferred velocities within clusters,
  e.g. LatentVelo vs scVelo.
when_to_use_negative:
- Velocities and a velocity graph have not yet been computed for the method being
  scored — run ltv.output_results and scv.tl.velocity_graph first
- The goal is cross-cluster (cross-boundary) directional accuracy rather than within-cluster
  velocity coherence — use the CBDir metric instead
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scvelo
  role: Computes the velocity graph that ICCoh is calculated from
  repo: github.com/Spencerfar/LatentVelo
- name: scib
  role: Provides the ICCoh in-cluster coherence benchmark metric
- name: LatentVelo
  role: Produces latent-space velocities (vkey='spliced_velocity') being scored
  repo: github.com/Spencerfar/LatentVelo
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:c49b036ebf681d659c6a86701e690e1567df0186912aba5c2dee3f578f5dc608
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
  - numpy
  - pandas
  - scanpy
  - scib
  - scvelo
  - scvi-tools
  - sklearn
  - torchdiffeq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# in-cluster-coherence-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the inner-cluster coherence (ICCoh) score, an scIB-based metric that quantifies how consistently neighboring cells' velocity vectors point in the same direction within clusters, used to benchmark RNA velocity methods such as LatentVelo against scVelo stochastic/dynamical modes.

## When to use

Use when benchmarking RNA velocity methods on h5ad developmental datasets and you need to score the directional coherence of inferred velocities within clusters, e.g. LatentVelo vs scVelo.

## When NOT to use

- Velocities and a velocity graph have not yet been computed for the method being scored — run ltv.output_results and scv.tl.velocity_graph first
- The goal is cross-cluster (cross-boundary) directional accuracy rather than within-cluster velocity coherence — use the CBDir metric instead

## Inputs

- AnnData (.h5ad) objects with computed velocities per method (LatentVelo latent_adata with vkey='spliced_velocity', or scVelo outputs)
- Velocity graph computed via scv.tl.velocity_graph
- Dataset-specific cross-boundary transition annotations and celltype labels

## Outputs

- Per-dataset ICCoh in-cluster coherence scores for each method/dataset pair
- Aggregated comparison table cbdir_iccoh_benchmark_scores.csv (Figure 6D-6E)

## How to apply

After each velocity method (LatentVelo via ltv.output_results with gene_velocity=True, or scVelo stochastic/dynamical) has produced velocities, compute the velocity graph on the latent AnnData with scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity'). Then score every method/dataset pair with the ICCoh in-cluster coherence metric from the scIB benchmark toolkit alongside the CBDir cross-boundary directedness score, using dataset-specific transition annotations. Aggregate per-dataset ICCoh values for LatentVelo, scVelo stochastic, and scVelo dynamical into the Figure 6D-6E comparison tables and save as cbdir_iccoh_benchmark_scores.csv. High ICCoh values indicate coherent neighboring velocity directions, which is the expected outcome for LatentVelo across the 10 real datasets.

## Related tools

- **scvelo** (Computes the velocity graph that ICCoh is calculated from) — github.com/Spencerfar/LatentVelo
- **scib** (Provides the ICCoh in-cluster coherence benchmark metric)
- **LatentVelo** (Produces latent-space velocities (vkey='spliced_velocity') being scored) — github.com/Spencerfar/LatentVelo

## Examples

```
scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity')
```

## Evaluation signals

- A velocity graph exists for the latent_adata (scv.tl.velocity_graph was run with vkey='spliced_velocity') before scoring
- ICCoh values are reported for every method/dataset pair (10 datasets × 3 methods: LatentVelo, scVelo stochastic, scVelo dynamical)
- LatentVelo is expected to achieve high ICCoh values for all datasets, indicating coherent neighboring velocity directions
- Scores are aggregated and saved in cbdir_iccoh_benchmark_scores.csv matching the Figure 6D-6E comparison tables

## Limitations

- LatentVelo velocities can be inaccurate in sparse cell clusters with few observed transient cells connecting trajectories, which would degrade ICCoh scores
- LatentVelo may incorrectly assign lowest latent times to disconnected clusters (e.g. microglia in the dentate gyrus dataset), affecting trajectory-based interpretations
- Choosing LatentVelo hyperparameters (latent dimension, cell-type annotated model, gene subsetting) remains heuristic and can affect the coherence scores

## Evidence

- [results] benchmark finding: [evidence span withheld — anchor status: not_found]
- [results] benchmark aggregation: [evidence span withheld — anchor status: not_found]
- [results] metric definition: [evidence span withheld — anchor status: not_found]
