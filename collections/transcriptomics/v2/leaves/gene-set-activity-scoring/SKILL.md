---
name: gene-set-activity-scoring
description: 'Use when projecting a scRNA-seq query onto a pre-built ProjectSVR reference
  model: the query must first be converted to gene set activity scores (via ComputeModuleScore
  with the model''s stored gss.method, e.g. AUCell or UCell) before ProjectNewdata/MapQuery
  can predict embeddings.'
when_to_use_negative:
- When the goal is de novo integration/clustering of query data rather than mapping
  onto a pre-built ProjectSVR reference model — scores only make sense as regression
  features tied to a specific model's gene.sets
- When the reference model was trained with AUCell but you score the query with UCell
  (or vice versa) — the query scoring method must match the model's stored gss.method
  or the SVR feature distribution breaks
- When projecting across platforms with differing UMI distributions (e.g. Smart-seq2
  to 10x), where projection performance is known to decline
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ProjectSVR
  role: Provides ComputeModuleScore wrapper that computes gene set activity scores
    for the query using the reference model's gene.sets and gss.method
  repo: github.com/jarninggau/ProjectSVR
- name: AUCell
  role: Default rank-based scoring engine computing 0–1 gene activity scores robust
    to normalization and usable without raw counts
- name: UCell
  role: Alternative external package for signature score calculation supported by
    ProjectSVR
  repo: https://github.com/carmonalab/UCell
- name: cNMF
  role: Derives gene sets for continuous cell states used as scoring signatures
  repo: https://github.com/dylkot/cNMF
- name: Seurat
  role: Hosts the query object whose assay receives the computed gene set activity
    scores
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:20df403cd84cc86642d59dcac8db28514ffb0edc870ce68da45599c0e6be7651
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jarninggau/ProjectSVR
  tools:
  - AUCell
  - ProjecTILs
  - ProjectSVR
  - Seurat
  - Symphony
  - UCell
  - cNMF
  - scArches
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# gene-set-activity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-cell gene set activity scores (0–1 scale) from a scRNA-seq query using AUCell or UCell, then uses these scores as regression features that ProjectSVR maps onto reference-embedding coordinates. This rank-based scoring step is what makes ProjectSVR projection platform-agnostic and independent of query normalization.

## When to use

Use when projecting a scRNA-seq query onto a pre-built ProjectSVR reference model: the query must first be converted to gene set activity scores (via ComputeModuleScore with the model's stored gss.method, e.g. AUCell or UCell) before ProjectNewdata/MapQuery can predict embeddings.

## When NOT to use

- When the goal is de novo integration/clustering of query data rather than mapping onto a pre-built ProjectSVR reference model — scores only make sense as regression features tied to a specific model's gene.sets
- When the reference model was trained with AUCell but you score the query with UCell (or vice versa) — the query scoring method must match the model's stored gss.method or the SVR feature distribution breaks
- When projecting across platforms with differing UMI distributions (e.g. Smart-seq2 to 10x), where projection performance is known to decline

## Inputs

- Query scRNA-seq dataset as a Seurat object (.qs or .rds, e.g. query_hPBMCs.seurat.slim.qs, query_Ythdc2-KO.seurat.slim.qs)
- Pre-trained ProjectSVR reference model (.rds, e.g. model.disco_pbmc.rds) carrying gene.sets and gss.method
- Cell-type/continuous-state gene sets derived from overexpression testing (markers with p_val_adj < 1e-6, avg_log2FC > log2(1.5)) or cNMF, top 25 genes per set

## Outputs

- Gene set activity score matrix (per-cell, 0–1 range) attached to the query Seurat object
- Input matrix consumed by ProjectSVR::ProjectNewdata to produce ref.umap embeddings
- Feature matrix for downstream label transfer (KnnLabelTransfer) or pseudotime regression (FitEnsembleSVM)

## How to apply

Load the pre-trained reference model (e.g. model.disco_pbmc.rds, model.mTCA.rds from Zenodo record 8350732) and read its stored gss.method to pick the scoring engine. Run ProjectSVR::ComputeModuleScore on the query Seurat object with the reference's gene.sets and the model's method (AUCell, which computes 0–1 scores based on rank expression and is robust to normalization differences and works on datasets lacking raw counts), typically with multi-core settings such as ncores = 10. Cell-type-specific gene sets are derived on the reference side from overexpression testing (for discrete types, filtering markers at p_val_adj < 1e-6 and avg_log2FC > log2(1.5)) or cNMF (for continuous states), with the top 25 genes per set used by default; results are consistent across gene-set-size settings. The resulting score matrix — not raw counts or a query-computed embedding — feeds ProjectNewdata, whose ensemble SVR sub-models predict each UMAP dimension, which is why scoring must replicate the reference's method exactly. For continuous-state references (e.g. mTCA pseudotime), the same gene-set-score inputs are used by FitEnsembleSVM to regress pseudotime.

## Related tools

- **ProjectSVR** (Provides ComputeModuleScore wrapper that computes gene set activity scores for the query using the reference model's gene.sets and gss.method) — github.com/jarninggau/ProjectSVR
- **AUCell** (Default rank-based scoring engine computing 0–1 gene activity scores robust to normalization and usable without raw counts)
- **UCell** (Alternative external package for signature score calculation supported by ProjectSVR) — https://github.com/carmonalab/UCell
- **cNMF** (Derives gene sets for continuous cell states used as scoring signatures) — https://github.com/dylkot/cNMF
- **Seurat** (Hosts the query object whose assay receives the computed gene set activity scores)

## Examples

```
seu.q <- ProjectSVR::ComputeModuleScore(seu.q, model, ncores = 10); seu.q <- ProjectSVR::ProjectNewdata(seu.q, model, reduction.name = "ref.umap")
```

## Evaluation signals

- Query scores computed with the same gss.method stored in the pre-trained model (AUCell for DISCO/MFI/mTCA workflows); mismatch invalidates the SVR feature space
- AUCell activity scores fall in the 0–1 range based on rank expression, verifiable by inspecting score min/max on the query object
- After scoring + projection, well-mapped query cells should cluster by biological identity: PBMC 10x-protocol batches show mean LISI ~2.30 post-mapping vs ~1.06 uncorrected baseline
- Label transfer after scoring/projection reaches accuracy and ARI above 0.92 on mTCA (query_Ythdc2-KO), comparable to de novo integration
- Pseudotime predicted from gene-set scores correlates with published values at r ≈ 0.973 for mTCA germ cells

## Limitations

- Gene set size is a hyperparameter; top 25 genes used by default, though results were consistent across different gene set size settings
- ProjectSVR performance declines when projecting across platforms (e.g. Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth
- Query cells containing novel states absent from the reference may receive misleading projections; compare unsupervised clustering to mapping results to detect inconsistencies

## Evidence

- [readme] External scoring packages required for ProjectSVR: [evidence span withheld — anchor status: external_doc]
