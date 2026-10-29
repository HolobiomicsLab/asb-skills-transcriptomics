---
name: baseline-method-benchmark-comparison
description: Use when proposing a graph-based single-cell method and you need to demonstrate
  it consistently outperforms a published baseline across a sweep of graph-construction
  hyperparameters (k neighbors, distance measures) using ARI/NMI.
when_to_use_negative:
- The goal is method development or hyperparameter tuning for a single configuration
  rather than head-to-head benchmarking against a published baseline
- Input data lacks true labels, making ARI/NMI clustering evaluation against ground
  truth impossible
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGCL
  role: 'Proposed method under test: graph contrastive learning with ZINB autoencoder,
    run via command line for the clustering task'
  repo: https://github.com/zehaoxiong123/scGCL
- name: scTAG
  role: Published baseline whose clustering performance is compared against scGCL
    across all k–distance combinations
- name: Python
  role: Runtime environment (version 3.8.8) for running scGCL and computing ARI/NMI
    metrics
- name: PyTorch
  role: Deep learning framework (version 1.10.0) backing the scGCL model
- name: torch-geometric
  role: Library (version 2.2.0) used for graph construction/contrastive learning operations
    on the KNN cell graph
- name: torch-cluster
  role: Library (version 1.6.0) supporting KNN graph construction
- name: faiss-cpu
  role: Library (version 1.7.3) supporting fast nearest-neighbor search for KNN cell-graph
    construction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:05db49c4255d4d8488b81e62360030d5d40cadde173dcc851d50f1891f630240
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - PyTorch
  - Python
  - faiss-cpu
  - scGCL
  - scTAG
  - torch-cluster
  - torch-geometric
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# baseline-method-benchmark-comparison

## Summary

Systematically compare a proposed method (e.g., scGCL graph contrastive learning with ZINB autoencoder) against a baseline (e.g., scTAG) across a hyperparameter grid, evaluating clustering performance with ARI and NMI on scRNA-seq data.

## When to use

Use when proposing a graph-based single-cell method and you need to demonstrate it consistently outperforms a published baseline across a sweep of graph-construction hyperparameters (k neighbors, distance measures) using ARI/NMI.

## When NOT to use

- The goal is method development or hyperparameter tuning for a single configuration rather than head-to-head benchmarking against a published baseline
- Input data lacks true labels, making ARI/NMI clustering evaluation against ground truth impossible

## Inputs

- h5 dataset file containing the scRNA-seq expression matrix and true labels (e.g., the adam Alzheimer's disease dataset)
- Hyperparameter grid: k in {5,10,15,20,25} and distance measures {city-block, cosine, euclidean, L2, manhattan}
- Baseline method (scTAG) results for comparison

## Outputs

- Sweep table of averaged ARI and NMI values for all k–distance combinations for both scGCL and scTAG
- Identified optimal configuration (cosine distance, k=15)
- Conclusion on whether the proposed method consistently outperforms the baseline

## How to apply

Build the KNN cell graph from the scRNA-seq expression matrix stored in an h5 file (e.g., the adam dataset with true labels), sweeping k over {5,10,15,20,25} and distance measures over {city-block, cosine, euclidean, L2, manhattan}. For each combination, run the method from the command line (python main.py --dataset adam with the clustering task and early stopping enabled) and compute ARI and NMI against true labels, averaging over repeated runs. Compile averaged ARI/NMI for every k–distance combination into a single sweep table alongside the baseline (scTAG) values. Verify correctness by checking that the table reproduces the reported finding: cosine distance best captures cell–cell distances on sparse scRNA-seq data, both methods peak at k=15, and scGCL consistently outperforms scTAG across all tested settings.

## Related tools

- **scGCL** (Proposed method under test: graph contrastive learning with ZINB autoencoder, run via command line for the clustering task) — https://github.com/zehaoxiong123/scGCL
- **scTAG** (Published baseline whose clustering performance is compared against scGCL across all k–distance combinations)
- **Python** (Runtime environment (version 3.8.8) for running scGCL and computing ARI/NMI metrics)
- **PyTorch** (Deep learning framework (version 1.10.0) backing the scGCL model)
- **torch-geometric** (Library (version 2.2.0) used for graph construction/contrastive learning operations on the KNN cell graph)
- **torch-cluster** (Library (version 1.6.0) supporting KNN graph construction)
- **faiss-cpu** (Library (version 1.7.3) supporting fast nearest-neighbor search for KNN cell-graph construction)

## Examples

```
python main.py --dataset adam --task clustering --es
```

## Evaluation signals

- Sweep table contains averaged ARI/NMI entries for every combination of k in {5,10,15,20,25} and all five distance measures
- Both scGCL and scTAG achieve their best clustering effect at k=15
- The cosine distance row yields the optimal average ARI/NMI among distance measures
- scGCL's ARI/NMI values exceed scTAG's at every tested k–distance combination
- All runs use early stopping enabled and results are averaged over repeated runs

## Limitations

- The comparison was performed on the adam dataset; results on other datasets or tissue types are not established by this sweep
- The grid is limited to k in {5,10,15,20,25} and five distance measures; configurations outside this grid are untested
- Clustering results depend on repeated runs and early stopping, so run-to-run variance should be averaged before drawing conclusions

## Evidence

- [results] cosine distance captures cell–cell distance most effectively on sparse scRNA-seq data: [evidence span withheld — anchor status: not_found]
- [methods] command-line invocation: "You can run the scGCL from the command line: $ python main.py --dataset adam --epochs 300"
- [readme] dataset argument definition: "dataset | A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [readme] supported downstream tasks: "task | Downstream task. Supported tasks are: node, clustering, similarity"
- [abstract] outperformance claim: "scGCL outperforms existing state-of-the-art imputation methods in clustering performance and gene imputation on 14 scRNA-seq datasets"
