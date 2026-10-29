---
name: cell-type-label-propagation-across-datasets
description: Use when multiple independent scRNA-seq datasets (e.g., three cortex
  studies totaling ~105,099 cells) are jointly integrated and one or more reference
  annotations must be propagated to unlabeled cells without re-clustering from scratch.
when_to_use_negative:
- Datasets have no shared biological populations or the reference annotations themselves
  are unreliable — propagation will faithfully transfer wrong labels
- A single dataset is being analyzed with no cross-study transfer needed; plain ACTIONet
  clustering and marker discovery suffices without the Harmony-kernel reduction and
  propagation step
- Input is already a feature/signature table rather than a raw count matrix — the
  filter/normalize/log1p/reduce_kernel pipeline is not applicable
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ACTIONet (R package)
  role: Hosts filter_adata, reduce_kernel (Harmony-kernel batch correction), multiresolution
    network construction, Leiden clustering, and network-based label propagation
  repo: https://github.com/shmohammadi86/ACTIONet
- name: UMAP
  role: SGD-based projection of the multiresolution network into 2D for validating
    that groups mix cells from all studies
- name: Leiden
  role: Graph clustering of archetypes over the pattern graph to define nonredundant
    equivalent pattern classes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:d7a153dac47e210473d874a544dc13dd1c60f4e55641f89942d8f3b99ff34e81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - ACTIONet (R package)
  - Leiden
  - R
  - UMAP
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cell-type-label-propagation-across-datasets

## Summary

Transfers cell-type and subtype annotations across independent single-cell datasets using ACTIONet's network-based label propagation over a batch-corrected (Harmony-kernel) joint embedding, then consolidates the propagated labels into a consensus vocabulary with nonredundant gene signatures per cell type.

## When to use

Use when multiple independent scRNA-seq datasets (e.g., three cortex studies totaling ~105,099 cells) are jointly integrated and one or more reference annotations must be propagated to unlabeled cells without re-clustering from scratch.

## When NOT to use

- Datasets have no shared biological populations or the reference annotations themselves are unreliable — propagation will faithfully transfer wrong labels
- A single dataset is being analyzed with no cross-study transfer needed; plain ACTIONet clustering and marker discovery suffices without the Harmony-kernel reduction and propagation step
- Input is already a feature/signature table rather than a raw count matrix — the filter/normalize/log1p/reduce_kernel pipeline is not applicable

## Inputs

- Merged single-cell count matrix from ≥2 independent studies (e.g., 105,099 human prefrontal cortex cells from refs 29–31)
- Per-dataset batch/study labels for Harmony-style kernel correction
- At least one dataset with reference cell-type annotations to propagate

## Outputs

- Propagated cell-type/subtype label per cell across all datasets
- Joint 2D embedding (UMAP SGD layout) of the multiresolution ACTIONet
- Consensus cell-type vocabulary consolidated from propagated annotations
- Nonredundant per-cell-type gene signature table (e.g., cortex_gene_signatures.csv)

## How to apply

Merge the independent count matrices (here: Velmeshev 52,556, Mathys 35,140, Schirmer 17,403 control samples into 105,099 cells), preprocess each with ACTIONet's filter_adata (min_cells_per_feature=0.01, min_features_per_cell=1000), library-size normalization (normalize_total), and log1p transformation. Apply ACTIONet's randomized-SVD reduce_kernel with Harmony-style batch correction, build the adaptive k*-NN multiresolution network, project it with the UMAP SGD-based layout, and cluster archetypes with Leiden. Confirm that network groups separate by cell type (neuronal vs glial) rather than by study of origin — this validates that batch effects are circumvented before propagation. Then run network-based label propagation over the ACTIONet graph to transfer reference cell-type labels across datasets, consolidate propagated annotations into a consensus vocabulary, and extract nonredundant gene signatures per cell type (e.g., LINGO2, CUX2, SST for neurons; SLC1A2, MBP, CLDN5 for glia), writing them to cortex_gene_signatures.csv.

## Related tools

- **ACTIONet (R package)** (Hosts filter_adata, reduce_kernel (Harmony-kernel batch correction), multiresolution network construction, Leiden clustering, and network-based label propagation) — https://github.com/shmohammadi86/ACTIONet
- **UMAP** (SGD-based projection of the multiresolution network into 2D for validating that groups mix cells from all studies)
- **Leiden** (Graph clustering of archetypes over the pattern graph to define nonredundant equivalent pattern classes)

## Examples

```
an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000); sc.pp.normalize_total(adata); sc.pp.log1p(adata); an.pp.reduce_kernel(adata)
```

## Evaluation signals

- In the joint embedding, each Leiden/UMAP group must mix cells from all three studies rather than separating by batch
- Propagated groups must correspond to cell type/subtype (neuronal vs glial) rather than to study of origin (Fig. 5a)
- Each extracted gene signature (e.g., LINGO2/CUX2/SST for neuronal, SLC1A2/MBP/CLDN5 for glial) must discriminate its assigned cell type/subtype in the joint embedding
- The signature set is nonredundant — equivalent patterns are pruned via Leiden clustering of the pattern graph, keeping the lowest-resolution pattern per equivalent class
- Preprocessing invariants: filter_adata thresholds (min_cells_per_feature=0.01, min_features_per_cell=1000) applied before normalize_total and log1p, then reduce_kernel

## Limitations

- No software/tool versions are stated for ACTIONet, R, Python, UMAP, or Leiden in the article (only CMake 3.19 is specified for the build)
- The three cortex datasets are cited only by reference numbers without accessions or download links, complicating reproduction
- Success depends on batch correction (Harmony-kernel reduce_kernel) sufficiently removing study-of-origin structure; if groups separate by batch, propagation is invalid

## Evidence

- [abstract] integration and annotation across studies: "We use ACTIONet to integrate and annotate cells across three human cortex data sets"
- [abstract] consensus vocabulary and signatures: "we define a consensus vocabulary and a consistent set of gene signatures discriminating against the transcriptomic cell types and subtypes of the human prefrontal cortex"
- [discussion] dataset scope: "Data for the joint analysis of the human prefrontal cortex was obtained from three independent studies29–31 and includes in total 105,099 cells"
- [methods] preprocessing parameters: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [discussion] batch-corrected reduction step: "an efficient randomized SVD-based low-rank approximation (reduction step) that enables scaling with input data size"
- [discussion] nonredundant pattern pruning: "We then applied the Leiden graph-clustering algorithms over this pattern graph to define equivalent patterns classes"
- [methods] installation command: "devtools::install_github("shmohammadi86/ACTIONet", ref = "R-release")"
