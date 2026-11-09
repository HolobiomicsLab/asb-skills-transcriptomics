---
name: batch-correction-method-benchmarking
description: Use when you have multi-sample/single-batch-annotated scRNA-seq or snRNA-seq
  embeddings and need a quantitative benchmark of how well alternative embedding/correction
  methods mix batches while retaining phenotypic information such as diagnosis, sex,
  or disease stage.
when_to_use_negative:
- Input is already batch-corrected or has no sample/batch annotation, making the neighborhood-score
  batch-mixing metric uncomputable
- The biological question is GRN inference accuracy (e.g. perturbation-target prediction),
  not embedding-quality comparison across correction methods
- The dataset lacks a phenotype annotation (diagnosis/sex/stage) needed for the phenotype
  F1 score
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: SCING
  role: 'Provides the GRN module-based cell embeddings under evaluation: Leiden-partitioned
    GRN subnetworks are combined with AUCell to produce module scores used for clustering
    and batch-correction assessment'
  repo: https://github.com/XiaYangLabOrg/SCING
- name: SCENIC
  role: Supplies the AUCell method used to compute module-specific expression (module
    scores) per cell from GRN subnetworks; pyscenic must be installed for AUCell-based
    dimensionality reduction
  repo: https://github.com/XiaYangLabOrg/SCING
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:a791470a58766537b93b265d672bbb2232d1d8241c5f36a51e7972502455f626
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - SCENIC
  - SCING
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# batch-correction-method-benchmarking

## Summary

Quantitatively compares the batch-effect correction and biological signal retention of SCING GRN module AUCell embeddings against dedicated batch correction methods (FastMNN, Harmony, Seurat) using neighborhood scores and phenotype F1 metrics on the Morabito human prefrontal cortex snRNA-seq AD dataset.

## When to use

Use when you have multi-sample/single-batch-annotated scRNA-seq or snRNA-seq embeddings and need a quantitative benchmark of how well alternative embedding/correction methods mix batches while retaining phenotypic information such as diagnosis, sex, or disease stage.

## When NOT to use

- Input is already batch-corrected or has no sample/batch annotation, making the neighborhood-score batch-mixing metric uncomputable
- The biological question is GRN inference accuracy (e.g. perturbation-target prediction), not embedding-quality comparison across correction methods
- The dataset lacks a phenotype annotation (diagnosis/sex/stage) needed for the phenotype F1 score

## Inputs

- Cell-by-gene snRNA-seq matrix (Morabito human prefrontal cortex AD/control dataset)
- Cell-level annotations: sample, batch, diagnosis, tangle stage, plaque stage, sex
- SCING AUCell module-score embedding (per-cell GRN module scores from Leiden subnetworks)
- FastMNN-, Harmony-, and Seurat-corrected baseline embeddings

## Outputs

- Per-method, per-annotation neighborhood scores across 0.25%-16% neighbor fraction sweeps
- Phenotype F1 scores for each embedding method (SCING AUCell vs FastMNN, Harmony, Seurat)
- Comparison table written to figure_s6_embedding_f1_comparison.csv

## How to apply

Load the cell-by-gene matrix together with cell-level annotations (sample, batch, diagnosis, tangle stage, plaque stage, sex), then compute a neighborhood score (Equation 1) on the first 20 principal components for each annotation, sweeping the neighbor fraction from 0.25% to 16% to characterize batch mixing versus biological signal retention. Generate SCING-based embeddings by running the AUCell approach (from SCENIC) on Leiden-partitioned GRN subnetworks via the ModuleBasedDimensionalityReduction workflow to obtain module scores per cell. Compute a phenotype F1 score (Equation 2) for the SCING AUCell embeddings and for FastMNN-, Harmony-, and Seurat-corrected baseline embeddings by predicting each phenotype annotation from its respective embedding. Tabulate neighborhood and F1 scores per embedding method and annotation; the expectation from the source study is that SCING module embeddings correct batch effects comparably to the dedicated methods, though they are not as optimized.

## Related tools

- **SCING** (Provides the GRN module-based cell embeddings under evaluation: Leiden-partitioned GRN subnetworks are combined with AUCell to produce module scores used for clustering and batch-correction assessment) — https://github.com/XiaYangLabOrg/SCING
- **SCENIC** (Supplies the AUCell method used to compute module-specific expression (module scores) per cell from GRN subnetworks; pyscenic must be installed for AUCell-based dimensionality reduction) — https://github.com/XiaYangLabOrg/SCING

## Examples

```
conda activate scing && pip install pyscenic && jupyter execute tutorials/ModuleBasedDimensionalityReduction.ipynb
```

## Evaluation signals

- Neighborhood score sweep over 0.25%-16% neighbor fractions shows SCING AUCell embeddings reduce sample/batch signal comparably to FastMNN/Harmony/Seurat while retaining diagnosis and sex signal
- Phenotype F1 scores for SCING embeddings fall in a similar range to those of the dedicated correction baselines on the same annotations
- The comparison table is written to figure_s6_embedding_f1_comparison.csv with rows for each embedding-method x annotation pair
- Module scores are produced for every cell in the dataset via AUCell on all Leiden GRN subnetworks (no missing cells)
- Reproducing the pipeline reproduces the qualitative conclusion of Figure S6: module-based embeddings carry intrinsic batch-correction capacity but are not as optimized as dedicated methods

## Limitations

- SCING module-based dimensionality reduction is not as optimized for batch correction as dedicated methods such as FastMNN, Harmony, and Seurat
- SCING links between genes are not necessarily causal or directional
- Many hyperparameters require selection, and while robustness is shown across a range of hyperparameters, selection can be dataset dependent
- Inferred GRN properties (and thus module embeddings) can be affected by choices such as the 20% edge-consensus threshold and pruning via conditional mutual information

## Evidence

- [supplementary] SCING GRN module-based dimensionality reduction carries the ability to correct for batch effects and retain biological information: [evidence span withheld — anchor status: not_found]
- [methods] AUCell on Leiden subnetworks: "We combine these subnetworks with the AUCell method to get module specific expression for each cell"
- [readme] ModuleBasedDimensionalityReduction notebook description: "ModuleBasedDimensionalityReduction.ipynb: Take the merged network, and divide it into subnetworks with the leiden graph partitioning algorithm, finally run the AUCell approach to get module scores"
- [readme] pyscenic requirement: "If you want to use the AUCell from SCENIC for graph based dimensionality reduction you must install pyscenic"
- [abstract] intrinsic batch-effect capacity: "have intrinsic capacity to correct for batch effects, retrieve disease relevant genes and pathways, and are informative on spatial specificity of disease pathogenesis"
