---
name: graph-convolutional-clustering
description: Use when clustering raw scRNA-seq count matrices (CSV or h5 format) into
  cell types, especially when high sparsity/heterogeneity defeats Seurat Louvain/Leiden,
  K-means, Spectral, scTAG, scGNN, scDCC, or other baselines and ARI is the accuracy
  metric.
when_to_use_negative:
- Input is already a processed feature/embedding table — preprocess.py expects raw
  CSV or h5 count matrices
- The analysis goal is CCI inference from existing ground-truth labels and no clustering
  is needed (use Feature.py with label_mode True directly)
- No GPU available and runtime budget cannot accommodate CPU clustering (~25 min with
  pretraining on GPU)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: DeepCCI
  role: GCN-based cluster model; Cluster.py performs cell clustering on the preprocessed
    feature file
  repo: https://github.com/JiangBioLab/DeepCCI
- name: conda
  role: Creates the deepcciEnv Python 3.7.4 virtual environment in which Cluster.py
    runs
- name: Python 3.7.4
  role: Runtime for preprocess.py and Cluster.py
- name: R / Seurat
  role: Downstream processing and visualization of clustering results; Seurat-based
    Louvain/Leiden clustering is a comparison baseline
- name: igraph
  role: R package installed for downstream processing and visualization of clustering/CCI
    results
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad596
    title: 'DeepCCI: a deep learning framework for identifying cell–cell interactions
      from single-cell RNA sequencing data'
  source_content_hash: sha256:9fa1123d23d68d88b97d528813fcef0395c1bf19425f5242b1143e52cb315c2d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiangBioLab/DeepCCI
  tools:
  - DeepCCI
  - Python
  - Python 3.7.4
  - R
  - R / Seurat
  - conda
  - git
  - igraph
  - pip
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# graph-convolutional-clustering

## Summary

Cluster scRNA-seq cells using DeepCCI's GCN-based cluster model, which learns latent cell embeddings to recover cell types from sparse, heterogeneous count data. It replaces purely statistical clustering and is evaluated by ARI against ground-truth labels.

## When to use

Use when clustering raw scRNA-seq count matrices (CSV or h5 format) into cell types, especially when high sparsity/heterogeneity defeats Seurat Louvain/Leiden, K-means, Spectral, scTAG, scGNN, scDCC, or other baselines and ARI is the accuracy metric.

## When NOT to use

- Input is already a processed feature/embedding table — preprocess.py expects raw CSV or h5 count matrices
- The analysis goal is CCI inference from existing ground-truth labels and no clustering is needed (use Feature.py with label_mode True directly)
- No GPU available and runtime budget cannot accommodate CPU clustering (~25 min with pretraining on GPU)

## Inputs

- Raw scRNA-seq count matrix in CSV format (e.g. Yan, GSE36552)
- Raw scRNA-seq count matrix in h5 format (e.g. Quake_10x_Limb_Muscle, GSE109774; PBMC3k)
- Preprocessed DeepCCI feature file produced by Cluster_model/preprocess.py
- Optional pretrained model files from the pretain_model folder

## Outputs

- Cell clustering results (cluster assignments per cell, e.g. nine clusters for PBMC3k)
- Predicted cluster labels suitable as input to Interaction_model/Feature.py (--label_mode True/False)
- ARI scores against ground-truth labels (10 repeated runs per dataset)

## How to apply

Preprocess the raw count matrix with Cluster_model/preprocess.py (e.g. `--name Yan --file_format csv` or `--name Quake_10x_Limb_Muscle --file_format h5`; ~5-10 min per dataset) to produce the processed feature file. Run `Cluster.py --device cuda` either with pre-training (`--pretain True --pretrain_epoch 50`, ~25 min) or reusing the provided pretain_model files (`--pretain False`, ~5 min). Score the resulting clusters against ground-truth cell-type labels with the Adjusted Rand Index, repeating 10 runs per dataset to control seed variance. Correct application is confirmed when ARI matches or exceeds reported results (0.9848 on Quake_10x_Limb_Muscle GSE109774; 0.71 on label-free PBMC3k against 9 benchmarked labels). The learned cluster labels then feed the DeepCCI interaction model (Feature.py/Interaction_inference.py) as CCI source labels.

## Related tools

- **DeepCCI** (GCN-based cluster model; Cluster.py performs cell clustering on the preprocessed feature file) — https://github.com/JiangBioLab/DeepCCI
- **conda** (Creates the deepcciEnv Python 3.7.4 virtual environment in which Cluster.py runs)
- **Python 3.7.4** (Runtime for preprocess.py and Cluster.py)
- **R / Seurat** (Downstream processing and visualization of clustering results; Seurat-based Louvain/Leiden clustering is a comparison baseline)
- **igraph** (R package installed for downstream processing and visualization of clustering/CCI results)

## Examples

```
python Cluster.py --name Yan --pretain True --pretrain_epoch 50 --device cuda
```

## Evaluation signals

- ARI against ground-truth labels must be highest among the 14 compared methods (DeepCCI + 13 baselines) on all 12 benchmark datasets, including ARI 0.9848 on Quake_10x_Limb_Muscle (GSE109774)
- Repeat each clustering 10 times with different seeds and aggregate mean ARI to confirm stability against seed variance
- On label-free PBMC3k, verify the clustering matches the 9 benchmarked labels at ARI 0.71 before using the labels for CCI inference
- Check preprocess.py completes (~5-10 min) and emits the processed feature file without format errors for both csv and h5 inputs
- Confirm Cluster.py runs on GPU (--device cuda) and that using pretain_model files (--pretain False) completes in ~5 minutes

## Limitations

- Pre-training takes about 25 minutes per dataset on GPU (pretrain_epoch 50); without pretain_model files this step cannot be skipped
- The cluster model accepts only CSV and h5 scRNA-seq formats; other formats require conversion first
- GCN training is seed-sensitive, so 10 repeats are needed to control seed variance when reporting ARI
- Paper content lacks a discussion section, so failure modes beyond the benchmarked datasets are not documented

## Evidence

- [intro] GCN-based deep learning framework for scRNA-seq clustering: "DeepCCI is a graph convolutional network (GCN)-based deep learning framework for Cell-Cell Interactions inference from scRNA-seq data."
- [methods] Cluster.py invocation with pretraining: "python Cluster.py --name Yan --pretain True --pretrain_epoch 50 --device cuda"
- [readme] README: runtime and pretrained-model option: "The pretrained model files are in the pretain_model folder. It will take about 5 minutes."
- [readme] README: accepted input formats: "The cluster model of DeepCCI accepts scRNA-seq data format: CSV and h5. The processed feature file of scRNA-seq data will be provided."
- [supplementary] Source card: benchmark validation with ARI: [evidence span withheld — anchor status: not_found]
