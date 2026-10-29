---
name: wilcoxon-signed-rank-testing
description: Use when per-event predictions from CLOUDe (classifications and theta1,
  theta2, log10(sigmaSq/(2*alpha)) values) must be statistically compared across deletion
  classes or against simulated expression distributions.
when_to_use_negative:
- Input is raw expression data before CLOUDe classification/prediction — no per-event
  parameters exist to compare yet
- The data are paired by design across more than two conditions requiring repeated-measures
  structure rather than a two-class rank comparison
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3299
tools:
- name: R
  role: Environment in which the Wilcoxon signed-rank, Mann-Whitney U, and exact binomial
    tests are executed on CLOUDe outputs
  repo: https://github.com/anddssan/CLOUDe
- name: dplyr
  role: Data manipulation to join classifications with predicted parameters and group
    by deletion class
  repo: https://github.com/anddssan/CLOUDe
- name: CLOUDe
  role: Produces the per-event classifications and predicted evolutionary parameters
    that serve as test inputs
  repo: https://github.com/anddssan/CLOUDe
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:20a35eba8ef2a26ac414c2b3c57fd148652c1e76f5ffd9e848aee3541ea25aba
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
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# wilcoxon-signed-rank-testing

## Summary

Apply Wilcoxon signed-rank and Mann-Whitney U nonparametric tests in R to assess whether CLOUDe's predicted evolutionary parameters (|theta1−theta2| and log10(sigmaSq/(2*alpha))) differ between 'Redundant' and 'Unique' Drosophila deletion classes, and to compare empirical lost-gene expression against simulated expectations.

## When to use

Use when per-event predictions from CLOUDe (classifications and theta1, theta2, log10(sigmaSq/(2*alpha)) values) must be statistically compared across deletion classes or against simulated expression distributions.

## When NOT to use

- Input is raw expression data before CLOUDe classification/prediction — no per-event parameters exist to compare yet
- The data are paired by design across more than two conditions requiring repeated-measures structure rather than a two-class rank comparison

## Inputs

- test_prefix.*.classifications files (predicted Redundant/Unique classes per deletion event)
- test_prefix.*.predictions files (predicted theta1, theta2, log10(sigmaSq/(2*alpha)) per deletion event)
- Empirical FPKM expression values for lost (L) genes
- Simulated L-gene expression generated from the OU process training data

## Outputs

- Mann-Whitney U p-value comparing |theta1-theta2| between Redundant and Unique classes (expected p<2.22e-16)
- Wilcoxon signed-rank p-values for log10(sigmaSq/(2*alpha)) within each class (expected p=6.96e-46 Redundant, p=7.30e-56 Unique)
- Exact binomial test p-values for lethal phenotypes (genome-wide p=4.81e-2; Unique-only p=3.65e-1)
- summary_statistics.csv with all test names, statistics, and p-values

## How to apply

Load the per-event CLOUDe outputs in R with dplyr: predicted classes from test_prefix.*.classifications and the 3m evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) from test_prefix.*.predictions. Run a Mann-Whitney U test comparing the absolute difference between predicted expression optima |theta1-theta2| between the Redundant and Unique classes, then Wilcoxon signed-rank tests on log10(sigmaSq/(2*alpha)) within each class. Complement these with exact binomial tests for lethal phenotypes (genome-wide and Unique-restricted) and a Mann-Whitney test comparing empirical lost-gene (L) expression to simulated L-gene expression, repeating the latter after filtering genes with FPKM<1 to check robustness. Assemble all test names, statistics, and p-values into a single summary table (summary_statistics.csv) and verify each value against the reported benchmarks as the acceptance criterion.

## Related tools

- **R** (Environment in which the Wilcoxon signed-rank, Mann-Whitney U, and exact binomial tests are executed on CLOUDe outputs) — https://github.com/anddssan/CLOUDe
- **dplyr** (Data manipulation to join classifications with predicted parameters and group by deletion class) — https://github.com/anddssan/CLOUDe
- **CLOUDe** (Produces the per-event classifications and predicted evolutionary parameters that serve as test inputs) — https://github.com/anddssan/CLOUDe

## Examples

```
library(dplyr); cls <- read.table('test_prefix.nn.3.classifications'); pred <- read.table('test_prefix.nn.3.predictions'); wilcox.test(log10(sigmaSq_div_2alpha) ~ 1, data=subset(d, class=='Unique'))
```

## Evaluation signals

- Mann-Whitney U p-value for |theta1-theta2| between classes is <2.22e-16 (Unique larger)
- Wilcoxon signed-rank p-values match reported 6.96e-46 (Redundant) and 7.30e-56 (Unique) for log10(sigmaSq/(2*alpha))
- Lethal-phenotype exact binomial tests reproduce p=4.81e-2 genome-wide and p=3.65e-1 for the Unique class
- Empirical vs simulated L-gene Mann-Whitney test yields p=2.70e-11, which loses significance after filtering genes with FPKM<1
- summary_statistics.csv is assembled with every reported test statistic and p-value matching the benchmark values

## Limitations

- The FPKM<1 filter removes the significance of the empirical-vs-simulated L-gene expression signal, so conclusions depend on low-expression gene inclusion
- Wilcoxon signed-rank tests are one-sample/within-class tests and cannot substitute for the between-class Mann-Whitney comparison of |theta1-theta2|
- Results apply to the OU-process framing of expression evolution modeled by CLOUDe for two-species deletion events

## Evidence

- [other] Run a Mann-Whitney U test comparing |theta1-theta2| between the Redundant and Unique deletion classes (reported as p<2.22e-16).: [evidence span withheld — anchor status: not_found]
- [other] Run Wilcoxon signed-rank tests on log10(sigmaSq/(2*alpha)) within each class (reported p=6.96e-46 for Redundant, p=7.30e-56 for Unique).: [evidence span withheld — anchor status: not_found]
- [readme] CLOUDe can predict classes ("Redundant" or "Unique") and evolutionary parameters (theta1, theta2, and log10(sigmaSq/(2*alpha))) for each gene in a test dataset.: "CLOUDe can predict classes ("Redundant" or "Unique") and evolutionary parameters (theta1, theta2, and log10(sigmaSq/(2*alpha))) for each gene in a test dataset."
- [other] The absolute difference between predicted expression optima is significantly larger for the 'unique' class (p<2.22×10-16, Mann-Whitney U test).: [evidence span withheld — anchor status: not_found]
