---
name: single-cell-metacell-construction
description: Use when you have a raw scRNA-seq count matrix (e.g. 10X/h5ad from GEO
  like GSE114953) and need to compress hundreds of thousands of single-cell profiles
  into ~75-cell metacells for downstream atlas-level analysis.
when_to_use_negative:
- The goal is metacell quality control or visualization of an already-built model
  — use MCView or MATK QC functions (mc_purity, qc_boxplot) instead of re-running
  construction.
- The dataset is small (e.g. the CD34 dataset bundled as MetacellAnalysisToolkit::CD34_mc/CD34_sc)
  and construction cost is irrelevant — heavy benchmarking instrumentation is unnecessary.
- Input is not gene-count data (e.g. already-aggregated pseudobulk or a feature table),
  since metacell algorithms operate on per-cell UMI profiles.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: MetacellAnalysisToolkit
  role: Integrated pipeline (MATK CLI + R package) to build metacells with SEACells,
    SuperCell, or MetaCell2 and evaluate them
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
- name: MC2 (tanaylab/metacells)
  role: Python implementation of the improved metacell algorithm; scalable to millions
    of cells; supports divide-and-conquer runs and projection
  repo: https://github.com/tanaylab/metacells
- name: SuperCell
  role: R package for graph-based metacell construction; projection-based variant
    extends tractability to 900k-cell datasets
  repo: https://github.com/GfellerLab/SuperCell
- name: SEACells
  role: Archetypal-analysis metacell construction on a nearest-neighbor kernel from
    PCA/SVD embedding; only tractable on subset sizes within its scaling limits
  repo: https://github.com/dpeerlab/SEACells
- name: Seurat
  role: Single-cell preprocessing (NormalizeData, variable features, scaling, PCA,
    UMAP) feeding the construction tools
- name: MCView
  role: Shiny app for interactive exploration of the resulting metacell models
  repo: https://github.com/tanaylab/MCView
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:de6eacd327feb85111a25e85e8257e147949fc72df86828d208f526da8e811f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GfellerLab/MetacellAnalysisToolkit
  tools:
  - MC2 (tanaylab/metacells)
  - MCView
  - MetacellAnalysisToolkit
  - SEACells
  - Seurat
  - SuperCell
  - ggplot2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-metacell-construction

## Summary

Constructs metacells — groups of cells of the same biological state with robust gene-expression statistics — from single-cell RNA-seq count matrices using MC2 (tanaylab/metacells), SuperCell, or SEACells, at a chosen graining level (e.g. 75), as codified in the MetacellAnalysisToolkit pipeline from Bilous et al. 2024.

## When to use

Use when you have a raw scRNA-seq count matrix (e.g. 10X/h5ad from GEO like GSE114953) and need to compress hundreds of thousands of single-cell profiles into ~75-cell metacells for downstream atlas-level analysis.

## When NOT to use

- The goal is metacell quality control or visualization of an already-built model — use MCView or MATK QC functions (mc_purity, qc_boxplot) instead of re-running construction.
- The dataset is small (e.g. the CD34 dataset bundled as MetacellAnalysisToolkit::CD34_mc/CD34_sc) and construction cost is irrelevant — heavy benchmarking instrumentation is unnecessary.
- Input is not gene-count data (e.g. already-aggregated pseudobulk or a feature table), since metacell algorithms operate on per-cell UMI profiles.

## Inputs

- scRNA-seq cell × gene UMI count matrix (e.g. 10X matrices or h5ad; MOCA subsets from GEO GSE114953, 10k–900k cells)
- Low-dimensional cell embedding for SEACells (X_pca from Seurat NormalizeData/PCA steps)
- Target graining level (number of cells per metacell, e.g. 75)

## Outputs

- Metacell object with cell-to-metacell membership assignments (e.g. cell_membership$membership)
- scaling_benchmark.csv with elapsed CPU time and peak memory per method × variant × dataset size
- fig5c_scaling_curves.png (ggplot2 CPU-time and peak-memory vs. number-of-cells curves per method)

## How to apply

Start from a cell × gene UMI count matrix; optionally pre-compute single-cell normalization, variable feature selection, scaling, PCA and UMAP as in the article's Seurat-based methods section. Choose a tool: SEACells runs archetypal analysis on a nearest-neighbor kernel built from a low-dimensional embedding (X_pca for scRNA-seq, X_svd for scATAC-seq); SuperCell builds a kNN-graph partition; MC2 (python metacells package) implements the improved metacell algorithm and is scalable to millions of cells. Run each tool through the MetacellAnalysisToolkit's `MATK` command line tool or conda environment (env/MetacellAnalysisToolkit_env.yml), setting the target metacell size (graining level 75 in the scaling benchmark). For large datasets (>several hundred thousand cells, e.g. MOCA at 900k cells), MC2's divide-and-conquer variant and the projection-based SuperCell variant are the only options that remain tractable; instrument each run to record elapsed CPU time and peak memory. Validate by checking coverage of every method × variant × dataset-size combination and by comparing scaling curves against the reference behavior (Fig 5C).

## Related tools

- **MetacellAnalysisToolkit** (Integrated pipeline (MATK CLI + R package) to build metacells with SEACells, SuperCell, or MetaCell2 and evaluate them) — https://github.com/GfellerLab/MetacellAnalysisToolkit
- **MC2 (tanaylab/metacells)** (Python implementation of the improved metacell algorithm; scalable to millions of cells; supports divide-and-conquer runs and projection) — https://github.com/tanaylab/metacells
- **SuperCell** (R package for graph-based metacell construction; projection-based variant extends tractability to 900k-cell datasets) — https://github.com/GfellerLab/SuperCell
- **SEACells** (Archetypal-analysis metacell construction on a nearest-neighbor kernel from PCA/SVD embedding; only tractable on subset sizes within its scaling limits) — https://github.com/dpeerlab/SEACells
- **Seurat** (Single-cell preprocessing (NormalizeData, variable features, scaling, PCA, UMAP) feeding the construction tools)
- **MCView** (Shiny app for interactive exploration of the resulting metacell models) — https://github.com/tanaylab/MCView

## Examples

```
conda env create -f env/MetacellAnalysisToolkit_env.yml && matk --tool SEACells --input counts.h5ad --graining 75 --out metacells.h5ad
```

## Evaluation signals

- Benchmark table covers every method × variant × dataset-size combination required by the design (10k–900k cells at graining level 75).
- Divide-and-conquer MC2 and projection-based SuperCell runs extend to the 900k-cell MOCA dataset, whereas standard runs fail or are dropped on the largest sizes — matching Fig 5C behavior.
- Per-metacell size is consistent with the graining level target (~75 cells per metacell), and membership assignments cover all input cells.
- Recorded CPU time and peak memory are monotonically increasing with dataset size for each method, with plottable scaling curves.
- Reconstructed metacells pass MATK QC metrics (e.g. purity boxplots) as in the tutorial workflow.

## Limitations

- Except for the divide-and-conquer MC2 method, metacell construction is time- and memory-consuming on large datasets, so SEACells and standard variants cannot run on 900k-cell datasets.
- tanaylab/metacells version 0.9 breaks API compatibility with version 0.8 for result annotations; pip currently installs 0.8-era versions and a conversion script is provided.
- Choosing the right lists of excluded genes and cells (e.g. mitochondrial genes, low-UMI cells) is a supervised step that is crucial for high-quality metacells.
- The tanaylab metacells package is migrating from AnnData to daf, which will again break API compatibility.

## Evidence

- [abstract] a comprehensive tutorial on the construction and analysis of metacells from single-cell RNA-seq data: "a comprehensive tutorial on the construction and analysis of metacells from single-cell RNA-seq data ( https://github.com/GfellerLab/MetacellAnalysisTutorial )"
- [readme] MATK command line tool description: "It consists of the `MATK` command line tool to easily identify metacells with either [SEACells](https://github.com/dpeerlab/SEACells), [SuperCell](https://github.com/GfellerLab/SuperCell) or"
- [readme] SEACells algorithm description: [evidence span withheld — anchor status: external_doc]
- [readme] MC2 scalability: [evidence span withheld — anchor status: external_doc]
- [methods] Toolkit load in methods: "library(MetacellAnalysisToolkit)"
