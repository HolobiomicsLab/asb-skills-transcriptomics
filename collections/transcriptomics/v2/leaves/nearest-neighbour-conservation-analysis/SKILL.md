---
name: nearest-neighbour-conservation-analysis
description: Use when evaluating whether a batch-integration embedding (e.g., JOINTLY
  on a multi-donor scRNA-seq dataset) is robust to dropping input batches — subsample
  the donors and compare neighbour structure and metrics to the full reference.
when_to_use_negative:
- The dataset is a complex multi-condition, multi-sample dataset where samples do
  not share the same axes of variation — the paper advises regular PCA in JOINTLY
  rather than consensus PCA there, so neighbour-conservation relative to a consensus-PCA
  reference is not the right robustness check
- The goal is comparing JOINTLY against other integration methods (Harmony, scVI,
  fastMNN) rather than testing dependence on the number of input samples
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: JOINTLY
  role: Runs the full five-batch reference and each 2-4 batch subsample integration
    via jointly(liver, batch.var = 'batch_label') to produce comparable JOINTLY embeddings
  repo: https://github.com/madsen-lab/rJOINTLY
- name: Seurat
  role: Handles the liver Seurat object, subsampling of donor batches, and FindNeighbors/FindClusters
    clustering on the JOINTLY reduction
  repo: https://github.com/satijalab/seurat
- name: aricode
  role: Computes ARI (and NMI) between transferred cell type labels and seurat_clusters
    for each subsample
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:07e41eee9d7019e2c67aa8ab2093835bf69a715fc77e5eb05eccfaaaab96d890
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - JOINTLY
  - Seurat
  - aricode
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# nearest-neighbour-conservation-analysis

## Summary

Quantifies how many nearest neighbours each cell retains in a JOINTLY embedding built from a subsampled batch set compared with the full-dataset embedding, to test whether integration performance depends on the number of input samples. Used with ARI/LISI/ASW metrics to reproduce the liver subsampling robustness results (Fig. 2D-I).

## When to use

Use when evaluating whether a batch-integration embedding (e.g., JOINTLY on a multi-donor scRNA-seq dataset) is robust to dropping input batches — subsample the donors and compare neighbour structure and metrics to the full reference.

## When NOT to use

- The dataset is a complex multi-condition, multi-sample dataset where samples do not share the same axes of variation — the paper advises regular PCA in JOINTLY rather than consensus PCA there, so neighbour-conservation relative to a consensus-PCA reference is not the right robustness check
- The goal is comparing JOINTLY against other integration methods (Harmony, scVI, fastMNN) rather than testing dependence on the number of input samples

## Inputs

- Human_Liver.rds scRNA-seq Seurat object with five donor batches (batch_label) from Zenodo record 8298157
- Human_Liver_labels.tsv transferred cell type labels matched by cell barcode
- JOINTLY reduction (embedding) from the full five-batch reference run
- JOINTLY reductions from each of the 25 batch-subsampled runs

## Outputs

- subsample_integration_metrics.csv with per-subsample ARI, cell type and integration LISI, cell type and batch ASW, and fraction of conserved nearest neighbours
- Fraction of conserved nearest neighbours per subsample relative to the full five-batch JOINTLY embedding

## How to apply

Download the five-batch Human Liver dataset (Human_Liver.rds) from Zenodo record 8298157 and run the full reference JOINTLY with batch.var='batch_label'. Create 25 random subsamples retaining 2-4 of the five donor batches (batch_label), run jointly() on each, and cluster each JOINTLY reduction with FindNeighbors/FindClusters (dims 1:15, res=0.05), computing ARI against transferred cell type labels with aricode. On each subsample's JOINTLY embeddings compute LISI and ASW for both cell type and batch. For each subsample, compute the fraction of conserved nearest neighbours relative to the full five-batch JOINTLY embedding — this directly tests the paper's concern that consensus PCA makes each sample's initial decomposition depend on the other samples. Collate all metric values across subsamples into subsample_integration_metrics.csv; the expected outcome is no difference in ARI, LISI, or ASW versus the full dataset, indicating JOINTLY's performance is not strongly dependent on the number of input datasets.

## Related tools

- **JOINTLY** (Runs the full five-batch reference and each 2-4 batch subsample integration via jointly(liver, batch.var = 'batch_label') to produce comparable JOINTLY embeddings) — https://github.com/madsen-lab/rJOINTLY
- **Seurat** (Handles the liver Seurat object, subsampling of donor batches, and FindNeighbors/FindClusters clustering on the JOINTLY reduction) — https://github.com/satijalab/seurat
- **aricode** (Computes ARI (and NMI) between transferred cell type labels and seurat_clusters for each subsample)

## Examples

```
liver <- jointly(liver, batch.var = "batch_label"); liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15); aricode::ARI(liver$object$celltype, liver$object$seurat_clusters)
```

## Evaluation signals

- ARI, cell type and integration LISI, and cell type and batch ASW for each 2-4 batch subsample show no difference from the full five-batch dataset
- Fraction of conserved nearest neighbours relative to the full five-batch JOINTLY embedding remains high across all 25 subsamples
- The subsample ARI is in the same range as the README reference run (ARI 0.7415981, NMI 0.6325386 for the full liver dataset)
- All 25 subsamples were generated as combinations of 2-4 of the five batch_label donor batches and each yielded a valid jointly() solution
- Results collated into subsample_integration_metrics.csv reproduce the quantitative trends of Fig. 2D-I

## Limitations

- Setting a seed and re-running JOINTLY produces a different result; the README notes it can be worth running JOINTLY up to 5 times and choosing the best solution
- For complex multi-condition, multi-sample datasets where samples do not share the same axes of variation, the authors advise regular PCA in JOINTLY instead of consensus PCA
- Software version numbers are not stated in the methods, complicating exact reproduction

## Evidence

- [results] Subsampling no difference in ARI/LISI/ASW: [evidence span withheld — anchor status: not_found]
- [methods] workflow neighbour conservation: [evidence span withheld — anchor status: not_found]
- [intro] research question dependence on samples: [evidence span withheld — anchor status: not_found]
- [readme] clustering parameters: "liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15)"
- [readme] ARI reference value: "aricode::ARI(liver$object$celltype, liver$object$seurat_clusters) # 0.7415981"
- [discussion] consensus PCA caveat: "For more complex datasets, where all samples do not necessarily share the same axes of variation, such as multi-condition, multi-sample datasets, we advise users to use regular PCA in JOINTLY"
