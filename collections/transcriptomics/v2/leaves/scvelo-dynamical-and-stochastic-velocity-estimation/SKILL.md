---
name: scvelo-dynamical-and-stochastic-velocity-estimation
description: Use when a benchmark requires traditional linear RNA velocity baselines
  — e.g., comparing against LatentVelo on dyngen synthetic AnnData (spliced/unspliced
  layers) or real scRNA-seq with spliced and unspliced counts.
when_to_use_negative:
- The data lacks unspliced (or spliced) count layers — scVelo's dynamical and stochastic
  models both rely on the unspliced-to-spliced relationship
- The goal is batch-corrected or lineage-aware velocity; the article shows traditional
  linear methods fail with multiple lineages, transcriptional boosts, lineage-dependent
  kinetics, and weak unspliced signals, and LatentVelo is designed for those settings
- Comparing against LatentVelo outputs on its latent AnnData, where velocities use
  vkey='spliced_velocity' and must not be overwritten by scVelo-computed velocities
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scVelo
  role: Computes the dynamical and stochastic RNA velocity baseline estimates on spliced/unspliced
    AnnData; also used to compute velocity graphs (scv.tl.velocity_graph) for evaluation
    and plotting
  repo: https://github.com/theislab/scvelo
- name: LatentVelo
  role: Method under comparison; the benchmark notebooks run scVelo alongside LatentVelo
    (benchmarks/ directory contains 'the code used to run scVelo')
  repo: https://github.com/Spencerfar/LatentVelo
- name: scanpy
  role: Reads the dyngen AnnData datasets (sc.read('.../example_bifurcation.h5ad'))
    used in the benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:c2272828c800c2deacc36979a3912130981394851de24fcdadaf6388e653ee20
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Spencerfar/LatentVelo
  repo_url: https://github.com/theislab/scvelo
  tools:
  - LatentVelo
  - scVelo
  - scanpy
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# scvelo-dynamical-and-stochastic-velocity-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run scVelo in its dynamical and stochastic modes to produce baseline RNA velocity estimates that can be benchmarked against latent-space methods such as LatentVelo on synthetic and real datasets, using metrics like cosine similarity, CBDir, and ICCoh.

## When to use

Use when a benchmark requires traditional linear RNA velocity baselines — e.g., comparing against LatentVelo on dyngen synthetic AnnData (spliced/unspliced layers) or real scRNA-seq with spliced and unspliced counts.

## When NOT to use

- The data lacks unspliced (or spliced) count layers — scVelo's dynamical and stochastic models both rely on the unspliced-to-spliced relationship
- The goal is batch-corrected or lineage-aware velocity; the article shows traditional linear methods fail with multiple lineages, transcriptional boosts, lineage-dependent kinetics, and weak unspliced signals, and LatentVelo is designed for those settings
- Comparing against LatentVelo outputs on its latent AnnData, where velocities use vkey='spliced_velocity' and must not be overwritten by scVelo-computed velocities

## Inputs

- AnnData .h5ad objects with spliced and unspliced count layers (e.g., dyngen synthetic datasets: linear, bifurcation, trifurcation, binary tree; 5000 cells each, e.g. example_bifurcation.h5ad)
- Ground-truth velocities for synthetic datasets (for cosine similarity evaluation)

## Outputs

- scVelo velocity estimates (dynamical mode and stochastic mode) per dataset
- Velocity graph for embedding/streamline visualization
- Benchmark metrics per method and topology (cosine similarity, CBDir, ICCoh on 50 PCs), aggregated e.g. as synthetic_benchmark_metrics.csv

## How to apply

Load each dataset (e.g., the four dyngen synthetic topologies — linear, bifurcation, trifurcation, binary tree; 5000 cells — stored as AnnData .h5ad with 'spliced' and 'unspliced' layers) and run scVelo in both dynamical and stochastic modes to obtain velocity estimates; the dynamical mode relaxes the steady-state assumption by modeling time-dependent transient cell states, while the stochastic mode builds on the original Velocyto steady-state formulation. Apply the same data preparation to all methods so comparisons are fair (for LatentVelo comparisons, the article uses ltv.utils.standard_clean_recipe). Compute velocities with scVelo and, for downstream embedding/streamline evaluation, calculate the velocity graph with scv.tl.velocity_graph. Evaluate all methods with cosine similarity between ground-truth and estimated velocities and with CBDir and ICCoh scores computed on 50 PCs, aggregating into a single metrics table. The article's benchmark found LatentVelo substantially outperforms both scVelo modes on the dyngen synthetic datasets (Figures 6A–6C), so these runs serve as the reference baselines for that ordering.

## Related tools

- **scVelo** (Computes the dynamical and stochastic RNA velocity baseline estimates on spliced/unspliced AnnData; also used to compute velocity graphs (scv.tl.velocity_graph) for evaluation and plotting) — https://github.com/theislab/scvelo
- **LatentVelo** (Method under comparison; the benchmark notebooks run scVelo alongside LatentVelo (benchmarks/ directory contains 'the code used to run scVelo')) — https://github.com/Spencerfar/LatentVelo
- **scanpy** (Reads the dyngen AnnData datasets (sc.read('.../example_bifurcation.h5ad')) used in the benchmark)

## Examples

```
import scvelo as scv; velo = scv.tl.velocity(adata, mode='dynamical'); scv.tl.velocity_graph(adata)
```

## Evaluation signals

- The aggregated metrics table (e.g., synthetic_benchmark_metrics.csv) exists and contains rows for both scVelo dynamical and stochastic modes on each of the four dyngen topologies
- Cosine similarity between ground-truth and estimated velocities is computed for both scVelo modes, and (per the article's finding) both score below LatentVelo
- CBDir and ICCoh scores are computed on 50 PCs for all three methods, matching the qualitative ordering shown in Figures 6A–6C (LatentVelo above both scVelo modes)
- Velocities are computed on identically prepared data across methods so the comparison is not confounded by preprocessing differences
- scVelo versions should match the paper's environment (scvelo 0.2.4) for reproducibility

## Limitations

- scVelo's dynamical and stochastic modes underperformed LatentVelo on dyngen synthetic datasets (cosine similarity, CBDir, ICCoh), so these baselines are not suitable when lineage-dependent or time-dependent dynamics dominate
- Traditional linear RNA velocity approaches use strict transcription/splicing modeling assumptions that 'can fail with multiple lineages or time-dependent rates' and encounter problems with transcriptional boosts, lineage-dependent kinetics, and weak unspliced signals
- These modes lack batch correction; LatentVelo's dynamics-based embedding batch correction outperformed comparable autoencoder batch correction methods for velocities

## Evidence

- [results] LatentVelo substantially outperforms scVelo's dynamical and stochastic modes on the synthetic dyngen datasets, as measured by cosine similarity between ground-truth and estimated velocities and by CBDir and ICCoh scores on 50 PCs: [evidence span withheld — anchor status: not_found]
