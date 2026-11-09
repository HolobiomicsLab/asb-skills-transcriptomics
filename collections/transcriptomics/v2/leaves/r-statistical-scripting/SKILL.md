---
name: r-statistical-scripting
description: Use when a workflow requires implementing, reproducing, or independently
  validating a statistical procedure (e.g. multiple-testing correction) in R, and
  comparing a hand-coded implementation against package output within numerical tolerance.
when_to_use_negative:
- The p-values have not yet been computed — this skill adjusts existing p-values,
  it does not run the underlying permutation tests (e.g. permTestTx) that produce
  them
- The required correction is not FDR/BH (e.g. family-wise error control such as Bonferroni)
  — the reconstructed procedure is specifically the Benjamini-Hochberg step-up FDR
  method
- Analysis is a genome-based colocalization test (GREAT, LOLA, GenomeRunner) rather
  than R-style transcriptome-level testing, and no R statistical scripting is needed
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: RgnTX
  role: R/Bioconductor package whose adjustMultipleTesting function implements the
    Benjamini-Hochberg correction being reconstructed and cross-validated
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: R
  role: Scripting environment in which the synthetic p-values, the hand-coded BH procedure,
    and the p.adjust(method='BH') cross-reference are implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:3e3bb977f4750779b027c1b31b2fb40b7a84f0193eac3e79abcce1a41d6faeef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# r-statistical-scripting

## Summary

Implement and verify statistical procedures in R/Bioconductor, exemplified by reconstructing RgnTX's Benjamini-Hochberg multiple-testing correction (sorting m p-values, step-up pFDR(i)=p(i)*m/i, backward min-accumulation, rejection at p(k) <= k*alpha/m) and cross-checking it against p.adjust(method='BH').

## When to use

Use when a workflow requires implementing, reproducing, or independently validating a statistical procedure (e.g. multiple-testing correction) in R, and comparing a hand-coded implementation against package output within numerical tolerance.

## When NOT to use

- The p-values have not yet been computed — this skill adjusts existing p-values, it does not run the underlying permutation tests (e.g. permTestTx) that produce them
- The required correction is not FDR/BH (e.g. family-wise error control such as Bonferroni) — the reconstructed procedure is specifically the Benjamini-Hochberg step-up FDR method
- Analysis is a genome-based colocalization test (GREAT, LOLA, GenomeRunner) rather than R-style transcriptome-level testing, and no R statistical scripting is needed

## Inputs

- Vector of m raw p-values (numeric, e.g. generated with runif under a fixed seed)
- Significance threshold alpha (e.g. 0.05)
- Optionally, permutation test result objects from RgnTX (e.g. permTestTx output containing p-values)

## Outputs

- Vector of BH-adjusted p-values from the custom implementation
- Vector of adjusted p-values from RgnTX::adjustMultipleTesting and p.adjust(method='BH')
- Rejection set / rejection proportion at the chosen alpha
- Side-by-side comparison table written to bh_adjustment_check.csv (raw p, custom BH adjusted p, adjustMultipleTesting adjusted p, agreement flags, rejection proportions)

## How to apply

Write an R script that generates a controlled synthetic input, such as a vector of m raw p-values via runif with a fixed seed, so results are reproducible. Implement the procedure step by step: for BH, sort p-values ascending, compute candidate adjusted values pFDR(i)=p(i)*m/i, apply a backward cumulative minimum from the (m-1)-th item to the first, and determine the rejection set as the largest k with p(k) <= k*alpha/m (alpha=0.05). Run the package function under test (RgnTX::adjustMultipleTesting) and an authoritative cross-reference (R's built-in p.adjust(method='BH')) on the same input. Compare all three element-wise and write side-by-side results (raw p, custom adjusted p, package adjusted p, agreement flags, rejection proportions) to a CSV such as bh_adjustment_check.csv. This mirrors how RgnTX itself packages 'multiple hypothesis testing under Benjamini-Hochberg correction' as a post-processing step after permutation tests.

## Related tools

- **RgnTX** (R/Bioconductor package whose adjustMultipleTesting function implements the Benjamini-Hochberg correction being reconstructed and cross-validated) — https://github.com/yue-wang-biomath/RgnTX
- **R** (Scripting environment in which the synthetic p-values, the hand-coded BH procedure, and the p.adjust(method='BH') cross-reference are implemented)

## Examples

```
p <- runif(100); adj_custom <- sort(p) * length(p) / seq_along(p); adj_custom <- cummin(rev(adj_custom))[rev(seq_along(p))]; adj_pkg <- p.adjust(p, method='BH')
```

## Evaluation signals

- Every custom adjusted p-value matches RgnTX::adjustMultipleTesting within numerical tolerance (e.g. absolute difference <= 1e-10)
- Both the custom implementation and RgnTX reject the same set of hypotheses (identical rejection proportion at alpha=0.05)
- Custom BH output also matches R's built-in p.adjust(method='BH') element-wise on the same input
- Adjusted p-values are monotone non-decreasing with sorted raw p-values (enforced by the backward min-accumulation step)
- Results are reproducible across runs because the synthetic p-values are generated with a fixed random seed

## Limitations

- The README and article do not state version numbers for RgnTX, R, or dependencies, so reproducibility depends on recording package versions manually
- Permutation-based p-values in RgnTX depend on the number of permutations (e.g. ntimes = 50 in the README examples), so low permutation counts yield coarse p-values that limit correction precision
- No changelog is available for the repository, making it hard to detect behavior changes in correction functions over time

## Evidence

- [abstract] easy to be utilized by users for visualizing permutation results, calculating shifted z-scores and conducting multiple hypothesis testing under Benjamini-Hochberg correction: "easy to be utilized by users for visualizing permutation results, calculating shifted z-scores and conducting multiple hypothesis testing under Benjamini-Hochberg correction."
- [results] RgnTX provided Benjamini-Hochberg correction to adjust the p-values, reducing the number of false positives: "RgnTX provided Benjamini-Hochberg correction to adjust the p-values, reducing the number of false positives"
- [abstract] Here, we developed RgnTX, an R/Bioconductor tool for the colocalization analysis of transcriptome elements with permutation tests.: "Here, we developed RgnTX, an R/Bioconductor tool for the colocalization analysis of transcriptome elements with permutation tests."
- [readme] The function permTestTx performs permutation test to evaluate association between region set 'RS1' and region set 'RS2'.: [evidence span withheld — anchor status: external_doc]
