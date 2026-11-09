---
name: qc-plot-generation
description: Use when a raw Cell Ranger gene-expression matrix has been loaded into
  a SingCellaR object and you need QC plots (e.g. UMIs vs detected genes) to choose
  min/max UMI, gene, and mitochondrial-percentage filtering cutoffs before filter_cells_and_genes.
when_to_use_negative:
- Input is already a filtered, normalized SingCellaR/SingleCellExperiment object past
  the QC stage — QC plots would reflect post-filtering data rather than guide threshold
  selection.
- Data are targeted-sequencing reads rather than 10x Cell Ranger UMI output, where
  TargetSeq_process_cells_annotation (with ERCC spike-in percentages) is the appropriate
  annotation path instead.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SingCellaR
  role: R package providing get_cells_annotation, plot_UMIs_vs_Detected_genes, DoubletDetection_with_scrublet,
    and filter_cells_and_genes for QC plotting and filtering
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: Environment in which the SingCellaR QC workflow is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:206c5a5d01c0b11014e326d9af86dc87531876837c5ba6c0cda14ba2200732ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# qc-plot-generation

## Summary

Generates quality-control diagnostic plots for an individual scRNA-seq sample (e.g. eFL_1) within a SingCellaR workflow, visualizing relationships between UMIs, detected genes, and mitochondrial gene percentages to inform cell-filtering thresholds.

## When to use

Use when a raw Cell Ranger gene-expression matrix has been loaded into a SingCellaR object and you need QC plots (e.g. UMIs vs detected genes) to choose min/max UMI, gene, and mitochondrial-percentage filtering cutoffs before filter_cells_and_genes.

## When NOT to use

- Input is already a filtered, normalized SingCellaR/SingleCellExperiment object past the QC stage — QC plots would reflect post-filtering data rather than guide threshold selection.
- Data are targeted-sequencing reads rather than 10x Cell Ranger UMI output, where TargetSeq_process_cells_annotation (with ERCC spike-in percentages) is the appropriate annotation path instead.

## Inputs

- Cell Ranger gene-expression matrix (loaded via load_matrices_from_cellranger)
- SingCellaR object with cell metadata/annotation from get_cells_annotation

## Outputs

- QC plots including plot_UMIs_vs_Detected_genes output comparable to the protocol's Figure 1 QC panels
- Justified numeric filter thresholds for filter_cells_and_genes (min/max UMIs, min/max detected genes, min/max percent mito)
- Filtered SingCellaR object saved as eFL_1_SingCellaR_object.rds

## How to apply

After loading the cellranger matrix with load_matrices_from_cellranger (setting cellranger.version, isMultiFeatures, and selectedFeature), compute cell metadata with get_cells_annotation, which derives per-cell metrics including total UMIs, number of detected genes, and percent mitochondrial expression. Then call plot_UMIs_vs_Detected_genes to visualize the bivariate relationship between UMI counts and detected genes per cell, using the plot to identify low-quality cells (low UMIs/genes), doublet-like outliers (very high UMIs/genes), and dead cells (high mitochondrial percentage). The observed plot distributions justify the numeric thresholds later supplied to filter_cells_and_genes (min_UMIs, max_UMIs, min_detected_genes, max_detected_genes, min_percent_mito, max_percent_mito). Doublet detection with DoubletDetection_with_scrublet (passing expected.doublet.rate, seed, PCs, min.cells, min_gene_variablity) complements the visual inspection, and isRemovedDoublets drops scrublet-detected doublets during filtering. Export the QC plots and save the processed object as eFL_1_SingCellaR_object.rds so plots comparable to the protocol's Figure 1 QC panels can be validated.

## Related tools

- **SingCellaR** (R package providing get_cells_annotation, plot_UMIs_vs_Detected_genes, DoubletDetection_with_scrublet, and filter_cells_and_genes for QC plotting and filtering) — https://github.com/supatt-lab/SingCellaR
- **R** (Environment in which the SingCellaR QC workflow is executed)

## Examples

```
load_matrices_from_cellranger(eFL_1, cellranger.version, isMultiFeatures, selectedFeature); get_cells_annotation(eFL_1); plot_UMIs_vs_Detected_genes(eFL_1)
```

## Evaluation signals

- Confirm the saved object eFL_1_SingCellaR_object.rds exists on disk after the workflow completes.
- Verify QC filtering, normalization (normalize_UMIs with scale.factor and use.scaled.factor), and variable-gene fitting (get_variable_genes_by_fitting_GLM_model) executed without error.
- QC plots should be visually comparable to the protocol's Figure 1 QC panels, allowing for rotation differences caused by software version and seed number.
- Filter thresholds supplied to filter_cells_and_genes should be traceable to features visible in the UMIs-vs-detected-genes plot (low-quality tail, doublet outliers, mito-high cells).
- Scrublet doublet calls (expected.doublet.rate, seed, PCs, min.cells, min_gene_variablity) should be removed when isRemovedDoublets is set in filter_cells_and_genes.

## Limitations

- UMAP/FDG plot rotations may differ from the protocol figures due to different software versions and seed number settings.
- SingCellaR lacks 'automatic object transformation' for interoperability with other packages such as Seurat.
- No software versions are stated in the protocol methods, so exact reproduction of plots may vary.

## Evidence

- [other] Compute cell metadata/annotation with get_cells_annotation and plot the QC relationships using plot_UMIs_vs_Detected_genes.: [evidence span withheld — anchor status: not_found]
- [other] High-quality cells are selected by specifying filtering parameters based on the observed number of UMIs, the number of detected genes per cell, and the percentage of mitochondrial gene expression.: [evidence span withheld — anchor status: not_found]
- [methods] filter_cells_and_genes(object, min_UMIs, max_UMIs, min_detected_genes, max_detected_genes, min_percent_mito, max_percent_mito, genes_with_expressing_cells, isRemovedDoublets): "`filter_cells_and_genes(object, min_UMIs, max_UMIs, min_detected_genes, max_detected_genes, min_percent_mito, max_percent_mito, genes_with_expressing_cells, isRemovedDoublets)`"
- [readme] Tutorial URL from README: "The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/"
- [discussion] Plot rotation caveat from discussion: "This is caused by different software versions for generating plots and the seed number setting."
