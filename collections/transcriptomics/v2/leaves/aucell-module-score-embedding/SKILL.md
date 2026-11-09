---
name: aucell-module-score-embedding
description: Use when a SCING GRN has been built and you need cell-level embeddings
  for clustering, phenotypic association, or batch-effect-corrected dimensionality
  reduction driven by regulatory modules rather than raw genes.
when_to_use_negative:
- You do not yet have a merged, pruned SCING GRN — build the network first with BuildNetwork.ipynb;
  AUCell here operates on GRN subnetworks, not on raw co-expression
- You only need maximally optimized batch correction — dedicated methods (FastMNN,
  Harmony, Seurat) outperform module embeddings on that axis alone
- You need explicitly causal or directional regulatory links — SCING-derived module
  links are not necessarily causal or directional
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCING
  role: Source of the merged GRN and of ModuleBasedDimensionalityReduction.ipynb,
    which performs Leiden partitioning and AUCell module scoring
  repo: https://github.com/XiaYangLabOrg/SCING
- name: SCENIC
  role: Provides the AUCell method used to compute module-specific expression scores
    per cell from the GRN subnetworks
- name: enrichr
  role: Optional downstream annotation of each Leiden-defined gene subnetwork via
    pathway enrichment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:7549da901a176e23025043409c9dfdb437be6871cfd05dcf8ebf0059203439fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - SCENIC
  - SCING
  - enrichr
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# aucell-module-score-embedding

## Summary

Generates cell embeddings from SCING gene regulatory network modules by partitioning the merged GRN into Leiden subnetworks and scoring each cell with AUCell (SCENIC), producing a module-score feature space that intrinsically corrects batch effects while retaining biological information.

## When to use

Use when a SCING GRN has been built and you need cell-level embeddings for clustering, phenotypic association, or batch-effect-corrected dimensionality reduction driven by regulatory modules rather than raw genes.

## When NOT to use

- You do not yet have a merged, pruned SCING GRN — build the network first with BuildNetwork.ipynb; AUCell here operates on GRN subnetworks, not on raw co-expression
- You only need maximally optimized batch correction — dedicated methods (FastMNN, Harmony, Seurat) outperform module embeddings on that axis alone
- You need explicitly causal or directional regulatory links — SCING-derived module links are not necessarily causal or directional

## Inputs

- Merged, pruned SCING GRN (adjacency/network file produced by BuildNetwork.ipynb)
- Processed single-cell/snRNA-seq expression matrix (e.g., human prefrontal cortex snRNAseq from AD and Control patients)
- Cell-level annotations (sample, batch, diagnosis, tangle stage, plaque stage, sex) for evaluation
- pyscenic installation (required for the AUCell approach)

## Outputs

- Leiden-partitioned GRN subnetworks (gene modules)
- AUCell module-score matrix (cells x modules) usable as cell embeddings
- Clusterings / phenotypic associations built on module-score embeddings
- Comparison tables (e.g., figure_s6_embedding_f1_comparison.csv) of neighborhood scores and phenotype F1 scores vs. FastMNN/Harmony/Seurat baselines

## How to apply

After merging and pruning the bagged GRNs (BuildNetwork.ipynb: supercells, subsampled GRNs, consensus edges present in 20% of networks, conditional mutual information pruning), run ModuleBasedDimensionalityReduction.ipynb: apply the Leiden graph partitioning algorithm to the merged network to divide it into GRN subnetworks (modules), then combine these subnetworks with the AUCell method from SCENIC to compute module-specific expression scores for every cell. The resulting module-score matrix serves as the cell embedding for downstream clustering, phenotypic association, and biological annotation via pathway enrichment (enrichr on each Leiden subnetwork's genes). In the Morabito human prefrontal cortex AD snRNA-seq application, these AUCell embeddings were benchmarked against FastMNN, Harmony, and Seurat corrected embeddings using a neighborhood score (Equation 1) on the first 20 PCs over neighbor fractions of 0.25%-16%, and a phenotype F1 score (Equation 2), showing comparable batch correction and biological signal retention though not as optimized as dedicated methods.

## Related tools

- **SCING** (Source of the merged GRN and of ModuleBasedDimensionalityReduction.ipynb, which performs Leiden partitioning and AUCell module scoring) — https://github.com/XiaYangLabOrg/SCING
- **SCENIC** (Provides the AUCell method used to compute module-specific expression scores per cell from the GRN subnetworks)
- **enrichr** (Optional downstream annotation of each Leiden-defined gene subnetwork via pathway enrichment)

## Examples

```
conda activate scing && pip install pyscenic  # then run ModuleBasedDimensionalityReduction.ipynb on the merged SCING network
```

## Evaluation signals

- The embedding matrix has one column per Leiden-partitioned GRN subnetwork (module) and one row per cell, with AUCell module scores as values
- Neighborhood score (Equation 1) computed on the first 20 PCs across neighbor fractions 0.25%-16% shows batch mixing comparable to FastMNN/Harmony/Seurat while retaining diagnosis/sex signal
- Phenotype F1 scores (Equation 2) from embeddings reproduce the Figure S6 comparison (e.g., rows in figure_s6_embedding_f1_comparison.csv)
- Pathway enrichment of each subnetwork's genes (enrichr) yields interpretable biological annotations for modules
- Clustering on module scores recovers cell types/phenotypic associations consistent with the GRN modules

## Limitations

- Module-embedding batch correction is similar to, but 'not as optimized as', dedicated batch correction methods such as FastMNN, Harmony, and Seurat
- Inferred GRN links feeding the modules are not necessarily causal or directional
- Many hyperparameters require selection, and while robustness is shown across a range, selection can be dataset dependent
- Spatial information is not explicitly utilized during SCING GRN construction, so module scores reflect no explicit spatial model

## Evidence

- [methods] Leiden partitioning + AUCell scoring: "We apply the leiden graph partitioning algorithm to identify GRN subnetworks. We combine these subnetworks with the AUCell method to get module specific expression for each cell"
- [readme] Module notebook description: "ModuleBasedDimensionalityReduction.ipynb: Take the merged network, and divide it into subnetworks with the leiden graph partitioning algorithm, finally run the AUCell approach to get module scores"
- [abstract] Intrinsic batch correction: "SCING GRNs reveal unique disease subnetwork modeling capabilities, have intrinsic capacity to correct for batch effects, retrieve disease relevant genes and pathways"
- [intro] Modules used for clustering: "SCING GRNs are then partitioned into modules, to compute module specific expression for each cell. These modules can be used for clustering, phenotypic association, and biological annotation through"
- [readme] pyscenic prerequisite: "If you want to use the AUCell from SCENIC for graph based dimensionality reduction you must install pyscenic"
