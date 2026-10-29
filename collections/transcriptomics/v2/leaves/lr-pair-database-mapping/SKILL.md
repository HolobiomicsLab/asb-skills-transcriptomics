---
name: lr-pair-database-mapping
description: Use when screening ligand-receptor pairs from a curated LR database against
  pseudotime-ordered scRNA-seq trajectories to identify co-varying LR pairs in temporal
  developmental data.
when_to_use_negative:
- Input data has no temporal/trajectory ordering — CellPhoneDB or CellChat are designed
  for communication between given cell types and would be more appropriate for static
  cell-type comparisons
- Goal is to call active (not merely co-varying) LR pairs — the |PCC| = 0.1 filter
  is only the first screen; candidate eLR and active LR require Granger causality
  (p < 0.01) and SCC_ens > 0.8 or < -0.8
- Interest is in maternal-embryo or EV-mediated communication — the TimeTalk autocrine/eLR
  screening assumption precludes maternal-embryo interactions and other non-LR-mediated
  communication types
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: TimeTalk
  role: R package implementing the co-varying LR screening via RunTimeTalk on pseudotime-interpolated
    ligand-receptor series
  repo: https://github.com/ChengLiLab/TimeTalk
- name: monocle3
  role: Provides the pseudotime-ordered cell dataset (cds) on which ligand-receptor
    expression series are extracted
  repo: https://github.com/cole-trapnell-lab/monocle3
- name: Seurat
  role: Supplies the expression matrix (seu) used alongside the cds for LR expression
    extraction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:94e401b15b0af92ac577433a81bcc4bbeda03c48f2e98141943b0b6983754a8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - Seurat
  - TimeTalk
  - monocle3
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# lr-pair-database-mapping

## Summary

Maps curated ligand-receptor database pairs (e.g. the CellTalkDB-derived Mouse-2020-Shao-LR-pairs.txt, 2034 mouse LR pairs) onto time-course single-cell expression data and retains co-varying LR pairs by computing Pearson correlation coefficients on pseudotime-ordered series with a |PCC| >= 0.1 cutoff.

## When to use

Use when screening ligand-receptor pairs from a curated LR database against pseudotime-ordered scRNA-seq trajectories to identify co-varying LR pairs in temporal developmental data.

## When NOT to use

- Input data has no temporal/trajectory ordering — CellPhoneDB or CellChat are designed for communication between given cell types and would be more appropriate for static cell-type comparisons
- Goal is to call active (not merely co-varying) LR pairs — the |PCC| = 0.1 filter is only the first screen; candidate eLR and active LR require Granger causality (p < 0.01) and SCC_ens > 0.8 or < -0.8
- Interest is in maternal-embryo or EV-mediated communication — the TimeTalk autocrine/eLR screening assumption precludes maternal-embryo interactions and other non-LR-mediated communication types

## Inputs

- Pseudotime-ordered monocle3 cell dataset (cds_demo.rds) and Seurat object (seu_demo.rds)
- CellTalkDB-derived ligand-receptor pair table (e.g. Mouse-2020-Shao-LR-pairs.txt, 2034 mouse LR pairs)
- Interpolation parameters numPts = 200 and winsz = 0.1

## Outputs

- Table of co-varying ligand-receptor pairs with |PCC| above cutoff (analogous to TimeTalk.result rows with category == 'PASS')
- Unique co-varying LR identifiers (e.g. written to covarying_lr_pairs.csv)

## How to apply

Load a monocle3 cell dataset (cds) with cells ordered along reconstructed trajectory pseudotime and extract ligand and receptor expression series for each LR pair from a database table (e.g. CellTalkDB Mouse-2020-Shao-LR-pairs.txt). Interpolate both series onto a common grid (numPts = 200, winsz = 0.1 as in the RunTimeTalk call) so that series of different cell orders become comparable. Compute the Pearson correlation coefficient between ligand and receptor series for each pair and retain pairs with |PCC| above 0.1 (0.2 SCC cutoff in the extended workflow, tmp.SCC.cutoff = 0.2) as co-varying LR pairs. Filter the TimeTalk.result table on category == 'PASS' and pull unique LR values to collect the co-varying LR identifiers. Co-varying LRs are an intermediate stage — they are further narrowed to candidate eLRs by a Granger test (p cutoff 0.01) and active LR by SCC_ens > 0.8 or < -0.8.

## Related tools

- **TimeTalk** (R package implementing the co-varying LR screening via RunTimeTalk on pseudotime-interpolated ligand-receptor series) — https://github.com/ChengLiLab/TimeTalk
- **monocle3** (Provides the pseudotime-ordered cell dataset (cds) on which ligand-receptor expression series are extracted) — https://github.com/cole-trapnell-lab/monocle3
- **Seurat** (Supplies the expression matrix (seu) used alongside the cds for LR expression extraction)

## Examples

```
TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small, tmp.seu=seu.small, LRpairs.df=LRpairs.df, tmp.mra.res=tmp.mra.res, tmp.winsz=0.1, tmp.lags=1, numPts=200, tmp.SCC.cutoff=0.2, tmp.granger.cutoff=1e-2)
```

## Evaluation signals

- Each retained pair has |PCC| > 0.1 (or SCC > 0.2 in the extended workflow) between its ligand and receptor pseudotime series
- Both ligand and receptor series are interpolated to a common grid of numPts = 200 points with winsz = 0.1 before correlation
- Final LR list contains unique identifiers obtained by filtering TimeTalk.result on category == 'PASS' and pulling unique LR values
- Downstream Granger screening (p < 0.01) yields a non-empty subset of candidate eLRs from the co-varying set
- All co-varying LR identifiers are members of the input CellTalkDB-derived LR pair table (2034 mouse pairs)

## Limitations

- Granger causality conclusions are affected by the choice of the interpolation parameter winsz; users are advised to test multiple parameter combinations
- Co-varying LR pairs are only a precursor set — correlation does not establish causal LR-TF regulation, which the authors state should be confirmed by further wet experiments
- The framework assumes autocrine LR signaling within the embryo, precluding maternal-embryo interaction analysis

## Evidence

- [discussion] PCC cutoff on pseudotime series: [evidence span withheld — anchor status: not_found]
- [discussion] SCC cutoff for extended workflow: "We used SCC = 0.2 as a cutoff to identify co-varying LR."
- [discussion] Granger narrowing: "We used 0.01 as a cutoff to get candidate eLR from co-varying LR."
- [discussion] LR pair database source: "Mouse ligand-receptor gene pairs (2034 pairs) were downloaded from CellTalkDB16 (http://tcm.zju.edu.cn/celltalkdb/download.php)."
- [readme] winsz parameter sensitivity: "the Granger causality conclusion is affected by choice of parameter winsz."
- [readme] RunTimeTalk call: "tmp.SCC.cutoff = 0.2,
                               tmp.granger.cutoff = 1e-2)"
