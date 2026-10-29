---
name: grn-benchmark-evaluation-auroc-aupr
description: Use when benchmarking cell-specific network inference (CSNI) methods
  such as pySCENIC, LIONESS, or SSN on simulated single-cell data where the ground-truth
  cellwise regulatory network is known.
when_to_use_negative:
- The data are real single-cell measurements without ground-truth cellwise regulatory
  networks — quantitative AUROC/AUPR evaluation requires dyngen's known ground truth
- The method requires motif enrichment information for regulon filtering (e.g. full
  SCENIC motif step) — dyngen does not generate motif data, so that step must be skipped
- Evaluating trajectory inference, trajectory alignment, or RNA velocity rather than
  cell-specific regulatory network inference
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: dyngen
  role: Simulator generating the 42 benchmark datasets with ground-truth cellwise
    regulatory networks for evaluation
  repo: https://github.com/dynverse/dyngen
- name: pySCENIC
  role: CSNI method under evaluation; infers coexpression modules with arboreto and
    reassembles a cell-specific GRN from AUCell scores
- name: arboreto
  role: Infers coexpression modules using stochastic gradient boosting machines (step
    1 of the pySCENIC pipeline)
- name: AUCell
  role: Computes per-cell module activity scores that are combined into the cell-specific
    GRN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:cd44d2d4b90ee90e155dce1a64c278f40d08dc35ab540739ada9445a9eacba85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - AUCell
  - arboreto
  - dyngen
  - pySCENIC
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# GRN benchmark evaluation with AUROC and AUPR

## Summary

Evaluate cell-specific gene regulatory network (GRN) inference methods by comparing predicted (cell, regulator, target, score) tuples against dyngen's ground-truth cellwise regulatory networks using AUROC and AUPR scores, averaged across cells and datasets.

## When to use

Use when benchmarking cell-specific network inference (CSNI) methods such as pySCENIC, LIONESS, or SSN on simulated single-cell data where the ground-truth cellwise regulatory network is known.

## When NOT to use

- The data are real single-cell measurements without ground-truth cellwise regulatory networks — quantitative AUROC/AUPR evaluation requires dyngen's known ground truth
- The method requires motif enrichment information for regulon filtering (e.g. full SCENIC motif step) — dyngen does not generate motif data, so that step must be skipped
- Evaluating trajectory inference, trajectory alignment, or RNA velocity rather than cell-specific regulatory network inference

## Inputs

- 42 dyngen datasets (14 predefined backbones x 3 seeds) with transcriptomics profiles
- Ground-truth cell-specific regulatory network per dataset (regulatory effect of each regulator on each target)
- Predicted cell-specific GRN as (cell, regulator, target, score) tuples from the method under test

## Outputs

- Per-cell AUROC and AUPR scores against the ground-truth cell-specific GRN
- Mean AUROC and mean AUPR per dataset (bounded in [0, 1])
- Aggregated mean AUROC and mean AUPR per method across the 42 dyngen datasets for method ranking

## How to apply

Load each dyngen-simulated dataset together with its ground-truth cell-specific regulatory network, computed from the regulatory effect of each regulator on its target gene. Run the candidate CSNI method on the transcriptomics profiles (for pySCENIC: infer coexpression modules with arboreto's gradient boosting, keep the top 10 regulators per target, skip motif filtering since dyngen does not generate motif data, compute AUCell module activities, and reassemble a cell-specific GRN of (cell, regulator, target, score) tuples). Retain the top 10,000 interactions per cell and compute AUROC and AUPR against the ground-truth cell-specific GRN. Average the per-cell scores within each dataset, and aggregate mean AUROC and mean AUPR across all 42 datasets (14 backbones x 3 seeds) to rank methods; scores are bounded in [0, 1].

## Related tools

- **dyngen** (Simulator generating the 42 benchmark datasets with ground-truth cellwise regulatory networks for evaluation) — https://github.com/dynverse/dyngen
- **pySCENIC** (CSNI method under evaluation; infers coexpression modules with arboreto and reassembles a cell-specific GRN from AUCell scores)
- **arboreto** (Infers coexpression modules using stochastic gradient boosting machines (step 1 of the pySCENIC pipeline))
- **AUCell** (Computes per-cell module activity scores that are combined into the cell-specific GRN)

## Examples

```
Rscript analysis/usecase_network_inference/01_run_methods.R  # evaluate pySCENIC/LIONESS/SSN on 42 dyngen datasets
```

## Evaluation signals

- Each dataset yields a mean AUROC and mean AUPR value bounded in [0, 1]
- Scores are comparable to those reported in the paper's CSNI benchmark (pySCENIC > LIONESS > SSN on mean AUROC and AUPR)
- Predicted GRN per cell is restricted to the top 10,000 interactions before evaluation
- Aggregation covers all 42 datasets (14 backbones x 3 seeds) before drawing conclusions
- For pySCENIC, exactly the top 10 regulators per target gene are grouped into modules, and no motif-filtering step is applied

## Limitations

- dyngen assumes cells are well-mixed and independent from each other, so simulated ground truth may not fully reflect real tissue biology
- dyngen does not generate motif data, so motif-analysis filtering steps of methods like SCENIC must be skipped or approximated
- No software version numbers are reported for the tools used in the benchmark
- Conclusions are specific to dyngen-simulated data; performance on real data with unknown ground truth cannot be verified quantitatively

## Evidence

- [results] Mean AUROC/AUPR evaluation of CSNI: "For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells"
- [results] Gap in CSNI benchmarking: "a quantitative assessment of their performance is until now lacking"
- [intro] dyngen provides cellwise regulatory network ground truth: [evidence span withheld — anchor status: not_found]
- [readme] Reproduction code location for the CSNI use case: [evidence span withheld — anchor status: not_found]
- [intro] Manuscript source availability: "All source code for reproducing the results in this manuscript are available on GitHub"
