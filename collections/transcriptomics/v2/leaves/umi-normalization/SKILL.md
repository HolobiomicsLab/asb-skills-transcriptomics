---
name: umi-normalization
description: Use when a cellranger gene-expression matrix has been loaded into a SingCellaR
  object and passed through filter_cells_and_genes (UMI, detected-gene, percent-mito
  thresholds, doublet removal), and counts must be scaled before variable-gene fitting
  or clustering.
when_to_use_negative:
- Input is a targeted-sequence dataset processed with TargetSeq_process_cells_annotation
  / TargetSeq_filter_cells_and_genes, where read-based (not UMI-based) QC and normalization
  semantics apply
- The object has not yet passed quality control (UMI/gene/mito filtering and doublet
  removal), since normalization would act on low-quality cells and detected doublets
- Interoperability with Seurat is required without conversion — SingCellaR lacks 'automatic
  object transformation' to interact with other packages such as Seurat
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SingCellaR
  role: R package providing normalize_UMIs, filter_cells_and_genes, and get_variable_genes_by_fitting_GLM_model
    used in the normalization workflow; built on the SingleCellExperiment object
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: Language environment in which SingCellaR normalization and the surrounding
    protocol steps are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:557592b8a04c694e07e760209990599c516c0f6a065643707d7ff869d4675cdd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# umi-normalization

## Summary

Normalizes UMI counts in a SingCellaR object using the normalize_UMIs function with a scale.factor and the use.scaled.factor option, a required post-QC step of the SingCellaR scRNA-seq protocol that prepares filtered cells for variable-gene selection and downstream analyses.

## When to use

Use when a cellranger gene-expression matrix has been loaded into a SingCellaR object and passed through filter_cells_and_genes (UMI, detected-gene, percent-mito thresholds, doublet removal), and counts must be scaled before variable-gene fitting or clustering.

## When NOT to use

- Input is a targeted-sequence dataset processed with TargetSeq_process_cells_annotation / TargetSeq_filter_cells_and_genes, where read-based (not UMI-based) QC and normalization semantics apply
- The object has not yet passed quality control (UMI/gene/mito filtering and doublet removal), since normalization would act on low-quality cells and detected doublets
- Interoperability with Seurat is required without conversion — SingCellaR lacks 'automatic object transformation' to interact with other packages such as Seurat

## Inputs

- SingCellaR object loaded from a cellranger gene-expression matrix via load_matrices_from_cellranger (e.g. the eFL_1 sample)
- QC-filtered cell set produced by filter_cells_and_genes with doublets removed via DoubletDetection_with_scrublet
- scale.factor value and use.scaled.factor flag for normalize_UMIs

## Outputs

- SingCellaR object with normalized UMI expression values stored in the SingleCellExperiment object
- Normalized object suitable for get_variable_genes_by_fitting_GLM_model and export as .rds (e.g. eFL_1_SingCellaR_object.rds)

## How to apply

After quality control with filter_cells_and_genes (setting min_UMIs, max_UMIs, min_detected_genes, max_detected_genes, min_percent_mito, max_percent_mito, genes_with_expressing_cells, and isRemovedDoublets to drop scrublet-detected doublets), call normalize_UMIs on the SingCellaR object, supplying a scale.factor and the use.scaled.factor option to scale each cell's UMI counts to a common denominator. Rationale: raw UMI counts reflect both expression and sequencing depth/cell capture efficiency, so normalization removes library-size differences before downstream steps. Immediately follow normalization with get_variable_genes_by_fitting_GLM_model (mean_expr_cutoff, disp_zscore_cutoff, quantile_genes_expr_for_fitting, quantile_genes_cv2_for_fitting), which consumes the normalized expression values. Then save the object (e.g. eFL_1_SingCellaR_object.rds) so the normalized state is persisted for integrative or single-sample downstream analyses.

## Related tools

- **SingCellaR** (R package providing normalize_UMIs, filter_cells_and_genes, and get_variable_genes_by_fitting_GLM_model used in the normalization workflow; built on the SingleCellExperiment object) — https://github.com/supatt-lab/SingCellaR
- **R** (Language environment in which SingCellaR normalization and the surrounding protocol steps are executed)

## Examples

```
normalize_UMIs(eFL_1, scale.factor = 10000, use.scaled.factor = T)
```

## Evaluation signals

- normalize_UMIs executes without error after filter_cells_and_genes and before get_variable_genes_by_fitting_GLM_model in the pipeline order
- The saved object (eFL_1_SingCellaR_object.rds) exists after the step, confirming the normalized state was persisted
- Subsequent GLM-based variable-gene fitting (mean_expr_cutoff, disp_zscore_cutoff) completes successfully, which requires normalized expression as input
- Normalization was applied after doublet removal (isRemovedDoublets) so library-size scaling is not skewed by scrublet-detected doublets
- Resulting downstream plots (e.g. UMI-vs-detected-genes QC panels, clustering visualizations) are comparable to the protocol's Figure 1 QC panels modulo seed/software-version rotation differences

## Limitations

- SingCellaR still lacks 'automatic object transformation' to interact with other existing packages, such as Seurat, limiting reuse of the normalized object outside SingCellaR
- UMAP/FDG plot rotations may differ from the protocol due to software version and seed number differences, complicating visual comparisons after normalization-driven downstream steps
- No software versions are stated in the protocol's methods section, so reproducing exact normalization behavior may require checking the SingCellaR repository
- AUCell-based gene-set scoring after normalization can face running time and memory issues on large-scale datasets

## Evidence

- [methods] Normalize UMIs with normalize_UMIs using a scale.factor and the use.scaled.factor option.: [evidence span withheld — anchor status: not_found]
- [methods] normalize_UMIs function signature with scale.factor and use.scaled.factor parameters.: "`normalize_UMIs(object, scale.factor, use.scaled.factor)`"
- [methods] Pipeline order: filter cells and genes before normalize UMIs, then variable gene selection.: [evidence span withheld — anchor status: not_found]
- [discussion] SingCellaR stores data in the SingleCellExperiment standard object in R.: "SingCellaR uses the SingleCellExperiment object, the standard object for storing single-cell experimental data in R."
- [readme] SingCellaR repository and tutorial location from README.: "The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/"
