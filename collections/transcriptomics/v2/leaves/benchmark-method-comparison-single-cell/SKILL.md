---
name: benchmark-method-comparison-single-cell
description: Use when a new single-cell fate-mapping method must be validated against
  a published baseline (e.g., CellRank) by checking whether known terminal states,
  such as hormone-producing α, β, and ε cells, are consistently recovered.
when_to_use_negative:
- The dataset is a simulation-only ground truth (e.g., Dyngen) where no published
  baseline terminal-state calls exist to compare against
- Only transcription-factor activities are of interest — FateCompass's linear model
  of gene regulation, not method benchmarking, is the appropriate skill
- The input adata lacks cell-type annotations or an RNA velocity field, so neither
  CellRank's velocity kernel nor FateCompass's RNA-velocity-driven mode can be run
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FateCompass
  role: Method under test; run in RNA-velocity-driven trajectory mode via the fatecompass.py
    module to estimate transition probabilities and rank terminal fates
  repo: https://github.com/sarajimenez/fatecompass
- name: CellRank
  role: Baseline method; velocity kernel + Markov-chain absorption-probability estimator
    whose terminal-state calls are compared against FateCompass
- name: Scanpy
  role: Suggested pre-processing of the scRNA-seq E15.5 dataset before running both
    methods
- name: scVelo
  role: Suggested pre-processing and source of the RNA velocity field used by both
    CellRank's velocity kernel and FateCompass's Markov chain
- name: Python
  role: Implementation language of FateCompass and the comparison workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:5b47658b8afe37134e533e93940feca557376efb679afb318e0896968b92b974
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  repo_url: https://github.com/sarajimenez/fatecompass
  tools:
  - CellRank
  - FateCompass
  - Python
  - Scanpy
  - scVelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# benchmark-method-comparison-single-cell

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a novel trajectory-inference method (FateCompass) against an established baseline (CellRank) on the same pre-processed scRNA-seq dataset, comparing terminal-fate calls and fate probabilities across kernels and parameters.

## When to use

Use when a new single-cell fate-mapping method must be validated against a published baseline (e.g., CellRank) by checking whether known terminal states, such as hormone-producing α, β, and ε cells, are consistently recovered.

## When NOT to use

- The dataset is a simulation-only ground truth (e.g., Dyngen) where no published baseline terminal-state calls exist to compare against
- Only transcription-factor activities are of interest — FateCompass's linear model of gene regulation, not method benchmarking, is the appropriate skill
- The input adata lacks cell-type annotations or an RNA velocity field, so neither CellRank's velocity kernel nor FateCompass's RNA-velocity-driven mode can be run

## Inputs

- Pre-processed E15.5 mouse pancreas anndata object with 'clusters' cell-type annotations (including Ductal and Ngn3-low EP populations) and scVelo RNA velocity field
- RNA velocity kernel options for CellRank (velocity kernel on the KNN graph)
- FateCompass run configuration (RNA-velocity-driven trajectory mode, parameters and kernels to sweep)

## Outputs

- cellrank_fate_probabilities.csv with one row per cell population and columns: CellRank fate probability, CellRank terminal-state call, FateCompass terminal-fate ranking
- Aggregated per-cell-type fate probabilities and terminal-state calls from both methods

## How to apply

Load the pre-processed E15.5 mouse pancreas anndata object (Scanpy/scVelo pre-processing, cell-type annotations including Ductal and Ngn3-low EP clusters) and its scVelo-derived RNA velocity field. Build a directed velocity kernel with CellRank on the KNN graph, estimate absorption probabilities with CellRank's Markov-chain estimator, and record which populations are called terminal. Run FateCompass (fatecompass.py) in RNA-velocity-driven trajectory mode on the same adata, which directs the edges of the discrete Markov chain using RNA velocity, to estimate transition probabilities and rank likely terminal fates. Aggregate per-cell-type fate probabilities and terminal-state calls from both methods into one table and write it to cellrank_fate_probabilities.csv. The comparison is judged correct when both methods agree on β as a terminal fate, and the baseline's known failure mode (α cells not identified as terminal by CellRank) is reproduced while FateCompass still ranks β as the most likely terminal fate across all tested kernels and parameters.

## Related tools

- **FateCompass** (Method under test; run in RNA-velocity-driven trajectory mode via the fatecompass.py module to estimate transition probabilities and rank terminal fates) — https://github.com/sarajimenez/fatecompass
- **CellRank** (Baseline method; velocity kernel + Markov-chain absorption-probability estimator whose terminal-state calls are compared against FateCompass)
- **Scanpy** (Suggested pre-processing of the scRNA-seq E15.5 dataset before running both methods)
- **scVelo** (Suggested pre-processing and source of the RNA velocity field used by both CellRank's velocity kernel and FateCompass's Markov chain)
- **Python** (Implementation language of FateCompass and the comparison workflow)

## Examples

```
conda env create -f fatecompass.yml
```

## Evaluation signals

- CellRank baseline reproduces the published failure mode: ε and β identified as terminal, but glucagon-producing α cells NOT identified as terminal
- FateCompass identifies β as the most likely terminal fate for endocrine progenitors across all tested parameters and kernels
- cellrank_fate_probabilities.csv has exactly one row per cell population with the three required columns (CellRank fate probability, CellRank terminal-state call, FateCompass terminal-fate ranking)
- Both methods agree that β is a terminal fate, supporting the positive control; disagreement is only expected on the α lineage

## Limitations

- FateCompass's pipeline depends on input parameters that must be hand tuned; tutorials are provided to guide the parameter setup
- CellRank's terminal-state identification is kernel- and parameter-dependent — results must be reported across all tested kernels/parameters, not a single run
- FateCompass relies on prior biological knowledge in its differentiation-potential mode, so this benchmark should use the RNA-velocity-driven mode for a fair comparison

## Evidence

- [results] On the E15.5 dataset including Ductal and Ngn3-low EP populations, CellRank identified ε and β cells as terminal hormone-producing fates but failed to identify the glucagon-producing α cells: [evidence span withheld — anchor status: not_found]
- [results] FateCompass accurately identified the β fate as the most likely terminal fate for endocrine progenitors across all tested parameters and kernels.: [evidence span withheld — anchor status: not_found]
- [discussion] FateCompass uses RNA velocity to direct the Markov chain: "When RNA velocity profiles are robust, FateCompass uses them to direct the edges of the Markov chain."
- [readme] FateCompass module file and pre-processing suggestion: "The FateCompass module is provided in the fatecompass.py file."
- [readme] Pre-processing guidance for the adata input: "We suggest performing basic pre-processing using [Scanpy](https://scanpy.readthedocs.io/en/stable/) and [scVelo](https://scvelo.readthedocs.io/en/stable/)."
