---
name: cell-cycle-stage-interpretation-of-marker-genes
description: Use when you need to demonstrate or verify that individual cell cycle
  marker genes cannot classify single cells into cycle stages, motivating a multi-gene,
  genome-wide approach such as reCAT.
when_to_use_negative:
- You already have reCAT's multi-gene stage assignments and need a stage predictor
  rather than a demonstration that single markers are insufficient
- Input data are bulk RNA-seq, qPCR or microarray measurements rather than single-cell
  expression matrices — the insufficiency argument is specific to single-cell variability
- The goal is to order cells along the cycle or segment G0/G1/S/G2/M with scores;
  use reCAT's get_ordIndex/get_score/get_hmm_order workflow instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: reCAT
  role: provides the mESC-SMARTer test data, the log2(TPM+1) preprocessing (get_test_exp),
    and the three-class HMM cell cycle stage assignments used to split cells for the
    violin plots
  repo: github.com/tinglab/reCAT
- name: R
  role: implementation environment for the marker-gene visualization analysis
- name: ggplot2
  role: used to draw the violin plots of each marker gene's expression distribution
    by cell cycle stage
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:1d07fa24dfcdc439a545c9ec46ba022bd7297488653a6c0ca2ed1bb22221bfe9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ggplot2
  - reCAT
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cell-cycle-stage-interpretation-of-marker-genes

## Summary

A diagnostic skill from the reCAT paper showing that expression of single cell cycle marker genes (e.g., Ccne1, Pcna, Cdc6, Hjurp) overlaps strongly across G1, S and G2/M stages in mESC-SMARTer scRNA-seq data, so no single marker can determine a single cell's cycle stage.

## When to use

Use when you need to demonstrate or verify that individual cell cycle marker genes cannot classify single cells into cycle stages, motivating a multi-gene, genome-wide approach such as reCAT.

## When NOT to use

- You already have reCAT's multi-gene stage assignments and need a stage predictor rather than a demonstration that single markers are insufficient
- Input data are bulk RNA-seq, qPCR or microarray measurements rather than single-cell expression matrices — the insufficiency argument is specific to single-cell variability
- The goal is to order cells along the cycle or segment G0/G1/S/G2/M with scores; use reCAT's get_ordIndex/get_score/get_hmm_order workflow instead

## Inputs

- mESC-SMARTer scRNA-seq expression matrix from the reCAT package (e.g., ola_mES_2i.RData)
- Normalized test_exp matrix with values in log2(TPM+1)
- Cell cycle stage assignments per cell (G1, S, G2/M via reCAT HMM with cls_num = 3)
- Marker gene list: Ccne1, Pcna, Cdc6, Hjurp

## Outputs

- Violin plots (ggplot2) of each marker gene's expression distribution split by G1, S and G2/M
- Qualitative assessment of distribution overlap between stages for each single marker

## How to apply

Load the mESC-SMARTer scRNA-seq matrix from the reCAT package (e.g., ola_mES_2i.RData) and normalize it as log2(TPM+1) per the reCAT convention, using get_test_exp to keep only the cycle base genes. Extract per-cell expression of the marker genes Ccne1, Pcna, Cdc6 and Hjurp, then assign each cell to a stage with reCAT's three-class scheme (cls_num = 3: G1, S, G2/M), obtained via get_ordIndex, get_score, and get_hmm_order with a user-judged rdata region file. Plot each marker's distribution split by stage with ggplot2 violin plots. Rationale: the paper's Fig. 1a argument is that one marker gene alone is unreliable, which justifies reCAT's use of 378 cell cycle genes with TSP ordering and HMM segmentation rather than single-gene readouts.

## Related tools

- **reCAT** (provides the mESC-SMARTer test data, the log2(TPM+1) preprocessing (get_test_exp), and the three-class HMM cell cycle stage assignments used to split cells for the violin plots) — github.com/tinglab/reCAT
- **R** (implementation environment for the marker-gene visualization analysis)
- **ggplot2** (used to draw the violin plots of each marker gene's expression distribution by cell cycle stage)

## Examples

```
source("get_test_exp.R"); load("../data/ola_mES_2i.RData"); test_exp <- get_test_exp(t(test_exp))
```

## Evaluation signals

- Every single marker gene (Ccne1, Pcna, Cdc6, Hjurp) shows strongly overlapping expression distributions across G1, S and G2/M in the violin plots
- The reproduced figure matches the paper's Fig. 1a conclusion that one marker gene is not sufficient for reliable stage information in mESC-SMARTer cells
- test_exp values conform to the reCAT convention log2(TPM+1) before marker extraction
- Cells are stratified into the three reCAT default classes (G1, S, G2/M) via get_hmm_order with cls_num = 3

## Limitations

- reCAT relies on a preselected gene set of 378 cycle genes rather than data-driven gene selection; the authors prefer semi-supervised selection for better future performance
- The Bayes-scores and mean-scores used for stage discrimination are noisy and need improvement to be more informative
- Stage assignment quality depends on user-judged rdata regions and a chosen start (myord), typically resolved with get_start

## Evidence

- [intro] single marker insufficiency claim: "we first show one marker gene is not sufficient to give reliable information about cell cycle stages in scRNA-seq data sets"
- [methods] normalization convention: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use get_test_exp function to get standard input data"
- [readme] three-class HMM parameterization: "cls_num is class number you want, generally is 3(G1, S, G2/M) or 4(G0, G1, S, G2/M)"
- [readme] user-judged stage regions: "in ola_mES_2i, we can easily judge cells in c(210:230) is G1, cells in c(40:60) is G2/M, and cells in c(170:190) is S"
