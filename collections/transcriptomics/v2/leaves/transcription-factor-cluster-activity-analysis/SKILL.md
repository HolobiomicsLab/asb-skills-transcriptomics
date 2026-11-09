---
name: transcription-factor-cluster-activity-analysis
description: Use when a time-course single-cell dataset needs TF activity profiles
  aligned to ligand-receptor dynamics — e.g. to test whether co-varying LR pairs are
  causally linked to temporal TF clusters.
when_to_use_negative:
- When the goal is paracrine communication between defined cell types rather than
  autocrine/temporal LR-to-TF linkage — CellPhoneDB/CellChat-style tools are the intended
  choice for given cell types
- 'When studying maternal-embryo interactions: TimeTalk''s autocrine screening hypothesis
  precludes this analysis'
- When high-confidence LR-TF regulatory confirmation is required — the article states
  these inferred relationships need wet-lab confirmation and Granger conclusions are
  winsz-sensitive
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TimeTalk
  role: R package whose RunTimeTalk function consumes the MRA result (tmp.mra.res)
    and performs tTF-cluster-based Granger causal classification of eLRs
  repo: https://github.com/ChengLiLab/TimeTalk
- name: RTN
  role: Builds transcriptional network ('tni') objects and runs tna.mra master regulator
    analysis to identify tTFs from variable genes and their variance as phenotype
- name: cellAlign
  role: Interpolates the aligned LR-score and TF-cluster-activity time series (tmp.winsz
    = 0.1, tmp.lags = 1, numPts = 200) before Granger testing
  repo: https://github.com/shenorrLab/cellAlign
- name: lmtest
  role: Provides grangertest used across candidate lags to classify LR-tTF-cluster
    direction (LR->TF, TF->LR, or both) with p < 0.01 cutoff
- name: monocle3
  role: RunMonocle3 reconstructs the trajectory whose variable genes (GetVariableGeneInTrajectory)
    define the phenotype input for tTF MRA
  repo: https://github.com/cole-trapnell-lab/monocle3
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:d85dcc90e232ac0156c2ae19790e0be8db5d5754bdc10343b7140ef15c80f7f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - RTN
  - TimeTalk
  - cellAlign
  - lmtest
  - monocle3
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# transcription-factor-cluster-activity-analysis

## Summary

Identifies temporal transcription factors (tTFs) by master regulator analysis on cell-type-specific TRNs, clusters them by ward.D2 hierarchical clustering, and derives TF-cluster activity profiles (average expression per cluster over pseudotime) that feed TimeTalk's Granger causal testing of eLRs, yielding 430 eLRs and 229 tTFs.

## When to use

Use when a time-course single-cell dataset needs TF activity profiles aligned to ligand-receptor dynamics — e.g. to test whether co-varying LR pairs are causally linked to temporal TF clusters.

## When NOT to use

- When the goal is paracrine communication between defined cell types rather than autocrine/temporal LR-to-TF linkage — CellPhoneDB/CellChat-style tools are the intended choice for given cell types
- When studying maternal-embryo interactions: TimeTalk's autocrine screening hypothesis precludes this analysis
- When high-confidence LR-TF regulatory confirmation is required — the article states these inferred relationships need wet-lab confirmation and Granger conclusions are winsz-sensitive

## Inputs

- Seurat object (Seurat 3.2.3) with normalized scRNA-seq expression and cell-type annotations (e.g. EPI/PE blastoid vs blastocyst)
- monocle3 cell_data_set (cds) with trajectory/pseudotime from RunMonocle3
- TF list from AnimalTFDB version 3.0
- RTN 'tna' objects per cell type (myGetCellTypeSpecificTRN output)
- Marker-gene table from FindAllMarkers (p_val_adj < 0.05 per cluster)
- Co-varying LR score table from TimeTalk (SCC = 0.2 cutoff)

## Outputs

- tTF cluster assignment (ward.D2 hierarchical clustering; six clusters, 229 tTFs total)
- TF-cluster activity profiles (average expression per tTF cluster across aligned pseudotime)
- Interpolated TF-activity time series from cellAlign (numPts = 200)
- Granger classification table (eLR_granger_results.csv) labeling eLRs as forward/backward/feedback relative to tTF clusters at p < 0.01

## How to apply

First build cell-type-specific TRNs with RTN (AnimalTFDB v3.0 TF list, log10(RPKM+1) normalized data as 'tni' object) and run master regulator analysis with 'tna.mra', keeping regulators with Pvalue < 0.05 whose Regulon overlaps per-cluster FindAllMarkers genes at p_val_adj < 0.05. The variable genes and their variance are input as a phenotype for the MRA to define the tTFs. Cluster the identified tTFs by hierarchical clustering with the ward.D2 agglomeration method, producing six tTF clusters that correspond to the six temporal windows of eLRs. Use the average expression of each cluster's members as the TF-cluster activity time series. Before Granger testing, interpolate the aligned TF-activity and LR-score series with cellAlign (tmp.winsz = 0.1, tmp.lags = 1, numPts = 200) to obtain smooth temporal series; downstream classification retains eLRs passing tmp.granger.cutoff = 1e-2.

## Related tools

- **TimeTalk** (R package whose RunTimeTalk function consumes the MRA result (tmp.mra.res) and performs tTF-cluster-based Granger causal classification of eLRs) — https://github.com/ChengLiLab/TimeTalk
- **RTN** (Builds transcriptional network ('tni') objects and runs tna.mra master regulator analysis to identify tTFs from variable genes and their variance as phenotype)
- **cellAlign** (Interpolates the aligned LR-score and TF-cluster-activity time series (tmp.winsz = 0.1, tmp.lags = 1, numPts = 200) before Granger testing) — https://github.com/shenorrLab/cellAlign
- **lmtest** (Provides grangertest used across candidate lags to classify LR-tTF-cluster direction (LR->TF, TF->LR, or both) with p < 0.01 cutoff)
- **monocle3** (RunMonocle3 reconstructs the trajectory whose variable genes (GetVariableGeneInTrajectory) define the phenotype input for tTF MRA) — https://github.com/cole-trapnell-lab/monocle3

## Examples

```
TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small, tmp.seu=seu.small, LRpairs.df=LRpairs.df, tmp.mra.res=tmp.mra.res, tmp.winsz=0.1, tmp.lags=1, numPts=200, tmp.SCC.cutoff=0.2, tmp.granger.cutoff=1e-2)
```

## Evaluation signals

- Output table must contain 430 eLRs classified at the Granger p < 0.01 cutoff with forward/backward/feedback labels, matching Fig. 2d-e
- tTF set must total 229 TFs partitioned into six temporal-window clusters after ward.D2 clustering
- MRA result must be filtered to regulators with Pvalue < 0.05 whose Regulon is in the cluster's FindAllMarkers genes (p_val_adj < 0.05)
- Downstream eLR list must retain known examples such as Fgf4-Fgfr1 and Fgf4-Fgfr2
- Granger classification must be re-checked at alternative winsz values, since the conclusion is documented as sensitive to this parameter

## Limitations

- Granger causality conclusions are sensitive to the interpolation parameter winsz (Supplementary Figs. 20 and 21); users are advised to test multiple parameter combinations
- Inferred LR-to-TF regulatory relationships require wet-lab experimental confirmation, which was not performed; scarcity of high-temporal-resolution data limits validation
- Building cell-type-specific TRNs is very time consuming; the README discourages running it on the demo data and recommends testing on the user's own data

## Evidence

- [results] tTF identification via RTN transcriptional network and master regulator analysis: "the variable gene and their variance were input as a phenotype to perform master regulator analysis by function “tna.mra”."
- [results] ward.D2 clustering of tTFs: "we clustered the identified tTFs by hierarchical clustering (the agglomeration method was “ward.D2”)."
- [intro] Granger testing against tTFs: "we used Granger causality and network analysis to discover and check the potential regulation relationship between eLR and corresponding temporal TFs (tTFs)"
- [readme] RunTimeTalk parameters: "tmp.winsz = 0.1,
                               tmp.lags = 1,
                               numPts = 200,
                               tmp.SCC.cutoff = 0.2,"
- [readme] winsz sensitivity limitation: "the Granger causality conclusion is affected by choice of parameter winsz."
