---
name: ligand-receptor-pair-graph-construction
description: Use when scRNA-seq data (CSV or h5) with cell-type labels must be converted
  into a ligand-receptor pair graph so a GCN model (e.g. DeepCCI's ResNet+GCN interaction
  model) can classify significant CCIs.
when_to_use_negative:
- When the goal is cell clustering rather than interaction inference — that requires
  the separate Cluster_model pipeline (preprocess.py + Cluster.py), not the interaction-model
  pair graph
- When cells lack reliable type labels and no downstream CCI classifier is needed
  — Feature.py with --label_mode False is a different mode than the labeled supervision
  path this skill describes
- When data are bulk RNA-seq or non-scRNA-seq expression data; the pair-graph construction
  presumes scRNA-seq with sparse, heterogeneous single-cell profiles
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DeepCCI
  role: Framework providing Feature.py for ligand-receptor pair feature preprocessing
    (with SVD reduction) and the GCN-based interaction model that consumes the pair
    graph
  repo: https://github.com/JiangBioLab/DeepCCI
- name: Python 3.7.4
  role: Runtime environment (conda env deepcciEnv) in which Feature.py and Interaction_inference.py
    are executed
- name: conda
  role: Recommended virtual environment manager for installing the DeepCCI Python
    and R dependencies
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad596
    title: 'DeepCCI: a deep learning framework for identifying cell–cell interactions
      from single-cell RNA sequencing data'
  source_content_hash: sha256:bd184e508e2dc784c3734db15fb00081b562aa80709f3a3f3e22aefd2bbcf0b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiangBioLab/DeepCCI
  tools:
  - DeepCCI
  - Python
  - Python 3.7.4
  - conda
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# ligand-receptor-pair-graph-construction

## Summary

Constructs a ligand-receptor pair graph for DeepCCI's GCN-based interaction model by preprocessing scRNA-seq data into processed ligand-receptor pair features (with SVD dimensionality reduction) via Feature.py, aggregating consensus significant-interaction labels by majority vote across CCI methods, and linking similar pairs with similarity threshold tau=0.95.

## When to use

Use when scRNA-seq data (CSV or h5) with cell-type labels must be converted into a ligand-receptor pair graph so a GCN model (e.g. DeepCCI's ResNet+GCN interaction model) can classify significant CCIs.

## When NOT to use

- When the goal is cell clustering rather than interaction inference — that requires the separate Cluster_model pipeline (preprocess.py + Cluster.py), not the interaction-model pair graph
- When cells lack reliable type labels and no downstream CCI classifier is needed — Feature.py with --label_mode False is a different mode than the labeled supervision path this skill describes
- When data are bulk RNA-seq or non-scRNA-seq expression data; the pair-graph construction presumes scRNA-seq with sparse, heterogeneous single-cell profiles

## Inputs

- scRNA-seq count matrix in CSV or h5 format (e.g. Yan GSE36552 CSV, Quake_10x_Limb_Muscle GSE109774 h5)
- Cell-type labels for cells (--label_mode True) when available
- Known ligand-receptor database for the chosen species (Human)
- Per-method CCI prediction results used to build majority-vote consensus significant-interaction labels

## Outputs

- Processed ligand-receptor pair feature file with SVD-dimensionality-reduced pair features
- Majority-vote consensus significant/non-significant labels for every ligand-receptor/cell-type pair
- Ligand-receptor pair graph (edges connecting pairs with similarity above tau=0.95) feeding the ResNet+GCN interaction model

## How to apply

Run DeepCCI's Interaction_model Feature.py (e.g. python Feature.py --label_mode True --species Human) to generate the processed ligand-receptor pair feature file, applying SVD dimensionality reduction to the pair features. Species must be chosen explicitly (Human in the paper's examples) and cell-label mode must match whether ground-truth cell labels exist. Aggregate ground-truth significant-interaction labels across the constituent CCI-prediction methods by majority vote to obtain consensus labels for every ligand-receptor/cell-type pair; these serve as supervision targets. Build the ligand-receptor pair graph with similarity threshold tau=0.95, connecting pair nodes whose features are sufficiently similar, so that a GCN (trained with Focal loss to handle significant/non-significant class imbalance on a cuda device) can propagate information across related pairs. Preprocessing takes about 10-20 minutes depending on scRNA-seq file size.

## Related tools

- **DeepCCI** (Framework providing Feature.py for ligand-receptor pair feature preprocessing (with SVD reduction) and the GCN-based interaction model that consumes the pair graph) — https://github.com/JiangBioLab/DeepCCI
- **Python 3.7.4** (Runtime environment (conda env deepcciEnv) in which Feature.py and Interaction_inference.py are executed)
- **conda** (Recommended virtual environment manager for installing the DeepCCI Python and R dependencies)

## Examples

```
cd Interaction_model && python Feature.py --label_mode True --species Human
```

## Evaluation signals

- Feature.py completes within the documented ~10-20 minute window for the input scRNA-seq file size and emits the processed pair feature file
- Pair features are SVD-dimensionality-reduced and the pair graph is built with similarity threshold tau=0.95
- Consensus significant-interaction labels exist for every ligand-receptor/cell-type pair and are consistent with the majority vote across constituent CCI-prediction methods
- Downstream 5-fold cross-validation (repeated 20 times) reproduces the five metrics (F1, Recall, ACC, Precision, AUC) reported in Fig. 3h-i, and Interaction_inference.py generates the predicted interaction outfile within ~1-2 minutes on --device cuda
- Graph construction runs without GPU error in the deepcciEnv conda environment (Python 3.7.4) and Focal loss handles the significant/non-significant class imbalance

## Limitations

- Statistical CCI methods underlying the consensus labels 'ignore some key latent information contained in scRNA-seq data with high sparsity and heterogeneity', so majority-vote labels inherit that bias
- Preprocessing time scales with scRNA-seq file size (~10-20 minutes) and inference requires a CUDA GPU; the example test file must be downloaded from http://jianglab.org.cn/deepcci_download/
- Species must be supplied explicitly (--species Human for the paper's examples); the labeled mode requires cell-type labels to be present in the input

## Evidence

- [methods] Preprocess input files (Interaction model): "python Feature.py --label_mode True --species Human"
- [intro] GCN-based framework rationale: "DeepCCI is a graph convolutional network (GCN)-based deep learning framework for Cell-Cell Interactions inference"
- [abstract] Sparse data motivation: "current computational methods based on statistical strategies ignore some key latent information contained in scRNA-seq data with high sparsity and heterogeneity"
- [readme] Interaction-model preprocessing setup: "The example test file can be download from  http://jianglab.org.cn/deepcci_download/. The processed feature file will be provided. Depending on the size of the scRNA-seq file，the process will take"
- [readme] Labeled preprocessing mode: "##### With cell-label:

cd Interaction_model
python Feature.py --label_mode True --species Human"
