---
name: single-cell-clustering-annotation-workflow
description: 'Use when a raw single-cell RNA-seq count matrix must be quality-filtered,
  normalized, reduced to a PCA embedding, clustered by a neighborhood graph, tested
  for cluster-specific markers, and annotated against an external tissue marker reference.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 6
  member_skills:
  - single-cell-rna-seq-quality-filtering
  - scanpy-single-cell-data-preprocessing
  - scanpy-scrna-seq-data-handling
  - scanpy-single-cell-data-loading
  - scrna-cpm-normalization-log1p-scaling
  - scanpy-expression-normalization
  - single-cell-count-matrix-preprocessing--c487867256c234ac
  - scanpy-single-cell-preprocessing-pipeline
  - single-cell-expression-preprocessing--09b3e71f0eac3812
  - pca-dimensionality-reduction-scrna--260081bc53a44a5a
  - pca-dimensionality-reduction-of-sc-rna-seq-counts
  - pca-latent-space-embedding
  - knn-graph-construction-from-pca-embeddings
  - single-cell-rnaseq-pca-dimensionality-reduction
  - leiden-clustering-of-low-dimensional-embeddings
  - leiden-neighborhood-graph-clustering
  - leiden-graph-community-detection
  - louvain-graph-clustering--ae6203732f5c17a9
  - leiden-graph-community-clustering-on-topic-embeddings
  - marker-gene-identification-across-clusters
  - sccatch-object-construction
  - scrnaseq-marker-gene-identification
  - tissue-specific-marker-database-matching
  - cell-type-annotation-scoring-against-reference-database
  - evidence-based-cell-type-scoring
  - scrnaseq-cluster-benchmarking
  - cellmatch-reference-database-querying
  member_tools:
  - AnnData
  - AnnData (Annotated Data object)
  - Cellar
  - Python
  - faiss
  - numpy
  - scanpy
  - scikit-learn
  - scipy
  - scSemiProfiler
  - Cobolt
  - MOFA+
  - Mowgli
  - Multigrate
  - Seurat v4
  - Seurat
  - scCATCH
  - CellMatch
  - rev_gene
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/transcriptomics/v2
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Single-Cell RNA-seq Quality Control, Clustering, and Cell-Type Annotation

## Summary

Filter raw counts, construct a scaled log-CPM matrix and PCA embedding, cluster cells with Leiden, identify cluster-specific markers, and assign cell types.


## When to use

Use when a raw single-cell RNA-seq count matrix must be quality-filtered, normalized, reduced to a PCA embedding, clustered by a neighborhood graph, tested for cluster-specific markers, and annotated against an external tissue marker reference.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — filter_cells_and_genes

**Goal:** remove low-quality cells and uninformative genes from raw single-cell counts

**EDAM operation:** http://edamontology.org/operation_3218

**Inputs:** single-cell-object/raw-rna-counts, tsv/cell-metadata · **Outputs:** single-cell-object/filtered-raw-rna-counts

**Candidate leaf skills:** `single-cell-rna-seq-quality-filtering` (primary), `scanpy-single-cell-data-preprocessing`, `scanpy-scrna-seq-data-handling`, `scanpy-single-cell-data-loading`

**Tools (primary):** AnnData, AnnData (Annotated Data object), Cellar, Python, faiss, numpy, scanpy, scikit-learn, scipy

**Other candidate tools:** FastMNN, Harmony, MIRA, Scanorama, Seurat, scANVI, scVI, AE-TPGG, Python 3, gssnng

**Grounding:** 4 KB(s); DOIs: 10.1007/s11704-022-2011-y, 10.1038/s41467-022-29744-0, 10.1038/s41467-023-39494-2, 10.1093/bioadv/vbad150

### Stage 2 — normalize_expression

**Goal:** construct a scaled log-CPM expression matrix for dimensionality reduction

**EDAM operation:** http://edamontology.org/operation_3435

**Inputs:** single-cell-object/filtered-raw-rna-counts · **Outputs:** single-cell-matrix/scaled-log-cpm

**Candidate leaf skills:** `scrna-cpm-normalization-log1p-scaling` (primary), `scanpy-expression-normalization`, `single-cell-count-matrix-preprocessing--c487867256c234ac`, `scanpy-single-cell-preprocessing-pipeline`, `single-cell-expression-preprocessing--09b3e71f0eac3812`

**Tools (primary):** AnnData, AnnData (Annotated Data object), Cellar, faiss, scanpy, scikit-learn

**Other candidate tools:** Cancer-Finder, PyTorch, numpy, pandas, scMGCA, scipy, Python, UMAP, numba, pegasus, scikit-learn (sklearn), h5py, python-louvain, scCAD

**Grounding:** 5 KB(s); DOIs: 10.1038/s41467-022-29744-0, 10.1038/s41467-022-31107-8, 10.1038/s41467-023-36134-7, 10.1038/s41467-024-46413-6 …

### Stage 3 — compute_pca_embedding

**Goal:** reduce normalized single-cell expression to a compact PCA representation

**EDAM operation:** http://edamontology.org/operation_2939

**Inputs:** single-cell-matrix/scaled-log-cpm · **Outputs:** cell-embedding/pca

**Candidate leaf skills:** `pca-dimensionality-reduction-scrna--260081bc53a44a5a` (primary), `pca-dimensionality-reduction-of-sc-rna-seq-counts`, `pca-latent-space-embedding`, `knn-graph-construction-from-pca-embeddings`, `single-cell-rnaseq-pca-dimensionality-reduction`

**Tools (primary):** AnnData, AnnData (Annotated Data object), Cellar, SCANPY, Scikit-learn, faiss, scSemiProfiler

**Other candidate tools:** irlba, lme4, lmerTest, scDist, scTransform, simData, PhyloVision, R, VISION, PyTorch, Python, conda, pip, pytorch-cluster, scSHARP, SCALA, Seurat

**Grounding:** 5 KB(s); DOIs: 10.1016/j.crmeth.2022.100200, 10.1016/j.csbj.2023.10.032, 10.1038/s41467-022-29744-0, 10.1038/s41467-024-51649-3 …

### Stage 4 — cluster_cells

**Goal:** build a neighborhood graph on the PCA embedding and assign Leiden clusters

**EDAM operation:** http://edamontology.org/operation_3935

**Inputs:** cell-embedding/pca · **Outputs:** cluster-labels/leiden

**Candidate leaf skills:** `leiden-clustering-of-low-dimensional-embeddings` (primary), `leiden-neighborhood-graph-clustering`, `leiden-graph-community-detection`, `louvain-graph-clustering--ae6203732f5c17a9`, `leiden-graph-community-clustering-on-topic-embeddings`

**Tools (primary):** Cobolt, MOFA+, Mowgli, Multigrate, Python, Scanpy, Seurat v4, scikit-learn

**Other candidate tools:** FIt-SNE, Miscell, Seurat, anndata, scVI, Cellar, UMAP (umap-learn), faiss, python-louvain, scCAD, AnnData (Annotated Data object), Leiden, cisTopic, diffxpy

**Grounding:** 4 KB(s); DOIs: 10.1016/j.isci.2021.103200, 10.1038/s41467-022-29744-0, 10.1038/s41467-023-43019-2, 10.1038/s41467-024-51891-9

### Stage 5 — identify_cluster_markers

**Goal:** identify genes specifically upregulated in each cluster

**EDAM operation:** http://edamontology.org/operation_3223

**Inputs:** single-cell-object/filtered-raw-rna-counts, cluster-labels/leiden, yaml/species-tissue-cancer-context, marker-database/tissue-specific · **Outputs:** sccatch-object/cluster-markers, marker-table/cluster-specific

**Candidate leaf skills:** `marker-gene-identification-across-clusters` (primary), `sccatch-object-construction`, `scrnaseq-marker-gene-identification`

**Tools (primary):** Seurat, scCATCH

**Other candidate tools:** CHETAH, CellAssign, Garnett, Matrix, SingleR, devtools, scMap

**Grounding:** 1 KB(s); DOIs: 10.1016/j.isci.2020.100882

### Stage 6 — annotate_cell_types

**Goal:** assign a tissue-supported cell type to each single-cell cluster

**EDAM operation:** http://edamontology.org/operation_0314

**Inputs:** sccatch-object/cluster-markers · **Outputs:** annotation-table/cell-type-by-cluster, sccatch-object/annotated-clusters

**Candidate leaf skills:** `tissue-specific-marker-database-matching` (primary), `cell-type-annotation-scoring-against-reference-database`, `evidence-based-cell-type-scoring`, `scrnaseq-cluster-benchmarking`, `cellmatch-reference-database-querying`

**Tools (primary):** CellMatch, Seurat, rev_gene, scCATCH

**Other candidate tools:** CHETAH, CellAssign, Garnett, Matrix, SingleR, devtools, scMap

**Grounding:** 1 KB(s); DOIs: 10.1016/j.isci.2020.100882

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
