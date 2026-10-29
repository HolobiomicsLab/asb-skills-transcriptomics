---
name: spearman-cross-correlation-lr-tf-covariation
description: Use when you have two single-cell trajectories (e.g. EPI and PE lineages
  from blastoid/blastocyst scRNA-seq ordered by monocle3), a ligand-receptor pair
  list, and RTN master-regulator results, and need to find co-varying LR–TF pairs
  prior to Granger causal testing.
when_to_use_negative:
- Communication between already-defined, stable cell types without temporal/trajectory
  information — CellPhoneDB or CellChat are designed for that setting
- Analysis of non-ligand-receptor communication (e.g. extracellular-vesicle-mediated)
  — the current TimeTalk framework is designed for cell-cell communication mediated
  by ligand-receptor interactions
- Maternal-embryo interactions — the autocrine/inter-lineage hypothesis underlying
  LR screening precludes exploring maternal-embryo signaling
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: TimeTalk
  role: R package whose RunTimeTalk function orchestrates trajectory alignment, SCC
    computation with tmp.SCC.cutoff = 0.2, and Granger filtering with tmp.granger.cutoff
    = 1e-2
  repo: https://github.com/ChengLiLab/TimeTalk
- name: monocle3
  role: Orders cells along the EPI and PE trajectories (RunMonocle3) to supply the
    temporal information the SCC step correlates over
  repo: https://github.com/cole-trapnell-lab/monocle3
- name: cellAlign
  role: Aligns and interpolates the two lineage trajectories (tmp.winsz = 0.1, numPts
    = 200, tmp.lags = 1) so ligand and TF profiles can be cross-correlated on a common
    axis
  repo: https://github.com/shenorrLab/cellAlign
- name: Seurat
  role: Provides the expression object (SeuratObject) holding EPI/PE identities and
    expression profiles input to RunTimeTalk
- name: RTN
  role: Builds cell-type-specific TRNs and performs master regulator analysis (tna.mra)
    to produce the TF candidates whose expression profiles are cross-correlated with
    ligands
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:1f77c71db6799a4827e4eaab0b2ac22ca94568954aa54b1f2f297e37288d8a40
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
  - monocle3
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# Spearman cross-correlation of ligand–receptor and TF expression along aligned trajectories

## Summary

A TimeTalk screening step that computes Spearman cross-correlations (SCC) between interpolated ligand and transcription-factor expression profiles from two aligned developmental trajectories, retaining LR–TF pairs with SCC ≥ 0.2 as co-varying candidates before Granger causality testing. It converts trajectory-based temporal information into a filtered candidate set of communication-linked regulatory pairs.

## When to use

Use when you have two single-cell trajectories (e.g. EPI and PE lineages from blastoid/blastocyst scRNA-seq ordered by monocle3), a ligand-receptor pair list, and RTN master-regulator results, and need to find co-varying LR–TF pairs prior to Granger causal testing.

## When NOT to use

- Communication between already-defined, stable cell types without temporal/trajectory information — CellPhoneDB or CellChat are designed for that setting
- Analysis of non-ligand-receptor communication (e.g. extracellular-vesicle-mediated) — the current TimeTalk framework is designed for cell-cell communication mediated by ligand-receptor interactions
- Maternal-embryo interactions — the autocrine/inter-lineage hypothesis underlying LR screening precludes exploring maternal-embryo signaling

## Inputs

- Seurat object with EPI and PE cell-type identities (blastoid/blastocyst scRNA-seq)
- monocle3 cds object with ordered cell trajectories
- RTN master-regulator analysis results (e.g. B_blastoid_RTN_mra_result.rds)
- Mouse-2020-Shao ligand-receptor pair list (Mouse-2020-Shao-LR-pairs.txt, 2034 CellTalkDB-derived pairs)

## Outputs

- RunTimeTalk result table with SCC values, Granger p-values, and category column
- Set of unique PASS-filtered ligand-receptor pairs (~311 EPI-PE LR pairs)
- Bidirectional Granger p-values for specific LR-TF pairs (e.g. Fgf4-Fgfr2/Gata6: p_LR_to_TF = 0.0184, p_TF_to_LR = 4.74e-5), exported as TimeTalk_EPI_PE_result.csv

## How to apply

Order cells along the two lineages with monocle3 via RunMonocle3 (controlling tmp.minimal_branch_len), then align and interpolate the trajectories with cellAlign using window size tmp.winsz = 0.1, numPts = 200, and tmp.lags = 1 so that the two lineages share a common axis. Compute Spearman cross-correlations between the interpolated ligand expression profiles of each ligand-receptor pair and the expression profiles of candidate TFs from the RTN master-regulator analysis (B_blastoid_RTN_mra_result.rds). Retain co-varying LR–TF pairs passing tmp.SCC.cutoff = 0.2; this SCC cutoff supersedes the |PCC| = 0.1 cutoff used in the core pre-implantation workflow because interpolation enables cross-lineage comparison. Pass the retained pairs to Granger causality tests (lmtest) with tmp.granger.cutoff = 1e-2 and keep rows classified as category == "PASS" in the RunTimeTalk output. The rationale is that interpolation across aligned trajectories exposes temporal coupling between LR signaling and TF activity, revealing paracrine-like communication settings that fixed cell-type methods miss.

## Related tools

- **TimeTalk** (R package whose RunTimeTalk function orchestrates trajectory alignment, SCC computation with tmp.SCC.cutoff = 0.2, and Granger filtering with tmp.granger.cutoff = 1e-2) — https://github.com/ChengLiLab/TimeTalk
- **monocle3** (Orders cells along the EPI and PE trajectories (RunMonocle3) to supply the temporal information the SCC step correlates over) — https://github.com/cole-trapnell-lab/monocle3
- **cellAlign** (Aligns and interpolates the two lineage trajectories (tmp.winsz = 0.1, numPts = 200, tmp.lags = 1) so ligand and TF profiles can be cross-correlated on a common axis) — https://github.com/shenorrLab/cellAlign
- **Seurat** (Provides the expression object (SeuratObject) holding EPI/PE identities and expression profiles input to RunTimeTalk)
- **RTN** (Builds cell-type-specific TRNs and performs master regulator analysis (tna.mra) to produce the TF candidates whose expression profiles are cross-correlated with ligands)

## Examples

```
TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small, tmp.seu=seu.small, tmp.ident.1="EPI", tmp.ident.2="PE", LRpairs.df=LRpairs.df, tmp.mra.res=tmp.mra.res, tmp.winsz=0.1, tmp.lags=1, numPts=200, tmp.SCC.cutoff=0.2, tmp.granger.cutoff=1e-2); tmp.res <- TimeTalk.result %>% filter(category=="PASS") %>% pull(LR) %>% unique()
```

## Evaluation signals

- PASS-filtered EPI-PE ligand-receptor set recovered at ~311 unique pairs after SCC = 0.2 and Granger 1e-2 filtering
- Fgf4-Fgfr2/Gata6 bidirectional Granger p-values match reported values: p_LR_to_TF = 0.0184 and p_TF_to_LR = 4.74e-5
- Result table rows with category == "PASS" constitute the retained LR set; non-PASS rows are excluded before counting unique LR pairs
- Granger conclusions are checked for sensitivity to the interpolation parameter winsz (Supplementary Figs. 20 and 21), so reproducibility should be verified across winsz parameter combinations
- Reproduces Fig. 5a-e and Supplementary Fig. 19 of the source article when applied to the EPS-blastoid/natural blastocyst data

## Limitations

- Granger causality conclusions are affected by the choice of the interpolation parameter winsz; users are advised to experiment with various parameter combinations for dependable results
- Inferred LR-to-TF regulatory relationships require wet-lab experimental confirmation, which was not performed; causal analysis is only a reference
- Scarcity of high temporal resolution datasets hinders validation of the inferred regulatory relationships

## Evidence

- [discussion] We used SCC = 0.2 as a cutoff to identify co-varying LR.: "We used SCC = 0.2 as a cutoff to identify co-varying LR."
- [intro] Granger and network analysis for eLR-tTF regulation: "we used Granger causality and network analysis to discover and check the potential regulation relationship between eLR and corresponding temporal TFs (tTFs)"
- [discussion] Core workflow PCC cutoff for comparison: "we calculated the Pearson’s correlation coefficient (PCC) and used |PCC| = 0.1 as a cutoff to get co-varying LR."
- [readme] winsz sensitivity of Granger conclusions: "According to our simulation results, the Granger causality conclusion is affected by choice of parameter winsz."
- [readme] RunTimeTalk demo invocation with cutoffs: "tmp.SCC.cutoff = 0.2,
                               tmp.granger.cutoff = 1e-2)"
