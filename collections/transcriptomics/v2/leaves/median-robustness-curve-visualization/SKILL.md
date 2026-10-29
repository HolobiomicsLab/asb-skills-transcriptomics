---
name: median-robustness-curve-visualization
description: Use when stress-testing a trajectory-level differential test against
  trajectory-inference errors — you have a simulated reference with true pseudotimes/lineage
  labels and want sensitivity to pseudotime noise and lineage misassignment.
when_to_use_negative:
- The trajectory topology itself differs between conditions — differential topology
  should be assessed first with topologyTest, not progressionTest robustness curves
- Input is real data without ground-truth pseudotimes and lineage assignments — perturbation-based
  robustness quantification requires known truth
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: condiments
  role: Provides progressionTest (custom KS test, global=TRUE, lineages=TRUE) run
    on each perturbed replicate
  repo: github.com/HectorRDB/condiments
- name: dyngen
  role: Generates the T1 reference simulation with known pseudotimes and lineage assignments
    (noise m=0.85)
- name: slingshot
  role: Trajectory inference supplying pseudotimes/lineage assignments used in the
    perturbation pipeline
- name: ggplot2
  role: Plots the median-p-value curves versus noise level and misassignment proportion
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:5b209677be399986055361d5f2114d6e6d7d0342bb76e38caff728893090a94b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - condiments
  - dyngen
  - ggplot2
  - slingshot
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# median-robustness-curve-visualization

## Summary

Reproduces the condiments progressionTest noise-robustness simulation (Fig. S4): perturb known pseudotimes with Gaussian noise and randomly misassign cells to lineages, then aggregate progressionTest p-values across replicates into median-p-value curves plotted with ggplot2.

## When to use

Use when stress-testing a trajectory-level differential test against trajectory-inference errors — you have a simulated reference with true pseudotimes/lineage labels and want sensitivity to pseudotime noise and lineage misassignment.

## When NOT to use

- The trajectory topology itself differs between conditions — differential topology should be assessed first with topologyTest, not progressionTest robustness curves
- Input is real data without ground-truth pseudotimes and lineage assignments — perturbation-based robustness quantification requires known truth

## Inputs

- dyngen T1 reference simulation (noise parameter m=0.85) with ground-truth pseudotimes and lineage assignments
- Grid of pseudotime noise standard deviations over [0, 4]
- Lineage misassignment proportions p in {0, 0.1, 0.2, 0.3, 0.4, 0.5}
- 100 perturbed replicates per noise/misassignment combination

## Outputs

- Median p-value per noise/misassignment setting (aggregated across 100 replicates)
- median_pvalue_curves.png — ggplot2 curves of median p-value versus pseudotime noise level and lineage misassignment proportion

## How to apply

Generate the dyngen T1 reference simulation with noise parameter m=0.85 as the ground-truth trajectory with known pseudotimes and lineage assignments. For each pseudotime noise level, perturb the true pseudotimes with additive Gaussian noise whose standard deviation ranges over [0, 4] (up to 200% of the pseudotime scale); separately, for each misassignment proportion p in {0, 0.1, 0.2, 0.3, 0.4, 0.5}, randomly swap lineage assignments for that fraction of cells. Run condiments::progressionTest on each perturbed replicate using the custom KS test with global=TRUE and lineages=TRUE across 100 replicates per noise/misassignment combination. Compute the median p-value across replicates for each setting and plot median-p-value curves versus pseudotime noise and misassignment proportion with ggplot2, saving as median_pvalue_curves.png. The rationale is that the paper's reported robustness — correct rejection of the null at the 5% nominal level despite TI errors — depends on aggregating over replicates via the median rather than single runs.

## Related tools

- **condiments** (Provides progressionTest (custom KS test, global=TRUE, lineages=TRUE) run on each perturbed replicate) — github.com/HectorRDB/condiments
- **dyngen** (Generates the T1 reference simulation with known pseudotimes and lineage assignments (noise m=0.85))
- **slingshot** (Trajectory inference supplying pseudotimes/lineage assignments used in the perturbation pipeline)
- **ggplot2** (Plots the median-p-value curves versus noise level and misassignment proportion)

## Examples

```
Rscript robustness_sim.R  # per replicate: condiments::progressionTest(sds, conditions = cond, global = TRUE, lineages = TRUE); ggplot2::ggsave('median_pvalue_curves.png')
```

## Evaluation signals

- Median p-value curves show the null is still rejected at the 5% nominal level with up to 60% of cells improperly assigned and 200% pseudotime noise
- At p=0 misassignment and zero noise, the median p-value matches the unperturbed progressionTest result
- Each median is computed over exactly 100 replicates per noise/misassignment combination
- Figure median_pvalue_curves.png contains one median-p-value curve per setting versus pseudotime noise and versus misassignment proportion

## Limitations

- Robustness is demonstrated only for moderate trajectory-inference errors on a dyngen simulation; no independent benchmarking of the workflow has been performed and the authors call for it
- P-values are not given a probabilistic interpretation, only numerical-summary status, so the median p-value is a summary rather than a calibrated error rate
- Dependence of pseudotime/weight estimates (double-dipping) is ignored rather than resolved

## Evidence

- [supplementary] Perturbation grid defines robustness metric: [evidence span withheld — anchor status: not_found]
- [supplementary] Replicate aggregation procedure: [evidence span withheld — anchor status: not_found]
- [methods] Test underpinning the curves: "To test for differential progression, we use the __progressionTest__."
- [readme] Package installation entry point: [evidence span withheld — anchor status: external_doc]
