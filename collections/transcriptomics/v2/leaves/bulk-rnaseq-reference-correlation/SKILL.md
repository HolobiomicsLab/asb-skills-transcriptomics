---
name: bulk-rnaseq-reference-correlation
description: Use when a single-cell pattern/decomposition method (ACTIONet archetypes,
  cNMF components) must be benchmarked against labeled bulk cell-sorted RNA-seq profiles
  to test whether learned cell states match known cell-type signatures.
when_to_use_negative:
- No labeled bulk sorted-cell reference profiles exist for the tissue/cell types under
  study — correlation cannot be interpreted without known ground-truth profiles.
- The goal is to select the number of decomposition components rather than to validate
  already-learned patterns against known profiles — this skill compares fixed pattern
  sets, not component-number selection.
- Single-cell data have not been identically preprocessed across methods being compared
  (e.g., cNMF and ACTIONet must run on the same filtered/normalized matrix) — correlation
  differences would be confounded.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ACTIONet (Python package)
  role: Produces the multiresolution archetypal pattern profiles whose correlation
    against bulk references is computed; filter_adata, reduce_kernel, and run_ACTIONet
    prepare and decompose the 14,311-cell PBMC matrix.
  repo: https://github.com/shmohammadi86/ACTIONet
- name: Leiden
  role: Graph-clustering over the pattern graph to prune redundant archetypes, keeping
    the lowest-resolution pattern per equivalent class before correlation.
- name: cNMF
  role: Consensus NMF baseline run with k=12 on the same preprocessed matrix; its
    pattern profiles are correlated against the same bulk references for comparison.
- name: scanpy
  role: Preprocessing of the PBMC scRNA-seq matrix (normalize_total, log1p) shared
    by both decompositions prior to validation.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:539e4ee2c8c3d1747c34e4fcfe14fd3c54fab106c25d80cfb128904c5549156f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - ACTIONet (Python package)
  - Leiden
  - cNMF
  - scanpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bulk-rnaseq-reference-correlation

## Summary

Validates single-cell decomposition outputs (e.g., ACTIONet archetype profiles or cNMF patterns) by computing correlations between the learned pattern-profile matrices and labeled bulk cell-sorted RNA-seq reference profiles, as done to benchmark ACTIONet against cNMF on PBMC data (Fig. 2c).

## When to use

Use when a single-cell pattern/decomposition method (ACTIONet archetypes, cNMF components) must be benchmarked against labeled bulk cell-sorted RNA-seq profiles to test whether learned cell states match known cell-type signatures.

## When NOT to use

- No labeled bulk sorted-cell reference profiles exist for the tissue/cell types under study — correlation cannot be interpreted without known ground-truth profiles.
- The goal is to select the number of decomposition components rather than to validate already-learned patterns against known profiles — this skill compares fixed pattern sets, not component-number selection.
- Single-cell data have not been identically preprocessed across methods being compared (e.g., cNMF and ACTIONet must run on the same filtered/normalized matrix) — correlation differences would be confounded.

## Inputs

- Single-cell pattern-profile matrix from ACTIONet's multiresolution archetypal decomposition (dominant archetypes, e.g., 14 nonredundant patterns for 14,311 PBMC cells)
- Pattern-profile matrix from cNMF run on the same preprocessed matrix (k=12 via its suggested protocol)
- Labeled cell-sorted bulk PBMC RNA-seq reference profiles (ref. 27)

## Outputs

- Two pattern-vs-bulk correlation similarity matrices (ACTIONet and cNMF against labeled bulk profiles)
- Comparison table of correlations (e.g., fig2c_similarity_matrix.csv) enabling qualitative reproduction of Fig. 2c

## How to apply

Run the single-cell decomposition on the same preprocessed matrix for each method under comparison — e.g., ACTIONet on a 14,311-cell PBMC dataset (after filter_adata with min_cells_per_feature=0.01, min_features_per_cell=1000, normalize_total and log1p), and cNMF with k fixed at 12 per its suggested protocol. Prune redundant ACTIONet archetypes by Leiden graph-clustering over the pattern graph, keeping the lowest-resolution pattern per equivalent class to obtain a nonredundant pattern-profile set. Load the labeled cell-sorted bulk PBMC RNA-seq reference profiles (ref. 27), compute correlation of both the ACTIONet and cNMF pattern-profile matrices against the labeled bulk profiles, and assemble the two pattern-vs-bulk similarity matrices for side-by-side comparison (e.g., saved as fig2c_similarity_matrix.csv). Correct application is judged by whether the method's recovered profiles correlate more closely with known bulk cell-type profiles — ACTIONet showed higher correlations for heterogeneous myeloid cells and higher-resolution T-cell subtypes such as CD4 memory cells, while cNMF showed less cell-state heterogeneity and lower consistency.

## Related tools

- **ACTIONet (Python package)** (Produces the multiresolution archetypal pattern profiles whose correlation against bulk references is computed; filter_adata, reduce_kernel, and run_ACTIONet prepare and decompose the 14,311-cell PBMC matrix.) — https://github.com/shmohammadi86/ACTIONet
- **Leiden** (Graph-clustering over the pattern graph to prune redundant archetypes, keeping the lowest-resolution pattern per equivalent class before correlation.)
- **cNMF** (Consensus NMF baseline run with k=12 on the same preprocessed matrix; its pattern profiles are correlated against the same bulk references for comparison.)
- **scanpy** (Preprocessing of the PBMC scRNA-seq matrix (normalize_total, log1p) shared by both decompositions prior to validation.)

## Examples

```
an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000); sc.pp.normalize_total(adata); sc.pp.log1p(adata); an.pp.reduce_kernel(adata); H = an.run_ACTIONet(adata)
```

## Evaluation signals

- ACTIONet profile correlations with labeled bulk cell-sorted profiles exceed cNMF correlations for the same cell types, especially heterogeneous myeloid cells and T-cell subtypes such as CD4 memory cells.
- Both methods' similarity matrices are computed against the identical labeled bulk reference set, making correlation values directly comparable.
- The ACTIONet pattern set is nonredundant: Leiden clustering over the pattern graph has merged redundant archetypes, keeping one lowest-resolution pattern per equivalent class (14 dominant patterns for the PBMC data).
- The comparison table (e.g., fig2c_similarity_matrix.csv) qualitatively reproduces the expected ordering of Fig. 2c.

## Limitations

- cNMF requires a priori selection of the number of patterns (k fixed at 12 via its suggested protocol), which constrains its recovered heterogeneity compared with ACTIONet's multiresolution decomposition.
- Validation depends entirely on the quality and coverage of the labeled bulk cell-sorted reference (ref. 27); cell states absent from the reference cannot be evaluated.
- Datasets are cited only by reference numbers without accessions or download links, complicating exact reproduction.

## Evidence

- [discussion] ACTIONet recapitulates expected bulk profiles more precisely: [evidence span withheld — anchor status: not_found]
- [discussion] Leiden pruning of redundant archetypes: "We then applied the Leiden graph-clustering algorithms over this pattern graph to define equivalent patterns classes"
- [intro] cNMF a priori component selection limitation: "A priori selection of the number of underlying dominant patterns to be learned is required but not trivial to determine"
- [methods] Preprocessing parameters from methods: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
