---
name: deg-method-rank-overlap-analysis
description: Use when two DEG-prediction methods (one clustering-independent like
  singleCellHaystack, one clustering-based like Seurat FindAllMarkers) must be benchmarked
  on the same scRNA-seq dataset and their top gene lists compared for agreement.
when_to_use_negative:
- The goal is method benchmarking on artificial/simulated data with known ground-truth
  DEGs, where accuracy metrics (not rank agreement) are the appropriate evaluation
- You need DEG p-values that are not inflated by double use of gene expression data
  — both approaches here suffer p-value inflation when coordinates/clusters derive
  from the same expression matrix
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: Clustering-independent DEG prediction using Kullback-Leibler divergence; supplies
    one method's ranked gene list and p-values
  repo: github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: Clusters cells (FindClusters) and calls per-cluster markers (FindAllMarkers,
    default Wilcoxon rank-sum test) supplying the second method's ranked gene list
  repo: github.com/satijalab/seurat
- name: R
  role: Implementation environment for both packages and the comparison/table compilation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:5dd012232abcb7c34bb73c990c2b6dc41af43784062bdf0dfff56042bdfe0a84
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# DEG method rank-overlap analysis

## Summary

Head-to-head comparison of clustering-independent DEG prediction (singleCellHaystack, Kullback-Leibler divergence) against clustering-based DEG calling (Seurat FindAllMarkers with the Wilcoxon rank-sum test) by compiling per-gene p-values and quantifying rank overlap of top-scoring genes, as done on the Tabula Muris bone marrow dataset.

## When to use

Use when two DEG-prediction methods (one clustering-independent like singleCellHaystack, one clustering-based like Seurat FindAllMarkers) must be benchmarked on the same scRNA-seq dataset and their top gene lists compared for agreement.

## When NOT to use

- The goal is method benchmarking on artificial/simulated data with known ground-truth DEGs, where accuracy metrics (not rank agreement) are the appropriate evaluation
- You need DEG p-values that are not inflated by double use of gene expression data — both approaches here suffer p-value inflation when coordinates/clusters derive from the same expression matrix

## Inputs

- scRNA-seq gene expression matrix for the Tabula Muris bone marrow dataset
- Precomputed singleCellHaystack results (haystack object) with per-gene D_KL scores and p-values
- Seurat object with cell clusters (FindClusters output) for FindAllMarkers

## Outputs

- Merged per-gene table of p-values from both methods (p-value scatterplot input)
- Top-100 rank-overlap count between the two methods' gene lists
- Per-gene rank table for disagreement genes (Ramp1, Il2rb, Grm8, Tnni1) in each method

## How to apply

Load the bone marrow scRNA-seq expression data together with the precomputed singleCellHaystack results (haystack object from `haystack` run on a 2D embedding such as t-SNE). In parallel, cluster cells with Seurat FindClusters and call marker genes per cluster using FindAllMarkers with its default Wilcoxon rank-sum test. Genes detected in fewer than 10 cells with non-zero counts should be filtered beforehand to drop very lowly expressed genes. Compile the per-gene p-values from both methods into a single table suitable for a p-value scatterplot, then compute rank-overlap statistics for the top 100 high-scoring genes of each method. Finally, extract and record the ranks of individual disagreement genes (Ramp1, Il2rb, Grm8, Tnni1) in both methods to characterize where the methods diverge.

## Related tools

- **singleCellHaystack** (Clustering-independent DEG prediction using Kullback-Leibler divergence; supplies one method's ranked gene list and p-values) — github.com/alexisvdb/singleCellHaystack
- **Seurat** (Clusters cells (FindClusters) and calls per-cluster markers (FindAllMarkers, default Wilcoxon rank-sum test) supplying the second method's ranked gene list) — github.com/satijalab/seurat
- **R** (Implementation environment for both packages and the comparison/table compilation)

## Examples

```
res <- haystack(dat.tsne, dat.expression)
```

## Evaluation signals

- The top-100 overlap count must equal exactly 3 shared genes between singleCellHaystack and FindAllMarkers (Supplementary Fig. 8)
- The extracted ranks of Ramp1, Il2rb, Grm8, and Tnni1 in both methods must match the values reported in the manuscript
- FindAllMarkers must be run with the default Wilcoxon rank-sum test (no test override)
- Genes with non-zero counts in fewer than 10 cells must be excluded before both methods' gene lists are compiled

## Limitations

- Both methods return inflated p-values because of double use of gene expression data (defining clusters/coordinates and DEG prediction from the same matrix)
- singleCellHaystack requires binary detection data as input and uses a hard detection threshold; genes with different expression levels but the same detection pattern receive identical D_KL and p-value
- Agreement between existing DEG methods is low in general, so low top-100 overlap is an expected outcome, not a pipeline failure

## Evidence

- [supplementary] The top 100 high-scoring genes of both approaches have only 3 genes in common (Supplementary Fig. 8).: "The top 100 high-scoring genes of both approaches have only 3 genes in common (Supplementary Fig. 8)."
- [methods] res <- haystack(dat.tsne, dat.expression): "res <- haystack(dat.tsne, dat.expression)"
- [methods] We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes.: "We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes."
- [readme] singleCellHaystack uses Kullback-Leibler Divergence to find features that have patterns of activity in subsets of samples that are non-randomly positioned inside any input space.: "singleCellHaystack uses Kullback-Leibler
Divergence to find features that have patterns of activity in subsets of
samples that are non-randomly positioned inside any input space."
- [discussion] DEG prediction methods often return inflated p-values because of the double use of gene expression data (for defining clusters and for DEG prediction).: [evidence span withheld — anchor status: not_found]
