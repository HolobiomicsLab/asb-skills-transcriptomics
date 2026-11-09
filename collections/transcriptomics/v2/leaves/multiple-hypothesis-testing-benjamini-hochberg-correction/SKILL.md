---
name: multiple-hypothesis-testing-benjamini-hochberg-correction
description: Use when a family of m raw p-values (e.g. from RgnTX permutation tests
  over m6A site groups) must be FDR-adjusted before declaring colocalization significant
  at alpha=0.05.
when_to_use_negative:
- Only a single hypothesis test is present — no family of m p-values to correct, so
  FDR adjustment is inappropriate
- The input is already an adjusted p-value vector or a q-value/FDR table rather than
  raw p-values
- The analysis requires family-wise error rate control (e.g. Bonferroni guarantees)
  rather than FDR control
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: RgnTX
  role: R/Bioconductor package that runs the permutation tests whose p-values are
    corrected and provides multiple hypothesis testing under Benjamini-Hochberg correction
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: R
  role: Environment for implementing the stepwise BH procedure and cross-checking
    via p.adjust(method='BH')
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:c1e3aaf2bcd99329c5c0cc3614a1a566cfbc61639a4c94ce6ba43e54a622e3bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# multiple-hypothesis-testing-benjamini-hochberg-correction

## Summary

Applies the Benjamini-Hochberg step-up FDR procedure to adjust raw permutation p-values (e.g. from RgnTX colocalization tests), computing adjusted p-values via pFDR(i)=p(i)*m/i with backward min-accumulation and rejecting the largest k with p(k) <= k*alpha/m to control false positives across many tested region-set pairs.

## When to use

Use when a family of m raw p-values (e.g. from RgnTX permutation tests over m6A site groups) must be FDR-adjusted before declaring colocalization significant at alpha=0.05.

## When NOT to use

- Only a single hypothesis test is present — no family of m p-values to correct, so FDR adjustment is inappropriate
- The input is already an adjusted p-value vector or a q-value/FDR table rather than raw p-values
- The analysis requires family-wise error rate control (e.g. Bonferroni guarantees) rather than FDR control

## Inputs

- Numeric vector of m raw p-values from permutation tests (e.g. RgnTX permTestTx / permTestTxIA_customPick p-value field)
- Significance level alpha (e.g. 0.05)

## Outputs

- Vector of m BH-adjusted p-values (monotonically non-decreasing with rank)
- Set of rejected null hypotheses (largest k with p(k) <= k*alpha/m) and rejection proportion
- Side-by-side comparison table (e.g. bh_adjustment_check.csv) with raw p, custom BH adjusted p, RgnTX adjusted p, and agreement flags

## How to apply

Sort the m raw p-values in ascending order and compute candidate corrected values pFDR(i)=p(i)*m/i for each rank i. Apply a backward cumulative minimum from the (m-1)-th item to the first: pFDR(i)=min(pFDR(i), pFDR(i+1)), enforcing monotonicity. Determine the rejection set as the largest k satisfying p(k) <= k*alpha/m (alpha=0.05 in the RgnTX m6A analysis). Cross-validate the implementation against R's built-in p.adjust(method='BH') and RgnTX::adjustMultipleTesting on the same input vector, requiring element-wise agreement within ~1e-10 and identical rejection sets. In the RgnTX paper this correction was applied to p-values from permutation tests run on independent groups of 100 m6A regions (100 permutations each) tested against stop codon regions.

## Related tools

- **RgnTX** (R/Bioconductor package that runs the permutation tests whose p-values are corrected and provides multiple hypothesis testing under Benjamini-Hochberg correction) — https://github.com/yue-wang-biomath/RgnTX
- **R** (Environment for implementing the stepwise BH procedure and cross-checking via p.adjust(method='BH'))

## Examples

```
sorted_p <- sort(raw_p); pFDR <- p.adjust(raw_p, method = "BH"); rejected <- sum(sorted_p <= seq_along(sorted_p) * 0.05 / length(raw_p))
```

## Evaluation signals

- Every custom BH adjusted p-value matches RgnTX::adjustMultipleTesting output within absolute tolerance ~1e-10
- Custom adjusted p-values and R's p.adjust(method='BH') agree element-wise on the same synthetic p-value vector
- Adjusted p-values are monotonically non-decreasing once p-values are sorted in ascending order
- Both the custom implementation and RgnTX reject the identical set of hypotheses at alpha=0.05 (identical rejection proportion)
- Comparison written to bh_adjustment_check.csv contains raw p, both adjusted columns, agreement flags, and rejection proportions with no missing entries

## Limitations

- BH correction assumes valid, (weakly independent or positively dependent) p-values; permutation p-values from few permutations (e.g. ntimes=50) are granular, limiting resolution of small adjusted values
- Article and repository state no changelog, version number, or R/dependency versions, so exact numerical behavior across package versions is not pinned
- Correction controls false discovery rate, not family-wise error rate; individual rejections can still be false positives

## Evidence

- [abstract] easy to be utilized by users for visualizing permutation results, calculating shifted z-scores and conducting multiple hypothesis testing under Benjamini-Hochberg correction.: "easy to be utilized by users for visualizing permutation results, calculating shifted z-scores and conducting multiple hypothesis testing under Benjamini-Hochberg correction."
- [results] RgnTX provided Benjamini-Hochberg correction to adjust the p-values, reducing the number of false positives: "RgnTX provided Benjamini-Hochberg correction to adjust the p-values, reducing the number of false positives"
- [results] We randomly shuffled all the m6A sites and divided them into independent groups. Each independent group contains 100 regions: "We randomly shuffled all the m6A sites and divided them into independent groups. Each independent group contains 100 regions"
