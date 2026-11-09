---
name: cluster-purity-cell-informativeness-classification
description: Use when after filtering a healthy/reference signal from case (e.g. COVID-19
  PBMC) cells with SiFT, you must identify which single cells carry disease-related
  dynamics and which subpopulations they belong to.
when_to_use_negative:
- The data has not yet been SiFT-filtered against the healthy reference — purity on
  the original (unfiltered) neighborhood structure fails to expose the relevant cell
  types and should not be interpreted as informativeness
- The goal is removal of unwanted variation (e.g. cell cycle, sex, batch) rather than
  identifying disease-informative cells via a reference trajectory
- No reference samples or prior mapping exist to define the cell-cell similarity kernel
  — SiFT filtering and downstream purity classification both depend on it
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SiFT (sift-sc)
  role: Filters the healthy reference signal from disease-cell gene expression to
    produce the filtered matrix on which cluster purity is computed
  repo: https://github.com/nitzanlab/sift-sc
- name: sift.kernels.KnnKernel
  role: Builds the cell-cell similarity kNN graph mapping disease cells to nearest
    healthy reference cells; the filtered neighborhood structure is what purity is
    scored on
  repo: https://github.com/nitzanlab/sift-sc
- name: sift.SiFT / sift.sifter
  role: API entry points for projecting expression onto the healthy signal and deducting
    it
  repo: https://github.com/nitzanlab/sift-sc
- name: bbknn
  role: bbknn-style neighbor correction used to harmonize disease and healthy cells
    in a joint PCA latent space before kernel construction
- name: pip
  role: Installs the sift-sc package via pip install sift-sc
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:3af998b62d81e912d8b66ce348fb17a23849356ccbc3584bc0f00071228d7be2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Python
  - SiFT
  - SiFT (sift-sc)
  - bbknn
  - pip
  - sift.SiFT / sift.sifter
  - sift.kernels.KnnKernel
  - sift.sifter
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Cluster-purity cell-informativeness classification

## Summary

Classifies individual cells as disease-informative vs non-informative by scoring neighborhood cluster purity on a kNN graph built from SiFT-filtered expression data (healthy reference signal removed) and thresholding purity at tau = 0.55, then aggregating informative labels per annotated cell type.

## When to use

Use when, after filtering a healthy/reference signal from case (e.g. COVID-19 PBMC) cells with SiFT, you must identify which single cells carry disease-related dynamics and which subpopulations they belong to.

## When NOT to use

- The data has not yet been SiFT-filtered against the healthy reference — purity on the original (unfiltered) neighborhood structure fails to expose the relevant cell types and should not be interpreted as informativeness
- The goal is removal of unwanted variation (e.g. cell cycle, sex, batch) rather than identifying disease-informative cells via a reference trajectory
- No reference samples or prior mapping exist to define the cell-cell similarity kernel — SiFT filtering and downstream purity classification both depend on it

## Inputs

- Annotated scRNA-seq count matrix (e.g. .h5ad) with disease (COVID-19 patient) and healthy-control PBMC cells
- SiFT-filtered gene expression matrix for disease cells (healthy projection deducted)
- kNN graph / neighborhood structure computed in the filtered space (from a sift.kernels.KnnKernel)
- Cell-type annotations (e.g. CD14+ monocytes, CD16+ monocytes, Plasmablasts, Platelets)

## Outputs

- Per-cell informativeness label (disease-informative vs non-informative) determined by the tau = 0.55 purity threshold
- Per-cell-type counts of disease-informative subpopulations (summary akin to the paper's Fig. 6-7)
- IFN-response enrichment scores (e.g. IFN gene-set scores) per cell type on the filtered data for validation

## How to apply

First filter the healthy signal: harmonize disease and healthy PBMC cells in a joint PCA latent space with a bbknn-style integration, build a cell-cell similarity kernel with sift.kernels.KnnKernel mapping each disease cell to nearest healthy reference cells, and use sift.SiFT/sift.sifter to deduct the healthy projection from gene expression. Then compute cluster purity over the filtered neighborhood structure: for each disease cell, measure the consistency of its neighborhood in the filtered kNN graph, and label cells with purity above the threshold tau = 0.55 as disease-informative, per the paper's classification scheme. Aggregate informative labels per annotated cell type to expose disease-informative subpopulations (expected: CD14+ monocytes, CD16+ monocytes, Plasmablasts, Platelets). Validate by checking that IFN-response gene-set enrichment in monocytes, DCs, and HSPC emerges in the filtered data but not the original data. Rationale: filtering removes shared healthy trajectory variation so the residual neighborhood structure encodes disease-specific signal, making purity a meaningful informativeness score.

## Related tools

- **SiFT (sift-sc)** (Filters the healthy reference signal from disease-cell gene expression to produce the filtered matrix on which cluster purity is computed) — https://github.com/nitzanlab/sift-sc
- **sift.kernels.KnnKernel** (Builds the cell-cell similarity kNN graph mapping disease cells to nearest healthy reference cells; the filtered neighborhood structure is what purity is scored on) — https://github.com/nitzanlab/sift-sc
- **sift.SiFT / sift.sifter** (API entry points for projecting expression onto the healthy signal and deducting it) — https://github.com/nitzanlab/sift-sc
- **bbknn** (bbknn-style neighbor correction used to harmonize disease and healthy cells in a joint PCA latent space before kernel construction)
- **pip** (Installs the sift-sc package via pip install sift-sc)

## Examples

```
pip install sift-sc
```

## Evaluation signals

- Informativeness labels are stable around the tau = 0.55 purity threshold (sensitivity check on tau)
- Informative cells aggregate into the expected subpopulations: CD14+ monocytes, CD16+ monocytes, Plasmablasts, and Platelets
- IFN-response gene-set enrichment appears in monocytes, DCs, and HSPC on the filtered data but is absent when the same analysis runs on the original data
- Qualitative reproduction of the paper's Fig. 6-7 summary derived from healthy-trajectory-filtered data
- Filtered gene expression is no longer properly log-transformed — downstream scoring must account for this

## Limitations

- No ground truth reference exists to directly evaluate the filtering or the informativeness classification; application-specific integration metrics are suggested instead
- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal
- SiFT modifies gene expression so the filtered data is no longer properly log-transformed, affecting downstream analysis tools applied before purity scoring
- Statistical implications of applying commonly used analysis tools to corrected gene expression data remain unexplored

## Evidence

- [abstract] SiFT exposes COVID-19 disease-related cells, pathways, and dynamics by filtering healthy reference signals: "exposes COVID-19 disease-related cells, pathways, and dynamics by filtering healthy reference signals"
- [intro] Now, diverse analysis tools can be applied to study the filtered data and explore the underlying biological signals it encodes: "Now, diverse analysis tools can be applied to study the filtered data and explore the underlying biological signals it encodes"
- [discussion] In the context of COVID-19, SiFT exposed disease-related signals and single-cell dynamics by filtering a corresponding healthy trajectory: "In the context of COVID-19, SiFT exposed disease-related signals and single-cell dynamics by filtering a corresponding healthy trajectory"
- [discussion] SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel: "it is important to note that SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel."
- [readme] README describes SiFT as a tool for uncovering hidden biological processes, including reference-based trajectory filtering: "Signal FilTering is a tool for uncovering hidden biological processes in single-cell data."
