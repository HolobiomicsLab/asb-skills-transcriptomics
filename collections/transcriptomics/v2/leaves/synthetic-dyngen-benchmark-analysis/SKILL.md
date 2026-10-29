---
name: synthetic-dyngen-benchmark-analysis
description: Use when evaluating whether a dynamics-aware latent model batch-corrects
  cell states and velocities better than gene-space correction (ComBat, scGen) on
  multi-batch synthetic data with a bifurcating trajectory.
when_to_use_negative:
- When the dataset is single-batch — batch-correction initialization (batch_correction=True,
  batches) and integration metrics like kBET/iLISI/cLISI are meaningless without multiple
  batches
- When the goal is trajectory inference rather than batch-correction benchmarking
  — use ltv.cell_trajectories and latent-time analysis instead
- When a real developmental dataset with known topology is available; this skill is
  specific to the synthetic dyngen bifurcation ground truth
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: LatentVelo
  role: Dynamics-based latent model run with batch_correction=True to produce batch-corrected
    cell states and velocities
  repo: https://github.com/Spencerfar/LatentVelo
- name: UniTVelo
  role: Baseline RNA velocity method rerun on unintegrated, ComBat-corrected, and
    scGen-corrected matrices
- name: scGen
  role: Gene-space batch correction applied to counts before rerunning UniTVelo as
    a baseline
- name: scib
  role: Computes cell-state integration metrics kBET, iLISI, and cLISI
- name: scVelo
  role: Builds velocity graphs for nearest-neighbor velocity cosine similarity and
    velocity metrics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:7a06e7ce23162a76769c33e89ec14a6a0b7de90390e7db1855e0c7799bd0271d
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
  - UniTVelo
  - scGen
  - scVelo
  - scib
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# synthetic-dyngen-benchmark-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks RNA velocity batch-correction methods on the two-batch dyngen bifurcation synthetic dataset (example_bifurcation.h5ad), comparing LatentVelo against UniTVelo run on unintegrated, ComBat-corrected, and scGen-corrected data using kBET/iLISI/cLISI and velocity metrics.

## When to use

Use when evaluating whether a dynamics-aware latent model batch-corrects cell states and velocities better than gene-space correction (ComBat, scGen) on multi-batch synthetic data with a bifurcating trajectory.

## When NOT to use

- When the dataset is single-batch — batch-correction initialization (batch_correction=True, batches) and integration metrics like kBET/iLISI/cLISI are meaningless without multiple batches
- When the goal is trajectory inference rather than batch-correction benchmarking — use ltv.cell_trajectories and latent-time analysis instead
- When a real developmental dataset with known topology is available; this skill is specific to the synthetic dyngen bifurcation ground truth

## Inputs

- example_bifurcation.h5ad (two-batch dyngen synthetic bifurcation dataset with spliced/unspliced layers and a batch key)
- AnnData objects with spliced and unspliced count layers
- Batch-corrected count matrices (ComBat and scGen) for UniTVelo baselines
- Ground-truth dyngen trajectory/batch annotations for metric computation

## Outputs

- dyngen_bifurcation_batch_metrics.csv compiling kBET, iLISI, cLISI, CBDir, ICCoh, and nearest-neighbor velocity cosine similarity per method
- LatentVelo latent AnnData with spliced_velocity and velocity graph from scv.tl.velocity_graph
- Benchmark ordering confirming LatentVelo outperforms ComBat- and scGen-corrected UniTVelo on all metrics

## How to apply

Load the two-batch dyngen bifurcation dataset (example_bifurcation.h5ad) with scanpy and prepare it via ltv.utils.standard_clean_recipe with spliced_key='spliced', unspliced_key='unspliced', batch_key='batch'. Initialize ltv.models.VAE with batch_correction=True and the batch count, train with ltv.train, and export velocities with ltv.output_results. Run UniTVelo (unitvelo) as the baseline on (1) the unintegrated data, (2) ComBat-corrected counts, and (3) scGen-corrected counts. Compute cell-state integration metrics kBET, iLISI, and cLISI with scib, and velocity metrics CBDir, ICCoh, and nearest-neighbor velocity cosine similarity using scVelo velocity graphs for all methods. Compile results into dyngen_bifurcation_batch_metrics.csv and confirm LatentVelo scores best on every metric — the rationale is that the dynamics-based embedding batch corrects both cell states and velocities, whereas gene-space corrections that ignore expression dynamics fail to integrate the batches.

## Related tools

- **LatentVelo** (Dynamics-based latent model run with batch_correction=True to produce batch-corrected cell states and velocities) — https://github.com/Spencerfar/LatentVelo
- **UniTVelo** (Baseline RNA velocity method rerun on unintegrated, ComBat-corrected, and scGen-corrected matrices)
- **scGen** (Gene-space batch correction applied to counts before rerunning UniTVelo as a baseline)
- **scib** (Computes cell-state integration metrics kBET, iLISI, and cLISI)
- **scVelo** (Builds velocity graphs for nearest-neighbor velocity cosine similarity and velocity metrics)

## Examples

```
ltv.utils.standard_clean_recipe(adata, spliced_key='spliced', unspliced_key='unspliced', batch_key='batch'); model = ltv.models.VAE(observed=n_genes, latent_dim=20, zr_dim=2, h_dim=2, batch_correction=True, batches=2); epochs, val_ae, val_traj = ltv.train(model, adata)
```

## Evaluation signals

- LatentVelo scores best on every metric in dyngen_bifurcation_batch_metrics.csv (kBET, iLISI, cLISI, CBDir, ICCoh, NN velocity cosine)
- ComBat- and scGen-corrected UniTVelo baselines fail to properly integrate the batches on the two-batch simulated data
- LatentVelo velocity graph (scv.tl.velocity_graph on latent_adata with vkey='spliced_velocity') recovers the bifurcating dyngen topology in the latent space
- ltv.train returns decreasing validation autoencoder and trajectory reconstruction losses (val_ae, val_traj)

## Limitations

- LatentVelo assumes a single initial latent state from which all trajectories start, reachable by continuous trajectories
- Sparse cell clusters with few observed transient cells connecting trajectories can cause inferred trajectories to start from unexpected states (e.g., microglia in dentate gyrus)
- LatentVelo does not address stochastic dynamics or normalization; proposed fixes are SDEs and normalization factors as latent variables as in scVI
- Hyperparameter choice (latent dimension, cell-type annotated model, gene subsetting) remains heuristic

## Evidence
