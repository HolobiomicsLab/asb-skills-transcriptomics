---
name: cell-type-recovery-assessment-across-graining-levels
description: Use when selecting a graining level for metacell construction and you
  need to check that rare cell populations are not merged into other types, especially
  in datasets of varying complexity or size.
when_to_use_negative:
- The goal is metacell construction quality (compactness, separation, INV) rather
  than cell-type recovery — use the QC metric suite instead.
- Single-cell data lack a cell-type annotation, since purity requires sc_data$celltype
  to score metacell homogeneity.
- Data are already at metacell level (e.g. an existing MCView or h5ad metacell model)
  with no access to the underlying single-cell membership.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MetacellAnalysisToolkit
  role: Provides the CD34_sc/CD34_mc example data, metacell construction interface
    (SuperCell/SEACells backends), mc_purity and qc_boxplot functions
  repo: github.com/GfellerLab/MetacellAnalysisToolkit
- name: Seurat
  role: 'Preprocessing of single-cell data: NormalizeData (LogNormalize), FindVariableFeatures
    (nfeatures=2500), ScaleData, RunPCA, RunUMAP to obtain the latent embedding'
- name: SuperCell
  role: Optional backend for building metacells across the graining-level grid
  repo: github.com/GfellerLab/SuperCell
- name: SEACells
  role: Optional backend for building metacells across the graining-level grid
  repo: github.com/dpeerlab/SEACells
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:28cafe365219df86a13945675d0c0a00c2fa28450ce55b53d6f743eff23e9833
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetacellAnalysisToolkit
  - SEACells
  - Seurat
  - SuperCell
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cell-type-recovery-assessment-across-graining-levels

## Summary

A graining-level sweep that builds metacells from single-cell data at multiple coarseness settings and quantifies how well cell types are recovered via metacell purity, revealing when rare populations are merged away in complex datasets.

## When to use

Use when selecting a graining level for metacell construction and you need to check that rare cell populations are not merged into other types, especially in datasets of varying complexity or size.

## When NOT to use

- The goal is metacell construction quality (compactness, separation, INV) rather than cell-type recovery — use the QC metric suite instead.
- Single-cell data lack a cell-type annotation, since purity requires sc_data$celltype to score metacell homogeneity.
- Data are already at metacell level (e.g. an existing MCView or h5ad metacell model) with no access to the underlying single-cell membership.

## Inputs

- Single-cell expression matrix or Seurat object (e.g. MetacellAnalysisToolkit::CD34_sc) with a celltype annotation column
- Reference metacell object providing the cell_membership annotation format (e.g. MetacellAnalysisToolkit::CD34_mc)
- Grid of graining-level parameter values spanning coarse to fine average metacell sizes

## Outputs

- graining_scan_summary.csv — per-graining-level table of distinct cell types represented and purity statistics
- Per-metacell purity values computed via mc_purity
- QC boxplots of purity distributions per graining level (qc_boxplot)

## How to apply

Load the packaged example data with MetacellAnalysisToolkit (CD34_sc single cells and CD34_mc reference metacell object supplying the cell_membership annotation format), then preprocess the single-cell data with Seurat: NormalizeData (LogNormalize), FindVariableFeatures (nfeatures=2500), ScaleData, RunPCA (npcs=50) and RunUMAP (dims=1:50, n.neighbors=15, min.dist=0.5) to obtain the latent embedding. Using the MetacellAnalysisToolkit construction interface (SuperCell or SEACells backends), build metacells at a systematic grid of graining levels so average metacell size spans coarse to fine. At each level, assign every metacell to its most abundant cell category and compute purity with mc_purity(membership, annotation = sc_data$celltype). Tabulate the number of distinct cell types represented per graining level alongside purity statistics, visualize purity distributions with qc_boxplot(mc.obj, qc.metrics='purity'), and write the summary to graining_scan_summary.csv. Expect accurate recovery at relatively large graining levels in low-complexity datasets, but more complex datasets at the same graining level will miss rare cell populations as they merge with others.

## Related tools

- **MetacellAnalysisToolkit** (Provides the CD34_sc/CD34_mc example data, metacell construction interface (SuperCell/SEACells backends), mc_purity and qc_boxplot functions) — github.com/GfellerLab/MetacellAnalysisToolkit
- **Seurat** (Preprocessing of single-cell data: NormalizeData (LogNormalize), FindVariableFeatures (nfeatures=2500), ScaleData, RunPCA, RunUMAP to obtain the latent embedding)
- **SuperCell** (Optional backend for building metacells across the graining-level grid) — github.com/GfellerLab/SuperCell
- **SEACells** (Optional backend for building metacells across the graining-level grid) — github.com/dpeerlab/SEACells

## Examples

```
mc_data$purity <- mc_purity(membership = mc_data@misc$cell_membership$membership, annotation = sc_data$celltype); qc_boxplot(mc.obj = mc_data, qc.metrics = "purity")
```

## Evaluation signals

- Purity values per metacell computed with mc_purity lie in [0,1], with high purity indicating the metacell is dominated by one cell type.
- The summary table records the number of distinct cell types represented at each graining level; this count should decrease as graining becomes coarser, reflecting merging of rare populations.
- qc_boxplot(mc.obj, qc.metrics='purity') renders purity distributions; boxplots shifting downward or widening at coarse graining signal loss of cell-type recovery.
- All graining levels run through the same Seurat embedding (PCA npcs=50, UMAP dims=1:50) so comparisons across levels are on the identical latent space.
- Output file graining_scan_summary.csv exists with one row per graining level and purity statistic columns.

## Limitations

- Cell types are accurately recovered at relatively large graining levels only in low-complexity datasets; applying the same graining level to more complex datasets misses rare cell populations as they merge with other populations.
- Purity measures only the proportion of the most abundant cell type within a metacell, so partial mixing of minor populations within otherwise-pure metacells is not fully characterized.

## Evidence

- [methods] Purity definition: "the **purity** of a metacell is the proportion of the most abundant cell type within the metacell"
- [methods] mc_purity call: "mc_data$purity <- mc_purity(membership = mc_data@misc$cell_membership$membership, annotation = sc_data$celltype)"
- [methods] Seurat preprocessing: "sc_data <- NormalizeData(sc_data, normalization.method = "LogNormalize")"
- [methods] qc_boxplot usage: "qc_boxplot(mc.obj = mc_data, qc.metrics = "purity")"
- [abstract] Toolkit purpose: "a fully integrated pipeline to rapidly build, visualize and evaluate metacells with different methods"
- [readme] Toolkit README description: "It consists of the `MATK` command line tool to easily identify metacells with either [SEACells](https://github.com/dpeerlab/SEACells), [SuperCell](https://github.com/GfellerLab/SuperCell)"
