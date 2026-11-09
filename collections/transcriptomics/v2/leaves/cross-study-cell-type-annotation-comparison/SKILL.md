---
name: cross-study-cell-type-annotation-comparison
description: Use when query scRNA-seq datasets from two or more biological groups
  (e.g., Ctrl vs RPL decidual immune cells) have been projected onto a shared reference
  atlas and you need to compare cell-type label composition across groups.
when_to_use_negative:
- Query cells contain novel states absent in the reference — fixed-model projections
  may be misleading, so unsupervised clustering should be compared to mapping results
  instead of trusting transferred labels for cross-group comparison
- The query has not been quality-filtered by mapping metrics — low-quality mapped
  cells (high mean.knn.dist) will distort compositional comparisons across groups
- Cross-platform projection where performance declines (e.g., Smart-seq2 to 10x) —
  transferred labels may be too unreliable for quantitative composition comparison
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ProjectSVR
  role: Performs reference mapping (MapQuery) and label transfer (LabelTransfer) whose
    predicted labels (knn.pred.celltype) are compared across study groups; provides
    MapQCPlot and AlluviaPlot for quality and composition visualization
  repo: https://github.com/jarninggau/ProjectSVR
- name: Seurat
  role: Holds the query Seurat object with orig.ident group metadata; used for subsetting
    cells (e.g., dNK compartment, mean.knn.dist filter) and loading query/reference
    objects (Seurat/qs)
- name: pheatmap
  role: Renders cross-tabulation heatmaps of predicted vs manually annotated cell-type
    labels to validate label transfer consistency
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:3e3cbae4f28bb3e6d7aaf61539650d54f4b68fd08dd03c5fbf0377d9df5e3ca7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jarninggau/ProjectSVR
  repo_url: https://github.com/jarninggau/ProjectSVR
  tools:
  - ProjectSVR
  - Seurat
  - pheatmap
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-study-cell-type-annotation-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares predicted cell-type compositions across conditions (e.g., healthy controls vs RPL patients) after transferring reference labels onto mapped query cells, to test whether a disease state shifts cell population abundances relative to a reference atlas.

## When to use

Use when query scRNA-seq datasets from two or more biological groups (e.g., Ctrl vs RPL decidual immune cells) have been projected onto a shared reference atlas and you need to compare cell-type label composition across groups.

## When NOT to use

- Query cells contain novel states absent in the reference — fixed-model projections may be misleading, so unsupervised clustering should be compared to mapping results instead of trusting transferred labels for cross-group comparison
- The query has not been quality-filtered by mapping metrics — low-quality mapped cells (high mean.knn.dist) will distort compositional comparisons across groups
- Cross-platform projection where performance declines (e.g., Smart-seq2 to 10x) — transferred labels may be too unreliable for quantitative composition comparison

## Inputs

- Mapped query Seurat object with knn.pred.celltype label-transfer column and mapping-quality metrics (after mean.knn.dist < 2 filtering)
- Group metadata factor derived from orig.ident (e.g., Ctrl vs RPL)
- Reference cell-type label column ('cell_type') from the pre-trained reference model (e.g., model.Vento2018.MFI.rds)
- Manually annotated cell-type labels from the query study for cross-tabulation

## Outputs

- Predicted label composition plots split by group (reference palette)
- AlluviaPlot of dNK subtype composition by group
- pheatmap cross-tabulation of predicted (knn.pred.celltype) vs manually annotated labels
- Per-sample abundance summary table (e.g., dnk_proportions.csv) with dNK subtype frequencies per Ctrl/RPL sample

## How to apply

After mapping query cells with ProjectSVR::MapQuery, filtering to well-mapped cells (mean.knn.dist < 2 with map.q.cutoff = 2), and transferring reference labels via ProjectSVR::LabelTransfer with ref.label.col = 'cell_type', define the group factor from metadata (e.g., Ctrl vs RPL from orig.ident). Visualize predicted labels split by group using the reference color palette, then subset to a compartment of interest (e.g., dNK subtypes 'dNK p', 'dNK1', 'dNK2', 'dNK3' in knn.pred.celltype) and render projection and AlluviaPlot visualizations of subtype composition by group. Cross-tabulate predicted vs manually annotated labels with pheatmap to check annotation consistency, and compute per-sample subtype frequencies for each group, writing the abundance summary to a CSV (e.g., dnk_proportions.csv). Validation is by reproducing the expected compositional shift — here that RPL dNK cells shift toward dNK3 (dNK1 decreased, dNK3 increased), matching Fig. 4e-f of Guo2021.

## Related tools

- **ProjectSVR** (Performs reference mapping (MapQuery) and label transfer (LabelTransfer) whose predicted labels (knn.pred.celltype) are compared across study groups; provides MapQCPlot and AlluviaPlot for quality and composition visualization) — https://github.com/jarninggau/ProjectSVR
- **Seurat** (Holds the query Seurat object with orig.ident group metadata; used for subsetting cells (e.g., dNK compartment, mean.knn.dist filter) and loading query/reference objects (Seurat/qs))
- **pheatmap** (Renders cross-tabulation heatmaps of predicted vs manually annotated cell-type labels to validate label transfer consistency)

## Examples

```
seu.q <- subset(seu.q, mean.knn.dist < 2); seu.q <- ProjectSVR::LabelTransfer(seu.q, reference, ref.label.col = "cell_type"); table(seu.q$knn.pred.celltype, seu.q$group)
```

## Evaluation signals

- Cross-tabulation heatmap (pheatmap) of knn.pred.celltype vs manual annotations shows high diagonal agreement — transferred labels match manual annotation
- Per-sample dNK frequency table (dnk_proportions.csv) reproduces the reported pattern: dNK1 decreased and dNK3 increased in RPL vs Ctrl, matching Fig. 4e-f of Guo2021
- All retained cells pass the mapping-quality cutoff (mean.knn.dist < 2 after MapQCPlot with map.q.cutoff = 2)
- AlluviaPlot composition bars are computed per sample (not per cell pool), avoiding sample-level confounding in group comparisons

## Limitations

- If query cells contain novel states absent in the reference, fixed-model projections may be misleading; users should compare unsupervised clustering to mapping results to detect inconsistencies
- Performance declines when projecting across platforms (e.g. Smart-seq2 to 10x), due to differing UMI distributions and sequencing depth, which could bias label-based composition comparisons
- Independent SVR models per UMAP dimension neglect correlations among embedding dimensions, which may affect mapping fidelity underlying label transfer

## Evidence

- [other] Transfer reference cell-type labels with ProjectSVR::LabelTransfer using ref.label.col = 'cell_type' and visualize predicted labels split by group with the reference color palette.: [evidence span withheld — anchor status: not_found]
- [other] Projecting RPL and healthy decidual immune cells via ProjectSVR reproduced the previously reported shift in decidual NK composition.: [evidence span withheld — anchor status: not_found]
- [discussion] Projection offers an unbiased method to compare query cells across technologies, genotypes, disease states, and treatments.: "ProjectSVR offers an unbiased method to compare query cells across technologies, genotypes, disease states, and treatments within a unified reference space."
- [discussion] If query cells contain novel states absent in the reference, fixed-model projections may be misleading.: "If query cells contain novel states absent in the reference, fixed-model projections may be misleading. Users should compare unsupervised clustering to mapping results to detect inconsistencies."
- [other] The tabulated composition should reproduce the reported pattern that dNK cells in RPL are shifted toward the dNK3 population, matching Fig. 4e-f of the original Guo2021 study.: [evidence span withheld — anchor status: not_found]
- [readme] Decidual immune microenvironment of RPL patients tutorial listed in ProjectSVR README.: "Tutorials on how to project the query datasets onto reference atlas via
pre-build models.

- [Decidual immune microenvironment of RPL
  patients](https://jarninggau.github.io/ProjectSVR/articles/mapQu"
