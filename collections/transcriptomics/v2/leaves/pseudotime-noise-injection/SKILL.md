---
name: pseudotime-noise-injection
description: Use when you need to quantify how sensitive a trajectory-level differential
  progression test (e.g., condiments progressionTest) is to imperfect pseudotime estimates
  or incorrect cell-to-lineage assignments from upstream trajectory inference.
when_to_use_negative:
- The question is differential topology between conditions (use topologyTest) or gene-level
  differential expression along lineages (use conditionTest), not robustness of the
  progression test
- No ground-truth trajectory with known pseudotimes and lineage assignments is available,
  so perturbation errors cannot be quantified against truth
- The system is a steady-state process where trajectory-based differential progression
  is not the relevant contrast (cluster-based differential abundance methods are preferred
  instead)
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: condiments
  role: Provides progressionTest (custom KS test, global=TRUE, lineages=TRUE) evaluated
    under injected pseudotime noise and lineage misassignment
  repo: github.com/HectorRDB/condiments
- name: dyngen
  role: Generates the T1 reference simulation with ground-truth pseudotimes and lineage
    assignments used as the perturbation target
- name: slingshot
  role: Trajectory inference context for lineage and pseudotime estimates that progressionTest
    consumes
- name: ggplot2
  role: Plots the median-p-value robustness curves saved as median_pvalue_curves.png
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:8a3485bd6b6298ae0e3f9cc4280e9c4350fa16a33a5e085caf3b81b2edde2be5
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
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pseudotime-noise-injection

## Summary

A simulation-based robustness check for condiments::progressionTest that injects controlled errors into trajectory inference outputs — additive Gaussian noise on pseudotimes and random lineage misassignment of cells — on a dyngen ground-truth simulation with known pseudotimes and lineage assignments, to measure how TI errors affect rejection of the null of no differential progression.

## When to use

Use when you need to quantify how sensitive a trajectory-level differential progression test (e.g., condiments progressionTest) is to imperfect pseudotime estimates or incorrect cell-to-lineage assignments from upstream trajectory inference.

## When NOT to use

- The question is differential topology between conditions (use topologyTest) or gene-level differential expression along lineages (use conditionTest), not robustness of the progression test
- No ground-truth trajectory with known pseudotimes and lineage assignments is available, so perturbation errors cannot be quantified against truth
- The system is a steady-state process where trajectory-based differential progression is not the relevant contrast (cluster-based differential abundance methods are preferred instead)

## Inputs

- dyngen T1 reference simulation with known ground-truth pseudotimes and lineage assignments (noise parameter m=0.85)
- Grid of pseudotime noise standard deviations over [0, 4]
- Grid of lineage misassignment proportions p in {0, 0.1, 0.2, 0.3, 0.4, 0.5}

## Outputs

- Median progressionTest p-value per noise/misassignment combination across 100 replicates
- Median-p-value curves versus pseudotime noise and misassignment proportion (median_pvalue_curves.png, ggplot2)

## How to apply

Generate a dyngen reference simulation (T1, noise parameter m=0.85) that provides ground-truth pseudotimes and lineage assignments. For each noise level, perturb true pseudotimes with additive Gaussian noise whose standard deviation spans [0, 4] (up to 200% of the pseudotime scale); independently, randomly swap lineage assignments for a fraction p of cells with p in {0, 0.1, 0.2, 0.3, 0.4, 0.5}. Run condiments::progressionTest (custom KS test, global=TRUE, lineages=TRUE) on each perturbed replicate across 100 replicates per noise/misassignment combination, compute the median p-value per setting, and plot median-p-value curves versus noise level and misassignment proportion with ggplot2. Correct application is judged by whether the curves confirm that the test still rejects the null at the 5% nominal level up to roughly 60% misassigned cells and 200% pseudotime noise, matching the paper's Fig. S4.

## Related tools

- **condiments** (Provides progressionTest (custom KS test, global=TRUE, lineages=TRUE) evaluated under injected pseudotime noise and lineage misassignment) — github.com/HectorRDB/condiments
- **dyngen** (Generates the T1 reference simulation with ground-truth pseudotimes and lineage assignments used as the perturbation target)
- **slingshot** (Trajectory inference context for lineage and pseudotime estimates that progressionTest consumes)
- **ggplot2** (Plots the median-p-value robustness curves saved as median_pvalue_curves.png)

## Examples

```
BiocManager::install("condiments"); pt_noisy <- pst_true + rnorm(length(pst_true), sd = noise_sd); pvals[s, ] <- condiments::progressionTest(sds, conditions = cond, global = TRUE, lineages = TRUE)$pvalues
```

## Evaluation signals

- Median p-value curves show rejection of the null at the 5% nominal level up to approximately 60% lineage misassignment
- Rejection persists up to 200% pseudotime noise (standard deviation up to ~4 on the pseudotime scale)
- At zero noise and zero misassignment, median p-values are significant, confirming the unperturbed test detects true differential progression
- Results are stable across the 100 replicates per noise/misassignment combination (median, not mean, p-values used)
- Reproduced curves match the paper's Fig. S4 robustness pattern for progressionTest

## Limitations

- Findings hold for the dyngen T1 simulation setting; the authors note no independent benchmarking of the workflow has been performed and call for it
- P-value dependence from double-dipping (pseudotime/weight estimates reused by the test) is ignored rather than resolved
- P-values are given numerical-summary status only, not a strong probabilistic interpretation

## Evidence

- [methods] To test for differential progression, we use the __progressionTest__.: "To test for differential progression, we use the __progressionTest__."
- [discussion] Simulation frameworks such as dyngen24 are crucial.: "Simulation frameworks such as dyngen24 are crucial."
- [readme] The installation should only take a few seconds.: [evidence span withheld — anchor status: external_doc]
- [intro] In Step 2, it tests for global differences between conditions, both along lineages – differential progression – and between lineages – differential fate selection.: "In Step 2, it tests for global differences between conditions, both along lineages – differential progression – and between lineages – differential fate selection."
