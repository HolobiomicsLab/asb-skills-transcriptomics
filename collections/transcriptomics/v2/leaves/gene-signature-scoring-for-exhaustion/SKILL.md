---
name: gene-signature-scoring-for-exhaustion
description: Use when a cell population (e.g., 2064 melanoma T cells) has been partitioned
  into subpopulations (C1–C5) in reconstructed pseudo-space and you must test whether
  the subpopulation nearest malignant cells carries a higher functional signature
  score (exhaustion) than a distant one.
when_to_use_negative:
- Tissues with low spatial coherence (e.g., SCC Patients 5 and 9 in the paper), where
  pseudo-space reconstruction—and hence spatially-informed exhaustion comparisons—are
  unreliable
- Data that has not been partitioned into spatially informed subpopulations; exhaustion
  scoring alone cannot establish TIL identity without the spatial proximity context
- Spatial reference data inappropriate for the target tissue (reference choice directly
  influences the spatial reconstruction), e.g., a granular DLPFC reference for agranular
  cortex lacking layer 4
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scanpy
  role: Gene-signature (T-cell exhaustion) scoring of cells and neighbors/Leiden clustering
    support for the comparison
  repo: https://github.com/scverse/scanpy
- name: scSpace
  role: Upstream tool providing TCA integration, MLP pseudo-space projection, and
    space-informed clustering whose C1–C5 labels are compared by exhaustion score
  repo: https://github.com/ZJUFanLab/scSpace
- name: matplotlib
  role: Visualization of the C5-vs-C3 exhaustion scores and malignant-cell proximity
    panels (Fig. 5c-f analogs)
  repo: https://pypi.org/project/matplotlib/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:9d6dc86b7b1f8259fbe433714108eca8610500fcc9fa6e16f169b8d695f0df68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/scanpy
  tools:
  - igraph
  - leidenalg
  - matplotlib
  - pandas
  - scSpace
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Gene-signature scoring for T-cell exhaustion

## Summary

Score each cell in a pseudo-space-clustered scRNA-seq population against a T-cell exhaustion gene signature using scanpy, then compare scores between spatial subpopulations (e.g., melanoma C5 vs C3) to identify tumor-infiltrating lymphocyte (TIL) identity.

## When to use

Use when a cell population (e.g., 2064 melanoma T cells) has been partitioned into subpopulations (C1–C5) in reconstructed pseudo-space and you must test whether the subpopulation nearest malignant cells carries a higher functional signature score (exhaustion) than a distant one.

## When NOT to use

- Tissues with low spatial coherence (e.g., SCC Patients 5 and 9 in the paper), where pseudo-space reconstruction—and hence spatially-informed exhaustion comparisons—are unreliable
- Data that has not been partitioned into spatially informed subpopulations; exhaustion scoring alone cannot establish TIL identity without the spatial proximity context
- Spatial reference data inappropriate for the target tissue (reference choice directly influences the spatial reconstruction), e.g., a granular DLPFC reference for agranular cortex lacking layer 4

## Inputs

- Preprocessed melanoma scRNA-seq T-cell count matrix (2064 T cells, from the tutorial Google Drive data package)
- Pseudo-space coordinates and Leiden cluster labels (C1–C5) from scSpace space-informed clustering
- T-cell exhaustion gene signature gene set
- Pseudo-space coordinates of malignant cells for proximity computation

## Outputs

- Per-cell T-cell exhaustion signature scores (scanpy)
- C5-vs-C3 exhaustion-score comparison (Fig. 5c-d analog, visualized with matplotlib)
- Per-cluster spatial proximity-to-malignant-cells comparison (Fig. 5e-f analog)
- Output table with pseudo-space coordinates, five-cluster labels for all 2064 T cells, and score/proximity comparisons

## How to apply

After scSpace's TCA batch-effect removal, MLP pseudo-space projection, and space-informed Leiden clustering of the melanoma T cells into five clusters (C1–C5), run scanpy gene-signature scoring on each cell with the T-cell exhaustion gene signature over the preprocessed T-cell count matrix. Aggregate scores per cluster and compare the exhaustion score of the malignant-cell-proximal subpopulation (C5) against the distant subpopulation (C3), reproducing the paper's Fig. 5c-d panels. In parallel, compute spatial proximity of each subpopulation to malignant cells in the pseudo-space to confirm that high exhaustion coincides with spatial proximity (Fig. 5e-f). The rationale is that spatially proximal, exhaustion-high T cells are candidate tumor-infiltrating lymphocytes, a distinction traditional single-cell pipelines miss because of transcriptome similarity.

## Related tools

- **scanpy** (Gene-signature (T-cell exhaustion) scoring of cells and neighbors/Leiden clustering support for the comparison) — https://github.com/scverse/scanpy
- **scSpace** (Upstream tool providing TCA integration, MLP pseudo-space projection, and space-informed clustering whose C1–C5 labels are compared by exhaustion score) — https://github.com/ZJUFanLab/scSpace
- **matplotlib** (Visualization of the C5-vs-C3 exhaustion scores and malignant-cell proximity panels (Fig. 5c-f analogs)) — https://pypi.org/project/matplotlib/

## Examples

```
import scanpy as sc; adata.obs['exhaustion_score'] = sc.tl.score_genes(adata, gene_list=exhaustion_signature, score_name='exhaustion_score') and then compare adata.obs.groupby('leiden')['exhaustion_score'].mean() between C5 and C3
```

## Evaluation signals

- Output table contains pseudo-space coordinates and five-cluster labels for all 2064 melanoma T cells
- C5 subpopulation shows higher T-cell exhaustion score than C3, matching the paper's Fig. 5c-d finding
- C5 subpopulation shows higher spatial proximity to malignant cells than C3 in the reconstructed pseudo-space, matching Fig. 5e-f
- Exhaustion score distribution is per-cell (scanpy) and aggregated consistently per cluster before comparison
- TCA batch-effect removal produced shared characteristic matrices for both scRNA-seq and ST modalities before clustering, so scores are computed on integrated data

## Limitations

- The exhaustion score is only interpretable as TIL identity when coupled with spatial proximity to malignant cells; transcriptome similarity makes spatially heterogeneous subpopulations hard to identify by traditional single-cell analysis pipelines
- Spatial references derived from different biological replicates (as in melanoma) can affect reconstruction, though the paper reports scSpace performance is conserved in this case
- Intrinsic differences in spatial homogeneity between tissues may influence accuracy, so the exhaustion-vs-proximity conclusion is tissue-dependent

## Evidence

- [methods] C5-vs-C3 exhaustion comparison finding: [evidence span withheld — anchor status: not_found]
- [readme] joint embedding rationale: "so that gene expression and spatial weight of cells can be embedded jointly for the further cell typing with higher accuracy and precision"
- [discussion] spatially heterogeneous subpopulations hard to identify traditionally: "However, they are difficult to identify by traditional single-cell analysis pipelines due to the similarity in transcriptome"
- [discussion] melanoma spatial variability conserved across replicates: "while reconstructing the pseudo-space of melanoma, spatial references are derived from different biological replicates, however, the performance of scSpace on spatial variability identification is"
- [discussion] caution in low spatial coherence tissues: "scSpace should be applied cautiously to tissues with low spatial coherence, such as Patients 5 and 9 of SCC evaluated in this paper"
