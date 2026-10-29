---
name: cross-omics-prototype-alignment
description: Use when integrating unpaired annotated scRNA-seq with unlabeled scATAC-seq
  data (e.g., CITE-seq source + ASAP-seq target .h5ad) and you need to progressively
  align modalities without treating all cells homogeneously.
when_to_use_negative:
- Data is paired multi-omics where cross-omics pairing information is available (e.g.,
  scAI or MOFA+ perform joint integration and clustering on such data)
- Peak-by-cell scATAC-seq data has not yet been transformed into a gene activity matrix
  — the alignment operates on gene-space representations
- The integration goal is homogeneous treatment of all cells at once (e.g., scJoint-style
  semi-supervised training) rather than progressive, heterogeneity-aware integration
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: scBridge
  role: Implements the reliability-weighted cross-omics prototype alignment within
    its heterogeneous transfer learning integration loop
  repo: https://github.com/XLearning-SCU/scBridge
- name: PyTorch 1.12.1
  role: Framework hosting the encoder f and classifier g that produce the embeddings
    whose prototypes are aligned
- name: Scikit-learn 1.1.1
  role: Provides the Gaussian Mixture Model used to estimate per-cell reliability
    that weights the prototype alignment
- name: Signac
  role: Prepares the scATAC-seq gene activity matrix required as target input before
    alignment
  repo: https://github.com/stuart-lab/signac
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:fcffb2d06ad26007dd5e4da65d9a7d1e2e6ca1b41729925e0f6e1742c8222735
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - CUDA
  - PyTorch
  - PyTorch 1.12.1
  - Python
  - Scikit-learn 1.1.1
  - Signac
  - scBridge
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cross-omics-prototype-alignment

## Summary

A reliability-weighted prototype alignment step from scBridge's heterogeneous transfer learning loop that narrows the omics gap between annotated scRNA-seq and unlabeled scATAC-seq embeddings by aligning cross-omics cell-type prototypes, prioritizing scATAC-seq cells with smaller omics differences.

## When to use

Use when integrating unpaired annotated scRNA-seq with unlabeled scATAC-seq data (e.g., CITE-seq source + ASAP-seq target .h5ad) and you need to progressively align modalities without treating all cells homogeneously.

## When NOT to use

- Data is paired multi-omics where cross-omics pairing information is available (e.g., scAI or MOFA+ perform joint integration and clustering on such data)
- Peak-by-cell scATAC-seq data has not yet been transformed into a gene activity matrix — the alignment operates on gene-space representations
- The integration goal is homogeneous treatment of all cells at once (e.g., scJoint-style semi-supervised training) rather than progressive, heterogeneity-aware integration

## Inputs

- Annotated source scRNA-seq (CITE-seq) AnnData .h5ad with cell type annotation in obs["CellType"]
- Unlabeled target scATAC-seq (ASAP-seq) AnnData .h5ad with a gene activity matrix (peak data pre-transformed via Signac)
- Source and target count/activity matrices restricted to common genes

## Outputs

- Integrated source and target AnnData .h5ad files with Embedding/X_umap and Prediction/Reliability fields
- UMAP plots colored by cell type, domain, and reliability
- Predicted labels transferred from scRNA-seq to scATAC-seq cells

## How to apply

First warm up the PyTorch encoder f and classifier g on annotated scRNA-seq data with a weighted cross-entropy loss (learning_rate=5e-4, batch_size=512, train_epoch=20, early stop at 99% annotated-data accuracy). At each of up to max_iteration=20 iterations, fit a scikit-learn Gaussian Mixture Model over scATAC-seq embedding discriminability and classification confidence to estimate per-cell reliability, then perform reliability-weighted cross-omics prototype alignment between source and target prototypes (align_loss_epoch=1, prototype_momentum=0.9). Select scATAC-seq cells whose reliability exceeds reliability_threshold=0.95, merge them into the annotated data, and repeat until no new cells are selected — the rationale being that cells with smaller omics differences are easier to integrate and bridge the modality gap first, in a from-easy-to-hard fashion.

## Related tools

- **scBridge** (Implements the reliability-weighted cross-omics prototype alignment within its heterogeneous transfer learning integration loop) — https://github.com/XLearning-SCU/scBridge
- **PyTorch 1.12.1** (Framework hosting the encoder f and classifier g that produce the embeddings whose prototypes are aligned)
- **Scikit-learn 1.1.1** (Provides the Gaussian Mixture Model used to estimate per-cell reliability that weights the prototype alignment)
- **Signac** (Prepares the scATAC-seq gene activity matrix required as target input before alignment) — https://github.com/stuart-lab/signac

## Examples

```
git clone https://github.com/XLearning-SCU/scBridge && python main.py --reliability_threshold=0.95 --align_loss_epoch=1 --source_preprocess=Standard --target_preprocess=TFIDF
```

## Evaluation signals

- Annotated-data classification accuracy reaches the 99% early-stop threshold during warm-up before alignment begins
- At each iteration, scATAC-seq cells with reliability > 0.95 are selected and merged; the loop terminates when no new cells exceed the threshold (or at max_iteration=20)
- Integrated outputs contain non-empty Embedding/X_umap and Prediction/Reliability fields in both saved AnnData files
- Joint embedding quality and label transfer accuracy on the target scATAC-seq cells should improve relative to homogeneous baselines such as scJoint
- UMAP overlays by cell type, domain, and reliability should show progressively fewer misaligned scATAC-seq cells across iterations

## Limitations

- Novel-type discovery mode relies on a heuristic stopping rule — iteration stops when more than 75% of scATAC-seq cells have been selected as annotated data
- Over-integration (cell-type difference falsely treated as omics difference) or under-integration (omics difference falsely treated as cell-type difference) can occur if reliability estimation is poorly calibrated
- Detailed guidance and overall algorithm summaries live in Supplementary Notes 2–4, which were not available in the provided document

## Evidence

- [discussion] Reliability-weighted alignment drives the loop: "conducts cross-omics prototype alignment based on the estimated cell reliability"
- [discussion] Reliability estimation via GMM: "we model the reliability of scATAC-seq cells with Gaussian Mixture based on their discriminability and confidence."
- [abstract] Easy-to-hard progressive integration: "scBridge iterates between i) identifying reliable scATAC-seq cells that have smaller omics differences"
- [methods] Reliability threshold parameterization: "`--reliability_threshold=[0.0-1.0]` (Default=0.95) threshold for selecting reliable cells"
- [readme] Target data must be a gene activity matrix: "For scATAC-seq peak data, it needs to be first transformed into a scATAC-seq activity matrix using tools such as Signac"
