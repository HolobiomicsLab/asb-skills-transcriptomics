---
name: biologically-informed-loss-function-implementation
description: Use when implementing, re-implementing, or validating BIDCell's biologically-informed
  loss functions on subcellular spatial transcriptomics data (e.g. Xenium, CosMx,
  MERSCOPE) and you need per-component loss values and correctness checks.
when_to_use_negative:
- You need to run the full segmentation pipeline end-to-end rather than implement/verify
  loss functions — use the BIDCell run_pipeline workflow (preprocess, train, predict)
  instead.
- Input lacks a single-cell reference or positive/negative marker files, since the
  marker losses require cell-type-specific markers derived from the highest/lowest
  10th expression percentiles.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0625
tools:
- name: BIDCell
  role: Framework whose six biologically-informed loss functions are being implemented
    and whose BIDCellModel preprocessing loads the input patches
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: Source platform of the Xenium-BreastCancer1 dataset and the Xenium Output
    Bundle In Situ Replicate 1 used as real loss-evaluation inputs
- name: SydneyBioX/scClassify
  role: Source of example single-cell reference data used to derive cell-type information
    for marker losses
  repo: https://github.com/SydneyBioX/scClassify
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:efbee6b659c922ec4cd95a7344d510190604f1410406a3307e3985a3fb7aacee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  - SydneyBioX/scClassify
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# biologically-informed-loss-function-implementation

## Summary

Implements and evaluates the six biologically-informed self-supervised loss functions of BIDCell (nuclei encapsulation, cell-calling with eccentricity-driven elongated expansion, over-segmentation, overlap, positive-marker, negative-marker) and their weighted-sum total loss, so that segmentation of subcellular spatial transcriptomics data is guided by cell-type and morphology knowledge.

## When to use

Use when implementing, re-implementing, or validating BIDCell's biologically-informed loss functions on subcellular spatial transcriptomics data (e.g. Xenium, CosMx, MERSCOPE) and you need per-component loss values and correctness checks.

## When NOT to use

- You need to run the full segmentation pipeline end-to-end rather than implement/verify loss functions — use the BIDCell run_pipeline workflow (preprocess, train, predict) instead.
- Input lacks a single-cell reference or positive/negative marker files, since the marker losses require cell-type-specific markers derived from the highest/lowest 10th expression percentiles.

## Inputs

- Xenium Output Bundle In Situ Replicate 1 patches (transcripts.csv.gz transcript coordinates and morphology_mip.ome.tif DAPI image)
- Breast cancer single-cell reference with positive/negative marker files (sc_breast.csv, sc_breast_markers_pos.csv, sc_breast_markers_neg.csv)
- Initial nuclei segmentation / cell-calling masks for morphology guidance

## Outputs

- Per-component loss values for all six losses on real Xenium-BreastCancer1 patches
- Weighted-sum total loss value (Eq. 13) used in BIDCellModel training
- Validation report confirming each loss value lies within the normalized range [0, ~1]

## How to apply

Load real input patches from the Xenium Output Bundle In Situ Replicate 1 (transcripts.csv.gz and morphology_mip.ome.tif) using BIDCellModel preprocessing. Implement the nuclei encapsulation and cell-calling losses including the eccentricity-driven elongated expansion mask component (Eqs. 1-4), then the over-segmentation and overlap losses (Eqs. 5-8) that ensure cohesiveness of cell instances. Implement the positive-marker and negative-marker losses from sc_breast_markers_pos.csv and sc_breast_markers_neg.csv, whose markers come from the highest and lowest 10th expression percentile per cell type (Eqs. 9-12). Combine all six components in the weighted-sum total loss (Eq. 13) as used in BIDCellModel training, compute per-component and total values on real Xenium-BreastCancer1 patches, and validate that every value falls within the normalized range [0, ~1]; any out-of-range value indicates incorrect normalization or implementation. The loss design supports appropriate cell sizes by capturing expression patterns local to nuclei using guidance from cell-type informed morphologies (cell-calling), while the over-segmentation loss ensures instance cohesiveness.

## Related tools

- **BIDCell** (Framework whose six biologically-informed loss functions are being implemented and whose BIDCellModel preprocessing loads the input patches) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (Source platform of the Xenium-BreastCancer1 dataset and the Xenium Output Bundle In Situ Replicate 1 used as real loss-evaluation inputs)
- **SydneyBioX/scClassify** (Source of example single-cell reference data used to derive cell-type information for marker losses) — https://github.com/SydneyBioX/scClassify

## Examples

```
from bidcell import BIDCellModel; model = BIDCellModel("xenium_example_config.yaml"); model.preprocess(); model.train()
```

## Evaluation signals

- Every per-component loss value (nuclei encapsulation, cell-calling, over-segmentation, overlap, positive-marker, negative-marker) falls within the normalized valid range [0, ~1]
- Total weighted-sum loss (Eq. 13) also falls within [0, ~1]; any out-of-range value indicates incorrect normalization or implementation
- Positive and negative marker lists match the highest and lowest 10th expression percentile per cell type of the sc_breast reference
- Losses computed on real Xenium-BreastCancer1 patches (transcripts.csv.gz + morphology_mip.ome.tif) reproduce non-degenerate values, not zeros

## Limitations

- Marker-gene choice is sensitive: removing positive markers common to at least a third of cell types was found appropriate, and using a larger number of positive markers tends to increase the size of predicted cells
- Two-dimensional loss-driven segmentation may achieve limited representation of cells' three-dimensional structure, and elongated morphologies (melanocytes, fibroblasts, dendrites, axons) remain challenging
- Achieving more homogeneous cell bodies via the over-segmentation loss can also result from arbitrary over-segmentation of nuclei, motivating use of multiple metrics

## Evidence

- [results] BIDCell's loss design supports appropriate cell sizes by capturing expression patterns local to nuclei using guidance from cell-type informed morphologies (cell-calling), while ensuring the cohesiveness of cell instances via the over-segmentation loss.: [evidence span withheld — anchor status: not_found]
- [abstract] self-supervised deep learning-based framework with biologically-informed loss functions: "we present BIDCell, a self-supervised deep learning-based framework with biologically-informed loss functions that learn relationships between spatially resolved gene expression and cell morphology"
- [readme] The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample.: "The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample."
- [readme] Please see the provided breast cancer single-cell reference and positive/negative marker files: "Please see the provided breast cancer single-cell reference and positive/negative marker files (`sc_breast.csv`, `sc_breast_markers_pos.csv`, and `sc_breast_markers_neg.csv`)"
