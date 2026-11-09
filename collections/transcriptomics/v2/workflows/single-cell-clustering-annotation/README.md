# single-cell-clustering-annotation-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **filter_cells_and_genes** — remove low-quality cells and uninformative genes from raw single-cell counts  →  `single-cell-rna-seq-quality-filtering`, `scanpy-single-cell-data-preprocessing`, `scanpy-scrna-seq-data-handling`, `scanpy-single-cell-data-loading`
2. **normalize_expression** — construct a scaled log-CPM expression matrix for dimensionality reduction  →  `scrna-cpm-normalization-log1p-scaling`, `scanpy-expression-normalization`, `single-cell-count-matrix-preprocessing--c487867256c234ac`, `scanpy-single-cell-preprocessing-pipeline`, `single-cell-expression-preprocessing--09b3e71f0eac3812`
3. **compute_pca_embedding** — reduce normalized single-cell expression to a compact PCA representation  →  `pca-dimensionality-reduction-scrna--260081bc53a44a5a`, `pca-dimensionality-reduction-of-sc-rna-seq-counts`, `pca-latent-space-embedding`, `knn-graph-construction-from-pca-embeddings`, `single-cell-rnaseq-pca-dimensionality-reduction`
4. **cluster_cells** — build a neighborhood graph on the PCA embedding and assign Leiden clusters  →  `leiden-clustering-of-low-dimensional-embeddings`, `leiden-neighborhood-graph-clustering`, `leiden-graph-community-detection`, `louvain-graph-clustering--ae6203732f5c17a9`, `leiden-graph-community-clustering-on-topic-embeddings`
5. **identify_cluster_markers** — identify genes specifically upregulated in each cluster  →  `marker-gene-identification-across-clusters`, `sccatch-object-construction`, `scrnaseq-marker-gene-identification`
6. **annotate_cell_types** — assign a tissue-supported cell type to each single-cell cluster  →  `tissue-specific-marker-database-matching`, `cell-type-annotation-scoring-against-reference-database`, `evidence-based-cell-type-scoring`, `scrnaseq-cluster-benchmarking`, `cellmatch-reference-database-querying`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
