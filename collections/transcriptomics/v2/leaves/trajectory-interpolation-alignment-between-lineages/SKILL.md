---
name: trajectory-interpolation-alignment-between-lineages
description: Use when you have two lineage-specific single-cell trajectories (e.g.
  Seurat objects with EPI/PE identities and a monocle3 cds) and need to test whether
  LR pairs co-vary and causally precede master TFs along pseudotime.
when_to_use_negative:
- The goal is communication between well-defined static cell types rather than dynamic
  lineages along pseudotime — CellPhoneDB or CellChat are designed for that setting
  and the article states they 'cannot meet the requirements of early embryo development
  research, where definitive cell types do not emerge until blastocyst formation'
- The study concerns maternal-embryo interactions, since the autocrine assumption
  in eLR screening 'precludes our study from exploring maternal-embryo interactions
  during early embryo development'
- The communication of interest is not ligand-receptor mediated (e.g. EV-mediated),
  as the current framework is 'designed for cell-cell communication mediated by ligand-receptor
  interactions'
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
tools:
- name: monocle3
  role: Orders cells along each lineage trajectory (pseudotime/branch inference) via
    RunMonocle3 before alignment
  repo: cole-trapnell-lab/monocle3
- name: cellAlign
  role: Aligns and interpolates the two lineage trajectories (winsz = 0.1, numPts
    = 200) onto a common axis for cross-correlation
  repo: shenorrLab/cellAlign
- name: TimeTalk
  role: R package wrapping trajectory ordering, alignment, SCC filtering, and Granger
    causal testing in RunTimeTalk
  repo: https://github.com/ChengLiLab/TimeTalk
- name: Seurat
  role: Provides the input Seurat object with cell identities (EPI/PE) used as tmp.ident.1/tmp.ident.2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:7d7b8b4dfa63e3248f935ec4c2012d5ac258a2d9dbbc31e36ac482a009704bd9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cole-trapnell-lab/monocle3
  tools:
  - R
  - RTN
  - Seurat
  - TimeTalk
  - cellAlign
  - monocle3
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# trajectory-interpolation-alignment-between-lineages

## Summary

Aligns two developmental lineage trajectories (e.g. EPI and PE from blastoid and natural blastocyst scRNA-seq) by ordering cells with monocle3, interpolating each lineage onto a common axis with cellAlign, and computing Spearman cross-correlations (SCC ≥ 0.2) so ligand-receptor vs. TF temporal profiles can be causally compared via Granger tests (cutoff 1e-2).

## When to use

Use when you have two lineage-specific single-cell trajectories (e.g. Seurat objects with EPI/PE identities and a monocle3 cds) and need to test whether LR pairs co-vary and causally precede master TFs along pseudotime.

## When NOT to use

- The goal is communication between well-defined static cell types rather than dynamic lineages along pseudotime — CellPhoneDB or CellChat are designed for that setting and the article states they 'cannot meet the requirements of early embryo development research, where definitive cell types do not emerge until blastocyst formation'
- The study concerns maternal-embryo interactions, since the autocrine assumption in eLR screening 'precludes our study from exploring maternal-embryo interactions during early embryo development'
- The communication of interest is not ligand-receptor mediated (e.g. EV-mediated), as the current framework is 'designed for cell-cell communication mediated by ligand-receptor interactions'

## Inputs

- Seurat object (Seurat 3.2.3) with lineage identities (e.g. tmp.ident.1='EPI', tmp.ident.2='PE')
- monocle3 cds object (cell_data_set from RunMonocle3)
- RTN master-regulator analysis results (e.g. B_blastoid_RTN_mra_result.rds) identifying temporal TFs
- Ligand-receptor pair list, e.g. Mouse-2020-Shao-LR-pairs.txt from CellTalkDB-derived Mouse-2020-Shao set

## Outputs

- Aligned/interpolated lineage expression profiles on a common pseudotime axis (numPts = 200 points)
- Spearman cross-correlation matrix between interpolated LR ligand and TF profiles (retained pairs with SCC = 0.2)
- RunTimeTalk result table with per-LR category == 'PASS' Granger outcomes (e.g. TimeTalk_EPI_PE_result.csv)
- Unique PASS ligand-receptor pair set (~311 EPI-PE pairs) and bidirectional Granger p-values (e.g. Fgf4-Fgfr2/Gata6)

## How to apply

Order cells along each lineage with monocle3 via RunMonocle3 (the tmp.minimal_branch_len parameter controls which branches are included). Align and interpolate the two lineage trajectories with cellAlign using window size tmp.winsz = 0.1, tmp.lags = 1, and numPts = 200 interpolated points, so both lineages are expressed on a shared axis. Compute Spearman cross-correlations between interpolated ligand and TF expression profiles and retain co-varying LR-TF pairs with tmp.SCC.cutoff = 0.2; these retained pairs are the input to Granger causality tests (lmtest) filtered at tmp.granger.cutoff = 1e-2 inside RunTimeTalk. Note the article uses |PCC| = 0.1 for the shorter pre-implantation workflow but SCC = 0.2 for the extended blastoid workflow, so pick the cutoff matching your data. The interpolation is what makes cross-lineage comparison possible: the article notes 'With the interpolation strategy, TimeTalk is powerful in revealing paracrine settings'.

## Related tools

- **monocle3** (Orders cells along each lineage trajectory (pseudotime/branch inference) via RunMonocle3 before alignment) — cole-trapnell-lab/monocle3
- **cellAlign** (Aligns and interpolates the two lineage trajectories (winsz = 0.1, numPts = 200) onto a common axis for cross-correlation) — shenorrLab/cellAlign
- **TimeTalk** (R package wrapping trajectory ordering, alignment, SCC filtering, and Granger causal testing in RunTimeTalk) — https://github.com/ChengLiLab/TimeTalk
- **Seurat** (Provides the input Seurat object with cell identities (EPI/PE) used as tmp.ident.1/tmp.ident.2)

## Examples

```
TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small, tmp.seu=seu.small, tmp.ident.1="EPI", tmp.ident.2="PE", LRpairs.df=LRpairs.df, tmp.mra.res=tmp.mra.res, tmp.winsz=0.1, tmp.lags=1, numPts=200, tmp.SCC.cutoff=0.2, tmp.granger.cutoff=1e-2)
```

## Evaluation signals

- The PASS-filtered EPI-PE ligand-receptor set is recovered (~311 unique LR pairs after filter(category == "PASS") %>% pull(LR) %>% unique())
- Bidirectional Granger p-values for Fgf4-Fgfr2/Gata6 match the reported values (p_LR_to_TF = 0.0184, p_TF_to_LR = 4.74e-5)
- All retained LR-TF pairs satisfy the Spearman cross-correlation cutoff (SCC = 0.2) before Granger testing
- Interpolation parameters match the reference run: tmp.winsz = 0.1, tmp.lags = 1, numPts = 200
- Reproducing Fig. 5a-e and Supplementary Fig. 19 is the article-level validation criterion

## Limitations

- The Granger causality conclusion is affected by the choice of the interpolation parameter winsz (Supplementary Figs. 20 and 21); authors advise testing multiple parameter combinations for dependable results
- Inferred LR-to-TF regulatory relationships require wet-lab confirmation, which was not performed, and high temporal resolution datasets are too scarce to validate causality
- Causal analysis is only a reference (theoretical Granger simulation datasets were used given the absence of high temporal resolution data)

## Evidence

- [discussion] We used SCC = 0.2 as a cutoff to identify co-varying LR.: "We used SCC = 0.2 as a cutoff to identify co-varying LR."
- [methods] RunMonocle3(tmp.seu, tmp.assay, tmp.minimal_branch_len): "RunMonocle3(tmp.seu, tmp.assay, tmp.minimal_branch_len)"
- [readme] TimeTalk uses the temporal information unraveled by trajectory analysis and the Granger causal test: "TimeTalk uses the temporal information unraveled by trajectory analysis and the Granger causal test to test the ligand-receptor pairs with causal relationships to master transcription factors"
- [discussion] the Granger causality conclusion is affected by the choice of parameter winsz: "the Granger causality conclusion is affected by the choice of parameter winsz (Supplementary Figs. 20 and 21)"
- [abstract] TimeTalk is powerful in revealing paracrine settings and studying cell-cell communication: "With the interpolation strategy, TimeTalk is powerful in revealing paracrine settings and studying cell-cell communication during early embryo development"
