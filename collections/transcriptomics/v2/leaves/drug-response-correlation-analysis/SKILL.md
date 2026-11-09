---
name: drug-response-correlation-analysis
description: Use when a scRNA-seq dataset captures a time-course or drug-treatment
  progression and you have predicted per-cell response probabilities in adata.obs
  that you want to validate against trajectory inference (pseudotime).
when_to_use_negative:
- The dataset is not a drug-treatment or time-course study, so pseudotime has no relationship
  to treatment progression (e.g., static untreated tumor samples like GSE117872 without
  trajectory structure)
- The AnnData object lacks per-cell predicted response probabilities (adata.obs['sens_label'])
  because scDEAL prediction was never run
- The goal is bulk-level drug response prediction (IC50/area under curve regression)
  rather than single-cell response validation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDEAL
  role: Provides predicted drug-response probability scores (sens_label) and trajectory
    utilities (trajectory.py) for pseudotime analysis
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: Scanpy
  role: Loads and manipulates the AnnData (.h5ad) result object via scanpy.read_h5ad
    for correlation analysis
- name: AnnData
  role: Output container format holding cells, pseudotime, and response probabilities
    in .h5ad
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:fab1ed56ea23061886f355a5ca22bd5f6d62f081e565f5dc650041a7f6c0dbcf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - Scanpy
  - scDEAL
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# drug-response-correlation-analysis

## Summary

Validates single-cell drug response predictions by correlating scDEAL-predicted resistant/sensitive probability scores with Monocle3-style pseudotime, testing whether predicted response tracks treatment progression along a trajectory (e.g., I-BET-762-treated mouse AML cells from GSE110894).

## When to use

Use when a scRNA-seq dataset captures a time-course or drug-treatment progression and you have predicted per-cell response probabilities in adata.obs that you want to validate against trajectory inference (pseudotime).

## When NOT to use

- The dataset is not a drug-treatment or time-course study, so pseudotime has no relationship to treatment progression (e.g., static untreated tumor samples like GSE117872 without trajectory structure)
- The AnnData object lacks per-cell predicted response probabilities (adata.obs['sens_label']) because scDEAL prediction was never run
- The goal is bulk-level drug response prediction (IC50/area under curve regression) rather than single-cell response validation

## Inputs

- Pre-trained AnnData (.h5ad) object with scDEAL predictions (e.g., adata.zip for GSE110894)
- Per-cell response probability labels in adata.obs['sens_label'] (0 = resistant, 1 = sensitive)
- scDEAL trajectory utilities (trajectory.py)

## Outputs

- pseudotime_correlations.csv with resistant and sensitive Pearson correlation fields
- Regression plot of pseudotime versus response score (as in casestudy/analysis_tarined_anndata.ipynb)
- Pseudotime values per cell from Monocle3-style trajectory inference

## How to apply

Load the pre-trained AnnData result (e.g., adata.zip for GSE110894, 1419 mouse AML cells treated with I-BET-762) with scanpy.read_h5ad and extract scDEAL-predicted response probabilities from adata.obs['sens_label'] (0 = resistant, 1 = sensitive). Run Monocle3-style pseudotime trajectory inference on the same cells using scDEAL's trajectory.py utilities, which operate on Scanpy/AnnData objects. Compute Pearson correlations between inferred pseudotime and the predicted resistant and sensitive probability scores (expected: resistant r = +0.81, sensitive r = -0.93), and between differential-expression (DEG) scores and pseudotime, matching Fig. 5e-f and Source Data 9-10. Generate a regression plot of pseudotime versus response score following the casestudy/analysis_tarined_anndata.ipynb template and write all correlation values to pseudotime_correlations.csv. Accept the result if reproduced coefficients match the reported values within ±0.05.

## Related tools

- **scDEAL** (Provides predicted drug-response probability scores (sens_label) and trajectory utilities (trajectory.py) for pseudotime analysis) — https://github.com/OSU-BMBL/scDEAL.git
- **Scanpy** (Loads and manipulates the AnnData (.h5ad) result object via scanpy.read_h5ad for correlation analysis)
- **AnnData** (Output container format holding cells, pseudotime, and response probabilities in .h5ad)

## Examples

```
python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --checkpoint "save/sc_pre/integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl"
```

## Evaluation signals

- Pearson correlation between pseudotime and resistant probability score is +0.81 (tolerance ±0.05)
- Pearson correlation between pseudotime and sensitive probability score is -0.93 (tolerance ±0.05)
- pseudotime_correlations.csv exists and contains the required correlation fields
- Reproduced coefficients match Fig. 5e-f and Source Data 9-10 reported values
- adata.obs['sens_label'] contains binary labels (0 = resistant, 1 = sensitive) for all 1419 GSE110894 cells

## Limitations

- Due to the limited number of drug-treated mouse benchmark scRNA-seq data in the public domain, trans-species (human-to-mouse) reliability could not be systematically evaluated; Data 6 analysis relied on keeping only homologous genes in mouse data while the model was trained with human cancer cell lines
- Prediction accuracy could vary depending on the collection of bulk gene expression of cell lines used for scDEAL training
- No software version numbers are stated for tools or the environment, which may affect exact reproducibility of correlation values

## Evidence

- [results] Predicted drug-response probability scores are strongly correlated with pseudotime on Data 6: the resistant probability score is positively correlated with pseudotime (r = 0.81), and the sensitive probability score is negatively correlated (r = -0.93): [evidence span withheld — anchor status: not_found]
- [methods] Workflow for pseudotime correlation analysis: [evidence span withheld — anchor status: not_found]
- [readme] Input data and prediction labels: "python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512"
- [readme] Dataset description for Data 6: "Data 6    |     Bell, et al.       |     I-BET-762    |     GSE110894     |     1419     |     Mus   musculus    |     Acute   myeloid leukemia"
