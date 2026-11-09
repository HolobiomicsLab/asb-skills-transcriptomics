---
name: simulated-omics-data-generation
description: Use when a scalability or performance benchmark of decoupleR's activity-inference
  methods is needed and a controlled synthetic omics matrix must be produced instead
  of real perturbation data.
when_to_use_negative:
- When the goal is method-accuracy benchmarking on perturbation experiments (AUROC/AUPRC)
  — use the decoupleRBench pipeline (run_rna_bench.R / run_php_bench.R) instead of
  simulated data.
- When real transcriptomic or phospho-proteomic perturbation count tables and metadata
  are already available and runtime profiling is not the objective.
- When noise-sensitivity analysis is intended (run_rna_noise.R / run_php_noise.R),
  which evaluates robustness rather than execution speed.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: decoupleR
  role: Package whose 11 statistical methods are timed on the simulated matrices in
    both R and Python implementations
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: Python
  role: Implementation of decoupleR (decoupler-py) benchmarked against the R version
    for runtime comparison
  repo: https://github.com/saezlab/decoupler-py
- name: R
  role: Language in which the simulated data are generated (get_simmulated_data.R)
    and the R implementation of decoupleR is benchmarked
- name: decoupleRBench
  role: Separate R package built to benchmark decoupleR; installed via devtools::install_github('saezlab/decoupleRBench')
  repo: https://github.com/saezlab/decoupleRBench
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:7f992fdbcbbf02be30d80c4e1a2a3c5eb081aa5e4ece9f3b22364c485e82f7e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Simulated omics data generation

## Summary

Generates a simulated omics dataset from the decoupleR_manuscript repository (Zenodo record 5645208) to serve as the test matrix for decoupleR's runtime scalability benchmark, timing all 11 methods in both the R and Python implementations.

## When to use

Use when a scalability or performance benchmark of decoupleR's activity-inference methods is needed and a controlled synthetic omics matrix must be produced instead of real perturbation data.

## When NOT to use

- When the goal is method-accuracy benchmarking on perturbation experiments (AUROC/AUPRC) — use the decoupleRBench pipeline (run_rna_bench.R / run_php_bench.R) instead of simulated data.
- When real transcriptomic or phospho-proteomic perturbation count tables and metadata are already available and runtime profiling is not the objective.
- When noise-sensitivity analysis is intended (run_rna_noise.R / run_php_noise.R), which evaluates robustness rather than execution speed.

## Inputs

- Zenodo record 5645208 analysis data (https://zenodo.org/record/5645208)
- decoupleR_manuscript repository scripts (R/process/get_simmulated_data.R, run_scale_bench.R)
- decoupleR R (Bioconductor) and Python implementations installed

## Outputs

- Simulated omics matrix for the scalability test
- runtime_table.csv with median runtime per sample and regulator per implementation (R vs Python), reproducing Supplementary Fig. S7

## How to apply

Download the required analysis data from the Zenodo record (https://zenodo.org/record/5645208) referenced by the decoupleR manuscript. Generate the simulated omics dataset by running 'Rscript R/process/get_simmulated_data.R' from the decoupleR_manuscript repository, which produces the synthetic matrices used for the scalability test. Then execute the benchmark via 'bash run_scale_bench.R', timing each of decoupleR's methods in both the R (Bioconductor) and Python implementations on the simulated matrices. Compute the median runtime per sample and regulator across methods for each implementation, matching the paper's reporting convention of ms/sample/regulator on an Intel(R) Core(TM) i7-8550U CPU @ 1.80 GHz. Assemble the measurements into a comparison table reproducing the structure of Supplementary Fig. S7 (runtime_table.csv).

## Related tools

- **decoupleR** (Package whose 11 statistical methods are timed on the simulated matrices in both R and Python implementations) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **Python** (Implementation of decoupleR (decoupler-py) benchmarked against the R version for runtime comparison) — https://github.com/saezlab/decoupler-py
- **R** (Language in which the simulated data are generated (get_simmulated_data.R) and the R implementation of decoupleR is benchmarked)
- **decoupleRBench** (Separate R package built to benchmark decoupleR; installed via devtools::install_github('saezlab/decoupleRBench')) — https://github.com/saezlab/decoupleRBench

## Examples

```
Rscript R/process/get_simmulated_data.R && bash run_scale_bench.R
```

## Evaluation signals

- Median runtime per sample and regulator across the 11 methods is on the order of the published values: 1.44 ms in R and 0.44 ms in Python (Intel Core i7-8550U CPU @ 1.80 GHz).
- Python implementation shows a substantially lower per-sample-and-regulator runtime than R, reproducing the qualitative gap shown in Supplementary Fig. S7.
- runtime_table.csv reproduces the structure of Supplementary Fig. S7 (per-method runtimes for both implementations).
- Simulated dataset is generated successfully by get_simmulated_data.R before run_scale_bench.R is executed, per the README's stated ordering of steps.
- Runtime units match the paper's reporting convention: ms per sample and regulator.

## Limitations

- The scalability analysis 'might take a while' per the manuscript README, so long execution time is expected.
- Reported median runtimes are hardware-specific (Intel Core i7-8550U CPU @ 1.80 GHz); absolute values will differ on other hardware.
- The scverse decoupler README notes the R version is deprecated ('a deprecated version in R can be found here'), so R-side timing reflects a legacy implementation.
- Benchmarks cover the 11 methods available at publication time; newer method sets may not map 1:1 onto Supplementary Fig. S7.

## Evidence

- [results] Across the 11 methods, the median runtime is 1.44 ms per sample and regulator in R and 0.44 ms in Python (Intel Core i7-8550U CPU @ 1.80 GHz): [evidence span withheld — anchor status: not_found]
- [readme] Run scalablity analysis (This might take a while): "# Run scalablity analysis (This might take a while)
Rscript R/process/get_simmulated_data.R
bash run_scale_bench.R"
- [readme] This is its faster and memory efficient Python implementation, a deprecated version in R can be found: [evidence span withheld — anchor status: external_doc]
- [readme] In this manuscript we have built a separate R package to benchmark decoupleR, called decoupleRBench: "In this manuscript we have built a separate R package to benchmark decoupleR,
called decoupleRBench."
