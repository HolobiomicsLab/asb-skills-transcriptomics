---
name: t-cell-development-trajectory-analysis
description: Use when analyzing single-cell RNA-seq from differentiating T cells (e.g.
  DN to CD4+/CD8+ SP thymocytes) and you need clustering-independent DEG prediction
  along a pseudotime trajectory rather than between predefined clusters.
when_to_use_negative:
- The analysis goal is differential expression between multiple conditions (e.g. wild-type
  vs knockout) — the paper states such comparisons cannot be easily conducted with
  singleCellHaystack
- Results must feed directly into a cluster-oriented scRNA-seq analysis — the paper
  notes singleCellHaystack results are not straightforward to incorporate into cluster-oriented
  analyses
- Spatial transcriptomics where raw runtime is the binding constraint — SPARK-X is
  the faster alternative; singleCellHaystack trades runtime for better sensitivity
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: Predicts differentially active genes along the 1D pseudotime trajectory using
    Kullback-Leibler Divergence; top_features extracts the top 1000 DEGs and cluster_genes
    groups them (v1.0.0 and higher)
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: Normalization, highly-variable gene selection, scaling, PCA, and subsetting
    of DN/SP thymocyte clusters before trajectory inference
  repo: https://satijalab.org/seurat/
- name: monocle3
  role: Seurat-to-monocle3 conversion, learn_graph, and order_cells to infer the DN-to-SP
    pseudotime trajectory; accessible through the SeuratWrappers monocle3 vignette
  repo: https://cole-trapnell-lab.github.io/monocle3
- name: SeuratWrappers
  role: Provides the Seurat-to-monocle3 conversion used to move the subsetted thymus
    object into trajectory analysis
  repo: https://github.com/satijalab/seurat-wrappers
- name: R
  role: Implementation language for the Seurat + monocle3 + singleCellHaystack pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:8bc3633a6f184b4d3be3c51d6007f126d35454d31652f06a75ebb590c20dfbf4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Seurat
  - SeuratWrappers
  - monocle3
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# t-cell-development-trajectory-analysis

## Summary

A workflow for predicting genes with biased expression along thymocyte development by inferring a DN-to-SP pseudotime trajectory with monocle3 from Tabula Muris thymus scRNA-seq, then running singleCellHaystack v1.0.0 on the 1D pseudotime coordinates and clustering the top DEGs into 6 expression modules.

## When to use

Use when analyzing single-cell RNA-seq from differentiating T cells (e.g. DN to CD4+/CD8+ SP thymocytes) and you need clustering-independent DEG prediction along a pseudotime trajectory rather than between predefined clusters.

## When NOT to use

- The analysis goal is differential expression between multiple conditions (e.g. wild-type vs knockout) — the paper states such comparisons cannot be easily conducted with singleCellHaystack
- Results must feed directly into a cluster-oriented scRNA-seq analysis — the paper notes singleCellHaystack results are not straightforward to incorporate into cluster-oriented analyses
- Spatial transcriptomics where raw runtime is the binding constraint — SPARK-X is the faster alternative; singleCellHaystack trades runtime for better sensitivity

## Inputs

- Tabula Muris thymus scRNA-seq count matrix (Seurat-compatible)
- Cell-type cluster annotations identifying DN and CD4+/CD8+ SP thymocytes
- 1D pseudotime coordinates from monocle3 order_cells
- Processed (normalized, scaled) expression matrix or PCA-reduced Seurat/monocle3 object

## Outputs

- singleCellHaystack result object with per-gene DEG statistics along pseudotime
- Table of top 1000 predicted DEGs (via top_features)
- thymus_deg_modules.csv: gene-to-module assignment table for 6 k-means modules
- Plots of each module's mean expression along pseudotime

## How to apply

Load the Tabula Muris thymus count matrix into Seurat, normalize, identify highly variable genes, scale, and run PCA. Subset the double-negative (DN) and single-positive (CD4+ or CD8+ SP) T cell clusters, convert the Seurat object to monocle3 (e.g. via satijalab/seurat-wrappers), and run monocle3 learn_graph followed by order_cells to infer pseudotime from the DN to the SP trajectory. Run singleCellHaystack v1.0.0 on the processed expression matrix using the 1D pseudotime coordinates as the input space — the method uses Kullback-Leibler Divergence to find features with non-random activity patterns, so it handles 1D trajectories without clustering cells into arbitrary groups. Extract the top 1000 predicted DEGs with top_features, then cluster them with k-means (k=6, fixed seed) on their scaled expression profiles via cluster_genes or direct kmeans. Save the gene-to-module table as thymus_deg_modules.csv and plot each module's mean expression along pseudotime to confirm the patterns qualitatively match the published module structure (Fig 3B-C).

## Related tools

- **singleCellHaystack** (Predicts differentially active genes along the 1D pseudotime trajectory using Kullback-Leibler Divergence; top_features extracts the top 1000 DEGs and cluster_genes groups them (v1.0.0 and higher)) — https://github.com/alexisvdb/singleCellHaystack
- **Seurat** (Normalization, highly-variable gene selection, scaling, PCA, and subsetting of DN/SP thymocyte clusters before trajectory inference) — https://satijalab.org/seurat/
- **monocle3** (Seurat-to-monocle3 conversion, learn_graph, and order_cells to infer the DN-to-SP pseudotime trajectory; accessible through the SeuratWrappers monocle3 vignette) — https://cole-trapnell-lab.github.io/monocle3
- **SeuratWrappers** (Provides the Seurat-to-monocle3 conversion used to move the subsetted thymus object into trajectory analysis) — https://github.com/satijalab/seurat-wrappers
- **R** (Implementation language for the Seurat + monocle3 + singleCellHaystack pipeline)

## Examples

```
res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- The haystack run on 1D pseudotime coordinates returns significant genes with biased (non-random) activity along the DN-to-SP trajectory, not cluster-vs-cluster contrasts
- top_features(n=1000) returns exactly 1000 genes ranked by the haystack score
- k-means with k=6 and a fixed seed produces a stable gene-to-module assignment that is reproducible across runs
- Plotted mean expression per module shows distinct, monotonic or stage-biased patterns along pseudotime that qualitatively match the 6-module structure in Fig 3B-C
- The saved thymus_deg_modules.csv contains one gene-to-module row per DEG with 6 distinct module labels

## Limitations

- Comparisons between multiple conditions (e.g. wild-type and knockout) cannot be easily conducted in singleCellHaystack
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- For spatial transcriptomics, singleCellHaystack has relatively longer runtimes than SPARK-X, though it offers better sensitivity
- The Python implementation (singleCellHaystack-py) is in beta: weights.advanced.Q, the seeding grid method, and hierarchical clustering for cluster_genes are not yet available; this workflow should use the R package (v1.0.0+, CRAN)

## Evidence

- [other] singleCellHaystack v1.0.0 on the processed expression matrix using the 1D pseudotime coordinates as input: [evidence span withheld — anchor status: not_found]
- [abstract] it can handle 1D trajectories, 2-3D spatial coordinates: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
- [other] Subset the double-negative (DN) and single-positive (CD4+ or CD8+ SP) T cell clusters: [evidence span withheld — anchor status: not_found]
- [readme] singleCellHaystack uses Kullback-Leibler Divergence to find features: [evidence span withheld — anchor status: external_doc]
- [methods] cluster_genes hierarchical clustering method: "Hierarchical clustering method for `cluster_genes`."
