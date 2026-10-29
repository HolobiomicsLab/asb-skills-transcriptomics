---
name: single-cell-cell-communication-inference
description: Use when the input is time-course or pseudotime-ordered single-cell RNA-seq
  without definitive cell types (e.g. pre-blastocyst embryos) and the goal is to infer
  causal ligand-receptor communication and its link to temporal TF programs.
when_to_use_negative:
- The question is communication between well-defined cell types in a formed tissue
  — CellPhoneDB or CellChat are designed for that; TimeTalk targets settings where
  definitive cell types do not yet exist (e.g. pre-blastocyst embryos)
- The communication of interest is maternal-embryo interaction or non-LR-mediated
  (e.g. extracellular vesicle) signaling — TimeTalk's autocrine ligand-receptor assumption
  excludes these
- The data lack any temporal ordering or trajectory, since the method depends on temporal
  series information and interpolation of LR/TF activity profiles
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: TimeTalk
  role: Core R package that runs the temporal LR communication workflow (RunTimeTalk)
    using trajectory info and Granger causal testing
  repo: https://github.com/ChengLiLab/TimeTalk
- name: cellAlign
  role: Interpolates aligned LR-score and tTF-activity time series (tmp.winsz = 0.1,
    tmp.lags = 1, numPts = 200) before Granger testing
  repo: https://github.com/shenorrLab/cellAlign
- name: monocle3
  role: Provides the trajectory/pseudotime (cds) supplying the temporal information
    that TimeTalk consumes
  repo: https://github.com/cole-trapnell-lab/monocle3
- name: RTN
  role: Builds cell-type-specific TRNs and master regulator analysis (tna.mra) that
    yield the tTFs compared against LR pairs
- name: lmtest
  role: Provides grangertest for the causal LR-to-tTF-cluster tests across candidate
    lags
- name: Seurat
  role: Supplies the expression object and cell filtering (CreateSeuratObject, min.features
    = 2000) upstream of TimeTalk
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:5e2fa0620fc268aca3207d763705d8fb05d2f5e33bfa7a2ce8d6383aa1e7d4e4
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 24
---

# single-cell-cell-communication-inference

## Summary

TimeTalk-style inference of ligand-receptor cell-cell communication from time-course single-cell RNA-seq: it uses trajectory-derived temporal information, co-variation scoring of LR pairs (SCC/PCC cutoffs), and Granger causal tests against temporal TF clusters to identify development-linked eLRs.

## When to use

Use when the input is time-course or pseudotime-ordered single-cell RNA-seq without definitive cell types (e.g. pre-blastocyst embryos) and the goal is to infer causal ligand-receptor communication and its link to temporal TF programs.

## When NOT to use

- The question is communication between well-defined cell types in a formed tissue — CellPhoneDB or CellChat are designed for that; TimeTalk targets settings where definitive cell types do not yet exist (e.g. pre-blastocyst embryos)
- The communication of interest is maternal-embryo interaction or non-LR-mediated (e.g. extracellular vesicle) signaling — TimeTalk's autocrine ligand-receptor assumption excludes these
- The data lack any temporal ordering or trajectory, since the method depends on temporal series information and interpolation of LR/TF activity profiles

## Inputs

- Time-course or pseudotime-ordered single-cell RNA-seq data as a Seurat object and monocle3 cds (e.g. integrated public mouse pre-implantation scRNA-seq from GEO)
- Ligand-receptor pair table (CellTalkDB-derived 'Mouse-2020-Shao-LR-pairs.txt', 2034 mouse pairs)
- Cell-type-specific master regulator analysis results (RTN tna.mra output, e.g. B_blastoid_RTN_mra_result.rds) using an AnimalTFDB v3.0 TF list
- Ident cell types / sample identities for comparison (e.g. tmp.ident.1 = 'EPI', tmp.ident.2 = 'PE')

## Outputs

- TimeTalk result table of co-varying LR pairs with SCC scores, Granger direction (LR->TF, TF->LR, both) and category labels (forward/backward/feedback, PASS)
- Filtered set of candidate eLRs passing Granger p < 0.01 (430 eLRs and 229 tTFs in the reference run; classification table e.g. eLR_granger_results.csv)
- Temporal assignment of eLRs into six windows corresponding to sequential early-embryo development events (Fig. 2d-e)

## How to apply

Start with a monocle3 trajectory (cds) and Seurat object, cell-type-specific TRN master-regulator results from RTN (tna.mra on AnimalTFDB TF lists), and a ligand-receptor pair table (e.g. Mouse-2020-Shao-LR-pairs.txt with 2034 mouse pairs). Run RunTimeTalk with tmp.winsz = 0.1, tmp.lags = 1, numPts = 200, tmp.SCC.cutoff = 0.2 (co-varying LR screen; |PCC| = 0.1 is the alternative), and tmp.granger.cutoff = 1e-2: the LR-score and tTF-cluster activity series (tTFs clustered by ward.D2) are interpolated with cellAlign, then lmtest::grangertest assigns forward/backward/feedback direction per LR-tTF-cluster pair by minimal p-value. Retain LR pairs whose result table has category == "PASS" (Granger p < 0.01); optionally call active LR via SCC_ens > 0.8 or < -0.8. Applied to mouse pre-implantation data this yields 430 eLRs and 229 tTFs, which fall into six temporal windows matching sequential early-embryo events.

## Related tools

- **TimeTalk** (Core R package that runs the temporal LR communication workflow (RunTimeTalk) using trajectory info and Granger causal testing) — https://github.com/ChengLiLab/TimeTalk
- **cellAlign** (Interpolates aligned LR-score and tTF-activity time series (tmp.winsz = 0.1, tmp.lags = 1, numPts = 200) before Granger testing) — https://github.com/shenorrLab/cellAlign
- **monocle3** (Provides the trajectory/pseudotime (cds) supplying the temporal information that TimeTalk consumes) — https://github.com/cole-trapnell-lab/monocle3
- **RTN** (Builds cell-type-specific TRNs and master regulator analysis (tna.mra) that yield the tTFs compared against LR pairs)
- **lmtest** (Provides grangertest for the causal LR-to-tTF-cluster tests across candidate lags)
- **Seurat** (Supplies the expression object and cell filtering (CreateSeuratObject, min.features = 2000) upstream of TimeTalk)

## Examples

```
TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small, tmp.seu=seu.small, LRpairs.df=LRpairs.df, tmp.mra.res=tmp.mra.res, tmp.winsz=0.1, tmp.lags=1, numPts=200, tmp.SCC.cutoff=0.2, tmp.granger.cutoff=1e-2); tmp.res <- TimeTalk.result %>% filter(category == "PASS") %>% pull(LR) %>% unique()
```

## Evaluation signals

- The filtered result (category == "PASS", unique LR pairs) at Granger p < 0.01 reproduces the reference scale of 430 eLRs and 229 tTFs as in Fig. 2d-e
- Every retained eLR carries a direction label (forward, backward, or feedback) selected by the minimal-p-value rule across tested Granger lags
- Co-varying LR screen uses SCC = 0.2 (or |PCC| = 0.1) and active LR calls use SCC_ens > 0.8 or < -0.8 — check these thresholds appear in the run configuration
- GO/temporal-window analysis should place retained eLRs into six temporal windows matching sequential early-embryo events
- Sensitivity check: rerun with several winsz values, since Granger conclusions depend on this parameter (Supplementary Figs. 20 and 21)

## Limitations

- Granger causality conclusions are sensitive to the interpolation parameter winsz; the README advises users to 'experiment with various parameter combinations' for dependable results
- Inferred ligand-receptor-to-TF regulatory relationships are computational only and should be confirmed by further wet experiments; scarcity of high temporal-resolution datasets prevented validation with real data (theoretical Granger simulations were used)
- The framework covers only ligand-receptor-mediated communication and assumes autocrine signaling, so other modalities (e.g. EV-mediated) and maternal-embryo interactions are not accounted for

## Evidence

- [intro] we used Granger causality and network analysis to discover and check the potential regulation relationship between eLR and corresponding temporal TFs (tTFs): "we used Granger causality and network analysis to discover and check the potential regulation relationship between eLR and corresponding temporal TFs (tTFs)"
- [intro] Our analysis identified 430 eLRs, including previously reported Fgf4-Fgfr1 and Fgf4-Fgfr2.: "Our analysis identified 430 eLRs, including previously reported Fgf4-Fgfr1 and Fgf4-Fgfr2."
- [discussion] We used SCC = 0.2 as a cutoff to identify co-varying LR.: "We used SCC = 0.2 as a cutoff to identify co-varying LR."
- [discussion] We used 0.01 as a cutoff to get candidate eLR from co-varying LR.: "We used 0.01 as a cutoff to get candidate eLR from co-varying LR."
- [discussion] we clustered the identified tTFs by hierarchical clustering (the agglomeration method was "ward.D2").: [evidence span withheld — anchor status: not_found]
- [readme] TimeTalk uses the temporal information unraveled by trajectory analysis and the Granger causal test to test the ligand-receptor pairs with causal relationships to master transcription factors: "In brief, TimeTalk uses the temporal information unraveled by trajectory analysis and the Granger causal test to test the ligand-receptor pairs with causal relationships to master transcription"
- [readme] the Granger causality conclusion is affected by the choice of parameter winsz: "According to our simulation results, the Granger causality conclusion is affected by choice of parameter winsz."
- [intro] commonly used tools for cell-cell communication inference, such as CellPhoneDB10 and CellChat11, are designed to study cell-cell communication between given cell types: "commonly used tools for cell-cell communication inference, such as CellPhoneDB10 and CellChat11, are designed to study cell-cell communication between given cell types"
