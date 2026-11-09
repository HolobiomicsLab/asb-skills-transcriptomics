---
name: cortical-layer-marker-gene-visualization
description: Use when a pseudo-space reconstruction of a layered tissue (e.g., DLPFC)
  must be validated and rendered as a figure showing layer ordering and marker-gene
  patterns alongside numeric correlation metrics.
when_to_use_negative:
- Tissues with low spatial coherence (e.g., Patients 5 and 9 of SCC in the paper),
  where reconstruction accuracy is compromised
- Target tissue regions that differ architecturally from the spatial reference (e.g.,
  dACC, which is agranular and lacks cortical layer 4), unless an appropriateness
  assessment of the reference has been performed
- Spatial-domain identification of ST data itself — scSpace targets spatial analysis
  of scRNA-seq data, not ST domain calling (SpaGCN/BayesSpace territory)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: scSpace
  role: Provides the TCA batch-effect removal, MLP pseudo-space projection, and joint
    gene-expression/spatial-weight embedding whose output is visualized and validated
  repo: https://github.com/ZJUFanLab/scSpace
- name: scanpy
  role: Normalization and preprocessing of reference and target expression matrices
    before projection and marker-gene assessment
  repo: https://github.com/scverse/scanpy
- name: matplotlib
  role: Rendering the pseudo-space, layer ordering, and layer-specific marker-gene
    patterns into the validation panel
- name: scipy/pandas
  role: Computing pairwise-distance correlations and layer-ordering statistics from
    the coordinate table
- name: igraph/leidenalg
  role: Leiden space-informed clustering that assigns the spatially informed subpopulations
    shown in the visualization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:6ba8af7d1f7be981476424a291bd53a64eeaa5fde5fde2dfc76b8e972d48a5bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - PASTE
  - Python
  - RCTD
  - SpatialScope
  - StarDist
  - Visium
  - igraph/leidenalg
  - matplotlib
  - scSpace
  - scanpy
  - scipy/pandas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cortical-layer-marker-gene-visualization

## Summary

Visualize and quantitatively validate scSpace-reconstructed pseudo-space of cortical tissue (e.g., human DLPFC 10X Visium slice 151674) by confirming cortical layer ordering, pairwise-distance correlation with the spatial reference, and layer-restricted marker-gene expression patterns, as in Fig. 2a-d of the scSpace paper.

## When to use

Use when a pseudo-space reconstruction of a layered tissue (e.g., DLPFC) must be validated and rendered as a figure showing layer ordering and marker-gene patterns alongside numeric correlation metrics.

## When NOT to use

- Tissues with low spatial coherence (e.g., Patients 5 and 9 of SCC in the paper), where reconstruction accuracy is compromised
- Target tissue regions that differ architecturally from the spatial reference (e.g., dACC, which is agranular and lacks cortical layer 4), unless an appropriateness assessment of the reference has been performed
- Spatial-domain identification of ST data itself — scSpace targets spatial analysis of scRNA-seq data, not ST domain calling (SpaGCN/BayesSpace territory)

## Inputs

- Pseudo-space coordinates for target cells output by scSpace's MLP projection
- Preprocessed human DLPFC 10X Visium spatial reference (slice 151674) with coordinates
- Space-informed Leiden cluster/subpopulation assignments per cell
- Gene expression matrix (scanpy-normalized) for layer-specific marker assessment

## Outputs

- dlpfc_pseudospace_results.csv (coordinates and pairwise-distance correlation / layer-ordering metrics)
- figure2_ad_validation.png (pseudo-space, layer ordering, and marker-gene pattern panel)
- Pairwise-distance correlation statistic between pseudo-space and reference spatial positions

## How to apply

After scSpace's TCA and MLP steps project target cells into pseudo-space coordinates, jointly embed gene expression and spatial weights and run space-informed Leiden clustering (igraph/leidenalg) to assign spatially informed subpopulations. Quantify reconstruction quality with scipy/pandas by computing pairwise-distance correlations between pseudo-space coordinates and the spatial reference positions, and by checking the ordering of cortical layers against the expected DLPFC hierarchy. Visualize the pseudo-space, layer ordering, and layer-specific marker-gene expression patterns with matplotlib to reproduce the Fig. 2a-d panel. Accept the reconstruction only when layer ordering matches the expected cortical hierarchy, the pairwise-distance correlation is significantly positive, and markers show layer-restricted expression. Persist the coordinate/metric table (dlpfc_pseudospace_results.csv) and the figure (figure2_ad_validation.png) as deliverables.

## Related tools

- **scSpace** (Provides the TCA batch-effect removal, MLP pseudo-space projection, and joint gene-expression/spatial-weight embedding whose output is visualized and validated) — https://github.com/ZJUFanLab/scSpace
- **scanpy** (Normalization and preprocessing of reference and target expression matrices before projection and marker-gene assessment) — https://github.com/scverse/scanpy
- **matplotlib** (Rendering the pseudo-space, layer ordering, and layer-specific marker-gene patterns into the validation panel)
- **scipy/pandas** (Computing pairwise-distance correlations and layer-ordering statistics from the coordinate table)
- **igraph/leidenalg** (Leiden space-informed clustering that assigns the spatially informed subpopulations shown in the visualization)

## Examples

```
cd scSpace-master && pip install -r requirements.txt && python setup.py install && jupyter notebook vignettes/DLPFC_slice151674.ipynb
```

## Evaluation signals

- Pseudo-space shows a significantly positive pairwise-distance correlation with the spatial reference positions
- Reconstructed pseudo-space reproduces the expected cortical layer ordering of the DLPFC hierarchy
- Layer-specific marker genes display layer-restricted expression consistent with Fig. 2a-d of the paper
- Neighbor system of spots from the reference is preserved in the pseudo-space
- dlpfc_pseudospace_results.csv and figure2_ad_validation.png are produced with complete coordinate/metric tables

## Limitations

- The choice of spatial reference directly influences the spatial reconstruction; dACC is agranular and lacks cortical layer 4, so an appropriateness assessment of the reference is required before applying scSpace
- scSpace should be applied cautiously to tissues with low spatial coherence, such as Patients 5 and 9 of SCC; intrinsic differences in spatial homogeneity between tissues may influence accuracy
- Appropriate spatial ATAC-seq reference data was not available, limiting modality coverage of the pseudo-space approach

## Evidence

- [discussion] appropriateness assessment of reference: "performing an appropriateness assessment or necessary pre-processing steps of the spatial reference should be considered before applying scSpace"
- [discussion] caution on low spatial coherence: "scSpace should be applied cautiously to tissues with low spatial coherence, such as Patients 5 and 9 of SCC evaluated in this paper"
- [readme] MLP projection and joint embedding: [evidence span withheld — anchor status: not_found]
- [discussion] scope distinction from ST domain methods: "scSpace focuses on the spatial analysis of scRNA-seq data whereas other methods target the spatial domain identification of ST data"
- [readme] TCA feature extraction: [evidence span withheld — anchor status: not_found]
