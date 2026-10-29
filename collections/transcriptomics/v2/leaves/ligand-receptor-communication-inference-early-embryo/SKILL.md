---
name: ligand-receptor-communication-inference-early-embryo
description: Use when analyzing scRNA-seq from developmental systems (e.g. EPI/PE
  lineages of blastoids or blastocysts) to infer autocrine/paracrine LR signaling
  dynamics and LR-to-TF causal relationships along pseudotime trajectories.
when_to_use_negative:
- Communication is not mediated by ligand-receptor interactions (e.g. EV-mediated
  communication) — the framework does not account for other communication types
- Goal is maternal-embryo interaction analysis — the autocrine hypothesis in eLR screening
  precludes this
- Cell types are already definitive and static (e.g. mature tissue) — tools like CellPhoneDB
  or CellChat designed for communication between given cell types may suffice; TimeTalk
  targets temporal developmental data
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: TimeTalk
  role: Core R package orchestrating trajectory-based LR-to-TF causal inference via
    RunTimeTalk
  repo: https://github.com/ChengLiLab/TimeTalk
- name: Seurat
  role: Holds the annotated scRNA-seq expression object (CreateSeuratObject, FindClusters
    resolution = 0.2, FindAllMarkers)
- name: monocle3
  role: Trajectory ordering of EPI and PE lineages via RunMonocle3 with tmp.minimal_branch_len
    controlling branch inclusion
  repo: https://github.com/cole-trapnell-lab/monocle3
- name: cellAlign
  role: Aligns and interpolates the two lineage trajectories before cross-correlation
  repo: https://github.com/shenorrLab/cellAlign
- name: RTN
  role: Builds cell-type-specific transcriptional networks and runs master regulator
    analysis (tna.mra) to identify tTFs
- name: lmtest
  role: Granger causality testing on retained co-varying LR-TF pairs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:7bac025aabdaa688ccaa6f0469957d6d33d43530ea27115bed8a45b3ca67fede
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# ligand-receptor-communication-inference-early-embryo

## Summary

TimeTalk-based inference of ligand-receptor-mediated cell-cell communication from pseudotime-ordered scRNA-seq trajectories, testing LR-to-TF causal relationships via cellAlign trajectory alignment and Granger causality. Designed for early embryo development where definitive cell types emerge late.

## When to use

Use when analyzing scRNA-seq from developmental systems (e.g. EPI/PE lineages of blastoids or blastocysts) to infer autocrine/paracrine LR signaling dynamics and LR-to-TF causal relationships along pseudotime trajectories.

## When NOT to use

- Communication is not mediated by ligand-receptor interactions (e.g. EV-mediated communication) — the framework does not account for other communication types
- Goal is maternal-embryo interaction analysis — the autocrine hypothesis in eLR screening precludes this
- Cell types are already definitive and static (e.g. mature tissue) — tools like CellPhoneDB or CellChat designed for communication between given cell types may suffice; TimeTalk targets temporal developmental data

## Inputs

- Seurat object (.rds) with cell-type annotations (e.g. EPI, PE, blastocyst)
- monocle3 cell_data_set (.rds) for trajectory/pseudotime
- RTN master-regulator analysis result table (B_blastoid_RTN_mra_result.rds)
- Mouse ligand-receptor pair list (Mouse-2020-Shao-LR-pairs.txt)
- Optional: TF list from AnimalTFDB v3.0 for building cell-type-specific TRNs

## Outputs

- RunTimeTalk result table filtered to category == 'PASS' (~311 unique EPI-PE LR pairs in the blastoid case)
- Bidirectional Granger causal p-values for LR-TF pairs (e.g. Fgf4-Fgfr2/Gata6: p_LR_to_TF = 0.0184, p_TF_to_LR = 4.74e-5)
- Saved results table (TimeTalk_EPI_PE_result.csv)
- Figures reproducing Fig. 5a-e and Supplementary Fig. 19

## How to apply

Load a Seurat object, a monocle3 cds object, RTN master-regulator analysis results (e.g. B_blastoid_RTN_mra_result.rds built via myGetCellTypeSpecificTRN + tna.mra with Pvalue < 0.05 and adjusted p_val_adj < 0.05 regulons), and a ligand-receptor pair table (Mouse-2020-Shao-LR-pairs.txt). Order cells along each lineage trajectory with monocle3 via RunMonocle3, then align and interpolate the two trajectories with cellAlign (tmp.winsz = 0.1, numPts = 200, tmp.lags = 1). Compute Spearman cross-correlations between interpolated LR ligand and TF expression profiles, retaining co-varying pairs at tmp.SCC.cutoff = 0.2, then run Granger causality tests (lmtest) keeping pairs with tmp.granger.cutoff = 1e-2; classify results by the 'category == "PASS"' criterion in the RunTimeTalk output. Because Granger conclusions are winsz-sensitive, test multiple parameter combinations before trusting results.

## Related tools

- **TimeTalk** (Core R package orchestrating trajectory-based LR-to-TF causal inference via RunTimeTalk) — https://github.com/ChengLiLab/TimeTalk
- **Seurat** (Holds the annotated scRNA-seq expression object (CreateSeuratObject, FindClusters resolution = 0.2, FindAllMarkers))
- **monocle3** (Trajectory ordering of EPI and PE lineages via RunMonocle3 with tmp.minimal_branch_len controlling branch inclusion) — https://github.com/cole-trapnell-lab/monocle3
- **cellAlign** (Aligns and interpolates the two lineage trajectories before cross-correlation) — https://github.com/shenorrLab/cellAlign
- **RTN** (Builds cell-type-specific transcriptional networks and runs master regulator analysis (tna.mra) to identify tTFs)
- **lmtest** (Granger causality testing on retained co-varying LR-TF pairs)

## Examples

```
TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small, tmp.seu=seu.small, tmp.orig.ident="blastocyst", tmp.ident.1="EPI", tmp.ident.2="PE", LRpairs.df=LRpairs.df, tmp.mra.res=tmp.mra.res, tmp.winsz=0.1, tmp.lags=1, numPts=200, tmp.SCC.cutoff=0.2, tmp.granger.cutoff=1e-2)
```

## Evaluation signals

- PASS-filtered EPI-PE LR pair set is recovered (~311 unique pairs) after filter(category == "PASS") %>% pull(LR) %>% unique()
- Fgf4-Fgfr2/Gata6 feedback p-values match reported values: p_LR_to_TF = 0.0184 and p_TF_to_LR = 4.74e-5
- Fig. 5a-e and Supplementary Fig. 19 are reproducible from the saved TimeTalk_EPI_PE_result.csv
- Cell-type-specific TRN/mra results are reproduced only when regulons are restricted to DE genes with p_val_adj < 0.05 and mra Pvalue < 0.05
- Sensitivity check: rerun with multiple tmp.winsz values to confirm Granger conclusions are stable, since results are affected by winsz choice

## Limitations

- Granger causality conclusions are sensitive to the interpolation parameter winsz (Supplementary Figs. 20 and 21); users should test multiple parameter combinations
- Inferred LR-to-TF regulatory relationships require wet-lab confirmation, which was not performed
- Scarcity of high temporal resolution datasets limited validation; simulation datasets from theoretical Granger models were used instead
- Only LR-mediated communication is covered; other communication types (e.g. extracellular vesicle-mediated) are not accounted for
- Autocrine assumption precludes exploring maternal-embryo interactions

## Evidence

- [readme] TimeTalk uses the temporal information unraveled by trajectory analysis and the Granger causal test to test the ligand-receptor pairs with causal relationships to master transcription factors during development.: "TimeTalk uses the temporal information unraveled by trajectory analysis and the Granger causal test to test the ligand-receptor pairs with causal relationships to master transcription factors"
- [readme] RunTimeTalk demo call with parameters: "tmp.winsz = 0.1, tmp.lags = 1, numPts = 200, tmp.SCC.cutoff = 0.2, tmp.granger.cutoff = 1e-2"
- [discussion] Granger cutoff and candidate eLR selection: "We used 0.01 as a cutoff to get candidate eLR from co-varying LR."
- [discussion] SCC cutoff for extended workflow: "We used SCC = 0.2 as a cutoff to identify co-varying LR."
- [discussion] winsz sensitivity limitation: "the Granger causality conclusion is affected by the choice of parameter winsz"
- [other] Validation target from source card workflow: [evidence span withheld — anchor status: not_found]
- [intro] Comparison to existing tools: "commonly used tools for cell-cell communication inference, such as CellPhoneDB10 and CellChat11, are designed to study cell-cell communication between given cell types"
- [discussion] TRN construction inputs: "the log10(RPKM + 1) normalized scRNA-seq data and TF list from AnimalTFDB (version 3.0)71 to build the “tni” object"
