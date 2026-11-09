---
name: multi-dataset-method-comparison-table-generation
description: Use when a method's performance must be quantified against baselines
  across several real datasets with known trajectory directions, using trajectory-level
  metrics such as CBDir and ICCoh, and summarized into a per-dataset score table.
when_to_use_negative:
- Datasets lack spliced/unspliced layers or known transition annotations, making CBDir
  scoring undefined
- The goal is a single-dataset qualitative velocity plot rather than a multi-dataset
  quantitative method comparison
- Evaluating methods on synthetic data only — the paper's benchmarks/ directory separates
  synthetic-data benchmarking from real-dataset comparison
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: LatentVelo
  role: Method under test; trained per dataset with ltv.models.VAE/AnnotVAE and ltv.train,
    results exported via ltv.output_results
  repo: github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Baseline method run in stochastic and dynamical modes on the same 10 datasets;
    also computes velocity graphs via scv.tl.velocity_graph
- name: scIB
  role: Source of CBDir and ICCoh benchmark metrics used to score each method/dataset
    pair
- name: scanpy
  role: AnnData handling and dataset loading (sc.read) for the h5ad inputs
- name: pandas
  role: Aggregation of per-dataset scores into the cbdir_iccoh_benchmark_scores.csv
    table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:e54c54fc4acfb723a89319decca2f376d602661dd6a0246f0c1eb6f205b7013b
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
  - scIB
  - scVelo
  - scanpy
  - scib
  - scvi-tools
  - sklearn
  - torchdiffeq
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# multi-dataset-method-comparison-table-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks two or more RNA velocity methods (LatentVelo vs scVelo stochastic/dynamical) across multiple developmental h5ad datasets using cross-boundary directedness (CBDir) and inner-cluster coherence (ICCoh) scores, then aggregates per-dataset results into a comparison table (e.g., cbdir_iccoh_benchmark_scores.csv for Figure 6D-6E style comparisons).

## When to use

Use when a method's performance must be quantified against baselines across several real datasets with known trajectory directions, using trajectory-level metrics such as CBDir and ICCoh, and summarized into a per-dataset score table.

## When NOT to use

- Datasets lack spliced/unspliced layers or known transition annotations, making CBDir scoring undefined
- The goal is a single-dataset qualitative velocity plot rather than a multi-dataset quantitative method comparison
- Evaluating methods on synthetic data only — the paper's benchmarks/ directory separates synthetic-data benchmarking from real-dataset comparison

## Inputs

- h5ad AnnData datasets with 'spliced' and 'unspliced' count layers (10 developmental/reprogramming datasets)
- Dataset-specific cross-boundary transition annotations for CBDir scoring
- Cell-type and batch annotation keys (batch_key, celltype_key)
- Method outputs: LatentVelo latent_adata and scVelo stochastic/dynamical results with velocity keys

## Outputs

- Per-dataset CBDir and ICCoh scores for each method (LatentVelo, scVelo stochastic, scVelo dynamical)
- Aggregated comparison table saved as cbdir_iccoh_benchmark_scores.csv (Figure 6D-6E comparison tables)

## How to apply

Load each h5ad AnnData dataset and prepare it with ltv.utils.standard_clean_recipe (or ltv.utils.anvi_clean_recipe when using the cell-type-annotated model) with spliced_key='spliced' and unspliced_key='unspliced'. Train LatentVelo via ltv.models.VAE or ltv.models.AnnotVAE (with batch_correction=True and the number of batches where needed), then export results with ltv.output_results(model, adata, gene_velocity=True, embedding='umap'). Run the scVelo stochastic and dynamical baselines on the same datasets for a controlled comparison. Compute the velocity graph for each method with scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity'), score each method/dataset pair with the dataset-specific CBDir transition annotations and ICCoh in-cluster coherence metrics (scIB benchmark metrics), and aggregate per-dataset scores for LatentVelo, scVelo stochastic, and scVelo dynamical into the Figure 6D-6E comparison tables saved as cbdir_iccoh_benchmark_scores.csv.

## Related tools

- **LatentVelo** (Method under test; trained per dataset with ltv.models.VAE/AnnotVAE and ltv.train, results exported via ltv.output_results) — github.com/Spencerfar/LatentVelo
- **scVelo** (Baseline method run in stochastic and dynamical modes on the same 10 datasets; also computes velocity graphs via scv.tl.velocity_graph)
- **scIB** (Source of CBDir and ICCoh benchmark metrics used to score each method/dataset pair)
- **scanpy** (AnnData handling and dataset loading (sc.read) for the h5ad inputs)
- **pandas** (Aggregation of per-dataset scores into the cbdir_iccoh_benchmark_scores.csv table)

## Examples

```
latent_adata, adata = ltv.output_results(model, adata, gene_velocity=True, embedding='umap'); scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity')
```

## Evaluation signals

- LatentVelo consistently scores above scVelo on CBDir across all 10 real datasets
- LatentVelo achieves high ICCoh values for all datasets, indicating coherent neighboring velocity directions
- Every method/dataset pair has both a CBDir and an ICCoh score present in cbdir_iccoh_benchmark_scores.csv (no missing cells in the comparison table)
- The velocity graph was computed with vkey='spliced_velocity' on the latent_adata output before scoring
- Identical data preparation (standard_clean_recipe/anvi_clean_recipe with spliced_key='spliced', unspliced_key='unspliced') was applied across all compared methods

## Limitations

- LatentVelo assumes a single initial latent state from which all cell trajectories start and that each cell is reachable by a continuous trajectory
- Sparse cell clusters with few observed transient cells can cause latent trajectories to not start from expected initial states (e.g., the disconnected microglia cluster in the dentate gyrus dataset is incorrectly assigned the lowest latent times)
- Some datasets require additional hyperparameter adjustment (larger latent dimension, cell-type annotated model, or gene subsetting), complicating fair cross-dataset comparison

## Evidence

- [other] LatentVelo consistently scores above scVelo for CBDir scores on the 10 real datasets, and achieves high ICCoh values for all datasets: [evidence span withheld — anchor status: not_found]
