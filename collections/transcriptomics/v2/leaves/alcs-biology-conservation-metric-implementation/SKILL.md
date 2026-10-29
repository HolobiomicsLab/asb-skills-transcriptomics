---
name: alcs-biology-conservation-metric-implementation
description: Use when benchmarking cross-species integration strategies (e.g. LIGER,
  LIGER UINMF, fastMNN, SeuratV4, scANVI, scVI, Harmony, Scanorama, SAMap) and you
  need to quantify how much integration blurs distinct cell types, i.e. overcorrection,
  on integrated embeddings with ground-truth cell type labels.
when_to_use_negative:
- Cell type annotations are untrusted — the metric's quantitative scoring assumes
  accurate annotations and confident known cell type homology from literature
- The desired cell type correspondence across species is non-one-to-one; the benchmark
  is limited to one-to-one cell type mappings and valid non-one-to-one mappings are
  not assessed
- You need to evaluate species mixing rather than biology conservation — use the paper's
  species-mixing metrics alongside ALCS
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCCAF
  role: Trains the multinomial logistic regression self-projection classifier used
    to compute held-out accuracy for both the unintegrated baseline and integrated
    embeddings
- name: BENGAL
  role: The benchmarking pipeline in which ALCS is implemented and run across integration
    strategies
- name: LIGER
  role: Integration strategy under evaluation; shows noticeably higher ALCS than other
    strategies
  repo: https://github.com/welch-lab/liger
- name: LIGER UINMF
  role: Unshared-features iNMF integration strategy under evaluation; shows noticeably
    higher ALCS
  repo: https://github.com/welch-lab/liger
- name: fastMNN
  role: Mutual-nearest-neighbours integration strategy under evaluation; shows noticeably
    higher ALCS
  repo: https://github.com/LTLA/batchelor
- name: SeuratV4
  role: CCA-based integration strategy used for the per-species one-to-one orthologue
    ALCS analysis across the four heart tasks
  repo: https://github.com/satijalab/seurat-wrappers
- name: scOntoMatch
  role: Used to match cell ontology annotations across datasets so cell type labels
    are comparable before self-projection
  repo: https://github.com/Papatheodorou-Group/scOntoMatch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:b4268e5e28483b0524755b14db122f0920b77835cc69e9c139132fb7df58b2bc
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
  - scOntoMatch
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# ALCS biology conservation metric implementation

## Summary

Implements ALCS (accuracy loss of cell type self-projection), the BENGAL paper's new biology conservation metric for measuring maintenance of cell type distinguishability after cross-species scRNA-seq integration. It quantifies overcorrection (cell type blending) by comparing an SCCAF self-projection classifier's held-out accuracy on integrated embeddings versus the unintegrated per-species baseline.

## When to use

Use when benchmarking cross-species integration strategies (e.g. LIGER, LIGER UINMF, fastMNN, SeuratV4, scANVI, scVI, Harmony, Scanorama, SAMap) and you need to quantify how much integration blurs distinct cell types, i.e. overcorrection, on integrated embeddings with ground-truth cell type labels.

## When NOT to use

- Cell type annotations are untrusted — the metric's quantitative scoring assumes accurate annotations and confident known cell type homology from literature
- The desired cell type correspondence across species is non-one-to-one; the benchmark is limited to one-to-one cell type mappings and valid non-one-to-one mappings are not assessed
- You need to evaluate species mixing rather than biology conservation — use the paper's species-mixing metrics alongside ALCS

## Inputs

- Integrated low-dimensional embeddings from each homology-mapping × integration-algorithm combination (28 combinations benchmarked)
- Unintegrated per-species single-cell expression matrices (Seurat RDS objects, loadable via getSeuratRds)
- Ground-truth cell type labels per species
- Species divergence-time metadata

## Outputs

- Per method–task ALCS values (integrated test accuracy − unintegrated baseline accuracy)
- Per-method aggregated ALCS across tasks
- ALCS vs species divergence-time plot/regression
- Per-species SeuratV4 CCA one-to-one orthologue ALCS table for the four heart tasks

## How to apply

Load the integrated embeddings for each homology-mapping × integration-algorithm combination together with the unintegrated per-species expression matrices, ground-truth cell type labels, and species divergence-time metadata. For each species and method, train an SCCAF multinomial logistic regression self-projection classifier on the unintegrated per-species data and record the held-out test accuracy as the baseline; repeat the self-projection on the integrated embedding and compute ALCS = integrated test accuracy − unintegrated baseline accuracy for every method–task pair. Negative ALCS values indicate the integration degraded cell type distinguishability (overcorrection). Aggregate ALCS per method across tasks, and regress/plot ALCS against species divergence time to test whether biology conservation worsens as more distantly diverged species are integrated. Compute SeuratV4 CCA one-to-one orthologue ALCS separately per species across the four heart integration tasks to localize which species lose distinguishability.

## Related tools

- **SCCAF** (Trains the multinomial logistic regression self-projection classifier used to compute held-out accuracy for both the unintegrated baseline and integrated embeddings)
- **BENGAL** (The benchmarking pipeline in which ALCS is implemented and run across integration strategies)
- **LIGER** (Integration strategy under evaluation; shows noticeably higher ALCS than other strategies) — https://github.com/welch-lab/liger
- **LIGER UINMF** (Unshared-features iNMF integration strategy under evaluation; shows noticeably higher ALCS) — https://github.com/welch-lab/liger
- **fastMNN** (Mutual-nearest-neighbours integration strategy under evaluation; shows noticeably higher ALCS) — https://github.com/LTLA/batchelor
- **SeuratV4** (CCA-based integration strategy used for the per-species one-to-one orthologue ALCS analysis across the four heart tasks) — https://github.com/satijalab/seurat-wrappers
- **scOntoMatch** (Used to match cell ontology annotations across datasets so cell type labels are comparable before self-projection) — https://github.com/Papatheodorou-Group/scOntoMatch

## Examples

```
obj_list = getSeuratRds(metadata = metadata, sep = "\t")
```

## Evaluation signals

- Verify the reproduced ordering: LIGER, LIGER UINMF and fastMNN show the highest ALCS relative to other integration strategies
- Confirm ALCS increases (monotonically, via regression/plot) as species divergence time grows
- Confirm the ALCS identity holds per method–task pair: ALCS = integrated SCCAF test accuracy − unintegrated baseline accuracy
- Confirm per-species SeuratV4 CCA one-to-one orthologue ALCS values are produced for all four heart integration tasks
- Check the unintegrated baseline accuracies are computed from held-out SCCAF self-projection, not training accuracy

## Limitations

- Quantitative scoring assumes accurate cell type annotations and confident known cell type homology from literature
- Assessment uses one-to-one cell type mappings; valid non-one-to-one mappings across species are not assessed due to lack of ground-truth datasets, computational methods, and specialised metrics
- Cross-species integration relies on curated sequence-based gene orthology, ignoring species-specific genes, and expression similarities may arise from convergence rather than homology

## Evidence

- [abstract] New biology conservation metric: "We also develop a new biology conservation metric to address the maintenance of cell type distinguishability."
- [results] Methods that lose cell type structure: [evidence span withheld — anchor status: not_found]
- [abstract] Strategy balance claim: "Overall, scANVI, scVI and SeuratV4 methods achieve a balance between species-mixing and biology conservation."
- [readme] LIGER method basis: [evidence span withheld — anchor status: external_doc]
- [discussion] Assumption limitation: "Our quantitative scoring system of species mixing and biology conservation is supported by two assumptions: (1) cell type annotations are accurate"
