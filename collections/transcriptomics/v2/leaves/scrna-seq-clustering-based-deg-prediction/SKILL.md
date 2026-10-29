---
name: scrna-seq-clustering-based-deg-prediction
description: Use when a scRNA-seq dataset (e.g., Tabula Muris bone marrow) needs cluster-based
  marker discovery, or when benchmarking a clustering-independent DEG method (singleCellHaystack)
  against the standard cluster-then-test Wilcoxon pipeline.
when_to_use_negative:
- When the goal is DEG prediction itself and the number of biologically relevant clusters
  is unclear — the article warns that 'the definition of cell clusters is often not
  straightforward' and singleCellHaystack was built for this case.
- When a p-value on the DEG list must be strictly calibrated — clustering-based (and
  haystack) p-values are inflated by double use of gene expression data for clustering
  and testing.
- 'When genes must be ranked by expression level, not detection: both approaches discussed
  here rely on detection/zero-count handling, and singleCellHaystack requires binary
  detection data as input.'
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Clusters the cells (FindClusters) and calls per-cluster markers with FindAllMarkers
    using the default Wilcoxon rank-sum test — the clustering-based half of the comparison.
  repo: https://github.com/satijalab/seurat
- name: SeuratData
  role: Distributes example datasets as Seurat objects used to load data for the marker-based
    workflow.
  repo: https://github.com/satijalab/seurat-data
- name: singleCellHaystack
  role: Clustering-independent DEG predictor (Kullback-Leibler divergence) whose results
    are ranked and compared against the FindAllMarkers Wilcoxon output.
  repo: https://github.com/alexisvdb/singleCellHaystack
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:cf5b6c33c398e35107be6fff51806c08a1c5da8dbacccab46b00a552ff5318f4
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# scrna-seq-clustering-based-deg-prediction

## Summary

Baseline clustering-based DEG prediction workflow for scRNA-seq: cluster cells (e.g., Seurat FindClusters) and call per-cluster markers with a Wilcoxon rank-sum test (FindAllMarkers). It serves as the comparison standard against clustering-independent methods like singleCellHaystack, whose top-100 bone marrow DEGs overlap it in only 3 genes.

## When to use

Use when a scRNA-seq dataset (e.g., Tabula Muris bone marrow) needs cluster-based marker discovery, or when benchmarking a clustering-independent DEG method (singleCellHaystack) against the standard cluster-then-test Wilcoxon pipeline.

## When NOT to use

- When the goal is DEG prediction itself and the number of biologically relevant clusters is unclear — the article warns that 'the definition of cell clusters is often not straightforward' and singleCellHaystack was built for this case.
- When a p-value on the DEG list must be strictly calibrated — clustering-based (and haystack) p-values are inflated by double use of gene expression data for clustering and testing.
- When genes must be ranked by expression level, not detection: both approaches discussed here rely on detection/zero-count handling, and singleCellHaystack requires binary detection data as input.

## Inputs

- scRNA-seq gene expression count matrix (e.g., Tabula Muris bone marrow dataset)
- Precomputed singleCellHaystack results (haystack object with per-gene DKL and p-values)
- Clustering definition (Seurat clusters from FindClusters)

## Outputs

- Per-cluster marker gene table (p_val, avg_logFC, pct.1, pct.2 from FindAllMarkers/Wilcoxon)
- Merged per-gene p-value table for scatterplot comparison of both methods
- Top-100 rank-overlap statistics (number of shared genes, expected to be 3)
- Per-gene rank records for Ramp1, Il2rb, Grm8, and Tnni1 in both methods

## How to apply

Load the scRNA-seq count matrix (e.g., Tabula Muris bone marrow) into R with Seurat/SeuratData, cluster cells with Seurat FindClusters, then call marker genes per cluster with FindAllMarkers using the default Wilcoxon rank-sum test. Compile the per-gene p-values from this clustering-based route and the clustering-independent route (singleCellHaystack's Kullback-Leibler divergence on t-SNE/PCA coordinates, e.g., res <- haystack(dat.tsne, dat.expression)) into a single table suitable for a p-value scatterplot. Quantify disagreement by rank-overlap statistics on the top 100 high-scoring genes of each method. Rationale: the article highlights that cluster definition is 'often not straightforward', so this skill is best deployed as a benchmark baseline whose disagreement with clustering-independent methods is itself the biological signal.

## Related tools

- **Seurat** (Clusters the cells (FindClusters) and calls per-cluster markers with FindAllMarkers using the default Wilcoxon rank-sum test — the clustering-based half of the comparison.) — https://github.com/satijalab/seurat
- **SeuratData** (Distributes example datasets as Seurat objects used to load data for the marker-based workflow.) — https://github.com/satijalab/seurat-data
- **singleCellHaystack** (Clustering-independent DEG predictor (Kullback-Leibler divergence) whose results are ranked and compared against the FindAllMarkers Wilcoxon output.) — https://github.com/alexisvdb/singleCellHaystack

## Examples

```
library(Seurat); markers <- FindAllMarkers(object = seurat_obj, test.use = "wilcox")
```

## Evaluation signals

- Top-100 overlap count between FindAllMarkers and singleCellHaystack high-scoring genes must equal exactly 3 shared genes (Supplementary Fig. 8).
- Ranks of Ramp1, Il2rb, Grm8, and Tnni1 in both methods must match the per-gene values reported in the manuscript.
- FindAllMarkers output must contain the standard per-cluster columns (p_val, avg_logFC, pct.1, pct.2) with one block per cluster.
- The merged p-value table must have one row per gene with non-missing p-values from both methods, suitable for a p-value scatterplot.
- Cells must be clustered first (FindClusters) before marker calling; marker calls are per-cluster, not per-dataset.

## Limitations

- Inflated p-values: 'DEG prediction methods often return inflated p-values because of the double use of gene expression data (for defining clusters and for DEG prediction).'
- The definition of cell clusters and their number is often not straightforward, making the marker set sensitive to arbitrary clustering choices.
- singleCellHaystack (the comparator) requires binary detection data and uses a hard detection threshold, so genes with different expression levels but identical detection patterns receive identical D_KL and p-values.

## Evidence

- [intro] A major problem with clustering-based approaches for DEG prediction is that the definition of cell clusters is often not straightforward.: "A major problem with clustering-based approaches for DEG prediction is that the definition of cell clusters is often not straightforward."
- [supplementary] The top 100 high-scoring genes of both approaches have only 3 genes in common (Supplementary Fig. 8).: "The top 100 high-scoring genes of both approaches have only 3 genes in common (Supplementary Fig. 8)."
- [discussion] DEG prediction methods often return inflated p-values because of the double use of gene expression data (for defining clusters and for DEG prediction).: [evidence span withheld — anchor status: not_found]
- [readme] singleCellHaystack uses Kullback-Leibler Divergence to find features that have patterns of activity in subsets of samples that are non-randomly positioned inside any input space.: "singleCellHaystack uses Kullback-Leibler
Divergence to find features that have patterns of activity in subsets of
samples that are non-randomly positioned inside any input space."
