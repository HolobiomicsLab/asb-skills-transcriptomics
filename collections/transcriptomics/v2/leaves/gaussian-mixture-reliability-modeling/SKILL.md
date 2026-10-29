---
name: gaussian-mixture-reliability-modeling
description: Use when transferring labels from annotated scRNA-seq to unlabeled scATAC-seq
  and you must decide per-cell which target cells can safely be merged into the annotated
  source during iterative integration.
when_to_use_negative:
- Source and target share the same omics modality (e.g. scRNA-seq to scRNA-seq label
  transfer), where the cross-omics reliability problem does not arise
- Paired multi-omics data where joint integration tools such as scAI or MOFA+ can
  exploit cross-omics pairing information directly
- You need novel cell-type discovery without any annotated target cells — that requires
  the separate structure preservation loss (--novel_type) mode rather than plain reliability
  selection
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scBridge
  role: 'Hosts the reliability modeling loop: warm-up, GMM reliability estimation,
    prototype alignment, and reliable-cell merging'
  repo: https://github.com/XLearning-SCU/scBridge
- name: Scikit-learn
  role: Provides the Gaussian Mixture Model used to estimate per-cell scATAC-seq reliability
    (v.1.1.1)
- name: PyTorch
  role: Implements the encoder f and classifier g warmed up before reliability estimation
    (v.1.12.1)
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:3a397fbdde97d9b1333bb8ab2de2e0e0d7f9e4e82e5d21722326dcbe5c96d9a0
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
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# gaussian-mixture-reliability-modeling

## Summary

A scBridge component that models the reliability of scATAC-seq cells with a Gaussian Mixture Model over embedding discriminability and classification confidence, enabling iterative selection of reliable target cells whose omics difference from scRNA-seq is small.

## When to use

Use when transferring labels from annotated scRNA-seq to unlabeled scATAC-seq and you must decide per-cell which target cells can safely be merged into the annotated source during iterative integration.

## When NOT to use

- Source and target share the same omics modality (e.g. scRNA-seq to scRNA-seq label transfer), where the cross-omics reliability problem does not arise
- Paired multi-omics data where joint integration tools such as scAI or MOFA+ can exploit cross-omics pairing information directly
- You need novel cell-type discovery without any annotated target cells — that requires the separate structure preservation loss (--novel_type) mode rather than plain reliability selection

## Inputs

- Annotated source scRNA-seq AnnData (.h5ad) with cell types in obs['CellType']
- Unlabeled target scATAC-seq gene-activity AnnData (.h5ad)
- Source/target embeddings and classification confidence scores from the warmed-up encoder/classifier

## Outputs

- Per-cell reliability scores written to target AnnData .obs['Reliability']
- Transferred labels in target AnnData .obs['Prediction']
- Set of reliable scATAC-seq cells merged into the annotated data at each iteration

## How to apply

After warming up the PyTorch encoder and classifier on annotated scRNA-seq cells (weighted cross-entropy, lr=5e-4, batch_size=512, train_epoch=20, seed=2023, early stop at 99% annotated-data accuracy), embed the scATAC-seq cells and fit a scikit-learn Gaussian Mixture Model (v1.1.1) over their discriminability and confidence to estimate per-cell reliability. Cells whose reliability exceeds reliability_threshold=0.95 are treated as having smaller omics differences and are merged into the annotated data; prototype alignment (prototype_momentum=0.9, align_loss_epoch=1) then narrows the cross-omics gap, and the loop repeats (max_iteration=20) until no new cells are selected. Rationale: omics difference varies across cells, so reliability-weighted selection avoids over- or under-integration that occurs when cell-type and omics differences are conflated.

## Related tools

- **scBridge** (Hosts the reliability modeling loop: warm-up, GMM reliability estimation, prototype alignment, and reliable-cell merging) — https://github.com/XLearning-SCU/scBridge
- **Scikit-learn** (Provides the Gaussian Mixture Model used to estimate per-cell scATAC-seq reliability (v.1.1.1))
- **PyTorch** (Implements the encoder f and classifier g warmed up before reliability estimation (v.1.12.1))

## Examples

```
git clone https://github.com/XLearning-SCU/scBridge; python main.py --source_preprocess Standard --target_preprocess TFIDF --reliability_threshold 0.95 --max_iteration 20 --seed 2023
```

## Evaluation signals

- Reliability scores in .obs['Reliability'] are in [0,1] and reliable cells are exactly those with reliability > 0.95
- Label transfer accuracy grows monotonically across iterations (e.g. SNARE-seq cortex: 60.11% at first iteration rising to 71.95% final)
- The iteration terminates before max_iteration=20 when no new scATAC-seq cells exceed the reliability threshold
- Integrated target AnnData contains non-empty Prediction and Reliability fields for all scATAC-seq cells
- Reported accuracy matches the benchmark value (71.95% accuracy on the SNARE-seq mouse cortex dataset)

## Limitations

- Requires a scATAC-seq gene activity matrix; raw peak-by-cell data must first be converted with tools such as Signac
- Performance is threshold-sensitive: reliability_threshold defaults to 0.95, and miscalibrated GMM components can cause over-integration or under-integration
- Novel-type discovery mode relies on a heuristic stopping rule (stop when more than 75% of scATAC-seq cells are selected as annotated data)

## Evidence

- [discussion] GMM reliability estimation grounded in article methods: "we model the reliability of scATAC-seq cells with Gaussian Mixture based on their discriminability and confidence."
- [methods] Threshold parameter for reliable-cell selection: "`--reliability_threshold=[0.0-1.0]` (Default=0.95) threshold for selecting reliable cells"
- [discussion] Iterative merge-and-repeat behavior of the reliability loop: "scBridge selects the most reliable scATAC-seq cells as the annotated data and repeats the entire process."
- [readme] Input contract for the scBridge pipeline from the README: "The source data should include the cell type annotation in `obs.["CellType"]`"
