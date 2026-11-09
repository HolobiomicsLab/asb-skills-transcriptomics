---
name: multi-sample-batch-effect-integration
description: Use when you have ≥2 SRT sections (e.g. the 12-sample DLPFC 10x Visium
  set) with batch/individual effects and need joint spatial domain detection or cross-sample
  integrated clustering.
when_to_use_negative:
- You have a single SRT section only — the predecessor SC.MEB fits one sample and
  iSC.MEB's batch-effect estimation is unnecessary
- Your datasets have no (fully) shared cell/domain clusters and you are not comparing
  across batches, individuals, or conditions — integration machinery is not the right
  fit
- Input is already batch-corrected/integrated features rather than per-sample raw
  count Seurat objects requiring CreateiSCMEBObject
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: iSC.MEB
  role: Fits the HMRF empirical Bayes model across samples, estimates batch effect,
    and exports the integrated Seurat object via IntegrateSpaData
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: SC.MEB
  role: Predecessor single-sample empirical Bayes spatial clustering method that iSC.MEB
    extends to multi-sample integration
- name: Seurat
  role: Provides the per-section Seurat objects consumed as seuList and the integrated
    output object seuInt
- name: mclust
  role: adjustedRandIndex computes ARI for judging integration/clustering quality
    against manual annotations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:0811a563fe328a00d1f60b1ee2d77e32560c9aaf9debef2afaac0803b8eec154
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaoZhangryy/iSC.MEB
  tools:
  - R
  - SC.MEB
  - Seurat
  - iSC.MEB
  - mclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# multi-sample-batch-effect-integration

## Summary

Integrates multiple spatially resolved transcriptomics (SRT) samples by fitting the iSC.MEB hidden Markov random field empirical Bayes model, which simultaneously estimates batch effects and performs spatial clustering on shared low-dimensional representations, then exports an integrated Seurat object via IntegrateSpaData.

## When to use

Use when you have ≥2 SRT sections (e.g. the 12-sample DLPFC 10x Visium set) with batch/individual effects and need joint spatial domain detection or cross-sample integrated clustering.

## When NOT to use

- You have a single SRT section only — the predecessor SC.MEB fits one sample and iSC.MEB's batch-effect estimation is unnecessary
- Your datasets have no (fully) shared cell/domain clusters and you are not comparing across batches, individuals, or conditions — integration machinery is not the right fit
- Input is already batch-corrected/integrated features rather than per-sample raw count Seurat objects requiring CreateiSCMEBObject

## Inputs

- seuList: list of Seurat objects, one per SRT section (e.g. 12 human DLPFC 10x Visium sections from DLPFC.rda)
- Number of shared clusters K (e.g. K=7 for the 7 DLPFC cortical layers)
- Platform string for adjacency construction (e.g. "Visium", "mouse" species tag for integration)

## Outputs

- iSCMEBObj with fitted iSC.MEB clusters, per-spot labels accessible via idents(), and estimated batch effects
- seuInt: integrated Seurat object returned by IntegrateSpadata with batch-corrected low-dimensional embeddings
- Per-sample and pooled ARI/NMI values (mclust::adjustedRandIndex) against manual annotations, e.g. dlpfc_ari_nmi_results.csv

## How to apply

Download the pre-filtered seuList of Seurat objects (e.g. DLPFC.rda from the XiaoZhangryy/iSC.MEB vignettes_data URL) and build an iSCMEBObj with CreateiSCMEBObject(seuList, premin.spots=0, postmin.spots=0). Build platform-specific adjacency with CreateNeighbors(platform="Visium") and reduce each sample to 15 PCs using adjusted PCA (runPCA(npcs=15, pca.method="APCA")), which gives the low-dimensional representation the model joint-fits while estimating the batch effect. Fit the model with iSCMEB(iSCMEBObj, K=7) where K matches the number of shared cell/domain clusters (7 cortical layers for DLPFC), finalize labels with SelectModel, then call IntegrateSpaData(iSCMEBObj, species) to obtain an integrated Seurat object (seuInt) carrying batch-corrected embeddings. Rationale: unlike fitting SC.MEB per section, the HMRF empirical Bayes model is fit across samples jointly, so clusters are comparable across batches and integration quality can be scored by pooled ARI/NMI against manual annotations (layer_guess_reordered) using mclust::adjustedRandIndex.

## Related tools

- **iSC.MEB** (Fits the HMRF empirical Bayes model across samples, estimates batch effect, and exports the integrated Seurat object via IntegrateSpaData) — https://github.com/XiaoZhangryy/iSC.MEB
- **SC.MEB** (Predecessor single-sample empirical Bayes spatial clustering method that iSC.MEB extends to multi-sample integration)
- **Seurat** (Provides the per-section Seurat objects consumed as seuList and the integrated output object seuInt)
- **mclust** (adjustedRandIndex computes ARI for judging integration/clustering quality against manual annotations)

## Examples

```
iSCMEBObj <- iSCMEB(iSCMEBObj, K= 7); seuInt <- IntegrateSpaData(iSCMEBObj, "mouse")
```

## Evaluation signals

- Integrated ARI/NMI on the 12 DLPFC sections should reach ≈0.48/0.57 against layer_guess_reordered, with the highest median ARI/NMI among BASS, SC.MEB, BayesSpace, SpaGCN, Louvain
- IntegrateSpaData must return a non-null Seurat object (seuInt) with a batch-corrected reduction (e.g. named 'mouse' species tag in the vignette call) suitable for doDEG/doHeatmap
- Per-spot cluster labels exist after SelectModel (check via idents()) and cluster count matches K=7 shared domains
- Adjacency matrices were built with the correct platform (CreateNeighbors(platform="Visium")) — mis-set platform invalidates the HMRF neighborhood structure
- Dimensionality reduction used npcs=15 with pca.method="APCA"; deviations from the vignette settings change the low-dimensional input to the model

## Limitations

- Compilation requires devtools plus Rcpp/RcppArmadillo and correct Rtools (Windows) or Xcode (Mac OS/X) setup before install_github
- On Linux, parallel computation requires `ulimit -s unlimited` to avoid 'R Error: C stack usage is too close to the limit'; conda users may need bioconda packages such as bioconductor-scater
- K must correspond to shared cell/domain clusters; the method is aimed at datasets with partially shared clusters, and performance depends on the number of sections supplied (vignette uses 12 DLPFC or 6 seqFISH mouse embryo sections)

## Evidence

- [abstract] permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets: "permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets"
- [methods] seuInt <- IntegrateSpaData(iSCMEBObj, "mouse"): "seuInt <- IntegrateSpaData(iSCMEBObj, "mouse")"
- [methods] iSCMEBObj <- runPCA(iSCMEBObj, npcs = 15, pca.method = "APCA"): "iSCMEBObj <-  runPCA(iSCMEBObj, npcs = 15, pca.method = "APCA")"
- [readme] iSC.MEB is a package for integrating and analyzing multiple spatially resolved transcriptomics (SRT) datasets, developed by the Jin Liu's lab.: "iSC.MEB is a package for integrating and analyzing multiple spatially resolved transcriptomics (SRT) datasets, developed by the Jin Liu's lab."
- [intro] iSC.MEB can be used to compare and contrast experimental datasets in a variety of contexts: "iSC.MEB can be used to compare and contrast experimental datasets in a variety of contexts"
