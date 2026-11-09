---
name: spatial-neighborhood-consensus-filtering-of-spot-labels
description: Use when expert scribbles are unavailable for Visium/ST spatial transcriptomics
  data and an automated scribble (mclust_backbone.csv) must be derived from an initial
  non-spatial Mclust clustering.
when_to_use_negative:
- An expert-generated manual scribble (manual_scribble.csv) is available — then run
  with schema 'expert' instead of AutoScribbleDom's mclust schema
- The spatial platform is neither Visium nor ST, since the filter assumes a six-neighbor
  hexagonal or four-neighbor square spot lattice
- A fully labeled per-spot map is required — the filter intentionally clears labels
  on non-consensus spots, producing a partial scribble
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Mclust
  role: Gaussian mixture model that produces the initial non-spatial cluster labels
    which the consensus filter refines into the automated scribble
  repo: https://github.com/1alnoman/ScribbleDom
- name: ScribbleDom
  role: Pipeline that implements the consensus filter (six-neighbor Visium / four-neighbor
    ST lattice) and writes mclust_backbone.csv
  repo: https://github.com/1alnoman/ScribbleDom
- name: R
  role: Environment for running the preprocessing scripts (get_genex_data_from_10x_h5.R
    / get_genex_data_from_rds_ST_data.R) that invoke Mclust and generate the backbone
    file
  repo: https://github.com/1alnoman/ScribbleDom
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:54c2e3ae4b61e59506c71b754c1fdf83bddd1a77e39ffbfa40724005ed79c0c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1alnoman/ScribbleDom
  tools:
  - Mclust
  - R
  - ScribbleDom
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Spatial neighborhood consensus filtering of spot labels

## Summary

A consensus filter in the ScribbleDom/AutoScribbleDom pipeline that converts a non-spatial Mclust clustering into a spatially coherent automated scribble by retaining a spot's label only when it agrees with all of its lattice neighbors, then clearing labels elsewhere.

## When to use

Use when expert scribbles are unavailable for Visium/ST spatial transcriptomics data and an automated scribble (mclust_backbone.csv) must be derived from an initial non-spatial Mclust clustering.

## When NOT to use

- An expert-generated manual scribble (manual_scribble.csv) is available — then run with schema 'expert' instead of AutoScribbleDom's mclust schema
- The spatial platform is neither Visium nor ST, since the filter assumes a six-neighbor hexagonal or four-neighbor square spot lattice
- A fully labeled per-spot map is required — the filter intentionally clears labels on non-consensus spots, producing a partial scribble

## Inputs

- Preprocessed spot principal components (n_pcs, e.g. 15) under preprocessed_data/{dataset}/{sample}/
- Spot spatial coordinates for lattice adjacency construction
- config_mclust.json (n_pcs, n_cluster_for_auto_scribble, technology=visium/st, dataset, samples, schema=mclust)
- Raw count matrix from Space Ranger output (.h5 for Visium, .rds for ST)

## Outputs

- mclust_backbone.csv in preprocessed_data/{dataset}/{sample}/ containing spot-level labels retained after the neighbor-consensus filter (labels cleared on non-consensus spots)

## How to apply

First run Mclust on the spot principal components (configured via n_pcs, e.g. 15 in config_mclust.json) with the number of clusters set by n_cluster_for_auto_scribble (e.g. 2) to get an initial non-spatial label for every spot. Build the spot lattice adjacency from the spot coordinates: a six-neighbor hexagonal grid for Visium data or a four-neighbor square grid for ST data. Apply the consensus filter: keep a spot's scribble label only if it matches the Mclust labels of all of its lattice neighbors; clear labels on spots whose neighbors disagree. Write the retained spot-level labels to mclust_backbone.csv under preprocessed_data/{dataset}/{sample}/ as the automated scribble. The rationale is that requiring full neighborhood agreement yields a spatially consistent, high-precision scribble even though the underlying Mclust clustering ignores spatial location.

## Related tools

- **Mclust** (Gaussian mixture model that produces the initial non-spatial cluster labels which the consensus filter refines into the automated scribble) — https://github.com/1alnoman/ScribbleDom
- **ScribbleDom** (Pipeline that implements the consensus filter (six-neighbor Visium / four-neighbor ST lattice) and writes mclust_backbone.csv) — https://github.com/1alnoman/ScribbleDom
- **R** (Environment for running the preprocessing scripts (get_genex_data_from_10x_h5.R / get_genex_data_from_rds_ST_data.R) that invoke Mclust and generate the backbone file) — https://github.com/1alnoman/ScribbleDom

## Examples

```
Rscript get_genex_data_from_10x_h5.R config_mclust.json
```

## Evaluation signals

- mclust_backbone.csv exists under preprocessed_data/{dataset}/{sample}/
- The CSV contains a label column covering the retained spots, and spot identifiers match the input spot coordinates
- Every retained spot's label equals the Mclust label of all of its lattice neighbors (6 neighbors for Visium, 4 for ST); non-consensus spots have cleared labels
- The number of Mclust clusters used equals n_cluster_for_auto_scribble from config_mclust.json
- Clustering features are the configured n_pcs principal components of the top 2000 highly variable genes

## Limitations

- Coverage is partial by design: spots whose lattice neighbors disagree have their labels cleared, so the automated scribble annotates only spatially consistent spots
- Only Visium (hexagonal, six-neighbor) and ST (square, four-neighbor) technologies are supported by this pipeline
- Filter quality depends on the Mclust initialization, whose cluster count (n_cluster_for_auto_scribble, e.g. 2) must be chosen per dataset

## Evidence

- [methods] Also runs ```Mclust``` algorithm to create a automated scribble.: "Also runs ```Mclust``` algorithm to create a automated scribble."
- [readme] This step will produce preprocessed data with principal components of spots, coordinates and mclust_backbone.csv as Automated Scribble: "This step will produce preprocessed data with principal components of spots, coordinates and mclust_backbone.csv as Automated Scribble in the folder below:"
- [abstract] Notably, when the expert input is absent, ScribbleDom can still operate, in a fully unsupervised manner: "Notably, when the expert input is absent, ScribbleDom can still operate, in a fully unsupervised manner like the state-of-the-art methods, and produces results that remain competitive."
- [readme] n_cluster_for_auto_scribble : This field is used for automatic scribble generation.: ""n_cluster_for_auto_scribble" : This field is used for automatic scribble generation. Give number of cluster for mclust initialization of your data."
