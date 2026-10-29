---
name: spearman-correlation-robustness-assessment
description: Use when you need to measure how much adding or deleting a percentage
  of edges in a regulatory or kinase-substrate network changes downstream activity
  inference, and whether the change is statistically significant.
when_to_use_negative:
- Your goal is method performance benchmarking against perturbation ground truth (AUROC/AUPRC
  of perturbed vs unperturbed regulators) rather than network-noise robustness — use
  the decoupleRBench perturbation benchmark instead
- The prior knowledge network is unchanged and you only need to infer activities once
  — no perturbation-correlation assessment is applicable
- You are running the scalability analysis on simulated data rather than assessing
  robustness to edge noise
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: decoupleR
  role: Re-infers TF/kinase activity scores with its unified method suite after edge
    addition/deletion perturbations of the network
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupleRBench
  role: R package that applies edge-addition/deletion perturbations to DoRothEA and
    the kinase substrate network and runs the benchmark pipeline built on decoupleR's
    `decouple` wrapper
  repo: https://github.com/saezlab/decoupleRBench
- name: R
  role: Language in which the noise benchmark scripts (run_rna_noise.R, run_php_noise.R),
    Spearman correlation computation, ANOVA and Tukey HSD tests are executed
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:b107fc23a0754e9d677ec08a054caba5b35acdadc9edb2e56362a8f9d6301247
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# spearman-correlation-robustness-assessment

## Summary

Quantifies the robustness of biological activity estimates (TF or kinase activities inferred with decoupleR) to noise in prior knowledge networks by computing Spearman correlations between activities from perturbed (edge-added/edge-deleted) and original networks, followed by one-way ANOVA with Tukey HSD.

## When to use

Use when you need to measure how much adding or deleting a percentage of edges in a regulatory or kinase-substrate network changes downstream activity inference, and whether the change is statistically significant.

## When NOT to use

- Your goal is method performance benchmarking against perturbation ground truth (AUROC/AUPRC of perturbed vs unperturbed regulators) rather than network-noise robustness — use the decoupleRBench perturbation benchmark instead
- The prior knowledge network is unchanged and you only need to infer activities once — no perturbation-correlation assessment is applicable
- You are running the scalability analysis on simulated data rather than assessing robustness to edge noise

## Inputs

- Perturbation transcriptomics and phospho-proteomics datasets from Zenodo record 5645208
- DoRothEA gene regulatory network (signed, weighted interactions)
- Kinase substrate network
- Activity scores inferred with the decoupleR method suite from original and edge-perturbed networks

## Outputs

- Per-method, per-condition Spearman correlations between perturbed and original activity scores
- Median Spearman correlation per perturbation condition (target: ~0.84 addition, ~0.77 deletion)
- One-way ANOVA F statistic (target F = 131) and Tukey HSD adjusted P-values (<2.2e-16) for normal-addition, normal-deletion and deletion-addition comparisons
- Supplementary Figures S5 and S6

## How to apply

Retrieve the perturbation transcriptomics/phospho-proteomics data and the DoRothEA gene regulatory network plus kinase substrate network from Zenodo record 5645208 via `Rscript R/process/get_data.R`. Run the noise benchmarks (`Rscript R/process/run_rna_noise.R`, ~6 hours; `Rscript R/process/run_php_noise.R`, ~2 hours), which use decoupleRBench to perturb network edges and re-infer activities with the decoupleR method suite. For each method and perturbation condition, compute Spearman correlations between activity scores from the perturbed network and the original unperturbed network, then take the median correlation per condition. Finally, run a one-way ANOVA with Tukey's HSD posthoc test comparing predictability across normal, edge-addition and edge-deletion conditions. Expected result: median Spearman correlations of ~0.84 (addition) and ~0.77 (deletion), with Tukey HSD adjusted P-values <2.2e-16 (F = 131), reproducing Supplementary Figures S5 and S6.

## Related tools

- **decoupleR** (Re-infers TF/kinase activity scores with its unified method suite after edge addition/deletion perturbations of the network) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupleRBench** (R package that applies edge-addition/deletion perturbations to DoRothEA and the kinase substrate network and runs the benchmark pipeline built on decoupleR's `decouple` wrapper) — https://github.com/saezlab/decoupleRBench
- **R** (Language in which the noise benchmark scripts (run_rna_noise.R, run_php_noise.R), Spearman correlation computation, ANOVA and Tukey HSD tests are executed) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html

## Examples

```
Rscript R/process/run_rna_noise.R
```

## Evaluation signals

- Reproduced median Spearman correlation of activities to the original ones approximates 0.84 for edge addition and 0.77 for edge deletion
- Tukey HSD adjusted P-values are <2.2e-16 for all three pairwise comparisons (normal-addition, normal-deletion, deletion-addition)
- One-way ANOVA F statistic equals 131
- Results reproduce Supplementary Figures S5 (RNA noise) and S6 (phospho noise)
- Edge-deletion conditions should consistently show lower correlations than edge-addition conditions (deletion affects activity estimates more)

## Limitations

- The RNA noise benchmark is computationally expensive (~6 hours) and the phospho noise benchmark ~2 hours; plan runtime accordingly
- Results depend on the specific prior knowledge resources (DoRothEA, kinase substrate network) and Zenodo snapshot (record 5645208)
- Spearman correlation captures global rank agreement of activity estimates; it does not directly validate whether the perturbed activities are biologically more or less correct

## Evidence

- [readme] noise benchmark scripts: "# Run noise analysis (This might take a while)
Rscript R/process/run_rna_noise.R #(~6 hours)
Rscript R/process/run_php_noise.R #(~2 hours)"
- [readme] decoupleR activity inference basis: [evidence span withheld — anchor status: external_doc]
- [intro] reproduction scope: "Code to reproduce the results from decoupleR's manuscript"
- [abstract] method evaluation scope: "Using decoupleR, we evaluated the performance of contemporary methods on transcriptomic and phospho-proteomic perturbation experiments."
