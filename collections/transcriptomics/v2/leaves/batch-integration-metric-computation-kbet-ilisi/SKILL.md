---
name: batch-integration-metric-computation-kbet-ilisi
description: Use when comparing velocity models across batches on example_bifurcation.h5ad
  and you need quantitative evidence that batch correction improved both cell-state
  mixing (kBET, iLISI, cLISI) and velocity fidelity (CBDir, ICCoh, NN velocity cosine).
when_to_use_negative:
- Single-batch datasets where batch correction metrics (kBET, iLISI, cLISI) are undefined
  or uninformative
- Evaluation questions about trajectory or latent-time accuracy rather than batch
  integration and velocity consistency across batches
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LatentVelo
  role: Batch-correcting velocity model being benchmarked; dynamics-based embedding
    batch corrects cell states and velocities
  repo: github.com/Spencerfar/LatentVelo
- name: UniTVelo
  role: Baseline velocity model run on unintegrated and ComBat/scGen-corrected data
- name: scGen
  role: Gene-space batch correction applied to counts before rerunning UniTVelo
- name: scib
  role: Computes cell-state integration metrics kBET, iLISI, and cLISI
- name: scVelo
  role: Computes velocity graphs (scv.tl.velocity_graph) for the nearest-neighbor
    velocity cosine similarity metric
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:a115240bed7f6104017bd3fcd3b44745046ca85897c9730ef6b12944039d837d
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
  - UniTVelo
  - scGen
  - scVelo
  - scib
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# batch-integration-metric-computation-kbet-ilisi

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify batch correction quality of RNA velocity methods on the two-batch dyngen bifurcation synthetic dataset using scib cell-state integration metrics (kBET, iLISI, cLISI) alongside velocity metrics (CBDir, ICCoh, nearest-neighbor velocity cosine similarity), to test whether LatentVelo's dynamics-based embedding outperforms gene-space corrected UniTVelo baselines.

## When to use

Use when comparing velocity models across batches on example_bifurcation.h5ad and you need quantitative evidence that batch correction improved both cell-state mixing (kBET, iLISI, cLISI) and velocity fidelity (CBDir, ICCoh, NN velocity cosine).

## When NOT to use

- Single-batch datasets where batch correction metrics (kBET, iLISI, cLISI) are undefined or uninformative
- Evaluation questions about trajectory or latent-time accuracy rather than batch integration and velocity consistency across batches

## Inputs

- example_bifurcation.h5ad (two-batch dyngen synthetic bifurcation dataset with spliced/unspliced layers and batch key)
- Velocity outputs from LatentVelo (latent_adata with spliced_velocity), UniTVelo on unintegrated data, and UniTVelo on ComBat- or scGen-corrected counts
- Batch labels (adata.obs['batch']) and cell-type/cluster labels for kBET, iLISI, and cLISI computation

## Outputs

- dyngen_bifurcation_batch_metrics.csv containing kBET, iLISI, cLISI, CBDir, ICCoh, and nearest-neighbor velocity cosine similarity for all three methods
- Per-method scVelo velocity graphs used for the nearest-neighbor velocity cosine metric

## How to apply

Load example_bifurcation.h5ad with scanpy and run three velocity methods: LatentVelo (ltv.utils.standard_clean_recipe with batch_key='batch', ltv.models.VAE with batch_correction=True, ltv.train, ltv.output_results), UniTVelo on unintegrated counts, and UniTVelo after ComBat or scGen (scgen 2.1.0) gene-space batch correction. For each of the three methods compute cell-state integration metrics kBET, iLISI, and cLISI with scib 1.0.3, and velocity metrics CBDir, ICCoh, and nearest-neighbor velocity cosine similarity using scv.tl.velocity_graph velocity graphs. Compile all metrics into a single table (dyngen_bifurcation_batch_metrics.csv) and verify the reported ordering in which LatentVelo scores best on every metric. The rationale is that LatentVelo's dynamics-based latent space batch corrects both cell states and velocities, whereas ComBat- and scGen-corrected UniTVelo fail to properly integrate the batches.

## Related tools

- **LatentVelo** (Batch-correcting velocity model being benchmarked; dynamics-based embedding batch corrects cell states and velocities) — github.com/Spencerfar/LatentVelo
- **UniTVelo** (Baseline velocity model run on unintegrated and ComBat/scGen-corrected data)
- **scGen** (Gene-space batch correction applied to counts before rerunning UniTVelo)
- **scib** (Computes cell-state integration metrics kBET, iLISI, and cLISI)
- **scVelo** (Computes velocity graphs (scv.tl.velocity_graph) for the nearest-neighbor velocity cosine similarity metric)

## Examples

```
ltv.utils.standard_clean_recipe(adata, spliced_key='spliced', unspliced_key='unspliced', batch_key='batch'); model = ltv.models.VAE(observed=n_genes, latent_dim=latent_dim, zr_dim=zr_dim, h_dim=h_dim, batch_correction=True, batches=2); ltv.train(model, adata); scib.metrics.kBET(...)
```

## Evaluation signals

- LatentVelo scores best on every metric (kBET, iLISI, cLISI, CBDir, ICCoh, NN velocity cosine) relative to unintegrated, ComBat-corrected, and scGen-corrected UniTVelo
- All six metrics for all three methods are present in dyngen_bifurcation_batch_metrics.csv with no missing cells
- ComBat- and scGen-corrected UniTVelo runs show failure to properly integrate the two batches (poor kBET/iLISI/cLISI)
- Velocity cosine similarities are computed from scVelo velocity graphs built on each method's velocity output with vkey='spliced_velocity' for LatentVelo

## Limitations

- LatentVelo assumes a single initial latent state from which all cell trajectories start, with each cell reachable by a continuous trajectory
- LatentVelo may encounter issues with sparse cell clusters with few observed transient cells connecting trajectories
- Hyperparameter choices (latent state dimension, cell-type annotated model, gene subsetting) may need adjustment for some datasets and only heuristics are discussed

## Evidence

- [abstract] The dynamics-based embedding batch corrects cell states and velocities, outperforming comparable autoencoder batch correction methods that do not consider gene expression dynamics: "The dynamics-based embedding batch corrects cell states and velocities, outperforming comparable autoencoder batch correction methods that do not consider gene expression dynamics"
- [intro] modeling the latent space dynamics enables batch correction: "modeling the latent space dynamics enables batch correction and the incorporation of additional information such as annotated cell-type labels"
- [methods] scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity'): "scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity')"
- [readme] Batch correction is enabled by specifying batch correction and the number of batches: "Batch correction is enabled by specifying batch correction and the number of batches for either model"
- [readme] Subdirectories of this directory contain code to run benchmarking with synthetic data or batch correction: "Subdirectories of this directory contain code to run benchmarking with synthetic data or batch correction"
