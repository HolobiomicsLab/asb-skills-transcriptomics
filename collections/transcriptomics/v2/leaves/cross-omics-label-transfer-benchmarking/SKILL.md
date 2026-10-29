---
name: cross-omics-label-transfer-benchmarking
description: Use when integrating unpaired scRNA-seq and scATAC-seq (or CITE-seq/ASAP-seq)
  h5ad data and you need to benchmark label-transfer F1/accuracy under reduced annotations,
  noisy labels, or novel cell-type discovery.
when_to_use_negative:
- The data are paired multi-omics with cross-omics pairing information, where methods
  like scAI or MOFA+ perform joint integration and clustering instead
- The target modality is protein abundance rather than scATAC-seq gene activity and
  no ATAC preprocessing is available to build a gene activity matrix
- You only need a quick joint visualization without annotations on either modality;
  scBridge requires annotated scRNA-seq source data
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scBridge
  role: Primary label-transfer method; iteratively selects reliable scATAC-seq cells
    and merges them with annotated scRNA-seq data, supports --novel_type mode with
    structure preservation loss
  repo: https://github.com/XLearning-SCU/scBridge
- name: scJoint
  role: Semi-supervised baseline for label-transfer benchmarking under downsampling
    and label corruption
  repo: https://github.com/SydneyBioX/scJoint
- name: GLUE
  role: Baseline method (v0.3.2) run on corrupted-label PBMC benchmark data
  repo: https://github.com/gao-lab/GLUE
- name: Signac
  role: R tool to transform scATAC-seq peaks into a gene activity matrix before scBridge
    input
  repo: https://github.com/stuart-lab/signac
- name: scikit-learn
  role: Gaussian Mixture Model for target-cell reliability estimation and F1/accuracy
    metric computation
- name: harmonypy
  role: Post-hoc embedding harmonization (v0.0.5) used in the integration pipeline
  repo: https://github.com/slowkow/harmonypy
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:a3687aac0b983d482db95f669c9d5ab29398adc7e32640ff62eeb717fb644f2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - CUDA
  - GLUE
  - PyTorch
  - Python
  - Signac
  - harmonypy
  - scBridge
  - scJoint
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9583333333333334
n_claims: 24
---

# cross-omics-label-transfer-benchmarking

## Summary

Benchmark label transfer from annotated scRNA-seq to unlabeled scATAC-seq data using scBridge against baselines (scJoint, GLUE, Seurat), including annotation-downsampling and label-corruption robustness tests and novel-type discovery via a GMM-based reliability model.

## When to use

Use when integrating unpaired scRNA-seq and scATAC-seq (or CITE-seq/ASAP-seq) h5ad data and you need to benchmark label-transfer F1/accuracy under reduced annotations, noisy labels, or novel cell-type discovery.

## When NOT to use

- The data are paired multi-omics with cross-omics pairing information, where methods like scAI or MOFA+ perform joint integration and clustering instead
- The target modality is protein abundance rather than scATAC-seq gene activity and no ATAC preprocessing is available to build a gene activity matrix
- You only need a quick joint visualization without annotations on either modality; scBridge requires annotated scRNA-seq source data

## Inputs

- Source scRNA-seq h5ad with cell-type annotations in obs['CellType']
- Target unlabeled scATAC-seq h5ad (gene activity matrix derived from peaks, e.g., via Signac) restricted to genes common to both modalities
- PBMC CITE-seq/ASAP-seq example data (GSE156478 or preprocessed scJoint data.zip with 4,644 CITE-seq / 4,506 ASAP-seq cells)

## Outputs

- Transferred labels in target obs['Prediction'] plus per-cell Reliability scores and integrated Embedding/UMAP AnnData files
- Benchmark tables (e.g., kidney_label_transfer_f1.csv, corruption_benchmark_results.csv) with per-cell-type F1/accuracy/silhouette per method and condition
- UMAP plots colored by cell type, domain, and reliability (e.g., marker_validation.png)

## How to apply

Format the data as source (scRNA-seq) and target (scATAC-seq) h5ad files with cell-type labels in obs['CellType'] on the source, converting scATAC-seq peaks to a gene activity matrix (e.g., with Signac) and retaining only genes common to both matrices. Run scBridge via 'python main.py --data_path=<folder> --source_data=<source>.h5ad --target_data=<target>.h5ad' with defaults: source preprocessing 'Standard', target preprocessing 'TFIDF', reliability_threshold 0.95, batch_size 512, train_epoch 20, learning_rate 5e-4, random_seed 2023, max_iteration 20, early_stop_acc 0.99; add --novel_type for structure-preservation loss so the most unreliable cells are predicted as a novel type. For robustness benchmarks, subsample annotated scRNA-seq cells to 100/75/50/25% budgets (expected: scBridge 77.35% to 77.08% average F1 on 10x Multiome kidney vs scJoint dropping from 73.22% to 59.36%) or randomly shuffle 5/10/20% of source labels (seed 2023) and compare with scJoint, GLUE v0.3.2, and Seurat. For novel-type discovery, model target-cell reliability with a scikit-learn Gaussian Mixture on embedding discriminability and classification confidence, stopping the iteration when more than 75% of scATAC-seq cells are selected as annotated data. Evaluate with average per-cell-type label transfer F1 (scikit-learn metrics) against held-out target annotations, silhouette scores, and NovelF1 for novel-type recovery.

## Related tools

- **scBridge** (Primary label-transfer method; iteratively selects reliable scATAC-seq cells and merges them with annotated scRNA-seq data, supports --novel_type mode with structure preservation loss) — https://github.com/XLearning-SCU/scBridge
- **scJoint** (Semi-supervised baseline for label-transfer benchmarking under downsampling and label corruption) — https://github.com/SydneyBioX/scJoint
- **GLUE** (Baseline method (v0.3.2) run on corrupted-label PBMC benchmark data) — https://github.com/gao-lab/GLUE
- **Signac** (R tool to transform scATAC-seq peaks into a gene activity matrix before scBridge input) — https://github.com/stuart-lab/signac
- **scikit-learn** (Gaussian Mixture Model for target-cell reliability estimation and F1/accuracy metric computation)
- **harmonypy** (Post-hoc embedding harmonization (v0.0.5) used in the integration pipeline) — https://github.com/slowkow/harmonypy

## Examples

```
python main.py --data_path=PBMC/ --source_data=CITE-seq.h5ad --target_data=ASAP-seq.h5ad
```

## Evaluation signals

- Downsampled-annotation trend reproduced: scBridge average F1 77.35% at full annotation to 77.08% at 25% on 10x Multiome mouse kidney, versus scJoint dropping from 73.22% to 59.36%
- On PBMC with 5/10/20% label corruption (seed 2023), scBridge accuracy and silhouette outperform Seurat, scJoint, and GLUE at every corruption rate
- Novel-type discovery: scATAC-seq-only Myeloid cells receive relatively low confidence scores and NovelF1 computed against the two-component-GMM threshold matches Fig. 4b-d
- Training terminates via early stop when annotated-data classification accuracy reaches 99%, or when >75% of scATAC-seq cells are selected as annotated data in novel-type mode
- Source h5ad contains obs['CellType'] and both matrices share only common genes after gene activity conversion

## Limitations

- Novel-type discovery mode relies on a heuristic stopping rule (halt when more than 75% of scATAC-seq cells are selected as annotated data)
- Protein-data integration results versus scJoint and the overall algorithm summary are only in Supplementary Notes 1 and 4, not in the main text
- Requires a single GPU (PyTorch 1.12.1, CUDA 11.4 environment) and scATAC-seq peaks must first be converted to a gene activity matrix, introducing a preprocessing dependency

## Evidence

- [abstract] Iterative reliable-cell selection rationale: "scBridge iterates between i) identifying reliable scATAC-seq cells that have smaller omics differences, and ii) integrating reliable scATAC-seq cells with scRNA-seq data to narrow the omics gap"
- [discussion] Reliability modeled with GMM: "we model the reliability of scATAC-seq cells with Gaussian Mixture based on their discriminability and confidence."
- [discussion] Early stopping rule: "we early stopped the network optimization when the cell classification accuracy of the annotated data reached 99%"
- [results] Downsampling benchmark result: [evidence span withheld — anchor status: not_found]
- [readme] Novel-type flag in README: [evidence span withheld — anchor status: not_found]
