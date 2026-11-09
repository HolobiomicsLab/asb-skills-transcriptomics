---
name: archetypal-analysis-resolution-sweep
description: Use when a single-cell dataset must be decomposed without a priori choosing
  one number of archetypes, and the goal is to compare pattern-capture quality across
  resolutions (k=2..30) against a pruned multiresolution decomposition.
when_to_use_negative:
- When the goal is discrete clustering with a fixed, known number of cell populations
  — multiresolution decomposition targets continuous cell states, not a single optimal
  k
- When the analysis target is sparse scATAC-seq peak data; the article's ATACction
  extension reduces peaks first (ACTION/chromVAR/LSI), so the expression pipeline
  parameters (min_features_per_cell=1000) do not apply directly
- When no curated marker panel exists for the expected cell types, since capture-rate
  computation depends on marker-enrichment tests
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ACTIONet
  role: Framework combining archetypal analysis and manifold learning; runs multiresolution
    decomposition, annotation, and network construction
  repo: https://github.com/shmohammadi86/ACTIONet
- name: ACTIONet (Python package)
  role: Python interface used for filter_adata, reduce_kernel, run_ACTIONet, load_markers,
    and annotate_cells_using_markers
  repo: https://github.com/shmohammadi86/ACTIONet
- name: Scanpy
  role: Reads 10x h5 counts (sc.read_10x_h5) and performs normalize_total and log1p
    preprocessing
- name: ACTION
  role: Reference C++ implementation of the archetypal analysis numerical core (kernel
    reduction, successive projection, constrained regressions)
  repo: https://github.com/shmohammadi86/ACTION
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:fab315b131fbad81fca816e4806c629652ad93579ce85e11af1932377c59fcb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTION
  - ACTIONet
  - ACTIONet (Python package)
  - Scanpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# archetypal-analysis-resolution-sweep

## Summary

Runs ACTIONet's archetypal analysis at every resolution k=2..30 plus the multiresolution (pruned/unified) decomposition on a scRNA-seq dataset, then quantifies how well each cell type is captured per resolution via marker-enrichment capture rates, revealing whether multiresolution decomposition preserves both fine- and coarse-grain cell-type patterns that a single high resolution loses.

## When to use

Use when a single-cell dataset must be decomposed without a priori choosing one number of archetypes, and the goal is to compare pattern-capture quality across resolutions (k=2..30) against a pruned multiresolution decomposition.

## When NOT to use

- When the goal is discrete clustering with a fixed, known number of cell populations — multiresolution decomposition targets continuous cell states, not a single optimal k
- When the analysis target is sparse scATAC-seq peak data; the article's ATACction extension reduces peaks first (ACTION/chromVAR/LSI), so the expression pipeline parameters (min_features_per_cell=1000) do not apply directly
- When no curated marker panel exists for the expected cell types, since capture-rate computation depends on marker-enrichment tests

## Inputs

- 10x Genomics scRNA-seq count matrix in HDF5 format (e.g., pbmc_10k_v3.h5)
- Curated cell-type marker panel (e.g., PBMC_Monaco2019_12celltypes with markers, directions, names)
- AnnData (scanpy) or ACTIONetExperiment (ACE) object holding genes x cells counts

## Outputs

- Single-level archetypal decompositions at every resolution k=2..30 (archetype profiles H and cell loadings C)
- Multiresolution pruned/unified decomposition with nonredundant archetype set
- Per-cell type labels and annotation confidences from marker-based annotation
- Per-cell-type capture-rate table across resolutions with baselines (e.g., capture_rates_fig2ab.csv)

## How to apply

Load the raw count matrix (e.g., 10x pbmc_10k_v3.h5 via scanpy's sc.read_10x_h5, making variable names unique), then filter with an.pp.filter_adata(min_cells_per_feature=0.01, min_features_per_cell=1000), normalize library sizes with sc.pp.normalize_total, and log-transform with sc.pp.log1p. Run an.pp.reduce_kernel for the randomized SVD-based low-rank approximation, then an.run_ACTIONet to obtain single-level archetypal decompositions at every resolution k=2..30 and the multiresolution (pruned/unified) decomposition in which Leiden clustering over the pattern graph keeps the lowest-resolution nonredundant archetype per equivalent class. Annotate cells using a curated marker panel such as PBMC_Monaco2019_12celltypes via an.tl.load_markers and an.tl.annotate_cells_using_markers to obtain per-cell type labels and confidences. For each cell type, compute the capture rate (log-transformed Welch t-test p value of marker enrichment) at each single-level resolution, under the multiresolution decomposition, and under two baselines (highest-resolution k=30 and the average across resolutions); logFC comparisons of capture rates across cell types and resolutions reproduce the article's Fig. 2a,b analysis.

## Related tools

- **ACTIONet** (Framework combining archetypal analysis and manifold learning; runs multiresolution decomposition, annotation, and network construction) — https://github.com/shmohammadi86/ACTIONet
- **ACTIONet (Python package)** (Python interface used for filter_adata, reduce_kernel, run_ACTIONet, load_markers, and annotate_cells_using_markers) — https://github.com/shmohammadi86/ACTIONet
- **Scanpy** (Reads 10x h5 counts (sc.read_10x_h5) and performs normalize_total and log1p preprocessing)
- **ACTION** (Reference C++ implementation of the archetypal analysis numerical core (kernel reduction, successive projection, constrained regressions)) — https://github.com/shmohammadi86/ACTION

## Examples

```
import scanpy as sc, ACTIONet as an; adata = sc.read_10x_h5('pbmc_10k_v3.h5'); adata.var_names_make_unique(); an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000); sc.pp.normalize_total(adata); sc.pp.log1p(adata); an.pp.reduce_kernel(adata); an.run_ACTIONet(adata)
```

## Evaluation signals

- Marker-enrichment capture rate (log-transformed Welch t-test p value) is computable for every cell type at every resolution k=2..30
- Multiresolution capture rates match or exceed both baselines (highest-resolution k=30 and average across resolutions) for most cell types
- Low-variability cell types (e.g., NK cells in PBMC) lose capture rate at high single resolutions but remain captured under the multiresolution decomposition
- The unified archetype set is nonredundant: Leiden clustering of the pattern graph yields equivalent classes with one retained pattern each
- Annotation confidences from an.tl.annotate_cells_using_markers are consistent across resolutions for well-separated types

## Limitations

- The number of nontrivial archetypes plateaus at high resolutions, so additional resolutions yield diminishing returns (Supplementary Fig. 1 referenced but not included in the provided text)
- No software/tool versions stated for ACTIONet, R, Python, UMAP, or Leiden implementations, so reproducibility requires pinning; CMake 3.19 or newer is the only stated version requirement
- Datasets cited by reference numbers without accessions or download links; the PBMC data set was adopted from a profile of 14,311 single cells

## Evidence

- [discussion] Multiresolution balances fine- and coarse-grain capture: [evidence span withheld — anchor status: not_found]
- [methods] filter_adata parameters: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [methods] marker panel loading: "markers, directions, names = an.tl.load_markers('PBMC_Monaco2019_12celltypes')"
- [methods] 10x input read: "adata = sc.read_10x_h5('pbmc_10k_v3.h5')
adata.var_names_make_unique(join='.')"
- [discussion] nonredundant archetype pruning via Leiden: "We then applied the Leiden graph-clustering algorithms over this pattern graph to define equivalent patterns classes"
- [intro] single optimal k is not biologically justified: "the existence of multiple meaningful levels of resolution that prohibit the specification of a single"
- [readme] continuous states via archetype mixtures: "Redundant archetypes found at different resolutions are pruned and merged, leaving a non-redundant set that spans the dataset."
