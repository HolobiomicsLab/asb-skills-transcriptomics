---
name: simulation-based-grn-method-benchmarking
description: Use when evaluating or comparing cell-specific network inference methods
  (e.g. LIONESS with Pearson, SSN, pySCENIC) and synthetic ground truth is required
  because real single-cell data lacks a known regulatory network reference.
when_to_use_negative:
- The goal is benchmarking RNA velocity or trajectory/alignment methods rather than
  cell-specific regulatory network inference — those use dyngen's other evaluation
  protocols (velocity vectors, ABWAP alignment scores).
- PANDA is the intended method, since dyngen datasets provide no motif data and PANDA
  requires motif-tf-gene inputs.
- The analysis claims absolute performance on real biological data — the article only
  supports quantitative CSNI evaluation against synthetic ground truth.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: dyngen
  role: Simulates the 42 benchmark datasets with transcriptomics profiles and ground-truth
    cell-specific regulatory networks (14 backbones, 3 seeds each)
  repo: https://github.com/dynverse/dyngen
- name: lionessR
  role: Runs LIONESS with Pearson correlation to infer cell-specific GRNs from each
    dataset; used because PANDA is inapplicable (no motif data)
- name: dyngen_manuscript
  role: Contains analysis/usecase_network_inference scripts that reproduce the CSNI
    benchmarking experiment
  repo: https://github.com/dynverse/dyngen_manuscript
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:eedeaedeeef45aa20ac99e8a4ac39d091282a5001af755c95c9cacdc225030a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - dyngen
  - dyngen_manuscript
  - lionessR
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# simulation-based-grn-method-benchmarking

## Summary

Benchmark cell-specific gene regulatory network inference (CSNI) methods against dyngen's fully known ground-truth cellwise regulatory networks, scoring predictions per cell with mean AUROC and mean AUPR over simulated datasets (e.g. the 42 dyngen datasets from 14 backbones with three seeds each).

## When to use

Use when evaluating or comparing cell-specific network inference methods (e.g. LIONESS with Pearson, SSN, pySCENIC) and synthetic ground truth is required because real single-cell data lacks a known regulatory network reference.

## When NOT to use

- The goal is benchmarking RNA velocity or trajectory/alignment methods rather than cell-specific regulatory network inference — those use dyngen's other evaluation protocols (velocity vectors, ABWAP alignment scores).
- PANDA is the intended method, since dyngen datasets provide no motif data and PANDA requires motif-tf-gene inputs.
- The analysis claims absolute performance on real biological data — the article only supports quantitative CSNI evaluation against synthetic ground truth.

## Inputs

- dyngen-simulated single-cell transcriptomics profiles (42 datasets: 14 backbones × 3 seeds)
- dyngen cell-specific ground-truth regulatory networks per dataset
- Inferred per-cell gene regulatory networks from the CSNI method under test (e.g. LIONESS output from lionessR)

## Outputs

- Per-cell AUROC and AUPR scores for each dataset
- Mean AUROC and mean AUPR per dataset for the 42 dyngen datasets (e.g. lioness_pearson_auroc_aupr.csv with 42 rows)
- Cross-method benchmark comparison of mean AUROC/AUPR (e.g. LIONESS vs SSN vs pySCENIC)

## How to apply

Generate or load dyngen benchmark datasets (42 datasets derived from 14 predefined backbones with three different seeds), each pairing a transcriptomics profile with its cell-specific ground-truth regulatory network. For each dataset, run the CSNI method under test — for LIONESS, use the lionessR implementation with Pearson correlation as the base network method, since PANDA is inapplicable because dyngen provides no motif data. Retain the top 10,000 interactions from each per-cell inferred GRN, compare them against the cell-specific ground truth to compute per-cell AUROC and AUPR, then average across cells to obtain mean AUROC and mean AUPR per dataset. Write the 42 mean scores to a CSV (e.g. lioness_pearson_auroc_aupr.csv) and compare methods statistically across datasets. The rationale is that dyngen's ground-truth layers make quantitative CSNI assessment possible where 'a quantitative assessment of their performance is until now lacking'.

## Related tools

- **dyngen** (Simulates the 42 benchmark datasets with transcriptomics profiles and ground-truth cell-specific regulatory networks (14 backbones, 3 seeds each)) — https://github.com/dynverse/dyngen
- **lionessR** (Runs LIONESS with Pearson correlation to infer cell-specific GRNs from each dataset; used because PANDA is inapplicable (no motif data))
- **dyngen_manuscript** (Contains analysis/usecase_network_inference scripts that reproduce the CSNI benchmarking experiment) — https://github.com/dynverse/dyngen_manuscript

## Examples

```
git clone https://github.com/dynverse/dyngen_manuscript && cd dyngen_manuscript && Rscript -e 'devtools::install("package/", dependencies = TRUE)'
```

## Evaluation signals

- Output CSV (e.g. lioness_pearson_auroc_aupr.csv) contains exactly 42 rows, one per dataset.
- Every mean AUROC and mean AUPR value falls within the valid [0, 1] range.
- Per-dataset scores are averages of per-cell AUROC/AUPR computed against the cell-specific ground-truth GRN with the top 10,000 retained interactions.
- Method ranking is consistent with published results: LIONESS significantly outperforms SSN on mean AUROC and AUPR, while pySCENIC significantly outperforms LIONESS.
- The experiment is reproducible from the dyngen_manuscript repository's analysis/usecase_network_inference folder.

## Limitations

- PANDA cannot be benchmarked this way because dyngen provides no motif data, so Pearson correlation is used as LIONESS's underlying method instead.
- LIONESS only achieves a higher AUROC than competitors on a few datasets; overall it is outperformed by pySCENIC.
- dyngen assumes cells are well-mixed and independent from each other, and no software version numbers are reported for the evaluated tools.
- Real reference single-cell RNA-seq data used to emulate profiling protocols comes from a repository that is not named.

## Evidence

- [results] For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells: "For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells"
- [results] Quantitative assessment of CSNI method performance was lacking until now: "a quantitative assessment of their performance is until now lacking"
- [abstract] dyngen is more flexible than current single-cell simulation engines, and allows better method development and benchmarking: "dyngen is more flexible than current single-cell simulation engines, and allows better method development and benchmarking"
- [readme] Manuscript repo contains all code necessary to reproduce the CSNI analysis: [evidence span withheld — anchor status: external_doc]
