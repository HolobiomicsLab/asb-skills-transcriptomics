---
name: single-cell-qc-filtering
description: Use when raw Cell Ranger gene-expression matrices (e.g., the eFL_1 fetal
  liver sample) must be pre-processed into a high-quality cell population before clustering,
  using UMI/gene-count/mito-fraction thresholds and scrublet doublet removal.
when_to_use_negative:
- Input is targeted-sequence data rather than UMI-based scRNA-seq — use TargetSeq_filter_cells_and_genes
  with min_Reads and ERCC spike-in thresholds (min/max_percent_ERCC) instead
- Data are already filtered and normalized, e.g. an existing SingCellaR or SingleCellExperiment
  object past the QC stage
- The goal is integration across batches only, without re-establishing per-sample
  QC — the protocol requires re-running filter_cells_and_genes and normalize_UMIs
  on the merged object first
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SingCellaR
  role: R package providing load_matrices_from_cellranger, filter_cells_and_genes,
    normalize_UMIs, and get_variable_genes_by_fitting_GLM_model for the QC pipeline
  repo: github.com/supatt-lab/SingCellaR
- name: R
  role: Execution environment for the SingCellaR workflow and saving the .rds/.rdata
    objects
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:cf057151b3015cf6732b9530e628a364dd5e518d267078284b5d378218243713
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# single-cell-qc-filtering

## Summary

Quality-control workflow from the SingCellaR protocol (STAR Protocols, 2022) that removes low-quality cells and doublets from scRNA-seq data by thresholding UMI counts, detected genes, and percent mitochondrial expression, then normalizes UMIs and identifies variable genes via GLM fitting.

## When to use

Use when raw Cell Ranger gene-expression matrices (e.g., the eFL_1 fetal liver sample) must be pre-processed into a high-quality cell population before clustering, using UMI/gene-count/mito-fraction thresholds and scrublet doublet removal.

## When NOT to use

- Input is targeted-sequence data rather than UMI-based scRNA-seq — use TargetSeq_filter_cells_and_genes with min_Reads and ERCC spike-in thresholds (min/max_percent_ERCC) instead
- Data are already filtered and normalized, e.g. an existing SingCellaR or SingleCellExperiment object past the QC stage
- The goal is integration across batches only, without re-establishing per-sample QC — the protocol requires re-running filter_cells_and_genes and normalize_UMIs on the merged object first

## Inputs

- Cell Ranger gene-expression matrix (10x output) for an individual scRNA-seq sample, e.g. eFL_1
- SingCellaR object parameters: cellranger.version, isMultiFeatures, selectedFeature
- User-specified QC thresholds: min/max_UMIs, min/max_detected_genes, min/max_percent_mito, genes_with_expressing_cells
- Scrublet parameters: expected.doublet.rate, seed, PCs, min.cells, min_gene_variablity

## Outputs

- Filtered, normalized SingCellaR object saved as e.g. eFL_1_SingCellaR_object.rds
- QC plots (UMIs vs detected genes) comparable to the protocol's Figure 1 panels
- Set of highly variable genes from GLM fitting for downstream dimensionality reduction
- Integrated object eFL_All.SingCellaR.rdata when merging biological replicates

## How to apply

Load the cellranger matrix into a SingCellaR object with load_matrices_from_cellranger, specifying cellranger.version, isMultiFeatures, and selectedFeature; compute cell metadata with get_cells_annotation and inspect QC relationships with plot_UMIs_vs_Detected_genes to choose thresholds. Detect doublets with DoubletDetection_with_scrublet (setting expected.doublet.rate, seed, PCs, min.cells, min_gene_variablity), then apply filter_cells_and_genes with min/max_UMIs, min/max_detected_genes, min/max_percent_mito, genes_with_expressing_cells, and isRemovedDoublets=TRUE to drop scrublet-detected doublets. Normalize counts with normalize_UMIs (scale.factor, use.scaled.factor) and select highly variable genes with get_variable_genes_by_fitting_GLM_model (mean_expr_cutoff, disp_zscore_cutoff, quantile_genes_expr_for_fitting, quantile_genes_cv2_for_fitting). Thresholds should be set from the observed UMI/gene/mito distributions rather than copied blindly. When integrating replicates (e.g., eFL_1 and eFL_2), re-run filtering, normalization, and the integrative variant get_variable_genes_for_integrative_data_by_fitting_GLM_model on the merged object before PCA/UMAP/Louvain clustering.

## Related tools

- **SingCellaR** (R package providing load_matrices_from_cellranger, filter_cells_and_genes, normalize_UMIs, and get_variable_genes_by_fitting_GLM_model for the QC pipeline) — github.com/supatt-lab/SingCellaR
- **R** (Execution environment for the SingCellaR workflow and saving the .rds/.rdata objects)

## Examples

```
filter_cells_and_genes(eFL_1, min_UMIs=1000, max_UMIs=30000, min_detected_genes=500, max_detected_genes=7000, min_percent_mito=0, max_percent_mito=15, genes_with_expressing_cells=3, isRemovedDoublets=TRUE)
```

## Evaluation signals

- The saved SingCellaR object (eFL_1_SingCellaR_object.rds) exists and the filtering/normalization/variable-gene steps executed without error
- QC plots resemble the protocol's Figure 1 UMI-vs-gene panels; doublet fraction matches the expected.doublet.rate passed to scrublet
- Post-filtering cells fall within the specified min/max_UMIs, min_detected_genes, and max_percent_mito bounds
- After integrating eFL_1 and eFL_2, UMAP annotated with multi-lineage gene sets shows cells clustering by major cell types (erythroid, myeloid, lymphoid, megakaryocytic), confirming successful integration

## Limitations

- SingCellaR still lacks 'automatic object transformation' to interact with other existing packages, such as Seurat
- Thresholds are sample-specific and must be chosen from observed QC distributions; UMAP/FDG plot rotations may differ from the protocol due to software version and seed number differences
- Only 75 pre-curated hematopoiesis gene sets are provided; users must compile custom signature gene sets for other systems

## Evidence

- [methods] filter_cells_and_genes parameters: "`filter_cells_and_genes(object, min_UMIs, max_UMIs, min_detected_genes, max_detected_genes, min_percent_mito, max_percent_mito, genes_with_expressing_cells, isRemovedDoublets)`"
- [methods] variable gene selection by GLM: "`get_variable_genes_by_fitting_GLM_model(object, mean_expr_cutoff, disp_zscore_cutoff, quantile_genes_expr_for_fitting, quantile_genes_cv2_for_fitting)`"
- [methods] UMI normalization step: "`normalize_UMIs(object, scale.factor, use.scaled.factor)`"
- [methods] loading cellranger matrices: "`load_matrices_from_cellranger(object, cellranger.version, isMultiFeatures, selectedFeature)`"
- [readme] SingCellaR tutorial README: "The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/"
