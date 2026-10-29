---
name: regulator-recovery-sensitivity-evaluation
description: Use when evaluating which activity-inference methods recover perturbed
  regulators from phospho-proteomics perturbation data, or when reproducing the decoupleR
  phospho-proteomics method ranking.
when_to_use_negative:
- The goal is transcriptomic (RNA) perturbation benchmarking — run Rscript R/process/run_rna_bench.R
  instead of the phospho-proteomics benchmark
- The task is the noise or scalability analysis rather than the main perturbation
  benchmark (run_php_noise.R / get_simmulated_data.R are separate scripts)
- Input is single-dataset activity inference without perturbation ground truth — there
  is no response vector to compute AUROC/AUPRC against
edam_operation: http://edamontology.org/operation_1815
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0203
tools:
- name: decoupleR
  role: R package providing the 11 statistical methods (AUCell, fast GSEA, GSVA, ORA,
    ULM, VIPER, etc.) plus the consensus score used to infer regulator activities
    via the decouple wrapper
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupleRBench
  role: Dedicated benchmarking package installed with devtools::install_github('saezlab/decoupleRBench');
    concatenates experiments, computes AUROC/AUPRC with downsampling of negatives
  repo: https://github.com/saezlab/decoupleRBench
- name: R
  role: Execution environment; benchmark driven by Rscript commands from the manuscript
    repository
  repo: https://github.com/saezlab/decoupleR_manuscript
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:74199d87d90f6f753b647e3cd58fee56ac46c0868c59f8a0e61c011d1534c9c4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: http://edamontology.org/operation_1815
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# regulator-recovery-sensitivity-evaluation

## Summary

Benchmark procedure that ranks decoupleR statistical methods by their ability to recover perturbed regulators (kinases) from phospho-proteomics perturbation experiments, using decoupleRBench with sensitivity-focused evaluation. Reproduces the paper's finding that consensus, multivariate linear model (mlm) and ULM are the top three methods.

## When to use

Use when evaluating which activity-inference methods best recover perturbed regulators from phospho-proteomics perturbation data, or when reproducing the decoupleR phospho-proteomics method ranking.

## When NOT to use

- The goal is transcriptomic (RNA) perturbation benchmarking — run Rscript R/process/run_rna_bench.R instead of the phospho-proteomics benchmark
- The task is the noise or scalability analysis rather than the main perturbation benchmark (run_php_noise.R / get_simmulated_data.R are separate scripts)
- Input is single-dataset activity inference without perturbation ground truth — there is no response vector to compute AUROC/AUPRC against

## Inputs

- Phospho-proteomics perturbation datasets (Hernandez-Armenta et al. 2017 collection) downloaded from Zenodo record 5645208 via get_data.R
- Kinase substrate prior knowledge network
- Benchmark input tibble with user-specified settings, count table and corresponding metadata table

## Outputs

- php_bench_performance.csv containing per-method performance metrics (sensitivity, AUROC, AUPRC)
- Phospho-proteomics method ranking (top three: consensus, multivariate linear model, ULM) for visual comparison against the transcriptomics ranking

## How to apply

Retrieve the benchmark data by running 'Rscript R/process/get_data.R', which downloads the deposited perturbation datasets and prior knowledge resources from Zenodo record 5645208. Install the dedicated benchmarking package with devtools::install_github('saezlab/decoupleRBench'). Execute the phospho-proteomics benchmark with 'Rscript R/process/run_php_bench.R', which runs decoupleR methods on the Hernandez-Armenta et al. 2017 phospho-proteomics perturbation collection against the kinase substrate network, scoring recovery of perturbed kinases with evaluation focused on sensitivity. Activities are inferred for each regulator and experiment; all experiments are concatenated into a response vector (whether a regulator is perturbed) and a predictor vector (regulator activities), and AUROC/AUPRC are computed across thresholds with a downsampling strategy that randomly samples an equal number of negative classes per permutation. Aggregate the per-method performance metrics for the 11 decoupleR methods (AUCell, fast GSEA, GSVA, ORA, ULM, VIPER, etc.) plus the consensus score, write them to php_bench_performance.csv, and compare the phospho-proteomics ranking against the transcriptomics ranking.

## Related tools

- **decoupleR** (R package providing the 11 statistical methods (AUCell, fast GSEA, GSVA, ORA, ULM, VIPER, etc.) plus the consensus score used to infer regulator activities via the decouple wrapper) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupleRBench** (Dedicated benchmarking package installed with devtools::install_github('saezlab/decoupleRBench'); concatenates experiments, computes AUROC/AUPRC with downsampling of negatives) — https://github.com/saezlab/decoupleRBench
- **R** (Execution environment; benchmark driven by Rscript commands from the manuscript repository) — https://github.com/saezlab/decoupleR_manuscript

## Examples

```
Rscript R/process/get_data.R && devtools::install_github('saezlab/decoupleRBench') && Rscript R/process/run_php_bench.R
```

## Evaluation signals

- php_bench_performance.csv is produced with per-method metrics for the 11 decoupleR methods plus the consensus score
- The phospho-proteomics sensitivity ranking reproduces the paper's top three methods: consensus, multivariate linear model and ULM
- Evaluation is computed over concatenated experiments using a downsampling strategy where an equal number of negative classes are randomly sampled per permutation
- The phospho-proteomics ranking is visually compared against the transcriptomics ranking for consistency with the paper's finding that simple linear models and the consensus score perform best

## Limitations

- True positive classes are limited by the regulators covered in the perturbation experiments, requiring downsampling of negative classes
- The phospho-proteomics noise analysis (run_php_noise.R) takes ~2 hours and may be slow
- decoupleRBench builds on the R decoupleR implementation, which is deprecated in favor of the scverse/decoupler Python package

## Evidence

- [results] top three methods: [evidence span withheld — anchor status: not_found]
- [readme] benchmark installation: "You can install it by running:
``ndevtools::install_github('saezlab/decoupleRBench')
"
- [readme] evaluation strategy: [evidence span withheld — anchor status: external_doc]
- [readme] data retrieval: "To retrieve all the necessary data, please run:
``nRscript R/process/get_data.R
"
- [abstract] overall finding: "Our findings suggest that simple linear models and the consensus score across top methods perform better than other methods at predicting perturbed regulators."
