---
name: cell-metadata-annotation
description: Use when starting a SingCellaR workflow from a Cell Ranger gene-expression
  matrix and you need per-cell QC annotation (UMIs, detected genes, mitochondrial
  percentage) before filtering low-quality cells and doublets.
when_to_use_negative:
- Data are targeted-sequence reads rather than UMI-based Cell Ranger output — use
  TargetSeq_process_cells_annotation with min_Reads and ERCC (min_percent_ERCC/max_percent_ERCC)
  parameters instead
- The object has already been QC-filtered and normalized; re-annotating would not
  change thresholds already applied
- You need interoperable objects for other packages — SingCellaR lacks 'automatic
  object transformation' to interact with packages such as Seurat
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SingCellaR
  role: R package providing get_cells_annotation, plot_UMIs_vs_Detected_genes, filter_cells_and_genes,
    normalize_UMIs, and get_variable_genes_by_fitting_GLM_model for the QC workflow
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: Execution environment for SingCellaR and its QC/annotation functions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:d0119d75a716179538fcc6be108e76f58be0d788adac7ea7c8e2e24afdfe5f35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cell-metadata-annotation

## Summary

Computes per-cell QC metadata (UMI counts, detected genes, percent mitochondrial expression) for a SingCellaR object and visualizes their relationships, forming the basis for quality-control filtering decisions on individual scRNA-seq samples such as eFL_1.

## When to use

Use when starting a SingCellaR workflow from a Cell Ranger gene-expression matrix and you need per-cell QC annotation (UMIs, detected genes, mitochondrial percentage) before filtering low-quality cells and doublets.

## When NOT to use

- Data are targeted-sequence reads rather than UMI-based Cell Ranger output — use TargetSeq_process_cells_annotation with min_Reads and ERCC (min_percent_ERCC/max_percent_ERCC) parameters instead
- The object has already been QC-filtered and normalized; re-annotating would not change thresholds already applied
- You need interoperable objects for other packages — SingCellaR lacks 'automatic object transformation' to interact with packages such as Seurat

## Inputs

- Cell Ranger gene-expression matrix (output of 10x Genomics Cell Ranger, loaded via load_matrices_from_cellranger with cellranger.version, isMultiFeatures, selectedFeature)
- SingCellaR object (SingleCellExperiment-based)
- Parameter choices for mitochondrial gene prefix and expected doublet rate

## Outputs

- Per-cell metadata/annotation within the SingCellaR object (UMI counts, detected genes, percent mitochondrial expression)
- QC plots such as UMI vs detected genes scatter panels comparable to the protocol's Figure 1 QC panels
- Filtered SingCellaR object (post filter_cells_and_genes and doublet removal), e.g. saved as eFL_1_SingCellaR_object.rds

## How to apply

After loading the Cell Ranger output with load_matrices_from_cellranger (specifying cellranger.version, isMultiFeatures, and selectedFeature), compute cell metadata/annotation with get_cells_annotation on the SingCellaR object. Visualize the QC relationships using plot_UMIs_vs_Detected_genes to inspect the joint distribution of UMI counts, detected genes, and percent mitochondrial expression per cell. These observed distributions inform the choice of min_UMIs, max_UMIs, min/max_detected_genes, and min/max_percent_mito thresholds subsequently applied in filter_cells_and_genes, together with DoubletDetection_with_scrublet (expected.doublet.rate, seed, PCs, min.cells, min_gene_variablity) whose isRemovedDoublets flag drops scrublet-detected doublets. Annotate each sample (e.g., eFL_1) separately so thresholds reflect that sample's own QC distributions.

## Related tools

- **SingCellaR** (R package providing get_cells_annotation, plot_UMIs_vs_Detected_genes, filter_cells_and_genes, normalize_UMIs, and get_variable_genes_by_fitting_GLM_model for the QC workflow) — https://github.com/supatt-lab/SingCellaR
- **R** (Execution environment for SingCellaR and its QC/annotation functions)

## Examples

```
eFL_1 <- newSingCellaRObject(); load_matrices_from_cellranger(eFL_1, cellranger.version=3, isMultiFeatures=FALSE, selectedFeature='Gene Expression'); get_cells_annotation(eFL_1, mito_genes_start_with='MT-'); plot_UMIs_vs_Detected_genes(eFL_1)
```

## Evaluation signals

- QC plots (UMIs vs detected genes) show distributions comparable to the protocol's Figure 1 QC panels
- filter_cells_and_genes executed without error and the isRemovedDoublets flag removed scrublet-detected doublets
- normalize_UMIs and get_variable_genes_by_fitting_GLM_model completed after annotation/filtering
- Saved SingCellaR object (eFL_1_SingCellaR_object.rds) exists and can be reloaded in R
- Resulting variable-gene list is non-empty given the specified mean_expr_cutoff and disp_zscore_cutoff

## Limitations

- UMAP/FDG plot rotations may differ from the protocol due to different software versions and seed-number settings
- Software versions are not stated in the protocol, so exact reproductions of plots may vary
- Doublet-detection and QC results depend on seed numbers and parameter choices (expected.doublet.rate, min.cells, min_gene_variablity)

## Evidence

- [results] High-quality cells are selected by specifying filtering parameters based on the observed number of UMIs, the number of detected genes per cell, and the percentage of mitochondrial gene expression.: [evidence span withheld — anchor status: not_found]
- [methods] Compute cell metadata/annotation with get_cells_annotation and plot the QC relationships using plot_UMIs_vs_Detected_genes.: [evidence span withheld — anchor status: not_found]
- [methods] `filter_cells_and_genes(object, min_UMIs, max_UMIs, min_detected_genes, max_detected_genes, min_percent_mito, max_percent_mito, genes_with_expressing_cells, isRemovedDoublets)`: "`filter_cells_and_genes(object, min_UMIs, max_UMIs, min_detected_genes, max_detected_genes, min_percent_mito, max_percent_mito, genes_with_expressing_cells, isRemovedDoublets)`"
- [readme] The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/: "The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/"
- [discussion] SingCellaR uses the SingleCellExperiment object, the standard object for storing single-cell experimental data in R.: "SingCellaR uses the SingleCellExperiment object, the standard object for storing single-cell experimental data in R."
