---
name: benjamini-hochberg-multiple-testing-adjustment
description: Use when a hypothesis-testing co-expression method (CS-CORE, Pearson,
  Spearman, locCSN, normalisr, NoiseRegularization) yields one p-value per gene pair
  across a p x p matrix, and the network must be thresholded or PR curves computed.
when_to_use_negative:
- No hypothesis test p-values are available (e.g., methods producing only point correlation
  estimates without inference, or clustering/enrichment steps already operating on
  a binarized network)
- A local (per-pair) rather than global FDR control is required, since BH controls
  the overall false discovery rate across all gene-pair tests jointly
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: CS-CORE
  role: Generates the gene-pair p-values that are BH-adjusted to build the sparse
    cell-type-specific co-expression network
  repo: github.com/ChangSuBiostats/CS-CORE
- name: CSCORE (R package)
  role: R implementation (CSCORE()) producing the est and p-value matrices passed
    to p.adjust(method = "BH")
  repo: github.com/ChangSuBiostats/CS-CORE
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:87e6ac95842ca953464c3a0b402ab732c1fd508aa2fbb8fb551450bc004d60bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CS-CORE
  - CSCORE
  - CSCORE (R package)
  - NoiseRegularization
  - Seurat
  - locCSN
  - normalisr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# benjamini-hochberg-multiple-testing-adjustment

## Summary

Benjamini-Hochberg FDR adjustment applied to the large family of gene-pair p-values produced by co-expression hypothesis tests (e.g., CS-CORE), to control false positives and binarize the inferred co-expression network at a 0.05 threshold.

## When to use

Use when a hypothesis-testing co-expression method (CS-CORE, Pearson, Spearman, locCSN, normalisr, NoiseRegularization) yields one p-value per gene pair across a p x p matrix, and the network must be thresholded or PR curves computed.

## When NOT to use

- No hypothesis test p-values are available (e.g., methods producing only point correlation estimates without inference, or clustering/enrichment steps already operating on a binarized network)
- A local (per-pair) rather than global FDR control is required, since BH controls the overall false discovery rate across all gene-pair tests jointly

## Inputs

- Vector of raw p-values for all upper-triangular gene pairs from a co-expression hypothesis test (e.g., CSCORE_p)
- Raw UMI count matrix used to produce those tests (e.g., subsetted Seurat object for B cells / CD14 monocytes)

## Outputs

- BH-adjusted p-value matrix (p_matrix_BH) over the p x p gene pairs
- Sparse binary/denoised co-expression network with non-significant entries set to 0 (FDR threshold 0.05)
- Per-method adjusted p-values feeding precision-recall curves or Uniform(0,1) Q-Q plots

## How to apply

Collect the p-values for all upper-triangular gene pairs (e.g., CSCORE_p from a CSCORE() run on 5000 top-expressed genes) and apply p.adjust(..., method = "BH") over that vector, mirroring the article's BH-corrected network construction. Set co-expression entries with BH-adjusted p-values above 0.05 to zero to obtain a sparse network (CSCORE_coexp[p_matrix_BH > 0.05] <- 0). In simulation benchmarks (p=500 genes, n=5000 cells), apply BH per method before deriving precision-recall curves so methods are compared at matched FDR control. For null-data calibration checks (permuted PBMC COVID data), aggregate p-values per method, BH-correct them, and inspect Q-Q plots against Uniform(0,1) to quantify type-I error at the 0.05 threshold.

## Related tools

- **CS-CORE** (Generates the gene-pair p-values that are BH-adjusted to build the sparse cell-type-specific co-expression network) — github.com/ChangSuBiostats/CS-CORE
- **CSCORE (R package)** (R implementation (CSCORE()) producing the est and p-value matrices passed to p.adjust(method = "BH")) — github.com/ChangSuBiostats/CS-CORE

## Examples

```
p_matrix_BH <- p.adjust(CSCORE_p[upper.tri(CSCORE_p)], method = "BH")
```

## Evaluation signals

- Number of BH-adjusted p-values equals the number of upper-triangular pairs tested (p*(p-1)/2); none fall below the raw minimum p-value
- Post-thresholding at 0.05, all retained co-expression entries satisfy p_matrix_BH <= 0.05 and all others are exactly 0
- On permuted null PBMC COVID data, a calibrated method's BH-corrected p-values yield Q-Q points on the Uniform(0,1) identity diagonal (empirical type-I error at/near nominal 0.05)
- In simulation benchmarks (p=500, n=5000), per-method PR curves computed from BH-adjusted p-values reproduce CS-CORE achieving the highest AUC among test-based methods

## Limitations

- BH control assumes the test procedure itself is calibrated; existing methods in the paper exhibited inflated type-I errors and biased estimates, so BH-adjustment alone cannot remove their false positives
- Hard-thresholding at BH FDR 0.05 zeroes out non-significant edges, discarding weak but potentially real co-expression signal

## Evidence

- [methods] BH-adjust p-values and set non-significant co-expressions to 0: "p.adjust(CSCORE_p[upper.tri(CSCORE_p)], method = "BH")"
- [methods] Sparse network threshold at BH FDR 0.05: "CSCORE_coexp[p_matrix_BH > 0.05] <- 0"
- [full_text] Type-I error calibration via BH-corrected Q-Q plots on permuted null data: [evidence span withheld — anchor status: not_found]
- [abstract] Inflated false positives motivate FDR-controlled inference: "most existing methods suffered from inflated false positives as well as biased co-expression estimates and clustering analysis"
