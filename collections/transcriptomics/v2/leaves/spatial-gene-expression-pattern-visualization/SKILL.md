---
name: spatial-gene-expression-pattern-visualization
description: Use when a diagonal integration predicts expression for genes absent
  from one spatial modality (e.g. smFISH) and you need to inspect the imputed spatial
  patterns and quantify their agreement with held-out measurements.
when_to_use_negative:
- The gene of interest is already measured in the smFISH experiment, so its spatial
  pattern can be read directly without imputation
- No cross-modality feature connections exist between modalities (e.g. integrating
  electrophysiology with gene expression), since scConfluence relies on connections
  between features of different modalities
- The data are paired single-cell multimodal measurements; scConfluence targets diagonal
  (unpaired) integration and does not jointly integrate paired with unpaired data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scConfluence
  role: Learns shared latent embeddings via uncoupled autoencoders and Sinkhorn (rIOT)
    regularization used to impute spatial expression for genes not measured in smFISH
  repo: https://github.com/cantinilab/scconfluence
- name: Scanpy
  role: Visualization of imputed spatial expression patterns (plus normalize_total/log1p
    preprocessing)
- name: Muon
  role: Holds scRNA and smFISH modalities in a MuData object and provides filter_obs/filter_var
    for QC
- name: scipy
  role: scipy.spatial.distance.cdist computes the cross-modality gene distance matrix
    on weakly connected features
- name: Seurat v3.0
  role: Baseline anchor-based integration method for comparison of imputed spatial
    pattern accuracy
- name: Liger
  role: Baseline integration method (rliger v1.0.0) for comparison of imputed spatial
    pattern accuracy
- name: MultiMAP
  role: Baseline integration method for comparison of imputed spatial pattern accuracy
- name: Uniport
  role: Baseline integration method for comparison of imputed spatial pattern accuracy
- name: scGLUE
  role: Baseline integration method for comparison of imputed spatial pattern accuracy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:57b43f83e1c0a3f58590133a38a51d4f9a3ca76da0ddcea32310b633b7786805
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
  - C-SIDE
  - Liger
  - MultiMAP
  - Muon
  - SPARK-X
  - Scanpy
  - Seurat
  - Seurat v3.0
  - SpatialScope
  - Uniport
  - scConfluence
  - scGLUE
  - scipy
  - spacexr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# spatial-gene-expression-pattern-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize spatial expression patterns for genes predicted by scConfluence's cross-modality imputation — specifically genes measured in scRNA-seq but not in the smFISH assay (Scgn, Synpr, Olah) in the mouse somatosensory cortex — and benchmark imputation quality with average/median Spearman correlations against measured smFISH patterns.

## When to use

Use when a diagonal integration predicts expression for genes absent from one spatial modality (e.g. smFISH) and you need to inspect the imputed spatial patterns and quantify their agreement with held-out measurements.

## When NOT to use

- The gene of interest is already measured in the smFISH experiment, so its spatial pattern can be read directly without imputation
- No cross-modality feature connections exist between modalities (e.g. integrating electrophysiology with gene expression), since scConfluence relies on connections between features of different modalities
- The data are paired single-cell multimodal measurements; scConfluence targets diagonal (unpaired) integration and does not jointly integrate paired with unpaired data

## Inputs

- Mouse somatosensory cortex Smartseq2 scRNA-seq count matrix (QC-filtered, log-normalized to 10,000 counts then log1p)
- smFISH spatial expression matrix for the same tissue (with 11 held-out gene groups)
- MuData object with per-modality AnnData and cross-modality gene distance matrix registered in mdata.uns under the cross_keys entry
- Cross-modality distance matrix computed on weakly connected gene features with scipy.spatial.distance.cdist

## Outputs

- Imputed spatial expression patterns for held-out smFISH gene groups, plotted for Scgn, Synpr, and Olah with Scanpy/matplotlib
- Average and median Spearman correlations between imputed and measured smFISH spatial patterns across scConfluence and the five baselines
- scConfluence-vs-baseline ranking of imputation accuracy reproducing Fig. 4d-f spatial patterns

## How to apply

Load the mouse somatosensory cortex Smartseq2 scRNA-seq and smFISH datasets as modalities of a MuData object and register the cross-modality gene distance matrix (scipy.spatial.distance.cdist on weakly connected features) in mdata.uns under the cross_keys entry. Train per-modality AutoEncoders (scconfluence.unimodal) and fit scConfluence with uncoupled autoencoders and Sinkhorn (rIOT) regularization using a latent dimension of 16, tuning sinkhorn_loss_weight/reach if modality embeddings separate. Use the shared latent embeddings to impute expression for the 11 held-out smFISH gene groups, then plot the imputed spatial patterns for Scgn, Synpr, and Olah with Scanpy/matplotlib. Compute average and median Spearman correlations between imputed and measured smFISH spatial patterns, also running the same integration with Seurat v3.0 anchors, Liger, MultiMAP, Uniport, and scGLUE as baselines to confirm the scConfluence-vs-baseline ranking.

## Related tools

- **scConfluence** (Learns shared latent embeddings via uncoupled autoencoders and Sinkhorn (rIOT) regularization used to impute spatial expression for genes not measured in smFISH) — https://github.com/cantinilab/scconfluence
- **Scanpy** (Visualization of imputed spatial expression patterns (plus normalize_total/log1p preprocessing))
- **Muon** (Holds scRNA and smFISH modalities in a MuData object and provides filter_obs/filter_var for QC)
- **scipy** (scipy.spatial.distance.cdist computes the cross-modality gene distance matrix on weakly connected features)
- **Seurat v3.0** (Baseline anchor-based integration method for comparison of imputed spatial pattern accuracy)
- **Liger** (Baseline integration method (rliger v1.0.0) for comparison of imputed spatial pattern accuracy)
- **MultiMAP** (Baseline integration method for comparison of imputed spatial pattern accuracy)
- **Uniport** (Baseline integration method for comparison of imputed spatial pattern accuracy)
- **scGLUE** (Baseline integration method for comparison of imputed spatial pattern accuracy)

## Examples

```
from scipy.spatial.distance import cdist; D = cdist(rna_genes, fish_genes, metric='correlation'); mdata.uns['cross_keys'] = 'rna_smFISH_distances'
```

## Evaluation signals

- Average and median Spearman correlations between imputed and measured smFISH spatial patterns reproduce the scConfluence-vs-baseline ranking
- Plotted spatial patterns for Scgn, Synpr, and Olah match Fig. 4d-f of the paper
- scConfluence latent embeddings place cells of both modalities in a shared space without separation (tune sinkhorn_loss_weight/reach if modality embeddings separate)
- All 11 held-out smFISH gene groups are imputed using the learned shared latent embeddings
- The cross-modality distance matrix is correctly registered in mdata.uns under the cross_keys entry before model fitting

## Limitations

- Latent dimension was 16 for all datasets, but hidden layer sizes were dataset-dependent (Supplementary Table 5), so architectures may need adaptation for other dimensionalities
- Rare cell populations are hard to detect and under-represented in parameter estimation, especially with mini-batch gradient descent
- scConfluence requires cross-modality feature connections, which are unavailable for some modality pairs (e.g. electrophysiology vs gene expression)

## Evidence

- [methods] Unbalanced OT default transported mass: [evidence span withheld — anchor status: not_found]
