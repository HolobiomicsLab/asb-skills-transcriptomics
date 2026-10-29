---
name: benchmark-dataset-handling
description: Reference for when you need the standard scSHARP evaluation datasets
  (two Splatter simulations, mouse testis, human PBMC) to reproduce or compare cell-type
  annotation results.
when_to_use_negative:
- Your data is a novel tissue or simulation outside the four scSHARP benchmarks and
  you only need a general annotation tool — the notebooks and expected fractions are
  dataset-specific
- Your input is already normalized or feature-selected — the README specifies raw
  counts ('Please input raw counts')
- You need species or reference-annotation flexibility beyond the packaged SCINA,
  ScType, scSorter, SingleR, and scPred tool suite
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scSHARP
  role: Main pipeline whose repository distributes the benchmark datasets and per-dataset
    results notebooks (simulation, testis, PBMC, kNN experiments)
  repo: https://github.com/lewinsohndp/scSHARP
- name: R4scSHARP
  role: R package that runs the five component classification tools (SCINA, ScType,
    scSorter, SingleR, scPred) on each benchmark expression matrix
  repo: https://github.com/W-Holtz/R4scSHARP
- name: scSHARP_tool
  role: Python package implementing the GCN label-propagation tool applied to the
    benchmark predictions
  repo: https://github.com/mperozek11/scSHARP_tool
- name: conda
  role: Environment manager used to create the Python 3.9 environment in which benchmark
    notebooks are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:f0a5ae75cdc4391addfeb50e71ad1bb602d9435ef12afe525bb0fc72fa6eef7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lewinsohndp/scSHARP
  tools:
  - Python
  - R
  - R4scSHARP
  - conda
  - scSHARP
  - scSHARP_tool
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# benchmark-dataset-handling

## Summary

Obtains and runs scSHARP's four standard benchmark datasets — Splatter simulations at de.facScale 0.7 and 0.8, the mouse testis dataset (Jung et al. 2019), and the human PBMC dataset (Zheng et al. 2017) — from the scSHARP repository distribution so annotation methods are evaluated on a consistent basis.

## When to use

Reference for when you need the standard scSHARP evaluation datasets (two Splatter simulations, mouse testis, human PBMC) to reproduce or compare cell-type annotation results.

## When NOT to use

- Your data is a novel tissue or simulation outside the four scSHARP benchmarks and you only need a general annotation tool — the notebooks and expected fractions are dataset-specific
- Your input is already normalized or feature-selected — the README specifies raw counts ('Please input raw counts')
- You need species or reference-annotation flexibility beyond the packaged SCINA, ScType, scSorter, SingleR, and scPred tool suite

## Inputs

- Splatter simulated scRNA-seq count matrices at de.facScale 0.7 and 0.8 (from the scSHARP repository distribution)
- Mouse testis scRNA-seq dataset (Jung et al. 2019)
- Human PBMC scRNA-seq dataset (Zheng et al. 2017)
- Raw gene expression count matrices (not normalized)

## Outputs

- Per-dataset per-cell cell-type label predictions from the five component tools
- Per-dataset summary CSV of counts and fractions (e.g. unconfident cells 156/15.6%, 85/8.5%, 46/6.6%, 2310/3.0%)
- Notebook-generated results for each benchmark (simulation, testis, PBMC)

## How to apply

Retrieve the four benchmark datasets from the scSHARP repository distribution: Splatter simulations generated at de.facScale 0.7 and 0.8, the mouse testis dataset of Jung et al. 2019, and the human PBMC dataset of Zheng et al. 2017. Each dataset ships with a dedicated results notebook in the lewinsohndp/scSHARP repository (simulation_test_results.ipynb for both simulations, jung_results.ipynb for testis, pbmc_test_results.ipynb for PBMC), so run the corresponding notebook for the dataset being evaluated. Input the raw counts matrix to the pipeline, as the README explicitly instructs 'Please input raw counts.' For datasets that require the five R component tools (SCINA, ScType, scSorter, SingleR, scPred), first install R4scSHARP via devtools::install_github('W-Holtz/R4scSHARP') and its suggested packages (SingleR via BiocManager, scSorter/SCINA via install.packages, scPred via devtools with harmony). Evaluate outputs against the published reference numbers, e.g. 15.6% unconfident cells in the de.facScale 0.7 simulation versus 3.0% (2310 cells) in PBMC, to confirm the dataset and threshold handling are correct.

## Related tools

- **scSHARP** (Main pipeline whose repository distributes the benchmark datasets and per-dataset results notebooks (simulation, testis, PBMC, kNN experiments)) — https://github.com/lewinsohndp/scSHARP
- **R4scSHARP** (R package that runs the five component classification tools (SCINA, ScType, scSorter, SingleR, scPred) on each benchmark expression matrix) — https://github.com/W-Holtz/R4scSHARP
- **scSHARP_tool** (Python package implementing the GCN label-propagation tool applied to the benchmark predictions) — https://github.com/mperozek11/scSHARP_tool
- **conda** (Environment manager used to create the Python 3.9 environment in which benchmark notebooks are executed)

## Examples

```
conda create -n scsharp python=3.9; conda activate scsharp; ./anaconda/envs/scsharp/bin/pip install scSHARP
```

## Evaluation signals

- Unconfident-cell fractions under the 0.51 majority-vote threshold match published values: 15.6% (de.facScale 0.7 sim), 8.5% (0.8 sim), 6.6% (46 cells, testis), 3.0% (2310 cells, PBMC)
- The correct notebook is used per dataset: simulation_test_results.ipynb for simulations, jung_results.ipynb for testis, pbmc_test_results.ipynb for PBMC
- Counts of unconfidently labeled cells equal fraction × total cells (e.g. 46 testis cells out of ~700, 2310 PBMC cells out of ~77000)
- All five component-tool prediction columns are present in the aggregated prediction file returned by run_r4scsharp
- Input matrices are raw counts; the pipeline was not run on pre-normalized data

## Limitations

- The four datasets are fixed benchmarks shipped with the repository; extending to other datasets requires new simulation or download work outside these notebooks
- R4scSHARP documentation was incomplete at publication time ('More documentation is coming!'), and the package was GitHub-only initially with plans for CRAN
- Additional PyTorch Geometric dependencies such as torch-cluster may need manual installation depending on the system

## Evidence

- [readme] simulation_test_results.ipynb - Main results for both simulated data sets.: [evidence span withheld — anchor status: external_doc]
- [results] Obtain the four benchmark datasets (Splatter simulations at de.facScale 0.7 and 0.8, mouse testis Jung et al. 2019, human PBMC Zheng et al. 2017) from the scSHARP repository distribution.: [evidence span withheld — anchor status: not_found]
- [readme] the main function, run_r4scsharp, is used to run up to five cell classification tools on a RNA gene expression matrix: [evidence span withheld — anchor status: external_doc]
- [readme] install_github("W-Holtz/R4scSHARP"): [evidence span withheld — anchor status: external_doc]
