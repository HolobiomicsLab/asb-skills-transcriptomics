---
name: leiden-pattern-graph-pruning
description: Use when a multiresolution archetypal decomposition (e.g., ACTIONet run_ACTIONet
  output) yields many overlapping archetypes across resolutions and you need a nonredundant
  set of cell-state patterns for downstream comparison or annotation.
when_to_use_negative:
- When patterns come from a fixed-k decomposition (e.g., cNMF with a priori k=12)
  that already produces a single nonredundant set rather than a multiresolution stack
  of redundant archetypes.
- When the input is a clustering of cells rather than archetypal patterns — Leiden
  here is applied over the pattern graph, not the cell-cell graph used for cell clustering.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Leiden
  role: Graph-clustering algorithm applied over the pattern graph to define equivalent
    pattern classes
- name: ACTIONet (Python package)
  role: Provides the multiresolution ACTION decomposition whose redundant archetypes
    are pruned via the pattern graph
  repo: https://github.com/shmohammadi86/ACTIONet
- name: ACTIONet (R package)
  role: R interface to the same ACTIONet framework whose pattern graph pruning yields
    the nonredundant archetype set
  repo: https://github.com/shmohammadi86/ACTIONet/tree/R-release
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:78762071a118816868db56897293f5bccbd4e200cfbae506e45b19f0775c5565
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - ACTIONet (Python package)
  - ACTIONet (R package)
  - Leiden
  - cNMF
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# Leiden pattern graph pruning

## Summary

A network-based feature-selection step from ACTIONet that removes redundant archetypal patterns from a multiresolution decomposition: the Leiden graph-clustering algorithm is run over the pattern graph to define equivalent pattern classes, and one nonredundant representative pattern is kept per class.

## When to use

Use when a multiresolution archetypal decomposition (e.g., ACTIONet run_ACTIONet output) yields many overlapping archetypes across resolutions and you need a nonredundant set of cell-state patterns for downstream comparison or annotation.

## When NOT to use

- When patterns come from a fixed-k decomposition (e.g., cNMF with a priori k=12) that already produces a single nonredundant set rather than a multiresolution stack of redundant archetypes.
- When the input is a clustering of cells rather than archetypal patterns — Leiden here is applied over the pattern graph, not the cell-cell graph used for cell clustering.

## Inputs

- Multiresolution archetypal decomposition output from an.run_ACTIONet (archetype-specific pattern profiles across all resolution levels, e.g., 14 dominant patterns on the PBMC dataset)
- Pattern graph (nodes = archetypes, edges = pairwise pattern similarity/adjacency)

## Outputs

- Nonredundant set of cell-state pattern profiles (one lowest-resolution pattern per Leiden equivalent class)
- Pattern-to-class assignment from the Leiden clustering over the pattern graph
- Pruned pattern-profile matrix used for downstream correlation (e.g., fig2c_similarity_matrix.csv of pattern-vs-bulk similarities)

## How to apply

After running ACTIONet's multiresolution ACTION decomposition (e.g., on the 14,311-cell PBMC dataset after an.pp.filter_adata, sc.pp.normalize_total, sc.pp.log1p, and an.pp.reduce_kernel), construct the pattern graph where nodes are archetypes identified across all resolution levels and edges encode pattern similarity. Apply the Leiden graph-clustering algorithm over this pattern graph to partition archetypes into equivalent pattern classes. For each equivalent class, keep the pattern with the lowest resolution, discarding higher-resolution near-duplicates, to obtain a nonredundant pattern-profile matrix. This step exists because the ACTION decomposition returns 14 dominant patterns on the PBMC data with redundancy across resolutions, and pruning yields the compact set used for correlation against bulk cell-sorted reference profiles (outperforming cNMF's 12 a priori patterns in matching myeloid and CD4 memory T-cell profiles).

## Related tools

- **Leiden** (Graph-clustering algorithm applied over the pattern graph to define equivalent pattern classes)
- **ACTIONet (Python package)** (Provides the multiresolution ACTION decomposition whose redundant archetypes are pruned via the pattern graph) — https://github.com/shmohammadi86/ACTIONet
- **ACTIONet (R package)** (R interface to the same ACTIONet framework whose pattern graph pruning yields the nonredundant archetype set) — https://github.com/shmohammadi86/ACTIONet/tree/R-release

## Examples

```
an.run_ACTIONet(adata); # then Leiden-prune the pattern graph: for each equivalent class keep the lowest-resolution pattern
```

## Evaluation signals

- Each Leiden equivalent class contributes exactly one retained pattern, so the retained pattern count equals the number of nontrivial classes and is smaller than the total archetype count across resolutions.
- Retained patterns should correspond to the lowest-resolution member of each class, verifiable by comparing resolution indices before and after pruning.
- On the 14,311-cell PBMC dataset, the pruned ACTIONet set (14 dominant patterns) should show higher correlations with bulk cell-sorted PBMC reference profiles than cNMF's 12 patterns, especially for heterogeneous myeloid cells and CD4 memory T-cell subtypes (Fig. 2c qualitative pattern).
- The pruned pattern-profile matrix should remain a valid gene-by-pattern profile matrix aligned with the filtered/normalized gene space (post filter_adata with min_cells_per_feature=0.01, min_features_per_cell=1000).

## Limitations

- Redundancy pruning relies on the pattern graph structure; if the ACTION decomposition plateaus in the number of nontrivial columns (Supplementary Fig. 1 referenced but not included in provided text), the choice of classes may be sensitive to graph construction.
- No software/tool versions are stated for the Leiden implementation or ACTIONet in the article, so exact reproducibility of class boundaries is not guaranteed.
- Keeping the lowest-resolution pattern per class discards higher-resolution variants, which may omit fine-grained substructure within an equivalent class.

## Evidence

- [discussion] Leiden clustering rationale: "We then applied the Leiden graph-clustering algorithms over this pattern graph to define equivalent patterns classes (patterns clusters)."
- [discussion] Keeping lowest-resolution pattern per class: "a network-based feature selection method to robustly identify a nonredundant subset of underlying cell-state patterns"
- [methods] Preprocessing parameters before decomposition: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [methods] Decomposition step feeding the pattern graph: "an.pp.reduce_kernel(adata)"
- [readme] README framing of redundancy pruning: "Redundant archetypes found at different resolutions are
pruned and merged, leaving a non-redundant set that spans the dataset."
