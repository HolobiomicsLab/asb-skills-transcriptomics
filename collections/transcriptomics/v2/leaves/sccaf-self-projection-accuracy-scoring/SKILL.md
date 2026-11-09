---
name: sccaf-self-projection-accuracy-scoring
description: Use when benchmarking cross-species single-cell RNA-seq integration strategies
  (e.g. LIGER, LIGER UINMF, fastMNN, SeuratV4, scANVI, scVI) and you need to measure
  whether integration overcorrects and blends distinct cell types, reducing cell type
  distinguishability.
when_to_use_negative:
- Cell type annotations are unreliable or cell type homology across species is uncertain
  — the paper notes quantitative scoring assumes accurate annotations and confident
  known cell type homology
- The mapping between cross-species cell types is not one-to-one — the benchmark and
  this metric are limited to one-to-one cell type mappings and do not assess non-one-to-one
  mappings
- The goal is to measure species mixing rather than biology/cell-type conservation
  — use the species-mixing metrics instead of ALCS
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCCAF
  role: Trains multinomial logistic regression self-projection classifiers on unintegrated
    and integrated data to compute baseline and integrated cell-type classification
    accuracy
- name: LIGER
  role: One of the benchmarked integrators whose embeddings are scored for ALCS; reported
    to show noticeably higher ALCS than other strategies
  repo: https://github.com/welch-lab/liger
- name: LIGER UINMF
  role: Benchmarked integrator using unshared features; embeddings scored for ALCS,
    also among the highest-ALCS strategies
  repo: https://github.com/welch-lab/liger
- name: fastMNN
  role: Mutual-nearest-neighbours integrator whose embeddings are scored for ALCS;
    also among the highest-ALCS strategies
  repo: https://github.com/LTLA/batchelor
- name: SeuratV4
  role: Benchmarked integrator achieving a balance between species-mixing and biology
    conservation; its CCA one-to-one orthologue ALCS is tabulated per species for
    heart tasks
  repo: https://github.com/satijalab/seurat-wrappers
- name: SAMap
  role: Benchmarked integrator for whole-body atlases with challenging gene homology
    annotation; embeddings scored for ALCS
  repo: https://github.com/atarashansky/SAMap
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:157942c5a08a0d27560e91b789f2f3415daf0c8a4c70ee6fb088d980c66a231a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/welch-lab/liger
  tools:
  - BENGAL
  - Harmony
  - LIGER
  - LIGER UINMF
  - SAMap
  - SCCAF
  - Scanorama
  - SeuratV4
  - fastMNN
  - scANVI
  - scVI
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# sccaf-self-projection-accuracy-scoring

## Summary

Quantifies loss of cell type distinguishability (ALCS) after cross-species integration by training SCCAF multinomial logistic regression self-projection classifiers on unintegrated per-species data (baseline) and on integrated embeddings, and computing ALCS = integrated test accuracy − unintegrated baseline accuracy.

## When to use

Use when benchmarking cross-species single-cell RNA-seq integration strategies (e.g. LIGER, LIGER UINMF, fastMNN, SeuratV4, scANVI, scVI) and you need to measure whether integration overcorrects and blends distinct cell types, reducing cell type distinguishability.

## When NOT to use

- Cell type annotations are unreliable or cell type homology across species is uncertain — the paper notes quantitative scoring assumes accurate annotations and confident known cell type homology
- The mapping between cross-species cell types is not one-to-one — the benchmark and this metric are limited to one-to-one cell type mappings and do not assess non-one-to-one mappings
- The goal is to measure species mixing rather than biology/cell-type conservation — use the species-mixing metrics instead of ALCS

## Inputs

- Integrated low-dimensional embeddings per homology-mapping × integration-algorithm combination (e.g. from LIGER, LIGER UINMF, fastMNN, SeuratV4 CCA, scANVI, scVI, Harmony, Scanorama, SAMap)
- Unintegrated per-species scRNA-seq expression matrices (e.g. Seurat RDS objects read via getSeuratRds)
- Ground-truth cell type labels per species (cell ontology-annotated, e.g. matched via scOntoMatch)
- Species divergence-time metadata for the integrated tasks (e.g. four heart integration tasks)

## Outputs

- ALCS values (integrated test accuracy − unintegrated baseline accuracy) per method–task pair
- Per-method aggregated ALCS across tasks
- Per-species SeuratV4 CCA one-to-one orthologue ALCS values for the four heart integration tasks
- ALCS vs species divergence-time plots/regressions

## How to apply

Load the integrated embeddings from each homology-mapping × integration-algorithm combination together with the unintegrated per-species expression matrices, ground-truth cell type labels and species divergence-time metadata. For each species and each method, train an SCCAF multinomial logistic regression self-projection classifier on the unintegrated per-species data and record the held-out test accuracy as the baseline. Repeat the self-projection classification on the integrated embedding and compute ALCS = integrated test accuracy − unintegrated baseline accuracy for every method–task pair; negative ALCS indicates integration preserved or improved cell type distinguishability, larger ALCS indicates overcorrection/blending. Aggregate ALCS per method across tasks, and regress or plot ALCS against species divergence time to test whether ALCS increases as more distantly diverged species are integrated. The rationale is that a biology-conserving integrator keeps cell types separable in the joint embedding, so self-projection accuracy should not drop relative to the unintegrated baseline.

## Related tools

- **SCCAF** (Trains multinomial logistic regression self-projection classifiers on unintegrated and integrated data to compute baseline and integrated cell-type classification accuracy)
- **LIGER** (One of the benchmarked integrators whose embeddings are scored for ALCS; reported to show noticeably higher ALCS than other strategies) — https://github.com/welch-lab/liger
- **LIGER UINMF** (Benchmarked integrator using unshared features; embeddings scored for ALCS, also among the highest-ALCS strategies) — https://github.com/welch-lab/liger
- **fastMNN** (Mutual-nearest-neighbours integrator whose embeddings are scored for ALCS; also among the highest-ALCS strategies) — https://github.com/LTLA/batchelor
- **SeuratV4** (Benchmarked integrator achieving a balance between species-mixing and biology conservation; its CCA one-to-one orthologue ALCS is tabulated per species for heart tasks) — https://github.com/satijalab/seurat-wrappers
- **SAMap** (Benchmarked integrator for whole-body atlases with challenging gene homology annotation; embeddings scored for ALCS) — https://github.com/atarashansky/SAMap

## Examples

```
obj_list = getSeuratRds(metadata = metadata, sep = "\t")
```

## Evaluation signals

- The computed ALCS reproduces the reported ordering: LIGER, LIGER UINMF and fastMNN show the highest ALCS relative to other integrators
- ALCS increases monotonically (or near-monotonically) as species divergence time grows when plotted/regressed against divergence-time metadata
- Per-species SeuratV4 CCA one-to-one orthologue ALCS values are produced for all four heart integration tasks
- Unintegrated baseline accuracies are finite, per-species held-out test accuracies from the SCCAF classifiers, so every method–task pair has a defined ALCS value
- scANVI, scVI and SeuratV4 (balanced methods) should show lower ALCS than LIGER/LIGER UINMF/fastMNN, consistent with their reported species-mixing vs biology-conservation balance

## Limitations

- Quantitative scoring assumes accurate cell type annotations and confident known cell type homology from literature; violations undermine ALCS interpretation
- The metric is assessed using one-to-one cell type mappings; valid non-one-to-one cell type mappings across species are not captured
- Assessment is limited by the availability of ground-truth datasets, sophisticated computational methods, and specialised metrics for non-one-to-one mappings

## Evidence

- [abstract] new biology conservation metric: "We also develop a new biology conservation metric to address the maintenance of cell type distinguishability."
- [results] ALCS increases with divergence: [evidence span withheld — anchor status: not_found]
- [abstract] balanced integrators: "Overall, scANVI, scVI and SeuratV4 methods achieve a balance between species-mixing and biology conservation."
- [readme] LIGER iNMF rationale: [evidence span withheld — anchor status: external_doc]
