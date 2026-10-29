---
name: scrnaseq-marker-gene-identification
description: Use when clustering a scRNA-seq dataset (e.g., Chen, Xin, Gierahn benchmarks)
  requires per-cluster marker genes with explicit fold-change, significance, and expression-prevalence
  cutoffs to drive automatic cell-type annotation.
when_to_use_negative:
- When cells, not clusters, should be annotated individually — use cell-based methods
  (SingleR, CellAssign, Garnett, scMap, CHETAH) instead
- When the CellMatch reference database does not cover the tissue or cell type of
  interest, since a limited reference database can lead to incorrect annotation
- When clustering itself is unreliable — too many or too few clusters introduce errors
  into marker-based annotation
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3270
tools:
- name: scCATCH
  role: Provides createscCATCH, findmarkergene, and findcelltype to identify per-cluster
    marker genes and annotate cell types by evidence-based scoring against CellMatch
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Baseline marker-identification method using one-against-all cluster marker
    genes for comparison of annotation consistency
- name: SingleR
  role: Cell-based annotation baseline compared against cluster-marker-based annotation
- name: CellAssign
  role: Cell-based annotation baseline compared in the consistency-rate benchmark
- name: Garnett
  role: Cell-based annotation baseline compared in the consistency-rate benchmark
- name: scMap
  role: Cell-based annotation baseline compared in the consistency-rate benchmark
- name: CHETAH
  role: Cell-based annotation baseline compared in the consistency-rate benchmark
- name: devtools
  role: Used to install scCATCH from GitHub
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:1a6a7bc2bc0992508e2f68d5a8ad3f5ad61f1d225426bcf3d5fb3d07afee3f33
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
  - CHETAH
  - CellAssign
  - Garnett
  - Matrix
  - Seurat
  - SingleR
  - devtools
  - scCATCH
  - scMap
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3270
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# scrnaseq-marker-gene-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify putative marker genes for each cell cluster in scRNA-seq data using paired one-against-all differential testing, then use those markers for evidence-based cluster annotation against the CellMatch tissue-specific taxonomy database. This is the marker-gene step of the scCATCH workflow (findmarkergene) that underpins its comparison to Seurat's marker-based annotation.

## When to use

Use when clustering a scRNA-seq dataset (e.g., Chen, Xin, Gierahn benchmarks) requires per-cluster marker genes with explicit fold-change, significance, and expression-prevalence cutoffs to drive automatic cell-type annotation.

## When NOT to use

- When cells, not clusters, should be annotated individually — use cell-based methods (SingleR, CellAssign, Garnett, scMap, CHETAH) instead
- When the CellMatch reference database does not cover the tissue or cell type of interest, since a limited reference database can lead to incorrect annotation
- When clustering itself is unreliable — too many or too few clusters introduce errors into marker-based annotation

## Inputs

- scRNA-seq expression matrix (data) with gene-by-cell counts or normalized values
- Character vector of cluster assignments (sc_cluster) for each cell
- Species and tissue/cancer specification for findmarkergene
- CellMatch tissue-specific cell taxonomy reference database (cellmatch)

## Outputs

- Per-cluster list of marker genes meeting the log10FC >= 0.25, p < 0.05, >=25%-expression criteria
- Per-cluster cell-type annotations with evidence-based scores (top 1 type per cluster)
- Consistent-rate table comparing annotation agreement across methods (e.g., table1_consistency_rates.csv)

## How to apply

Construct an scCATCH object with createscCATCH(data, cluster) from the expression matrix and pre-assigned cluster labels. Run findmarkergene, which applies paired comparison of every cluster against all others so that genes significantly upregulated in only one cluster are retained; the article's criteria are log10 fold change >= 0.25, Wilcoxon test p < 0.05, and expression in more than a quarter of cells (>= 25%). Filter out pseudo marker genes (genes significantly upregulated in at least two clusters) since they cause incorrect annotation. Feed the validated marker genes into findcelltype, which scores cell types against the CellMatch reference database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human/mouse references) and assigns the top-1 scoring type per cluster. Benchmark the resulting annotations against Seurat's one-against-all marker approach and cell-based methods (SingleR, CellAssign, Garnett, scMap, CHETAH) by computing per-cluster and per-dataset consistent rates.

## Related tools

- **scCATCH** (Provides createscCATCH, findmarkergene, and findcelltype to identify per-cluster marker genes and annotate cell types by evidence-based scoring against CellMatch) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Baseline marker-identification method using one-against-all cluster marker genes for comparison of annotation consistency)
- **SingleR** (Cell-based annotation baseline compared against cluster-marker-based annotation)
- **CellAssign** (Cell-based annotation baseline compared in the consistency-rate benchmark)
- **Garnett** (Cell-based annotation baseline compared in the consistency-rate benchmark)
- **scMap** (Cell-based annotation baseline compared in the consistency-rate benchmark)
- **CHETAH** (Cell-based annotation baseline compared in the consistency-rate benchmark)
- **devtools** (Used to install scCATCH from GitHub) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer); obj <- findcelltype(obj)
```

## Evaluation signals

- Marker genes pass the article's thresholds: log10 fold change >= 0.25, Wilcoxon p < 0.05, expression in >= 25% of cells in the cluster
- Genes upregulated in two or more clusters (pseudo marker genes) are excluded from the marker set
- Per-cluster and per-dataset consistent rates (fraction of clusters whose automatic annotation matches manually assigned cell type) reproduce the values in Table 1 of the article
- scCATCH annotations achieve the reported range of 67%-100% accurate clusters (average 83%) across benchmark datasets
- Output CSV (e.g., table1_consistency_rates.csv) contains rows for scCATCH, Seurat, SingleR, CellAssign, Garnett, scMap, and CHETAH on the Chen, Xin, and Gierahn datasets

## Limitations

- Performance majorly depends on the reference database; a limited CellMatch database can lead to incorrect cell type annotation
- Inadequate clustering analysis (too many or too few clusters) introduces errors into annotation
- Multiple cell types may be present in one cluster, complicating annotation; pseudo marker genes from one-against-all testing cause incorrect assignment
- Effects of multiple clustering algorithms on annotation were not evaluated by the authors

## Evidence
