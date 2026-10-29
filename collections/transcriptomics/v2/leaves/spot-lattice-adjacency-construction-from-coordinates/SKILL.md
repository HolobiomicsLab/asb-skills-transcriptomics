---
name: spot-lattice-adjacency-construction-from-coordinates
description: Use when generating automated scribbles via AutoScribbleDom and you need
  neighbor adjacency to label-spot consensus-filter an initial non-spatial Mclust
  clustering on Visium or ST platform data.
when_to_use_negative:
- Input data is not from Visium or ST array platforms — the six-neighbor hexagonal
  and four-neighbor square grids are specific to those technologies ('technology'
  can be visium/st for this pipeline).
- You already have expert-generated scribbles (manual_scribble.csv via Loupe browser)
  and 'schema' is set to 'expert' — no adjacency-based auto-scribble filtering is
  needed.
tools:
- name: Mclust
  role: Provides the initial non-spatial cluster labels whose agreement with lattice
    neighbors is tested by the consensus filter built on this adjacency
  repo: https://github.com/1alnoman/ScribbleDom
- name: ScribbleDom
  role: Pipeline consuming the lattice adjacency to keep only spots whose labels match
    all lattice neighbors and clear the rest, yielding mclust_backbone.csv
  repo: https://github.com/1alnoman/ScribbleDom
- name: R
  role: Language in which the preprocessing scripts (get_genex_data_from_10x_h5.R,
    get_genex_data_from_rds_ST_data.R) run Mclust and produce the coordinates the
    lattice is built from
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:c63016ac9bf15a1945d408c829a7212fabab1eb79616d39aa1f2d9cd630767e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1alnoman/ScribbleDom
  tools:
  - Mclust
  - R
  - ScribbleDom
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# spot-lattice-adjacency-construction-from-coordinates

## Summary

Builds the spot-neighbor lattice used by AutoScribbleDom's consensus filter: a six-neighbor hexagonal grid for Visium data or a four-neighbor square grid for ST data, derived from spot coordinates produced during preprocessing.

## When to use

Use when generating automated scribbles via AutoScribbleDom and you need neighbor adjacency to label-spot consensus-filter an initial non-spatial Mclust clustering on Visium or ST platform data.

## When NOT to use

- Input data is not from Visium or ST array platforms — the six-neighbor hexagonal and four-neighbor square grids are specific to those technologies ('technology' can be visium/st for this pipeline).
- You already have expert-generated scribbles (manual_scribble.csv via Loupe browser) and 'schema' is set to 'expert' — no adjacency-based auto-scribble filtering is needed.

## Inputs

- preprocessed spot coordinates from preprocessed_data/{dataset}/{sample}/
- preprocessed spot principal components (n_pcs, e.g. 15)
- config_mclust.json with 'technology' set to 'visium' or 'st'

## Outputs

- spot lattice adjacency (six-neighbor hex for Visium, four-neighbor square for ST) feeding the AutoScribbleDom consensus filter
- consensus-filtered labels ultimately written to mclust_backbone.csv as the automated scribble

## How to apply

Load the preprocessed spot coordinates and principal components written to preprocessed_data/{dataset}/{sample}/ after running the R preprocessing script (Rscript get_genex_data_from_10x_h5.R config_mclust.json for Visium, get_genex_data_from_rds_ST_data.R for ST). Read the 'technology' field from config_mclust.json, which can be 'visium' or 'st', to select the grid topology: a six-neighbor hexagonal lattice for Visium spots versus a four-neighbor square lattice for ST spots. Connect each spot to its immediate lattice neighbors in the coordinate space, then hand the adjacency to the AutoScribbleDom consensus filter, which retains a spot's Mclust label only if it matches all of its lattice neighbors. The consensus-filtered labels are ultimately written to mclust_backbone.csv as the automated scribble.

## Related tools

- **Mclust** (Provides the initial non-spatial cluster labels whose agreement with lattice neighbors is tested by the consensus filter built on this adjacency) — https://github.com/1alnoman/ScribbleDom
- **ScribbleDom** (Pipeline consuming the lattice adjacency to keep only spots whose labels match all lattice neighbors and clear the rest, yielding mclust_backbone.csv) — https://github.com/1alnoman/ScribbleDom
- **R** (Language in which the preprocessing scripts (get_genex_data_from_10x_h5.R, get_genex_data_from_rds_ST_data.R) run Mclust and produce the coordinates the lattice is built from)

## Examples

```
Rscript get_genex_data_from_10x_h5.R config_mclust.json
```

## Evaluation signals

- mclust_backbone.csv exists in preprocessed_data/{dataset}/{sample}/ and contains a label column covering the retained spots
- Sample identifiers in mclust_backbone.csv match the input spot coordinates
- Lattice degree matches platform topology: each interior Visium spot has 6 neighbors, each interior ST spot has 4 neighbors
- Spots retained in mclust_backbone.csv agree with Mclust labels of all their lattice neighbors; spots with any disagreeing neighbor are cleared
- Preprocessing completed the upstream steps: top 2000 highly variable genes, n_pcs principal components, and Mclust run with n_cluster_for_auto_scribble (e.g. 2)

## Limitations

- Grid topology is hard-coded to two platforms; other spatial transcriptomics technologies are not supported by this pipeline ('technology': This can be visium/st for this pipeline).
- The consensus filter discards border/disagreement spots entirely, so scribble coverage is partial and depends on how well Mclust labels align with spatial neighborhoods.
- If manual_scribble.csv is not generated, results come from AutoScribbleDom only, inheriting any Mclust misclustering propagated through the lattice filter.

## Evidence

- [readme] lattice topology depends on platform: "4. "technology" : This can be visium/st for this pipeline."
- [readme] preprocessing produces coordinates consumed by the lattice and auto-scribble: "This step will produce preprocessed data with principal components of spots, coordinates and mclust_backbone.csv as Automated Scribble"
- [other] six-neighbor hex vs four-neighbor square grids: [evidence span withheld — anchor status: not_found]
- [other] consensus filter semantics over neighbors: [evidence span withheld — anchor status: not_found]
- [methods] Mclust provides the labels being filtered: "Also runs ```Mclust``` algorithm to create a automated scribble."
