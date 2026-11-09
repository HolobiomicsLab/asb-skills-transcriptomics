---
name: mbic-cluster-number-selection
description: Use when fitting iSC.MEB to multi-sample SRT data and the number of spatial
  clusters K is unknown — e.g. grid-searching K=16:19 on six seqFISH embryo sections
  and picking the MBIC-minimizing K with c_penalty=0.2.
when_to_use_negative:
- K is already known or fixed by design (e.g. a benchmark calling iSCMEB(K=7) on DLPFC
  layers) — fit a single model rather than grid-selecting
- Input is the raw Seurat list without iSC.MEB prerequisites (CreateNeighbors adjacency
  plus runPCA embeddings); SelectModel cannot run on raw counts
- Single-sample or non-spatial analysis where iSC.MEB's multi-sample batch-effect
  integration is not needed
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3316
tools:
- name: iSC.MEB
  role: R package whose iSCMEB fit (over the K grid), SelectModel(criteria='MBIC',
    c_penalty), and SelectKPlot implement MBIC-based selection of the cluster number
  repo: https://github.com/XiaoZhangryy/iSC.MEB
- name: SC.MEB
  role: Single-sample empirical Bayes hidden Markov random field predecessor that
    iSC.MEB extends; its MBIC selection framework carries over to the multi-sample
    setting
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad019
    title: 'iSC.MEB: an R package for multi-sample spatial clustering analysis of
      spatial transcriptomics data'
  source_content_hash: sha256:b6c262ca70ea457fae0365ad31ef3e1fb3c4c012e29857acb53ce90e1f152c33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaoZhangryy/iSC.MEB
  tools:
  - BASS
  - BayesSpace
  - R
  - SC.MEB
  - SpaGCN
  - iSC.MEB
  - mclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# mbic-cluster-number-selection

## Summary

Selects the optimal number of spatial clusters K for iSC.MEB by fitting the hidden Markov random field with empirical Bayes over a candidate K grid (e.g. K=16:19 for the six seqFISH mouse embryo sections) and choosing the MBIC-minimizing K with a tuned c_penalty (0.2 in the seqFISH vignette).

## When to use

Use when fitting iSC.MEB to multi-sample SRT data and the number of spatial clusters K is unknown — e.g. grid-searching K=16:19 on six seqFISH embryo sections and picking the MBIC-minimizing K with c_penalty=0.2.

## When NOT to use

- K is already known or fixed by design (e.g. a benchmark calling iSCMEB(K=7) on DLPFC layers) — fit a single model rather than grid-selecting
- Input is the raw Seurat list without iSC.MEB prerequisites (CreateNeighbors adjacency plus runPCA embeddings); SelectModel cannot run on raw counts
- Single-sample or non-spatial analysis where iSC.MEB's multi-sample batch-effect integration is not needed

## Inputs

- iSCMEBObj holding APCA low-dimensional embeddings (npcs=15) for multiple SRT sections
- Spatial adjacency matrices built with CreateNeighbors (e.g. radius.upper=10)
- Candidate cluster-number grid (e.g. K=16:19 for seqFISH) and an MBIC c_penalty value

## Outputs

- MBIC-selected optimal K (saved with labels in iscmeb_cluster_labels.rds)
- Fitted cluster labels per spot/cell extractable with idents()
- MBIC selection curve from SelectKPlot(criteria='MBIC')

## How to apply

After runPCA (npcs=15, pca.method='APCA') and CreateNeighbors (radius.upper=10), configure the ICM-EM optimizer with SetModelParameters (maxIter_ICM=4, maxIter=20, coreNum=4). Fit iSCMEB across the candidate K grid — e.g. K=16:19 for the six-section seqFISH embryo data or K=7 for the DLPFC data — where each fit also grid-searches the smoothness parameter beta. Call SelectModel(criteria='MBIC', c_penalty=...) to compute MBIC over the grid and take the K at its minimum; tune c_penalty large (5-10) when K<10, 0.5-2 for many clusters, and 0.2 for the seqFISH vignette setting. Inspect the selection curve with SelectKPlot(criteria='MBIC'), then extract fitted labels with idents() and save them with the MBIC-selected K (e.g. iscmeb_cluster_labels.rds). Correctness is judged by an interior MBIC minimum on the SelectKPlot curve and, for seqFISH, downstream integrated ARI/NMI of (0.52, 0.69) versus BASS (0.45, 0.66).

## Related tools

- **iSC.MEB** (R package whose iSCMEB fit (over the K grid), SelectModel(criteria='MBIC', c_penalty), and SelectKPlot implement MBIC-based selection of the cluster number) — https://github.com/XiaoZhangryy/iSC.MEB
- **SC.MEB** (Single-sample empirical Bayes hidden Markov random field predecessor that iSC.MEB extends; its MBIC selection framework carries over to the multi-sample setting)

## Examples

```
iSCMEBObj <- iSCMEB(iSCMEBObj, K = 16:19); iSCMEBObj <- SelectModel(iSCMEBObj, criteria = "MBIC", c_penalty = 0.2); SelectKPlot(iSCMEBObj, criteria = "MBIC")
```

## Evaluation signals

- SelectModel returns a K inside the candidate grid (one of K=16:19 on seqFISH), and the SelectKPlot(criteria='MBIC') curve shows an interior minimum rather than an edge value
- idents() on the fitted object returns labels for every spot across all sections with exactly the selected number of cluster levels
- On the seqFISH benchmark, downstream metrics recover integrated ARI/NMI of (0.52, 0.69) versus BASS (0.45, 0.66) and the highest median ARI/NMI among SC.MEB, BASS, BayesSpace and SpaGCN
- Re-running SelectModel with a different c_penalty (5-10 when K<10; 0.5-2 for many clusters) moves the selected K consistently, confirming the penalty controls the trade-off
- The saved iscmeb_cluster_labels.rds contains both the per-spot labels and the MBIC-selected K value

## Limitations

- c_penalty must be hand-tuned per dataset (large 5-10 when K<10, 0.5-2 for many clusters, 0.2 in the seqFISH vignette); a poorly chosen value biases K selection
- MBIC selection requires refitting iSCMEB for every candidate K, each fit grid-searching beta under ICM-EM (maxIter_ICM=4, maxIter=20, coreNum=4), so compute cost scales with grid size
- On Linux, parallel runs need 'ulimit -s unlimited' to avoid 'R Error: C stack usage is too close to the limit'; installation also needs Rcpp/RcppArmadillo with Rtools, Xcode or gfortran configured

## Evidence

- [methods] SelectModel with MBIC criterion and c_penalty on the fitted object: "iSCMEBObj <- SelectModel(iSCMEBObj, criteria="MBIC", c_penalty = 0.2)"
- [methods] ICM-EM optimizer settings configured before the K grid search: "iSCMEBObj <- SetModelParameters(iSCMEBObj, verbose = FALSE, maxIter_ICM = 4, maxIter = 20, coreNum=4)"
- [readme] iSC.MEB definition as an empirical Bayes hidden Markov random field method: "iSC.MEB: Integrate spatial clustering analysis with hidden Markov random field using empirical Bayes."
- [abstract] iSC.MEB as an extension of SC.MEB establishing the model-selection framework: "we introduce an extension to SC.MEB, denoted as integrated spatial clustering with hidden Markov random field using empirical Bayes (iSC.MEB)"
