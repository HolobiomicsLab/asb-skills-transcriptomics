---
name: potts-model-smoothness-grid-search
description: Use when multiple SRT sections need integrated spatial clustering and
  the number of domains K is unknown — e.g. after APCA reduction (npcs = 15) and CreateNeighbors
  adjacency, with batch effects to estimate jointly.
when_to_use_negative:
- The number of spatial domains K is already known and fixed — call iSCMEB with a
  single K instead of grid-searching.
- Input is raw count data without prior dimensionality reduction and neighbor adjacency
  (runPCA with APCA and CreateNeighbors must come first).
- Only one SRT section with no batch structure — the single-sample SC.MEB / DR.SC
  setting is the appropriate scope instead of iSC.MEB.
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: iSC.MEB
  role: R package implementing the hidden Markov random field model with empirical
    Bayes; the iSCMEB function fits the model and grid-searches beta over candidate
    K values
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: SC.MEB
  role: single-sample predecessor using a hidden Markov random field with empirical
    Bayes, extended by iSC.MEB for multi-sample data
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: R
  role: open-source language in which iSC.MEB is implemented and run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:9c386c2adfb16627e13828d61a6185a33af55baa72b27484ebbcf3dde7a2c686
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaoZhangryy/iSC.MEB
  tools:
  - R
  - SC.MEB
  - iSC.MEB
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# potts-model-smoothness-grid-search

## Summary

Fits the iSC.MEB hidden-Markov-random-field model with empirical Bayes via ICM-EM over a grid of candidate cluster numbers (e.g. K = 16:19), which grid-searches the Potts smoothness parameter beta, then selects K by MBIC with a tuned c_penalty.

## When to use

Use when multiple SRT sections need integrated spatial clustering and the number of domains K is unknown — e.g. after APCA reduction (npcs = 15) and CreateNeighbors adjacency, with batch effects to estimate jointly.

## When NOT to use

- The number of spatial domains K is already known and fixed — call iSCMEB with a single K instead of grid-searching.
- Input is raw count data without prior dimensionality reduction and neighbor adjacency (runPCA with APCA and CreateNeighbors must come first).
- Only one SRT section with no batch structure — the single-sample SC.MEB / DR.SC setting is the appropriate scope instead of iSC.MEB.

## Inputs

- iSCMEBObj with APCA low-dimensional embeddings (npcs = 15) in its reductions
- Spatial adjacency matrices built with CreateNeighbors (platform-specific, e.g. Visium)
- Candidate cluster-number grid (e.g. K = 16:19) and MBIC c_penalty value

## Outputs

- Fitted iSCMEBObj with cluster labels retrievable via idents()
- MBIC-selected optimal K and the MBIC selection curve (SelectKPlot)
- RDS of cluster labels with selected K (e.g. iscmeb_cluster_labels.rds)

## How to apply

Load the iSCMEBObj carrying APCA low-dimensional embeddings (npcs = 15) and spatial adjacency matrices built with CreateNeighbors, then configure ICM-EM settings with SetModelParameters (maxIter_ICM = 4, maxIter = 20, coreNum = 4). Run the iSCMEB fit over a grid of candidate cluster numbers (e.g. K = 16:19) rather than a single K, because fitting over the grid also searches the smoothness parameter beta of the Potts/hidden Markov random field. Inside ICM-EM, alternate between estimating the cluster label yti by maximizing its posterior given the batch-related embedding uti, and estimating uti by maximizing its posterior given yti. Pick the optimal K with SelectModel(criteria = 'MBIC'), tuning c_penalty (large, e.g. 5–10 when K < 10; 0.5–2 for many clusters; 0.2 used in the seqFISH vignette) and inspect the selection curve with SelectKPlot(criteria = 'MBIC'). Extract fitted labels with idents() and save them together with the MBIC-selected K.

## Related tools

- **iSC.MEB** (R package implementing the hidden Markov random field model with empirical Bayes; the iSCMEB function fits the model and grid-searches beta over candidate K values) — https://github.com/XiaoZhangryy/iSC.MEB
- **SC.MEB** (single-sample predecessor using a hidden Markov random field with empirical Bayes, extended by iSC.MEB for multi-sample data) — https://github.com/XiaoZhangryy/iSC.MEB
- **R** (open-source language in which iSC.MEB is implemented and run)

## Examples

```
iSCMEBObj <- iSCMEB(iSCMEBObj, K = 16:19); iSCMEBObj <- SelectModel(iSCMEBObj, criteria = "MBIC", c_penalty = 0.2)
```

## Evaluation signals

- The MBIC curve over the K grid (SelectKPlot with criteria = 'MBIC') shows a clear minimum at the selected K
- The ICM-EM iterations converge within the set maxIter_ICM = 4 and maxIter = 20 without errors (set coreNum = 4 for parallelism)
- Extracted labels via idents() have exactly the MBIC-selected number of clusters K and cover all spots across sections
- Clustering agreement with manual domain annotations can be quantified with ARI
- Spatial coherence of labels is visible in SpaHeatMap scatter plots of item = 'cluster'

## Limitations

- c_penalty tuning is required and regime-dependent: large (5–10) when K < 10, 0.5–2 for many clusters; the seqFISH vignette used 0.2
- On Linux, parallel computation can hit 'R Error: C stack usage is too close to the limit' — requires `ulimit -s unlimited`
- iSC.MEB assumes partially shared cell/domain clusters across samples; it is designed for multi-sample integration, not single-slide analysis

## Evidence

- [methods] In the ICM step of the ICM-EM algorithm, iSC.MEB alternates between estimating the cluster label yti by maximizing its posterior given the batch-related embedding uti, and estimating the batch-related embedding uti by maximizing its posterior given the label yti.: [evidence span withheld — anchor status: not_found]
- [abstract] permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets: "permits the users to simultaneously estimate the batch effect and perform spatial clustering for low-dimensional representations of multiple SRT datasets"
- [methods] iSCMEBObj <- SelectModel(iSCMEBObj, criteria="MBIC", c_penalty = 0.2): "iSCMEBObj <- SelectModel(iSCMEBObj, criteria="MBIC", c_penalty = 0.2)"
- [methods] iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4): "iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4)"
- [abstract] We have previously developed the tool SC.MEB-an empirical Bayes approach for SRT data analysis using a hidden Markov random field: "We have previously developed the tool SC.MEB-an empirical Bayes approach for SRT data analysis using a hidden Markov random field"
