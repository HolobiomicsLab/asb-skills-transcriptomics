---
name: multi-method-annotation-comparison-across-datasets
description: Use when evaluating whether a cluster-based annotation method (scCATCH)
  outperforms cell-based annotation tools on benchmark scRNA-seq datasets with published
  cluster assignments and manually assigned cell types.
when_to_use_negative:
- You need to annotate clusters without a reference comparison — scCATCH's performance
  depends on the CellMatch reference database, and a limited database can lead to
  incorrect annotation
- Your goal is annotating individual cells rather than clusters — the cell-based baselines
  (SingleR, CellAssign, Garnett, scMap, CHETAH) operate per cell and face the challenge
  that multiple cell types may be present in one cluster
- The dataset has no published cluster assignments or manually assigned ground-truth
  cell types, since the consistent-rate metric requires cluster-level agreement with
  manual annotation
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: Cluster-based annotation method under evaluation; constructs scCATCH objects
    and runs findmarkergene and findcelltype
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Baseline one-against-all marker-gene-based cluster annotation
- name: SingleR
  role: Cell-based annotation baseline
- name: CellAssign
  role: Cell-based annotation baseline
- name: Garnett
  role: Cell-based annotation baseline
- name: scMap
  role: Cell-based annotation baseline
- name: CHETAH
  role: Cell-based annotation baseline
- name: Matrix
  role: R package used to load the processed benchmark scRNA-seq data matrices
- name: devtools
  role: Installs scCATCH from GitHub
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:c389bc8dc2756bb74a540b8efb3a1503663c2bdf8a96d1bbae39e01a312742da
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multi-method-annotation-comparison-across-datasets

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks automatic cell-type annotation of scRNA-seq clusters by comparing scCATCH's cluster-based approach against Seurat's one-against-all marker method and cell-based methods (CellAssign, SingleR, Garnett, scMap, CHETAH) on the Chen, Xin, and Gierahn datasets, scoring each method by per-cluster and per-dataset consistent rates.

## When to use

Use when evaluating whether a cluster-based annotation method (scCATCH) outperforms cell-based annotation tools on benchmark scRNA-seq datasets with published cluster assignments and manually assigned cell types.

## When NOT to use

- You need to annotate clusters without a reference comparison — scCATCH's performance depends on the CellMatch reference database, and a limited database can lead to incorrect annotation
- Your goal is annotating individual cells rather than clusters — the cell-based baselines (SingleR, CellAssign, Garnett, scMap, CHETAH) operate per cell and face the challenge that multiple cell types may be present in one cluster
- The dataset has no published cluster assignments or manually assigned ground-truth cell types, since the consistent-rate metric requires cluster-level agreement with manual annotation

## Inputs

- Processed scRNA-seq count matrices for the Chen, Xin, and Gierahn benchmark datasets (loaded with the Matrix package in R)
- Published cluster assignments for each dataset
- Manually assigned cell-type labels per cluster (ground truth)
- CellMatch reference database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references)

## Outputs

- Per-cluster and per-dataset consistent rates for scCATCH, Seurat, CellAssign, SingleR, Garnett, scMap, and CHETAH
- table1_consistency_rates.csv consolidating all consistency rates
- Comparison against Table 1 of the article confirming agreement

## How to apply

Load the processed Chen, Xin, and Gierahn scRNA-seq benchmark datasets with their published cluster assignments using the Matrix package in R. Build scCATCH objects with createscCATCH(data, cluster), run findmarkergene with the paper's marker criteria (log10 fold change >= 0.25, Wilcoxon p < 0.05, expression in >= 25% of cells) against the CellMatch reference database, then findcelltype. Run the cell-based baselines SingleR, CellAssign, Garnett, scMap, and CHETAH plus Seurat's one-against-all marker-based annotation on the same cluster assignments. For each method and dataset, compute the consistent-rate as the fraction of clusters whose automatic annotation agrees with the manually assigned cell type, assemble the per-cluster and per-dataset rates into a single table (table1_consistency_rates.csv), and compare against Table 1 of the article to confirm agreement.

## Related tools

- **scCATCH** (Cluster-based annotation method under evaluation; constructs scCATCH objects and runs findmarkergene and findcelltype) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Baseline one-against-all marker-gene-based cluster annotation)
- **SingleR** (Cell-based annotation baseline)
- **CellAssign** (Cell-based annotation baseline)
- **Garnett** (Cell-based annotation baseline)
- **scMap** (Cell-based annotation baseline)
- **CHETAH** (Cell-based annotation baseline)
- **Matrix** (R package used to load the processed benchmark scRNA-seq data matrices)
- **devtools** (Installs scCATCH from GitHub)

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer); obj <- findcelltype(obj)
```

## Evaluation signals

- Consistent-rate per method per dataset equals the fraction of clusters whose automatic annotation matches the manually assigned cell type
- Garnett and CHETAH should show consistent rates as low as 0% on the Gierahn dataset, while scCATCH accurately annotates those clusters
- Assembled table1_consistency_rates.csv values match Table 1 of the article
- scCATCH's findmarkergene uses the paper's criteria: log10 fold change >= 0.25, Wilcoxon p < 0.05, expression in >= 25% of cells
- All methods are run on the same published cluster assignments so the comparison is controlled for clustering

## Limitations

- The performance of scCATCH majorly depends on the reference database; a limited reference database may lead to incorrect annotation of cell types
- Inadequate clustering analysis would introduce errors into cell type annotation, as too many or too few clusters are both problematic for labeling
- The study's consistent rates reflect the specific Chen, Xin, and Gierahn benchmark datasets and CellMatch version used

## Evidence

- [readme] The source code and results of performance comparison on the detail of the process among scCATCH, CellAssign, Garnett, SingleR, scMap and CHETAH, and CellMatch database: [evidence span withheld — anchor status: external_doc]
