---
name: pseudotime-ordered-expression-extraction
description: Use when time-course scRNA-seq data have been trajectory-ordered (e.g.
  monocle3 pseudotime) and you need aligned per-gene expression series along pseudotime
  to test ligand-receptor co-variation.
when_to_use_negative:
- When definitive cell types exist and pairwise communication between given cell types
  is the goal — tools like CellPhoneDB or CellChat are designed for that setting instead
- When data lack trajectory/pseudotime ordering and cannot be meaningfully ordered
  along a developmental axis
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: TimeTalk
  role: R package whose RunTimeTalk function extracts pseudotime-ordered ligand/receptor
    series, interpolates them, and applies the PCC/SCC co-varying LR filter
  repo: https://github.com/ChengLiLab/TimeTalk
- name: monocle3
  role: Provides the trajectory reconstruction and pseudotime ordering of cells that
    define the abscissa for expression series extraction
  repo: https://github.com/cole-trapnell-lab/monocle3
- name: Seurat
  role: Holds the expression matrix and cell metadata (assay, ident) used to pull
    ligand and receptor expression per ordered cell
- name: cellAlign
  role: Dependency providing the interpolation strategy for aligning expression series
    onto a common grid
  repo: https://github.com/shenorrLab/cellAlign
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:ac5e64e6a92998c70c1469e84e1866e4220a8261bb64e37cf1fd307007944d07
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
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# pseudotime-ordered-expression-extraction

## Summary

Extract ligand and receptor expression series from pseudotime-ordered single-cell RNA-seq cells and interpolate them onto a common grid so Pearson correlations can identify co-varying LR pairs in the TimeTalk workflow.

## When to use

Use when time-course scRNA-seq data have been trajectory-ordered (e.g. monocle3 pseudotime) and you need aligned per-gene expression series along pseudotime to test ligand-receptor co-variation.

## When NOT to use

- When definitive cell types exist and pairwise communication between given cell types is the goal — tools like CellPhoneDB or CellChat are designed for that setting instead
- When data lack trajectory/pseudotime ordering and cannot be meaningfully ordered along a developmental axis

## Inputs

- monocle3 cell dataset (cds) with pseudotime-ordered cells (e.g. cds_demo.rds)
- matching Seurat object (e.g. seu_demo.rds)
- CellTalkDB-derived mouse ligand-receptor pair table (Mouse-2020-Shao-LR-pairs.txt, 2034 pairs)

## Outputs

- Interpolated ligand and receptor expression series on a common pseudotime grid (numPts = 200, winsz = 0.1)
- PCC values per LR pair with |PCC| > 0.1 co-varying LR identifiers (e.g. covarying_lr_pairs.csv)

## How to apply

Load the monocle3 cell dataset (cds) and matching Seurat object, then order cells along the reconstructed trajectory pseudotime. For each pair in the CellTalkDB-derived ligand-receptor table (e.g. Mouse-2020-Shao-LR-pairs.txt, 2034 mouse LR pairs), extract the ligand and receptor expression series across the pseudotime-ordered cells and interpolate both onto a common grid using the RunTimeTalk parameters numPts = 200 and winsz = 0.1 (cellAlign-style interpolation). Interpolation is essential because cells at the same pseudotime across the two compared cell populations (e.g. EPI vs PE) are not sample-matched, so a shared abscissa is needed before computing correlation. Compute the Pearson correlation coefficient (PCC) between the interpolated ligand and receptor series for each pair, and pass |PCC| = 0.1 as the co-varying LR cutoff for the standard TimeTalk workflow (the extended analog uses tmp.SCC.cutoff = 0.2). Write the retained co-varying LR identifiers — e.g. filter TimeTalk.result on category == 'PASS' and pull unique LR values — to an output such as covarying_lr_pairs.csv.

## Related tools

- **TimeTalk** (R package whose RunTimeTalk function extracts pseudotime-ordered ligand/receptor series, interpolates them, and applies the PCC/SCC co-varying LR filter) — https://github.com/ChengLiLab/TimeTalk
- **monocle3** (Provides the trajectory reconstruction and pseudotime ordering of cells that define the abscissa for expression series extraction) — https://github.com/cole-trapnell-lab/monocle3
- **Seurat** (Holds the expression matrix and cell metadata (assay, ident) used to pull ligand and receptor expression per ordered cell)
- **cellAlign** (Dependency providing the interpolation strategy for aligning expression series onto a common grid) — https://github.com/shenorrLab/cellAlign

## Examples

```
TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small, tmp.seu=seu.small, LRpairs.df=LRpairs.df, tmp.mra.res=tmp.mra.res, tmp.winsz=0.1, numPts=200, tmp.SCC.cutoff=0.2, tmp.granger.cutoff=1e-2)
```

## Evaluation signals

- Number of co-varying LR pairs retained after the |PCC| = 0.1 cutoff should be a strict subset of the 2034 input CellTalkDB mouse pairs
- Interpolated series must all have length numPts = 200 on the common grid before correlation is computed
- Filtering TimeTalk.result on category == 'PASS' and pulling unique LR values should reproduce the co-varying LR identifier list
- Known pairs such as Fgf4-Fgfr1 and Fgf4-Fgfr2, previously reported in the literature, should be recoverable among identified eLRs

## Limitations

- The Granger causality conclusion is affected by the choice of parameter winsz; users are advised to test multiple parameter combinations
- The autocrine hypothesis in eLR screening precludes exploring maternal-embryo interactions during early development
- Inferred LR–TF regulatory relationships require wet-lab experimental confirmation, which was not performed

## Evidence

- [readme] For each LR pair, extract the ligand and receptor expression series across the pseudotime-ordered cells, interpolating onto a common grid (numPts = 200, winsz = 0.1 as in the RunTimeTalk call).: [evidence span withheld — anchor status: not_found]
- [discussion] Cutoff definition for co-varying LR pairs from the article discussion.: "we calculated the Pearson’s correlation coefficient (PCC) and used |PCC| = 0.1 as a cutoff to get co-varying LR."
- [abstract] Interpolation strategy rationale from the abstract.: "With the interpolation strategy, TimeTalk is powerful in revealing paracrine settings and studying cell-cell communication during early embryo development"
- [readme] Extended-workflow SCC cutoff used in the RunTimeTalk call.: "tmp.SCC.cutoff = 0.2,
                               tmp.granger.cutoff = 1e-2)"
