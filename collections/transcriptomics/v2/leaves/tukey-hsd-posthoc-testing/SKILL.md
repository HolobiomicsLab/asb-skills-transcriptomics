---
name: tukey-hsd-posthoc-testing
description: Use when you have per-method, per-condition activity-inference scores
  (e.g. Spearman correlations to unperturbed activities across normal, edge-addition
  and edge-deletion networks) and need an omnibus ANOVA plus pairwise Tukey HSD comparison
  of group means.
when_to_use_negative:
- The data are simple paired observations with only two groups (e.g. one perturbed
  vs one unperturbed condition) — a paired test suffices; ANOVA+Tukey adds no information
- The analysis compares method ranking performance (AUROC/AUPRC from decoupleRBench)
  rather than predictability across noise conditions — use the benchmark evaluation
  pipeline instead
- Only a single network perturbation condition exists (no normal/addition/deletion
  grouping), so the three-level ANOVA design cannot be formed
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: decoupleR
  role: Re-infers TF and kinase activities with its method suite on perturbed and
    original networks, producing the activity scores whose predictability is compared
    by ANOVA/Tukey HSD
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupleRBench
  role: Benchmarking package used to add or delete a percentage of edges from DoRothEA
    and the kinase substrate network and to run the decoupleR method suite under each
    condition
  repo: https://github.com/saezlab/decoupleRBench
- name: R
  role: Environment in which the noise benchmarks, ANOVA/Tukey HSD statistics and
    supplementary figure scripts are executed
  repo: https://github.com/saezlab/decoupleR_manuscript
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:fb71efae06e73861edf63626c43fd59de36d7a2a4ccb5aaa779342553b9b8eaf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# tukey-hsd-posthoc-testing

## Summary

Apply a one-way ANOVA followed by Tukey's HSD posthoc test to compare method predictability across prior-knowledge perturbation conditions (normal, edge addition, edge deletion) in the decoupleR noise-robustness benchmark, yielding adjusted P-values and F statistics.

## When to use

Use when you have per-method, per-condition activity-inference scores (e.g. Spearman correlations to unperturbed activities across normal, edge-addition and edge-deletion networks) and need an omnibus ANOVA plus pairwise Tukey HSD comparison of group means.

## When NOT to use

- The data are simple paired observations with only two groups (e.g. one perturbed vs one unperturbed condition) — a paired test suffices; ANOVA+Tukey adds no information
- The analysis compares method ranking performance (AUROC/AUPRC from decoupleRBench) rather than predictability across noise conditions — use the benchmark evaluation pipeline instead
- Only a single network perturbation condition exists (no normal/addition/deletion grouping), so the three-level ANOVA design cannot be formed

## Inputs

- Per-method activity scores from perturbed (edge addition/deletion) and original DoRothEA gene regulatory networks (transcriptomics benchmark)
- Per-method activity scores from perturbed and original kinase substrate networks (phospho-proteomics benchmark)
- Condition labels (normal, edge addition, edge deletion) per method and perturbation percentage
- Spearman correlation values between perturbed-network and original-network activity scores per method/condition

## Outputs

- One-way ANOVA F statistic comparing predictability across conditions (expected F = 131)
- Tukey HSD pairwise adjusted P-values for normal-addition, normal-deletion and deletion-addition (expected <2.2e-16)
- Median Spearman correlation per condition (expected 0.84 for edge addition, 0.77 for edge deletion)
- Supplementary Figures S5 and S6 (noise robustness plots)

## How to apply

After running the noise benchmarks (run_rna_noise.R ~6 h on DoRothEA; run_php_noise.R ~2 h on the kinase substrate network), compute per-method Spearman correlations between activity scores from each perturbed network and the original unperturbed network, then take the median correlation per condition. Perform a one-way ANOVA on predictability across the three conditions (normal, edge addition, edge deletion), then apply Tukey's HSD posthoc test for all pairwise comparisons with adjusted P-values. In the decoupleR manuscript this yielded F = 131 and adjusted P-values <2.2e-16 for normal-addition, normal-deletion and deletion-addition, showing edge deletion degrades activity estimates more than edge addition (median Spearman 0.77 vs 0.84). Validate the reproduction against Supplementary Figures S5 and S6.

## Related tools

- **decoupleR** (Re-infers TF and kinase activities with its method suite on perturbed and original networks, producing the activity scores whose predictability is compared by ANOVA/Tukey HSD) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupleRBench** (Benchmarking package used to add or delete a percentage of edges from DoRothEA and the kinase substrate network and to run the decoupleR method suite under each condition) — https://github.com/saezlab/decoupleRBench
- **R** (Environment in which the noise benchmarks, ANOVA/Tukey HSD statistics and supplementary figure scripts are executed) — https://github.com/saezlab/decoupleR_manuscript

## Examples

```
Rscript R/process/run_rna_noise.R && Rscript R/plot/supp_fig_5.R
```

## Evaluation signals

- Median Spearman correlation of perturbed-network activities to original activities approximates 0.84 (edge addition) and 0.77 (edge deletion)
- Tukey HSD adjusted P-values are <2.2e-16 for all three pairwise comparisons (normal-addition, normal-deletion, deletion-addition)
- One-way ANOVA F statistic equals 131
- Reproduced statistics match Supplementary Figures S5 and S6 of the manuscript
- Edge-deletion condition shows lower predictability than edge addition (deletion is the more damaging perturbation)

## Limitations

- The RNA noise benchmark takes ~6 hours and the phospho-proteomics noise benchmark ~2 hours, so the statistics cannot be recomputed quickly without the cached benchmark outputs
- Tukey HSD adjusted P-values saturate at the reported <2.2e-16 floor, so finer discrimination between condition effects relies on the F statistic and effect sizes (median correlations), not the P-value alone
- The comparison assumes predictability is comparable across methods and perturbation percentages within each condition; decoupleRBench notes true-positive classes are limited by covered regulators and uses downsampling, which conditions the response/predictor vectors

## Evidence

- [results] median Spearman correlations and ANOVA thresholds: [evidence span withheld — anchor status: not_found]
- [methods] ANOVA + Tukey procedure: [evidence span withheld — anchor status: not_found]
- [methods] correlation computation step: [evidence span withheld — anchor status: not_found]
- [readme] reproduction pipeline entry point: "To reproduce all the analyses performed in the manuscript, please run"
