---
name: pyscenic-pipeline-execution
description: Use when benchmarking cell-specific gene regulatory network inference
  methods (pySCENIC, LIONESS, SSN) on simulated single-cell data that has cellwise
  ground-truth regulatory networks, e.g. the 42 dyngen datasets.
when_to_use_negative:
- The data are real single-cell RNA-seq experiments without a known cellwise ground-truth
  regulatory network — AUROC/AUPR against ground truth cannot be computed
- Motif information is available and motif-based regulon filtering is desired — the
  skipped motif-analysis step would otherwise apply
- The goal is bulk (not cell-specific) gene regulatory network inference — AUCell-based
  per-cell module scoring is unnecessary
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: dyngen
  role: Generates the 42 simulated datasets (14 backbones x 3 seeds) with ground-truth
    cellwise regulatory networks used as the benchmark input
  repo: https://github.com/dynverse/dyngen
- name: pySCENIC
  role: 'CSNI method under evaluation: infers coexpression modules, selects top 10
    regulators per target, and builds cell-specific networks via AUCell'
- name: arboreto
  role: Performs step 1 of the pySCENIC pipeline — coexpression module inference using
    stochastic gradient boosting machines
- name: AUCell
  role: Computes per-cell module activity scores that are combined into the cell-specific
    GRN of (cell, regulator, target, score) tuples
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:2bd118b37663d2c942a62b3b040a6d5071294529642ec668b27c5ac6df2173ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - AUCell
  - arboreto
  - dyngen
  - pySCENIC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pyscenic-pipeline-execution

## Summary

Execute the pySCENIC cell-specific network inference (CSNI) pipeline on dyngen-simulated single-cell datasets and score it against ground-truth cellwise regulatory networks using mean AUROC and AUPR. This is the workflow used in the dyngen manuscript's network-inference use case to quantitatively benchmark CSNI methods.

## When to use

Use when benchmarking cell-specific gene regulatory network inference methods (pySCENIC, LIONESS, SSN) on simulated single-cell data that has cellwise ground-truth regulatory networks, e.g. the 42 dyngen datasets.

## When NOT to use

- The data are real single-cell RNA-seq experiments without a known cellwise ground-truth regulatory network — AUROC/AUPR against ground truth cannot be computed
- Motif information is available and motif-based regulon filtering is desired — the skipped motif-analysis step would otherwise apply
- The goal is bulk (not cell-specific) gene regulatory network inference — AUCell-based per-cell module scoring is unnecessary

## Inputs

- 42 dyngen-simulated datasets (14 predefined backbones x 3 seeds) with transcriptomics profiles
- Cell-specific ground-truth regulatory networks derived from dyngen's regulatory effect of each regulator on its target
- Dyngen-provided normalised expression matrix

## Outputs

- Per-dataset mean AUROC and mean AUPR values (each bounded in [0, 1]) for the SCENIC condition
- Cell-specific GRN as (cell, regulator, target, score) tuples, top 10,000 interactions per cell

## How to apply

Load the 42 dyngen datasets (14 predefined backbones x 3 seeds), each containing transcriptomics profiles and a cell-specific ground-truth regulatory network computed from the regulatory effect of each regulator on its target. Run step 1 of the pySCENIC pipeline: infer coexpression modules via GRNBoost-class stochastic gradient boosting machines using arboreto, then select the top 10 regulators of every target gene and group interactions into modules of one regulator and its targets. Skip the motif-analysis filtering step because dyngen does not generate motif data. Compute per-cell AUCell activity scores for each module and combine modules and scores back into a cell-specific GRN of (cell, regulator, target, score) tuples. Retain the top 10,000 interactions per cell and compute AUROC and AUPR against the ground-truth cell-specific GRN, then average across all cells per dataset, matching the parameters to the respective tutorial vignettes and using the same normalised data provided by dyngen with no extra cell or feature filtering.

## Related tools

- **dyngen** (Generates the 42 simulated datasets (14 backbones x 3 seeds) with ground-truth cellwise regulatory networks used as the benchmark input) — https://github.com/dynverse/dyngen
- **pySCENIC** (CSNI method under evaluation: infers coexpression modules, selects top 10 regulators per target, and builds cell-specific networks via AUCell)
- **arboreto** (Performs step 1 of the pySCENIC pipeline — coexpression module inference using stochastic gradient boosting machines)
- **AUCell** (Computes per-cell module activity scores that are combined into the cell-specific GRN of (cell, regulator, target, score) tuples)

## Examples

```
git clone https://github.com/dynverse/dyngen_manuscript && cd dyngen_manuscript && Rscript -e "devtools::install('package/', dependencies = TRUE)" && Rscript analysis/usecase_network_inference/run.R
```

## Evaluation signals

- Each of the 42 datasets yields a mean AUROC and mean AUPR value bounded in [0, 1] for the SCENIC condition
- Scores are comparable to the CSNI benchmark values reported in the dyngen paper
- pySCENIC's mean AUROC/AUPR should significantly exceed LIONESS's, and LIONESS's should exceed SSN's, on the dyngen benchmark
- Per-cell outputs contain exactly (cell, regulator, target, score) tuples with at most the top 10,000 interactions retained per cell
- Module grouping results in one regulator per module with that regulator's top 10 target genes

## Limitations

- The motif-analysis filtering step of the pySCENIC pipeline must be skipped because dyngen does not generate motif data, which may affect comparability to pySCENIC runs on real data
- dyngen assumes cells are well-mixed and independent from each other, which constrains the biological realism of the benchmark
- No software version numbers were reported for any of the tools used, so exact reproducibility of versioned behaviour is not guaranteed

## Evidence

- [results] For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells: "For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells"
- [intro] dyngen can cater to a wide range of benchmarking applications, including trajectory inference, trajectory alignment, and trajectory differential expression: "dyngen can cater to a wide range of benchmarking applications, including trajectory inference, trajectory alignment, and trajectory differential expression"
- [readme] analysis/usecase_network_inference: Evaluating the performance of cell-specific network inference algorithms.: [evidence span withheld — anchor status: not_found]
- [results] a quantitative assessment of their performance is until now lacking: "a quantitative assessment of their performance is until now lacking"
- [methods] we used the same normalised data as provided by dyngen, with no extra cell or feature filtering, but otherwise matched the parameters to their respective tutorial vignettes: "we used the same normalised data as provided by dyngen, with no extra cell or feature filtering, but otherwise matched the parameters to their respective tutorial vignettes"
