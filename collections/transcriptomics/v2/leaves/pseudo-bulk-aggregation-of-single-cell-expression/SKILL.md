---
name: pseudo-bulk-aggregation-of-single-cell-expression
description: Use when validating that curated single-cell RNA-seq data agree with
  matched bulk profiles — e.g. pseudo-bulk vs low-input bulk Pearson correlation and
  kBET batch mixing across early embryo stages (Fig. 1a-b, Supplementary Figs. 1-3).
when_to_use_negative:
- The downstream goal is cell-type-specific ligand-receptor communication (use the
  full TimeTalk RunTimeTalk workflow instead of stage-level pseudo-bulk aggregation)
- Input is already bulk RNA-seq — there is no per-cell layer to aggregate into pseudo-bulk
- Cell types are already defined and the analysis is between given cell types (e.g.
  CellPhoneDB/CellChat-style settings); this skill aggregates by developmental stage,
  not cell type
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: CreateSeuratObject with min.features = 2000, min.cells = 0 filters cells before
    pseudo-bulk aggregation
  repo: https://github.com/ChengLiLab/TimeTalk
- name: preprocessCore
  role: normalize.quantiles quantile-normalizes the merged pseudo-bulk and bulk expression
    matrix prior to correlation
- name: TimeTalk
  role: Provides the custom R scripts and Figshare deposit used to calculate pseudo-bulk
    expression per stage
  repo: https://figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498
- name: kBET
  role: Tests whether batches are well-mixed within random neighborhoods of fixed
    size in PCA space after aggregation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:def267da6ae5dc3f8c3f6fdd09fe9ca5a65f923a936c7c59084d3f3a3e9d2d24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - Seurat
  - TimeTalk
  - kBET
  - preprocessCore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# pseudo-bulk-aggregation-of-single-cell-expression

## Summary

Aggregates per-cell scRNA-seq counts into stage-level pseudo-bulk profiles for an 'average single cell' in each pre-implantation stage (GSE38495 MII-oocyte and GSE45719 E1–E4.0), then quantile-normalizes them so they can be directly compared against low-input bulk RNA-seq profiles from the TimeTalk Figshare deposit.

## When to use

Use when validating that curated single-cell RNA-seq data agree with matched bulk profiles — e.g. pseudo-bulk vs low-input bulk Pearson correlation and kBET batch mixing across early embryo stages (Fig. 1a-b, Supplementary Figs. 1-3).

## When NOT to use

- The downstream goal is cell-type-specific ligand-receptor communication (use the full TimeTalk RunTimeTalk workflow instead of stage-level pseudo-bulk aggregation)
- Input is already bulk RNA-seq — there is no per-cell layer to aggregate into pseudo-bulk
- Cell types are already defined and the analysis is between given cell types (e.g. CellPhoneDB/CellChat-style settings); this skill aggregates by developmental stage, not cell type

## Inputs

- GSE38495 MII-oocyte scRNA-seq count matrix (GEO)
- GSE45719 E1-E4.0 mouse pre-implantation scRNA-seq count matrices (GEO)
- Low-input bulk RNA-seq profiles from the TimeTalk Figshare deposit (figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498)
- Custom R pseudo-bulk scripts from the TimeTalk Figshare deposit

## Outputs

- Per-stage pseudo-bulk expression matrix (average single cell per pre-implantation stage)
- Quantile-normalized merged pseudo-bulk vs bulk expression matrix
- Pearson correlation coefficients per stage (Fig. 1a; r = 0.91 for late 2-cell)
- kBET batch-mixing results in PCA space (Supplementary Figs. 1-3)

## How to apply

Download the GEO scRNA-seq count matrices (GSE38495 MII-oocyte, GSE45719 E1–E4.0) and low-input bulk RNA-seq files from the TimeTalk Figshare project. Build Seurat objects with CreateSeuratObject using min.features = 2000 and min.cells = 0 to filter low-content cells. Run the custom R scripts shipped with the TimeTalk Figshare deposit to compute pseudo-bulk expression for an average single cell per pre-implantation stage. Merge pseudo-bulk and bulk profiles and apply preprocessCore::normalize.quantiles (version 1.48.0) before comparing stages. Judge success by Pearson correlation between matching stage profiles (e.g. r = 0.91 at the late 2-cell stage) and by kBET showing batches well-mixed within fixed-size neighborhoods in PCA space.

## Related tools

- **Seurat** (CreateSeuratObject with min.features = 2000, min.cells = 0 filters cells before pseudo-bulk aggregation) — https://github.com/ChengLiLab/TimeTalk
- **preprocessCore** (normalize.quantiles quantile-normalizes the merged pseudo-bulk and bulk expression matrix prior to correlation)
- **TimeTalk** (Provides the custom R scripts and Figshare deposit used to calculate pseudo-bulk expression per stage) — https://figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498
- **kBET** (Tests whether batches are well-mixed within random neighborhoods of fixed size in PCA space after aggregation)

## Examples

```
seu <- CreateSeuratObject(counts = counts_matrix, min.features = 2000, min.cells = 0)
```

## Evaluation signals

- Pearson correlation between matching stage pseudo-bulk and low-input bulk profiles is high; late 2-cell stage should reach ~0.91
- Cell filtering used CreateSeuratObject with min.features = 2000 and min.cells = 0 before aggregation
- Merged pseudo-bulk/bulk matrix was quantile-normalized with preprocessCore::normalize.quantiles before computing correlations
- kBET run in PCA space with a fixed neighborhood size shows data points from different batches well-mixed within random neighborhoods
- Reproduces Fig. 1a-b and Supplementary Figs. 1-3 of the TimeTalk manuscript

## Limitations

- The pseudo-bulk comparison is a QC validation of curated data; it does not itself infer ligand-receptor communication
- kBET and correlation conclusions depend on the fixed neighborhood size / preprocessing choices; the authors advise testing multiple parameter combinations for dependable results
- Granger-based downstream analyses are sensitive to the interpolation parameter winsz (Supplementary Figs. 20-21), a separate caveat from the pseudo-bulk QC step

## Evidence

- [discussion] Custom R scripts were used for the pre-implantation scRNA-seq data to calculate the pseudo-bulk expression for an average single cell in each stage.: "Custom R scripts were used for the pre-implantation scRNA-seq data to calculate the pseudo-bulk expression for an average single cell in each stage."
- [discussion] Quantile normalization was performed with the function "normalize.quantiles" from R package preprocessCore (version 1.48.0).: [evidence span withheld — anchor status: not_found]
- [discussion] "CreateSeuratObject" in Seurat (parameters: min.features = 2000, min.cells = 0).: [evidence span withheld — anchor status: not_found]
- [discussion] The kBET analysis assessed whether the data points with different batches in the PCA space were well-mixed within random neighborhoods of a fixed size.: "The kBET analysis assessed whether the data points with different batches in the PCA space were well-mixed within random neighborhoods of a fixed size."
- [readme] we advise users to conduct additional data analysis and experiment with various parameter combinations to achieve more dependable results: "we advise users to conduct additional data analysis and experiment with various parameter combinations to achieve more dependable results"
