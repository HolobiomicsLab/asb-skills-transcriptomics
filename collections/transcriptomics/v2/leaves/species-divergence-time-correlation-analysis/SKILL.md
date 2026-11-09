---
name: species-divergence-time-correlation-analysis
description: Use when benchmarking cross-species integration strategies (e.g. BENGAL
  tasks) and you need to test whether biology conservation, measured as ALCS from
  SCCAF self-projection, degrades with increasing species divergence time.
when_to_use_negative:
- The analysis involves only a single species (no cross-species divergence), so ALCS-versus-divergence-time
  trends cannot be defined
- Cell type annotations are uncurated or cross-species cell type homology is unknown
  — the paper states quantitative scoring assumes accurate annotations and confident
  known cell type homology
- The mapping of cell types across species is non-one-to-one; the benchmark is limited
  to one-to-one cell type mappings
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCCAF
  role: Trains the multinomial logistic regression self-projection classifier whose
    held-out accuracy differences define ALCS
- name: LIGER
  role: One of the benchmarked integrators whose ALCS is regressed against divergence
    time; identified as showing noticeably higher ALCS
  repo: https://github.com/welch-lab/liger
- name: fastMNN
  role: Benchmarked mutual-nearest-neighbours integrator showing noticeably higher
    ALCS in the divergence-time analysis
  repo: https://github.com/LTLA/batchelor
- name: SeuratV4
  role: Integrator whose CCA one-to-one orthologue ALCS is computed per species across
    the four heart tasks
  repo: https://github.com/satijalab/seurat-wrappers
- name: BENGAL
  role: Benchmarking pipeline whose BENGALSeurat-based workflow generates the ALCS
    self-projection results across strategies and tasks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:827b5e0302bca4626c2470886ccad18a245114fed893138a7f17e3fd37cd60b4
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# species-divergence-time-correlation-analysis

## Summary

Quantifies how the accuracy loss of cell type self-projection (ALCS) changes as more distantly diverged species are integrated, regressing or plotting per-method ALCS against species divergence-time metadata across cross-species scRNA-seq integration tasks.

## When to use

Use when benchmarking cross-species integration strategies (e.g. BENGAL tasks) and you need to test whether biology conservation, measured as ALCS from SCCAF self-projection, degrades with increasing species divergence time.

## When NOT to use

- The analysis involves only a single species (no cross-species divergence), so ALCS-versus-divergence-time trends cannot be defined
- Cell type annotations are uncurated or cross-species cell type homology is unknown — the paper states quantitative scoring assumes accurate annotations and confident known cell type homology
- The mapping of cell types across species is non-one-to-one; the benchmark is limited to one-to-one cell type mappings

## Inputs

- Integrated embeddings from benchmarked homology-mapping × integration-algorithm combinations (e.g. LIGER, LIGER UINMF, fastMNN, SeuratV4, scANVI, scVI, Harmony, Scanorama, SAMap outputs)
- Unintegrated per-species scRNA-seq expression matrices (Seurat .rds objects loadable via getSeuratRds)
- Ground-truth cell type labels per species
- Species divergence-time metadata for each integration task

## Outputs

- Per method–task ALCS values (integrated SCCAF test accuracy minus unintegrated baseline accuracy)
- Per-method aggregated ALCS across tasks
- Plot/regression of ALCS versus species divergence time
- Per-species SeuratV4 CCA one-to-one orthologue ALCS values tabulated across the four heart integration tasks

## How to apply

Load the integrated embeddings from each homology-mapping × integration-algorithm combination alongside the unintegrated per-species expression matrices, ground-truth cell type labels and species divergence-time metadata. For each species and method, train an SCCAF multinomial logistic regression self-projection classifier on the unintegrated data to record the held-out baseline accuracy, then repeat the classification on the integrated embedding and compute ALCS = integrated test accuracy − unintegrated baseline accuracy for every method–task pair. Aggregate ALCS per method across tasks and plot or regress ALCS against divergence time to confirm the reported monotonic increase; use the regression to flag strategies such as LIGER, LIGER UINMF and fastMNN that lose cell type distinguishability fastest. As a validation anchor, the computed ordering should reproduce the paper's pattern: LIGER/LIGER UINMF/fastMNN highest ALCS, and scANVI, scVI and SeuratV4 retaining a better balance between species mixing and biology conservation.

## Related tools

- **SCCAF** (Trains the multinomial logistic regression self-projection classifier whose held-out accuracy differences define ALCS)
- **LIGER** (One of the benchmarked integrators whose ALCS is regressed against divergence time; identified as showing noticeably higher ALCS) — https://github.com/welch-lab/liger
- **fastMNN** (Benchmarked mutual-nearest-neighbours integrator showing noticeably higher ALCS in the divergence-time analysis) — https://github.com/LTLA/batchelor
- **SeuratV4** (Integrator whose CCA one-to-one orthologue ALCS is computed per species across the four heart tasks) — https://github.com/satijalab/seurat-wrappers
- **BENGAL** (Benchmarking pipeline whose BENGALSeurat-based workflow generates the ALCS self-projection results across strategies and tasks)

## Evaluation signals

- Computed ALCS reproduces the reported ordering: LIGER, LIGER UINMF and fastMNN show noticeably higher ALCS than other strategies
- ALCS increases monotonically (or near-monotonically) as species divergence time grows in the plot/regression
- Per-species SeuratV4 CCA O2O ALCS values are produced for all four heart integration tasks
- Unintegrated SCCAF baseline accuracies are finite, non-trivial classifiers (well above chance) for every species, so ALCS is a valid accuracy difference
- Every method–task pair yields a defined ALCS value (no missing integrations) before aggregation

## Limitations

- Quantitative scoring assumes accurate cell type annotations and confident known cell type homology from literature
- The benchmark is limited to one-to-one cell type mappings; valid non-one-to-one cross-species cell type mappings are not assessed
- Cross-species integration relies on curated sequence-based gene orthology, ignoring species-specific genes; functional diversification between orthologs may exceed that between paralogs

## Evidence

- [results] ALCS increases with divergence time and LIGER-family/fastMNN overcorrection: [evidence span withheld — anchor status: not_found]
- [intro] Divergence time impacts integration output among strategies: "the number of species involved in the integration and the time that species have diverged can impact the output characteristics differently among strategies"
- [abstract] New biology conservation metric for cell type distinguishability: "using 9 established metrics. We also develop a new biology conservation metric to address the maintenance of cell type distinguishability."
- [discussion] Scoring assumptions: "Our quantitative scoring system of species mixing and biology conservation is supported by two assumptions: (1) cell type annotations are accurate;"
- [readme] LIGER iNMF basis: [evidence span withheld — anchor status: external_doc]
