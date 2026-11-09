---
name: zinb-autoencoder-dropout-estimation
description: Use when scRNA-seq expression data contain dropout events that need imputation
  prior to downstream clustering or gene-expression analysis, and a distribution-aware
  (ZINB prior) reconstruction is preferred over simple smoothing.
when_to_use_negative:
- The data are not single-cell RNA-seq with zero-inflation/dropout (e.g. bulk RNA-seq
  or already-complete feature matrices), since the ZINB prior models dropout events
  specific to scRNA-seq.
- The downstream goal does not require imputation — e.g. existing imputation methods
  are being compared only on raw data, or topological cell-relationship structure
  cannot be leveraged.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGCL
  role: Implements the ZINB-based autoencoder that reconstructs the scRNA-seq expression
    matrix against the prior distribution to estimate dropout values; run via main.py
    from the command line.
  repo: https://github.com/zehaoxiong123/scGCL
- name: PyTorch
  role: Deep-learning backend used to train the ZINB autoencoder (published version
    1.10.0).
- name: torch-geometric
  role: Graph-learning library supporting the graph contrastive component that supplies
    enhanced node representations alongside the ZINB reconstruction (version 2.2.0).
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:520ca934374fde255884ce2935c10cc52a5a4f919de087809f29c29489894841
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - PyTorch
  - Python
  - scGCL
  - torch-geometric
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# zinb-autoencoder-dropout-estimation

## Summary

Estimates dropout values in scRNA-seq data by reconstructing the expression matrix with a Zero-inflated Negative Binomial (ZINB) distribution–based autoencoder, as implemented inside scGCL. The ZINB prior captures the global probability distribution of dropout events so imputed values follow the generative structure of count data.

## When to use

Use when scRNA-seq expression data contain dropout events that need imputation prior to downstream clustering or gene-expression analysis, and a distribution-aware (ZINB prior) reconstruction is preferred over simple smoothing.

## When NOT to use

- The data are not single-cell RNA-seq with zero-inflation/dropout (e.g. bulk RNA-seq or already-complete feature matrices), since the ZINB prior models dropout events specific to scRNA-seq.
- The downstream goal does not require imputation — e.g. existing imputation methods are being compared only on raw data, or topological cell-relationship structure cannot be leveraged.

## Inputs

- h5 file containing a scRNA-seq expression matrix (cell-by-gene counts), true cell labels, and other accompanying information (e.g. the 'adam' Alzheimer's disease dataset)

## Outputs

- Imputed cell-by-gene scRNA-seq expression matrix with the same dimensions as the input (e.g. scgcl_ad_imputed_expression.csv)

## How to apply

Load the scRNA-seq expression matrix from an h5 file (e.g. the 'adam' Alzheimer's disease dataset), which also carries true labels and accompanying information. Run scGCL from the command line (python main.py --dataset adam --epochs 300), letting the ZINB-based autoencoder reconstruct the expression matrix against the prior distribution to estimate dropout values and yield the imputed profile. In scGCL this reconstruction runs alongside graph contrastive learning, which summarizes global and local cell-relationship semantics and selects positive samples for target nodes to enhance node representations. Keep the environment pinned to the published stack (Python 3.8.8, PyTorch 1.10.0, torch-geometric 2.2.0). Export the imputed matrix and verify it has the same cell-by-gene dimensions as the input h5 expression matrix before downstream tasks such as clustering.

## Related tools

- **scGCL** (Implements the ZINB-based autoencoder that reconstructs the scRNA-seq expression matrix against the prior distribution to estimate dropout values; run via main.py from the command line.) — https://github.com/zehaoxiong123/scGCL
- **PyTorch** (Deep-learning backend used to train the ZINB autoencoder (published version 1.10.0).)
- **torch-geometric** (Graph-learning library supporting the graph contrastive component that supplies enhanced node representations alongside the ZINB reconstruction (version 2.2.0).)

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- Dimensional invariant: the imputed expression matrix must have the same cell-by-gene dimensions as the input h5 expression matrix.
- On the AD dataset with published settings (300 epochs), downstream clustering on the imputed data should reproduce ARI ≈ 0.62 and NMI ≈ 0.70 against the true labels in the h5 file.
- AD-versus-control cell separation on the imputed data should reproduce the ordering and separation shown in Supplementary Fig. S3a, and be more efficient than on raw scRNA-seq data.
- The imputed values should be consistent with a Zero-inflated Negative Binomial prior distribution, i.e. reconstruction is performed against the prior distribution rather than a Gaussian assumption.
- Environment check: run with the pinned versions (Python 3.8.8, PyTorch 1.10.0, torch-geometric 2.2.0, torch-cluster 1.6.0, torch-scatter 2.0.9, torch-sparse 0.6.13, faiss-cpu 1.7.3) to avoid dependency drift.

## Limitations

- The ZINB autoencoder is a component of scGCL and assumes the input is zero-inflated scRNA-seq data with cell-relationship (topological) structure available; existing methods that ignore topological structure cannot comprehensively utilize relationships between cells.
- Results depend on the published hyperparameter settings (e.g. 300 epochs on the adam dataset); the task argument supports specific downstream tasks (node, clustering, similarity) rather than arbitrary analyses.

## Evidence

- [abstract] ZINB-based autoencoder reconstructs the expression matrix against the prior distribution to estimate dropout values: "scGCL introduces an autoencoder based on the ZINB distribution, which reconstructs the scRNA-seq data based on the prior distribution"
- [methods] Command-line entry point with dataset and epochs arguments: "You can run the scGCL from the command line: $ python main.py --dataset adam --epochs 300"
- [readme] h5 input format definition: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [readme] Published dependency versions required to run the model: "Python version: 3.8.8
- Pytorch version: 1.10.0
- torch-cluster version: 1.6.0
- torch-geometric version: 2.2.0"
- [abstract] Scope limitation of prior imputation methods that motivates the graph-plus-ZINB design: "existing scRNA-seq data imputation methods ignore the topological structure information of scRNA-seq data and cannot comprehensively utilize the relationships between cells"
