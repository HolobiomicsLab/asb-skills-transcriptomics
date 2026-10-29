---
name: batch-mixing-quantification-kbet
description: Use when an scRNA-seq/velocity dataset spans multiple sequencing batches
  (e.g. 3 mouse gastrulation batches) and you need to verify that a latent embedding
  corrects batch effects without destroying velocity signal.
when_to_use_negative:
- Dataset is a single sequencing batch with no batch_key — kBET batch-mixing scores
  and cross-batch velocity cosine similarity are undefined
- Method does not produce a latent embedding or velocity field (e.g. plain clustering
  pipelines) — there is no batch-corrected latent space to score
- Sparse cell clusters with few observed transient cells connecting trajectories,
  where the article warns LatentVelo may encounter issues
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: LatentVelo
  role: Produces the dynamics-based batch-corrected latent embedding and velocities
    that are scored; batch correction is enabled at model initialization
  repo: github.com/Spencerfar/LatentVelo
- name: scib
  role: Computes kBET batch-mixing benchmark scores across sequencing batches
- name: scanpy
  role: Utility computations for nearest-neighbor velocity cosine similarity and benchmark
    metrics
- name: scVelo
  role: Builds the velocity graph (scv.tl.velocity_graph) on the 'spliced_velocity'
    key used for cross-batch velocity comparisons
- name: scANVI
  role: Baseline embedding method that LatentVelo's kBET batch mixing is compared
    against
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:c4e6597b65ba79b820ef5ac3baf75c373322eacdc3efb34b298ac66d035b2b37
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
  - scANVI
  - scVelo
  - scanpy
  - scib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Batch mixing quantification with kBET and velocity cosine similarity

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how well a dynamics-based latent embedding (e.g. LatentVelo with batch_correction=True) mixes sequencing batches, using kBET batch-mixing scores and nearest-neighbor velocity cosine similarity across batches, benchmarked against scVI, scANVI, and unintegrated data.

## When to use

Use when an scRNA-seq/velocity dataset spans multiple sequencing batches (e.g. 3 mouse gastrulation batches) and you need to verify that a latent embedding corrects batch effects without destroying velocity signal.

## When NOT to use

- Dataset is a single sequencing batch with no batch_key — kBET batch-mixing scores and cross-batch velocity cosine similarity are undefined
- Method does not produce a latent embedding or velocity field (e.g. plain clustering pipelines) — there is no batch-corrected latent space to score
- Sparse cell clusters with few observed transient cells connecting trajectories, where the article warns LatentVelo may encounter issues

## Inputs

- AnnData object with 'spliced' and 'unspliced' layers, batch_key='batch', celltype_key='celltype' (e.g. 20,000-cell mouse gastrulation subset)
- Trained LatentVelo model (ltv.models.AnnotVAE or ltv.models.VAE) with batch_correction=True
- Latent-space AnnData produced by ltv.output_results containing the 'spliced_velocity' key

## Outputs

- Per-batch kBET batch-mixing scores for LatentVelo vs. scVI, scANVI, and unintegrated data
- Nearest-neighbor velocity cosine similarity values between sequencing batches
- CBDir cross-boundary directionality transition scores across the 3 batches
- Assembled benchmark panels (e.g. figure5_panels.png)

## How to apply

Prepare the AnnData with ltv.utils.anvi_clean_recipe (spliced_key='spliced', unspliced_key='unspliced', batch_key='batch', celltype_key='celltype'), then initialize the AnnotVAE with batch_correction=True and batches equal to the number of sequencing batches (3 for the 20,000-cell mouse gastrulation subset), using experimental sequencing-time information to regularize latent dynamics. Train with ltv.train and export with ltv.output_results, then compute the Figure 5 benchmark metrics across the sequencing batches: CBDir cross-boundary directionality transition scores, kBET batch-mixing scores, and nearest-neighbor velocity cosine similarity, using scib and scanpy utilities. Compare the LatentVelo kBET score against scVI, scANVI, and the original unintegrated data; correct application is indicated by better batch mixing (kBET) than these baselines while velocities remain highly aligned between batches (high NN velocity cosine similarity).

## Related tools

- **LatentVelo** (Produces the dynamics-based batch-corrected latent embedding and velocities that are scored; batch correction is enabled at model initialization) — github.com/Spencerfar/LatentVelo
- **scib** (Computes kBET batch-mixing benchmark scores across sequencing batches)
- **scanpy** (Utility computations for nearest-neighbor velocity cosine similarity and benchmark metrics)
- **scVelo** (Builds the velocity graph (scv.tl.velocity_graph) on the 'spliced_velocity' key used for cross-batch velocity comparisons)
- **scANVI** (Baseline embedding method that LatentVelo's kBET batch mixing is compared against)

## Examples

```
model = ltv.models.AnnotVAE(observed=n_genes, latent_dim=70, zr_dim=6, h_dim=2, celltypes=n_celltypes, batch_correction=True, batches=3); epochs, val_ae, val_traj = ltv.train(model, adata, name='gastrulation')
```

## Evaluation signals

- LatentVelo kBET batch-mixing score is better (lower batch-associated kNN label mixing bias) than scVI, scANVI, and the original unintegrated data
- NN velocity cosine similarity between sequencing batches is high, indicating velocities are aligned after correction
- ltv.train returns decreasing/plateauing validation autoencoder (val_ae) and trajectory reconstruction (val_traj) losses
- scv.pl.velocity_embedding_stream on the 'spliced_velocity' key shows consistent lineage directions across batches on the latent UMAP

## Limitations

- LatentVelo assumes a single initial latent state from which all cell trajectories start, with each cell reachable by a continuous trajectory
- May encounter issues with sparse cell clusters with few observed transient cells connecting trajectories; latent trajectories may not start from expected initial states (e.g. dentate gyrus microglia)
- Hyperparameter choice (latent state dimension, cell-type-annotated model, gene subsetting) relies on heuristics; more work is needed

## Evidence

- [abstract] abstract: "The dynamics-based embedding batch corrects cell states and velocities, outperforming comparable autoencoder batch correction methods that do not consider gene expression dynamics."
- [intro] intro: "modeling the latent space dynamics enables batch correction and the incorporation of additional information such as annotated cell-type labels"
- [readme] readme-batch-init: "Batch correction is enabled by specifying batch correction and the number of batches for either model"
