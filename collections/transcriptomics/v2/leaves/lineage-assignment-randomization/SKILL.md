---
name: lineage-assignment-randomization
description: 'Use when validating differential-progression results against imperfect
  trajectory inference: you have ground-truth pseudotimes/lineages (e.g. from dyngen)
  and need to know how much pseudotime noise or cell misassignment the progression
  test tolerates.'
when_to_use_negative:
- The goal is to detect differential progression on trustworthy trajectories — run
  progressionTest directly rather than randomizing assignments
- The question is whether trajectory topology itself differs between conditions (differential
  topology) — use topologyTest, not assignment randomization
- No ground-truth pseudotimes/lineages are available, so the perturbation magnitude
  cannot be calibrated against the true trajectory
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: condiments
  role: Provides progressionTest, the differential-progression test whose p-values
    are evaluated on perturbed pseudotimes/lineage assignments
  repo: github.com/HectorRDB/condiments
- name: dyngen
  role: Generates the ground-truth T1 simulation with known pseudotimes and lineage
    assignments used as the reference trajectory
- name: slingshot
  role: Trajectory inference supplying the lineages and pseudotimes that are subsequently
    perturbed
- name: ggplot2
  role: Plots the median-p-value curves versus pseudotime noise level and lineage
    misassignment proportion
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:f92f6fb7fbb5fbfc53820fa1e686d584e95166521e93d410692eaa818a15b027
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - condiments
  - dyngen
  - ggplot2
  - slingshot
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# lineage-assignment-randomization

## Summary

A robustness stress-test that deliberately corrupts trajectory-inference outputs — swapping lineage assignments for a fraction of cells and adding Gaussian noise to pseudotimes — to quantify how much imperfection condiments::progressionTest tolerates before failing to reject the null of no differential progression.

## When to use

Use when validating differential-progression results against imperfect trajectory inference: you have ground-truth pseudotimes/lineages (e.g. from dyngen) and need to know how much pseudotime noise or cell misassignment the progression test tolerates.

## When NOT to use

- The goal is to detect differential progression on trustworthy trajectories — run progressionTest directly rather than randomizing assignments
- The question is whether trajectory topology itself differs between conditions (differential topology) — use topologyTest, not assignment randomization
- No ground-truth pseudotimes/lineages are available, so the perturbation magnitude cannot be calibrated against the true trajectory

## Inputs

- dyngen-simulated single-cell dataset with known ground-truth pseudotimes and lineage assignments (T1 reference, noise parameter m=0.85)
- Vector of lineage-misassignment proportions p in {0, 0.1, 0.2, 0.3, 0.4, 0.5}
- Range of pseudotime noise levels (Gaussian standard deviations over [0, 4], up to 200% of pseudotime scale)
- Condition labels per cell for the progressionTest call

## Outputs

- Per-setting distribution of progressionTest p-values over 100 replicates (custom KS test, global=TRUE, lineages=TRUE)
- Median p-value per noise/misassignment combination
- ggplot2 figure of median-p-value curves versus pseudotime noise and misassignment proportion (median_pvalue_curves.png)
- Robustness threshold estimate (~60% misassignment, ~200% pseudotime noise at the 5% nominal level)

## How to apply

Start from a ground-truth dyngen simulation (T1 reference with noise parameter m=0.85) where the true pseudotimes and lineage assignments of every cell are known. For each pseudotime-noise level, perturb the true pseudotimes with additive Gaussian noise whose standard deviation is drawn over the range [0, 4], corresponding to up to 200% of the pseudotime scale. Independently, for each misassignment proportion p in {0, 0.1, 0.2, 0.3, 0.4, 0.5}, randomly swap lineage assignments for exactly that fraction of cells. Run condiments::progressionTest on each perturbed replicate (custom KS test, global=TRUE, lineages=TRUE) across 100 replicates per noise/misassignment combination, then compute the median p-value across replicates for each setting. Plot the median-p-value curves versus pseudotime noise level and misassignment proportion with ggplot2 (e.g. saved as median_pvalue_curves.png); the expected outcome is that progressionTest still correctly rejects the null at the 5% nominal level up to roughly 60% misassignment and 200% pseudotime noise.

## Related tools

- **condiments** (Provides progressionTest, the differential-progression test whose p-values are evaluated on perturbed pseudotimes/lineage assignments) — github.com/HectorRDB/condiments
- **dyngen** (Generates the ground-truth T1 simulation with known pseudotimes and lineage assignments used as the reference trajectory)
- **slingshot** (Trajectory inference supplying the lineages and pseudotimes that are subsequently perturbed)
- **ggplot2** (Plots the median-p-value curves versus pseudotime noise level and lineage misassignment proportion)

## Examples

```
condiments::progressionTest(sds, conditions = cond, method = "KS", global = TRUE, lineages = TRUE)
```

## Evaluation signals

- At 0% perturbation, median p-values match an unperturbed progressionTest run on the dyngen ground truth
- Median p-values remain below the 5% nominal level up to approximately 60% lineage misassignment and 200% (SD 4) pseudotime noise
- Each noise/misassignment combination is summarized over 100 replicates, so median estimates should be stable across reruns
- Perturbed lineage-assignment vectors contain exactly proportion p of swapped cell labels, verifiable by comparing against the dyngen ground-truth assignments
- The ggplot2 output is saved as median_pvalue_curves.png with curves spanning both noise and misassignment axes

## Limitations

- The robustness result is established on dyngen simulations with randomly swapped assignments; real trajectory-inference errors may be structured rather than random
- Dependence of pseudotime/weight estimates on the test statistic (double-dipping) is ignored rather than resolved
- p-values are given a numerical-summary status (median across replicates) rather than a probabilistic interpretation
- No independent benchmarking of the condiments workflow has been performed; the authors call for it

## Evidence

- [methods] To test for differential progression, we use the __progressionTest__.: "To test for differential progression, we use the __progressionTest__."
- [supplementary] Robustness finding from the Fig. S4 simulation: [evidence span withheld — anchor status: not_found]
- [supplementary] Randomization scheme applied per misassignment level: [evidence span withheld — anchor status: not_found]
- [readme] condiments package entry point and documentation: [evidence span withheld — anchor status: external_doc]
