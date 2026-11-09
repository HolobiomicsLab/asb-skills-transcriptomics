---
name: adjusted-rand-index-clustering-evaluation
description: Use when comparing unsupervised clusterings (e.g., Louvain on 50 PCs
  from different gene sets) against known reference labels (cortical layer annotations)
  to quantify recovery, such as benchmarking SVG-based vs. HVG-based feature selection
  on Visium DLPFC.
when_to_use_negative:
- There are no trusted reference labels (e.g., manual layer annotations) to compare
  against — ARI requires a ground-truth partition and cannot validate purely exploratory
  clusterings
- The goal is SVG detection itself rather than downstream clustering evaluation —
  use nnSVG ranking on LR_stat instead of ARI comparison
- Input is a single gene set with no competing feature-selection strategies to compare
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0625
tools:
- name: nnSVG
  role: Supplies the top-ranked SVG feature set whose downstream Louvain clustering
    is scored by ARI; results (rank, LR_stat, padj) stored in rowData of the SpatialExperiment
  repo: https://bioconductor.org/packages/nnSVG
- name: SPARK-X
  role: Competing nonparametric covariance-test feature-selection method whose top-1000
    gene clustering is benchmarked by ARI
- name: mclust
  role: Provides adjustedRandIndex, the metric used to score each clustering against
    manual cortical layer labels
- name: Bioconductor
  role: Framework for the SpatialExperiment object and nnSVG installation (R >= 4.2
    via BiocManager)
  repo: https://bioconductor.org/packages/nnSVG
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:eca9cb20486b7172a094279a536cf0971c5b20f3132fc2b72995467a2e7d7609
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - SPARK-X
  - mclust
  - nnSVG
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# adjusted-rand-index-clustering-evaluation

## Summary

Evaluates graph-based Louvain clustering of spatially-resolved transcriptomics data against manual layer annotations by computing the adjusted Rand index (ARI) with mclust::adjustedRandIndex, used here to compare feature-selection strategies (top 1000 SVGs from nnSVG, SPARK-X, Moran's I vs. top HVGs) on the Visium human DLPFC dataset.

## When to use

Use when comparing unsupervised clusterings (e.g., Louvain on 50 PCs from different gene sets) against known reference labels (cortical layer annotations) to quantify recovery, such as benchmarking SVG-based vs. HVG-based feature selection on Visium DLPFC.

## When NOT to use

- There are no trusted reference labels (e.g., manual layer annotations) to compare against — ARI requires a ground-truth partition and cannot validate purely exploratory clusterings
- The goal is SVG detection itself rather than downstream clustering evaluation — use nnSVG ranking on LR_stat instead of ARI comparison
- Input is a single gene set with no competing feature-selection strategies to compare

## Inputs

- SpatialExperiment object for Visium human DLPFC (from STexampleData / nnSVG package) with raw counts, spatial coordinates, and manually annotated layer labels
- Per-strategy gene rankings (top 1000 SVGs from nnSVG, SPARK-X, Moran's I, and top HVGs)
- Top-50-PC embeddings per strategy, Louvain cluster assignments, and manual layer annotations

## Outputs

- Adjusted Rand index per feature-selection strategy (nnSVG, SPARK-X, Moran's I, HVGs) against cortical layer annotations
- Comparison table written to ari_clustering_comparison.csv

## How to apply

Preprocess the Visium human DLPFC SpatialExperiment object by keeping in-tissue spots, running filter_genes, computing library-size factors with scran, and log-transforming (logNormCounts) before PCA on all spots. For each feature-selection strategy (nnSVG, SPARK-X, Moran's I, HVGs), take the top 1000 ranked genes, subset the top-50-PC embedding, build a shared nearest-neighbor graph, and run graph-based Louvain clustering. Compute the adjusted Rand index between each clustering and the manual cortical layer annotations with mclust::adjustedRandIndex. Assemble the four ARI values into a comparison table (ari_clustering_comparison.csv); the validation criterion is that the ARI ordering must match the reported result — nnSVG and Moran's I outperform HVGs and SPARK-X.

## Related tools

- **nnSVG** (Supplies the top-ranked SVG feature set whose downstream Louvain clustering is scored by ARI; results (rank, LR_stat, padj) stored in rowData of the SpatialExperiment) — https://bioconductor.org/packages/nnSVG
- **SPARK-X** (Competing nonparametric covariance-test feature-selection method whose top-1000 gene clustering is benchmarked by ARI)
- **mclust** (Provides adjustedRandIndex, the metric used to score each clustering against manual cortical layer labels)
- **Bioconductor** (Framework for the SpatialExperiment object and nnSVG installation (R >= 4.2 via BiocManager)) — https://bioconductor.org/packages/nnSVG

## Examples

```
spe <- Visium_humanDLPFC(); spe <- filter_genes(spe); spe <- logNormCounts(spe); spe <- nnSVG(spe); ari <- mclust::adjustedRandIndex(colData(spe)$label, louvain_clusters)
```

## Evaluation signals

- Reproduced ARI ordering matches the reported result: nnSVG and Moran's I outperform HVGs and SPARK-X in adjusted Rand index against cortical layer labels
- Each of the four strategies yields exactly one ARI value in the comparison table written to ari_clustering_comparison.csv
- Same preprocessing, top-50-PC embedding, and shared nearest-neighbor graph parameters are used across all four feature sets so ARI differences are attributable to feature selection only
- Preprocessing invariants match the README workflow: in-tissue spots retained (3639 of 4992), mitochondrial genes removed, ~30216 genes removed by filter_genes, logcounts assay present
- ARI values fall in a sensible range for layer recovery on Visium DLPFC (roughly between 0 and 1, with ARI > 0 for all SVG-based strategies)

## Limitations

- ARI depends on the number of clusters returned by Louvain; without fixing graph-based clustering parameters, comparisons across strategies may conflate feature quality with cluster-count sensitivity
- nnSVG on transcriptome-wide data with >=10,000 spatial locations has nontrivial runtimes (several hours on 10 cores on an HPC cluster), limiting how many feature sets can be regenerated for the comparison
- Small negative LR statistics observed for lower-ranked genes are difficult to interpret and could affect the tail of the top-1000 SVG ranking
- Rank averaging across multiple samples is a practical rather than principled approach, so multi-sample ARI comparisons inherit this limitation

## Evidence

- [discussion] Use nnSVG to generate lists of top SVGs during exploratory unsupervised analyses: "nnSVG can be used to generate lists of top SVGs during exploratory unsupervised analyses of SRT datasets"
- [discussion] Replace top HVGs with top SVGs for clustering: "replacing the set of top HVGs with the set of top SVGs from nnSVG, and then perform unsupervised clustering on the set of top SVGs"
- [discussion] Select arbitrary number of top-ranked SVGs: "either select an arbitrary set of top-ranked SVGs (e.g., top 100 genes) or select a set of statistically significant SVGs with adjusted p-values from the LR test"
- [readme] Input data format and results storage in rowData: [evidence span withheld — anchor status: not_found]
- [readme] README example dataset loading and preprocessing: "spe <- Visium_humanDLPFC()

dim(spe)"
- [readme] README gene filtering thresholds: [evidence span withheld — anchor status: external_doc]
