---
name: network-edge-perturbation-noise-analysis
description: Use when assessing sensitivity of regulator activity estimates to prior
  knowledge resource errors — e.g. simulating incomplete or spurious network edges
  before trusting downstream TF or kinase activity calls.
when_to_use_negative:
- The goal is to benchmark activity-inference accuracy against known perturbation
  labels (AUROC/AUPRC) — use the decoupleRBench perturbation benchmark (run_rna_bench.R
  / run_php_bench.R) instead
- The analysis targets scalability or runtime of methods on simulated data — use the
  scalability analysis (get_simmulated_data.R / run_scale_bench.R) instead
- Prior knowledge networks are unavailable or cannot be perturbed edge-wise, so no
  addition/deletion comparison is possible
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: decoupleR
  role: Re-infers TF and kinase activities with its method suite from networks perturbed
    by edge addition/deletion
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupleRBench
  role: Benchmarking wrapper built on decoupleR's decouple() that perturbs edges and
    evaluates activity-inference performance
  repo: https://github.com/saezlab/decoupleRBench
- name: R
  role: Runtime environment for the noise benchmark and statistical analysis scripts
    (Rscript)
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:a454c5a73d2bc4f9625a33c82c6c027f0d166ac5dcc09acaea5c380333dae51c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# network-edge-perturbation-noise-analysis

## Summary

Benchmarks how robustly biological activity inference methods (decoupleR) tolerate noise in prior knowledge networks by adding or deleting a percentage of edges from DoRothEA and kinase substrate resources, then re-inferring TF/kinase activities and correlating them to the unperturbed estimates.

## When to use

Use when assessing sensitivity of regulator activity estimates to prior knowledge resource errors — e.g. simulating incomplete or spurious network edges before trusting downstream TF or kinase activity calls.

## When NOT to use

- The goal is to benchmark activity-inference accuracy against known perturbation labels (AUROC/AUPRC) — use the decoupleRBench perturbation benchmark (run_rna_bench.R / run_php_bench.R) instead
- The analysis targets scalability or runtime of methods on simulated data — use the scalability analysis (get_simmulated_data.R / run_scale_bench.R) instead
- Prior knowledge networks are unavailable or cannot be perturbed edge-wise, so no addition/deletion comparison is possible

## Inputs

- Perturbation transcriptomics dataset (from Zenodo record 5645208)
- Perturbation phospho-proteomics dataset (from Zenodo record 5645208)
- DoRothEA gene regulatory network
- Kinase substrate network
- decoupleR method suite configuration via decoupleRBench input tibble

## Outputs

- Per-method, per-condition Spearman correlations between perturbed and original activity scores, with median correlation per condition
- One-way ANOVA and Tukey HSD statistics (F statistic, adjusted P-values) comparing predictability across normal, edge-addition and edge-deletion conditions
- Supplementary Figures S5 and S6 (noise robustness plots)

## How to apply

Retrieve the perturbation transcriptomics and phospho-proteomics datasets plus the DoRothEA gene regulatory network and kinase substrate network from Zenodo record 5645208 via `Rscript R/process/get_data.R` from the decoupleR_manuscript repository. Run the transcriptomics noise benchmark (`Rscript R/process/run_rna_noise.R`, ~6 hours) and the phospho-proteomics noise benchmark (`Rscript R/process/run_php_noise.R`, ~2 hours); these use decoupleRBench to add or delete a percentage of edges and re-infer activities with the decoupleR method suite. For each method and perturbation condition, compute Spearman correlations between activity scores from the perturbed and original unperturbed networks and take the median correlation per condition. Then perform a one-way ANOVA with Tukey's HSD posthoc test comparing predictability across normal, edge-addition and edge-deletion conditions, since deletions should hurt more than additions.

## Related tools

- **decoupleR** (Re-infers TF and kinase activities with its method suite from networks perturbed by edge addition/deletion) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupleRBench** (Benchmarking wrapper built on decoupleR's decouple() that perturbs edges and evaluates activity-inference performance) — https://github.com/saezlab/decoupleRBench
- **R** (Runtime environment for the noise benchmark and statistical analysis scripts (Rscript)) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html

## Examples

```
Rscript R/process/run_rna_noise.R && Rscript R/process/run_php_noise.R
```

## Evaluation signals

- Median Spearman correlation to original activities approximates 0.84 for edge addition and 0.77 for edge deletion
- Tukey HSD adjusted P-values are <2.2e-16 for normal-addition, normal-deletion and deletion-addition pairwise comparisons, with F = 131
- Edge deletion produces a larger drop in correlation and predictability than edge addition (expected ordering: addition > deletion robustness)
- Reproduced results match Supplementary Figures S5 and S6
- Runtimes fall in the expected range: ~6 hours for run_rna_noise.R and ~2 hours for run_php_noise.R

## Limitations

- Compute cost: the RNA noise benchmark takes ~6 hours and the phospho-proteomics noise benchmark ~2 hours
- Deletion of edges affects activity estimates more than addition, so conclusions about robustness to 'noise' are asymmetric and depend on the perturbation direction
- The manuscript README notes this analysis 'might take a while', implying it is not suitable for quick exploratory runs

## Evidence

- [results] Activity estimates are more affected by deleting edges than by adding them: median Spearman correlation to original activities was 0.84 for edge addition and 0.77 for edge deletion: [evidence span withheld — anchor status: not_found]
- [methods] one-way ANOVA followed by Tukey's HSD posthoc test comparing predictability across normal, edge-addition and edge-deletion conditions: [evidence span withheld — anchor status: not_found]
- [readme] Rscript R/process/run_rna_noise.R #(~6 hours): "# Run noise analysis (This might take a while)
Rscript R/process/run_rna_noise.R #(~6 hours)
Rscript R/process/run_php_noise.R #(~2 hours)"
- [readme] To retrieve all the necessary data, please run: Rscript R/process/get_data.R: "To retrieve all the necessary data, please run:
```
Rscript R/process/get_data.R
```"
- [abstract] the data in Zenodo (https://zenodo.org/record/5645208): "and the data in Zenodo (https://zenodo.org/record/5645208)"
