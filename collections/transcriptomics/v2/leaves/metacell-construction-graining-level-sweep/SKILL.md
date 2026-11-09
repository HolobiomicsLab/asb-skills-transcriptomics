---
name: metacell-construction-graining-level-sweep
description: 'Use when building metacells and you must choose a graining level: sweep
  the graining parameter across a dataset and measure purity/cell-type recovery to
  pick a level that preserves rare populations without fragmenting the analysis.'
when_to_use_negative:
- The input is a single fixed-resolution metacell model and no re-construction at
  different graining levels is possible (e.g. a pre-built .h5ad metacell model loaded
  only for interactive exploration in MCView).
- The goal is a cell-type atlas projection against a reference (metacell projection/atlas
  mapping) rather than choosing construction granularity.
- The dataset has no ground-truth celltype annotations, since purity (proportion of
  the most abundant cell type within the metacell) cannot be computed without them.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: MetacellAnalysisToolkit
  role: Provides the packaged CD34_sc/CD34_mc example data, the metacell construction
    interface (SuperCell/SEACells backends), and the QC functions mc_purity and qc_boxplot
    used to score each graining level
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
- name: Seurat
  role: Preprocessing of the single-cell data (NormalizeData LogNormalize, FindVariableFeatures
    nfeatures=2500, ScaleData, RunPCA npcs=50, RunUMAP dims=1:50) to produce the latent
    embedding used for metacell construction
- name: SuperCell
  role: Optional construction backend for building metacells at each graining level
    through the toolkit
  repo: https://github.com/GfellerLab/SuperCell
- name: SEACells
  role: Optional construction backend for building metacells at each graining level
    through the toolkit
  repo: https://github.com/dpeerlab/SEACells
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:39df76ceb0090787dffce0614e7ba96b5388cd1450f374c2ade9ad48a8d20556
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GfellerLab/MetacellAnalysisToolkit
  tools:
  - MetacellAnalysisToolkit
  - SEACells
  - Seurat
  - SuperCell
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# metacell-construction-graining-level-sweep

## Summary

A systematic scan of metacell graining levels (coarse-to-fine average metacell sizes) that quantifies how well cell types are recovered at each level, using metacell purity and the number of distinct cell types represented per graining level.

## When to use

Use when building metacells and you must choose a graining level: sweep the graining parameter across a dataset and measure purity/cell-type recovery to pick a level that preserves rare populations without fragmenting the analysis.

## When NOT to use

- The input is a single fixed-resolution metacell model and no re-construction at different graining levels is possible (e.g. a pre-built .h5ad metacell model loaded only for interactive exploration in MCView).
- The goal is a cell-type atlas projection against a reference (metacell projection/atlas mapping) rather than choosing construction granularity.
- The dataset has no ground-truth celltype annotations, since purity (proportion of the most abundant cell type within the metacell) cannot be computed without them.

## Inputs

- Single-cell count matrix as a Seurat object (e.g. CD34_sc from MetacellAnalysisToolkit) with per-cell celltype annotations
- Reference metacell object defining the cell_membership annotation format (e.g. CD34_mc from MetacellAnalysisToolkit)
- Grid of graining levels (target average metacell sizes, coarse to fine)

## Outputs

- graining_scan_summary.csv: per-graining-level table of metacell counts, purity statistics, and number of distinct cell types represented
- QC purity boxplots per graining level (qc_boxplot output)
- Per-level metacell membership assignments in the CD34_mc cell_membership format

## How to apply

Load the packaged example data (CD34_sc single cells and CD34_mc reference metacell membership format) via MetacellAnalysisToolkit, then preprocess the single-cell object with Seurat: NormalizeData (LogNormalize), FindVariableFeatures (nfeatures=2500), ScaleData, RunPCA (npcs=50) and RunUMAP (dims=1:50, n.neighbors=15, min.dist=0.5) to obtain the latent embedding used by the construction backends. Using the MetacellAnalysisToolkit interface with SuperCell or SEACells backends, build metacells at a grid of graining levels so that average metacell size spans coarse to fine. At each level, assign every metacell to its most abundant cell category and compute purity with mc_purity(membership, annotation = sc_data$celltype). Tabulate the number of distinct cell types represented per graining level alongside purity statistics, visualize purity distributions with qc_boxplot(mc.obj, qc.metrics='purity'), and write the table to graining_scan_summary.csv. Rationale: low-complexity datasets recover cell types accurately even at large graining levels, but the same graining applied to more complex datasets merges and hides rare cell populations, so the sweep reveals the granularity at which rare populations begin to disappear.

## Related tools

- **MetacellAnalysisToolkit** (Provides the packaged CD34_sc/CD34_mc example data, the metacell construction interface (SuperCell/SEACells backends), and the QC functions mc_purity and qc_boxplot used to score each graining level) — https://github.com/GfellerLab/MetacellAnalysisToolkit
- **Seurat** (Preprocessing of the single-cell data (NormalizeData LogNormalize, FindVariableFeatures nfeatures=2500, ScaleData, RunPCA npcs=50, RunUMAP dims=1:50) to produce the latent embedding used for metacell construction)
- **SuperCell** (Optional construction backend for building metacells at each graining level through the toolkit) — https://github.com/GfellerLab/SuperCell
- **SEACells** (Optional construction backend for building metacells at each graining level through the toolkit) — https://github.com/dpeerlab/SEACells

## Examples

```
R> mc_data$purity <- mc_purity(membership = mc_data@misc$cell_membership$membership, annotation = sc_data$celltype); qc_boxplot(mc.obj = mc_data, qc.metrics = "purity")
```

## Evaluation signals

- At coarse graining levels on low-complexity data, purity remains high while the number of distinct cell types represented equals the number of annotated cell types.
- As graining becomes coarser on complex data, rare cell populations should be observed merging into other populations (drop in distinct cell types represented) — the sweep must capture the level at which this first occurs.
- Each metacell's purity equals the proportion of its most abundant cell type in sc_data$celltype; recompute mc_purity on the membership and confirm values fall in [0,1] and match the summary table.
- Every cell is assigned to exactly one metacell at each graining level (membership partitions CD34_sc), and every metacell has a defined most-abundant category.
- qc_boxplot(mc.obj, qc.metrics='purity') renders a purity distribution per graining level without errors, and graining_scan_summary.csv contains one row per swept graining level.

## Limitations

- A single graining level optimal for low-complexity datasets will miss rare cell populations in more complex datasets by merging them with other populations — no one-size-fits-all graining exists.
- Purity is computed against provided celltype annotations, so the sweep is only as good as the annotation quality.
- The sweep requires re-running metacell construction for each graining level, which scales with dataset size; metacells are not cell types and multiple metacells may share the same state.

## Evidence

- [results] Cell types are accurately recovered at relatively large graining levels in low-complexity datasets, while applying the same graining level to more complex datasets results in missing rare cell populations as they are merged with other populations.: [evidence span withheld — anchor status: not_found]
- [methods] purity definition: "the **purity** of a metacell is the proportion of the most abundant cell type within the metacell"
- [methods] mc_purity call: "mc_data$purity <- mc_purity(membership = mc_data@misc$cell_membership$membership, annotation = sc_data$celltype)"
- [methods] qc_boxplot call: "qc_boxplot(mc.obj = mc_data, qc.metrics = "purity")"
- [methods] packaged datasets: "sc_data <- MetacellAnalysisToolkit::CD34_sc"
- [readme] toolkit README pipeline description: "Toolkit for metacell analysis. It consists of the `MATK` command line tool to easily identify metacells with either [SEACells](https://github.com/dpeerlab/SEACells),"
