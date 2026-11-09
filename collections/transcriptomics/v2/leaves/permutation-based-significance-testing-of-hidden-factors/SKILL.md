---
name: permutation-based-significance-testing-of-hidden-factors
description: Use when running IA-SVA (or validating a reimplementation) on scRNA-seq
  read counts and you must decide whether an extracted PC1/SV is significant, or how
  many hidden factors (SVs) to retain via the significance index (SI).
when_to_use_negative:
- When the analysis goal is only speed validation or SV estimation with a pre-specified
  number of SVs — run iasva with permute=FALSE, which skips the significance testing
  entirely
- When the data are not scRNA-seq counts adjusted for known covariates (patient identity,
  geometric library size) — the permutation null and PC1 significance test are defined
  on the IA-SVA residual structure
- When many unknown factors are expected and no stringent SI cut-off has been chosen
  — the article notes the SI is not a formal empirical p-value and recommends a stringent
  cut-off be selected by the user
tools:
- name: iasva
  role: Reference implementation whose iasva() function performs the permutation-based
    significance testing (or omits it with permute=FALSE) and returns SV estimates
    to validate against
  repo: github.com/UcarLab/iasva
- name: IRLBA
  role: Provides the partial SVD of the known-factor-adjusted residuals Y' from which
    PC1 and the eigenvalues lambda entering statistic T are obtained
- name: fast_iasva
  role: Alternative fast IA-SVA function that selects important SVs based on the proportion
    of variation explained instead of full permutation testing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:f1a3f0e3b86760ea2f9fb5e6f36fb687adf6eee7ba35abf99eaa4de7a14b7067
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IRLBA
  - SummarizedExperiment
  - fast_iasva
  - iasva
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# permutation-based-significance-testing-of-hidden-factors

## Summary

Permutation-based test from IA-SVA that determines whether the PC1 extracted from known-factor-adjusted single-cell read counts represents a statistically significant hidden factor, using the statistic T = lambda1^2 / sum_k(lambda_k^2) computed against permuted columns of the count matrix.

## When to use

Use when running IA-SVA (or validating a reimplementation) on scRNA-seq read counts and you must decide whether an extracted PC1/SV is significant, or how many hidden factors (SVs) to retain via the significance index (SI).

## When NOT to use

- When the analysis goal is only speed validation or SV estimation with a pre-specified number of SVs — run iasva with permute=FALSE, which skips the significance testing entirely
- When the data are not scRNA-seq counts adjusted for known covariates (patient identity, geometric library size) — the permutation null and PC1 significance test are defined on the IA-SVA residual structure
- When many unknown factors are expected and no stringent SI cut-off has been chosen — the article notes the SI is not a formal empirical p-value and recommends a stringent cut-off be selected by the user

## Inputs

- Log-transformed single-cell read count matrix Y (genes x cells), e.g., alpha-cell counts from iasva_counts_test.Rds loaded via system.file()
- Known-factors design matrix X built with model.matrix(~patient_id + geo_lib_size) from sample annotations (iasva_anns_test.Rds)
- SVs from previous IA-SVA iterations, if testing additional hidden factors
- Number of permutations (paper used 50) and significance cutoff (alpha = 0.05)

## Outputs

- Observed T = lambda1^2 / sum_k(lambda_k^2) statistic for PC1 and its permutation null distribution
- Permutation p-value / significance index (SI) per hidden factor used to decide how many SVs to retain
- Significant PC1 carried forward for per-gene R-squared computation and surrogate variable estimation

## How to apply

Within the six-step IA-SVA loop, first regress the log-transformed gene-by-cell count matrix Y on all known factors (e.g., patient_id and geometric library size, computed as colSums of log(counts+1), combined as model.matrix(~patient_id + geo_lib_size)) plus any surrogate variables from previous iterations, and obtain residuals Y'. Run SVD (via IRLBA) on Y' to extract PC1, then test its significance with the permutation-based statistic T = lambda1^2 / sum_k(lambda_k^2), comparing observed T against T values from permuted columns of Y (the paper's simulations used 50 permutations at alpha = 0.05). If significant, the PC1 is used to identify genes associated with the hidden factor and derive the surrogate variable; the factor is then added as a known factor in the next iteration. The proportion of significant factors becomes the significance index (SI) for choosing the number of SVs, e.g., requesting num.sv=5 via iasva(summ_exp, mod[,-1], permute=TRUE, num.sv=5), or skipping permutation with permute=FALSE for speed. Validation of a reimplementation compares its SV1 against the reference iasva package output by Pearson correlation, benchmarked near the paper's reported average correlation of 0.926 for the brain cell-type SV over 100 runs.

## Related tools

- **iasva** (Reference implementation whose iasva() function performs the permutation-based significance testing (or omits it with permute=FALSE) and returns SV estimates to validate against) — github.com/UcarLab/iasva
- **IRLBA** (Provides the partial SVD of the known-factor-adjusted residuals Y' from which PC1 and the eigenvalues lambda entering statistic T are obtained)
- **fast_iasva** (Alternative fast IA-SVA function that selects important SVs based on the proportion of variation explained instead of full permutation testing)

## Examples

```
iasva(summ_exp, mod[, -1], permute = TRUE, num.sv = 5)  # reference run; use permute = FALSE to skip significance testing
```

## Evaluation signals

- Empirical Type I error should stay near the reported level of 0.04 for IA-SVA (vs 0.09 for USVA/SSVA) at alpha = 0.05 with 50 permutations
- On the brain cell-type benchmark, the resulting SV should reach an average Pearson correlation near 0.926 over 100 runs (SVA/PCA ~0.85)
- A reimplemented SV1 on the alpha-cell data (n = 101) should correlate near 1 with the reference iasva package SV1; record both vectors plus the correlation in sv1_comparison.csv
- Permutation null T values must be generated by permuting columns of Y (cells), not rows/genes, to preserve the gene covariance structure
- When testing multiple factors, a factor only counts toward the SI if its T is significant after earlier significant factors were absorbed as known covariates

## Limitations

- The significance index (SI) is not an empirical p-value, and caveats remain in determining the number of hidden factors
- Users must select a stringent SI cut-off, especially when many unknown factors are expected, but no specific cut-off threshold is provided in the paper
- Permutation testing is the slow mode: the paper benchmarks a No-Permutation mode and fast_iasva for speed, so full permutation is costly on large datasets (e.g., 16,000 genes x 640 cells)
- Empirical Type I error, while best-in-class at 0.04, was evaluated only on simulated data (10,000 genes, 50 cells, Polyester simulations)

## Evidence

- [methods] Permutation test statistic definition: [evidence span withheld — anchor status: not_found]
- [methods] SVD of residuals gives PC1: "Conduct SVD on the obtained residuals ( ′ × Y m n ) and obtain the first PC (PC1)"
- [results] Permutation settings in benchmark simulations: "we generated 1,000 scRNA-seq data sets (10,000 genes and 50 cells) and applied IA-SVA, USVA and SSVA (α = 0.05, 50 permutations) on them"
- [results] Type I error benchmark: "controlling the Type I error rate (0.04 for IA-SVA versus 0.09 for USVA and SSVA)"
- [discussion] SI caveats: "despite its caveats SI is still useful in determining the number of hidden factors in single cell data analyses"
- [readme] IA-SVA framework description: "ii) test the significance of the putative hidden factor for explaining the variation in the data; and iii), if significant, use the estimated factor as an additional known factor"
