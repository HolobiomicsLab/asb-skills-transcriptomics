---
name: imputed-merfish-transcriptome-analysis
description: Use when you have a SpatialScope-implemented (imputed) image-based ST
  dataset such as MERFISH mouse1_slice180 with paired snRNA-seq cell type labels and
  want cell-type-specific or global spatially DE genes beyond the original gene panel.
when_to_use_negative:
- Input is raw (non-imputed) MERFISH limited to its original gene panel, where only
  63 cell-type-specific DE genes are detectable and transcriptome-wide imputation
  is required first
- The goal is spot-level cell type deconvolution of seq-based ST data (Visium/Slide-seq)
  rather than differential expression on single-cell-resolution image-based data
- No paired scRNA-seq/snRNA-seq reference with cell type labels is available, since
  both C-SIDE cell-type-specific testing and covariate adjustment in SPARK-X require
  labeled cell types
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3516
- http://edamontology.org/topic_3308
tools:
- name: SpatialScope
  role: Produces the imputed transcriptome-wide MERFISH expression matrix used as
    input to the DE analyses
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: C-SIDE (spacexr)
  role: Cell-type-specific inference of differential expression via run.CSIDE.nonparametric
    with gene_threshold=0.001, cell_type_threshold=10, fdr=0.2
  repo: https://github.com/dmcable/spacexr
- name: SPARK-X
  role: Global spatially DE gene detection on the imputed data with cell type labels
    as covariates
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:2c6a7a841b0ee6c75990706c6b092bd7cb9f5879dafc09f3899014f4390b3fd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - C-SIDE
  - C-SIDE (spacexr)
  - SPARK-X
  - SpatialScope
  - spacexr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3516
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# imputed-merfish-transcriptome-analysis

## Summary

Detects cell-type-specific and globally spatially differentially expressed genes on SpatialScope-imputed MERFISH data (e.g. mouse1_slice180), testing whether imputation of non-MERFISH genes expands the set of detectable spatially DE genes (63 in the original 254-gene panel vs 293 significant cell-type-specific genes at FDR 1% across ~1,938 imputed genes).

## When to use

Use when you have a SpatialScope-implemented (imputed) image-based ST dataset such as MERFISH mouse1_slice180 with paired snRNA-seq cell type labels and want cell-type-specific or global spatially DE genes beyond the original gene panel.

## When NOT to use

- Input is raw (non-imputed) MERFISH limited to its original gene panel, where only 63 cell-type-specific DE genes are detectable and transcriptome-wide imputation is required first
- The goal is spot-level cell type deconvolution of seq-based ST data (Visium/Slide-seq) rather than differential expression on single-cell-resolution image-based data
- No paired scRNA-seq/snRNA-seq reference with cell type labels is available, since both C-SIDE cell-type-specific testing and covariate adjustment in SPARK-X require labeled cell types

## Inputs

- SpatialScope-imputed MERFISH mouse1_slice180 single-cell expression matrix (~1,938 genes including imputed non-MERFISH genes)
- Cell type labels derived from the paired MOp droplet-based snRNA-seq reference (26,431 cells, 20 cell types)
- Spatial coordinates of MERFISH cells (for SPARK-X and C-SIDE covariates)

## Outputs

- Per-cell-type C-SIDE results (cell-type-specific spatially DE genes with FDR, e.g. 293 significant at FDR 1% vs 63 in original MERFISH genes)
- SPARK-X global spatially DE gene results (e.g. 243 genes, ~2.3x the MERFISH-only count)
- Spatial pattern summaries for markers (Ryr3 layer-6 excitatory neurons; Lingo2 upper layers)
- spatial_de_results.csv combining C-SIDE per-cell-type results, SPARK-X global results, and pattern summaries

## How to apply

Load the SpatialScope-imputed single-cell expression matrix (~1,938 genes including imputed non-MERFISH genes) for mouse1_slice180 together with cell type labels from the paired MOp droplet-based snRNA-seq reference (26,431 cells, 20 cell types). Run C-SIDE via run.CSIDE.nonparametric from spacexr on the imputed data with gene_threshold=0.001, cell_type_threshold=10, and fdr=0.2, then tabulate significant calls per cell type at FDR 1%; the expected contrast is 63 significant genes in the original MERFISH genes vs 293 after imputation. Run SPARK-X on the same imputed dataset with default parameters, using cell type labels as covariates to exclude genes spatially DE purely due to cell-type spatial distribution, yielding 243 globally spatial DE genes (~2.3x the MERFISH-only count). Visualize spatial patterns of reported markers such as Ryr3 in layer-6 excitatory neurons and Lingo2 in upper layers to confirm biological plausibility, and write per-cell-type C-SIDE results, SPARK-X global results, and pattern summaries to spatial_de_results.csv.

## Related tools

- **SpatialScope** (Produces the imputed transcriptome-wide MERFISH expression matrix used as input to the DE analyses) — https://github.com/YangLabHKUST/SpatialScope
- **C-SIDE (spacexr)** (Cell-type-specific inference of differential expression via run.CSIDE.nonparametric with gene_threshold=0.001, cell_type_threshold=10, fdr=0.2) — https://github.com/dmcable/spacexr
- **SPARK-X** (Global spatially DE gene detection on the imputed data with cell type labels as covariates)

## Examples

```
cside <- run.CSIDE.nonparametric(puck, RCTD, gene_threshold=0.001, cell_type_threshold=10, fdr=0.2); write.csv(get_de_genes(cside, 0.01), 'spatial_de_results.csv')
```

## Evaluation signals

- Per-cell-type C-SIDE significant calls at FDR 1% total 293 genes on imputed data vs 63 on original MERFISH genes, matching Fig. 6e-h
- SPARK-X identifies 243 globally spatial DE genes, approximately 2.3x the MERFISH-only count
- Recovered spatial patterns include Ryr3 enrichment in layer-6 excitatory neurons and Lingo2 in upper cortical layers
- spatial_de_results.csv is produced with per-cell-type C-SIDE results, SPARK-X global results, and pattern summaries
- C-SIDE parameters match the protocol: gene_threshold=0.001, cell_type_threshold=10, fdr=0.2

## Limitations

- Image-based ST platforms such as MERFISH lack whole-transcriptome coverage, so imputation accuracy bounds the reliability of DE calls on imputed non-MERFISH genes
- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid, potentially affecting imputation that feeds this analysis
- Platform effects must be appropriately corrected, otherwise downstream results would not be satisfactory

## Evidence

- [results] Applying C-SIDE, 63 cell-type-specific spatially DE genes were detected in the original MERFISH genes at FDR 1%, while incorporating SpatialScope-imputed non-MERFISH genes increased the count of significant genes (FDR < 1%) to 293 (1,938 genes total).: [evidence span withheld — anchor status: not_found]
- [methods] Run C-SIDE via run.CSIDE.nonparametric from spacexr on the imputed data with gene_threshold=0.001, cell_type_threshold=10, and fdr=0.2 to identify cell-type-specific spatially DE genes.: [evidence span withheld — anchor status: not_found]
- [abstract] SpatialScope not only enhances seq-based ST data to achieve single-cell resolution, but also accurately infers transcriptome-wide expression levels for image-based ST data: "SpatialScope not only enhances seq-based ST data to achieve single-cell resolution, but also accurately infers transcriptome-wide expression levels"
- [readme] Cell type-Specific Inference of Differential Expression (C-SIDE) learns cell type-specific differential expression on spatial transcriptomics dataset.: [evidence span withheld — anchor status: external_doc]
- [intro] they often lack whole-transcriptome coverage, resulting in only a few hundred genes in real applications: "they often lack whole-transcriptome coverage, resulting in only a few hundred genes in real applications"
