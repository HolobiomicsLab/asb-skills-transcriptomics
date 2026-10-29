---
name: batch-correction-model-conditioning
description: Use when spliced/unspliced AnnData contain multiple sequencing batches
  and you need batch-corrected RNA velocities and latent cell states that preserve
  gene expression dynamics, e.g. with ltv.models.VAE(batch_correction=True, batches=n).
when_to_use_negative:
- The dataset contains a single sequencing batch — batch conditioning adds unnecessary
  complexity with no correction benefit
- You need batch correction of static expression embeddings without regard to spliced/unspliced
  gene dynamics — a dynamics-agnostic autoencoder correction method is the (weaker)
  alternative the article benchmarks against
- Cells come from sparse clusters with few observed transient cells connecting trajectories
  — the article notes latent trajectories may then not start from expected initial
  states
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: LatentVelo
  role: Provides ltv.models.VAE and ltv.models.AnnotVAE with batch_correction and
    batches arguments, plus ltv.utils.standard_clean_recipe/anvi_clean_recipe and
    ltv.train to fit batch-conditioned latent dynamics
  repo: https://github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Computes and plots velocity graphs/streamlines (vkey='spliced_velocity') on
    the batch-corrected latent_adata output
- name: AnnData
  role: Holds the spliced/unspliced layers and batch/celltype keys consumed by the
    LatentVelo recipes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:5095c1c738986e19a260f1b7ddc94f5a232b77e5244b1f77f93616b7d20a53b7
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
  - AnnData
  - LatentVelo
  - PyTorch
  - scVelo
  - scanpy
  - torchdiffeq
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# batch-correction-model-conditioning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Enable RNA velocity batch correction in LatentVelo by conditioning the structured latent neural-ODE model on sequencing batch (batch_correction=True with a batches argument in ltv.models.VAE or ltv.models.AnnotVAE), so that the dynamics-based latent space yields batch-corrected cell states and velocities rather than correcting expression alone.

## When to use

Use when spliced/unspliced AnnData contain multiple sequencing batches and you need batch-corrected RNA velocities and latent cell states that preserve gene expression dynamics, e.g. with ltv.models.VAE(batch_correction=True, batches=n).

## When NOT to use

- The dataset contains a single sequencing batch — batch conditioning adds unnecessary complexity with no correction benefit
- You need batch correction of static expression embeddings without regard to spliced/unspliced gene dynamics — a dynamics-agnostic autoencoder correction method is the (weaker) alternative the article benchmarks against
- Cells come from sparse clusters with few observed transient cells connecting trajectories — the article notes latent trajectories may then not start from expected initial states

## Inputs

- AnnData object with spliced and unspliced count layers and a batch key (prepared via ltv.utils.standard_clean_recipe or ltv.utils.anvi_clean_recipe)
- Number of batches (batches argument) and, for AnnotVAE, number of celltypes
- Hyperparameters: observed genes, latent_dim, zr_dim (latent regulation dimension), h_dim (conditioning dimension)

## Outputs

- Trained LatentVelo model with batch-conditioned latent dynamics
- epochs, val_ae, and val_traj validation losses returned by ltv.train
- latent_adata (and optionally adata with gene velocities) from ltv.output_results for downstream scv.tl.velocity_graph stream plotting

## How to apply

Prepare the multi-batch AnnData with ltv.utils.standard_clean_recipe (or ltv.utils.anvi_clean_recipe for the cell-type-annotated variant), passing the batch_key so batch labels are retained. Initialize the model with batch_correction=True and batches=number_of_batches, either as ltv.models.VAE or ltv.models.AnnotVAE (the annotated variant additionally takes celltypes=number_of_celltypes), keeping latent_dim, zr_dim, and h_dim to structure the latent space into zu, zs, and the lineage-regulating zr. Train with ltv.train(model, adata, batch_size, epochs, name), which returns epochs plus validation autoencoder (val_ae) and trajectory (val_traj) reconstruction losses used to monitor fit. The rationale is that modeling dynamics on the latent space lets the embedding itself batch-correct cell states and velocities, which the article reports outperforms comparable autoencoder batch correction methods that ignore gene expression dynamics; results are exported with ltv.output_results and velocities computed with scv.tl.velocity_graph on the latent space.

## Related tools

- **LatentVelo** (Provides ltv.models.VAE and ltv.models.AnnotVAE with batch_correction and batches arguments, plus ltv.utils.standard_clean_recipe/anvi_clean_recipe and ltv.train to fit batch-conditioned latent dynamics) — https://github.com/Spencerfar/LatentVelo
- **scVelo** (Computes and plots velocity graphs/streamlines (vkey='spliced_velocity') on the batch-corrected latent_adata output)
- **AnnData** (Holds the spliced/unspliced layers and batch/celltype keys consumed by the LatentVelo recipes)

## Examples

```
model = ltv.models.VAE(observed=n_genes, latent_dim=20, zr_dim=2, h_dim=2, batch_correction=True, batches=n_batches); epochs, val_ae, val_traj = ltv.train(model, adata, batch_size=512, epochs=500, name='batch_corrected')
```

## Evaluation signals

- ltv.train runs to completion and returns finite val_ae and val_traj losses on the multi-batch dataset
- latent_adata from ltv.output_results is a valid AnnData with a 'spliced_velocity' vkey usable by scv.tl.velocity_graph
- Velocity streamlines colored by latent_time show consistent trajectories across batch labels rather than batch-segregated structure
- Benchmark comparison (as in the article) shows corrected cell states/velocities outperforming dynamics-agnostic autoencoder batch correction baselines such as scgen/scIB workflows

## Limitations

- The dynamics-based embedding assumes a single initial latent state from which all cell trajectories start and continuous reachability; disconnected clusters (e.g. microglia in the dentate gyrus) can be incorrectly assigned the lowest latent times and seed trajectories
- Sparse cell clusters with few transient connecting cells may yield trajectories that do not start from expected initial states even when velocities are generally accurate
- Some datasets require additional hyperparameter adjustment (larger latent state dimension, cell-type annotated model, or gene subsetting), and only heuristics exist for hyperparameter choice

## Evidence

- [abstract] The dynamics-based embedding batch corrects cell states and velocities, outperforming comparable autoencoder batch correction methods that do not consider gene expression dynamics.: "The dynamics-based embedding batch corrects cell states and velocities, outperforming comparable autoencoder batch correction methods that do not consider"
- [readme] README: enabling batch correction with batch_correction=True and the number of batches: "Batch correction is enabled by specifying batch correction and the number of batches for either model"
- [readme] README: model constructor with batch correction: "model = ltv.models.VAE(observed = number_of_genes, latent_dim = latent_dimension,"
- [intro] modeling the latent space dynamics enables batch correction and the incorporation of additional information such as annotated cell-type labels: "modeling the latent space dynamics enables batch correction and the incorporation of additional information such as annotated cell-type labels"
