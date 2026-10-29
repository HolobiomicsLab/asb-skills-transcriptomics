---
name: paired-cluster-marker-gene-identification
description: Use when scRNA-seq data are already clustered and you need reliable per-cluster
  marker genes to feed cell type annotation, especially when pseudo marker genes from
  one-against-all tests would corrupt downstream labels.
when_to_use_negative:
- Clusters have not yet been computed — annotation and marker identification in scCATCH
  operate on pre-identified clusters, not raw cells
- You need single-cell-level annotation rather than cluster-level annotation — cell-based
  methods (SingleR, CellAssign) address a different problem where multiple cell types
  may coexist in one cluster
- Highly expressed genes must be identified by a method other than the default; the
  wiki documents alternate methods, but results may diverge from the paper's Wilcoxon-based
  criteria
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: Provides createscCATCH() and findmarkergene(), which implement the paired
    one-against-all cluster marker gene identification with the paper's thresholds
  repo: https://github.com/ZJUFanLab/scCATCH
- name: cellmatch
  role: Tissue-specific cell taxonomy reference database (353 cell types, 686 subtypes,
    184 tissue types, 2,096 human/mouse references) against which the identified markers
    are matched
  repo: https://github.com/ZJUFanLab/scCATCH/tree/master/data
- name: rev_gene
  role: scCATCH utility for cross-species gene-symbol conversion of marker genes via
    geneinfo when mouse markers must be mapped to human symbols
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:f13e4d29b0bca8c69e39fa6fbbe91c88f80addf162535af8b97f3d7b25b0950d
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
  - cellmatch
  - rev_gene
  - scCATCH
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# paired-cluster-marker-gene-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies marker genes for each pre-computed scRNA-seq cluster via scCATCH's findmarkergene(), which uses paired one-against-all cluster comparisons (instead of Seurat-style one-against-all methods) so that only genes significantly upregulated in a single cluster are retained, avoiding pseudo marker genes that cause incorrect annotation.

## When to use

Use when scRNA-seq data are already clustered and you need reliable per-cluster marker genes to feed cell type annotation, especially when pseudo marker genes from one-against-all tests would corrupt downstream labels.

## When NOT to use

- Clusters have not yet been computed — annotation and marker identification in scCATCH operate on pre-identified clusters, not raw cells
- You need single-cell-level annotation rather than cluster-level annotation — cell-based methods (SingleR, CellAssign) address a different problem where multiple cell types may coexist in one cluster
- Highly expressed genes must be identified by a method other than the default; the wiki documents alternate methods, but results may diverge from the paper's Wilcoxon-based criteria

## Inputs

- scRNA-seq count matrix (e.g. demo_data or benchmark Chen/Xin/Gierahn count matrices)
- Character vector of pre-computed cluster labels for each cell
- Species (human or mouse), tissue, and optionally cancer specifications
- geneinfo table for cross-species gene-symbol conversion (e.g. demo_geneinfo)

## Outputs

- Per-cluster marker gene list satisfying the paired-comparison criteria
- Updated scCATCH object carrying marker genes for downstream findcelltype() scoring

## How to apply

Build an scCATCH object with createscCATCH(data, cluster) from the count matrix and cluster vector, then run findmarkergene(obj, species, marker, tissue, cancer). The function performs paired comparison of every cluster against every other cluster, accepting a gene as a marker only if it is significantly upregulated in one cluster (log10 fold change >= 0.25, p < 0.05 by Wilcoxon test) and expressed in >= 25% of cells in that cluster. Set the species and tissue arguments (and cancer where relevant) so tissue-specific marker context is respected; use rev_gene(data, data_type, species, geneinfo) for cross-species gene-symbol conversion when needed. The rationale, per the paper, is that one-against-all methods can yield pseudo marker genes upregulated in at least two clusters, leading to incorrect cell type annotation, whereas the paired strategy restricts markers to genes specific to a single cluster before matching against CellMatch.

## Related tools

- **scCATCH** (Provides createscCATCH() and findmarkergene(), which implement the paired one-against-all cluster marker gene identification with the paper's thresholds) — https://github.com/ZJUFanLab/scCATCH
- **cellmatch** (Tissue-specific cell taxonomy reference database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human/mouse references) against which the identified markers are matched) — https://github.com/ZJUFanLab/scCATCH/tree/master/data
- **rev_gene** (scCATCH utility for cross-species gene-symbol conversion of marker genes via geneinfo when mouse markers must be mapped to human symbols) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species = 'Human', marker = 'cellmatch', tissue = 'Pancreas', cancer = 'Pancreatic cancer')
```

## Evaluation signals

- Every returned marker gene meets all three thresholds: log10 fold change >= 0.25, Wilcoxon p < 0.05, and expression in >= 25% of cells in the cluster
- Each marker gene is upregulated in exactly one cluster under the paired comparison — no gene significant in two or more clusters is retained
- Downstream findcelltype() on benchmark datasets (Chen kidney, Xin pancreatic islet, Gierahn peripheral blood) yields a 100% cluster-level consistency rate with literature-reported cell types (Table 1, Figure 2)
- Marker output matches the demo_marker reference object shipped with scCATCH when run on demo_data with equivalent parameters

## Limitations

- Performance of the overall annotation depends heavily on the reference database; a limited CellMatch can lead to incorrect cell type assignment even with correct marker genes
- Inadequate upstream clustering (too many or too few clusters) introduces errors into marker identification and annotation
- Effects of multiple clustering algorithms on marker identification and annotation were not evaluated by the authors

## Evidence

- [intro] a bunch of pseudo marker genes (significantly upregulated at least two clusters rather than in one cluster) may occur, which would lead to incorrect cell type annotation: [evidence span withheld — anchor status: not_found]
