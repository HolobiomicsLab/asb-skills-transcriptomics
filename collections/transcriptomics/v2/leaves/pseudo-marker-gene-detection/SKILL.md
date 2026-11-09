---
name: pseudo-marker-gene-detection
description: Use when scRNA-seq clusters are annotated via one-against-all marker
  gene tests (e.g. Seurat Wilcoxon) and you need to flag markers upregulated in multiple
  clusters that could cause incorrect or inconsistent cell type annotation.
when_to_use_negative:
- Cluster annotation already uses paired-comparison or evidence-based scoring approaches
  (e.g. scCATCH itself) that exclude genes upregulated in more than one cluster by
  design
- The dataset has only two clusters, so 'upregulated in at least two clusters' cannot
  be evaluated meaningfully
- The CellMatch reference database lacks coverage for the tissue of interest, since
  scCATCH performance majorly depends on the reference database and a limited reference
  can itself cause incorrect annotation independent of pseudo markers
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scCATCH
  role: Provides createscCATCH() to build the analysis object from count matrix +
    cluster labels, findmarkergene() to run marker identification, and findcelltype()
    to annotate clusters against CellMatch; used to detect pseudo markers and measure
    annotation consistency
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Source of the one-against-all Wilcoxon rank-sum test marker gene identification
    whose pseudo markers (upregulated in multiple clusters) are being detected and
    evaluated
- name: CellMatch
  role: Tissue-specific cellular taxonomy reference database used by findcelltype()
    to score and assign cell types per cluster, enabling measurement of annotation
    consistency
  repo: https://github.com/ZJUFanLab/scCATCH/tree/master/data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:3308e7ee70e968f4b4740788c81ebdccf0c7563041cdcf7fd9aa8ce7ef75c47c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scCATCH
  repo_url: https://github.com/ZJUFanLab/scCATCH
  tools:
  - CellMatch
  - Seurat
  - scCATCH
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# pseudo-marker-gene-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects pseudo marker genes — genes significantly upregulated in at least two clusters rather than one — that arise from Seurat-style one-against-all cluster marker identification in scRNA-seq data, and quantifies how they impair cell type annotation consistency.

## When to use

Use when scRNA-seq clusters are annotated via one-against-all marker gene tests (e.g. Seurat Wilcoxon) and you need to flag markers upregulated in multiple clusters that could cause incorrect or inconsistent cell type annotation.

## When NOT to use

- Cluster annotation already uses paired-comparison or evidence-based scoring approaches (e.g. scCATCH itself) that exclude genes upregulated in more than one cluster by design
- The dataset has only two clusters, so 'upregulated in at least two clusters' cannot be evaluated meaningfully
- The CellMatch reference database lacks coverage for the tissue of interest, since scCATCH performance majorly depends on the reference database and a limited reference can itself cause incorrect annotation independent of pseudo markers

## Inputs

- scRNA-seq count matrix (e.g. Gierahn PBMC dataset) with per-cell cluster labels
- Cluster assignment vector (character) for each cell
- CellMatch tissue-specific cell taxonomy reference database (353 cell types, 686 subtypes, 184 tissue types, 2096 human/mouse references)

## Outputs

- Marker gene table (e.g. pbmc_marker_genes.csv) including pseudo marker genes upregulated in >= 2 clusters (CCL22, SWAP70, KLRF1)
- Per-cluster annotation agreement summary (e.g. pbmc_annotation_consistency.csv) recording which clusters match manual cell type labels
- Cluster-level annotation consistency percentage (e.g. 40% on Gierahn PBMC, only clusters 2 and 5 correct)

## How to apply

Build an scCATCH object with createscCATCH(data, cluster) from the count matrix and cluster labels, then run one-against-all paired cluster comparisons with Seurat's Wilcoxon rank-sum test via findmarkergene(). Retain candidate markers using the paper's thresholds: log10 fold change >= 0.25, p < 0.05 (Wilcoxon), expressed in >= 25% of cells. Flag as pseudo marker genes any genes significantly upregulated in at least two clusters rather than one (e.g. CCL22, SWAP70, KLRF1 on the Gierahn PBMC dataset). Then annotate each cluster with findcelltype() against the CellMatch reference and record which clusters match manually assigned labels; pseudo markers explain mismatches because they push multiple clusters toward the same erroneous cell type. In the Gierahn PBMC case this reproduces only 40% annotation consistency, with clusters 2 (T cells) and 5 (monocytes) correctly annotated.

## Related tools

- **scCATCH** (Provides createscCATCH() to build the analysis object from count matrix + cluster labels, findmarkergene() to run marker identification, and findcelltype() to annotate clusters against CellMatch; used to detect pseudo markers and measure annotation consistency) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Source of the one-against-all Wilcoxon rank-sum test marker gene identification whose pseudo markers (upregulated in multiple clusters) are being detected and evaluated)
- **CellMatch** (Tissue-specific cellular taxonomy reference database used by findcelltype() to score and assign cell types per cluster, enabling measurement of annotation consistency) — https://github.com/ZJUFanLab/scCATCH/tree/master/data

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species = 'Human', marker = cellmatch, tissue = 'Peripheral blood', cancer = 'normal'); obj <- findcelltype(obj)
```

## Evaluation signals

- pbmc_marker_genes.csv contains the pseudo marker genes CCL22, SWAP70, and KLRF1, each flagged as significantly upregulated in at least two clusters
- pbmc_annotation_consistency.csv reproduces 40% cluster annotation consistency on the Gierahn PBMC dataset, with only cluster 2 (T cells) and cluster 5 (monocytes) correctly annotated
- All retained marker genes meet the filter: log10 fold change >= 0.25, Wilcoxon p < 0.05, and expression in >= 25% of cells
- Each cluster receives a cell type assignment from findcelltype() against CellMatch, and mismatched clusters can be explained by shared pseudo marker genes
- The scCATCH object is created successfully via createscCATCH(data = sc_data, cluster = sc_cluster) before any downstream steps run

## Limitations

- scCATCH performance majorly depends on the reference database; a limited CellMatch database can itself lead to incorrect cell type annotation, confounding pseudo-marker-driven errors
- Inadequate clustering analysis (too many or too few clusters) would introduce errors into cell type annotation and change which pseudo markers appear
- Findings demonstrated here are on the Gierahn PBMC dataset with manual labels as ground truth; consistency percentages may not transfer to other tissues or datasets

## Evidence

- [intro] Pseudo marker genes defined as genes upregulated in at least two clusters causing incorrect annotation: [evidence span withheld — anchor status: not_found]
- [results] Gierahn PBMC annotation consistency result: only clusters 2 (T cells) and 5 (monocytes) correctly annotated (40%): [evidence span withheld — anchor status: not_found]
- [methods] scCATCH core functions create object, find markers, and find cell types: [evidence span withheld — anchor status: not_found]
