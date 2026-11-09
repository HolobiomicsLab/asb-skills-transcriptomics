---
name: zero-inflation-dropout-tuning
description: Use when you need synthetic scRNA-seq data at multiple controlled zero-inflation
  levels (e.g., ~12-30% dropouts) to stress-test a method's robustness to dropout,
  as in scGAE's scenario4 benchmarks.
when_to_use_negative:
- You need empirical (real) dropout patterns — this skill produces Splatter-model
  dropout only, matching the paper's simulation-based scenario4, not dataset-specific
  zero distributions
- Your goal is zero-inflation correction/denoising of an existing matrix (e.g., DCA
  or ZINB-WaVE usage) rather than generating dropout-graded benchmark data
- A different dimensionality is required — the documented recipe is tied to 2000 cells
  x 5000 genes and the reported 12-30% rates were calibrated at that size
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Splatter
  role: R package used to simulate the base scenarios and apply dropout with dropout.shape=-1
    and swept dropout.mid values to create the zero-inflation variants
- name: scGAE
  role: Downstream method benchmarked on the resulting zero-inflation variants; simulation
    code locations are documented in Run_scGAE.ipynb in its repository
  repo: github.com/ZixiangLuo1161/scGAE
- name: PROSSTT
  role: Python simulator used for the non-Splatter scenarios (data2 and data5) in
    the same nine-matrix benchmark set
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:1d13efdba0cbabdf1d8db885f1f20a9f69b9eea859d0fef7693c850f6f2b63d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PROSSTT
  - Splatter
  - scGAE
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# zero-inflation-dropout-tuning

## Summary

Generates controlled zero-inflation variants of simulated scRNA-seq count matrices by sweeping Splatter's dropout parameters (dropout.shape=-1 with graded dropout.mid values), yielding ~12%, 17%, 23%, and 30% zero inflation for benchmarking dropout robustness as done in the scGAE scenario4 experiments.

## When to use

Use when you need synthetic scRNA-seq data at multiple controlled zero-inflation levels (e.g., ~12-30% dropouts) to stress-test a method's robustness to dropout, as in scGAE's scenario4 benchmarks.

## When NOT to use

- You need empirical (real) dropout patterns — this skill produces Splatter-model dropout only, matching the paper's simulation-based scenario4, not dataset-specific zero distributions
- Your goal is zero-inflation correction/denoising of an existing matrix (e.g., DCA or ZINB-WaVE usage) rather than generating dropout-graded benchmark data
- A different dimensionality is required — the documented recipe is tied to 2000 cells x 5000 genes and the reported 12-30% rates were calibrated at that size

## Inputs

- Splatter simulation parameter settings (2000 cells, 5000 genes) for a base simulated scenario
- Base simulated count matrix (CSV/h5ad) from Splatter
- Dropout parameter values: dropout.shape=-1 and dropout.mid in {-0.5, 0, 0.5, 1}

## Outputs

- Four zero-inflated count matrix variants (CSV or h5ad) with approximately 12%, 17%, 23%, and 30% zeros, each 2000 rows (cells) x 5000 columns (genes)

## How to apply

First generate the base simulated dataset with the Splatter R package at the target dimensions (2000 cells x 5000 genes, as in the scGAE data1-data4 scenarios). Then create zero-inflation variants by setting dropout.shape=-1 and sweeping dropout.mid over -0.5, 0, 0.5, and 1, which produces approximately 12%, 17%, 23%, and 30% dropout rates respectively. Write each resulting count matrix to disk as a CSV or h5ad file alongside the base scenarios (data1-data5 plus the four scenario4 variants = nine matrices in the scGAE workflow). Rationale: massively parallel scRNA-seq data are 'of high dropout and high noise', so varying dropout.mid systematically while fixing dropout.shape isolates zero-inflation as the tested variable. Validate every output by reloading it and confirming dimensions and the expected zero fraction per variant.

## Related tools

- **Splatter** (R package used to simulate the base scenarios and apply dropout with dropout.shape=-1 and swept dropout.mid values to create the zero-inflation variants)
- **scGAE** (Downstream method benchmarked on the resulting zero-inflation variants; simulation code locations are documented in Run_scGAE.ipynb in its repository) — github.com/ZixiangLuo1161/scGAE
- **PROSSTT** (Python simulator used for the non-Splatter scenarios (data2 and data5) in the same nine-matrix benchmark set)

## Examples

```
library(splatter); p <- setParams(newSplatParams(dim=c(2000,5000)), dropout.shape=-1, dropout.mid=0.5); sim <- splatSimulate(p)
```

## Evaluation signals

- Each output file loads as a valid CSV/h5ad count matrix with exactly 2000 rows (cells) and 5000 columns (genes)
- Zero fraction per variant matches the target range: approximately 12%, 17%, 23%, and 30% for dropout.mid = -0.5, 0, 0.5, 1 at dropout.shape=-1
- Zero rate increases monotonically with dropout.mid across the four variants
- All nine expected matrices exist on disk (data1-data5 plus the four scenario4 variants)

## Limitations

- The documented rates (12%, 17%, 23%, 30%) are approximate ('approximately 12-30% zero inflation per variant') and calibrated to the 2000-cell x 5000-gene Splatter setup
- Zero-inflation variants were only generated for scenario4 in the scGAE benchmark; the recipe does not cover PROSSTT-generated data2/data5
- The article reports no random seed or stochastic-variation control for the simulations, so exact zero fractions may vary run-to-run

## Evidence

- [methods] scenario4 zero-inflation variants generated with fixed shape: [evidence span withheld — anchor status: not_found]
- [intro] high dropout motivates controlled dropout benchmarking: "However, the data generated by massively parallel scRNA-seq are of high dropout and high noise with complex structure, which posed a series of challenges on dimensionality reduction."
- [readme] where the simulation parameters are documented: [evidence span withheld — anchor status: external_doc]
