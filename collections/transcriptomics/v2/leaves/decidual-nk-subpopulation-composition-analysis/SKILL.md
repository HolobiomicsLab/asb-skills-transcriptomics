---
name: decidual-nk-subpopulation-composition-analysis
description: Use when a Ctrl-vs-RPL decidual immune query scRNA-seq dataset must be
  mapped onto the maternal-fetal interface reference to test whether dNK composition
  shifts (e.g. dNK1 decrease, dNK3 increase) reproduce.
when_to_use_negative:
- Query cells contain novel states absent from the Vento2018 MFI reference — fixed-model
  projections may be misleading, and unsupervised clustering should be compared against
  mapping results instead
- Query data derive from a different platform or UMI distribution than the reference
  (e.g. Smart-seq2 query vs 10x-based reference), where ProjectSVR projection performance
  declines
- The query is not decidual/MFI tissue — use the pre-built models matched to the tissue,
  e.g. model.disco_pbmc.rds (PBMC), model.Zheng2021.CD8Tcell.rds (pan-cancer T cells),
  or model.mTCA.rds (testicular germ cells)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ProjectSVR
  role: Runs MapQuery projection into the reference UMAP space with map-quality metrics,
    MapQCPlot quality assessment, LabelTransfer for cell-type annotation, and AlluviaPlot
    composition visualization
  repo: https://github.com/JarningGau/ProjectSVR
- name: Seurat
  role: Loads and manipulates the Guo2021 query and Vento2018 reference Seurat objects,
    subsets cells on map-quality and dNK-label criteria, and supports group-split
    visualization
- name: AUCell
  role: Computes the 0–1 gene activity scores based on rank expression that ProjectSVR's
    SVR models regress onto the reference embedding
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:324751bc7a3d4dcd154b71e9f6afb859c71a81f62cfe02a6d7d271587f7dfa45
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# decidual-nk-subpopulation-composition-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies shifts in decidual NK (dNK) subpopulation composition between healthy controls and recurrent pregnancy loss (RPL) patients by projecting Guo2021 decidual immune cells onto the Vento2018 maternal-fetal interface (MFI) reference atlas with ProjectSVR, transferring reference labels, and tabulating per-sample dNK1/dNK2/dNK3 frequencies.

## When to use

Use when a Ctrl-vs-RPL decidual immune query scRNA-seq dataset must be mapped onto the maternal-fetal interface reference to test whether dNK composition shifts (e.g. dNK1 decrease, dNK3 increase) reproduce.

## When NOT to use

- Query cells contain novel states absent from the Vento2018 MFI reference — fixed-model projections may be misleading, and unsupervised clustering should be compared against mapping results instead
- Query data derive from a different platform or UMI distribution than the reference (e.g. Smart-seq2 query vs 10x-based reference), where ProjectSVR projection performance declines
- The query is not decidual/MFI tissue — use the pre-built models matched to the tissue, e.g. model.disco_pbmc.rds (PBMC), model.Zheng2021.CD8Tcell.rds (pan-cancer T cells), or model.mTCA.rds (testicular germ cells)

## Inputs

- Guo2021 decidual immune query Seurat object (Guo2021.MFI.seurat.slim.qs) with orig.ident defining Ctrl vs RPL groups
- Pre-trained Vento2018 maternal-fetal interface reference model (model.Vento2018.MFI.rds from Zenodo record 8350732)
- Vento2018 MFI reference Seurat atlas with a 'cell_type' label column

## Outputs

- Query Seurat object with ref.umap projection coordinates, mapping-quality metrics, and knn.pred.celltype transferred labels
- Filtered dNK-compartment object containing only 'dNK p', 'dNK1', 'dNK2', 'dNK3' cells
- dnk_proportions.csv: per-sample dNK subtype frequency table split by Ctrl and RPL
- Alluvia plots and pheatmap cross-tabulation of predicted vs manual labels

## How to apply

Load the pre-trained Vento2018.MFI reference model (model.Vento2018.MFI.rds) and the Guo2021 decidual immune query Seurat object (Guo2021.MFI.seurat.slim.qs), defining the Ctrl vs RPL group factor from orig.ident. Run ProjectSVR::MapQuery on the query with add.map.qual = TRUE and ncores = 10 to place cells into the reference UMAP space (ref.umap), where AUCell-computed gene activity scores (0–1, rank-based) serve as regression features. Assess mapping quality with MapQCPlot using map.q.cutoff = 2 and retain only well-mapped cells by subsetting on mean.knn.dist < 2. Transfer reference cell-type labels via ProjectSVR::LabelTransfer with ref.label.col = 'cell_type', then subset the dNK compartment to knn.pred.celltype values 'dNK p', 'dNK1', 'dNK2', 'dNK3' and render projection and AlluviaPlot views of subtype composition split by group. Cross-tabulate predicted vs manually annotated labels with pheatmap, compute per-sample dNK subtype frequencies for Ctrl and RPL, and write the abundance summary to dnk_proportions.csv. Validation: the table should reproduce the reported pattern of dNK1 decreased and dNK3 increased in RPL, matching Fig. 4e-f of the original Guo2021 study.

## Related tools

- **ProjectSVR** (Runs MapQuery projection into the reference UMAP space with map-quality metrics, MapQCPlot quality assessment, LabelTransfer for cell-type annotation, and AlluviaPlot composition visualization) — https://github.com/JarningGau/ProjectSVR
- **Seurat** (Loads and manipulates the Guo2021 query and Vento2018 reference Seurat objects, subsets cells on map-quality and dNK-label criteria, and supports group-split visualization)
- **AUCell** (Computes the 0–1 gene activity scores based on rank expression that ProjectSVR's SVR models regress onto the reference embedding)

## Examples

```
seu.q <- ProjectSVR::MapQuery(seu.q, reference, ref.umap = "ref.umap", add.map.qual = TRUE, ncores = 10); seu.q <- subset(seu.q, mean.knn.dist < 2)
```

## Evaluation signals

- dnk_proportions.csv reproduces the reported Guo2021 pattern: dNK1 decreased and dNK3 increased in RPL patients (Fig. 4e-f)
- Retained query cells all satisfy mean.knn.dist < 2 after MapQCPlot assessment with map.q.cutoff = 2
- pheatmap cross-tabulation of knn.pred.celltype vs manual annotation shows high concordance for the dNK compartment
- The dNK subset contains only the labels 'dNK p', 'dNK1', 'dNK2', 'dNK3'
- Per-sample frequencies are computed at sample resolution with the Ctrl/RPL group factor derived from orig.ident

## Limitations

- Fixed-model projections may mislead for query cells containing novel states absent in the reference; users should compare unsupervised clustering to mapping results
- Performance declines when projecting across platforms (e.g. Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth
- The mean.knn.dist cutoff is dataset-dependent (examples in the paper use 1.6, 2, and 2.4), so the threshold must be determined per dataset via MapQCPlot

## Evidence

- [readme] readme: [evidence span withheld — anchor status: external_doc]
