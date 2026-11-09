---
name: cross-dataset-benchmark-evaluation
description: Use when validating an automatic cluster annotation method on several
  independent published scRNA-seq datasets with known manual annotations, to report
  per-dataset consistency rates and an overall average.
when_to_use_negative:
- Evaluating annotation accuracy on a single dataset with no published manual annotations
  available for comparison
- The goal is tool-to-tool comparison of annotation methods rather than validating
  one tool against published manual labels (use the scCATCH_performance_comparison
  comparison workflow for that)
- Datasets whose tissues or species are not covered by the CellMatch reference database,
  since the limited reference database potentially leads to incorrect annotation
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scCATCH
  role: Annotates clusters automatically via createscCATCH(), findmarkergene() and
    findcelltype() using the CellMatch database; the tool whose accuracy is being
    benchmarked across datasets
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Used for clustering each dataset to obtain the per-cluster labels fed to createscCATCH();
    scCATCH objects can be created directly from a Seurat object's RNA @data and Idents
- name: CellMatch
  role: Tissue-specific cell taxonomy reference database against which cluster marker
    genes are scored to assign the top-1 cell type per cluster
  repo: https://github.com/ZJUFanLab/scCATCH/tree/master/data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:328b751f197709a7a91f1417b8206690d99ce716e986429ba5290529facf1c96
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
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-dataset-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a cell-type annotation tool's accuracy across multiple independent published scRNA-seq datasets by comparing automatic cluster annotations against published manual annotations and computing per-dataset cluster-consistency rates plus an average, as scCATCH did across Enge, Wu, Lindsey, Zheng, Zeisel and Heng (67%-100%, average 83%).

## When to use

Use when validating an automatic cluster annotation method on several independent published scRNA-seq datasets with known manual annotations, to report per-dataset consistency rates and an overall average.

## When NOT to use

- Evaluating annotation accuracy on a single dataset with no published manual annotations available for comparison
- The goal is tool-to-tool comparison of annotation methods rather than validating one tool against published manual labels (use the scCATCH_performance_comparison comparison workflow for that)
- Datasets whose tissues or species are not covered by the CellMatch reference database, since the limited reference database potentially leads to incorrect annotation

## Inputs

- Published scRNA-seq count/expression matrices (Enge, Wu, Lindsey, Zheng, Zeisel, Heng)
- Per-dataset cluster assignments (e.g., from Seurat clustering)
- Published manual cell type annotations for each dataset's clusters
- CellMatch tissue-specific cell taxonomy reference database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human/mouse references)

## Outputs

- Per-dataset cluster-consistent annotation rates
- Table 2-equivalent multi-dataset consistency table (consistency_table.csv) with 67%-100% per-dataset range and ~83% average
- scCATCH objects with per-cluster cell type annotations from findcelltype()

## How to apply

Download the published scRNA-seq datasets to be benchmarked (here Enge, Wu, Lindsey, Zheng, Zeisel, Heng — three recorded in CellMatch, three unrecorded) and cluster each (e.g., with Seurat) to obtain per-cluster labels. Build an scCATCH object per dataset with createscCATCH(data = sc_data, cluster = sc_cluster), then identify cluster marker genes with findmarkergene(obj, species, marker, tissue, cancer) using the criteria log10 fold change >= 0.25, p < 0.05 (Wilcoxon), expression in >= 25% of cells. Annotate each cluster against the CellMatch reference database with findcelltype(obj), assigning the top-scoring (top 1) cell type per cluster. Compare automatic annotations against the published manual annotations per dataset and compute a cluster-consistent rate for each; assemble the six per-dataset rates into a Table 2-equivalent table (consistency_table.csv). Judgement: each dataset's rate must fall in the reported 67%-100% range and the multi-dataset average must approximate 83%.

## Related tools

- **scCATCH** (Annotates clusters automatically via createscCATCH(), findmarkergene() and findcelltype() using the CellMatch database; the tool whose accuracy is being benchmarked across datasets) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Used for clustering each dataset to obtain the per-cluster labels fed to createscCATCH(); scCATCH objects can be created directly from a Seurat object's RNA @data and Idents)
- **CellMatch** (Tissue-specific cell taxonomy reference database against which cluster marker genes are scored to assign the top-1 cell type per cluster) — https://github.com/ZJUFanLab/scCATCH/tree/master/data

## Examples

```
obj <- createscCATCH(data = Seurat_obj[['RNA']]@data, cluster = as.character(Idents(Seurat_obj))); obj <- findmarkergene(obj, species = 'Human', marker = cellmatch, tissue = '', cancer = ''); obj <- findcelltype(obj)
```

## Evaluation signals

- Each dataset's cluster-consistent rate falls within the reported 67%-100% range
- The average consistency rate across the six datasets equals approximately 83%
- Per-cluster annotations match the top-1 cell type assigned by findcelltype() based on CellMatch scoring
- Marker genes passed the filter criteria: log10 fold change >= 0.25, p < 0.05, expressed in >= 25% of cells
- The assembled table reproduces the Table 2 structure (internal + external datasets, per-dataset rates plus average) and is saved as consistency_table.csv

## Limitations

- scCATCH performance majorly depends on the reference database; a limited CellMatch database potentially led to incorrect annotation of cell types
- Inadequate clustering analysis (too many or too few clusters) can introduce errors into cell type annotation
- Datasets not recorded in CellMatch may show lower consistency than recorded ones

## Evidence

- [readme] README usage block: [evidence span withheld — anchor status: not_found]
