---
name: single-cell-gene-expression-dimensionality-reduction
description: Use when the input is a raw single-cell or tabular gene expression matrix
  (e.g., scRNA-seq count data, Slide-seq locations, BDTNP dge.
when_to_use_negative:
- The dataset contains data points from only one class (no contrast between data points)
  — CCSF then works similarly to kernel PCA and its only benefits are data linearization
  and computational speed
- A reference atlas or marker-gene-driven spatial method is explicitly required and
  a discriminative similarity-based reduction is not the goal
- The input is already a low-dimensional embedding or feature table rather than a
  gene-by-cell expression matrix
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: CCSF
  role: 'Core framework: learns the cell-cell similarity metric, computes cell-centroid
    similarity and its Gram matrix, and performs class-discriminative dimensionality
    reduction (cMap/cPHATE/cSpaRc entry points)'
  repo: github.com/xinglab-ai/ccsf
- name: novosparc
  role: Provides preprocessing utility subsample_dge for downsampling cells (e.g.,
    to 3039 of 3040) before CCSF spatial mapping
- name: Splatter
  role: Simulator used to generate 1,300 simulated datasets on which CCSF-PHATE-based
    dimensionality reduction is benchmarked against PHATE using the DeMAP index
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2023.100840
    title: Leveraging cell-cell similarity for high-performance spatial and temporal
      cellular mappings from gene expression data
  source_content_hash: sha256:c6c8cf11245276ea7b6da8e95d9a2b9ddbb7b6c7daec5723aac65d230477c469
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinglab-ai/ccsf
  tools:
  - CCSF
  - Splatter
  - novosparc
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# single-cell-gene-expression-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduce the dimensionality of single-cell gene expression count matrices using the CCSF (cell-cell similarity-driven framework), which learns a cell-cell similarity metric, computes cell-centroid similarity and its Gram matrix, and applies a class-discriminative optimization (linear) to produce a low-dimensional cMAP/cPHATE embedding that outperforms PCA and PHATE-style reductions.

## When to use

Use when the input is a raw single-cell or tabular gene expression matrix (e.g., scRNA-seq count data, Slide-seq locations, BDTNP dge.txt) and the goal is a discriminative low-dimensional embedding for visualization, clustering, or downstream spatial/temporal mapping, as a faster replacement for PCA.

## When NOT to use

- The dataset contains data points from only one class (no contrast between data points) — CCSF then works similarly to kernel PCA and its only benefits are data linearization and computational speed
- A reference atlas or marker-gene-driven spatial method is explicitly required and a discriminative similarity-based reduction is not the goal
- The input is already a low-dimensional embedding or feature table rather than a gene-by-cell expression matrix

## Inputs

- Single-cell gene expression count matrix (e.g., dge.txt with 84 columns of transcription factors for BDTNP/Drosophila, or organoidData.csv)
- Optional class/label vector (e.g., tcga_label.csv) for class-discriminative optimization and cMAP visualization coloring
- For spatial mapping, target-space coordinates (e.g., geometry.txt with 3D locations subset to selected cells)

## Outputs

- Low-dimensional CCSF embedding (cMAP or cPHATE coordinates, e.g., embedding[:,0] vs embedding[:,1] for 2D scatter)
- Learned similarity metric string (expected: 'correlation') and number of data classes (numClass)
- CCSF components used as a compact representation of the original high-dimensional expression data

## How to apply

Load the expression data as a count matrix and z-score it per gene with scipy.stats.zscore(axis=0, ddof=1). If the dataset has more than 2,000 data points, randomly sample 2,000 points (fixed rngState for reproducibility) and learn the similarity metric and number of classes via manifolder.metric_learning(dataSampled) — the README expects 'correlation' as the learned metric for the TCGA example, and numClass is analogous to choosing the number of principal components in PCA (e.g., 33 for TCGA/organoid data, 7 for BDTNP). Instantiate CCSF(n_clusters=numClass, num_comp=np.array([numClass-1]), metric=metric, random_state=rngState), locate the P centroids by minimizing distance to all data points, compute cell-centroid similarity, form the Gram matrix as the cell-cell similarity matrix, and run the class-discriminative optimization to reduce dimensionality. Call manifolder.cMap(data=feed_data) for visualization/clustering or manifolder.cPHATE(data=feed_data) for temporal mapping; the number of CCSF components used is numClass-1.

## Related tools

- **CCSF** (Core framework: learns the cell-cell similarity metric, computes cell-centroid similarity and its Gram matrix, and performs class-discriminative dimensionality reduction (cMap/cPHATE/cSpaRc entry points)) — github.com/xinglab-ai/ccsf
- **novosparc** (Provides preprocessing utility subsample_dge for downsampling cells (e.g., to 3039 of 3040) before CCSF spatial mapping)
- **Splatter** (Simulator used to generate 1,300 simulated datasets on which CCSF-PHATE-based dimensionality reduction is benchmarked against PHATE using the DeMAP index)

## Examples

```
from ccsf import CCSF; import numpy as np; manifolder = CCSF(n_clusters=33, num_comp=np.array([32]), metric='correlation', random_state=0); embedding_CMAP = manifolder.cMap(data={'data': data})
```

## Evaluation signals

- manifolder.metric_learning on the z-scored 2,000-point sample should return 'correlation' as the metric (per the TCGA example in the README)
- The number of CCSF components used equals numClass - 1 (e.g., 32 components when numClass=33)
- On Splatter simulations, the CCSF-based reduction (CCSF-PHATE) should outperform PHATE by more than 10% in most cases in DeMAP index, and in the ablation the linear class-discriminative variant should attain the highest DeMAP among linear, GDA-Gaussian, GDA-polynomial, QDA, MDS, and PCA variants
- Embedding scatter plots colored by label (e.g., plt.scatter(embedding[:,0], embedding[:,1], c=label.T, s=0.5)) should show separated cell classes/stages
- Reproducibility check: reruns with the same rngState (random_state=rngState, random.seed(a=rngState)) should yield near-identical embeddings; the paper reports <1% DEMaP change over 1,000 CCSF initializations on zebrafish data and no change on Drosophila data

## Limitations

- If the dataset contains data points from only one class, CCSF works similarly to kernel PCA and offers only data linearization and computational speed over PCA
- Choice of numClass is analogous to choosing the number of principal components in PCA and must be set or learned; numInitCls is derived as floor(data.shape[0]/500)
- Datasets with more than 2,000 data points use only 2,000 randomly selected points for learning the similarity metric in the first CCSF step
- Perfect spatial reconstruction may not hold when space geometry is complex (such as tissue cross-section) or gene expression count measurements are inaccurate

## Evidence

- [intro] Workflow: Gram matrix and dimensionality reduction steps: [evidence span withheld — anchor status: not_found]
