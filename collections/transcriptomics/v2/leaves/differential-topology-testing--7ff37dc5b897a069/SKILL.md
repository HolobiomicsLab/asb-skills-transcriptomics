---
name: differential-topology-testing
description: Use when comparing single-cell dynamic processes across conditions and
  you must first decide whether conditions share a common trajectory topology before
  testing differential progression, fate selection, or gene expression.
when_to_use_negative:
- Cells form a steady-state population rather than a dynamic process — cluster cells
  and assess differential abundance per cluster instead, since cluster-based approaches
  ignore continuity of cell-state transitions and trajectory tests assume a trajectory
  exists
- Gene-level differences along a lineage are already the target — use tradeSeq fitGAM
  + conditionTest (Step 3) rather than a topology-level test
- Differences in cell abundance along a shared trajectory are the target — use progressionTest
  or fateSelectionTest (Step 2), not a topology test
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: condiments
  role: Provides topologyTest, the permutation-based test of common vs separate trajectory
    topologies (Step 1 of the condiments workflow)
  repo: https://github.com/HectorRDB/condiments
- name: slingshot
  role: Infers the common trajectory (lineages and pseudotime) on reduced dimensions
    that topologyTest evaluates and re-infers under permuted condition labels
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:88cedb02192a16b21609dc258f5ba078b269406c1b0cbab2fd4e5e391b3a3fcc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HectorRDB/condiments
  tools:
  - condiments
  - ggplot2
  - slingshot
  - tradeSeq
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# differential-topology-testing

## Summary

Step 1 of the condiments workflow: use topologyTest to decide whether cells from different conditions follow a fundamentally different trajectory topology or can be analyzed on a single common trajectory, by permuting condition labels and re-inferring trajectories.

## When to use

Use when comparing single-cell dynamic processes across conditions and you must first decide whether conditions share a common trajectory topology before testing differential progression, fate selection, or gene expression.

## When NOT to use

- Cells form a steady-state population rather than a dynamic process — cluster cells and assess differential abundance per cluster instead, since cluster-based approaches ignore continuity of cell-state transitions and trajectory tests assume a trajectory exists
- Gene-level differences along a lineage are already the target — use tradeSeq fitGAM + conditionTest (Step 3) rather than a topology-level test
- Differences in cell abundance along a shared trajectory are the target — use progressionTest or fateSelectionTest (Step 2), not a topology test

## Inputs

- Integrated, normalized single-cell count matrix with per-cell condition labels (e.g., mock vs TGF-β)
- Low-dimensional reduced-dimension coordinates for all cells
- Cell cluster labels used for slingshot trajectory inference
- A SlingshotDataSet (sds) fitted on the pooled conditions

## Outputs

- P-value from topologyTest for common vs separate topologies (e.g., p = 0.38 in the TGF-β EMT case study)
- Binary decision: common trajectory (fail to reject) vs condition-specific topologies
- Test-statistic table entries assembled into results such as Table 2 row 1 and Fig. 3 annotations of the condiments paper

## How to apply

Infer a single common trajectory on the integrated, normalized multi-condition dataset with slingshot on the reduced-dimension coordinates (e.g., PCA/UMAP) using cluster labels. Then run condiments::topologyTest, which repeatedly permutes the condition labels and re-runs trajectory inference to compare the observed topology against the null of identical topologies across conditions (in the TGF-β EMT case study this yielded p = 0.38, so the null of a common trajectory was not rejected). If the test fails to reject (e.g., at FDR 5%), fit and interpret one common trajectory and proceed to progressionTest/fateSelectionTest on its pseudotime and curve weights; if it rejects, infer separate trajectories per condition. The topologyTest thereby assesses the quality of the common trajectory inference before any downstream differential-progression or gene-level (tradeSeq conditionTest) analysis is attempted.

## Related tools

- **condiments** (Provides topologyTest, the permutation-based test of common vs separate trajectory topologies (Step 1 of the condiments workflow)) — https://github.com/HectorRDB/condiments
- **slingshot** (Infers the common trajectory (lineages and pseudotime) on reduced dimensions that topologyTest evaluates and re-infers under permuted condition labels)

## Examples

```
library(condiments); topologyTest(sds = sds, conditions = df$conditions)
```

## Evaluation signals

- The topologyTest reports a p-value for the null of a common topology; for the TGF-β EMT case study the expected result is p = 0.38 (fail to reject common trajectory)
- Verify the test actually permutes condition labels and re-runs trajectory inference each time, not a single fixed trajectory
- Check that downstream progressionTest/fateSelectionTest are only run on a common trajectory when topologyTest failed to reject
- Assemble the topologyTest statistic into the condiments_emt_results.csv table matching Table 2 row 1 of the paper
- Confirm the trajectory was inferred with slingshot on reduced dimensions using cluster labels from the integrated, normalized count matrix

## Limitations

- Dependence of pseudotime/weight estimates on the inferred trajectory (double-dipping) is ignored rather than resolved
- P-values are treated as numerical summaries rather than given strong probabilistic interpretations, which would involve hard-to-verify assumptions
- Batch-condition confounding can be inherent and unresolvable in some settings
- No independent benchmarking of the workflow has been performed; the authors call for it

## Evidence

- [intro] Step 1 differential topology definition: "In Step 1, condiments considers the trajectory inference question, assessing whether the dynamic process is fundamentally different between conditions"
- [methods] topologyTest purpose: "The __topologyTest__ assess the quality of the common trajectory inference"
- [discussion] Workflow ordering: "by first studying overall between-condition differences in trajectories with the topologyTest"
- [full_text] Case-study workflow detail: [evidence span withheld — anchor status: not_found]
- [readme] Package availability: [evidence span withheld — anchor status: external_doc]
