---
name: single-cell-rnaseq-clustering-and-annotation
description: Use when raw or unannotated scRNA-seq data (e.g., GEO-accessioned BALF,
  breast epithelial, or TNBC samples) must be quality-controlled, clustered, and assigned
  cell-type labels before downstream drug repurposing or per-cluster DEG computation.
when_to_use_negative:
- Input is already a labeled Seurat object or annotated cell-type table with cluster
  assignments — skip directly to DEG/GetDrug steps
- Data are bulk RNA-seq or drug-response profiles rather than single-cell counts;
  ASGARD's cluster-level scoring assumes single-cell heterogeneity
- Only a drug-reference library build is needed (PrepareReference/GetDrugRef on LINCS
  L1000 gctx files), which requires no scRNA-seq clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Normalization, QC filtering, sample integration (FindIntegrationAnchors, dims
    1:15), Louvain clustering at resolution 0.4, and UMAP visualization
  repo: https://github.com/lanagarmire/Asgard
- name: SingleR
  role: Automatic cluster annotation against the HumanPrimaryCellAtlasData reference
    panel (with celldex)
  repo: https://github.com/lanagarmire/Asgard
- name: limma
  role: Per-cluster case-vs-control differential expression (lmFit/eBayes on model.matrix(~0+type))
    on the annotated clusters, feeding GetDrug
  repo: https://github.com/lanagarmire/Asgard
- name: cmapR
  role: Handling of LINCS L1000 gctx perturbational profile files during downstream
    reference building
  repo: https://github.com/lanagarmire/Asgard
merged_aliases:
- single-cell-rna-seq-cluster-annotation
merged_alias_records:
- alias: single-cell-clustering-and-cell-type-annotation
  slug: single-cell-clustering-and-cell-type-annotation
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
- alias: scrnaseq-clustering-and-cell-annotation
  slug: scrnaseq-clustering-and-cell-annotation
  cosine_score: 0.821
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:e32acfc8a56d71fc4b53aac56918c9decc2819db887b67ff41dccd81ebf7170f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/Asgard
  tools:
  - ASGARD
  - CHETAH
  - CellAssign
  - Garnett
  - Matrix
  - Seurat
  - SingleR
  - TRANSACT
  - cmapR
  - devtools
  - limma
  - pROC
  - scMap
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# single-cell-rnaseq-clustering-and-annotation

## Summary

Preprocess, cluster, and annotate scRNA-seq datasets with Seurat and SingleR to produce cell-type-labeled clusters that serve as input for ASGARD's per-cluster differential expression and drug repurposing steps, addressing intercellular heterogeneity within each patient.

## When to use

Use when raw or unannotated scRNA-seq data (e.g., GEO-accessioned BALF, breast epithelial, or TNBC samples) must be quality-controlled, clustered, and assigned cell-type labels before downstream drug repurposing or per-cluster DEG computation.

## When NOT to use

- Input is already a labeled Seurat object or annotated cell-type table with cluster assignments — skip directly to DEG/GetDrug steps
- Data are bulk RNA-seq or drug-response profiles rather than single-cell counts; ASGARD's cluster-level scoring assumes single-cell heterogeneity
- Only a drug-reference library build is needed (PrepareReference/GetDrugRef on LINCS L1000 gctx files), which requires no scRNA-seq clustering

## Inputs

- Raw scRNA-seq count matrices from GEO (e.g., GSE145926 BALF, GSE113197/GSE123926 breast epithelial, GSE161529 TNBC, GSE132509 Pre-T ALL) as .txt expression matrices or RAW tar archives
- Optional pre-computed cell-type annotation tables (e.g., Normal_celltype.txt from the Single-cell-drug-repositioning repo)

## Outputs

- Integrated Seurat object with Louvain clusters (resolution 0.4) and UMAP embedding
- Per-cell SingleR cell-type annotations against HumanPrimaryCellAtlasData
- Cell metadata table (cell barcodes, sample, cell type, case/control status) for downstream DrugScore calls

## How to apply

Load count matrices into Seurat (CreateSeuratObject with min.cells=3, min.features=200), applying dataset-specific QC: for COVID-19 BALF (GSE145926), remove cells with <200 unique genes, >6000 unique genes, or >10% mitochondrial genes, and drop genes detected in fewer than 3 cells (rowSums(expr>0)<3); for TNBC/breast data, remove cells with <200 unique genes and genes seen in fewer than 3 cells. Normalize, keep the 2000 genes with highest standardized variance, integrate multiple patient samples with FindIntegrationAnchors (dims 1:15, anchor.features=2000), regress out cell-cycle scores, and cluster via PCA, KNN graph, and Louvain modularity optimization at resolution 0.4, with UMAP for visualization. Annotate clusters automatically with SingleR against HumanPrimaryCellAtlasData (celldex reference), yielding e.g., neutrophil, NK, T cell, and monocyte populations in COVID-19 BALF or 8 breast epithelial clusters. The resulting Seurat object plus cell metadata feed GetDrug (per-cluster DEGs via limma model.matrix(~0+type) contrast typeCase - typeControl) and DrugScore aggregation; omit cell types with fewer than min.cells=3 cells per group.

## Related tools

- **Seurat** (Normalization, QC filtering, sample integration (FindIntegrationAnchors, dims 1:15), Louvain clustering at resolution 0.4, and UMAP visualization) — https://github.com/lanagarmire/Asgard
- **SingleR** (Automatic cluster annotation against the HumanPrimaryCellAtlasData reference panel (with celldex)) — https://github.com/lanagarmire/Asgard
- **limma** (Per-cluster case-vs-control differential expression (lmFit/eBayes on model.matrix(~0+type)) on the annotated clusters, feeding GetDrug) — https://github.com/lanagarmire/Asgard
- **cmapR** (Handling of LINCS L1000 gctx perturbational profile files during downstream reference building) — https://github.com/lanagarmire/Asgard

## Examples

```
devtools::install_github("lanagarmire/Asgard"); library(Seurat); obj <- CreateSeuratObject(counts=data, min.cells=3, min.features=200)
```

## Evaluation signals

- QC invariant: no cells remain with <200 unique genes (or >6000 genes / >10% mitochondrial for COVID-19 BALF) and no genes detected in fewer than 3 cells
- Annotation sanity: clusters map to expected populations (e.g., neutrophil, NK cell, T cell, monocyte for GSE145926 BALF; 8 breast epithelial clusters for GSE123926/GSE113197)
- Cell types retained for DEG analysis must exceed min.cells=3 cells in both case and control groups
- Integrated object contains ~2000 highest-variance genes and clusters were produced at Louvain resolution 0.4 with dims 1:15
- Downstream validation: ASGARD DrugScore rankings on the annotated clusters reproduce published results (e.g., ASGARD mean AUC 0.83 over 8 breast clusters; rescinnamine rank 2 and enalapril rank 4 for COVID-19)

## Limitations

- Sparsity and heterogeneity of single-cell data can cause false discoveries of differentially expressed genes in downstream DEG steps
- Differential expression method choice affects downstream AUC performance; Limma (Bayes) showed the best average AUC among tested methods, so selection must be made per dataset
- Building the drug reference from LINCS L1000 (GSE70138/GSE92742) requires >64 GB RAM and takes more than one hour on a standard computer

## Evidence

- [discussion] QC thresholds for BALF cells: "BALF cells from COVID-19 patients with fewer than 200 unique genes, more than 6000 unique genes or have a proportion of mitochondrial genes larger than 10%"
- [discussion] SingleR annotation method: "we ran an automatic annotation of single cells based on similarity to the referenced single-cell panel using the SingleR package75"
- [discussion] Variance-based gene filtering: "We used the 2000 genes with the highest standardized variance for downstream analysis"
- [other] Cluster-level DEG computation in workflow: [evidence span withheld — anchor status: not_found]
- [readme] Seurat object creation parameters from README: "epithelial2 <- CreateSeuratObject(counts=data, project="Epithelial", min.cells=3, min.features=200, meta.data=metada)"
