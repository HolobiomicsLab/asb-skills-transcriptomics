---
name: scrna-seq-count-aggregation-to-metacells
description: Use when a scRNA-seq dataset needs to be reduced from many individual
  cells to metacells (e.g. ~100-fold cell reduction on large atlases), or when metacell-level
  DE/DA/PCA must be weighted by metacell size to match single-cell reference results.
when_to_use_negative:
- When the biological question requires per-cell resolution (metacells are not cell
  types; multiple metacells can share the same state, and inter-cell heterogeneity
  is averaged out within each metacell).
- When metacells fail QC (low purity — the most abundant cell type does not dominate
  the metacell), since aggregation would mix biological states.
- When metacell-level DE/DA results must be reported without applying metacell size
  weights — unweighted aggregation-based comparisons produce type I/II errors relative
  to the single-cell reference.
- When working with the tanaylab/metacells v0.8 API, which breaks compatibility with
  v0.9 result annotations (use the provided conversion script instead).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: MetacellAnalysisToolkit
  role: Integrated pipeline (MATK CLI + R package) to build metacells with SEACells,
    SuperCell or MetaCell2 and perform QC/visualization of the aggregated metacells
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
- name: Seurat
  role: R framework used to normalize single-cell data, aggregate expression into
    metacell Seurat objects, and run weighted/unweighted metacell-level differential
    expression
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
- name: SEACells
  role: One of the metacell identification methods selectable through MATK (archetypal
    analysis on a nearest-neighbor kernel)
  repo: https://github.com/dpeerlab/SEACells
- name: SuperCell
  role: One of the metacell identification methods selectable through MATK
  repo: https://github.com/GfellerLab/SuperCell
- name: MCView
  role: Shiny-app for interactive exploration and annotation of the resulting metacell
    models
  repo: https://github.com/tanaylab/MCView
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:4f0a58f8ba22fa0d3e4e77cc7434f78f02d602662577e7b5797e9d1e6be81a45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GfellerLab/MetacellAnalysisToolkit
  tools:
  - MCView
  - MetacellAnalysisToolkit
  - SEACells
  - Seurat
  - SuperCell
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# scrna-seq-count-aggregation-to-metacells

## Summary

Aggregates single-cell RNA-seq profiles into metacells (groups of cells of the same biological state) using the MetacellAnalysisToolkit pipeline, which reduces data size and sampling noise while preserving biologically relevant information. Constituent single-cell counts (metacell sizes) must be carried as weights into downstream metacell-level analyses to avoid spurious differential expression and abundance artifacts.

## When to use

Use when a scRNA-seq dataset needs to be reduced from many individual cells to metacells (e.g. ~100-fold cell reduction on large atlases), or when metacell-level DE/DA/PCA must be weighted by metacell size to match single-cell reference results.

## When NOT to use

- When the biological question requires per-cell resolution (metacells are not cell types; multiple metacells can share the same state, and inter-cell heterogeneity is averaged out within each metacell).
- When metacells fail QC (low purity — the most abundant cell type does not dominate the metacell), since aggregation would mix biological states.
- When metacell-level DE/DA results must be reported without applying metacell size weights — unweighted aggregation-based comparisons produce type I/II errors relative to the single-cell reference.
- When working with the tanaylab/metacells v0.8 API, which breaks compatibility with v0.9 result annotations (use the provided conversion script instead).

## Inputs

- Raw single-cell UMI count matrix (e.g. 10X scRNA-seq)
- Single-cell Seurat object (e.g. MetacellAnalysisToolkit::CD34_sc)
- Metacell Seurat object (e.g. MetacellAnalysisToolkit::CD34_mc)
- Cell-to-metacell membership table (cell.membership / mc_data@misc$cell_membership$membership)
- Cell type annotation labels (e.g. sc_data$celltype)

## Outputs

- Metacell expression matrix (aggregated mean gene expression per metacell) as a Seurat object
- Metacell QC metrics per metacell: purity, compactness, separation, INV
- Metacell projection on the single-cell UMAP
- Weighted vs unweighted metacell-level DE/DA/PCA comparison figures (e.g. Fig 7A-D equivalent PNGs in './MetacellAnalysisToolkit_vignette_files/')

## How to apply

Build metacells from a normalized scRNA-seq object (NormalizeData with 'LogNormalize', variable feature selection, scaling, PCA, then a diffusion map via get_diffusion_comp on dims 1:30) using the MATK command line tool, which wraps SEACells, SuperCell or MetaCell2 to identify metacells. Convert the resulting cell-to-metacell membership table (cell_membership) into a Seurat metacell object of aggregated mean expression profiles. Compute metacell QC metrics with mc_purity (proportion of the most abundant cell type), mc_compactness (variance of components within the metacell), mc_separation (distance to closest metacell) and mc_INV (mean-normalized variance of gene expression within the metacell), and visualize with qc_boxplot; project metacells back onto the single-cell UMAP with mc_projection. Rationale: metacells sum profiles of cells of the same state to reduce sampling variance, but metacells are not cell types and each metacell may contain different numbers of cells, so downstream differential expression and differential abundance must be run in two modes — unweighted and weighted by metacell size (number of constituent single cells) — and validated against a single-cell-level reference. Unweighted analyses can yield type I errors (e.g. Gene B) or type II errors (e.g. Gene C) in differential expression (Fig 7B equivalent).

## Related tools

- **MetacellAnalysisToolkit** (Integrated pipeline (MATK CLI + R package) to build metacells with SEACells, SuperCell or MetaCell2 and perform QC/visualization of the aggregated metacells) — https://github.com/GfellerLab/MetacellAnalysisToolkit
- **Seurat** (R framework used to normalize single-cell data, aggregate expression into metacell Seurat objects, and run weighted/unweighted metacell-level differential expression) — https://github.com/GfellerLab/MetacellAnalysisToolkit
- **SEACells** (One of the metacell identification methods selectable through MATK (archetypal analysis on a nearest-neighbor kernel)) — https://github.com/dpeerlab/SEACells
- **SuperCell** (One of the metacell identification methods selectable through MATK) — https://github.com/GfellerLab/SuperCell
- **MCView** (Shiny-app for interactive exploration and annotation of the resulting metacell models) — https://github.com/tanaylab/MCView

## Examples

```
mc_data <- MetacellAnalysisToolkit::CD34_mc; sc_data <- MetacellAnalysisToolkit::CD34_sc; mc_data$purity <- mc_purity(membership = mc_data@misc$cell_membership$membership, annotation = sc_data$celltype)
```

## Evaluation signals

- Metacell purity is computed via mc_purity from the cell_membership table and celltype annotation, and inspected with qc_boxplot(mc.obj = mc_data, qc.metrics = "purity"); low-purity metacells indicate mixed states.
- Weighted metacell-level DE and DA results reproduce the single-cell-level reference results.
- Unweighted metacell-level analyses, if run as a control, display the expected artifacts: spurious DE calls (Gene B, type I error) and missed calls (Gene C, type II error) as in Fig 7B.
- Every single cell appears in the cell-to-metacell membership table with exactly one membership assignment feeding mc_compactness, mc_separation and mc_INV.
- Comparison figures render as PNGs in the vignette figure directory (fig.path = './MetacellAnalysisToolkit_vignette_files/').

## Limitations

- Metacell sizes vary (different numbers of constituent cells); ignoring size in downstream analyses creates distorted abundance estimates and differential expression artifacts.
- A metacell is not a cell type or a parametric model of the cell state — it is a robust description of one state; over-sampled states produce multiple metacells.
- Constructing high-quality metacells requires supervised choice of excluded genes/cells (e.g. mitochondrial genes, low-UMI cells) based on prior knowledge.
- tanaylab/metacells version 0.9 breaks compatibility with v0.8 APIs and result annotation names; a conversion script is required to migrate existing metacell data.
- The article's discussion notes 'No changelog found' for MetacellAnalysisToolkit, limiting version-change traceability.

## Evidence

- [abstract] metacells have been introduced to reduce the size and complexity of single-cell genomics data while preserving biologically relevant information and improving interpretability: "metacells have been introduced to reduce the size and complexity of single-cell genomics data while preserving biologically relevant information and improving interpretability"
- [methods] mc_purity membership call: "mc_data$purity <- mc_purity(membership = mc_data@misc$cell_membership$membership, annotation = sc_data$celltype)"
- [methods] mc_projection call: "we can project the metacells on the single-cell UMAP representation using the `mc_projection()` function"
- [readme] Metacells reduce sampling variance: [evidence span withheld — anchor status: external_doc]
- [readme] MATK tool description: "Toolkit for metacell analysis. It consists of the `MATK` command line tool to easily identify metacells"
