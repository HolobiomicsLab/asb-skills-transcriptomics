---
name: graph-contrastive-learning-imputation-run
description: Use when scRNA-seq count matrices in h5 files (expression values, true
  labels, metadata) have dropout artifacts and the goal is imputation plus clustering
  benchmarked by ARI/NMI against methods like MAGIC or scTAG.
when_to_use_negative:
- When the analysis goal is a downstream task other than node classification, clustering,
  or similarity — these are the only tasks scGCL's --task argument supports
- When the input is not an h5-formatted scRNA-seq expression matrix with labels/metadata;
  the dataset argument expects an h5 file
- When the environment cannot match the pinned dependency versions (Python 3.8.8,
  PyTorch 1.10.0, torch-geometric 2.2.0, faiss-cpu 1.7.3), which the paper reports
  as required
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGCL
  role: 'Core method: graph contrastive learning + ZINB autoencoder for scRNA-seq
    imputation, run from the command line via main.py'
  repo: https://github.com/zehaoxiong123/scGCL
- name: PyTorch
  role: Deep learning framework (version 1.10.0) backing the scGCL model implementation
- name: torch-geometric
  role: Graph neural network library (version 2.2.0) used to build the cell-cell graph
    for contrastive learning
- name: torch-cluster
  role: Graph clustering/neighbor operations (version 1.6.0) supporting the graph
    contrastive learning component
- name: torch-scatter
  role: Scatter reduction ops (version 2.0.9) for message passing in the graph model
- name: torch-sparse
  role: Sparse tensor ops (version 0.6.13) for efficient graph computations
- name: faiss-cpu
  role: Fast similarity search / nearest-neighbor library (version 1.7.3) used for
    tasks such as positive-sample selection or clustering
- name: MAGIC
  role: Baseline imputation method compared against scGCL for clustering ARI/NMI across
    the 14 datasets
- name: AutoClass
  role: Baseline imputation method compared against scGCL for clustering ARI/NMI across
    the 14 datasets
- name: GraphSCI
  role: Baseline imputation method compared against scGCL for clustering ARI/NMI across
    the 14 datasets
- name: scTAG
  role: Baseline imputation method compared against scGCL for clustering ARI/NMI across
    the 14 datasets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:3d179597fe77a2d5b1886ce4d4e61249859f9db3412299284867b4b65d4ef9ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - AutoClass
  - GraphSCI
  - MAGIC
  - PyTorch
  - Python
  - faiss-cpu
  - scGCL
  - scTAG
  - torch-cluster
  - torch-geometric
  - torch-scatter
  - torch-sparse
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 24
---

# graph-contrastive-learning-imputation-run

## Summary

Run scGCL, a graph contrastive learning method with a ZINB-based autoencoder, to impute dropout values in scRNA-seq expression matrices and produce cell representations for downstream clustering on h5-format datasets.

## When to use

Use when scRNA-seq count matrices in h5 files (expression values, true labels, metadata) have dropout artifacts and the goal is imputation plus clustering benchmarked by ARI/NMI against methods like MAGIC or scTAG.

## When NOT to use

- When the analysis goal is a downstream task other than node classification, clustering, or similarity — these are the only tasks scGCL's --task argument supports
- When the input is not an h5-formatted scRNA-seq expression matrix with labels/metadata; the dataset argument expects an h5 file
- When the environment cannot match the pinned dependency versions (Python 3.8.8, PyTorch 1.10.0, torch-geometric 2.2.0, faiss-cpu 1.7.3), which the paper reports as required

## Inputs

- h5 file containing a matrix of scRNA-seq expression values, true labels, and other information (e.g. the 'adam' Alzheimer's disease dataset)
- Command-line parameters: --dataset, --task (node|clustering|similarity), --es (early stopping criterion), --epochs (e.g. 300)

## Outputs

- Imputed scRNA-seq expression matrix with dropout values estimated via the ZINB autoencoder
- Cell cluster assignments for ARI/NMI evaluation against true labels (e.g. scGCL ARI 0.94 / NMI 0.87 on Quake_10x_Spleen)

## How to apply

Load each dataset as an h5 file containing the scRNA-seq expression matrix, true labels, and metadata. Run scGCL from the command line (e.g. python main.py --dataset <name> --task clustering --epochs 300), enabling the early stopping criterion ('es' argument) where desired; supported downstream tasks are node, clustering, and similarity. scGCL summarizes global and local semantic information through graph contrastive learning, selects appropriate positive samples to enhance target-node representation, and reconstructs data via a ZINB autoencoder to estimate dropout values. Use the pinned environment: Python 3.8.8, PyTorch 1.10.0, torch-geometric 2.2.0, torch-cluster 1.6.0, torch-scatter 2.0.9, torch-sparse 0.6.13, faiss-cpu 1.7.3. Judge success by comparing ARI/NMI of predicted clusters against true cell labels (scGCL reportedly averages ARI 0.82 / NMI 0.80 across 14 datasets, peaking at ARI 0.94 / NMI 0.87 on Quake_10x_Spleen) and by checking per-cluster cell counts against true cell-type numbers.

## Related tools

- **scGCL** (Core method: graph contrastive learning + ZINB autoencoder for scRNA-seq imputation, run from the command line via main.py) — https://github.com/zehaoxiong123/scGCL
- **PyTorch** (Deep learning framework (version 1.10.0) backing the scGCL model implementation)
- **torch-geometric** (Graph neural network library (version 2.2.0) used to build the cell-cell graph for contrastive learning)
- **torch-cluster** (Graph clustering/neighbor operations (version 1.6.0) supporting the graph contrastive learning component)
- **torch-scatter** (Scatter reduction ops (version 2.0.9) for message passing in the graph model)
- **torch-sparse** (Sparse tensor ops (version 0.6.13) for efficient graph computations)
- **faiss-cpu** (Fast similarity search / nearest-neighbor library (version 1.7.3) used for tasks such as positive-sample selection or clustering)
- **MAGIC** (Baseline imputation method compared against scGCL for clustering ARI/NMI across the 14 datasets)
- **AutoClass** (Baseline imputation method compared against scGCL for clustering ARI/NMI across the 14 datasets)
- **GraphSCI** (Baseline imputation method compared against scGCL for clustering ARI/NMI across the 14 datasets)
- **scTAG** (Baseline imputation method compared against scGCL for clustering ARI/NMI across the 14 datasets)

## Examples

```
python main.py --dataset adam --task clustering --epochs 300 --es
```

## Evaluation signals

- ARI and NMI computed between predicted clusters and true cell labels exceed those of AutoClass, GraphSCI, scTAG, and MAGIC on the same datasets (reported averages ARI 0.82 / NMI 0.80; Quake_10x_Spleen ARI 0.94 / NMI 0.87)
- Per-cluster cell counts from scGCL clustering are closest to the true cell-type numbers (e.g. on Quake_10x_Spleen, reproducing Supplementary Fig. S2)
- Training completes the specified --epochs (300) or stops via the --es early stopping criterion without errors in the pinned PyTorch 1.10.0 / torch-geometric 2.2.0 environment
- Input validation: the --dataset argument resolves to a valid h5 file with an expression matrix, true labels, and metadata
- Imputed expression on the Alzheimer's disease ('adam') dataset shows enhanced expression patterns of specific genes, as reported in the paper

## Limitations

- Existing imputation methods, including the motivation for scGCL, ignore topological structure information of scRNA-seq data and cannot comprehensively utilize relationships between cells — this skill presumes a cell-graph representation is appropriate
- Only three downstream tasks are supported (--task: node, clustering, similarity); other analyses require different tooling
- Strict dependency versions are required (Python 3.8.8, PyTorch 1.10.0, torch-geometric 2.2.0, torch-scatter 2.0.9, torch-sparse 0.6.13, torch-cluster 1.6.0, faiss-cpu 1.7.3), which can complicate installation

## Evidence

- [abstract] Graph contrastive learning with ZINB autoencoder: "scGCL, which integrates graph contrastive learning and Zero-inflated Negative Binomial (ZINB) distribution to estimate dropout values"
- [intro] Outperformance claim on 14 datasets: "we verify that scGCL outperforms existing state-of-the-art imputation methods in clustering performance and gene imputation on 14 real scRNA-seq datasets"
- [methods] CLI invocation: "$ python main.py --dataset adam --epochs 300"
- [readme] Dataset format: "dataset     | A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [readme] Supported tasks: "task  | Downstream task. Supported tasks are: node, clustering, similarity"
