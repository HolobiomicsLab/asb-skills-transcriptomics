---
name: metacell-purity-quantification
description: Use when metacells have been constructed (SuperCell/SEACells backends)
  and you need to check whether each metacell corresponds to a single cell type, e.g.
  when tuning graining level or comparing datasets of differing complexity.
when_to_use_negative:
- Before metacells are actually constructed — purity requires a cell-to-metacell membership
  mapping, not just a PCA/UMAP embedding
- When no per-cell type annotations exist for the dataset, since purity is computed
  against an annotation vector such as sc_data$celltype
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MetacellAnalysisToolkit
  role: Provides mc_purity and qc_boxplot for computing and visualizing metacell purity,
    plus the CD34_sc/CD34_mc example data
  repo: github.com/GfellerLab/MetacellAnalysisToolkit
- name: Seurat
  role: Preprocessing and hosting the single-cell object with the celltype annotation
    used as reference for purity
- name: SuperCell
  role: Metacell construction backend whose memberships are evaluated by purity
  repo: github.com/GfellerLab/SuperCell
- name: SEACells
  role: Alternative metacell construction backend whose memberships are evaluated
    by purity
  repo: github.com/dpeerlab/SEACells
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:edac20bcb7f3425599cbb62fb812ebcdcf0c5b398b1b4916d3ee42e1197e799d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetacellAnalysisToolkit
  - SEACells
  - Seurat
  - SuperCell
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# metacell-purity-quantification

## Summary

Quantifies metacell purity — the proportion of the most abundant cell type within each metacell — using the MetacellAnalysisToolkit mc_purity function, to evaluate how faithfully metacells preserve cell-type identity (e.g., rare populations lost at coarse graining levels).

## When to use

Use when metacells have been constructed (SuperCell/SEACells backends) and you need to check whether each metacell corresponds to a single cell type, e.g. when tuning graining level or comparing datasets of differing complexity.

## When NOT to use

- Before metacells are actually constructed — purity requires a cell-to-metacell membership mapping, not just a PCA/UMAP embedding
- When no per-cell type annotations exist for the dataset, since purity is computed against an annotation vector such as sc_data$celltype

## Inputs

- Seurat single-cell object with metacell membership annotations (e.g. mc_data@misc$cell_membership$membership)
- Per-cell type annotation column (e.g. sc_data$celltype)
- Packaged example data MetacellAnalysisToolkit::CD34_sc and reference metacell object CD34_mc

## Outputs

- Per-metacell purity values (stored e.g. as mc_data$purity)
- qc_boxplot figure of purity distributions across metacells
- Summary table of cell-type counts and purity per graining level (e.g. graining_scan_summary.csv)

## How to apply

Load the packaged CD34_sc single-cell object and CD34_mc reference metacell object from MetacellAnalysisToolkit, which provide the cell_membership annotation format. After building metacells, compute purity per metacell with mc_purity, passing the membership vector and the per-cell annotation column (e.g. sc_data$celltype); the purity of a metacell is defined as the proportion of the most abundant cell type within the metacell. Assign each metacell to its most abundant cell category, tabulate the number of distinct cell types represented alongside purity statistics at each graining level, and visualize purity distributions with qc_boxplot(mc.obj, qc.metrics="purity"). Low purity or shrinking cell-type counts at coarse graining indicates rare populations being merged into other populations.

## Related tools

- **MetacellAnalysisToolkit** (Provides mc_purity and qc_boxplot for computing and visualizing metacell purity, plus the CD34_sc/CD34_mc example data) — github.com/GfellerLab/MetacellAnalysisToolkit
- **Seurat** (Preprocessing and hosting the single-cell object with the celltype annotation used as reference for purity)
- **SuperCell** (Metacell construction backend whose memberships are evaluated by purity) — github.com/GfellerLab/SuperCell
- **SEACells** (Alternative metacell construction backend whose memberships are evaluated by purity) — github.com/dpeerlab/SEACells

## Examples

```
mc_data$purity <- mc_purity(membership = mc_data@misc$cell_membership$membership, annotation = sc_data$celltype)
```

## Evaluation signals

- mc_purity returns a numeric vector with one purity value per metacell; purity is bounded in [1/|celltypes in metacell|, 1], with pure single-type metacells at 1.0
- Cross-check with qc_boxplot(mc.obj, qc.metrics="purity") — most metacells should show high purity in a well-tuned model
- At each graining level, the number of distinct cell types represented should match the annotation's known repertoire; a drop indicates rare populations merged with others
- Sanity-check membership vectors against the CD34_mc reference format (mc_data@misc$cell_membership$membership) before computing purity

## Limitations

- Purity only captures the dominant cell type; mixed or doublet-like metacells of two populations still receive 0.5–1.0 values and may need complementary QC metrics
- High purity at coarse graining can be misleading: rare cell populations can be entirely merged away while remaining metacells stay pure
- Purity depends on the quality/completeness of the supplied annotation labels

## Evidence

- [methods] purity definition: "the **purity** of a metacell is the proportion of the most abundant cell type within the metacell"
- [methods] purity computation call: "mc_data$purity <- mc_purity(membership = mc_data@misc$cell_membership$membership, annotation = sc_data$celltype)"
- [methods] qc boxplot: "qc_boxplot(mc.obj = mc_data, qc.metrics = "purity")"
- [readme] toolkit repo: "It consists of the `MATK` command line tool to easily identify metacells with either [SEACells](https://github.com/dpeerlab/SEACells), [SuperCell](https://github.com/GfellerLab/SuperCell)"
