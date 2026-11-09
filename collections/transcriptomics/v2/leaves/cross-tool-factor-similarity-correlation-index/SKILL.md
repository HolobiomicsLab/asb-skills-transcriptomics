---
name: cross-tool-factor-similarity-correlation-index
description: Use when several cell-cell communication scoring methods (e.g., CellChat,
  CellPhoneDB, NATMI, SingleCellSignalR, rank-based cell2cell scores) have been applied
  to the same multi-sample dataset and you need to assess whether Tensor-cell2cell
  decompositions agree across tools.
when_to_use_negative:
- You have only one communication scoring method or a single context/sample — there
  are no multiple factorizations to cross-compare
- Tensors contain many LR pairs or cell types missing in some contexts, since the
  algorithm is not optimized for missing values and shared-members filtering is required
- The goal is tool accuracy benchmarking against ground truth rather than cross-tool
  consistency — the article notes benchmarking CCC prediction tools is challenging
  due to the lack of a ground truth
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Tensor-cell2cell
  role: Decomposes each scoring method's 4D communication tensor via non-negative
    tensor component analysis and provides CorrIndex-based factor-comparison functions
  repo: https://github.com/earmingol/cell2cell
- name: cell2cell
  role: Python suite providing cell2cell.tensor for building 4D communication tensors
    and cell2cell.preprocessing for per-sample preprocessing
  repo: https://github.com/earmingol/cell2cell
- name: LIANA
  role: Runs CellChat, CellPhoneDB, NATMI, and SingleCellSignalR scoring methods on
    each sample to generate comparable communication scores
  repo: https://github.com/saezlab/liana-py
- name: CellChat
  role: Source of the Human-2020-Jin LR pair database used across all scoring methods,
    and one of the compared scoring tools
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:43ec9dc72803dd571f2240aa73b343e5f227483e2f780d80c73b391ea95fdf87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - LIANA
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cross-tool-factor-similarity-correlation-index

## Summary

Quantify whether tensor decompositions of cell-cell communication are consistent across different LR-scoring tools by computing pairwise 1-CorrIndex similarities between the factorizations of multiple 4D communication tensors.

## When to use

Use when several cell-cell communication scoring methods (e.g., CellChat, CellPhoneDB, NATMI, SingleCellSignalR, rank-based cell2cell scores) have been applied to the same multi-sample dataset and you need to assess whether Tensor-cell2cell decompositions agree across tools.

## When NOT to use

- You have only one communication scoring method or a single context/sample — there are no multiple factorizations to cross-compare
- Tensors contain many LR pairs or cell types missing in some contexts, since the algorithm is not optimized for missing values and shared-members filtering is required
- The goal is tool accuracy benchmarking against ground truth rather than cross-tool consistency — the article notes benchmarking CCC prediction tools is challenging due to the lack of a ground truth

## Inputs

- Per-sample single-cell RNA-seq data (e.g., BALF COVID-19 dataset, 63k immune and epithelial cells across 12 patients)
- Communication scores from multiple LR-scoring methods (CellChat, CellPhoneDB, NATMI, SingleCellSignalR, built-in rank-based scores) run via LIANA on the same samples
- A shared ligand-receptor pair list (e.g., Human-2020-Jin-LR-pairs from CellChat/Jin et al. 2021)

## Outputs

- Five decomposed 4D communication tensors (one per scoring method) with non-negative TCA factors
- 5×5 pairwise decomposition similarity matrix of 1-CorrIndex values (e.g., tensor_similarity_scores.csv)
- Mean pairwise similarity statistic (expected ≈ 0.82 for the BALF COVID-19 case)

## How to apply

First, run each communication scoring method (CellChat, CellPhoneDB, NATMI, SingleCellSignalR via LIANA, plus Tensor-cell2cell's built-in rank-based scores) on each context/sample of the same dataset — e.g., the 12 BALF COVID-19 samples (3 control, 3 moderate, 6 severe) — using a common LR database such as the CellChat Human-2020-Jin LR pairs. Build one 4D communication tensor (LR pairs × sender cells × receiver cells × contexts) per method with cell2cell.tensor, retaining only LR pairs and cell types shared across all samples (e.g., 172 shared LR pairs common to all five methods). Decompose each tensor with non-negative tensor component analysis in Tensor-cell2cell, switching init='svd' to init='random' if memory errors occur. Then compute all pairwise 1-CorrIndex similarities between the five factorizations with cell2cell's factor-comparison functions; the BALF case study reports a mean pairwise similarity of approximately 0.82, indicating the method identifies consistent context-dependent communication patterns regardless of input scoring tool.

## Related tools

- **Tensor-cell2cell** (Decomposes each scoring method's 4D communication tensor via non-negative tensor component analysis and provides CorrIndex-based factor-comparison functions) — https://github.com/earmingol/cell2cell
- **cell2cell** (Python suite providing cell2cell.tensor for building 4D communication tensors and cell2cell.preprocessing for per-sample preprocessing) — https://github.com/earmingol/cell2cell
- **LIANA** (Runs CellChat, CellPhoneDB, NATMI, and SingleCellSignalR scoring methods on each sample to generate comparable communication scores) — https://github.com/saezlab/liana-py
- **CellChat** (Source of the Human-2020-Jin LR pair database used across all scoring methods, and one of the compared scoring tools) — https://github.com/sqjin/CellChat

## Examples

```
import cell2cell; similarities = cell2cell.tensor.elbow_rank_comparison(...)  # then compute pairwise 1-CorrIndex between the five decomposed tensors and save as tensor_similarity_scores.csv
```

## Evaluation signals

- The 5×5 pairwise similarity matrix contains off-diagonal 1-CorrIndex values, and the mean off-diagonal similarity is approximately 0.82 (BALF COVID-19 reference value)
- All five tensors share an identical dimensions: exactly the LR pairs and cell types shared across all samples (e.g., 172 shared LR pairs common to all five methods)
- Each tensor decomposition was run as non-negative TCA; if init='svd' raised memory errors, init='random' was used as the documentation recommends
- All methods were scored on the same samples with the same LR database (Human-2020-Jin), so differences in factors reflect scoring methods rather than data or LR-list discrepancies
- Reproducibility check: results match the Code Ocean capsule runs (https://doi.org/10.24433/CO.0051950.v2) with the exact deposited code and software versions

## Limitations

- The implemented tensor decomposition algorithm is not optimized for missing values, so only LR pairs and cell types shared across all samples can be included, potentially missing rare cell types
- Consistency (high 1-CorrIndex similarity) does not establish correctness — benchmarking CCC prediction tools is challenging due to the lack of a ground truth and qualitative differences in tool outputs
- Decomposition with init='random' (used in place of init='svd' under memory constraints) may add run-to-run variability in factorizations

## Evidence

- [results] Tensor-cell2cell consistently identifies context-dependent communication patterns regardless of which communication scoring method is used as input: [evidence span withheld — anchor status: not_found]
- [discussion] One can choose any tool of interest, run it on each context separately, and use the resulting communication scores to build and deconvolve a 4D-communication tensor.: "One can choose any tool of interest, run it on each context separately, and use the resulting communication scores to build and deconvolve a 4D-communication tensor."
- [results] a lung dataset of 63k immune and epithelial cells across three control, three moderate, and six severe COVID-19 patients: "a lung dataset of 63k immune and epithelial cells across three control, three moderate, and six severe COVID-19 patients"
- [discussion] we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
- [methods] Reproducible runs of the analyses in the Tensor-cell2cell paper are available at CodeOcean.com (https://doi.org/10.24433/CO.0051950.v2): [evidence span withheld — anchor status: not_found]
