---
name: single-cell-marker-based-loss-computation
description: Use when training or reproducing BIDCell-style segmentation on SST data
  (e.g. Xenium transcripts.csv.gz + morphology_mip.ome.tif) and you need per-component
  and total biologically-informed loss values on real image patches.
when_to_use_negative:
- Input is already a segmented cell-gene matrix and you only need downstream expression
  analysis, not segmentation training
- Data are segmentation-free transcript-only assignments (e.g. Baysor-style clustering)
  where morphology-informed losses on image patches do not apply
- No single-cell reference with cell-type annotations is available, so marker-based
  losses cannot be constructed
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_2229
tools:
- name: BIDCell
  role: Framework providing BIDCellModel preprocessing, training, and the six biologically-informed
    loss functions applied to Xenium patches
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: SST platform supplying the breast cancer dataset (transcripts.csv.gz, morphology_mip.ome.tif)
    on which losses are computed
- name: SydneyBioX/scClassify
  role: Source of example single-cell reference data used for cell-type-informed losses
  repo: https://github.com/SydneyBioX/scClassify
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:feb85a5a6790b7419e58c25abc957c203915b6530c4c2fe5b1065dc4a685465c
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
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Single-cell marker-based loss computation for biologically-informed cell segmentation

## Summary

Implements and computes the six biologically-informed self-supervised loss functions of BIDCell (nuclei encapsulation, cell-calling, over-segmentation, overlap, positive-marker, negative-marker) that operationalise cell-type-informed morphology and local expression patterns from single-cell reference data to guide segmentation of Xenium spatial transcriptomics images.

## When to use

Use when training or reproducing BIDCell-style segmentation on SST data (e.g. Xenium transcripts.csv.gz + morphology_mip.ome.tif) and you need per-component and total biologically-informed loss values on real image patches.

## When NOT to use

- Input is already a segmented cell-gene matrix and you only need downstream expression analysis, not segmentation training
- Data are segmentation-free transcript-only assignments (e.g. Baysor-style clustering) where morphology-informed losses on image patches do not apply
- No single-cell reference with cell-type annotations is available, so marker-based losses cannot be constructed

## Inputs

- Xenium transcripts.csv.gz
- DAPI/nuclei image morphology_mip.ome.tif
- Breast cancer single-cell reference sc_breast.csv (average expression per cell type)
- Positive marker file sc_breast_markers_pos.csv (top 10th expression percentile per cell type)
- Negative marker file sc_breast_markers_neg.csv (lowest 10th expression percentile per cell type)
- BIDCellModel configuration yaml (e.g. xenium_example_config.yaml)

## Outputs

- Per-component loss values for all six biologically-informed losses
- Weighted-sum total loss value (Eq. 13)
- Trained BIDCell segmentation model applicable to prediction of cell masks

## How to apply

Load real input patches from the Xenium Output Bundle In Situ Replicate 1 using BIDCellModel preprocessing (transcripts.csv.gz and morphology_mip.ome.tif). Implement the nuclei encapsulation and cell-calling losses including the eccentricity-driven elongated expansion mask component (Eqs. 1-4), then the over-segmentation and overlap losses (Eqs. 5-8), then the positive- and negative-marker losses using sc_breast_markers_pos.csv and sc_breast_markers_neg.csv, whose markers come from the highest and lowest 10th expression percentile per cell type (Eqs. 9-12). Combine all six components into the weighted-sum total loss (Eq. 13) as used in BIDCellModel training. Compute all values on real Xenium-BreastCancer1 patches and record per-component and total values; use the single-cell reference (sc_breast.csv) average expressions per cell type, noting that more positive markers tends to increase predicted cell size and positive markers common to at least a third of cell types should be removed.

## Related tools

- **BIDCell** (Framework providing BIDCellModel preprocessing, training, and the six biologically-informed loss functions applied to Xenium patches) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (SST platform supplying the breast cancer dataset (transcripts.csv.gz, morphology_mip.ome.tif) on which losses are computed)
- **SydneyBioX/scClassify** (Source of example single-cell reference data used for cell-type-informed losses) — https://github.com/SydneyBioX/scClassify

## Examples

```
from bidcell import BIDCellModel; model = BIDCellModel("xenium_example_config.yaml"); model.preprocess(); model.train()
```

## Evaluation signals

- Every per-component and total loss value falls within the normalized valid range [0, ~1]; any out-of-range value indicates incorrect normalization or implementation
- All six components (Eqs. 1-12) plus the weighted total (Eq. 13) are computed and recorded per patch
- Positive/negative marker lists match the highest/lowest 10th percentile criterion per cell type from the sc_breast reference
- Losses computed on real Xenium-BreastCancer1 patches, not synthetic placeholders
- Pipeline stages (preprocess, train, predict) run without truncation errors, e.g. pickle.UnpicklingError

## Limitations

- Using a larger number of positive markers tends to increase the size of predicted cells, so marker curation affects segmentation outcomes
- Removing positive markers common to at least a third of cell types was found appropriate, but manual curation or alternative marker-determination approaches may be needed
- Only <1,000 genes are needed for segmentation, so gene selection must be specified carefully (see Stereo-seq example)

## Evidence

- [results] BIDCell's loss design supports appropriate cell sizes by capturing expression patterns local to nuclei using guidance from cell-type informed morphologies (cell-calling), while ensuring the cohesiveness of cell instances via the over-segmentation loss.: [evidence span withheld — anchor status: not_found]
- [readme] Positive and negative markers derive from top/bottom 10th percentile expression: "The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample."
- [abstract] Model framework with biologically-informed loss functions from abstract: "we present BIDCell, a self-supervised deep learning-based framework with biologically-informed loss functions that learn relationships between spatially resolved gene expression and cell morphology"
- [readme] Marker curation guidance from README: "We found that removing positive markers that were common to at least a third of cell types in each dataset was appropriate across various datasets."
