---
name: single-cell-embedding-quality-evaluation
description: Use when comparing a novel embedding (e.g., GRN module AUCell scores)
  against batch-correction baselines on annotated snRNA-seq data, to quantify batch
  mixing vs retention of annotations like diagnosis, sex, or batch/sample.
when_to_use_negative:
- The goal is GRN inference or edge pruning itself (BuildNetwork / supercell construction),
  not evaluating embeddings downstream of networks
- The input data lacks cell-level annotations (batch/sample/diagnosis/sex), so neither
  neighborhood scores nor phenotype F1 can be computed
- A single embedding is being built for clustering without any baseline comparison
  — the skill is specifically a comparative evaluation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCING
  role: Source of the GRN module AUCell embeddings being evaluated (ModuleBasedDimensionalityReduction)
  repo: https://github.com/XiaYangLabOrg/SCING
- name: SCENIC
  role: Provides the AUCell method used to score GRN modules per cell for graph-based
    dimensionality reduction
  repo: https://github.com/XiaYangLabOrg/SCING
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:2cb8c13f9e2fc38adfe6d5dd5956ea5a98180af2fd64d5964170c78477e5c81d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - SCENIC
  - SCING
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-embedding-quality-evaluation

## Summary

Quantitative evaluation of single-cell embeddings (e.g., SCING GRN module AUCell embeddings) for batch-effect correction versus biological signal retention, benchmarked against dedicated batch-correction methods such as FastMNN, Harmony, and Seurat using neighborhood scores and phenotype F1 scores (Figure S6).

## When to use

Use when comparing a novel embedding (e.g., GRN module AUCell scores) against batch-correction baselines on annotated snRNA-seq data, to quantify batch mixing vs retention of annotations like diagnosis, sex, or batch/sample.

## When NOT to use

- The goal is GRN inference or edge pruning itself (BuildNetwork / supercell construction), not evaluating embeddings downstream of networks
- The input data lacks cell-level annotations (batch/sample/diagnosis/sex), so neither neighborhood scores nor phenotype F1 can be computed
- A single embedding is being built for clustering without any baseline comparison — the skill is specifically a comparative evaluation

## Inputs

- Cell-level annotated snRNA-seq dataset (e.g., Morabito human prefrontal cortex AD/Control dataset with sample, batch, diagnosis, tangle stage, plaque stage, sex annotations)
- SCING GRN module AUCell embeddings (module scores per cell from Leiden-partitioned subnetworks)
- Baseline batch-corrected embeddings from FastMNN, Harmony, and Seurat

## Outputs

- Neighborhood score per embedding method and annotation across neighbor fractions (0.25%–16%)
- Phenotype F1 score per embedding method and annotation
- Comparison table (e.g., figure_s6_embedding_f1_comparison.csv) reproducing the Figure S6 comparison

## How to apply

Load the cell-level annotated dataset (here the Morabito human prefrontal cortex AD snRNA-seq dataset with sample, batch, diagnosis, tangle stage, plaque stage, and sex annotations). Generate the candidate embedding by applying the AUCell method to Leiden-partitioned GRN subnetworks (ModuleBasedDimensionalityReduction) to obtain module scores per cell, and compute baseline corrected embeddings with FastMNN, Harmony, and Seurat. Quantify batch mixing versus biological retention with the neighborhood score (Equation 1) computed on the first 20 principal components, sweeping the neighbor fraction from 0.25% to 16% for each annotation. Compute the phenotype F1 score (Equation 2) per embedding and annotation by predicting each phenotype label from the embedding. Tabulate neighborhood scores and phenotype F1 scores per method and annotation (e.g., to figure_s6_embedding_f1_comparison.csv) and interpret: GRN module embeddings correct batch effects similarly to dedicated methods, though not as optimized.

## Related tools

- **SCING** (Source of the GRN module AUCell embeddings being evaluated (ModuleBasedDimensionalityReduction)) — https://github.com/XiaYangLabOrg/SCING
- **SCENIC** (Provides the AUCell method used to score GRN modules per cell for graph-based dimensionality reduction) — https://github.com/XiaYangLabOrg/SCING

## Examples

```
conda activate scing; pip install pyscenic  # then run ModuleBasedDimensionalityReduction.ipynb to get AUCell module scores per cell for the embedding comparison
```

## Evaluation signals

- Neighborhood score curves (Equation 1, first 20 PCs, neighbor fraction swept 0.25%–16%) show batch mixing comparable to FastMNN/Harmony/Seurat baselines
- Phenotype F1 scores (Equation 2) for SCING AUCell embeddings are within a similar range to the dedicated batch-correction baselines for annotations like diagnosis and sex
- The comparison table (figure_s6_embedding_f1_comparison.csv) contains one row per embedding-method × annotation combination
- AUCell module scores are obtained for every cell in the dataset from the Leiden-partitioned GRN subnetworks
- SCING embeddings retain biological information (diagnosis, sex, etc.) while correcting batch/sample effects, per Figure S6 expectations

## Limitations

- SCING module embeddings correct batch effects 'though it is not as optimized as those dedicated methods' (FastMNN, Harmony, Seurat)
- SCING GRN hyperparameters require selection and 'selection can be dataset dependent'
- Inferred GRN links are not necessarily causal or directional, so module embeddings reflect regulatory module activity rather than causal state

## Evidence

- [methods] AUCell module scoring: "We combine these subnetworks with the AUCell method to get module specific expression for each cell"
- [methods] Leiden partitioning: "We apply the leiden graph partitioning algorithm to identify GRN subnetworks"
- [abstract] batch-effect capacity: "have intrinsic capacity to correct for batch effects, retrieve disease relevant genes and pathways, and are informative on spatial specificity of disease pathogenesis"
- [readme] README notebook: "ModuleBasedDimensionalityReduction.ipynb: Take the merged network, and divide it into subnetworks with the leiden graph partitioning algorithm, finally run the AUCell approach to get module scores"
- [readme] pyscenic dependency: "If you want to use the AUCell from SCENIC for graph based dimensionality reduction you must install pyscenic"
