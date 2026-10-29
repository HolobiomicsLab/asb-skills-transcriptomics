---
name: aucroc-aupr-performance-evaluation
description: Use when evaluating a CSNI method (e.g. LIONESS, SSN, pySCENIC) against
  dyngen's cell-specific ground-truth regulatory networks and you need comparable
  AUROC/AUPR scores per dataset.
when_to_use_negative:
- The task is bulk/aggregate network inference — this protocol specifically scores
  per-cell predictions against cell-specific ground truth
- The chosen underlying inference method requires motif data (e.g. PANDA) — dyngen
  provides no motif data, so Pearson-based LIONESS was used instead
- You need absolute method rankings on real biological data — this evaluation is only
  valid against dyngen's simulated ground truth
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: dyngen
  role: Provides the 42 simulated benchmark datasets with transcriptomics profiles
    and ground-truth cell-specific regulatory networks used for evaluation
  repo: https://github.com/dynverse/dyngen
- name: lionessR
  role: Runs LIONESS cell-specific network inference with Pearson correlation as the
    underlying network inference method
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:e8fd56344b3cd3d365ee137a78215fe02c5fb2d55e9ad7bd3e132bf7aae33db6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - dyngen
  - lionessR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# aucroc-aupr-performance-evaluation

## Summary

Benchmarks a cell-specific network inference (CSNI) method on dyngen-simulated datasets by computing per-cell AUROC and AUPR against ground-truth cell-specific regulatory networks, then averaging to one mean AUROC and mean AUPR per dataset (42 datasets from 14 backbones × 3 seeds).

## When to use

Use when evaluating a CSNI method (e.g. LIONESS, SSN, pySCENIC) against dyngen's cell-specific ground-truth regulatory networks and you need comparable AUROC/AUPR scores per dataset.

## When NOT to use

- The task is bulk/aggregate network inference — this protocol specifically scores per-cell predictions against cell-specific ground truth
- The chosen underlying inference method requires motif data (e.g. PANDA) — dyngen provides no motif data, so Pearson-based LIONESS was used instead
- You need absolute method rankings on real biological data — this evaluation is only valid against dyngen's simulated ground truth

## Inputs

- dyngen benchmark dataset transcriptomics profile (expression matrix, cells × genes)
- dyngen cell-specific ground-truth regulatory network per dataset
- Inferred cell-specific GRN predictions (e.g. from lionessR with Pearson)

## Outputs

- Per-cell AUROC and AUPR scores
- Per-dataset mean AUROC and mean AUPR (42 rows, e.g. lioness_pearson_auroc_aupr.csv)

## How to apply

Load each dyngen benchmark dataset, which contains a transcriptomics profile and its cell-specific ground-truth regulatory network. Run the CSNI method per dataset — e.g. LIONESS via lionessR with Pearson correlation, since PANDA is inapplicable because dyngen provides no motif data. Retain the top 10,000 interactions per cell's inferred GRN, then compute AUROC and AUPR of these predicted interactions against the cell-specific ground truth per cell. Average the per-cell scores to obtain the mean AUROC and mean AUPR for each dataset (the dyngen paper's protocol computes 'the mean AUROC and AUPR scores of the individual cells'). Write the scores to a CSV such as lioness_pearson_auroc_aupr.csv for downstream comparison with other CSNI methods.

## Related tools

- **dyngen** (Provides the 42 simulated benchmark datasets with transcriptomics profiles and ground-truth cell-specific regulatory networks used for evaluation) — https://github.com/dynverse/dyngen
- **lionessR** (Runs LIONESS cell-specific network inference with Pearson correlation as the underlying network inference method)

## Examples

```
write.csv(mean_scores, "lioness_pearson_auroc_aupr.csv")
```

## Evaluation signals

- Output CSV (e.g. lioness_pearson_auroc_aupr.csv) contains exactly 42 rows, one per dyngen dataset
- All mean AUROC and mean AUPR values fall within the valid [0, 1] range
- Both mean AUROC and mean AUPR columns are present and non-missing for every dataset
- Per-cell scores were computed on the retained top 10,000 predicted interactions before averaging
- Comparison against baselines reproduces the expected ordering: LIONESS > SSN on mean AUROC and AUPR, but pySCENIC > LIONESS

## Limitations

- LIONESS only achieves a higher AUROC on a few datasets; pySCENIC significantly outperforms LIONESS overall
- PANDA-backed LIONESS cannot be applied because dyngen provides no motif data
- dyngen assumes cells are well-mixed and independent from each other, which constrains what the ground truth represents

## Evidence

- [results] For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells: "For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells"
- [methods] we used the same normalised data as provided by dyngen, with no extra cell or feature filtering: "we used the same normalised data as provided by dyngen, with no extra cell or feature filtering"
- [readme] Evaluating the performance of cell-specific network inference algorithms.: [evidence span withheld — anchor status: external_doc]
