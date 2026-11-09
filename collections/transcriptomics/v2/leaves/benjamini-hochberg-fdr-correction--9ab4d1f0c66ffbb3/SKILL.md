---
name: benjamini-hochberg-fdr-correction
description: Use when you have raw p-values from multiple testing methods (e.g., progressionTest,
  fateSelectionTest, DAseq, milo) on the same features/genes and need binary differential
  calls at a controlled false discovery rate of 5% for downstream performance evaluation.
when_to_use_negative:
- When a per-test pre-specified significance threshold (e.g., nominal p < 0.05) is
  required rather than family-wise FDR control across a large gene set
- When the p-values are not from independent or positively dependent families where
  BH's assumptions are intended, and stronger (e.g., BY) or weaker correction is explicitly
  mandated by the protocol
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: condiments
  role: Source of progressionTest and fateSelectionTest p-values that are BH-corrected
    at 5% FDR
  repo: github.com/HectorRDB/condiments
- name: DAseq
  role: Source of differential abundance p-values compared under the same BH 5% FDR
    threshold
- name: milo
  role: Source of differential abundance p-values compared under the same BH 5% FDR
    threshold
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:3d6d74545d8d70d3a2120f2962486d8ac263d98f36fbc1e332aa91ba6b870030
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DAseq
  - condiments
  - milo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# benjamini-hochberg-fdr-correction

## Summary

Applies Benjamini-Hochberg (BH) multiple-testing correction at a 5% FDR threshold to raw p-values from trajectory-aware tests (condiments progressionTest, fateSelectionTest) and differential abundance methods (DAseq, milo), converting them into binary differential calls for benchmark scoring against dyngen ground truth.

## When to use

Use when you have raw p-values from multiple testing methods (e.g., progressionTest, fateSelectionTest, DAseq, milo) on the same features/genes and need binary differential calls at a controlled false discovery rate of 5% for downstream performance evaluation.

## When NOT to use

- When a per-test pre-specified significance threshold (e.g., nominal p < 0.05) is required rather than family-wise FDR control across a large gene set
- When the p-values are not from independent or positively dependent families where BH's assumptions are intended, and stronger (e.g., BY) or weaker correction is explicitly mandated by the protocol

## Inputs

- Named numeric vector of raw p-values from condiments progressionTest (global and per-lineage)
- Named numeric vector of raw p-values from condiments fateSelectionTest
- Raw p-values from DAseq differential abundance testing
- Raw p-values from milo differential abundance testing
- dyngen simulation ground-truth labels for true differential genes (for downstream scoring)

## Outputs

- BH-adjusted p-value vectors for each of the four methods
- Binary differential calls (adjusted p-value <= 0.05) per method per topology (T1–T4)
- fig2e_performance_table.csv with TNR, TPR, PPV, NPV, and F1 for each of the four methods across T1–T4

## How to apply

Collect the unadjusted p-values from each of the four testing frameworks independently: condiments progressionTest (global and per-lineage pseudotime distribution testing downstream of slingshot or monocle3), fateSelectionTest (pairwise lineage-weight testing downstream of slingshot), DAseq, and milo differential abundance testing. Apply the Benjamini-Hochberg step-up procedure to each method's p-value vector separately, at a fixed FDR threshold of 5%, to obtain adjusted p-values and binary differential calls. This uniform BH correction across methods is the decision point that makes TNR, TPR, PPV, NPV, and F1 comparisons against the dyngen simulation ground truth (topologies T1–T4 with 2, 3, 2, and 5 lineages under WT/KO and WT/KO/UP designs) fair. Write the resulting performance metrics to fig2e_performance_table.csv. The rationale, per the condiments paper, is that trajectory-aware methods leveraging pseudotime better detect true condition changes than neighborhood-only differential abundance methods, and the BH thresholding quantifies this at a common error rate.

## Related tools

- **condiments** (Source of progressionTest and fateSelectionTest p-values that are BH-corrected at 5% FDR) — github.com/HectorRDB/condiments
- **DAseq** (Source of differential abundance p-values compared under the same BH 5% FDR threshold)
- **milo** (Source of differential abundance p-values compared under the same BH 5% FDR threshold)

## Examples

```
pvals_adj <- p.adjust(pvals, method = "BH"); calls <- pvals_adj <= 0.05
```

## Evaluation signals

- All BH-adjusted p-values fall within [0, 1] and preserve the monotonic ranking of the raw p-values
- The compiled fig2e_performance_table.csv contains all five metrics (TNR, TPR, PPV, NPV, F1) for each of the four methods across topologies T1–T4
- At the 5% FDR threshold, trajectory-aware methods (progressionTest, fateSelectionTest) should outperform neighborhood-only methods (DAseq, milo) on TPR/F1, with expected ordering progressionTest > fateSelectionTest > DAseq > milo
- Binary calls are exactly the set of features with BH-adjusted p-value <= 0.05; re-running with a fixed random seed on regenerated dyngen simulations reproduces the same calls
- Spot-check a small p-value vector manually against the BH step-up adjustment to confirm the correction is applied per-method, not pooled across methods

## Limitations

- The paper explicitly avoids strong probabilistic interpretation of p-values, which 'would involve a variety of hard-to-verify assumptions'; BH-adjusted values are treated as numerical summaries for thresholding
- Dependence between pseudotime/weight estimates and downstream tests (double-dipping) is ignored rather than resolved, which can affect calibration of BH-adjusted p-values
- Independent benchmarking of the condiments workflow was noted as still needed in the paper's discussion; simulation-based rankings may not transfer to all real datasets

## Evidence

- [other] task_005 workflow BH correction step: [evidence span withheld — anchor status: not_found]
- [discussion] p-value interpretation caveat: "rather than attaching strong probabilistic interpretations to p-values (which, as in most RNA-Seq applications, would involve a variety of hard-to-verify assumptions"
- [intro] benchmark against DA methods: [evidence span withheld — anchor status: not_found]
