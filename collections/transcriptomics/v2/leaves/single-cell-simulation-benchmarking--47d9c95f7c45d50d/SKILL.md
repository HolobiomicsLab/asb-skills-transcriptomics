---
name: single-cell-simulation-benchmarking
description: Use when a computational method for trajectory inference, trajectory
  alignment, RNA velocity, or cell-specific gene regulatory network inference (CSNI)
  must be evaluated quantitatively against fully known ground truth.
when_to_use_negative:
- You need to evaluate methods on real biological data without ground truth — dyngen's
  value is completely known synthetic ground truth, not realism
- You need to simulate only the technical protocol (e.g. benchmarking clustering or
  differential expression) with existing simulators like Splatter, powsimR, PROSSTT,
  or SymSim
- You require ODE-based bulk dynamics — dyngen targets low-molecule single-cell regimes
  where ODE simulators are ill-suited
tools:
- name: dyngen
  role: Simulation engine generating the 42 benchmark datasets with cell-specific
    ground-truth regulatory networks for CSNI, trajectory alignment, and RNA velocity
    evaluation
  repo: github.com/dynverse/dyngen
- name: dyngen_manuscript
  role: Reproduction codebase; each experiment (usecase_network_inference, usecase_rna_velocity,
    usecase_trajectory_alignment) is run from its analysis/ folder
  repo: github.com/dynverse/dyngen_manuscript
- name: R / tidyverse
  role: Implementation language for evaluated CSNI methods (e.g. SSN via leave-one-out
    Pearson correlation differencing) and for scoring AUROC/AUPR against ground truth
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:f292f45bca853cd0f7f7ae4690b9d04551c26d1c4abf573b36a567e7b5361c8b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R / tidyverse
  - dyngen
  - dyngen_manuscript
  - tidyverse
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# single-cell-simulation-benchmarking

## Summary

Benchmark single-cell computational methods using dyngen, a multi-modal simulation engine that provides completely known ground truth (cell state, reaction propensities, and cellwise regulatory networks) so methods can be evaluated quantitatively on 42 synthetic datasets built from 14 predefined backbones across three seeds.

## When to use

Use when a computational method for trajectory inference, trajectory alignment, RNA velocity, or cell-specific gene regulatory network inference (CSNI) must be evaluated quantitatively against fully known ground truth.

## When NOT to use

- You need to evaluate methods on real biological data without ground truth — dyngen's value is completely known synthetic ground truth, not realism
- You need to simulate only the technical protocol (e.g. benchmarking clustering or differential expression) with existing simulators like Splatter, powsimR, PROSSTT, or SymSim
- You require ODE-based bulk dynamics — dyngen targets low-molecule single-cell regimes where ODE simulators are ill-suited

## Inputs

- dyngen simulation specification: module network / predefined backbone (14 backbones x 3 seeds)
- Per-cell transcriptomics profiles produced by dyngen's Gillespie SSA simulation
- Cell-specific ground-truth regulatory networks emitted by dyngen
- Real reference datasets for profile emulation (FANTOM5 GRN, zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca)
- Predicted cell-specific networks from each method under evaluation (e.g. LIONESS, pySCENIC, SSN)

## Outputs

- 42 benchmarked datasets with per-cell AUROC and AUPR scores per method condition
- Mean AUROC and mean AUPR per dataset and condition (scores table with one entry per dataset)
- Summary figures and rankings (e.g. SSN worse than LIONESS and pySCENIC) reproducible via manuscript/render.Rmd

## How to apply

Generate simulation datasets with dyngen by defining a module network, generating the gene regulatory network (TF sampling, TF interactions, target subnetwork, housekeeping subnetwork), converting the GRN to transcription/splicing/translation/degradation reactions, simulating cells with Gillespie's SSA, and emulating single-cell omics profiling with real reference datasets (e.g. FANTOM5 GRNs and a zenodo-hosted silver-standard count dataset). Reproduce each manuscript experiment from the analysis folder of the dynverse/dyngen_manuscript repository (e.g. analysis/usecase_network_inference for CSNI), running each method (LIONESS, pySCENIC, SSN) in R with tidyverse on the same normalised dyngen data with no extra cell or feature filtering. For CSNI, compute for each cell the AUROC and AUPR of the predicted interactions against the cell-specific ground-truth regulatory network, restricting predictions to the top 10,000 interactions per cell, then average per-cell scores within each of the 42 datasets to obtain mean AUROC and mean AUPR per method condition. For other applications, use the ABWAP score (alignment vs worst possible alignment) for trajectory alignment and comparison of inferred velocity vectors against the known dyngen velocity vector for scVelo and velocyto.

## Related tools

- **dyngen** (Simulation engine generating the 42 benchmark datasets with cell-specific ground-truth regulatory networks for CSNI, trajectory alignment, and RNA velocity evaluation) — github.com/dynverse/dyngen
- **dyngen_manuscript** (Reproduction codebase; each experiment (usecase_network_inference, usecase_rna_velocity, usecase_trajectory_alignment) is run from its analysis/ folder) — github.com/dynverse/dyngen_manuscript
- **R / tidyverse** (Implementation language for evaluated CSNI methods (e.g. SSN via leave-one-out Pearson correlation differencing) and for scoring AUROC/AUPR against ground truth)

## Examples

```
git clone https://github.com/dynverse/dyngen_manuscript && Rscript -e 'devtools::install("package/", dependencies = TRUE)' ; run analysis/usecase_network_inference scripts from the dyngen_manuscript working directory
```

## Evaluation signals

- Each of the 42 datasets yields a mean AUROC and mean AUPR value within [0, 1]
- The resulting scores table contains exactly one entry per dataset for every method condition (e.g. SSN, LIONESS, pySCENIC)
- Per-cell scores are computed over the top 10,000 predicted interactions per cell, as specified in the CSNI workflow
- All 14 predefined backbones appear across three seeds among the benchmarked datasets
- Plotted rankings reproduce the manuscript's findings (e.g. cellAlign significantly better than DTW; SSN worse than LIONESS and pySCENIC)

## Limitations

- One major assumption of dyngen is that cells are assumed to be well-mixed and independent from each other
- No software version numbers are reported for dyngen or the evaluated tools; parameters were matched to tutorial vignettes
- Real reference single-cell RNA-seq datasets for profile emulation are downloaded from an unnamed repository
- dyngen's evaluation scope covers CSNI, RNA velocity, and trajectory alignment; it does not cover benchmarking of clustering or differential expression protocols

## Evidence

- [results] For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells: "For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells"
- [abstract] dyngen is more flexible than current single-cell simulation engines, and allows better method development and benchmarking: "dyngen is more flexible than current single-cell simulation engines, and allows better method development and benchmarking"
- [intro] dyngen can cater to a wide range of benchmarking applications, including trajectory inference, trajectory alignment, and trajectory differential expression: "dyngen can cater to a wide range of benchmarking applications, including trajectory inference, trajectory alignment, and trajectory differential expression"
- [readme] This repository contains all code necessary to reproduce the corresponding manuscript.: "dyngen is a multi-modal simulator for spearheading single-cell omics analyses. 
This repository contains all code necessary to reproduce the corresponding manuscript."
- [methods] The GRN is generated based on the given module network in four main steps (Supplementary Fig. 6).: "The GRN is generated based on the given module network in four main steps (Supplementary Fig. 6)."
