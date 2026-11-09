---
name: single-cell-rna-seq-multi-dataset-integration
description: Use when several independently generated scRNA-seq datasets (different
  labs, platforms, or batches) must be merged so cells group by biological cell type
  rather than by study of origin.
when_to_use_negative:
- Analyzing a single dataset with no batch/study structure to correct — plain ACTIONet
  without the Harmony-kernel reduction suffices
- When the goal is differential-state or activity analysis within one condition rather
  than cross-study integration and label harmonization
- When input is already a reduced/embedded feature representation rather than raw
  gene-by-cell counts
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ACTIONet (R package)
  role: Performs filter_adata preprocessing, Harmony-kernel reduce_kernel reduction,
    multiresolution network construction, label propagation, and signature extraction
    on the merged dataset
  repo: https://github.com/shmohammadi86/ACTIONet
- name: UMAP
  role: SGD-based 2D projection of the multiresolution network for visual inspection
    of batch mixing
- name: Leiden
  role: Graph clustering of archetypes/patterns used to define cell-type groups in
    the joint network
- name: ACTIONetExperiment
  role: AnnData-like SummarizedExperiment container for storing and manipulating the
    merged count data
  repo: https://github.com/shmohammadi86/ACTIONetExperiment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:ade1956b2d4de81d6a2f46d72d45a35e65d01adcaac423fb2bc8371c1ba6e5bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - ACTIONet (R package)
  - ACTIONetExperiment
  - Leiden
  - R
  - UMAP
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-rna-seq-multi-dataset-integration

## Summary

Integrate multiple independent scRNA-seq studies into a joint analysis using ACTIONet's Harmony-kernel batch-corrected reduction, then transfer cell-type labels across datasets via network-based label propagation and consolidate them into consensus gene signatures.

## When to use

Use when several independently generated scRNA-seq datasets (different labs, platforms, or batches) must be merged so cells group by biological cell type rather than by study of origin.

## When NOT to use

- Analyzing a single dataset with no batch/study structure to correct — plain ACTIONet without the Harmony-kernel reduction suffices
- When the goal is differential-state or activity analysis within one condition rather than cross-study integration and label harmonization
- When input is already a reduced/embedded feature representation rather than raw gene-by-cell counts

## Inputs

- Multiple independent scRNA-seq count matrices (e.g., three human prefrontal cortex studies: Velmeshev 52,556 cells; Mathys 35,140 cells; Schirmer 17,403 cells)
- Per-dataset batch/study labels for Harmony-style correction in reduce_kernel
- Optional reference cell-type annotations in one dataset to propagate to the others

## Outputs

- Joint 2D ACTIONet embedding and multiresolution network grouping cells by cell type across all input studies
- Propagated cell-type labels for every cell in the joint dataset under a consensus vocabulary
- Nonredundant gene signature set per cell type/subtype (e.g., cortex_gene_signatures.csv) with markers such as LINGO2, CUX2, SST, SLC1A2, MBP, CLDN5

## How to apply

Merge the per-dataset count matrices into one joint matrix (e.g., Velmeshev 52,556 + Mathys 35,140 + Schirmer 17,403 = 105,099 cells) and preprocess each with ACTIONet's filter_adata (min_cells_per_feature=0.01, min_features_per_cell=1000), library-size normalization (normalize_total), and log1p transformation. Run ACTIONet's randomized-SVD-based reduce_kernel with Harmony-style batch correction on the merged data, then build the adaptive k*-NN multiresolution network, project it with the UMAP SGD-based 2D layout, and cluster archetypes with the Leiden algorithm. Confirm that network groups correspond to cell types (neuronal vs glial) rather than to study of origin, then transfer cell-type labels across datasets by network-based label propagation over the shared ACTIONet graph. Consolidate propagated annotations into a consensus vocabulary and extract nonredundant gene signatures per cell type/subtype (e.g., neuronal markers LINGO2, CUX2, SST; glial markers SLC1A2, MBP, CLDN5), written to cortex_gene_signatures.csv.

## Related tools

- **ACTIONet (R package)** (Performs filter_adata preprocessing, Harmony-kernel reduce_kernel reduction, multiresolution network construction, label propagation, and signature extraction on the merged dataset) — https://github.com/shmohammadi86/ACTIONet
- **UMAP** (SGD-based 2D projection of the multiresolution network for visual inspection of batch mixing)
- **Leiden** (Graph clustering of archetypes/patterns used to define cell-type groups in the joint network)
- **ACTIONetExperiment** (AnnData-like SummarizedExperiment container for storing and manipulating the merged count data) — https://github.com/shmohammadi86/ACTIONetExperiment

## Examples

```
devtools::install_github("shmohammadi86/ACTIONet", ref = "R-release"); an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000); an.pp.reduce_kernel(adata)
```

## Evaluation signals

- 2D embedding/network groups correspond to cell types (neuronal vs glial) rather than to study of origin — annotated groups mix cells from all three studies
- Each propagated signature discriminates its assigned cell type/subtype in the joint embedding
- Filtering and preprocessing thresholds match the method defaults: min_cells_per_feature=0.01, min_features_per_cell=1000, normalize_total, log1p
- Consensus vocabulary resolves per-study label conflicts; signatures are nonredundant across cell types/subtypes
- Cell totals reconcile: merged joint dataset contains 105,099 cells from the 52,556 + 35,140 + 17,403 inputs

## Limitations

- Datasets cited only by reference numbers (refs 29–31) without accessions or download links — acquisition of the cortex inputs is left to the practitioner
- No software/tool versions stated for ACTIONet, R, Python, UMAP, or Leiden, limiting reproducibility pinning
- Batch correction depends on the Harmony-kernel reduction in reduce_kernel; without it, groups may separate by batch instead of cell type

## Evidence

- [abstract] Harmony-kernel integration of three cortex datasets: "We use ACTIONet to integrate and annotate cells across three human cortex data sets"
- [methods] Preprocessing pipeline parameters: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [abstract] Label propagation and consensus signature definition: "we define a consensus vocabulary and a consistent set of gene signatures discriminating against the transcriptomic cell types and subtypes of the human prefrontal cortex"
- [discussion] Dataset composition and scale: "Data for the joint analysis of the human prefrontal cortex was obtained from three independent studies29–31 and includes in total 105,099 cells"
- [methods] Installation entry point for the R implementation: "devtools::install_github("shmohammadi86/ACTIONet", ref = "R-release")"
