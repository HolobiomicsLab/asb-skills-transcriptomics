---
name: holdout-embedding-accuracy-evaluation
description: Use when a trained ProjectSVR ensemble SVR model (m=8000 subsample, t=10
  median-aggregated models) must be validated before release or reuse, by scoring
  held-out reference cells and checking prediction accuracy against reference embeddings.
when_to_use_negative:
- Evaluating projection of query cells whose gene expression comes from a different
  platform (e.g. Smart-seq2 to 10x) — performance declines due to differing UMI distributions
  and sequencing depth
- Query data may contain novel cell states absent from the reference — fixed-model
  projections may be misleading, so compare unsupervised clustering to mapping results
  instead
- The task is cell-type annotation rather than embedding accuracy — use ProjectSVR's
  ensemble SVM classifier (LabelTransfer) instead of SVR coordinate regression
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: ProjectSVR
  role: Provides FitEnsembleSVM to train the t=10 SVR models on m=8000 subsampled
    cells and ComputeModuleScore (method='AUCell') to produce gene-set scores for
    held-out cells
  repo: https://github.com/JarningGau/ProjectSVR
- name: ProjectSVR-benchmark
  role: Reference accuracy metrics (FigureS3 embeddings, Figure3 summary metrics)
    comparing ProjectSVR against Symphony, Seurat V4, ProjecTILs, scArches, online
    iNMF, SCALEX
  repo: https://github.com/JarningGau/ProjectSVR-benchmark
- name: AUCell
  role: Computes 0-1 rank-based gene activity scores robust to normalization, used
    as the regression features for held-out cells
- name: Seurat
  role: Stores the reference atlas and UMAP_1/2 embeddings used as SVR regression
    targets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:2c2c542c2af44429ff336dfd63422763591e5c0530fc1ce49530c58e4d59ee67
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
  - ProjectSVR-benchmark
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# holdout-embedding-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates a ProjectSVR ensemble-SVR reference mapping model by predicting low-dimensional reference embeddings (e.g. UMAP_1/2) for held-out reference cells and comparing predicted vs. reference coordinates, to confirm the gene-set-score-to-embedding regression recovers the reference structure.

## When to use

Use when a trained ProjectSVR ensemble SVR model (m=8000 subsample, t=10 median-aggregated models) must be validated before release or reuse, by scoring held-out reference cells and checking prediction accuracy against reference embeddings.

## When NOT to use

- Evaluating projection of query cells whose gene expression comes from a different platform (e.g. Smart-seq2 to 10x) — performance declines due to differing UMI distributions and sequencing depth
- Query data may contain novel cell states absent from the reference — fixed-model projections may be misleading, so compare unsupervised clustering to mapping results instead
- The task is cell-type annotation rather than embedding accuracy — use ProjectSVR's ensemble SVM classifier (LabelTransfer) instead of SVR coordinate regression

## Inputs

- Fitted ProjectSVR ensemble model (e.g. model.mTCA.rds from zenodo.org/record/8350732)
- Reference atlas Seurat object (e.g. mTCA.seurat.slim.qs from zenodo.org/record/8350746)
- Gene-set score matrix computed with AUCell (cNMF-derived K70 gene sets)
- Reference UMAP_1/2 embeddings as regression targets

## Outputs

- Predicted UMAP_1/2 coordinates for held-out reference cells (median over t=10 SVR sub-models)
- Correlation/error metrics between predicted and reference embeddings
- Accept/reject decision on the trained reference model

## How to apply

After fitting the ensemble with ProjectSVR::FitEnsembleSVM on the gene-set score matrix (AUCell scores of the cNMF-derived K70 gene sets) and the reference UMAP_1/2 coordinates, reserve a subset of reference cells not used during training. Score these held-out cells with ProjectSVR::ComputeModuleScore (method='AUCell'), pass them through the fitted model, and aggregate the t=10 per-dimension SVR predictions by taking the median for each embedding coordinate. Compare predicted vs. reference UMAP coordinates using correlation and error metrics on the held-out cells. Accept the model when held-out prediction accuracy is consistent with the paper's claim that ProjectSVR achieves comparable accuracy and robustness to state-of-the-art reference mapping methods (cf. Fig. 3, Table 2 benchmarks). The default ensemble uses m=8000 randomly subsampled cells and t=10 sampling iterations, chosen to balance performance and efficiency given the ~O(Nc^2) reference-building cost.

## Related tools

- **ProjectSVR** (Provides FitEnsembleSVM to train the t=10 SVR models on m=8000 subsampled cells and ComputeModuleScore (method='AUCell') to produce gene-set scores for held-out cells) — https://github.com/JarningGau/ProjectSVR
- **ProjectSVR-benchmark** (Reference accuracy metrics (FigureS3 embeddings, Figure3 summary metrics) comparing ProjectSVR against Symphony, Seurat V4, ProjecTILs, scArches, online iNMF, SCALEX) — https://github.com/JarningGau/ProjectSVR-benchmark
- **AUCell** (Computes 0-1 rank-based gene activity scores robust to normalization, used as the regression features for held-out cells)
- **Seurat** (Stores the reference atlas and UMAP_1/2 embeddings used as SVR regression targets)

## Examples

```
model <- ProjectSVR::FitEnsembleSVM(seu.ref, gene.sets = K70.sets); scores <- ProjectSVR::ComputeModuleScore(seu.heldout, method = "AUCell")
```

## Evaluation signals

- Held-out predicted vs. reference UMAP coordinates show correlation/error consistent with the paper's benchmark accuracy (Fig. 3, Table 2)
- Aggregation across the t=10 SVR sub-models uses the median (not the mean) per embedding coordinate
- Held-out cells were excluded from the m=8000 random subsample used to fit each sub-model
- Model passes when accuracy matches the claim of comparable accuracy and robustness to Seurat, scArches, Symphony, and ProjecTILs
- For downstream query mapping, low-quality projections are removed with thresholds such as mean.knn.dist < 1.6 or mapQ.p.adj < 0.1

## Limitations

- Independent SVR models per UMAP dimension neglect correlations among embedding dimensions, which may limit the model's ability to learn joint structures
- Reference building time is ~O(Nc^2); only the 4000-10,000 cell subsample regime was validated
- Performance declines when projecting across platforms (e.g. Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth

## Evidence

- [discussion] Independent per-dimension SVR limitation: [evidence span withheld — anchor status: not_found]
- [readme] Benchmark repository with reference/query embeddings and summary metrics: [evidence span withheld — anchor status: external_doc]
