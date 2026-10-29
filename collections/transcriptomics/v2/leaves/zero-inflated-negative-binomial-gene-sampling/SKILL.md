---
name: zero-inflated-negative-binomial-gene-sampling
description: Use when benchmarking perturbation-analysis methods like ICAT and you
  need synthetic control/perturbed AnnData pairs with known ground-truth perturbed
  cell states and tunable perturbation severity.
when_to_use_negative:
- The input is real experimental scRNA-seq data rather than synthetic counts — use
  ICAT's analysis workflow directly instead of the simulation generator
- You need perturbation effects with non-multiplicative or condition-specific kinetics;
  this generator only applies Gamma(2,2) scalar multipliers to randomly selected target
  genes and Gamma(3,3)-shifted marker genes
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3524
tools:
- name: ICAT
  role: Downstream consumer of the simulated control/perturbed dataset; the simulation
    exists to benchmark ICAT's perturbation-unique state identification
  repo: https://github.com/BradhamLab/icat
- name: icat-sc
  role: pip-installable package providing the icat.simulate module (SingleCellDataset
    and perturb) used to run the generator
  repo: https://github.com/BradhamLab/icat
- name: AnnData
  role: Container format for the simulated count matrices plus treatment obs metadata
  repo: https://github.com/BradhamLab/icat
- name: numpy
  role: Samples per-gene dispersion values via np.random.choice([1, 2, 3], 1000)
- name: scanpy
  role: Concatenates control and perturbed AnnData and applies sc.pp.log1p normalization
    to the combined object
  repo: https://github.com/BradhamLab/icat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:7987f459ae7f9c9a9595730bcc01ed54f4bb1925f7d2fa3fcb131a5b5cd42726
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - ICAT
  - icat-sc
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# zero-inflated-negative-binomial-gene-sampling

## Summary

Generates benchmark scRNA-seq datasets for ICAT by simulating control counts from a zero-inflated negative-binomial model (simulate.SingleCellDataset, populations=2, genes=1000, per-gene dispersion from [1,2,3]) and applying Gamma-sampled multipliers to perturbation-target genes via simulate.perturb.

## When to use

Use when benchmarking perturbation-analysis methods like ICAT and you need synthetic control/perturbed AnnData pairs with known ground-truth perturbed cell states and tunable perturbation severity.

## When NOT to use

- The input is real experimental scRNA-seq data rather than synthetic counts — use ICAT's analysis workflow directly instead of the simulation generator
- You need perturbation effects with non-multiplicative or condition-specific kinetics; this generator only applies Gamma(2,2) scalar multipliers to randomly selected target genes and Gamma(3,3)-shifted marker genes

## Inputs

- Simulation parameters: population count (2), gene count (1000), per-gene dispersion values drawn via numpy from [1, 2, 3]
- Control AnnData produced by data_model.simulate() under the zero-inflated negative-binomial generative model

## Outputs

- Control AnnData with raw counts and obs['treatment']='control'
- Perturbed AnnData with Gamma(2,2)-scaled perturbation-target genes and Gamma(3,3)-shifted marker genes, obs['treatment']='perturbed'
- Concatenated AnnData (e.g. saved as simulated_counts.h5ad) usable for ICAT benchmarking against Supplementary Tables S1-S4

## How to apply

Instantiate simulate.SingleCellDataset with populations=2, genes=1000, and dispersion=np.random.choice([1, 2, 3], 1000), then call data_model.simulate() to emit the zero-inflated negative-binomial control count matrix and label obs['treatment']='control'. Call simulate.perturb(controls) to build the perturbed condition: randomly selected perturbation-target genes have their average expression scaled by multipliers drawn from a Gamma(2,2) distribution, while marker genes per population receive Gamma(3,3)-shifted effects. Concatenate the two AnnData objects so the combined object holds raw counts plus a treatment obs column with 'control' and 'perturbed' levels, matching the parameter settings in Supplementary Tables S1-S4. The resulting dataset can be log1p-normalized (sc.pp.log1p) and fed to ICAT's control-guided clustering to test whether it recovers a perturbation-unique cell state.

## Related tools

- **ICAT** (Downstream consumer of the simulated control/perturbed dataset; the simulation exists to benchmark ICAT's perturbation-unique state identification) — https://github.com/BradhamLab/icat
- **icat-sc** (pip-installable package providing the icat.simulate module (SingleCellDataset and perturb) used to run the generator) — https://github.com/BradhamLab/icat
- **AnnData** (Container format for the simulated count matrices plus treatment obs metadata) — https://github.com/BradhamLab/icat
- **numpy** (Samples per-gene dispersion values via np.random.choice([1, 2, 3], 1000))
- **scanpy** (Concatenates control and perturbed AnnData and applies sc.pp.log1p normalization to the combined object) — https://github.com/BradhamLab/icat

## Examples

```
from icat import simulate; import numpy as np; data_model = simulate.SingleCellDataset(populations=2, genes=1000, dispersion=np.random.choice([1, 2, 3], 1000)); controls = data_model.simulate(); perturbed = simulate.perturb(controls)
```

## Evaluation signals

- Emitted AnnData contains 2 populations x 1000 genes with dispersion values drawn from [1, 2, 3]
- obs['treatment'] is populated and contains exactly the levels 'control' and 'perturbed'
- Perturbed condition shows altered average expression in randomly selected perturbation-target genes, consistent with Gamma(2,2)-sampled multipliers
- Simulation parameter settings are consistent with the parameter tables in Supplementary Tables S1-S4
- Concatenated count matrix plus treatment metadata can be saved to simulated_counts.h5ad without losing levels

## Limitations

- The Gamma(2,2) multipliers are applied to randomly selected perturbation-target genes, so the location of true effects is stochastic across runs unless the RNG is seeded
- The README manuscript workflow 'is not guaranteed to run on your system out-of-the-box' (CentOS HPC, conda environment icat, snakemake pipeline)
- Article supplementary discussion content was not extractable ('No changelog found'), so full simulation parameter space beyond S1-S4 is not verifiable here

## Evidence

- [methods] Zero-inflated negative-binomial simulation with Gamma-sampled perturbation multipliers: "data_model = simulate.SingleCellDataset(
        populations=2,
        genes=1000,"
- [methods] perturb step produces the perturbed condition: "perturbed = simulate.perturb(controls)"
- [readme] dispersion sampling from [1,2,3] via numpy: "dispersion=np.random.choice([1, 2, 3], 1000)"
- [intro] ICAT simulation supplies the benchmark data: "ICAT is a tool developed to better identify cell states in scRNAseq experiments where perturbations or some other biologic heterogeneity is present, such as gene knock-outs."
