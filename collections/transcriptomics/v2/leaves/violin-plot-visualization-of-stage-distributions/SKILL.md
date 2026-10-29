---
name: violin-plot-visualization-of-stage-distributions
description: Use when you need to visually assess whether one marker gene's expression
  distribution discriminates cell cycle stages in single-cell RNA-seq data (overlapping
  violins = insufficient marker).
when_to_use_negative:
- You need reliable per-cell stage assignment from expression — the violins show distributions
  overlap, so a single marker gene cannot determine a single cell's stage; use the
  reCAT TSP+HMM pipeline instead
- Your data is bulk RNA-seq or synchronized-cell data, where stage is already known
  or averaged out
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: reCAT
  role: provides the mESC-SMARTer example data (ola_mES_2i.RData) and the get_test_exp/get_hmm_order
    functions that produce the log2(TPM+1) normalized matrix and the 3-class (G1,
    S, G2/M) stage assignments used to split the violins
  repo: github.com/tinglab/reCAT
- name: R
  role: implementation environment in which reCAT and the plotting are run
- name: ggplot2
  role: draws the violin plots of marker gene expression distributions per cell cycle
    stage
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:e47a785124200c47f93ee3c8c160af2f3b7adfc63f05a32add9bbaa2f07bb5d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ggplot2
  - reCAT
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# violin-plot-visualization-of-stage-distributions

## Summary

Draw ggplot2 violin plots of individual cell cycle marker gene expression (Ccne1, Pcna, Cdc6, Hjurp) split by reCAT-assigned cell cycle stage (G1, S, G2/M) on the mESC-SMARTer scRNA-seq data, to demonstrate that a single marker gene is insufficient to determine a single cell's stage.

## When to use

Use when you need to visually assess whether one marker gene's expression distribution discriminates cell cycle stages in single-cell RNA-seq data (overlapping violins = insufficient marker).

## When NOT to use

- You need reliable per-cell stage assignment from expression — the violins show distributions overlap, so a single marker gene cannot determine a single cell's stage; use the reCAT TSP+HMM pipeline instead
- Your data is bulk RNA-seq or synchronized-cell data, where stage is already known or averaged out

## Inputs

- mESC-SMARTer scRNA-seq count matrix from reCAT /data (e.g. ola_mES_2i.RData)
- DESeq size-factor normalized, log2(TPM+1) expression matrix (test_exp)
- reCAT HMM stage assignments (cls_num = 3: G1, S, G2/M) from get_hmm_order
- Per-cell expression values of marker genes Ccne1, Pcna, Cdc6, Hjurp

## Outputs

- ggplot2 violin plots of each marker gene's expression distribution split by cell cycle stage (Fig. 1a-style panel)

## How to apply

Load the mESC-SMARTer expression matrix from the reCAT package data (e.g. ola_mES_2i.RData) and normalize to log2(TPM+1) using DESeq size-factor normalization, following the reCAT convention that test_exp values are log2(TPM+1) and can be standardised with the get_test_exp function. Extract per-cell expression of the four marker genes Ccne1, Pcna, Cdc6 and Hjurp from the normalized matrix. Assign each cell a stage with the reCAT HMM via get_hmm_order using the three-class scheme cls_num = 3 (G1, S, G2/M), supplying the bayes_score, mean_score, ordIndex, myord start and rdata region table (e.g. in ola_mES_2i, G1 c(210:230), G2/M c(40:60), S c(170:190)). Plot each marker's expression distribution as a violin plot split by stage with ggplot2. The figure is correct if every marker's violins overlap strongly across G1, S and G2/M, reproducing Fig. 1a's conclusion that one marker gene is not sufficient to give reliable cell cycle stage information.

## Related tools

- **reCAT** (provides the mESC-SMARTer example data (ola_mES_2i.RData) and the get_test_exp/get_hmm_order functions that produce the log2(TPM+1) normalized matrix and the 3-class (G1, S, G2/M) stage assignments used to split the violins) — github.com/tinglab/reCAT
- **R** (implementation environment in which reCAT and the plotting are run)
- **ggplot2** (draws the violin plots of marker gene expression distributions per cell cycle stage)

## Examples

```
source("get_test_exp.R"); load("../data/ola_mES_2i.RData"); test_exp <- get_test_exp(t(test_exp)); hmm_result <- get_hmm_order(bayes_score = score_result$bayes_score, mean_score = score_result$mean_score, ordIndex = ordIndex, cls_num = 3, myord = c(4:1, 295:5), rdata = rdata)
```

## Evaluation signals

- For each of Ccne1, Pcna, Cdc6 and Hjurp, the violin distributions across G1, S and G2/M overlap strongly (no marker cleanly separates stages), matching the paper's Fig. 1a conclusion
- Expression matrix values are on the log2(TPM+1) scale and were DESeq size-factor normalized per the reCAT test_exp convention
- Every cell in the plot carries exactly one of the three reCAT classes (cls_num = 3: G1, S, G2/M)
- The reproducible figure visually matches the published single-marker insufficiency illustration from the mESC-SMARTer dataset

## Limitations

- A single marker gene is not sufficient to give reliable information about cell cycle stages in scRNA-seq data sets — this skill demonstrates insufficiency, not a working classifier
- Stage labels depend on reCAT's scoring metrics, which the authors note are noisy ('Bayes-scores and mean-scores ... need improvements to be less noisy and more informative')
- reCAT uses a preselected gene set (378 genes) rather than data-driven gene selection, which the authors say may limit performance

## Evidence

- [intro] one marker gene is not sufficient to give reliable information about cell cycle stages in scRNA-seq data sets: "we first show one marker gene is not sufficient to give reliable information about cell cycle stages in scRNA-seq data sets"
- [readme] test_exp log2(TPM+1) and get_test_exp: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
- [readme] get_hmm_order with cls_num = 3 classes: "you can use ***get_hmm_order*** function
to sort the cells"
- [readme] cls_num = 3 (G1, S, G2/M) and rdata regions for ola_mES_2i: "in ola_mES_2i, we can easily judge cells in c(210:230) is G1, cells in c(40:60) is G2/M, and cells in c(170:190) is S"
- [discussion] scoring metrics are noisy: "The scoring metrics (i.e., Bayes-scores and mean-scores) to indicate cell cycle stages also need improvements to be less noisy and more informative"
