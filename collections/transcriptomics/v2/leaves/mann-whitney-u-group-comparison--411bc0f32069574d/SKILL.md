---
name: mann-whitney-u-group-comparison
description: Use when comparing a non-normally distributed continuous variable (e.g.,
  predicted |theta1−theta2| from CLOUDe .predictions files) between two groups (Redundant
  vs Unique deletion classes) without assuming normality.
when_to_use_negative:
- Input groups are paired per-observation or the data are approximately normal with
  equal variances, where a t-test would be more powerful
- Comparison requires >2 groups or covariate adjustment — Mann–Whitney U is strictly
  a two-sample rank test
- Expression data have not yet been processed by CLOUDe (no .classifications/.predictions
  files), so class labels and predicted parameters do not exist
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Environment in which per-event CLOUDe outputs are loaded and Mann–Whitney
    U, Wilcoxon signed-rank, and exact binomial tests are run
  repo: https://github.com/anddssan/CLOUDe
- name: dplyr
  role: Data wrangling to join classifications and predictions per deletion event
    and group parameter values by class before testing
  repo: https://github.com/anddssan/CLOUDe
- name: CLOUDe
  role: Generates the predicted classes and evolutionary parameters (theta1, theta2,
    log10(sigmaSq/(2*alpha))) that constitute the tested variables
  repo: https://github.com/anddssan/CLOUDe
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:bed3336e442ad942204a0b11fa9c5a6b7237e90d55d792929816793c80b7d17d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - dplyr
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# mann-whitney-u-group-comparison

## Summary

Apply Mann–Whitney U tests (and related Wilcoxon/binomial tests in R) to compare CLOUDe-predicted OU parameters — |theta1−theta2| and log10(sigmaSq/(2*alpha)) — between 'Redundant' and 'Unique' deletion classes, and to validate predictions against reported significance values.

## When to use

Use when comparing a non-normally distributed continuous variable (e.g., predicted |theta1−theta2| from CLOUDe .predictions files) between two groups (Redundant vs Unique deletion classes) without assuming normality.

## When NOT to use

- Input groups are paired per-observation or the data are approximately normal with equal variances, where a t-test would be more powerful
- Comparison requires >2 groups or covariate adjustment — Mann–Whitney U is strictly a two-sample rank test
- Expression data have not yet been processed by CLOUDe (no .classifications/.predictions files), so class labels and predicted parameters do not exist

## Inputs

- test_prefix.*.classifications (predicted 'Redundant'/'Unique' class per deletion event)
- test_prefix.*.predictions (predicted theta1, theta2, log10(sigmaSq/(2*alpha)) per deletion event)
- EmpiricalData.data (space-delimited expression file, N+1 rows x 3m columns: derived, survived, lost gene columns)
- Simulated L-gene expression from the OU training process for the empirical-vs-simulated comparison

## Outputs

- summary_statistics.csv (test names, statistics, p-values per comparison)
- Mann–Whitney U p-value for |theta1−theta2| Redundant vs Unique (expected <2.22e-16)
- Wilcoxon signed-rank p-values for log10(sigmaSq/(2*alpha)) within each class
- Exact binomial p-values for lethal-phenotype enrichment (genome-wide and Unique-only)

## How to apply

Load per-event CLOUDe outputs in R: predicted classes from test_prefix.*.classifications and the 3m evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) from test_prefix.*.predictions, joining them per deletion event. Compute |theta1−theta2| per event and run a two-sided Mann–Whitney U test (wilcox.test in R) comparing Redundant vs Unique; the paper reports p<2.22e-16, consistent with the OU model predicting larger expression-optima differences for uniquely deleted genes. Complement this with Wilcoxon signed-rank tests on log10(sigmaSq/(2*alpha)) within each class (reported p=6.96e-46 Redundant, p=7.30e-56 Unique), genome-wide and Unique-only lethal-phenotype exact binomial tests (p=4.81e-2 and p=3.65e-1), and a Mann–Whitney test of empirical vs simulated lost-gene (L) expression (p=2.70e-11; not significant after filtering FPKM<1). Assemble all test names, statistics, and p-values into summary_statistics.csv and check each against the reported values as the acceptance criterion.

## Related tools

- **R** (Environment in which per-event CLOUDe outputs are loaded and Mann–Whitney U, Wilcoxon signed-rank, and exact binomial tests are run) — https://github.com/anddssan/CLOUDe
- **dplyr** (Data wrangling to join classifications and predictions per deletion event and group parameter values by class before testing) — https://github.com/anddssan/CLOUDe
- **CLOUDe** (Generates the predicted classes and evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) that constitute the tested variables) — https://github.com/anddssan/CLOUDe

## Examples

```
wilcox.test(abs(theta1 - theta2) ~ class, data = cloude_results)
```

## Evaluation signals

- Mann–Whitney U test of |theta1−theta2| between Redundant and Unique classes yields p<2.22e-16, with larger values for the Unique class
- Wilcoxon signed-rank tests on log10(sigmaSq/(2*alpha)) reproduce p=6.96e-46 (Redundant) and p=7.30e-56 (Unique)
- Genome-wide lethal-phenotype exact binomial test reproduces p=4.81e-2; Unique-only test reproduces p=3.65e-1
- Mann–Whitney test of empirical vs simulated L-gene expression reproduces p=2.70e-11, and is not significant after filtering genes with FPKM<1
- summary_statistics.csv is populated with all test names, statistics, and p-values matching the reported values

## Limitations

- The empirical-vs-simulated L-gene expression signal (p=2.70e-11) is not significant after filtering genes with FPKM<1, so results are sensitive to low-expression filtering
- Random forest and SVM models are trained on-the-fly, so predictions feeding the tests are algorithm-dependent; the neural network architecture gives optimal performance per the paper
- The Unique-only lethal-phenotype binomial test is not significant (p=3.65e-1), so genome-wide significance should not be over-interpreted per class

## Evidence

- [other] Run a Mann-Whitney U test comparing |theta1-theta2| between the Redundant and Unique deletion classes (reported as p<2.22e-16).: [evidence span withheld — anchor status: not_found]
- [other] wilcoxon and binomial tests: [evidence span withheld — anchor status: not_found]
- [readme] expression file format: "A space-delimited file with N+1 rows and 3m columns, where N is the number of genes and m is the number of conditions"
- [methods] CLOUDe predictions available: "CLOUDe can predict classes ("Redundant" or "Unique") and evolutionary parameters (theta1, theta2, and log10(sigmaSq/(2*alpha))) for each gene in a test dataset."
- [intro] article workflow context: "CLOUDe is an R implementation of Campelo dos Santos, DeGiorgio and Assis' (2023) suite of machine learning methods"
