---
name: cell-type-gene-signature-definition
description: Use when cell types have been annotated across merged datasets (e.g.,
  after Harmony-kernel integration and label propagation) and a unified marker-gene
  signature per type/subtype is needed, such as neuronal (LINGO2, CUX2, SST) and glial
  (SLC1A2, MBP, CLDN5) markers.
when_to_use_negative:
- Cell-type labels have not yet been propagated/integrated across datasets — the signatures
  are defined on the consensus annotation, not on per-study clusters
- Input is a single dataset without batch structure to reconcile — the consensus-vocabulary
  step exists to merge annotations across independent studies
- The goal is de novo marker discovery from raw counts without an integrated cell-state
  decomposition (e.g., ACTIONet archetypes/network) in place
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ACTIONet (R package)
  role: Framework for archetypal decomposition, network construction, label propagation,
    and extraction of nonredundant gene signatures per annotated cell type
  repo: https://github.com/shmohammadi86/ACTIONet
- name: ACTIONet
  role: Computational environment (R and Python) implementing the multiresolution
    cell-state decomposition underlying annotation and marker extraction
  repo: https://github.com/shmohammadi86/ACTIONet
- name: ACTIONetExperiment
  role: SummarizedExperiment-style data container for storing the ACE object consumed
    by ACTIONet during annotation and signature extraction
  repo: https://github.com/shmohammadi86/ACTIONetExperiment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:9d944bd7627a4673191868efd903de4d85f9437b3f1c703252e50670114bab24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - ACTIONet (R package)
  - ACTIONetExperiment
  - R
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cell-type-gene-signature-definition

## Summary

Consolidates network-propagated cell-type annotations from multiple integrated scRNA-seq datasets into a consensus vocabulary and extracts a nonredundant set of gene signatures per cell type/subtype, demonstrated on the ACTIONet joint analysis of 105,099 human prefrontal cortex cells.

## When to use

Use when cell types have been annotated across merged datasets (e.g., after Harmony-kernel integration and label propagation) and a unified marker-gene signature per type/subtype is needed, such as neuronal (LINGO2, CUX2, SST) and glial (SLC1A2, MBP, CLDN5) markers.

## When NOT to use

- Cell-type labels have not yet been propagated/integrated across datasets — the signatures are defined on the consensus annotation, not on per-study clusters
- Input is a single dataset without batch structure to reconcile — the consensus-vocabulary step exists to merge annotations across independent studies
- The goal is de novo marker discovery from raw counts without an integrated cell-state decomposition (e.g., ACTIONet archetypes/network) in place

## Inputs

- Merged single-cell count matrix from multiple datasets (e.g., three human prefrontal cortex studies, 105,099 cells in total)
- Preprocessed/normalized expression matrix (filter_adata with min_cells_per_feature=0.01 and min_features_per_cell=1000, normalize_total, log1p)
- ACTIONet multiresolution network with adaptive k*-NN structure and UMAP/Leiden-derived cell-type annotations propagated across studies

## Outputs

- Consensus cell-type vocabulary reconciled across the three source studies
- Nonredundant gene signatures per cell type/subtype (e.g., neuronal: LINGO2, CUX2, SST; glial: SLC1A2, MBP, CLDN5)
- Signature file, e.g., cortex_gene_signatures.csv

## How to apply

After merging datasets (e.g., Velmeshev 52,556 + Mathys 35,140 + Schirmer 17,403 = 105,099 cells) and running ACTIONet's reduce_kernel with Harmony-style batch correction, propagate cell-type labels across datasets via network-based label propagation on the ACTIONet graph. Consolidate the propagated annotations into a single consensus vocabulary, then extract per cell type/subtype a nonredundant set of gene signatures from the archetype/network structure, e.g., neuronal markers LINGO2, CUX2, SST and glial markers SLC1A2, MBP, CLDN5. The ACTIONet decomposition supports this because each cell carries a full vector of archetype weights rather than a single label, so markers follow from the same decomposition used for annotation. Write the signatures to a file (e.g., cortex_gene_signatures.csv) for downstream reuse.

## Related tools

- **ACTIONet (R package)** (Framework for archetypal decomposition, network construction, label propagation, and extraction of nonredundant gene signatures per annotated cell type) — https://github.com/shmohammadi86/ACTIONet
- **ACTIONet** (Computational environment (R and Python) implementing the multiresolution cell-state decomposition underlying annotation and marker extraction) — https://github.com/shmohammadi86/ACTIONet
- **ACTIONetExperiment** (SummarizedExperiment-style data container for storing the ACE object consumed by ACTIONet during annotation and signature extraction) — https://github.com/shmohammadi86/ACTIONetExperiment

## Examples

```
devtools::install_github("shmohammadi86/ACTIONet", ref = "R-release")
```

## Evaluation signals

- Each extracted signature discriminates its assigned cell type/subtype within the joint (Harmony-kernel) embedding
- Annotated cell-type groups mix cells from all three source studies rather than separating by batch
- The signature set is nonredundant: redundant archetypes are pruned, keeping the lowest-resolution pattern per Leiden-defined equivalent class
- Neuronal vs glial top-level groups are recoverable with the stated markers (e.g., LINGO2/CUX2/SST vs SLC1A2/MBP/CLDN5) and reconcile the per-study annotation vocabularies

## Limitations

- Datasets are cited only by reference numbers without accessions or download links, complicating reproduction
- No software/tool versions stated for ACTIONet, R, Python, UMAP, or Leiden implementations (only CMake 3.19 mentioned)
- Archetypal patterns require robustness checks because the number of nontrivial columns must plateau (Supplementary Fig. 1), and no changelog is provided in the supplementary information

## Evidence

- [abstract] Consensus vocabulary and gene signatures (abstract): "we define a consensus vocabulary and a consistent set of gene signatures discriminating against the transcriptomic cell types and subtypes of the human prefrontal cortex"
- [abstract] Integration-and-annotation finding (abstract): "We use ACTIONet to integrate and annotate cells across three human cortex data sets"
- [readme] Archetype-based decomposition supports markers and annotations (readme): "downstream analysis works on continuous cell state: markers, annotations, and comparisons across datasets all follow from the same decomposition"
- [methods] Filtering thresholds preceding signature extraction (methods): "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [discussion] Redundant-pattern pruning underlying nonredundant signatures (discussion): "We then applied the Leiden graph-clustering algorithms over this pattern graph to define equivalent patterns classes"
- [discussion] ACTIONet AA implementation (discussion): "ACTIONet uses (1) an ACTION-based implementation of AA for sparse matrix representation"
