---
name: smfish-gene-imputation-evaluation
description: Use when integrating unpaired scRNA-seq with smFISH spatial data to impute
  and validate spatial expression patterns for genes measured in one modality but
  not the other, or to benchmark diagonal integration methods on held-out spatial
  gene predictions.
when_to_use_negative:
- Modalities lack cross-modality feature connections (e.g. electrophysiology vs gene
  expression), which scConfluence's rIOT cost matrix requires
- The task is paired/matched multiomic integration rather than diagonal (unpaired)
  integration, which scConfluence does not address
- Data contain very rare cell populations that are unlikely to be co-sampled in mini-batches,
  degrading rIOT alignment
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scConfluence
  role: 'Primary method: uncoupled autoencoders plus regularized Inverse Optimal Transport
    (Sinkhorn/rIOT) to learn shared latent embeddings for cross-modality gene imputation'
  repo: https://github.com/cantinilab/scconfluence
- name: Muon
  role: MuData object handling, QC filtering (filter_obs/filter_var), and normalization
    for the scRNA and smFISH modalities
- name: Scanpy
  role: Normalization (normalize_total, log1p) and plotting of imputed spatial gene
    patterns
- name: scipy
  role: scipy.spatial.distance.cdist to compute the cross-modality distance matrix
    on weakly connected gene features
- name: Seurat v3.0
  role: 'Baseline: anchor-based integration for comparison of imputation ranking'
- name: Liger
  role: 'Baseline: integrative NMF integration (rliger v1.0.0) for imputation comparison'
- name: MultiMAP
  role: Baseline diagonal integration method for imputation comparison
- name: Uniport
  role: Baseline optimal-transport-based integration method for imputation comparison
- name: scGLUE
  role: Baseline graph-linked integration method for imputation comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:db7b9a58bf83c3f52682625afeeaab31ad9cdcde8c1615bed47b6eb492448e41
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cantinilab/scconfluence
  repo_url: https://github.com/cantinilab/scconfluence
  tools:
  - Liger
  - MultiMAP
  - Muon
  - Scanpy
  - Seurat
  - Seurat v3.0
  - Uniport
  - scConfluence
  - scGLUE
  - scipy
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# smfish-gene-imputation-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the scConfluence scRNA-smFISH diagonal integration of the mouse somatosensory cortex, imputes spatial expression for genes absent from the smFISH panel (Scgn, Synpr, Olah), and evaluates imputation quality against five baselines via Spearman correlations on 11 held-out smFISH gene groups.

## When to use

Use when integrating unpaired scRNA-seq with smFISH spatial data to impute and validate spatial expression patterns for genes measured in one modality but not the other, or to benchmark diagonal integration methods on held-out spatial gene predictions.

## When NOT to use

- Modalities lack cross-modality feature connections (e.g. electrophysiology vs gene expression), which scConfluence's rIOT cost matrix requires
- The task is paired/matched multiomic integration rather than diagonal (unpaired) integration, which scConfluence does not address
- Data contain very rare cell populations that are unlikely to be co-sampled in mini-batches, degrading rIOT alignment

## Inputs

- Mouse somatosensory cortex Smartseq2 scRNA-seq counts (MuData modality)
- Mouse somatosensory cortex smFISH spatial expression data (MuData modality)
- Cross-modality gene feature linkage used to build the cdist cost matrix on weakly connected features

## Outputs

- Shared latent cell embeddings populated in mdata.obsm per modality
- Imputed spatial expression patterns for the 11 held-out smFISH gene groups, including Scgn, Synpr, and Olah
- Average and median Spearman correlations between imputed and measured smFISH spatial patterns for scConfluence and the five baselines
- Spatial pattern plots for Scgn, Synpr, and Olah (Scanpy/matplotlib), comparable to Fig. 4d-f

## How to apply

Load the mouse somatosensory cortex Smartseq2 scRNA-seq and smFISH datasets as modalities of a MuData object, then compute the cross-modality distance matrix on weakly connected gene features with scipy.spatial.distance.cdist and register it in mdata.uns under the cross_keys entry. Train a per-modality AutoEncoder (scconfluence.unimodal) and fit scConfluence with uncoupled autoencoders and Sinkhorn (rIOT) regularization using a latent dimension of 16, tuning sinkhorn_loss_weight/reach if modality embeddings separate in the shared latent space. Use the learned shared latent embeddings to impute expression for the 11 held-out smFISH gene groups. Re-run the identical integration with Seurat v3.0 anchors, Liger, MultiMAP, Uniport, and scGLUE, then compute average and median Spearman correlations between imputed and measured smFISH spatial patterns across all methods and plot imputed spatial patterns for Scgn, Synpr, and Olah with Scanpy/matplotlib.

## Related tools

- **scConfluence** (Primary method: uncoupled autoencoders plus regularized Inverse Optimal Transport (Sinkhorn/rIOT) to learn shared latent embeddings for cross-modality gene imputation) — https://github.com/cantinilab/scconfluence
- **Muon** (MuData object handling, QC filtering (filter_obs/filter_var), and normalization for the scRNA and smFISH modalities)
- **Scanpy** (Normalization (normalize_total, log1p) and plotting of imputed spatial gene patterns)
- **scipy** (scipy.spatial.distance.cdist to compute the cross-modality distance matrix on weakly connected gene features)
- **Seurat v3.0** (Baseline: anchor-based integration for comparison of imputation ranking)
- **Liger** (Baseline: integrative NMF integration (rliger v1.0.0) for imputation comparison)
- **MultiMAP** (Baseline diagonal integration method for imputation comparison)
- **Uniport** (Baseline optimal-transport-based integration method for imputation comparison)
- **scGLUE** (Baseline graph-linked integration method for imputation comparison)

## Examples

```
from scipy.spatial.distance import cdist; D = cdist(rna_genes, fish_genes, metric='correlation'); mdata.uns['cross_keys'] = ['rna','fish']
```

## Evaluation signals

- Average and median Spearman correlations between imputed and measured smFISH spatial patterns reproduce the scConfluence-vs-baseline ranking from the paper
- Imputed spatial patterns for Scgn, Synpr, and Olah visually match Fig. 4d-f of the paper
- Latent embeddings from all modalities co-localize by biological similarity in the 16-dimensional shared space; tune sinkhorn_loss_weight/reach if modality embeddings separate
- The cross-modality distance matrix computed with cdist is correctly registered in mdata.uns under the cross_keys entry before fitting
- Imputation succeeds for all 11 held-out smFISH gene groups without modality-specific cell types being lost (unbalanced OT handles cells absent in a modality)

## Limitations

- scConfluence relies on connections between features of different modalities; such connections are not always available (e.g. electrophysiology vs gene expression)
- Rare cell populations are hard to detect and under-represented in parameter estimation, especially with mini-batch gradient descent
- Integration of paired with unpaired multimodal data is not yet addressed by scConfluence
- A GPU is not required but strongly recommended per the README; hidden-layer sizes for the autoencoders are dataset-dependent (Supplementary Table 5)

## Evidence
