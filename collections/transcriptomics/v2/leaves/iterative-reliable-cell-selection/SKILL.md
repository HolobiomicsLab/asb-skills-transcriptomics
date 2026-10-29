---
name: iterative-reliable-cell-selection
description: Use when integrating unpaired annotated scRNA-seq (source) with unlabeled
  scATAC-seq (target) h5ad data and cell heterogeneity means some target cells are
  easy to bridge while others differ strongly across omics.
when_to_use_negative:
- Data are paired multi-omics (same cells profiled in both omics) — scAI/MOFA+ style
  joint integration with pairing information applies instead
- The goal is novel cell-type discovery without the structure preservation loss enabled;
  the reliability loop as described assumes all target cells belong to known source
  types
- The target is not scATAC-seq gene-activity data derived from peaks (peak-by-cell
  matrices must first be transformed into an activity matrix, e.g., with Signac)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scBridge
  role: Implements the iterative reliable-cell-selection integration loop (warm-up,
    GMM reliability, prototype alignment, merge-and-repeat)
  repo: https://github.com/XLearning-SCU/scBridge
- name: PyTorch
  role: Framework for the encoder and classifier warm-up and reliability-weighted
    prototype alignment training
- name: Scikit-learn
  role: Provides the Gaussian Mixture Model used to estimate per-cell reliability
    from discriminability and confidence
- name: Signac
  role: Transforms scATAC-seq peak data into the gene activity matrix required as
    target input
  repo: https://github.com/stuart-lab/signac
- name: scJoint
  role: Homogeneous baseline against which the heterogeneous iterative selection is
    compared
  repo: https://github.com/SydneyBioX/scJoint
- name: CUDA
  role: GPU runtime used to execute scBridge on a single Nvidia RTX 3090 with CUDA
    11.4
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:41a092af210119a410611c94a0247225c057f249bf2e05f23e2af05671b0d132
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - CUDA
  - PyTorch
  - Python
  - Scikit-learn
  - Signac
  - scBridge
  - scJoint
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# iterative-reliable-cell-selection

## Summary

The scBridge loop that progressively identifies reliable scATAC-seq cells (via GMM-modeled reliability over embedding discriminability and classification confidence) and merges them into the annotated scRNA-seq set, integrating cells from-easy-to-hard to narrow the omics gap without treating all cells homogeneously.

## When to use

Use when integrating unpaired annotated scRNA-seq (source) with unlabeled scATAC-seq (target) h5ad data and cell heterogeneity means some target cells are easy to bridge while others differ strongly across omics.

## When NOT to use

- Data are paired multi-omics (same cells profiled in both omics) — scAI/MOFA+ style joint integration with pairing information applies instead
- The goal is novel cell-type discovery without the structure preservation loss enabled; the reliability loop as described assumes all target cells belong to known source types
- The target is not scATAC-seq gene-activity data derived from peaks (peak-by-cell matrices must first be transformed into an activity matrix, e.g., with Signac)

## Inputs

- Annotated source scRNA-seq AnnData (.h5ad) with obs['CellType'] (e.g., CITE-seq PBMC from GSE156478)
- Unlabeled target scATAC-seq AnnData (.h5ad) converted to a gene activity matrix (e.g., ASAP-seq PBMC)
- Common gene set shared between source count and target activity matrices
- Preprocessed target peak data (TF-IDF default) and source data (Standard default)

## Outputs

- Integrated source and target AnnData files with Embedding/X_umap, Prediction, and Reliability fields
- UMAP plots colored by cell type, domain, and reliability
- Label-transfer predictions (cell types) for scATAC-seq cells

## How to apply

First warm up the PyTorch encoder and classifier on annotated scRNA-seq cells with a weighted cross-entropy loss (learning_rate=5e-4, batch_size=512, train_epoch=20, random_seed=2023, early stop at 99% annotated-data classification accuracy). At each iteration (max_iteration=20), fit a scikit-learn Gaussian Mixture Model over scATAC-seq embedding discriminability and classification confidence to estimate per-cell reliability, then perform reliability-weighted cross-omics prototype alignment (align_loss_epoch=1, prototype_momentum=0.9). Select scATAC-seq cells whose reliability exceeds reliability_threshold=0.95, merge them into the annotated data, and repeat steps until no new cells are selected. The rationale is that cells with smaller omics differences are integrated first, progressively teaching the model to bridge harder cells, unlike homogeneous methods such as scJoint.

## Related tools

- **scBridge** (Implements the iterative reliable-cell-selection integration loop (warm-up, GMM reliability, prototype alignment, merge-and-repeat)) — https://github.com/XLearning-SCU/scBridge
- **PyTorch** (Framework for the encoder and classifier warm-up and reliability-weighted prototype alignment training)
- **Scikit-learn** (Provides the Gaussian Mixture Model used to estimate per-cell reliability from discriminability and confidence)
- **Signac** (Transforms scATAC-seq peak data into the gene activity matrix required as target input) — https://github.com/stuart-lab/signac
- **scJoint** (Homogeneous baseline against which the heterogeneous iterative selection is compared) — https://github.com/SydneyBioX/scJoint
- **CUDA** (GPU runtime used to execute scBridge on a single Nvidia RTX 3090 with CUDA 11.4)

## Examples

```
git clone https://github.com/XLearning-SCU/scBridge && python main.py --reliability_threshold=0.95 --align_loss_epoch=1
```

## Evaluation signals

- Per iteration, the set of newly selected scATAC-seq cells should be non-empty until convergence; the loop terminates when no cell exceeds reliability_threshold=0.95
- Warm-up phase must reach 99% classification accuracy on the annotated scRNA-seq data before the iterative loop begins
- Reliability scores are in [0.0-1.0]; histograms per iteration should show progressively more cells above the 0.95 threshold as reliable cells are merged
- Integrated outputs must contain Embedding/X_umap, Prediction, and Reliability fields and UMAP plots colored by cell type, domain, and reliability
- Label-transfer accuracy and joint embedding quality should exceed homogeneous baselines such as scJoint on PBMC CITE-seq/ASAP-seq data

## Limitations

- Over-integration can occur when cell-type difference is falsely treated as omics difference, and under-integration when omics difference is treated as cell-type difference
- Novel-type discovery mode relies on a heuristic stopping rule (stop when more than 75% of scATAC-seq cells are selected as annotated data)
- The reliability loop assumes 7 common cell types in the PBMC benchmark; 'Unknown'-annotated cells must be excluded beforehand
- Requires a GPU (single GPU, e.g., RTX 3090 with CUDA 11.4) and comparison details versus scJoint for protein data are only in Supplementary Note 1

## Evidence

- [abstract] Iterative reliable-cell selection loop: "scBridge iterates between i) identifying reliable scATAC-seq cells that have smaller omics differences, and ii) integrating reliable scATAC-seq cells with scRNA-seq data"
- [discussion] GMM reliability modeling: "we model the reliability of scATAC-seq cells with Gaussian Mixture based on their discriminability and confidence."
- [discussion] Merge and repeat: "scBridge selects the most reliable scATAC-seq cells as the annotated data and repeats the entire process."
- [methods] Reliability threshold parameter: "`--reliability_threshold=[0.0-1.0]` (Default=0.95) threshold for selecting reliable cells"
- [readme] Input requirements: "For scATAC-seq peak data, it needs to be first transformed into a scATAC-seq activity matrix using tools such as Signac"
