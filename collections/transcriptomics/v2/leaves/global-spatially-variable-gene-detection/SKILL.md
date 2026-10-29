---
name: global-spatially-variable-gene-detection
description: Use when you have single-cell-resolution spatial expression (e.g. ~1,938-gene
  SpatialScope-imputed MERFISH mouse1_slice180 matrix) and need genes significantly
  spatially variable, not just cell-type-composition artifacts.
when_to_use_negative:
- The spatial data is unimputed seq-based ST (e.g. Visium) at spot resolution — SpatialScope
  imputation to single-cell resolution is a prerequisite for this cell-type-aware
  detection.
- The goal is cell-type-specific differential expression only — C-SIDE alone (via
  run.CSIDE.nonparametric) addresses that without the global SPARK-X screen.
- No spatial coordinates are available for the expression matrix, since spatial testing
  (SPARK-X, C-SIDE spatial covariates) requires cell/spot locations.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SPARK-X
  role: Detects globally spatially DE genes on the imputed MERFISH data with default
    parameters, using cell type labels as covariates
- name: spacexr (C-SIDE)
  role: run.CSIDE.nonparametric identifies cell-type-specific spatially DE genes (gene_threshold=0.001,
    cell_type_threshold=10, fdr=0.2) to complement the global screen
  repo: https://github.com/dmcable/spacexr
- name: SpatialScope
  role: Produces the imputed transcriptome-wide expression matrix on which spatial
    DE testing is run
  repo: https://github.com/YangLabHKUST/SpatialScope
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:f0c709f2a4fb1a533c2d63099c5f49baebdd5a52ec3b1ab8fccab5a5f2080d2c
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Global spatially variable gene detection

## Summary

Detects genes whose expression varies across spatial coordinates (globally spatially DE genes) on SpatialScope-imputed MERFISH data, using SPARK-X with cell-type labels as covariates so that genes merely mirroring cell-type spatial distributions are excluded.

## When to use

Use when you have single-cell-resolution spatial expression (e.g. ~1,938-gene SpatialScope-imputed MERFISH mouse1_slice180 matrix) and need genes significantly spatially variable, not just cell-type-composition artifacts.

## When NOT to use

- The spatial data is unimputed seq-based ST (e.g. Visium) at spot resolution — SpatialScope imputation to single-cell resolution is a prerequisite for this cell-type-aware detection.
- The goal is cell-type-specific differential expression only — C-SIDE alone (via run.CSIDE.nonparametric) addresses that without the global SPARK-X screen.
- No spatial coordinates are available for the expression matrix, since spatial testing (SPARK-X, C-SIDE spatial covariates) requires cell/spot locations.

## Inputs

- SpatialScope-imputed single-cell expression matrix (mouse1_slice180, ~1,938 genes including imputed non-MERFISH genes)
- Spatial coordinates / spot locations for each cell
- Cell type labels derived from the paired MOp droplet-based snRNA-seq reference (26,431 cells, 20 cell types)

## Outputs

- SPARK-X global results table of spatially variable genes (p-values, adjusted p-values at FDR 1%)
- Per-cell-type C-SIDE result tables of spatially DE genes
- spatial_de_results.csv combining C-SIDE per-cell-type results, SPARK-X global results, and spatial pattern summaries (e.g. Ryr3 layer-6, Lingo2 upper-layer patterns)

## How to apply

Load the SpatialScope-imputed expression matrix (mouse1_slice180, ~1,938 genes including imputed non-MERFISH genes) with spatial coordinates and cell type labels from the paired MOp droplet-based snRNA-seq reference (26,431 cells, 20 cell types). Run SPARK-X with default parameters, supplying cell type labels as covariates so that globally spatial DE genes explained purely by cell-type spatial distribution are excluded. Complement this with C-SIDE (run.CSIDE.nonparametric from spacexr; gene_threshold=0.001, cell_type_threshold=10, fdr=0.2) to separate cell-type-specific from global spatial effects. Tabulate significant calls at FDR 1% per cell type and write global results to spatial_de_results.csv; expect ~243 globally spatial DE genes (~2.3x the MERFISH-only count).

## Related tools

- **SPARK-X** (Detects globally spatially DE genes on the imputed MERFISH data with default parameters, using cell type labels as covariates)
- **spacexr (C-SIDE)** (run.CSIDE.nonparametric identifies cell-type-specific spatially DE genes (gene_threshold=0.001, cell_type_threshold=10, fdr=0.2) to complement the global screen) — https://github.com/dmcable/spacexr
- **SpatialScope** (Produces the imputed transcriptome-wide expression matrix on which spatial DE testing is run) — https://github.com/YangLabHKUST/SpatialScope

## Examples

```
results <- sparkx(imputed_counts_matrix, cell_locations, numCores = 1)  # SPARK-X on ~1,938-gene SpatialScope-imputed MERFISH with cell-type covariates
```

## Evaluation signals

- Significant cell-type-specific DE gene counts at FDR 1% match 63 (original MERFISH genes) vs 293 (after SpatialScope imputation, 1,938 genes tested)
- SPARK-X recovers ~243 globally spatial DE genes, ~2.3x the MERFISH-only count
- Recovered spatial patterns include Ryr3 in layer-6 excitatory neurons and Lingo2 in upper layers
- Genes whose spatial signal is explained purely by cell-type distribution are excluded (verified by cell-type covariates in SPARK-X)
- Output schema matches spatial_de_results.csv with C-SIDE per-cell-type results, SPARK-X global results, and pattern summaries

## Limitations

- C-SIDE parameters in the reproduction use fdr=0.2 for the model fit, with significant calls tabulated at FDR 1% — mixing these thresholds can change gene counts.
- SpatialScope training of the score-based generative model is time-consuming (measured in hours or days), and platform effects must be corrected or downstream results are unsatisfactory.
- The Potts-like assumption that neighboring cells share cell types may not hold in all tissues.

## Evidence

- [methods] SPARK-X with cell-type covariates on imputed data: [evidence span withheld — anchor status: not_found]
- [readme] C-SIDE cell-type-specific differential expression: [evidence span withheld — anchor status: external_doc]
- [methods] C-SIDE parameters used in the reproduction: [evidence span withheld — anchor status: not_found]
- [abstract] Imputation enables spatially DE detection: "facilitating downstream analysis, including detecting cellular communication through ligand-receptor interactions, localizing cellular subtypes, and identifying spatially differentially expressed"
- [methods] Gene filter applied to spatial data: "we retained genes with non-zero expression level on at least 10 spots"
