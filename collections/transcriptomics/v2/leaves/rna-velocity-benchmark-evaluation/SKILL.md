---
name: rna-velocity-benchmark-evaluation
description: Use when comparing RNA velocity methods on real datasets with known trajectory
  directions, or when quantitative CBDir/ICCoh benchmark scores are needed to validate
  velocity estimates.
when_to_use_negative:
- Data lacks unspliced/spliced layers — RNA velocity (both LatentVelo and scVelo)
  requires AnnData objects with spliced and unspliced count layers
- The goal is synthetic-data or batch-correction benchmarking rather than real-dataset
  trajectory direction evaluation — those use separate benchmark subdirectories in
  the LatentVelo repo
- Datasets contain sparse cell clusters with few observed transient cells connecting
  trajectories — velocity scores may remain accurate but inferred latent trajectories
  can start from incorrect initial states
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: LatentVelo
  role: Method under evaluation; produces latent-space velocities via ltv.models.VAE/AnnotVAE,
    ltv.train, and ltv.output_results
  repo: https://github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Baseline method run in stochastic and dynamical modes on the same datasets;
    also computes velocity graphs with scv.tl.velocity_graph
- name: scIB
  role: Source of the CBDir and ICCoh benchmark metrics used to score each method/dataset
    pair
- name: scanpy
  role: AnnData handling and neighborhood/embedding operations underlying scoring
- name: pandas
  role: Aggregation of per-dataset CBDir/ICCoh scores into the comparison CSV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:e54ab131eda01bf6efe75bcbf5d70c1dea1dfe3dc8511542a1512865321671ac
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
  - LatentVelo
  - numpy
  - pandas
  - scIB
  - scVelo
  - scanpy
  - scib
  - scvi-tools
  - sklearn
  - torchdiffeq
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# rna-velocity-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark RNA velocity methods (LatentVelo vs scVelo stochastic/dynamical) across developmental h5ad datasets using cross-boundary directedness (CBDir) and inner-cluster coherence (ICCoh) scores from the scIB benchmark suite, aggregating results into comparison tables.

## When to use

Use when comparing RNA velocity methods on real datasets with known trajectory directions, or when quantitative CBDir/ICCoh benchmark scores are needed to validate velocity estimates.

## When NOT to use

- Data lacks unspliced/spliced layers — RNA velocity (both LatentVelo and scVelo) requires AnnData objects with spliced and unspliced count layers
- The goal is synthetic-data or batch-correction benchmarking rather than real-dataset trajectory direction evaluation — those use separate benchmark subdirectories in the LatentVelo repo
- Datasets contain sparse cell clusters with few observed transient cells connecting trajectories — velocity scores may remain accurate but inferred latent trajectories can start from incorrect initial states

## Inputs

- h5ad AnnData files with 'spliced' and 'unspliced' count layers (e.g. scEU_organoid.h5ad, example_bifurcation.h5ad)
- Dataset-specific cross-boundary transition annotations for CBDir scoring
- Trained method outputs: LatentVelo latent_adata and scVelo velocity results on the same datasets

## Outputs

- Per-method, per-dataset CBDir (cross-boundary directedness) scores
- Per-method, per-dataset ICCoh (inner-cluster coherence) scores
- Aggregated comparison table (cbdir_iccoh_benchmark_scores.csv) for LatentVelo, scVelo stochastic, and scVelo dynamical

## How to apply

Load each h5ad AnnData dataset and prepare it with ltv.utils.standard_clean_recipe (spliced_key='spliced', unspliced_key='unspliced', plus batch_key/celltype_key where applicable) or ltv.utils.anvi_clean_recipe for the celltype-annotated model. Run LatentVelo (ltv.models.VAE or AnnotVAE with observed, latent_dim, zr_dim, h_dim; batch_correction=True with the number of batches where needed), train via ltv.train, and export results with ltv.output_results(model, adata, gene_velocity=True, embedding='umap'). Run scVelo stochastic and dynamical baselines on the same datasets, then compute each method's velocity graph with scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity'). Score every method/dataset pair using the dataset-specific CBDir cross-boundary transition annotations and ICCoh in-cluster coherence metrics (scIB benchmark metrics), then aggregate per-dataset scores into the Figure 6D-6E comparison tables (cbdir_iccoh_benchmark_scores.csv).

## Related tools

- **LatentVelo** (Method under evaluation; produces latent-space velocities via ltv.models.VAE/AnnotVAE, ltv.train, and ltv.output_results) — https://github.com/Spencerfar/LatentVelo
- **scVelo** (Baseline method run in stochastic and dynamical modes on the same datasets; also computes velocity graphs with scv.tl.velocity_graph)
- **scIB** (Source of the CBDir and ICCoh benchmark metrics used to score each method/dataset pair)
- **scanpy** (AnnData handling and neighborhood/embedding operations underlying scoring)
- **pandas** (Aggregation of per-dataset CBDir/ICCoh scores into the comparison CSV)

## Examples

```
model = ltv.models.VAE(observed=n_genes, latent_dim=20, zr_dim=2, h_dim=2); ltv.train(model, adata, batch_size=512, epochs=100); latent_adata, adata = ltv.output_results(model, adata, gene_velocity=True); scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity')
```

## Evaluation signals

- LatentVelo consistently scores above scVelo stochastic and dynamical on CBDir across all 10 real datasets
- LatentVelo achieves high ICCoh values for all datasets, indicating coherent neighboring velocity directions
- Velocity graph computation succeeds with vkey='spliced_velocity' on the latent_adata output
- Per-dataset CBDir and ICCoh entries exist for all three methods (LatentVelo, scVelo stochastic, scVelo dynamical) in cbdir_iccoh_benchmark_scores.csv
- Training returns validation autoencoder and trajectory reconstruction losses (val_ae, val_traj) that can be inspected for convergence

## Limitations

- LatentVelo assumes a single initial latent state from which all cell trajectories start, with each cell reachable by a continuous trajectory
- Sparse cell clusters with few observed transient cells connecting trajectories cause inferred latent trajectories to not start from expected initial states (e.g. the disconnected microglia cluster in the dentate gyrus dataset)
- Some datasets require additional hyperparameter adjustment (larger latent dimension, cell-type annotated AnnotVAE model, or gene subsetting), and only heuristics exist for hyperparameter choice
- Stochastic dynamics and normalization challenges are not addressed by LatentVelo

## Evidence

- [results] LatentVelo consistently scores above scVelo for CBDir scores on the 10 real datasets, and achieves high ICCoh values for all datasets: [evidence span withheld — anchor status: not_found]
