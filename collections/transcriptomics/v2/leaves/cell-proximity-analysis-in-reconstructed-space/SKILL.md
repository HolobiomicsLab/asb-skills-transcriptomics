---
name: cell-proximity-analysis-in-reconstructed-space
description: Use when scRNA-seq cells have been projected into pseudo-space via scSpace
  and clustered, and you must test whether a subpopulation near a reference cell type
  (e.g., malignant cells) shows a distinct functional signature such as T cell exhaustion.
when_to_use_negative:
- You want spatial domain identification of ST data itself — scSpace (and this proximity
  analysis) targets spatial analysis of scRNA-seq data, not SpaGCN/BayesSpace-style
  ST domain calling
- The tissue has low spatial coherence (e.g., Patients 5 and 9 of SCC), where the
  article advises applying scSpace cautiously and pseudo-space proximity inference
  would be unreliable
- Cells have not yet been projected into pseudo-space coordinates and clustered —
  proximity and signature comparisons require the space-informed Leiden clusters first
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scSpace
  role: Reconstructs cells into pseudo-space via TCA batch-effect removal and MLP
    projection, providing the coordinates in which proximity is computed
  repo: https://github.com/ZJUFanLab/scSpace
- name: scanpy
  role: Scores the T-cell exhaustion gene signature per cell and provides neighbors/clustering
    utilities for space-informed clustering
  repo: https://github.com/scverse/scanpy
- name: leidenalg
  role: Space-informed Leiden clustering that partitions projected T cells into subpopulations
    (C1–C5) prior to proximity comparison
- name: matplotlib
  role: Visualization of exhaustion-score and proximity comparisons reproducing Fig.
    5c–f panels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:c15e3634f824f575739a441ce80641aa2608042ef43e65876e3e27db0c4debdc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - igraph
  - leidenalg
  - matplotlib
  - pandas
  - scSpace
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# cell-proximity-analysis-in-reconstructed-space

## Summary

Quantify pairwise spatial associations between reconstructed cell subpopulations by comparing exhaustion scores and proximity to reference cell types (e.g., malignant cells) in the pseudo-space coordinates produced by scSpace, to identify tissue-infiltrating identities such as tumor-infiltrating lymphocytes.

## When to use

Use when scRNA-seq cells have been projected into pseudo-space via scSpace and clustered, and you must test whether a subpopulation near a reference cell type (e.g., malignant cells) shows a distinct functional signature such as T cell exhaustion.

## When NOT to use

- You want spatial domain identification of ST data itself — scSpace (and this proximity analysis) targets spatial analysis of scRNA-seq data, not SpaGCN/BayesSpace-style ST domain calling
- The tissue has low spatial coherence (e.g., Patients 5 and 9 of SCC), where the article advises applying scSpace cautiously and pseudo-space proximity inference would be unreliable
- Cells have not yet been projected into pseudo-space coordinates and clustered — proximity and signature comparisons require the space-informed Leiden clusters first

## Inputs

- Pseudo-space coordinates and cluster labels for scRNA-seq cells (e.g., 2064 melanoma T cells, five clusters C1–C5)
- Gene expression count matrix (preprocessed scRNA-seq T-cell .csv loaded with pandas)
- Malignant-cell positions in the reconstructed pseudo-space (spatial reference from melanoma ST data)

## Outputs

- Per-subpopulation gene-signature scores (e.g., T-cell exhaustion score per cell via scanpy.scoring)
- C5-vs-C3 exhaustion score comparison (Fig. 5c–d panels)
- Per-cluster proximity-to-malignant-cell metrics and cross-cluster comparison (Fig. 5e–f panels)
- Output table containing pseudo-space coordinates, five-cluster labels for all 2064 T cells, and the C5-vs-C3 comparisons

## How to apply

After scSpace's TCA extracts shared characteristic matrices and the MLP projects cells (e.g., 2064 melanoma T cells) into pseudo-space, jointly embed gene expression and spatial weights and run space-informed Leiden clustering (leidenalg/igraph) to define subpopulations (C1–C5). Score each cell for a functional signature — for melanoma T cells, the T-cell exhaustion gene signature via scanpy — and compare scores between the subpopulation spatially proximal to malignant cells (C5) and a distant one (C3). In parallel, compute each subpopulation's proximity to malignant cells in the reconstructed pseudo-space and compare across clusters. Conclude tumor-infiltrating lymphocyte identity only when the proximal cluster shows both higher exhaustion score and higher malignant-cell proximity, matching the paper's Fig. 5c–f comparisons.

## Related tools

- **scSpace** (Reconstructs cells into pseudo-space via TCA batch-effect removal and MLP projection, providing the coordinates in which proximity is computed) — https://github.com/ZJUFanLab/scSpace
- **scanpy** (Scores the T-cell exhaustion gene signature per cell and provides neighbors/clustering utilities for space-informed clustering) — https://github.com/scverse/scanpy
- **leidenalg** (Space-informed Leiden clustering that partitions projected T cells into subpopulations (C1–C5) prior to proximity comparison)
- **matplotlib** (Visualization of exhaustion-score and proximity comparisons reproducing Fig. 5c–f panels)

## Examples

```
import scspace; adata = scspace.tl.project(counts_csv='tcells.csv', st_ref='melanoma_ST.csv'); sc.pp.neighbors(adata); sc.tl.leiden(adata, resolution=1); sc.tl.score_genes(adata, exhaustion_genes)
```

## Evaluation signals

- Output table contains pseudo-space coordinates and five-cluster labels for all 2064 T cells
- C5 (proximal to malignant cells) shows higher exhaustion score than C3 (distant), matching Fig. 5c–d
- C5 also shows higher proximity to malignant cells than C3 in pseudo-space, matching Fig. 5e–f
- Clustering yields exactly five subpopulations (C1–C5) via space-informed Leiden clustering

## Limitations

- Choice of spatial reference directly influences the spatial reconstruction, so reference appropriateness must be assessed before trusting proximity conclusions
- Intrinsic differences in spatial homogeneity between tissues may influence the accuracy of scSpace, undermining proximity-based identity assignment
- scSpace should be applied cautiously to tissues with low spatial coherence (e.g., Patients 5 and 9 of SCC)

## Evidence

- [abstract] rationale for proximity + signature identity call: "The application of scSpace in melanoma and COVID-19 exhibits a broad prospect in the discovery of spatial therapeutic markers."
- [methods] pseudo-space construction prerequisite: "project single cells into a pseudo space via a [multiple layer perceptron (MLP)](https://en.wikipedia.org/wiki/Multilayer_perceptron) model"
- [methods] joint embedding for cell typing before proximity analysis: "so that gene expression and spatial weight of cells can be embedded jointly for the further cell typing with higher accuracy and precision"
- [readme] melanoma T-cell tutorial supporting this exact analysis: "4. [Spatial analysis of T cells subpopulations in melanoma](vignettes/melanoma.ipynb)"
