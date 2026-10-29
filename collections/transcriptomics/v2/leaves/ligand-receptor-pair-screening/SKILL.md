---
name: ligand-receptor-pair-screening
description: Use when analyzing time-course scRNA-seq with a reconstructed trajectory
  (e.g. monocle3 pseudotime) and a ligand-receptor database to identify which LR pairs
  co-vary along development.
when_to_use_negative:
- The goal is communication between pre-defined, stable cell types (e.g. mature tissue
  labels) — CellPhoneDB or CellChat are designed for that setting, whereas this temporal
  co-variation screen targets developmental time series where definitive cell types
  do not yet exist
- Input lacks a trajectory/pseudotime ordering or is bulk bulk-only data without single-cell
  temporal structure
- Analysis targets maternal-embryo or extracellular-vesicle-mediated communication,
  which the autocrine ligand-receptor assumption of TimeTalk explicitly does not cover
tools:
- name: TimeTalk
  role: R package whose RunTimeTalk performs LR-pair extraction, interpolation, and
    correlation-based co-varying LR screening
  repo: https://github.com/ChengLiLab/TimeTalk
- name: monocle3
  role: Provides trajectory reconstruction and pseudotime ordering of cells used as
    the expression time series for LR correlation
  repo: https://github.com/cole-trapnell-lab/monocle3
- name: Seurat
  role: Holds the single-cell expression object (tmp.seu) with cluster annotations
    passed into RunTimeTalk
- name: cellAlign
  role: Interpolates aligned ligand/receptor and TF-activity time series onto a smooth
    common grid for scoring
  repo: https://github.com/shenorrLab/cellAlign
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:3468b6bdc168617b09e71406e72783c2414dd754f9b543cb2bef842a3f24e110
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# ligand-receptor-pair-screening

## Summary

Screen a ligand-receptor database against pseudotime-ordered single-cell expression series to find co-varying LR pairs, as in TimeTalk's use of 2034 CellTalkDB mouse pairs with |PCC| = 0.1 (or SCC = 0.2) correlation filtering on interpolated time series.

## When to use

Use when analyzing time-course scRNA-seq with a reconstructed trajectory (e.g. monocle3 pseudotime) and a ligand-receptor database to identify which LR pairs co-vary along development.

## When NOT to use

- The goal is communication between pre-defined, stable cell types (e.g. mature tissue labels) — CellPhoneDB or CellChat are designed for that setting, whereas this temporal co-variation screen targets developmental time series where definitive cell types do not yet exist
- Input lacks a trajectory/pseudotime ordering or is bulk bulk-only data without single-cell temporal structure
- Analysis targets maternal-embryo or extracellular-vesicle-mediated communication, which the autocrine ligand-receptor assumption of TimeTalk explicitly does not cover

## Inputs

- Pseudotime-ordered single-cell expression data (monocle3 cell_dataset cds, e.g. cds_demo.rds; Seurat object seu_demo.rds)
- Ligand-receptor pair table (CellTalkDB-derived 'Mouse-2020-Shao-LR-pairs.txt', 2034 mouse LR pairs)
- Cell type / cluster annotations (e.g. tmp.ident.1 = 'EPI', tmp.ident.2 = 'PE') and tTF master-regulator analysis results (tmp.mra.res)

## Outputs

- Co-varying LR identifier list (LR values from TimeTalk.result rows with category == 'PASS')
- Filtered co-varying LR pairs table (e.g. covarying_lr_pairs.csv) with correlation scores passing the cutoff

## How to apply

Order cells along the reconstructed trajectory pseudotime (monocle3 cds, e.g. cds_demo.rds from the TimeTalk package) and load a ligand-receptor pair table, e.g. CellTalkDB-derived 'Mouse-2020-Shao-LR-pairs.txt' with 2034 mouse LR pairs. For each LR pair, extract ligand and receptor expression across pseudotime-ordered cells and interpolate onto a common grid (numPts = 200, winsz = 0.1 as in the RunTimeTalk call). Compute the Pearson correlation coefficient (PCC) between ligand and receptor series per pair and retain pairs with |PCC| above 0.1; in the extended workflow the correlation used is Spearman with SCC = 0.2 (tmp.SCC.cutoff = 0.2 in RunTimeTalk). Filter the result table on category == 'PASS' and pull the unique LR identifiers as the co-varying LR set, which then feeds downstream Granger-causality classification against tTF clusters.

## Related tools

- **TimeTalk** (R package whose RunTimeTalk performs LR-pair extraction, interpolation, and correlation-based co-varying LR screening) — https://github.com/ChengLiLab/TimeTalk
- **monocle3** (Provides trajectory reconstruction and pseudotime ordering of cells used as the expression time series for LR correlation) — https://github.com/cole-trapnell-lab/monocle3
- **Seurat** (Holds the single-cell expression object (tmp.seu) with cluster annotations passed into RunTimeTalk)
- **cellAlign** (Interpolates aligned ligand/receptor and TF-activity time series onto a smooth common grid for scoring) — https://github.com/shenorrLab/cellAlign

## Examples

```
TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small, tmp.seu=seu.small, LRpairs.df=LRpairs.df, tmp.mra.res=tmp.mra.res, tmp.winsz=0.1, tmp.lags=1, numPts=200, tmp.SCC.cutoff=0.2, tmp.granger.cutoff=1e-2)
```

## Evaluation signals

- Every retained LR pair is present in the input CellTalkDB table (2034 mouse pairs) — identifiers should be a subset of the database
- All retained pairs satisfy the correlation cutoff: |PCC| > 0.1 in the base workflow, or SCC > 0.2 (tmp.SCC.cutoff = 0.2) in the extended workflow
- Output identifiers match TimeTalk.result rows where category == 'PASS', deduplicated by unique LR value
- Interpolation parameters match the reference call: numPts = 200 and winsz = 0.1, so series lengths are comparable across pairs
- Downstream Granger screening (task_005 analog) starting from these co-varying LRs should reproduce 430 eLRs at Granger p < 0.01

## Limitations

- The Granger causality conclusions downstream are sensitive to the interpolation parameter winsz (Supplementary Figs. 20, 21); users are advised to test multiple parameter combinations
- The autocrine hypothesis in eLR screening precludes analysis of maternal-embryo interactions, and the framework only covers ligand-receptor-mediated communication (e.g. EV-mediated signaling is not accounted for)
- Inferred LR-to-TF regulatory relationships require further wet-lab confirmation; scarcity of high-temporal-resolution datasets hinders validation

## Evidence

- [discussion] we calculated the Pearson’s correlation coefficient (PCC) and used |PCC| = 0.1 as a cutoff to get co-varying LR.: "we calculated the Pearson’s correlation coefficient (PCC) and used |PCC| = 0.1 as a cutoff to get co-varying LR."
- [discussion] Mouse ligand-receptor gene pairs (2034 pairs) were downloaded from CellTalkDB16: "Mouse ligand-receptor gene pairs (2034 pairs) were downloaded from CellTalkDB16 (http://tcm.zju.edu.cn/celltalkdb/download.php)."
- [discussion] We used SCC = 0.2 as a cutoff to identify co-varying LR.: "We used SCC = 0.2 as a cutoff to identify co-varying LR."
- [readme] TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small, ... tmp.SCC.cutoff = 0.2, tmp.granger.cutoff = 1e-2): "TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small,
                               tmp.seu=seu.small,
                               tmp.orig.ident = "blastocyst","
