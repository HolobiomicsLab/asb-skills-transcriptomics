---
name: granger-causality-testing
description: Use when you have aligned, interpolated temporal profiles of ligand-receptor
  pairs and TF-cluster activity (e.g. from cellAlign) and need to infer directionality
  — LR->TF, TF->LR, or bidirectional feedback — between them.
when_to_use_negative:
- Communication is not mediated by ligand-receptor interactions (e.g. EV-mediated
  signaling), which the TimeTalk framework does not account for
- The goal is maternal-embryo interaction analysis — the autocrine assumption in eLR
  screening precludes it
- Data are static cross-sections of defined cell types with no temporal/trajectory
  information — use CellPhoneDB or CellChat instead
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: TimeTalk
  role: R package orchestrating the temporal Granger causality workflow (RunTimeTalk)
    with tmp.SCC.cutoff and tmp.granger.cutoff parameters
  repo: https://github.com/ChengLiLab/TimeTalk
- name: lmtest
  role: Provides grangertest, used across candidate lags with the minimal-p-value
    rule to classify direction
- name: cellAlign
  role: Interpolates/aligns LR and TF-activity time series (tmp.winsz = 0.1, numPts
    = 200) before causal testing
  repo: https://github.com/shenorrLab/cellAlign
- name: monocle3
  role: Orders cells along EPI and PE trajectories (RunMonocle3) to supply temporal
    profiles for testing
  repo: https://github.com/cole-trapnell-lab/monocle3
- name: RTN
  role: Builds transcriptional networks and performs master regulator analysis (tna.mra)
    to identify tTFs (229 tTFs reported)
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:73fd058ad4e1a964f89cd8f0856c6d242ab9c326bddbd3256e8abbce6702ba8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - RTN
  - Seurat
  - TimeTalk
  - cellAlign
  - lmtest
  - monocle3
  - tidyverse
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# granger-causality-testing

## Summary

Apply Granger causal tests (lmtest::grangertest) to interpolated ligand-receptor and temporal-TF time series from TimeTalk, classifying co-varying LR pairs as forward, backward, or feedback regulation relative to tTF clusters at a p < 0.01 cutoff.

## When to use

Use when you have aligned, interpolated temporal profiles of ligand-receptor pairs and TF-cluster activity (e.g. from cellAlign) and need to infer directionality — LR->TF, TF->LR, or bidirectional feedback — between them.

## When NOT to use

- Communication is not mediated by ligand-receptor interactions (e.g. EV-mediated signaling), which the TimeTalk framework does not account for
- The goal is maternal-embryo interaction analysis — the autocrine assumption in eLR screening precludes it
- Data are static cross-sections of defined cell types with no temporal/trajectory information — use CellPhoneDB or CellChat instead

## Inputs

- Co-varying ligand-receptor interaction scores (e.g. from TimeTalk with SCC = 0.2 cutoff)
- tTF-cluster activity profiles from ward.D2 hierarchical clustering of RTN master-regulator analysis results (B_blastoid_RTN_mra_result.rds)
- Aligned lineage trajectories from monocle3 (RunMonocle3 with tmp.minimal_branch_len)
- Ligand-receptor pair list (Mouse-2020-Shao-LR-pairs.txt, 2034 CellTalkDB mouse pairs)

## Outputs

- Classification table of eLRs with direction labels (forward/backward/feedback) and Granger p-values (eLR_granger_results.csv)
- PASS-filtered LR pair set (e.g. ~311 EPI-PE pairs in TimeTalk_EPI_PE_result.csv)
- Bidirectional Granger p-values for specific LR-TF pairs (e.g. Fgf4-Fgfr2/Gata6)

## How to apply

Interpolate the aligned LR-score and TF-activity time series with cellAlign (tmp.winsz = 0.1, tmp.lags = 1, numPts = 200) so the series are smooth enough for causal testing. First retain co-varying LR-TF pairs using the correlation cutoff (|PCC| = 0.1 in the original workflow; SCC = 0.2 in the extended blastoid workflow). For each LR-tTF-cluster pair, run lmtest::grangertest across candidate lags and select the direction (LR->TF, TF->LR, or both) by the minimal-p-value rule. Retain candidate eLRs passing the Granger p < 0.01 cutoff (tmp.granger.cutoff = 1e-2) and label them forward, backward, or feedback; the original run yielded 430 eLRs and 229 tTFs, and the blastoid EPI-PE run recovered ~311 PASS LR pairs with Fgf4-Fgfr2/Gata6 feedback (p_LR_to_TF = 0.0184, p_TF_to_LR = 4.74e-5). Because conclusions are sensitive to the interpolation parameter winsz (Supplementary Figs. 20 and 21), test multiple parameter combinations for dependable results.

## Related tools

- **TimeTalk** (R package orchestrating the temporal Granger causality workflow (RunTimeTalk) with tmp.SCC.cutoff and tmp.granger.cutoff parameters) — https://github.com/ChengLiLab/TimeTalk
- **lmtest** (Provides grangertest, used across candidate lags with the minimal-p-value rule to classify direction)
- **cellAlign** (Interpolates/aligns LR and TF-activity time series (tmp.winsz = 0.1, numPts = 200) before causal testing) — https://github.com/shenorrLab/cellAlign
- **monocle3** (Orders cells along EPI and PE trajectories (RunMonocle3) to supply temporal profiles for testing) — https://github.com/cole-trapnell-lab/monocle3
- **RTN** (Builds transcriptional networks and performs master regulator analysis (tna.mra) to identify tTFs (229 tTFs reported))

## Examples

```
TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small, tmp.seu=seu.small, tmp.ident.1="EPI", tmp.ident.2="PE", LRpairs.df=LRpairs.df, tmp.mra.res=tmp.mra.res, tmp.winsz=0.1, tmp.lags=1, numPts=200, tmp.SCC.cutoff=0.2, tmp.granger.cutoff=1e-2); tmp.res <- TimeTalk.result %>% filter(category == "PASS") %>% pull(LR) %>% unique()
```

## Evaluation signals

- Output contains 430 eLRs classified at Granger p < 0.01 with forward/backward/feedback labels, matching Fig. 2d-e
- Extended EPI-PE run recovers ~311 PASS LR pairs via filter(category == "PASS")
- Fgf4-Fgfr2/Gata6 p-values match p_LR_to_TF = 0.0184 and p_TF_to_LR = 4.74e-5 (Supplementary Fig. 19)
- 229 tTFs retained alongside the 430 eLRs
- Sensitivity check: re-running with different winsz values (Supplementary Figs. 20 and 21) to confirm Granger conclusions are robust

## Limitations

- Granger causality conclusions are sensitive to the interpolation parameter winsz (Supplementary Figs. 20 and 21); users are advised to test multiple parameter combinations
- Inferred LR-to-TF regulatory relationships require wet-lab confirmation, which was not performed; scarcity of high temporal resolution datasets hinders validation
- Due to absent high temporal resolution data, simulation datasets from theoretical Granger causality models were used instead of real data to assess robustness
- Causal analysis serves only as a helpful reference and is intended to be improved with more accurate data

## Evidence

- [intro] Granger causality and network analysis were used to discover and check potential regulation relationships between eLRs and temporal TFs.: [evidence span withheld — anchor status: not_found]
- [methods] Granger cutoff 0.01 for candidate eLRs.: "We used 0.01 as a cutoff to get candidate eLR from co-varying LR."
- [readme] README RunTimeTalk demo invocation.: "TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small, tmp.seu=seu.small, tmp.orig.ident = "blastocyst", tmp.ident.1="EPI", tmp.ident.2 = "PE""
- [readme] winsz sensitivity in README recommendation.: "the Granger causality conclusion is affected by choice of parameter winsz"
- [discussion] Simulation datasets used due to lack of high temporal resolution data.: "Given the absence of high temporal resolution data, we utilized two models based on theoretical Granger causality to create simulation datasets."
