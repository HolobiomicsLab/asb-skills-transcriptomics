---
name: ligand-receptor-interaction-scoring
description: Use when testing whether cell-cell communication strength of candidate
  eLR pairs depends on a developmental perturbation, such as ZGA inhibition with alpha-amanitin
  across early/late 2-cell stages.
when_to_use_negative:
- The question concerns maternal-embryo interactions — TimeTalk's autocrine assumption
  in eLR screening precludes analysis of maternal-embryo communication
- The communication of interest is not ligand-receptor mediated (e.g. extracellular-vesicle
  mediated) — the current TimeTalk framework only covers ligand-receptor interactions
- The data lack temporal/trajectory structure or defined developmental stages needed
  for condition-wise interaction-score comparison
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: TimeTalk
  role: R framework whose scripts compute interaction scores per eLR pair on pseudo-bulk
    per-condition expression and support delta-threshold classification of altered
    pairs
  repo: https://github.com/ChengLiLab/TimeTalk
- name: R (version 3.6.3)
  role: Environment for custom scripts computing pseudo-bulk expression per condition
    and for the delta log-ratio calculation
- name: Seurat 3.2.3
  role: Pre-processing of merged expression matrices (CreateSeuratObject, min.features
    = 2000) feeding the scoring workflow
- name: preprocessCore 1.48.0
  role: Quantile normalization (normalize.quantiles) of the merged expression matrix
    before scoring
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:e7a5a25278640377287f58533670a52c48be65943c943db883afd01a13a9082a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - R (version 3.6.3)
  - Seurat 3.2.3
  - TimeTalk
  - preprocessCore 1.48.0
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# ligand-receptor-interaction-scoring

## Summary

Computes TimeTalk interaction scores for early-embryo ligand-receptor pairs (eLRs) on pseudo-bulk per-condition expression and derives delta log-ratio values to classify which eLR pairs change communication strength between conditions, e.g. unperturbed vs alpha-amanitin ZGA-inhibited 2-cell embryos.

## When to use

Use when testing whether cell-cell communication strength of candidate eLR pairs depends on a developmental perturbation, such as ZGA inhibition with alpha-amanitin across early/late 2-cell stages.

## When NOT to use

- The question concerns maternal-embryo interactions — TimeTalk's autocrine assumption in eLR screening precludes analysis of maternal-embryo communication
- The communication of interest is not ligand-receptor mediated (e.g. extracellular-vesicle mediated) — the current TimeTalk framework only covers ligand-receptor interactions
- The data lack temporal/trajectory structure or defined developmental stages needed for condition-wise interaction-score comparison

## Inputs

- ZGA-inhibition RNA-seq expression matrix (GEO GSE71434, alpha-amanitin-treated embryos, ref. 30)
- Unperturbed pre-implantation scRNA-seq expression matrices (early 2-cell and late 2-cell conditions, GEO refs 12, 13)
- Mouse ligand-receptor pair database (CellTalkDB 2034 pairs or Mouse-2020-Shao-LR-pairs.txt)
- Pseudo-bulk per-condition expression vectors

## Outputs

- Interaction score per eLR pair under each condition (early 2-cell, late 2-cell, alpha-amanitin-treated)
- Delta (log-ratio) value per eLR pair comparing ZGA-inhibited vs unperturbed 2-cell interaction scores
- Classified set of significantly altered eLR pairs (expected: 56 pairs)
- Reproduction of Fig. 3e and Supplementary Figs. 12-13 summarizing altered pairs

## How to apply

Download the perturbation expression data (e.g. ZGA-inhibition RNA-seq GSE71434, alpha-amanitin-treated embryos) plus the unperturbed time-course, and compute pseudo-bulk expression for an average single cell in each condition from custom R scripts. Quantile-normalize the merged expression matrix with preprocessCore's normalize.quantiles (v1.48.0). Using TimeTalk's R scripts on the pseudo-bulk per-condition matrices, compute interaction scores for each eLR pair from the ligand-receptor database (2034 mouse pairs from CellTalkDB, or Mouse-2020-Shao-LR-pairs.txt). Derive a delta per eLR pair with the log-ratio formula comparing interaction scores between the perturbed and unperturbed 2-cell conditions, then apply the delta threshold to flag significantly altered pairs — recovering the manuscript's set of 56 altered eLR pairs. Output a summary figure (Fig. 3e) and per-pair tables (Supplementary Figs. 12-13); validate by matching the recovered altered-pair set to the reported 56 pairs.

## Related tools

- **TimeTalk** (R framework whose scripts compute interaction scores per eLR pair on pseudo-bulk per-condition expression and support delta-threshold classification of altered pairs) — https://github.com/ChengLiLab/TimeTalk
- **R (version 3.6.3)** (Environment for custom scripts computing pseudo-bulk expression per condition and for the delta log-ratio calculation)
- **Seurat 3.2.3** (Pre-processing of merged expression matrices (CreateSeuratObject, min.features = 2000) feeding the scoring workflow)
- **preprocessCore 1.48.0** (Quantile normalization (normalize.quantiles) of the merged expression matrix before scoring)

## Examples

```
TimeTalk.result <- RunTimeTalk(tmp.cds=cds.small, tmp.seu=seu.small, LRpairs.df=LRpairs.df, tmp.mra.res=tmp.mra.res, tmp.winsz=0.1, tmp.lags=1, numPts=200, tmp.SCC.cutoff=0.2, tmp.granger.cutoff=1e-2)
```

## Evaluation signals

- The recovered altered-pair set matches the 56 eLR pairs reported in the manuscript's ZGA-inhibition analysis (Fig. 3e)
- Interaction scores are computed on quantile-normalized pseudo-bulk expression per condition, not raw per-cell values
- Delta values follow the defined log-ratio formula comparing ZGA-inhibited vs unperturbed 2-cell interaction scores
- Blocking transcription with alpha-amanitin abolishes the communication strength change between early and late 2-cell stages, consistent with Fig. 3e
- Results are stable across reasonable parameter combinations (authors warn outputs are parameter-sensitive, e.g. winsz affects Granger-based conclusions)

## Limitations

- Granger causality conclusions in the broader TimeTalk pipeline are sensitive to the interpolation parameter winsz (Supplementary Figs. 20-21); users are advised to test multiple parameter combinations
- The autocrine hypothesis in eLR screening excludes maternal-embryo interactions during early development
- Inferred ligand-receptor-to-TF regulatory relationships require wet-lab confirmation, which was not performed; scarcity of high temporal resolution datasets hinders validation

## Evidence

- [discussion] Pseudo-bulk per-condition input: "Custom R scripts were used for the pre-implantation scRNA-seq data to calculate the pseudo-bulk expression for an average single cell in each stage."
- [discussion] Quantile normalization step: "Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0)."
- [discussion] Active LR classification thresholds: "We used SCC_ens > 0.8 or SCC_ens < −0.8 to get active LR."
- [readme] TimeTalk scoring rationale (README): "the Granger causal test to test the ligand-receptor pairs with causal relationships to master transcription factors during development."
