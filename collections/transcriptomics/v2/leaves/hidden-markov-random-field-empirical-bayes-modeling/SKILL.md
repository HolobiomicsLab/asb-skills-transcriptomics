---
name: hidden-markov-random-field-empirical-bayes-modeling
description: Use when integrating and spatially clustering multiple SRT sections (e.g.,
  12-sample DLPFC Visium or 6-section mouse embryo seqFISH) where batch effects must
  be estimated jointly with domain/cell-type detection.
when_to_use_negative:
- Single-sample SRT data where simpler SC.MEB or BayesSpace suffice — iSC.MEB's joint
  batch-effect estimation targets multi-sample integration
- Datasets lacking spatial coordinates or adjacency structure, since CreateNeighbors
  requires spatial locations to build the Markov random field
- Analyses needing per-section independent clustering with no shared clusters or batch-effect
  correction across samples
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: iSC.MEB
  role: R package implementing the hidden Markov random field empirical Bayes model
    for multi-sample spatial clustering (CreateiSCMEBObject, CreateNeighbors, runPCA,
    iSCMEB, SelectModel)
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: SC.MEB
  role: Predecessor single-sample empirical Bayes spatial clustering method that iSC.MEB
    extends
- name: mclust
  role: Provides adjustedRandIndex for computing ARI against manual annotations
- name: Seurat
  role: Container format (Seurat objects) for the input seuList of SRT sections
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:f3c118e240303ab5dc5a1d9635fa2fc55e9dc3aff405e8e67c7ecad782cc6bbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaoZhangryy/iSC.MEB
  tools:
  - BASS
  - BayesSpace
  - R
  - SC.MEB
  - Seurat
  - SpaGCN
  - iSC.MEB
  - mclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# hidden-markov-random-field-empirical-bayes-modeling

## Summary

Fit the iSC.MEB hidden Markov random field empirical Bayes model to jointly estimate batch effects and perform spatial clustering across multiple spatially resolved transcriptomics (SRT) sections, as implemented in the iSC.MEB R package.

## When to use

Use when integrating and spatially clustering multiple SRT sections (e.g., 12-sample DLPFC Visium or 6-section mouse embryo seqFISH) where batch effects must be estimated jointly with domain/cell-type detection.

## When NOT to use

- Single-sample SRT data where simpler SC.MEB or BayesSpace suffice — iSC.MEB's joint batch-effect estimation targets multi-sample integration
- Datasets lacking spatial coordinates or adjacency structure, since CreateNeighbors requires spatial locations to build the Markov random field
- Analyses needing per-section independent clustering with no shared clusters or batch-effect correction across samples

## Inputs

- List of Seurat objects (seuList) for multiple SRT sections, e.g., DLPFC.rda (12 human DLPFC Visium samples) or seqFISH.rda (6 mouse embryo sections)
- Custom gene list for feature selection (seqFISH workflow)
- Reference labels for evaluation: layer_guess_reordered (DLPFC) or celltype_mapped_refined (seqFISH)

## Outputs

- iSCMEBObj with fitted spatial cluster labels per spot/cell (accessible via idents())
- Per-section and pooled (integrated) ARI/NMI metrics, e.g., integrated ARI/NMI of (0.48, 0.57) on DLPFC and (0.52, 0.69) on seqFISH
- Low-dimensional iSC.MEB embedding usable for CalculateTSNE and IntegrateSpaData

## How to apply

Download the preprocessed multi-sample data (e.g., DLPFC.rda or seqFISH.rda from the iSC.MEB vignettes_data directory) and load the list of Seurat objects (seuList). Build the object with CreateiSCMEBObject, construct adjacency matrices with CreateNeighbors (Visium platform, or L2-distance-based with radius.upper=10 for seqFISH), and reduce dimensionality with runPCA using npcs=15 and pca.method='APCA'. Configure fitting via SetModelParameters (maxIter_ICM=4, maxIter=20, coreNum=4), then fit the hidden Markov random field model with iSCMEB at a chosen K (e.g., K=7 for the 7 cortical DLPFC layers, or K=16:19 for seqFISH) and finalize labels with SelectModel (criteria='MBIC', c_penalty=0.2). Evaluate with mclust::adjustedRandIndex (ARI) and NMI against manual annotations (e.g., layer_guess_reordered or celltype_mapped_refined), computing both per-section and pooled integrated metrics.

## Related tools

- **iSC.MEB** (R package implementing the hidden Markov random field empirical Bayes model for multi-sample spatial clustering (CreateiSCMEBObject, CreateNeighbors, runPCA, iSCMEB, SelectModel)) — https://github.com/XiaoZhangryy/iSC.MEB
- **SC.MEB** (Predecessor single-sample empirical Bayes spatial clustering method that iSC.MEB extends)
- **mclust** (Provides adjustedRandIndex for computing ARI against manual annotations)
- **Seurat** (Container format (Seurat objects) for the input seuList of SRT sections)

## Examples

```
iSCMEBObj <- iSCMEB(iSCMEBObj, K=7); iSCMEBObj <- SelectModel(iSCMEBObj)
```

## Evaluation signals

- Integrated ARI/NMI against manual annotations: expect ~ (0.48, 0.57) on the 12-sample DLPFC dataset and ~ (0.52, 0.69) vs BASS's (0.45, 0.66) on the 6-section seqFISH dataset
- iSC.MEB should attain the highest median ARI and NMI among compared methods (SC.MEB, BASS, BayesSpace, SpaGCN, Louvain on DLPFC)
- SelectModel with criteria='MBIC' and c_penalty=0.2 converges on a cluster number within the candidate range (e.g., K=16:19 for seqFISH, K=7 for DLPFC cortical layers)
- Per-section cluster spatial maps (SpaHeatMap) form spatially contiguous domains consistent with known tissue structure

## Limitations

- Requires system-dependent compilation setup: Rcpp/RcppArmadillo need Rtools (Windows) or Xcode and gfortran (Mac OS/X)
- On Linux, parallel computation requires 'ulimit -s unlimited' to avoid 'R Error: C stack usage is too close to the limit'
- Dependent packages (e.g., scater) may fail to install in conda environments and require bioconda or BiocManager fallbacks

## Evidence

- [abstract] we introduce an extension to SC.MEB, denoted as integrated spatial clustering with hidden Markov random field using empirical Bayes (iSC.MEB): "we introduce an extension to SC.MEB, denoted as integrated spatial clustering with hidden Markov random field using empirical Bayes (iSC.MEB)"
- [abstract] permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets: "permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets"
- [methods] iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4): "iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4)"
- [methods] iSCMEBObj <- SelectModel(iSCMEBObj, criteria="MBIC", c_penalty = 0.2): "iSCMEBObj <- SelectModel(iSCMEBObj, criteria="MBIC", c_penalty = 0.2)"
- [readme] For parallel computation on Linux, users must use the following system command to set the C_stack unlimited in case of the error `R Error: C stack usage is too close to the limit`.: [evidence span withheld — anchor status: external_doc]
