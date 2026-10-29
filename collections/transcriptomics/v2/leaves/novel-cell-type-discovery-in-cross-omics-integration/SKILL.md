---
name: novel-cell-type-discovery-in-cross-omics-integration
description: Use when integrating unpaired annotated scRNA-seq with unlabeled scATAC-seq
  (h5ad, gene activity matrices) and you suspect the target data contains cell types
  missing from the source annotations.
when_to_use_negative:
- Source and target datasets are known to share all cell types — use standard scBridge
  label transfer without --novel_type instead
- Data are paired multi-omics (e.g., CITE-seq/ASAP-seq from the same cells) — scAI,
  MOFA+, or scJoint's paired modes are the appropriate integrators
- Target data are raw scATAC-seq peak matrices not yet converted to a gene activity
  matrix — convert with Signac first
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scBridge
  role: Heterogeneous transfer learning framework; the --novel_type flag enables structure
    preservation loss for novel type discovery
  repo: https://github.com/XLearning-SCU/scBridge
- name: PyTorch 1.12.1
  role: Deep learning framework in which scBridge is implemented; trained on a single
    GPU
- name: Scikit-learn
  role: Provides the Gaussian Mixture Model used to estimate the confidence threshold
    from discriminability and confidence
- name: Signac
  role: Transforms scATAC-seq peak data into the gene activity matrix required as
    scBridge input
  repo: https://github.com/stuart-lab/signac
- name: CUDA 11.4
  role: GPU computing environment (Nvidia RTX 3090, Ubuntu 20.04) for running scBridge
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:ca01f327328990d5cf0853b32d4471c62c5cf6503503da1852b158fa6786515f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - CUDA
  - CUDA 11.4
  - PyTorch
  - PyTorch 1.12.1
  - Python
  - Scikit-learn
  - Signac
  - scBridge
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# novel-cell-type-discovery-in-cross-omics-integration

## Summary

Discovery of scATAC-seq cells from cell types absent in annotated scRNA-seq data using scBridge's structure preservation loss: unreliable target cells receive low confidence scores, are modeled with a two-component Gaussian M Model, and are predicted as a novel type. Validated by recovering scATAC-seq-only Myeloid cells in the human myocardial infarction dataset.

## When to use

Use when integrating unpaired annotated scRNA-seq with unlabeled scATAC-seq (h5ad, gene activity matrices) and you suspect the target data contains cell types missing from the source annotations.

## When NOT to use

- Source and target datasets are known to share all cell types — use standard scBridge label transfer without --novel_type instead
- Data are paired multi-omics (e.g., CITE-seq/ASAP-seq from the same cells) — scAI, MOFA+, or scJoint's paired modes are the appropriate integrators
- Target data are raw scATAC-seq peak matrices not yet converted to a gene activity matrix — convert with Signac first

## Inputs

- Source scRNA-seq AnnData h5ad file with cell type annotation in obs.["CellType"]
- Target scATAC-seq gene activity AnnData h5ad file (peak matrix transformed via Signac), common genes selected with source
- Common gene set shared between source and target (e.g., after subsampling cell types with >10,000 cells for myocardial infarction data)

## Outputs

- Integrated AnnData files with Embedding, X_umap, Prediction, and Reliability fields per scATAC-seq cell
- UMAP plots colored by cell type, domain, and reliability
- Novel-type assignments: low-confidence scATAC-seq cells labeled as novel type (e.g., scATAC-only Myeloid cells)
- GMM-estimated confidence threshold separating reliable vs. unreliable cells

## How to apply

Run scBridge with the `--novel_type` flag, which enables the structure preservation loss so that the most unreliable cells are predicted as a novel type rather than force-assigned to known labels. Keep the default preprocessing (source_preprocess='Standard' for scRNA-seq, target_preprocess='TFIDF' for scATAC-seq activity), reliability_threshold=0.95, prototype_momentum=0.9, batch_size=512, train_epoch=20, learning_rate=5e-4, random_seed=2023, and max_iteration=20. During iterative integration, model scATAC-seq cell reliability with a two-component Gaussian Mixture Model (scikit-learn) based on embedding discriminability and classification confidence, and stop iterating when more than 75% of scATAC-seq cells are selected as annotated data. Rationale: cells with smaller omics differences are integrated first, so cells from novel types remain low-confidence and segregate in the GMM; early stop training when annotated-data classification accuracy reaches 99%.

## Related tools

- **scBridge** (Heterogeneous transfer learning framework; the --novel_type flag enables structure preservation loss for novel type discovery) — https://github.com/XLearning-SCU/scBridge
- **PyTorch 1.12.1** (Deep learning framework in which scBridge is implemented; trained on a single GPU)
- **Scikit-learn** (Provides the Gaussian Mixture Model used to estimate the confidence threshold from discriminability and confidence)
- **Signac** (Transforms scATAC-seq peak data into the gene activity matrix required as scBridge input) — https://github.com/stuart-lab/signac
- **CUDA 11.4** (GPU computing environment (Nvidia RTX 3090, Ubuntu 20.04) for running scBridge)

## Examples

```
python main.py --data_path=<folder> --source_data=<source>.h5ad --target_data=<target>.h5ad --novel_type --umap_plot
```

## Evaluation signals

- NovelF1 computed for recovery of scATAC-seq-only Myeloid cells; scBridge with the structure preservation loss should achieve the highest F1-score among settings
- GMM-based reliability threshold should assign relatively low confidence scores to novel Myeloid scATAC-seq cells and match Fig. 4b-d of the paper
- Annotated-data classification accuracy reaches 99%, triggering early stopping of network optimization
- Stopping rule satisfied: iteration halts when more than 75% of scATAC-seq cells are selected as annotated data
- Saved AnnData contains non-empty Reliability scores and Prediction fields for every target cell; UMAP colored by reliability separates low-confidence novel-type cells

## Limitations

- Novel-type discovery mode relies on a heuristic stopping rule (halt when >75% of scATAC-seq cells are selected as annotated data) rather than a principled criterion
- Detailed guidance on novel type discovery is deferred to Supplementary Note 2, which is not included in the provided document
- Training requires a single GPU (PyTorch 1.12.1, CUDA 11.4); stopping parameters were tuned on the paper's datasets (random_seed=2023, max_iteration=20)

## Evidence

- [results] Equipped with the structure preservation loss, scBridge assigns relatively low confidence scores to novel scATAC-seq Myeloid cells and, using a two-component GMM: [evidence span withheld — anchor status: not_found]
- [methods] novel type discovery with the structure loss: "`--novel_type` enables novel type discovery with the structure loss, where most unreliable cells would be predicted as novel type"
- [discussion] heuristic stopping rule: "we recommend stopping the iteration when more than 75% scATAC-seq cells are selected as the annotated data under such a setting."
- [discussion] reliability modeling: "we model the reliability of scATAC-seq cells with Gaussian Mixture based on their discriminability and confidence."
- [discussion] early stopping: "we early stopped the network optimization when the cell classification accuracy of the annotated data reached 99%"
- [readme] input requirements: "The source data should include the cell type annotation in `obs.["CellType"]`."
