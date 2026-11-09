---
name: ilisi-degradation-metric-computation
description: Use when comparing robustness of signal-removal methods (SiFT, regress_out,
  scVI) to partial or noisy marker gene priors, with iLISI percentage degradation
  as the metric on single-cell RNA-seq data.
when_to_use_negative:
- No cell annotations or label sets exist to compute iLISI, so degradation cannot
  be evaluated (the article notes SiFT 'suffers from the general challenge of any
  computational method which lacks a ground truth reference').
- The goal is batch correction with known discrete batch labels rather than robustness
  testing of continuous biological-signal filtering under a partially corrupted marker
  prior.
- Downstream analysis requires properly log-transformed expression, since filtered/corrected
  gene expression is no longer properly log-transformed.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SiFT (sift-sc)
  role: Method under test; filters cell cycle / cell cycle + sex signal using a cell-cell
    similarity kernel built from the weakened marker gene set
  repo: https://github.com/nitzanlab/sift-sc
- name: scVI
  role: Latent-space integration baseline trained on the same counts with cell cycle
    and sex scores as continuous covariates
  repo: https://github.com/scverse/scvi-tools
- name: scanpy
  role: Provides the regress_out covariate-removal baseline applied to the same data
- name: pip
  role: Installation channel for sift-sc used to set up the filtering environment
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:f44a5c96cfdfac81be6f9dbb17f5016d975d4710180361f2ec6bf57b3c248965
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Python
  - SiFT
  - SiFT (sift-sc)
  - pip
  - scVI
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ilisi-degradation-metric-computation

## Summary

A benchmarking procedure that quantifies how much each single-cell signal-filtering method's label-mixing performance (iLISI) degrades when prior marker knowledge is weakened, used to show SiFT's robustness versus regress_out and scVI on Drosophila wing disc cell cycle/sex filtering.

## When to use

Use when comparing robustness of signal-removal methods (SiFT, regress_out, scVI) to partial or noisy marker gene priors, with iLISI percentage degradation as the metric on single-cell RNA-seq data.

## When NOT to use

- No cell annotations or label sets exist to compute iLISI, so degradation cannot be evaluated (the article notes SiFT 'suffers from the general challenge of any computational method which lacks a ground truth reference').
- The goal is batch correction with known discrete batch labels rather than robustness testing of continuous biological-signal filtering under a partially corrupted marker prior.
- Downstream analysis requires properly log-transformed expression, since filtered/corrected gene expression is no longer properly log-transformed.

## Inputs

- Drosophila wing disc single-cell RNA-seq count matrix (AnnData/h5ad)
- Original curated cell cycle and sex marker gene lists
- Weakened 55-gene marker set with 10 markers replaced by genes of interest
- Cell labels/attributes for iLISI computation: cell cycle, and cell cycle + sex

## Outputs

- Per-repetition iLISI scores per method for cell-cycle-only and cell-cycle+sex attributes
- Percentage iLISI degradation relative to the original marker set, averaged over 10 repetitions
- degradation_scores.csv with mean degradation per method/attribute (targets: SiFT -0.6%/-4.2%, regress_out -1.0%/-5.9%, scVI -13.5%/-16.3%)

## How to apply

Load the Drosophila wing disc single-cell expression dataset together with a weakened marker set (a 55-gene cell cycle/sex marker set in which 10 markers are replaced by genes of interest). For each of 10 repetitions, run SiFT (via sift.SiFT with a KnnKernel or MappingKernel built from the weakened marker signal), scanpy regress_out, and scVI with the cell cycle and cell cycle + sex scores as continuous covariates to produce corrected data. After each repetition and method, compute the graph iLISI score for the cell-cycle-only and cell-cycle+sex attributes, then compute the percentage degradation relative to iLISI obtained with the original (full) marker set, averaging across the 10 repetitions. Correct application is judged by the aggregated mean degradation approaching the Supplementary Fig. 4 values: SiFT −0.6%/−4.2%, regress_out −1.0%/−5.9%, scVI −13.5%/−16.3% (iLISI cell cycle / iLISI cell cycle + sex). Write per-method, per-attribute means to degradation_scores.csv. The rationale is that robust methods should show minimal iLISI loss when the marker prior is degraded; SiFT relies on the cell-cell similarity kernel, so a partially wrong marker set should hurt it less than regression or latent-space baselines.

## Related tools

- **SiFT (sift-sc)** (Method under test; filters cell cycle / cell cycle + sex signal using a cell-cell similarity kernel built from the weakened marker gene set) — https://github.com/nitzanlab/sift-sc
- **scVI** (Latent-space integration baseline trained on the same counts with cell cycle and sex scores as continuous covariates) — https://github.com/scverse/scvi-tools
- **scanpy** (Provides the regress_out covariate-removal baseline applied to the same data)
- **pip** (Installation channel for sift-sc used to set up the filtering environment)

## Examples

```
pip install sift-sc
```

## Evaluation signals

- Mean iLISI degradation over 10 repetitions lands near Supplementary Fig. 4 values: SiFT -0.6% (cell cycle) / -4.2% (cell cycle + sex), regress_out -1.0% / -5.9%, scVI -13.5% / -16.3%.
- SiFT's degradation is consistently smaller (or comparable) to regress_out's and clearly smaller than scVI's across both attributes, reflecting robustness to the partial marker prior.
- degradation_scores.csv contains one mean degradation value per (method, attribute) pair covering all 3 methods x 2 attributes.
- Each method is run the full 10 repetitions and the same weakened 55-gene marker set (10 replaced markers) is used for all methods.
- iLISI is computed both for the cell-cycle-only label setting and the combined cell cycle + sex label setting.

## Limitations

- No ground truth reference exists for filtering results; evaluation relies on integration metrics such as iLISI, which 'in general may be application-specific'.
- SiFT 'strongly relies on the prior knowledge used to define the cell-cell similarity kernel', so an inaccurate or coarse-grained marker mapping may lead to a loss of signal and skew degradation estimates.
- Filtered gene expression is no longer properly log-transformed, affecting some downstream analyses.

## Evidence

- [intro] Applying SiFT to Drosophila wing disc filtering: "applying SiFT to filter nuisance signals in Drosophila wing disc development single-cell data and removing cell cycle effects from a semi-synthetic single-cell dataset"
- [intro] Kernel-based correction rationale: [evidence span withheld — anchor status: not_found]
- [discussion] iLISI as application-specific evaluation metric: "To mitigate this, we suggest using integration metrics (which in general may be application-specific), as has been used throughout this manuscript"
- [discussion] Prior-knowledge dependency caveat: [evidence span withheld — anchor status: not_found]
- [methods] Package installation: "Install `SiFT` in one of the following ways: Through **pip**:: pip install sift-sc"
