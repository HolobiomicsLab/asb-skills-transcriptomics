---
name: single-cell-trajectory-cell-cell-communication-analysis
description: Use when scRNA-seq data span a developmental time course without definitive
  cell types (e.g., early embryo pre-blastocyst stages), and the goal is to find development-related
  ligand-receptor pairs whose expression co-varies or causally regulates TF activity
  along pseudotime.
when_to_use_negative:
- Communication is between well-defined, given cell types at steady state — CellPhoneDB
  or CellChat are designed for that setting, whereas TimeTalk targets temporal/autocrine
  dynamics without definitive cell types
- The communication of interest is not ligand-receptor mediated (e.g. extracellular-vesicle-mediated
  signaling), which the current TimeTalk framework does not account for
- 'The question concerns maternal-embryo interactions: the autocrine hypothesis of
  eLR screening precludes that analysis'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: TimeTalk
  role: 'Core R package: interpolates pseudotime expression series, tests LR co-variation
    and Granger causality to TFs (RunTimeTalk)'
  repo: https://github.com/ChengLiLab/TimeTalk
- name: monocle3
  role: Trajectory reconstruction and pseudotime ordering of cells feeding the temporal
    series (RunMonocle3)
  repo: https://github.com/cole-trapnell-lab/monocle3
- name: Seurat
  role: Holds the annotated scRNA-seq object (Seurat 3.2.3 in the article) supplying
    expression and cell identities to TimeTalk
- name: cellAlign
  role: Dependency used for aligning/interpolating expression along pseudotime (numPts
    = 200, winsz = 0.1)
  repo: https://github.com/shenorrLab/cellAlign
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:2b7389b02f0dd09e144341b49445cbf5790462ad2b867bb69d556565199718a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - Seurat
  - TimeTalk
  - cellAlign
  - monocle3
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# single-cell-trajectory-cell-cell-communication-analysis

## Summary

Identify co-varying ligand-receptor pairs and infer causal LR-to-TF communication from time-course single-cell RNA-seq by ordering cells along a trajectory (pseudotime), interpolating expression series, and testing temporal correlations and Granger causality. Embodied by the TimeTalk R package for early embryo development data.

## When to use

Use when scRNA-seq data span a developmental time course without definitive cell types (e.g., early embryo pre-blastocyst stages), and the goal is to find development-related ligand-receptor pairs whose expression co-varies or causally regulates TF activity along pseudotime.

## When NOT to use

- Communication is between well-defined, given cell types at steady state — CellPhoneDB or CellChat are designed for that setting, whereas TimeTalk targets temporal/autocrine dynamics without definitive cell types
- The communication of interest is not ligand-receptor mediated (e.g. extracellular-vesicle-mediated signaling), which the current TimeTalk framework does not account for
- The question concerns maternal-embryo interactions: the autocrine hypothesis of eLR screening precludes that analysis

## Inputs

- Pseudotime-ordered monocle3 cell dataset (cell_data_set object, e.g. cds_demo.rds) or monocle2 DDRTree trajectory
- Seurat object with cell type annotations (e.g. seu_demo.rds, ident.1 = 'EPI', ident.2 = 'PE')
- Ligand-receptor pair table (CellTalkDB mouse pairs, TSV: Mouse-2020-Shao-LR-pairs.txt)
- Master regulator analysis results from RTN cell-type-specific TRN (e.g. B_blastoid_RTN_mra_result.rds)
- TimeTalk parameters: numPts = 200, winsz = 0.1, lags = 1, SCC cutoff = 0.2, Granger cutoff = 1e-2

## Outputs

- TimeTalk.result data frame with per-LR-pair correlation, SCC, Granger statistics and category (PASS/FAIL)
- Filtered list of co-varying/passing LR identifiers (e.g. covarying_lr_pairs.csv)
- Candidate and active eLR sets (Granger p < 0.01; SCC_ens > 0.8 or < -0.8)

## How to apply

Order cells along a reconstructed trajectory using monocle3/monocle2 pseudotime (e.g., DDRTree with differentially expressed genes at qval = 0.01, or TimeTalk's RunMonocle3 on a Seurat object). For each ligand-receptor pair from a CellTalkDB-derived table (2034 mouse pairs, e.g. Mouse-2020-Shao-LR-pairs.txt), extract ligand and receptor expression series across pseudotime-ordered cells and interpolate onto a common grid (numPts = 200, winsz = 0.1 in RunTimeTalk). Compute Pearson correlation coefficients and retain co-varying LR pairs with |PCC| >= 0.1; the extended workflow analog applies SCC cutoff = 0.2 (run RunTimeTalk with tmp.SCC.cutoff = 0.2, tmp.granger.cutoff = 1e-2, tmp.lags = 1). Then apply Granger causality testing (p-value cutoff 0.01) against master regulators from RTN master regulator analysis (tna.mra) to call candidate eLRs, and SCC_ens > 0.8 or < -0.8 to call active LRs. Filter the result table on category == 'PASS' and pull unique LR identifiers as the final co-varying/passing LR set.

## Related tools

- **TimeTalk** (Core R package: interpolates pseudotime expression series, tests LR co-variation and Granger causality to TFs (RunTimeTalk)) — https://github.com/ChengLiLab/TimeTalk
- **monocle3** (Trajectory reconstruction and pseudotime ordering of cells feeding the temporal series (RunMonocle3)) — https://github.com/cole-trapnell-lab/monocle3
- **Seurat** (Holds the annotated scRNA-seq object (Seurat 3.2.3 in the article) supplying expression and cell identities to TimeTalk)
- **cellAlign** (Dependency used for aligning/interpolating expression along pseudotime (numPts = 200, winsz = 0.1)) — https://github.com/shenorrLab/cellAlign

## Examples

```
TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small, tmp.seu=seu.small, tmp.ident.1="EPI", tmp.ident.2="PE", LRpairs.df=LRpairs.df, tmp.mra.res=tmp.mra.res, tmp.winsz=0.1, tmp.lags=1, numPts=200, tmp.SCC.cutoff=0.2, tmp.granger.cutoff=1e-2)
```

## Evaluation signals

- All retained co-varying LR pairs satisfy |PCC| >= 0.1 (or SCC >= 0.2 in the extended workflow); a sanity check re-derives these correlations from the interpolated pseudotime series
- Result table passes the category == 'PASS' filter and unique LR values are non-empty; previously reported pairs such as Fgf4-Fgfr1 and Fgf4-Fgfr2 should be recoverable among the 430 identified eLRs
- Candidate eLRs pass the Granger p-value cutoff of 0.01, and active LRs meet |SCC_ens| > 0.8
- Interpolation grid matches RunTimeTalk settings (numPts = 200, winsz = 0.1); deviations in winsz are known to change Granger causality conclusions, so results should be checked for parameter sensitivity across combinations

## Limitations

- Granger causality conclusions are affected by the choice of the winsz parameter; users are advised to experiment with various parameter combinations for dependable results
- Inferred LR-to-TF regulatory relationships require wet-lab experimental confirmation, which was not performed, and scarcity of high temporal resolution datasets hinders validation
- The autocrine hypothesis in eLR screening precludes exploring maternal-embryo interactions during early embryo development
- Only ligand-receptor mediated communication is covered; other communication types (e.g. EV-mediated) are not accounted for

## Evidence

- [discussion] PCC cutoff on pseudotime series: "we calculated the Pearson’s correlation coefficient (PCC) and used |PCC| = 0.1 as a cutoff to get co-varying LR."
- [discussion] SCC cutoff: "We used SCC = 0.2 as a cutoff to identify co-varying LR."
- [readme] RunTimeTalk demo call: [evidence span withheld — anchor status: not_found]
- [readme] category filter: "tmp.res <- TimeTalk.result %>% filter(category == "PASS") %>% pull(LR) %>% unique()"
- [intro] scope vs existing tools: "commonly used tools for cell-cell communication inference, such as CellPhoneDB10 and CellChat11, are designed to study cell-cell communication between given cell types"
