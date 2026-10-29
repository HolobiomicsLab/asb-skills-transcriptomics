---
name: ou-evolutionary-parameter-interpretation
description: Use when per-event CLOUDe predictions (test_prefix.*.classifications
  and test_prefix.*.predictions with theta1, theta2, log10(sigmaSq/(2*alpha))) exist
  and you need to test whether predicted parameters differ between 'Redundant' and
  'Unique' deletion classes.
when_to_use_negative:
- You only need CLOUDe class or parameter predictions themselves — use the CLOUDeClassify*/CLOUDePredict*
  functions instead of the downstream significance-test suite
- The expression data are not from a two-species deletion-event design with derived/survived/lost
  genes per event (the 3m-column input format), since the OU parameter interpretation
  does not apply
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CLOUDe
  role: Produces the per-event predicted classes and OU evolutionary parameters that
    the significance tests operate on
  repo: https://github.com/anddssan/CLOUDe
- name: R
  role: Environment in which CLOUDe runs and the Mann–Whitney U, Wilcoxon signed-rank,
    and exact binomial tests are performed
- name: dplyr
  role: Required R library for CLOUDe, used for manipulating the per-event classification
    and prediction tables before testing
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:36d1ea206ce71b9d7e22629357390ebf7df7e16baa82635f013e201c3636f503
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - dplyr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ou-evolutionary-parameter-interpretation

## Summary

Statistically tests CLOUDe's predicted Ornstein–Uhlenbeck parameters — the absolute difference between expression optima |theta1−theta2| and the log-transformed stationary variance log10(sigmaSq/(2*alpha)) — between 'Redundant' and 'Unique' deletion classes, validating that the machine-learning predictions recapitulate OU-model expectations.

## When to use

Use when per-event CLOUDe predictions (test_prefix.*.classifications and test_prefix.*.predictions with theta1, theta2, log10(sigmaSq/(2*alpha))) exist and you need to test whether predicted parameters differ between 'Redundant' and 'Unique' deletion classes.

## When NOT to use

- You only need CLOUDe class or parameter predictions themselves — use the CLOUDeClassify*/CLOUDePredict* functions instead of the downstream significance-test suite
- The expression data are not from a two-species deletion-event design with derived/survived/lost genes per event (the 3m-column input format), since the OU parameter interpretation does not apply

## Inputs

- test_prefix.*.classifications file (per-deletion-event predicted class: Redundant or Unique)
- test_prefix.*.predictions file (per-deletion-event evolutionary parameters theta1, theta2, log10(sigmaSq/(2*alpha)))
- Empirical expression data in EmpiricalData.data format (space-delimited, N+1 rows by 3m columns: derived, survived, lost gene columns per condition)

## Outputs

- Mann–Whitney U test result comparing |theta1−theta2| between Redundant and Unique classes
- Wilcoxon signed-rank test results for log10(sigmaSq/(2*alpha)) within each class
- Exact binomial tests for lethal phenotypes (genome-wide and Unique-restricted)
- summary_statistics.csv assembling all test names, statistics, and p-values

## How to apply

Load the per-event CLOUDe outputs in R: predicted classes from test_prefix.*.classifications and the 3m evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) from test_prefix.*.predictions. Compute |theta1−theta2| per deletion event and run a Mann–Whitney U test comparing this quantity between the Redundant and Unique classes (reported p<2.22e-16 for Drosophila, consistent with the OU expectation that unique deletions have more divergent optima). Run Wilcoxon signed-rank tests on log10(sigmaSq/(2*alpha)) within each class (reported p=6.96e-46 Redundant, p=7.30e-56 Unique) to test whether stationary variance differs from expectation. Supplement with lethal-phenotype exact binomial tests genome-wide (p=4.81e-2) and within the Unique class (p=3.65e-1), and a Mann–Whitney test comparing empirical vs. simulated lost-gene (L) expression (p=2.70e-11; not significant after filtering genes with FPKM<1). Assemble all test names, statistics, and p-values into summary_statistics.csv and check each value against the reported p-values as the acceptance criterion.

## Related tools

- **CLOUDe** (Produces the per-event predicted classes and OU evolutionary parameters that the significance tests operate on) — https://github.com/anddssan/CLOUDe
- **R** (Environment in which CLOUDe runs and the Mann–Whitney U, Wilcoxon signed-rank, and exact binomial tests are performed)
- **dplyr** (Required R library for CLOUDe, used for manipulating the per-event classification and prediction tables before testing)

## Examples

```
source("CLOUDe.R"); wilcox.test(abs(pred$theta1 - pred$theta2) ~ pred$class)  # Mann–Whitney U on |theta1−theta2|, Redundant vs Unique
```

## Evaluation signals

- Mann–Whitney U test on |theta1−theta2| between Redundant and Unique classes yields p<2.22e-16 on the Drosophila data
- Wilcoxon signed-rank tests on log10(sigmaSq/(2*alpha)) reproduce p=6.96e-46 (Redundant) and p=7.30e-56 (Unique)
- Genome-wide lethal-phenotype exact binomial test gives p=4.81e-2, restricted to the Unique class p=3.65e-1
- Empirical vs. simulated lost-gene (L) expression Mann–Whitney test gives p=2.70e-11, and the signal is not significant after filtering genes with FPKM<1
- summary_statistics.csv contains every reported test name, statistic, and p-value matching the paper's reported values

## Limitations

- The empirical-vs-simulated lost-gene expression signal is not significant after filtering genes with FPKM<1, so conclusions depend on expression-filter choices
- Predicted parameter significance is contingent on CLOUDe's upstream prediction quality; optimal classifier/predictor performance was achieved by the neural-network architecture, so other architectures (XGB, RF, SVM) may yield weaker inputs to these tests
- Interpretation assumes expression evolution follows the Ornstein–Uhlenbeck process used to generate CLOUDe training data

## Evidence

- [abstract] CLOUDe models expression evolution as an Ornstein–Uhlenbeck process, and uses multi-layer neural network, extreme gradient boosting, random forest, and support vector machine architectures: "CLOUDe models expression evolution as an Ornstein–Uhlenbeck process, and uses multi-layer neural network, extreme gradient boosting, random forest, and support vector machine architectures"
- [methods] CLOUDe can predict classes and evolutionary parameters for each gene in a test dataset.: "CLOUDe can predict classes ("Redundant" or "Unique") and evolutionary parameters (theta1, theta2, and log10(sigmaSq/(2*alpha))) for each gene in a test dataset."
- [methods] GenerateTrainingData outputs features and parameter ranges for the OU process.: "the rest of the parameters are used to define the ranges that the evolutionary parameters theta (optimal expression), alpha (strength of selection) and sigmaSq (strength of phenotypic drift) of the"
- [readme] Input file format specification for empirical expression data.: "A space-delimited file with N+1 rows and 3m columns, where N is the number of genes and m is the number of conditions (e.g., tissues, ages, etc.) for expression data. The first row is a header."
- [abstract] CLOUDe application to Drosophila data finding.: "Application of CLOUDe to empirical data from Drosophila suggests that deletion primarily targets genes with unique functions, with further analysis showing these functions to be enriched for protein"
