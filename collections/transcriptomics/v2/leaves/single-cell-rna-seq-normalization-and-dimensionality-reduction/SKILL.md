---
name: single-cell-rna-seq-normalization-and-dimensionality-reduction
description: Use when raw single-cell RNA-seq count data (e.g. CD34_sc from MetacellAnalysisToolkit)
  must be normalized and embedded prior to building metacells or computing QC metrics
  like purity, compactness, or separation.
when_to_use_negative:
- Input is already normalized and embedded (e.g. a processed .h5ad or Seurat object
  with PCA/UMAP already computed) — re-running would change the latent space and invalidate
  prior metacell memberships
- Data are scATAC-seq, where SEACells uses an LSI/SVD embedding (X_svd) rather than
  the scRNA-seq PCA pipeline described here
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Seurat
  role: Performs LogNormalize normalization, variable feature selection (nfeatures=2500),
    scaling, PCA (npcs=50) and UMAP embedding of the single-cell data
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
- name: MetacellAnalysisToolkit
  role: Packages the example single-cell data (CD34_sc) and reference metacell object
    (CD34_mc) and provides the downstream construction/QC interface that consumes
    the embedding
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
- name: SEACells
  role: One of the metacell backends (alongside SuperCell) that builds metacells from
    the low-dimensional embedding produced by this preprocessing step
  repo: https://github.com/dpeerlab/SEACells
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:aa9d048fed011d8628344f76319efcca5c99857bbd08e4825c8198954c62cdbb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GfellerLab/MetacellAnalysisToolkit
  tools:
  - MetacellAnalysisToolkit
  - SEACells
  - Seurat
  - SuperCell
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# single-cell-rna-seq-normalization-and-dimensionality-reduction

## Summary

Preprocess single-cell RNA-seq data with Seurat (LogNormalize normalization, variable feature selection, scaling, PCA, UMAP) to obtain the latent embedding required before metacell construction and QC in the MetacellAnalysisToolkit workflow.

## When to use

Use when raw single-cell RNA-seq count data (e.g. CD34_sc from MetacellAnalysisToolkit) must be normalized and embedded prior to building metacells or computing QC metrics like purity, compactness, or separation.

## When NOT to use

- Input is already normalized and embedded (e.g. a processed .h5ad or Seurat object with PCA/UMAP already computed) — re-running would change the latent space and invalidate prior metacell memberships
- Data are scATAC-seq, where SEACells uses an LSI/SVD embedding (X_svd) rather than the scRNA-seq PCA pipeline described here

## Inputs

- Seurat single-cell object with raw UMI counts (e.g. MetacellAnalysisToolkit::CD34_sc)
- Cell-level annotation (e.g. sc_data$celltype) for later purity computation

## Outputs

- Seurat object with normalized, scaled data and PCA (npcs = 50)
- UMAP embedding (dims = 1:50, n.neighbors = 15, min.dist = 0.5) used as the latent embedding for metacell construction
- Optionally a diffusion map from PCA via get_diffusion_comp(sc.obj = sc_data, dims = 1:30) for QC metrics

## How to apply

Load the packaged single-cell object (sc_data <- MetacellAnalysisToolkit::CD34_sc) and run the Seurat chain: NormalizeData with normalization.method = "LogNormalize" to convert counts to log-scale expression, FindVariableFeatures with nfeatures = 2500 to restrict to informative genes, ScaleData to z-score the variable features, then RunPCA with npcs = 50. From the PCA, compute the UMAP embedding with dims = 1:50, n.neighbors = 15, min.dist = 0.5 — this embedding is the latent space downstream metacell tools (SuperCell/SEACells backends) and QC metrics such as mc_compactness, mc_separation, and diffusion-map-based metrics (get_diffusion_comp, dims = 1:30) operate on. The rationale is that metacell grouping and metacell quality assessment are computed in a low-dimensional representation, so normalization and dimensionality-reduction choices directly control metacell purity and topology.

## Related tools

- **Seurat** (Performs LogNormalize normalization, variable feature selection (nfeatures=2500), scaling, PCA (npcs=50) and UMAP embedding of the single-cell data) — https://github.com/GfellerLab/MetacellAnalysisToolkit
- **MetacellAnalysisToolkit** (Packages the example single-cell data (CD34_sc) and reference metacell object (CD34_mc) and provides the downstream construction/QC interface that consumes the embedding) — https://github.com/GfellerLab/MetacellAnalysisToolkit
- **SEACells** (One of the metacell backends (alongside SuperCell) that builds metacells from the low-dimensional embedding produced by this preprocessing step) — https://github.com/dpeerlab/SEACells

## Examples

```
sc_data <- MetacellAnalysisToolkit::CD34_sc; sc_data <- NormalizeData(sc_data, normalization.method = "LogNormalize"); sc_data <- FindVariableFeatures(sc_data, nfeatures = 2500); sc_data <- ScaleData(sc_data); sc_data <- RunPCA(sc_data, npcs = 50); sc_data <- RunUMAP(sc_data, dims = 1:50, n.neighbors = 15, min.dist = 0.5)
```

## Evaluation signals

- After NormalizeData, expression values are log-normalized (LogNormalize method) rather than raw counts
- FindVariableFeatures identifies exactly nfeatures = 2500 variable genes
- RunPCA returns 50 principal components (npcs = 50) and RunUMAP uses dims = 1:50 with n.neighbors = 15, min.dist = 0.5
- Downstream metacells built from this embedding show nonzero purity when scored with mc_purity(membership, annotation = sc_data$celltype)
- Diffusion components computed from the PCA (get_diffusion_comp, dims = 1:30) succeed, confirming the embedding is valid input for QC metrics

## Limitations

- The tutorial's parameters (2500 variable features, 50 PCs, 15 neighbors, 0.5 min.dist) were tuned for the CD34_sc example; other datasets may need different settings
- Diffusion-map-based QC metrics use dims = 1:30 from the PCA, a different dimensionality than the UMAP embedding, so the two representations are not interchangeable
- A metacell is not a cell type — multiple metacells may share the same state — so the embedding quality alone does not guarantee correct cell-type recovery at coarse graining levels

## Evidence

- [methods] single-cell normalization, variable feature selection, scaling, PCA and UMAP embedding: [evidence span withheld — anchor status: not_found]
- [methods] NormalizeData LogNormalize: "sc_data <- NormalizeData(sc_data, normalization.method = "LogNormalize")"
- [methods] diffusion map from PCA: "We compute these metrics in a diffusion map obtained from the pca."
- [methods] metacell purity definition: "the **purity** of a metacell is the proportion of the most abundant cell type within the metacell"
- [methods] CD34 example data packaged in the toolkit: "sc_data <- MetacellAnalysisToolkit::CD34_sc"
