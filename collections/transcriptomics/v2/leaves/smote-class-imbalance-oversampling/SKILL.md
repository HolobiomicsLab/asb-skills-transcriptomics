---
name: smote-class-imbalance-oversampling
description: Use when preparing disease/stress and control tab-separated expression
  matrices for BoostDiff and one condition has far fewer samples than the other (below
  the recommended ~30 per condition or below BoostDiff's min_samples=15 default).
when_to_use_negative:
- When both conditions already have ≥30 samples each — the README recommends this
  as sufficient input, and SMOTE would only add synthetic noise
- When the number of samples per condition would fall below BoostDiff's min_samples
  default of 15 even after oversampling is deemed inappropriate for very low-count
  conditions, or when applying single-sample/single-replicate pilot data where synthetic
  interpolation is unreliable
- When the differential network inference method is not tree-based (e.g. correlation-only
  network inference), since class balance mainly matters for bootstrapped, sample-resampling
  learners like BoostDiff's n_subsamples=20 ensemble
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pySMOTE
  role: Python implementation of SMOTE used to oversample the minority stress condition
    (via SMOTE.oversample) until 48 samples per condition are reached before running
    BoostDiff
  repo: https://github.com/akazs/pySMOTE
- name: BoostDiff
  role: Downstream consumer of the balanced matrices — its AdaBoost differential-tree
    ensemble with n_subsamples=20 bootstraps per condition motivates balancing the
    classes beforehand
  repo: https://github.com/scibiome/boostdiff_inference
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:d704c85e7db86d98f233b65845876ed7441269b99f5786b2a13f2bf94f519169
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akazs/pySMOTE
  tools:
  - BoostDiff
  - GEOquery
  - Python
  - R
  - dplyr
  - pySMOTE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# smote-class-imbalance-oversampling

## Summary

Balances two-condition expression matrices by oversampling the minority class with Synthetic Minority Over-sampling Technique (SMOTE) so BoostDiff receives ≥30–48 samples per condition (e.g. B. subtilis stress vs control from GSE27219), since the AdaBoost ensemble of differential trees requires sufficient samples in each class.

## When to use

Use when preparing disease/stress and control tab-separated expression matrices for BoostDiff and one condition has far fewer samples than the other (below the recommended ~30 per condition or below BoostDiff's min_samples=15 default).

## When NOT to use

- When both conditions already have ≥30 samples each — the README recommends this as sufficient input, and SMOTE would only add synthetic noise
- When the number of samples per condition would fall below BoostDiff's min_samples default of 15 even after oversampling is deemed inappropriate for very low-count conditions, or when applying single-sample/single-replicate pilot data where synthetic interpolation is unreliable
- When the differential network inference method is not tree-based (e.g. correlation-only network inference), since class balance mainly matters for bootstrapped, sample-resampling learners like BoostDiff's n_subsamples=20 ensemble

## Inputs

- Tab-separated expression matrix for the minority (stress/disease) condition, rows=genes, columns=samples, first column 'Gene'
- Tab-separated expression matrix for the majority (control) condition with the same gene set
- Target number of samples per condition after oversampling (e.g. 48 for the GSE27219 stress contexts)

## Outputs

- SMOTE-oversampled expression matrix for the minority condition, merged with original samples (48 samples per condition in the salt/heat/ethanol stress runs)
- Balanced disease/stress and control .txt matrices ready as file_disease/file_control inputs to BoostDiff

## How to apply

After mapping gene IDs, aggregating replicates to a single matrix per condition (first column named 'Gene'), and filtering genes by a variance cutoff (0.4 in the GSE27219 salt/heat/ethanol stress workflow), run pySMOTE on the under-represented condition's sample matrix to synthesize new samples until each condition is balanced to the target size (48 samples per condition in the paper's stress-response runs). Use a modest k_neighbors value so synthetic samples interpolate within the local minority-class neighborhood rather than crossing into the majority class. Write the oversampled data back to tab-separated files with the same gene row order so the two inputs to BoostDiff share an identical feature set. Balance matters because BoostDiff builds n_subsamples=20 bootstrapped samples per condition when growing the differential-tree ensemble and requires at least min_samples=15 per condition by default.

## Related tools

- **pySMOTE** (Python implementation of SMOTE used to oversample the minority stress condition (via SMOTE.oversample) until 48 samples per condition are reached before running BoostDiff) — https://github.com/akazs/pySMOTE
- **BoostDiff** (Downstream consumer of the balanced matrices — its AdaBoost differential-tree ensemble with n_subsamples=20 bootstraps per condition motivates balancing the classes beforehand) — https://github.com/scibiome/boostdiff_inference

## Examples

```
import pySMOTE; smote = pySMOTE.SMOTE(ratio=100, k_neighbors=6); new_samples = smote.oversample(sample_data, merge=True)
```

## Evaluation signals

- After oversampling, both condition matrices contain the target number of samples (48 per condition in the paper's stress runs) while the gene set (rows) is unchanged and identical across the two files
- The first column of both tab-separated files is still named 'Gene', matching the BoostDiff input schema
- No gene was dropped or reordered by SMOTE — row identifiers in the oversampled matrix match the variance-filtered input (variance cutoff 0.4)
- BoostDiff accepts the matrices without triggering its min_samples=15-per-condition guard, and the subsequent run completes with the intended parameters (n_estimators=50, n_features=2500, n_subsamples=20, loss='exponential')

## Limitations

- SMOTE synthesizes samples by interpolating between minority-class neighbors, so it can distort the variance structure that BoostDiff's differential variance improvement (DVI) splitting criterion depends on — synthetic samples should be inspected for implausible variance profiles
- The README recommends the Crohn's disease dataset (GSE126124) is too large and needs an HPC cluster; oversampling adds further compute, so apply it only after variance filtering reduces the gene set
- pySMOTE was tested only on Python 2.7/3.6 with scipy>=0.19.0, numpy>=1.13.0, scikit-learn>=0.18.1; older or newer stacks are untested

## Evidence

- [other] SMOTE oversample to 48 samples per condition to balance the two classes: [evidence span withheld — anchor status: not_found]
- [readme] ideal number of samples per condition: "The ideal number of samples per condition as input to BoostDiff would be at least 30 samples each."
- [readme] n_subsamples bootstrapped samples per condition: "n_subsamples`: *int*, the no. of bootstrapped samples to be used per condition (disease samples and control samples) when building the tree ensemble"
- [readme] pySMOTE usage snippet: "smote = pySMOTE.SMOTE(ratio=100, k_neighbors=6)
synthetic_samples = smote.oversample(sample_data)"
- [readme] differential variance improvement criterion: "BoostDiff modifies regression trees to use differential variance improvement (DVI) as the novel splitting criterion."
