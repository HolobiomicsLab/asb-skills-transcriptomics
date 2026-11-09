---
name: dropout-masking-benchmarking
description: Use when a single-cell AS probability matrix (sites × cells) with NA
  entries is available and you need to quantitatively validate an imputation strategy
  (e.g., SCASL's weighted KNN) against held-out ground truth under graded dropout
  levels.
when_to_use_negative:
- When the goal is cell clustering on the AS probability matrix rather than validating
  imputation accuracy — use SCASL's spectral clustering path instead.
- When the input is raw junction/bam data rather than a normalized AS probability
  matrix; first run SCASL's AS probability inference and normalization steps.
- When naturally missing (NA) entries alone suffice — this skill requires held-out
  ground truth values that are masked only artificially.
edam_operation: http://edamontology.org/operation_3552
edam_topics:
- http://edamontology.org/topic_3420
- http://edamontology.org/topic_3320
tools:
- name: SCASL
  role: Provides the iterative weighted KNN imputation step being benchmarked and
    the normalized AS probability matrix it operates on
  repo: github.com/xryanglab/SCASL
- name: Python + pandas + scikit-learn
  role: Implement forced dropout masking, run imputation, and quantify recovery rates
    and t-tests
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:58eede8d640a134f4dd4d386acd3b34724421eacc412608b2e81113657d1b87d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Python + pandas + scikit-learn
  - SCASL
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3552
  edam_topics:
  - http://edamontology.org/topic_3420
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dropout-masking-benchmarking

## Summary

Benchmark imputation fidelity in the SCASL pipeline by artificially forcing dropouts (10%, 30%, 90%) of non-NA AS probability values in the normalized single-cell AS probability matrix and measuring how many masked entries the iterative weighted KNN imputation recovers correctly.

## When to use

Use when a single-cell AS probability matrix (sites × cells) with NA entries is available and you need to quantitatively validate an imputation strategy (e.g., SCASL's weighted KNN) against held-out ground truth under graded dropout levels.

## When NOT to use

- When the goal is cell clustering on the AS probability matrix rather than validating imputation accuracy — use SCASL's spectral clustering path instead.
- When the input is raw junction/bam data rather than a normalized AS probability matrix; first run SCASL's AS probability inference and normalization steps.
- When naturally missing (NA) entries alone suffice — this skill requires held-out ground truth values that are masked only artificially.

## Inputs

- Normalized AS probability matrix (rows = AS sites, columns = cells) from the SCASL pipeline, e.g., the `normalized_matrix` intermediate file
- TNBC-2 demo intermediate junction files in `data/junction` as test input
- SCASL config YAML (configs/srr.yaml) with knn and num_iteration imputation parameters
- NA indicator / NA position information from SCASL intermediate outputs

## Outputs

- Recovery rate of imputed vs. original values at each masked position per dropout level (10%, 30%, 90%)
- Plot of recovery rate as a function of dropout proportion
- t-test comparing imputed versus true values across dropout levels
- Recorded positions of artificially dropped values (NA indicator structure)

## How to apply

Load the normalized AS probability matrix produced by the SCASL pipeline — using the TNBC-2 demo intermediate files in `data/junction` as the test input — with rows as AS sites and columns as cells. For each dropout proportion (10%, 30%, 90%), randomly mask that fraction of the non-NA AS probability entries and record the masked positions in an NA indicator structure, mirroring how SCASL tracks naturally missing entries. Run the iterative weighted KNN imputation step in the scasl package, setting the `knn` (number of adjacent cells) and `num_iteration` (imputation iterations) parameters in the config YAML alongside the standard QC thresholds (sites_initial, runs_initial, sites_quality, runs_quality). At each dropout level, extract the imputed values at the masked positions and compare them to the original non-NA values to compute the recovery rate. Plot recovery rate as a function of dropout proportion and perform a t-test comparing imputed versus true values across dropout levels; at 90% dropout the majority of dropped points should still be recovered (Fig. S1A) with no significant t-test difference.

## Related tools

- **SCASL** (Provides the iterative weighted KNN imputation step being benchmarked and the normalized AS probability matrix it operates on) — github.com/xryanglab/SCASL
- **Python + pandas + scikit-learn** (Implement forced dropout masking, run imputation, and quantify recovery rates and t-tests)

## Examples

```
python main.py -y configs/srr.yaml  # then, per dropout level (10%, 30%, 90%): mask non-NA entries of normalized_matrix with pandas/numpy, rerun SCASL's weighted KNN imputation (knn, num_iteration), and score recovery at masked positions
```

## Evaluation signals

- Majority of masked AS probability values correctly recovered even at the 90% dropout condition, consistent with Fig. S1A
- t-test across dropout levels shows no significant difference between imputed and true values
- Recovery-rate plot shows monotonically reasonable behavior as dropout proportion increases (10% → 30% → 90%)
- Masked positions are recorded in the NA indicator structure so imputed values can be extracted exactly at held-out coordinates

## Limitations

- The SCASL demo files are very small (test time < 3 minutes) and GitHub-hosted, so results are noisy; random imputation, dimensionality-reduction, and unsupervised clustering steps introduce run-to-run variability.
- Single-cell AS profiles suffer heavily from naturally occurring dropouts during scRNA-seq analysis and AS event identification, so available non-NA ground truth is sparse.
- Results are stochastic; recovery rates will vary between runs due to the random masking and interpolation.

## Evidence

- [results] Imputed AS probabilities were highly accurate compared to true values, and even when 90% of AS probability values were dropped out, the majority of these data points were still correctly recovered (Fig. S1A).: [evidence span withheld — anchor status: not_found]
- [discussion] SCASL employs an iterative, weighted KNN process for imputation of the missing values: "SCASL employs an iterative, weighted KNN process for imputation of the missing values"
- [readme] you can also choose to use the demo of the intermediate files (from TNBC-2) in `data/junction` for testing: "you can also choose to use the demo of the intermediate files (from TNBC-2) in `data/junction` for testing"
- [readme] **normalized_matrix**: The final AS probability matrix, the rows correspond to sites, while the columns represent cells.: "**normalized_matrix**: The final AS probability matrix, the rows correspond to sites, while the columns represent cells."
- [discussion] SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix: "SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix"
