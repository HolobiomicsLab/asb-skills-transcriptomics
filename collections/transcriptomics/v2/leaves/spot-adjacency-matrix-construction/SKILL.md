---
name: spot-adjacency-matrix-construction
description: Use when preparing SRT data (e.g., Visium DLPFC sections) for spatial
  clustering methods that need a neighbor graph — iSC.MEB requires adjacency matrices
  derived from spot coordinates before model fitting.
when_to_use_negative:
- Data lack spatial coordinates (e.g., single-cell data without spatial layout), so
  no neighbor graph can be defined
- The downstream clustering method does not require an adjacency matrix (e.g., methods
  run on expression alone)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: iSC.MEB
  role: Provides CreateNeighbors, which constructs the spot adjacency matrix list
    from Visium spot coordinates and stores it in the iSCMEBObj
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: R
  role: Open-source environment in which iSC.MEB and the adjacency construction are
    executed
- name: Seurat
  role: Supplies the Seurat objects (seuList) carrying per-spot spatial coordinates
    used to derive neighbors
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:d475e27cc98b97a0de3a4afbbafea8d6259336a132fa352ecd2b785e5d34a22f
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# spot-adjacency-matrix-construction

## Summary

Constructs spot adjacency matrices from spatial transcriptomics spot coordinates (e.g., Visium platforms) as required input for spatial clustering models like iSC.MEB, encoding which spots are spatial neighbors.

## When to use

Use when preparing SRT data (e.g., Visium DLPFC sections) for spatial clustering methods that need a neighbor graph — iSC.MEB requires adjacency matrices derived from spot coordinates before model fitting.

## When NOT to use

- Data lack spatial coordinates (e.g., single-cell data without spatial layout), so no neighbor graph can be defined
- The downstream clustering method does not require an adjacency matrix (e.g., methods run on expression alone)

## Inputs

- iSCMEBObj object created by CreateiSCMEBObject from a seuList of Seurat objects (e.g., 12-section DLPFC 10x Visium data from DLPFC.rda)
- Spot spatial coordinates per section (embedded in the Seurat objects)
- Platform specification string, e.g., 'Visium'

## Outputs

- iSCMEBObj updated with a list of spot adjacency matrices, one per section
- Per-section adjacency matrix whose dimensions equal the number of spots of that section (extractable for downstream use, e.g., saved to dlpfc_pcs_adjacency.rds)

## How to apply

After building the iSCMEBObj with CreateiSCMEBObject (e.g., from a seuList of Seurat objects loaded from DLPFC.rda with premin.spots=0, postmin.spots=0), call CreateNeighbors(iSCMEBObj, platform='Visium') to derive neighbors from the Visium spot coordinates. The platform argument controls how the neighbor graph is built (Visium's hexagonal spot layout vs. other platforms; a radius-based variant with radius.upper=10 is also shown). Store the resulting adjacency matrix list in the iSCMEBObj slots; per-section adjacency matrices should have dimensions equal to that section's number of spots. These adjacency matrices are then used alongside the 15-PC embedding from runPCA (npcs=15, pca.method='APCA') as low-dimensional representations for the hidden Markov random field model.

## Related tools

- **iSC.MEB** (Provides CreateNeighbors, which constructs the spot adjacency matrix list from Visium spot coordinates and stores it in the iSCMEBObj) — https://github.com/XiaoZhangryy/iSC.MEB
- **R** (Open-source environment in which iSC.MEB and the adjacency construction are executed)
- **Seurat** (Supplies the Seurat objects (seuList) carrying per-spot spatial coordinates used to derive neighbors)

## Examples

```
iSCMEBObj <- CreateNeighbors(iSCMEBObj, platform = "Visium")
```

## Evaluation signals

- The resulting adjacency matrix dimensions equal the number of spots of the selected DLPFC section
- CreateNeighbors is called with the correct platform argument ('Visium' for 10x Visium DLPFC data)
- The adjacency matrix (optionally with the 15-column PC embedding from runPCA npcs=15) can be extracted from iSCMEBObj slots and saved, e.g., to dlpfc_pcs_adjacency.rds
- All spots are retained for the vignette settings (premin.spots=0, postmin.spots=0), so no spots are dropped during object construction before adjacency derivation

## Limitations

- Neighbor derivation is platform-dependent — the 'Visium' platform assumes the hexagonal Visium spot layout; other platforms or radius-based settings (radius.upper=10) must be chosen appropriately
- Linux users running parallel computation may hit 'R Error: C stack usage is too close to the limit' and must run 'ulimit -s unlimited'; missing dependencies like scater may need conda (bioconda) or BiocManager installation

## Evidence

- [methods] CreateNeighbors on Visium coordinates: "iSCMEBObj <-  CreateNeighbors(iSCMEBObj, platform = "Visium")"
- [methods] CreateiSCMEBObject retains all spots in the vignette: "iSCMEBObj <- CreateiSCMEBObject(seuList = seuList, verbose = FALSE, premin.spots=0, postmin.spots=0)"
- [methods] Radius-based alternative adjacency call: "iSCMEBObj <-  CreateNeighbors(iSCMEBObj, radius.upper=10)"
- [methods] DLPFC data download context for adjacency construction: "githubURL <- "https://github.com/XiaoZhangryy/iSC.MEB/blob/main/vignettes_data/DLPFC.rda?raw=true""
- [readme] README describes the package's purpose of integrating multiple SRT datasets: "iSC.MEB is a package for integrating and analyzing multiple spatially resolved transcriptomics (SRT) datasets"
