---
name: single-cell-rnaseq-autoencoder-clustering
description: Use when clustering raw or normalized scRNA-seq count/expression matrices
  (rows = cells, columns = genes) with tens of thousands of genes or millions of cells,
  especially when the number of cell types is unknown and rare cell types may be present.
when_to_use_negative:
- Input is not a gene-by-cell expression matrix (e.g. already-reduced embeddings,
  feature tables, or metadata only) — scCAN expects expression values and performs
  its own min-max scaling and log2(data+1) transformation
- Supervised cell-type annotation is the goal and reference labels already exist —
  scCAN is unsupervised and estimates cluster number via eigengap, not against labels
- Datasets with ambiguous/dropped cell labels that should be removed first (e.g. 'zothers',
  'dropped', 'not applicable', 'Not available') without prior label cleanup
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scCAN
  role: 'Primary clustering method: non-negative-kernel autoencoder feature selection,
    stacked VAE dimensionality reduction, network fusion, and spectral clustering;
    installed via devtools::install_github(''bangtran365/scCAN'') or CRAN'
  repo: https://cran.r-project.org/package=scCAN
- name: SHARP
  role: Benchmarked baseline clustering method against which scCAN's accuracy and
    scalability were compared; also listed as a tool surfaced by the source card
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:af52363c920ee08766d49e658390a0ab60c4b25919f650dac11554024b766f9b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SHARP
  - scCAN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-rnaseq-autoencoder-clustering

## Summary

Clusters single-cell RNA-seq data using scCAN, which compresses the expression matrix through a non-negative-kernel autoencoder (min-max scaled to 0-1, top 5000 genes) and a stacked variational autoencoder (latent dimension d=15), then partitions cells via similarity-network fusion and eigengap-based spectral clustering. Suited to large, noisy scRNA-seq datasets where accurate cell-type separation and automatic estimation of the number of clusters are required.

## When to use

Use when clustering raw or normalized scRNA-seq count/expression matrices (rows = cells, columns = genes) with tens of thousands of genes or millions of cells, especially when the number of cell types is unknown and rare cell types may be present.

## When NOT to use

- Input is not a gene-by-cell expression matrix (e.g. already-reduced embeddings, feature tables, or metadata only) — scCAN expects expression values and performs its own min-max scaling and log2(data+1) transformation
- Supervised cell-type annotation is the goal and reference labels already exist — scCAN is unsupervised and estimates cluster number via eigengap, not against labels
- Datasets with ambiguous/dropped cell labels that should be removed first (e.g. 'zothers', 'dropped', 'not applicable', 'Not available') without prior label cleanup

## Inputs

- Normalized or raw scRNA-seq expression matrix (cells as rows, genes as columns), e.g. the Zilionis (GSE127465) human lung matrix with 34,558 cells from the Broad Single Cell Portal
- Optional: annotated cell-type labels for scoring cluster separation (e.g. F1 or adjusted Rand index against tRBC labels)

## Outputs

- Cluster assignments per cell (result$cluster from scCAN(data, r.seed = 1))
- Estimated number of cell types determined by the eigengap method
- Per-cluster separation scores, e.g. F1 score of stage-two sub-clusters against annotated rare-cell labels

## How to apply

Load the expression matrix with cells as rows and genes as columns (log2-transform with log2(data+1) if max(data)>100, per the README). scCAN then (1) min-max rescales the data to 0-1 and filters uninformative genes with a non-negative-kernel autoencoder keeping the top 5000 genes, (2) generates low-dimensional representations with a stacked variational autoencoder (latent dimension d=15), and (3) builds a cell-similarity network for each representation (k=30 neighbors), fuses them with the SNF approach, determines the number of clusters with the eigengap method, and runs spectral clustering on the fused network, with weighted meta-clustering (wMetaC) to consolidate results. For datasets with more than 5000 cells, use Module 3: randomly select a 5000-cell training set, cluster it as above, and classify the remaining cells with k-NN. For rare cell types, re-run scCAN on each stage-one cluster in a second stage (multi-stage clustering), as demonstrated on the Zilionis lung dataset (34,558 cells) where stage two split cluster 2 into 2_1 and 2_2, separating tRBC cells with F1 = 1.

## Related tools

- **scCAN** (Primary clustering method: non-negative-kernel autoencoder feature selection, stacked VAE dimensionality reduction, network fusion, and spectral clustering; installed via devtools::install_github('bangtran365/scCAN') or CRAN) — https://cran.r-project.org/package=scCAN
- **SHARP** (Benchmarked baseline clustering method against which scCAN's accuracy and scalability were compared; also listed as a tool surfaced by the source card)

## Examples

```
if(max(data)>100) data <- log2(data + 1); result <- scCAN(data, r.seed = 1); cluster <- result$cluster; ari <- round(scCAN::adjustedRandIndex(cluster,label), 2)
```

## Evaluation signals

- Adjusted Rand Index (ARI) computed with scCAN::adjustedRandIndex(cluster, label) against known cell-type labels, as in the README example
- F1 score of stage-two clusters against annotated rare-cell labels; the Zilionis two-stage run with samp.size=5000 achieved F1 = 1 for tRBC separation
- scCAN should correctly estimate the number of true cell types via the eigengap method on the fused network
- Confirm preprocessing invariants: data rescaled to the 0-1 range, log2(data+1) applied only when max(data)>100, top 5000 genes retained by the kernel autoencoder
- For n > 5000, verify the Module 3 train/test split: a 5000-cell randomly selected training set clustered by network fusion, with remaining cells assigned via k-NN

## Limitations

- No software version numbers are stated for scCAN or the compared tools (CIDR, SEURAT3, Monocle3, SHARP, SCANPY) in the article sections provided
- Cells with ambiguous labels ('zothers', 'dropped', 'not applicable', 'Not available') must be removed from certain datasets (Chen, Wang, Segerstolpe, Guo, Kanton, Miller) before evaluation
- scRNA-seq dropouts and high dimensionality (tens of thousands of genes, up to millions of cells) are the stated challenges; very large datasets require the 5000-cell training-set approximation rather than clustering all cells directly
- The article provides no DOI/accession IDs or explicit install URL in some sections; the CRAN package documentation must be consulted separately

## Evidence

- [intro] scCAN, a single-cell clustering approach that consists of three modules: (1) a non-negative kernel autoencoder to filter out uninformative features, (2) a stacked, variational: "Here we introduce scCAN, a single-cell clustering approach that consists of three modules: (1) a non-negative kernel autoencoder to filter out uninformative features, (2) a stacked, variational"
- [methods] Data rescaling to 0-1 range: [evidence span withheld — anchor status: not_found]
- [methods] 5000-cell training set and k-NN classification for big data: "we split the cells into two sets: a training set of 5000 randomly selected cells and a testing set that consists of the remaining cells"
- [methods] Eigengap method for cluster number: "we use the eigengap method48 to determine the number of clusters"
- [results] Zilionis dataset accession and size: "ZilionisGSE127465Human lung34,5589inDrop"
- [readme] README invocation and ARI evaluation: "Generate clustering result, the input matrix has rows as samples and columns as genes: `result <- scCAN(data, r.seed = 1)`"
