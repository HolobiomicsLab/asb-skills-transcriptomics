---
name: single-cell-multiomics-integration
description: Use when integrating unpaired annotated scRNA-seq (source, h5ad) with
  unlabeled scATAC-seq gene activity data (target, h5ad) for label transfer, atlas-scale
  joint embedding, or discovering scATAC-seq cell types absent from scRNA-seq annotations.
when_to_use_negative:
- Paired multi-omics data with cross-omics pairing information, where joint integration
  methods such as scAI or MOFA+ apply instead
- Peak-by-cell scATAC-seq matrices that have not yet been transformed into a gene
  activity matrix — run Signac or a similar tool first
- Omics combinations outside scRNA-seq/scATAC-seq label transfer scope (e.g. methylation
  trajectories) unless the semi-supervised classification mode of a baseline like
  scJoint is more appropriate
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scBridge
  role: Primary integration method; iteratively identifies reliable scATAC-seq cells
    and merges them into annotated scRNA-seq data
  repo: https://github.com/XLearning-SCU/scBridge
- name: PyTorch 1.12.1
  role: Framework implementing the scBridge encoder, classifier, and weighted cross-entropy
    training
- name: Scikit-learn
  role: Provides the Gaussian Mixture Model used to estimate per-cell reliability
    from embedding discriminability and classification confidence
- name: Signac
  role: R tool to transform scATAC-seq peak data into a scATAC-seq activity matrix
    suitable as scBridge target input
  repo: https://github.com/stuart-lab/signac
- name: harmonypy
  role: Harmonizes joint embeddings across omics when required for silhouette-score
    evaluation
  repo: https://github.com/slowkow/harmonypy
- name: scJoint
  role: Baseline semi-supervised integration method used for benchmarking label transfer
    (e.g. 64.60% F1 vs scBridge 89.66% on the mouse atlas)
  repo: https://github.com/SydneyBioX/scJoint
- name: GLUE
  role: Baseline integration method (v0.3.2) adopted for comparison
  repo: https://github.com/gao-lab/GLUE
merged_aliases:
- single-cell-multiomics-data-integration
merged_alias_records:
- alias: single-cell-multiomics-data-integration
  slug: single-cell-multiomics-data-integration
  jaccard_score: 0.8
  method: token-set-jaccard
  decision: needs_review
- alias: single-cell-multi-omics-integration
  slug: single-cell-multi-omics-integration
  cosine_score: 0.9863
  method: embedding-cosine
  decision: auto
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:da5ca48526dcd2af3f0337a6485fb19b725992fa2101d1e68ad702ccca276715
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - CUDA
  - GLUE
  - PyTorch
  - PyTorch 1.12.1
  - Python
  - Scikit-learn
  - Signac
  - harmonypy
  - scBridge
  - scJoint
  - scanpy
  - scikit-learn
  - uniPort
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# single-cell-multiomics-integration

## Summary

Integrates annotated scRNA-seq with unlabeled scATAC-seq data using scBridge, a heterogeneous transfer learning method that iteratively identifies reliable scATAC-seq cells with smaller omics differences and merges them into the annotated data to enable label transfer, joint embedding, and novel cell-type discovery.

## When to use

Use when integrating unpaired annotated scRNA-seq (source, h5ad) with unlabeled scATAC-seq gene activity data (target, h5ad) for label transfer, atlas-scale joint embedding, or discovering scATAC-seq cell types absent from scRNA-seq annotations.

## When NOT to use

- Paired multi-omics data with cross-omics pairing information, where joint integration methods such as scAI or MOFA+ apply instead
- Peak-by-cell scATAC-seq matrices that have not yet been transformed into a gene activity matrix — run Signac or a similar tool first
- Omics combinations outside scRNA-seq/scATAC-seq label transfer scope (e.g. methylation trajectories) unless the semi-supervised classification mode of a baseline like scJoint is more appropriate

## Inputs

- Source annotated scRNA-seq count matrix as h5ad AnnData with obs['CellType']
- Target unlabeled scATAC-seq gene activity matrix as h5ad AnnData (peak data first transformed to an activity matrix, e.g. with Signac)
- Optional target cell-type annotations for accuracy/F1 evaluation
- GPU (e.g. Nvidia RTX 3090, CUDA 11.4)

## Outputs

- Integrated target AnnData with transferred labels in obs['Prediction'] and reliability scores in obs['Reliability']
- Joint embeddings in obsm['Embedding'] (with X_umap for visualization)
- Integrated source AnnData and UMAP plots colored by cell type, domain, and reliability
- Label transfer accuracy / per-cell-type F1 scores for the target cells

## How to apply

Restrict source and target AnnData h5ad files to common genes, preprocess the scRNA-seq source with the 'Standard' strategy and the scATAC-seq target with 'TFIDF', and store cell-type annotations in obs['CellType']. Warm up the PyTorch encoder and classifier on annotated cells with weighted cross-entropy (learning_rate=5e-4, batch_size=512, train_epoch=20, random_seed=2023, early stop at 99% annotated-data classification accuracy). Then run up to max_iteration=20 rounds in which a scikit-learn Gaussian Mixture Model over embedding discriminability and classification confidence estimates per-cell reliability, reliability-weighted cross-omics prototype alignment (align_loss_epoch=1, prototype_momentum=0.9) narrows the omics gap, and cells with reliability above reliability_threshold=0.95 are merged into the annotated data. Enable the structure preservation loss (--novel_type) when scATAC-seq cells of novel types are expected, and stop early when more than 75% of scATAC-seq cells are selected as annotated data. The rationale is that cell heterogeneity can be exploited rather than eliminated: cells with smaller omics differences bridge the modality gap, whereas treating all cells homogeneously risks over- or under-integration.

## Related tools

- **scBridge** (Primary integration method; iteratively identifies reliable scATAC-seq cells and merges them into annotated scRNA-seq data) — https://github.com/XLearning-SCU/scBridge
- **PyTorch 1.12.1** (Framework implementing the scBridge encoder, classifier, and weighted cross-entropy training)
- **Scikit-learn** (Provides the Gaussian Mixture Model used to estimate per-cell reliability from embedding discriminability and classification confidence)
- **Signac** (R tool to transform scATAC-seq peak data into a scATAC-seq activity matrix suitable as scBridge target input) — https://github.com/stuart-lab/signac
- **harmonypy** (Harmonizes joint embeddings across omics when required for silhouette-score evaluation) — https://github.com/slowkow/harmonypy
- **scJoint** (Baseline semi-supervised integration method used for benchmarking label transfer (e.g. 64.60% F1 vs scBridge 89.66% on the mouse atlas)) — https://github.com/SydneyBioX/scJoint
- **GLUE** (Baseline integration method (v0.3.2) adopted for comparison) — https://github.com/gao-lab/GLUE

## Examples

```
python main.py --data_path=<folder> --source_data=<source>.h5ad --target_data=<target>.h5ad --novel_type --umap_plot
```

## Evaluation signals

- On the SNARE-seq mouse cortex benchmark (GSE126074), accuracy grows from 60.11% at the first iteration to a final 71.95% (42.26% weighted F1)
- On the mouse atlas (Tabula Muris + Cusanovich scATAC-seq, 18 common types, 12,689 common genes, ~102,103 cells), label transfer F1 reaches 89.66%, vs scJoint 64.60% and Seurat 44.64%
- Obs['Prediction'] and obs['Reliability'] fields are populated in the integrated target AnnData, and cell-type/omics silhouette scores of the joint embedding improve after harmonization
- In novel-type mode, scATAC-seq-only Myeloid cells in the myocardial infarction data receive low confidence and NovelF1 recovery matches the GMM threshold behavior in Fig. 4b-d
- Wall-clock time and peak memory scale linearly/constantly over ~102,103 cells as reported in Fig. 3; deviations flag the run as a reproduction failure

## Limitations

- Novel-type discovery mode relies on a heuristic stopping rule — halt when more than 75% of scATAC-seq cells are selected as annotated data
- Key protocol details (protein-data comparison vs scJoint, novel-type guidance, TF-IDF discussion, overall algorithm) are only in Supplementary Notes 1–4
- Falsely treating cell-type difference as omics difference causes over-integration, and vice versa; cell heterogeneity makes omics and cell-type differences less significant

## Evidence

- [abstract] iterative reliable cell selection: "scBridge iterates between i) identifying reliable scATAC-seq cells that have smaller omics differences, and ii) integrating reliable scATAC-seq cells with scRNA-seq data to narrow the omics gap"
- [discussion] reliability modeling: "we model the reliability of scATAC-seq cells with Gaussian Mixture based on their discriminability and confidence."
- [methods] preprocessing defaults: "`--source_preprocess=["Standard", "TFIDF"]` (Default="Standard") the preprocessing strategy for source data"
- [readme] input format: "scBridge accepts AnnData (https://anndata.readthedocs.io/en/latest/index.html) as inputs, including the source (scRNA-seq) and target (scATAC-seq) h5ad files. The source data should include the cell"
- [discussion] environment: "All experiments are conducted on an Nvidia RTX 3090 GPU with CUDA 11.4, on the Ubuntu 20.04 OS."
