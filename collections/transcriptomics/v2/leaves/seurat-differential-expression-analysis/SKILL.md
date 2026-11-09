---
name: seurat-differential-expression-analysis
description: Use when investigating why one-against-all differential expression produces
  pseudo marker genes shared across clusters, or when benchmarking cluster-marker-based
  annotation accuracy against manually assigned cell type labels on scRNA-seq data.
when_to_use_negative:
- When the goal is accurate cell type annotation rather than diagnosing marker gene
  identification failure — scCATCH's evidence-based scoring is the alternative being
  benchmarked against
- When clusters are defined by paired (one-against-one) comparisons, since pseudo
  marker genes arise specifically from one-against-all methods
- When the dataset is not clustered — inadequate clustering (too many or few cells
  per cluster) introduces errors into cell type annotation
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scCATCH
  role: 'Framework wrapping the comparison: createscCATCH() builds the object, findmarkergene()
    runs one-against-all marker identification, findcelltype() annotates clusters
    against CellMatch'
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Provides the one-against-all Wilcoxon rank-sum test for cluster marker gene
    identification whose pseudo marker gene output is being reproduced
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:e5d7f6bcfba7ac554d90361aa88a42a83b1ccc41ebabd9de68d65a04ba7e0d13
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
  - Seurat
  - scCATCH
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# seurat-differential-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce Seurat's one-against-all Wilcoxon rank-sum marker gene identification on the Gierahn PBMC dataset to demonstrate how pseudo marker genes (genes upregulated in multiple clusters, e.g. CCL22, SWAP70, KLRF1) arise and degrade cell type annotation consistency.

## When to use

Use when investigating why one-against-all differential expression produces pseudo marker genes shared across clusters, or when benchmarking cluster-marker-based annotation accuracy against manually assigned cell type labels on scRNA-seq data.

## When NOT to use

- When the goal is accurate cell type annotation rather than diagnosing marker gene identification failure — scCATCH's evidence-based scoring is the alternative being benchmarked against
- When clusters are defined by paired (one-against-one) comparisons, since pseudo marker genes arise specifically from one-against-all methods
- When the dataset is not clustered — inadequate clustering (too many or few cells per cluster) introduces errors into cell type annotation

## Inputs

- Gierahn PBMC scRNA-seq count matrix with cluster labels (from scCATCH_performance_comparison tutorial materials)
- CellMatch tissue-specific cell taxonomy reference database (cellmatch)
- Manually assigned cell type labels per cluster

## Outputs

- pbmc_marker_genes.csv (per-cluster marker gene table including pseudo marker genes CCL22, SWAP70, KLRF1)
- pbmc_annotation_consistency.csv (per-cluster annotation agreement summary)

## How to apply

Load the Gierahn PBMC scRNA-seq count matrix with cluster labels from the scCATCH performance comparison tutorial materials and build the analysis object with createscCATCH(data, cluster). Run one-against-all paired cluster comparisons using Seurat's Wilcoxon rank-sum test via findmarkergene(), retaining genes with log10 fold change >= 0.25, p < 0.05, and expression in >= 25% of cells. Flag pseudo marker genes as those significantly upregulated in at least two clusters rather than one (e.g. CCL22, SWAP70, KLRF1), since the article shows such genes lead to incorrect cell type annotation. Annotate each cluster with findcelltype() against the CellMatch reference (353 cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references) and record which clusters match manually assigned labels. Write the marker gene table to pbmc_marker_genes.csv and the per-cluster annotation agreement summary to pbmc_annotation_consistency.csv.

## Related tools

- **scCATCH** (Framework wrapping the comparison: createscCATCH() builds the object, findmarkergene() runs one-against-all marker identification, findcelltype() annotates clusters against CellMatch) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Provides the one-against-all Wilcoxon rank-sum test for cluster marker gene identification whose pseudo marker gene output is being reproduced)

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species = 'human', marker = 'cellmarker', tissue = 'Blood', cancer = 'cancer'); obj <- findcelltype(obj)
```

## Evaluation signals

- pbmc_annotation_consistency.csv reproduces 40% cluster consistency, with only cluster 2 (T cells) and cluster 5 (monocytes) correctly annotated
- pbmc_marker_genes.csv contains the pseudo marker genes CCL22, SWAP70, and KLRF1, each significantly upregulated in at least two clusters
- Marker gene filtering retains only genes with log10 fold change >= 0.25, p < 0.05, and expression in >= 25% of cells
- Each cluster receives exactly one top-1 cell type assignment from findcelltype() scored against CellMatch
- Clusters flagged as incorrectly annotated correspond to clusters whose marker sets include multi-cluster pseudo marker genes

## Limitations

- The performance of scCATCH majorly depends on the reference database; a limited CellMatch database may itself lead to incorrect cell type annotation
- Inadequate clustering analysis (too many or few cells per cluster) introduces errors into cell type annotation independent of marker gene quality
- Findings are demonstrated on a single benchmark (Gierahn PBMC) with specific clusters; generalization to other tissues or clustering algorithms is not established

## Evidence

- [intro] Pseudo marker gene definition and consequence: [evidence span withheld — anchor status: not_found]
- [readme] scCATCH function workflow from README: [evidence span withheld — anchor status: not_found]
