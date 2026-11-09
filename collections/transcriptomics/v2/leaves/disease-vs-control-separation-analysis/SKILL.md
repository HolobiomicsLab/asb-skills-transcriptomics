---
name: disease-vs-control-separation-analysis
description: Use when an imputed or latent scRNA-seq expression matrix from a disease
  dataset (e.g. the adam h5 file) must be checked for AD-versus-control cell separation
  and clustering quality (ARI/NMI).
when_to_use_negative:
- The goal is gene-level imputation accuracy (e.g. dropout recovery on simulated data)
  rather than disease-vs-control cell separation or clustering.
- The dataset has no true cell labels available to serve as a reference partition
  for ARI/NMI computation.
- The input is raw data only and you are not comparing imputed against raw separation
  — the paper's claim concerns scGCL imputed data separating AD and control cells
  more efficiently than raw scRNA-seq data.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGCL
  role: graph contrastive learning + ZINB autoencoder that produces the imputed expression
    matrix on which AD-vs-control separation and clustering (ARI/NMI) are evaluated;
    also provides the clustering task
  repo: https://github.com/zehaoxiong123/scGCL
- name: Python
  role: command-line runtime for launching scGCL (python main.py) on the adam dataset
- name: PyTorch
  role: deep-learning framework underlying scGCL's ZINB autoencoder and graph contrastive
    learning (version 1.10.0)
- name: torch-geometric
  role: graph neural network library supporting scGCL's graph contrastive learning
    over cell-relationship graphs (version 2.2.0)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:4143ae7bdba1dab91abf1e3bd7422d40582ac782de2a441a8a8228146e6284b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - PyTorch
  - Python
  - scGCL
  - torch-geometric
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# disease-vs-control-separation-analysis

## Summary

Quantify how well a scRNA-seq representation (raw or imputed) separates disease cells from control cells, visualized as in Supplementary Fig. S3a of the scGCL paper, alongside clustering metrics ARI and NMI on the Alzheimer's disease (adam) dataset.

## When to use

Use when an imputed or latent scRNA-seq expression matrix from a disease dataset (e.g. the adam h5 file) must be checked for AD-versus-control cell separation and clustering quality (ARI/NMI).

## When NOT to use

- The goal is gene-level imputation accuracy (e.g. dropout recovery on simulated data) rather than disease-vs-control cell separation or clustering.
- The dataset has no true cell labels available to serve as a reference partition for ARI/NMI computation.
- The input is raw data only and you are not comparing imputed against raw separation — the paper's claim concerns scGCL imputed data separating AD and control cells more efficiently than raw scRNA-seq data.

## Inputs

- adam h5 dataset (scRNA-seq expression matrix, true cell labels, accompanying information)
- scGCL imputed AD expression matrix (e.g. scgcl_ad_imputed_expression.csv)

## Outputs

- ARI and NMI clustering metrics written to scgcl_ad_metrics.csv
- AD-versus-control cell separation quantification and visualization comparable to Supplementary Fig. S3a

## How to apply

First run scGCL imputation on the disease dataset: load the adam h5 file (containing the scRNA-seq expression matrix, true labels, and accompanying information), launch scGCL from the command line (python main.py --dataset adam --epochs 300) so that graph contrastive learning summarizes global and local cell-relationship semantics and selects positive samples for target nodes, and let the ZINB-based autoencoder reconstruct the expression matrix against the prior distribution to estimate dropout values. Export the imputed profile (e.g. scgcl_ad_imputed_expression.csv), confirming it retains the same cell-by-gene dimensions as the input h5 expression matrix. Then run scGCL's clustering task on the imputed data using the true labels in the h5 file as the reference partition, compute ARI and NMI (expected on the AD dataset: ARI 0.62, NMI 0.70), and quantify and visualize AD-versus-control cell separation as shown in Supplementary Fig. S3a, writing the metrics to scgcl_ad_metrics.csv. The rationale is that scGCL's enhancement of expression patterns of specific genes in the Alzheimer's disease dataset should separate AD and control cells more efficiently than the raw scRNA-seq data.

## Related tools

- **scGCL** (graph contrastive learning + ZINB autoencoder that produces the imputed expression matrix on which AD-vs-control separation and clustering (ARI/NMI) are evaluated; also provides the clustering task) — https://github.com/zehaoxiong123/scGCL
- **Python** (command-line runtime for launching scGCL (python main.py) on the adam dataset)
- **PyTorch** (deep-learning framework underlying scGCL's ZINB autoencoder and graph contrastive learning (version 1.10.0))
- **torch-geometric** (graph neural network library supporting scGCL's graph contrastive learning over cell-relationship graphs (version 2.2.0))

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- Computed ARI and NMI on the AD dataset should reproduce the published ordering (ARI 0.62, NMI 0.70).
- AD-versus-control cell separation must reproduce the ordering and separation visualized in Supplementary Fig. S3a.
- The imputed matrix must have the same cell-by-gene dimensions as the input h5 expression matrix.
- Imputed data should separate AD and control cells more efficiently than the raw scRNA-seq data.
- Metrics file scgcl_ad_metrics.csv is produced with the ARI/NMI values from the clustering task using the true labels as reference partition.

## Limitations

- Evaluation depends on true labels being present in the h5 file to act as the reference partition for ARI/NMI.
- Performance values (ARI 0.62, NMI 0.70) are tied to the adam dataset with the published settings (--epochs 300); other datasets or hyperparameters are not covered by this result.
- Separation claim assumes scGCL's ZINB-based imputation enhances expression patterns of specific genes in Alzheimer's disease datasets; results may not transfer to other disease contexts.

## Evidence

- [results] scGCL achieves clustering indicators ARI of 0.62 and NMI of 0.70 on the AD dataset, and separates AD and control cells more efficiently than the raw scRNA-seq data.: [evidence span withheld — anchor status: not_found]
- [readme] CLI invocation from README: "$ python main.py --dataset adam --epochs 300"
- [readme] h5 file description from README arguments table: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [abstract] ZINB autoencoder rationale: "scGCL introduces an autoencoder based on the ZINB distribution, which reconstructs the scRNA-seq data based on the prior distribution"
- [intro] contrastive learning step: "scGCL summarizes global and local semantic information through contrastive learning and selects appropriate positive samples to enhance the representation of target nodes"
