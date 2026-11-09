---
name: knn-neighborhood-score-computation
description: Use when comparing embedding methods on a multi-sample single-cell dataset
  for batch-effect correction versus biological signal retention, e.g. scoring SCING
  module AUCell embeddings against FastMNN/Harmony/Seurat on snRNA-seq cell annotations.
when_to_use_negative:
- Input data has no batch/sample covariate or no cell-level phenotype annotations,
  so neither mixing nor signal retention can be scored
- You only need standard clustering quality metrics (silhouette, ARI) rather than
  an explicit batch-mixing versus biological-retention trade-off
- The embedding space was already produced by a dedicated batch-correction method
  and the goal is correction itself, not comparison of correction quality
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCING
  role: Provides the GRN-module AUCell embeddings whose batch-correction ability is
    scored via the neighborhood score
  repo: github.com/XiaYangLabOrg/SCING
- name: SCENIC
  role: Supplies the AUCell implementation used to compute module scores for each
    cell from Leiden-partitioned GRN subnetworks
  repo: github.com/XiaYangLabOrg/SCING
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:b3e1b8dcfe84f39ce394e7f48acec1e25618cd33e42a158045f09af59bdc64a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCENIC
  - SCING
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# knn-neighborhood-score-computation

## Summary

A KNN-based neighborhood score (Equation 1) that quantifies how well a cell-embedding space mixes cells across batches/samples while retaining biological annotations (diagnosis, sex, tangle/plaque stage), used in the SCING paper to benchmark GRN module AUCell embeddings against FastMNN, Harmony, and Seurat batch correction (Figure S6).

## When to use

Use when comparing embedding methods on a multi-sample single-cell dataset for batch-effect correction versus biological signal retention, e.g. scoring SCING module AUCell embeddings against FastMNN/Harmony/Seurat on snRNA-seq cell annotations.

## When NOT to use

- Input data has no batch/sample covariate or no cell-level phenotype annotations, so neither mixing nor signal retention can be scored
- You only need standard clustering quality metrics (silhouette, ARI) rather than an explicit batch-mixing versus biological-retention trade-off
- The embedding space was already produced by a dedicated batch-correction method and the goal is correction itself, not comparison of correction quality

## Inputs

- Cell-by-feature embedding matrix (e.g. first 20 principal components or SCING AUCell module scores per cell)
- Cell-level batch/sample annotations
- Cell-level phenotype annotations (diagnosis, sex, tangle stage, plaque stage)
- Baseline corrected embeddings from FastMNN, Harmony, and Seurat for comparison

## Outputs

- Neighborhood scores per embedding method, annotation, and neighbor-fraction (0.25%-16% sweep)
- Phenotype F1 scores per embedding method and annotation
- Comparison table (e.g. figure_s6_embedding_f1_comparison.csv)

## How to apply

Load cell-level embeddings (e.g. the first 20 principal components of the Morabito human prefrontal cortex snRNA-seq data, or SCING AUCell module scores obtained by running AUCell on Leiden-partitioned GRN subnetworks via ModuleBasedDimensionalityReduction) together with per-cell sample/batch and phenotype annotations (diagnosis, sex, tangle stage, plaque stage). For each annotation, sweep the KNN neighbor fraction across 0.25% to 16%: for each cell, retrieve its k nearest neighbors in the embedding, and compute the neighborhood score (Equation 1) as the balance between cross-batch/sample neighbor mixing (batch-effect correction) and neighbor agreement on the biological annotation (signal retention). Complement this with a phenotype F1 score (Equation 2), predicting the phenotype annotation from the embedding, to quantify retained biological information. Tabulate neighborhood and F1 scores per embedding method and annotation and compare SCING AUCell embeddings against FastMNN-, Harmony-, and Seurat-corrected baselines to judge whether the GRN-module embedding corrects batch effects comparably to dedicated methods.

## Related tools

- **SCING** (Provides the GRN-module AUCell embeddings whose batch-correction ability is scored via the neighborhood score) — github.com/XiaYangLabOrg/SCING
- **SCENIC** (Supplies the AUCell implementation used to compute module scores for each cell from Leiden-partitioned GRN subnetworks) — github.com/XiaYangLabOrg/SCING

## Examples

```
pip install pyscenic && jupyter notebook tutorials/ModuleBasedDimensionalityReduction.ipynb
```

## Evaluation signals

- Neighborhood score is computed for every annotation and across the full neighbor-fraction sweep (0.25%-16%), not a single k
- Embeddings use the first 20 principal components (or equivalent module-score space) as stated in the workflow
- Phenotype F1 scores (Equation 2) are reported alongside neighborhood scores for the same embeddings
- SCING AUCell embeddings show batch mixing and phenotype F1 comparable to FastMNN/Harmony/Seurat baselines, though not as optimized
- Results are tabulated per embedding method and annotation into a single CSV comparison table

## Limitations

- SCING module-based dimensionality reduction is not as optimized as dedicated batch-correction methods such as FastMNN, Harmony, and Seurat
- Inferred GRN links are not necessarily causal or directional, so module scores reflect association structure rather than causal regulation
- Hyperparameter selection (including neighbor fraction and network construction parameters) can be dataset dependent

## Evidence

- [abstract] intrinsic batch-correction capacity: "have intrinsic capacity to correct for batch effects, retrieve disease relevant genes and pathways, and are informative on spatial specificity of disease pathogenesis"
- [methods] AUCell module scoring: "We combine these subnetworks with the AUCell method to get module specific expression for each cell"
- [methods] Leiden partitioning: "We apply the leiden graph partitioning algorithm to identify GRN subnetworks"
- [readme] module use for clustering: "These can be used for clustering or phenotypic association."
- [methods] module-based dimensionality reduction: "ModuleBasedDimensionalityReduction.ipynb: Take the merged network, and divide it into subnetworks with the leiden graph partitioning algorithm, finally run the AUCell approach to get module scores"
