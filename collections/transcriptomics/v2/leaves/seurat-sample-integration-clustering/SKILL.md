---
name: seurat-sample-integration-clustering
description: Use when multi-patient scRNA-seq samples (e.g., 2 Pre-T ALL patients
  vs 3 healthy controls from GSE132509) must be integrated across batches, clustered,
  and cell-type-annotated before per-cluster differential expression and drug scoring.
when_to_use_negative:
- Input is already an integrated, annotated Seurat object or a per-cluster DEG table
  — rerunning integration/clustering would discard prior annotation
- The analysis targets a single homogeneous sample without patient/control batches,
  where anchor-based integration is unnecessary and can distort expression
- Only bulk RNA-seq data are available — the skill requires single-cell count matrices
  and cell-level clustering
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0099
tools:
- name: Seurat
  role: Performs filtering, cell-cycle regression, anchor-based integration (FindIntegrationAnchors/IntegrateData),
    Louvain clustering, and UMAP visualization of multi-sample scRNA-seq data
  repo: https://github.com/lanagarmire/Asgard
- name: SingleR
  role: Automatically annotates single cells by similarity to a referenced single-cell
    panel to label T-cell clusters
  repo: https://github.com/lanagarmire/Asgard
- name: celldex
  role: Supplies the HumanPrimaryCellAtlasData reference used by SingleR for cluster
    annotation
  repo: https://github.com/lanagarmire/Asgard
- name: ASGARD
  role: 'Downstream consumer of the annotated Seurat object: per-cluster DEGs and
    metadata feed GetDrug and DrugScore'
  repo: https://github.com/lanagarmire/Asgard
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:658f085fb29c8c9609269007e8a0928ee744866bd533797fcebd16014348cd0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/Asgard
  tools:
  - ASGARD
  - Seurat
  - SingleR
  - celldex
  - cmapR
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# seurat-sample-integration-clustering

## Summary

Integrates multiple patient/control scRNA-seq samples with Seurat's anchor-based integration, clusters cells via Louvain modularity optimization, and annotates cell types with SingleR — the preprocessing stage of the ASGARD drug-repurposing pipeline.

## When to use

Use when multi-patient scRNA-seq samples (e.g., 2 Pre-T ALL patients vs 3 healthy controls from GSE132509) must be integrated across batches, clustered, and cell-type-annotated before per-cluster differential expression and drug scoring.

## When NOT to use

- Input is already an integrated, annotated Seurat object or a per-cluster DEG table — rerunning integration/clustering would discard prior annotation
- The analysis targets a single homogeneous sample without patient/control batches, where anchor-based integration is unnecessary and can distort expression
- Only bulk RNA-seq data are available — the skill requires single-cell count matrices and cell-level clustering

## Inputs

- Raw scRNA-seq count matrices from multiple patients and controls (e.g., GEO GSE132509 Pre-T ALL, GSE161529 TNBC, GSE145926/GSE158055 COVID-19 BALF)
- Cell cycle marker gene sets (cc.genes S and G2M gene sets)
- SingleR reference single-cell panel (HumanPrimaryCellAtlasData from celldex)
- Cell-type annotation table for normal samples (e.g., Normal_celltype.txt from the Single-cell-drug-repositioning repo)

## Outputs

- Seurat integrated object with batch-corrected expression across samples
- Louvain cell-cluster assignments (e.g., 4 T-cell clusters at resolution 0.4) and UMAP embeddings
- Cell-type annotations per cell/cluster from SingleR
- Cluster metadata ready for per-cluster differential expression (GetDrug input Gene.list and cell_metadata)

## How to apply

First preprocess the count matrices in Seurat: remove cells with fewer than 200 unique genes and genes detected in fewer than 3 cells, regress out cell cycle effects using the cc.genes S and G2M gene sets, and restrict downstream analysis to the top 2000 genes with the highest standardized variance. Integrate all patient and control samples with Seurat's FindIntegrationAnchors/IntegrateData using anchor.features = 2000 and dims 1:15 so that diseased and normal cells become comparable for per-cluster case-control signatures. Cluster the integrated object by Louvain modularity optimization (resolution 0.4, using PCA/KNN graph and Jaccard similarity, with UMAP for visualization), then automatically annotate cell types (e.g., T-cell clusters) with the SingleR package against the HumanPrimaryCellAtlasData reference from celldex. The resulting annotated, integrated Seurat object with cluster assignments feeds ASGARD's GetDrug/DrugScore steps; retaining cell types with more than min.cells = 3 cells per group is required before differential expression.

## Related tools

- **Seurat** (Performs filtering, cell-cycle regression, anchor-based integration (FindIntegrationAnchors/IntegrateData), Louvain clustering, and UMAP visualization of multi-sample scRNA-seq data) — https://github.com/lanagarmire/Asgard
- **SingleR** (Automatically annotates single cells by similarity to a referenced single-cell panel to label T-cell clusters) — https://github.com/lanagarmire/Asgard
- **celldex** (Supplies the HumanPrimaryCellAtlasData reference used by SingleR for cluster annotation) — https://github.com/lanagarmire/Asgard
- **ASGARD** (Downstream consumer of the annotated Seurat object: per-cluster DEGs and metadata feed GetDrug and DrugScore) — https://github.com/lanagarmire/Asgard

## Examples

```
devtools::install_github("lanagarmire/Asgard"); epithelial2 <- CreateSeuratObject(counts=data, project="Epithelial", min.cells=3, min.features=200)
```

## Evaluation signals

- Cells with fewer than 200 unique genes and genes detected in fewer than 3 cells are removed before integration (min.cells=3, min.features=200 in CreateSeuratObject)
- Integration uses anchor.features = 2000 and dims 1:15; downstream analysis restricted to the 2000 genes with highest standardized variance
- Louvain clustering at resolution 0.4 recovers the expected T-cell clusters (e.g., 4 clusters in the Pre-T ALL dataset) consistent with SingleR annotations
- Each retained cell type has more than min.cells = 3 cells in both case and control groups, a prerequisite for the limma/DESeq2/edgeR/Seurat-Wilcoxon differential expression step
- The integrated object's metadata contains cluster and cell-type labels compatible with ASGARD's DrugScore(cell_metadata, cluster_degs, ...) call

## Limitations

- Sparsity and heterogeneity of single-cell data usually cause false discoveries of differentially expressed genes, so the downstream DE method must be selected per dataset (limma showed the best average AUC)
- For COVID-19 BALF cells, additional filters are needed (remove cells with <200 genes, >6000 genes, or >10% mitochondrial genes) — the simpler breast/leukemia filter is not universal
- Cluster resolution and annotation quality depend on the SingleR reference panel; proper normal-tissue controls may not be feasible for some diseases

## Evidence

- [other] Integration anchor settings: [evidence span withheld — anchor status: not_found]
- [discussion] Filtering rule: "genes identified in fewer than three cells are removed from the dataset"
- [discussion] SingleR annotation: "we ran an automatic annotation of single cells based on similarity to the referenced single-cell panel using the SingleR package75"
- [discussion] Louvain clustering: "Then we applied the Louvain algorithm of modularity optimization to iteratively group cell pairs together"
- [readme] README install/example: "epithelial2 <- CreateSeuratObject(counts=data, project="Epithelial", min.cells=3, 
								  min.features=200"
