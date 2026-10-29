---
name: gene-space-batch-correction-combat-scgen
description: Use when evaluating RNA velocity models on multi-batch data and you need
  gene-space batch-corrected baselines (ComBat or scGen feeding UniTVelo) to compare
  against dynamics-based correction such as LatentVelo's batch_correction=True VAE.
when_to_use_negative:
- Single-batch datasets with no 'batch' key — gene-space batch correction and batch_comparison
  metrics are not applicable
- When you need batch correction of velocities, not just cell states — ComBat and
  scGen operate in gene space and 'do not consider gene expression dynamics'
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LatentVelo
  role: Dynamics-based batch-corrected velocity method (batch_correction=True VAE)
    against which ComBat/scGen gene-space baselines are benchmarked
  repo: https://github.com/Spencerfar/LatentVelo
- name: UniTVelo
  role: Velocity model rerun on unintegrated and ComBat-/scGen-corrected data to produce
    baseline velocities
- name: scGen
  role: Gene-space autoencoder batch correction applied to counts before rerunning
    UniTVelo (version 2.1.0 in the paper's environment)
- name: ComBat
  role: Gene-space batch correction applied to counts before rerunning UniTVelo
- name: scib
  role: Computes cell-state integration metrics kBET, iLISI, and cLISI for all batch-correction
    methods (version 1.0.3)
- name: scVelo
  role: Computes velocity graphs for NN velocity cosine similarity and CBDir/ICCoh
    velocity metrics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:5543b4277ddd660a6636a1c8dbb10aec38ec90762bd9162ba4723820f00227ce
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
  - ComBat
  - LatentVelo
  - UniTVelo
  - scGen
  - scVelo
  - scib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# gene-space-batch-correction-combat-scgen

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies gene-space batch correction (ComBat or scGen) to spliced/unspliced count matrices so that batch-corrected baselines (UniTVelo on corrected data) can be benchmarked against LatentVelo's dynamics-based latent-space correction using scib integration and scVelo velocity metrics.

## When to use

Use when evaluating RNA velocity models on multi-batch data and you need gene-space batch-corrected baselines (ComBat or scGen feeding UniTVelo) to compare against dynamics-based correction such as LatentVelo's batch_correction=True VAE.

## When NOT to use

- Single-batch datasets with no 'batch' key — gene-space batch correction and batch_comparison metrics are not applicable
- When you need batch correction of velocities, not just cell states — ComBat and scGen operate in gene space and 'do not consider gene expression dynamics'

## Inputs

- example_bifurcation.h5ad — two-batch dyngen bifurcation synthetic dataset with 'spliced' and 'unspliced' layers and a 'batch' key (AnnData)
- Raw count matrices amenable to ComBat or scGen correction in gene space

## Outputs

- ComBat- and scGen-corrected gene expression matrices (batch-corrected counts)
- UniTVelo velocity estimates on unintegrated, ComBat-corrected, and scGen-corrected data
- dyngen_bifurcation_batch_metrics.csv — kBET, iLISI, cLISI (scib) and CBDir, ICCoh, NN velocity cosine (scVelo) for all three methods
- LatentVelo latent_adata with spliced_velocity for velocity graph computation

## How to apply

Load the two-batch dyngen bifurcation dataset (example_bifurcation.h5ad) with scanpy, keeping the 'batch' key and spliced/unspliced layers intact. Apply ComBat or scGen gene-space batch correction directly to the count matrix (scGen 2.1.0 is listed among the paper's package versions), then run UniTVelo 0.1.5 on the corrected matrices to obtain baseline velocities. For comparison, prepare the raw data with ltv.utils.standard_clean_recipe(batch_key='batch'), initialize ltv.models.VAE with batch_correction=True and the number of batches, train with ltv.train, and export velocities with ltv.output_results. Evaluate all methods with scib 1.0.3 metrics (kBET, iLISI, cLISI) for cell-state integration and with CBDir, ICCoh, and nearest-neighbor velocity cosine similarity via scVelo velocity graphs (scv.tl.velocity_graph with vkey='spliced_velocity' for LatentVelo). Compile everything into a single metrics table (dyngen_bifurcation_batch_metrics.csv); the rationale is that gene-space correction ignores expression dynamics, whereas the dynamics-based embedding batch corrects both cell states and velocities.

## Related tools

- **LatentVelo** (Dynamics-based batch-corrected velocity method (batch_correction=True VAE) against which ComBat/scGen gene-space baselines are benchmarked) — https://github.com/Spencerfar/LatentVelo
- **UniTVelo** (Velocity model rerun on unintegrated and ComBat-/scGen-corrected data to produce baseline velocities)
- **scGen** (Gene-space autoencoder batch correction applied to counts before rerunning UniTVelo (version 2.1.0 in the paper's environment))
- **ComBat** (Gene-space batch correction applied to counts before rerunning UniTVelo)
- **scib** (Computes cell-state integration metrics kBET, iLISI, and cLISI for all batch-correction methods (version 1.0.3))
- **scVelo** (Computes velocity graphs for NN velocity cosine similarity and CBDir/ICCoh velocity metrics)

## Evaluation signals

- scib metrics kBET, iLISI, and cLISI are computed identically for the unintegrated, ComBat-corrected, and scGen-corrected runs so integration quality is comparable
- Velocity metrics CBDir, ICCoh, and nearest-neighbor velocity cosine similarity (scVelo velocity graphs) are computed for all three methods
- Expected ordering: LatentVelo scores best on every batch-correction and velocity metric in dyngen_bifurcation_batch_metrics.csv
- ComBat- and scGen-corrected UniTVelo runs should fail to properly integrate the two batches, confirming the gene-space baselines are correctly implemented as weak comparators

## Limitations

- Gene-space corrections (ComBat, scGen) ignore expression dynamics; they 'fail to properly integrate the batches' on the simulated two-batch dataset, so corrected UniTVelo scores poorly on both integration and velocity metrics
- LatentVelo's dynamics-based correction assumes a single initial latent state and continuous trajectories; it may mishandle sparse cell clusters with few transient connecting cells
- LatentVelo batch correction may require hyperparameter adjustment (larger latent_dim, celltype-annotated AnnotVAE, or gene subsetting) on some datasets

## Evidence
