---
name: time-series-interpolation-cellalign
description: Use when pseudo-bulk or stage-ordered LR-score and TF-activity series
  from time-course scRNA-seq are too sparse/discontinuous for Granger causality testing
  and must be interpolated before running lmtest::grangertest.
when_to_use_negative:
- When communication is between distinct, stable cell types (e.g. post-blastocyst)
  — CellPhoneDB or CellChat are the article's designated alternatives, since TimeTalk
  targets temporal/autocrine settings where definitive cell types do not yet exist
- When input series already have high temporal resolution and no interpolation is
  needed — interpolation is only necessitated by the absence of high-resolution time-course
  data
- When the goal is EV-mediated or maternal-embryo communication — the TimeTalk framework
  only covers ligand-receptor mediated autocrine signaling
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: cellAlign
  role: Performs the interpolation/alignment of LR-score and TF-activity temporal
    series with tmp.winsz = 0.1, tmp.lags = 1, numPts = 200
  repo: https://github.com/shenorrLab/cellAlign
- name: TimeTalk
  role: R package wrapper (RunTimeTalk) that invokes cellAlign with these parameters
    alongside SCC (0.2) and Granger (1e-2) cutoffs
  repo: https://github.com/ChengLiLab/TimeTalk
- name: monocle3
  role: Provides the trajectory/pseudotime ordering of cells that defines the temporal
    axis the series are interpolated along
  repo: https://github.com/cole-trapnell-lab/monocle3
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:4103c7e0af4bc3247f20eed50ab2559f2a81b48b5f0a8cfac6cceaacd2ff38dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shenorrLab/cellAlign
  tools:
  - R
  - RTN
  - TimeTalk
  - cellAlign
  - lmtest
  - monocle3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# time-series-interpolation-cellalign

## Summary

Interpolate and align stage-ordered single-cell time series (co-varying ligand-receptor interaction scores and tTF-cluster activity profiles) with the cellAlign package so they are smooth enough for Granger causal testing in TimeTalk. Interpolation is the step that converts sparse embryo-stage snapshots into continuous temporal series suitable for causal inference.

## When to use

Use when pseudo-bulk or stage-ordered LR-score and TF-activity series from time-course scRNA-seq are too sparse/discontinuous for Granger causality testing and must be interpolated before running lmtest::grangertest.

## When NOT to use

- When communication is between distinct, stable cell types (e.g. post-blastocyst) — CellPhoneDB or CellChat are the article's designated alternatives, since TimeTalk targets temporal/autocrine settings where definitive cell types do not yet exist
- When input series already have high temporal resolution and no interpolation is needed — interpolation is only necessitated by the absence of high-resolution time-course data
- When the goal is EV-mediated or maternal-embryo communication — the TimeTalk framework only covers ligand-receptor mediated autocrine signaling

## Inputs

- Co-varying ligand-receptor interaction score series ordered by developmental pseudotime (SCC >= 0.2 cutoff)
- Six tTF-cluster activity profiles (average expression per ward.D2 cluster) over the same ordered stages
- Seurat object (tmp.seu) and monocle3 cell dataset (tmp.cds) providing trajectory/pseudotime ordering
- TimeTalk MRA result table (e.g. B_blastoid_RTN_mra_result.rds) supplying master TFs

## Outputs

- Smooth interpolated and aligned LR-score and TF-activity time series (numPts = 200 points)
- Granger-classified eLR table (forward/backward/feedback) written to eLR_granger_results.csv, 430 eLRs at p < 0.01

## How to apply

First obtain aligned temporal series from TimeTalk: co-varying LR interaction scores (identified with the SCC = 0.2 cutoff) and the six tTF-cluster average activity profiles from ward.D2 hierarchical clustering. Pass both series through cellAlign with the TimeTalk defaults tmp.winsz = 0.1, tmp.lags = 1, and numPts = 200 to produce smooth interpolated series, via RunTimeTalk(...). The winsz parameter controls the interpolation window and directly affects downstream Granger causality conclusions, so it is the key decision point: the authors recommend testing multiple parameter combinations and report sensitivity in Supplementary Figs. 20-21. The interpolated series then feed lmtest::grangertest, where directions (LR->TF, TF->LR, or both) are selected by the minimal-p-value rule and retained at Granger p < 0.01 (tmp.granger.cutoff = 1e-2), ultimately yielding 430 eLRs and 229 tTFs.

## Related tools

- **cellAlign** (Performs the interpolation/alignment of LR-score and TF-activity temporal series with tmp.winsz = 0.1, tmp.lags = 1, numPts = 200) — https://github.com/shenorrLab/cellAlign
- **TimeTalk** (R package wrapper (RunTimeTalk) that invokes cellAlign with these parameters alongside SCC (0.2) and Granger (1e-2) cutoffs) — https://github.com/ChengLiLab/TimeTalk
- **monocle3** (Provides the trajectory/pseudotime ordering of cells that defines the temporal axis the series are interpolated along) — https://github.com/cole-trapnell-lab/monocle3

## Examples

```
TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small, tmp.seu=seu.small, tmp.orig.ident="blastocyst", tmp.ident.1="EPI", tmp.ident.2="PE", LRpairs.df=LRpairs.df, tmp.mra.res=tmp.mra.res, tmp.winsz=0.1, tmp.lags=1, numPts=200, tmp.SCC.cutoff=0.2, tmp.granger.cutoff=1e-2)
```

## Evaluation signals

- Interpolated series contain numPts = 200 points per series (default in RunTimeTalk)
- Parameters tmp.winsz = 0.1, tmp.lags = 1, tmp.SCC.cutoff = 0.2, tmp.granger.cutoff = 1e-2 are used as in the article's workflow
- Downstream Granger classification reproduces the article's reported 430 eLRs and 229 tTFs at Granger p < 0.01, matching Fig. 2d-e
- Sensitivity check: re-running with different winsz values changes Granger causality conclusions (Supplementary Figs. 20-21), so a robust run should test several parameter combinations
- TimeTalk.result filtered to category == 'PASS' yields a non-empty, unique set of LR pairs

## Limitations

- The Granger causality conclusion is affected by the choice of the cellAlign winsz parameter (Supplementary Figs. 20-21); simulation datasets were used because of the absence of high temporal resolution data
- Authors advise users to conduct additional analysis and experiment with various parameter combinations to achieve more dependable results
- Scarcity of high temporal resolution datasets prevents validation of inferred LR-TF regulatory relationships; causal analysis serves only as a helpful reference

## Evidence

- [methods] cellAlign install from methods: "devtools::install_github("shenorrLab/cellAlign")"
- [readme] README workflow rationale: "TimeTalk uses the temporal information unraveled by trajectory analysis and the Granger causal test to test the ligand-receptor pairs with causal relationships to master transcription factors"
- [readme] README winsz sensitivity warning: "the Granger causality conclusion is affected by choice of parameter winsz"
- [discussion] Article discussion parameter advice: "we advise users in the TimeTalk documentation to conduct additional data analysis and experiment with various parameter combinations to achieve more dependable results"
