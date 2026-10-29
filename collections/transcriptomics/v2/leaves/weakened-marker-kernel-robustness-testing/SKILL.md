---
name: weakened-marker-kernel-robustness-testing
description: Use when you must quantify robustness of a signal-filtering or integration
  method to weakened prior knowledge — e.g., testing whether replacing marker genes
  in a cell cycle/sex kernel destabilizes iLISI-based filtering quality.
when_to_use_negative:
- Downstream analysis requires properly log-transformed expression, since SiFT modifies
  gene expression so the data is no longer properly log-transformed
- No prior marker knowledge is available at all — the kernel cannot be defined, and
  no robustness-to-partial-knowledge comparison is possible
- The question is absolute filtering accuracy with a ground-truth reference, which
  the paper notes is generally unavailable for computational filtering
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SiFT (sift-sc)
  role: 'Primary method under test: builds a cell-cell similarity kernel (KnnKernel
    or MappingKernel) from the weakened marker signal to filter cell cycle and cell
    cycle + sex signals at gene-expression level'
  repo: https://github.com/nitzanlab/sift-sc
- name: scVI
  role: Latent-space integration baseline (via scvi-tools) trained on the same filtered
    attributes; expected to show the largest degradation (~-13.5%/-16.3%)
  repo: https://github.com/scverse/scvi-tools
- name: scanpy regress_out
  role: Regression-based baseline applying scanpy regress_out to remove cell cycle
    and cell cycle + sex covariates; expected degradation ~-1.0%/-5.9%
- name: pip
  role: Installation channel for the SiFT python package
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:a49a390fc34f85c7ad35678ba9cb7a067e27b1c00f7b5a5732edbe4c8e8d90d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - SiFT
  - SiFT (sift-sc)
  - pip
  - scVI
  - scanpy regress_out
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# weakened-marker-kernel-robustness-testing

## Summary

Benchmark protocol for measuring how much filtering/integration methods degrade when prior knowledge (marker gene sets) is weakened: SiFT, scanpy regress_out, and scVI are re-run on Drosophila wing disc data with a 55-gene partial cell cycle/sex marker set, and percentage degradation of iLISI is averaged over 10 repetitions per method and attribute.

## When to use

Use when you must quantify robustness of a signal-filtering or integration method to weakened prior knowledge — e.g., testing whether replacing marker genes in a cell cycle/sex kernel destabilizes iLISI-based filtering quality.

## When NOT to use

- Downstream analysis requires properly log-transformed expression, since SiFT modifies gene expression so the data is no longer properly log-transformed
- No prior marker knowledge is available at all — the kernel cannot be defined, and no robustness-to-partial-knowledge comparison is possible
- The question is absolute filtering accuracy with a ground-truth reference, which the paper notes is generally unavailable for computational filtering

## Inputs

- Drosophila wing disc single-cell expression dataset (AnnData/h5ad)
- Weakened 55-gene marker set with 10 markers replaced by genes of interest (cell cycle / sex prior knowledge)
- Cell cycle and cell cycle + sex attribute labels per cell (for iLISI computation)
- Original full marker set (reference for percentage degradation calculation)

## Outputs

- degradation_scores.csv with mean percentage iLISI degradation per method/attribute (SiFT -0.6%/-4.2%, regress_out -1.0%/-5.9%, scVI -13.5%/-16.3%)
- Per-repetition iLISI degradation values (10 repetitions per method per attribute)
- Filtered expression matrices from SiFT kernel projection for each repetition

## How to apply

Load the Drosophila wing disc single-cell expression dataset together with the weakened 55-gene marker set in which 10 markers are replaced by genes of interest. Build a cell-cell similarity kernel from the weakened marker signal (sift.kernels.KnnKernel or sift.kernels.MappingKernel via sift.SiFT) and filter the cell cycle signal, then separately the combined cell cycle + sex signal. As baselines, apply scanpy regress_out on the same covariates and train scVI (scverse scvi-tools) as a latent-space integration baseline. For each method and attribute (cell cycle; cell cycle and sex), run 10 repetitions, compute the percentage degradation of iLISI relative to the original marker set each time, and average across repetitions. Rationale: the paper notes SiFT 'strongly relies on the prior knowledge used to define the cell–cell similarity kernel', so this protocol empirically tests how much an inaccurate mapping costs — targeting Supplementary Fig. 4 values of SiFT -0.6%/-4.2%, regress_out -1.0%/-5.9%, scVI -13.5%/-16.3%.

## Related tools

- **SiFT (sift-sc)** (Primary method under test: builds a cell-cell similarity kernel (KnnKernel or MappingKernel) from the weakened marker signal to filter cell cycle and cell cycle + sex signals at gene-expression level) — https://github.com/nitzanlab/sift-sc
- **scVI** (Latent-space integration baseline (via scvi-tools) trained on the same filtered attributes; expected to show the largest degradation (~-13.5%/-16.3%)) — https://github.com/scverse/scvi-tools
- **scanpy regress_out** (Regression-based baseline applying scanpy regress_out to remove cell cycle and cell cycle + sex covariates; expected degradation ~-1.0%/-5.9%)
- **pip** (Installation channel for the SiFT python package)

## Examples

```
pip install sift-sc; import sift; sifter = sift.SiFT(kernel=sift.kernels.MappingKernel())
```

## Evaluation signals

- Mean iLISI degradation across 10 repetitions reproduces Supplementary Fig. 4 targets: SiFT -0.6% (cell cycle) / -4.2% (cell cycle and sex); regress_out -1.0% / -5.9%; scVI -13.5% / -16.3%
- SiFT's degradation is minimal (well under ~5%) for both attributes, confirming robustness to the weakened 55-gene kernel
- degradation_scores.csv contains one mean row per method/attribute pair (6 rows total: 3 methods x 2 attributes) with finite percentage values
- Per-repetition iLISI values are computed identically for all three methods on the same Drosophila wing disc inputs
- SiFT filtering runs use a kernel built only from the weakened marker gene signal (KnnKernel or MappingKernel), not the original full marker set

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal
- Evaluation relies on integration metrics such as iLISI, which may be application-specific, and no ground truth reference exists to directly validate filtering results
- SiFT modifies gene expression so the corrected data is no longer properly log-transformed, which matters for downstream tasks expecting log-transformed input

## Evidence

- [discussion] kernel reliance: "it is important to note that SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel."
- [discussion] inaccurate mapping: "As we showcase in various examples, an inaccurate or coarse-grained mapping may lead to a loss of signal."
- [intro] kernel definition: "we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter"
- [discussion] evaluation metrics: "we suggest using integration metrics (which in general may be application-specific), as has been used throughout this manuscript"
- [readme] installation: "Install SiFT via PyPI by running::

    pip install sift-sc"
