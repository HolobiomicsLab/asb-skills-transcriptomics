---
name: boosted-tree-ensemble-hyperparameter-tuning
description: Use when inferring a differential gene regulatory network from two condition-specific
  gene expression matrices with BoostDiff and you must set ensemble size, regulator
  pool size, subsample count, and AdaBoost loss.
when_to_use_negative:
- The dataset is very large, e.g. the Crohn's disease dataset (GSE126124), which the
  authors note requires a high-performance computing cluster — reduce scope or move
  to HPC before tuning parameters locally
- A condition has fewer than the minimum 15 samples per condition required by BoostDiff
  (the README recommends at least 30 samples per condition)
- The goal is differential expression testing rather than differential network inference
  — BoostDiff is a tree-based network inference method, not a DE analysis tool
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: BoostDiff
  role: The boosted differential tree package whose run() parameters (n_estimators,
    n_features, n_subsamples, loss, normalize, regulators) are tuned by this skill
  repo: https://github.com/scibiome/boostdiff_inference
- name: conda
  role: Creates the bdenv environment from environment.yml in which BoostDiff is installed
    and parameterized
- name: pip
  role: Installs the cloned boostdiff_inference package inside the activated conda
    environment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:9ed09fed13fe9e142560595344293b46db9d5f326b0ec8294772cd36e08493fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/boostdiff_inference
  tools:
  - BoostDiff
  - GEOquery
  - Python
  - conda
  - gseapy
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# boosted-tree-ensemble-hyperparameter-tuning

## Summary

Configuration of BoostDiff's modified AdaBoost ensemble of differential trees — selecting n_estimators, n_features (regulator count), n_subsamples, loss, and normalization — so the differential-network inference is predictive without overfitting on transcriptomics data.

## When to use

Use when inferring a differential gene regulatory network from two condition-specific gene expression matrices with BoostDiff and you must set ensemble size, regulator pool size, subsample count, and AdaBoost loss.

## When NOT to use

- The dataset is very large, e.g. the Crohn's disease dataset (GSE126124), which the authors note requires a high-performance computing cluster — reduce scope or move to HPC before tuning parameters locally
- A condition has fewer than the minimum 15 samples per condition required by BoostDiff (the README recommends at least 30 samples per condition)
- The goal is differential expression testing rather than differential network inference — BoostDiff is a tree-based network inference method, not a DE analysis tool

## Inputs

- Disease-condition gene expression matrix (tab-separated; rows = genes, columns = samples, first column named 'Gene')
- Control-condition gene expression matrix (tab-separated; same feature set as the disease matrix)
- Parameter choices: n_estimators, n_features, n_subsamples, loss, keyword, n_processes

## Outputs

- Raw differential network file (boostdiff_network_<keyword>.txt) per run
- Trained AdaBoost ensemble of differential trees per target condition ('disease' and 'control' output folders)

## How to apply

Import BoostDiff from boostdiff.main_boostdiff and call run(file_disease, file_control, output_folder, n_estimators, n_features, n_subsamples, keyword, n_processes). Set n_estimators=50 for real transcriptomics datasets — the authors recommend a relatively low number of base learners to avoid overfitting — and n_features=2500 for the regulator (predictor gene) pool. Choose the AdaBoost loss from {'square','exponential','linear'} (the COVID-19 GSE156063 workflow uses loss='exponential') and rely on the default unit_variance normalization (normalize=True). Use n_subsamples=20 bootstrapped samples per condition (minimum 15 samples per condition is required), set regulators='all' so every gene can be a predictor, and raise n_processes (≥10 on the ~15,000-gene, ~100-samples-per-condition COVID-19 dataset) for parallelism. Run BoostDiff twice — once with the disease condition as target and once with the control as target — writing outputs to separate 'disease' and 'control' subfolders.

## Related tools

- **BoostDiff** (The boosted differential tree package whose run() parameters (n_estimators, n_features, n_subsamples, loss, normalize, regulators) are tuned by this skill) — https://github.com/scibiome/boostdiff_inference
- **conda** (Creates the bdenv environment from environment.yml in which BoostDiff is installed and parameterized)
- **pip** (Installs the cloned boostdiff_inference package inside the activated conda environment)

## Examples

```
n_estimators = 50; n_features = 2500; n_subsamples = 20; normalize = True; loss = 'exponential'; model.run(file_etha, file_ctrl, output_folder, n_estimators, n_features, n_subsamples, n_processes=n_processes)
```

## Evaluation signals

- Two runs complete: Run 1 with the disease condition as target (control baseline) and Run 2 with the control as target, each writing to its own output subfolder
- Output file boostdiff_network_<keyword>.txt exists in each output folder for the chosen keyword
- n_estimators is kept low (50) relative to dataset size so the 50 adaptively boosted differential trees do not overfit; regulator pool capped via n_features=2500
- Both input matrices share the identical gene feature set with the first column named 'Gene' before run() is called
- Runtime is tractable on the target machine (the ~15,000-gene, ~100-sample-per-condition COVID-19 dataset used n_processes≥10)

## Limitations

- BoostDiff was tested on Linux only
- Large datasets (e.g. Crohn's disease) need a high-performance computing cluster
- Very large ensemble sizes risk overfitting; the authors recommend keeping n_estimators low for real transcriptomics data
- At least 15 samples per condition are required per ensemble (minimum); ~30 per condition is the ideal input

## Evidence

- [readme] we recommend to set a relatively low number of base learners, e.g. set `n_estimators=50`, so that 50 adaptively boosted differential trees will be built to avoid overfitting.: "we recommend to set a relatively low number of base learners, e.g. set `n_estimators=50`, so that 50 adaptively boosted differential trees will be built to avoid overfitting."
- [readme] n_processes = 4 n_estimators = 50 n_features = 2500 n_subsamples = 20 keyword = "saltvssmm" normalize = True loss = "exponential": "n_estimators = 50
n_features = 2500
n_subsamples = 20"
- [methods] model.run(file_etha, file_ctrl, output_folder, n_estimators, n_features, n_subsamples, n_processes=n_processes: "model.run(file_etha, file_ctrl, output_folder, n_estimators, n_features, n_subsamples, n_processes=n_processes"
- [abstract] BoostDiff builds an adaptively boosted (AdaBoost) ensemble of differential trees with respect to a target condition.: "BoostDiff builds an adaptively boosted (AdaBoost) ensemble of differential trees with respect to a target condition."
