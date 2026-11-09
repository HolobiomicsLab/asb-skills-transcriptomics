---
name: scrnaseq-cluster-benchmarking
description: Use when evaluating or comparing automatic cell type annotation of scRNA-seq
  clusters (scCATCH vs Seurat-based one-against-all markers), especially to quantify
  pseudo marker gene effects on annotation consistency.
when_to_use_negative:
- When the goal is de novo cell type annotation of a single dataset rather than benchmarking
  or reproducing the performance comparison between annotation methods
- When cluster labels have not been precomputed — the workflow requires an already-clustered
  count matrix passed to createscCATCH()
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: 'Core toolkit: createscCATCH() builds the object from count matrix + cluster
    labels, findmarkergene() runs cluster marker identification, findcelltype() annotates
    clusters against CellMatch'
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: 'Baseline method: its one-against-all Wilcoxon rank-sum test marker gene identification
    is benchmarked to expose pseudo marker gene failure'
- name: CellMatch
  role: Tissue-specific cellular taxonomy reference database used by findcelltype()
    for evidence-based scoring and annotation
  repo: https://github.com/ZJUFanLab/scCATCH/tree/master/data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:7ad9608a85ddf34ac0534bdd65eac13b91250cee917e415f179d6a344afa891c
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# scrnaseq-cluster-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that reproduces scCATCH's performance comparison against Seurat's one-against-all marker gene identification on scRNA-seq clusters, quantifying how pseudo marker genes (e.g. CCL22, SWAP70) impair cell type annotation consistency on datasets like the Gierahn PBMC matrix.

## When to use

Use when evaluating or comparing automatic cell type annotation of scRNA-seq clusters (scCATCH vs Seurat-based one-against-all markers), especially to quantify pseudo marker gene effects on annotation consistency.

## When NOT to use

- When the goal is de novo cell type annotation of a single dataset rather than benchmarking or reproducing the performance comparison between annotation methods
- When cluster labels have not been precomputed — the workflow requires an already-clustered count matrix passed to createscCATCH()

## Inputs

- scRNA-seq count matrix with cluster labels (e.g. Gierahn PBMC dataset from scCATCH_performance_comparison tutorial materials)
- Manually assigned cell type labels per cluster (ground truth for consistency calculation)
- CellMatch tissue-specific cell taxonomy reference database (cellmatch)

## Outputs

- Per-cluster marker gene table (e.g. pbmc_marker_genes.csv) including flagged pseudo marker genes
- Per-cluster annotation agreement summary (e.g. pbmc_annotation_consistency.csv) reporting annotation consistency percentage
- List of pseudo marker genes upregulated in >= 2 clusters

## How to apply

Load the target scRNA-seq count matrix with precomputed cluster labels (e.g. the Gierahn PBMC dataset from the scCATCH performance comparison tutorial materials) and build the analysis object with createscCATCH(data, cluster). Run marker gene identification with findmarkergene() using Seurat's Wilcoxon rank-sum test with log10 fold change >= 0.25, p < 0.05, and expression in >= 25% of cells. Flag pseudo marker genes that are significantly upregulated in at least two clusters rather than one (e.g. CCL22, SWAP70, KLRF1), since the paper shows these cause incorrect annotation. Annotate each cluster via findcelltype() against the CellMatch reference (353 cell types, 686 subtypes, 184 tissue types, 2,096 human/mouse references) and record which clusters match manually assigned labels. Write the marker gene table (e.g. pbmc_marker_genes.csv) and per-cluster annotation agreement summary (e.g. pbmc_annotation_consistency.csv) for validation against known consistency rates (e.g. 40% on the Gierahn PBMC data, with only clusters 2 and 5 correctly annotated).

## Related tools

- **scCATCH** (Core toolkit: createscCATCH() builds the object from count matrix + cluster labels, findmarkergene() runs cluster marker identification, findcelltype() annotates clusters against CellMatch) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Baseline method: its one-against-all Wilcoxon rank-sum test marker gene identification is benchmarked to expose pseudo marker gene failure)
- **CellMatch** (Tissue-specific cellular taxonomy reference database used by findcelltype() for evidence-based scoring and annotation) — https://github.com/ZJUFanLab/scCATCH/tree/master/data

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species = 'Human', marker = 'cellmatch', tissue = 'Blood', cancer = 'hematopoietic system neoplasm'); obj <- findcelltype(obj)
```

## Evaluation signals

- Annotation consistency on the Gierahn PBMC dataset must reproduce 40% cluster consistency, with only cluster 2 (T cells) and cluster 5 (monocytes) correctly annotated
- pbmc_marker_genes.csv must contain the pseudo marker genes CCL22, SWAP70, and KLRF1
- Pseudo marker genes must be significantly upregulated in at least two clusters rather than one
- Marker gene filters must match: log10 fold change >= 0.25, p < 0.05 (Wilcoxon), expressed in >= 25% of cells
- pbmc_annotation_consistency.csv must record per-cluster agreement with manually assigned cell type labels

## Limitations

- The performance of scCATCH majorly depends on the reference database; a limited CellMatch database potentially leads to incorrect cell type annotation
- Inadequate clustering analysis (too many or too few cells per cluster) introduces errors into cell type annotation
- Effects of multiple clustering algorithms on cell type annotations were not yet evaluated in the study

## Evidence

- [intro] Pseudo marker gene definition: [evidence span withheld — anchor status: not_found]
- [methods] scCATCH workflow functions: [evidence span withheld — anchor status: not_found]
