---
name: gmm-based-cell-reliability-modeling
description: Use when integrating unpaired annotated scRNA-seq (source) with unlabeled
  scATAC-seq gene-activity (target) h5ad data and you must decide per-cell which scATAC-seq
  cells are trustworthy for label transfer or are candidate novel cell types.
when_to_use_negative:
- Data are paired across omics (cross-omics pairing information available), where
  scAI or MOFA+ joint integration is the appropriate choice
- The scATAC-seq peak data have not been converted into a gene activity matrix with
  common genes with the scRNA-seq data, so reliability modeling on incomparable features
  would be meaningless
- The goal is a single-shot label transfer of all scATAC-seq cells to known types
  with no need to separate reliable from unreliable cells or detect novel types
tools:
- name: scBridge
  role: Iterative heterogeneous transfer-learning framework whose reliability estimation,
    prototype alignment, and structure preservation loss implement this GMM-based
    filtering step
  repo: https://github.com/XLearning-SCU/scBridge
- name: Scikit-learn
  role: Provides the Gaussian Mixture Model used to estimate the per-cell reliability
    threshold (v1.1.1 in the paper)
- name: PyTorch
  role: Framework in which the scBridge encoder, classifier, and alignment/structure-preservation
    losses are trained (v1.12.1)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:f95d1e4fa5dbfaab5a9536caeec53a64864a6cb9ae44b4b418dc768e0a93bc89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - CUDA
  - PyTorch
  - Python
  - Scikit-learn
  - scBridge
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# gmm-based-cell-reliability-modeling

## Summary

Model the reliability of scATAC-seq cells during unpaired scRNA-seq/scATAC-seq integration with a two-component Gaussian Mixture Model over embedding discriminability and classification confidence, so that reliable cells are merged into the annotated scRNA-seq data while unreliable cells can be flagged as novel types.

## When to use

Use when integrating unpaired annotated scRNA-seq (source) with unlabeled scATAC-seq gene-activity (target) h5ad data and you must decide per-cell which scATAC-seq cells are trustworthy for label transfer or are candidate novel cell types.

## When NOT to use

- Data are paired across omics (cross-omics pairing information available), where scAI or MOFA+ joint integration is the appropriate choice
- The scATAC-seq peak data have not been converted into a gene activity matrix with common genes with the scRNA-seq data, so reliability modeling on incomparable features would be meaningless
- The goal is a single-shot label transfer of all scATAC-seq cells to known types with no need to separate reliable from unreliable cells or detect novel types

## Inputs

- Source scRNA-seq AnnData (.h5ad) with cell type annotations in obs.["CellType"], standard preprocessing
- Target scATAC-seq gene-activity AnnData (.h5ad) (peak matrix transformed via Signac), common genes selected, TFIDF preprocessing
- Per-cell classification confidence scores and embedding discriminability values computed by scBridge

## Outputs

- Per-cell reliability scores and a GMM-estimated reliability threshold
- Integrated AnnData with Embedding, X_umap, Prediction, and Reliability fields
- UMAP plots colored by cell type, domain, and reliability; novel-type (e.g. Myeloid) predictions for unreliable scATAC-seq cells

## How to apply

During scBridge's iterative integration, after warming up the encoder and classifier on annotated scRNA-seq data (weighted cross-entropy, early stop at 99% annotated accuracy), compute per-cell confidence scores and embedding discriminability for scATAC-seq cells, then fit the Gaussian Mixture Model from scikit-learn (v1.1.1 in the paper) to these two signals to obtain a reliability threshold (reliability_threshold defaults to 0.95 in the CLI). Select cells above the threshold as newly annotated data and repeat prototype-alignment training; under the novel-type setting (structure preservation loss enabled, --novel_type), the most unreliable cells are predicted as a novel type, and iteration should be halted when more than 75% of scATAC-seq cells are selected as annotated. Rationale: scATAC-seq cells with smaller omics differences (higher correlation with scRNA-seq expression) are integrated first, letting cell heterogeneity narrow the modality gap instead of being treated as noise. In the myocardial infarction case, this assigned low confidence to scATAC-seq-only Myeloid cells, yielding the highest novel-type discovery F1 (NovelF1) versus baselines.

## Related tools

- **scBridge** (Iterative heterogeneous transfer-learning framework whose reliability estimation, prototype alignment, and structure preservation loss implement this GMM-based filtering step) — https://github.com/XLearning-SCU/scBridge
- **Scikit-learn** (Provides the Gaussian Mixture Model used to estimate the per-cell reliability threshold (v1.1.1 in the paper))
- **PyTorch** (Framework in which the scBridge encoder, classifier, and alignment/structure-preservation losses are trained (v1.12.1))

## Examples

```
python main.py --data_path=<folder> --source_data=<source>.h5ad --target_data=<target>.h5ad --novel_type --umap_plot
```

## Evaluation signals

- The GMM-estimated reliability threshold should match the reliability landscapes shown in Fig. 4b-d for the myocardial infarction dataset
- Under --novel_type, unreliable scATAC-seq cells should be predicted as a novel type and NovelF1 for recovery of scATAC-seq-only Myeloid cells should be the highest among compared methods
- Iteration is stopped once more than 75% of scATAC-seq cells are selected as annotated data, per the recommended stopping rule
- Annotated-data classification accuracy reaches 99% before early stopping of network optimization
- UMAP plots colored by reliability should show relatively low confidence scores concentrated on the novel Myeloid cells rather than uniformly across types

## Limitations

- Novel-type discovery mode relies on a heuristic stopping rule: halt when more than 75% of scATAC-seq cells are selected as annotated data, which may not generalize to all datasets
- More detailed guidance on novel type discovery is deferred to Supplementary Note 2, which is not included in the main document
- scBridge runs on a single GPU and cell types with more than 10,000 cells must be subsampled (myocardial infarction dataset) before analysis

## Evidence

- [discussion] GMM reliability modeling: "we model the reliability of scATAC-seq cells with Gaussian Mixture based on their discriminability and confidence."
- [methods] GMM implementation: "`--reliability_threshold=[0.0-1.0]` (Default=0.95) threshold for selecting reliable cells"
- [abstract] iterative reliable-cell selection rationale: "identifying reliable scATAC-seq cells that have smaller omics differences"
- [discussion] novel-type stopping rule: "we recommend stopping the iteration when more than 75% scATAC-seq cells are selected as the annotated data under such a setting."
- [readme] input annotation requirement: "The source data should include the cell type annotation in `obs.["CellType"]`."
