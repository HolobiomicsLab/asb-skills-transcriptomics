---
name: scaling-benchmark-measurement
description: Use when reproducing an integration pipeline at atlas scale and you must
  verify the reported computational efficiency (runtime and memory scaling) against
  baselines such as scJoint and Seurat.
when_to_use_negative:
- When the goal is only label-transfer accuracy (F1) or embedding quality and no efficiency/scaling
  claim needs verification — use the label-transfer evaluation skill instead.
- When the dataset is small paired multi-omics data (e.g., the 4,644 CITE-seq / 4,506
  ASAP-seq PBMC cells from GSE156478), where atlas-scale scaling benchmarks are not
  the claim under test.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scBridge
  role: Integration method whose runtime and peak memory are benchmarked on the mouse
    atlas dataset
  repo: https://github.com/XLearning-SCU/scBridge
- name: PyTorch 1.12.1
  role: Framework in which scBridge is implemented and executed on a single GPU for
    the timing runs
  repo: https://github.com/XLearning-SCU/scBridge
- name: scJoint
  role: Baseline whose runtime/memory scaling is compared against scBridge at atlas
    scale
  repo: https://github.com/SydneyBioX/scJoint
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:d36b89d94a8be052e780beb8ba0b6f30d153181b31a445e1f0867be76ec4eb89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - PyTorch
  - PyTorch 1.12.1
  - Python
  - scBridge
  - scJoint
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# scaling-benchmark-measurement

## Summary

Measures wall-clock time and peak memory of a multi-omics integration method (e.g., scBridge) over atlas-scale inputs (~102,103 mouse atlas cells) to verify the linear-time/constant-memory scaling profile reported in the paper's Fig. 3.

## When to use

Use when reproducing an integration pipeline at atlas scale and you must verify the reported computational efficiency (runtime and memory scaling) against baselines such as scJoint and Seurat.

## When NOT to use

- When the goal is only label-transfer accuracy (F1) or embedding quality and no efficiency/scaling claim needs verification — use the label-transfer evaluation skill instead.
- When the dataset is small paired multi-omics data (e.g., the 4,644 CITE-seq / 4,506 ASAP-seq PBMC cells from GSE156478), where atlas-scale scaling benchmarks are not the claim under test.

## Inputs

- Mouse atlas scRNA-seq count matrices with cell-type annotations (Tabula Muris, from tabula-muris.ds.czbiohub.org)
- Mouse atlas scATAC-seq gene activity and peak-by-cell matrices with cell-type annotations (GSE111586, from atlas.gs.washington.edu/mouse-atac/)
- scBridge configuration: reliability_threshold=0.95, max_iteration=20, batch_size=512, train_epoch=20, learning_rate=5e-4, early stopping at annotated-data accuracy 0.99

## Outputs

- Wall-clock runtime of the integration over ~102,103 cells
- Peak memory usage of the integration run
- Scaling profile comparison (time/memory vs input size) against scJoint and Seurat baselines as in Fig. 3

## How to apply

Run the scBridge integration (main.py via Python/PyTorch on a single GPU, with reliability_threshold=0.95, max_iteration=20, batch_size=512, train_epoch=20, learning_rate=5e-4) on the mouse atlas data restricted to cells of the 18 common cell types and the 12,689 common genes between Tabula Muris scRNA-seq and Cusanovich mouse atlas scATAC-seq (GSE111586). Instrument the run to record wall-clock time and peak memory over the ~102,103 cells, matching the paper's hardware setting (Nvidia RTX 3090 GPU with CUDA 11.4, Ubuntu 20.04). Compare measured F1-scores (scBridge 89.66% vs scJoint 64.60% vs Seurat 44.64%), harmonized cell-type/omics silhouette scores, and the time/memory scaling profile against the values in Fig. 3. Flag the run as a reproduction failure if the scaling measurements deviate from the reported linear-time/constant-memory behavior.

## Related tools

- **scBridge** (Integration method whose runtime and peak memory are benchmarked on the mouse atlas dataset) — https://github.com/XLearning-SCU/scBridge
- **PyTorch 1.12.1** (Framework in which scBridge is implemented and executed on a single GPU for the timing runs) — https://github.com/XLearning-SCU/scBridge
- **scJoint** (Baseline whose runtime/memory scaling is compared against scBridge at atlas scale) — https://github.com/SydneyBioX/scJoint

## Evaluation signals

- Measured wall-clock time follows the linear-time scaling profile reported in the paper's Fig. 3 for the mouse atlas dataset
- Measured peak memory follows the constant-memory scaling profile reported in Fig. 3
- F1-score on the transferred labels reaches ~89.66% (vs scJoint 64.60%, Seurat 44.64%) for the same run
- Cell type and omics silhouette scores of the joint embedding match the paper after harmonization (e.g., with harmonypy)
- Run completes on a single GPU within the paper's hardware envelope (Nvidia RTX 3090, CUDA 11.4); otherwise flag as reproduction failure

## Limitations

- Hardware-dependent: results were obtained on an Nvidia RTX 3090 GPU with CUDA 11.4 on Ubuntu 20.04; other hardware will shift absolute times/memory.
- Scaling measurements are only meaningful on the atlas-scale mouse data (~102,103 cells); smaller datasets cannot exercise the linear-time/constant-memory regime.
- A deviation of computed F1-scores, harmonized silhouette scores, or scaling measurements from Fig. 3 values must be flagged as a reproduction failure.

## Evidence

- [discussion] We reserved cells of 18 common types and 12,689 common genes for the analysis: "We reserved cells of 18 common types and 12,689 common genes for the analysis"
- [discussion] All experiments are conducted on an Nvidia RTX 3090 GPU with CUDA 11.4, on the Ubuntu 20.04 OS.: "All experiments are conducted on an Nvidia RTX 3090 GPU with CUDA 11.4, on the Ubuntu 20.04 OS."
- [discussion] scBridge is implemented in Python using the PyTorch framework, v.1.12.1.: "scBridge is implemented in Python using the PyTorch41 framework, v.1.12.1."
- [readme] Note: scBridge runs on a single GPU.: "Note: scBridge runs on a single GPU."
