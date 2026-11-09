---
name: ensemble-feature-selection-single-cell-rnaseq
description: Use when a preprocessed, log-normalized cells x genes scRNA-seq matrix
  plus initial cluster labels are available and a compact gene subset (up to ~4000
  genes) is needed before rare-cell anomaly detection or clustering.
when_to_use_negative:
- The input is already a reduced feature table or a preselected gene set — re-selecting
  genes would discard information
- The data are raw counts that have not yet been filtered (genes expressed in fewer
  than three cells removed) and log-normalized with cell-specific size factors
- The goal is a full scCAD run (clustering, decomposition, anomaly scoring) rather
  than just constructing the feature-selected gene set
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCAD
  role: Implements the ensemble feature selection module that produces the gene set
    used for rare cell type identification
  repo: https://github.com/xuyp-csu/scCAD
- name: Seurat
  role: 'HVG selection strategy: binned mean-expression versus dispersion ranking
    of the top 2000 most variable genes'
- name: scikit-learn
  role: Random forest classifier whose Gini-impurity-based feature importances rank
    the top 2000 HIG
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:1db9f778d47144cb81c496aafafa5ce273e02d569c05fee125e90dca7f390965
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  repo_url: https://github.com/xuyp-csu/scCAD
  tools:
  - Seurat
  - scCAD
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# ensemble-feature-selection-single-cell-rnaseq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

scCAD's ensemble feature selection step builds the gene set for downstream rare-cell identification by merging the top 2000 highly variable genes (HVG, Seurat-style binned mean/dispersion ranking) with the top 2000 highly important genes (HIG) ranked by random-forest Gini importance, deduplicating the union.

## When to use

Use when a preprocessed, log-normalized cells x genes scRNA-seq matrix plus initial cluster labels are available and a compact gene subset (up to ~4000 genes) is needed before rare-cell anomaly detection or clustering.

## When NOT to use

- The input is already a reduced feature table or a preselected gene set — re-selecting genes would discard information
- The data are raw counts that have not yet been filtered (genes expressed in fewer than three cells removed) and log-normalized with cell-specific size factors
- The goal is a full scCAD run (clustering, decomposition, anomaly scoring) rather than just constructing the feature-selected gene set

## Inputs

- Preprocessed, log-normalized expression matrix (cells x genes), e.g. from an .h5 file like 1%Jurkat.h5
- Initial cluster labels per cell (e.g. Louvain labels)
- Gene name vector and cell name vector for the matrix

## Outputs

- Merged, deduplicated gene list of at most 4000 genes (2000 HVG + 2000 HIG), written e.g. to selected_genes.csv
- Two intermediate ranked gene lists: top 2000 HVG and top 2000 HIG with random-forest Gini importances

## How to apply

Start from the preprocessed, log-normalized expression matrix (cells x genes) and initial Louvain cluster labels (e.g. from 1%Jurkat.h5). First compute each gene's mean expression and dispersion (variance/mean), bin genes with similar mean as in Seurat's HVG approach, and retain the top 2000 most variable genes. Next train a random forest classifier (scikit-learn) on the expression matrix against the initial cluster labels and keep the top 2000 genes by Gini-impurity-based feature importance (HIG). Merge the HVG and HIG sets and remove duplicates — the union is at most 4000 unique genes — then write the final gene list (e.g. to selected_genes.csv) for use in scCAD's downstream cluster decomposition. The rationale is that HVGs capture dominant global variability while random-forest importance concentrates on cluster-discriminating, rare-cell-type-specific genes; scCAD reports that over 86% of its selected genes were rare cell type-specific on average.

## Related tools

- **scCAD** (Implements the ensemble feature selection module that produces the gene set used for rare cell type identification) — https://github.com/xuyp-csu/scCAD
- **Seurat** (HVG selection strategy: binned mean-expression versus dispersion ranking of the top 2000 most variable genes)
- **scikit-learn** (Random forest classifier whose Gini-impurity-based feature importances rank the top 2000 HIG)

## Examples

```
from sklearn.ensemble import RandomForestClassifier; rf = RandomForestClassifier(random_state=2023).fit(data, labels); hig = np.argsort(rf.feature_importances_)[-2000:]; selected = sorted(set(hvg) | set(hig))
```

## Evaluation signals

- The output gene table (e.g. selected_genes.csv) parses as a gene-name table with at most 4000 unique genes (2000 HVG + 2000 HIG after deduplication)
- Both intermediate sets contain exactly 2000 genes each: verify HVG count via Seurat-style binned mean/dispersion ranking and HIG count via sorted random-forest importances
- Every selected gene was retained by the upstream filter (expressed in at least three cells) and is present in the input gene-name vector
- Downstream check: scCAD reports over 86% of selected genes were rare cell type-specific on average, so the selected set should be enriched for cluster-discriminating genes rather than housekeeping genes

## Limitations

- Requires prior preprocessing: genes expressed in fewer than three cells removed, log-normalization with cell-specific size factors, and initial Louvain cluster labels; applying it to raw counts will produce poor HVG/HIG rankings
- Union is capped at 4000 genes; with heavy overlap between HVG and HIG sets the final feature set may be substantially smaller than 4000
- Random forest HIG ranking depends on the quality of the initial clustering — rare cell types missed during the initial clustering phase may be underrepresented in the selected genes
- Random seed affects the forest fit (scCAD default seed: 2023); OS supported is Linux/macOS only, requiring Python 3.7.13 with scikit-learn 1.0.2

## Evidence

- [discussion] Ensemble feature selection: top 2000 highly variable genes plus top 2000 random-forest-importance genes: [evidence span withheld — anchor status: not_found]
- [methods] Random forest importance ranking and merged gene set output from the task workflow: [evidence span withheld — anchor status: not_found]
