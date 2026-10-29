---
name: random-forest-gene-importance-ranking
description: Use when a log-normalized scRNA-seq expression matrix with initial cluster
  labels needs a supervised, label-aware gene ranking (HIG) to complement unsupervised
  HVG selection before rare cell type detection.
when_to_use_negative:
- Input is raw counts without initial cluster labels — the random forest requires
  supervised cluster assignments and cannot rank genes without them
- The analysis goal is purely unsupervised gene dispersion filtering — use Seurat-style
  mean vs dispersion HVG selection instead
- The dataset has few or degenerate clusters (e.g. no rare populations at all), making
  Gini importance dominated by abundant-type marker genes
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scCAD
  role: Pipeline that orchestrates HIG ranking as part of its ensemble feature selection
    module before cluster decomposition
  repo: https://github.com/xuyp-csu/scCAD
- name: scikit-learn
  role: Random forest classifier providing Gini-impurity-based gene importance (version
    1.0.2 pinned in scCAD requirements)
- name: Seurat
  role: Reference for the mean-vs-dispersion HVG selection whose top 2000 genes are
    merged with the HIG set
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:304ab885398d51fcbcfd25ebc816ef1b7be8fdbafe11b8bea96a27d20ea97e98
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# random-forest-gene-importance-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rank genes by supervised importance using a random forest classifier trained on initial Louvain cluster labels, retaining the top 2000 highly important genes (HIG). In scCAD these HIG are merged with the top 2000 mean-dispersion HVG to build the gene set used for downstream rare cell type identification.

## When to use

Use when a log-normalized scRNA-seq expression matrix with initial cluster labels needs a supervised, label-aware gene ranking (HIG) to complement unsupervised HVG selection before rare cell type detection.

## When NOT to use

- Input is raw counts without initial cluster labels — the random forest requires supervised cluster assignments and cannot rank genes without them
- The analysis goal is purely unsupervised gene dispersion filtering — use Seurat-style mean vs dispersion HVG selection instead
- The dataset has few or degenerate clusters (e.g. no rare populations at all), making Gini importance dominated by abundant-type marker genes

## Inputs

- Preprocessed, log-normalized scRNA-seq expression matrix (cells x genes, e.g. loaded from 1%Jurkat.h5 via h5py)
- Initial cluster labels per cell (Louvain labels from scCAD's PCA/kNN preprocessing)
- Gene name array (e.g. data_mat['gn'])

## Outputs

- Ranked list of top 2000 highly important genes (HIG) by random forest Gini importance
- Merged, de-duplicated ensemble gene set (<= 4000 unique genes from HVG+HIG) used for downstream cluster decomposition

## How to apply

Start from the preprocessed, log-normalized cells-by-genes matrix (genes expressed in at least three cells retained, size-factor normalization, log2(x+1)) and the initial Louvain cluster labels obtained on a k=15 KNN graph over the top 40 principal components. Train a random forest classifier (scikit-learn, version 1.0.2 per scCAD requirements) on the expression matrix using these cluster labels, and rank genes by Gini-impurity-based importance. Keep the top 2000 highly important genes (HIG); binning by mean expression is NOT applied here — that is only the Seurat-style mean vs dispersion binning used for the parallel top-2000 HVG set. Merge the HIG with the top 2000 HVG and de-duplicate to form the ensemble gene set scCAD feeds into iterative cluster decomposition and isolation-forest anomaly scoring. This ensemble is the reason scCAD retains ~86.75% of reference DE genes (OR1) versus 67.80% for HVG-only and 80.20% for HIG-only selection (Supplementary Table 8).

## Related tools

- **scCAD** (Pipeline that orchestrates HIG ranking as part of its ensemble feature selection module before cluster decomposition) — https://github.com/xuyp-csu/scCAD
- **scikit-learn** (Random forest classifier providing Gini-impurity-based gene importance (version 1.0.2 pinned in scCAD requirements))
- **Seurat** (Reference for the mean-vs-dispersion HVG selection whose top 2000 genes are merged with the HIG set)

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- The HIG list contains exactly 2000 genes ranked by RF importance scores
- The merged HVG+HIG ensemble set contains at most 4000 unique genes after deduplication (e.g. selected_genes.csv parses as a gene-name table)
- On benchmark scRNA-seq datasets with rare-type annotations, average OR1 = |S_ref ∩ S_select|/|S_ref| should approximate 80.20% for HIG-only and 86.75% for the scCAD ensemble versus 67.80% for HVG-only (Supplementary Table 8 figures)
- Reference DE genes per rare cell type are defined via Seurat FindMarkers (Wilcoxon rank sum, FDR < 0.05, |fold-change| > 1.5, top 50 per type) before computing overlap rates
- HIG ranking should capture cluster-discriminating genes that dispersion-based HVG binning misses; a sanity check is that the HIG set is not identical to the HVG set

## Limitations

- Supervised nature makes the ranking dependent on the quality of the initial Louvain clustering; rare types overlooked during the clustering phase will not drive importance
- Supplementary Table 8 (overlap-rate validation figures) and Supplementary Tables 21/22 are referenced but not included in the retrieved document, so the reported OR1/OR2/OR3 values cannot be independently verified from this context
- Exact versions of comparative packages are mostly unstated in the section body; only scCAD's own requirements.txt pins versions (scikit-learn 1.0.2, Scanpy 1.9.1)
- Gini importance from random forests is a general supervised signal, not a guarantee that retained genes are rare cell type-specific

## Evidence

- [discussion] Ensemble feature selection: top 2000 highly variable genes plus top 2000 random-forest-importance genes: [evidence span withheld — anchor status: not_found]
