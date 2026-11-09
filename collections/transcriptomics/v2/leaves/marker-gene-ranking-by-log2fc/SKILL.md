---
name: marker-gene-ranking-by-log2fc
description: Use when Louvain/SNN cluster labels exist per cell and you must characterize
  each cluster by its top differentially expressed genes, e.g. to reproduce per-cluster
  marker depictions like the paper's Fig. 2E.
when_to_use_negative:
- Cells are not yet clustered — marker testing requires pre-existing cluster labels
  from a prior Louvain/SNN step
- The goal is annotating clusters against reference signatures (e.g. CIPR-style Cluster
  Identity Prediction) rather than de novo one-vs-rest marker discovery
- Input is scATAC-seq accessibility data, for which gene-expression DE tests like
  Wilcoxon/MAST/DESeq2 do not directly apply
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCALA
  role: Interactive R/Shiny pipeline that implements the cluster markers identification
    step (one-vs-rest DE testing and Log2FC ranking)
- name: Seurat
  role: Runs cluster-specific differential expression against all remaining cells
    with Wilcoxon rank sum, MAST, or DESeq2 tests
- name: presto
  role: Fast Wilcoxon rank-sum test and AUC across groups for large-scale marker discovery
  repo: https://github.com/immunogenomics/presto
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:6710de3532dd50fefdd423934eb7e379ef2439d997526b01973523dd1357923b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/immunogenomics/presto
  tools:
  - R
  - SCALA
  - Seurat
  - presto
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Marker gene ranking by Log2FC

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify and rank cluster-defining marker genes from scRNA-seq data by running one-vs-rest differential expression (Wilcoxon rank sum test, with MAST and DESeq2 as alternatives in SCALA) and ranking genes per cluster by Log2FC, as demonstrated on the PBMC3k dataset in SCALA (Seurat-based pipeline).

## When to use

Use when Louvain/SNN cluster labels exist per cell and you must characterize each cluster by its top differentially expressed genes, e.g. to reproduce per-cluster marker depictions like the paper's Fig. 2E.

## When NOT to use

- Cells are not yet clustered — marker testing requires pre-existing cluster labels from a prior Louvain/SNN step
- The goal is annotating clusters against reference signatures (e.g. CIPR-style Cluster Identity Prediction) rather than de novo one-vs-rest marker discovery
- Input is scATAC-seq accessibility data, for which gene-expression DE tests like Wilcoxon/MAST/DESeq2 do not directly apply

## Inputs

- QC-filtered, normalized and scaled scRNA-seq expression matrix (e.g. PBMC3k) with variable features (~2000) and PCA coordinates
- Per-cell Louvain cluster labels from SNN graph partitioning (cluster_assignments.csv barcode-to-cluster table)

## Outputs

- marker_genes_per_cluster.csv: per-cluster marker table ranked by Log2FC, top 10 markers per cluster (as in Fig. 2E)
- cluster_assignments.csv: cell barcode to Louvain cluster label assignment table

## How to apply

Starting from the QC-filtered, normalized, scaled scRNA-seq object (e.g. PBMC3k with ~2000 variable features and PCA coordinates chosen via 10-fold SVD cross-validation), build the shared-nearest-neighbour (SNN) graph in PCA space and partition it with the Louvain algorithm to obtain cluster labels. Then run cluster-specific differential expression in Seurat, testing each cluster's cells against all remaining cells using the Wilcoxon rank sum test on normalized expression values (MAST and DESeq2 are available as alternative tests in SCALA). Rank resulting genes per cluster by Log2FC and export the top 10 markers per cluster, mirroring the per-cluster marker depiction in Fig. 2E. The fast Wilcoxon implementation in presto (wilcoxauc) provides the same rank-sum + AUC statistics at scale if Seurat's default is too slow.

## Related tools

- **SCALA** (Interactive R/Shiny pipeline that implements the cluster markers identification step (one-vs-rest DE testing and Log2FC ranking))
- **Seurat** (Runs cluster-specific differential expression against all remaining cells with Wilcoxon rank sum, MAST, or DESeq2 tests)
- **presto** (Fast Wilcoxon rank-sum test and AUC across groups for large-scale marker discovery) — https://github.com/immunogenomics/presto

## Examples

```
markers <- FindAllMarkers(pbmc, test.use = "wilcox", only.pos = TRUE); top10 <- markers %>% group_by(cluster) %>% top_n(n = 10, wt = avg_log2FC)
```

## Evaluation signals

- marker_genes_per_cluster.csv exists with non-zero rows and contains per-cluster entries with log2FC-ranked genes
- cluster_assignments.csv exists with non-zero rows and every barcode carries a Louvain cluster label
- Top 10 markers per cluster match the paper's per-cluster marker depiction (Fig. 2E)
- Each cluster's marker test compared cells of that cluster against ALL remaining cells (one-vs-rest design)
- Marker ranking is by Log2FC and consistent with the primary statistical test (Wilcoxon rank sum on normalized expression)

## Limitations

- Only human and mouse datasets are currently supported by SCALA for this analysis
- The online SCALA version restricts dataset size to < 2 GB and 2 CPU threads per session; some steps cannot complete for datasets larger than 50,000 cells — prefer the desktop version
- Software version numbers for Seurat employed in SCALA are not stated in the publication, so exact reproducibility of marker tables may vary across Seurat versions

## Evidence

- [methods] SCALA identifies cluster marker genes by comparing each cluster's transcriptomes against the rest of the cells' transcriptomes through the Wilcoxon rank sum test on the normalized expression values.: [evidence span withheld — anchor status: not_found]
- [methods] Run cluster-specific differential expression in Seurat, testing each cluster's cells against all remaining cells, with the Wilcoxon rank sum test, MAST, and DESeq2.: [evidence span withheld — anchor status: not_found]
