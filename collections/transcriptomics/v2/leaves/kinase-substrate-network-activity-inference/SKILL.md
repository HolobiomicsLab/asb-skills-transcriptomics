---
name: kinase-substrate-network-activity-inference
description: Use when evaluating which biological activity inference methods recover
  perturbed regulators (kinases) from phospho-proteomics perturbation experiments
  using a kinase-substrate network.
when_to_use_negative:
- Input is transcriptomics rather than phospho-proteomics perturbation data — use
  the run_rna_bench.R transcriptomics benchmark instead
- Goal is downstream activity inference in Python (scverse/decoupler) rather than
  benchmarking method performance in R — decoupleRBench is a benchmarking pipeline,
  not an inference API
- Task is the noise or scalability analyses (run_php_noise.R, run_scale_bench.R) rather
  than the perturbation benchmark ranking
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0204
tools:
- name: decoupleR
  role: Provides the unified framework of statistical methods (AUCell, fast GSEA,
    GSVA, ORA, ULM, VIPER, multivariate linear model, consensus score, etc.) whose
    kinase activity inferences are benchmarked
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupleRBench
  role: 'Dedicated R package that runs the perturbation benchmark: concatenates experiments,
    computes AUROC/AUPRC with class downsampling, and evaluates each method'
  repo: https://github.com/saezlab/decoupleRBench
- name: R
  role: Language in which the benchmark pipeline scripts (get_data.R, run_php_bench.R)
    are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:a2277facdc2c8310d8fc95d2f86c8ab1c2a405a366ebd17281594d77dde80423
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# kinase-substrate-network-activity-inference

## Summary

Reproduce the decoupleR phospho-proteomics perturbation benchmark, which scores recovery of perturbed kinases from the Hernandez-Armenta et al. 2017 dataset using a kinase substrate prior knowledge network. It identifies which decoupleR methods (top three: consensus, multivariate linear model, ULM) best infer kinase activities from phospho-proteomics data.

## When to use

Use when evaluating which biological activity inference methods best recover perturbed regulators (kinases) from phospho-proteomics perturbation experiments using a kinase-substrate network.

## When NOT to use

- Input is transcriptomics rather than phospho-proteomics perturbation data — use the run_rna_bench.R transcriptomics benchmark instead
- Goal is downstream activity inference in Python (scverse/decoupler) rather than benchmarking method performance in R — decoupleRBench is a benchmarking pipeline, not an inference API
- Task is the noise or scalability analyses (run_php_noise.R, run_scale_bench.R) rather than the perturbation benchmark ranking

## Inputs

- Phospho-proteomics perturbation collection (Hernandez-Armenta et al. 2017) downloaded via Zenodo record 5645208
- Kinase substrate prior knowledge network
- Benchmark data as a count table and corresponding metadata table (decoupleRBench input tibble with user-specified settings)

## Outputs

- php_bench_performance.csv with per-method performance metrics (sensitivity, AUROC, AUPRC) for the 11 decoupleR methods plus the consensus score
- Method ranking of the phospho-proteomics benchmark comparable to the transcriptomics ranking

## How to apply

Retrieve all benchmark data by running 'Rscript R/process/get_data.R', which downloads the deposited perturbation datasets and prior knowledge resources from Zenodo record 5645208. Install the dedicated benchmarking package with devtools::install_github('saezlab/decoupleRBench'), which wraps the decoupleR 'decouple' function. Execute the phospho-proteomics benchmark with 'Rscript R/process/run_php_bench.R', which runs the 11 decoupleR methods (AUCell, fast GSEA, GSVA, ORA, ULM, VIPER, etc.) plus the consensus score on the Hernandez-Armenta et al. 2017 phospho-proteomics perturbation collection against the kinase substrate network. For each method, activities are inferred for each regulator and experiment, then concatenated to build a response vector (whether a kinase is perturbed) and a predictor vector (the kinase activities); AUROC and AUPRC are computed across thresholds with a downsampling strategy that randomly samples an equal number of negative classes per permutation, and evaluation focuses on sensitivity. Write per-method performance metrics to php_bench_performance.csv and compare the phospho-proteomics ranking against the transcriptomics ranking from run_rna_bench.R.

## Related tools

- **decoupleR** (Provides the unified framework of statistical methods (AUCell, fast GSEA, GSVA, ORA, ULM, VIPER, multivariate linear model, consensus score, etc.) whose kinase activity inferences are benchmarked) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupleRBench** (Dedicated R package that runs the perturbation benchmark: concatenates experiments, computes AUROC/AUPRC with class downsampling, and evaluates each method) — https://github.com/saezlab/decoupleRBench
- **R** (Language in which the benchmark pipeline scripts (get_data.R, run_php_bench.R) are executed)

## Examples

```
Rscript R/process/run_php_bench.R
```

## Evaluation signals

- php_bench_performance.csv is produced with per-method metrics covering all 11 decoupleR methods plus the consensus score
- The sensitivity ranking reproduces the paper's reported result: top three methods are consensus, multivariate linear model and ULM
- The phospho-proteomics ranking is visually comparable to the transcriptomics ranking generated by run_rna_bench.R
- AUROC/AUPRC are computed with the decoupleRBench downsampling strategy (equal number of negative classes randomly sampled per permutation) rather than naive class imbalance

## Limitations

- True positive classes are limited by the regulators covered in the perturbation experiments, requiring a downsampling strategy for AUROC/AUPRC calculation
- The data retrieval step depends on external resources (Zenodo record 5645208); decoupleRBench requires decoupleR to be installed beforehand
- Simple linear models and the consensus score outperform other methods on these perturbation benchmarks, so single-method rankings may not generalize to all omics contexts

## Evidence

- [readme] top methods for phospho-proteomics: "To retrieve all the necessary data, please run:
```
Rscript R/process/get_data.R
```"
- [readme] benchmark install: "devtools::install_github('saezlab/decoupleRBench')"
- [readme] benchmark evaluation design: [evidence span withheld — anchor status: external_doc]
- [readme] downsampling rationale: [evidence span withheld — anchor status: external_doc]
- [abstract] key finding: "Our findings suggest that simple linear models and the consensus score across top methods perform better than other methods at predicting perturbed regulators."
- [abstract] scope of evaluation: "Using decoupleR, we evaluated the performance of methods on transcriptomic and phospho-proteomic perturbation experiments."
