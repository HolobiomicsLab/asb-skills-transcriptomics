---
name: single-cell-reference-atlas-model-construction
description: Use when constructing a reference mapping model for a single-cell atlas
  (e.g. mTCA, DISCO PBMC, pan-cancer T cell landscape) by learning gene-set score
  → embedding relationships with ensemble SVR, to later project query scRNA-seq data
  without re-running integration.
when_to_use_negative:
- Query data may contain novel cell states absent from the reference — fixed-model
  projections may be misleading; compare unsupervised clustering to mapping results
  instead
- Projection across platforms (e.g. Smart-seq2 to 10x) — performance declines due
  to differing UMI distributions and sequencing depth
- You need a model capturing correlations among embedding dimensions — independent
  per-dimension SVR explicitly neglects joint structure (multi-output SVR is future
  work)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ProjectSVR
  role: Provides ComputeModuleScore (AUCell wrapper), FitEnsembleSVM ensemble training,
    LabelTransfer, and map-quality filtering for reference model construction
  repo: https://github.com/JarningGau/ProjectSVR
- name: AUCell
  role: Computes 0–1 rank-based gene activity scores used as regression features for
    the SVR ensemble
- name: cNMF
  role: Derives gene sets for continuous cell states via ProjectSVR's RunCNMF wrapper
    (installed through reticulate)
  repo: https://github.com/dylkot/cNMF
- name: Seurat
  role: Stores the reference atlas object and UMAP embeddings used as SVR targets
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:71df1c404650864217ea077ead39a3ba7ffd286ca585c1814d0077a3bdc4f810
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jarninggau/ProjectSVR
  repo_url: https://github.com/JarningGau/ProjectSVR
  tools:
  - AUCell
  - ProjectSVR
  - Seurat
  - cNMF
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-reference-atlas-model-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build a reusable ProjectSVR reference-mapping model by regressing a reference atlas's low-dimensional embeddings (UMAP_1/2) on AUCell gene-set activity scores with an ensemble of SVR sub-models (m=8000 subsampled cells, t=10 models, median aggregation). This decouples the model from any specific integration technique and yields a pre-trained .rds model that can map query cells platform-agnostically.

## When to use

Use when constructing a reference mapping model for a single-cell atlas (e.g. mTCA, DISCO PBMC, pan-cancer T cell landscape) by learning gene-set score → embedding relationships with ensemble SVR, to later project query scRNA-seq data without re-running integration.

## When NOT to use

- Query data may contain novel cell states absent from the reference — fixed-model projections may be misleading; compare unsupervised clustering to mapping results instead
- Projection across platforms (e.g. Smart-seq2 to 10x) — performance declines due to differing UMI distributions and sequencing depth
- You need a model capturing correlations among embedding dimensions — independent per-dimension SVR explicitly neglects joint structure (multi-output SVR is future work)

## Inputs

- Reference single-cell atlas as a Seurat object (e.g. mTCA.seurat.slim.qs from zenodo.org/record/8350746)
- Gene sets derived by overexpression testing or cNMF (e.g. K70 cNMF gene sets), top 25 genes per set by default
- Reference low-dimensional embedding (UMAP_1/2 cell embeddings) used as regression targets

## Outputs

- Trained ProjectSVR reference model (.rds), e.g. model.mTCA.rds distributable via Zenodo
- Per-dimension ensemble predictions aggregated by median across t=10 SVR sub-models
- Held-out evaluation metrics (predicted vs. reference UMAP correlation/error)

## How to apply

First derive cell-type/continuous-state gene sets from the reference atlas — overexpression testing for discrete types or cNMF for continuous states, keeping the top 25 genes per set by default (results were consistent across gene set sizes). Compute a gene-set score matrix for all reference cells with ProjectSVR::ComputeModuleScore using method='AUCell', which computes rank-based 0–1 gene activity scores robust to normalization. Fit the ensemble with ProjectSVR::FitEnsembleSVM on the score matrix and UMAP_1/2 coordinates: train t=10 independent SVR models (e1071 svm defaults: C=1, epsilon=0.1, radial kernel), each on m=8000 randomly subsampled cells, and aggregate per-dimension predictions by median — reference building time is ~O(Nc^2), so subsampling to 4000–10,000 cells balances performance and efficiency. Validate by comparing predicted vs. reference UMAP coordinates on held-out cells (correlation/error metrics) before saving the model as an .rds artifact (e.g. model.mTCA.rds, model.disco_pbmc.rds).

## Related tools

- **ProjectSVR** (Provides ComputeModuleScore (AUCell wrapper), FitEnsembleSVM ensemble training, LabelTransfer, and map-quality filtering for reference model construction) — https://github.com/JarningGau/ProjectSVR
- **AUCell** (Computes 0–1 rank-based gene activity scores used as regression features for the SVR ensemble)
- **cNMF** (Derives gene sets for continuous cell states via ProjectSVR's RunCNMF wrapper (installed through reticulate)) — https://github.com/dylkot/cNMF
- **Seurat** (Stores the reference atlas object and UMAP embeddings used as SVR targets) — https://github.com/satijalab/seurat

## Examples

```
devtools::install_github("JarningGau/ProjectSVR"); scores <- ProjectSVR::ComputeModuleScore(seu.ref, method = "AUCell")
```

## Evaluation signals

- Held-out reference cells show predicted UMAP coordinates correlating with reference embeddings at accuracy consistent with ProjectSVR's claimed parity with state-of-the-art mapping methods
- Ensemble hyperparameters match defaults: m=8000 subsampled cells per model, t=10 SVR sub-models (e1071 radial kernel, C=1, epsilon=0.1)
- Per-embedding-coordinate outputs equal the median of the 10 sub-model predictions (not the mean)
- Results are stable across gene set size settings with the top 25 genes used by default
- Trained model reproduces known benchmark behavior against Seurat, scArches, Symphony, and ProjecTILs (JarningGau/ProjectSVR-benchmark)

## Limitations

- Reference building time is ~O(Nc^2), requiring subsampling to 4000–10,000 cells for scalability
- Independent SVR models per UMAP dimension neglect potential correlations among embedding dimensions
- Projections may be misleading for query cells containing novel states absent in the reference; users should compare unsupervised clustering to mapping results
- Performance declines when projecting across platforms (e.g. Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth
- Software version numbers for ProjectSVR and its dependencies are not stated

## Evidence

- [methods] ensemble hyperparameters: [evidence span withheld — anchor status: not_found]
- [discussion] per-dimension SVR: [evidence span withheld — anchor status: not_found]
