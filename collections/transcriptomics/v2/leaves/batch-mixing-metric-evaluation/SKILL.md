---
name: batch-mixing-metric-evaluation
description: Use when you have embeddings from ≥2 batch integration methods (e.g.
  JOINTLY, scVI, Harmony, Scanorama, LIGER, fastMNN, Seurat, scGPT) run on the same
  multi-batch sxRNA-seq datasets with cell type labels and need a comparable rank
  table.
when_to_use_negative:
- You need to know which genes or features drive integration/clustering — the paper
  notes such metrics 'do not provide insight into the genes or features driving integration
  and clustering'; use JOINTLY's interpretable modules/UCell scores instead
- You are evaluating a single method on a single dataset in isolation — the paper
  stresses performance varies task-to-task, so single-task ARI is not a defensible
  ranking on its own
- The method's output is smoothed/imputed expression (e.g. JOINTLY's H/F-matrix recon_expression
  values) rather than a clustering-ready embedding; the paper advises using such values
  only for visualization
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: aricode
  role: Computes ARI and NMI between cell type labels and cluster labels for every
    method-dataset combination
- name: Seurat
  role: Clusters each method's embedding (FindNeighbors/FindClusters on the method
    reduction, dims 1:15, res 0.05) to produce cluster labels for ARI/NMI
  repo: https://github.com/satijalab/seurat
- name: JOINTLY
  role: One of the nine benchmarked methods; its JOINTLY reduction (dims 1:15) supplies
    embeddings scored by the metrics
  repo: https://github.com/madsen-lab/rJOINTLY
- name: scVI
  role: Competing deep learning-based integration method whose embeddings are scored;
    ranks first overall in the reproduced benchmark
- name: Harmony
  role: Statistical integration method benchmarked in the five-task rank comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:9c90073fcebf37d986210a2b6695cb33a7659413b3cfbb8da54aa8fafbdd1be4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat
  tools:
  - Harmony
  - JOINTLY
  - LIGER
  - MAESTRO
  - PyTorch
  - Scanorama
  - Seurat
  - aricode
  - fastMNN
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# batch-mixing-metric-evaluation

## Summary

Benchmark protocol from the JOINTLY paper for scoring batch-integration methods: cluster on each method's embedding and compute ARI/NMI between cell type labels and clusters, plus batch-mixing and cell-type-conservation metrics (iLISI, cLISI, bASW, cASW), then aggregate into per-task ranks across five integration tasks (Lung, Pancreas, Kidney, Liver, PBMC).

## When to use

Use when you have embeddings from ≥2 batch integration methods (e.g. JOINTLY, scVI, Harmony, Scanorama, LIGER, fastMNN, Seurat, scGPT) run on the same multi-batch sxRNA-seq datasets with cell type labels and need a comparable rank table.

## When NOT to use

- You need to know which genes or features drive integration/clustering — the paper notes such metrics 'do not provide insight into the genes or features driving integration and clustering'; use JOINTLY's interpretable modules/UCell scores instead
- You are evaluating a single method on a single dataset in isolation — the paper stresses performance varies task-to-task, so single-task ARI is not a defensible ranking on its own
- The method's output is smoothed/imputed expression (e.g. JOINTLY's H/F-matrix recon_expression values) rather than a clustering-ready embedding; the paper advises using such values only for visualization

## Inputs

- Method-generated cell embeddings (reductions) from integration methods run on the same datasets
- Multi-batch sxRNA-seq datasets with cell type labels (e.g. Human_Liver.rds + Human_Liver_labels.tsv from Zenodo record 8298157)
- Cell type label table mapping cell barcodes to Transferred_labels
- Cluster labels from Seurat FindClusters (seurat_clusters)

## Outputs

- Per method-dataset ARI and NMI clustering accuracy scores
- Batch-mixing metrics (iLISI, bASW) and cell-type-conservation metrics (cLISI, cASW) per embedding
- Aggregated per-task rank table across all methods and datasets (integration_benchmark_rank_table.csv)

## How to apply

Run each integration method with its standard pipeline settings on the same labelled, multi-batch datasets (the paper uses five tasks: Lung, Pancreas, Kidney, Liver, PBMC). For clustering-based scoring, cluster each embedding with Seurat's FindNeighbors and FindClusters on the method-specific reduction (e.g. reduction = "JOINTLY", dims = 1:15, res = 0.05), subset to named cell types with ≥10 cells, then compute aricode::ARI and aricode::NMI between celltype labels and seurat_clusters. Separately compute batch-mixing and cell-type-conservation metrics on each embedding: iLISI, cLISI, bASW and cASW. Aggregate all metric scores across methods and datasets, convert to per-task ranks, and write the summary table (e.g. integration_benchmark_rank_table.csv, reproducing Fig. 2A-C). Because no method is always best across tasks, interpret ranks per task rather than only on the grand average.

## Related tools

- **aricode** (Computes ARI and NMI between cell type labels and cluster labels for every method-dataset combination)
- **Seurat** (Clusters each method's embedding (FindNeighbors/FindClusters on the method reduction, dims 1:15, res 0.05) to produce cluster labels for ARI/NMI) — https://github.com/satijalab/seurat
- **JOINTLY** (One of the nine benchmarked methods; its JOINTLY reduction (dims 1:15) supplies embeddings scored by the metrics) — https://github.com/madsen-lab/rJOINTLY
- **scVI** (Competing deep learning-based integration method whose embeddings are scored; ranks first overall in the reproduced benchmark)
- **Harmony** (Statistical integration method benchmarked in the five-task rank comparison)

## Examples

```
liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15); liver$object <- FindClusters(liver$object, res = 0.05); aricode::ARI(liver$object$celltype, liver$object$seurat_clusters)
```

## Evaluation signals

- The liver dataset ARI for JOINTLY must match the deposited reference value 0.7415981 (and NMI 0.6325386)
- The reproduced rank table should show JOINTLY ranked second of nine methods overall, superseded only by scVI, with the other interpretable methods (e.g. LIGER) ranked eighth and ninth
- Every metric is computed on the same embedding and label set for all method-dataset combinations (same subsetting rule: named cell types with >10 cells)
- Clusters are produced with the fixed hyperparameters (dims 1:15, res 0.05) so ARI values are comparable across methods
- The summary table reproduces the per-task structure of Fig. 2A-C, with ranks per task rather than a single global average

## Limitations

- No method is always the best across all tasks, so performance must be evaluated on each new integration task rather than trusted from one benchmark
- Batch-integration metrics do not reveal which genes or features drive integration and clustering
- Setting a seed and re-running JOINTLY produces a different result; it can be worth running up to 5 times and choosing the best solution, which affects reproducibility of single-run scores
- The integration/clustering task balances batch-effect removal against biological-variation conservation, so metric scores are task-dependent

## Evidence

- [results] Reference liver ARI is a deposited validation value: [evidence span withheld — anchor status: not_found]
- [results] Metrics computed on each method's embedding: [evidence span withheld — anchor status: not_found]
- [abstract] Benchmark outcome reproduced by the rank table: "JOINTLY performs on par or better than state-of-the-art batch integration methods in clustering tasks and outperforms other intrinsically interpretable methods."
- [discussion] Metrics lack mechanistic insight: "such metrics do not provide insight into the genes or features driving integration and clustering."
- [readme] Concrete metric computation from README: "aricode::ARI(liver$object$celltype, liver$object$seurat_clusters) # 0.7415981"
- [discussion] No universal best method across tasks: "no method is always the best across all tasks, and therefore, that it is critical to evaluate the performance of a chosen method on any new integration task."
