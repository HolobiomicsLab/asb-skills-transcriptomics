---
name: differential-progression-testing
description: Use when scRNA-Seq data from two or more conditions share a common trajectory
  (topologyTest fails to reject it) and the goal is to detect condition-specific shifts
  in pseudotime distributions along lineages.
when_to_use_negative:
- The topologyTest rejects the common-trajectory null, indicating fundamentally different
  topologies between conditions rather than a progression shift along a shared path
- The question is which lineage cells commit to (differential fate selection) rather
  than how fast they progress; use fateSelectionTest on lineage weights instead
- Data are steady-state with no trajectory structure; cluster-based differential abundance
  methods like DAseq or milo are the appropriate family, though the paper's simulations
  show trajectory-aware tests outperform them when a trajectory exists
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: condiments
  role: Provides progressionTest (custom Kolmogorov-Smirnov or classifier test) to
    compare pseudotime distributions between conditions, plus topologyTest and fateSelectionTest
    in the surrounding workflow
  repo: github.com/HectorRDB/condiments
- name: slingshot
  role: Infers the common trajectory and per-lineage pseudotimes (slingPseudotime)
    that progressionTest consumes
- name: monocle3
  role: Alternative trajectory inference whose pseudotimes can feed progression testing
    in the dyngen simulation benchmark
- name: dyngen
  role: Generates simulated multi-lineage datasets (topologies T1-T4) with ground-truth
    pseudotimes used to benchmark progressionTest against DAseq and milo
- name: ggplot2
  role: Visualizes per-lineage pseudotime density distributions by condition before
    testing
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_008
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:04f43de31181727c13b786e6ff78b841508aac0714bc4304eabae24e945a037d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DAseq
  - condiments
  - dplyr
  - dyngen
  - ggplot2
  - milo
  - monocle3
  - slingshot
  - tidyr
  - tradeSeq
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# differential-progression-testing

## Summary

Test whether cells from different conditions progress at different rates along a shared trajectory by comparing per-lineage pseudotime distributions with condiments' progressionTest, typically downstream of slingshot-inferred common trajectories.

## When to use

Use when scRNA-Seq data from two or more conditions share a common trajectory (topologyTest fails to reject it) and the goal is to detect condition-specific shifts in pseudotime distributions along lineages.

## When NOT to use

- The topologyTest rejects the common-trajectory null, indicating fundamentally different topologies between conditions rather than a progression shift along a shared path
- The question is which lineage cells commit to (differential fate selection) rather than how fast they progress; use fateSelectionTest on lineage weights instead
- Data are steady-state with no trajectory structure; cluster-based differential abundance methods like DAseq or milo are the appropriate family, though the paper's simulations show trajectory-aware tests outperform them when a trajectory exists

## Inputs

- Slingshot (slingshot::SlingshotDataSet / slingPseudotime output) inferred on a common trajectory spanning all conditions
- Per-cell condition labels (e.g. df$conditions, such as control vs ILD or mock vs TGF-beta)
- Reduced-dimension cell coordinates (e.g. (Dim1, Dim2) from condiments toy_dataset) and cluster labels for trajectory inference
- Monocle3 trajectory pseudotimes are also supported as input

## Outputs

- P-value table with global and per-lineage statistics and p-values (e.g. rendered with knitr::kable and saved as progression_test_pvalues.csv)
- Binary differential-progression calls per lineage after Benjamini-Hochberg correction at 5% FDR (for benchmark comparisons)
- Optional ggplot2 density plots of pseudotime per lineage colored by condition

## How to apply

First infer a common trajectory with slingshot on the reduced-dimension coordinates (e.g. the (Dim1, Dim2) matrix from condiments' toy_dataset) using cluster labels, and confirm via topologyTest that a common topology is adequate. Extract per-lineage pseudotimes with slingPseudotime and optionally visualize pseudotime density distributions per lineage colored by condition with ggplot2 to spot shifts. Run condiments::progressionTest on the slingshot object with conditions = df$conditions, global = TRUE and lineages = TRUE; the default is a custom Kolmogorov-Smirnov test comparing pseudotime distributions between conditions, with a classifier-based test as an alternative. Call differential progression when the global and/or per-lineage p-values pass a 5% FDR threshold (Benjamini-Hochberg in benchmark settings); a small global p-value driven by a single lineage indicates which lineage carries the difference. The test is robust to trajectory-inference error: it still rejects the null with up to ~60% lineage misassignment and ~200% pseudotime noise, so moderate TI imperfections need not invalidate results.

## Related tools

- **condiments** (Provides progressionTest (custom Kolmogorov-Smirnov or classifier test) to compare pseudotime distributions between conditions, plus topologyTest and fateSelectionTest in the surrounding workflow) — github.com/HectorRDB/condiments
- **slingshot** (Infers the common trajectory and per-lineage pseudotimes (slingPseudotime) that progressionTest consumes)
- **monocle3** (Alternative trajectory inference whose pseudotimes can feed progression testing in the dyngen simulation benchmark)
- **dyngen** (Generates simulated multi-lineage datasets (topologies T1-T4) with ground-truth pseudotimes used to benchmark progressionTest against DAseq and milo)
- **ggplot2** (Visualizes per-lineage pseudotime density distributions by condition before testing)

## Examples

```
progressionTest(sds, conditions = df$conditions, global = TRUE, lineages = TRUE)
```

## Evaluation signals

- On the condiments toy_dataset, the global test and the lineage-2 test should yield small p-values (differential progression on the bottom lineage) while lineage 1 is non-significant
- In the TGF-beta EMT case study, progressionTest should reject the null of identical pseudotime distributions with p-value <= 2.2e-16 after topologyTest fails to reject the common trajectory (p = 0.38)
- In benchmark settings, Benjamini-Hochberg-adjusted calls at 5% FDR should achieve TNR, TPR, PPV, NPV, and F1 scores exceeding DAseq and milo on dyngen T1-T4 simulations
- Robustness check: median p-values across 100 perturbed replicates should remain significant at 5% up to ~60% lineage misassignment and pseudotime noise up to 200% of the pseudotime scale
- Output CSV (e.g. progression_test_pvalues.csv) must contain both global and per-lineage statistics and p-values

## Limitations

- Dependence of pseudotime estimates on trajectory inference (double-dipping) is ignored rather than resolved, as in other differential abundance methods
- P-values are treated as numerical summaries rather than given strong probabilistic interpretations, which would require hard-to-verify assumptions
- Robustness holds only up to moderate TI error; performance beyond ~60% misassignment or 200% pseudotime noise is not supported
- The authors note no independent benchmarking of the full workflow has been performed and call for it

## Evidence

- [methods] progressionTest defines differential progression: "To test for differential progression, we use the __progressionTest__."
- [methods] topologyTest precedes progression testing: "The __topologyTest__ assess the quality of the common trajectory inference"
- [discussion] condiments tests progression and fate selection at trajectory level: "then differences in abundance at the trajectory level with the progressionTest and fateSelectionTest"
- [results] robustness to trajectory-inference error: [evidence span withheld — anchor status: not_found]
- [full_text] KS-test definition of the null hypothesis: [evidence span withheld — anchor status: not_found]
- [readme] installation instructions from README: [evidence span withheld — anchor status: external_doc]
