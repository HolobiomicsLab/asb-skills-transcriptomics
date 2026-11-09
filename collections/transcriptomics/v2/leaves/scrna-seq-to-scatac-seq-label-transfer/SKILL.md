---
name: scrna-seq-to-scatac-seq-label-transfer
description: Use when integrating unpaired scRNA-seq (annotated source, h5ad with
  obs['CellType']) and scATAC-seq (unlabeled target) data to transfer cell-type labels,
  e.g. SNARE-seq or mouse-atlas benchmarks.
when_to_use_negative:
- Paired multi-omics data with per-cell pairing information — scAI or MOFA+ perform
  joint integration using cross-omics pairing instead
- Data with expected novel cell types not present in the scRNA-seq annotations — use
  scBridge's --novel_type mode with structure preservation loss rather than plain
  label transfer
- Peak-by-cell scATAC-seq matrices not converted to a gene activity matrix — first
  transform with a tool such as Signac
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scBridge
  role: Core heterogeneous transfer learning method that iteratively selects reliable
    scATAC-seq cells and transfers labels
  repo: https://github.com/XLearning-SCU/scBridge
- name: PyTorch
  role: Framework in which scBridge is implemented (v.1.12.1), run on a single GPU
- name: scikit-learn
  role: Provides the Gaussian Mixture Model used to model scATAC-seq cell reliability
    (v.1.1.1/1.1.3)
- name: Signac
  role: Transforms scATAC-seq peak data into a gene activity matrix before label transfer
  repo: https://github.com/stuart-lab/signac
- name: scJoint
  role: Baseline semi-supervised integration method for comparison (64.60% F1 vs scBridge
    89.66% on mouse atlas)
  repo: https://github.com/SydneyBioX/scJoint
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:9e9279255efc94a8e5a08f65bb0933d2874f1298b1e00fdf8a66ab57d779c07c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - CUDA
  - PyTorch
  - Python
  - Signac
  - scBridge
  - scJoint
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8846153846153846
n_claims: 26
---

# scrna-seq-to-scatac-seq-label-transfer

## Summary

Transfer cell-type annotations from an annotated scRNA-seq source to an unlabeled scATAC-seq target using scBridge's heterogeneous transfer learning, which exploits cell heterogeneity by iteratively selecting reliable scATAC-seq cells via a Gaussian Mixture reliability model.

## When to use

Use when integrating unpaired scRNA-seq (annotated source, h5ad with obs['CellType']) and scATAC-seq (unlabeled target) data to transfer cell-type labels, e.g. SNARE-seq or mouse-atlas benchmarks.

## When NOT to use

- Paired multi-omics data with per-cell pairing information — scAI or MOFA+ perform joint integration using cross-omics pairing instead
- Data with expected novel cell types not present in the scRNA-seq annotations — use scBridge's --novel_type mode with structure preservation loss rather than plain label transfer
- Peak-by-cell scATAC-seq matrices not converted to a gene activity matrix — first transform with a tool such as Signac

## Inputs

- Source scRNA-seq AnnData (.h5ad) with cell-type annotations in obs['CellType']
- Target scATAC-seq gene activity matrix (.h5ad), unlabeled
- Optional target ground-truth annotations for accuracy/F1 evaluation
- Common-gene intersection between modalities (e.g. 12,689 genes for mouse atlas)

## Outputs

- Integrated target AnnData with transferred labels in .obs['Prediction']
- Per-cell reliability scores in .obs['Reliability']
- Joint embeddings in .obsm['Embedding']
- Label transfer accuracy and per-cell-type F1-scores against target annotations

## How to apply

Format the data as source (scRNA-seq) and target (scATAC-seq) h5ad AnnData files, with cell-type annotations in obs['CellType'] for the source, and restrict both modalities to common genes (e.g. 12,689 common genes for the mouse atlas). Preprocess with Standard strategy for the source and TF-IDF strategy for the target, normalizing each cell to 10,000 total counts. Train scBridge via main.py on a single GPU with default hyperparameters (reliability_threshold=0.95, prototype_momentum=0.9, max_iteration=20, batch_size=512, train_epoch=20, learning_rate=5e-4, random_seed=2023), early stopping when annotated-data classification accuracy reaches 0.99. The Gaussian Mixture model iteratively scores scATAC-seq cell reliability from embedding discriminability and classification confidence, and cells above the reliability threshold are merged into the annotated data each iteration. Read the transferred labels (.obs['Prediction']) and reliability scores (.obs['Reliability']) from the integrated target AnnData, then evaluate against provided target annotations (accuracy, per-cell-type weighted F1).

## Related tools

- **scBridge** (Core heterogeneous transfer learning method that iteratively selects reliable scATAC-seq cells and transfers labels) — https://github.com/XLearning-SCU/scBridge
- **PyTorch** (Framework in which scBridge is implemented (v.1.12.1), run on a single GPU)
- **scikit-learn** (Provides the Gaussian Mixture Model used to model scATAC-seq cell reliability (v.1.1.1/1.1.3))
- **Signac** (Transforms scATAC-seq peak data into a gene activity matrix before label transfer) — https://github.com/stuart-lab/signac
- **scJoint** (Baseline semi-supervised integration method for comparison (64.60% F1 vs scBridge 89.66% on mouse atlas)) — https://github.com/SydneyBioX/scJoint

## Examples

```
git clone https://github.com/XLearning-SCU/scBridge && cd scBridge && conda env create -f environment.yml && conda activate scBridge && python main.py
```

## Evaluation signals

- On the SNARE-seq mouse cortex benchmark (GSE126074), accuracy should reach 71.95% overall (60.11% at the first iteration), with weighted F1 of 42.26%
- On the mouse atlas benchmark (~102,103 cells, 18 common types), F1 should reach 89.66%, outperforming scJoint (64.60%) and Seurat (44.64%)
- The transferred labels must appear in .obs['Prediction'] and reliability scores in .obs['Reliability'] of the integrated target AnnData
- Early stopping should trigger when annotated-data classification accuracy reaches 0.99
- Wall-clock time and peak memory over ~102,103 cells should follow the linear-time/constant-memory scaling profile reported in Fig. 3

## Limitations

- Protein-data integration results versus scJoint are deferred to Supplementary Note 1, which is not included in the provided document
- Novel-type discovery mode relies on a heuristic stopping rule (stop when more than 75% of scATAC-seq cells are selected as annotated) with details in Supplementary Note 2, not included here
- False treatment of cell-type difference as omics difference causes over-integration, while false treatment of omics difference as cell-type difference causes under-integration

## Evidence

- [abstract] abstract: "scBridge iterates between i) identifying reliable scATAC-seq cells that have smaller omics differences, and ii) integrating reliable scATAC-seq cells with scRNA-seq"
- [methods] methods: "`--reliability_threshold=[0.0-1.0]` (Default=0.95) threshold for selecting reliable cells"
- [discussion] discussion: "we model the reliability of scATAC-seq cells with Gaussian Mixture based on their discriminability and confidence."
- [readme] readme: "The source data should include the cell type annotation in `obs.["CellType"]`."
