---
name: graph-contrastive-representation-learning
description: Use when scRNA-seq count matrices stored as h5 (expression values plus
  true labels) suffer from dropout and existing imputation methods that ignore topological
  structure cannot comprehensively utilize relationships between cells.
when_to_use_negative:
- When the analysis goal requires probabilistic features only — the ablation shows
  Only_ZINB can outperform full scGCL (e.g. on the Romanov dataset), so graph representation
  learning may sometimes disrupt the ZINB distribution's benefits
- When the input is not single-cell expression data with cell-cell relationships to
  model (the method is designed for scRNA-seq dropout imputation, not generic tabular
  data)
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scGCL
  role: Reference implementation combining graph contrastive learning with a ZINB-based
    autoencoder for scRNA-seq imputation and clustering
  repo: https://github.com/zehaoxiong123/scGCL
- name: PyTorch
  role: Deep-learning backend (version 1.10.0) for training the contrastive graph
    model
- name: torch-geometric
  role: Graph neural network library (version 2.2.0) implementing the graph convolutions
    used by scGCL
- name: faiss-cpu
  role: Dependency (version 1.7.3) supporting nearest-neighbor positive-sample selection
    in the contrastive module
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:9483ca29d57707e1eab0f31ae191a20dda29e3fd06956a230eaf10bc27071ca7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - PyTorch
  - Python
  - faiss-cpu
  - scGCL
  - torch-geometric
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# graph-contrastive-representation-learning

## Summary

Graph contrastive learning that summarizes global and local cell-relationship semantics and selects positive samples for target nodes, combined with a ZINB-based autoencoder, to impute dropout values in scRNA-seq data (as implemented in scGCL).

## When to use

Use when scRNA-seq count matrices stored as h5 (expression values plus true labels) suffer from dropout and existing imputation methods that ignore topological structure cannot comprehensively utilize relationships between cells.

## When NOT to use

- When the analysis goal requires probabilistic features only — the ablation shows Only_ZINB can outperform full scGCL (e.g. on the Romanov dataset), so graph representation learning may sometimes disrupt the ZINB distribution's benefits
- When the input is not single-cell expression data with cell-cell relationships to model (the method is designed for scRNA-seq dropout imputation, not generic tabular data)

## Inputs

- h5 file containing a scRNA-seq expression matrix, true cell labels, and other information
- Optionally, scVI-style synthetically masked data (10%, 30% or 50% of non-zero entries set to zero) with retained true values as ground truth

## Outputs

- Imputed cell-by-gene expression matrix (e.g. scgcl_ad_imputed_expression.csv) with the same dimensions as the input h5 matrix
- Cell embeddings for downstream node/clustering/similarity tasks
- Clustering metrics (ARI/NMI, e.g. ARI 0.62 and NMI 0.70 on the adam dataset) and imputation metrics (median L1 distance, RMSE, cosine similarity)

## How to apply

Load the h5 expression matrix with true labels, then run scGCL's main.py from the command line (e.g. --dataset adam --epochs 300; optional task and early-stopping arguments are supported: node, clustering, similarity). The graph contrastive module summarizes global and local semantic information and selects appropriate positive samples to enhance the representation of target nodes, while a ZINB-based autoencoder reconstructs the scRNA-seq data against the prior distribution to estimate dropout values. For clustering use the learned cell embeddings and compute ARI/NMI against the true labels; for imputation quality compute median L1 distance, RMSE and cosine similarity against ground-truth values under scVI-style synthetic dropout (10%/30%/50%). Use the pinned environment: Python 3.8.8, PyTorch 1.10.0, torch-geometric 2.2.0, torch-cluster 1.6.0, torch-scatter 2.0.9, torch-sparse 0.6.13, faiss-cpu 1.7.3.

## Related tools

- **scGCL** (Reference implementation combining graph contrastive learning with a ZINB-based autoencoder for scRNA-seq imputation and clustering) — https://github.com/zehaoxiong123/scGCL
- **PyTorch** (Deep-learning backend (version 1.10.0) for training the contrastive graph model)
- **torch-geometric** (Graph neural network library (version 2.2.0) implementing the graph convolutions used by scGCL)
- **faiss-cpu** (Dependency (version 1.7.3) supporting nearest-neighbor positive-sample selection in the contrastive module)

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- Imputed matrix must have the same cell-by-gene dimensions as the input h5 expression matrix
- On simulated dropout (10%, 30%, 50% scVI-style masking on Zeisel and Klein), scGCL should achieve the best (lowest) median L1 distance and RMSE at all three rates, with cosine similarity competitive with AutoClass and MAGIC
- Clustering on the adam dataset should reproduce approximately ARI 0.62 and NMI 0.70, and separate AD versus control cells more efficiently than raw data (Supplementary Fig. S3a)
- Ablation sanity check: removing the ZINB encoder should degrade results, confirming the ZINB distribution is necessary
- Reported result scope: outperforms state-of-the-art imputation methods on 14 real scRNA-seq datasets

## Limitations

- Graph representation learning can sometimes disrupt the probabilistic (ZINB) features of the data — Only_ZINB outperformed full scGCL on the Romanov dataset in the ablation
- Method relies on graph/topological structure of cells; existing imputation methods that ignore topological structure 'cannot comprehensively utilize the relationships between cells', implying performance depends on meaningful cell-relationship graphs
- Specific pinned dependency versions (Python 3.8.8, PyTorch 1.10.0, torch-geometric 2.2.0, etc.) may be required for reproducibility

## Evidence

- [abstract] scGCL summarizes global and local semantic information through contrastive learning and selects appropriate positive samples to enhance the representation of target nodes.: "scGCL summarizes global and local semantic information through contrastive learning and selects appropriate positive samples to enhance the representation of target nodes."
- [intro] ZINB autoencoder reconstructs data from prior distribution: "scGCL introduces an autoencoder based on the ZINB distribution, which reconstructs the scRNA-seq data based on the prior distribution"
- [readme] Command-line usage with dataset and epochs arguments: "You can run the scGCL from the command line: $ python main.py --dataset adam --epochs 300"
- [readme] Task argument enumerates supported downstream tasks: "Downstream task. Supported tasks are: node, clustering, similarity"
- [results] Ablation caveat about graph learning disrupting probabilistic features: [evidence span withheld — anchor status: not_found]
