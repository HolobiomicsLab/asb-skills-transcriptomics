---
name: phosphoproteomics-perturbation-benchmarking
description: Use when evaluating which decoupleR methods recover perturbed kinases
  (regulators) from phospho-proteomics perturbation data, or when comparing phospho-proteomics
  method rankings against transcriptomics rankings.
when_to_use_negative:
- The goal is to benchmark methods on transcriptomic perturbation experiments (RNA)
  rather than phospho-proteomics — use run_rna_bench.R instead
- You only need to run inference on a single dataset with one chosen method rather
  than evaluate/benchmark multiple methods across perturbation experiments
- Your prior knowledge network is not a kinase-substrate resource and the perturbation
  targets are transcription factors, not kinases
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0203
tools:
- name: decoupleR
  role: R package whose decouple() wrapper infers regulator (kinase) activities from
    phospho-proteomics data using the ensemble of statistical methods being benchmarked
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupleRBench
  role: Dedicated benchmarking package that evaluates decoupleR methods on perturbation
    experiments, computing AUROC/AUPRC with a downsampling strategy
  repo: https://github.com/saezlab/decoupleRBench
- name: R
  role: Language in which the benchmark pipeline (get_data.R, run_php_bench.R) is
    implemented and executed
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: OmniPath
  role: Meta-resource of prior knowledge used by decoupleR to provide regulator-interaction
    networks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:b1fb587989e01c142481816e6725cf750fef25da9e75cedbce078057700e15cf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OmniPath
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# phosphoproteomics-perturbation-benchmarking

## Summary

Benchmark decoupleR statistical methods for inferring kinase activities from phospho-proteomics perturbation experiments using the decoupleRBench package and the Hernandez-Armenta et al. 2017 collection, ranking 11 methods plus the consensus score by sensitivity of perturbed-kinase recovery.

## When to use

Use when evaluating which decoupleR methods best recover perturbed kinases (regulators) from phospho-proteomics perturbation data, or when comparing phospho-proteomics method rankings against transcriptomics rankings.

## When NOT to use

- The goal is to benchmark methods on transcriptomic perturbation experiments (RNA) rather than phospho-proteomics — use run_rna_bench.R instead
- You only need to run inference on a single dataset with one chosen method rather than evaluate/benchmark multiple methods across perturbation experiments
- Your prior knowledge network is not a kinase-substrate resource and the perturbation targets are transcription factors, not kinases

## Inputs

- Perturbation datasets and prior knowledge resources downloaded from Zenodo record 5645208 (via Rscript R/process/get_data.R)
- Hernandez-Armenta et al. 2017 phospho-proteomics perturbation collection
- Kinase substrate network (prior knowledge resource)
- Benchmark input tibble with user-specified settings, count table, and corresponding metadata table

## Outputs

- php_bench_performance.csv with per-method performance metrics
- Comparison of the phospho-proteomics method ranking against the transcriptomics ranking

## How to apply

Retrieve all benchmark data by running 'Rscript R/process/get_data.R', which downloads the deposited perturbation datasets and prior knowledge resources from Zenodo record 5645208. Install the dedicated benchmarking package with devtools::install_github('saezlab/decoupleRBench'), which wraps decoupleR's decouple() function. Execute the phospho-proteomics benchmark with 'Rscript R/process/run_php_bench.R', which runs the decoupleR methods (AUCell, fast GSEA, GSVA, ORA, ULM, VIPER, multivariate linear model, etc., plus the consensus score) on the Hernandez-Armenta et al. 2017 phospho-proteomics perturbation collection against the kinase substrate network. decoupleRBench evaluates performance by concatenating all experiments into a response vector (whether a regulator is perturbed) and a predictor vector (regulator activities), then computing AUROC and AUPRC over threshold sweeps with a downsampling strategy that randomly samples an equal number of negative classes per permutation, because true positive classes are limited by regulators covered in the perturbation experiments. Aggregate the scores into per-method performance metrics, write them to php_bench_performance.csv, and visually compare the phospho-proteomics ranking with the transcriptomics ranking from run_rna_bench.R.

## Related tools

- **decoupleR** (R package whose decouple() wrapper infers regulator (kinase) activities from phospho-proteomics data using the ensemble of statistical methods being benchmarked) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupleRBench** (Dedicated benchmarking package that evaluates decoupleR methods on perturbation experiments, computing AUROC/AUPRC with a downsampling strategy) — https://github.com/saezlab/decoupleRBench
- **R** (Language in which the benchmark pipeline (get_data.R, run_php_bench.R) is implemented and executed) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **OmniPath** (Meta-resource of prior knowledge used by decoupleR to provide regulator-interaction networks)

## Examples

```
Rscript R/process/get_data.R ; Rscript R/process/run_php_bench.R
```

## Evaluation signals

- The phospho-proteomics benchmark reproduces the paper's sensitivity ranking in which the top three methods are consensus, multivariate linear model and ULM
- php_bench_performance.csv is produced containing per-method metrics covering all 11 decoupleR methods plus the consensus score
- AUROC and AUPRC are computed with the downsampling strategy (equal number of negative classes randomly sampled per permutation)
- Aggregated regulator activity scores span multiple experiments, concatenated into a single response/predictor vector pair per method
- The phospho-proteomics ranking is visually comparable against the transcriptomics ranking generated by run_rna_bench.R

## Limitations

- True positive classes are limited by the regulators covered in the perturbation experiments, requiring a downsampling strategy of negative classes
- The phospho-proteomics noise analysis (run_php_noise.R) can take around 2 hours to run
- The decoupleR R/Bioconductor version is deprecated in favor of the scverse/decoupler Python implementation
- Evaluation focused on sensitivity of perturbed-kinase recovery, so rankings are sensitivity-oriented rather than a full performance profile

## Evidence

- [abstract] Methods were evaluated on transcriptomic and phospho-proteomic perturbation experiments.: "Using decoupleR, we evaluated the performance of methods on transcriptomic and phospho-proteomic perturbation experiments."
- [readme] Benchmark pipeline: "In this manuscript we have built a separate R package to benchmark decoupleR, called decoupleRBench."
- [readme] Evaluation downsampling: [evidence span withheld — anchor status: external_doc]
- [readme] Evaluation procedure: [evidence span withheld — anchor status: external_doc]
- [abstract] Consensus and linear models finding: "Our findings suggest that simple linear models and the consensus score across top methods perform better than other methods at predicting perturbed regulators."
