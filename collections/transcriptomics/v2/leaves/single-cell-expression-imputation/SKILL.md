---
name: single-cell-expression-imputation
description: Use when raw or dropout-affected scRNA-seq count matrices (h5 files with
  expression values and true labels) need imputation to improve downstream clustering
  (ARI/NMI) or gene-expression recovery under synthetic dropout.
when_to_use_negative:
- When the analysis goal is to model cell-cell relationships from scratch rather than
  impute dropout — note the paper's motivation that existing imputation methods ignore
  the topological structure information of scRNA-seq data, but scGCL is an imputation
  tool, not a general graph-construction utility
- When the input is not a scRNA-seq expression matrix (e.g. bulk RNA-seq or already-complete
  feature tables without dropout), since the ZINB dropout model is specific to single-cell
  zero inflation
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGCL
  role: 'Core imputation method: graph contrastive learning plus ZINB autoencoder,
    run from the command line via main.py'
  repo: https://github.com/zehaoxiong123/scGCL
- name: PyTorch
  role: Deep-learning framework underlying the scGCL model training (version 1.10.0)
- name: torch-geometric
  role: Graph neural-network library implementing the graph contrastive learning component
    (version 2.2.0)
- name: faiss-cpu
  role: Nearest-neighbor search support for selecting positive samples among cell
    nodes (version 1.7.3)
- name: AutoClass
  role: Baseline imputation method compared against scGCL under synthetic dropout
- name: GraphSCI
  role: Baseline imputation method compared against scGCL under synthetic dropout
- name: MAGIC
  role: Baseline imputation method compared against scGCL under synthetic dropout
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:e5e5e6bdeddc4db19de5b5585284514e7305d86906681bff970179427946e009
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
  - torch-geometric
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-expression-imputation

## Summary

Impute dropout values in scRNA-seq expression matrices using scGCL, which combines graph contrastive learning (to summarize global and local cell-relationship semantics and select positive samples for target nodes) with a ZINB-based autoencoder that reconstructs expression against the prior distribution.

## When to use

Use when raw or dropout-affected scRNA-seq count matrices (h5 files with expression values and true labels) need imputation to improve downstream clustering (ARI/NMI) or gene-expression recovery under synthetic dropout.

## When NOT to use

- When the analysis goal is to model cell-cell relationships from scratch rather than impute dropout — note the paper's motivation that existing imputation methods ignore the topological structure information of scRNA-seq data, but scGCL is an imputation tool, not a general graph-construction utility
- When the input is not a scRNA-seq expression matrix (e.g. bulk RNA-seq or already-complete feature tables without dropout), since the ZINB dropout model is specific to single-cell zero inflation

## Inputs

- scRNA-seq expression matrix in h5 file format (e.g. the 'adam' Alzheimer's disease dataset) containing expression values, true labels, and other information
- Optionally, a scVI-style masked version of the matrix (10%, 30% or 50% of non-zero entries set to zero) with retained true values as ground truth

## Outputs

- Imputed cell-by-gene expression matrix with the same dimensions as the input h5 expression matrix (e.g. scgcl_ad_imputed_expression.csv)
- Clustering quality metrics (ARI, NMI) and AD-vs-control cell separation scores (e.g. scgcl_ad_metrics.csv)
- Imputation recovery metrics: median L1 distance, RMSE and cosine similarity per dropout rate (imputation_results.csv)

## How to apply

Load the scRNA-seq dataset from its h5 file, which contains the expression matrix, true cell labels, and other information. Run scGCL from the command line (e.g. python main.py --dataset adam --epochs 300) so graph contrastive learning summarizes global and local semantic information and selects appropriate positive samples to enhance the representation of target nodes; the ZINB-based autoencoder then reconstructs the expression matrix against the prior distribution to estimate dropout values. For clustering evaluation, run scGCL's clustering task on the imputed data using the true labels from the h5 file as the reference partition, and compute ARI and NMI (on the Alzheimer's 'adam' dataset the published values are ARI 0.62 and NMI 0.70). For imputation-quality evaluation, apply the scVI-style synthetic dropout strategy — randomly masking non-zero expression entries at 10%, 30% and 50% rates while retaining true values as ground truth — then compute median L1 distance, RMSE and cosine similarity between imputed and true values, comparing against baselines such as AutoClass, GraphSCI and MAGIC. The imputed matrix must retain the same cell-by-gene dimensions as the input h5 expression matrix.

## Related tools

- **scGCL** (Core imputation method: graph contrastive learning plus ZINB autoencoder, run from the command line via main.py) — https://github.com/zehaoxiong123/scGCL
- **PyTorch** (Deep-learning framework underlying the scGCL model training (version 1.10.0))
- **torch-geometric** (Graph neural-network library implementing the graph contrastive learning component (version 2.2.0))
- **faiss-cpu** (Nearest-neighbor search support for selecting positive samples among cell nodes (version 1.7.3))
- **AutoClass** (Baseline imputation method compared against scGCL under synthetic dropout)
- **GraphSCI** (Baseline imputation method compared against scGCL under synthetic dropout)
- **MAGIC** (Baseline imputation method compared against scGCL under synthetic dropout)

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- On the adam (Alzheimer's disease) dataset with published settings, computed ARI ≈ 0.62 and NMI ≈ 0.70, and AD-vs-control cell separation reproduces the ordering shown in Supplementary Fig. S3a
- Imputed matrix has the same cell-by-gene dimensions as the input h5 expression matrix
- Under scVI-style synthetic dropout (10%, 30%, 50%), scGCL achieves the best (lowest) median L1 distance and RMSE at all three rates versus AutoClass, GraphSCI and MAGIC
- Cosine similarity under synthetic dropout is competitive with AutoClass and MAGIC
- Imputed data improves clustering and separation relative to the raw scRNA-seq data

## Limitations

- Validated on 14 real scRNA-seq datasets with PyTorch 1.10.0 / torch-geometric 2.2.0 and Python 3.8.8; other versions may not be compatible
- Baseline comparison scope is limited to methods like AutoClass, GraphSCI and MAGIC under the scVI-style masking strategy; performance on other dropout simulations is not established here
- Downstream tasks supported by the CLI are limited to node, clustering and similarity (per README arguments)

## Evidence

- [abstract] graph contrastive learning + ZINB autoencoder pipeline: "scGCL, which integrates graph contrastive learning and Zero-inflated Negative Binomial (ZINB) distribution to estimate dropout values"
- [methods] CLI usage: "$ python main.py --dataset adam --epochs 300"
- [readme] dataset argument definition: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [intro] 14-dataset benchmark claim: "we verify that scGCL outperforms existing state-of-the-art imputation methods in clustering performance and gene imputation on 14 real scRNA-seq datasets"
- [intro] AD enhancement finding: "we find that scGCL can enhance the expression patterns of specific genes in Alzheimer's disease datasets"
