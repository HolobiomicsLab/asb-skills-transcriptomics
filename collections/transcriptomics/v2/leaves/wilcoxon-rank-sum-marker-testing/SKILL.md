---
name: wilcoxon-rank-sum-marker-testing
description: 'Use when you need a clustering-based DEG baseline for benchmarking:
  cells can be clustered into biologically meaningful groups and per-cluster marker
  genes (e.g. via FindAllMarkers) must be ranked by p-value to compare against an
  alternative method.'
when_to_use_negative:
- The goal is clustering-independent DEG prediction without defining cell clusters
  — use singleCellHaystack's `haystack()` directly instead of cluster-restricted marker
  testing
- Clusters are not biologically interpretable or the number of relevant clusters is
  unclear, which the paper identifies as a major problem for clustering-based DEG
  prediction
- The input is spatial transcriptomics data where the aim is to test genes against
  actual 2D tissue coordinates rather than per-cluster markers
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Provides FindClusters and FindAllMarkers (default Wilcoxon rank-sum test)
    to produce the clustering-based DEG baseline
  repo: https://github.com/satijalab/seurat
- name: singleCellHaystack
  role: Clustering-independent DEG prediction method whose ranked genes are compared
    against the FindAllMarkers results
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: SeuratData
  role: R package used to load Seurat-formatted example datasets alongside the comparison
    workflow
  repo: https://github.com/satijalab/seurat-data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:5143927979c620e67a3035f64c35d12a401f7f75bc44e56d3196b00241e51b8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat
  tools:
  - R
  - Seurat
  - SeuratData
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# wilcoxon-rank-sum-marker-testing

## Summary

Cluster-wise differential expression testing with Seurat's FindAllMarkers using the default Wilcoxon rank-sum test, used in this paper as the clustering-based baseline against which singleCellHaystack's clustering-independent DEG prediction is compared on the Tabula Muris bone marrow dataset.

## When to use

Use when you need a clustering-based DEG baseline for benchmarking: cells can be clustered into biologically meaningful groups and per-cluster marker genes (e.g. via FindAllMarkers) must be ranked by p-value to compare against an alternative method.

## When NOT to use

- The goal is clustering-independent DEG prediction without defining cell clusters — use singleCellHaystack's `haystack()` directly instead of cluster-restricted marker testing
- Clusters are not biologically interpretable or the number of relevant clusters is unclear, which the paper identifies as a major problem for clustering-based DEG prediction
- The input is spatial transcriptomics data where the aim is to test genes against actual 2D tissue coordinates rather than per-cluster markers

## Inputs

- scRNA-seq gene expression matrix (e.g. Tabula Muris bone marrow counts from figshare)
- Seurat object with normalized expression and cell-cluster identities from FindClusters
- Precomputed singleCellHaystack results (haystack object with per-gene DKL and p-values)
- Embedding (t-SNE/UMAP) or first 20 principal components used as the input space for haystack

## Outputs

- Per-cluster marker table from FindAllMarkers (gene, cluster, p_val, p_val_adj, avg_logFC) from the Wilcoxon rank-sum test
- Combined table of per-gene p-values from both methods for a p-value scatterplot
- Top-100 rank-overlap count between the two methods' high-scoring genes (expected: 3 shared genes)
- Per-gene rank comparison for Ramp1, Il2rb, Grm8, and Tnni1 across both methods

## How to apply

Load the scRNA-seq expression matrix (e.g. the Tabula Muris bone marrow dataset downloaded from figshare) into a Seurat object in R, cluster cells with Seurat FindClusters, and call marker genes per cluster with FindAllMarkers under its default Wilcoxon rank-sum test. Compile the per-gene p-values from FindAllMarkers and from singleCellHaystack (which runs `haystack` on embedding or PCA coordinates using Kullback-Leibler divergence on binary detection data) into a table suitable for a p-value scatterplot. Compute rank-overlap statistics for the top 100 highest-scoring genes of each method — in the paper only 3 genes are shared (Supplementary Fig. 8) — and extract per-gene ranks for the disagreeing genes Ramp1, Il2rb, Grm8, and Tnni1. The rationale is that FindAllMarkers inherits any bias from the cluster definitions, while singleCellHaystack does not rely on explicit clustering, so rank agreement quantifies how much cluster choice drives DEG calls.

## Related tools

- **Seurat** (Provides FindClusters and FindAllMarkers (default Wilcoxon rank-sum test) to produce the clustering-based DEG baseline) — https://github.com/satijalab/seurat
- **singleCellHaystack** (Clustering-independent DEG prediction method whose ranked genes are compared against the FindAllMarkers results) — https://github.com/alexisvdb/singleCellHaystack
- **SeuratData** (R package used to load Seurat-formatted example datasets alongside the comparison workflow) — https://github.com/satijalab/seurat-data

## Examples

```
markers <- FindAllMarkers(seu_bone_marrow, only.pos = TRUE, test.use = "wilcox")
```

## Evaluation signals

- The count of shared genes among the top 100 high-scoring genes of singleCellHaystack and FindAllMarkers equals 3 (Supplementary Fig. 8)
- The extracted ranks of Ramp1, Il2rb, Grm8, and Tnni1 in both methods match the values reported in the manuscript
- The FindAllMarkers call uses the default Wilcoxon rank-sum test (test.use = 'wilcox') rather than an alternative test such as MAST or DESeq2
- The compiled table contains per-gene p-values from both methods and renders as a valid p-value scatterplot without missing entries for the ranked genes
- Genes were pre-filtered consistently before testing (the paper filters genes detected in fewer than 10 cells) so both methods operate on the same gene universe

## Limitations

- Clustering-based DEG prediction inflates p-values because gene expression data is used twice — once for defining clusters and once for DEG prediction — a caveat the paper notes also applies to singleCellHaystack
- Results depend on the cluster definitions from FindClusters; the paper notes the number of biologically relevant clusters is often not obvious, so the Wilcoxon marker set changes with clustering choices
- singleCellHaystack, the comparison counterpart, requires binary detection data and a user-chosen detection threshold, which constrains how directly its p-values can be compared to Wilcoxon p-values on continuous counts

## Evidence

- [supplementary] The top 100 high-scoring genes of both approaches have only 3 genes in common (Supplementary Fig. 8).: "The top 100 high-scoring genes of both approaches have only 3 genes in common (Supplementary Fig. 8)."
- [methods] res <- haystack(dat.tsne, dat.expression): "res <- haystack(dat.tsne, dat.expression)"
- [readme] singleCellHaystack uses Kullback-Leibler Divergence to find features that have patterns of activity in subsets of samples that are non-randomly positioned inside any input space.: [evidence span withheld — anchor status: external_doc]
- [methods] We filter genes with less 10 cells with non-zero counts.: "We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes."
