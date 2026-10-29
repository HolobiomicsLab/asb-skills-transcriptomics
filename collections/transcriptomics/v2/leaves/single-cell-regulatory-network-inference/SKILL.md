---
name: single-cell-regulatory-network-inference
description: Use when evaluating how well CSNI methods (e.g. pySCENIC, LIONESS, SSN)
  recover cell-specific regulatory interactions, and simulated single-cell data with
  known cellwise ground-truth GRNs is available for quantitative AUROC/AUPR evaluation.
when_to_use_negative:
- The data lack ground-truth cellwise regulatory interactions (e.g. dyngen does not
  generate motif data, so motif-analysis filtering steps cannot be applied)
- The analysis goal is bulk or population-level GRN inference rather than per-cell
  (cell-specific) regulatory networks
- You need to evaluate trajectory, RNA velocity, or trajectory alignment methods —
  these use dyngen's other ground-truth layers and different evaluation metrics such
  as ABWAP
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: dyngen
  role: Generates the 42 simulated datasets with ground-truth cellwise regulatory
    networks used as the benchmark reference
  repo: https://github.com/dynverse/dyngen
- name: pySCENIC
  role: 'CSNI method under evaluation: coexpression module inference, top-10 regulator
    selection, AUCell module activity scoring, recombination into cell-specific GRN
    tuples'
- name: arboreto
  role: 'Step 1 of the pySCENIC pipeline: infers coexpression modules using stochastic
    gradient boosting machines'
- name: AUCell
  role: Computes per-cell module activity scores that are combined back into cell-specific
    GRN (cell, regulator, target, score) tuples
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:5f8f34972976bfe09a760da1ac698f742dbbf50f40a1507d04073dc2341ef211
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - AUCell
  - arboreto
  - dyngen
  - pySCENIC
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# single-cell-regulatory-network-inference

## Summary

Benchmark cell-specific network inference (CSNI) methods by reconstructing per-cell gene regulatory networks from single-cell transcriptomics and scoring them against dyngen's ground-truth cellwise regulatory networks using mean AUROC and AUPR over 42 simulated datasets (14 backbones x 3 seeds).

## When to use

Use when evaluating how well CSNI methods (e.g. pySCENIC, LIONESS, SSN) recover cell-specific regulatory interactions, and simulated single-cell data with known cellwise ground-truth GRNs is available for quantitative AUROC/AUPR evaluation.

## When NOT to use

- The data lack ground-truth cellwise regulatory interactions (e.g. dyngen does not generate motif data, so motif-analysis filtering steps cannot be applied)
- The analysis goal is bulk or population-level GRN inference rather than per-cell (cell-specific) regulatory networks
- You need to evaluate trajectory, RNA velocity, or trajectory alignment methods — these use dyngen's other ground-truth layers and different evaluation metrics such as ABWAP

## Inputs

- 42 dyngen-simulated single-cell transcriptomics datasets (14 backbones x 3 seeds)
- dyngen ground-truth cell-specific regulatory networks (regulator, target, regulatory effect per cell)
- CSNI method outputs (pySCENIC, LIONESS, SSN) as per-cell ranked interaction lists

## Outputs

- Per-cell AUROC and AUPR scores for each reconstructed cell-specific GRN
- Mean AUROC and mean AUPR per dataset per CSNI method, bounded in [0, 1]
- Cross-dataset comparison of CSNI method performance (e.g. pySCENIC > LIONESS > SSN)

## How to apply

Load the 42 dyngen datasets (14 predefined backbones x 3 seeds), each with transcriptomics profiles and a ground-truth cell-specific regulatory network derived from the regulatory effect of each regulator on its target. Run the pySCENIC pipeline: first infer coexpression modules with arboreto's stochastic gradient boosting (GRNBoost-style classical network inference), then select the top 10 regulators of every target gene and group them into modules of one regulator and its targets; skip motif filtering because dyngen does not generate motif data. Compute per-cell AUCell activity scores for each module and recombine modules and scores into a cell-specific GRN of (cell, regulator, target, score) tuples. Retain the top 10,000 interactions per cell and compute AUROC and AUPR against the ground-truth cellwise GRN, averaging scores across all cells per dataset; compare methods (e.g. pySCENIC vs LIONESS vs SSN) on mean AUROC and mean AUPR across the 42 datasets.

## Related tools

- **dyngen** (Generates the 42 simulated datasets with ground-truth cellwise regulatory networks used as the benchmark reference) — https://github.com/dynverse/dyngen
- **pySCENIC** (CSNI method under evaluation: coexpression module inference, top-10 regulator selection, AUCell module activity scoring, recombination into cell-specific GRN tuples)
- **arboreto** (Step 1 of the pySCENIC pipeline: infers coexpression modules using stochastic gradient boosting machines)
- **AUCell** (Computes per-cell module activity scores that are combined back into cell-specific GRN (cell, regulator, target, score) tuples)

## Examples

```
git clone https://github.com/dynverse/dyngen_manuscript; cd dyngen_manuscript; Rscript analysis/usecase_network_inference/run.R
```

## Evaluation signals

- Each dataset yields a mean AUROC and mean AUPR value bounded in [0, 1] for each CSNI method condition
- Scores are comparable to the CSNI benchmark values reported in the dyngen paper
- Averaged over the 42 datasets, pySCENIC should significantly outperform LIONESS, and LIONESS should significantly outperform SSN on mean AUROC and mean AUPR
- Per-cell interaction lists are capped at the top 10,000 interactions before AUROC/AUPR computation
- The ground-truth comparison is made against dyngen's cellwise regulatory network computed from each regulator's regulatory effect on its target

## Limitations

- dyngen does not generate motif data, so pySCENIC's motif-analysis filtering step must be skipped, changing the method relative to its standard pipeline
- dyngen assumes cells are well-mixed and independent from each other, which may not reflect real tissues
- Evaluation depends on dyngen's simulation assumptions; results reflect synthetic-data performance and use the same normalised data provided by dyngen with no extra cell or feature filtering

## Evidence

- [results] For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells: "For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells"
- [results] Quantitative assessment of CSNI method performance was lacking: "a quantitative assessment of their performance is until now lacking"
- [intro] dyngen benchmarking scope includes CSNI: "dyngen can cater to a wide range of benchmarking applications, including trajectory inference, trajectory alignment"
- [readme] Network inference use case script location: [evidence span withheld — anchor status: not_found]
