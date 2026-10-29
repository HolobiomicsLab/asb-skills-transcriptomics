---
name: clustering-accuracy-metric-computation
description: Use when a JOINTLY (or competing integration method) clustering must
  be quantitatively scored against known cell type labels, e.g. benchmark ARI/NMI
  tables or rank aggregation across integration tasks.
when_to_use_negative:
- No ground-truth cell type labels are available — ARI/NMI cannot be computed and
  only batch-mixing metrics (iLISI, cLISI, bASW, cASW) apply
- The goal is insight into which genes or features drive integration and clustering
  — the article notes that 'such metrics do not provide insight into the genes or
  features driving integration and clustering'
- Input is a raw expression matrix without a JOINTLY reduction — clustering must first
  be produced before accuracy can be scored
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: aricode
  role: Computes ARI (and NMI) between cell type labels and cluster labels for each
    method–dataset combination
- name: JOINTLY
  role: Produces the joint-clustering embedding (via jointly() with batch.var = "batch_label")
    that is clustered and then scored
  repo: https://github.com/madsen-lab/rJOINTLY
- name: Seurat
  role: Provides FindNeighbors/FindClusters (dims 1:15, res 0.05) to generate the
    cluster labels fed into aricode
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:431ccd5265e3f42e98b59d4123bebea781dbf720cb5e0707163d89cbbb809541
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - Harmony
  - JOINTLY
  - LIGER
  - Scanorama
  - Seurat
  - aricode
  - fastMNN
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# clustering-accuracy-metric-computation

## Summary

Quantitatively validates joint-clustering output against transferred cell type labels by computing ARI and NMI with the aricode R package, as used to score JOINTLY against eight batch-integration methods across five integration tasks (Lung, Pancreas, Kidney, Liver, PBMC).

## When to use

Use when a JOINTLY (or competing integration method) clustering must be quantitatively scored against known cell type labels, e.g. benchmark ARI/NMI tables or rank aggregation across integration tasks.

## When NOT to use

- No ground-truth cell type labels are available — ARI/NMI cannot be computed and only batch-mixing metrics (iLISI, cLISI, bASW, cASW) apply
- The goal is insight into which genes or features drive integration and clustering — the article notes that 'such metrics do not provide insight into the genes or features driving integration and clustering'
- Input is a raw expression matrix without a JOINTLY reduction — clustering must first be produced before accuracy can be scored

## Inputs

- Seurat (or SingleCellExperiment) object with a 'JOINTLY' dimensional reduction and seurat_clusters
- Cell type label file (e.g. Human_Liver_labels.tsv from Zenodo record 8298157) with Transferred_labels keyed by cell barcode
- Cluster assignments from FindNeighbors/FindClusters (dims = 1:15, res = 0.05) on the JOINTLY reduction

## Outputs

- Scalar ARI and NMI values per method–dataset pair (reference liver ARI: 0.7415981, NMI: 0.6325386)
- Aggregated per-task rank table (integration_benchmark_rank_table.csv, reproducing Fig. 2A–C)
- Per-subsample metric tables (subsample_integration_metrics.csv, reproducing Fig. 2D–I trends)

## How to apply

After running jointly() with batch.var = "batch_label", cluster the JOINTLY reduction via FindNeighbors/FindClusters (dims = 1:15, res = 0.05). Import the labels file (e.g. Human_Liver_labels.tsv), match label rows to colnames of the object, and subset to named cell types with ≥10 cells (excluding unnamed ''). Then call aricode::ARI and aricode::NMI between the transferred celltype labels and seurat_clusters; the deposited liver reference values are ARI = 0.7415981 and NMI = 0.6325386. For benchmarking, repeat this for every method–dataset combination, aggregate the scores into per-task ranks, and write a summary CSV (e.g. integration_benchmark_rank_table.csv reproducing Fig. 2A–C); for robustness testing, compute ARI per subsample against the full five-batch reference. Because results vary with seed, run JOINTLY up to 5 times and keep the best solution before scoring.

## Related tools

- **aricode** (Computes ARI (and NMI) between cell type labels and cluster labels for each method–dataset combination)
- **JOINTLY** (Produces the joint-clustering embedding (via jointly() with batch.var = "batch_label") that is clustered and then scored) — https://github.com/madsen-lab/rJOINTLY
- **Seurat** (Provides FindNeighbors/FindClusters (dims 1:15, res 0.05) to generate the cluster labels fed into aricode) — https://github.com/satijalab/seurat

## Examples

```
aricode::ARI(liver$object$celltype, liver$object$seurat_clusters)
```

## Evaluation signals

- The Human Liver reference values must be reproduced exactly: aricode::ARI returns 0.7415981 and aricode::NMI returns 0.6325386
- All ARI/NMI values fall within [0, 1], with ARI ≈ 0 indicating label-cluster agreement no better than chance
- Aggregated ranks reproduce the benchmark outcome in which JOINTLY ranks second of nine methods, superseded only by scVI, and the other interpretable methods rank eighth and ninth
- Subsampling 2–4 of the five liver donor batches yields no difference in ARI versus the full five-batch dataset
- Labels were matched to colnames and filtered to named cell types with ≥10 cells before scoring — unmatched or empty-label ('') cells would change the metric

## Limitations

- Results are seed-dependent: 'Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution.'
- Clustering accuracy must be re-evaluated per integration task, since 'no method is always the best across all tasks'
- Subsampling comparisons assume the consensus-PCA initialization; for complex multi-condition, multi-sample datasets the article advises regular PCA in JOINTLY instead

## Evidence

- [methods] article_methods_ari: "aricode::ARI(liver$object$celltype, liver$object$seurat_clusters)"
- [readme] readme_reference_values: "aricode::ARI(liver$object$celltype, liver$object$seurat_clusters) # 0.7415981
aricode::NMI(liver$object$celltype, liver$object$seurat_clusters) # 0.6325386"
- [readme] readme_subset_filter: "liver$object <- subset(liver$object, celltype %in% names(which(table(liver$object$celltype) >= 10)))"
- [discussion] discussion_rank_context: "JOINTLY performs on par with state-of-the-art batch integration tools, such as scVI3 and Harmony2, in clustering tasks"
- [abstract] abstract_performance_claim: "JOINTLY performs on par or better than state-of-the-art batch integration methods in clustering tasks and outperforms other intrinsically interpretable methods."
