---
name: distance-metric-sweep-analysis
description: Use when building a KNN cell graph from sparse scRNA-seq expression data
  and you must choose k (e.g., 5–25) and a distance measure (cosine, euclidean, city-block,
  manhattan, L2) based on clustering ARI/NMI.
when_to_use_negative:
- The goal is gene imputation quality assessment rather than cell-graph-driven clustering
  performance.
- A validated graph configuration already exists for the dataset, so a full 5x5 sweep
  would only waste compute.
- The downstream task is node labeling or similarity ('node', 'similarity') rather
  than clustering, since ARI/NMI against true cell labels is not the target metric.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGCL
  role: Graph contrastive learning model whose clustering task is run once per (k,
    distance) combination to score the sweep
  repo: https://github.com/zehaoxiong123/scGCL
- name: faiss-cpu
  role: Fast approximate KNN neighbor search used to construct the cell graph from
    expression profiles
- name: torch-cluster
  role: KNN graph construction operations consumed by the PyTorch/PyG pipeline
- name: torch-geometric
  role: Graph data structures and message passing for the KNN cell graph fed to scGCL
- name: PyTorch
  role: Training backend for the scGCL clustering runs under early stopping
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:c9a7a996d650a68fa20af2d9607bb16d1e050ebd31fd10a20ccc0fa60d5b7ca4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - PyTorch
  - Python
  - faiss-cpu
  - scGCL
  - torch-cluster
  - torch-geometric
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# distance-metric-sweep-analysis

## Summary

Systematic sweep of KNN cell-graph hyperparameters — number of neighbors k and cell–cell distance measure — for scGCL clustering on scRNA-seq data, scoring each (k, metric) combination by ARI/NMI against true labels to identify the optimal graph configuration.

## When to use

Use when building a KNN cell graph from sparse scRNA-seq expression data and you must choose k (e.g., 5–25) and a distance measure (cosine, euclidean, city-block, manhattan, L2) based on clustering ARI/NMI.

## When NOT to use

- The goal is gene imputation quality assessment rather than cell-graph-driven clustering performance.
- A validated graph configuration already exists for the dataset, so a full 5x5 sweep would only waste compute.
- The downstream task is node labeling or similarity ('node', 'similarity') rather than clustering, since ARI/NMI against true cell labels is not the target metric.

## Inputs

- h5 file containing a scRNA-seq expression matrix, true labels, and other information (e.g., the adam dataset)
- List of neighbor counts k (5, 10, 15, 20, 25)
- List of distance measures (city-block, cosine, euclidean, L2, manhattan)
- scTAG baseline ARI/NMI values for comparison

## Outputs

- Sweep table of averaged ARI/NMI for every (k, distance-measure) combination
- Identified optimal graph configuration (cosine, k=15)
- Per-setting comparison of scGCL vs. scTAG clustering performance

## How to apply

Load the adam h5 file containing the scRNA-seq expression matrix and true labels. For each k in {5,10,15,20,25} and each distance in {city-block, cosine, euclidean, L2, manhattan}, build the KNN cell graph (faiss-cpu handles the neighbor search; torch-cluster/torch-geometric materialize the graph) and run the scGCL clustering task from the command line with early stopping enabled. Compute ARI and NMI against the true labels for every combination, averaging over repeated runs to smooth run-to-run variance. Compile the averaged ARI/NMI values into a single sweep table alongside the scTAG baseline. The expected optimum is the cosine distance with k=15: cosine captures cell–cell distance most effectively on sparse scRNA-seq data, and both scGCL and scTAG peak at k=15, with scGCL outperforming scTAG across all tested settings.

## Related tools

- **scGCL** (Graph contrastive learning model whose clustering task is run once per (k, distance) combination to score the sweep) — https://github.com/zehaoxiong123/scGCL
- **faiss-cpu** (Fast approximate KNN neighbor search used to construct the cell graph from expression profiles)
- **torch-cluster** (KNN graph construction operations consumed by the PyTorch/PyG pipeline)
- **torch-geometric** (Graph data structures and message passing for the KNN cell graph fed to scGCL)
- **PyTorch** (Training backend for the scGCL clustering runs under early stopping)

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- The cosine metric combined with k=15 yields the maximum averaged ARI/NMI in the sweep table.
- scGCL's averaged ARI/NMI exceeds scTAG's for every tested (k, metric) combination.
- ARI and NMI are computed against the true labels stored in the adam h5 file, not a proxy.
- Each sweep cell is averaged over repeated runs, not a single run.
- Early stopping is enabled for every sweep run so epoch count does not confound the metric comparison.

## Limitations

- The sweep covers only k in {5,10,15,20,25} and five distance measures; other grid values are untested.
- Results are anchored to the adam Alzheimer's disease dataset and may not transfer to other scRNA-seq datasets without re-running the sweep.
- Conclusions depend on averaging over repeated runs; single-run ARI/NMI can be noisy and may mask the true optimum.
- The comparison is relative to scTAG; other clustering baselines are not part of this sweep.

## Evidence

- [abstract] scGCL summarizes global and local semantic information through contrastive learning and selects positive samples to enhance the representation of target nodes: "scGCL summarizes global and local semantic information through contrastive learning and selects positive samples to enhance the representation of target nodes"
- [intro] we verify that scGCL outperforms existing state-of-the-art imputation methods in clustering performance and gene imputation on 14 real scRNA-seq datasets: "we verify that scGCL outperforms existing state-of-the-art imputation methods in clustering performance and gene imputation on 14 real scRNA-seq datasets"
- [methods] You can run the scGCL from the command line:: "You can run the scGCL from the command line:"
- [readme] A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information.: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [readme] Downstream task. Supported tasks are: node, clustering, similarity: "Downstream task. Supported tasks are: node, clustering, similarity"
