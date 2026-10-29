---
name: cell-type-specific-spatial-de-testing
description: Use when the goal is to identify genes whose spatial expression varies
  within a specific cell type — e.g. testing whether imputation of non-MERFISH genes
  into a MERFISH slice increases detectable cell-type-specific spatially DE genes.
when_to_use_negative:
- The spatial data already has whole-transcriptome coverage and no imputation is needed
  — de novo imputation plus DE testing adds a model-generated layer that is inappropriate
  when measured genes suffice.
- The question is spot-level deconvolution of cell-type proportions rather than gene-level
  differential expression testing (use RCTD/CARD/cell2location instead).
- No cell type labels or paired scRNA-seq/snRNA-seq reference exist to define the
  cell types along which DE is tested.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: spacexr (C-SIDE)
  role: Fits cell-type-specific spatially differential expression models (run.CSIDE.nonparametric)
    with gene_threshold=0.001, cell_type_threshold=10, fdr=0.2
  repo: https://github.com/dmcable/spacexr
- name: SPARK-X
  role: Global spatially DE gene testing on the imputed data, with cell type labels
    as covariates to remove DE explained by cell-type spatial distribution
- name: SpatialScope
  role: Produces the gene-imputed single-cell expression matrix (non-MERFISH genes
    imputed from the snRNA-seq reference) that the DE tests run on
  repo: https://github.com/YangLabHKUST/SpatialScope
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:116d9425c1eacd4f61bcb0cd4e10922213f111b9c3dcf0b93368d43c18409960
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dmcable/spacexr
  tools:
  - C-SIDE
  - SPARK-X
  - SpatialScope
  - spacexr
  - spacexr (C-SIDE)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cell-type-specific-spatial-de-testing

## Summary

Detects cell-type-specific spatially differentially expressed genes in a spatial transcriptomics dataset by running C-SIDE (spacexr) and SPARK-X on gene-imputed data, e.g. SpatialScope-imputed MERFISH, and tabulating significant calls at FDR 1% per cell type.

## When to use

Use when the goal is to identify genes whose spatial expression varies within a specific cell type — e.g. testing whether imputation of non-MERFISH genes into a MERFISH slice increases detectable cell-type-specific spatially DE genes.

## When NOT to use

- The spatial data already has whole-transcriptome coverage and no imputation is needed — de novo imputation plus DE testing adds a model-generated layer that is inappropriate when measured genes suffice.
- The question is spot-level deconvolution of cell-type proportions rather than gene-level differential expression testing (use RCTD/CARD/cell2location instead).
- No cell type labels or paired scRNA-seq/snRNA-seq reference exist to define the cell types along which DE is tested.

## Inputs

- SpatialScope-imputed single-cell spatial expression matrix (~1,938 genes) for MERFISH mouse1_slice180
- Cell type labels from the paired MOp droplet-based snRNA-seq reference (26,431 cells, 20 cell types)
- Spatial coordinates of segmented cells/pixels

## Outputs

- Per-cell-type C-SIDE differential expression result tables with FDR-significant calls (63 genes original vs 293 genes imputed at FDR 1%)
- SPARK-X global spatially DE gene results (243 globally spatial DE genes)
- spatial_de_results.csv containing C-SIDE results, SPARK-X global results, and pattern summaries
- Spatial pattern plots of marker genes (Ryr3, Lingo2)

## How to apply

Load the gene-imputed single-cell expression matrix (e.g. SpatialScope-imputed mouse1_slice180, ~1,938 genes including imputed non-MERFISH genes) together with cell type labels transferred from the paired MOp droplet-based snRNA-seq reference (26,431 cells, 20 cell types). Run C-SIDE via run.CSIDE.nonparametric from spacexr on the imputed data with gene_threshold=0.001, cell_type_threshold=10, and fdr=0.2, then tabulate significant calls at FDR 1% per cell type. Run SPARK-X on the same imputed dataset with default parameters, treating cell type labels as covariates to exclude spatially DE genes explained purely by the spatial distribution of cell types. Visualize spatial expression patterns of reported markers (Ryr3 in layer-6 excitatory neurons, Lingo2 in upper layers) and write per-cell-type C-SIDE results, SPARK-X global results, and pattern summaries to spatial_de_results.csv. Rationale: image-based ST such as MERFISH captures only a few hundred genes, so transcriptome-wide imputation from the snRNA-seq reference expands the space of testable DE genes before applying cell-type-resolved spatial DE models.

## Related tools

- **spacexr (C-SIDE)** (Fits cell-type-specific spatially differential expression models (run.CSIDE.nonparametric) with gene_threshold=0.001, cell_type_threshold=10, fdr=0.2) — https://github.com/dmcable/spacexr
- **SPARK-X** (Global spatially DE gene testing on the imputed data, with cell type labels as covariates to remove DE explained by cell-type spatial distribution)
- **SpatialScope** (Produces the gene-imputed single-cell expression matrix (non-MERFISH genes imputed from the snRNA-seq reference) that the DE tests run on) — https://github.com/YangLabHKUST/SpatialScope

## Examples

```
results <- run.CSIDE.nonparametric(puck, cell_type_info, gene_threshold=0.001, cell_type_threshold=10, fdr=0.2)
```

## Evaluation signals

- Regenerated C-SIDE counts at FDR 1% reproduce 63 significant cell-type-specific DE genes (original MERFISH genes) vs 293 (with imputed genes, 1,938 genes total), matching Fig. 6e-h
- SPARK-X on the imputed data recovers ~243 globally spatial DE genes, ~2.3x the MERFISH-only count
- Visualized marker patterns include Ryr3 in layer-6 excitatory neurons and Lingo2 in upper cortical layers
- spatial_de_results.csv contains per-cell-type C-SIDE tables, SPARK-X global results, and pattern summaries with the expected schema
- Gene space of the tested matrix matches the imputed ~1,938-gene set and cell type labels match the 20-cell-type MOp reference

## Limitations

- Imputed gene expression is model-inferred; DE calls on imputed non-MERFISH genes depend on imputation accuracy, which existing methods like Tangram, gimVI and SpaGE are reported to lack on sparse ST data.
- Training the deep generative model underlying imputation is time-consuming (measured in hours or days).
- The Potts assumption that neighboring cells share cell type may not always be valid, affecting the decomposition feeding DE testing.
- Platform effects must be appropriately corrected or cell type identification and gene expression decomposition (and hence DE testing) results are unsatisfactory.

## Evidence

- [readme] C-SIDE identifies cell-type-specific DE genes: [evidence span withheld — anchor status: external_doc]
- [methods] MERFISH slice used for imputation evaluation: "we used the slice, mouse1_slice180, from mouse1_sample4 to evaluate the imputation performance"
- [methods] Paired snRNA-seq reference defining the 20 cell types: "a paired droplet-based snRNA-seq profiles collected from mouse MOp as the reference, which measures the expression of 26,431 cells across 20 cell types"
- [abstract] Downstream DE analysis motivation: "facilitating downstream analysis, including detecting cellular communication through ligand-receptor interactions, localizing cellular subtypes, and identifying spatially differentially expressed"
