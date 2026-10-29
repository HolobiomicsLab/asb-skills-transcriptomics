---
name: spatial-domain-recovery-in-snrnaseq
description: Use when you have single-cell/single-nucleus RNA-seq data lacking spatial
  coordinates and a trained CeLEry spatial-domain model (e.g., LIBD DLPFC sections),
  and need to assign each cell to a tissue layer or spatial domain.
when_to_use_negative:
- The query data are single-cell-resolution ST platform data with small targeted gene
  panels (MERFISH/MERSCOPE/Xenium-style panels) — CeLEry is suboptimal on small gene
  panels and awaits transcriptome-wide single-cell-resolution ST
- Cells already carry spatial coordinates or the goal is 2D coordinate recovery rather
  than categorical spatial domain / layer assignment
- No trained spatial-domain model exists for the relevant tissue and no ST training
  sections are available to train one
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CeLEry
  role: Supervised deep neural network (pip package CeLEryPy) that predicts the spatial
    domain/layer of each scRNA-seq cell from gene expression, with uncertainty estimates
  repo: github.com/QihuangZhang/CeLEry
- name: Scanpy
  role: Preprocessing of the snRNA-seq query (anndata loading, quality filtering,
    rank_genes_groups-based differentially expressed gene selection for spatial domain
    recovery)
- name: LIBD
  role: Source of the human DLPFC spatial transcriptomics training sections used for
    the pretrained cortical-layer model
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:6c574c85e709be02d6591fcbe31fac86ae97b4152ea5b9cb070c95d176a053ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CeLEry
  - LIBD
  - Scanpy
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spatial-domain-recovery-in-snrnaseq

## Summary

Recover the spatial domain (e.g., cortical layer L1-L6/WM) of dissociated snRNA-seq cells by applying a pretrained CeLEry supervised deep neural network that learned gene-expression-to-location relationships from spatial transcriptomics training sections, with per-cell uncertainty estimates.

## When to use

Use when you have single-cell/single-nucleus RNA-seq data lacking spatial coordinates and a trained CeLEry spatial-domain model (e.g., LIBD DLPFC sections), and need to assign each cell to a tissue layer or spatial domain.

## When NOT to use

- The query data are single-cell-resolution ST platform data with small targeted gene panels (MERFISH/MERSCOPE/Xenium-style panels) — CeLEry is suboptimal on small gene panels and awaits transcriptome-wide single-cell-resolution ST
- Cells already carry spatial coordinates or the goal is 2D coordinate recovery rather than categorical spatial domain / layer assignment
- No trained spatial-domain model exists for the relevant tissue and no ST training sections are available to train one

## Inputs

- snRNA-seq/scRNA-seq query dataset (e.g., AD postmortem middle frontal neocortex snRNA-seq, ~122,606 nuclei) as an anndata/scanpy object
- Pretrained CeLEry spatial domain model (e.g., trained on LIBD DLPFC sections 151673-151676)
- Gene set shared between query and training model (non-differentially expressed genes)

## Outputs

- Per-nucleus spatial domain / cortical layer assignment in {L1, L2, L3, L4, L5, L6, WM} with prediction uncertainty
- Layer-by-cell-type composition table (e.g., L1 dominated by astrocytes, oligodendrocytes enriched in L6/WM)
- Layer-by-condition composition (e.g., neuron fractions stratified by A+T-, A+T+, A-T-), exportable as e.g. AD_layer_composition.csv

## How to apply

Load the pretrained CeLEry model (e.g., trained on LIBD DLPFC sections 151673-151676) and the snRNA-seq query data with scanpy/anndata; eliminate cells with low UMI counts or few expressed genes, then normalize with cell-level log transformation of UMI counts followed by gene-level standardization (subtract mean, divide by standard deviation across cells), restricting to the non-differentially expressed gene set shared with the training model. Run the CeLEry network (ReLU hidden layers, Sigmoid output scaled to [0,1]) trained with a rank-consistent logistic regression loss to predict spatial domain membership for each nucleus. For cortical-layer recovery the output is a categorical label from {L1, L2, L3, L4, L5, L6, WM} plus an associated prediction uncertainty. Optionally enable the variational-autoencoder data augmentation on the training ST data (choosing replicate count by cross-validation) to improve robustness to scRNA-seq noise. Aggregate per-nucleus layer assignments by cell type to obtain layer-wise composition and stratify by condition (e.g., pathology groups A+T-, A+T+, A-T-) for downstream comparisons such as neuron depletion in deeper layers.

## Related tools

- **CeLEry** (Supervised deep neural network (pip package CeLEryPy) that predicts the spatial domain/layer of each scRNA-seq cell from gene expression, with uncertainty estimates) — github.com/QihuangZhang/CeLEry
- **Scanpy** (Preprocessing of the snRNA-seq query (anndata loading, quality filtering, rank_genes_groups-based differentially expressed gene selection for spatial domain recovery))
- **LIBD** (Source of the human DLPFC spatial transcriptomics training sections used for the pretrained cortical-layer model)

## Examples

```
pip install CeLEryPy; import CeLEry; pred = CeLEry.CeLEry.pretrainedTrainOrPredict(adata_query=sn_seq, predictModel='LIBD_SpatialDomain', spatialDomainPrediction=True)
```

## Evaluation signals

- Every predicted layer/domain label falls within the categorical output set {L1, L2, L3, L4, L5, L6, WM}
- Layer-wise composition matches known biology: L1 dominated by astrocytes, oligodendrocytes enriched in L6/WM
- Reproduced qualitative pattern matches Fig. 4 of the article: neuron depletion in deeper neocortical layers L5/L6 for A+T- vs A-T-, extending to L4/L5/L6 in A+T+
- Prediction uncertainty is recorded alongside every per-nucleus assignment
- Query genes are restricted to the non-differentially expressed gene set shared with the training model (no out-of-vocabulary genes fed to the network)

## Limitations

- CeLEry is suboptimal on single-cell-resolution platforms with small gene panels; performance improves when ST data with single-cell resolution and transcriptome-wide coverage are available
- CeLEry is a supervised method: it requires spatial transcriptomics training data with known spatial domains from the same tissue context
- The number of VAE-augmented training replicates is a user-specified hyperparameter that must be tuned via cross-validation rather than a fixed recommended setting
- Identifying gradient/metagene expression patterns in tissues remains unresolved, limiting anchor-free applicability

## Evidence

- [abstract] CeLEry can infer the spatial origins of cells in scRNA-seq at multiple levels, including 2D location and spatial domain of a cell, while providing uncertainty estimates: [evidence span withheld — anchor status: not_found]
- [methods] domain gene selection via Scanpy: "differentially expressed genes are selected using the rank_genes_groups function in the Scanpy package (for spatial domain recovery)"
- [methods] quality filtering rule: "cells with low UMI counts or a small number of expressed genes were eliminated"
- [other] network architecture and loss: [evidence span withheld — anchor status: not_found]
- [discussion] VAE augmentation rationale: "include an optional data augmentation step based on a variational autoencoder to generate ST data replicates"
- [discussion] panel limitation: "We believe CeLEry will perform even better when ST data with single-cell resolution and transcriptome-wide coverage become more widely available"
- [other] AD application finding: [evidence span withheld — anchor status: not_found]
- [readme] installation command: "pip install CeLEryPy"
