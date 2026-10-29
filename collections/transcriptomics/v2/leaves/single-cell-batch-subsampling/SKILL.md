---
name: single-cell-batch-subsampling
description: Use when you need to test whether an integration method's clustering
  or batch-mixing metrics depend on how many donor batches (or which subsets) are
  provided as input, e.g., for consensus-PCA-based methods where each sample's initial
  decomposition depends on the other samples.
when_to_use_negative:
- When the goal is to benchmark one integration method against others on a fixed task
  rather than to test robustness to input batch number/composition for a single method
- When the dataset has only one or two batches, leaving too few samples to form 2-4
  batch subsets to compare
- When the integration method does not couple samples during initial decomposition
  (e.g., regular PCA per sample), since the motivation here is JOINTLY's consensus
  PCA dependency between samples
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: JOINTLY
  role: Batch integration method applied to each batch subsample and to the full five-batch
    reference via jointly() with batch.var = 'batch_label'
  repo: https://github.com/madsen-lab/rJOINTLY
- name: Seurat
  role: Loads Human_Liver.rds, performs FindNeighbors/FindClusters (dims 1:15, res
    = 0.05) on the JOINTLY reduction, and RunUMAP for embedding
  repo: https://github.com/satijalab/seurat
- name: aricode
  role: Computes ARI between transferred cell type labels and JOINTLY-derived cluster
    labels for each subsample
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:d62b55845afa49b8575d25e2a54030b69af62765a1a952809a16bdda1e76a2b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - JOINTLY
  - Seurat
  - aricode
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-batch-subsampling

## Summary

Assesses whether a multi-sample single-cell batch integration workflow's performance depends on the number and composition of input batches by systematically subsampling batch combinations and comparing integration metrics against the full-dataset reference.

## When to use

Use when you need to test whether an integration method's clustering or batch-mixing metrics depend on how many donor batches (or which subsets) are provided as input, e.g., for consensus-PCA-based methods where each sample's initial decomposition depends on the other samples.

## When NOT to use

- When the goal is to benchmark one integration method against others on a fixed task rather than to test robustness to input batch number/composition for a single method
- When the dataset has only one or two batches, leaving too few samples to form 2-4 batch subsets to compare
- When the integration method does not couple samples during initial decomposition (e.g., regular PCA per sample), since the motivation here is JOINTLY's consensus PCA dependency between samples

## Inputs

- Human_Liver.rds scRNA-seq Seurat object (5 donor batches, from Zenodo record 8298157)
- Human_Liver_labels.tsv transferred cell type labels
- batch_label batch annotation variable

## Outputs

- 25 subsampled JOINTLY integrations with JOINTLY reductions and cluster assignments
- Per-subsample ARI against transferred cell type labels
- Per-subsample cell type and batch LISI and ASW values
- Per-subsample nearest-neighbour conservation fraction vs. the full five-batch embedding
- Collated metrics table (subsample_integration_metrics.csv) reproducing Fig. 2D-I trends

## How to apply

Download the five-donor Human_Liver.rds scRNA-seq dataset and Human_Liver_labels.tsv from Zenodo record 8298157 and load with Seurat. Create 25 random subsamples, each retaining a random subset of 2-4 of the five batches (batch_label). Run the full five-batch JOINTLY reference once with jointly(liver, batch.var = 'batch_label'), then run the same call on each subsample. For each subsample, cluster the JOINTLY reduction with FindNeighbors/FindClusters (dims 1:15, res = 0.05), compute ARI against transferred cell type labels with aricode, and calculate LISI and ASW (cell type and batch) on the JOINTLY embeddings. Additionally compute the fraction of nearest neighbours conserved relative to the full five-batch embedding. Collate all metrics (e.g., into subsample_integration_metrics.csv) and judge success by whether subsampled metrics match the full-dataset values, indicating no strong dependence on the number of input datasets.

## Related tools

- **JOINTLY** (Batch integration method applied to each batch subsample and to the full five-batch reference via jointly() with batch.var = 'batch_label') — https://github.com/madsen-lab/rJOINTLY
- **Seurat** (Loads Human_Liver.rds, performs FindNeighbors/FindClusters (dims 1:15, res = 0.05) on the JOINTLY reduction, and RunUMAP for embedding) — https://github.com/satijalab/seurat
- **aricode** (Computes ARI between transferred cell type labels and JOINTLY-derived cluster labels for each subsample)

## Examples

```
liver <- jointly(liver, batch.var = "batch_label"); liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15); liver$object <- FindClusters(liver$object, res = 0.05); aricode::ARI(liver$object$celltype, liver$object$seurat_clusters)
```

## Evaluation signals

- ARI, cell type/batch LISI, and cell type/batch ASW of each 2-4 batch subsample match the full five-batch reference values (no difference reported in the finding)
- Nearest-neighbour conservation fraction relative to the full five-batch JOINTLY embedding is high across all 25 subsamples
- All 25 subsamples run successfully through the same jointly(batch.var = 'batch_label') pipeline with clusterable JOINTLY reductions (dims 1:15)
- Collated metrics table (subsample_integration_metrics.csv) reproduces the quantitative trends of Fig. 2D-I

## Limitations

- Consensus PCA in JOINTLY makes each sample's initial decomposition depend on the other samples; for complex multi-condition, multi-sample datasets the authors advise regular PCA instead, which changes the premise of this robustness test
- Results were demonstrated on a single five-batch human liver dataset; generalization to other tissues or conditions is not established by this analysis
- JOINTLY is stochastic: setting a seed and re-running produces different results, and the README advises running it up to 5 times and choosing the best solution, which complicates strict subsample comparisons
- Software version numbers were not stated in the methods, affecting reproducibility

## Evidence

- [methods] Subsampling robustness motivation: "liver <- jointly(liver, batch.var = "batch_label")"
- [methods] Clustering and metric computation parameters: "liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15)
liver$object <- FindClusters(liver$object, res = 0.05)"
- [methods] ARI computation on transferred labels: "aricode::ARI(liver$object$celltype, liver$object$seurat_clusters)"
- [readme] Data provenance for the liver dataset: "Datasets, labels, embeddings and results used for testing JOINTLY can be downloaded here: https://zenodo.org/record/8298157"
- [readme] Stochasticity of JOINTLY runs: "Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."
