---
name: pca-latent-space-embedding
description: Use when a library-normalized (scaled, non-log-transformed) genes x cells
  expression matrix must be embedded into a low-dimensional latent space before building
  a KNN graph or PhyloVision report, especially when cell counts exceed available
  memory (VISION 'pool' defaults to 'auto', micropooling.
when_to_use_negative:
- A user-provided latent space already exists and can be passed directly to VISION
  instead of recomputing PCA.
- Cell similarity should derive from a lineage tree or a cell-cell distance matrix
  (e.g. Hotspot's 'tree' or distances input) rather than transcriptomic PCA coordinates.
- The dataset is below the micropooling threshold and small enough for direct PCA
  over the full cell set, making pooling unnecessary overhead.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: VISION
  role: Hosts the filterGenesFano informative-gene filter, the PCA latent-space computation,
    the 'pool' (micropooling) mechanism, and addProjection() for embedding the UMAP
    view in the report
  repo: https://github.com/YosefLab/VISION
- name: PhyloVision
  role: 'Downstream consumer of the latent-space embedding: it projects single-cell
    transcriptomic profiles to 2D with UMAP and builds the VISION-based interactive
    report'
  repo: https://github.com/YosefLab/VISION
- name: R
  role: Implementation language in which the VISION/PhyloVision embedding pipeline
    runs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:69c25f5e05fffe39f463c186790c69448355f213dd2cbb0b33f344c10812730e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - PhyloVision
  - R
  - VISION
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# pca-latent-space-embedding

## Summary

Compresses a library-normalized scRNA-seq expression matrix into a low-dimensional PCA latent space over informative genes (e.g. selected with VISION's filterGenesFano), providing the cell-similarity geometry that downstream KNN graphs, UMAP projections, and VISION/PhyloVision analyses operate on. For very large matrices (e.g. >15,000 cells), it couples with VISION's micropooling so latent-space computation fits within memory and CPU limits.

## When to use

Use when a library-normalized (scaled, non-log-transformed) genes x cells expression matrix must be embedded into a low-dimensional latent space before building a KNN graph or PhyloVision report, especially when cell counts exceed available memory (VISION 'pool' defaults to 'auto', micropooling above ~15,000 cells).

## When NOT to use

- A user-provided latent space already exists and can be passed directly to VISION instead of recomputing PCA.
- Cell similarity should derive from a lineage tree or a cell-cell distance matrix (e.g. Hotspot's 'tree' or distances input) rather than transcriptomic PCA coordinates.
- The dataset is below the micropooling threshold and small enough for direct PCA over the full cell set, making pooling unnecessary overhead.

## Inputs

- Library-normalized (scaled, non-log-transformed) expression matrix, genes x cells, e.g. expression_matrix.txt.gz
- Informative-gene selection (filterGenesFano output) used as the PCA feature set
- Optional user-provided latent space (cells x components matrix)
- cellsPerPartition integer target (micropooling) when N > 15,000 cells

## Outputs

- Low-dimensional PCA latent-space embedding (cells x components)
- 2D UMAP projection of cells (attachable via addProjection())
- Micropool assignments and pooled expression profiles written to micropools.csv
- K-nearest-neighbor graph over the latent space with Gaussian kernel edge weights

## How to apply

Start from the library-normalized (scaled, non-log-transformed) expression matrix in genes x cells orientation, such as the vignette expression_matrix.txt.gz dataset. Select informative genes with VISION's filterGenesFano function (default parameters) and run PCA on those genes to obtain the latent space, or accept a user-supplied latent space if one already exists. For large datasets, invoke the VISION 'pool' mechanism (default 'auto'), which triggers micropooling above ~15,000 cells: Louvain clusters are iteratively subdivided with K-means until each pool holds approximately cellsPerPartition cells, and cells within each pool are collapsed by gene-wise mean to pooled profiles. The resulting cells x components latent matrix is then used to build a K-nearest-neighbor graph with Gaussian-kernel edge weights and to project cells onto two dimensions with UMAP, which can be attached to the report via VISION's addProjection() method. The rationale is that coordinated gene variation is easier to detect in a denoised embedding than in raw counts, and pooling bounds the computational cost of PCA and KNN construction on datasets like the Tusi et al. 2018 haematopoiesis matrix.

## Related tools

- **VISION** (Hosts the filterGenesFano informative-gene filter, the PCA latent-space computation, the 'pool' (micropooling) mechanism, and addProjection() for embedding the UMAP view in the report) — https://github.com/YosefLab/VISION
- **PhyloVision** (Downstream consumer of the latent-space embedding: it projects single-cell transcriptomic profiles to 2D with UMAP and builds the VISION-based interactive report) — https://github.com/YosefLab/VISION
- **R** (Implementation language in which the VISION/PhyloVision embedding pipeline runs)

## Evaluation signals

- The number of micropools equals floor(N / cellsPerPartition) and each pool contains approximately the target number of cells.
- Micropool assignments and pooled profiles are successfully written to micropools.csv.
- The expression matrix is confirmed to be library-normalized, scaled, and non-log-transformed before PCA.
- The KNN graph built over the latent space supports low-latency selections and subtree collapsing in the report (tested on trees up to ~4,000 leaves).
- The 2D UMAP projection is attachable to the output report via addProjection().

## Limitations

- Micropooling collapses cells within pools by gene-wise mean, trading single-cell resolution for scalability on datasets exceeding memory/CPU resources.
- PhyloVision's R implementation may create interoperability challenges with Python-only latent-space methods such as scvi-tools or ZIFA.
- Trees/datasets far larger than the tested ~4,000-leaf scale may push the interactive viewer beyond its validated performance envelope.

## Evidence

- [methods] Informative genes were found using the filterGenesFano function in VISION with default parameters: "Informative genes were found using the filterGenesFano function in VISION with default parameters"
- [results] Projection of cells to 2D: "we first projected the single-cell transcriptomic profiles onto two dimensions using Uni-form Mani-fold Approximation and Projection (UMAP"
- [methods] Projection method in report: "the `addProjection()` method can be used to add this view of the data to the output report"
- [readme] VISION purpose: "VISION aids in the interpretation of single-cell RNA-seq (scRNA-seq) data by selecting for gene signatures which describe coordinated variation between cells."
- [readme] Latent-space representation in Hotspot: [evidence span withheld — anchor status: not_found]
