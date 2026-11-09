---
name: phenotype-prediction-f1-scoring
description: Use when you have cell embeddings (GRN module AUCell scores or batch-corrected
  PCs) plus cell-level phenotype labels and need a quantitative readout of biological
  signal retention, e.g., reproducing Figure S6-style embedding comparisons.
when_to_use_negative:
- Input is a raw count matrix without any embedding or corrected representation —
  build embeddings first (e.g., SCING AUCell module scores) before scoring phenotypes.
- The goal is measuring batch mixing alone rather than biological signal retention
  — the neighborhood score (Equation 1) sweep is the more direct metric.
- Data has no meaningful cell-level phenotype labels (diagnosis, sex, stage); F1 of
  phenotype prediction would be undefined or uninformative.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCING
  role: 'Provides the GRN module-based embeddings being scored: merged network is
    partitioned into Leiden subnetworks and converted to per-cell AUCell module scores
    via ModuleBasedDimensionalityReduction'
  repo: https://github.com/XiaYangLabOrg/SCING
- name: SCENIC
  role: Supplies the AUCell method used to compute module-specific expression (module
    scores) for each cell from GRN subnetworks
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:1cae7eaccaf51353d84211d46ab557d1239c60b41215f537193290a77e5cace1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - SCENIC
  - SCING
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# phenotype-prediction-f1-scoring

## Summary

Quantifies how well a cell embedding (e.g., SCING GRN module AUCell scores) retains biological information by predicting a cell-level phenotype annotation (diagnosis, sex, tangle/plaque stage) and scoring it with the phenotype F1 score (Equation 2), enabling comparison against batch-correction baselines like FastMNN, Harmony, and Seurat.

## When to use

Use when you have cell embeddings (GRN module AUCell scores or batch-corrected PCs) plus cell-level phenotype labels and need a quantitative readout of biological signal retention, e.g., reproducing Figure S6-style embedding comparisons.

## When NOT to use

- Input is a raw count matrix without any embedding or corrected representation — build embeddings first (e.g., SCING AUCell module scores) before scoring phenotypes.
- The goal is measuring batch mixing alone rather than biological signal retention — the neighborhood score (Equation 1) sweep is the more direct metric.
- Data has no meaningful cell-level phenotype labels (diagnosis, sex, stage); F1 of phenotype prediction would be undefined or uninformative.

## Inputs

- Cell-level phenotype annotations for the snRNA-seq dataset: sample, batch, diagnosis, tangle stage, plaque stage, sex
- SCING AUCell module-score embeddings (per-cell module scores from Leiden-partitioned GRN subnetworks)
- Baseline corrected embeddings from FastMNN, Harmony, and Seurat (e.g., first 20 principal components)

## Outputs

- Phenotype F1 score per embedding method and per annotation (Equation 2)
- Neighborhood scores per annotation across 0.25%-16% neighbor fractions (Equation 1)
- Comparison table written to figure_s6_embedding_f1_comparison.csv (Figure S6 reproduction)

## How to apply

Load the Morabito human prefrontal cortex snRNA-seq dataset with cell-level annotations (sample, batch, diagnosis, tangle stage, plaque stage, sex). Obtain SCING AUCell embeddings by running ModuleBasedDimensionalityReduction.ipynb: take the merged SCING network, divide it into subnetworks with the Leiden graph partitioning algorithm, and run the AUCell approach (from SCENIC/pyscenic) on each Leiden-partitioned GRN subnetwork to get module scores for every cell. For each embedding (SCING AUCell module scores plus FastMNN, Harmony, and Seurat corrected baselines) and each phenotype annotation, train a predictor on the embedding and compute the phenotype F1 score (Equation 2). Complement F1 with the neighborhood score (Equation 1) computed on the first 20 principal components, sweeping the neighbor fraction over 0.25%-16% to quantify batch mixing versus biological retention. Tabulate both metrics per embedding method and annotation and write the comparison to figure_s6_embedding_f1_comparison.csv.

## Related tools

- **SCING** (Provides the GRN module-based embeddings being scored: merged network is partitioned into Leiden subnetworks and converted to per-cell AUCell module scores via ModuleBasedDimensionalityReduction) — https://github.com/XiaYangLabOrg/SCING
- **SCENIC** (Supplies the AUCell method used to compute module-specific expression (module scores) for each cell from GRN subnetworks)

## Examples

```
jupyter nbconvert --to notebook --execute ModuleBasedDimensionalityReduction.ipynb
```

## Evaluation signals

- SCING AUCell-embedding phenotype F1 scores should be comparable to FastMNN, Harmony, and Seurat baselines across annotations (diagnosis, sex, tangle/plaque stage), though not as optimized as the dedicated methods
- Neighborhood score sweep over 0.25%-16% neighbor fractions should show SCING embeddings correct batch/sample effects while retaining biological information (Figure S6 pattern)
- Every cell must receive a module score from each Leiden-partitioned GRN subnetwork (AUCell applied to all subnetworks)
- Output table figure_s6_embedding_f1_comparison.csv contains one row per (embedding method, annotation) pair with both neighborhood score and phenotype F1

## Limitations

- SCING module-based dimensionality reduction corrects batch effects in a manner similar to, but not as optimized as, dedicated batch correction methods such as FastMNN, Harmony, and Seurat
- Many hyperparameters (e.g., Leiden partitioning resolution, neighbor fraction range) require selection, and selection can be dataset dependent
- AUCell-based dimensionality reduction requires pyscenic to be installed separately

## Evidence

- [methods] AUCell module scoring for embeddings: "We combine these subnetworks with the AUCell method to get module specific expression for each cell"
- [intro] Modules usable for phenotypic association: "These modules can be used for clustering, phenotypic association, and biological annotation through"
- [readme] ModuleBasedDimensionalityReduction notebook: "ModuleBasedDimensionalityReduction.ipynb: Take the merged network, and divide it into subnetworks with the leiden graph partitioning algorithm"
- [abstract] Batch-effect correction capacity of SCING GRNs: "have intrinsic capacity to correct for batch effects, retrieve disease relevant genes and pathways"
- [methods] Dataset applied in the paper: "We apply SCING to human prefrontal cortex snRNAseq data with AD and Control patients"
